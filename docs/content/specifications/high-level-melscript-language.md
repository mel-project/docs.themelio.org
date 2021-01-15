# MelScript: high-level covenant language

The high-level MelScript language is essentially an extension of simply typed lambda calculus, packaged in a friendly syntax.

## Structure of a HLMS covenant

A HLMS covenant consists of zero or more **definitions** followed by a single **expression**. Each definition itself follows an analogous structure.

Here's an example that constraints a coin to only be spent in a transaction whose total output in micromels is odd, and has less than 16 outputs:

```lua
using globals::CURRENT_TX
using std

function total_output[$OCOUNT]() -> Nat
    CURRENT_TX.outputs |>
                    limit[$OCOUNT]() |>
                    filter((coin: std::TxOutput) => coin.coin_type == std::TMEL) |>
                    map((coin: std::TxOutput) => coin.value)
end

total_output[$16]() % 2 == 1
```

## HLMS's type system

HLMS uses a type system heavily based on "const generics" to enable richly-featured programming with compile-time-determined runtime cost. The type system is fundamentally structural, and types can be completely understood as describing sets of values. Parametric polymorphism \("generics"\) is supported, implemented entirely by monomorphization \("templates"\) _after_ typechecking, like Rust and unlike C++.

### Base types

There are four base types:

* `Nat`, representing an unsigned 256-bit integer
* `Int`, representing a signed 256-bit integer
* `Bool`, representing a boolean
* `Bytes[$N]`, a const-generic type representing a byte array with _exactly_ `N` elements.
* `VarBytes[$N]`, a const-generic type representing a byte array with _at most_ `N` elements. `VarBytes[$N]` is a supertype of `Bytes[$N]` 

### Product types

There are three kinds of product types:

* Tuples, written as `(T, U, V...)`. They are fixed-length, and elements can be accessed as `tuple.0` etc.
* Arrays, written as `Array[T; $N]` representing a fixed-length array of exactly `N` elements, all of type `T`.
* Variable-length arrays, written as `VarArray[T; $N]`, representing an array that has up to `N` elements. `VarArray[T; $N]` is a supertype of `Array[T; $N]`.

### Sum types

A sum type between `T` and `U` is denoted as `T+U`. For example, a value with type `Nat + (Int, Int)` may either be an unsigned integer or a tuple of two signed integers.

The special type `Any` essentially denotes a type that could be any type.

Sum types can be disambiguated through conditionals and the `is` operator:

```lua
-- if v is an integer, return itself
-- otherwise return 0
function force_to_int(v: Any) -> Int
    if v is Int then
        v
    else
        0
    end
end
```

### Subtraction types

A subtraction type between `T` and `U` is denoted as `T \ U`. This denotes a value that is in type `T` but not in type `U`.

For example, a value with type `Int \ Nat` must be a negative integer.

### Type aliases

Type aliases give a name to a complex type so that type signatures don't become extremely cumbersome to type.

#### Simple aliases

```lua
-- Point is just another name for (Int, Int)
alias Point = (Int, Int)
```

#### Struct aliases

Struct aliases have special behavior: they allow field-accessor syntax that can be used whenever a variable is explicitly given a particular alias. This exception to purely structural typing allows us to define semantically distinct structures:

```lua
-- Point is essentially equivalent to (Int, Int), but with a field-accessor syntax
alias Point = {
    x: Int,
    y: Int,
}

-- syntax examples
let pt = Point { x: 3, y: 5 }
pt.x -- "struct" syntax
pt.0 -- but this still works
(1, 5) is Point -- but this returns true
-- and we can do this
let pp = (1, 5)
if pp is Point then
    pp.x
else
    0
end
```

### Occurrence typing

In previous examples, we've already seen HLMS's **occurrence typing** in action. The type of a variable may be **refined** within a certain scope based on predicates.

The simplest example is the `is` operator that returns `true` iff the argument is a certain type:

```lua
let foo: Int | (Nat, Nat) = ...
-- at this point, foo has type "Int | (Nat, Nat)"
if foo is Int then
    -- now, foo has type "Int"
    foo + 1
else 
    -- we know that here foo could only be of type "(Nat, Nat)"
    foo.0 + 1
```

We can also use `assert` to assert that something is a certain type, aborting the whole script if it fails:

```lua
let foo: Int = ...
-- this is only defined for positive integers. Nat is a subtype of Int
int_sqrt(assert foo as Nat)
```

## Syntax

### Arithmetic expressions

We use the usual infix syntax

```lua
x + y -- addition
x * y -- multiplication
x - y -- subtraction
x / y -- integer division
x % y -- remainder
-x    -- negation

      -- bitwise operations only on Nat
x ^ y -- bitwise xor. NOT exponentiation, which wouldn't be constant-time
x | y -- bitwise or
x & y -- bitwise and
~x    -- bitwise not
```

### Boolean expressions

Boolean expressions are short-circuited as expected.

```lua
x and y
x or y
not x
```

### Looping and flow control

The `if` form is an **expression**:

```lua
let foo = if bar then baz else nuu end
```

The `for..do` form is a statement \(an expression with type `()`\), and is run for its side effects:

```lua
for elem in list do
    if elem % 2 == 0 then
        return elem
    else
end
```

The `for..collect..where` form collects its contents into a list:

```lua
let even_elems = for elem in list 
    collect elem
    where elem % 2 == 0
end
```

### Defining functions

Functions are defined like this:

```lua
function f(x: Int, y: Int) -> Int
...
end
```

HLMS does not at the moment support higher-order functions or anonymous functions.

### Modules

Each module corresponds to a file.

Something is exported by prepending its definition by `public`:

```lua
public function...
public type ...
```

Importing another module uses the `using` keyword. Local files can be used too.

```lua
using std
using "some_file" as sfile -- creates namespace sfile::* in this file
```









