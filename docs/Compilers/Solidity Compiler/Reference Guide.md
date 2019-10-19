## Decrements

The current compiler version does not support Solidity decrements operation.

Don't use expressions like:

```javascript
b--;
```

Use instead:

```javascript
b - 1;
```

## String Literals

The current compiler version does not support Solidity string literal. Avoid expressions like:

```javascript
bytes32 b = "foo";   
```

And:

```javascript
string memory b = "foo";  
```

## Unfixed Decimals

Unfixed decimals are not currently supported by the compiler, avoid expressions like:

```javascript
ufixed b = a * 2.5;
```

## Conditional Loops

The compiler supports *for* and *while* conditional loops, but does not support *do .. while..* expressions.

Don't use:

```javascript
do {
        i++;
 } 
while (i < x);
```

Use instead:

```javascript
i++;
while (i < x) {
     i++;
}
```

## Comma Operator

The comma operator is not yet supported by the compiler, use the suggested workaround.

Don't use expressions like:

```javascript
a = (b++, c++);
```

Use instead:

```js
b++;
a = c++;                  
```

All in all, this operator is not used often, but you may need for an expression like:

```javascript
void rev(char *s, size_t len) 
{ 
            char *first; 
            for (first = s, s += len; s >= first; --s) { 
                    putchar(*s); 
            } 
}
```

## Function Modifiers

Currently, modifiers are not supported by the sol2tmv compiler, though it is a planned feature and we have a workaround.

Modifiers must be called directly as functions. The modifier should be defined as a pure or view internal function and should compile correctly.

### Example

```javascript
//original solidity
contract Test {
    address owner;

    modifier onlyOwner() {
        require(msg.sender == owner);
        _;
    }

    function doSmthg() public onlyOwner {

    }
}

//sol2tvm
contract Test {
    address owner;

    function onlyOwner() view internal {
        require(msg.sender == owner);
    }

    function doSmthg() public {
        onlyOwner();
    }
}
```

## Assembly

Inline, Solidity and Standalone assembly implementations are not used outside the Ethereum VM. TON Labs sol2tvm compiler uses its own equivalent of [inline Assembly](https://solidity.readthedocs.io/en/v0.4.24/assembly.html), other versions are not supported and cannot be compiled.

Assembly is used sparingly in smart contracts and there is no hard and fast way to provide a workaround. If you have to migrate a smart contract containing assembly code, remove it or rewrite it in Solidity. 

Below we have some standard conversion samples taken from public sources.

### Sample 1

**Assembly**

```assembly
mul(1, add(2, 3)) 
```

**Solidity**

```javascript
uint a = (2+3) * 1;                      
```

### Sample 2

**Assembly**

```assembly
function add(uint a, uint b){ assembly { let sum := add(a, b) mstore(mload(0x40), sum) }}
```

​                       

**Solidity**

```javascript
function add(uint a, uint b) { uint sum = a + b;}
```

## Call Functions

These functions are best described in Solidity documentation here: <https://solidity.readthedocs.io/en/v0.5.11/types.html#address> 

**Call, delegatecall and staticcall** are used to interface with contracts that do not follow the ABI, or to get more direct control over the encoding. 

We are focused on providing the highest level of security in our products.  So, we feel that interacting with contracts that do not have an ABI public description is not in line with what we want to achieve. Therefore, we do not support or have any plans to support these features or to create an analogue. Workarounds have not been specified yet.

## Return and Callback

**Return** works as expected only for internal contract calls according to TON architecture. It is not possible to use **return** to receive a message back from external contract.

By design, in TON you need to use **callback** as a separate function in the second contract that sends a message back.

If contract A calls contract B and requires a return of value, a **callback** should be specified in contract A to receive the answer.

Example:

```javascript
contract A { 
 function a() { 
   B.b(); //make request
 } 
 function c() { 
 //this is callback
  }
}
contract B { 
 function b() { 
     A.c() //send answer }
}
```

