# Block and transaction format

## Overview

This document is a full specification of all the data structures used in Themelio.

We represent structures in Go syntax, which should be fairly straightforward. Serialization is done with Ethereum's RLP syntax, with the Go-to-RLP mapping used by geth's `rlp` package.

## Block headers

Block headers in Themelio are designed to maximize the utility of ultrathin clients who only keep track of a reasonably recent, _single_ block header.

Each Themelio block header is as follows:

```text
type BlockHeader struct {
    PrevHash   [32]byte
    HistoryRH  [32]byte
    StateRH    [32]byte
    TxHashesRH [32]byte
    StakeH     [32]byte
    FeePoolC   uint
    FeePoolL   uint
    Height     uint
    Timestamp  uint
}
```

The parent hash points to the previous block header. The history tree maps _little-endian_ block numbers to block header hashes.

### State tree

The state tree of each block is an MBPT storing mappings generally related to state useful for light clients. The purpose of lumping it into one tree is to support future features while keeping backwards compatibility with clients \(though not auditors\). This _currently_ is the following:

* A mapping from `["utxo.value", txInput]`, for every UTXO represented as a TxInput structure, to a TxOutput structure. Both these structures are defined in the subsequent section on transactions.d
* A mapping from `["utxo.coincount", consHash]` for every constraint with active coins \(the double hashing is to prevent DoS vectors from really huge constraints\), to the \(RLP-encoded\) number of unspent UTXOs with that constraint. Of course, constraints with no active coin would not have an entry here.

These mappings allow the basic function of a "SPV" client to be entirely trust-free, unlike the situation in Bitcoin where hiding coins or misrepresenting spent status of coins is possible.

### Transaction tree

The transaction tree is an MBPT mapping transaction hashes of new transactions in the block to the transaction itself.

### Block number

The block number is included, unlike, say, Bitcoin, for a client to quickly check whether or not its copy of the latest block header is outdated or not.

### Transaction time

The timestamp is simply whatever happens to be the clock of the successful proposer of the block, and is not checked when blocks are validated. Later blocks are not even guaranteed to have higher timestamps than earlier blocks. Thus, timestamps should never be trusted alone; instead, for trusted timestamping applications using the median of, say, the surrounding 100 blocks is strongly recommended.

## Transactions

### General format

The general format of a transaction is as follows:

```go
type Transaction struct {
    Kind    uint8
    Inputs  []TxInput
    Outputs []TxOutput
    Fee     uint
    Scripts []melscript.Script
    Data    []byte
    Sigs    []TxSig
}

// TxInput is a transaction input.
type TxInput struct {
    TxHash melcrypt.Bytes32
    Index  uint
}

// TxOutput is a transaction output.
type TxOutput struct {
    ScriptHash   melcrypt.Bytes32
    Value        uint
    CoinType     string
}

// TxSig is an algorithm-agnostic signature structure.
type TxSig struct {
    Algorithm byte
    PubKey    []byte
    Signature []byte
}
```

As in all classic UTXO-based blockchains, each transaction is at its core a collection of inputs and outputs, each input spending a previous transaction's output. Every output includes a **constraint script**, which constrains what sort of transaction may spend it --- for example, by requiring the transaction spending it to have a signature from a particular public key. An output also indicates the **coin type**; it's either "C" for mels \("coins"\) or "L" for mets \("land"\) or a custom user-created token, as we will describe later.

Unlike Bitcoin, however, we do not supply input scripts to satisfy individual output constraints; the entire transaction is fed as the input to each output constraint it attempts to spend. Another difference is that the **fee**, which is always denominated in mels, is explicit; this mostly simplifies implementation. 

Furthermore, outputs always specify constraint scripts by their **script hash**, not by an actual script. When inputs are spent, scripts with the right hash must exist in the Scripts field of the transaction. This is actually very important, as it ensures that the fees associated with running the script are incurred when the script is actually run, not when the script is first put on the blockchain. This aligns fees with actual stakeholder costs, reducing a potentially huge source of fee market distortion. 

The **kind** of a transaction determines the interpretation of the other fields. The default kind, used in all usual fund-transferring contexts, is `0x00`; other kinds are used for special transactions, such as minting new currency and staking.

Some notes:

* The signature part can be malleable and thus is removed when txids are computed. This is for the same rationale as in SegWit etc.
* Sigs is intended only for cryptographic signatures indicating "who" sent the transaction for satisfying some input constraint, as its contents are not included in cryptographic hashes and cannot be relied on. Non-signature constraint-satisfaction data, or just random metadata, should be placed in Data instead.
* The maximum size of a transaction is 1 MB.

### Minting new mels

As described in the Melmint document, minting in Themelio is a two-step process involving **registering** and **solving** puzzles.

#### Registering puzzles

To register a puzzle, a transaction with `Kind == 0x10` is broadcast, with `Data` looking like:

```go
type PuzzleRegister struct {
    ScriptHash melcrypt.Bytes32 
    Difficulty uint
}
```

`ScriptHash` poses an additional constraint on what sort of transaction is allowed to solve the puzzle. Typically it checks a signature to make sure the person solving the puzzle is the same minter that posed the puzzle.

Registration puzzles' inputs and outputs are validated as usual.

#### Solving puzzles

To solve a puzzle, a transaction with `Kind == 0x11` is broadcast, with `Data` looking like:

```go
type PuzzleSolve struct {
    Solution []byte
}
```

The transaction inputs and outputs are validated as usual, except the input number of mels is incremented by the reward determined by the difficulty of the puzzle. The result is a transaction with more output than input, thus minting new mels.

### Staking and unstaking

TBD!!

