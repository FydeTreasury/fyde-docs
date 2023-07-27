---
title: Fyde contract
author: Fyde  
date: 2022-02-04
category: Jekyll
layout: post
---

## Main overview

The Fyde contract serves as the core contract of the Fyde protocol. It handles the logic for depositing, withdrawing, and swapping assets within the protocol.

Fyde acts as an index pool that accepts certain assets and with a specific target concentration for each asset,that represents the desired weight of the asset in the protocol relative to the TVL in USD. Target concentrations represent the ideal weight of a given asset in the protocol as part of our portfolio management strategy. Users can deposit, withdraw, and swaps these assets in the Fyde protocol.

### Deposit

Approved assets can be deposited by users, and upon deposit, TRSY is minted to the user. Minted TRSY represents a share of the pool that is proportional to the deposited amount denominated in USD. 

For example, if a user deposits 100 TokenA representing 100k USD and the TVL is 1mm USD with a circulation of 500k TRSY, the user will receive 50k TRSY.


### Withdraw
Users can withdraw assets by burning their TRSY, the value of the withdraw is computed using a similar mechanism than depositing. 

For example, if a user has 50k TRSY and the total value locked (TVL) is 1mm USD, with a total circulation of 500k TRSY, the user withdraw 10% of the index pool and can withdraw the equivalent of 100k USD.


### Swap
Users can swap assetIn for assetOut. During the swap, we exchange the USD value of assetIn for an amount of assetOut with the same USD value. (Note that for swap an incentive factor can be applied as an incentive for swapping given assets).


### Tax 

In the event of an action (deposit, withdraw, swap) that unbalances the target concentration, a tax is applied to penalize these actions in order to maintain the current concentrations close to the portfolio management strategy. 

For example, depositing an asset that will change the concentration towards overweighting concentration will be taxed, same for withdrawing an asset that is underweight.


## Smart contract overview

The Fyde contract is primarily responsible for executing the logic related to depositing and withdrawing funds from the index pool and the governance pool. Additionally, the Fyde contract inherits the logic of the following modules:

<img src="{{site.baseurl}}/illustrations/fyde.png">


- TRSY: an ERC20 used to represent the pool in the form of shares (logic similar to vault ERC-4626)
- AddressRegistry: contains the addresses of contracts that interact with the protocol
- ProtocolState: contains the accounting logic of the protocol, as well as various parameters for the protocol
- GovernanceAccess: functions to connect Fyde with the governanceModule contract
- AssetRegistry: contains the logic for adding assets to the protocol
- Tax: calculates potential taxes on deposits, withdrawals, and swaps.



## Computation of the tax 

### Deposit tax

This system is encapsulated by the following function, which charges no tax if the deposit remains underneath the target concentration of the token pool. A tax proportional to the difference between the current ($C_{i}$) and target concentrations ($C_{i}^0$) will be levied on the portion of assets that are deposited into overweight pools ($D_{i}^a$). This tax is calculated for every pool being deposited into, where $D_{i}$ is the USDC value of each individual deposit and $T_{D}$ is the sum of all deposits.

\begin{equation}
D^a_i = \min\left(\max\left(D_i + T_{VL}(C_i - C^0_i) - T_DC^0_i,0\right),D_i\right)\ .
\end{equation}

The total value of the tax is computed from each deposit, then subtracted from the number of TRSY tokens minted. The total tax amount (denominated in USD) is written as follows:

\begin{equation}
  T_{tax} = \sum_{i=1}^N D^a_i \min\left(\dfrac{T_{VL}C_i+D_i}{T_{VL}C^0_i + T_DC^0_i} - 1,1\right)\ .
\end{equation}

### Withdrawal tax

The withdrawal tax would work identically to the deposit tax, where the tax is levied on all deposits that either occur, or move, token concentrations below their target ($W_{i}^b$). This withdrawal balance can be computed as follows:

\begin{equation}
W^b_i = \min\left(\max\left(W_i -  T_{VL}(C_i - C^0_i) - T_WC^0_i,0\right),W_i\right)\ .
\end{equation}

The USD value that should be subtracted from each individual token withdrawal can be written as:

\begin{equation}
  T^{tax}_i = W^b_i \min\left(1-\dfrac{T_{VL}C_i-W_i}{T_{VL}C^0_i - T_WC^0_i},1\right)\ .
\end{equation}

With the total value of TRSY tokens transferred to the tax contract being:

\begin{equation}
  T_{tax} = \sum_{i=1}^N T^{tax}_i\ .
\end{equation}


### Swap tax

Swap can be seen as a deposit and withdraw in the same transaction, so the tax logic is similar. Deposit tax is applied on the assetIn and withdraw tax is applied on the assetOut.

