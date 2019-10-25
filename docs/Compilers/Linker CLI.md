## Key Use Cases

The linker Toolchain tool is called the *tvm_linker* and can be used in one of the following modes:

- **Compilation**. In this mode the linker compiles several modules together and provides final contract (a .tvc file) for further running using TVM.
- **Test run**. In this mode the linker runs previously prepared compiled contract using TVM emulator. This may be used to check the correctness of some smart contract parts without running them on a node.
- **Message preparation**. In this mode, the linker prepares an inbound message for a contract in the *.boc* format.

### Compilation

Following command line format should be used to compile several modules together:

```scala
tvm_linker compile [--debug] [--lib <STDLIB>] <ASM_FILE> ...

```

where:

- `*--* `(first instance) stands for the path to an assembler file; there can be several assembler files for compilation;
- `*--*` (second instance) stands for the path a to standard library assembler file (for example, stdlib_c.tvm);
- `--debug` command line option enables debug output of linking; for example, it can be used to get addresses of global symbols (for example, of functions).

After the above command is executed, the linker delivers a compiled contract in the *.tvc* format.

### Test Run

After assembler files are linked together, you can use the linker to run a compiled contract. This command does not start the node and has a limited usage.

Mainly, test runs are used for local debugging of code parts the do not require interaction between smart contracts (e.g.: unit tests).

Command line format for test run is following:

```scala
tvm_linker test <CONTRACT_ADDRESS> [--trace] [--decode-c6] --body 00<ENTRY_ADDRESS>

```

where:

- `*--*` (instance one) stands for the address of a contract that has to be run. Note that the .tvc extension is not included;
- `--` (instance two) stands for the contract starting point; it can be extracted from the `tvm_linker compile` output in `--debug` mode;
- `--trace` option is used to trace VM execution; each time a VM command is executed, stack and registers are printed;
- `--decode-c6` option is used to display output actions in a friendly format.

### Message Creation

Messages can be sent to a contract in a *.boc* format. These messages use contract ABI and a list of input key-value pairs.

In the command line a message has the following format:

```scala
tvm_linker message <contract-address> [--init] [-w] --abi-json <json-file-with-abi> --abi-method <method-name> --abi-params <json-string-with-params>
```

where:

- `--` (instance one) stands for the address of a target contract; note that the .tvc extension is omitted;
- `--` (instance two) stands for the workchain ID used with the contract address;
- `--` (instance three) stands for the path to a relevant ABI file that defines methods and their input/output parameters (see description below);
- `--` (instance four) stands for the contract method called via the message;
- `-- `(instance five) stands for contract method parameters in JSON format (for example, {“a”: “0x123”, “b” : “456”});

## ABI File Format

An ABI file defines available contract methods, input parameters and outputs. ABI has a JSON-based syntax with the following structure:

- `“ABI version”`: the version of ABI file format (should be 0 for now);
- `“functions”`: array of contract methods.

Each contract method has following format:

- `*"name"*` - the name of a method (the method requires the` *_Impl* suffix`; for example, if an ABI file has a `*‘fn1’*` method, then contract should have implementation of this method entitled `*‘fn1_Impl’*`);
- `*“signed”*` defines whether the method is signed or not (values are `“true"` or `“false”`)
- `*“inputs”*` contains the list of input parameters;
- `*“outputs”*` stands for the list of output parameters.

Each method parameter has the following format:

- `“name”` stands for name of a parameter;
- `“type”` type of a parameter ( only “uint64” is supported now).

For example:

```javascript
{
    "ABI version" : 0,
    "functions" : [
      {
        "name" : "fn1",
        "signed" : "false",
        "inputs" : [
          {"name" : "arg1", "type" : "uint64"}
        ],
        "outputs" : [
          {"name" : "result", "type" : "uint64"}
        ]
      }
    ]
}
```

## More Help

Use `tvm_linker --help` for detailed description about all options, flags and sub-commands.