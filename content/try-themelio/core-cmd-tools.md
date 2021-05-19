---
title: "Core command-line tools"
weight: 1
draft: false
# search related keywords
keywords: [""]
---

The core functionality of Themelio is encapsulated in a few command-line tools:

- `themelio-node` is the **full node** reference implementation. You'll use `themelio-node` to contribute to the network, either as a [nonvoting auditor node]({{< ref auditor-node.md>}}), or a [voting staker node]({{< ref auditor-node.md>}}) that stakes Sym to participate in Themelio's proof-of-stake consensus.
- `melwalletd` is a **thin-client wallet daemon**. It exposes a local REST API that can be directly used, but is intended mostly as a microservice that wallet GUIs, trading bots, and other programs use to transact on the blockchain.
- `melminter` is a **Melmint minter** that uses CPU power to mint _nominal DOSCs_, which can then be converted to Mel or Sym. It interfaces with `melwalletd`.

## Installing the CLI tools

The recommended way of installing Themelio software at the moment is by compiling from source. Since Themelio's CLI tools are written purely in Rust, this is very easy using Rust's Cargo package manager.

### Installing a Rust toolchain

The best way to install a Rust toolchain is to follow the [official guide](https://www.rust-lang.org/learn/get-started).

Make sure that your Cargo version is **at least 1.49**:

```shell
$ cargo --version
cargo 1.52.1
```

### Compiling `themelio-node`

```shell
$ cargo install --git https://github.com/themeliolabs/themelio-core.git themelio-node
...
```

### Compiling `melwalletd`

```shell
$ cargo install --git https://github.com/themeliolabs/melwalletd.git
```

### Compiling `melminter`

```shell
$ cargo install --git https://github.com/themeliolabs/melwalletd.git
```

## Using the CLI tools

We will cover how to use the CLI tools in subsequent pages on [running full nodes]({{< ref auditor-node.md>}}), [using the wallet daemon]({{< ref melwalletd.md>}}), and [minting DOSC]().
