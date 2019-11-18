# Modifying Contracts

Unlike other platforms, TON blockchain allows modifying code of a previously deployed contract. The feature is implemented via TVM SETCODE primitive (see the original TON specification).

In TON Labs Solidity Compiler there is `tvm_setcode` function. 	The function itself is private and it works in combination with a public function (e.g. `main`, as in the example below) that takes the new code and passes it to `tvm_setcode`.

​	Code samples below show how the function is defined in Solidity and how it is used in the SDK to actually update code in a test script.

The test script calls the `getVersion` function then the `main` function to indicate that an update in `getVersion` is to be made ( `input: { newcode: code.codeBase64 }`) and then redefines the `getVersion` function with new parameters. Note that the updated code is not used during the current session. It applies next time the contract is called.

    pragma solidity ^0.5.0;
    pragma experimental ABIEncoderV2;
    
    contract Test07b {
    
        struct TvmCell { uint _; }
        
        function tvm_setcode(TvmCell memory newcode) private pure {}
        
        function main(TvmCell memory newcode) public pure returns (uint) {
            tvm_setcode(newcode);
            return 0;
        }
    
        function getVersion() public pure returns (uint) {
            return 1;
        }
    }


```javascript

test('testSetCode', async () => {
    const { contracts, crypto } = tests.client;
    const keys = await crypto.ed25519Keypair();

    const deployed = await deploy_with_giver({
        package: setCode1_package,
        constructorParams: {},
        keyPair: keys,
    });

    const version1 = await contracts.run({
        address: deployed.address,
        functionName: 'getVersion',
        abi: setCode1_package.abi,
        input: { },
        keyPair: keys,
    });

    const code = await contracts.getCodeFromImage({
        imageBase64: setCode2_imageBase64
    });

    const result = await contracts.run({
        address: deployed.address,
        functionName: 'main',
        abi: setCode1_package.abi,
        input: { newcode: code.codeBase64 },
        keyPair: keys,
    });

    const version2 = await contracts.run({
        address: deployed.address,
        functionName: 'getVersion',
        abi: setCode1_package.abi,
        input: { },
        keyPair: keys,
    });

    expect(version1).not.toEqual(version2);
});
```

A `setcode_package` consists of the ABI version and imageBase64 (the compiled contract code as a TVC file). The package is sent in two parts : first package deploys the contract, second one changes the code to a newer version. Then we use `getversion` to check that code has been replaced.

```javascript
const setCode1_package: TONContractPackage = {
    abi: {
        "ABI version": 1,
        "functions": [
            {
                "name": "main",
                "inputs": [
                    {"name":"newcode","type":"cell"}
                ],
                "outputs": [
                    {"name":"value0","type":"uint256"}
                ]
            },
            {
                "name": "getVersion",
                "inputs": [
                ],
                "outputs": [
                    {"name":"value0","type":"uint256"}
                ]
            },
            {
                "name": "constructor",
                "inputs": [
                ],
                "outputs": [
                ]
            }
        ],
        "events": [
        ],
        "data": [
        ]
    } ,
    imageBase64: //TVC file here
}
```

