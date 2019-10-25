# Archive

# October 11, 2019

## TON Labs Node SE

### Local Node

1. Local Node now starts with pre-deployed giver contract with address **0:ce709b5bfca589eb621b5a5786d0b562761144ac48f59e0b0d35ad0973bcdb86**
2. GraphQL fixes:

- NEW: Log errors thrown on subscription filter test
- FIX: Use empty filter if it was not specified in subscription params
- FIX: Some u64 fields are now declared as float

### **Client Libraries**

1. NEW: Automatic money transfer to the address before deployment was removed from deploy method. Now, you have to carry out the following procedure:
2. Pre-calculate address of the contract, using `createDeployMessage.`
3. Transfer Grams to that address from pre-deployed giver or from another contract.
4. Run the Deploy method to deploy the Contract.

For more details, go to the [Deployment](https://zeroheight.com/86757ecb2/p/665008/t/286597) section.

​     \2.  NEW: Error handling based on the `error-chain` crate implemented in the Rust client.

​     \3. NEW: JS Client Libraries connect to the Internet at request. So, you can now use the crypto module [offline/externally](https://docs.ton.dev/86757ecb2/p/891ee9/t/359b46).

## Documentation

See the [Deployment](https://zeroheight.com/86757ecb2/p/665008/t/286597) section for updates.

## Component Versions

- tonlabs/local-node:0.13.0
- ton-client-node-js:0.13.0
- ton-client-js:0.13.0
- ton-client-web-js:0.13.0
- ton-client-react-native-js:0.13.0
- ton-client-rs:0.13.0
- ton-q-server:0.13.0

## Update Guide

The way the contracts are deployed has been changed. See documentation for more details.

# October 07, 2019

> Note: this version is incompatible with the previous. Please, read the update guide below the change-log.

## Local Node

1. Bug fixes in GraphQL service.
2. Gas cost optimizations in the TVM.

## **Client Libraries**

1. Shared Client Instance is removed from ton-client-js because of its redundancy. It is not accessible from JS bindings any more. See the Upgrade Guide below.
2. All JavaScript unit tests are moved from `ton-client-node-js` to `ton-client-js` repo.

## CLI

1. Dramatic changes in the source code. New commands added:
2. `restart` to restart containers.
3. `recreate` to recreate containers.
4. Called without parameters, `tondev` is similar to the` info` command.

## Component Versions

- tonlabs/local-node:0.12.1 (ton-q-server:0.12.0)
- tonlabs/compilers:0.12.1
- ton-client-node-js:0.12.1
- ton-client-js:0.12.1
- ton-client-rs: 0.12.100 with binaries 0.12.100
- ton-dev-cli:0.12.3

## Update Guide for JS Projects

Read the suggestions below and apply these fixes to your projects to ensure new version compatibility with your projects.

All usage instances of the TONClient.shared must be replaced with an instance of TONClient. Create it yourself in the application code. The initialization procedure is unchanged.

Take the code below:

```javascript
const ton = TONClient.shared;
ton.config.setData({
    defaultWorkchain: 0,
    servers: ['<http://0.0.0.0>'],
    log_verbose: true,
});
```

and replace it with the following:

```javascript
const client = new TONClient();
client.config.setData({
    defaultWorkchain: 0,
    servers: ['http://0.0.0.0'],
    log_verbose: true,
});
await client.setup();
```

Also, we introduced a convenient method to create and setup a new client instance.

This example is full  equivalent to the above in terms of functionality.

```javascript
const client = await TONClient.create({
    defaultWorkchain: 0,
    servers: ['http://0.0.0.0'],
    log_verbose: true,
});
```

# September 28, 2019

## TON Labs Node SE 

### Client Libraries

*Version 0.12.0*

- To save gas, dynamic arrays are now passed as dictionaries using `HashmapE` type from the TVM specification.  Previously, these were passed as contract function parameters (far more gas consumed);
-  Queries module and local TVM call function added to the Rust Client.

### Compiler Kit

*Version 0.12.0*

Solidity compiler:

- Added support for the fallback function
- Optimized array storage format. Deserialization of an array now consumes way less gas than before
- Improved error reporting

### Local Node

*Version 0.12.0*

- Optimizations were made

# September 20, 2019

#### Version 0.11.1

## Solidity Compiler Updates

### Changes and improvements

- Changed ABI encoding for events:` _event` suffix removed
- Improved support for `sha256` and `abi.encode`
- Support for primitives from TVM specification of September 6, 2019
- Improved Tuples support
- Support for virtual functions, delete

### Bug-fixes

Fixed crashes that took place at attempt to parse an expression of an unsupported type (in particular, explicit return value from an external function call).

## LLVM C Сompiler

Optimizations & bug-fixes. 

# September 19, 2019

## tondev 0.11.8 Released

Version 0.11.8 of tondev utility allows you to:

1. Use containers in multi-user environment. Username is added to the  container name.
2. Use `info` command to get the current Node SE state. The command shows:
3. the list of images and docker containers related to Node SE. 
4. current container state
5. list of versions available at docker hub. 
6. container settings
7. the version in use
8. Set an alternative port for the local node.
9. Switch between container versions with the  `use <version>` command.

By default` :latest` is used.

# September 09, 2019

## TON Labs Node SE First Release

### **TON Labs Node SE**  Highlights

- **TON Labs Local Node:** debug and test your smart contracts in a controlled environment emulating a production one.
- **TON Labs Compiler Kit:** compile TON-ready smart contracts from source code in Solidity and	in LLVM-compliant languages. Reuse your programming skills to create TON smart contracts in C or to migrate code from previous Ethereum projects to TON with minor tweaks. The kit contains the latest version of compilers (see below).
- **TON Labs Client Libraries** for Node.js, React Native, Web and Rust with modules for cryptography,	contract development, blockchain querying and monitoring powered by GraphQL protocol.
- **Detailed online documentation**: сheck it to start with TON Labs Node SE and to learn more about the options it offers.

## Solidity Compiler Updates

Support of the July 20, 2019 specification. Additional features supported:

- contract inheritance
- bool type variables
- address(this)
- ternary operator
- member constants
- assignments in expressions
- array pop, push, length
- arrays as function parameters
- default member variables
- now
- sha256(abi.encode)
- nested structures encoding
- return arrays in public methods
- arrays in events

## LLVM Compiler Optimizations

- Support for the July 20, 2019 specification
- Limited support for C++:
- C++98, C++03, C++11, C++14, most features of C++17. For more details see <https://clang.llvm.org/cxx_status.html> for Clang 7
- It was checked that classes, inheritance without virtual methods, templates and overloading work.
- Note that virtual methods and exceptions are not supported.

All in all, for TVM we recommend using C++ as C with classes and templates rather than as an idiomatic C++.

- Additional performance optimizations, debug printouts

# July 25 Release

The updated beta includes:

- C compiler based on the LLVM framework with samples (see below);
- Solidity compiler improvements (full list below);
- More sample contracts: [https://github.com/tonlabs/samples](https://github.com/tonlabs/samples;);
- Documentation updates covering new contract features and options and tvm-linker usage.

## Sol2TVM Changelog

- Added support for:
- `msg.sender`
- `tvm_logstr()`
- `pure interfaces`
- `address.transfer()`
- `tvm_signed() `modifier
- `address(this).balance`
- Additional range checks when working with limited types like uint8
- Improved conversions between addresses and contracts
- New sample contracts in Solidity:
- contract01: demonstrates how to save data to a persistent storage. Use the **getaccount** command to obtain the contract state;
- contract02: demonstrates how to call a remote contract by passing an integer. The remote contract saves this integer and it is available for checking;
- contract03: demonstrates how to use the **msg.sender** function to get address of a contract that called you;
- contract04: demonstrates how to transfer grams with the **address.transfer()** function;
- contract05: callback implementation. One contract calls another one and gets response with a callback;
- contract06: demonstrates implementation of mappings, external methods and callbacks as well.

## C2TVM Release

> **Warning**: This compiler version is a beta meaning some unexpected behavior could be observed. Please, avoid commercial use. Notify us of any bugs and errors you encounter. Also note that contract performance needs further optimization and we are working on it.

Sample contracts in C:

- example-1-hello-world: simplest contract in the repository, it adds 2 and 2 and returns the sum as an external message. This contract is a good starting point.
- example-2-echo: stores the last input value in a persistent variable. This allows the user to check the results of the contract's method invocations using **getaccount** command.
- example-3-transfer-80000001: sends 0xAAAA nanograms (43960 in decimal) to the account 0x80000001. This contract shows how to create and send internal messages.

### Supported Features

- Integer arithmetic
- Functions
- Arrays, structures, pointers
- Loops
- Global (+ persistent) and automatic memory

### Known C Compiler Issues & Limitations

Below is the list of known issues (we keep on working to resolve them):

- Persistent & global memory: always specify a value at initialization.

```c
int x;                // bad
int y_persistent;     // bad
int p = 0;            // good
int q_persistent = 0; // good
```

- Global memory: before the first use always initialize values in the code

```c
int x = 0;
int func1_Impl () {
    int p = 0;
    int q = p;    // fine: local initialization works
    return x * 2; // not guaranteed to work
}

int func2_Impl () {
    x = 0;
    return x * 2; // good
}
```

- Function pointers: not granted
- Compile-time arithmetic may be different from runtime; arithmetic operations in TVM are quite different from C standards
- Long integer and unsigned literals (> 64 bits)
- Only int and uint ABI types are supported now; no values of other types can be transferred to or from contract
- Character values and literals
- Dynamic arrays on stack



```c
int f (int n) {
    int array [n]; // will not work
}
```

- Heap memory: not supported with no immediate plans to support
- Float values: not supported with no immediate plans to support
- Goto operators

# July 04 Release

Full development cycle for smart-contracts in Solidity language, version 0.5.0.

Supported TVM specification version: March 29, 2019.

## Components

### Online IDE (Eclipse Che)

- The help menu is currently disabled. 
- Terminal delays can be expected (especially, when left unattended). Open a new one, if it happens.
- Several sample contracts are available at [https://github.com/tonlabs/sample](https://github.com/tonlabs/samples) more to be added later.

### Solidity compiler

- Programming limitations and features covered in the relevant document.

> **Warning**: This compiler version is a beta meaning some unexpected behavior can be expected. Please, avoid commercial use. Notify us of any bugs and errors you encounter.