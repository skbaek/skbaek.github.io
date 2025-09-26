# ELEVM

Executable Lean EVM (ELEVM) is an implementation of the Ethereum Virtual Machine 
(EVM) in the Lean 4 theorem prover. ELEVM can run EVM state tests in the standard JSON 
format of Ethereum's [execution spec tests](https://github.com/ethereum/execution-spec-tests). 
It passes all general state tests for the Prague fork generated & used by the 
latest mainnet version[^1] of [execution-specs](https://github.com/ethereum/execution-specs).

ELEVM puts strong emphasis on executability because it is the best way to achieve 
high confidence in its correctness. Formally verifying smart contracts against a 
definition of EVM won't do much good if the EVM definition itself contains 
errors. Unfortunately, EVM contains too many moving parts and edge cases for 
manual inspection --- you can't just 'squint really hard' for a few minutes to 
satisfy yourself the definitions are all correct as you would for, say, a formalization 
of Euclidean geometry. The closest we can get to 100% confidence is making the 
specification executable, so that its output can be compared against both comprehensive 
test sets and output from other implementations.

ELEVM first began as an effort to replace the old EVM definition in 
[Blanc](https://github.com/skbaek/blanc/).  Since the formalization is 
application-agnostic and can be useful for other purposes, it was spun off as 
a separate library.

One notable feature of ELEVM is that it is completely self contained in Lean 4.
It uses no foreign function interface with external libraries written in C or 
other langauges. Its only dependency is Lean's generic [mathematics library](https://github.com/leanprover-community/mathlib4). 
In particular, all hash functions and elliptic curve cryptography used by ELEVM 
is implemented natively in Lean 4. Although this design choice was mainly motivated 
by convenience of maintenance and personal curiosity (and makes ELEVM unsuitable 
for powering an actual Ethereum client), it could be useful for 
projects that need to reason about the properties of such functions. Note that 
some parts of precompiled contracts and their related cryptographic functions 
remain unimplemented, because they were not necessary for passing the latest set 
of state tests from execution-specs. These are are slated for future work.

Unless otherwise noted, most of ELEVM is ported from the Python codebase of 
execution-specs.

[^1]:As of 2025/09/19, commit [`4198...7694`](https://github.com/ethereum/execution-specs/tree/4198b9c5996713b268aed602739d5aa40e277694)