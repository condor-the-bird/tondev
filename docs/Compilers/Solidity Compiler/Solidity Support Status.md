# Solidity Support Status

## Fully Supported/Pending Release

Fully supported features and feature groups.

| **FEATURE**                                | **USAGE**                                                    | **NOTES**                                                    |
| ------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| **Pragmas**                                | Pragma...                                                    |                                                              |
| **Version**                                | pragma solidity >=0.5.0 <0.6.0                               |                                                              |
| **Experimental**                           | pragma experimental ...                                      |                                                              |
| **ABIEncoderV2**                           | pragma experimental ABIEncoderV2                             |                                                              |
| **Import**                                 | `import "filename" ``import * as symbolName from "filename" ``import {symbol1 as alias, symbol2} from "filename"` | Additional test are in the pipeline                          |
| **Comments**                               | //, /* */                                                    |                                                              |
| **Boolean:**                               | bool, true, false                                            |                                                              |
| **operators**                              | !, &&, II, ==, !=                                            |                                                              |
| **lazy evaluation**                        | `f(x) == true ⇒ (f(x) || g(y)) == true`                      |                                                              |
| **integers**                               | `intN/uintN`, `N=8..256`                                     |                                                              |
| **comparisons**                            | `<=`, `<`,` ==`,` !=`,` >=`,` >`                             |                                                              |
| **Bit Operators:**                         | `&`, `|`,` ^`,` ~`                                           |                                                              |
| **shift**                                  | `<<`, `>>`                                                   |                                                              |
| **base arithmetic**                        | `+`, `-`, `unary -`, `*`                                     |                                                              |
| **division**                               | `/`                                                          |                                                              |
| **modulo**                                 | `%`                                                          |                                                              |
| **exponent**                               | `**`                                                         |                                                              |
| **Ternary operator**                       | `b = true ? 2 : 3;`                                          |                                                              |
| **Enums**                                  | `enum ActionChoices { GoLeft, GoRight, GoStraight, SitStill }` |                                                              |
| **Return variables**                       |                                                              |                                                              |
| **Structs**                                | `struct`                                                     |                                                              |
| **Mapping types**                          | `mapping`                                                    |                                                              |
| **Integer types**                          |                                                              |                                                              |
| **Conversions between Elementary Types**   | Implicit conversions	Explicit conversions                 |                                                              |
| **Order of Evaluation of Expressions**     | not specified, lazy bools                                    |                                                              |
| **Assignments for arrays and structs**     |                                                              |                                                              |
| **Assignments in expressions**             |                                                              |                                                              |
| **Member constants**                       |                                                              |                                                              |
| **Default member values**                  |                                                              |                                                              |
| **View functions**                         | `function f(uint a) public view returns (uint)`              |                                                              |
| **Pure functions**                         | `function f(uint a) public pure returns (uint)`              |                                                              |
| **Now**                                    |                                                              |                                                              |
| **Arrays in events**                       |                                                              |                                                              |
| **Return arrays in public methods**        |                                                              |                                                              |
| **Nested structure encoding**              |                                                              |                                                              |
| **Contract types**                         | `My contractC`                                               |                                                              |
| **Using for**                              | Using B for A                                                | Library functions are added to a type                        |
| **Abstract contracts**                     | `contract A { function u() public; } `                       |                                                              |
| **Metadata**                               |                                                              | ABI, version, etc.                                           |
| **Super**                                  | `super.method()`                                             |                                                              |
| **Return**                                 |                                                              | |
| **Multiple Inheritance and linearization** |                                                              |                                                              |
| **Arguments for base constructors**        |                                                              |                                                              |
| **Returning multiple values**              | `return (n1, n2, n3);`                                       |                                                              |

## Partially  Supported/Planned

Features and feature groups that are now partially supported and/or planned to be supported. 

Note that implementation priority depends on the demand for the feature, its relevance to the TVM and on the overall effort estimate. 

| **NAME**                                                  | **USAGE**                                                    | **NOTES AND LINKS**                                          |
| --------------------------------------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| **Data Location:**                                        |                                                              |                                                              |
| **memory**                                                |                                                              | Fully supported                                              |
| **storage**                                               |                                                              | Ethereum-specific, unsupported now. Always use **memory** location |
| **calldata**                                              |                                                              | Same as above                                                |
| **Block and Transaction Properties:**                     | Global symbols initially provided by EVM developers for debugging | The feature set is partially supported. You can use: 	*get_block_lt**tvm_trans_lt**get_balance**get_address* |
| **hash of the given block**                               | `blockhash (uint blockNumber) `returns (bytes32)             | Not yet fully supported; only available for 256 most recent blocks excluding the current one |
| **current block miner address**                           | `block.coinbase (address payable)`                           | Not yet supported                                            |
| **current block difficulty**                              | `block.difficulty (uint)`                                    | Not yet supported                                            |
| **current block gas limit**                               | `block.gaslimit (uint)`                                      | Not yet supported                                            |
| **current block number**                                  | `block.timestamp (uint)`,` now(uint)`                        | Supported                                                    |
| **current block timestamp;** **seconds since Unix epoch** | `block.number (uint)`                                        | Supported                                                    |
| **remaining gas**                                         | `gasleft() returns (uint256)`                                | Not yet supported                                            |
| **complete call data**                                    | `msg.data (bytes calldata)`                                  | Not yet supported                                            |
| **message sender (current call)**                         | `msg.sender (address payable)`                               | Supported                                                    |
| **first 4 bytes of the call data (function identifier)**  | `msg.sig (bytes4)`                                           | Supported                                                    |
| **number of WEI sent with the message**                   | `msg.value (uint)`                                           | Supported                                                    |
| **transaction gas price**                                 | `tx.gasprice (uint)`                                         | Not yet supported                                            |
| **transaction sender (full call chain)**                  | `tx.origin (address payable)`                                | Not yet supported                                            |
| **Contract Related:**                                     |                                                              | Partial support of the feature group                         |
| **this**                                                  | `address(this)`                                              | Supported                                                    |
| **inheritance**                                           |                                                              | Supported                                                    |
| **sefldestruct**                                          | Deletes the active contract and sends funds the provided Address | Not yet supported, planned.                                  |
| **Value types**                                           |                                                              | Partially supported.  Value type variable handling optimization & behavior documenting pending. The suggested workaround is to create a copy explicitly, if needed |
| **Reference types**                                       |                                                              | Partially supported feature. Reference types are opposite of fixed types. They can be deployed after TON TVM memory features are supported completely. No workarounds or ETA at the moment. |
| **Rational literals**                                     | `.1, 2e-10`                                                  | Planned                                                      |
| **Hexadecimal literals**                                  | `hex"001122FF"`                                              | Currently tested                                             |
| **Interfaces**                                            | `interface Token { struct ... function ... }`                | Only pure interfaces are supported                           |
| **Function Modifiers**                                    |                                                              | Unsupported now, planned to support.                         |
| **Function calls**                                        |                                                              | The feature group is partially supported                     |
| **internal function calls**                               |                                                              | Supported                                                    |
| **external function calls**                               |                                                              | Supported                                                    |
| **named calls and anonymous function parameters**         | `function f() public { set({value: 2, key: 3}); } `	`function set(uint key, uint value) public { data[key] = value; }` | Not supported, tests are being carried out. A fairly complex and rarely used feature. Implementation depends on feedback |
| **omitted function parameter names**                      | `function func(uint k, uint) public pure returns(uint) { return k; }` | Supported, tests in the pipeline                             |
| **creating contracts via new**                            | explicit constructor call                                    | Not supported, inheritance-dependent.                        |
| **Fallback**                                              | `function () external { ... }`                               | Currently in development.                                    |
| **Address:**                                              | `address` `addr`                                             | The group is partially supported. Address literals and address payable types are not supported yet. |
| **address payable**                                       | `address payable addrPayable`                                | More research of the type needed                             |
| **Address Literals**                                      | `"0xdCad3a6d3569DF655070DEd06cb7A1b2Ccd1D3AF"`               | It is suggested to convert to uint160                        |
| **address comparisons**                                   | `<=`, `<`, `==`, `!=`, `>=`, `>`                             | It is suggested to  convert to uint160                       |
| **balance**                                               | `address.balance`                                            | Supported                                                    |
| **transfer**                                              | `addressPayable.transfer(N)`                                 | Supported                                                    |
| **send**                                                  | `addressPayable.send(N)`                                     | Supported                                                    |
| **call, delegatecall, staticcall**                        | *`call(bytes)`                                               |                                                              |
| **Arrays:**                                               |                                                              | This feature groups is partially supported                   |
| **fixed size**                                            | `[k]`                                                        | Supported                                                    |
| **dynamic size**                                          | `[]`                                                         | Supported                                                    |
| **bytes and strings as arrays**                           |                                                              | Planned                                                      |
| **memory arrays**                                         | `bytes[] `                                                   | Partial support, all arrays are in memory                    |
| **literals**                                              | `[1, 2, 3]`                                                  | Supported, tests under way                                   |
| **length**                                                |                                                              | Supported                                                    |
| **push**                                                  |                                                              | Supported                                                    |
| **pop**                                                   |                                                              | Supported                                                    |
| **Events:**                                               | `event Deposit(address _from, uint _value); `                | Features from this group are partially supported             |
| **emit**                                                  | `emit Deposit(msg.sender, msg.value);`                       | Partially supported                                          |
| **indexed**                                               |                                                              | Stores event parameter as topic, not supported now           |
| **anonymous**                                             |                                                              | Does not store event signature as topic, not supported now   |
| **Math and Crypto:**                                      |                                                              | The feature group is partially supported                     |
| **(x + y) % k**                                           | `addmod(uint x, uint y, uint k) returns (uint)`              | Not supported; not relevant for the TVM                      |
| **(x \* y) % k**                                          | `mulmod(uint x, uint y, uint k) returns (uint)`              | Not supported now; not relevant for the TVM                  |
| **Keccak-256 hash**                                       | `keccak256(bytes memory) returns (bytes32)`                  | This option will not be supported. An analog is planned for release within the next few months based on `tvm_hash()`. No workarounds at this moment. |
| **SHA-256 hash**                                          | `sha256(bytes memory) returns (bytes32)`                     | Supported partially                                          |
| **RIPEMD-160 hash**                                       | `ripemd160(bytes memory) returns (bytes20)`                  | This option will not be supported. An analog is planned for release within the next few months based on tvm_hash(). No workarounds at this moment. |
| **erecovery**                                             | `ecrecover(bytes32 hash, uint8 v, bytes32 r, bytes32 s) returns (address)` | EVM-specific operation; replaced by signing and verification mechanic described in the [ABI Spec](https://docs.ton.dev/86757ecb2/p/70c253). ercovery itself is unavailable in TON, as address cannot be derived from a public key (no relation between these entities) |
| **Fixed-size byte arrays**                                | `bytes1..32 bts`                                             | Partial support soon. **bytes32** pending release, testing under way. |
| **Control structures:**                                   |                                                              | The feature groups is partially supported, see details below. |
| **if, else**                                              | `if () { } else { }`                                         | Supported                                                    |
| **?:**                                                    | `bool isTrue = (a == 1) ? true : false`                      | Supported                                                    |
| **for, while**                                            | `for (uint p = 0; p < N; p++) { }`                           | Supported                                                    |
| **do while**                                              |                                                              | Planned                                                      |
| **break, continue**                                       |                                                              | Supported                                                    |
| **return**                                                |                                                              | Pending release                                              |
| **Parameters:**                                           |                                                              | The feature groups is partially supported                    |
| **int/uint**                                              |                                                              | Fully supported                                              |
| **arrays**                                                |                                                              | Partial support, only arrays of ints/addresses supported     |
| **structs**                                               |                                                              | Fully supported                                              |
| **address**                                               |                                                              | Fully supported                                              |
| **contract**                                              |                                                              | Supported as uint256                                         |
| **Error handling:**                                       |                                                              | In this group only **require** is relevant to the TVM. TVM handles errors itself |
| **assert**                                                | `assert(bool condition)`                                     | Not relevant for TVM                                         |
| **require**                                               | `require(bool condition<, string memory message>)`           | Supported                                                    |
| **revert**                                                | `revert(<string memory reason>)`                             | Not relevant for TVM                                         |
| **Visibility:**                                           |                                                              | The feature group is partially supported                     |
| **external**                                              |                                                              | Partial, can be called internally now                        |
| **public**                                                |                                                              | Supported                                                    |
| **internal**                                              |                                                              | Supported                                                    |
| **private**                                               |                                                              | Not supported. Can be called from children, depend on inheritance |
| **ABI:**                                                  |                                                              | Features in this group are mostly not supported, ABI operation mapping required |
| **encoding/decoding**                                     | `abi.encode(...) returns (bytes memory)``abi.decode(bytes memory encodedData (...)) returns (...)` | Pending release                                              |
| **packed encoding**                                       | `abi.encodePacked(...) returns (bytes memory)`               | Not supported                                                |
| **encoding with selector**                                | `abi.encodeWithSignature(string memory signature, ...) returns (bytes memory)` | Not supported                                                |
| **encoding with hash code**                               |                                                              | Not supported                                                |
| **Operations involving LValues**                          | `+=`,` -=`,` *=`,` /=`, `%=`,` |=`, `&=`, `^=`, `a++`,` a—`, `++a`,` —a` | Only operations with integers are supported. More research and development needed for other cases. |
| **Delete**                                                | Assigns initial value. Handled in `ExpressionCompiler?`      | Supported                                                    |
| **Scoping and declarations**                              |                                                              | Partial support                                              |

## Unsupported/Irrelevant

Supporting features and feature groups from this category has low priority or is out of the project scope, mainly because they are EVM-specific. 

In some cases we suggest a workaround. 

Depending on the project progress, we may decide to provide support for some of these at later stages. 

| **NAME**                                                   | **NOTES/USAGE**                                              | **MORE**                                                     |
| ---------------------------------------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| **Inline, Solidity, Standalone Assembly**                  | Ethereum-specific feature                                    |  |
| **SMTChecker**                                             | pragma experimental SMTChecker                               |                                                              |
| **Natspec**                                                | ///, @title, @author, @dev, @param, @return, @notice         | Currently not used in the TVM. Later support may be considered. Refer to the [original documentation](https://solidity.readthedocs.io/en/v0.5.11/natspec-format.html) for more details on the feature. |
| **Call Functions**                                         |                                                              | Not supported for security reasons. See also the Reference guide. |
| **Overloading**                                            | `function f(uint _in) ``function f(uint _in, bool _b)`       | Low priority, support may be considered in the future        |
| **Overload resolution and argument matching**              | `function f(uint8 _in) ``function f(uint256 _in)`            | Low priority, support may be considered in the future        |
| **Reference types**                                        |                                                              | Reference types are opposite of fixed types. They can be deployed after TON TVM memory features are supported completely. No workarounds or ETA at the moment. |
| **Function types**                                         |                                                              | Function to type conversion not supported. Low priority feature |
| **Tuples**                                                 |                                                              | Generally not supported; limited support in  in Return statements. More testing needed. |
| **Dynamically-sized byte arrays:**                         |                                                              | The feature groups is not currently supported.               |
| **byte**                                                   |                                                              | Can be supported after fixed size is deployed and stable.    |
| **string**                                                 | `string`                                                     | A string encoded in UTF-8 are not supported. Strings can not be implemented before data storage on TON TVM is completely supported. At this point, we do not support data storage completely. The suggested workaround is to use short fixed-size byte arrays once released. |
| **String literals and Types**                              | "foo", 'bar'                                                 | To be deployed in the framework of literals support; encode as byte arrays |
| **Special chars**                                          | `\<newline>`, `\\`,` \'` ,`\"` ,`\b `,`\f` ,`\n,` `\r`, `\t`, `\v`, `\xNN`, `\uNNNN` | To be deployed in the framework of literals support          |
| **Fixed point numbers, operations**                        | fixed/ufixed, fixed/ufixedMxN, <=, <, ==, !=, >=, >, +, -, unary -, *, /, % | Currently the feature is not supported. It is planned to implement mapping connecting text representation of an instruction to its numerical counterpart. On the total, there are 476 instructions. The map will be used for text assembly generation. |
| **Ether units**                                            | wei, finney, szabo, ether                                    | Not supported, EVM-specific                                  |
| **Time units**                                             | seconds, minutes, hours, days, weeks                         | Not supported now. |
| **Type information:**                                      |                                                              | No plans to support this whole group yet. Usage potential unclear. |
| **type**                                                   | `type(c)`                                                    |                                                              |
| **name**                                                   |                                                              |                                                              |
| **creationCode**                                           |                                                              |                                                              |
| **runtimeCode**                                            |                                                              |                                                              |
| **Explicit conversions**                                   |                                                              | Not supported or very limited support for address types. Cannot be implemented before fallback. |
| **Implicit conversions**                                   |                                                              | Not supported, requires inheritance. Will not be implemented before explicit conversions. |
| **Constant state variables**                               | `constant`                                                   | Not supported now, research needed                           |
| **Getter functions**                                       | `getArray`                                                   | Not supported now, research needed                           |
| **Function modifiers**                                     |                                                              | Not supported                                                |
| **Exceptions**                                             |                                                              | Not supported now, research needed                           |
| **Inheriting Different Kinds of Members of the Same Name** |                                                              | Inheritance-dependent (see above).                           |
| **Call protection for libraries**                          |                                                              | Not supported now                                            |
| **Copy of operations**                                     |                                                              | No plans to support                                          |
| **Libraries**                                              | `library Set { ... }`                                        |                                                              |
| **Creating contracts via new**                             |                                                              |                                                              |




