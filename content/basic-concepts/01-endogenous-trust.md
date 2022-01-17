---
title: "Endogenous trust"
weight: 1
draft: false
# search related keywords
keywords: [""]
---

## The killer feature of blockchains…

Themelio’s entire design is motivated by what we believe is the most important feature of blockchains: **endogenous trust**. That is, _we can trust that a blockchain protocol will behave correctly without trusting any of the people running it_. In blockchains, trust emerges from within the protocol, not from preexisting trust in the parties that run the protocol.

Endogenous trust is enabled by cryptoeconomics, the intersection of game theory and cryptography, which powers self-incentivizing mechanisms that, given enough participants, allows us to trust a group’s overall behavior without trusting any individual participant. For example, someone can only securely use a traditional bank if they trust that bank, or at least third-party regulators. But a Bitcoin user doesn’t even need to know which miner ends up processing their transaction, let alone establish any sort of trust with that miner. Instead, because the cryptoeconomics of Bitcoin mining internally rewards good miners and heavily punishes bad miners, our Bitcoin user can confidently trust the Bitcoin network as a whole.

In fact, endogenous trust is the only property unique to public blockchains. Almost all other widely-touted blockchain properties can be found in non-blockchain systems that are often much more efficient than blockchains: BitTorrent has decentralization, Keybase has transparency, and TLS has security, to name a few. Endogenous trust is the key feature that gives blockchain apps superpowers elusive to pre-blockchain systems.

## …that existing blockchains don’t really have

Unfortunately, existing blockchains have weak endogenous trust. In fact, sometimes endogenous trust is totally ignored as a design goal, like in "private blockchains". Even in public blockchains, endogenous trust is lost every time a protocol is upgraded: governance events hand over the network’s behavior to people _external_ to the blockchain, making those decision-makers the ultimate root of trust.

Among the many factors that force blockchain protocol upgrades, the biggest culprit is **application-blockchain friction**, which occurs when a blockchain protocol becomes increasingly unsuitable for its main applications.

The resulting loss of user confidence usually forces a contentious out-of-band protocol upgrade to prevent the blockchain from passing into the dustbin of history. The most well-known example of loss of trust due to application-blockchain friction is the Bitcoin block-size controversy. General-purpose blockchains like Ethereum experience even more challenges to their endogenous trust due to application-blockchain friction: the Ethereum protocol has been upgraded many times for fairly minor tweaks in order to support unanticipated applications.

Application-blockchain friction is prevalent because blockchains are _on the wrong protocol layer_. Both “application blockchains” (like Bitcoin) and “platform blockchains” (like Ethereum) are _too close to applications_. Bitcoin rigidly couples its core payment application to its blockchain, as do other application blockchains; platform blockchains like Ethereum sit directly beneath applications to make immediate app deployment easy. Such a direct interface between application and blockchain, however, is a sure recipe for contention between ever-changing application requirements and ideally immutable blockchain protocols.

## The solution: Themelio

Learning from existing blockchains' failures, Themelio secures endogenous trust by getting two things right.

- First, Themelio is founded upon a uniquely solid cryptoeconomic mechanism: proof-of-stake that works even when [all participants selfishly collude](https://docs.themelio.org/basic-concepts/03-consensus/). This ensures that Themelio's core security properties will stay reliable in the long term without any form of external governance.
- More importantly, we minimize application-blockchain friction by placing Themelio further down the protocol stack. Easily upgraded middleware protocols separate the blockchain from the ever-changing needs of applications, giving the Themelio ecosystem both strong endogenous trust and great flexibility.

We are confident in this architectural strategy because it has already succeeded in powering most of today's telecommunication. Blockchains today are like telecom networks before the Internet: vertically integrated systems offering high-level functions like app-level VMs, voice calling, and teletype. Just like Ethereum, old telecom platforms needed costly, centralized upgrades when new applications and technologies forced them to change.

What replaced these integrated telecom systems was an architectural paradigm shift. Instead of a monolithic platform, the internet is a stack of ever-changing apps and transmission technologies sandwiching a simple, 40-year-old protocol. The Internet Protocol (IP) gets packets on a best-effort basis from point A to point B, nothing more. While IP's unreliable datagrams don't make for an application-friendly interface, they do provide a firm foundation for ever-changing application protocol stacks.

IP is a great illustration of a successful foundational technology. Such protocols are often too simple to support rich applications without intervening protocol stacks, but they are easy to conceptualize, simple to implement, and brutally robust. It is precisely this simplicity that allows IP to support the dazzling variety of Internet applications today with practically no changes since IPv4's publication in 1981.

![This is an image](/images/hourglass.png)

Inspired by IP, Themelio is a minimal, low-level protocol with straightforward semantics designed for indefinite deployment. Themelio brings a new architectual paradigm for blockchains: it is an immutable **layer-0 endogenous trust engine** that powers a flourishing system of decentralized applications.
