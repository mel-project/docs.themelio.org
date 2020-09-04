# Your first alphanet transaction

This document will guide you through setting up a Themelio alphanet client and sending your first transaction. Before you follow the steps listed here, you probably want to read the introduction to Themelio to understand some basic concepts.

## Assumptions

All the instructions here assume that

* You're running a Unix \(Linux or macOS\) system. The alphanet client should work on Windows, but it isn't well-tested.
* You have a working Internet connection
* You have `git` installed
* You have a stable Rust compiler, including the `cargo`  command

## Install themelio-core

Install `themelio-core` with `cargo` directly from GitHub:

```text
$ cargo install --git https://github.com/themeliolabs/themelio-core.git
```

This will take a while as `cargo` downloads and compiles the entire Themelio codebase and all its dependencies.

## Create Alice's and Bob's wallets

Before we send any transactions, we first create two wallets between which we will send money.

### Start the client

`themelio-core` is a monolithic program with a large number of subcommands that each implement some Themelio-related service. Right now all you want is to run a thin-client wallet, so run

```text
$ themelio-core anet-client

[anet client v0.1.0]%
```

You should see a command prompt show up. This prompt will be the main interface you use to interact with Themelio's alphanet as a thin client.

### Create two wallets

To create a wallet, use the `wallet-new` command:

```text
[anet client v0.1.0]% wallet-new alice
>> Created wallet "alice"
>> Address: THQ2Q-JHSKJ-7VTMR-3BKM4-VPXGE-GT4LH-NBPDE-ZWIG4-UTXJE-JKZHH-GUWJA
>> Secret: d41279293facd91d854ce55f7310d3e2ced0bc64cd906e52774912ac9ce6a592
```

This generates and stores to disk a new wallet called "alice", printing out the **address** and the **secret**. Note both of these values.

{% hint style="warning" %}
The wallet secret **will not be saved to disk!** You should back up the secret somewhere safe if you need to recover the contents of the wallet.
{% endhint %}

Repeat the process for Bob, and you're done for this step.

## Add money to Alice's wallet

### Open Alice's wallet

We first need to open Alice's wallet using the secret:

```text
[anet client v0.1.0]% wallet-open alice d41279293facd91d854ce55f7310d3e2ced0bc64cd906e52774912ac9ce6a592
>> Wallet unlocked successfully!
[anet client v0.1.0](alice)%
```

This gives us a command prompt within the context of the `alice` wallet.

### Use the alphanet faucet

Themelio's alphanet has a _faucet_ facility that allows anybody to print mels out of thin air. The faucet would obviously be removed in the mainnet, but it allows easy testing on the alphanet.

Let's print 1000 mels:

```text
[anet client v0.1.0](alice)% faucet 1000 TML
>> Faucet transaction for 1000 broadcast!
>> Waiting for confirmation...
>> CID=0475c44ade579a35bdb4b094817ff1eb828fad0c05b95e256f84b332cfbdc02d
```



