---
title: "Melmint: a trustless stablecoin"
weight: 4
draft: false
# search related keywords
keywords: [""]
---

All decentralized blockchains, starting from Bitcoin, are intimately tied to on-chain cryptocurrencies. These cryptocurrencies, in addition to serving as protocol-internal units of value for incentive design, also provide a trustless and irreversible payment medium to end users. But largely due to extremely volatile prices, existing cryptocurrencies are pretty bad at actually _being money_ --- an asset that's simultaneously a store of value, unit of account, and medium of exchange. At best, cryptocurrency is used as a "hot-potato" payment intermediary; at worst, it is used entirely as a speculative asset (often sitting in exchanges).

This state of affairs is unacceptable, especially for Themelio. Without a usable monetary unit, not only will it be difficult to build robust applications on top of Themelio, mechanism design becomes close to impossible if nothing on-chain is reliably valuable.

Most existing attempts to fix this problem are _pegged stablecoins_, or cryptocurrencies that are pegged to an external value-stable asset, generally a fiat currency like the US dollar. Stablecoin schemes include centralized currencies like Tether that act as fiat-denominated IOUs against a trusted bank as well as semi-decentralized systems such as MakerDAO which attempt to hold a peg through algorithmic monetary policy involving complex on-chain financial assets. A problem common to all stablecoins targeting an exchange rate to an asset external to the blockchain, though, is that there is no trustless way of measuring this value on the blockchain. Even “decentralized” systems like MakerDAO rely on trusted _price oracles_. In addition, coins tied to external assets are inherently vulnerable to shocks in the price of that external asset.

_Melmint_, an algorithm to mint an asset known as Mel, is our solution to this problem. Mel is an **endogenous stablecoin**. This is a completely novel asset class introduced by Themelio, and it means that the mel maintains a stable value without being pegged to any external asset, such as US dollars or gold. Mels maintain their value without non-endogenously-trusted parties present in every other stablecoin system, such as oracles, governance DAOs, and issuers.

The details are available in the [protocol specification]({{<ref "../specifications/tech-melmint.md">}}), as well as in an earlier conception as [a paper published in Cryptoeconomic Systems 2020]({{<ref "../whitepapers/melmint.md">}}), but in broad strokes, Melmint pegs the Mel to an index known as DOSC, for “day of sequential computation”. The value of a DOSC at a given time _t_ is defined as the cost of running 24 hours of sequential computation, verified through a non-interactive proof of sequential work, on the fastest processor available at time _t_. This is through a two-step process:

- First, a trustless minting mechanism, called Elasticoin, mints nomDOSC, a on-chain synthetic asset whose _minting cost_ is a DOSC. No oracles are needed at all in this process, as the behavior of minters participating in the mechanism already provides a trustless source of the value of a DOSC.

- Automatic monetary policy adjust the supply of Mel so that the Mel/nomDOSC exchange rate, measured through a Uniswap-like on-chain decentralized exchange, is close to 1:1. This is done through printing Mel to buy Sym (the proof-of-stake token) to expand Mel supply, and vice-versa to contract Mel supply.

Both stochastic market simulation and _a priori_ arguments, that we [detail in the whitepaper]({{<ref "../whitepapers/melmint.md">}}), show that Melmint is highly resilient in a wide variety of market environments.
