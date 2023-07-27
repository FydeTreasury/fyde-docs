---
title: Governance module
author: Fyde  
date: 2022-02-07
category: Jekyll
layout: post
---

## Main overview

The role of the governance module is to allow depositors to exercise voting rights associated with their deposited governance tokens. This feature is aimed at users with concentrated token holdings who want to benefit from Fyde's diversification and liquidity while still actively participating in decision making (e.g. whales, VCs, founders). Each user of Fyde's governance has their own proxy contract which holds the governance token and can vote on their behalf. The amount of governance token a user is entitled to is equivalent to the USD value of their TRSY holdings.

## Smart contract overview

- GovernanceModule.sol: main contract and entry point for all governance related calls.
- CloneFactory.sol: Deploys sTRSY-govToken and minimal proxies according to the cloning pattern. Is inherited by GovernanceModule.sol
- ProxyRouter.sol: Logic for access authorization and retrieval of implementation address for minimal proxy. Is inherited by GovernanceModule.sol
- sTRSY.sol: ERC20 contract for staked TRSY. A dedicated sTRSY exists for each underlying governance token (e.g. sTRSY-UNI) and allows holders to exercise voting rights.
- UserProxy.sol: Implementation contract for the governance proxies. Currently allows for on-chain and Snapshot voting. Can be upgraded to include more governance mechanisms.

## Governance functionalities

### unstakeGov
Converts sTRSY 1:1 to TRSY which can be sold or used to withdraw tokens from Fyde. This process is IRREVERSIBLE and should only be called when user is willing to give up their voting rights indefinitely.

### rebalanceProxy
Redistributes token between proxies and standard pool to make sure user gets the exact amount of votes they are entitled to. Should be called regularly for users who are active participants in their tokens governance.

### delegateVotingRights
For governance tokens that implement ERC20Votes. Delegate voting rights from the proxy to a delegate of choice. This could be a user's EOA or multisig and allows them to participate in a project's governance without interacting with Fyde at all - as if they were holding the tokens themselves.

### setDelegate/clearDelegate (Snapshot)
For snapshot spaces that implement the SnapshotDelegation voting strategy. Allows to delegate voting rights on snapshot to address of choice. The delegate can then vote off-chain on the snapshot website.

### SnapshotVoting
For all snapshot spaces. Allows users to vote on snapshot proposals via the Fyde website. This is a purely off-chain process, but the validity of the vote is confirmed by a user's proxy via ERC-1271.

## Upgradability
 In order to keep pace with the evolving landscape of DAO governane, the contract implementing voting functions is upgradable following a modified beacon proxy pattern. The proxies that hold the token are minimal proxies which get the implementation address from the governance module. They delegate call to UserProxy which implements the logic for on-chain voting and integration with Snapshot. When new governance functions are added in the future, the implementation address can be changed by Fyde's developer team. To prevent malicious upgrades, a user's proxy is deactivated until they approve of the upgrade.

## Voting Process

The governance related functions delegateVotingRights, setDelegate/clearDelegate are directly called on the GovernanceModule.sol contract. While on the backend token are send to a user's proxy and the voting function call are forwarded to the respective proxy, these processes are abstracted away from the user.

## Proxy Rebalancing

Holders of sTRSY-govToken can use voting rights of the underlying govToken, equivalent to the USD value of the sTRSY. Since the price of TRSY and a specific governance token fluctuate, the amount of tokens/votes a user is entitled to changes over time. This means tokens need to be redistributed between proxies and the standard pool. It is the responsibility of the user to trigger the rebalancing of their proxy if they don't have their full voting rights assigned.

Example: User deposits 100 tokenA at a price of 1$ into Fyde and wants to keep governance rights. At the time of deposit the price of TRSY is 1$, so they receive 100 sTRSY-tokenA. At this time they can use 100 votes for tokenA governance.
Over time the prices of tokenA stays constant while TRSY increases to 1.20$. Since the $ equivalent of 100 sTRSY-tokenA is now 120 tokenA, user should call rebalanceProxy in order to get 120 tokenA assigned. 
If TRSY then drops to 0.5$ user would be entitled to 50 tokenA votes, but has 120 assigned. As long as no one else is claiming these tokens, user may keep additional voting rights. If another is underweight and calls rebalanceProxy, tokens will be transferred to them.
When rebalancing the tokens are taken from other users that have to many votes or from Fyde's non-governance pool. If due to strong price fluctuations there are temporarily not enough token in the protocol, the amount of token will be distributed to sTRSY-tokenA holders according to their fair share.
Rebalancing is automatically triggered by functions that change a user's sTRSY balance. These are unstakeGov, withdrawGov, sTRSY.transfer.


<img src="{{site.baseurl}}/illustrations/GovRebalancing.png">