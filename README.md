Xemono
=

Statically typed impure functional programming language based strongly on OCaml, with influences from Python, Haskell and Clojure

Features:

* Support for functional and imperative programming
* First class functions
* First class modules
* Static Typing
* Algebraic Data Types and pattern matching
* Hindley-Milner type inference
* Lexical scoping
* Variable definitions as name-bindings
* Mutable ref and array types
* Significant indentation
* Literals and comprehensions for common data structures
* where and let bindings to specify variable scope
* $ syntax for controlling function application

Areas to investigate:

* Objects
* Modular implicits
* Monad/Applicative interface
* Lazy streams as unifying data structure interface

## Syntax

### Comments

Comments begin with `//` and continue until the end of the line

### Variables

Variables are defined and bound by the `=` operator:

```
x = 10
y = x * 2
z = x + y // x = 10, y = 20, z = 30
```

These variables represent names that are accessible to statements that occur below it within the same block (with several notable exceptions such as let/where bindings and within a recursive block)

It is possible to shadow a previously bound variable with a new binding:

```
x = 10
x = 11
```

It is important to note that this is shadowing, not mutation of the memory address pointed to by x.

Mutable references can be created using the `ref` function.  The value of the ref can be accessed using the `!` operator, and a new value can be assigned to the reference using the `:=` operator:

```
x = ref 0
y = x
z = !x
x := 10 // x = y = ref 10, z = 0
```

Changes to references will go beyond the current block scope, assuming that the reference itself lives beyond that scope.

### Functions

Functions are defined using the `:` symbol, preceded by the function name and its arguments, with no additional qualifiers.  A simple function can be defined in one line:

```
f a b: a + b
x = f 2 3 // x = 5
```

Alternatively a function can be defined across multiple lines, forming a block delineated by appropriate levels of indentation:

```
f a b:
  x = 2 * a
  y = 5 * b
  x + y
z = f 1 2 // z = 12, x and y are not bound outside of the function block
```

Anonymous function objects can be created using the `function` keyword and the `:` symbol:

```
adder n:
  function m:
    n + m
f = adder 2
x = f 3 // x = 5, f is a function that takes an integer and adds 2
```

Functions of no arguments are defined and called with the `()` argument:

```
f ():
  5
x = 5 + f () // x = 10
```

### let/where

The `let` and `where` keywords create block scopes whose bindings are local to the immediately following/preceding statements:

```
x = 1
let:
  x = 2
y = x
z = x // Somewhat confusing example, x = 1, y = 2, z = 1

x = y * z // x = 110, y and z are not bound beyond the where block
where:
  y = 10
  z = y + 1
```

`where` can be used in conjunction with a single line function:

```
f a b: x + y
where:
  x = a * b
  y = a + b
```

Note that this is not possible using `let` - the function parameters are not available as local variables inside the `let` scope.  Therefore the `where` binding is more expressive than the `let` binding.
(Is this syntax really necessary/useful, and does it compose well with other parts of the language?)

### Algebraic Data Types

Algebraic data types can be defined using the `type` keyword:

```
type suit: Spade | Heart | Club | Diamond
type card:
  Card int suit
  Joker
```

### Pattern matching

Pattern matching on literals and ADTs can be done using the `match` keyword:

```
x = match y:
  true: 1
  false: 0
```

The `_` symbol can be used to match anything and throw away the result:

```
x = match y:
  Card _ n: n > 10
  Joker: true
```

Cases can be qualified with arbitrary boolean expressions using the `when` keyword:

```
x = match y:
  Card _ n when n > 10: true
  Joker: true
  _: false
```

Multiple cases that evaluate to the same value can be written in one line using the `|` symbol:

```
x = match y:
  Joker | Card _ n when n > 10: true
  _: false
```

The `as` keyword can be used to rebind whole or parts of expressions that were destructured as part of the match:

```
x = match y:
  Card Heart _ as c: c 
  Joker: Card Heart 13
  _: Card Diamond 1
```

### Recursion

Recursive definitions are made using the `recursive` keyword, which creates a new block scope:

```
recursive:
  fib n:
    if n < 2:
      1
    else:
      fib (n-1) + fib (n-2)
```

Mutually recursive definitions can be made within the same block scope created by the `recursive` keyword:

```
recursive:
  odd n:
    if n == 1:
      true
    else:
      not (even (n-1))

  even n:
    if n == 2:
      true
    else:
      not (odd (n-1))
```

Mutually recursive types can also be defined using the `recursive` keyword:

```
recursive:
  type exp: Exp statement
  type statement: Assign string exp
```

Singly-recursive types do not require the `recursive` keyword.
A function definition and type definition cannot occur in the same `recursive` block scope.

### Collapsing multiple block scopes

If a keyword starting a block scope is immediately followed by another keyword starting a block scope, it is possible to collapse the two block scopes into one:

```
recursive: fib n:
  if n < 2:
    1
  else:
    fib (n-1) + fib (n-2)
```

A single expression can also be collapsed:

```
if n < 2: true
else: false
```

Single line function definitions mentioned previously are one example of this block scope collapsing.

It is not valid to collapse blocks containing multiple sub-blocks at the same level:

```
match x: Card _ _: true
  Joker: false  // Not valid

match x: Card _ _: true
         Joker: false  // Still not valid
```

Examples such as the above will need to have the sub-blocks appear on their own separate lines.

### Data structure literals and comprehensions

Lists, arrays, maps and hashtables can be created using literals and comprehensions:

Literals:

```
mylist = [1, 2, 3]
myarray = [| 1, 2, 3 |]
mymap = M{ 1:1, 2:1, 3:2, 4:3, 5,5 }
myhtbl = H{ 1:1, 2:1, 3:2, 4:3, 5,5 }
```

Comprehensions:

```
mylist = [f x for x in xlist if g x]
myarray = [| f x for x in xlist if g x |]
mymap = M{ f x : g x for x in xlist if h x }
myhtbl = H{ f x : g x for x in xlist if h x }
```

### $ syntax

The `$` symbol can be used to control the scope of nested function applications.  Tokens appearing to the right of the symbol will be considered as its own expression, to be evaluated first before the whole expression is evaluated:

```
f a b: a * b

f 1 f 2 3 // Syntax error

f 1 (f 2 3) // ok

f 1 $ f 2 3 // equivalent to the above
```

TBD: Occurrence inside pattern matching and type definitions

### Modules

```
module X:
  x = 5
  y = 10

module type A:
  type t
  x :: int
  y :: int
  z :: t

module Y:
  signature:
    type t // abstract type
    x :: int
    y :: int
    z :: t

  type t: MyInt int
  x = 5
  y = 10
  z = MyInt (x + y)

module Z1: Y // Option 1
module Z1 = Y // Option 2 - this allows module X: ... to omit the structure keyword

module Z2 :: A : X // Option 1
module Z2 :: A = X // Option 2
```

Does Option 2 conflict with another way in which modules are used in OCaml?

Is it better to require `structure` if the module also contains `signature`?

# Functors

```
module X (Y :: Z):
  signature:
    type t: Y.t list
    empty :: t
  type t: Y.t list
  empty = []
```

# First class modules

```
x = module M :: S
f x y (module Z :: S)
```
