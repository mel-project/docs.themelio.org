---
title: "My first transaction"
weight: 2
draft: false
# search related keywords
keywords: [""]
---

This document will guide you through setting up a Themelio wallet and sending your first transaction. Right now, you must use the headless wallet daemon `melwalletd` and the `melwallet-cli` CLI tool, though soon we will have a user-friendly GUI wallet that will be as easy to use as Metamask or Electrum.

## Assumptions

All the instructions here assume that

- You're running a Unix \(Linux or macOS\) system. The code should work on Windows, but it isn't well-tested.
- You have a working Internet connection
- You have `git` installed
- You have a 1.58+ stable Rust compiler, including the `cargo` command

## Install melwalletd and melwallet-cli

Install the tools with `cargo` directly:

```text
$ cargo install --locked melwalletd

$ cargo install --locked melwallet-client
```

`cargo` downloads and compiles the entire codebase and all its dependencies from scratch. This will take a while.

## Start melwalletd

In a _separate_ terminal window, leave the following command running:

```text
melwalletd --wallet-dir ~/.themelio-wallets/ --network testnet
```

You can also use `tmux` or similar to run it in the background. This starts the persistent wallet daemon that the frontend `melwallet-cli` will communicate with.

## Create Alice's and Bob's wallets

Before we send any transactions, we first create two wallets between which we can send money.

Let's now create two wallets: `alice` and `bob`. To do so, first switch to another terminal and use `melwallet-cli create-wallet` to create two wallets:

```text
$ melwallet-cli create -w alice 
Wallet name:  alice
Network:      testnet
Address:      <ALICE_ADDRESS>
Balance:      0 µMEL

$ melwallet-cli create -w bob 
Wallet name:  bob
Network:      testnet
Address:      <BOB_ADDRESS>
Balance:      0 µMEL
```

This generates and stores to disk the two wallets. We create the wallets on the _testnet_ network, because right now there isn't a way of purchasing mainnet tokens yet. Note that the **address** is a public identifier that uniquely identifies a wallet. It's what you give other people when you want to receive money.

`melwallet-cli` will also prompt you for passwords. These passwords are used to encrypt the private keys of the wallets, which are stored in `~/.themelio-wallets/.secrets.json`. So make sure to pick something strong! In the future, more interesting forms of authentication may be implemented.

## Add money to Alice's wallet

### Use the testnet faucet

Because we created testnet wallets, we can use the _testnet faucet_: special transaction rules that allows anybody to print mels out of thin air. This allows easy testing on the testnet and is of course absent in the mainnet.

`melwallet-cli send-faucet` sends a faucet transaction to credit the wallet with 1000 MEL:

```text
$ melwallet-cli send-faucet -w alice
Transaction hash:  9974a514351a0696b6d7e3851da957ff508e44857b4967e3d46b8d16685b9769
(wait for confirmation with melwallet-cli wait-confirmation -w alice 9974a514351a0696b6d7e3851da957ff508e44857b4967e3d46b8d16685b9769)
```

The faucet transaction is now already on its way. As the output suggests, you can wait for the transaction to confirm:

```text
$ melwallet-cli wait-confirmation -w alice 9974a514351a0696b6d7e3851da957ff508e44857b4967e3d46b8d16685b9769
...............Confirmed at height 93807
(in block explorer: https://scan-testnet.themelio.org/blocks/93807/9974a514351a0696b6d7e3851da957ff508e44857b4967e3d46b8d16685b9769)
```

## Send money to Bob

### Send the transaction

Now we are ready to send money to Bob. We first unlock the wallet by entering our password:

```text
$ melwallet-cli unlock -w alice
```

Let's send over 500 MEL:

```text
$ melwallet-cli send-tx -w alice --to <BOB_ADDRESS>,500000000
TRANSACTION RECIPIENTS
Address         Amount          Additional data
<BOB_ADDRESS>   500000000 µMEL  ""
 (network fees) 23 µMEL
Proceed? [y/N] y
Transaction hash:  35149dd7e23e4acbc3823578ddd73aa09e0ddd08f970b2b673e7f5e58dab6dc9
(wait for confirmation with melwallet-cli wait-confirmation -w alice 35149dd7e23e4acbc3823578ddd73aa09e0ddd08f970b2b673e7f5e58dab6dc9)
```

Note that we specify the units as µMEL (1 million µMEL = 1 MEL).

We can now wait until the money settles on the blockchain with the given command:

```text
$ melwallet-cli wait-confirmation -w alice 35149dd7e23e4acbc3823578ddd73aa09e0ddd08f970b2b673e7f5e58dab6dc9
Confirmed at height 93819
(in block explorer: https://scan-testnet.themelio.org/blocks/93819/35149dd7e23e4acbc3823578ddd73aa09e0ddd08f970b2b673e7f5e58dab6dc9)
```

### Receiving the money

`melwallet-cli` does not constantly scan the blockchain for incoming transactions, so we need to "sync" Bob's wallet. We run `melwallet-cli sync -w bob`.

This adds the money to Bob's wallet:

```text
Coin successfully added!
Wallet name:  bob
Network:      testnet
Address:      t0cvmtcqrtepb8tbsasmp3rm4kcv8w4s5s0w80ff46ecgbfafa7k5g
Balance:      500000000  µMEL
```

We now see that Bob has the 500 MEL from Alice!

## Congratulations!

You've successfully sent 500 mel from Alice to Bob.

## Next steps

In this guide, you used a validating thin client that does not synchronize the entire blockchain state. This has slightly less security and doesn't allow much functionality without a reliable Internet connection, so in some applications you would want to run an full node to replicate and fully validate blocks. That's covered in the next guide.
