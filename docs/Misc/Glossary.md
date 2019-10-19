# The Moonspeak

This brief set of terms and definitions is in no way exhaustive or final. It is based on personal experience with blockchain or related concepts and aims at making complex things a little more clear.

What makes blockchain talk complicated, is that this rapidly evolving phenomenon pools together multiple concepts, methodologies and ideas from numerous areas and domains from math to behaviorist studies. Considered separately, each item may be clear and logical, but once pooled together and interwoven, they are not easy to define and describe in plain well-structured phrases.

For example, explanation of the root term - Blockchain - can start from the basic notion of database, but just as well we can choose P2P networks as the starting place. Bringing both together within a single definition makes it long and more confusing.

But we keep on trying. Hope you will have a good time reading it. Feedback and suggestions are welcome.

## B

### Blockchain

Blockchain is basically a database for storing transactions packed in blocks. Blockchain has some technical particularities that make it ideal for fintech services but quite awkward for general DB uses: data in blockchain cannot be changed (i.e. transactions are unchangeable), all data is linked to its owner, all data meets some mandatory rules (i.e. it is consistent) and, ta-da, its decentralized. Blockchains rely on peer-to-peer interaction and require no third-party regulation, control or coordination. To ensure fairness, there is consensus (validation procedures that enable mining) and encryption technologies.

Another, broader, term for this type of databases is Distributed Ledger.

> **Tip**: for further reading we would suggesting articles about peer-to-peer networks, sharding, workchains, multi-chain models.

## C

### Cold Wallet

Cold wallet (aka hardware wallet) is a physical appliance (an nfc-card, a usb drive) where wallet private keys are stored. Cold wallets are considered more secure, as there is no way to hack them. Also, in some cases a cold wallet enables easier and faster access to transactions.

For example, with an nfc-card you only have to hover it over the back side of your device to start transactions. No more mess with passwords and pin-codes. 

### Consensus

Consensus is a protocol implemented within a blochchain to make sure that its nodes (engines that maintain the blockchain and may process transactions, depending on the architecture) agree on requirements to transactions to validate them.

Interestingly, validation enabled crypto-currency mining, as newly mined coins are the reward for validating a transaction.

With consensus in place, no centralized regulation is needed to prevent frauds. There are several popular consensus protocols: Proof of Work (PoW), Proof of Stake, Delegated Proof of Stake, etc. New consensus mechanisms evolve to reduce energy consumption associated with the PoW and optimize overall performance. Moreover, consensus without mining is being developed and researched.

### Crypto-currency

Surprise, surprise, but crypto-currency is not necessarily blockchain and blockchain is not necessarily crypto-currency. Technically speaking, to generate crypto-currency one needs a distributed ledger meeting two core requirements: decentralized storage and encryption technology. Yet, blockchain remains the most popular, because it enables mining (i.e. generating new currency units) as a reward for transaction validation thus reducing commissions imposed on the parties.

> **Tip**: to dig deeper, you might want to research the related concepts: fiat money, token, burning, mining vs. minting, Byzantine Generals Problem, TenderMint.

## D

### Dapp, Decentralized application

In short, it is an application (usually fintech) with a blockchain smart-contract behind it to ensure fairness and decentralization. An extended definition though has to cover the infrastructure a bit. A standard app either operates locally (e.g. a single-player game) or uses some standard server-client architecture (e.g. a bank app). In blockchain, every Dapp is basically a smart contract that describes transactions and rules applied to them. Roughly speaking, each smart contract has a server side that is stored in the blockchain and executed by the virtual machine (back-end for machine-to-machine interaction) and the client-side that is available to a user and has a GUI for human to machine interaction (front-end). The two parts interact via messaging (basically, requests to the contract and its responses). The front-end part can be implement in many popular programming languages, most popular are .js and .ts, Python is on the rise.

### Decentralized economy

In the real world anarchy turns out to be a disaster and everybody's business is known to be nobody's business. Communism failed and is dabbed a Utopian concept forever. In the digital world, however, blockchain is the game-changer when it comes to equal rights, fair play, data security, privacy, decentralization, democracy, transparency. You can have it all without any central authority, regulations and dictatorship. Hmm, is it as good as it sounds? Let's see.

Booming peer to peer networks with consensus methods allow parties to safely and transparently carry out financial (or other) transactions without any regulator. Being just a technical thing, P2P solutions (in particular, blockchain) have no geographical, social, political or gender bias.

## N

### Node

Basically, Node is a blockchain server that has a working instance of a virtual machine (VM) and a set of relevant instruments that ensure VM interaction with clients (i.e. for example, with customer applications). Every blockchain platform has its own messaging standards and protocols, let alone VM's, so node architectures differ across platforms. What's more even within a single platform variations may exist, although all nodes have to be compatible within a platform. 

> Visit [TON Dev](https://ton.dev/) for its Node SE offer.

## S

### Smart contract

Smart contract is not a legal document, rather, it is a piece of code deployed in blockchain. This code contains an executable sequence of operations. Each operation manipulates some available data (e.g. an account balance), retrieves it and\or changes it (e.g. transfers an amount). Each instance of smart contract execution is a blockchain transaction with block generation.

Smart contract is the core part of each Dapp (roughly speaking, it is its back end). Smart contracts may interact with each other.

There are several types of standard smart contracts that implement tokens. The most famous is ERC20.

## T

### Tokenomics

This concept does not take blockchain beyond crypto-currencies, but rather it expands the idea of crypto-currencies beyond money the way we have known it. In a solid ecosystem a token becomes more than a means of transaction settlement (buy, pay, transfer, invoice, etc.), it is an attitude, a vote, a stake. For example, you can use tokens to vote for some initiative. If it gets enough votes in tokens, your contribution will be written off your account and used to finance the initiative. This is a basic case, of course, many more can be invented. What makes tokenomics attractive, is the blockchain and smart-contract technology behind. These ensure decentralization, security, fairness, privacy and transparency at the same time.

## V

### Virtual machine

A VM is a blockchain core or engine that takes smart contracts compiled to a blockchain-specific bytecode and executes them (performs transactions). Between transactions smart contracts are stored in the blockchain with persistent data (i.e. the data that remains in the contract). TON VM is called TVM, Ethereum VM is EVM. You can find detailed specification of each in the Internet.

## W

### Web 3.0

The original concept suggested as far back as in 2007 by Jason Calacanis implies a switch from Web 2.0's powerful services and tools to a whole new interaction and work culture. Then it was proposed to define Web 3.0 as the Internet interaction with the physical world.

The latter idea, although criticized and over-used, resonates with our vision and with the decentralized economy concept. Indeed, blockchain has the potential to foster penetration of the Internet into the real world. Fintech services can be used to manage daily chores and bills, devices (IoT) and many more.

