# Deploying contract from contract

TON blockchain features contract ability to create another contract. Deploy is done through sending an internal message including relevant code and data attached to it and a `bounce` flag set to 0. This internal message has optional `init` ****field which; if present, must be serialized as `StateInit` structure (see TL-B scheme in TON blockchain spec): `bounce` flag equal to 0.

```
_
	split_depth:(Maybe (## 5)) 
	special:(Maybe TickTock)
	code:(Maybe ^Cell) 
	data:(Maybe ^Cell)
	library:(Maybe ^Cell) 
= StateInit;
```

There are a few ways to make that happen. We will review the most optional ones in the next part of this document.

### Assumptions

Assume that contract A deploys contract B.

All of the examples are based on TON Labs Solidity compiler.

The compiler uses a special type called `TvmCell` to represent an arbitrary tree of cells.

Functions with `tvm_*` prefix are compiler intrinsics (predefined built-in functions). They should have empty bodies.

### Method 1

Store contract as `StateInit` structure in persistent data of another contract. Add `deploy` method that receives public key and constructor parameters for new contract:

```
contract A {
	struct TvmCell { uint _; }
	TvmCell _contract; // StateInit as tree of cells of contract B

	//Compiler intrinsics
	function tvm_insert_pubkey(
		TvmCell memory cellTree,
		uint256 pubkey
	) private pure returns(TvmCell memory /*contract*/)  {}

	function tvm_hashcu(TvmCell memory cellTree) pure private returns (uint256) { }

	function tvm_deploy_contract(
		TvmCell memory my_contract,
		address addr,
		uint128 gram,
		uint constuctor_id,
		uint32 constuctor_param0,
		uint8 constuctor_param1) pure private { }

	function tvm_deploy_contract(
		TvmCell memory my_contract,
		address addr,
		uint128 grams,
		TvmCell memory payload
	) pure private { }

	function tvm_make_address(
		int8 wid,
		uint256 addr
	) private pure returns (address payable) {}

	//constructor
	constructor(TvmCell memory contract) public {
		_contract = contract;
	}


	function deploy(
		uint256 pubkey,
		uint128 gram,
		uint constuctor_id,
		uint32 constuctor_param0,
		uint8 constuctor_param1
	) public returns (address) {
		//insert new public key into initiala data of the contract B
		TvmCell memory contract = tvm_insert_pubkey(_contract, pubkey);
		//calculate the address of contract B
		address addr = tvm_make_address(0, tvm_hashcu(contr));
		//deploy contract B: this intrinsic creates internal message and sets
		//msg.header.value = gram, msg.header.dest = addr. Also it encodes
	  //constructor call in msg.body.
		tvm_deploy_contract(
			contract, 
			addr, //destination address of the message
			gram, //value of grams transferred with message
			constuctor_id, //constructor function id
			//constructor parameters; specify additional parameters if need, also you can
			//use zero parameters.
			constuctor_param0,
			constuctor_param1
	  ); 
		return addr;
	}
}
```

### Method 2

Store only contract code in persistent data of contract A. And write a `deploy` function that receives  initial data of contract B as tree of cells.

```
contract A {
	struct TvmCell { uint _; }
	TvmCell _code; //code of contract B

	//Compiler intrinsics tvm_*
	function tvm_build_state_init(
		TvmCell memory code,
	  TvmCell memory data
	) private pure returns (TvmCell memory cell) { }

	function tvm_deploy_contract(
		TvmCell memory my_contract, 
		address addr,
		uint128 gram,
		uint constuctor_id		
	) pure private { }

	function tvm_make_address(
		int8 wid,
		uint256 addr
	) private pure returns (address payable) {}

	// initialize contract with code of contract B
	constructor(TvmCell memory code) public {
		_code = code;
	}

	function deploy(
		TvmCell memory data, //initial data of contract B
		uint128 grams,
		uint constuctor_id
	) public returns (address) {
		//build StateInit structure from code and data
		TvmCell memory contr = tvm_build_state_init(_code, data);
		//calculate address of contract B
		address addr = tvm_make_address(0, tvm_hashcu(contr));
		//deploy contract B
		tvm_deploy_contract(
			contr,
			addr,
			grams,
			constuctor_id
		); 
		//return address of contract B		
		return addr;
	}
}
```

### Method 3

Store code and data as in method 1 or 2, but define a payload that will be used as body of internal message to contract B.

```javascript
contract A {
	struct TvmCell { uint _; }
	TvmCell _code; //code of contract B

	//Compiler intrinsics
	function tvm_build_state_init(
		TvmCell memory code,
	  TvmCell memory data
	) private pure returns (TvmCell memory cell) { }

	function tvm_deploy_contract(
		TvmCell memory contract,
		address addr,
		uint128 grams,
		TvmCell memory payload
	) pure private { }

	function tvm_make_address(
		int8 wid,
		uint256 addr
	) private pure returns (address payable) {}

	constructor(TvmCell memory code) public {
		_code = code;
	}

	function deploy(
		TvmCell memory data, //initial data of contract B
		uint128 grams,
		TvmCell memory payload // payload is used as a body of internal message
	) public returns (address) {
		//build StateInit structure from code and data
		TvmCell memory contr = tvm_build_state_init(_code, data);
		//calculate address of contract B
		address addr = tvm_make_address(0, tvm_hashcu(contr));
		//deploy contract
		tvm_deploy_contract(contr, addr, grams, payload);
		return addr;
	}

}
```