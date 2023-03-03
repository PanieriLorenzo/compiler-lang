# Compiler Lang
> Disclaimer: the name is a temporary place-holder

A multi-paradigm programming language for making tools

## Introduction

Compiler-lang (CL) is a light-weight, statically-typed compiled language designed for making tools, such as parsers, CLIs and compilers. It is somewhat general purpose, but If you try and implement complex applications in compiler-lang, you will likely have a bad time.

Compiler-lang is multi paradigm, combining elements of modular, functional and procedural programming. It has no classes, instead it uses *modules* as the basic unit of encapsulation. Variables are immutable, with the only exception being variables in the top-level of a module, which are allowed to be declared mutable. Functions are pure, and they can capture their environment (they are closures). Procedures are functions which are allowed to mutate the top-level variables. Complex data can be modelled using algebraic types (similar to structs and enums).

Modules, unlike classes, have a static lifetime (they exist for the entire duration of the program execution) and there can only ever be one instance of a module. Modules are used to structure the overarching architecture of a program, but they are not portable containers like classes are. 

CL is pass-by-value by default. Because almost every variable is immutable, the compiler can avoid performing copies of data, but from the user's stand point, the language behaves as if copies were made every time a function is called. Since mutability is restricted to the module top-level, mutable variables always have a static lifetime, meaning we can alwyas safely reference them. This has to be done explicitly, using the reference type. This allows to pass top-level variables by reference.

In terms of style and sytax, CL prefers a good balance between conciseness and legibility, limiting the number of keywords and weird symbols. Another important stylistic choice is to minimize ambiguity, by making things more explicit. For instance, all variables are immutable, unless we explicitly state that they aren't. All variables are passed by value, unless we explicitly reference them. Functions don't capture variables unless we explicitly request it. Type inference is quite conservative, meaning that the compiler will only infer the type of a variable if it is trivial to deduce it from context.

## Why

I was playing around with making a compiler in Rust and in Python and was frustrated by both languages. Rust forces you to write good code, but sometimes, if I'm just prototyping, this distracts me from quickly developing something. Python hides away too much information, the way objects are passed to functions is unintuitive, and trying to guess whether you are making a copy of sometething or you are mutating the original becomes a headache.

Similarly, I was also playing around with Ruby, which has great tools for string processing, but again, I was frustrated by its object passing semantics. I wanted an elegant and simple language like Python or Ruby, but with clear copy/reference semantics.

## Syntax

The syntax is inspired by Rust, Ruby, OCaml and Julia.

### Modules

Modules can be defined in two ways.
1. Make a CL source file, the name of the module is the name of the file:
```julia
# foo.cl
use bar;

# bar.cl
hello = fn {
	println!("hello world!");
};
```
1. Declare and define inside the parent module:
```julia
# foo.cl
mod bar {
	hello = fn {
		println!("hello world!");
	}
}
```

To access a member of a module, you use the scope resolution operator `::`:

```julia
mod foo {
	const bar = "hello world!";
}

hello = fn {
	println!(foo::bar);
}
```

### Variables and storage modifiers

Most constructs in CL are stored in variables, including functions

```
an_integer: u16 = 42;
a_string = "foo";
a_function = fn -> u16 { return 42; };
a_type = type {a: u16, b: u16};
```

Variables are immutable by default.

Mutable variables use the storage modifier `mut`, they can only be declared at the top-level of a module

```
mut i_am_mutable = 1;

increment! = fn {
	self::i_am_mutable += 1;
}
```

While immutable variables don't change their value during execution, their value may still be unknown at compile time. You can enforce compile time evaluation with the `const` modifier:

```
const i_am_constant = 1;

# this is illegal
const i_am_constant = input!(u16);
```

By default, variables are only accessible from within a module and from the direct ancestors of the module:

```
mod foo {
	bar: u8 = 42;
};

# this is ok
main = fn {
	println!(foo::bar);
};
```

```
bar: u8 = 42;

# this won't compile
mod foo {
	println!(::bar);
};
```

You can change this behaviour with the `pub`, `internal` and `hidden` modifiers.
- `pub` makes the variable public, which means it is visible from any module, usually this is used for library functions.
- `internal` makes the variable visible from anywhere within the same module tree as the variable.
- `hidden` completely hides the variable, also from ancestor modules.

