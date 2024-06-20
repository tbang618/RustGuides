# Types and Modules
In this part, we explore:
- Primitive Data Types
- Data Types from the Standard Library
- Modules
## Primitive Types
Rust has a few **primitive types**; these are *built-in* to Rust.  Later we'll learn types that are provided by the **Standard Library**.

We'll see some values have functions that operate on themselves; these are **methods**, similar to those in Java or C++.
### 1. Booleans
Formally introducing **booleans** (`bool`):
- `true`
- `false`

As said before, these do not type cast into integers, as in C++.
### 2. Characters
**Characters** (`char`), like in C, are enclosed in single-quotation marks.  Some methods are introduced in this exercise:

- `.is_alphabetic()` returns `true` if the character is a alphabet character.
- `.is_numeric()` returns `true` if the character is a number character.

In this exercise you can experiment with different characters.
### 3. Arrays
**Arrays** are one of Rust's primitive **compound types**: objects that hold other elements.  Arrays collect elements of the *same type*.  Arrays are of a *fixed size*.  To get the length of an array, use the method `.len()`.   Arrays (and all Rust container types) are *zero-indexed*.

The **type annotation** for arrays is: `[<element-type>; <number-of-elements>]`

Arrays may be declared explicitly by listing each element.  Like C, there is a shorthand to declare an array of all the same elements, repeated for `n` times: `[<expr>; <number-of-elements>]`

For example, to declare an array of a hundred `13`'s:
```Rust
let a: [u32; 100] = [13; 100] 
```

An empty array may be declared with `[<expr>; 0]` but note that `expr` will still be evaluated, so beware of side-effects (strict evaluation).

For a mutable array, you can change the value of an element like in C, Python, etc:
```Rust
let mut x = [0, 1, 2, 3];
x[0] = 100;
```
### 4. Slices
Here we first come across Rust's unique concepts of **ownership** and **borrowing**.  We'll cover them in greater detail later on.

A **slice type** refers to a contiguous sequence of objects, all of the same type.  The slice type consists of a pointer to the first object and the size of the object's type.  The difference from an array is that the size of a slice type is determined *at runtime* and are typically *unowned*, meaning they are references to parts of compound types.

The *type annotation* for a slice is `&[T]`.  You can think of it as an "un-owned array", for now.

In this exercise, a slice is used to refer to a sub-array of an array.  For example:
```Rust
let a = [1, 2, 3, 4, 5];

let nice_slice = &a[1 .. 4]; 

// nice_size == [2, 3, 4]
```

The `&` symbol is Rust's **reference operator**, which gets or "borrows" the value of a variable.  This is the same as C's reference operator (`&`), which gets the memory address of a variable.

Here `&a[1 .. 5]` means "go to the location" of `a` and get the "first up-to fifth" objects of the sequence to which `a` "points": so this slice contains four objects.
### 5. Tuples
A **tuple** is a Rust's other primitive **compound type** that consists of elements that may be of *different* types.  Like arrays, tuples are of fixed size.

Tuple's are *ordered*.  The type-annotation of a tuple needs to order the types of the elements as they appear.  For example:
``` Rust
let my_tuple: (u8, &str, f16) = (8, "Hello World", -13.2);
```

You can access elements in a tuple by **destructuring**:
```Rust
let my_tuple = ("Furry McFurson", 3.5);
let (name, age) = my_tuple;

// name == "Furry McFurson"
// age == 3.5
```

### 6. Indexing
Elements in an array or tuple may also be accessed by indexing.  Indexing for a tuple is different from that of indexing for an array.

- For an array: `someArray[<index>]`.  
- For a tuple: `someTuple.<index>`.  

For example:
```Rust
let numbers = (1, 2, 3);
let second = numbers.1; // second = 2
```

## Vectors
A **vector** can be thought of as a *dynamically sizable* array.  As such, Rust's vectors are stored on the *heap* while arrays are stored on the *stack*.  Vectors are not a primitive data type, but instead provided by Rust's Standard Library.
### 1. Instantiating Vectors
Declare a vector with `Vec::new()` and add elements with the `push()` method.  Make sure that the vector is *mutable* in order to push elements.  As an example:
```Rust
let mut x: Vec<i32> = Vec::new();
x.push(32);
```

The easier way to instantiate a vector is with the `vec![]` macro:
``` Rust
let v = vec![10, 20, 30, 40];
```

### 2. Iterating over Vectors
Here we use something called **iterators** that let's us operate on all the elements of a vector; either one-by-one or all-at-once.  Iterators are covered in depth much later (part 4).
#### Mutable Iterator 
The method `.iter_mut()` returns an *iterator* that "produces" *references* to each vector element, one-by-one.   In the for-loop of the exercise, `element` is a *reference* to a vector element and each iteration of the loop refers to the *next* element in the vector.  

Remember a reference is a "pointer" that holds the "memory address" of a value.  To get the actual value, we use the **dereference operator**: `*<reference>`.  To get the actual value of `element` and multiply it by two, we use `*element * 2`.  To store that now as the value of the element, we use:
```Rust
*element = *element * 2;
```

Note that we are *mutating* the original vector `v`.
#### Mapping
Here, `.iter()` returns an iterator but `.map()` takes a reference `element` and maps it to a new value in the iterator.

As the hint notes, with mapping you don't need explicitly set the new value to `element`, you just need to give the new value (as an expression or return value):
```Rust
*element * 2
```

You don't actually need to dereference `element`; details about mapping are covered in the iterators section.

Note the iterator now needs to be reformed into a vector with the `.collect()` method.  Here, this returns a *new* vector.
## Move Semantics
These exercises dive into the concepts of **ownership** and **borrowing**; together they play important roles in **move semantics**, part of Rust's unique memory management schema (in lieu of a traditional garbage collector).

Every *value* in Rust has an **owner**: this is the *variable* in a certain *scope* that is responsible for *freeing* that value from memory.  That is, when the owner goes out of scope (the scope it inhabits ends), the value it owns is freed from memory.

There is only one owner of a value at a time.  Ownership of a value may be *moved* to a new variable; afterwards, the old variable (the previous owner) becomes invalid.  
```Rust
let a = 10;
let b = a;
// `b` now owns `10` and `a` is an invalid variable (holds nothing)
```

### 1. Moving Variables
Here is a first-pass understanding of what is happening in this exercise:
1. The vector in variable `vec0` is *moved* to argument `vec` in the scope of `fill_vec()`.   
	- Now `vec0` is invalid.  Try adding `println!("{}", vec0[0])` at the end of `main()` to see an error message.
2. The vector is *moved* from the argument `vec` to newly declared variable `vec` in the body.
	- The argument `vec` is not invalid.
	- Since we want to change the vector, the new `vec` in the body needs to be declared as *mutable* with `mut`.
3. The vector is mutated: a new value is pushed onto the vector.
4. `vec` is *returned* from the function (i.e. placed to the outside scope) and *moved* to `vec1`.  
	- At this point, only `vec1` is a valid variable.

### 2. Borrowing
In this exercise, we want to keep `vec0` valid in the scope of `main()` after "moving" its value into `fill_vec()`.

Similar to *pass-by-reference* in C/C++, a different variable or scope can **borrow** a value.  The *owner* essentially allows another variable, the *borrower*, to access its owned value, but the borrower *cannot free* the value; when the borrower goes out of scope, the value remains valid with the owner.  A borrowed value, or **reference**, is prefixed by the symbol `&`.   We already saw this when introducing slice types.

As for the exercise: first, the function definition of `fill_vec()` needs to be modified to accept a borrowed vector instead:
```Rust
fn fill_vec(vec: &Vec<i32>) -> Vec<i32>
```

The function call of `fill_vec()` then needs to pass `vec0` as a borrowed value:
```Rust
fill_vec(&vec0)
```

In the function body, we need to *copy* the vector held in the argument `vec` to the newly-decleared `vec`:
```Rust
let vec = vec.clone();
```

In summary:
1. The *reference* to `vec0` is moved into `fill_vec()` as `vec`, which is now essentially a vector *pointer*, borrowing `vec0`'s value.
2. The method `.clone()` is used to copy the vector pointed-to by the argument `vec` into the newly declared vector variable `vec` in the body.
	- We can think of `let vec` as declaring a new owner.
3. `vec` is moved out from the scope of `fill_vec()`.
4. The ownership of `vec`'s value is moved to `vec1`.
	- At this point, only `vec0` and `vec1` are valid variables: two different owners.

### 3.  Moving values into Mutable Variables
When we pass a variable into a function, we are really moving the ownership from the passed variable to the *parameter* variable.  In the previous two exercises, this was simply `vec`.  Hopefully that's not too confusing.

In the first exercise of this section, we saw that we could move an immutable value to a mutable variable:
```Rust 
let mut vec = vec;
```

So we can do the same when it comes to variable passing, by declaring the parameter as mutable:
```Rust
fn fill_vec(mut vec: Vec<i32>) -> Vec<i32>
```

This can be thought of as a shorthand to the first exercise.
### 4. Moving values out of Functions
This exercise illustrates that returning a value from a function is really moving that value out from the function's scope.

First, change the function call to take no arguments.  Then *create* a new vector in the scope of the function.  When the new vector is returned, it is being moved out from the function's scope.

### 5. Mutable Borrows
A *borrow* (or reference), as we saw previously, lets a variable (the *borrower*) access the value of another variable (the *owner*).  

A **mutable borrow** lets the borrower access the owner's value and *modify* that value.  I think of it as "asking" the owner to change the value.  Note the owner must be declared as a mutable variable with `mut`.

There can only be *one mutable borrow* of an owned value at a time.

A borrower ends it borrowing the last time it is used in its scope.

This exercise asks to rearrange the *lifetimes* of variables so there is only one mutable borrow of `x` at a time.  

### 5. Function Signatures
This exercise brings together the lessons of this section, but highlights that Rust, unlike C/C++, strictly differentiates between a value and a reference to that value.

As an aside: functions in Rust are technically pass-by-value, but the way ownership works means that borrowing (passing the address of a variable) operates much like pass-by-reference in C/C++.

In this exercise, we want `get_char()` to take a reference to `data`, so that `data` remains valid (in scope) to be moved into `string_uppercase()`.

To solve this exercise: first, the function signature and call of `get_char()` needs to borrow `data`: this is done by adding `&` at the appropriate places.  Similarly, the `&`'s need to be removed from the signature and call of `string_uppercase()`.

## Structs
**Structs** are Rust's **user-defined data types**.  Similar to tuples, they group related elements called **fields** together.  Unlike tuples, the elements of a struct are *named types*.

#### 1. Declaring Structs
Structs can be defined like in C/C++, where we list the name of the fields and give their types:
``` Rust
struct ColorClassicStruct {
    red: u32,
    green: u32,
    blue: u32,
}
 ```
Note we can end the last field with a comma.

Structs can be defined like tuples, where fields do not have names:
```Rust
struct ColorTupleStruct(u32, u32, u32);
```

There is also something called a **Unit-Like Struct**, which handles like an *empty tuple*.  It is useful for implementing *traits*, which we'll learn later.
```Rust
struct UnitLikeStruct;
```

Structs can be declared like any other data type:
```Rust
let green = ColorClassicStruct{red:0, green:255, blue:0};
let green = ColorTupleStruct(0,255,0);
let unit_like_struct = UnitLikeStruct;
```


#### 2. Copying Structs
We can create a new struct by copying the fields from an existing struct.
```Rust
let your_order = Order {
	name: "Hacker in Rust".to_string(),
	count: 1,
	year: order_template.year,
	..order_template
};
```
Note that a new declaration of a struct cannot end with a comma in its last field.

The shorthand to fill the rest of the fields with copies from an existing struct is: 
```Rust
..<other_struct>
```

#### 3. Struct Methods
Besides fields, structs can contain functions.  These are also called **methods**.  The functions of a structure need to be defined in a separate block from the structure definition called the **implementation block**, which begins with keyword `impl`. 

Implemented functions that need to access a structure's fields need to define `&self` as the function parameter (same as Python).  A field or method can then be accessed with `self.some_field`.  For example:
```Rust
fn get_fees(&self, cents_per_gram: u32) -> u32 {
	return self.weight_in_grams * cents_per_gram;
}
```

## Enums
**Enums** are another user-defined type, similar to enums in C/C++ or algebraic types in some functional languages.  An enum represents a *range* of *possible* values (enumerations).  The classic example is an enum representing the day of the week.
### 1. Defining an Enum
Defined as the type `enum` that takes the different possible values:
```Rust
enum Day {Monday, Tuesday, Wednesday, Thursday, Friday,}
```

### 2. Different Types
Unlike in C/C++, the possible values in an enum may take different types, called **variants**.  In the exercise, we create a `Message` enum, where messages may be different data depending on what is intended:
```Rust
enum Message {
    Move {x:u32, y:u32}, // anonymous struct
    Echo (String), // of type String
    ChangeColor (u32, u32, u32), // anonymous tuple
    Quit, // "type-less"
}
```

Note that we must wrap the type in parentheses unless it is of a compound type (struct or tuple in the previous example).

### 3. Using Enums
This exercise stresses an enum value is really a stand-in for another data structure.

Here we first introduce a **match-case** (or **switch-case**) in Rust.  Each branch of a match-case is denoted with the symbol `=>`.

```Rust
enum Message {
    Move (Point),
    Echo (String),
    ChangeColor (u8, u8, u8),
    Quit,
}

// ...

match message {
	Message::Move (pt) => self.move_position(pt),
	Message::Echo (msg) => self.echo(msg),
	Message::ChangeColor (r,g,b) => self.change_color( (r,g,b) ),
	Message::Quit => self.quit(),
}
```

Note the hint in the exercise: we match the *destructured* tuple `(r, g, b)`.  In the arm, when we want to pass `change_color()` a tuple, we have to recreate the tuple:
```Rust
change_color( (r, g, b) ),
```
## Strings
**Strings** are types provided by the Standard Library.  There are actually two "string"-types:
- `String` which is like strings in C/C++: a wrapper or object around the character array.
- `&str` which is called the **string slice**.  It is also called the **string literal**, to match other languages, but it is itself a special subset of the *slice type*, which we met in the primitive data types section.

### 1. Converting between Types
A simple string slice, or string literal, is given in double quotations: `"blue"`.

There are a few ways to convert a string slice `&str` to a `String` object.  For a string literal `"blue"`:
- `"blue".to_string()` 
- `String::From("blue")`
- `"blue".to_owned()`
- `"blue".into()`

The first two are preferred; the last two give the result we want here, but do something differently, which we won't get into right now.

The hint mentions that string slices have *static lifetimes*, meaning they are "hard-wired" into the code (static variables in C/C++, as part of the code section) and thus *immutable*.

### 2. String References
A reference to a string is a string slice.  In other words, when we borrow a value from a `String` variable, we are borrowing a value of type `&str`.

In more other words, `&String` and `&str` are the "same".  You can test this by changing the function signature for a `is_a_color_word()` to accept `attempt` as of type either `&String` or `&str`.  I say "same" because what's actually happening is something called **reference conversion** or **implicit dereference coercion**, which is beyond this course.
### 3. Methods on Strings
You can *chain* methods, like in Java or Javascript.

There are many methods to manipulate strings and string slices.  Here are a few for this exercise that works on string slices:

- `.trim()`: returns a string slice with leading and  trailing white-spaces removed.
- `.push_str(<str>)`: appends a string slice to a string object (returns nothing).
- `.replace(<pattern>, <replacement>)`: in a given string slice, take a (sub-)string slice to match and replace any instance with the replacement (sub-)string slice.  It returns a `String`.
- `.repeat(<n : u32>)` returns a string of the string slice repeated `n` times.

To concatenate strings, there are several options.  Using `push_str()` may be tedious, because it is not an expression (returns nothing, so can't be chained), but mutates the string it operates on in place, so that you would need to explicitly return the new string.  The shorthand expression:
``` Rust
<String> + <string slice>
```
yields a `String` with the `<string slice>` appended.

There is also a macro called `format!` that allows for combining strings together.

In the exercise, remember to convert the results of these methods to the `String` type (probably with `to_string()`).

### 4. Mini-Quiz
This exercise simply tests if you can recognize the difference between values of type `String` and `&str`.

## Modules
**Modules** are Rust's way to package code together as a unit.  They are very much like namespaces in C++, modules in Scheme/Racket, or packages in Python.
### 1. Public Attribute
A **module** is declared with the keyword `mod` followed by the name and curly-braces that demarcate the module.  By default, everything in a module is *private*: inaccessible to anything outside the scope of the module.  To make something *public*, use the `pub` keyword.

### 2. Using Namespace
You can nest modules within each other.

You can import a module or part of a module with a new particular name, similar to importing in Python, using `as`.  This may also be done in a module, and the new name made publicly available.  That is, in a module you can export a part under a new name (remembering to include `pub`).  For example:
```Rust
    pub use self::fruits::PEAR as fruit;
    pub use self::veggies::CUCUMBER as veggie;
```

Rather than `self`, another option is `super`, which means to use the module above the current level (similar to `..` in navigating file-systems).
### 3. Standard Library
The standard library itself can be imported as a module, with particular modules nested within.  You can import specific functions or values.  This exercise introduces two of them:
- `std::time::SystemTime` : a function.
- `std::time::UNIX_EPOCH` : a value.

Declare their use at the beginning of your source, as in most languages.

## Hash Maps
A **hash map** is a sort of *associative array* and a very useful data structure.  In Python they are called dictionaries.  In JavaScript, objects serve as a sort of hash map.

### 1. Creating and Using Hash Maps
Hash maps are provided by the Standard Library in `std::collections::HashMap`.

To create a new hash map, use the create trait: `HashMap::new()`

Insert new *associates*, consisting of a keyword-value pair, with method `.insert(<keyword>, <value>)`.  If the keyword already exists, then `.insert()` updates the value.

Note that the keyword needs to be an "own-able" type: so a keyword may be a `String` type but not `&str`.

### 2. Hash Map Methods
This exercise shows two methods useful when using hash maps.

The method `.entry(<keyword>)` gets the value associated with the keyword in a hash map.   If the keyword-value pair does not exist, the method returns `None`, an `option` value we'll learn later.

You can *chain* the method `.or_insert(<value>)` after `.entry()`: if `.entry()` returns `None`, a new keyword-value pair is inserted into the hash map, with the given value.

For example, in the exercise:
```Rust
basket.entry(fruit).or_insert(1);
```

### 3. Mini-Quiz
The exercise uses knowledge from the previous exercises.

As a hint: you can break up chained methods onto individual lines, or arguments into individual lines; like in C/C++, white-space is ignored/compressed by the Rust compiler.

Remember the value is the struct `Team`.

Here's a working example:
```Rust
        let team1 = scores.entry(team_1_name)
            .or_insert(Team {goals_scored:0, goals_conceded:0});
        team1.goals_scored += team_1_score;
        team1.goals_conceded += team_2_score;

        let team2 = scores.entry(team_2_name)
            .or_insert(Team {goals_scored:0, goals_conceded:0});
        team2.goals_scored += team_2_score;
        team2.goals_conceded += team_1_score;
```

## Quiz 2
When using a module in the same source-file, use the `super::` keyword to denote the file as the top-level module.

Because a match-case is an expression, it can be used as the value in variable initialization.

The method `.to_uppercase()` returns a `String`, but `.trim()` returns a `&str`.  Note the variable `string` in the loop is of type `&str`.

The method `<string slice>.repeat(<size:u32>)` returns a `String` of `<string slice>` repeated `<size>` times. 

Here's part of the working solution, as an example:
```Rust
mod my_module {
    use super::Command;

    // TODO: Complete the function signature!
    pub fn transformer(input: Vec<(String, Command)>) -> Vec<String> {
        // TODO: Complete the output declaration!
        let mut output: Vec<String> = vec![];
        for (string, command) in input.iter() {
            // TODO: Complete the function body. You can do it!
            let nstr = match command {
                Command::Uppercase => string.to_uppercase(),
                Command::Trim => string.trim().into(),
                Command::Append(size) => {
                    string.to_string() + &"bar".repeat(*size)},
            };
            output.push(nstr);
        }
        output
    }
}
```

An interesting note: using `.into()` instead of `.to_string()` or `.to_owned()` will result in an error; earlier we mentioned that `.into()` doesn't exactly convert a string slice to a `String`.

Also note that we need to use the reference operator to convert `"bar".repeat(*size)` into a string slice (as required by string concatenation).

Also note that `*size` indicates that `size` is actually a pointer we need to dereference.