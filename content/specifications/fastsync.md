---
title: "Fastsync protocol"
date: 2018-12-29T11:02:05+06:00
lastmod: 2020-01-05T10:42:26+06:00
weight: 112

draft: false
# search related keywords
keywords: [""]
---

The **fastsync** protocol is used by both stakers and auditors to catch up to the latest confirmed `SealedState` without starting from the genesis block and applying block after block.

This is done through a two-step process:

- The "catcher-up" does a `InitFastSync` request. The server responds with a `host:port` where the fastsync protocol will be run.
- The catcher-up connects to `host:port` over TCP and initiates the fastsync protocol. This consists of the server serializing the entire state and streaming it, while the client receives.

## Serializing a `SealedState`

Serialization of a sealed state is rather straightforward. First, the `Header` is sent over the wire. Then, each `SmtMapping` is serialized.

How do we serialize a `SmtMapping`? We simply traverse the entire SMT from left to right, sending over

```rust
pub struct SmtEntry {
    value: Vec<u8>,
    proof: CompressedProof
}
```

one after the other.

The receiver, for its part, checks each proof against the header and gradually builds a SMT.

## Compression

One potential problem is that the Merkle proofs take up a lot of space in each entry. Fortunately, adjacent Merkle proofs are generally very similar, so we can simply apply encapsulate the entire stream in DEFLATE. This should take care of compressing away the redundancies.
