# TON and TVM

**Q: How to deploy a contract to TON testnet if I use your Toolchain?**

**A**: Follow the procedure [here](https://docs.ton.dev/86757ecb2/p/14cfee/t/57cbaf). If you are a new user, check the related topics as well (Node SE Installation, Deployment). 

**Q: Which messaging format is used in smart contracts?**

**A**: TON Labs uses the same message format as specified at [ton](http://ton.org/).[org](http://test.ton.org/). The message header format is covered in the blockchain whitepaper ([TON Whitepaper](../../TON Blockchain/TON Specifications/TON Whitepaper/) clause 2.4.9), the message layout is covered by the [blockchain specification](../../TON Blockchain/TON Specifications/TON Blockchain/) 3.1.7.

**Q: How is hash calculated?**

**A**: Hash calculation principles used in the TON test node are covered in the official TON VM documentation ([TON Virtual Machine](../../TON Blockchain/TON Specifications/TON Virtual Machine/) clause 3.1.4 -3.1.7, [TON Blockchain Specification](../../TON Blockchain/TON Specifications/TON Blockchain/) 1.1.4). We use another method to calculate the hash of bag of cells, but we get the same result.

**Q: Is there any standard multisig contract we can use for TON? How can a multisig wallet be implemented in the TON network?**

**A**: The standard TON multisig contract specs and source code are unavailable at the moment. Existing open source multisig contracts can be used, but not all Solidity features are supported now by our compiler. 

**Q: Do you have a good way of doing multiple receive addresses for the same wallet?**

**A**: Within TON every contract has only one address. You can use your **Forwarder.sol** contract, but it cannot be compiled with the current compiler version without some fixes. 

**Q: Are wallets supposed to be deployed on the workchain, masterchain or some other chain?**

**A**: Any chain will do, but the basic workchain 0 is the recommended option. Masterchain has very high fees.

**Q: How do fees work? Does a smart contract pay its own fee or is it charged on the caller address? Does the TON use similar concepts of gas price and gas limit?**

**A**: Fees are charged on the contract that executes a transaction. There is *a storage fee, a gas fee and a fee for sending messages from contracts*. TON contracts consume gas ([TON Virtual Machine](../../Blockchain/TON Specifications/TON Virtual Machine/) clause 1.4, appendix A.1) and have gas limits with specific features.


