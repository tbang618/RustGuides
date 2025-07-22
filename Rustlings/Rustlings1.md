# Basic Rust

## Introduction
These are the notes I wrote while completing the Rust tutorial course: **Rustlings**.  Some parts include full answers for the exercise, as to better explain concepts: *so be warned!*

This is accurate for Rustlings version 5.6.1 (2024).
## Installation
Install Rustlings with:
``` Shell
$ curl -L https://raw.githubusercontent.com/rust-lang/rustlings/main/install.sh | bash -s mypath/
```
Where `mypath/` is where Rustlings (including the exercises) will be installed.
## Start
Instructions for Rustlings can be read with:
``` Shell
$ rustlings
```

To start the course:
``` Shell
$ rustlings watch
```

---
## Intro
### 1. Comments
Some introductory notes on Rustlings.  To begin, comments in Rust:
+ In-line comments are marked with `//`
+ Multi-line comments are wrapped with `/* <code> */`
### 2. Printing
The macro `println!` acts similar to the function `printf()` in the C language, where you can format output text with arguments:
``` Rust
println!("Hello {}", "World!");
```

We'll learn more about macros later on.

---
## Variables
### 1. Declaring Variables
Variables are declared with the `let` keyword.
### 2. Type Annotation
The compiler can usually infer the type of a declared variable, but we can also explicitly declare a type.  This is called **type annotation**.  For example, in this exercise:
``` Rust
let x: i32 = -13;
```
The variable `x` is of type signed 32-bit integer.

The list of numeric types are:

| Unsigned | Signed |
| -------- | ------ |
| `u8`     | `i8`   |
| `u16`    | `i16`  |
| `u32`    | `i32`  |
| `u64`    | `i64`  |
| `u128`   | `i128` |

We'll come across more types as the course progresses.

### 3. Binding Values
Variables need to be bound to a value before they can used, as in most programming languages.

### 4. Mutable Variables
In Rust, variables are *immutable* by default: once a value is bound, it cannot be changed.  To make a variable *mutable*, we use the `mut` keyword in a variable declaration:
``` Rust
let mut x = 1729;
```

### 5. Shadowing
**Shadowing** allows the reuse of a variable name.  This is useful for:
- changing a variable's type
- reusing the same variable name

To shadow, re-declare the variable with `let`.

### 6. Constants
Constants in Rust are declared with the `const` keyword and require type annotation:
``` Rust
const NUMBER : u16 = 123;
```

---
## Functions
### 1. Declaring Functions
Functions are declared using the following syntax, known as the **function signature**:
``` Rust
fn function_name(<parameter> : <type>) -> <return-type> {
	<body-statement>
	...
	<return-statement>
}
```

Unlike the C language, functions don't need to be declared before their use.

### 2. Parameters
Functions can be defined with parameters.  These parameters need type annotation.
``` Rust
fn call_me(num:i32) {
	...
}
```

### 3. Arguments 
Function calls require all expected arguments to be passed, with matching types.

### 4. Return Values
A function that returns a value needs to declare the type of its return value.  This is done in its function signature.

### 5. Expressions vs Statements
In Rust, a **statement** is a line of code that does not output a value.  An **expression** is a line of code that outputs a value.  

A statement is distinguished by ending with a semi-colon (`;`) as in the following example:
```Rust
3 * 3;
// outputs nothing
```

An expression does not end with a semi-colon:
``` Rust
3 * 3
// outputs 6
```

A function will return the value of the first statement marked by keyword `return`.  Alternatively, the last *expression* of a function will be taken as the return value; no other statements can come after or a compiler error will occur.

In other words, a function that returns a number can be defined as:
``` Rust
fn square(num: i32) -> i32 {
    return num * num;
}
```

Or, with an expression:
``` Rust
fn square(num: i32) -> i32 {
    num * num
}
```

---
## If
### 1. Control-Flow 
The **guards** in an **if-else** statement do not need be encased in parentheses (but you can if you want), however they do need to be **boolean** expressions.  Unlike C, where `0` is `false` and every other value is `true`, Rust has explicit boolean types and they do not cast into integers (like in C++).

The branches are called **conditionals**, which must be expressions (ends by giving back a value) and need to be encased in *curly-brackets*.

``` Rust
if a > b {
	return a;
} else {
	return b;
}
```

### 2. Conditionals
Multiple conditionals are possible using `else if`, but all branches need to return the *same type*.

``` Rust
    if fizzish == "fizz" {
        "foo"
    } else if fizzish = "fuzz" {
        1
    } else {
	    "bar"
    }
```

The above *does not compile* because the first and last branches return a `String` and the second returns a number.

### 3. Conditionals Continued
When returning a number from a multi-branch if-else, all numbers from each branch need to be of the same type: no mixing of floats and unsigned/signed.

Also note that an if-else can be used to bind a value to a variable:
``` Rust
let identifier = if animal == "crab" { } ...
```

## Quiz 1
Putting everything we learned so far together.
