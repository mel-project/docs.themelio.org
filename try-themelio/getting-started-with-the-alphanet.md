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

## Using the wallet

### Starting the wallet

`themelio-core` is a monolithic program with a large number of subcommands that each implement some Themelio-related service. Right now all you want is to run a thin client, so run

```text
$ themelio-core anet-client
```





