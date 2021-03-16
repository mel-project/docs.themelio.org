---
title: "Running an auditor node"
date: 2018-12-29T11:02:05+06:00
lastmod: 2020-01-05T10:42:26+06:00
weight: 9
draft: false
# search related keywords
keywords: [""]
---

# Running an auditor node

This document will guide you through setting up an **auditor**, which is the equivalent of a **full node** in most other blockchains. An auditor replicates every consensus-confirmed block, validating its contents and ensuring network security while providing a local cache of the entire blockchain state.

## Assumptions

All the instructions here assume that you have an up-to-date `themelio-node` binary installed. If not, simply install it with `cargo`:

```text
$ cargo install --git https://github.com/themeliolabs/themelio-core.git themelio-node
```

## Running the auditor

To run an auditor, just run:

```text
$ themelio-node node --bootstrap 94.237.109.116:11814
```

You should see output similar to the following:

```text
Dec 29 20:47:55.627  INFO run_main:run_node: themelio_core: themelio-node v0.1.0 initializing...
Dec 29 20:47:55.627  INFO run_main:run_node: themelio_core: bootstrapping with [94.237.109.116:11814]
Dec 29 20:47:55.627  INFO run_main:run_node:open_testnet{path="/tmp/testnet"}: themelio_core::storage: creating a testnet genesis state from scratch
Dec 29 20:47:56.526 DEBUG blksync_loop:apply_block: themelio_core::storage: apply_block at height 0 with 0 transactions
Dec 29 20:47:56.526 DEBUG blksync_loop:apply_block: themelio_core::storage: apply_block special case when height is zero
Dec 29 20:47:56.527 DEBUG blksync_loop:apply_block: themelio_core::storage: block 0, txcount=0, hash=#<f8dbd36a0f> APPLIED
Dec 29 20:47:57.377 DEBUG blksync_loop:apply_block: themelio_core::storage: apply_block at height 1 with 0 transactions
Dec 29 20:47:57.377 DEBUG blksync_loop:apply_block: themelio_core::storage: block 1, txcount=0, hash=#<c2182ca753> APPLIED
Dec 29 20:47:58.379 DEBUG blksync_loop:apply_block: themelio_core::storage: apply_block at height 2 with 0 transactions
Dec 29 20:47:58.380 DEBUG blksync_loop:apply_block: themelio_core::storage: block 2, txcount=0, hash=#<d741c12dcc> APPLIED
Dec 29 20:47:59.336 DEBUG blksync_loop:apply_block: themelio_core::storage: apply_block at height 3 with 0 transactions
Dec 29 20:47:59.337 DEBUG blksync_loop:apply_block: themelio_core::storage: block 3, txcount=0, hash=#<f6a0829cd2> APPLIED
Dec 29 20:48:00.285 DEBUG blksync_loop:apply_block: themelio_core::storage: apply_block at height 4 with 0 transactions

```

Your auditor is now running and replicating blocks within the auditor peer-to-peer gossip network.

**Note**: right now the auditor implementation is not very useful, since not much happens in the network that's interesting to replicate!
