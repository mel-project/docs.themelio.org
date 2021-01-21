---
title: "My first alphanet transaction"
date: 2018-12-29T11:02:05+06:00
lastmod: 2020-01-05T10:42:26+06:00
weight: 9
draft: false
# search related keywords
keywords: [""]
---

This document will guide you through setting up a Themelio alphanet client and sending your first transaction. Before you follow the steps listed here, you probably want to read [the introduction to Themelio](./) to understand some basic concepts.

## Assumptions

All the instructions here assume that

- You're running a Unix \(Linux or macOS\) system. The alphanet client should work on Windows, but it isn't well-tested.
- You have a working Internet connection
- You have `git` installed
- You have a stable Rust compiler, including the `cargo` command

## Install themelio-core

Install `themelio-core` with `cargo` directly from GitHub:

```text
$ cargo install --git https://github.com/themeliolabs/themelio-core.git themelio-core
```

`cargo` downloads and compiles the entire Themelio codebase and all its dependencies. This will take a while.

## Create Alice's and Bob's wallets

Before we send any transactions, we first create two wallets between which we can send money.

### Start the client

`themelio-core` is a monolithic program with a large number of subcommands that each implement some Themelio-related service. Right now, all you want is to run a thin-client wallet, so run

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
>> Address: <ALICE_ADDRESS>
>> Secret: <ALICE_SECRET>
```

This generates and stores to disk a new wallet called "alice", printing out the **address** and the **secret**. Note both of these values.

> The wallet secret **will not be saved to disk!** You should back up the secret somewhere safe if you need to recover the contents of the wallet.

Repeat the process for Bob, and you're done for this step.

## Add money to Alice's wallet

### Open Alice's wallet

We first need to open Alice's wallet using the secret:

```text
[anet client v0.1.0]% wallet-unlock alice <ALICE_SECRET>
>> Wallet unlocked successfully!
[anet client v0.1.0](alice)%
```

This gives us a command prompt within the context of the `alice` wallet.

### Use the alphanet faucet

Themelio's alphanet has a _faucet_ facility that allows anybody to print mels out of thin air. The faucet allows easy testing on the alphanet and would of course be removed in the mainnet.

Let's print 1000 mels:

```text
[anet client v0.1.0](alice)% faucet 1000 TML
>> Faucet transaction for 1000 broadcast!
>> Waiting for confirmation...
>> Confirmed at block 10!
>> CID=<FAUCET_CID>
```

This gives us a **coin ID**, or CID, that we use as a "receipt" to insert coins into the wallet:

```text
[anet client v0.1.0](alice)% coin-add <FAUCET_CID>
>> Syncing state...
>> Coin found! Added 1000.0000 TML to wallet
```

## Send money to Bob

### Send the transaction

Now we are ready to send money to Bob. Let's send over 500 TML:

```text
[anet client v0.1.0](alice)% tx-send <BOB_ADDRESS> 500 TML
>> Syncing state...
>> Fee required: 0.0123 TML. Accept? [y/n] y
>> Transaction <TXID> broadcast!
>> Waiting for confirmation...
>> Confirmed at block 10!
>> CID=<ALICE_TO_BOB_CID>
```

This gives us another CID that Bob will use to receive the money.

### Open Bob's wallet

We now open Bob's wallet to receive Alice's money:

```text
[anet client v0.1.0](alice)% exit
[anet client v0.1.0]% wallet-open bob <BOB_SECRET>
>> Wallet unlocked successfully!
[anet client v0.1.0](bob)%
```

### Receiving the money

We use the CID to receive the money from Alice:

```text
[anet client v0.1.0](bob)% coin-add <ALICE_TO_BOB_CID>
>> Syncing state...
>> Coin found! Added 500.0000 TML to wallet
```

## Congratulations!

You've successfully sent 500 mels from Alice to Bob. Alice now has 499.987 TML in her wallet, while Bob has 500 TML.

## Next steps

In this guide, you used a validating thin client that does not synchronize the entire blockchain state. This has slightly less security and doesn't allow much functionality without a reliable Internet connection, so in some applications you would want to run an auditor node to replicate and fully validate blocks. That's covered in the next guide.
