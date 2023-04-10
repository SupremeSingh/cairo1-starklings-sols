## Glossary of Additional Concepts - 

### 1. Enums

Where structs give you a way of grouping together related fields and data, like a `Rectangle` with its `width` and `height`, enums give you a way of saying a value is one of a possible set of values. This may be useful when, for instance, a variable in our program is only supposed to have one of a set of values. Eg ... 

```
enum  Colors { 
	Red: (), 
	Green: (), 
	Blue: () 
}
```

Moreover, you can reference one of these values, with respect to the enum, as - 

```
fn  get_favorite_color() -> Colors {  
	Colors::Green(())  
}
```

### 2. Pattern Matching 

Pattern matching is a feature from the Rust programming language that allows developers to specify a set of patterns to check against the structure of a value. Based on which pattern matches, a corresponding block of code is executed. 

A simple example of matching the value of a variable would be - 

```
fn  main() {  
	let  x = 5;  
	match x {  
		1 => println!("x is 1"),  
		2 => println!("x is 2"),  
		3 => println!("x is 3"),  
		4 => println!("x is 4"),  
		5 => println!("x is 5"),  
		_ => println!("x is something else"),  
	}  
}
```

So far, this looks quite like a switch statement from Java does it not ? But it is actually way more powerful. Here are some examples of pattern matching with more complex data structures - 

Tuples: 

```
let  pair = (5, -5);  
  
match pair {  
	(0, 0) => println!("The pair is (0, 0)"),  
	(x, 0) => println!("The pair is ({}, 0)", x),  
	(0, y) => println!("The pair is (0, {})", y),  
	(x, y) => println!("The pair is ({}, {})", x, y),  
}
```

Enum:

```
enum  OptionalInt {  
	Value(i32),  
	Missing,  
}  
  
fn  main() {  
	let  optional = OptionalInt::Value(5);  
	  
	match optional {  
		OptionalInt::Value(x) => println!("Got an int: {}", x),  
		OptionalInt::Missing => println!("No value"),  
	}  
}
```

Structs:

```
let  point = Point { x: 5, y: 10 };  
  
match point {  
	Point { x: 0, y: 0 } => println!("The point is at the origin"),  
	Point { x, y } => println!("The point is at ({}, {})", x, y),  
}
```

We also have the concept of pattern bindings. Basically, the operator `@` lets us create a variable that holds a value at the same time as we’re testing that value for a pattern match. 

For instance - 

```
match msg {
    Message::Hello {
        id: id_variable @ 3..=7,
	} => println!("Found an id in range: {}", id_variable),
}
```

Here, we are binding the values of `id` to the `id_variable` so we can use it for the code to come. 

### 3. Ownership Model 

Rust uses a stack based architecture and manages its memory using this concept of "ownership". I recommend watching [this](https://www.youtube.com/watch?v=8M0QfLUDaaA) video to learn about it. Ownership is the reason Rust is able to work without a background garbage collector. 

As per the Rust textbook, there are 3 rules of ownership in this language - 

- Every value has a variable which is called its owner. 
- A value can only have 1 owner at a time. 
- When a value goes out of scope, the value will be dropped.

First, let's see how simple numbers are stored in this architecture. Numbers (Integers / Felts more precisely) are stored in the stack. So, they can be "copied" when being assigned as follows ...
```
let a = 3;
let b = a; // 5 is copied to b, still retained with a
```
Do remember, now `a` and `b` have ownership of different memory slots with the same value. 

Further, let's look at how strings are stored. Since a string may not have a fixed length, Rust stores it in a heap and returns the pointer to this heap to the stack. Unlike the stack, values in the heap cannot be copied around.

This means, we cannot assign 2 variables to the same pointer value - since in this case they will point to the same place in the heap and break rule 2.  So how does Rust process the following ...

```
let x = String:from("hello");
let a = x; // x no longer has ownership of "hello"
```
Simple (kinda) ... it "moves" the ownership of "hello"'s pointer from x to a. Now, x does not own any value in the program. And if you do want x to retain it's original value - you will have to clone it ...

```
let x = String:from("hello");
let a = x.clone();
```

Not both `a` and `x` have ownership of memory locations containing the word "hello".

Moreover, in Cairo, when an argument is passed to a function and it's not explicitly returned, you can't use the original variable anymore. You can, however, use a different one that it returns to. We call this "moving" a variable. Case in point - this works ...

```
fn main() {
    let mut a = ArrayTrait::new();
    pass_by_snapshot(@a); // Does not modify a 
    pass_by_ref(ref a); // Can be used to modify - is a mutable reference
    let mut b = pass_by_value(a); // would have failed if this was a and not b
    pass_by_ref(ref b);
}

fn pass_by_value(mut arr: Array<felt252>) -> Array<felt252> {
    arr
}

fn pass_by_ref(ref arr: Array<felt252>) {}

fn pass_by_snapshot(x: @Array<felt252>) {}
```

Variables that are moved into a function (or block scope) and aren't explicitly returned get "dropped" at the end of that function. This is also what happens here. There's a few ways to fix this, try them all if you want:
- Make another, separate version of the data that's in `arr0` and pass that to `fill_arr` instead.
- Make `fill_arr` *mutably* borrow a reference to its argument (which will need to be mutable) with the `ref` keyword , modify it directly, then not return anything. Then you can get rid of `arr1` entirely -- note that this will change what gets printed by the first `print`
- Make `fill_arr` borrow an immutable view of its argument instead of taking ownership by using the snapshot operator `@`,and then copy the data within the function in order to return an owned `Array<felt>`. This requires an explicit clone of the array and should generally be avoided in Cairo, as the memory is write-once and cloning can be expensive.

Finally, the third rule is useful when thinking about functions - since functions in Rust have their own [scope](https://en.wikipedia.org/wiki/Scope_%28computer_science%29). Passing values to a function follows the same ownership rules, meaning they can only have one owner at a time, and free up memory once out of scope.

### 4. References and Borrowing

Ok wait a minute ... 

This procedure of moving ownerships can get quite messy in complicated programs - so Rust introduces this new idea of "borrowing". 

References allow coders to use a function that has a reference to an object as a parameter instead of taking ownership of the value. Now, instead of passing objects by value, objects can be passed by their reference.

Let's understand it through this example - 

```
#[derive(Drop)]  
struct  Point {  
	x: felt,  
	y: felt  
}  
  
fn  main() {  
	let  mut point_value = Point { x: 10, y: 20 };  
	let  mut point_ref = Point { x: 10, y: 20 };  
	
	pass_by_value(point_value);  
	pass_by_ref(ref point_ref);  
	
	let  val_x = point_value.x; // This instruction generates a  "Variable was previously moved" error  
	let  ref_x = point_ref.x; // However, this one will work because the ownership was returned to us.  
}  
  
fn  pass_by_value(mut val: Point) {}  
  
fn  pass_by_ref(ref val: Point) {}RT
```

The compiler statically guarantees (via its borrow checker) that references **always** point to valid objects. That is, while references to an object exist, the object cannot be destroyed.

### 5. Dictionaries

Cairo 1 also supports dictionaries in the core library. It is currently possible to create dictionaries that map felts to  `u128`,  `felt`,  `u8`,  `u64`  and  `Nullable::<T>`. Because short strings are actually felts, it is also possible to create string keys in a dictionary.

Dictionaries can be created as - 

```
use dict::DictFeltToTrait;  
  
fn  create_dict() -> DictFeltTo::<felt> {  
	let  mut d = DictFeltToTrait::new();  
	d.insert('First', 1); // Write.  
	d.insert('Second', 2);  
	d  
}
```

And the values in a dictionary can be consumed as - 

```
let  a = dict.get('First');  
let  b = dict.get('Second');  
dict.squash();
```

### 6. Assertions and Panic 

This set of concepts is particularly useful when a coder is debugging their program.  

The Panic function is used to raise a message which indicates that the program encountered an error. It takes an `Array` from the core library filled with felts as a parameter. You can use felt string as error messages, as long as your string can fit into a felt. A simple example is ... 

```
let  mut data = ArrayTrait::new();  
data.append('out of bounds');  
panic(data);
```

Assert is a function that verifies a boolean condition and panics if the condition is not satisfied. The first parameter is the boolean check to perform, and the second one is the error code to panic with if the assertion fails. For instance - 

```
assert(1_u128 + 2_u128 == 3_u128, 1);
```
### 7. Options 

Remember how Rust is a strongly typed language ... that is ... all variables must have a data type associated with them. 

[Options](https://doc.rust-lang.org/std/option/enum.Option.html "Option") represents an optional value in a Rust program. Everyoption is either a [Some](https://doc.rust-lang.org/std/option/enum.Option.html#variant.Some "Some") and contains a value, or a [None](https://doc.rust-lang.org/std/option/enum.Option.html#variant.None "None"), and does not. Option types are commonly found in Rust code. 

Further, Rust’s pointer types must always point to a valid location; there are no “null” references. Instead, Rust has optional pointers, like the optional owned box `Option<Box<i32>>`.

### 8. Traits and Implementations

Traits are basically the interfaces of Rust, and by extension those of Cairo 1. An abstract method in a trait is able to access other methods within that trait. A trait can be created in Cairo 1 as - 

```
trait  MyTrait {  
	fn  foo() -> felt;  
}
```

If we wish to create a working model of this trait, we will have to implement it. This will be (rather un-creatively) called a trait implementation. 

The `impl` keyword is primarily used to define implementations on traits. Trait implementations are used to implement a specific behavior for a trait, meaning that all the functions from the trait must be implemented.

```
impl  MyImpl of MyTrait {  
	fn  foo() -> felt {  
		1  
	}  
}  
  
fn  main() -> felt {  
	MyTrait::foo() // returns 1  
}
```

### 9. Macros

Macros are a concept in Computer Science which allow a program to make language transformation using a part of the language itself. At first glance, Macros may look like functions, but the big difference is they are swapped in place (pre-processes) before the program is compiled. 

This reduces work for the compiler to have to process the functions as separate units during compile time (since functions are compiled separately and executed at run-time). Hence, programs which use Macros have significantly faster run time.  For instance ... 

```
#[derive(Debug)]  
struct Point { x: i32, y: i32, }
```

is a lot easier than saying ... 

```
struct Point { x: i32, y: i32, } 

use std::fmt; 

impl fmt::Debug  for Point 
{ 
	fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result 
	{ 
	write!(f, "Point {{ x: {}, y: {} }}", self.x, self.y) 
	} 		
}
```

**Procedural macros** are a way for you to extend the Rust compiler's functionalities and provide plugins for the language. Each procedural macros is included in its own [crate](https://rustwiki.org/en/book/ch07-01-packages-and-crates.html). 

In Cairo 1, we are concerned with procedural macros called "custom derive macro". I highly recommend reading up on them [here](https://web.mit.edu/rust-lang_v1.25/arch/amd64_ubuntu1404/share/doc/rust/html/book/first-edition/procedural-macros.html). For now, just know that any time you create a custom data structure, do derive the `copy` and `drop` traits for it to ensure smooth memory management. 

### 10. Generics 

Cairo 1.0 **will** offer us another amazing feature from Rust - `generics`.  Generics (i.e general data type) is a flexible way to write code that can work with multiple types rather than being tied to a specific type, avoiding code redundancy. 

You can define a generic type by adding a placeholder type within angle brackets `<>` after the name of the type or function. You can read more about this general concept [here](https://web.mit.edu/rust-lang_v1.25/arch/amd64_ubuntu1404/share/doc/rust/html/book/first-edition/generics.html). 

When Cairo 1 has this concept implemented - it will look something like ... 

```
fn  swap<T>(x: T, y: T) -> (T, T) {  
	(y, x)  
}  
  
fn  main() {  
	let (two,one) = swap::<felt>(1, 2);  
	let (two_128,one_128) = swap::<u128>(1_u128, 2_u128);  
}
```









 

