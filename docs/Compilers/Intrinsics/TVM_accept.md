# Gas acceptance

## Some Theory

Anyone can send external message to your contract. When a message arrives, the contract initial gas limit is equal to 10_000 units of credit gas that should be bought later by the ACCEPT tvm primitive. Otherwise when credit gas falls to zero, the TVM throws the `out of gas` exception. The contract is supposed to spend these 10_000 units of 'free' gas to check the body of an inbound message tp make sure that it is valid and can be processed by contract successfully. 

The idea of credit gas allowance is that as long as it is beyond zero, any exception thrown by contract  prevents all further gas charges. But once the contract accepts a message, all gas consumed by contract is converted to gas fees regardless of whether a transaction is aborted or not.

ACCEPT is useful in internal messages too. When another contract sends an internal message to your contract, initial gas limit is equal to an inbound message value divided by the gas_price or global gas limit, if it is smaller. If this value is not enough to finish execution, the contract then can increase its gas limit by calling ACCEPT or SETGASLIMIT primitive. The ACCEPT primitive increases  the limit to the value of its balance divided by the gas_price, and the SETGASLIMIT primitive sets the current gas limit to the value popped from the TVM stack (the value cannot be bigger than the **gm** limit). 

With the ACCEPT command a contract can choose whether gas for its execution is paid by the caller contract or  by the contract itself.

## Implementation

In TON Labs the ACCEPT primitive is implemented in Solidity as a private function called by public functions.

```javascript
contract MyContract {

	function tvm_accept() private pure {}
	
	modifier alwaysAccept {
		tvm_accept(); _;
	}
	
	uint64 m_result;

	function sendMoneyAndNumber(address remote, uint64 number) public alwaysAccept {
		IRemoteContract(remote).acceptMoneyAndNumber.value(3000000)(number);
		return;
	}
}
```

Find below actual usage examples. All can be compiled by TON Labs Solidity compiler.

### Accept gas inside function


```javascript
contract AcceptExample1 {
	uint _a = 0;

	function foo(uint a) public {
		require(a > 0, 100); //do checks
		tvm_accept();        //accept gas only after check
		_a = a;              //do other logic
	}

}
```

To avoid gas payment when the `foo` function is called by another contract, we can use the following code:

    function foo(uint a) public {
    		require(a > 0, 100); //do checks
    		if (msg.sender == 0) {
    			//accept gas only in case of external message.			
    			tvm_accept(); 
    		}
    		_a = a;              //do other logic
    	}

Remember that the caller contract should attach enough Grams to its message to cover all gas that will be spend by `foo` function.

### Accept gas inside modifier

```javascript
contract AcceptExample2 {
	uint _sum = 0;
	
	modifier AlwaysAccept() {
		tvm_accept(); _;
	}
	
	function foo(uint a, uint b) AlwaysAccept() public {
		_sum = a+ b;
	}

}
```

> **Important**: modifier is called before arguments are deserialized from inbound message body. In the example above `AlwaysAccept()` will be called before `a` and `b` are be loaded from the message body slice.