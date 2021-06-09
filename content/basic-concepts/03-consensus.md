---
title: "Collusion-tolerant consensus"
weight: 3
draft: false
# search related keywords
keywords: [""]
---

## Incentive-compatible consensus is hard

When describing the [state model]({{<ref "02-state.md">}}), we imagined Themelio as if it were a magic oracle that always correctly maintains the world state and applies updates to it every block. In the real world, though, we require a procedure to ensure both _consistency_ -- all Themelio users observe the same world state at every block height — and _validity_ — the world state can only change through valid transactions. In fact, consensus is ultimately what drives the [all-important endogenous trust]({{<ref "01-endogenous-trust.md">}}) of blockchains.

Unfortunately, consensus is also a source of major, headache-inducing problems. Contentious blockchain governance problems are often tied to consensus-related issues such as decentralization (for example, the controversy surrounding Bitcoin block sizes), and attacks on existing blockchains, such as the 51% attack on Namecoin, selfish mining attacks on Bitcoin, etc, focus on exploiting problems in consensus.

An important fact to note is that _blockchain consensus is unlike consensus in other distributed systems_. Because it aims at endogenous trust, it must rigorously model incentives, rather than simply making assumptions about fault tolerance. To truly achieve incentive-compatible endogenous trust, we cannot rely on the typical approach of considering ideal "honest" behavior and then positing an "adversary" with certain powers.

Yet cryptoeconomic mechanism design is a relatively young field full of uncertainties — game-theoretical models often give results different from actual empirical observations — and creating a consensus mechanism that is incentive-compatible under a wide variety of real-world conditions has proven to be perniciously difficult.

## Simulating a monopoly with Synkletos

With its Synkletos consensus game, the details of which are available as a [design whitepaper]({{<ref "../whitepapers/synkletos.md">}}) and a [concrete specification]({{<ref "../specifications/consensus-spec.md">}}), Themelio takes a unique, "economics-first" approach to designing consensus and related cryptoeconomic mechanisms. We start by introducing a very pessimistic model: a _despotic blockchain_ totally controlled by one rational profit-maximizing entity. We consider what sort of blockchain rules would such a despot enforce, in the absence of external attackers. Perhaps surprisingly, we find that a rational monopoly would in fact behave in a trustworthy manner --- the problem with centralized systems that leads to trust failures is generally "irrationality", not monopoly.

This insight lets us design Themelio’s consensus game. In Synkletos’ design whitepaper , we show that a variant of proof-of-stake with a few critical departures from existing designs elegantly incentivizes a large, permissionless group of stakeholders to simulate this ideal rational monopoly as a whole, regardless of whether they coordinate their actions or not. The most notable departure is the presence of a "fee cartel" influenced by EIP-1551, where the protocol charges a minimum fee for transactions as a part of the transaction validation rules, and this minimum fee is sequentially voted upon by stakeholders, converging to the median of their votes. Such a fee cartel also eliminates the well-known incentive incompatibility of a blockchain funded entirely through auction-based transaction fees, as well as stabilizing fee levels.
