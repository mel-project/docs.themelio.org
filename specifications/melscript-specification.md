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

## Core functions

### Arithmetic

Arithmetic operations all operate on 64-bit big-endian unsigned integers. Only the last 8 bytes of the inputs are considered, and the operations always return 8-byte values.

```scheme
(~add64 x y)
(~sub64 x y)
(~mul64 x y)
(~div64 x y)
(~rem64 x y)
```



