These final set of exercises introduce some more miscellaneous, but no less interesting or important, topics in Rust.
## Macros
As in other languages, macros allow for meta-programming in Rust.  These exercises only give a cursory introduction to macros.
### 1. Declarative Macros
The most common type of macros used in Rust are **declarative macros**.  These macros essentially fill in Rust code.  Some common ones we've already seen:
- `println!` to print to standard output
- `Vec!` to create vectors
- `panic!` to panic

These macros take arguments called **tokens** which can be further parsed before filling in code.  This exercise doesn't use any tokens.

To create our own is similar to declaring a function:
``` Rust
macro_rules! my_macro {
    () => {
        println!("Check out my macro!");
    };
}

fn main() {
    my_macro!();
}
```

Notably, the definition is prefaced with the **construct** `macro_rules!` and calling the macro requires the `!` suffix.

The other class of macros are **procedural macros**, which behave more like functions rather than merely "filling in code".  These exercises won't cover them.
### 2. Order of Definition
Unlike functions, where the macro is defined matters.  Like functions in C, the declaration/definition of a Rust macro must come before its use.

### 3. Exporting Macros
To make a macro available outside of the module it is declared within (for example, the parent file), use the annotation `#[macro_export]` like in an example of `vec!`:
``` Rust
#[macro_export]
macro_rules! vec {
    ( $( $x:expr ),* ) => {
        {
            let mut temp_vec = Vec::new();
            $(
                temp_vec.push($x);
            )*
            temp_vec
        }
    };
}
```

### 4. Passing Tokens
Macros essentially work like match-statements but for *Rust code* (the tokens) instead of values.  However, unlike regular match-statements, branches must be blocks (encased in curly-brackets `{` and `}`) and separated by semi-colons `;`.

---

## Clippy
Rust's official static code analysis tool, or **linter**, is **Clippy**.  It is a *component* of the Rust toolchain and can be added with:
``` Shell
$ rustup component add clippy
```

Clippy is automatically installed if you have installed Rustlings.  The following exercises just demonstrate Clippy in action, as well as some common mistakes to catch.  Just follow the compiler errors/warnings to fix the code.

### 1. Clippy in Action: Constants
In this exercise, Clippy is telling you to use Rust's officially provided constant for Pi (`f32::constants::PI`) rather than using your own.

### 2. Clippy in Action: If-Let
In this exercise, Clippy is telling you that the for-loop over `Option` types is better handled by the `while let` or `if let` statement.

### 3. Clippy in Action: Minor Mistakes
Clippy and the compiler note some minor mistakes or adjustments in the code:
- Since `my_option.unwrap()` always panics, replace it with an explicit `panic!()`
- There's a missing comma in the declaration of `my_arr`.
- Since `my_empty_vec` is an empty data structure anyway, you could get rid of its declaration and replace instances of `my_empty_vec` with `()` in the following line.
- A more efficient way to swap the values of two variables is with `std::mem::swap()`

---
## Type Conversion
Rust has a few official ways to convert values between different types.

### 1. Type Casting with `as`
The keyword `as` can be used to typecast between different *primitive data types*.  The syntax is simple, for example:
``` Rust
13 as f64
```

The value `13` will be used as a `f64` type number.

Type casting can also work with pointers and addresses.

### 2. Traits `From` and `Into`
#### From
The `From` trait and associated function `::from()` is used to convert a value into a specified type.  For example, to convert the string literal `'Hello'` into a `String`:
```Rust
String::from('Hello');
```

Rust's standard library already implements the `From` trait for the `String` type, which is why the above works.  We can also implement the `From` trait for user-defined types ourselves; we just need to define the `::from()` associated function for our type:

``` Rust
impl From for <user-defined-type> {
	fn from(<...>) {
		...
	}
}
```

This exercise has us implement `From` for the struct `Person`.
#### Into
The `Into` trait has the method `.into()` that is used to convert a value into a specified type.  For example, to convert a string literal `'Hello'` into a `String`:
``` Rust
'Hello'.into<String>();
```

Again, we can implement `Into` for our user-defined types as well.

Using `.into()` is useful because we can often leave out the type specification and leave the compiler to infer the type we are converting the value into.  For example:
``` Rust
let new_string : String = 'Hello'.into();
```

The `Into` trait isn't used this exercise, but it's good to know, as it complements `From`.
#### Parsing
Strings and string literals have the method `.parse::<T>()` which attempts to parse their contents as type `T`, returning a `Result`. 

This is useful for converting a string, either of `String` or `&str`, into a number, as this requires special consideration because the string may not actually be convertible to the desired type.  For example:

``` Rust
let x = '130'.parse::<f32>();
let y = 'Hi'.parse::<f32>();

// x.unwrap() == 130
// y == Err(ParseIntError)
```

Notice we have another instance of the *turbo fish* syntax `::<T>`, which we also saw in the section on iterators.

### 3. Trait `FromStr`
The previous exercise was a situation of taking a string input and converting into struct.  Such a case is better served by the `FromStr` trait and the `from_str()` associated function, because it handles errors for when the string cannot be properly converted.

There's a few different ways to finish this exercise.  Some hints:
- A method to check if a string is empty: `.is_empty()`
- There's no way to count the elements of an iterator without consuming it; the method `.count()` literally uses `.next()` until `None` is returned.
- Following from the previous hint, splitting a string into an iterator with `.split()` does not consume the string, so you can create multiple iterators from the same string.
- Using `.map_err()` and `?` might be useful.

### 4. Traits `TryFrom` and `TryInto`
The trait `TryFrom` is the same as the trait `From`, except it also performs error handling i.e. the return type is a `Result` instead of the desired type itself.

As with `From` and `Into`, there is the mirror trait `TryInto` which has the method `.try_into()`.

Types that implement the `TryFrom` trait have the associated function `::try_from(<value>)`.  Sine this function returns a `Result`, you can chain a `.map_err()` or `.or()` method to handle an `Err` result.  For example:

``` Rust
u8::try_from(red).or(Err(IntColorError::IntConversion))?
```

In this exercise, we are converting a tuple of three numbers into the struct `Color`, which has three fields for the RGB values of type `u8`.  The exercise asks you to handle errors when the given numbers cannot be converted into type `u8`.

Some hints:
- Once you implement the `TryFrom` trait for the tuple, you can use it in the implementation for the array and slice.
- The type inference for `.try_into()` is very strong; you can leave off *a lot* and it still works.
- The type `u8` is bounded by the values `[0, 255]`.
- You can destructure tuples, arrays, and slices.  Note that destructuring a slice returns *references* to its values.
- The operator `?` works on `.unwrap()`, `.map_err()`, and `.or()`; `.or(<value>)` is very convenient in this exercise. 

### 5. Reference to Reference
In C/C++, we can cast certain values as pointers, as in `(int *) someVariable`.  The `AsRef` trait does something similar with the `.as_ref()` method.  We are taking some value, typically a number or another pointer, and converting it to a pointer of some type.  In other words, converting to a reference.  The Rust documentation calls this *reference-to-reference conversion*.

To be clear, `.as_ref()` is not the same as referencing/borrowing a value, though they behave the same in most cases:
``` Rust
let x = Box::new(5i32);

let y: &i32 = x.as_ref(); // this is not the same...
let y: &i32 = &x;         // ...as this.
```

Borrowing is actually handled by another trait, aptly named `Borrow`.

The best use case of `AsRef` is as a trait bound in function signatures; this allows us to accept arguments that are *references* of various types (like generic types, but for references).  For example, our function can use the trait bound `AsRef<str>` and it will accept arguments of type `String` and `&str`, since both implement the `AsRef` trait, meaning they can be referenced as string slices.

``` Rust
fn is_hello<T: AsRef<str>>(s: T) {
   assert_eq!("hello", s.as_ref());
}

// The following two assert as true:
let s = "hello";
is_hello(s);

let s = "hello".to_string();
is_hello(s);
```

The trait `AsMut` is the same as `AsRef` except for mutable references.  It has the method `.as_ref_mut()`.  As references, they be dereferenced with the `*`, as in `*arg.as_mut()` for some variable `arg`.

In this exercise, we pass string slices (with slices just being pointers, like in C) to functions.

--- 
## Other Learning Material
Rustlings covers most of the major topics in Rust, but not in much detail.  Here are some other resources for learning Rust:

- The official [Rust Website](https://www.rust-lang.org/learn) lists many other resources for further learning.
- Brown University has an interactive version of the [Rust Book](https://rust-book.cs.brown.edu/).
- Another hands-on resource for learning Rust is [100 Exercises to Learn Rust](https://rust-exercises.com/).
- More on Rust lifetimes at [Lifetime Kata](https://tfpk.github.io/lifetimekata/).
- More on Rust macros at [The Little Book of Rust Macros](https://veykril.github.io/tlborm/).

