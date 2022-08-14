---
title: "Running a full node"
date: 2018-12-29T11:02:05+06:00
lastmod: 2020-01-05T10:42:26+06:00
weight: 3
draft: false
# search related keywords
keywords: [""]
---

This document will guide you through setting up a **full node**. Full nodes replicate every consensus-confirmed block, validating its contents and ensuring network security while providing a local cache of the entire blockchain state. Running a full node helps contribute to the security and (read) performance of the network.

There are two kinds of full nodes:

- **Auditor** nodes comprise the vast majority of full nodes. They replicate and verify blocks but do not vote in consensus.
- **Staker** nodes, the ultimate guardians of Themelio security, have Sym locked up and participate in consensus. They are analogous to miners in proof-of-work blockchains like Bitcoin.

## Running an auditor

### On the persistent mainnet

To run an auditor on the "mainnet" (which at the moment is far from stable, but does have a persistent history), just run:

```text
$ themelio-node --listen 127.0.0.1:11814 --database ~/.themelio-blocks
```

`themelio-node` strives to have sane default options, and by default, it connects to a few default bootstrap nodes and uses mainnet validation rules. You should see output similar to the following:

```text
2022-05-15T02:20:51Z INFO  themelio_node] themelio-core v0.6.6 initializing...
[2022-05-15T02:20:51Z DEBUG themelio_node::args] database opened at /home/user/.themelio-blocks
[2022-05-15T02:20:51Z INFO  themelio_node::storage::storage] HIGHEST AT 0
[2022-05-15T02:20:51Z DEBUG themelio_node::args] node storage opened
[2022-05-15T02:20:51Z INFO  themelio_node] bootstrapping with [146.59.84.29:11814]
[2022-05-15T02:20:52Z DEBUG themelio_node::storage::storage] applied block 1 of length 215 in 0.92ms (insert 0.23ms)
[2022-05-15T02:20:52Z DEBUG themelio_node::storage::storage] applied block 2 of length 215 in 0.58ms (insert 0.18ms)
[2022-05-15T02:20:52Z DEBUG themelio_node::storage::storage] applied block 3 of length 215 in 0.58ms (insert 0.24ms)
[2022-05-15T02:20:52Z DEBUG themelio_node::storage::storage] applied block 4 of length 215 in 0.45ms (insert 0.17ms)
[2022-05-15T02:20:52Z DEBUG themelio_node::storage::storage] applied block 5 of length 215 in 0.46ms (insert 0.18ms)
[2022-05-15T02:20:52Z DEBUG themelio_node::storage::storage] applied block 6 of length 215 in 0.45ms (insert 0.18ms)
[2022-05-15T02:20:52Z DEBUG themelio_node::storage::storage] applied block 7 of length 215 in 0.46ms (insert 0.18ms)
[2022-05-15T02:20:52Z DEBUG themelio_node::storage::storage] applied block 8 of length 215 in 0.51ms (insert 0.13ms)
[2022-05-15T02:20:52Z DEBUG themelio_node::storage::storage] applied block 9 of length 215 in 0.45ms (insert 0.07ms)
[2022-05-15T02:20:52Z DEBUG themelio_node::storage::storage] applied block 10 of length 215 in 0.45ms (insert 0.07ms)
[2022-05-15T02:20:52Z DEBUG themelio_node::storage::storage] applied block 11 of length 215 in 0.46ms (insert 0.06ms)
[2022-05-15T02:20:52Z DEBUG themelio_node::storage::storage] applied block 12 of length 215 in 0.48ms (insert 0.08ms)
[2022-05-15T02:20:52Z DEBUG themelio_node::storage::storage] applied block 13 of length 215 in 0.47ms (insert 0.06ms)
[2022-05-15T02:20:52Z DEBUG themelio_node::storage::storage] applied block 14 of length 215 in 0.46ms (insert 0.06ms)
[2022-05-15T02:20:52Z DEBUG themelio_node::storage::storage] applied block 15 of length 215 in 0.46ms (insert 0.08ms)

```

Your auditor is now running and replicating blocks within the auditor peer-to-peer gossip network.

### On the non-persistent testnet

To run the auditor on the _non-persistent_ testnet, where most covenant development and testing will happen during the betanet period, run instead

```text
$ themelio-node --listen 127.0.0.1:11814 --bootstrap tm-1.themelio.org:11814 --testnet
```

Note that three things were needed to connect to the testnet:

- Connecting to a testnet bootstrap node
- Specifying `--testnet`, to use testnet validation rules

## Running a staker

_Right now, the staker network is not yet open to public participation. This section will be updated with more detailed information once anyone can become a staker._

## More details

A complete guide to `themelio-node`, including functionality on running "simnets" for testing blockchain functionality entirely over a local network, can be found at its [GitHub repo](https://github.com/themeliolabs/themelio-node)
