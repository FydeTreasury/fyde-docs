---
title: Relayer contract
author: Fyde  
date: 2022-02-02
category: Jekyll
layout: post
---

## Main overview

The role of the relayer is twofold : 
- Firstly, it acts as the entry point for external users to interact with the protocol through request functions such as requestDeposit, requestWithdraw, and requestSwap.
- Secondly, it is utilized for settlement automation in Fyde by monitoring and updating the protocol AUM (Assets Under Management), and managing the process request. Automation is overseen by external keepers like the Gelato Network and our Fyde Keeper bot.


## Why is a relayer necessary? 

In designing our protocol, we opted to denominate assets and TVL (Total Value Locked) in USD, with on-chain pricing. However, calculating TVL requires iterating through n assets, rendering on-chain pricing excessively expensive and unscalable. The keeper's objective is to ascertain the value of protocolAUM through on-chain functions, but as this computation occurs off-chain, the process remains cost-effective.


## Relayer functionalities

### Requesting

Users call the request functions to convey their intended actions, including parameters such as assets, amounts, governance rights retention, and slippage parameters. Subsequently, the request is added to the relayer queue, awaiting processing by a keeper. Users making a request will also transfer some ETH to cover the keeper’s gas fees (refer to the User Flow section for more details).

### Processing

Keepers, currently consisting of the Gelato Bot and a self-created keeper, supervise the relayer queue. When the queue becomes populated, a keeper is activated to process the pending requests. The input of the processing functions is protocolAUM, calculated off-chain.


### AUM monitoring and protection from keeper manipulation


Since the keeper plays a pivotal role by providing essential input for the protocol's operation (protocolAUM), it creates an attack vector. For this reason, we also secure the protocolAUM value within the Fyde contract. When the keeper calls the processRequest function, we ensure the input value aligns within a reasonable threshold to thwart manipulation attacks.

The protocolAUM value is also monitored by off-chain keepers. Should the off-chain value diverge beyond a specific percentage, the keeper is prompted to update the internal value, ensuring a consistent on-chain protocolAUM. However, the keepers' actions are limited, and they can only update the protocolAUM within a coherent range to prevent atomic manipulation and draining of the protocol in a single tx. Consequently, even in scenarios where the Gelato Network is compromised or the Fyde keeper’s private key is stolen, the protocol remains safeguarded against immediate, single-transaction manipulation attacks, giving us a window to respond and suspend the protocol.


### Access control and roles

Relayer inherit access control logic, with the following roles : 
- User : If whitelist activated only specific user can interact, if zero address is whitelisted, it removes the whitelist logic.
- Owner : Can add/remove the roles below (functions are triggered manually, this will be a Gnosis Safe multisig)
Following roles can have multiple addresses assigned : 
- Keeper : Gelato network and Fyde keeper for updatingAUM and processRequest
- Guard : Can pause/unpause the protocol and add asset to quarantine list
- IncentiveManager : Can set swap incentiveFactor on Fyde
  
  

<img src="{{site.baseurl}}/illustrations/AccessControl.svg">



## Quarantine list

In the context of our portfolio management strategy, we may have to quarantine assets based on our risk management strategy. When an asset is quarantined, action for this given asset (deposit, withdraw and swap) are disabled. The quarantine list is managed by the guard role. 
