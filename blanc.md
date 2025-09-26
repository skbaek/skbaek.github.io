# Blanc

[GitHub repo](https://github.com/skbaek/blanc)

## What is it?

Blanc is an EVM programming language designed for ease of formal verification.
You can use it to write EVM smart contracts, and use theorem provers to write 
and prove their precise formal specifications. This gives you extremely strong 
guarantees that the contracts behave correctly per their specification.


## Who is it for?

Those who find the following attractive.

- Low abstraction level: in essence, Blanc is just EVM bytecode + replacing jump 
instructions with branches and function calls. Although this minimal 
abstraction makes a big difference for ease of programming and verification, 
it is still lightweight enough to retain key benefits of staying close to the EVM 
metal:

  - Verified compiler: verify statements about the bytecode that actually 
    runs on EVM, not about high-level semantics whose relationship to runtime 
    behaviour has to be taken on trust.

  - Freedom of low-level control: virtually any contract written in EVM bytecode
    can also be written in Blanc, with the same benefits of minute control and 
    optimization. The foremost target users of Blanc are developers who need 
    both bytecode level optimization *AND* formal verification, but are finding 
    direct verification of bytecode too laborious.

  - No pesky transformations: no need for a transformation step bridging the 
    high level at which programs & specifications are written vs. the low level 
    (typically definitions in a theorem prover) at which they are proven. Always 
    reason about the code directly. 
    
- Natural modeling of contract calls: some formalizations of EVM semantics, such 
as the widely referenced [LEM implementation](https://yoichihirai.com/malta-paper.pdf), make a sharp distinction 
between the current contract vs. the outside world, and model calls to external 
contracts as opaque transitions in which anything can happen unless explicitly 
constrained. This makes it difficult to show properties of contracts that make 
external calls (which is a *lot* of them, considering even a simple ETH transfer 
is a `CALL`!), and requires extra assumptions about invariant conservation over 
external calls, whose addition is only justified by informal arguments. Blanc is 
based on an EVM formalization in which every EVM execution from start to finish 
is a seamless whole, such that invariant conservation over external calls is 
merely an inductive hypothesis that 'falls out for free' from structural 
induction. 

- EVM formalization verified by execution (work in progress): verified compilers 
can bridge the gap between EVM and your programming language semantics, but has 
limited utility if your formal definition of EVM is faulty to begin with.
How do you know that your EVM definition is correct? Even if it was correct for
older versions of EVM, how do you know later EVM updates did not break your proofs? 
The only way to robustly address such concerns, to my 
mind, is to have an *executable* formalization of EVM whose behaviour can be 
automatically checked agaist a large number of test cases. The [Executable Lean
EVM (ELEVM)](https://github.com/skbaek/elevm) is a new Lean 4 formalization of EVM that can run state tests
in standard JSON format, whose correctness respect to Prague has been verified
by passing all general state tests from [execution-specs](https://github.com/ethereum/execution-specs). 
Although Blanc is currently still based on an older EVM 
definition based on the yellow paper, work is underway to replace it with ELEVM. 
When the transition is complete, Blanc programs can be verified against the 
exact version of EVM live on mainnet with high levels of correctness 
guarantees.

## Who is it NOT for?

Blanc, at least in its current form, is a decidedly 'batteries not included'
language. It has none of the programming structs you would normally expect: 
no loops, no types, no data structures, no operator syntax, not even local 
variables, nothing. This will be a dealbreaker for most users who prefer 
the convenience and features of familliar langauges, and are content with 
slightly less streamlined verification or relaxed levels of formal rigor. 
It's not designed to be a go-to language, even for users who need formal 
verification (but it doesn't have to stay that way - see below).

## Any examples?

As an initial proof of concept, the Wrapped Ether (WETH) ERC-20 contract was 
re-implemented in Blanc, together with a formal proof that it always stays 
solvent (i.e. contract ETH balance ≧ sum of all liabilities) through all 
possible transactions. Notably, the Blanc implementation's compiled bytecode 
is 3.66 times smaller than the original (854 bytes vs. 3124 bytes), and it 
also shows moderate gas savings around a single digit % for every function 
in the contract, showing the effectiveness of low-level optimizations. 
In addition, the proof of concept also demonstrates of how formal verification 
is better than any test set at catching even the most improbable edge cases:
for instance, the WETH implementation in Blanc had to be modified to store 
user WETH balance data in a custom mapping instead of standard hashmaps because 
the solvency proof required a guarantee that data corruption from hash 
collisions cannot happen.

## What's next?

Blanc is still very nascent, and much work is needed to make it a pragmatic 
programming language. Current areas of focus include:

- Using ELEVM: the low hanging fruit, already mentioned above. Required to ensure 
correctness in respect to the current Ethereum mainnet.

- Libraries: a standard library that implements a few of the most heavily used 
programming structures & features, and proves their basic properties frequently 
used in verification, will go a long way to making Blanc more usable out of the 
box.  A crude example of this already exists in the WETH implementation: its 
function dispatcher is completely standard for all EVM smart contracts and can 
be reused as-is in other projects, along with basic related lemmas like "if all 
functions in this dispatch tree preserves property P, then running the contract 
preserves P." The standard library should include basic data structures, helper 
functions for iteration, local variable management, etc.

- Tooling: Lean 4 is a 'good enough' IDE for Blanc for now, but eventually Blanc 
will need its own dedicated toolchain. Blanc currently has some very rough edges,
such as forcing users to manually manage the indices of functions in a contract, 
or mentally keep track of stack state after each opcode. Proper tooling should 
abstract and automate them away so that programmers can focuse on higher level 
features.

- More real usage: ultimately, all of the efforts above have to prove their 
usefulness by application to examples more complex than an ERC-20 token. Ideally, 
the example would be a smart contract difficult to write and verify using other 
existing methods due to requiring both aggressive optimizations and a tiny trust 
base, thereby showcasing Blanc's strengths.

