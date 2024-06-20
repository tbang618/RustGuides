# Wrapper Types, Generics, and Traits
A **wrapper** is an object that encapsulates another data type.  In this section, we learn of Rust's two major wrapper types: **Options** and **Results**.

**Generics** or **parametrized types** allows a ..

**Traits** are Rust's version of an **class interface**: a set of methods and that an object or class  

## Options
An **option** is a special type of `enum` provided in the Standard Library.  It is a wrapper around "optional values".

### 1. Using Options
There are two variants of options (being an enum):
- `Some(<value>)` : is a wrapper for an optional value
- `None` : represents no value given

Use the `.unwrap()` method on an `Some` option to get the inner value, similar to destructuring tuples.

The type annotation is `Option<type>` where `type` is the type of the optional value.  For example: `Option<u16>`.
### 2. If-Let and While-Let
We want to check if a variable has an optional value (is `Some` and not `None`).  The straightforward solution is to use a *match-statement*.  For example, in the Rust-By-Example documentation:
``` Rust
// Make `optional` of type `Option<i32>`
let optional = Some(7);

match optional {
    Some(i) => {
        println!("This is a really long string and `{:?}`", i);
        // ^ Needed 2 indentations just so we could destructure
        // `i` from the option.
    },
    _ => {},
    // ^ Required because `match` is exhaustive. Doesn't it seem
    // like wasted space?
};
```

The shorthand for this is the `if let` statement:  
```Rust
let optional_target = Some(target);

if let Some(inner_value) = optional_target {
	// Do something...
}
```

In C, variable binding expresses a value:
```C
int x;
printf("x is: %d", x = 13); // outputs "x is: 13"

// In C, anything not 0 is "true"
// Note the following is assignment, not equality (==)
if (x = 23) { 
	printf("x is now: %d", x);
} // will output "x is now: 23"
```

This is not the case in Rust.  However, we can think of `if let` as a special case of an if-statement where destructuring an option (`let Some(x) = optional_value`) expresses `True` if the destructuring was successful and `False` if not (in which case `optional_value` is `None`). 

We have the equivalent `while let` for checking the optional value of a variable in a while-loop, the other major control-flow statement.
```Rust
while let Some(inner_value) = optional_value {
	// Do something
}
```

The loop ends when the destructuring to `None` occurs.

When you `.pop()` a value from a vector, the value is wrapped in `Some`.  So you may need to `.unwrap()` the returned value from `.pop()`; in a `while let` you may also nest `Some`:
```Rust
        while let Some(Some(integer)) = optional_integers.pop() {
            assert_eq!(integer, cursor);
            cursor -= 1;
        }
```

A `.pop()` on an empty vector returns `None`.
### 3. Partial Moves
When we *destructure* a compound data type (like a `tuple` or `struct`), we perform what is called a **partial move**.

Take the following struct:
```Rust
struct Point {
    x: i32,
    y: i32,
}

let p = Point { x: 100, y: 200 };
```
The variable `p` is the **parent variable** and we can call `x` and `y` **child variables**.

As a review: between two simple variables, reassigning a value moves the *ownership* from one variable to another.  This occurs also when passing a variable to a function, as we saw before.

The same thing happens when we move the ownership of a value from a *child variable* to another variable (such as when passing the child variable to a function).  For example:
```Rust
consume(p.y); // ownership of p.y is moved into consume()
```

The *parent variable* `p` is now no longer valid because a "part" (the child variable `y`) is invalid; however, references to other parts (child variable `x`) are still possible.  Moving the ownership of "part" of a compound data type variable is called a **partial move**.

Warning: unfortunately in this exercise, we have a parent variable named `y` and a child variable also names `y`.

In this exercise, a partial move occurs from `Point y` to `Point p` during the *destructuring* in the match arm `Some(p) => ...` of match-case.  The parent variable `y` holds a compound data type (`Point`), so it cannot be moved "as a whole" to `p`.  Instead, what is happening is that the child variables `y.x` and `y.y` are being moved to ownership by `p.x` and `p.y`.

We want to use `Point y` later, so we want to let `p` borrow the values from `y` in the match-arm.  There are two problems:
1. Match-cases by default move ownership (also described as *consuming*).
2. The match case is *destructuring*, which is an instance of *pattern matching*.

We can't use `&p` in the match-arm because that means we expect to pattern a match against a *reference* to a `Point` object; instead we just want to match to a `Point` object but not move its ownership.  A subtle point and confusing, I know!

The solution is to use the keyword `ref`: used in a pattern match (like destructuring in a match-arm), it means to match the object but then use it as a borrow.

So `Some(ref p) => ...` means to match to `Some(p)` but make `p` borrow the value from whatever it matched against.

I realize this section is probably not clear; I'll try to tidy it up in the future.
## Error Handling
The Option type is a special enum to handle optional values.  We also have the type **Result**, a special enum to handle *error values*.

### 1. Result Type
The Option type has two variants: `Some(<value>)` and `None`.  The **Result** type has two variants:
- `Ok(<value>)`: the `value` when returning a non-error.
- `Err(<error-value>)`: the `error-value` when an error occurs.  In this particular exercise, we will be returning a `String` error messages.

The type annotation is: `Result<T, E>` where `T` and `E` are generic type parameters for the `Ok` and `Err` variant, respectively.  They can be the same type, as in this exercise (both `String`).
### 2. Matching Errors
The Rust philosophy with errors: since most errors are not serious enough to stop the program, a program should be able to respond to an error instead.

In the exercise, the method `.parse::<i32>()` attempts to parse argument `item_quantity` as a number of type `i32`.  
```Rust
    let qty = item_quantity.parse::<i32>();
```
If successful, then the value of variable `qty` is the number wrapped in `Ok`.  Otherwise, `qty` is `Err(ParseIntError)`. 

To figure out whether `qty` holds `Ok` or `Err`, the straightforward solution is to use a match statement and destructure either the value or error.
```Rust
    match qty {
        Err(e) => Err(e),
        Ok(qty) => Ok(qty * cost_per_item + processing_fee),
    }
```

This checking is so common that Rust provides a shortcut with the `?` operator:
```Rust
    let qty = item_quantity.parse::<i32>()?;

    return Ok(qty * cost_per_item + processing_fee);
```

Placing `?` after a method/function that returns a `Result` does two things:
- If the method returns `Err(<some_error>)`, this error is immediately returned by the function `total_cost()`.
- If the method returns `Ok(<some_value>)`, then the operator returns `Ok(<some_value>).unwrap()` i.e. just the inner value `<some_value>`.

### 3. Return Types
The `main()` function in Rust is like any other function: it can take arguments and -importantly for this exercise- give return values.

**Void functions** are those that on success return no value or "nothing".  In Rust, a void function returns `()`, the empty tuple; we don't need to indicate this return type by default, which is what we were doing in the previous exercises.  

But in this exercise, where the `main()` function should return "nothing" on success or throw `ParesIntError`, the return type needs to be: 
```Rust
main() -> Result<(), ParseIntError>
```

To stress: without any return statements or expressions, a function will return `()` by default, which is *not* what we want here.  Instead, we need to wrap the empty tuple in `Ok`:
```Rust
return Ok( () );
```

### 4. Multiple Errors as Enum
What if we want to return multiple types of errors?  The return type `Result<T, E>` only allows specifying one error type `E`.  One solution: if we let `E` be a user-defined error `enum`, then we can return different error types as variants of that enum.

In this exercise, we want to check if `value` passed to the function `new()` is positive, zero, or negative.

We are also introduced to the keyword `as` being used to **cast between types**: `value as u64` where `value` is of type `i64`.  Without going to much into details here, this only works for primitive types and pointers; `String` and `Vec` types need to use special methods `From` and `Into` (these are called *traits* and we introduce them next section). 

### 5. Multiple Errors with Boxes/Traits
A more general way to return multiple data types is with **Boxes** and **Traits**, Rusts's equivalent of *generic types* and *interfaces*.  We'll learn more about these in depth later.

We can represent a generic type by the syntax `Box<dyn <trait>>` meaning an object type that implements the `<trait>` interface.  The keyword `dyn` is short for *dynamic dispatch* and indicates the following word `<trait>` refers to a *trait object* (interface).

In this exercise, the `main()` function needs to return two possible error types: `ParseIntError` and the user-defined `CreationError`.  Both are *propagated* with the `?` operator.  As the hint points out, all Rust errors implement the `std::error:Error` *trait* (interface).  So the generic error type in Rust is `Box<dyn std::error::Error>`.

As a passing note, what the `?` operator is doing is using `std::From::from` to convert passed errors into type `Box<dyn std::error::Error>`.

### 6. Mapping Errors
Rather than further propagate an error, it may be preferred to *handle* the error within the program.  Rust let's you *map* an error to some handling function through `map_err()`; I think of this similar to try-catch blocks in Java and C++.

In this exercise, we need to handle two separate errors that may arise at different points in the function: attempt to parse a value as a valid integer, then attempt to return the value as a positive-integer.  To handle an error in this exercise, there are few possible solutions.

First, the most straightforward is to use a match case:
1. Change the type annotation for `x` to `Result<i62, ParseIntError>`; or remove the type annotation for implicit typing.
2. Declare (but don't initialize)  a new variable `let v: i64`
3. Write a match statement on the `x`:
	- if `Ok(x)`, set `v = x`; make `PositiveNonzeroInteger::new(v)` gets the new variable `v`.  Then `v` can be used to attempt to create a new positive integer, and `map_err()` handles the possible failure.
	- if `Err(err)`, pass `err` to the error-handler `ParsePosnonzeroError::ParseInt(err)`, wrap it in an `Err` and return it.

```Rust
    let x: Result<i64, ParseIntError> = s.parse();
    let v: i64;
    match x {
        Ok(x) => v = x,
        Err(err) => return Err(ParsePosNonzeroError::ParseInt(err))
    };

    PositiveNonzeroInteger::new(v).map_err(ParsePosNonzeroError::from_creation)
```

The second solution is to map `ParseIntError` to `ParseInt()` after `.parse()` by chaining `.map_err()`.  To make things even more simpler, we can use the `?` operator. 
// TODO: more details
```Rust
fn parse_pos_nonzero(s: &str) -> Result<PositiveNonzeroInteger, ParsePosNonzeroError> {
    // TODO: change this to return an appropriate error instead of panicking
    // when `parse()` returns an error.
    let x: i64 = s.parse().map_err(ParsePosNonzeroError::ParseInt)?;
    
    PositiveNonzeroInteger::new(x).map_err(ParsePosNonzeroError::from_creation)
}
```

## Generics
In Rust, **generics** enables generalizing types and functions.

### 1. Generics with Vectors
We already encountered generics when using **vectors**.  Rust's vectors are defined to take generic types; in the type annotation we just need to fill in what specific type we want the vector to hold.
### 2. Using Generics
Generics work very similarly to generic types in Java and C++.  If you want to use a generic type `T`, you declare the generic type in the function signature, implementation signature, or struct signature as `<T>`:

```Rust
struct Wrapper <T> {
    value: T,
}

impl <T> Wrapper <T> {
    pub fn new(value: T) -> Self {
        Wrapper { value }
    }
}
```

Note the second `<T>` after the struct name in the implementation definition.  You can think of this as passing the `T` declared by `impl` being passed to the struct `Wrapper`.

## Traits
A **trait** is a collection of methods and serves as Rust's version of an *interface* in Java or C++ (where they are called abstract classes).

### 1. Defining Traits
Similar to user-defined structs, we define a trait in a `trait` block, which contains all the function signatures of the trait/interface.

The actual function definitions for a particular type goes in a separate `impl` block:

```Rust
trait AppendBar {
    fn append_bar(self) -> Self;
}

impl AppendBar for String {
    // TODO: Implement `AppendBar` for type `String`.
    fn append_bar(self) -> Self {
        self + "Bar"
    }
}
```

### 2. Defining Traits for Specific Types
Last exercise required implementing `AppendBar` for `String` types.  Now we do the same for a vector of strings, which requires a different definition of `append_bar()`.  

The function body can be different across different types, but so can the parameters, to a degree.  For this particular exercise, the vector needs to be mutable, so the parameter can be `mut self` and it still works okay.

### 3. Default Methods
Rather than implementing each method for each type that uses a trait, you can define a **default method** that the type will use.  The default method is defined in the definition of the trait itself.

Since the definition won't have the particulars about the type implementing the trait, the default method is restricted in what can be defined.  In this exercise, even though `SomeSoftware` and `OtherSoftware` have their own `version_number` field, the default method just returns the string `"Some information"`.

### 4. Parameters with Traits
Rather than a concrete type annotation, we can denote a type as implementing a specific interface/trait.  For example:
```Rust
software: impl Licensed
```

### 5. Parameters with Multiple Traits
Continuing from the last exercise, a type may implement *multiple* traits.   For example:
```Rust
impl SomeTrait for SomeStruct {}
impl OtherTrait for SomeStruct {}
```

Use the operator `+` to connect the multiple traits in an annotation:
```Rust
item: impl SomeTrait + OtherTrait
```

## Quiz 3
Though this is a quiz, we actually need to learn something a little new to progress.  

We want to declare a generic type for `grade`, but it should be restricted to types that can be printed on the screen.  The types that can do this implement the `std::fmt::Display` trait.

We saw that parameters in functions can be enforced to implement a trait.  This is actually a shorthand ("syntactic sugar") for something called **trait bound syntax**:
```Rust
pub fn notify<T: Summary>(item1: &T, item2: &T) {
```
where `Summary` is a trait.  We are *bounding* the possible types that `T` may be.   

When we want to bound a generic in an `impl`, we must use this syntax; there is no shorthand like for bounding parameters:
```Rust
impl <T : std::fmt::Display> ReportCard <T> {
	...
}
```

In the exercise, change the types were necessary to the correct generic type.
