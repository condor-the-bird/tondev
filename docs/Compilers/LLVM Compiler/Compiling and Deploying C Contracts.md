# Compiling and Deploying C Contracts

Contract interaction is based on message exchange. A contract gets a message, performs the relevant computation and sends a response message. 

The computation stage consumes gas, and if the limit of gas exceeded the contract terminates with an exception.

## Syntax differences

Contracts are similar to standard programs in C, but there are important syntax differences:

1. ABI file. An ABI file encodes signatures and properties of public functions. This information is not stored in the blockchain and comes in JSON format to enable interaction between contracts originally written in different languages. `abi_parser` produces header and source code files required for compilation, and the ABI file itself is required to deploy a contract.
2. A public method must not take an argument or produce a result. Instead, it deserializes parameters from incoming messages and then serializes output values to send a message. `abi_parser` generates boilerplate functions for each public method from abi. These functions perform serialization, call corresponding `*_Impl` function (e.g. `transfer_Impl` for `transfer`), and serialize its result.
3. There are persistent and non-persistent global data. Persistent data is marked with `_persistent` suffix (we are going to use GCC-style attribute in future). Persistent values are preserved between public contract's functions call.

## Compilation 

To compile a contract, get Clang for TVM (see https://github.com/tonlabs/TON-Compiler) built from binaries or delivered with Node SE distribution. 

Apart from the compiler, you need an ABI parser tool, C runtime, SDK headers and sources. They are all located in `stdlib` subdirectory of the repository or of the distribution package:

- ABI parser - `stdlib/abi_parser.py`
- C runtime - `stdlib/stdlib_c.tvm`
- SDK headers and sources - `stdlib/ton-sdk`

To demonstrate the compilation workflow, we use a sample piggybank contract from the compiler repository (`samples/sdk-prototype/piggybank.c` ) .

1. Run `abi_parser`:

```
abi_parser.py path/to/piggybank/piggybank
```

**Note** the `.abi` extension is not mandatory.

The script produces `piggybank.h`, `piggybank_wrapper.c`, `piggybank.err` files. 

The last file has to be empty if compilation ended successfully.

 `piggybank.h`, `piggybank_wrapper.c` contain the boilerplate code for public functions.

2. Run the compiler:

```
clang -target tvm -O3 -S piggybank.c piggybank_wrapper.c -I/path/to/stdlib
```

Note that now Clang for TVM is unable to generate an object or a `.boc` binary file, so the `-S` flag is mandatory to produce an assembly output.

 `-O3` is the recommended optimization level that we also use in tests.

3. Aside from the contract itself, compile the SDK sources:

```
clang -target tvm -O3 -S /path/to/stdlib/ton-sdk/*.c -I/path/to/stdlib
```

4. Concatenated the output assembly files for the linker (this utility does not support multiple file input now):

```
cat *.s > piggybank.combined.s
```

An alternative option is to use the `llvm-linker` tool to link modules on LLVM IR level. 

5. To link the contract you need `piggybank.combined.s` and `piggybank.abi`:

```
tvm_linker compile piggybank.combined.s --abi-json piggybank.abi --lib /path/to/stdlib/stdlib_c.tvm
```

## Deployment

Deploying steps are language independent. Please, refer to the relevant documents in Node SE documentation or in the TVM Linker specification.