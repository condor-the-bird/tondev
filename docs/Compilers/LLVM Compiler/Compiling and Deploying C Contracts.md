# Compiling and Deploying C Contracts

Contract interaction is based on message exchange. A contract gets a message, performs the relevant computation and sends a response message. 

The computation stage consumes gas, and if the limit of gas exceeded the contract terminates with an exception.

## Syntax differences

Contracts are similar to standard programs in C, but there are important syntax differences:

1. ABI file. An ABI file encodes signatures and properties of public functions. This information is not stored in the blockchain and comes in JSON format for the sake of interoperability between contracts written in different languages. `abi_parser` produces header and source code files required for compilation, and the ABI file itself is needed for deploying the contract.
2. A public method must not take an argument or produce a result. Instead, it deserializes parameters from incoming messages and then serializes output values to send a message. `abi_parser` generates boilerplate functions for each public method from abi. These functions perform serialization, call corresponding `*_Impl` function (e.g. `transfer_Impl` for `transfer`), and serialize its result.
3. There are persistent and non-persistent global data. Persistent data is marked with `_persistent` suffix (we are going to use GCC-style attribute in future). Persistent values are preserved between public contract's functions call.

## Compiling a contract

To compile a contract you need Clang for TVM (https://github.com/tonlabs/TON-Compiler) either built from binaries or delivered as a part of Node SE distribution. Aside from the compiler, you need ABI parser tool, C runtime and SDK headers and sources. They are all located in `stdlib` subdirectory of the repository or the distribution:

- ABI parser - `stdlib/abi_parser.py`
- C runtime - `stdlib/stdlib_c.tvm`
- SDK headers and sources - `stdlib/ton-sdk`

To demonstrate the compilation workflow, we use piggybank contract. It's located at `samples/sdk-prototype/piggybank.c` in the source code repository of the compiler.

First, we need to run `abi_parser`:

```
abi_parser.py path/to/piggybank/piggybank
```

**Note** that `.abi` is not need to be specified at the end of the filename.

The script produces `piggybank.h`, `piggybank_wrapper.c`, `piggybank.err` files. The last one should be empty if no errors happen during compilation. `piggybank.h`, `piggybank_wrapper.c` contains the boilerplate code for public functions we described before.

The compilation of the contract itself is relatively straightforward

```
clang -target tvm -O3 -S piggybank.c piggybank_wrapper.c -I/path/to/stdlib
```

Note that Clang for TVM is not able at the moment generate object or a boc binary file, so `-S` flag is necessary to produce assembly output. `-O3` is recommended to use, because the compiler is more reliable with this option.

Aside from the contract itself, SDK sources also need to be compiled (in future we plan to introduce bootstrapping phase and the work will be performed when the compiler is built from sources)

```
clang -target tvm -O3 -S /path/to/stdlib/ton-sdk/*.c -I/path/to/stdlib
```

The resulting assembly files need to be concatenated because the linker tool doesn't currently support multiple input files

```
cat *.s > piggybank.combined.s
```

Alternatively (and it's better from optimization point of view) you can use `llvm-linker` tool to link modules on LLVM IR level. Clang for TVM provides a script to automate the work with LTO, and we describe how to work with it later.

To link the contract you need `piggybank.combined.s` and `piggybank.abi`:

```
tvm_linker compile piggybank.combined.s --abi-json piggybank.abi --lib /path/to/stdlib/stdlib_c.tvm
```

## Deploying the contract

Deploying steps are language independent. Please, refer to the relevant documents in Node SE documentation or in the TVM Linker specification.