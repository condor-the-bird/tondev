## About the ABI

Despite the fact that each TON contract is actually a single function, the TON SDK allows defining multiple functions within it. To achieve it, the original multi-functional contract is compiled into a single function one with an incoming message dispatcher.

For correct encoding incoming messages to these contracts and decoding output ones, the SDK Library uses an ABI: a structural description of input/output messages related to contract functions.

## Compiling to TVC

With the SDK compiler you can:

- obtain TVM-ready code from Solidity sources.
- obtain TVM-ready code from our LLVM-based compiler that can potentially take source code in various general purpose languages. Now we have an implementation of the C language.

The [Toolchain documentation](https://docs.ton.dev/86757ecb2/p/09bb3d) contains all information on TON Labs Toolchain.

The Compiler Kit is shipped as a Docker container with pre-configured tools ready to work.

## Deploying contracts

The Contracts module of the TON Labs Client Library allows you to deploy a compiled contract to the blockchain. 

To deploy a contract, you need TVM-ready code (*.tvc file), an ABI (*.abi.json file) and a key pair.

> **Note**: a contract cannot be deployed before it has some amount of Grams on its balance. All attempts to deploy a contract with a zero balance will fail.

So, before deploying a contract, run another one to transfer Grams to the contract you plan to deploy. Make sure that the transferred amount covers all deployment fees and costs.

Or, in case you are using Node SE instance, use the pre-deployed Giver for it.

Add this code to your index.js file:

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
async function get_grams_from_giver(client, account) {
    const { contracts, queries } = client;
    const result = await contracts.run({
        address: giverAddress,
        functionName: 'sendGrams',
        abi: giverAbi,
        input: {
            dest: `0x${account}`,
            amount: 10000000000
        },
        keyPair: null,
    });

    const wait = await queries.accounts.waitFor(
        {
            id: { eq: account },
            storage: {
                balance: {
                    Grams: { gt: "0" }
                }
            }
        },
		'id storage {balance {Grams}}'
    );
};
```

Now you need to calculate future address by generating deploy message of the contract to know where to transfer funds for deploy.

```javascript
const futureHelloAddress = (await client.contracts.createDeployMessage({
        package: HelloContract.package,
        constructorParams: {},
        keyPair: helloKeys,
    })).address;
    console.log(`Future address of the contract will be: ${futureHelloAddress}`);
```

Transfer some grams to this address:

```javascript
    await get_grams_from_giver(client, futureHelloAddress);
    console.log(`Grams were transfered from giver to ${futureHelloAddress}`); 
```

Now you are ready to initialize the account with contract code by deploying it. Run

```javascript
const helloAddress = (await client.contracts.deploy({
    package: HelloContract.package,
    constructorParams: {},
    keyPair: helloKeys,
})).address;
console.log(`Hello contract was deployed at address: ${helloAddress}`);
```

The **package** parameter is a structure with two fields: `abi` and `imageBase64`, where:

- `abi` is a contract ABI;
- `imageBase64` is the TVC code encoded with base64.

The `contructorParams` includes parameters passed to the constructor. It can be empty and expressed as `{}`, if the constructor has no parameters.

The `keyPair` is a mandatory parameter that specifies the following:

- public key is placed into contract initial state as a rule for deploying ABI-based contracts;
- secret key is used to sign a constructor invocation.

The deployment method executes the following sequence:

1. Prepares a deploy request.
2. Sends a constructor message to a node 
3. Waits until the deployment is complete.
4. Generates the constructor invocation request.
5. Sends the invocation to the blockchain node.
6. Waits until the invocation phase is complete.

The returned result contains an account address assigned to the newly deployed contract.

## Running contracts

### Running functions

Running a contract implies the following steps:

1. Generating an input message to contract with a function name and parameter values.
2. Posting this message to the node.
3. Waiting until the node runs the contract at the TVM, recording the result into blockchain and synchronizing the changes with TON.
4. Reading an output message with function result from the blockchain and decoding it.

In the SDK all these steps are incorporated within the `run` library method.

To run a contract, we need its address, an ABI, a function name with parameters and a key pair if message signing is required.

```javascript
const resut = await client.contracts.run({
    address: helloAddress,
    abi: HelloContract.package.abi,
    functionName: 'sayHello',
    input: {},
    keyPair: helloKeys,
});
```

The result contains the `output` field with a decoded output message returned by the source contract function.

If an optional `keyPair` parameter is specified, the input message is signed and accompanied with a public key. So, the contract can perform authorization using a verifiable public key passed to it.

### Running  locally

Cases are when all a contract function does is calculate some data based on the current contract state and return the calculation result. In these cases we do not need the standard sequence to run a contract. Instead, the following steps are taken:

1. Generate an input message to contract with a function name and parameter values.
2. Read the current contract state into an application. The contract state is taken from ArangoDB.
3. Run the contract code on a lightweight TVM included into the client library by passing a contract state and an input message.
4. Decode the contract output message.

This running method is less time-consuming and involves no fees related to a regular contract execution.

> **Note**: local contract invocation does not impact the blockchain: all transactions, message and state change produces are just discarded.

To execute a contract inside an application just use the `runLocal` method instead of `run`. The parameters are the same, only the method name differs.

```javascript
const localResponse = await client.contracts.runLocal({
    address: helloAddress,
    abi: HelloContract.package.abi,
    functionName: 'sayHello',
    input: {},
    keyPair: helloKeys,
});   
```

## Decoding messages

Some contract functions can generate output messages targeted at some external services. Mainly, these are integration services designed as a *glue* between blockchain contracts and regular REST (or similar) services.

Typically, these services use the following scenario:

1. The service subscribes to messages changes in a blockchain. Usually a subscription is filtered by messages related to a specific account.
2. When a new message comes to a blockchain, an integration service is triggered. It reads and decodes this message.
3. Then, the integration service invokes an offchain service using parameters according to the decoded data.

For these services the Client library provides the `decodeOutputMessage` method. It decodes output messages recorded into a blockchain.

In addition to decoding output messages the Client library provides the `decodeInputMessage` method that decodes input messages.

**Note**: Decoding requires an ABI and is only applicable to ABI compliant messages.

Advanced Features

## Advanced Features

### Signing messages externally

Although the library provides friendly and simple functions to deploy and run contracts, there are cases when more precision and control is required to deploy and run an application contract.

For example, an application can use a crypto provider inaccessible from the client library (e.g. a JavaCard crypto provider).

To solve this, the library offers several functions:

- `createUnsignedDeployRequest` and `createUnsignedRunRequest` – these functions take the same parameters as the `deploy` or `run` functions except that instead of `keyPair` only the public key is passed. This function returns an encoded message body and a byte buffer required to produce a signature.
- `createSignedDeployMessage` and `createSignedRunMessage`– this functions take an unsigned message body, sign bytes and the public key. The output is a message compatible with a blockchain node.
- `createDeployMessage` and `createRunMessage` – these functions generate the same sequence.

The example below shows how to create an unsigned deploy message, sign it and then combine the unsigned message with the signature and finally to deploy the signed message.

```javascript
async function testExternalSigning(client) {
    const { contracts, crypto } = client;

		// Generate Key Pair

    const masterKeys = await crypto.ed25519Keypair();

		// Prepare deploy params and use only public key

    const deployParams = {
        package: events_package,
        constructorParams: {},
        keyPair: { public: masterKeys.public, secret: '' },
    };

		// Create unsigned deploy message

    const unsignedMessage = await contracts.createUnsignedDeployMessage(deployParams);
		const bytesToSignBase64 = unsignedMessage.signParams.bytesToSignBase64;

		// Create signature for bytes buffer
		// This can be done in isolated secret place like a HSM
    
    const signBytesBase64 = await crypto.naclSignDetached(
			{ base64: bytesToSignBase64 }, 
			`${masterKeys.secret}${masterKeys.public}`, 
			TONOutputEncoding.Base64
		);

		// Create signed message with provided sign

    const signed = await contracts.createSignedDeployMessage({
        address: unsignedMessage.address,
        createSignedParams: {
            publicKeyHex: masterKeys.public,
            signBytesBase64: signBytesBase64,
            unsignedBytesBase64: unsignedMessage.signParams.unsignedBytesBase64,
        }
    });

		// Deploy signed message

    const message = await contracts.createDeployMessage(deployParams);
    expect(signed.message.messageBodyBase64).toEqual(message.message.messageBodyBase64);
}
```





  




  