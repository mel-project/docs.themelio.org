---
title: "Token overview"
weight: 5
draft: false
# search related keywords
keywords: [""]
---

This document summarizes Themelio's "tokenomics" --- the incentives and rewards surrounding Themelio's two built-in tokens, **mel** and **sym**.

## Two tokens

Themelio has two main built-in cryptocurrency tokens:

- **Mel** is the default "money" of the blockchain. All fees and protocol-internal incentives are paid out in mel. It is an _endogenous stablecoin_ --- a currency that is both fully trustless and [can keep a relatively stable value]({{<ref "../basic-concepts/04-melmint.md">}}). Mel supply is not fixed, adjusting elastically to demand to keep a stable value.
- **Sym** is the special-purpose _proof-of-stake token_. Protocol value accrues to sym, and buying sym is the best way of "investing in the Themelio protocol". Stakers stake sym to participate in protocol consensus, receiving _fees_ denominated in mel. Unlike mel, sym issuance follows a fixed, bitcoin-like schedule.

> **Note**: the "ticker symbols" for mel and sym are, somewhat confusingly, "MEL" and "SYM". The relationship between "mel" and "MEL" is analogous to that between "ether" and "ETH".

The mechanics of how mel is stabilized is discussed in the [Melmint document]({{<ref "04-melmint.md">}}). Here we instead largely focus on the _mechanisms that accrue value to sym_.

## Staking sym for transaction fees

Anybody can _stake_ sym. This consists of [locking up sym for a fixed amount of time]({{<ref "../specifications/consensus-spec.md">}}), designating a particular _staker_ that then receives proportionate voting power in the consensus. (A staker corresponds to a "validator" in many other systems)

Stakers sequentially up/downvote a _fee multiplier_. This is multiplied by the _weight_ of every transaction to determines the minimum fee it must pay, known as the _base fee_. Any excess fees are called _tips_.

Base fees are deposited into a global fee pool. Every staker who creates a new block is then entitled to withdraw a small fraction of this fee pool. The effect of this is to divide base fees in a stake-weighted fashion among all the stakers. Tips, on the other hand, are more like traditional blockchain fees and are paid to only the creator of the block.

A good way of thinking about this is that base fees build up a "trust fund" out of which relatively stable "block rewards" are paid. This gives a similar revenue stream as traditional blockchains to stakers, while ensuring that rewards come from fees and not inflation.

### On stake pooling

When locking up sym, any staker can be designated as the staker that receives , not just oneself. This allows an easy form of stake pooling, but there is a crucial difference with most "delegated PoS" systems --- designating a third-party staker requires trusting that staker. If the staker gets slashed as punishment for misbehaviour, everybody who designated stake to that staker will get slashed. It is also up to the staker to distribute rewards it receives to people designating it in their stakes, though by default `themelio-node` does attempt to return staking rewards to people contributing stake.

## Sym inflation

As a bootstrap mechanism, we have a small amount of _sym inflation_ in the early years of the protocol that gradually decays to zero, similar to the block reward in Bitcoin.

In particular, we have an initial $2^{20}$ microSYM/block (around 1.04 SYM) inflation rate, that decays by half every 1,000,000 blocks (approximately 1 year).

Unlike in other blockchains, this inflation is not simply paid out as a block reward to the stakers. Instead, it is [_split_ between two uses](https://github.com/themeliolabs/themelio-node/issues/86):

- 99.61% **funds a mel-denominated pseudo-block-reward**. This is by using the sym to buy mel in the [Melswap]({{<ref "../specifications/tech-melmint.md#specification">}}) MEL/SYM market. The mel bought is then deposited into the fee pool to be distributed among the stakers.
- 0.39% **subsidizes Melmint**, by using the sym to buy erg in the SYM/ERG market. The erg bought is then destroyed. This is essentially an "auction" of newly printed sym for Melmint computation, bootstrapping a small amount ofactivity in the Melmint mechanism.

## Staker income summary

The following Sankey diagram summarizes the entire MEL-denominated "block reward" of a staker:

![](/images/staker-reward.png)

## Sym allocation

Of course, no "tokenomics" page is complete without a token allocation section. At the time of our planned token launch, we will have an initial supply of 1,048,576 SYM, divided **very roughly** as follows:

- **20%**: private investors
- **10%**: other insiders
- **70%**: general public

> **Note**: until we make an official token sale announcement, the current token allocation is preliminary and subject to change!

We plan on selling the "general public" portion over a period of time, rather than all at once.

After the initial sale, due to sym inflation the above percentages will decrease. This is illustrated in this animated piechart:

![](/images/tokenalloc.gif)
