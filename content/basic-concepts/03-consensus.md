---
title: "Collusion-tolerant consensus"
weight: 3
draft: false
# search related keywords
keywords: [""]
---

## Incentive-compatible consensus is hard

In the article on [endogenous trust]({{<ref "01-endogenous-trust.md">}}), we saw that cryptoeconomic mechanisms, not humans, constitute blockchains' root of trust. These mechanisms make up the blockchain's _consensus game_, the beating heart of any blockchain.

Unfortunately, consensus is also a source of major, headache-inducing problems. Contentious blockchain governance problems are often related to consensus (e.g., the controversy surrounding Bitcoin block sizes), and attacks on existing blockchains, such as the 51% attack on Namecoin and selfish mining attacks on Bitcoin, focus on exploiting problems in consensus.

Yet cryptoeconomic mechanism design is a young field full of uncertainties: game-theoretical models often produce results different from actual empirical observations. Consensus modeling in other distributed systems can't help us either, because blockchain consensus is unlike consensus in any other distributed system. Other distributed systems trust that a certain fraction of their participants are "honest", and their consensus defends against dishonest participants, whereas blockchains don't assume any participant's honesty--their consensus generates endogenous trust. Hence to truly achieve incentive-compatible endogenous trust, we cannot rely on the typical distributed systems approach of considering ideal honest behavior and positing "adversaries" with certain powers.

In sum, creating a consensus mechanism that is incentive-compatible under a wide variety of real-world conditions has proven to be perniciously difficult.

## Simulating a monopoly with Synkletos

Designing a consensus mechanism and then trying to make it incentive-compatible is incredibly error-prone. So in designing Themelio's consensus game [Synkletos]({{<ref "../whitepapers/synkletos.md">}}), we started with incentive compatibility and structured the entire consensus mechanism around that.

Bitcoin's experience has shown that it is unrealistic to assume the majority of participants will not collude. So we dropped this unrealistic assumption and aimed for a system that will behave the same no matter how many participants collude. To do so, we first posited a very "pessimistic" model: a _despotic blockchain_ entirely controlled by one rational profit-maximizing entity. We carefully considered what blockchain rules such a despot would enforce, in the absence of external attackers. Perhaps surprisingly, we discovered that a rational monopoly would behave in a trustworthy manner. This shows that the source of trust failures in traditional centralized systems is generally irrationality, not monopoly.

This insight enabled us to design an entirely collusion-resistant consensus game for Themelio. We prove in the [Synkletos whitepaper]({{<ref "../whitepapers/synkletos.md">}}) that a proof-of-stake design with a few critical innovations elegantly incentivizes a large, permissionless group of stakeholders to simulate an ideal rational monopoly, regardless of whether their actions are coordinated.

The most notable innovation is a new fee economy that allows Themelio stakers (i.e., consensus participants) to be funded entirely by fees rather than through block-reward inflation, thus aligning incentives between users and stakers. Through a "fee cartel" mechanism distantly inspired by EIP-1559, we eliminate the well-known incentive incompatibility of a blockchain being funded entirely through auction-based transaction fees, as well as drastically stabilizing fee levels.

Through a consensus game that stays incentive-compatible regardless of how and whether stakers decide to collude, Themelio ensures robust, long-term endogenous trust.
