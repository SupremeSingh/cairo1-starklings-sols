## Cairo 1 Under the Hood - 

Let's try and understand what exactly happens when we "run" our Cairo 1 programs. In short, there are three important steps in the new our program's lifecycle - 

- Cairo Code (High Level)
- Sierra (Intermediate) Representation
- CASM (Assembly) 

After this, the program trace goes through the highly mathematical concept of arithmetization, various commitment stages, FRI and is finally deployed to StarkNet and proven as a STARK. More on that [here](https://starkware.co/stark-101/).  

You've already written the code - now let's look at what this "Sierra" thing is. 

In Cairo 0, your high level programs were compiled directly to assembly language. However, Cairo 1 introduces a new "safety-net" for it's programs called Sierra. You can think of Sierra as a semi low-level language which your program compiles to before going on to the assembly stage.

### Program to Proof ??? 

Cario instructions from a program (represented as assembly) can be translated to a set of polynomial constraints 
These constraints are then used to prove the cairo program’s validity to Ethereum 

If the instructions are not computationally valid, eg. ‘assert 0 = 1’, then they cannot be translated to a valid set of polynomial constraints 

This means, invalid or broken programs in Cairo CANNOT be proven to Ethereum


### Why Sierra ??? 

When a sequencer receives programs from users, it is basically receiving instructions to carry out a set of transactions 
The compilation step (Cairo > CASM) only checks if the program runs correctly, which it may even if it’s implied transactions are indeed invalid.

If the sequencer finds a program’s transactions are invalid, it does not include them in the state update it sends to Ethereum mainnet (since the’re not provable). Still, the sequencer spends some computation and memory, along with time, in validating these transactions. 

Note here, a sequencer can only charge users gas fees if it can prove to them that is has included their state updates in the Ethereum blockchain 

Fundamentally, the sequencer spends money on checking bad transactions but does not have a way to be compensated by the users for doing so. To deal with this, Starknet has come up with a fail-proof representation of transactions. 

In comes Sierra !! 

Basically, once you write your Cairo 1 program - the compiler auto-generates another program based on yours. This new program (written in a lower language) is written in a fail-safe way, i.e, it CANNOT fail. At best, it can have branching conditions that return `true` or `false` if a set of conditions is not met. 

For instance, it will turn the (potentially) error throwing pseudo code ... 

```
 // assert myBalance >= myTransactionSize
```
into 
```
if (myBalance < myTransactionSize) -> return false 
else -> go on 
```
so now, the program returns a valid (albeit `false`) value instead of breaking and outputting an error message. 


Now, the CASM has valid instructions, which can be proven and included in Ethereum state updates. Of course, the invalid business logic will fail when run in the EVM. This way, the sequencer can even send problematic transactions to Ethereum (to be rejected there), and verifiably charge users gas fees for their bad transactions. 	

### How Sierra Works 

First, let's define what can throw a fail when running a Cairo program - 

- Assert statements
- Dereferencing illegal addresses
- Undefined Opcodes
- Very long runs 

Now that we have concrete examples of things Sierra is supposed to protect - we want to make a language that is 

- Safe
- Efficient to compile
- Simple to understand 
- Has low overhead

You can take a look at the various safety features implemented in Sierra over [here](https://www.youtube.com/watch?v=-EHwaQuPuAA). Your Sierra code is converted to CASM at deployment time, so from the user's perspective - you basically end up deploying Sierra and not CASM. 

For more on Cairo 1 and it's future implementations - please take a look at [this](https://www.youtube.com/watch?v=qp2YIy8JN10). 





 

