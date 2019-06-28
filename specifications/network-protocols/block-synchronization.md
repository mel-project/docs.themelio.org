# State synchronization

## Overview

The **statesync** protocol is used whenever two nodes need to synchronize their picture of the blockchain. It's used to keep the protochain in Symphonia loosely synchronized, as well as to make sure all auditors have an up-to-date view of the blockchain. Statesync does not specify how the two nodes synchronizing connect to each other; that's the trouble of the upper-layer protocol like Symphonia.

## Starting the sync

### Initial handshake

After connecting, both parties **simultaneously** send an initial handshake message:

```go
type msgHandshake struct {
    // Require state to be fully committed
    ReqCommit bool
    // Height of "tip" block (simply highest if ReqCommit)
    TipHeight uint 
    // Hash of "tip" block
    TipHash celcrypt.Bytes32
}
```

After the handshake, the node with higher TipHeight becomes **sender**, and the other **receiver**. If TipHeight is identical, the node with higher TipHash is sender. 

If TipHash is identical, exit the protocol \(nothing to sync\). If ReqCommit isn't identical, exit the protocol \(we're wanting two different things\).

### Confirming metadata

The sender then sends **sync metadata** to the receiver:

```go
type msgMetadata struct {
    // is the receiver tip an ancestor of the sender tip?
    IsAncestor bool
    // proof associated with tip
    TipProof []byte
}
```

`TipProof` is an app-dependent proof about the tip. In Symphonia it's a signed 

The receiver then decides its course of action:

* If the tips are too far apart, and we fully trust the tip proof, then do a **fast sync**. 
* If the tips are too far apart but `ReqCommit == false`, then abort. There's nothing that can be done.
* Otherwise:
  * If IsAncestor, do a **batch sync**
  * Otherwise, do a **one-by-one sync**.

## Fast sync

A fast sync directly synchronizes the sender's state to the receiver, bypassing block validation entirely. The receiver assumes the honesty of the stakeholder quorum entirely.

{% hint style="danger" %}
**TODO: requires full spec of consensus sync info to proceed**
{% endhint %}

## Batch sync

During a batch sync, all the missing blocks will be sent at once.

The receiver sends a request that's a simple string `batch`.

The sender then simply spams a bunch of `chainstructs.Block` to the client.

The sender sends the messages from **higher to lower block numbers**. This allows the client to verify that each block correctly commits to its ancestor. 

## One-by-one sync

During one-by-one sync, clients request specific blocks.

Clients request a block by sending its hash as a single RLP value, while senders just respond by sending a single `chainstructs.Block`. 

Clients should give up under memory pressure to prevent DoS.

## 



### 

