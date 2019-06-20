# MelScript specification

## Basic concepts

MelScript is a Lisp-like, purely functional, Turing-incomplete language for writing constraints in Themelio. It's the basic "medium-level" language used in Themelio constraints.

It compiles in a very straightforward manner to constraint bytecode --- core functions map directly to bytecode, while everything else is macros layered on top.

The basic data type of MelScript is a linked-list representation of a RLP-encoded object: either a bytestring, or a list of bytestrings.

## Syntax

### Literals

Literals are one of:

* Decimal numbers, always representing unsigned 64-bit big-endian: `12345`
* Hexadecimal bytestrings: `#xdeadbeef00`
* Literal strings, interpreted as UTF-8: `"Hello World"`

### Program structure

Programs consist of:

* Macro definitions
* One body expression

### Macros

Macros look like:

```scheme
(macro (form args...) expr)
```

which will replace `(form args..)` with `expr` in the body of the program.

They may also simply define a constant:

```scheme
(macro form expr)
```

## Core forms

All core functions have a fixed number of arguments and directly correspond to assembly. For example, the expression `(~add256 x-expr y-expr)` compiles to

```text
(compile y-expr)
(compile x-expr)
ADD256
```

and the semantics of the `ADD256` instruction is `push(pop() + pop())`.

Core forms starting in a tilde should never appear directly in user code.

### Arithmetic

Arithmetic operations all operate on 256-bit big-endian unsigned integers. Only the last 32 bytes of the inputs are considered, and the operations always return 32-byte values.

```scheme
(~add256 x y)
(~sub256 x y)
(~mul256 x y)
(~div256 x y)
(~rem256 x y)
```

### Equality

Equality is checked only on the first 32 bytes.

```scheme
(~equal256? x y)
```

### Hashing

Hashing takes a single, arbitrary-length input and returns a 32-byte hash. It takes variable time, but this is okay, since building a bigger input would have taken more instructions anyway.

```scheme
(hash x)
```

### Signature verification

Both Q and E signatures can be verified.

```scheme
(sigQ-ok? public-key msg-hash signature)
(sigE-ok? public-key msg-hash signature)
```

### RLP operations

RLP structures are treated as nested Lisp-style linked lists. Explicit serialization and deserialization is intentionally not provided; core functions like `hash` implicitly serialize any RLP inputs.

```scheme
(cons head lst)
(car lst)
(cdr lst)
empty
```

## Bytecode weights

The weight of a constraint is computed as $$\ell + \sum_iw_i$$, where $$\ell$$ is the length of the constraint and $$w_i$$ is the weight of the ith instruction. Here are the weights of the instructions:

| Instruction | Encoding | Weight |
| :--- | :--- | :--- |
| ADD256 | 0x10 | 4 |
| SUB256 | 0x11 | 4 |
| MUL256 | 0x12 | 6 |
| DIV256 | 0x13 | 6 |
| REM256 | 0x14 | 6 |
| EQUAL256 | 0x15 | 4 |
| HASH | 0x20 | 50 |
| SIGEOK | 0x30 | 100 |
| SIGQOK | 0x31 | 1000 |
| CONS | 0x40 | 4 |
| CAR | 0x41 | 4 |
| CDR | 0x42 | 4 |
| PUSH | 0x00 | 0 |

