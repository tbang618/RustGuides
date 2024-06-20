# Advanced Topics
You can get pretty far with just the previous three sections.  However, this section includes important topics in creating more complex Rust programs.
## Lifetimes
The time from a variable's initialization to its invalidation, whether its value is moved or it goes out of scope, is called its **lifetime**.  It may be thought of as the "lifetime of a scope".  This is what the compiler checks to make sure borrows/references are legal.   Sometimes we need to explicitly indicate lifetimes for the compiler, though it can infer a lot by itself.

These exercises don't go into much detail of lifetimes, giving only an introductory understanding.
### 1. Declaring Lifetimes
If a function returns a reference, it needs to know in what lifetime (scope) the reference is valid. 

A **lifetime parameter** denotes the lifetime of a variable and is declared like a generic, except it is prefixed by a single quotation.  In this exercise:
```Rust
fn longest<'a>(x: &'a str, y: &'a str) -> &'a str {
...
}
```

Indicating a lifetime of a variable is called **lifetime annotation**.  It allows us to name a particular lifetime, like `'a`.  In this exercise, we are returning one of two borrowed string slices.  We are stating that both `x` and `y` borrow values with a lifetime `'a` and the return value will have a lifetime at most as long as `'a`.   

In another perspective, using lifetimes in a function call ensures that the arguments are valid borrows when they are passed.
### 2. Lifetimes across Scopes
Curly braces denote a separate scope.  The problem in this exercise is that `string2` only lives from line 23 to 25, but `result` -which potentially borrows its value from `string2`- lives from line 24 to 26 and *outlives* the owner of its value.  

The two solutions are:
- shorten the lifetime of `result` to match `string2` (by moving `printlin!` into the inner scope).
- increase the lifetime `string2` to encompass `result` (by moving into the scope of `main()`).
### 3. Lifetimes for Struct Fields
If we define a struct whose fields are *references* to other variables, we need to indicate the lifetimes of those variables.

Again, lifetimes are declared like generics, except prefixed with `'`.

```Rust
struct Book<'a> {
    author: &'a str,
    title: &'a str,
}
```

Again, we are saying that `author` and `title` should not life longer than the variables from which they are borrowing.

## Tests
Tests are used in program verification.  Test are special functions in Rust, marked by the directive `#[test]` that are compiled into a separate binary called the **test runner**.  By default, when Cargo creates a new project it also creates a separate module called `test` and places there some template test functions.  The module is annotated by the directive `#[cfg(test)]`, which tells the compiler to only run the module when in *test mode* i.e. `cargo test`.
### 1. Test Macro: Assert
Rust provides some macros to use within test functions.  In this exercise we explore `assert!(<condition>)`, which only passes the test if the `<condition>` evaluates to `true`.
### 2. Test Macro: Assert Equal
Another macro is `assert_eq!(<value1>, <value2>)`, which tests if two values are equal.
### 3. Testing Functions
We can pass functions to the previous test macros.  Remember the negation operator `!` if we want to test a function returns `false`.
### 4. Testing Panics
We can add additional meta-information about a testing function with additional compiler directives after `#[test]`.  The one we use in this exercise is `#[should_panic]`, indicating we are testing if a function correctly panics. 

## Iterators
In Rust, an **iterator** is a *trait*: a common interface (a set of methods and structs) for Rust's container types (also called **iterables**) that allow processing their elements (called **items**) sequentially.  

Perhaps confusingly, an iterator itself is not actually a data type (as in some other languages), but an interface that encapsulates how different iterables (such as a vector) would implement itself as an iterator.  However, we treat an iterator like an object, such that it has methods on itself.  The type annotation of an iterator is:
```Rust
let mut v : std::slice::Iter<i32> = vec![-1, 0, 1, 2, 3, 4].iter();
```
where `.iter()` creates an iterator from a vector of signed integers.
### 1. Creating Iterators
There are three methods to create an iterator over a containter type:
- `.iter()` will create an iterator that generates elements of type `&T`.
- `.iter_mut()` will create an  iterator that generates elements of type `&mut T`.
	- Changing the elements yielded by this iterator will mutate the original elements of the iterator in place.
- `.to_iter()` will create an iterator that "either yields `T`, `&T`, or `&mut T` based on the context".  That's taken straight from the Rust documentation; I won't go into further details and won't use this for the exercises.

Rust's strings can also be iterated over, but use special methods.  Namely, to create an iterators over a string slice (`&str`) we need to use the `.chars()` method, not `.iter()`.
#### Next Item
To get the next element from an iterator, the method `.next()` is used on the iterator.  The elements produced by an iterator are wrapped in the `Option`s type.  When an iterator is depleted (finished going through all the elements), `.next()` returns a `None`.
#### Collecting Iterators
To take the remaining elements in the iterator and return a container of the type from which the iterator was created, use the method `.collect()`.
### 2. Using Iterators
This exercise goes over using iterators.

In the first step, we are defining a function that capitalizes the first letter of a *string slice* (`&str`).  Note that string slices have their own method of creating an iterator from their characters: `.chars()`.  This acts similar to `.iter()`, in that the original iterable is not mutated.  To collect the iterator back into a string slice, we use the method `.as_str()`.  Remember that the method `.to_uppercase()` works on a string slice and returns a string slice.

In the second step, the function parameter will be of a `slice` type of `&str` elements.  The `.iter()` method works to create an iterator.  In this step, we introduce *for-loops over iterators*.  While we can make a regular for-loop that tests if the next item is `None`, Rust provides a syntax to iterate over an iterator:
```Rust
for item in iterable.iter() {
	println!("{item}");
}
```

The third step is similar to the second step, but instead of pushing the results of each iteration onto a vector, you should look into appending the result onto a string.

### 3. Mapping and Collecting
#### Map
This exercise makes use of the `.map()` method on iterator types.  We've seen `.map()` earlier when learning about vectors.  The syntax is:

```Rust
some_iterator.map(|item| <expression with item>);
```

The syntax `|item| <expression with item>` is known as a **closure**, also referred to as an **anonymous function**.  The body `<expression with item>` can be an expression (simple as `item + 1`) or another function call (such as `divide(item)`).  We'll learn closures more in detail later.

If `some_iterator` is a mutable iterator (created with `.into_iter()` or `iter_mut()`), then the same iterator is returned and the closure must return the same type as the original item.  For non-mutable iterators, we can `.map()` items into a new type.

#### Collecting
To turn an iterator back into a container type, we use the `.collect()` method.  Generally, the compiler can infer what container type should be returned.  Generally, it looks at either the original container type of the iterator, the type annotation of a binding variable, or the type of the items.

The `.collect()` method can return a non-container type as well.  Two common situations:
- an iterator of `char` items collects into a `String`.
- an iterator of `Result<T, E>` items will return `Result<Collection<T>, E>`.

The second instance occurs in part-two of this exercise.

We can explicitly state the container type by a special syntax called **turbo fish**: `::<T>`.   For example, to explicitly collect an iterator into a vector (ignoring any compiler inferences):
```Rust
let some_vector = some_iter.collect::<Vec<_>>();
```

### 4. Functional Programming with Iterators
If you ever used a functional programming language -like Racket- then you may be familiar with functions `foldl()` and `foldr()`.  Similar functions exist in Rust that operate on iterators.

Both functions act as shorthand for *reducing* or recursive accumulation .  So something like:
```Rust
let init = 0;
let sum = ((((init + 1) + 2) + 3) + 4);
```

Can be written as:
```Rust
let v = vec![1, 2, 3, 4].iter();
let init = 0;
let sum = v.fold(init, |acc, item| acc + item);
```

The **accumulator** `acc` is the result of the last recursive step; `item` refers to the value we are dealing with in the current step; the body of the closure details how to relate the current `item` to the `acc`.  In the above example, we just add `item` to the accumulated value.  The **initial value** `init` is the starting value of the accumulator.  At the end, the accumulated result `acc` is returned.

The above began at the "left" of the iteration (starting from `1` and ending with `4`).  The reverse, starting from the "right", can be done with `.foldr()`.  It is equivalent to:
```Rust
let init = 0;
let sum = (1 + (2 + (3 + (4 + init))));
```

#### Range
Both `.fold()` and `.foldr()` work on iterators.  For a simple range of numbers, that means creating a vector of those numbers and then creating the iterator.  As a shorthand, Rust provides **Range** operations.  We already saw this in a for-loop:
```Rust
for x in 1..10 {
	println!("{}", x); // Prints up to 1, 2, 3... 9 (but not 10)
}
```

A range can be thought of as a shorthand for creating an iterator of numbers.  For example, all of the following work just as on iterators:
```Rust
let r = 1 .. 10;
r.next(); // gets 1

let sum = (2 .. 15).fold(0, |acc, item| acc + item);
```

As an aside, technically a range is a distinct data type (`std::ops::Range`) that happens to implement the iterator trait (`std::slice::Iter`), so that it may be used as an iterator.  If it quacks like a duck and waddles like a duck...

### 5. Iterators of Hash Maps
For an iterator of a **hash map**, the items are tuples consisting of each Hash Map's entry i.e. a ke and value pair.  For example:
```Rust
// some_hash_map = {"one": 1, "two": 2, "three": 3}
let hm_iter = some_hash_map.iter();
hm_iter.next() // ("one", 1)
```

Alternatively, Rust's hash map implements two trait methods:
- `.values()` returns an iterator of just the hash map values.
- `.keys()` returns an iterator of just the hash map keys.

This exercise wants you to recreate two existings functions using iterator methods rather than simply iterating over the values.  Some useful functions to use:
- `.fold(<initial value>, <closure>)`, which we saw last exercise.
- `.filter(<closure>)` returns a new iterator consisting  of only items that returned `true` when passed to the argument closure.
	- As an example: `some_iterator.filter(|x| *x == &some_value)`
	- Note that the closure takes the *reference* of each each item, hence we must dereference with `*` if we want to compare the item value to some other value.
- `.count()` returns the `usize` number of items in the iterator.

See the Rust documentation for more methods.  There are a lot, including more familiar methods from other languages like `.zip()` and `.enumerate()`.
## Smart Pointers
A general **pointer** is a variable that holds an address to memory; these are the *references* we encountered in previous exercises.  A **smart pointer** is essentially a pointer with additional automatic memory management abilities.  

In Rust, smart pointers refer to data structures that can own (in context of Rust's owernship) and manipulate memory; in fact, `Vec<T>` and `String` types are considered types of smart pointers.  There is no *primitive* smart pointer; Rust's standard library provides some smart pointers designed for specific purposes, or you can write one yourself (though that's not covered in this course).

In short: a smart pointer is a wrapper around a value on the heap.

The following exercises introduces the most commonly used smart pointers from the Standard Library.
### 1. Box
 A **Box** is a smart pointer that holds a value on the heap.  The type annotation is `Box<T>` where `T` is the type of value we want on the heap.  The name is apt: it really is just a "box" that holds a value.

The straightforward way to place some value on the heap is with `Box::new(<some value>)`.  For example:
```Rust
let a = Box::new(13);
```

A box is useful when you are defining a function or user-defined structure that requires a type of variable size.  

### 2. Reference Counting
Previously I said that there can only be one owner of a value in Rust.  However, there are cases when we really do need more than one owner; a common case is in creating graph data structures.  Rust provides for this with the **reference counting** smart pointer type `Rc<T>`.

Basically, an `rc<T>` smart pointer holds a value of type `T` on the heap, and this value can have *multiple* owners.  The value won't be freed until every owner is invalid i.e. no more references are made to this value.

To place some value on the heap, use: `Rc::new(<some value>)`.  In this exercise, we are placing a `Sun` structure on the heap.  Note then that:
```Rust
let sun = Rc::new(Sun {});
```
has one owner to begin (the variable `sun`).

To make another variable also an owner, use `Rc::clone(<reference to rc>)`:
```Rust
let another_variable = Rc::clone(&already_existing_owner);
```

To explicitly make a variable drop its ownership of a value, use `drop(<variable>)`.   This function actually comes from the `std::mem` library.  If the variable was the last or sole owner of the value (whether of the `Rc` type or not), the value is disposed.

To see how many owners a `Rc` value has, use `Rc::strong_count(<reference to rc>)`.

### 3. Atomically Referenced Counted
We'll cover threads in more depth later.  For now, if we want multiple threads to access a value simultaneously, we can wrap that value in an **atomically referenced counted** smart pointer `Arc<T>`.

```Rust
let v = 12;
let arc_value = Arc::new(v);
let arc_copy = Arc::clone(&arc_value);
```

### 4. Clone-On-Write
The **Clone-On-Write** smart pointer `Cow` automatically handles whether a value needs to be cloned or not.  Specifically:
- It provides its value as an immutable borrow if no mutation occurs.
- It clones its value if mutation (or some other ownership manipulation) occurs.

## Threads
Threading is provided by Rust through its standard library via `std::thread`.  The standard library that deals with time -`std::time`- is also useful here.

Thread