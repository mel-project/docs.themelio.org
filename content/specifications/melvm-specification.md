---
title: "MelVM: low-level covenant VM
"
date: 2018-12-29T11:02:05+06:00
lastmod: 2020-01-05T10:42:26+06:00
weight: 11
draft: false
# search related keywords
keywords: [""]
---

## Overview

MelVM is the low-level, stack-based virtual machine that powers Themelio covenants. By only allowing looping for a fixed number of iterations, MelVM makes the worst-case cost of each covenant computable, trading off Turing completeness. Unlike Bitcoin scripts, however, MelVM covenants can compute all [primitive recursive functions](https://en.m.wikipedia.org/wiki/Primitive_recursive_function), allowing expressing almost all "interesting" constructs.

## Memory model

MelVM uses a [Harvard architecture](https://en.m.wikipedia.org/wiki/Harvard_architecture) where the code itself is not part of the memory space. Memory is divided into two regions: the **stack** and the **heap**. Except when data is transferred between the stack and the heap, MelVM instructions directly operate only on stack data, pushing and popping items from the top of the stack.

A big difference from most low-level VMs is that the basic unit of data isn't a word of binary data, but a recursive data structure:

```rust
pub enum Value {
    Int(U256),
    Bytes(im::Vector<u8>),
    Vector(im::Vector<Value>),
}

```

That is, there are two **base types**:

- 256-bit unsigned integers
- Arbitrary-length bytestrings

As well as a **product type**: an immutable vector.

The stack is just a FIFO stack of `Value`s, while the heap is a mapping from 16-bit **addresses** to `Value`s.

## Execution model

MelVM takes as input:

```rust
/// Heap address where the transaction trying to spend the coin encumbered by this covenant (spender) is put
pub const ADDR_SPENDER_TX: u16 = 0;
/// Heap address where the spender's hash is put.
pub const ADDR_SPENDER_TXHASH: u16 = 1;
/// Heap address where the *parent* (the transaction that created the coin now getting spent)'s hash is put
pub const ADDR_PARENT_TXHASH: u16 = 2;
/// Heap address where the index, at the parent, of the coin being spent is put. For example, if we are spending the third output of some transaction, `Heap[ADDR_PARENT_INDEX] = 2`.
pub const ADDR_PARENT_INDEX: u16 = 3;
/// Heap address where the hash of the running covenant is put.
pub const ADDR_SELF_HASH: u16 = 4;
/// Heap address where the face value of the coin being spent is put.
pub const ADDR_PARENT_VALUE: u16 = 5;
/// Heap address where the denomination of the coin being spent is put.
pub const ADDR_PARENT_DENOM: u16 = 6;
/// Heap address where the additional data of the coin being spent is put.
pub const ADDR_PARENT_ADDITIONAL_DATA: u16 = 7;
/// Heap address where the height of the parent is put.
pub const ADDR_PARENT_HEIGHT: u16 = 7;
/// Heap address where the "spender index" is put. For example, if this coin is spent as the first input of the spender, then `Heap[ADDR_SPENDER_INDEX] = 0`.
pub const ADDR_SPENDER_INDEX: u16 = 8;
/// Heap address where the header of the last block is put. If the covenant is being evaluated for a transaction in block N, this is the header of block N-1.
pub const ADDR_LAST_HEADER: u16 = 9;
```

## List of opcodes

In the "meaning" field, all expressions are evaluated left to right. For example, `pop() - pop()` means pop a value `x`, then another value `y`, and then compute `x-y`.

### Arithmetic

Overflow always wraps. Whether or not the previous instruction overflowed can be queried.

| Opcode | Encoding | Meaning               |
| :----- | :------- | :-------------------- |
| ADD    | 0x10     | `push(pop() + pop())` |
| SUB    | 0x11     | `push(pop() - pop())` |
| MUL    | 0x12     | `push(pop() * pop())` |
| DIV    | 0x13     | `push(pop() / pop())` |
| REM    | 0x14     | `push(pop() % pop())` |

### Logic

All operators operate on integers and are bitwise.

| Opcode | Encoding | Meaning                |
| :----- | :------- | :--------------------- |
| AND    | 0x20     | `push(pop() & pop())`  |
| OR     | 0x21     | `push(pop() \| pop())` |
| XOR    | 0x22     | `push(pop() ^ pop())`  |
| NOT    | 0x23     | `push(~pop())`         |
| EQL    | 0x24     | `push(pop() == pop())` |
| LT     | 0x25     | `push(pop() < pop())`  |
| GT     | 0x26     | `push(pop() > pop())`  |

### Cryptography

Operators take in a bytestring and return a bytestring.

| Opcode      | Encoding     | Meaning                                                           |
| :---------- | :----------- | :---------------------------------------------------------------- |
| HASH\(n\)   | 0x30 + u16be | `push(blake3(pop()[..n]))`                                        |
| SIGEOK\(n\) | 0x32 + u16be | `push(ed25519_verify(msg = pop()[..n], pk = pop(), sig = pop()))` |

### Heap access

| Opcode      | Encoding     | Meaning               |
| :---------- | :----------- | :-------------------- |
| LOAD        | 0x40         | `push(heap[pop()])`   |
| STORE       | 0x41         | `heap[pop()] = pop()` |
| LOADIMM(n)  | 0x42 + u16be | `push(heap[n])`       |
| STOREIMM(n) | 0x43 + u16be | `heap[n] = pop()`     |

### Vectors

Despite their appearance, these operations, as well as those for bytestrings, are all quasi-constant-time because vectors and bytestrings can be represented as RRB trees or similar.

| Opcode  | Encoding | Meaning                     |
| :------ | :------- | :-------------------------- |
| VREF    | 0x50     | `push(pop()[pop()])`        |
| VAPPEND | 0x51     | `push(pop() ++ pop())`      |
| VEMPTY  | 0x52     | `push(empty vector)`        |
| VLENGTH | 0x53     | `push(pop().length)`        |
| VSLICE  | 0x54     | `push(pop[pop()..pop()])`   |
| VSET    | 0x55     | `push(pop[pop() => pop()])` |
| VPUSH   | 0x56     | `push(pop().push(pop()))`   |
| VCONS   | 0x57     | `push(cons(pop(), pop()))`  |

### Bytestrings

| Opcode  | Encoding | Meaning                     |
| :------ | :------- | :-------------------------- |
| BREF    | 0x70     | `push(pop()[pop()])`        |
| BAPPEND | 0x71     | `push(pop() ++ pop())`      |
| BEMPTY  | 0x72     | `push(empty vector)`        |
| BLENGTH | 0x73     | `push(pop().length)`        |
| BSLICE  | 0x74     | `push(pop[pop()..pop()])`   |
| BSET    | 0x75     | `push(pop[pop() => pop()])` |
| BPUSH   | 0x76     | `push(pop().push(pop()))`   |
| BCONS   | 0x77     | `push(cons(pop(), pop()))`  |

### Control flow

| Opcode          | Encoding             | Meaning                       |
| :-------------- | :------------------- | :---------------------------- |
| JMP\(n\)        | 0xa0                 | Jump n instructions forward   |
| BEZ\(n\)        | 0xa1                 | If `pop()` is zero, jump n    |
| BNZ\(n\)        | 0xa2                 | If `pop()` isn't zero, jump n |
| LOOP\(n, body\) | 0xb0 + u16be + u16be | Loop `body` `n` times         |

### Type conversions

| Opcode | Encoding | Meaning                                                                          |
| :----- | :------- | :------------------------------------------------------------------------------- |
| ITOB   | 0xc0     | Pops an integer and converts to bytes                                            |
| BTOI   | 0xc1     | Pops bytes and converts first 32B to integer                                     |
| TYPEQ  | 0xc2     | Pops a value and returns what type it is: 0 if integer, 1 if bytes, 2 if vector. |

### Literals

| Opcode         | Encoding                 | Meaning                       |
| :------------- | :----------------------- | :---------------------------- |
| PUSHB\(bytes\) | 0xf0 + u8 length + bytes | Push the bytes to the stack   |
| PUSHI\(int\)   | 0xf1 + u256be            | Push the integer to the stack |
