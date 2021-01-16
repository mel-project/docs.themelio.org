---
title: "Synkletos"
date: 2018-12-29T11:02:05+06:00
lastmod: 2020-01-05T10:42:26+06:00
weight: 9
draft: false
# search related keywords
keywords: [""]
---
# Synkletos: Themelio's collusion-resistant consensus mechanism

## Abstract

In this whitepaper, we discuss Synkletos, the core cryptoeconomic mechanism that powers Themelio's proof-of-stake consensus. Consensus algorithms used to secure public blockchains differ significantly in their design constraints from those driving traditional fault-tolerant distributed systems. This is because blockchains must rigorously model trust within a game-theoretical model, rather than simply making assumptions about fault tolerance. To truly achieve incentive-compatible security, we cannot rely on the typical approach of considering ideal honest behavior and then positing an adversary with certain powers. Unfortunately, game-theoretical analysis of multi-party coordination problems, of which blockchain consensus is an instance, tends to be pernicuously difficult, leading to most consensus algorithms in use lacking formal analysis.

Synkletos is a blockchain consensus algorithm designed through a novel approach that drastically simplifies incentive analysis. Instead of modeling the ideal blockchain as decentralized parties participating in a coordination game to produce a certain optimal behavior, we start by proposing an ideal _monopoly_ blockchain, where blockchain rules are such that even a selfish monopoly will behave in a "faulty" way. We then design an simple incentive structure and consensus algorithm based on a variation of proof of stake that incentivizes any number of uncoordinated or coordinated parties to simulate such a monopoly as a whole. We argue that such a mechanism, though it is slightly less economically efficient compared to systems such as Nakamoto consensus that rely on noncoordination assumptions, is much easier to analyze and far more robust to game-theoretical attacks. Finally, we validate this by constructing Synkletos for Themelio and running an agent-based incentive simulation.

{% file src="assets/synkletos.pdf" caption="Read full PDF" %}



