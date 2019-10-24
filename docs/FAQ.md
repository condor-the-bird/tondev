# TON and TVM

**Q: How to deploy a contract to TON testnet if I use your Toolchain?**

**A**: Follow the procedure [here](https://docs.ton.dev/86757ecb2/p/14cfee/t/57cbaf). If you are a new user, check the related topics as well (Node SE Installation, Deployment). 

**Q: Which messaging format is used in smart contracts?**

**A**: TON Labs uses the same message format as specified at [ton](http://ton.org/).[org](http://test.ton.org/). The message header format is covered in the blockchain whitepaper (ton.pdf clause [2.4.9](https://docs.ton.dev/86757ecb2/p/822e19/t/055645)), the message layout is covered by the blockchain specification (tblkch.pdf [3.1.7.](https://docs.ton.dev/86757ecb2/p/931de7/t/821704)).

**Q: How is hash calculated?**

**A**: Hash calculation principles used in the TON test node are covered in the official TON VM documentation (tvm.pdf [clause 3.1.4 -3.1.7](https://zeroheight.com/86757ecb2/p/836380/t/81c0c8), tblkch.pdf [1.1.4](https://zeroheight.com/86757ecb2/p/67dde7/t/58f8f6)). We use another method to calculate the hash of bag of cells, but we get the same result.

**Q: Is there any standard multisig contract we can use for TON? How can a multisig wallet be implemented in the TON network?**

**A**: The standard TON multisig contract specs and source code are unavailable at the moment. Existing open source multisig contracts can be used, but not all Solidity features are supported now by our compiler. 

**Q: Do you have a good way of doing multiple receive addresses for the same wallet?**

**A**: Within TON every contract has only one address. You can use your **Forwarder.sol** contract, but it cannot be compiled with the current compiler version without some fixes. 

**Q: Are wallets supposed to be deployed on the workchain, masterchain or some other chain?**

**A**: Any chain will do, but the basic workchain 0 is the recommended option. Masterchain has very high fees.

**Q: How do fees work? Does a smart contract pay its own fee or is it charged on the caller address? Does the TON use similar concepts of gas price and gas limit?**

**A**: Fees are charged on the contract that executes a transaction. There is *a storage fee, a gas fee and a fee for sending messages from contracts*. TON contracts consume gas (tvm.pdf clause [1.4](https://zeroheight.com/86757ecb2/p/77e11f/t/24a009), appendix [A.1](https://zeroheight.com/86757ecb2/p/17e4d8/t/783524)) and have gas limits with specific features.

# SDK and Integration

**Q**: Based on your documentation and the white paper, there is a gas limit on transactions/transfers but this is not set by the sender but is a function of the transfer amount or contract balance.  Do senders/contracts have any ability to set an upper limit on the gas consumed by a transaction

**A**: contracts can set upper gas limit in `msg.value` (in nanograms), but receiver contract can increase this limit buying more gas (accept cmd). We cannot set gas limit  for external messages; it is automatically calculated as minimal contract balance or global gas limit per transaction.

------

**Q**: Do you have any insight into the various send modes for `SENDRAWMSG`?

I'm trying to understand  failure scenarios and if that call fails when `mode=0`, then the internal contract state fails to update and we risk the failed message being replayed until the account is drained.

`mode=2` is supposed to "ignore errors" such that the contract state will get updated even if the msg fails to send.  But by changing the mode to 2 I'm now seeing some additional fees taken out of my transfer amount (specifically my transfer amount is reduced by 0.001 Grams which is the `total_fwd_fee`)  Do you have any insight here?

**A**: Forward fees reduce transfer amount even if mode=0; these are always present in internal messages.



# Solidity Compiler

**Q: There are samples of transaction generation via the LLVM Compiler and via the SOL2TVM Compiler. Yet, there is no signing operation in the sample Solidity code.**

**A**: Check the **contract04.sol** example, it demonstrates how to transfer grams with the `**address.transfer()**`function; Signing is not yet supported, plan to add it in the next release

**Q: Does Solidity compiler support ecrecover? Now it throws “std::exception::what: unknown variable: ecrecover”. If not, will there be support soon? There is nothing in the guide about it.**

**A**: No, this is an Ethereum-specific operation unavailable in TMV (in TVM public address is not related to the public key) But we plan to provide an equivalent, for now use the ABI.

**Q: Is 'address(this)' operational?**

**A**: Only `address(this).balance` is available at this moment. The `address(this) `function will be available soon.

**Q: Can structures be transferred as function parameters?**

**A**: Not for public functions. The feature is to be released later. For internal functions, yes, this is possible.

**Q: The .push method is unavailable. How do I add a new element?**

**A**: You can use `array[array.length] = new_element;.`The`.push`method will be added later.

**Q: How an address is formed for a Solidity contract?**

**A**: The address of a Solidity smart-contract for TON is deterministic and is computed prior to its deployment. Full address of the contract consists of a 32-bit ID of a workchain the contract is being deployed to and of the 256-bit internal address (or account identifier) inside the chosen workchain.

The internal address is a representative hash of the contract initial state. The contract Initial state consists of the contract code serialized according to the TON blockchain specification, section 5.3.10, and its data.

Hash computation principles: the hash function applied to the relevant hash code computation is called "representation hash". Its detailed description is available in the TON blockchain specification, section 1.1.8. Essentially, the representation hash is sha256 function recursively applied to the storage cell of its argument.

# SDK and Integration

**Q**: Would you be able to point me to where BOC messages are created or signed inside the JavaScript or Rust SDK? In the JS SDK, I just see that all calls end up hitting `this.requestLibrary` with some library specified, and that looks like it ends up calling a `TONClientLibrary` module, but it's not clear where these libraries actually exist (they seem to be on the node itself).

**A**: BOC messages are created and signed inside the Rust SDK library. The library is located at the client side. For the Node.js SDK client, the native Node.js addon is used. For React Native, there is a native library for target platform where the client application is running. For the Web client, there is the Web Assembly module downloaded and executed at the client device. The Rust SDK library contains core SDK functions used on all platforms.

The SDK will be available as Open Source and more platform and language rappers are to be developed. We are open to customer and community suggestions and contributions to the repository, once it is open.

------

**Q**: Don’t see any code in the SDK's that is actually generating keys locally or constructing .boc messages. It seems they are just calling your node to do that for them, so it seems like more of a client to the node. Please correct me if that’s not right.

**A**: See the previous answer. Keys are generated in the Rust core library running on a client device. The message BOC construction code is also located there. The SDK's interaction with the node is limited to sending external blockchain messages calling contracts and querying results from the GraphQL server. All crypto operations are performed locally.

------

**Q**: Is there an API call to the full node that returns a given account’s balance along with the block height at which that balance was calculated, in a single API call? Note: This IS NOT the same thing as historical balance lookup. (Example: GET /balance/account1 -> {balance:10, height: 123902}) The current account state provided by the lite-client does not include this information. Can we be given this information if it is not available today?

**A**: It is possible to get this information by the series of graphQL queries:

- query an account by its ID (address) and remember `last_trans_lt` (last transaction logical time)

```SQL
 query {
   accounts(filter: {id: {eq: "0000000000000000000000000000000000000000000000000000000000000000"}}) {
     id,
     storage {last_trans_lt}
   }
 }
```

- result:

```html
{
   "data": {
     "accounts": [
       {
         "id": "0000000000000000000000000000000000000000000000000000000000000000",
         "storage": {
           "last_trans_lt": 4
         }
       }
     ]
   }
 }
```

- query the transaction with your account ID and the logical time; remember the corresponding `block_id`

```sql
query {
   transactions(filter: {
     account_addr: {eq: "0000000000000000000000000000000000000000000000000000000000000000"}
     lt: {eq: 4}
   }) {
     id,
     block_id
   }
 }
```

- result:

```HTML
{
   "data": {
     "transactions": [
       {
         "id": "3badb9e5707db8e61e0e335e02eacb6df2a118512791b620c73a71d826e840c3",
         "block_id": "4a8537f7499e122fc539b6250cd70168e80a129dcd684de6d0928d86105cf430"
       }
     ]
   }
 }
```

- query a block by an ID with the required data:

```sql
query {
   blocks(filter: {
     id: {eq: "4a8537f7499e122fc539b6250cd70168e80a129dcd684de6d0928d86105cf430"}
   }) {
     id,
        info {
       seq_no
     }
   }
 }
```

- result:

```html
{
   "data": {
     "blocks": [
       {
         "id": "4a8537f7499e122fc539b6250cd70168e80a129dcd684de6d0928d86105cf430",
         "info": {
           "seq_no": 1
         }
       }
     ]
   }
 }
```

------

**Q**: Does your node currently (or will you) provide an API to submit pre-signed messages (e.g. a pre-signed bag-of-cells message)? Can we have this endpoint (not a CLI interface) if it does not exist today?

**A**: To send pre-generated messages, use the `processMessage` function of the SDK JS client. Also, there will be a function to construct unsigned messages. You will be able to sign generated message yourself and then to send it using the SDK. 

For example you can generate message to call the contract by SDK function. That function returns a serialized message and its hash to be signed. You sign it using your own implementation of the signature algorithm (or using some hardware security module). Then you call another SDK function that adds your signature to the message generated at first step. Finally, the resulting signed message can be sent by `processMessage` function.

------

**Q**: How does one derive the message (or transaction) hash from a signed bag-of-cells message to a contract? Can you provide details on how we can compute this on our own? (Ideally we can compute this so we can look it up after submitting to the network.)

**A**: The SDK returns hash of the generated message as a message ID. We will provide a function which will return hash for a specific message in the next few weeks. To calculate hash manually, deserialize the bag of cells into a tree of cells and calculate the representation hash of the root cell as described in paragraph `3.1.5` of the original TVM specification. It's quite an effort, though.

------

**Q**: It seems that messages are not replay protected on their own. Is this the case? Is there any default replay protection provided by the network or must every contract implement their own sequence number or similar method to replay protection?

**A**: There is no default replay attack protection in TON. A contract developer is supposed to implement it manually. There are only "best practices" by TON developers [here](https://test.ton.org/smc-guidelines.txt). TON Labs compilers will have a default inline implementation of replay protection.

------

**Q**: Can you provide more explicit details or provide an example of how an account’s address is derived from the compiled contract and initial state and specifically how that hash is computed and over what components? Source code is also helpful here if you can provide it.

**A**: A blockchain account contains the `State Init` field. It stores the contract code and persisted data. `State init` is also tree of cells. At contract deployment this field is transferred alongside the deploying message (so called constructor message) and is set as the contract initial state. The representation hash (see paragraph **3.1.5** of the TVM specification) of the contract initial state is the account address. The node checks that the state init hash of the deployment message is equal to the destination address. The SDK will have a function for computing account address by its initial state. The SDK source code will be available soon.

------

**Q**: Is (or will be) the solidity-to-tvm and tvm-linker source code available to users? If not, can agreements be signed to make it available to partners?

**A**: Yes, both Sol2Tvm compiler and TVM-Linker will be available as open source.

------

**Q**: With a Solidity contract, how does one properly specify an address parameter? In solidity, addresses are uint256 but this can only capture the address portion and not the workchain_id. How would one specify the full address parameter correctly here? (e.g. “0:D702CC0414858D83A5538A3C1C86231872F006453AF7C825A59F9DD12D636AF” is invalid in solidity for an address param)

**A**: Now we only use the `address` field of *MsgAddressInt* structure as Solidity `address` type. We assume that worckchain_id is 0. But in future the `address` type will represent the whole *MsgAddressInt* structure. Now workchain_id can be defined separately as int8.

------

**Q**: When deploying a contract init message that was created using a solidity contract, the Sol2TVM compiler and tvm_linker, I’m seeing the init succeed initially but then also get replayed dozens of times after. How does one replay protect a contract init message for a solidity-compiled contract? (example message that was replayed after the initialization was already successful previously: <https://test.ton.org/testnet/transaction?account=EQDXAswEFIWNg6VTijwchiMYcvAGRTr3yCWln53RLWNq8CvQ<=195287000001&hash=7F09EC8EEBBAF510D7873CD3DEC494B2DD28A790A7A46E83D0A183F3E672B509>)

**A**: The sample message at <https://test.ton.org/testnet/transaction?account=EQDXAswEFIWNg6VTijwchiMYcvAGRTr3yCWln53RLWNq8CvQ<=195287000001&hash=7F09EC8EEBBAF510D7873CD3DEC494B2DD28A790A7A46E83D0A183F3E672B509> has no init (it is `init:nothing`), and a transaction with this message is aborted. But in general, the StateInit data attached to a message is used by the node only once: when a contract is in the Uninit state. Replays of such message can succeed, but the contract code will not be changed. An Init msg created with sol2tvm compiler and tvm_linker contains an encoded constructor call. Now you can call constructor multiple times, but in future releases it will be changed.


