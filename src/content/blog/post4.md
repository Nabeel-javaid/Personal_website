---
title: "What Are Flash Loan Attacks and How to Prevent Them?"
description: "Explore the key pitfalls that smart contract auditors must steer clear of. From overlooked code interactions to inadequate testing, this guide safeguards the integrity of blockchain systems."
pubDate: "Aug 16 2022"
heroImage: "/flashLoan.webp"
---

## Table of Contents

1. [Overstudying](#mistake-1-overstudying)
2. [Overemphasis on CTF’s](#mistake-2-overemphasis-on-ctfs)
3. [Intimidation by Huge Codebases](#mistake-3-intimidation-by-huge-codebases)
4. [Fear of Previously Audited Protocols](#mistake-4-fear-of-previously-audited-protocols)
5. [Lack of Discipline](#mistake-5-lack-of-discipline)

## What Is A Flash Loan Attack?
A Flash Loan Attack refers to an act where individuals exploit smart contracts under Decentralized Finance (DeFi) across the Ethereum network, making them vulnerable to unauthorized modifications or violations in the cryptocurrency market. Here, the attacker takes out a flash loan, uses the contract to manipulate market variables for their cause, makes a profit, and repays the loan within the same transaction.

## Flash Loan Attack Explained
Flash loan attacks are <a href="https://www.wallstreetmojo.com/decentralized-finance/"> DeFi </a> attacks under the pretext of flash loans. They are <a href="https://www.wallstreetmojo.com/unsecured-loans/"> unsecured loans </a>, i.e., given out without collateral and enforced by smart contracts. These loans are a facility Aave provides—it is a well-known decentralized lending DeFi protocol.

The loan, once taken, is settled in the same transaction. These loans are convenient as they are given out instantly without collateral, limits, or <a href="https://www.wallstreetmojo.com/credit/"> credit </a> checks; the only condition is that the <a href="https://www.wallstreetmojo.com/borrower/"> borrower </a> repay them before the transaction concludes. Funds are taken out of the blockchain and returned, and the attacks take place within the time the funds are repaid or returned.

Each <a hef="https://www.wallstreetmojo.com/blockchain/">blockchain </a> has a different execution time. For example, a <a href="https://www.wallstreetmojo.com/bitcoin/">Bitcoin</a> transaction takes 10 to 30 minutes to get confirmed and added to a blockchain. Ethereum has a duration of 13 seconds. If an attacker uses an Ethereum blockchain, the flash loan must be repaid within 13 seconds. In a planned attack, an attacker uses this tiny window of 13 seconds to manipulate certain variables (typically a price-related vulnerability, discrepancy, or alteration) and gain an advantage.

Attackers often manipulate the market to create opportunities for arbitrage trading. Multiple trades of coins and tokens are executed within a tiny timeframe or small window. Contracts are written to exchange large amounts of borrowed coins for other coins while executing flash loan attacks in <a href="https://www.wallstreetmojo.com/cryptocurrency-top/">crypto</a>.

The huge volume increases the demand for the coins bought, and the prices of these coins also increase. Attackers make every possible attempt to benefit from this volatility in a very short span. The coins are then immediately sold to make a profit. The smart contract will reverse all buy/sell activities if the <a href="https://www.wallstreetmojo.com/debt/">debt</a> is not repaid in a single transaction, making it appear like the loan was never availed.

## Examples
Let us study some examples to understand the concept in greater detail.

### Platypus Finance ($8.5 Million)
On February 2023, the Platypus Finance project on the Avalanche network was hit by a flash loan attack, <a href="https://medium.com/@numencyberlabs/platypus-finance-project-hit-by-8-5m-flash-loan-attack-bbc21ac27a2a">resulting in a loss of $8.5 million</a>. The attacker borrowed 44 million USDC from Aave and used it to stake and borrow more funds from the PlatypusTreasure contract. They then called the emergencyWithdraw function of the MasterPlatypusV4 contract to withdraw the staked lp tokens, despite not repaying the borrowed funds.

This allowed the attacker to directly withdraw the stake. The vulnerability was caused by a logic issue in the emergencyWithdraw function, which did not check the user’s status in repaying their borrowed funds.


### Euler Finance ($196 Million)
In March 2023, Euler Finance, a lending protocol, experienced a flash loan attack <a href ="https://medium.com/@numencyberlabs/a-detailed-analysis-of-euler-finances-196-million-flash-loan-attack-81cdef370024">resulting in a loss of approximately $196 million</a>, making it the largest hack of 2023. The attacker exploited a vulnerability in the donateToReserves function of the Etoken, allowing them to generate profit by executing multiple calls with different currencies.

While Euler Finance managed to negotiate the recovery of the tokens from the attacker, the $196 million flash loan attack on the platform serves as a stark reminder of the inherent risks associated with flash loan attacks. It emphasizes the need for DeFi projects to prioritize security and adopt rigorous security measures, including regular vulnerability checks, to minimize the potential for similar attacks in the future.


## How Can We Prevent Flash Loan Attacks?
There are several ways to prevent flash loan attacks from being successful.

## Circuit Breakers
One of the most effective is to implement circuit breakers, which are automated mechanisms that halt trading on a platform if certain conditions are met, such as a sudden drop in liquidity or a large price movement. <strong>Circuit breakers</strong> can prevent flash loan attacks by preventing large price movements from occurring, which can make it more difficult for attackers to manipulate the price of an asset.

## Risk Assessment and Monitoring
Continuously assess the risks associated with flash loan attacks. Stay updated with the latest attack vectors and vulnerabilities. Implement monitoring systems to detect abnormal behaviour or suspicious transactions within the platform.

## Secure Smart Contract Development
Follow best practices for secure smart contract development, such as using well-audited libraries, implementing input validation and sanitization, and avoiding unchecked external calls. Employing formal verification techniques can also help ensure the correctness of the contract code.

## Implement Time Delays
Introduce time delays or cooldown periods before allowing borrowed funds to be used in certain transactions. This can provide an opportunity for manual review or intervention if suspicious activity is detected.

## Code Audits
Conducting thorough security audits of <a href="https://www.numencyber.com/what-is-a-smart-contract/"> smart contracts</a> and protocols with the <a href="https://twitter.com/0xepley">notable and experince</a> smart contract auditorsto identify vulnerabilities and potential attack vectors is also an excellent way to prevent potential flash loan attacks. Independent security firms or specialized auditors can review the code and suggest improvements to make it more resilient against flash loan attacks.