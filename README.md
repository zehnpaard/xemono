Xemono
=

Statically typed impure functional programming language based strongly on OCaml, with influences from Python, Haskell and Clojure

Features:

* Hindley-Milner type inference
* Algebraic Data Types and pattern matching
* Lexical scoping
* Variable definitions as name-bindings
* Mutable ref and array types
* Significant indentation
* Literals and comprehensions for common data structures
* where and let bindings to specify variable scope

Areas to investigate:

* Modular implicits
* Monad/Applicative interface
* Lazy streams as unifying data structure interface

##Syntax

###Variables

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

###Functions

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

###let/where

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
