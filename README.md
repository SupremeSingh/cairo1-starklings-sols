#  Starklings - Cairo 1
  
Here, we will be solving the much awaited version of Starklings for the newest release of Cairo - Cairo 1. You can find the original repository containing these exercises (and instructions) over [here](https://github.com/shramee/starklings-cairo1). 

In each of the following sections, titled to match the sections in Starklings - I will quickly summarize the most important learning and reference other resources you can read up to learn more about them. Please go over the problem sets in exactly the order given below - 
  
## Pre Req - Setting up Cairo 1 

Cairo 1.0 is a rust-inspired fully typed language, which allows users to write sound and complete provable programs for general computation. Please follow the instructions highlighted [here](https://github.com/starkware-libs/cairo#getting-started) to set up [Rust](https://www.rust-lang.org/) and the Cairo [Language Server](https://learn.microsoft.com/en-us/visualstudio/extensibility/language-server-protocol?view=vs-2022) on your local machine. 

What's more - I do highly recommend you go over the Cairo core [library](https://github.com/starkware-libs/cairo/tree/main/corelib) for useful functions. You should also review a few concepts of the Rust programming language to get used to this style of coding. [Here](https://web.mit.edu/rust-lang_v1.25/arch/amd64_ubuntu1404/share/doc/rust/html/book/first-edition/README.html) is my favorite free Rust course. 

Ready ? Let's go ... 

## Set 1 - Introduction  
  
This set introduces us to the general syntax of declaring a function in Cairo 1 - 

```
fn function-name(input-name: data-type) -> return-type {
	return variable-name;
}
``` 
As well as the procedure for creating a new variable of any data type - 

```
let variable-name: data-type = initial-value
```

Remember here, that a new variable must always be provided an initial value for its declaration to be considered valid by the compiler. Finally, this set introduces an interesting trick - that if you skip out the semicolon at the end of a statement, a Cairo 1 function simply returns the value of that statement. For example - 

```
fn add(a: felt, b: felt) -> felt {
	a + b
}
```

Simply terminates the function and returns the value of `a + b`.

## Set 2 - Variables 

Variables in Cairo 1 use the same immutable memory model as Cairo 0. However, Cairo 1 introduces an additional feature called variable mutability for the user's convenience. This means, if a variable is re-assigned in a program, the compiler simply provides a new memory slot to that variable. This way, the same variable can take different values over the course of the program. 

For instance - 

```
let mut x : felt = 3;
x = 5;
```
Moreover, this set introduces a great new debugging tool which allows users to log values to the console during runtime - 

```
use debug::PrintTrait;
x.print();
('x is equal to ten!').print();
```

In addition to reassigning a variable to a different value, a variable can be reassigned to a different data type as well. This is made possible by the concept  of [shadowing](https://medium.com/nethermind-eth/a-first-look-at-cairo-1-0-a-safer-stronger-simpler-provable-programming-language-892ce4c07b38#4fe8).  For instance, 

```
let number: u8 = 1_u8; 
let number: felt = 3; 
```

Further, if you wish to pass a pointer to a function, and do not want to modify it's ownership - you can always just use a reference to it as follows - 

```
fn is_low_char(ref char: felt252) -> bool {
    if char >= 'a' {
        if char <= 'z' {
            return true;
        }
	}
}
```

Even though Cairo 1 still has the same primitive data type as Cairo 0 - the felt - it allows for several new data types to be declared (such as the 8 bit unsigned integer above). Lastly, Cairo 1 allows users to declare constant variables as well ...  

```
const NUMBER: felt = 3;
```
## Set 3 - Functions

This set mostly goes over the basic declaration of functions in Cairo 1. However, it does also point out some interesting features such as - 

- New data types implemented in the core library, including u256 and others. To declare a number as a u256 instead of felt, simply do something like ... 

	```let x: u256 = 1_u256```
- Unsigned integers can be converted to felts at runtime by using the expression `use traits::Into;` and the simple method call `variable-name.into()`

- A close example to the ternary operator in Cairo 1 would be ... 
	```
	fn is_even(num: u32) -> bool {
		num % 2_u32 == 0_u32
	}
	```

## Set 4 - Arrays 

Cairo 1 allows for dynamically sized, immutable and append-only arrays in it's core library.  The values in these arrays are 	`felt`	by default. Simple arrays can be declared as follows - 

```
use array::ArrayTrait;


let mut a = ArrayTrait::new();
a.append(0);
```
Further, you can manipulate arrays using specially designed functions in the core library. For instance, you can remove the first element from an array using ...

```
a.pop_front().unwrap();
```
or you can access an item at an index from an array safely using - 

```
a.get(index_usize)
```

Finally, to print an array, you can use the following code template - 

```
use array::ArrayTrait;
use array::ArrayTCloneImpl;
use array::SpanTrait;
use debug::PrintTrait;
use clone::Clone;

let mut arr1 = fill_arr(arr0);
arr1.append(88);
arr1.span().snapshot.clone().print();
```

## Set 5 - Primitive Types 

The Cairo 1 compiler has a few primitive types hard coded into it. These include felts (duh), strings less than 31 characters (which are also felts), booleans and unsigned integers. There are various size specifications which can be made for the unsigned integers - such as u8, u64 or u256. More on that [here](https://medium.com/nethermind-eth/a-first-look-at-cairo-1-0-a-safer-stronger-simpler-provable-programming-language-892ce4c07b38#6d64). 

Note: Math operations can only be performed between 2 unsigned integers of the same size. And if the result of this operation exceeds their defined size, the compiler will throw an error. The way to deal with this is to return an option (look through Rust fundamentals). 

You can also destructure complex data types in Cairo 1 in a simple way. Eg. 

```
let cat = ('Furry McFurson', 3);
let (name, age) = cat;
```

There is also the `self` keyword which, just like rust, refers to the function / module that the current instrucution is running in. 

Lastly, integers can be converted to felts as ... 
```
use traits::Into;


.into()
```
and vice versa as ... 

```
use traits::TryInto;


.try_into()
```

The conversion of a felt into an integer returns an option, so it has to be unwrapped as - 

```
use option::OptionTrait;


.unwrap();
```

Note: Strings are felt252s and are named in the _naming_scheme style. 

## Set 6 - Control Flow w/ If Statements 

Rust, and by extension Cairo 1, follows a simple if/else workflow which goes something like - 

```
if condition1 {
	result1
} else if condition2 {
	result 2
}
result3 
```
Where the last value is returned if all other checks fail. 

## Set 7 - Structs

Cairo 1 offers a single type of struct. These are named collections of related data stored in fields. We can declare a struct like ... 

```
#[derive(Copy, Drop)]
struct Person {
    name: felt,
    age: felt,
}
```

Note the derive keyword is absolutely essential to this declaration. To instantiate this struct, you can do something like - 

```let john = Person { name: 'John', age: 29 };```

Cairo requires you to initialize all fields when creating a struct. As of now,  there is no no way to update the values in a struct once assigned. Further, you can have multiple data types in a struct, including other structs.

There are some shortcuts that can be taken when destructuring structs,
```
let Foo {x, y} = foo; // Creates variables x and y with values foo.x and foo.y
let Foo {x: a, y: b} = foo; // Creates variables a and b with values foo.x and foo.y
```

Finally, it is also possible to implement logic associated with the fields in a struct in Cairo 1. To understand that, first read up on the concept of traits and implementations over [here](https://medium.com/nethermind-eth/a-first-look-at-cairo-1-0-a-safer-stronger-simpler-provable-programming-language-892ce4c07b38#83b5).  Finally, see an example of how this logic is added to implementations in part 3 of this problem set. 

## Set 8 - Enums 

Cairo allows you to define types called "enums" which enumerate possible values.

Enums are a feature in many languages, but their capabilities differ in each language. Cairo’s enums are most similar to Rust and algebraic data types in functional languages. Useful in combination with enums is Cairo's "pattern matching" facility, which makes it easy to run different code for different values of an enumeration.

Define arbitrary enums as follows - 

```
enum Message {
    Quit: (),
    Echo: felt252,
    Move: (u32, u32),
    ChangeColor: (u8, u8, u8),
```

And declare this as follows - 

```
Message::Move((10_u32, 30_u32))
```

## Set 9 - Snapshots 

Every data type in Cairo 1, say `T`, has a snapshot version `@T` which is always droppable and duplicatable. It represents a snapshot of the value in a given time in the past. This is similar from references - which allow you to shift ownership temporarily and *non-mutably* in a scope - like a function. Few differences - 

- The snapshot is not a pointer, it's the full object, so using it doesn't help with performance 
- If the object is changed after creating the snapshot, the snapshot will not be affected

If you have a value `x` of type `T`, you can use `@x` to get a snapshot of it. For example, if you take a snapshot of an array, and then add a value to the array, the snapshot will still have the original values, without the new value

Some functions, like `Array::get` require a snapshot of the object instead of the object. This is like getting a read-only instance, so it doesn't take ownership of the object to invoke the function. 

Eg. 

```
fn display_grades(student: @Student, index: usize) {
	let mut msg = ArrayTrait::new();
    msg.append(*student.name);
    msg.append('\'s grades:');
    debug::print(msg);
}
```