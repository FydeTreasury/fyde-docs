---
title: Oracle module
author: Fyde  
date: 2022-02-06
category: Jekyll
layout: post
---

## Fyde's need for oracles
Fyde Protocol operates by allowing users to deposit assets to mint TRSY, burn TRSY for asset withdrawal, and swap the USD value of one asset for another. To facilitate these functionalities, accurate and current price information for all supported assets is crucial. This is where oracles come into play, providing reliable and real-time price feeds, ensuring the protocol functions accurately and securely.

## Fyde's oracle design
Fyde’s design incorporates a blend of Chainlink and Uniswap V3 TWAP (Time-Weighted Average Price) oracles, utilizing a 30-minute time window to fetch price data. This hybrid approach aims to harness the benefits of both Chainlink’s decentralized oracle network and Uniswap V3’s on-chain TWAP oracles to achieve more robust and reliable price feeds.

## Aggregation and circuit breakers
When the protocol needs to ascertain the value of an asset, it doesn’t rely on a single source. Instead, it aggregates prices from Chainlink and Uniswap V3 TWAP, enhancing reliability and accuracy. Alongside aggregation, the implementation of circuit breakers adds an extra layer of security. Circuit breakers can halt operations based on predefined conditions, adding robustness against abnormal price movements or possible oracle failures.

## Managing risks and asset whitelisting
Recognizing the potential risks associated with on-chain TWAPs, such as susceptibility to attacks and price manipulations, Fyde implements several in-depth risk management strategies. One such strategy is the application of a stringent whitelist criterion, ensuring that only assets meeting specific standards, such as substantial liquidity in Uniswap V3 pools, notable trading volumes and market caps, doxxed founders, etc. are integrated into the protocol. Another strategy involves assessing the historical price performance of a token in comparison to a broad spectrum of crypto tokens over the same timeframe (drilling down to the minute and second level), ensuring that the token exhibits "normal" price behavior in the moments leading up to it's deposit into the pool. These precautions aim to mitigate the risks of price manipulations, thereby maintaining the integrity and stability of the Fyde Protocol’s operations.

### Oracle flow 

<img src="{{site.baseurl}}/illustrations/OracleFlow.svg">

* see next graph for the circuit breaker implementation


### Circuit breaker implementation

Embedded within Fyde are circuit breakers to prevent the use of incorrect prices. For Chainlink, primary checks involve ensuring that the price is up-to-date and the data is accurate. In the case of Uniswap, Fyde compares the values between the 30-minute TWAP (Time-Weighted Average Price) and the 1-minute TWAP to avoid consuming prices during periods of heightened volatility.

<img src="{{site.baseurl}}/illustrations/CircuitBreaker.svg">

