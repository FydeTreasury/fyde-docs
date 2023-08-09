---
title: Escrow state
author: Fyde  
date: 2022-02-08
category: Jekyll
layout: post
---

## Main overview

The escrow contract's purpose is to pool deposits from users and transfer them into the core protocol at once in order to bootstrap a pool and/or reduces taxes on deposits. As an independent contract, it has no special access and interacts with the core contracts in the same way any user would

## Escrow timeline

### Escrow period
The escrow period starts when the contract is deployed. Allowed assets, which are set in the constructor, can be deposited by whitelisted users during the escrow period.

### Freeze period
During the freeze period the owner of the escrow can set the amounts of tokens accepted to be deposited into the protocol. Once the amounts are set the assets will be transferred into the Fyde protocol and the escrow contract holds the corresponding sTRSY/TRSY. 
Alternatively, the escrow process can be aborted by the owner, after which the deposited tokens is refunded to the depositors.

### Claim period
After successful deposit into Fyde, users can claim their respective share of sTRSY/TRSY and a proportional refund of the tokens not accepted for deposit. This period continues indefinitely.

## Escrow functions

### deposit
Whitelisted users deposit the maximum amount of tokens they are willing to deposit into the protocol and indicate whether they want to retain governance rights.

### setConcentrationAmounts
Sets the amounts of tokens to be deposited into Fyde. Can be less than the amount deposited into the escrow, in which case the rest will be refunded.

### depositToFyde
Request the deposit of tokens into the protocol via the Relayer's requestDeposit function. Due to gas limitations, the function requests deposits in batches of 5 assets. If more than 5 different assets are in the escrow, the function has to be called multiple times.

### updateInternalAccounting
After the deposits have been processed by Fyde, checks how much TRSY was minted for all deposits and updates the accounting accordingly.

### claimAndRefund
Depositors get their respective share of sTRSY/TRSY and token refund depending on their deposits and amount of accepted tokens.

### refund
Depositors claim their deposit refund in case of a revoked escrow process.

### revoke
Revoke the escrow process. Can be called during freeze period by owner in case they don't want to accept any assets.

### returnStuckFunds
Owner can transfer assets out of the escrow in case of malfunction. Should be removed for the second and trustless version of the protocol .


