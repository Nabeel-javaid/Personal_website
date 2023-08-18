---
title: "When “SafeMint” Becomes Unsafe:"
description: "Unveiling the Perils: The Transformation of SafeMint from Secure to Risky"
pubDate: "Aug 19 2022"
heroImage: "/Reentrancy.png"
---


## Table of Contents

1. [Introduction](#introduction)
2. [The Root Cause](#the-root-cause)
3. [The Attack](#the-attack)
4. [Lessons](#lessons)

## Introduction
On the morning of Feb 3rd (+8 timezone), an attack was reported on the transaction <a href="https://etherscan.io/tx/0xfa97c3476aa8aeac662dae0cc3f0d3da48472ff4e7c55d0e305901ec37a2f704">0xfa97c3476aa8aeac662dae0cc3f0d3da48472ff4e7c55d0e305901ec37a2f704 </a>towards the HypeBears NFT contract. After the investigation, it was found that it’s a re-entrancy attack caused by the `_safeMint` function of ERC721.

## The root cause
The project has a limitation of the NTFs that an account can mint. Basically, it has a map `addressMinted` that logs whether an account has minted the NTFs.

When minting NFTs, the code uses `_safeMint` function of the OZ reference implementation. This function is safe because it checks whether the receiver can receive ERC721 tokens. The can prevent the case that a NFT will be minted to a contract that cannot handle ERC721 tokens. According to the <a href="https://docs.openzeppelin.com/contracts/4.x/api/token/erc721">document:</a>
```solidity
  function _safeMint(address to, uint256 tokenId, bytes memory data) internal virtual {
        _mint(to, tokenId);
        _checkOnERC721Received(address(0), to, tokenId, data);
    }
```

However, this external function call creates a security loophole. Specifically, the attacker can perform a reentrant call inside the `onERC721Received` callback. For instance, in the <a href="https://etherscan.io/address/0x14e0a1f310e2b7e321c91f58847e98b8c802f6ef#code"> vulnerable HypeBears contract</a>, the attacker can invoke the  `mintNFT` function again in the `onERC721Received` callback (since 1addressMinted` has not been updated.)

<img src="/safemint.webp" class="hero-image"> 


## The attack
The following screenshot shows the <a href="https://etherscan.io/tx/0xfa97c3476aa8aeac662dae0cc3f0d3da48472ff4e7c55d0e305901ec37a2f704">attack transaction.</a>


<img src="/safemint_attack.webp" class="hero-image"> 


## Lessons
The risk called by `SafeMint` has been discussed by security researchers. However, we can still see the vulnerable code and the attack in the wild.using a safe function does not guarantee a safe contract.