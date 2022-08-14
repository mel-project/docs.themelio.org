---
title: "Glossary"
date: 2018-12-29T11:02:05+06:00
lastmod: 2020-01-05T10:42:26+06:00
weight: 112

draft: false
# search related keywords
keywords: [""]
---

**State elements** constitute the components of the **world state**

- **Blocks** are defined by their constituent **transactions**, optionally with a consensus proof
- **Consensus proofs** are attached to blocks, turning them into **confirmed** blocks.
- The world state contains **history** and **coin state**

---

Each **transaction** has:

- **Transaction inputs**, each of which spends a **coin** \(mapping in the coin state\) by **unlocking** its covenant
- **Transaction outputs**, each of which has a **value**, a **denomination**, and a MelVM **covenant**
- **Attached data**

---

Currencies:

- The base currency is "**mel**", but the ticker symbol is MEL. This is like "ether" vs "ETH".
  - _We accept mel, Bitcoin, and other cryptocurrencies._
  - _The burger cost 4 mel_
- Decimal SI prefixes can be used.
  - _A burger and a side of fries costs 6 mel and 33 centimel._
  - _The smallest unit of money on Themelio is 1 micromel._
- **SYM** and **sym** is similar.
- Actual usage that emerges will probably be different and will be "correct".

---

Algorithms:

- The PoS system as a whole is **Synkletos**.
- **Melmint** is the cryptocurrency issuance mechanism. It's not a currency.
  - _In the last week Melmint minted a record 3 megamel onto the Themelio blockchain, half of which were locked up and reissued as ERC-20 tokens._
