---
title: "Minting mels with melminter"
weight: 4
draft: false
# search related keywords
keywords: [""]
---

`melminter` is a tool that integrates with `melwalletd` to provide a convenient, CLI interface for participating in Melmint by minting DOSCs and converting them to Mel.

## Prerequisites

Before using `melminter`, ensure that you have:

- An up-to-date `melwalletd`
- Some wallet in `melwalletd` containing Mel, perhaps created through `melwallet-cli`

## Starting the melminter

The easiest way to run the melminter is to first start `melwalletd`, then simply run the following command:

```
melminter --connect <address of a full node> --daemon <where melwalletd is listening> --backup-wallet <name of an unlocked wallet>
```

For example, if we have a testnet wallet called `foobar` and a daemon listening on `127.0.0.1:11773`, we can run:

```
melminter --connect 185.70.196.63:11814 --backup-wallet foobar
```

(where `185.70.196.63:11814` is the address of a testnet node)

This will start the melminter:

```
Sep 16 21:48:19.288  INFO melminter: starting worker 1
Sep 16 21:48:19.293  INFO melminter: starting worker 2
Sep 16 21:48:19.297  INFO melminter: starting worker 3
Sep 16 21:48:19.300  WARN melminter: worker 3 does not have enough money, transferring money from the backup wallet!
Sep 16 21:48:19.332  WARN melminter: waiting for txhash TxHash(#<119d7d27264a1508308473a040a842528ccb8d445b092e52885ee8219de6070b>)...
Sep 16 21:48:24.342  INFO melminter::worker: ** [worker-1] My speed: 943.551 kH/s
Sep 16 21:48:24.342  INFO melminter::worker: ** [worker-1]  Max speed: 33.333 kH/s
Sep 16 21:48:24.342  INFO melminter::worker: ** [worker-1] Estimated return: 28.31 rDOSC/day
Sep 16 21:48:24.342  INFO melminter::worker: ** [worker-1] Selected difficulty: 32 (approx. 4551.9198208s / tx)
Sep 16 21:48:24.342 DEBUG melminter::worker: approx 4551.919714496s left in iteration
Sep 16 21:48:24.391  INFO melminter::worker: ** [worker-2] My speed: 938.475 kH/s
Sep 16 21:48:24.391  INFO melminter::worker: ** [worker-2]  Max speed: 33.333 kH/s
Sep 16 21:48:24.391  INFO melminter::worker: ** [worker-2] Estimated return: 28.15 rDOSC/day
Sep 16 21:48:24.391  INFO melminter::worker: ** [worker-2] Selected difficulty: 32 (approx. 4576.538281984s / tx)
Sep 16 21:48:24.391 DEBUG melminter::worker: approx 4576.538202338s left in iteration
```

## What does the melminter do?

In Melmint, _nominal DOSCs_ (nomDOSCs) are minted by creating a `DoscMint` transaction with a proof of sequential work over its inputs. This is an inherently sequential process, so `melminter` starts off by creating a bunch of different coins (in different wallets starting with `__melminter_`), spending them with `DoscMint` transactions with proof-of-sequential-work attached to generate nominal DOSC.

This nominal DOSC is then continually converted through Melswap into Mel, and used to pay transaction fees. Any excess is sent back to the backup wallet. All of this can be configured through `melminter` command-line arguments.

TODO: document this

## Caveats

### Minting may not be profitable

Minting nomDOSCs and converting them into mels is not always profitable. Because of the mechanics of Melmint, minting is _probably not_ profitable unless you have a top-of-the-line CPU and cheap electricity. Transient volatility in the mel/nomDOSC exchange rate may also affect profitability.

`melminter` makes no attempt at guessing whether or not minting is profitable.

### Wallet security

When `melminter`'s workers for some reason do not have enough funds to pay transaction fees, funds are transferred from the backup wallet. This is most commonly when `melminter` is started for the first time.

To transfer money successfuly, `melminter` requires the backup wallet to be unlocked, and will fail if it is locked. It's strongly recommended, however, that the backup wallet be kept locked unless `melminter` is being run for the first time, so that its private key is not exposed in memory for longer than needed.
