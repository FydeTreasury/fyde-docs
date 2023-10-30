---
title: Escrow state
author: Fyde  
date: 2022-02-08
category: Jekyll
layout: post
---

## Main overview

The escrow contract's purpose is to pool deposits from users and transfer them into the core protocol collectively, either to bootstrap a pool and/or reduce taxes on deposits. As an independent contract, it has no special access and interacts with the core contracts just as any user would.

## Escrow timeline

### Escrow period
The escrow period commences when the contract is deployed. During this period, allowed assets, predefined in the constructor, can be deposited by whitelisted users.

### Freeze period
During the freeze period, the owner of the escrow can define the amounts of tokens to be accepted for deposit into the protocol. Once the amounts are established, the assets are transferred into the Fyde protocol, and the escrow contract retains the corresponding stTRSY/TRSY. Alternatively, the owner can abort the escrow process, resulting in the refund of the deposited tokens to the depositors.

### Claim period
After successfully depositing into Fyde, users can claim their respective shares of stTRSY/TRSY and a proportional refund of the tokens not accepted for deposit. This period persists indefinitely.

## Escrow functions

### deposit
Whitelisted users deposit the maximum amount of tokens they are willing to deposit into the protocol and indicate whether they want to retain governance rights.

### setConcentrationAmounts
Sets the amounts of tokens to be deposited into Fyde. Can be less than the amount deposited into the escrow, in which case the rest will be refunded.

### depositToFyde
Request the deposit of tokens into the protocol via the Relayer's requestDeposit function. Due to gas limitations, this function processes deposits in batches of 5 assets. If there are more than 5 different assets in the escrow, this function should be invoked multiple times.

### updateInternalAccounting
After the deposits have been processed by Fyde, checks how much TRSY was minted for all deposits and updates the accounting accordingly.

### claimAndRefund
Depositors get their respective share of stTRSY/TRSY and token refunds depending on their deposits and amount of accepted tokens.

### refund
Depositors claim their deposit refunds in case of a revoked escrow process.

### revoke
Allows the owner to revoke the escrow process during the freeze period, in cases where they decide not to accept any assets.

### returnStuckFunds
Owner can transfer assets out of the escrow in case of malfunction. Should be removed for the second and trustless version of the protocol.


