# Solidity Compiler

**Q: There are samples of transaction generation via the LLVM Compiler and via the SOL2TVM Compiler. Yet, there is no signing operation in the sample Solidity code.**

**A**: Check the **contract04.sol** example, it demonstrates how to transfer grams with the `**address.transfer()**`function; Signing is not yet supported, plan to add it in the next release

**Q: Does Solidity compiler support ecrecover? Now it throws “std::exception::what: unknown variable: ecrecover”. If not, will there be support soon? There is nothing in the guide about it.**

**A**: No, this is an Ethereum-specific operation. But we plan to provide an equivalent, for now use the ABI.

**Q: Is 'address(this)' operational?**

**A**: Only `address(this).balance` is available at this moment. The `address(this) `function will be available soon.

**Q: Can structures be transferred as function parameters?**

**A**: Not for public functions. The feature is to be released later. For internal functions, yes, this is possible.

**Q: The .push method is unavailable. How do I add a new element?**

**A**: You can use `array[array.length] = new_element;.`The`.push`method will be added later.

**Q: How an address is formed for a Solidity contract?**

**A**: The address of a Solidity smart-contract for TON is deterministic and is computed prior to its deployment. Full address of the contract consists of a 32-bit ID of a workchain the contract is being deployed to and of the 256-bit internal address (or account identifier) inside the chosen workchain.

The internal address is a representative hash of the contract initial state. The contract Initial state consists of the contract code serialized according to the [TON blockchain specification](../../TON Blockchain/TON Specifications/TON Blockchain), section 5.3.10, and its data.

Hash computation principles: the hash function applied to the relevant hash code computation is called "representation hash". Its detailed description is available in the [TON blockchain specification](../../TON Blockchain/TON Specifications/TON Blockchain), section 1.1.8. Essentially, the representation hash is sha256 function recursively applied to the storage cell of its argument.