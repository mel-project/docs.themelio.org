# MelVM specification

## Overview

MelVM is the low-level, stack-based virtual machine that powers Themelio covenants. By only allowing looping for a fixed number of iterations, MelVM makes the worst-case cost of each covenant computable, trading off Turing completeness. Unlike Bitcoin scripts, however, MelVM covenants can compute all [primitive recursive functions](https://en.m.wikipedia.org/wiki/Primitive_recursive_function), allowing expressing almost all "interesting" constructs.

## Memory model

MelVM uses a [Harvard architecture](https://en.m.wikipedia.org/wiki/Harvard_architecture) where the code itself is not part of the memory space. Memory is divided into two regions: the **stack** and the **heap**. Except when data is transferred between the stack and the heap, MelVM instructions directly operate only on stack data, pushing and popping items from the top of the stack.

A big difference from most low-level VMs is that the basic unit of data isn't a word of binary data, but a recursive data structure:

```rust
pub enum Value {
    Int(U256),
    Bytes(im::Vector<u8>),
    Vector {
        members: im::Vector<Value>, 
        is_struct: bool
    },
}
```

That is, there are two **base types**:

* 256-bit unsigned integers
* Arbitrary-length bytestrings

As well as two **product types**:

* Vectors, with `is_struct = false`, semantically represent homogenous vectors such as lists of signatures
* Structs, with `is_struct = true`, semantically represent fixed-length structures such as transactions

The stack is just a FIFO stack of `Value`s, while the heap is a mapping from 16-bit **addresses** to `Value`s.

## List of opcodes

In the "meaning" field, all expressions are evaluated left to right. For example, `pop() - pop()` means pop a value `x`, then another value `y`, and then compute `x-y`.

### Arithmetic

| Opcode | Encoding | Meaning |
| :--- | :--- | :--- |
| ADD | 0x10 | `push(pop() + pop())` |
| SUB | 0x11 | `push(pop() - pop())` |
| MUL | 0x12 | `push(pop() * pop())` |
| DIV | 0x13 | `push(pop() / pop())` |
| REM | 0x14 | `push(pop() % pop())` |

### Logic

All operators operate on integers and are bitwise.

| Opcode | Encoding | Meaning |
| :--- | :--- | :--- |
| AND | 0x20 | `push(pop() & pop())` |
| OR | 0x21 | `push(pop() | pop())` |
| XOR | 0x22 | `push(pop() ^ pop())` |
| NOT | 0x23 | `push(~pop())` |

### Cryptography

Operators take in a bytestring and return a bytestring

| Opcode | Encoding | Meaning |
| :--- | :--- | :--- |
| HASH | 0x30 | `push(blake3(pop))` |
| SIGEOK | 0x32 | `push(ed25519_verify(msg = pop(), pk = pop(), sig = pop()))` |

### Heap access

| Opcode | Encoding | Meaning |
| :--- | :--- | :--- |
| LOAD | 0x40 | `push(heap[pop()])` |
| STORE | 0x41 | `heap[pop()] = pop()` |

### Vector operations

### Control flow

### Literals



