In TON, smart-contract life-cycle is implemented via messages sent to the node. All messages are serialized and deserialized (encoded/decoded) according to the ABI.  For example, contract deployment is, actually, a complex message that involves several operations (message creation, signing, sending and processing).

TON Labs SDK offers several methods for flexible message handling to build any architecture you need. In most cases you can do with the deploy and run functions, but if for some reason you need to separate message creation, message signing, deploy and run, use the atomic functions covered below.

Also we suggest reading these document to better understand how message encoding and decoding works in TON.

## Basic Methods

Deploy and Run are basic methods to manage smart contracts. Arguments and parameters of these methods are covered in details, as they are used in advanced methods as well.

### Deploy

Creates a deploy message with contract code, its initial data and constructor parameters, pushes it to the blockchain and waits for the contract to deploy. Returns the address of the deployed contract.

#### Arguments

| Name                  | Type                    | Mandatory | Description                                                  |
| :-------------------- | :---------------------- | :-------- | :----------------------------------------------------------- |
| {data_for_deployment} | TONContractDeployParams | YES       | Object, containing all the data required for contract deployment |

####  Return

| Name     | Type                    | Description                                  |
| :------- | :---------------------- | :------------------------------------------- |
| {result} | TONContractDeployResult | Object, containing the results of the deploy |

####  TONContractDeployParams fields

| Name              | Type               | Mandatory | Description                                                  |
| :---------------- | :----------------- | :-------- | :----------------------------------------------------------- |
| package           | TONContractPackage | YES       | Contract package, containing code and ABI, that is used for deploy |
| constructorParams | Object             | YES       | Constructor parameters                                       |
| initParams        | Object             | NO        | Additional parameters in a form of an object that are saved right to the Persistent Storage (c4) during deployment. This field determines the contract address (in combination with the contract code). In previous versions it only included the public key, but now it can store additional data. |
| keyPair           | TONKeyPairData     | YES       | Key Pair that is used for deploy. See Queries Module.        |

####  TONContractDeployResult fields

| Name            | Type    | Description                                                  |
| :-------------- | :------ | :----------------------------------------------------------- |
| address         | string  | address of the deployed contract                             |
| alreadyDeployed | boolean | false - everything is okay, contract has been deployed *COMING SOON true - contract was not deployed because it had already been deployed before |

------

#### Code sample

```javascript
const keys = await crypto.ed25519Keypair();
const piggyBankParams = {
    amount: 123,
    goal: stringToArray('Some goal'),
};

const piggyBankAddress = (await contracts.deploy({
        package: PiggyBankPackage,
        constructorParams: piggyBankParams,
				initParams:{},
        keyPair: keys,
    })).address;
console.log('[PiggyBank] Piggy Bank address:', piggyBankAddress);
```

### Run

This method is used to call contract methods within the blockchain. Calling `run` creates a message with the following serialized parameters and sends it to the blockchain. Message is serialized according to the ABI rules. After the message is sent `run` waits for it to be executed and returns a JSON-object with the resulting parameters. 

------

####  Arguments

| Name           | Type                 | Mandatory | Description                                                  |
| :------------- | :------------------- | :-------- | :----------------------------------------------------------- |
| {data_for_run} | TONContractRunParams | YES       | Object containing all the data required for contract function call |

####  Return

| Name     | Type                 | Description                                  |
| :------- | :------------------- | :------------------------------------------- |
| {result} | TONContractRunResult | Object, containing the results of the deploy |

#### TONContractRunParams fields

| Name         | Type           | Required | Description                     |
| :----------- | :------------- | :------- | :------------------------------ |
| address      | string         | YES      | Address of the called contract  |
| abi          | TONContractABI | YES      | Contract ABI                    |
| functionName | string         | YES      | Contract function name          |
| input        | Object         | YES      | Contract function parame        |
| keyPair      | TONKeyPairData | YES      | Message signature contract keys |

#### TONContractRunResult fields

| Name        | Type         | Description                                                  |
| :---------- | :----------- | :----------------------------------------------------------- |
| output      | Object       | Function resulting parameter values                          |
| transaction | QTransaction | Transaction that included this function. The `out_msgs` field holds all the messages within the transaction. You can trace the transaction tree from this transaction and below transactions from here. |

------

## Advanced Methods

### createDeployMessage

This method is a part of the standard deploy script and is used to create a message to the blockchain to get a contract address before an actual deploy. It has the same parameters as the general `deploy` method. It returns an address in raw format. In TON you need contract address before an actual deploy to send [test] Grams to it. Given the gas logic, you cannot deploy a contract with zero balance. 

**Sample script:**

```javascript
const futureHelloAddress = (await client.contracts.createDeployMessage({
        package: HelloContract.package,
        constructorParams: {},
        keyPair: helloKeys,
    })).address;
    console.log(`Future address of the contract will be: ${futureHelloAddress}`);
```

The script is based on the SDK Contracts module that contains the following method template:

```javascript
async createDeployMessage(params: TONContractDeployParams): Promise<TONContractDeployMessage> {
        const message: {
            address: string,
            messageId: string,
            messageIdBase64: string,
            messageBodyBase64: string,

        } = await this.requestLibrary('contracts.deploy.message', {
            abi: params.package.abi,
            constructorParams: params.constructorParams,
            initParams: params.initParams,
            imageBase64: params.package.imageBase64,
            keyPair: params.keyPair,
        });
        return {
            message: {
                messageId: message.messageId,
                messageIdBase64: message.messageIdBase64,
                messageBodyBase64: message.messageBodyBase64,
            },
            address: message.address
        }
```

 

Note that the function uses `TONContractDeployMessage` type that in turn refers to `TonContractMessage` type. The combination of these types enables the method to return what we need: a message and a contract address.

```javascript
type TONContractUnsignedMessage = {
    unsignedBytesBase64: string,
    bytesToSignBase64: string,
}

type TONContractMessage = {
    messageId: string,
    signParams: TONContractUnsignedMessage,
}

type TONContractDeployMessage = {
    address: string,
    message: TONContractMessage;
```

### createRunMessage

This method is similar to `createDeployMessage` but it applies to active contracts. It yields a `TONContractMessage` message body and the ID of public contract function that was called, not the contract address. 

```javascript
async createRunMessage(params: TONContractRunParams): Promise<TONContractRunMessage> {
        const message = await this.requestLibrary('contracts.run.message', {
            address: params.address,
            abi: params.abi,
            functionName: params.functionName,
            input: params.input,
            keyPair: params.keyPair,
        });
        return {
            abi: params.abi,
            functionName: params.functionName,
            message
        }
    }
```

### createUnsignedDeployMessage

This function allows creating a separate deploy message without a signature. You may need this function if in your architecture signing is performed at an HSM or some kind of offline hardwallet where the secret key is stored. The method creates a byte block that has to be signed via a specific crypto-method. 

**Method syntax**

```javascript
async createUnsignedDeployMessage(params: TONContractDeployParams): Promise<TONContractUnsignedDeployMessage> {
        const result: {
            encoded: TONContractUnsignedMessage,
            addressHex: string,
        } = await this.requestLibrary('contracts.deploy.encode_unsigned_message', {
            abi: params.package.abi,
            constructorParams: params.constructorParams,
            initParams: params.initParams,
            imageBase64: params.package.imageBase64,
            publicKeyHex: params.keyPair.public,
        });
        return {
            address: result.addressHex,
            signParams: result.encoded,
        }
    }
```

**Usage example**

The code below demonstrates how the method is used to create an `unsignedMessage`. Then a signature is obtained to create a signed deploy message. For example, the key is unknown, but there is a device with an API where the unsigned block (not the whole message) is sent. The signature (not the whole message) is returned back via the REST API. 

Finally, `createDeployMessage` is used. 

```javascript
test('External Signing', async () => {
    const { contracts, crypto } = tests.client;
    const keys = await crypto.ed25519Keypair();
    
    var contract_package = events_package;
    contract_package.abi["setTime"] = false;

    const deployParams = {
        package: contract_package,
        constructorParams: {},
        keyPair: keys,
    };
    const unsignedMessage = await contracts.createUnsignedDeployMessage(deployParams);
    const signKey = await crypto.naclSignKeypairFromSecretKey(keys.secret);
    const signBytesBase64 = await crypto.naclSignDetached({
        base64: unsignedMessage.signParams.bytesToSignBase64,
    }, signKey.secret, TONOutputEncoding.Base64);
    const signed = await contracts.createSignedDeployMessage({
        address: unsignedMessage.address,
        createSignedParams: {
            publicKeyHex: keys.public,
            signBytesBase64: signBytesBase64,
            unsignedBytesBase64: unsignedMessage.signParams.unsignedBytesBase64,
        }
    });

    const message = await contracts.createDeployMessage(deployParams);
    expect(signed.message.messageBodyBase64).toEqual(message.message.messageBodyBase64);
});
```

### сreateUnsignedRunMessage

This method works similarly to the above two, but it is designed to create a message returning the ID of the public contract function that is called without a signature. 

This function also can be used in a distributed architecture (when messages are signed externally).

### createSignedMessage

This method can also be used used in distributed architectures where message signing is carried out externally.

### createSignedDeployMessage

This method creates a signed messaged from `createUnsignedDeploy` message and the signature obtained externally. The resulting message can be passed to the `sendMessage` or `processMessage` method.

### createSignedRunMessage

content under construction

### createRunBody

`createRunBody` creates only a message body with parameters encoded according to the ABI

```javascript
const runBody = await contracts.createRunBody({
        abi: deployee_package.abi,
        function: "constructor",
        params: {
            _param1: 1,
            _param2: 2
        },
        internal: true,
    });
```

 

### decodeRunOutput

 `decodeRunOutput` is used in the `Run()` method to decode a response message from a called contract.

### decodeInputMessageBody

 `decodeInputMessageBody`  decodes an inbound message to a contract. The general syntax declared in TON SDK Contracts module implies that the function takes `TONContractDecodeMessageBodyParams` (the ABI and bodyBase64). The code below demonstrates that the function actually takes an ABI for decoding rules and the message Base64 body image.

```javascript
test('decodeInputMessageBody', async () => {
    const { contracts } = tests.client;
    const body = 'te6ccoEBAgEAcwARcwEbACfvUIcBgJTr3AOCAGABAMDr2GubWXYR6wuk6WFn4btjW3w+DbidhSrKArHbqCaunLGN9LwAbQFT9kyOpN6DR6DJbuKkvC94KwJgan7xeTUHS89H/vKbWZbzZEHu4euhqvQE2I9aW+PNdn2BKZJXlA4=';
    const result = await contracts.decodeInputMessageBody({
        abi: WalletContractPackage.abi,
        bodyBase64: body
    });
    expect(result.function).toEqual('createLimit');
    expect(result.output).toEqual({ type: "0x1", value: "0x3b9aca00", meta: "x01" });
});
*/
const events_package: TONContractPackage = {
    abi: {
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
                "name": "emitValue",
                "inputs": [
                    {"name":"id","type":"uint256"}
                ],
                "outputs": [
                ]
            },
            {
                "name": "returnValue",
                "inputs": [
                    {"name":"id","type":"uint256"}
                ],
                "outputs": [
                    {"name":"value0","type":"uint256"}
                ]
            }
        ],
        "events": [
            {
                "name": "EventThrown",
                "inputs": [
                    {"name":"id","type":"uint256"}
                ],
                "outputs": [
                ]
            }
        ],
        "data": [
        ]
    },
    imageBase64: "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"
};
```

### **decodeOutputMessageBody**

`decodeOutputMessageBody` - decodes an outbound message body created externally.

### **decodeRunOutput**

decodeRunOutput is used in the Run() method to decode a response message from a called contract.

### **sendMessage**

​	The `sendMessage` method sends messages to the node without waiting for the result. The method takes data generated by `createDeployMessage`.

### **processMessage**

​	The `processMessage`method sends messages to the node and waits for the result. It provides notifications in case of an error at the node.

```javascript
async processMessage(message: TONContractMessage, resultFields: string): Promise<QTransaction> {
        await this.sendMessage(message);
        return this.queries.transactions.waitFor({
            id: { eq: message.messageId },
            status: { in: ['Preliminary', 'Proposed', 'Finalized'] },
        }, resultFields);
    }
```

### **processDeployMessage**

This method is similar to the above, but reserved for deploy messages created externally.

Method core

```javascript
async processDeployMessage(params: TONContractDeployMessage): Promise<TONContractDeployResult> {
        const transaction = await this.processMessage(
            params.message,
            'id status description { ...on TransactionDescriptionOrdinaryVariant { Ordinary { aborted } } }',
        );
        const ordinary = transaction.description.Ordinary;
        if (ordinary.aborted) {
            throw {
                code: 3050,
                message: 'Deploy failed',
            };
        }
        return {
            address: params.address,
            alreadyDeployed: false,
        };
    }
```

### **processRunMessage**

This method is similar to the above, but reserved for run messages created externally.

### **getCodeFromImage**

This method decodes a Base64 message body according to the ABI. It supports `SETCODE` and `deploy` of one contract by another contract.

### **getDeployData**

`getDeployData` is the method designed to work with the original TON Lite-client. It takes a raw message body without headers (address, bounce flag, etc.). These are then added by the Lite-client. This method supports `SETCODE` and `deploy` of one contract by another contract.

​	 

​	 