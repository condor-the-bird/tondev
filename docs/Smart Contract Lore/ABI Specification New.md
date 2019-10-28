# ABI V1

## Abstract

This specification determines message body encoding for proper communication with TON blockchain smart contracts.

## New in ABI v1

ABI v.1 is intended to minimize gas consumption by contracts for encoding/decoding operations.

We’ve redesigned types for array encoding and introduced TON blockchain-specific types supported by TVM.

ABI v.1 complies with TON best practices for smart contracts  (see https://test.ton.org/smc-guidelines.txt).

## Motivation

Currently, ABI encoding is well-optimized, but decoding it consumes a lot of gas from contracts. For example, TON Labs compilers store arrays as dictionaries (Hashmap) in contracts, and every element of a tightly encoded ABI array must be parsed and stored in the dictionary. The encoding and decoding algorithm is written manually in TVM Assembly and is non-optimizable.

Another addressed issue is dynamic typing. For example, the `T[]` array type has 2 variants of encoding, and contracts must support both increasing the contract code size.

Finally, ABI v. 0 restricted the encoding of some essential TVM-supported types, in particular: hashmaps, MsgAddress, and the raw tree of cells.

## Message body

### External Inbound Message Body Structure

Message body with encoded function call has the following format:

```
Function ID`+ `Timestamp` + `Enc(Arguments)
```

The first ***32 bits\*** of the message body identify which contract functions are called. The function ID comes within the first 32 bits of the SHA256 hash of the function signature. The highest bit is set to 0 for function ID in external inbound messages, and 1 for external outbound messages.

The next ***64 bits\*** contain the message timestamp in milliseconds (only for external messages).

`Function ID` and `Timestamp` are stored in the message body root cell.

The function parameters are next. They are encoded in compliance with the present specification and stored either to the root cell or the next one in the chain.

**Note**: an encoded parameter can't be split between different cells.

### External Outbound Message Body Structure

External outbound messages are used to return values from functions or emit events.

Return values are encoded and put into the message response:

```
Function ID`+`Enc(Return values)
```

Function ID's highest bit is set to 1.

Events are encoded as follows:

```
Event ID + Enc(event args)
```

`Event ID` - 32 bits of SHA256 hash of the event function signature with highest bit set to 0.

### Internal Message Body Structure

Internal messages are used to communicate between contracts. The body has the following format:

```
Function ID`+`Enc(Arguments)
```

`Function ID` - 32 bits function id calculated as first 32 bits SHA256 hash of function signature. The highest bit of function ID is 0 for requests and 1 for answers.

## Message Body Signing

The message body can be protected with a cryptographic signature to identify a user outside the blockchain. In such a case, an *External inbound message* that calls the function carries a user *private key* signature. This requirement applies only to *External inbound messages* because *Internal inbound messages* are generated within the blockchain, and *src address* can be used to identify the caller. **The signature is stored in a separate cell referenced by *index 0* from the message body root cell.

If a user does not want to sign a message, an empty signature cell must be attached to the message body at reference *index 0* of the body root cell.

The message body signature is generated from the *representation hash* of the root cell before the reference to the signature cell is appended to it.

### Signing Algorithm

1. ABI serialization generates the message body as a bag of cells. One empty reference is reserved at the root cell. At this stage, the signature cell is not yet created.
2. *Representation hash* of the message body is signed using the *Ed25519* algorithm.
3. The signature is saved to the new cell. The public key is also placed in this cell after the signature data.
4. The Signature cell is added to the body root cell as a reference at position 0, shifting existing references to the next positions.

## Function Signature (Function ID)

The following syntax is used to define a signature:

- function name
- list of input parameter types (input list) in parenthesis, possibly prefixed by optional keyword `time`
- list of return values types (output list) in parenthesis
- ABI version

A single comma is used to divide each input parameter and return value type from one another, no spaces used.

Parameter and return value names are not included.

The function name, input, and output lists follow each other immediately without separation.

If a function has no input parameters or does not return any values, the corresponding input or output lists are empty (empty parenthesis).

### Function Signature Syntax

```
signature = "function_name([time,]type1,type2,...,typeN)(out_type1,out_type2,...,out_typeM)v1"
FunctionID = SHA256(signature)
```

### Encoding

The goal of the present specification is to design ABI types in a cheap to read way for the contract to reduce gas consumption. Some types are optimized for storing without write access.

### ABI types:

- `uint`: unsigned `M` bit integer. Big-endian encoded unsigned integer stored in the cell-data.

- `int`: two’s complement signed `M` bit integer. Big-endian encoded signed integer stored in the cell-data.

- `bool`: equivalent to uint1.

- tuple `(T1, T2, ..., Tn)`: tuple that includes `T1`, ..., `Tn`, `n>=0` types encoded in the following way:

  Enc(X(1)) Enc(X(2)) . . ., Enc(X(n)); where X(i) is type of T(i) for i in 1..n

- `T[]` - a dynamic array of `T` type elements. Encoded as a TVM dictionary.  `uint32` defines the array elements count placed to the cell body. Then `HashmapE` (see TL-B schema in TVM spec) struct is added (one bit as a dictionary root and one reference with data if the dictionary is not empty). The key of the dictionary is a serialized `uint32` index of an array element, and the value is a serialized array element as `T` type.

- `T[k]` a static size array of `T` type elements. Encoding is equivalent to `T[]`

- `bytes`: an array of `uint8` type elements. Array is put into a separate cell. In the case of array overflow, the maximum cell-data size it's split into multiple sequential cells.

  - Remark: contract stores this type as-is without parsing. For high-speed decoding, cut reference from body slice as `LDREF`. This type is helpful if some raw data must be stored in the contract without write or random access to elements.
  - Remark: analog of `bytes` in Solidity. In C lang can be used as `void*`.

- `fixedbytes`: a fixed-size array of `M` `uint8` type elements. Encoding is equivalent to `bytes`

- `map(K,V)`:  a dictionary of `V` type values with `K` type key. `K` may be any of `int/uint` types with `M` from `1` to `1023`. Dictionary is encoded as `uint10` indicating key length followed by `HashmapE` type (one bit put into cell data as a dictionary root and one reference with data is added if the dictionary is not empty).

- `address`: account address in TON blockchain. Encoded as `MsgAddress` struct (see TL-B schema in TON blockchain spec).

- `cell`: a type for defining a raw tree of cells. Stored as a reference in the current cell. Must be decoded with command `LDREF` and stored as-is.

  - Remark: the type is useful to store some payload as a tree of cells like contract code and data in the form of `StateInit` structure of `message` structure.

## Cell Data Overflow

In case of parameter data not fitting into the available space of the current cell-data, it moves to a separate new cell. This cell attaches to the current one as reference. The new cell becomes a current cell.

## Cell Reference Limit Overflow

For simplicity, this ABI version reserves the last cell-reference spot for cell-data overflow. If the cell-reference limit in the current cell is already reached (save for the reserved spot) and a new cell is required, the current cell is considered complete, and a new one generates. The reserved spot stores the reference to the new cell, and it continues with the new cell as a current one.

## Contract Interface Specification

The contract interface is stored as a JSON file called contract ABI. It includes all public functions, and data is described using ABI types. ABI file has the following structure:

```javascript
{
	"ABI version": "1",
	"functions": [
		...		
	],
	"data": [
		...
	],
	"events": [
		...	
	]
}
```

### **Functions** section

'Functions' section specifies each interface function signature, including its name, input, and output parameters. Functions specified in the contract interface can be called from other contracts or from outside the blockchain via ABI call.

**Functions** section has the following fields:

```javascript
"functions": [
	{ 		
		"name": "method_name",
		"inputs": [{"name": "func_name", "type": "ABI_type"}, ..],
		"outputs": [...],
		"id": "0xXXXXXXXX", //optional
	},
	...
]
```

- `name`: function name;
- `inputs`: an array of objects, each containing: 
  - `name`: parameter name;
  - `type`: the canonical parameter type.
  - `components`: used for tuple types, optional.
  - `id`: an optional `uint32` `id` parameter can be added. This `id` will be used as a `Function ID` instead of automatically calculated. PS: last case can be used for contracts that are not ABI-compatible.
- `outputs`: an array of objects similar to `inputs`. It can be omitted if the function does not return anything;

### Events section

This section describes events used in contract. Event is an external outbound message with ABI-encoded parameters in the body.

```javascript
"events": [
	{ 		
		"name": "event_name",
		"inputs": [...],
		"outputs": [] 
	},
	...
]
```

`inputs` and `outputs` have the same format as for functions.

### Data section

This section describes contract's global public variables.

```javascript
"data": [
	{ 		
		"name": "var_name",
		"type": "abi_type",
		"key": "<number>" // index of variable in contract data dictionary
	},
	...
]
```

