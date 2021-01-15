# MelVM: low-level covenant VM

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

## Execution model

MelVM takes as input:

* An **initial stack** consisting of two items:
  * On the very top, the spending transaction
  * Underneath it, a custom input \(e.g. an index indicating which signature spends the coin\)
* An **initial heap** consisting of the following items:
  * Addr 0x10: the fully decoded _previous_ block header \(e.g. if the covenant is spent in block 100, this is header 99\)
  * Addr  

## List of opcodes

In the "meaning" field, all expressions are evaluated left to right. For example, `pop() - pop()` means pop a value `x`, then another value `y`, and then compute `x-y`.

### Arithmetic

Overflow always wraps. Whether or not the previous instruction overflowed can be queried.

| Opcode | Encoding | Meaning |
| :--- | :--- | :--- |
| ADD | 0x10 | `push(pop() + pop())` |
| SUB | 0x11 | `push(pop() - pop())` |
| MUL | 0x12 | `push(pop() * pop())` |
| DIV | 0x13 | `push(pop() / pop())` |
| REM | 0x14 | `push(pop() % pop())` |
| OFLO | 0x15 | `push(overflowed ? 1 : 0)`  |

### Logic

All operators operate on integers and are bitwise.

| Opcode | Encoding | Meaning |
| :--- | :--- | :--- |
| AND | 0x20 | `push(pop() & pop())` |
| OR | 0x21 | `push(pop() | pop())` |
| XOR | 0x22 | `push(pop() ^ pop())` |
| NOT | 0x23 | `push(~pop())` |

### Cryptography

Operators take in a bytestring and return a bytestring.

| Opcode | Encoding | Meaning |
| :--- | :--- | :--- |
| HASH\(n\) | 0x30 + u16be | `push(blake3(pop()[..n]))` |
| SIGEOK\(n\) | 0x32 + u16be | `push(ed25519_verify(msg = pop()[..n], pk = pop(), sig = pop()))` |

### Heap access

| Opcode | Encoding | Meaning |
| :--- | :--- | :--- |
| LOAD | 0x40 | `push(heap[pop()])` |
| STORE | 0x41 | `heap[pop()] = pop()` |

### Vectors

Despite their appearance, these operations, as well as those for bytestrings, are all quasi-constant-time because vectors and bytestrings can be represented as RRB trees or similar. 

| Opcode | Encoding | Meaning |
| :--- | :--- | :--- |
| VEMPTY | 0x50 | `push(empty vector)` |
| VREF | 0x51 | `push(pop()[pop()])` |
| VLENGTH | 0x52 | `push(pop().length)` |
| VAPPEND | 0x53 | `push(pop() || pop())` |
| VPUSH | 0x54 | `push(pop().push(pop()))` |
| VSLICE | 0x55 | `push(pop[pop()..pop()])` |

### Bytestrings

| Opcode | Encoding | Meaning |
| :--- | :--- | :--- |
| BEMPTY | 0x60 | `push(empty bytestring)` |
| BREF | 0x61 | `push(pop()[pop()])` |
| BLENGTH | 0x62 | `push(pop().length)` |
| BAPPEND | 0x63 | `push(pop() || pop())` |
| BPUSH | 0x64 | `push(pop().push(pop()))` |
| BSLICE | 0x65 | `push(pop[pop()..pop()])` |

### Control flow

| Opcode | Encoding | Meaning |
| :--- | :--- | :--- |
| JMP\(n\) | 0xa0 | Jump n instructions forward |
| BEZ\(n\) | 0xa1 | If `pop()` is zero, jump n |
| BNZ\(n\) | 0xa2 | If `pop()` isn't zero, jump n |
| LOOP\(n, body\) | 0xb0 + u16be + u16be | Loop `body` `n` times |

### Type conversions

| Opcode | Encoding | Meaning |
| :--- | :--- | :--- |
| ITOB | 0xc0 | Pops an integer and converts to bytes |
| BTOI | 0xc1 | Pops bytes and converts first 32B to integer |
| ~~SERIAL\(n\)~~ | ~~0xd0~~ | ~~Serialize with "weight limit". If more than n nodes are visited abort.~~ |
| ~~DESERIAL\(n\)~~ | ~~0xd1~~ | ~~Bincode-deserialize with length limit~~. |

### Literals

| Opcode | Encoding | Meaning |
| :--- | :--- | :--- |
| PUSHB\(bytes\) | 0xf0 + u8 length + bytes | Push the bytes to the stack |
| PUSHI\(int\) | 0xf1 + u256be | Push the integer to the stack |



