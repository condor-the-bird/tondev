# Overview

## Accessible Toolkit

TON Labs SDK comes with tools and features helping [TON Dev](https://ton.dev/) developers to:

- Compile smart contracts into the TVC format

> See the [Toolchain documentation](/Compilers/About/) for detailed information about TON Labs compilers. For additional information on the TVC format, refer to the original TON blockchain documentation. 

- Deploy, run, test and debug smart contracts using a local Node Server (Node SE);
- Quickly and easily interact with blockchain data and track changes.

## **Components**

- **TON Labs Local Node** – Debug and test your smart contracts in a controlled environment with an instance that acts just like a production node.
- **TON Labs Compiler Kit** – Compile TVC files from Solidity source code and from C using our LLVM-based solution. 
- **TON Labs Client Libraries** – An open standard to develop smart contract and test them locally.
- TON Dev CLI that glues the components together and enables smooth installation and development. 

Each TON Labs Client Library includes:

- **Crypto** – TON-related cryptography functions.
- **Contracts** - smart contract deployment and management.
- **Queries** – monitoring and querying blockchain data in real time.

## Prerequisites 

To use the product, you need a machine that supports [Docker](https://docs.docker.com/compose/install/). All the necessary docker images are available at docker hub. You also have to install one of the newest stable versions of Node.js. 

Any supported development/runtime environment can run TON Labs Client Libraries installed with the proper package manager.

> **Warning**: Docker utilizes a built-in virtual machine management component called Hyper-V to run itself. It is not a feature of Windows Home edition. Therefore, the solution cannot be installed on Windows Home Edition. All the other Windows editions fit the required criteria. 

> **Tip**: If you are a Windows Home user, try installing a VM according to the doc [here](https://docs.ton.dev/86757ecb2/p/69f25e).

## Sources

You can use the installation procedure provided in the this guide to smoothly create your own development lab, but, in case you are interested, here are the links to sources:

- <https://github.com/tonlabs/ton-client-node-js>
- <https://github.com/tonlabs/ton-client-web-js>
- <https://github.com/tonlabs/ton-client-react-native-js>
- <https://github.com/tonlabs/ton-client-rs>
- <https://hub.docker.com/r/tonlabs/local-node>
- <https://hub.docker.com/r/tonlabs/compilers>
