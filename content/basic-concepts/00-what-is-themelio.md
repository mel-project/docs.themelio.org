---
title: "What is Themelio?"
date: 2018-12-29T11:02:05+06:00
lastmod: 2020-01-05T10:42:26+06:00
weight: 1
draft: false
# search related keywords
keywords: [""]
---

## What is Themelio?

Themelio is a public blockchain that enables open, secure, and resilient decentralized apps. An example is Mel, an independent, stable-value cryptocurrency that anyone can freely use. Inspired by previous blockchains like Bitcoin and Ethereum, Themelio nonetheless takes a very different design approach: **radical immutability** inspired by the architecture of the Internet.

![This is an image](/images/hourglass.png)

Traditional blockchains either implement one application or try to provide a comprehensive solution to all use cases—in this aspect they are like pre-Internet telecom protocols. Like those protocols, they suffer from damaging _application-protocol friction._ While basic infrastructure like Ethereum or the telephone network is very difficult to change once widely adopted, application requirements constantly and unpredictably evolve. This inevitably causes either poor usability, destabilizing protocol upgrades, or both, as exemplified in highly contentious blockchain hard forks like Ethereum's DAO fork or Bitcoin's SegWit upgrade.

On the other hand, Themelio’s design is firmly centered around decoupling the blockchain from user-facing applications. Themelio does one thing --- provide robust low-level functionality critical to secure decentralized apps --- and strives to do it very well.

In fact, we aim to develop a protocol flexible and robust enough to never need change. Themelio will be _completely governance-free_ after the stable network launches, simply sidestepping the [contentious problem of coordinating consensus-critical protocol upgrades](https://nullchinchilla.me/against-blockchain-governance/).

We envision Themelio as a radically simple, stable, and long-term foundation that can power a vast and diverse ecosystem. As such, we expect Themelio to play a fundamental and ubiquitous role in the security infrastructure of a new and decentralized Internet, much like how IP, "immutable" [since 1983](https://tools.ietf.org/html/rfc791), still underpins Internet communication.

---

## Self-security through stability: the heart of Themelio

Themelio's core mission is to serve as an **self-secure, endogenous root of trust**. Endogenous trust, or trust that emerges from within the protocol rather than from its users, is the "killer feature" of blockchains. It allows us to trust the blockchain with minimal assumptions about who runs it. Endogenous trust distinguishes blockchains from all previous protocols, including decentralized and federated ones like BitTorrent and IRC, by _enabling the protocol to secure itself_. By focusing on this one key aspect of blockchains and implementing it exceptionally well, Themelio aims to support a new blockchain-rooted, multilayered paradigm of decentralized Internet apps.

A radically immutable protocol is the most important precondition for maintaining endogenous trust. A protocol with regular tweaks makes "governance" an unpredictable _exogenous_ process that must rely on preexisting trust in whomever guides protocol governance.

But immutability alone is not enough. We want an immutable protocol that can effectively power endogenous trust for applications, not just itself. This requires a more comprehensive notion of "stability": not just a stable protocol, but also a stable cryptocurrency to reliably and trustlessly store and measure value on the blockchain, as well as stable behavior that enables easy, secure abstractions that don't compromise endogenous trust.

### Stable, composable protocol

The most important pillar of Themelio's stability is its stable protocol. Themelio's core blockchain logic is much simpler than other general-purpose blockchains like Ethereum and Tezos, heavily encouraging immutability by simply lacking most knobs that might be tweaked through governance. It employs a seemingly rudimentary transaction model based on spending coins, i.e., unspent transaction outputs, or UTXOs. With a small, carefully selected set of additional features, this allows Themelio to power entire application classes impossible with conventional coin-based blockchains like Bitcoin and Litecoin.

The two most important of these "superpowers" are MelVM and state commitments. MelVM is a non-Turing-complete virtual machine that nevertheless can compute all [primitive recursive functions](https://en.m.wikipedia.org/wiki/Primitive_recursive_function) and enables attaching sophisticated **covenants** to coins. It powers on-chain protocols like tokens and financial instruments without the pitfalls of stateful smart contracts. Then, state commitments use [sparse Merkle trees](https://ethresear.ch/t/optimizing-sparse-merkle-trees/3751) to commit in the block header all the information needed to validate transactions, like the set of all unspent coins. This enables scaling strategies crucial to global adoption, such as full nodes with limited storage space and thin clients that can securely verify much more information than conventional techniques like Bitcoin's SPV allows.

### Stable cryptocurrency

The second pillar of Themelio's stability is Mel, its base cryptocurrency. Mel is the **first ever endogenous stablecoin**: a cryptocurrency that keeps a relatively stable purchasing power without pegs to external assets like the US dollar. The purchasing power a mel is backed by Melmint, an equity-based stablecoin mechanism similar to Basis and Seigniorage Shares, with two crucial differences:

- Instead of a price oracle, we use an auction mechanism to measure the on-chain value of a _day of sequential computation_, or "DOSC". The value of 1 DOSC at a given time is the cost of 24 hours of sequential computation on the fastest processor available at that time --- a value that maintained surprisingly stable purchasing power over the last two decades. This frees Mel entirely from trust in external oracles and fiat currency.
- Rather than shares backed by future stablecoin issuance, Melmint issues Sym, Themelio's proof-of-stake token, to support the mel's value in times of decreasing demand. This eliminates "death spiral" scenarios common in dual-token stablecoin mechanisms and guarantees Mel's stability as a blockchain's base currency even when issuance is expected to decline.

We've published Melmint as a [peer-reviewed paper at Cryptoeconomic Systems 2020](https://cryptoeconomicsystems.pubpub.org/pub/2ggmf2k0/release/4), and a [further revised version is available here](melmint-trustless-stable-cryptocurrency.md) for those interested in the technical details of Melmint.

### Stable behavior

Finally, stable behavior during blockchain operation is crucial to developing apps that leverage endogenous trust reliably and securely. This is largely supported by **Synkletos**, the cryptoeconomic protocol that secures Themelio's consensus and powers its on-chain fee economy.

Synkletos is based on Byzantine fault-tolerant proof-of-stake consensus between "stakers" with a separate token, Sym. Sym is fixed-issuance like traditional cryptocurrencies and pays dividends from transaction fees. By using BFT consensus, Synkletos ensures that transactions are either fully confirmed or not present in the blockchain, eliminating the complex latency-dependent guesswork required in "chain-based" blockchains like Bitcoin and Ethereum when deciding whether or not to permanently trust a transaction.

Unlike present blockchains' fee systems, Synkletos drives staker income largely from stabilized, market-determined fees rather than inflation-based block rewards. This aligns incentives between stakers and users, rendering collusion between stakers to subvert the protocol unprofitable even with a small number of stakers, while making fees much stabler and fee estimation trivial.

You can find [a detailed treatment of Synkletos right here](../whitepapers/synkletos) in this knowledge base.

---

## Get started now!

Themelio has launched its **betanet**. This gives us a proof-of-concept to tinker with that you can also participate in. At this point, the protocol is largely stable, but hard forks will still happen, with governance becoming increasingly community-driven and increasingly conservative.

To get started, you can learn more about Themelio in this knowledge base, [join our Discord channel](https://discord.gg/2HmurW9T9K), and try transacting in some placeholder "Monopoly money" with our [alphanet wallet]({{< ref "../try-themelio/my-first-tx.md" >}}).
