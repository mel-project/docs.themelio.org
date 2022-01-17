---
title: "Melmint: a trustless stablecoin"
weight: 4
draft: false
# search related keywords
keywords: [""]
---

## Crypto today is bad at being money

All decentralized blockchains, starting from Bitcoin, are intimately tied to on-chain cryptocurrencies. These cryptocurrencies have 2 functions:

- providing a trustless and irreversible payment medium to end users

- serving as protocol-internal units of value for incentive design.

However, largely due to extremely volatile prices, existing cryptocurrencies are pretty bad at both of these jobs:

- Cryptocurrencies are bad at being _money_ --- an asset that is simultaneously a store of value, unit of account, and medium of exchange. At best, a cryptocurrency is used as a "hot-potato" payment intermediary; at worst, it is used entirely as a speculative asset sitting in exchanges.

- Designing secure financial contracts and mechanism incentives is close to impossible without a stable monetary unit. If nothing on-chain has a reliable purchasing power, pricing future assets (like in loans, etc) becomes impossible, and any kind of fixed rewards or punishments are forced to vary wildly based on the currency price.

Most existing attempts to fix this problem are _pegged stablecoins_, i.e., cryptocurrencies pegged to an external value-stable asset, usually a fiat currency like the US dollar. Stablecoin schemes include centralized currencies like Tether (they act as fiat-denominated IOUs against a trusted bank) as well as semi-decentralized systems like MakerDAO (these attempt to hold a peg through algorithmic monetary policy involving complex on-chain financial assets). In fact, stablecoins pegged to the US dollar have gained widespread use as _de facto_ units of account in on-chain mechanisms --- even infrastructural protocols like ENS (Ethereum Name System) are [starting to adopt dollar-denominated](https://medium.com/the-ethereum-name-service/ens-integrates-chainlink-eth-usd-price-oracle-183e64a05d89) incentives.

Unfortunately, fiat stablecoins are unacceptable as a general-purpose cryptocurrency. As cryptoassets pegged to external goods, _they compromise endogenous trust_. This is because there is no way to autonomously measure any external asset's value on the blockchain: even "decentralized" systems like MakerDAO rely on trusted _price oracles_. Furthermore, coins tied to external assets are inherently dependent upon the stability of that external asset, which in the case of fiat currencies relies on old-fashioned centralized trust.

## Mel, an endogenous stablecoin

_Melmint_ is our solution to this problem. Melmint is an algorithm that mints mel, an **endogenous stablecoin** belonging to a completely novel asset class introduced by Themelio. Mel maintains a stable value without being pegged to any external asset, thereby avoiding the non-endogenously-trusted parties present in every other stablecoin system (e.g., oracles, governance DAOs, and issuers).

![](/images/melmint-v2-overview.png)

The details of Melmint are available in [a separate document]({{<ref "../whitepapers/melmint-v2.md">}}), as well as in an earlier conception as [a paper published in Cryptoeconomic Systems 2020]({{<ref "../whitepapers/melmint.md">}}), but in broad strokes, Melmint pegs mel to an index we call DOSC, for "day of sequential computation". As its names suggests, the value of a DOSC at a given time is defined as the cost of running 24 hours of sequential computation, verified through a non-interactive proof of sequential work, on the fastest processor available at that time. Melmint maintains mel's peg to DOSC through two steps:

- First, we trustlessly peg an on-chain synthetic asset called the _erg_ to DOSC. No oracles are needed at all in this process, because the erg's minting mechanism automatically updates the DOSC to the fastest processor it has seen, and minters are economically incentivized to use the fastest machines.
- Then, automatic monetary policy adjust the supply of mel so that the mel/sym exchange rate, measured through a Uniswap-like on-chain decentralized exchange, is close to 1:_1 DOSC worth of sym_. This is done through printing mel to buy sym (Themelio's proof-of-stake token) to expand mel supply, and vice-versa to contract mel supply. "1 DOSC worth of sym" is then defined by observing the sym/erg market.

As detailed in [the whitepaper]({{<ref "../whitepapers/melmint.md">}}), both stochastic market simulation and _a priori_ arguments show that Melmint is highly resilient in a wide variety of market environments.
