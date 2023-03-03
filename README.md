# Compiler Lang
> Disclaimer: the name is a temporary place-holder
A multi-paradigm programming language for making tools

## Introduction

Compiler-lang (CL) is a light-weight, statically-typed compiled language designed for making tools, such as parsers, CLIs and compilers. It is somewhat general purpose, but If you try and implement complex applications in compiler-lang, you will likely have a bad time.

Compiler-lang is multi paradigm, combining elements of modular, functional and procedural programming. It has no classes, instead it uses *modules* as the basic unit of encapsulation. Variables are immutable, with the only exception being variables in the top-level of a module, which are allowed to be declared mutable. Functions are pure, and they can capture their environment (they are closures). Procedures are functions which are allowed to mutate the top-level variables. Complex data can be modelled using algebraic types (similar to structs and enums).

Modules, unlike classes, have a static lifetime (they exist for the entire duration of the program execution) and there can only ever be one instance of a module. Modules are used to structure the overarching architecture of a program, but they are not portable containers like classes are. 

CL is pass-by-value by default. Because almost every variable is immutable, the compiler can avoid performing copies of data, but from the user's stand point, the language behaves as if copies were made every time a function is called. Since mutability is restricted to the module top-level, mutable variables always have a static lifetime, meaning we can alwyas safely reference them. This has to be done explicitly, using the reference type. This allows to pass top-level variables by reference.

In terms of style and sytax, CL prefers a good balance between conciseness and legibility, limiting the number of keywords and weird symbols. Another important stylistic choice is to minimize ambiguity, by making things more explicit. For instance, all variables are immutable, unless we explicitly state that they aren't. All variables are passed by value, unless we explicitly reference them. Functions don't capture variables unless we explicitly request it. Type inference is quite conservative, meaning that the compiler will only infer the type of a variable if it is trivial to deduce it from context.
