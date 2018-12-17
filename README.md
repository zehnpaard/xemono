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
