---
title: "Coins with superpowers"
weight: 2
draft: false
# search related keywords
keywords: [""]
---

## Old-fashioned coins

Blockchains are fundamentally _state machines_: there is a **world state** that represents the current status of the global ledger, and **transactions** _transition_ one state to another. For example, in an account-based blockchain like Ethereum, the world state consists of accounts with balances, and a transaction that moves 1 ETH from Alice's to Bob's account transitions the state by reducing Alice's balance and increasing Bob's balance by 1 ETH.

Unlike Ethereum, though, Themelio's state does not consist of accounts and balances. Instead, its basic state model belongs to a family usually known as "UTXO-based" or **coin-based** models. This is the oldest family of blockchain models, including first-generation blockchains like Bitcoin and Litecoin. In Themelio's coin-based model, the world state consists of a pool of **coins**, each one represents a given amount of cryptocurrency, known as its **value**, and it includes a **covenant** that specifies what sort of transaction can spend the coin.

Transactions transition the state by _spending_ coins, deleting them from the state, and _creating_ new coins in the state. Because spending a coin deletes it from the state, each coin can only be spent once, and the sum of the values of all the coins spent by a transaction must equal the sum of the values of all the coins created by it. Covenants can often be thought of as representing "ownership" of coins: a "this coin is owned by Alice" covenant simply checks that any spending transaction is cryptographically signed by Alice. "All the coins Alice owns" is thus another way of saying "all the coins Alice knows how to spend". The following table contrasts Themelio's coin-based model with Ethereum (and many other blockchains') account-based model:

|                     | Coin-based                                   | Account-based                         |
| ------------------- | -------------------------------------------- | ------------------------------------- |
| **World state**     | Set of coins, each with a value and covenant | Set of accounts, each with a balance  |
| **Transactions**    | Delete and create coins                      | Deduct one account and credit another |
| **Asset ownership** | Implicit in covenant logic                   | Explicitly attached to accounts       |

As we will see soon, coins are a very elegant and highly performant abstraction, but there's no highly precise analogy to these coins in the physical world, so let's use a simple example to illustrate how coin-based transactions work in practice. Suppose all Alice owns (i.e. knows how to spend) is a coin with 10 MEL in it, and wishes to send 1 MEL to Bob. She will create a transaction that

- Spends that coin, supplying whatever signatures, etc needed to satisfy its covenant
- Creates a coin worth 1 MEL, attached to a covenant that only Bob can satisfy
- Creates a coin worth 9 MEL, attached to a covenant that only Alice can satisfy

By doing so, Alice deletes a 10-MEL coins from the state, splitting it into a 1-MEL coin in Bob's ownership and a 9-MEL coin in her own ownership. The latter _change coin_ is required because coins can only be spent in their entirety.

## Why coins?

But coin-based models are not popular at all among general-purpose blockchains. Most blockchains attempting to support general decentralized apps use account-based models that directly map owners to sums of money that can be transferred, while "smart contracts" are implemented essentially as accounts owned by on-chain code. Why do we believe coins are the way to go?

First of all, coins allow Themelio to _process transactions in parallel_. In an account-based model, like in Ethereum or traditional banking, strict global transaction ordering is necessary. Yet coin-based transactions can be processed in a much freer order — we simply need to process the transaction that produces a coin before the transaction that spends it. In fact, within the same block even this constraint is removed and validation can happen entirely in parallel, due to a happy consequence to _canonical transaction ordering_, an idea that first emerged as a [proposed modification to Bitcoin Cash](https://blog.vermorel.com/pdf/canonical-tx-ordering-2018-06-12.pdf). This greatly increases transaction throughput.

Secondly, a coin-based architecture **simplifies state transitions**. To support functionality beyond basic payments, account-based blockchains like Ethereum generally use arbitrarily mutable state, accessed by user-programmable “smart contracts”. However, programming decentralized apps with mutable state is notoriously prone to error. Complex state transitions are associated with difficult-to-find bugs and blockchain-level performance problems. In a coin-based blockchain, state is extremely simple: the set of all unspent coins. All transitions simply correspond to individual transactions deleting and adding coins atomically. This leads to clearer logic in decentralized apps and faster performance.

Finally, coin-based transactions are **surprisingly expressive**. A very large class of security-critical problems boils down to establishing a consistent, valid graph of interdependent events. For example, in a naming system, a successful name transfer depends on previous events like the previous owner relinquishing control, that owner first registering the name, and so forth. Centralized roots of trust, like notaries, certificate authorities, and banks, almost always serve the role of ensuring consistency of an event graph. In a coin-based blockchain model, the transaction DAG maps extremely well to these event graphs. This means it’s easy to write decentralized apps that replicate centralized authorities on Themelio's coin-based model.

## Two crucial superpowers

However, traditional coin-based architectures exactly like Bitcoin clearly cannot support a wide variety of decentralized apps. Otherwise, why would anybody use other blockchains? Themelio refines the traditional coin-based model with two significant changes:

- **Expressive covenant scripting**: Themelio uses [MelVM]({{<ref "../specifications/melvm-specification.md" >}}), a scripting language that allows complex covenants to be attached to coins in the Themelio state. MelVM allows much more powerful constraints on what transaction can spend a coin compared to traditional coin-based scripting languages like Bitcoin scripts.

  In particular, self-referential covenants that enforce that the same covenant is propagated to the output coins of the spending coin can be used to build powerful trustless constructs similar to stateful smart contracts in account-based blockchains, but taking full advantages of a coin-based model.

- **Coin-oriented application interface**: Strange as it may seem, existing coin-based blockchains don’t actually have coins explicitly in the model that applications see. Instead, blockchain users have to download the entire transaction history, building a transaction DAG and working out which transactions spent which coins by themselves. Without full blockchain access, it’s not even possible to securely obtain simple facts like “is this coin still unspent”.

  This means that in existing coin-based blockchains, even simple applications like cryptocurrency wallets can’t be secure and scalable at the same time. Either every user downloads the huge and growing transaction history, or a centralized server that does sync the blockchain is trusted to provide users with information. This is true even with technologies like Bitcoin’s SPV that let clients verify claims that a certain transaction exists. SPV is unable to defend against dishonest nodes that hide coins or claim that spent coins are unspent, rendering basic wallet information untrustworthy.

  In Themelio, though, the coin-based world state is first-class citizens. Participants synchronize the coin state, not the entire blockchain history. Sparse Merkle trees committing to the entire world state allow thin clients to securely obtain information about coins without trusting anyone. Apps see the coin state as a secure database they can freely query. Thus, coin-driven applications, ranging from simple wallets to complex covenant-driven apps, can scale without needing any centralized trust.
