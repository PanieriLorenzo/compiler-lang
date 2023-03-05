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

The syntax is inspired by Rust, Ruby, OCaml and Julia.

## Identifiers

Identifiers have the form `[a-zA-Z_][a-zA-Z0-9_]*` but they follow some style rules that can optionally be enforced (with pedantic warnings):

- Modules: `snake_case`
- Variables: `snake_case`
- Functions `snake_case`
- Types: `PascalCase`
- Constants: `SCREAMING_CASE`

There are also some special identifiers:

- Procedure: `snake_case!`

## Modules

Modules can be defined in two ways.
1. Make a CL source file, the name of the module is the name of the file:
```julia
# foo.cl
use bar;	# bar.cl
```

1. Declare and define inside the parent module:
```julia
# foo.cl
mod bar {
	hello: fn = {
		println!("hello world!");
	}
}
```

To access a member of a module, you use the scope resolution operator `::`:

```julia
mod foo {
	pub const bar = "hello world!";
}

hello = fn {
	println!(foo::bar);
}
```

If the name of a module is not in scope, you can locate it using an *anchor* for the resolution operator:
- `self::foo`: used to capture module variables, because capturing is explicit.
- `::foo`: absolute path, from package root. If you don't have a package configured, this is the entry point.
- `super::foo`: relative to parent, can be stacked: `super::super::foo`.

Importing a module is quite easy
```julia
# brings module package into the hierarchy
use package;	

# now we can access it as usual
package::foo;
```

You can also import sub-modules
```julia
use alpha::bravo::charlie;

# now the module charlie is in scope
charlie::foo;
```

Module resolution can never be omitted, but it might get tiring to write out the module name every time, in these cases we can alias an imported module:

```julia
use very_long_module_name as short;
short::foo;
```

Note that if two modules import the same module, two independent instances of the module are created. You can use the `global` storage modifier to make variables shared across all instances of a module.

## Variables and storage modifiers

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

By default, variables are *private*, meaning they cannot be accessed from outside:

```
mod foo {
	bar: u8 = 42;
};

# this is illegal
main = fn {
	println!(foo::bar);
};
```

You can change this behaviour with the `pub`, `protected` and `global`
- `pub` makes the variable accessible from outside the hierarchy
- `protected` makes the variable accessible only from direct ancestors
- `global` makes the variable be shared across all instances of the module.


## Type system

The typing system is statically and strongly typed, and uses algebraic types.

There are several built-in types:
- Atomic types:
  - Never: `never`
  - Nothing/null: `nil`
  - Unsigned integers: `u8`, `u16`, `u32`, `u64`, `ubig`
  - Signed ingegers: `i8`, `i16`, `i32`, `i64`, `ibig`
  - Booleans: `bool`
  - Floating point: `f16`, `f32`, `f64`
  - UTF-8 character: `char`
  - UTF-8 string: `str`
  - Regex pattern: `rxp`
  - Type: `type`
  - Function/procedure: `fn`
- Data structures (sum types):
  - List: e.g.: `list(u8)` or `[u8]`
  - Tuple: e.g.: `tuple(u8, u8)` or `(u8, u8)` or `(a: u8, b: u8)`
  - Array: e.g.: `array(u8, 10)`
  - Dictionaries: e.g.: `dict(str, u16)`
- Pointers:
  - Reference: e.g.: `ref(u8)`

### Literals

Ingegers:
```julia
42;
+42;
-42;
100_000_000;	# digit separators
0xff_ff_ff;	# hexadecimal
0b0000_0000;	# binary
```

Floats:
```julia
3.14;
+3.14;
-3.14;
.5;
5.;
3.14E-10;
```

Strings:
```julia
"this string supports interpolation and escaping, e.g.: \"{2 + 3}\""
'this string does not use escaping and interpolation'
```

Misc:
- Null/nothing: `nil`
- Function: 
  - `(a: u8, b: u8 = 5) -> (u8, u8) { ... }`

### Algebraic types

You can define custom types by using algebraic types.

The general syntax for defining a type is:

```julia
<name>: type = <type_constructor_expression>
```

The simplest type you can define is a *unit* type, which has a single variant with no value:

```julia
Foo: type = ();
```

Usually you will use the built-in unit type `nil`, which you could model as:

```julia
nil: type = ();
```

The next simplest type is a *unary* type. A unary type is like an alias for another type:

```julia
LongInt: type = i64;
```

You can also create types that combines multiple other types. This is often called a *struct*, *tuple* or *product type* in other languages:

```julia
Point2D: type = (f32, f32);
Point3D: type = (x: f32 = 5.2, y: f32, z: f32 = 3.2);	# named and with defaults
Bytes: type = [u8];
Buffer: type = array(u8, 128);
Translations: type = dict(str, str);
```

You can make a type with multiple *variants* with the `|` (pipe) operator. This is often called an *enum* or a *sum type* in other languages:

```julia
SomeResult: type = Ok | Err(str);
```

### Type inference (`auto` and `type`)

The special type `auto` can be used to explicitly request type inference.

```julia
foo: auto = 3.2 + 3.3;
```

You can limit the allowed types by using arguments:

```julia
foo: auto(i8, i16) = bar;
```

This is especially useful for defining generic functions:

```julia
Real: type = auto(f16, f32, f64);
mul_add: fn = (a: Real, b: type(a), c: type(a)) -> type(a) {
	return a * b + c;
};
```

Here the `type(a)` is a type constructor that takes a variable and evaluates to the type of that variable at compile time, here it is used to force all arguments to have the same type as the first.

## Pattern matching

Sum types can be matched with pattern matching:

```julia
Color: type
	= RGBA(u8,u8,u8,u8)
	| HEX(u32)

a: Color = RGB(0xfedf0000);
match a {
	case RGBA(r,g,b,a) {
		# something
	},
	case HEX(hex) {
		# something
	}
};
```

Pattern matching is an expression:

```julia
foo: auto = match bar {
	case A(a) {
		return a;
	},
	case B(b) {
		return b;
	},
}
```

Pattern matching can also use regular expressions:

```julia
a: str = input!;

Number: type
	= Int(u64)
	| Float(f64);

b: Number = match a {
	case rxp('(\d+)') {
		return Int(u64($1));
	},
	case rxp('(\d+.\d+)') {
		return Float(f64($1));
	},
};
```

## Functions and procedures

The distinction between functions and procedures is that functions have no side effects, whereas procedures do. Functions and procedures are variables of the `fn` type. Procedures have a `!` at the end of their identifier.

```julia
increment: fn (a: u8) -> u8 = {
	return a + 1;
};

mut counter: u8 = 0;
increment! : fn (a: ref(u8)) -> () = {
	a += 1;
};
```

Functions can capture their environmnet, by using the `self::` anchor:

```julia
mut counter: u8 = 0;
increment! : fn = {
	self::counter += 1;
};
```

Functions allow partial application and currying:

```julia
foo: fn (a: f32, b: f32, c: f32) -> f32 = {
	return a + b + c;
};

bar: auto = foo(1.0,,3.0);
println!(type(bar));
# fn (b: f32) -> f32

baz: auto = foo(1.0)(2.0);
println!(type(baz));
# fn (c: f32) -> f32
```

Functions can be passed to other functions:

```julia
foo: fn (a: fn f32 -> f32) -> f32 = {
	return a(1.0);
};
```

Functions can be chained by using the `.` (dot) operator, which applies the function on the right to the variable on the left.

```julia
a: f32 = [1.0, 2.0, 3.0]
	.iter
       	.map(fn (a: f32) -> f32 { return a * 2.0; })
	.fold(+);
```

Functions can be overloaded, if multiple definitions match the arguments, the most specific wins.

```julia
mul_add: (a: auto, b: auto, c: auto) -> auto = {
	return a * b + c;
};
mul_add: (a: str, b: str, c: str) -> str = {
	return a.repeat(b).concat(c);
};
```

Note that this syntax is allowed for all variables, but it is discouraged.

## Built-ins



## Cool stuff

### Efficient regexes

- Literal and `const` regexes are compiled at compile time, if you want to be sure, use the `const` modifier.
