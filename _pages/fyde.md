---
title: Fyde contract
author: Fyde  
date: 2022-02-04
category: Jekyll
layout: post
---

## Main overview

The Fyde contract serves as the core contract of the Fyde protocol. It handles the logic for depositing, withdrawing, and swapping assets within the protocol.

Fyde operates as a diversified pool, accepting certain assets, each with a specific target concentration representing the asset's desired weight in the protocol relative to the Total Value Locked (TVL) in USD. Target concentrations represent the ideal weight of a given asset in the protocol as part of an overarching portfolio management strategy. Users can deposit, withdraw, and swaps these assets in the Fyde protocol.

### Deposit

Approved assets can be deposited by users, and upon deposit, TRSY is minted to the user. Minted TRSY represents a share of the pool that is proportional to the deposited amount denominated in USD. 

For instance, if a user deposits 100 TokenA, equivalent to 100k USD, and the TVL is 1 million USD with 500k TRSY in circulation, the user will receive 50k TRSY.


### Withdraw
Users can withdraw assets by burning their TRSY tokens. The withdrawal value is computed using mechanisms akin to those used in depositing.

For example, if a user holds 50k TRSY and the TVL is 1 million USD with 500k TRSY in circulation, the user can withdraw 10% of the index pool, equivalent to 100k USD.


### Swap
Users can exchange one asset for another (assetIn for assetOut). During this process, the USD value of assetIn is traded for an equal USD value of assetOut. (Note: Incentive factors are applied to encourage swapping of specific assets).

### Tax 

Actions like deposits, withdrawals, and swaps that deviate the asset concentrations from the target are subjected to a tax. This tax penalizes actions that shift the asset concentrations away from the strategy, maintaining closer adherence to the portfolio management strategy.

For instance, deposits altering the concentration towards an overweight position will incur a tax, as will withdrawals from an underweight asset.


## Smart contract overview

The Fyde contract is primarily responsible for executing the logic related to depositing and withdrawing funds from the diversified pool and the governance pool. Additionally, the Fyde contract inherits the logic of the following modules:

<img src="{{site.baseurl}}/illustrations/fyde.png">


- TRSY: an ERC20 used to represent the pool in the form of shares (logic similar to vault ERC-4626)
- AddressRegistry: contains the addresses of contracts that interact with the protocol
- ProtocolState: contains the accounting logic of the protocol, as well as various parameters for the protocol
- GovernanceAccess: functions to connect Fyde with the governanceModule contract
- AssetRegistry: contains the logic for adding assets to the protocol
- Tax: calculates potential taxes on deposits, withdrawals, and swaps.



## Computation of the tax 

### Deposit tax

This system is encapsulated by the following function, which charges no tax if the deposit remains underneath the target concentration of the token pool. A tax proportional to the difference between the current ($C_{i}$) and target concentrations ($C_{i}^0$) will be levied on the portion of assets that are deposited into overweight pools ($D_{i}^a$). The tax is computed for each pool where a deposit is made, where $D_{i}$ is the USDC value of each individual deposit and $T_{D}$ is the total deposit value.

\begin{equation}
D^a_i = \min\left(\max\left(D_i + T_{VL}(C_i - C^0_i) - T_DC^0_i,0\right),D_i\right)\ .
\end{equation}

The total tax value, subtracted from the minted TRSY tokens and denominated in USD, is calculated as:

\begin{equation}
  T_{tax} = \sum_{i=1}^N D^a_i \min\left(\dfrac{T_{VL}C_i+D_i}{T_{VL}C^0_i + T_DC^0_i} - 1,1\right)\ .
\end{equation}

### Withdrawal tax

The withdrawal tax functions identically to the deposit tax, applying to withdrawals that reduce token concentrations below their targets ($W_{i}^b$). This withdrawal balance can be computed as follows:

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

A swap is treated as a deposit and withdrawal within the same transaction, hence the tax logic is very similar.  Deposit tax is applied on the assetIn and withdraw tax is applied on the assetOut.

