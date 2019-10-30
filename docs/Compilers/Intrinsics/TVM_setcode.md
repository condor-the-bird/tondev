# TVM_setcode

`tvm_setcode` is designed to implement the TVM SETCODE primitive (see the original TON specification) in the compiler.

The function itself is private and it works in combination with a public function (e.g. `main`, as in the example below) that takes the new code and passes it to `tvm_setcode`.

Code samples below show how the function is defined in Solidity and how it works to actually update code in a test script.

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

