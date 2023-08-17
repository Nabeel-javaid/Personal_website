---
title: "Most Common Smart Contract Vulnerabilities"
description: "This overview outlines key vulnerabilities in blockchain-based smart contracts, including reentrancy attacks, integer problems, data mishandling, and weak access controls. Addressing these issues is vital for enhancing smart contract security in decentralized systems."
pubDate: "Aug 16 2023"
heroImage: "/3.webp"
---


## Table of Contents

1. [Reentrancy attack](#reentrancy-attack)
2. [Front-running](#front-running)
3. [Integer overflow and underflow](#integer-overflow-and-underflow)
4. [Simple logic error](#simple-logic-error)
5. [Block gas limit vulnerability](#block-gas-limit-vulnerability)
6. [Default visibility](#default-visibility)
7. [Timestamp dependence](#timestamp-dependence)



## Reentrancy attack
Reentrancy is one of the most iconic exploitable smart contract vulnerabilities. It occurs when a smart contract calls another smart contract in its code and, when the new call is finished, continues with execution. This action requires the vulnerable contract to submit an external call.

Scammers steal these external calls and make a recursive call back to the contract with the help of the callback function. They can create a contract at an external address using malicious code.

When the smart contract fails to update its state before sending funds, the scammer can continuously call the withdraw function, thus allowing them to drain the contract funds.

#### Reentrancy attack real-life example

The most famous example of reentrancy is <a href="https://papers.ssrn.com/sol3/papers.cfm?abstract_id=3014782">The DAO attack</a>
that occurred only three months after its launch. An anonymous hacker managed to drain most of the $150M worth of ETH from the DAO’s smart contract over the course of a few weeks. This resulted in the loss of investors’ trust and struck a significant blow to Ethereum’s credibility.

After the attack, the Ethereum community voted to return the network to its original state and shutter the DAO.




## Front-running
Interestingly, smart contracts and transactions become fully public once you submit them to the network as a pending transaction. These transactions are visible to the entire network in the mempools of Ethereum nodes, enabling block miners to select transactions with the highest gas fees.

There is a significant side effect of this visibility. It allows malicious actors to see the intended outcome of a smart contract before it’s confirmed on the blockchain. Imagine you have a smart contract that, when run, will execute an arbitrage that costs 0,04 ETH to deploy. Knowing this information, fraudsters can copy your smart contract and submit it with a higher gas fee. This way they successfully front-run your smart contract and steal your arbitrage opportunity by submitting their transaction first.

Unfortunately, these attacks are difficult to avoid.  Even so, there are a variety of cutting-edge practices that can help you secure your contract. These include gas limiting, which presupposes accepting only transactions with a gas price below the appointed threshold, and also using the pre-commit scheme, which implies submitting a hash instead of your data in the first commit and providing details at a later time.

#### Front-running real-life example

A noteworthy example of a front-running attack is the <a href="https://www.halborn.com/blog/post/explained-the-dodo-dex-hack-march-2021"> DODO DEX hack.</a> During this hack, an original attacker became the victim of two cryptocurrency trading bots. This decreased the impact of the hack as they frontrunned some of the attacker’s attempts to exploit smart contract vulnerabilities. The owners of both cryptocurrency trading bots agreed to return the stolen funds, which totaled $3.1 million, but $700,000 remained stolen by the original attacker.


## Integer overflow and underflow
This smart contract vulnerability is common to many programming languages, including Solidity. A Solidity smart contract is built using 256 bits as the word size, which equates to 4.3 billion Ether. If you reduce the value of an unsigned integer to zero, it will return to the maximum value.

A scammer may exploit the smart contract using a malicious address that is recorded by the smart contract to make a zero balance send 1 unit of Ether. It will force the smart contract’s balance to cycle back to the maximum value allowed (4.3 billion Ether).

As the smart contract believes the address has a balance of 4,3 billion Ether, it may allow withdrawals from that account until the smart contract is drained of funds.

Both underflow and overflow issues cause significant differences between the calculation’s actual outcome and expected results, thus undermining the smart contract’s inherent logic, and leading to the contract’s funds being lost.

A simple measure to avoid this hack is to use the 0.8 version of the Solidity compiler, which automatically checks for underflows and overflows.

#### Integer overflow and underflow real-life example

A good example of underflow and overflow vulnerabilities would be a cryptocurrency Ponzi scheme: <a href ="https://medium.com/@ebanisadr/how-800k-evaporated-from-the-powh-coin-ponzi-scheme-overnight-1b025c33b530" > Proof of Week Hands Coin. </a> The project promised a legitimate pyramid scheme, which quickly gained value of over a million dollars. But in just one night it lost $800K due to arithmetic flaws.

The project’s implementation of ERC-20 allowed a person to approve another user to transfer tokens on their behalf. A malicious actor enabled a second account to sell coins from the first account. However, these coins were taken off the second account’s balance. As a result, integer underflow left the second account with an extremely large balance of PoWH Coins.


##  Simple logic error
Logic errors tend to be one of the most common types of blockchain smart contract vulnerabilities. These may include typographical errors, misinterpretation of specifications, and the more serious programming errors that decrease the security of smart contracts.

The good news is that these problems can be identified and eliminated during the smart contract audit, which is why it is recommended that you do not ignore this step before deploying your smart contracts to the blockchain.

#### Simple logic error real-life example

<a href = "https://www.publish0x.com/interestingcrypto/hegic-case-48-dollars-000-cents-typo-or-why-dofi-protocols-n-xejoomg" > The Hegic case </a> is an interesting example of how a minor typo can cause financial loss. Hegic is a platform that allows users to insure against price volatility options. The platform was forced to restart its protocol when it spotted a simple typo in the code: instead of the “OptionsIDs” function which unlocks liquidity in expired contracts, it had the non-existent “OptionIDs” command, which omitted the letter “s”.

Because of this error, users’ assets were blocked whenever they didn’t use their options, resulting in no liquidity for expired contracts. Fixing this error and providing the affected users with a refund cost Hedic $48K.


## Block gas limit vulnerability
The block gas limit helps ensure that blocks do not grow too large. If a transaction consumes too much gas, it will not fit the block and, ultimately, will not be executed.

The result is a block gas limit vulnerability: if data is serviced in arrays and further accessed through loops over these arrays, the transaction may run out of gas and get a refund. This can lead to a Denial of Service (DoS) attack.

#### Block gas limit vulnerability real-life example

<a href ="https://applicature.com/blog/blockchain-technology/history-of-ethereum-security-vulnerabilities-hacks-and-their-fixes" > GovernMental </a> is yet another failed Ponzi scheme project. To join the project a user was required to send a certain amount of Ether to the contract. At a certain point, the list of project participants grew so long that it would have required more gas to clear the arrays than the maximum amount allowed for a single transaction. From this point onwards, all attempts to clear the arrays have failed.

## Default visibility
Visibility defines whether a function can be called internally or externally by users. The default visibility state for functions is public.

It becomes a problem when smart contract developers do not specify the visibility of functions that should be private or only callable within the contract itself.

#### Default visibility real-life example

The <a href = "https://haseebq.com/a-hacker-stole-31m-of-ether/" >Parity MultiSig Wallet </a> hack occurred as a result of developers accidentally leaving two functions public. The attacker had an opportunity to call these functions and change the ownership to the attacker’s address. This mistake allowed the hacker to steal $31M worth of Ether from three wallets.



## Timestamp dependence
If the smart contract uses the block.timestamp function to display StartTime and EndTime, the malicious miner can manipulate the timestamp for a few seconds and change the output so that it is in their favor. This is why it is not recommended to use the block.timestamp function to get the current time, due to the blockchain’s decentralized nature.

It’s worth mentioning that this vulnerability is serious only if it is used in the critical components of a smart contract. To prevent it, you can either avoid using the block.timestamp function, or allow a range of +900 seconds of error — so if the timestamp value returned by the node is increased by a value between 1 to 900 seconds it will not have a huge impact on the contract.

#### Timestamp dependence real-life example

Check out the smart contract source code for EtherLotto — a lottery game where users are meant to send money to the smart contract function play. The amount of money should be equal to the TICKET_AMOUNT. In any other case, it will fail.

The smart contract retrieves the time when the contract is executed. It goes on to apply a formula to it and services the value in a random variable as shown in line 14. Afterward, it checks if this value is equal to zero and, in the event that it does, the transaction is named the winner.

The smart contract calls the block.timestamp function to get the actual time. The value of this variable is given by the node, meaning that the malicious user can easily manipulate it until it gets the result of line 14 as zero.

These have been the most common smart contract vulnerabilities that can lead to serious issues. Other potential vulnerabilities that are regularly spotted by smart contract auditors, among them our PixelPlex specialists, include:

<li>Irrelevant code
<li>Improper initialization
<li>Improper locking
<li>Uncontrolled resource consumption
<li>Incorrect behavior and business logic
<li>Poor adherence to coding standards
<li>Incorrectly handled exceptions
<li>Incorrect work with ERC-20 tokens
<li>Using the blockchain function
<li>Missing withdraw functions
<li>Using an obsolete function

Nonetheless, you should realize that this list is to all intents and purposes even larger, as hackers are continually searching for novel ways to deceive businesses and users. Therefore, you should take all possible preventative measures to secure your smart contracts and avoid financial loss and reputation damage.

