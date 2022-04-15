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
  cargo install --locked melminter melwalletd
  ```

- A small amount of MEL.

The second requirement may be surprising: how can minting mel require mel? This is because Melmint transactions are required to pay transaction fees just like any other transaction, so you must have some "starting" mel to even begin to mint more.

## Starting the melminter

The easiest way to run the melminter is to first start `melwalletd`, then simply run the following command:

```
melminter --payout <some wallet address>
```

For example, if we have a wallet with address `t19rcw46nfdy3vfwejw1a4qpng3s2km6myevcm5erjk0e2094rg640`, we can run:

```
melminter --payout t19rcw46nfdy3vfwejw1a4qpng3s2km6myevcm5erjk0e2094rg640
```

This will start a cool TUI interface that looks like this:

![](/images/melminter.png)

Note the _daily return_ line, which predicts how much computational work (in DOSC) the minter will do in 24 hours, as well as how much MEL that will generate.

Note that the first time you run `melminter`, it will ask you to send a particular address a small amount of MEL. You must do so for `melminter` to start; otherwise it has nothing to pay initial transaction fees.

## What is going on?

To understand what is happening in `melminter`, recall the [overall Melmint v2 procedure]({{< relref path="../whitepapers/melmint-v2.md" >}}) that aims to peg 1 MEL to 1 DOSC, or "day of sequential computation":

![](/images/melmint-v2-overview.png)

`melminter` first mints _erg_, a constant $k$ of which takes a DOSC of computation to create. This is done through creating a `DoscMint` transaction with a proof of sequential work over its inputs. Since this is an inherently sequential process, `melminter` starts off by creating a bunch of different coins, one for each CPU core, spending them with separate `DoscMint` transactions with proofs-of-sequential-work, created in separate threads, attached to generate nominal DOSC. This lets `melminter` fully utilize CPU resources.

(You can also reconfigure the number of threads used through the `--threads` argument)

This nominal DOSC is then continually converted through Melswap into mel, and used to pay transaction fees. Any excess is sent to the payout address. All of this can be configured through `melminter` command-line arguments; `melminter -h` gives a full explanation.

## Caveats

### Minting may not be profitable

Minting erg and converting into mel is not always profitable. Because of the mechanics of Melmint, **minting is _probably not_ profitable** unless you either have a top-of-the-line CPU and cheap electricity, or Melmint is off-peg. This is because Melmint is not a proof-of-work consensus system, but rather a _pegging arbitrage_ system that is only really used to restore the MEL/DOSC peg.

Transient volatility in the mel/erg exchange rate may also affect profitability.

`melminter` makes no attempt at guessing whether or not minting is profitable.
