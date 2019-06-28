# MelScript specification

## Basic concepts

MelScript is a Lisp-like, purely functional, Turing-incomplete language for writing constraints in Themelio. It's the basic "medium-level" language used in Themelio constraints.

It compiles in a very straightforward manner to constraint bytecode --- core functions map directly to bytecode, while everything else is macros layered on top.

The basic data type of MelScript is a linked-list representation of a RLP-encoded object: either a bytestring, or a list of bytestrings.

## Syntax

### Literals

Literals are one of:

* Decimal numbers, always representing unsigned big-endian represented with the smallest number of bytes: `12345`
* Hexadecimal bytestrings: `#xdeadbeef00`
* Literal strings, interpreted as UTF-8: `"Hello World"`

### Program structure

Programs consist of:

* Macro definitions
* One body expression

### Macros

Macros look like:

```scheme
(define (form args...) expr)
```

which will replace `(form args..)` with `expr` in the body of the program.

They may also simply define a constant:

```scheme
(define form expr)
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

### Boolean operations

Boolean operations are _bitwise_ on the last 32 bytes. They only return 32 bytes.

```scheme
(~and256 x y)
(~or256 x y)
(~xor256 x y)
(~not256 x)
```

There's also a **non short-circuited** if operator, which treats its condition as "false" if its last 32 \(or less\) bytes are all zero, and "true" otherwise:

```scheme
(~if256 c true-case false-case)
```

### Equality

Equality is checked only on the first 32 bytes. Hash bigger inputs before comparing them.

```scheme
(~equal256? x y)
```

### Hashing

Hashing takes a single input and returns a 32-byte hash. This input can either an RLP structure or a byte string; RLP structures are implicitly serialized.. It takes variable time, but this is okay, since building a bigger input would have taken more instructions anyway.

```scheme
(hash x)
```

### Signature verification

Both Q and E signatures can be verified.

```scheme
(sigQ-ok? public-key msg-hash signature)
(sigE-ok? public-key msg-hash signature)
```

**As a special case**, if both the `signature` and `msg-hash` field is zero-length, then we iterate through the `Signatures` field in the spending transaction to try to find a matching signature. This special case is used in the vast majority of single-signature and multisignature wallets.

In general, using `sigQ-ok?` outside this special case \(say, with `Data` members\) requires some care, since signatures may be malleable.

### RLP operations

RLP structures are treated as nested Lisp-style linked lists with explicit length tracking. Explicit serialization and deserialization is intentionally not provided; core functions like `hash` implicitly serialize any RLP inputs.

```scheme
(cons head lst)
(car lst)
(cdr lst)
(length lst)
empty
```

### Environment

The execution environment is accessed through the special "function" `env`. This is literally compiled as, for example

```text
PUSH "SELFHASH"
ENV
```

Accessing a nonexistent environment variable will result in instantly failing the constraint.

```scheme
;; Hash of the compiled constraint code itself 
(env 'SELFHASH)

;; Transaction output in which the constraint is embedded, as an RLP structure
(env 'SELFTXO) ; TxOutput structure
(env 'SELFTXI) ; TxInput structure

;; Spender (the transaction spending the coin having this constraint)
(env 'SPENDTX) ; whole transaction as RLP

;; Last block header (if TX at block n, header n-1)
(env 'LASTHEADER)
```

## Built-in macros

### Arithmetic, boolean, and equality

Arithmetic, boolean, and equality operators can take multiple arguments.

```scheme
(+ x y z...) => (~add256 x (~add256 y (~add256 z ...)))
(= x y z...) => (~equal256? x (~equal256? y (~equal256? z ...)))
(or x y z...)
...
```

### List operations

List operations expand to a bunch of cars, conds, etc.

```scheme
;; get the nth element of a list
(ref lst 0) => (car lst)
(ref lst n) => (ref (cdr lst) (- n 1))
```

### Conditions

There's two ways of accessing the `~if256` core form:

```scheme
(if c x y) => (~if256 c x y)

(cond
  [c1 e1]
  [c2 e2]
  ...
  [else ex]) => (~if256 c1 e1 
                  (cond [c2 e2]...))
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
| AND256 | 0x1a | 4 |
| OR256 | 0x1b | 4 |
| XOR256 | 0x1c | 4 |
| IF256 | 0x1f | 4 |
| EQUAL256 | 0x20 | 4 |
| HASH | 0x30 | 50 |
| SIGEOK | 0x40 | 100 |
| SIGQOK | 0x41 | 1000 |
| CONS | 0x50 | 50 |
| CAR | 0x51 | 6 |
| CDR | 0x52 | 6 |
| LENGTH | 0x53 | 6 |
| ENV | 0xa0 | 50 |
| PUSH | 0x00 | 0 |

