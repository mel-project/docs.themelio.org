---
title: "Melmint v2 overview"
date: 2018-12-29T11:02:05+06:00
lastmod: 2020-01-05T10:42:26+06:00
weight: 9
draft: false
# search related keywords
keywords: [""]
---

This document gives a high-level overview of _Melmint_: the mechanism that issues and stabilizes Mel, Themelio's base currency.

## Why an endogenous stablecoin?

The most unique thing about Mel is that it is an _endogenous stablecoin_. That is,

- It has an issuance mechanism entirely based on endogenous trust. That means that like Bitcoin's issuance schedule, no trusted parties are involved at all --- no central bank, no oracles, no DAOs.
- It nevertheless maintains a stable purchasing power.

Mel is the first currency to have both properties. But why do we want an endogenous stablecoin in the first place?

First of all, volatile cryptocurrency prices pose a serious problem. The value of a currency going up and down erratically obviously makes it bad as money: it becomes less useful as a store of value or unit of account. More importantly, a volatile unit of account greatly damages the ability to create interesting financial instruments. Nobody is going to sign a 30-year mortgage in Bitcoin without knowing whether Bitcoin will be worth 1, 1,000, or 1,000,000 apples 30 years in the future. This also greatly impacts mechanism design

Unsurprisingly, the DeFi ecosystem relies heavily on fiat-pegged stablecoins like USDC and Dai. But these stablecoins are not what we want: they inherently rely on exogenous trust, in oracles, central issuers, etc. Note that this is a problem even when the stablecoin aims to track a basket of commodities rather than fiat currency --- we may lose centralized trust in the Fed, but not trust in oracles. Oracles supplying external information about the commodity market are crucial for the feedback loop of any stablecoin backed by off-chain assets, and they rely heavily on preexisting trust in people or institutions, not endogenous trust guaranteed by protocol incentives. Even "decentralized oracles" don't cut it: endogenous trust is about incentives, not how many people vote on decisions. Conventional stablecoins, then, cannot be the base currency for a blockchain focused on strict endogenous trust.

Thus, Melmint uses an oracle-free system that doesn't try to peg Mel to any external asset. Instead, we define a new, _trustlessly-measurable value unit_, called the DOSC (day of sequential computation), which can be measured on-chain by an autonomous mechanism with full endogenous trust. This, and not dollars or gold, is what the core Melmint mechanism then pegs Mel to.

## A very high-level overview

The one-sentence summary of Melmint's job is to maintain **peg 1 mel to 1 DOSC**. But what is a "DOSC"?

A "DOSC" is a "day of sequential computation". It's defined as the _cost of running a sequential computation for 24 hours, using the fastest processor available_. For example, a DOSC in the year 2000 is the cost of occupying the fastest single CPU core _available in 2000_ for 24 hours, while a DOSC in the year 2021 is the cost of doing the same with a 2021 processor.

The DOSC has two really cool properties that make it a great target for a peg:

- It has a relatively stable purchasing power. Empirically, the "fastest processor" typically costs about the same, despite its performance drastically increasing over time. We explore this further in TODO.
- More importantly, it's _trustlessly measurable_ through a "sequential proof of work". A hash-based sequential proof of work, like the one invented by Cohen and Pietrzak, produces a succinct proof that starting from some seed $x$, a large amount of sequentially nested hashes $H(H(H(H(..H(x)..))))$ have been computed. By using a cleverly incentivized on-chain benchmark, we can then measure how fast the fastest processor is. This gets us a way of showing on-chain that we've wasted a day's worth of sequential computation, letting us measure the DOSC with strong endogenous trust.

  The DOSC's trustless measurability is the _key to Melmint's oracle-free mechanism_.

But how is this peg maintained? There are two main steps: _erg-minting_ and the _central mechanism_, summarized in the following picture:

![](/images/melmint-v2-overview.png)

Erg-minting involves allowing anybody to use a special transaction type (`DoscMint`) to prove that they completed a certain amount of sequential proof-of-work. This transaction then generates $k$ "ergs" for each DOSC of work done. $k$ here is not a constant, but an exponentially increasing conversion factor --- while 1 DOSC of work may generate $10$ ergs today, it might generate $100$ ergs a year from now. Due to this rapid inflation, ergs act as an on-chain, tokenized representation of _recent_ sequential work.

The central mechanism uses **Melswap**, a built-in, Uniswap-like decentralized exchange that supports all Themelio-based tokens. Its objective is to _peg 1 mel to 1 DOSC worth of syms_, using a feedback loop that prints mels and buys syms or vice-versa. Recall that the Sym is Themelio's separate, proof-of-stake token. First, we read two exchange rates off of Melswap:

- $s$: how many syms can 1 mel buy
- $t$: how many syms can 1 erg buy

Then, Melswap targets an exchange rate of 1 mel = $tk$ syms --- that is, 1 DOSC ($k$ ergs) worth of syms. When this $s=tk$ peg fails to hold, we use _inflation_ to back it:

- **Mel too cheap**: This is the blue box in the picture, where $s<tk$. In this case, Melmint continually prints syms out of thin air, using them to buy mels, which are subsequently destroyed. This artificially increases the demand for mel, increasing its purchasing power until $s=tk$.
- **Mel too expensive**: This is the red box in the picture, where $s>tk$. Here, Melmint would instead print mels, using them to buy syms. This increases the supply of mels, decreasing its price until $s=tk$.

We now take a look at each individual step in this process:

## Step 1: Minting ergs

The first step in Melmint is for minters --- which can be anybody --- to mint **ergs**, $k$ of which represent a DOSC.

### Proving work

Minters create ergs through a DoscMint transaction, something detailed in the [yellow paper]({{< relref path="../specifications/yellow.md#applying-special-actions" >}} "Yellow Paper").

In summary, the [`data` field]({{< relref path="../specifications/yellow.md#transactions" >}} "Yellow Paper") of a DoscMint transaction contains two values: a _difficulty exponent_ $z$, as well as a _proof_ $\pi$. This uses MelPoW, a non-interactive proof-of-sequential-work system whose details aren't important for this document, but in short, $(D,\pi)$ is a proof that roughly $2^z$ nested hashes have been computed on the _seed_ $\chi$, which consists of the first input to the transaction, hashed together with the block hash of the block in which that first input was confirmed.

The key property here is that _the minter cannot predict $\chi$ before the first input to the transaction is confirmed_. This means that the minter must have finished the $2^z$ hashes after the first input has confirmed, but before the DoscMint transaction itself has confirmed. We now have a time interval, and therefore a _trustless measure of the minter's speed in hashes per second_.

By simply remembering the fastest minter ever seen, the blockchain knows how many hashes the fastest minter can do in a day; let's call that $M$. Then, we know that our minter did $d=2^z/M$ DOSC of work --- that is, the same amount of sequential work as the fastest minter run for $2^z/M$ days would do.

### Creating ergs

Now, the DoscMint transaction has proven that the minter did $d$ DOSCs of work. The blockchain needs to award the minter with $k d$ ergs (where $k$ is the erg-DOSC conversion factor) --- in Themelio's coin-based model, this is simply done by allowing imbalance in incoming and outgoing ergs: the transaction's allowed to produce $k d$ more ergs than it consumed.

**Important note**: $k$ here is an exponentially increasing conversion factor: $k=1$ at the genesis block, and then $k$ increases by 0.00005% every block. This translates to approximately a 70\% annual inflation rate. The purpose of an exponentially increasing $k$ is so that _the supply of ergs is dominated by freshly minted ergs_ --- this ensures that the market value of $k$ ergs tracks the 1 DOSC cost of creating those ergs. If $k$ were instead constant, a drop in demand would cause a glut of old ergs that would not sell, where $k$ ergs become worth less than 1 DOSC despite them costing 1 DOSC when they were minted. This would make ergs useless as way of quantifying the value of a DOSC.

### Selling the ergs

Because of the astronomical inflation rate, ergs are not very useful as money. Instead, minters would almost always want to sell their ergs soon for a more value-stable asset.

Themelio conveniently comes with a built-in decentralized exchange, called Melswap, where any Themelio-based token (including [custom tokens](< relref path="../specifications/yellow.md#applying-a-transactions-outputs" >)) can be exchanged for any other. This, of course, includes ergs. Minters can easily take their ergs and exchange them for other tokens, most likely the two main Themelio tokens Sym and Mel.

In particular, the sym/erg market is "special" --- it is crucial for driving Melmint's core pegging mechanism, and in fact it's subsidized by diverting half of all Sym block rewards to the sym/erg Melswap pool.

## Step 2: Core pegging mechanism

### Inflation-backed mel/sym peg

The core pegging mechanism of Melmint is that _through inflating either Mel or Sym, 1 mel is pegged to $k$ ergs (1 DOSC) worth of syms_. In particular, at every block height, either some syms or some mels are printed out of thin air and used to buy the other token in the mel/sym market on Melswap, always "nudging" the exchange rate closer to 1 mel = 1 DOSC worth of syms.

For example, consider a Melswap market with the following exchange rates:

- $1$ MEL = $2$ SYM
- $1$ ERG = $1.5$ SYM
- $k=1.5$

Here, "1 DOSC worth of syms" would be "1.5 ergs worth of syms", or $1.5\times 1.5 = 2.25$ syms. Yet the current market exchange rate is only $2$ syms, meaning that _mels are too cheap_.

Thus, to support the peg, every block Melmint will print up some syms to buy up mels, until the peg holds.

### What backs Mel?

Any stablecoin can easily hold a peg when the stablecoin is too expensive --- just print more. The true challenge is when the peg must be defended when the market price is below the peg. The stablecoin must in some way be "backed" by another asset, with an independent value, that it can be redeemed for at the pegged value.

In our case, the Mel's value is backed by the _value that can be extracted by inflating Sym_. We call this value the **implicit reserve** of Melmint. Because inflating sym is essentially a tax on all holders of syms, the implicit reserve is very roughly the market capitalization of Sym. We can therefore say that the Sym backs the Mel.

Thus, to be stable in a worst-case scenario where everybody wants to sell their mels, the **Mel marketcap must stay below the Sym marketcap**. Fortunately, as the [original Melmint paper shows](https://docs.themelio.org/assets/mel.pdf) that's likely to be the case in any reasonable economic conditions.

## Failure scenarios

### Drastic changes in DOSC purchasing power

Because Melmint pegs the Mel to the DOSC, if the value of a DOSC drastically changes, mels will lose their stable purchasing power. Historically, this has not really happened, and due to the definition of a DOSC, things like routine technological improvements will not cause shocks to the value of a DOSC. Instead, an external shock must greatly affect _how expensive is running the fastest processor available_. Here are some hypothetical scenarios where the value of a DOSC will drastically change:

**Sudden increases in DOSC value**:

- Massive energy crisis makes electricity 10x more expensive than before.
- Somebody finds a way to build an extremely expensive, energy-inefficient machine that does sequential computation 10x faster than top-end machines, at 100x the daily cost.

**Sudden decreases in DOSC value**:

- Breakthrough in energy generation makes electricity nearly free
- Breakthrough in processor design leads to the domination of extremely high core-count machines where the per-core operational cost is drastically lower, yet sequential speed is comparable to current processors
  - "GPU except every core is a fully general-purpose CPU"

### Sudden decrease in Themelio market sentiment

For a variety of reasons (say, a general cryptocurrency crash), there might be a scenario where a large amount of Sym or Mel is panic-sold. This threatens the basis of the Melmint peg, and in a case where Mel issuance cannot be backed by the implicit reserve, the peg may no longer be tenable.

The danger here is hyperinflation of Sym, as Melmint desperately prints syms to buy and prop up the value of Mel. Fortunately, this is inherently prevented by the way Melmint works --- per the Melmint specification, the amount of syms or mels printed to support the peg is _proportional to existing liquidity in the sym/mel market_. This means that "how fast" Melmint reacts depends on market conditions. In a panic where market participants anticipate a possible Melmint-driven Sym hyperinflation that will cause both mels and syms to become worthless, liquidity will quickly drain from the mel/sym market as both mel and sym are dumped for alternative assets.

Without mel/sym liquidity, Melmint is effectively turned off. The peg will obviously fail, massively increasing Mel volatility, but a wholesale monetary collapse will be averted, especially because unlike users of USD stablecoins, Mel users never expected zero exchange-rate risk and are unlikely to completely dump the Mel due to a temporary pause of the peg. Once sufficient liquidity returns to the Melswap mel/sym market, the peg will gradually be restored.
