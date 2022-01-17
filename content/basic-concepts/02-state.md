---
title: "Coins with superpowers"
weight: 2
draft: false
# search related keywords
keywords: [""]
---

## Coins are perfect for blockchains

Fundamentally, blockchains are _state machines_, and Themelio implements an extended UTXO- or coin-based state model. (If you're not familiar with blockchains as state machines or the coin-based model, check out our [quick explanation](https://medium.com/themelio/utxos-vs-accounts-54b3bbeb4428).) But coin-based models are not popular among general-purpose blockchains. Most blockchains attempting to support general decentralized apps use account-based models that directly map owners to sums of money; even "smart contracts" are implemented essentially as accounts owned by on-chain code. Why, then, did we choose coins?

Let us return to the core function of blockchains: providing endogenous roots of trust. In other words, blockchains' mission is to replace centralized roots of trust with decentralized, autonomous alternatives that are much more secure by nature.

It turns out that centralized roots of trust, such as notaries, certificate authorities, and banks, almost always serve one role: ensuring the consistency of a graph of interdependent events. This is because a very large class of security-critical problems boils down to establishing a consistent, valid event graph. For example, a money transfer from Alice to Bob depends on the previous transfers or deposits that gave Alice enough balance to make the transfer to Bob. In a naming system, a successful name transfer also depends on a series of previous events: the previous owner relinquishing control, the new owner registering the name, and so forth. For a PKI, one CA issuing a valid certificate for a domain depends on the absence of conflicting histories of other CAs issuing other certificates for that same domain.

The coin-based blockchain model is extremely suitable for establishing event dependencies: its transaction DAG **is** an event graph. So using coins makes it easy to write decentralized apps that replicate centralized trust authorities on Themelio. In fact, this also gives huge performance benefits:

- **Coins allow Themelio to process transactions in parallel.** Unlike account-based models (e.g., in Ethereum), the coins model does not require total global transaction ordering. We only need to process the transaction that produces a coin before the transaction that spends it. Within the same block, even this constraint is removed: validation can happen [entirely in parallel]({{< relref path="../specifications/yellow.md#batch-applying-transactions" >}} "Yellow Paper"), greatly increasing transaction throughput.
- **Elegant state transitions reduce error and improve performance.** Account-based blockchains like Ethereum have complex, arbitrarily mutable state that slows performance and makes building apps on them notoriously difficult. In a coin-based blockchain, state is extremely simple: the set of all unspent coins. Transitions simply correspond to transactions atomically deleting and adding coins. This means significantly clearer logic in decentralized apps and faster blockchain performance.

## Two superpowers of Themelio coins

However, traditional coin-based architectures (like Bitcoin’s) have trouble supporting a wide variety of decentralized apps. These blockchains’ interfaces makes it very difficult for applications other than the blockchain itself to use them as roots of trust. Themelio makes two significant additions to the coin-based model that makes endogenous trust **transferrable**:

- **Expressive covenant scripting**: MelVM is a scripting environment for complex coin covenants more powerful than any traditional coin-based blockchain’s scripting language. With self-propagating covenants, MelVM even enables constructs analogous to stateful smart contracts in account-based blockchains, despite the lack of any mutable storage.

- **Coin-oriented thin client interface**: Strangely, existing coin-based blockchains don’t explicitly include coins in the model that applications can trustlessly query. Blockchain users must download the entire transaction history to do anything: without full blockchain access, it’s not even possible to securely obtain simple facts like “is this coin still unspent”.

  This means that simple apps like cryptocurrency wallets cannot be secure and scalable at the same time. Either every user downloads the huge and growing transaction history, or we need to trust a central server to provide the information. This is true even with technologies like Bitcoin’s SPV: it lets clients verify claims that a certain transaction exists but is unable to defend against dishonest nodes that hide information, therefore rendering basic wallet information untrustworthy.

  In contrast, Themelio explicitly represents the coin state in its protocol to sidestep all these issues. Sparse Merkle trees committing to the entire coin state allow thin clients to securely obtain information about coins without trusting anyone. Even full nodes can function by only synchronizing the coin state rather than the entire blockchain history, with huge benefits to storage efficiency. As a result, coin-driven applications, from simple wallets to complex covenant-driven apps, can scale without any centralized trust.
