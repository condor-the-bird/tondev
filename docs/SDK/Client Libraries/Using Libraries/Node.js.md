Follow guidelines in this section to create your own environment based on Node.js with the Node SE tools and to develop your first test contract. 

> Note that this guide is also good for React Native and WebAssembly. The only difference is the source library used.

First and foremost, create a local folder for all your test projects. Note that in this documentation the sample folder is called **ton-dev**. If your folder has a different name, make sure to edit the code accordingly.

## Creating a contract

1. Create "hello" folder and place the "hello.sol" contract source code into it:

```javascript
pragma solidity ^0.5.4;

contract HelloTON {
    uint32 timestamp;

    modifier alwaysAccept {
      tvm_accept();
      _;
  }

    function tvm_accept() private pure {}

    constructor() public {
        timestamp = uint32(now);
    }

    function touch() external alwaysAccept {
        timestamp = uint32(now);
    }

    function sayHello() external view returns (uint32) {
        return timestamp;
    }
}
```

​      

<iframe class="no-border max-full-width full-height flex-grow" src="https://www.youtube.com/embed/TyMOi1kLz20?autohide=1&amp;showinfo=0&amp;rel=0&amp;fs=0" style="max-width: 100%; height: 346px; -webkit-box-flex: 1; flex-grow: 1; border: none;"></iframe>
2. Call `cd hello` to navigate to the new folder.

3. Run TON labs Sol2TVM compiler:

```shell
tondev sol hello -l js -L deploy
```

`-l js` option is used to generate JavaScript client helper code for the compiled contract.

`-L deploy is` used to include an `imageBase64` field into the generated JavaScript contract client code.

## Creating an app

Before being able to play with a smart contract on the blockchain, we need a blockchain infrastructure for contract testing and debugging.

Make sure that you started a local node instance according to the guidelines provided in the [**Installation**](/SDK/Installation/) section.

```shell
tondev start
```



Let's start with Node.js to show how to build a test application, deploy and run its smart contract.

> **Note:** To create an application according to this procedure, you have to install Node.js. It is recommended to have the latest version.

1. Let's initialize a Node.js application:

```shell
~/ton-dev/hello$ touch index.js
~/ton-dev/hello$ npm init
```

 Add a section to package.json:

```shell

"dependencies": { "ton-client-node-js": "^0.12.1"}
```

Then execute (in the project folder):

```shell
~/ton-dev/hello$ npm install 
```

2. Connect to Node SE or to TON Labs testnet. In JS Client Libraries the `TONClient`class is used to connect to TON Blockchain node, that can work with SDK. 

If you use the local node (NodeSE), specify `'<http://0.0.0.0>'` in the following code:

```javascript
const client = new TONClient();
    client.config.setData({
        servers: ['<http://0.0.0.0>']
    });
```

> **Important**: for Windows use <http://127.0.0.1/> or [http://localhost](http://localhost/).

If you use TON Labs testnet, specify: `servers: ['Node URL'].`For testing we use `'<https://testnet.ton.dev>'`:

```javascript
  const client = new TONClient();
    client.config.setData({
        servers: ['<https://https://testnet.ton.dev>']
    });
```

> In JS Client you can simultaneously use several nodes. Create a separate TONClient object for each connection.

3. Open the "index.js" file in your preferred editor and enter the code below.

```javascript
const { TONClient } = require('ton-client-node-js');

async function main(client) {
}

(async () => {
    try {
        const client = new TONClient();
        client.config.setData({
            servers: ['NodeSE/Testnet URL']
        });
        await client.setup();
        await main(client);
        console.log('Hello TON Done');
    process.exit(0);
    } catch (error) {
        console.error(error);
    }
})();
```

4. Run the client app as follows:

```shell
~/ton-dev/hello$ node index.js
Hello TON Done
~/ton-dev/hello$

```

## Deployment

Before a contract is deployed, it has to be defined in your node.js application. The necessary elements are: a compatible TVM code and an ABI structure. Both elements were obtained at the compilation stage before in the `helloPackage.js`file.

> **Tip**: For more details on the ABI, see the [specification](https://docs.ton.dev/86757ecb2/p/70c253).

For deployment, you also have to take the following steps:

1. Generate the key pair. Each time a contract is deployed, you can generate keys with a built-in `ton.crypto.ed25519Keypair` crypto module.

2. Switch to your test app source code to declare a smart contract package and the relevant key pair:

```javascript
...

// Define contract package

const HelloContract = require('./helloContract');

// Define keys for our contract

const helloKeys = {
    public: '55d7bab463a6a3ef5e03bb5f975836ddfb589b9ccb00329be7da8ea981c5268a',
    secret: 'de93a97c7103c2d44e47972265cfdfe266fd28c8cadc4875804ee9f57cf786d6',
};

async function main(client) {
}

...
```

3. The contract is almost ready for deployment, but **in TON blockchain you must deposit GRAMs to the address of the deployed contract before the actual deploy**. **Otherwise deploy will fail.** 

You can send Grams from another contract or use our giver. To learn how to use giver, check the relevant section in the document covering the [Contracts](SDK/Client Libraries/Library Modules/Contracts/) module. There is a detailed usage example. 

```javascript
...

async function main(client) {
    const helloAddress = (await client.contracts.deploy({
        package: HelloContract.package,
        constructorParams: {},
        keyPair: helloKeys,
    })).address;
    console.log(`Hello contract was deployed at address: ${helloAddress}`);
}

...
```

Now run to check how it works.

```shell
~/ton-dev/hello$ node index.js
Hello contract was deployed at address: 516c7a2bc72c5728526eb73064da07a2876d964c3da5ed2488e1aba3da20be3f
~/ton-dev/hello$
```

## Running a contract

Running your contract on blockchain is also quite simple

```javascript
...

async function main(client) {
    const helloAddress = (await client.contracts.deploy({
        package: HelloContract.package,
        constructorParams: {},
        keyPair: helloKeys,
    })).address;
    console.log(`Hello contract was deployed at address: ${helloAddress}`);

    const response = await client.contracts.run({
        address: helloAddress,
        abi: HelloContract.package.abi,
        functionName: 'sayHello',
        input: {},
        keyPair: helloKeys,
    });
    console.log('Hello contract was responded to sayHello:', response);
}

...
```

Now run the app:

```shell
~/ton-dev/hello$ node index.js
Hello contract was deployed at address: 516c7a2bc72c5728526eb73064da07a2876d964c3da5ed2488e1aba3da20be3f
Hello contract was responded to sayHello: { output: { value0: '0x5d6fba2e' } }
Hello TON Done
~/ton-dev/hello$
```

Alternatively, you can run a contract in the TVM instance included into a client library without interaction with TVM node:

```javascript
...

async function main(client) {
    const helloAddress = (await client.contracts.deploy({
        package: HelloContract.package,
        constructorParams: {},
        keyPair: helloKeys,
    })).address;
    console.log(`Hello contract was deployed at address: ${helloAddress}`);

    const response = await client.contracts.run({
        address: helloAddress,
        abi: HelloContract.package.abi,
        functionName: 'sayHello',
        input: {},
        keyPair: helloKeys,
    });
    console.log('Hello contract was responded to sayHello:', response);

    const localResponse = await client.contracts.runLocal({
        address: helloAddress,
        abi: HelloContract.package.abi,
        functionName: 'sayHello',
        input: {},
        keyPair: helloKeys,
    });
    console.log('Hello contract was ran on a client TVM and also responded to sayHello:', localResponse);
}

...
```

Run:

```shell
~/ton-dev/hello$ node index.js
Hello contract was deployed at address: 516c7a2bc72c5728526eb73064da07a2876d964c3da5ed2488e1aba3da20be3f
Hello contract was responded to sayHello: { output: { value0: '0x5d6fba2e' } }
Hello contract was ran on a client TVM and also responded to sayHello: { output: { value0: '0x5d6fba2e' } }
Hello TON Done
~/ton-dev/hello$
```



> Find more information about deploying and running in the [**Contracts**](SDK/Client Libraries/Library Modules/Contracts/) section.

## Querying blockchain

Each node server is equipped with a database that tracks the relevant blockchain.

This database is accessible through a GraphQL based protocol for querying blockchain.

The Client library contains the Query Module designed to perform GraphQL queries over a blockchain. The simplest way to query a blockchain is using the following `query` method:

```javascript
async function queries(client) {
    const transactions = await client.queries.transactions.query({}, 'id now status');
    console.log('All Transactions: ', transactions);    
}
```

Then all transactions in the relevant blockchain are displayed (the first 50, to be precise). 50  is the default number used when no limit is specified or when it exceeds 50.

For each transaction, we have three result fields: **id**, **now** and **status**.

We have several options to filter the results:

```javascript
const transactions = await client.queries.transactions.query({
    now: { eq: 1567601735 }
}, 'id now status');
console.log('Filtered Transactions: ', transactions);
```

The example gets all transactions with **now** equals to 1567601735. 

> Find more information about a filtering in the [**Queries**](SDK/Client Libraries/Library Modules/Queries) section.