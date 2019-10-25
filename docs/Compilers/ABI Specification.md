## Message Body 

### Structure

The **first 8 bits (1 byte)** of the message body represent the contract ABI specification version.

Next **32 bits** of the message body identify which contract functions is called. The function ID comes within the first bits of the SHA256 hash of the function signature.

The ABI version and the function ID are stored in the message body root cell.

Then follow the function parameters. They are encoded in compliance with the present specification and stored either to the root cell or the next one in the chain.

A serialized function call has the following format:

```
Version` + `Function ID`+ `Enc(Arguments)
```

Function return values are also encoded and put into the message response:

```
Version` + `Function ID`+`Enc(Return values)
```

###  Signing

To identify user outside the blockchain the message body can be protected with a cryptographic signature. Then an *External inbound message* that calls the function is signed by the user *private key*. This requirement applies only to *External inbound messages* because *Internal inbound messages* are generated within the blockchain and *src address* can be used to identify the caller. **The signature is stored to a separate cell referenced by *index 0* from the message body root cell.

If user doesn't want to sign message, empty signature cell must be attached to message body at reference *index 0* of the body root cell.

The message body signature is generated from the *representation hash* of the root cell before the reference to signature cell is appended to it.

**Signing Algorithm**

1. ABI serialization generates the message body as a bag of cells. One empty reference is reserved at the root cell. At this stage the signature cell is not yet created.
2. *Representation hash* of the message body is signed using the *Ed25519* algorithm.
3. The signature is saved to the new cell. Public key is also placed to this cell after signature data.
4. The Signature cell is added to the body root cell as reference at position 0 shifting existing references to the next positions.

## Function Signature 

To define a signature (Function ID) the following syntax is used:

- function name
- list of input parameter types (input list) in parenthesis
- list of return values types (output list) in parenthesis

Single comma is used to divide each input parameter and return value type from one another. No spaces are used.

Parameter and return value names are not included.

The function name, input and output lists are not separated and immediately follow each other.

If a function has no input parameters or does not return any values, the corresponding input or output lists are empty (empty parenthesis).

### Function Signature Syntax

```
function_name(input_type1,input_type2,...,input_typeN)(output_type1,output_type2,...,output_typeM)
```

### Signature Calculation Syntax

```
SHA256("function_name(input_type1,input_type2,...,input_typeN)(output_type1,output_type2,...,output_typeM)")
```

### Sample Implementation

**Function**

```
func(int64 param1, bool param2) -> uint32
```

**Function Signature**

```
func(int64,bool)(uint32)
```

**Function Hash**

```
sha256("func(int64,bool)(uint32)") = 0xA6BF22A36FD64ADD2A0EE012A2E137834E0609D24499A2CBFBACC8187C0D288F
```

And the **function ID** then is `0xA6BF22A3`

## Types

- `uint<M>`: unsigned `M` bit integer. Big-endian encoded unsigned integer stored in the cell-data.
- `int<M>`: two’s complement signed `M` bit integer. Big-endian encoded signed integer stored in the cell-data.
- `dint`: signed integer value with dynamic size. Encoded as `Base 128 Varints` proposed by Google in [Protocol Buffers documentation](https://developers.google.com/protocol-buffers/docs/encoding#varints). See further for more information.
- `duint`: two’s complement dynamic sized usigned integer value. Encoded as `Base 128 Varints` proposed by Google in [Protocol Buffers documentation](https://developers.google.com/protocol-buffers/docs/encoding#varints). See further for more information.
- `bool`: equivalent to uint1.
- tuple `(T1, T2, ..., Tn)`: tuple that includes `T1`, ..., `Tn`, `n>=0` types encoded in the following way:
- Enc(X(1)) Enc(X(2)) . . ., Enc(X(n)); where X(i) is type of T(i) for i in 1..n
- `T[]`: dynamic array of `T` type elements encoded in the following way:
- 2 bits stored in the cell-data define serialization algorithm.
- [00] - array is put into a separate cell. In case of array overflowing the maximum cell-data size it's split into multiple sequential cells.
- [10] - bits sequence is put in the cell-data. The next 8 bits (1 byte) define the number of elements of the array. It's followed by the sequence of encoded array elements.
- [01]/[11] - reserved.
- `T[k]`: static size array of `T` type elements encoded in the following way:
- 2 bits put in the cell-data define serialization algorithm.
- [00] - array is put into a separate cell. In case of array overflowing the maximum cell-data size it's split into multiple sequential cells.
- [10] - bits sequence is put in the cell-data. It's followed by the sequence of encoded array elements.
- [01]/[11] - reserved.
- `bits<M>`: static sized bits sequence. Encoding is equivalent to `bool[M]`
- `bitsring`: dynamic sized bits sequence. Encoding is equivalent to `bool[]`

## Base 128 for Varints 

*Varints* are used to serialize integers having one or more bytes (smaller numbers take a smaller number of bytes).

Each byte in a *varint* except the last one has the *most significant bit* (msb) set; it indicates that there are more bytes to come. The lower 7 bits in each byte store the two's complement representation of the number in groups of 7 bits; **least significant group first**.

For example, here is the number 1, it is a single byte, so no msb is set:











```
0000 0001
```







Syntax

​                            C, C++, C#                                         CoffeeScript                                         CSS                                         Dockerfile                                         diff                                         Go                                         Groovy                                         HAML                                         Handlebars                                         HTML templates (Rails, ASP.NET)                                         HTML                                         HTTP                                         Java                                         JavaScript                                         JSX (React)                                         LESS                                         Objective C                                         Perl                                         PHP                                         Pug                                         Puppet                                         Python                                         Ruby                                         Rust                                         Sass                                         Scala                                         SCSS                                         Shell                                         SQL                                         Swift                                         Vue.js                                         XML                                         YAML                     HTML   

In this example we handle the number 300, it is more complicated:











```
1010 1100 0000 0010
```







Syntax

​                            C, C++, C#                                         CoffeeScript                                         CSS                                         Dockerfile                                         diff                                         Go                                         Groovy                                         HAML                                         Handlebars                                         HTML templates (Rails, ASP.NET)                                         HTML                                         HTTP                                         Java                                         JavaScript                                         JSX (React)                                         LESS                                         Objective C                                         Perl                                         PHP                                         Pug                                         Puppet                                         Python                                         Ruby                                         Rust                                         Sass                                         Scala                                         SCSS                                         Shell                                         SQL                                         Swift                                         Vue.js                                         XML                                         YAML                     HTML   

Deserializing this number from the above binary string starts from dropping the msb from each byte including the byte with msb set to 0 (it is the set in the first byte, as there is more than one byte in the *varint*):











```
1010 1100 0000 0010
→ 010 1100  000 0010
```







Syntax

​                            C, C++, C#                                         CoffeeScript                                         CSS                                         Dockerfile                                         diff                                         Go                                         Groovy                                         HAML                                         Handlebars                                         HTML templates (Rails, ASP.NET)                                         HTML                                         HTTP                                         Java                                         JavaScript                                         JSX (React)                                         LESS                                         Objective C                                         Perl                                         PHP                                         Pug                                         Puppet                                         Python                                         Ruby                                         Rust                                         Sass                                         Scala                                         SCSS                                         Shell                                         SQL                                         Swift                                         Vue.js                                         XML                                         YAML                     HTML   

Then, to get the final value, the two groups of 7 bits are put in the reverse order and concatenated:











```
000 0010  010 1100
→  000 0010 ++ 010 1100
→  100101100
→  256 + 32 + 8 + 4 = 300
```







Syntax

​                            C, C++, C#                                         CoffeeScript                                         CSS                                         Dockerfile                                         diff                                         Go                                         Groovy                                         HAML                                         Handlebars                                         HTML templates (Rails, ASP.NET)                                         HTML                                         HTTP                                         Java                                         JavaScript                                         JSX (React)                                         LESS                                         Objective C                                         Perl                                         PHP                                         Pug                                         Puppet                                         Python                                         Ruby                                         Rust                                         Sass                                         Scala                                         SCSS                                         Shell                                         SQL                                         Swift                                         Vue.js                                         XML                                         YAML                     HTML   

## Overflow

### Cell Data 

If the data does not fit into the available space of the current cell-data, it is split for maximum utilization of space in the current cell. A reference to a new cell is added to the current one and it continues with the remaining part, and a new cell as a current.

### Cell Reference Limit 

For the simplicity, this ABI version reserves the last cell-reference spot for the cell-data overflow case. If the cell-reference limit in the current cell is already reached (save for the reserved spot) and a new cell is required, the current cell is considered complete and a new one is created. The reference to a new cell is stored in the reserved spot and it continues with the new cell as a current one.

## Contract Interface 

Contract interface functions are specified in JSON format. The contract JSON description specified each interface function signature, including its name, input and output parameters. Functions specified in the contract interface can be called from other contracts or from outside the blockchain via ABI call.

A function description is a JSON object with the following fields:

- `name`: function name;
- `inputs`: an array of objects, each containing:
- `name`: parameter name;
- `type`: the canonical parameter type.
- `components`: used for tuple types, optional.
- `outputs`: an array of objects similar to `inputs`, can be omitted if function does not return anything;
- `signed`: boolean indicating the need for message signing

The list of interface function descriptions is included to contract interface specification as the `functions` field. The interface specification also specifies an ABI encoding format version:

```
"ABI version" : x
```

The whole JSON contract interface consists of the ABI version field and list of interface function descriptions. A contract specification example is provided below.

## Modification Support

As specified above, the JSON contract interface description contains versions of the ABI encoding format and the contract interface. So, a JSON description in a contract defines the used version of ABI encoding algorithm and its own interface version.

Having this data, a caller determines how to encode parameters for the current contract version. The first byte of encoded function parameters is reserved for the ABI format version, enabling the deserializer to check whether the caller uses the same version and parameters are encoded correctly.

## Rationale

The present serialization method is selected to reduce messages represented as a tree of cells according to TON blockchain specification. Simple types are encoded "in place", i.e. encoded data is stored to the current cell data. For composite types with varying size ranges two serialization methods are provided: the same encoding "in place" and encoding in a chain of separate cells. The method selection takes place during the run time and depends on the array size. Thus, small arrays fitting into a 8 bit field are stored in the current cell, while larger ones that take several cells are stored into a chain of separate cells to simplify access to parameters that come next. Two bits are reserved for array encoding type definition, so that other two encoding types could be defined.

### Sample Code

Contract interface specification example:

```js
{
	"ABI version" : 0,

	"functions" :	[
	    {
	        "inputs": [
	            {
	                "name": "recipient",
	                "type": "bits256"
	            },
	            {
	                "name": "value",
	                "type": "duint"
	            }
	        ],
	        "name": "sendTransaction",
					"signed": true,
	        "outputs": [
	            {
	                "name": "transaction",
	                "type": "uint64"
	            },
							{
	                "name": "error",
	                "type": "int8"
	            }
	        ]
	    },
	    {
	        "inputs": [
						  {
	                "name": "type",
	                "type": "uint8"
	            },
							{
	                "name": "value",
	                "type": "duint"
	            },
							{
	                "name": "meta",
	                "type": "bitstring"
	            }
					],
	        "name": "createLimit",
					"signed": true,
	        "outputs": [
							{
	                "name": "limitId",
	                "type": "uint8"
	            },
							{
	                "name": "error",
	                "type": "int8"
	            }
	        ]
	    },
	    {
	        "inputs": [
							{
	                "name": "limitId",
	                "type": "uint8"
	            },
							{
	                "name": "value",
	                "type": "duint"
	            },
							{
	                "name": "meta",
	                "type": "bitstring"
	            }
	        ],
	        "name": "changeLimitById",
					"signed": true,
	        "outputs": [
							{
	                "name": "error",
	                "type": "int8"
	            }
	        ]
	    },
			{
	        "inputs": [
							{
	                "name": "limitId",
	                "type": "uint8"
	            }
	        ],
	        "name": "removeLimit",
					"signed": true,
	        "outputs": [
							{
	                "name": "error",
	                "type": "int8"
	            }
	        ]
	    },
			{
	        "inputs": [
							{
	                "name": "limitId",
	                "type": "uint8"
	            }
	        ],
	        "name": "getLimitById",
	        "outputs": [
							{
									"name": "limitInfo",
					        "type": "tuple",
					        "components": [
											{
					                "name": "value",
					                "type": "duint"
					            },
											{
					                "name": "type",
					                "type": "uint8"
					            },
											{
					                "name": "meta",
					                "type": "bitstring"
					            }
									]
							},
							{
	                "name": "error",
	                "type": "int8"
	            }
	        ]
	    },
			{
	        "inputs": [],
	        "name": "getLimits",
	        "outputs": [
							{
									"name": "list",
					        "type": "uint8[]",
							},
							{
	                "name": "error",
	                "type": "int8"
	            }
	        ]
	    },
			{
	        "inputs": [],
	        "name": "getVersion",
	        "outputs": [
							{
									"name": "version",
					        "type": "tuple",
					        "components": [
											{
					                "name": "major",
					                "type": "uint16"
					            },
											{
					                "name": "minor",
					                "type": "uint16"
					            }
									]
							},
							{
	                "name": "error",
	                "type": "int8"
	            }
	        ]
	    },
			{
	        "inputs": [],
	        "name": "getBalance",
	        "outputs": [
							{
	                "name": "balance",
	                "type": "uint64"
	            }
	        ]
	    }
	]
}
```
