---
title: "Lending/Borrowing DeFi Attacks"
description: "Dive into the vulnerabilities of DeFi lending/borrowing platforms. Learn about potential attacks, from collateral manipulation to unexpected inputs, and explore strategies to enhance security and protect investments."
pubDate: "Aug 13 2023"
heroImage: "/image.webp"
---

<!-- <style>
.hero-image {
  border: 3px solid black;
  padding: 4px;
  max-width: 100%;
  height: auto;
}
</style>

<img src="/defi-attacks.jpg" alt="DeFi Attacks" class="hero-image"> -->

 ## Table of Contents

- [Liquidation Before Default](#liquidation-before-default)
- [Borrower Can’t Be Liquidated](#borrower-cant-be-liquidated)
- [Debt Closed Without Repayment](#debt-closed-without-repayment)
- [Repayments Paused While Liquidations Enabled](#repayments-paused-while-liquidations-enabled)
- [Token Disallow Stops Existing Repayment & Liquidation](#token-disallow-stops-existing-repayment--liquidation)
- [Borrower Immediately Liquidated After Repayments Resume](#borrower-immediately-liquidated-after-repayments-resume)
- [Liquidator Takes Collateral With Insufficient Repayment](#liquidator-takes-collateral-with-insufficient-repayment)
- [Repayment Sent to Zero Address](#repayment-sent-to-zero-address)
- [Borrower Repayment Only Partially Credited](#borrower-repayment-only-partially-credited) 





## Liquidation Before Default
Liquidation allows a Borrower’s collateral to be seized and either given to the Lender as compensation or paid to a liquidator (or shared in some manner between them). Liquidation should only be possible if:

<li> The Borrower has failed to meet their repayment schedule obligations, by being late on a scheduled repayment, </li>

<li> The value of the Borrower’s collateral has fallen below a set threshold </li>


If the Lender, Liquidator or another market participant can liquidate a Borrower’s collateral before the Borrower is in default, this results in a critical loss of funds vulnerability for the Borrower. Consider this simplified example from  <a href="https://app.sherlock.xyz/audits/contests/62">Sherlock’s TellerV2 audit contest</a>

```solidity
function lastRepaidTimestamp(Loan storage loan) internal view returns (uint32) {
    return
        // @audit if no repayments have yet been made, lastRepaidTimestamp()
        // will return acceptedTimestamp - time when loan was accepted
        loan.lastRepaidTimestamp == 0
            ? loan.acceptedTimestamp
            : loan.lastRepaidTimestamp;
}

function canLiquidateLoan(uint loanId) public returns (bool) {
    Loan storage loan = loans[loanId];

    // Make sure loan cannot be liquidated if it is not active
    if (loan.state != LoanState.ACCEPTED) return false;

    return (uint32(block.timestamp) - lastRepaidTimestamp(loan) > paymentDefaultDuration);
    // @audit if no repayments have been made:
    // block.timestamp - acceptedTimestamp > paymentDefaultDuration
    // doesn't check paymentCycleDuration (when next payment is due)
    // if paymentDefaultDuration < paymentCycleDuration, can be liquidated
    // *before* first payment is due. If paymentDefaultDuration is very small,
    // can be liquidated very soon after taking loan, way before first payment
    // is due!
}
```

`canLiquidateLoan()` doesn’t check when the next repayment is due; if the loan is new and the first repayment hasn’t been made (as it won’t be due for some time “paymentCycleDuration”), the Borrower can be liquidated before their first repayment is due if paymentDefaultDuration < paymentCycleDuration.

If paymentDefaultDuration is small, the Borrower could be liquidated very soon after taking the loan! The liquidation threshold paymentDefaultDuration should always be calculated as an offset from when the next repayment is due; only once the next repayment is late by paymentDefaultDuration should the Borrower be able to be liquidated.

Another example of this vulnerability comes from Hats Finance Tempus Raft audit contest. Here an attacker can <a href="https://github.com/tempusfinance/raft-contracts/issues/323">pass a different or zero value for collateral to liquidate a Borrower</a> who should not be subject to liquidation:
<p>This is also an example of an <a href="https://dacian.me/exploiting-developer-assumptions#heading-unexpected-empty-inputs">Unexpected/Unchecked Input Vulnerability</a>.</p>
<p>More examples: [<a href="https://code4rena.com/reports/2022-12-backed/#h-04-users-may-be-liquidated-right-after-taking-maximal-debt">1</a>, <a href="https://code4rena.com/reports/2022-04-abranft/#h-03-critical-oracle-manipulation-risk-by-lender">2</a>, <a href="https://code4rena.com/reports/2022-04-abranft/#h-04-lender-is-able-to-seize-the-collateral-by-changing-the-loan-parameters">3</a>, <a href="https://github.com/sherlock-audit/2023-02-blueberry-judging/issues/126">4</a>, <a href="https://github.com/sherlock-audit/2022-10-union-finance-judging/issues/115">5</a>]</p>



## Borrower Can’t Be Liquidated
Another serious vulnerability occurs if the Borrower can devise a loan offer that results in their collateral not being able to be liquidated. Examine this simplified example also from Sherlock’s TellerV2 audit:

```solidity
// AddressSet from https://github.com/OpenZeppelin/openzeppelin-contracts-upgradeable
// a loan must have at least one collateral
// & only one amount per token is permitted
struct CollateralInfo {
    EnumerableSetUpgradeable.AddressSet collateralAddresses;
    // token => amount
    mapping(address => uint) collateralInfo;
}

// loanId -> validated collateral info
mapping(uint => CollateralInfo) internal _loanCollaterals;

function commitCollateral(uint loanId, address token, uint amount) external {
    CollateralInfo storage collateral = _loanCollaterals[loanId];

    // @audit doesn't check return value of AddressSet.add()
    // returns false if not added because already exists in set
    collateral.collateralAddresses.add(token);

    // @audit after loan offer has been created & validated, borrower can call
    // commitCollateral(loanId, token, 0) to overwrite collateral record 
    // with 0 amount for the same token. Any lender who accepts the loan offer
    // won't be protected if the borrower defaults since there's no collateral
    // to lose
    collateral.collateralInfo[token] = amount;
}
```
<p>This code contains an <a href="https://dacian.me/exploiting-developer-assumptions#heading-unchecked-return-values">unchecked return value vulnerability</a> as the return value of AddressSet.add() is never checked; this will return false if the token is already in the set. As this isn’t checked the code will continue to execute and the existing collateral token’s amount can simply be overwritten with a new value, 0! More examples: [<a href="https://code4rena.com/reports/2022-11-debtdao/#h-05-borrower-can-craft-a-borrow-that-cannot-be-liquidated-even-by-arbiter-">1</a>, <a href="https://code4rena.com/reports/2022-04-abranft/#h-01-avoidance-of-liquidation-via-malicious-oracle">2</a>, <a href="https://code4rena.com/reports/2021-09-wildcredit/#h-02-liquidation-can-be-escaped-by-depositing-a-uni-v3-position-with-0-liquidity">3</a>, <a href="https://github.com/sherlock-audit/2023-02-notional-judging/issues/21">4</a>, <a href="https://github.com/sherlock-audit/2023-03-taurus-judging/issues/61">5</a>, <a href="https://github.com/sherlock-audit/2022-11-isomorph-judging/issues/72">6</a>, <a href="https://code4rena.com/reports/2022-11-paraspace/#h-07-user-can-pass-auction-recovery-health-check-easily-with-flashloan">7</a>, <a href="https://code4rena.com/reports/2022-11-paraspace/#h-04-anyone-can-prevent-themselves-from-being-liquidated-as-long-as-they-hold-one-of-the-supported-nfts">8</a>]</p>

## Debt Closed Without Repayment

<p>Normally to get their collateral back, the Borrower has to repay the Lender their principal + interest. If the Borrower can close the debt without repaying the full amount <em>and</em> keep their collateral, this results in a critical loss of funds vulnerability for the Lender. Examine this code [<a href="https://github.com/debtdao/Line-of-Credit/blob/e8aa08b44f6132a5ed901f8daa231700c5afeb3a/contracts/modules/credit/LineOfCredit.sol#L388-L409">1</a>,<a href="https://github.com/debtdao/Line-of-Credit/blob/e8aa08b44f6132a5ed901f8daa231700c5afeb3a/contracts/modules/credit/LineOfCredit.sol#L483-L507">2</a>] from <a href="https://code4rena.com/reports/2022-11-debtdao/#h-04-borrower-can-close-a-credit-without-repaying-debt">code4rena’s DebtDAO audit</a>:</p>

```solidity
// amount of open credit lines on a Line of Credit facility
uint256 private count; 

// id -> credit line provided by a single Lender for a given token on a Line of Credit
mapping(bytes32 => Credit) public credits; 

// @audit attacker calls close() with non-existent id
function close(bytes32 id) external payable override returns (bool) {
    // @audit doesn't check that id exists in credits, if it doesn't
    // exist an empty Credit with default values will be returned
    Credit memory credit = credits[id];

    address b = borrower; // gas savings
    // @audit borrower attacker will pass this check
    if(msg.sender != credit.lender && msg.sender != b) {
      revert CallerAccessDenied();
    }

    // ensure all money owed is accounted for. Accrue facility fee since prinicpal was paid off
    credit = _accrue(credit, id);
    uint256 facilityFee = credit.interestAccrued;
    if(facilityFee > 0) {
      // only allow repaying interest since they are skipping repayment queue.
      // If principal still owed, _close() MUST fail
      LineLib.receiveTokenOrETH(credit.token, b, facilityFee);

      credit = _repay(credit, id, facilityFee);
    }

    // @audit _closed() called with empty credit, non-existent id
    _close(credit, id); // deleted; no need to save to storage

    return true;
}

function _close(Credit memory credit, bytes32 id) internal virtual returns (bool) {
    if(credit.principal > 0) { revert CloseFailedWithPrincipal(); }

    // return the Lender's funds that are being repaid
    if (credit.deposit + credit.interestRepaid > 0) {
        LineLib.sendOutTokenOrETH(
            credit.token,
            credit.lender,
            credit.deposit + credit.interestRepaid
        );
    }

    delete credits[id]; // gas refunds

    // remove from active list
    ids.removePosition(id);

    // @audit calling with non-existent id still decrements count, can
    // keep calling close() with non-existent id until count decremented to 0
    // and loan marked as repaid!
    unchecked { --count; }

    // If all credit lines are closed the the overall Line of Credit facility is declared 'repaid'.
    if (count == 0) { _updateStatus(LineLib.STATUS.REPAID); }

    emit CloseCreditPosition(id);

    return true;
}
```

<p>The Borrower can simply call close() with a non-existent id, and every call will end up decrementing count. Doing this until count == 0 results in the loan being marked as repaid! This is also an example of the <a href="https://dacian.me/exploiting-developer-assumptions#heading-unexpected-empty-inputs">unexpected empty inputs vulnerability</a>, where the developer is not expecting a non-existent value to be passed so hasn’t correctly handled that. More examples: [<a href="https://code4rena.com/reports/2022-03-timeswap/#h-01-wrong-timing-of-check-allows-users-to-withdraw-collateral-without-paying-for-the-debt">1</a>, <a href="https://github.com/sherlock-audit/2023-03-taurus-judging/issues/11">2</a>, <a href="https://github.com/sherlock-audit/2022-10-astaria-judging/issues/233">3</a>]</p>

## Repayments Paused While Liquidations Enabled

<p>Lending &amp; Borrowing DeFi platforms should never be able to enter a state where <a href="https://github.com/sherlock-audit/2023-02-blueberry-judging/issues/290">repayments are paused but liquidations are enabled</a>, since this would unfairly prevent Borrowers from making their repayments while still allowing them to be liquidated. If repayments can be paused then liquidations must also be paused at the same time. Examining the <a href="https://github.com/sherlock-audit/2023-02-blueberry/blob/main/contracts/BlueBerryBank.sol#L740-L747">repay()</a> function from Sherlock’s <a href="https://app.sherlock.xyz/audits/contests/41">Blueberry contest</a> shows that repayments can be turned on/off, but there is no similar check within <a href="https://github.com/sherlock-audit/2023-02-blueberry/blob/main/contracts/BlueBerryBank.sol#L511-L516">liquidate()</a>.</p>

```solidity
function liquidate(uint256 positionId, address debtToken, uint256 amountCall) 
    external override lock poke(debtToken) {

function repay(address token, uint256 amountCall)
    external override inExec poke(token) onlyWhitelistedToken(token) {
    if (!isRepayAllowed()) revert REPAY_NOT_ALLOWED();
```

<p>Developers of Lending &amp; Borrowing platforms should ensure that if repayments are paused then liquidations must also be paused, and auditors should examine whether this invariant can be violated. More examples: [<a href="https://github.com/sherlock-audit/2022-11-isomorph-judging/issues/69">1</a>]</p>

## Token Disallow Stops Existing Repayment & Liquidation

<p>Some Lending &amp; Borrowing platforms allow governance to disallow accepting previously allowed tokens, either repayment or collateral tokens. If this also stops existing loans using that token from being repaid or liquidated, this can result in a critical loss of funds vulnerability for the Lender and/or the protocol.</p>

<p>BlueBerry addressed the previous ”<a href="https://dacian.me/lending-borrowing-defi-attacks#heading-repayments-paused-while-liquidations-enabled">repayments revert but liquidations allowed</a>” issue by adding the same isRepayAllowed() call into liquidate(), such that the two functions now looked like this:</p>

```solidity
function liquidate(uint256 positionId, address debtToken, uint256 amountCall) 
    external override lock poke(debtToken) {
    if (!isRepayAllowed()) revert Errors.REPAY_NOT_ALLOWED();

function repay(address token, uint256 amountCall) 
    external override inExec poke(token) onlyWhitelistedToken(token) {
    if (!isRepayAllowed()) revert Errors.REPAY_NOT_ALLOWED();
```

<p>In Sherlock’s <a href="https://app.sherlock.xyz/audits/contests/69">BlueBerry Update 1 contest</a> after completing the initial version of this Deep Dive I <a href="https://github.com/sherlock-audit/2023-04-blueberry-judging/issues/4">discovered</a> another way to reach the same state that was missed in the original audit contest. In this alternate path repayments are never paused but instead a previously allowed token is disallowed. The inconsistent usage of the onlyWhitelistedToken() modifier results in a Borrower with an existing position not being able to repay, but still being able to be liquidated.</p>

<p>Governance disallowing of previously allowed tokens should only apply to new loans but existing loans using the disallowed tokens must continue to be able to be repaid and liquidated. More examples: [<a href="https://github.com/sherlock-audit/2022-11-isomorph-judging/issues/57">1</a>, <a href="https://github.com/sherlock-audit/2023-02-gmx-judging/issues/168">2</a>]</p>


## Borrower Immediately Liquidated After Repayments Resume

Let us re-examine the code in Sherlock’s Blueberry Update 1 contest and we’ll also remove the inconsistent onlyWhitelistedToken modifier per the recommendation from the previous section:

```solidity
function liquidate(uint256 positionId, address debtToken, uint256 amountCall) 
    external override lock poke(debtToken) {
    if (!isRepayAllowed()) revert Errors.REPAY_NOT_ALLOWED();

function repay(address token, uint256 amountCall) 
    external override inExec poke(token) {
    if (!isRepayAllowed()) revert Errors.REPAY_NOT_ALLOWED();
```

This code now correctly prevents a Borrower from being liquidated while the Borrower is unable to repay, as pausing repayments also pauses liquidations. If repayments are paused liquidate() will revert, and if a previously allowed token is disallowed, a Borrower with an existing position with that token can continue to repay and be liquidated.

However one more issue remains; if repayments are paused, during that pause market fluctuations can cause a Borrower to become subject to liquidation as the Borrower is unable to repay(). As soon as repayments are resumed, such a Borrower will be immediately liquidated by liquidation bots, with the only possibility to save their position being if the Borrower themselves runs a repayment bot & can successfully front-run the liquidation bots.

<p>This situation unfairly disadvantages Borrowers as such Borrowers became subject to liquidation through no fault of their own. Upon <a href="https://github.com/sherlock-audit/2023-04-blueberry-judging/issues/117">repayments resuming a Borrower will be immediately liquidated</a>, unfairly disadvantaging the Borrower and giving a huge advantage to the Liquidator.</p>

To fix the game theory such that neither Borrowers nor Liquidators are unfairly favored, after repayments are resumed there should be a grace period during which Borrowers can’t be liquidated. This grace period could be equal to the period that repayments were paused with a hard cap of a max number of hours, which provides even fairness to both Borrowers & Liquidators.

## Liquidator Takes Collateral With Insufficient Repayment

When the Borrowers is in default, two things can happen:

<li> Lender liquidates the Borrower by forgoing repayment of the loan and seizing the collateral, </li>

<li> Liquidator repays the Borrower and seizes the collateral </li>

<p>In the second case, advanced platforms allow a Liquidator to partially repay the Borrower’s bad debt and receive a proportional amount of the collateral. If the Liquidator can <a href="https://github.com/sherlock-audit/2023-02-blueberry-judging/issues/127">take the collateral with an insufficient (or no) repayment</a>, this represents a critical loss of funds vulnerability for the Lender. Consider this <a href="https://github.com/sherlock-audit/2023-02-blueberry/blob/main/contracts/BlueBerryBank.sol#L511-L531">collateral share calculation</a> from Blueberry’s Sherlock audit:</p>

```solidity
function liquidate(uint256 positionId, address debtToken, uint256 amountCall)
    external override lock poke(debtToken) {
    // checks
    if (amountCall == 0) revert ZERO_AMOUNT();
    if (!isLiquidatable(positionId)) revert NOT_LIQUIDATABLE(positionId);

    // @audit get position to be re-paid by liquidator, however
    // borrower may have multiple debt positions
    Position storage pos = positions[positionId];
    Bank memory bank = banks[pos.underlyingToken];
    if (pos.collToken == address(0)) revert BAD_COLLATERAL(positionId);

    // @audit oldShare & share proportion of the one position being liquidated
    uint256 oldShare = pos.debtShareOf[debtToken];
    (uint256 amountPaid, uint256 share) = repayInternal(
        positionId,
        debtToken,
        amountCall
    );

    // @audit collateral shares to be given to liquidator calculated using
    // share / oldShare which only correspond to the one position being liquidated,
    // not to the total debt of the borrower (which can be in multiple positions)
    uint256 liqSize = (pos.collateralSize * share) / oldShare;
    uint256 uTokenSize = (pos.underlyingAmount * share) / oldShare;
    uint256 uVaultShare = (pos.underlyingVaultShare * share) / oldShare;

    // @audit if the borrower has multiple debt positions, the liquidator
    // can take the whole collateral by paying off only the lowest value
    // debt position, since the shares are calculcated only from the one
    // position being liquidated, not from the total debt which can be
    // spread out across multiple positions
```

share / oldShare is the proportion of the one debt position being paid off by the Liquidator, not the entire debt of the Borrower which can be spread across multiple positions. Hence if the Borrower’s debt is spread across multiple positions, a Liquidator can take all of the collateral by repaying only the smallest debt position.


## Repayment Sent to Zero Address

<p>Care must be taken when implementing the repayment code such that the <a href="https://github.com/sherlock-audit/2023-01-cooler-judging/issues/33">repayment is not lost by sending it to the zero address</a>. Examine this code from Cooler’s Sherlock audit:</p>

```solidity
function repay (uint256 loanID, uint256 repaid) external {
    Loan storage loan = loans[loanID];

    if (block.timestamp > loan.expiry) 
        revert Default();

    uint256 decollateralized = loan.collateral * repaid / loan.amount;

    // @audit loans[loanID] is deleted here
    // which means that loan which points to loans[loanID]
    // will be an empty object with default/0 member values
    if (repaid == loan.amount) delete loans[loanID];
    else {
        loan.amount -= repaid;
        loan.collateral -= decollateralized;
    }

    // @audit loan.lender = 0 due to the above delete
    // hence repayment will be sent to the zero address
    // some erc20 tokens will revert but many will happily
    // execute and the repayment will be lost forever
    debt.transferFrom(msg.sender, loan.lender, repaid);
    collateral.transfer(owner, decollateralized);
}
```

`loan` points to storage loans[loanID], but loans[loanID] is deleted then afterward the repayment is transferred to loan.lender which will be 0 due to the previous deletion. Some ERC20 tokens will revert but many will happily execute causing the repayment to be sent to the zero address and lost forever. More examples: [<a href="https://github.com/sherlock-audit/2022-11-bullvbear-judging/issues/127">1</a>]

## Borrower Repayment Only Partially Credited
Borrowing & Lending systems can allow Borrowers to take out multiple loans. Borrowers can then attempt to repay as much as possible with one call to the repay() function, the idea being that if the repayment amount can pay off the first loan, then any repayment amount should be used to pay off the second loan and so on.

<p>A critical loss of funds error occurs for the Borrower if once the first loan has been paid off, the overflow is not used to at least partially pay off the second loan but the Lender receives the full amount, resulting in the <a href="https://github.com/sherlock-audit/2022-10-astaria-judging/issues/190">Borrower’s repayment only being partially credited</a>.</p>

Developers should test & auditors should verify that bulk repayment functionality does indeed pay off as many of the loans as possible and that none of the repayment amount is lost.