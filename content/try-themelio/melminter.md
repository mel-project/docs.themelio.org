---
title: "Minting mel with melminter"
weight: 4
draft: false
# search related keywords
keywords: [""]
---

`melminter` is a tool that integrates with `melwalletd` to provide a convenient, CLI interface for participating in Melmint by minting erg and converting to mel.

## Prerequisites

Before using `melminter`, ensure that you have:

- An up-to-date `melminter` and `melwalletd`. Both can be installed using `cargo`:

  ```
  cargo install --locked melminter
  cargo install --locked melwalletd
  ```

- Some wallet in `melwalletd` **already containing some mel** (at least 0.1 MEL), perhaps created through `melwallet-cli`

The second requirement may be surprising: how can minting mel require mel? This is because Melmint transactions are required to pay transaction fees just like any other transaction.

## Starting the melminter

The easiest way to run the melminter is to first start `melwalletd`, then simply run the following command:

```
melminter --backup-wallet <name of an unlocked wallet> [--daemon <where melwalletd is listening>]
```

For example, if we have a testnet wallet called `foobar` and a daemon listening on `127.0.0.1:11773`, we can run:

```
melminter --backup-wallet foobar
```

Note that we did not need to specify `--daemon`, because `127.0.0.1:11773` is the default value of `--daemon`.

This will start a cool TUI interface that looks like this:

![](/images/melminter.png)

## What is going on?

To understand what is happening in `melminter`, recall the [overall Melmint v2 procedure]({{< relref path="../whitepapers/melmint-v2.md" >}}) that aims to peg 1 MEL to 1 DOSC, or "day of sequential computation":

![](/images/melmint-v2-overview.png)

`melminter` first mints _erg_, a constant $k$ of which takes a DOSC of computation to create. This is done through creating a `DoscMint` transaction with a proof of sequential work over its inputs. Since this is an inherently sequential process, `melminter` starts off by creating a bunch of different coins, one for each CPU core, spending them with separate `DoscMint` transactions with proofs-of-sequential-work, created in separate threads, attached to generate nominal DOSC. This lets `melminter` fully utilize CPU resources.

(You can also reconfigure the number of threads used through the `--threads` argument)

This nominal DOSC is then continually converted through Melswap into mel, and used to pay transaction fees. Any excess is sent back to the backup wallet. All of this can be configured through `melminter` command-line arguments; `melminter -h` gives a full explanation.

## Caveats

### Minting may not be profitable

Minting erg and converting into mel is not always profitable. Because of the mechanics of Melmint, **minting is _probably not_ profitable** unless you either have a top-of-the-line CPU and cheap electricity, or Melmint is off-peg. This is because Melmint is not a proof-of-work consensus system, but rather a _pegging arbitrage_ system that is only really used to restore the MEL/DOSC peg.

Transient volatility in the mel/erg exchange rate may also affect profitability.

`melminter` makes no attempt at guessing whether or not minting is profitable.

### Wallet security

When `melminter`'s workers for some reason do not have enough funds to pay transaction fees, funds are transferred from the backup wallet. This is most commonly when `melminter` is started for the first time.

To transfer money successfully, `melminter` requires the backup wallet to be unlocked, and will fail if it is locked. It's strongly recommended, however, that the backup wallet be kept locked unless `melminter` is being run for the first time, so that its private key is not exposed in memory for longer than needed.
