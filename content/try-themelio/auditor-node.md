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
$ themelio-node --listen 127.0.0.1:11814
```

`themelio-node` strives to have sane default options, and the only required argument is where to listen to for RPC calls. By default, it connects to a few default bootstrap nodes, uses mainnet validation rules, and saves blocks to disks in `/tmp/themelio-mainnet`. You should see output similar to the following:

```text
May 04 21:16:14.505  INFO run_node: themelio_node::tasks::node: themelio-core v0.1.0 initializing...
May 04 21:16:14.610  INFO run_node: themelio_node::tasks::node: bootstrapping with [51.83.255.223:11814, 188.227.35.45:11814]
May 04 21:16:14.934 DEBUG blksync_loop{netid=Mainnet}: themelio_node::protocols::node: got 11 blocks from other side
May 04 21:16:14.938 DEBUG blksync_loop{netid=Mainnet}: themelio_node::services::storage: block 14060, txcount=0, hash=#<ee289b25363d3ac6a286871dd2ad806ffe14a76054530cbf15074a5dd4b2671b> APPLIED
May 04 21:16:14.940 DEBUG blksync_loop{netid=Mainnet}: themelio_node::services::storage: block 14061, txcount=0, hash=#<3606533419294df58b345d9d50a312aec14cdd9edc5be6b6f64af0ec6ca07ced> APPLIED
May 04 21:16:14.941 DEBUG blksync_loop{netid=Mainnet}: themelio_node::services::storage: block 14062, txcount=0, hash=#<9a3ff17ddb3c1421c8c1db0d89014019a312805e1521896aebec7f5f0a364ee6> APPLIED
May 04 21:16:14.942 DEBUG blksync_loop{netid=Mainnet}: themelio_node::services::storage: block 14063, txcount=0, hash=#<0772babbdbde5323bdd03801f5b404bb5171e573d3d40025c6320885e6e7e0ce> APPLIED
May 04 21:16:14.944 DEBUG blksync_loop{netid=Mainnet}: themelio_node::services::storage: block 14064, txcount=0, hash=#<d48f31997766c0ba418d70e2d37b93a7fac1d5801efc8e1affbcdd2d5cff2ef9> APPLIED
May 04 21:16:14.945 DEBUG blksync_loop{netid=Mainnet}: themelio_node::services::storage: block 14065, txcount=0, hash=#<0b518878e979bbb11a60bf000adc113715198415a198c96dfce92c7eda59446b> APPLIED
May 04 21:16:14.947 DEBUG blksync_loop{netid=Mainnet}: themelio_node::services::storage: block 14066, txcount=0, hash=#<b6ea0339aa691f9c3071ac08693f6bc394266e449116ce776ddb7be36a86e272> APPLIED
May 04 21:16:14.948 DEBUG blksync_loop{netid=Mainnet}: themelio_node::services::storage: block 14067, txcount=0, hash=#<892a6b4dac6de4d2ff346c4be74dde641392abcca63bd01b687650fc2a225916> APPLIED
May 04 21:16:14.949 DEBUG blksync_loop{netid=Mainnet}: themelio_node::services::storage: block 14068, txcount=0, hash=#<d7a15bcbed22d63280b0bd930f2f0b840d04309528ab3a506de05e36bf2ed102> APPLIED
May 04 21:16:14.950 DEBUG blksync_loop{netid=Mainnet}: themelio_node::services::storage: block 14069, txcount=0, hash=#<f94c6d2cb6bbcd94807dbae7cc462d02452d07154315cba7db7492640bad4e19> APPLIED
May 04 21:16:14.952 DEBUG blksync_loop{netid=Mainnet}: themelio_node::services::storage: block 14070, txcount=0, hash=#<09378e717f9f1eb22524afdc543cd75fbe9621d52b96c26e32ee61c3998bd1a8> APPLIED

```

Your auditor is now running and replicating blocks within the auditor peer-to-peer gossip network.

### On the non-persistent testnet

To run the auditor on the _non-persistent_ testnet, where most covenant development and testing will happen during the betanet period, run instead

```text
$ themelio-node --listen 127.0.0.1:11814 --bootstrap tm-1.themelio.org:11814 --testnet --database /tmp/testnet
```

Note that three things were needed to connect to the testnet:

- Connecting to a testnet bootstrap node
- Specifying `--testnet`, to use testnet validation rules
- Specifying a non-default database path to ensure that the blocks are not confused with mainnet blocks on disk.

## Running a staker

_Right now, the staker network is not yet open to public participation. This section will be updated with more detailed information once anyone can become a staker._

## Other options

One can see all the options for running a Themelio auditor node as follows:

```text
$ themelio-node --help
themelio-node 0.1.0

USAGE:
    themelio-node [FLAGS] [OPTIONS] --listen <listen>

FLAGS:
    -h, --help       Prints help information
        --testnet    If set to true, default to the testnet. Otherwise, mainnet validation rules are used
    -V, --version    Prints version information

OPTIONS:
        --bootstrap <bootstrap>...
            Bootstrap addresses. May be given as a DNS name [default: mainnet-bootstrap.themelio.org:11814]

        --database <database>                        Database path [default: /tmp/themelio-mainnet]
        --listen <listen>                            Listen address
        --override-genesis <override-genesis>
            If given, uses this TOML file to configure the network genesis rather than following the known
            testnet/mainnet genesis
        --staker-bootstrap <staker-bootstrap>...     Bootstrap addresses for the staker network
        --staker-listen <staker-listen>              Listen address for the staker network
        --staker-payout-addr <staker-payout-addr>    Payout address for staker rewards
        --staker-sk <staker-sk>                      Specifies the secret key for staking
```
