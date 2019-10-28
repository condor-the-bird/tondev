# Overview

TON Labs Client Libraries provide [TON Dev](https://ton.dev/) community with functionality that allows your applications to interact with blockchain. All functions are divided into:

- **crypto** – most cryptography functions required to work with blockchain.
- **contracts** - interaction with nodes to deploy and run smart contracts on the blockchain.
- **queries** – monitoring and querying realtime data. The functionality relies on an indexing database over blockchain.

Regardless of the selected language, each library usage follows the same principles. There is a similar set of modules, functions and their respective parameters.

Each TON Labs Client Library is stateful, so at the first step you have to create an instance of a TON Labs Client object, configure it and setup. Usually, all three actions are performed by a single call to the relevant TON Labs Client constructor.

# Contracts

## About the ABI

Despite the fact that each TON contract is actually a single function, the TON SDK allows defining multiple functions within it. To achieve it, the original multi-functional contract is compiled into a single function one with an incoming message dispatcher.

For correct encoding incoming messages to these contracts and decoding output ones, the SDK Library uses an ABI: a structural description of input/output messages related to contract functions.

## Compiling to TVC

With the SDK compiler you can:

- obtain TVM-ready code from Solidity sources.
- obtain TVM-ready code from our LLVM-based compiler that can potentially take source code in various general purpose languages. Now we have an implementation of the C language.

The [Toolchain documentation](/Compilers/About) contains all information on TON Labs Toolchain.

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
const result = await client.contracts.run({
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

# Queries

Use the SDK to make queries to blockchain objects. A *live blockchain snapshot* is accessible through an ArangoDB instance in the local node.

To query it, GraphQL protocol with subscription options is implemented (get the schema at <http://127.0.0.1/graphql>). All data in the database are divided into following collections:

- *accounts*: blockchain account data;
- *transactions*: transactions related to accounts;
- *messages:* input and output messages related to transactions;
- *blocks:* blockchain blocks.

The structure of each collection item matches that on the TON blochchain. So, for additional details on specific fields refer to official TON documentation.

The `queries` module of the Client library defines four objects to access each collection: `accounts`, `transactions`, `messages` and `blocks`. 

**Sample query to an account that returns account balance for an address.**

```SQL
  query {
  accounts(
    filter: {
      id: {eq: "a46af093b38fcae390e9af5104a93e22e82c29bcb35bf88160e4478417028884"}
    }
  ) {
    id
    storage{
      balance{
        Grams
      }
    }
  }
}
```

Each object has a set of query methods: 

- `query`: filters collection items according to the requested condition. The available filtration functionality covers a wide range of tasks. If none of the collection items matches the request, an empty set is returned.
- `waitFor`: same as above, but this method never returns until the requested item appears (i.e. it actually waits).
- `subscribe`: starts monitoring the relevant blockchain for items matching the requests. The subscription monitors all insert and update operations.

## Making queries

To perform a query over the relevant blockchain, choose a collection and then specify a filter, result projection, sorting order and the maximum number of items in the results list.

```javascript
const transactions = await client.queries.transactions.query({
    now: { eq: 1567601735 }
}, 'id now status');
```

The example above demonstrates a query to the `transactions` collection with the following parameters:

- `filter`: a JSON object matching the internal collection structure. It supports additional conditions for particular fields. In the example above, the `now` field of a transaction must be equal to 1567601735.
- `result`: is a result projection that deter structural subset used for returning items. In the example above the request is limited to three fields: `id`, `now` and `status`. Note that results have to follow GraphQL rules.

Given that the `now` field is unique in the example, the `transactions` array is either empty or contains one item.

The `waitFor` method can be used to obtain items that are not yet written to the blockchain but are expected to appear.

```javascript
const transactions = await client.queries.transactions.waitFor({
    now: { eq: 1567601735 }
}, 'id now status');
```

The signature of the `waitFor` is exactly the same as for the `query`. The only difference is behavior: if there is no transaction with the specified `now` in the requested blockchain, this method waits indefinitely until the transaction appears in the blockchain.

## Subscriptions

Some applications monitor blockchain for a specific data set or for potential updates. The subscribe method (included in collection object) handles these scenarios:

```javascript
const subscription = client.queries.blocks.subscribe({}, 'id', (e, doc) => {
    console.log('On block have created: ', doc);
});

setTimeout(() => {
    subscription.unsubscribe();
    resolve();
}, 10*60*1000);
```

In this example, we start a subscription and react whenever a block is inserted or updated in the relevant blockchain.

The `filter` and `result` parameters are the same as in the `query` method. The `filter` parameter narrows the action down to a subset of monitored items. In this case, the filter is empty: all items are included into monitoring.

The last parameter is an event handler. This event is triggered every time the monitored block is inserted or updated in the relevant blockchain.

The return value of the `subscribe` method is a subscription object with one available method:`unsubscribe`. The subscription remains active until it is called. In the example above the subscription is cancelled within 10 min.

## Filtration and sorting

Filters applied to querying functions are data structures matching collection items with several extra features:

- The value for scalar fields (e.g. strings, numbers etc.) is a structure with the scalar filter.
- The value for array fields is a structure with an array filter.
- The value for nested structures is a filter for nested structure.

### Scalar filters

Scalar filter is a structure with one or more predefined fields. Each field defines a specific scalar operation and a reference value:

- `eq`: item value must be equal to the specified value;
- `ne` : item value must not be equal to the specified value;
- `gt`: item value must be greater than the specified value;
- `lt`: item value must be less than specified value;
- `ge`: item value must be greater than or equal to the specified value;
- `le`: item value must be less than or equal to the specified value;
- `in`: item value must be contained in the specified array of values;
- `notIn`: item value must not be contained within the specified array of values.

Filter example:

```javascript
{
    id: { eq: 'e19948d53c4fc8d405fbb8bde4af83039f37ce6bc9d0fc07bbd47a1cf59a8465'},
    status: { in: ["Preliminary", "Proposed", "Finalized"] }
}
```



> Note that when a scalar filter for a field contains multiple operators, the`AND` logical operator is used to combine all the conditions:

```javascript
{
    now: { gt: 1563449, lt: 2063449 }
} 
```

The logic from the above snippet can be expressed in the following way:

```javascript
(transaction.now > 1563449) && (transaction.now < 2063449)
```

### Array filters

Array filters are used for array (list) fields. Each has to contain at least one of the predefined operators:

- `any` : used when at least one array item matches the nested filter;
- `all`: used when all items matches the nested filter.

The `any` or `all` must contain a nested filter for an array item.

Array operators are mutually exclusive and can not be combined. For empty arrays, the array filter is assumed to be false.

### Structure filters

If an item is a structure, then a filter has to contain fields named as fields of this item. Each nested filter field contains a condition for the appropriate field of an item. The `AND` operator is used to combine conditions for several fields.

### Joins

The NoSQL database contains additional fields that work as cross-references for related collections. For example, the *transactions* collection has the `in_message` field that stores the relevant message item. The message item exists in *messages* collection and has the `id` value equal to the `in_msg` value in *transactions*.

Joined items are represented as nested structures in a filter and in the result projection.

### Sorting and limiting

By default, retrieval order for several items is not defined. To specify it, use the `orderBy` parameter of `query` method.

The sort order is represented by an array or sort descriptors. These structures contain two fields: `path` and `direction`:

- `path` specifies a path from a root item of the collection to the field that determines the order of return items. The path includes field names separated by dot.
- `direction` specifies the sorting order: `ASC` or `DESC` (ascending and descending).

You can specify more than one field to define an order. If two items have equal values for the first sort descriptor, then second descriptor is used for comparison, etc. If values of sorting fields are the same for several items, then the order of these items is not defined.

The `limit` parameter determines the maximum number of items returned. This parameter has a default value of 50 and can not exceed it. If specified limit exceeds 50, 50 is used.

## Special fields

Each items in each collection has a unique key stored in the `id` field. This ID is the same as the item blockchain identifier.

## Variability and Nested Levels 

GraphQL is designed to search for entities similar to documents. In our case these entities are represented by the above mentioned collections.

Obviously, each entity may have a set of fields. A field can be used as a filter. But, some complex fields (e.g. a message header) also include fields (values) and whole structures. These enclosed fields are not consistent even within a single collection. Therefore, you cannot make a query that is filtered by, say, `header` field alone. 

You have to build a complex query that drills down to а particular scalar field or field with primitive value (string, number or boolean) at the bottom level of the whole nested structure. 

As mentioned before, field structures may depend on document type or other conditions. In this case we have to use the GraphQL 'union' type which means that a value can have one of alternative types. For example the message `header` field depends on message type: internal, external inbound or external outbound, so we have three alternative variants for message header structure with three fields in the header (`MsgInt`, `MsgExtIn` or` MsgExtOut`).

In the projection part of GraphQl queries we can use '`...on`' syntax to specify a result set for every variant at any level. Thus, the sample implies two nested levels and the number is unlimited. you can drill down as deep, as needed.

```
...on field_name {
   field_name {
      filter_value
      }
  }
```

Once you type `...on`, autocomplete hints appear. 

See the usage example below.

```sql
query{
  messages(filter: { 
    header: {
      IntMsgInfo: {
        created_lt: { gt: 281 }
      },
    }
  }, orderBy: [{path:"header.IntMsgInfo.created_lt", direction: ASC}])
  { 
    id 
    header {
      ...on MessageHeaderIntMsgInfoVariant {
        IntMsgInfo {
          created_lt
        }
      }
    }
  }
}
```

We see that the existing approach and schema are not perfect. There are improvement options and wee seek to implement them.

# Crypto Functions

## Library Overview

TON Client Library is shipped with a crypto module that contains the following set of crypto functions for TON blockchain.

- math and random: generate_random_bytes, modular_power, factorize;
- sha256, sha512: used for address and signature hash calculation, see the ABI spec;
- ed25519 key generation functions;
- scrypt;

### Hierarchical deterministic keys

These functions are used to generate BIP39 keys. Keys are derived from a known seed.

hdkey_xprv_from_mnemonic, hdkey_secret_from_xprv, hdkey_public_from_xprv, hdkey_derive_from_xprv, hdkey_derive_from_xprv_path; 

### Mnemonic Functions

These functions are used to implement a seed phrase at the application side.

`mnemonic_is_valid`: checks validity of a seed phrase. Not every 24 words can be used for a seed phrase and the order matters. The function controls it; 

`mnemonic_get_words `is used to show the list of words in seed input form.

`mnemonic_generate_random`: random seed generation when an application is created. 

### Messaging Cryptography 

In TON Labs we use naclBox. Let's assume that there is Party A and Party B interacting via blockchain. Each party has a key pair (public and private keys): aPair and bPair.

When naclBox is implemented, each pair uses its `сrypto.naclBoxKeypairFromSecretKey` library to generate an auxiliary key pair: aCryptoPair and bCryptoPair.

Our flow only uses the public encrypted keys (aCryptoPair.public and bCryptoPair.public) exchanged by the parties.

Then, if A wants to send an encrypted message to B, it encrypts it with aPair.private and bCryptoPair.public keys. Also, the `nonce` factor may be used, see further.

B then de-crypts the received message with bPair.private and aCryptoPair.public and the above mentioned `nonce` factor.

**nonce Factor**

nonce can be implemented for successful decrypting. It is believed to add security when other messaging channels are used in interaction, apart from blockchain (e.g. email).

## Key Store

The crypto module holds a set of key pairs accessible through a handle. A key pair can be added to a key store or removed from it. Once added to a keystore, a key pair gets a handle assigned to it. This handle can be used in relevant crypto functions instead of the key pair itself.

## Reference Links

Follow the link to our github repository to get the list of our crypto-functions: https://github.com/tonlabs/ton-client-js/blob/75270514d6e1051fe7159b7bcc79f1110ebe6d1b/types.js#L55

These functions are standard and familiar to the professional audience. For a brief guide, follow the link: https://github.com/dchest/tweetnacl-js/blob/master/README.md#documentation (search for functions starting with `nacl`).