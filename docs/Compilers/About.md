# Intro

The TON Labs Compiler Toolchain enables developers to compile source contract code in popular COP languages and GPLs into the TVM bytecode.

The Toolchain includes a Solidity to TVM (Sol2TVM) compiler and an LLVM-based compiler that now is ready to take sources code in C (C2TVM). Support of other GPLs and COP languages is planned. 

The compiler Toolchain also includes the so-called tvm-linker that packs code into binary format. 

Read further for more details on components, their roles and usage. 

Also, visit [TON Dev](https://ton.dev/) for more product, company and community info.

## Before your take the leap

To ensure that smart-contract is TVM-ready it has to compile to the virtual machine language (we will call it TVM Assembly for the documentation purposes) and support its value types, data management methods and primitives (instructions implemented to manipulate these values and implement methods).

In particular it is worth to mention TVM-specific primitives introduced to manipulate TVM persistent data storage containers or *Cells ,* intermediate storage containers or *Builders*, and sub-cells (*Slices*) ([TON Virtual Machine cl. 3.1, A. 4.2](/Ton Blockchain/TON Specification/TON Virtual Machine). For example, NEWC and ENDC Cell-level primitives start and end the *Cell* creation process.

TVM's key and most specific methods - *Serialization* and *Deserialization* - are, basically, construct cell (pack data) and extract cell data instructions.

**Note**: For exhaustive descriptions of the TVM and TON please, refer to original specifications.

To help developers, we developed high-level to low-level language compilers. Once fully implemented, our Toolchain will allow developers to reuse their current skills in our ecosystem.



> Visit [TON Dev](https://ton.dev/) for additional product, company & community info.


