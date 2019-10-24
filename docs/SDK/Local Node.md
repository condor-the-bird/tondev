# Overview

TON Labs Local Node is a pre-configured Docker image with a simplified standalone node server instance designed only for debugging and testing.

> **Note:** TON Labs Local Node (Node SE) is not designed to interact with the regular TON network.

# Node SE Test Giver

> Note that all references to Gram in this document or other documents in this documentation imply test Grams, not the real ones.

## Configuration

If you are using the local node (Node SE) for your projects, you can use its pre-deployed Giver to transfer test Grams, deploy and run other contracts. On the start Node SE giver has 1.5 billion test Grams when you first create the Node container. 

In the course of usage, the balance dries, so to restore it, you have to recreate the container with tondev CLI `recreate` command 

To access Node SE pre-deployed giver, use this address and ABI in your projects:

```javascript
const nodeSeGiverAddress = 'a46af093b38fcae390e9af5104a93e22e82c29bcb35bf88160e4478417028884';
const nodeSeGiverAbi = {
	"ABI version": 1,
	"functions": [
		{
			"name": "constructor",
			"inputs": [
			],
			"outputs": [
			]
		},
		{
			"name": "sendGrams",
			"inputs": [
				{"name":"dest","type":"uint256"},
				{"name":"amount","type":"uint64"}
			],
			"outputs": [
			]
		}
	],
	"events": [
	],
	"data": [
	]
};
```

## Usage example

Check it on our ton-client-js tests.

Declare `get_grams_from_giver` function, and then invoke it to transfer test Grams to an account that you need. 

```javascript
async function get_grams_from_giver(account) {
    const { contracts, queries } = tests.client;

        const result = await contracts.run({
            address: nodeSeGiverAddress,
            functionName: 'sendGrams',
            abi: nodeSeGiverAbi,
            input: {
                dest: `0x${account}`,
                amount: 500000000
            },
            keyPair: null,
        });
    }
```

For your information we demonstrate the Giver Solidity code here:

```javascript
pragma solidity >=0.5.0 <0.6.0;

contract Giver {
    constructor() public {}

    function sendGrams(address payable dest, uint64 amount) public {
        require(address(this).balance > amount, 60);
        dest.transfer(amount);
    }
}

```

