---
title: "Endogenous trust"
weight: 1
draft: false
# search related keywords
keywords: [""]
---

## The killer feature of blockchains...

Themelio's entire design is motivated by what we believe to be the most important feature of blockchains: **endogenous trust**. That is, we can trust that a blockchain protocol will behave correctly _without trusting the people running it_. In blockchains, trust emerges from _within_ the protocol, not from preexisting trust in the parties that run the protocol. Crucial to endogenous trust is _cryptoeconomics_, the intersection of game theory and cryptography that enables self-incentivizing mechanisms so that given enough participants, we can trust the group's overall behavior without trusting any individual participant.

Unlike other widely-touted properties like decentralization (which BitTorrent also has) or transparency (which Keybase, etc also have), endogenous trust is pretty much unique to public blockchains. It's the key feature that gives blockchain apps "superpowers" elusive to pre-blockchain systems.

For example, no matter what kind of communication technology is used, someone using a bank can only do so securely if they trust the bank, or at least third-party authorities like regulatory agencies. Yet a Bitcoin user does not need to even know which miner ends up processing their transaction, let alone establish any sort of trust with that miner. Instead, the cryptoeconomics of Bitcoin mining internally rewards good miners and punishes bad miners, so that the Bitcoin user can safely trust the Bitcoin network as a whole.

## ...that existing blockchains don't really have

Unfortunately, _existing blockchains have weak endogenous trust!_ There are many reasons, including the fact that endogenous trust is often totally ignored as a design goal (e.g. in "private blockchains"), but the biggest problem is **application-blockchain friction**. That's when a blockchain protocol becomes increasingly unsuitable for its main applications and loses its users' confidence.

Usually, this forces a contentious out-of-band protocol upgrade to prevent the blockchain from passing into the dustbin of history. The Bitcoin block-size controversy is the most well-known case of loss of trust from application-blockchain friction --- unsurprising given Bitcoin's rigid coupling of its core payment application to its blockchain. Unfortunately, general-purpose blockchains like Ethereum experience even more challenges to their endogenous trust due to application-blockchain friction. In fact most of the protocol upgrades to Ethereum so far involved fairly minor tweaks to functionality in order to support newly emerging, unanticipated applications.

The prevalence of application-blockchain friction is because both "application blockchains" (like Bitcoin) and "platform blockchains" (such as Ethereum) are are _on the wrong protocol layer_. Both are _too close to applications_.

Platform blockchains like Ethereum sit directly underneath applications to allow apps' easy deployment --- a new cryptocurrency can be implemented on Ethereum in a few dozen lines of code. Such a direct interface between application and blockchain, however, inevitably results in contention between ever-changing application requirements and ideally immutable blockchain protocols. The situation is analogous to telecommunication networks before the Internet, where a vertically integrated system directly offered relatively high-level functions like voice calling and teletype. Like Ethereum, these complex platforms were extremely costly to upgrade when they were forced to change by the rise of new applications and technological advances.

## The solution: Themelio

Learning from previous these mistakes, it is clear that a blockchain with endogenous trust must be built upon a solid cryptoeconomic foundation. This ensures that its core security properties will require no out-of-band social coordination to uphold.

More importantly, it must also minimize application-blockchain friction by using a minimal, low-level protocol with straightforward semantics designed for indefinite deployment. This allows easily upgraded "middleware" protocols to separate the blockchain from the ever-changing needs of applications.

This points us towards a new blockchain paradigm --- a **layer-0 endogenous trust engine**, acting as a bare-bones root-of-trust infrastructure for supporting endogenous trust in decentralized applications. Rather than looking like an application or platform, takes inspiration from the technology underpinning most of modern telecommunication: the Internet Protocol (IP). IP gets packets on a best-effort basis from point A to point B, and nothing more. Unreliable datagrams don't make a developer-friendly interface, but they do provide a firm foundation for ever-changing application protocol stacks.

IP is a great illustration of a successful foundational technology. Such protocols are often too simple to support rich applications without intervening protocol stacks, but they are easy to conceptualize, simple to implement, and brutally robust. It is precisely this simplicity that allows it to support the dazzling variety of Internet applications today with practically no changes since the IPv4 specification's publication in 1981.

## Learn more

How does Themelio achieve this goal? We are now ready to dive into three crucial pillars of Themelio's design:

- Themelio's [**state model**]({{< ref "02-state.md" >}}) --- the basic abstraction of consistent global state that transactions mutate --- is different from all existing blockchains. Themelio rejects the "stateful accounts" model of Ethereum and most existing platform blockchains in favor of a seemingly primitive model --- a "coin-based" (also known as "UTXO-based") transaction graph like that of Bitcoin. Combined with a few key innovations, such as a highly expressive scripting language, this model turns out to provide a nearly ideal abstraction for rooting endogenous trust.

- Synkletos, Themelio's [**consensus game**]({{< ref "03-consensus.md" >}}), or the core consensus algorithm and its attendant incentive mechanism. The consensus game of a blockchain is key to its endogenous trust, yet existing blockchains tend to depend on rather shaky cryptoeconomic assumptions, like non-coordination assumptions. In Synkletos, on the other hand, the optimal strategy for consensus participants is insensitive to collusion --- in fact, Synkletos tries to simulate a perfectly colluding cartel. Combined with Streamlet, a stripped-down Byzantine-tolerant consensus protocol, Synkletos makes expectations of extremely long-term security grounded without the need for exogenous interventions when consensus fails.

- Finally, [**cryptocurrency stability**]({{< ref "04-melmint.md" >}}) is crucial for endogenous trust. Decentralized finance and mechanism design is highly impaired without a good unit of value, and it speaks volumes that current DeFi applications heavily rely on price oracles, fiat-pegged stablecoins, and other mechanisms that lack endogenous trust. Themelio's base currency, the Mel, is a groundbreaking _endogenous stablecoin_ that has low purchasing-power volatility through pegging to an trustlessly computed index. Unlike all existing cryptocurrencies, Mel achieves the ``holy grail'' of both endogenous trust and stability.
