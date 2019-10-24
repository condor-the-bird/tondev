# TON Whitepaper

## A Brief Description of TON Components

The Telegram Open Network (TON) is a combination of the following components: 

- A flexible multi-blockchain platform (TON Blockchain; cf. Chapter 2), capable of processing millions of transactions per second, with Turingcomplete smart contracts, upgradable formal blockchain specifications, multi-cryptocurrency value transfer, support for micropayment channels and off-chain payment networks. TON Blockchain presents some new and unique features, such as the “self-healing” vertical blockchain mechanism (cf. 2.1.17) and Instant Hypercube Routing (cf. 2.4.20), which enable it to be fast, reliable, scalable and self-consistent at the same time. 
- A peer-to-peer network (TON P2P Network, or just TON Network; cf. Chapter [3](https://zeroheight.com/86757ecb2/p/48c113)), used for accessing the TON Blockchain, sending transaction candidates, and receiving updates about only those parts of the blockchain a client is interested in (e.g., those related to the client’s accounts and smart contracts), but also able to support arbitrary distributed services, blockchain-related or not. 
- A distributed file storage technology (TON Storage; cf. 4.1.8), accessible through TON Network, used by the TON Blockchain to store archive copies of blocks and status data (snapshots), but also available for storing arbitrary files for users or other services running on the platform, with torrent-like access technology. 
- A network proxy/anonymizer layer (TON Proxy; cf. 4.1.11 and 3.1.6), similar to the I 2P (Invisible Internet Project), used to hide the identity and IP addresses of TON Network nodes if necessary (e.g., nodes committing transactions from accounts with large amounts of cryptocurrency, or high-stake blockchain validator nodes who wish to hide their exact IP address and geographical location as a measure against DDoS attacks). 
- A Kademlia-like distributed hash table (TON DHT; cf. 3.2), used as a “torrent tracker” for TON Storage (cf. 3.2.10), as an “input tunnel locator” for TON Proxy (cf. 3.2.14), and as a service locator for TON Services (cf. 3.2.12).
- A platform for arbitrary services (TON Services; cf. Chapter 4), residing in and available through TON Network and TON Proxy, with formalized interfaces (cf. 4.3.14) enabling browser-like or smartphone application interaction. These formal interfaces and persistent service entry points can be published in the TON Blockchain (cf. 4.3.17); actual nodes providing service at any given moment can be looked up through the TON DHT starting from information published in the TON Blockchain (cf. 3.2.12). Services may create smart contracts in the TON Blockchain to offer some guarantees to their clients (cf. 4.1.7). 
- TON DNS (cf. 4.3.1), a service for assigning human-readable names to accounts, smart contracts, services and network nodes. 
- TON Payments (cf. Chapter 5), a platform for micropayments, micropayment channels and a micropayment channel network. It can be used for fast off-chain value transfers, and for paying for services powered by TON Services. 
- TON will allow easy integration with third-party messaging and social networking applications, thus making blockchain technologies and distributed services finally available and accessible to ordinary users (cf. 4.3.24), rather than just to a handful of early cryptocurrency adopters. We will provide an example of such an integration in another of our projects, the Telegram Messenger (cf. 4.3.19). 

While the TON Blockchain is the core of the TON project, and the other components might be considered as playing a supportive role for the blockchain, they turn out to have useful and interesting functionality by themselves. Combined, they allow the platform to host more versatile applications than it would be possible by just using the TON Blockchain (cf. 2.9.13 and 4.1). 

## 1. Introduction

The aim of this text is to provide a first description of the Telegram Open Network (TON) and related blockchain, peer-to-peer, distributed storage and service hosting technologies. To reduce the size of this document to reasonable proportions, we focus mainly on the unique and defining features of the TON platform that are important for it to achieve its stated goals.

The Telegram Open Network (TON) is a fast, secure and scalable blockchain and network project, capable of handling millions of transactions per second if necessary, and both user-friendly and service provider-friendly. We aim for it to be able to host all reasonable applications currently proposed and conceived. One might think about TON as a huge distributed supercomputer, or rather a huge “superserver”, intended to host and provide a variety of services. This text is not intended to be the ultimate reference with respect to all implementation details. Some particulars are likely to change during the development and testing phases.

## 2. TON Blockchain

### 2.1 TON Blockchain as a Collection of 2-Blockchains 

The TON Blockchain is actually a collection of blockchains (even a collection of blockchains of blockchains, or 2-blockchains—this point will be clarified later in 2.1.17), because no single blockchain project is capable of achieving our goal of processing millions of transactions per second, as opposed to the now-standard dozens of transactions per second.

#### 2.1.1. List of blockchain types. 

The blockchains in this collection are: 

- The unique master blockchain, or masterchain for short, containing general information about the protocol and the current values of its parameters, the set of validators and their stakes, the set of currently active workchains and their “shards”, and, most importantly, the set of hashes of the most recent blocks of all workchains and shardchains. 
- Several (up to 2^32) working blockchains, or workchains for short, which are actually the “workhorses”, containing the value-transfer and smartcontract transactions. Different workchains may have different “rules”, meaning different formats of account addresses, different formats of transactions, different virtual machines (VMs) for smart contracts, different basic cryptocurrencies and so on. However, they all must satisfy certain basic interoperability criteria to make interaction between different workchains possible and relatively simple. In this respect, the TON Blockchain is heterogeneous (cf. 2.8.8), similarly to the EOS (cf. 2.9.7) and PolkaDot (cf. 2.9.8) projects. 
- Each workchain is in turn subdivided into up to 2^60 shard blockchains, or shardchains for short, having the same rules and block format as the workchain itself, but responsible only for a subset of accounts, depending on several first (most significant) bits of the account address. In other words, a form of sharding is built into the system (cf. 2.8.12). Because all these shardchains share a common block format and rules, the TON Blockchain is homogeneous in this respect (cf. 2.8.8), similarly to what has been discussed in one of Ethereum scaling proposals.*

> *[ https://github.com/ethereum/wiki/wiki/Sharding-FAQ](https://github.com/ethereum/wiki/wiki/Sharding-FAQ) 

- Each block in a shardchain (and in the masterchain) is actually not just a block, but a small blockchain. Normally, this “block blockchain” or “vertical blockchain” consists of exactly one block, and then we might think this is just the corresponding block of the shardchain (also called “horizontal blockchain” in this situation). However, if it becomes necessary to fix incorrect shardchain blocks, a new block is committed into the “vertical blockchain”, containing either the replacement for the invalid “horizontal blockchain” block, or a “block difference”, containing only a description of those parts of the previous version of this block that need to be changed. This is a TON-specific mechanism to replace detected invalid blocks without making a true fork of all shardchains involved; it will be explained in more detail in 2.1.17. For now, we just remark that each shardchain (and the masterchain) is not a conventional blockchain, but a blockchain of blockchains, or 2D-blockchain, or just a 2-blockchain. 

#### 2.1.2. Infinite Sharding Paradigm. 

Almost all blockchain sharding proposals are “top-down”: one first imagines a single blockchain, and then discusses how to split it into several interacting shardchains to improve performance and achieve scalability. 

The TON approach to sharding is “bottom-up”, explained as follows. Imagine that sharding has been taken to its extreme, so that exactly one account or smart contract remains in each shardchain. Then we have a huge number of “account-chains”, each describing the state and state transitions of only one account, and sending value-bearing messages to each other to transfer value and information. 

Of course, it is impractical to have hundreds of millions of blockchains, with updates (i.e., new blocks) usually appearing quite rarely in each of them. In order to implement them more efficiently, we group these “account-chains” into “shardchains”, so that each block of the shardchain is essentially a collection of blocks of account-chains that have been assigned to this shard. Thus the “account-chains” have only a purely virtual or logical existence inside the “shardchains”. 

We call this perspective the *Infinite Sharding Paradigm*. It explains many of the design decisions for the TON Blockchain.

#### 2.1.3. Messages. Instant Hypercube Routing. 

The Infinite Sharding Paradigm instructs us to regard each account (or smart contract) as if it were in its own shardchain by itself. Then the only way one account might affect the state of another is by sending a message to it (this is a special instance of the so-called Actor model, with accounts as Actors; cf. 2.4.2). Therefore, a system of messages between accounts (and shardchains, because the source and destination accounts are, generally speaking, located in different shardchains) is of paramount importance to a scalable system such as the TON Blockchain. In fact, a novel feature of the TON Blockchain, called Instant Hypercube Routing (cf. 2.4.20), enables it to deliver and process a message created in a block of one shardchain into the very next block of the destination shardchain, regardless of the total number of shardchains in the system. 

#### 2.1.4. Quantity of masterchains, workchains and shardchains. 

A TON Blockchain contains exactly one masterchain. However, the system can potentially accommodate up to 2^32 workchains, each subdivided into up to 2^60 shardchains. 

#### 2.1.5. Workchains can be virtual blockchains, not true blockchains.

Because a workchain is usually subdivided into shardchains, the existence of the workchain is “virtual”, meaning that it is not a true blockchain in the sense of the general definition provided in 2.2.1 below, but just a collection of shardchains. When only one shardchain corresponds to a workchain, this unique shardchain may be identified with the workchain, which in this case becomes a “true” blockchain, at least for some time, thus gaining a superficial similarity to customary single-blockchain design. However, the Infinite Sharding Paradigm (cf. 2.1.2) tells us that this similarity is indeed superficial: it is just a coincidence that the potentially huge number of “accountchains” can temporarily be grouped into one blockchain. 

#### 2.1.6. Identification of workchains. 

Each workchain is identified by its *number* or *workchain identifier (workchain_id : uint32)*, which is simply an unsigned 32-bit integer. Workchains are created by special transactions in the masterchain, defining the (previously unused) workchain identifier and the formal description of the workchain, sufficient at least for the interaction of this workchain with other workchains and for superficial verification of this workchain’s blocks. 

#### 2.1.7. Creation and activation of new workchains. 

The creation of a new workchain may be initiated by essentially any member of the community, ready to pay the (high) masterchain transaction fees required to publish the formal specification of a new workchain. However, in order for the new workchain to become active, a two-thirds consensus of validators is required, because they will need to upgrade their software to process blocks of the new workchain, and signal their readiness to work with the new workchain by special masterchain transactions. The party interested in the activation of the new workchain might provide some incentive for the validators to support the new workchain by means of some rewards distributed by a smart contract. 

#### 2.1.8. Identification of shardchains. 

Each shardchain is identified by a couple

` (w, s) = (workchain_id,shard_prefix)`, where workchain_id : uint32 identifies the corresponding workchain, and `shard_prefix : 2^0...60` is a bit string of length at most 60, defining the subset of accounts for which this shardchain is responsible. Namely, all accounts with account_id starting with `shard_prefix` (i.e., having `shard_prefix` as most significant bits) will be assigned to this shardchain.

#### 2.1.9. Identification of account-chains. 

Recall that account-chains have only a virtual existence (cf. 2.1.2). However, they have a natural identifier— namely, (workchain_id, account_id)—because any account-chain contains information about the state and updates of exactly one account (either a simple account or smart contract—the distinction is unimportant here).

#### 2.1.10. Dynamic splitting and merging of shardchains; cf. 2.7. 

A less sophisticated system might use static sharding—for example, by using the top eight bits of the `account_id` to select one of 256 pre-defined shards. 

An important feature of the TON Blockchain is that it implements dynamic sharding, meaning that the number of shards is not fixed. Instead, shard (w, s) can be automatically subdivided into shards` (w, s.0) `and `(w, s.1)` if some formal conditions are met (essentially, if the transaction load on the original shard is high enough for a prolonged period of time). Conversely, if the load stays too low for some period of time, the shards (w, s.0) and (w, s.1) can be automatically merged back into shard (w, s). Initially, only one shard (w, ∅) is created for workchain `*w*`. Later, it is subdivided into more shards, if and when this becomes necessary (cf. 2.7.6 and 2.7.8). 

#### 2.1.11. Basic workchain or Workchain Zero. 

While up to 2^32 workchains can be defined with their specific rules and transactions, we initially define only one, with `workchain_id = 0`. This workchain, called Workchain Zero or the basic workchain, is the one used to work with TON smart contracts and transfer TON coins, also known as Grams (cf. Appendix A). Most applications are likely to require only Workchain Zero. Shardchains of the basic workchain will be called basic shardchains. 

#### 2.1.12. Block generation intervals. 

We expect a new block to be generated in each shardchain and the masterchain approximately once every five seconds. This will lead to reasonably small transaction confirmation times. New blocks of all shardchains are generated approximately simultaneously; a new block of the masterchain is generated approximately one second later, because it must contain the hashes of the latest blocks of all shardchains. 

#### 2.1.13. Using the masterchain to make workchains and shardchains tightly coupled. 

Once the hash of a block of a shardchain is incorporated into a block of the masterchain, that shardchain block and all its ancestors are considered “canonical”, meaning that they can be referenced from the subsequent blocks of all shardchains as something fixed and immutable. In fact, each new shardchain block contains a hash of the most recent masterchain block, and all shardchain blocks referenced from that masterchain block are considered immutable by the new block. Essentially, this means that a transaction or a message committed in a shardchain block may be safely used in the very next blocks of the other shardchains, without needing to wait for, say, twenty confirmations (i.e., twenty blocks generated after the original block in the same blockchain) before forwarding a message or taking other actions based on a previous transaction, as is common in most proposed “loosely-coupled” systems (cf. 2.8.14), such as EOS. This ability to use transactions and messages in other shardchains a mere five seconds after being committed is one of the reasons we believe our “tightly-coupled” system, the first of its kind, will be able to deliver unprecedented performance (cf. 2.8.12 and 2.8.14). 

#### 2.1.14. Masterchain block hash as a global state. 

According to 2.1.13, the hash of the last masterchain block completely determines the overall state of the system from the perspective of an external observer. One does not need to monitor the state of all shardchains separately. 

#### 2.1.15. Generation of new blocks by validators; cf. [2.6](https://zeroheight.com/86757ecb2/p/822e19/t/24e919). 

The TON Blockchain uses a Proof-of-Stake (PoS) approach for generating new blocks in the shardchains and the masterchain. This means that there is a set of, say, up to a few hundred validators—special nodes that have deposited stakes (large amounts of TON coins) by a special masterchain transaction to be eligible for new block generation and validation. 

Then a smaller subset of validators is assigned to each shard `(w, s)` in a deterministic pseudorandom way, changing approximately every 1024 blocks. This subset of validators suggests and reaches consensus on what the next shardchain block would be, by collecting suitable proposed transactions from the clients into new valid block candidates. For each block, there is a pseudorandomly chosen order on the validators to determine whose block candidate has the highest priority to be committed at each turn. 

Validators and other nodes check the validity of the proposed block candidates; if a validator signs an invalid block candidate, it may be automatically punished by losing part or all of its stake, or by being suspended from the set of validators for some time. After that, the validators should reach consensus on the choice of the next block, essentially by an efficient variant of the BFT (Byzantine Fault Tolerant; cf. [2.8.4](https://zeroheight.com/86757ecb2/p/822e19/t/80f6d6)) consensus protocol, similar to PBFT [[4](https://zeroheight.com/86757ecb2/p/284c8c/t/78edfc)] or Honey Badger BFT [[11](https://zeroheight.com/86757ecb2/p/284c8c/t/49104f)]. If consensus is reached, a new block is created, and validators divide between themselves the transaction fees for the transactions included, plus some newly-created (“minted”) coins. Each validator can be elected to participate in several validator subsets; in this case, it is expected to run all validation and consensus algorithms in parallel. After all new shardchain blocks are generated or a timeout is passed, a new masterchain block is generated, including the hashes of the latest blocks of all shardchains. This is done by BFT consensus of all validators.* 

More detail on the TON PoS approach and its economical model is provided in section [2.6](https://zeroheight.com/86757ecb2/p/822e19/t/24e919).

> *Actually, two-thirds by stake is enough to achieve consensus, but an effort is made to collect as many signatures as possible.

#### 2.1.16. Forks of the masterchain. 

A complication that arises from our tightly-coupled approach is that switching to a different fork in the masterchain will almost necessarily require switching to another fork in at least some of the shardchains. On the other hand, as long as there are no forks in the masterchain, no forks in the shardchain are even possible, because no blocks in the alternative forks of the shardchains can become “canonical” by having their hashes incorporated into a masterchain block. 

The general rule is that *if masterchain block B' is a predecessor of B, B' includes hash Hash(B' w,s) of (w, s)-shardchain block B' w,s, and B includes hash Hash(Bw,s), then B'w,s* **must** *be a predecessor of Bw,s; otherwise, the masterchain block B is invalid.* 

We expect masterchain forks to be rare, next to non-existent, because in the BFT paradigm adopted by the TON Blockchain they can happen only in the case of incorrect behavior by a majority of validators (cf. [2.6.1](https://zeroheight.com/86757ecb2/p/822e19/t/20d26b) and [2.6.15](https://zeroheight.com/86757ecb2/p/822e19/t/2385e1)), which would imply significant stake losses by the offenders. Therefore, no true forks in the shardchains should be expected. Instead, if an invalid shardchain block is detected, it will be corrected by means of the “vertical blockchain” mechanism of the 2-blockchain (cf. [2.1.17](https://zeroheight.com/86757ecb2/p/822e19/t/83c8ba)), which can achieve this goal without forking the “horizontal blockchain” (i.e., the shardchain). The same mechanism can be used to fix non-fatal mistakes in the masterchain blocks as well. 

#### 2.1.17. Correcting invalid shardchain blocks. 

Normally, only valid shardchain blocks will be committed, because validators assigned to the shardchain must reach a two-thirds Byzantine consensus before a new block can be committed. However, the system must allow for detection of previously committed invalid blocks and their correction. Of course, once an invalid shardchain block is found—either by a validator (not necessarily assigned to this shardchain) or by a “fisherman” (any node of the system that made a certain deposit to be able to raise questions about block validity; cf. [2.6.4](https://zeroheight.com/86757ecb2/p/822e19/t/089c1e))—the invalidity claim and its proof are committed into the masterchain, and the validators that have signed the invalid block are punished by losing part of their stake and/or being temporarily suspended from the set of validators (the latter measure is important for the case of an attacker stealing the private signing keys of an otherwise benign validator). However, this is not sufficient, because the overall state of the system (TON Blockchain) turns out to be invalid because of the invalid shardchain block previously committed. This invalid block must be replaced by a newer valid version. 

Most systems would achieve this by “rolling back” to the last block before the invalid one in this shardchain and the last blocks unaffected by messages propagated from the invalid block in each of the other shardchains, and creating a new fork from these blocks. This approach has the disadvantage that a large number of otherwise correct and committed transactions are suddenly rolled back, and it is unclear whether they will be included later at all. 

The TON Blockchain solves this problem by making each “block” of each shardchain and of the masterchain (“horizontal blockchains”) a small blockchain (“vertical blockchain”) by itself, containing different versions of this “block”, or their “differences”. Normally, the vertical blockchain consists of exactly one block, and the shardchain looks like a classical blockchain. However, once the invalidity of a block is confirmed and committed into a masterchain block, the “vertical blockchain” of the invalid block is allowed to grow by a new block in the vertical direction, replacing or editing the invalid block. The new block is generated by the current validator subset for the shardchain in question. 

The rules for a new “vertical” block to be valid are quite strict. In particular, if a virtual “account-chain block” (cf. [2.1.2](https://zeroheight.com/86757ecb2/p/822e19/t/92b218)) contained in the invalid block is valid by itself, it must be left unchanged by the new vertical block. 

Once a new “vertical” block is committed on top of the invalid block, its hash is published in a new masterchain block (or rather in a new “vertical” block, lying above the original masterchain block where the hash of the invalid shardchain block was originally published), and the changes are propagated further to any shardchain blocks referring to the previous version of this block (e.g., those having received messages from the incorrect block). This is fixed by committing new “vertical” blocks in vertical blockchains for all blocks previously referring to the “incorrect” block; new vertical blocks will refer to the most recent (corrected) versions instead. Again, strict rules forbid changing account-chains that are not really affected (i.e., that receive the same messages as in the previous version). In this way, fixing an incorrect block generates “ripples” that are ultimately propagated towards the most recent blocks of all affected shardchains; these changes are reflected in new “vertical” masterchain blocks as well. 

Once the “history rewriting” ripples reach the most recent blocks, the new shardchain blocks are generated in one version only, being successors of the newest block versions only. This means that they will contain references to the correct (most recent) vertical blocks from the very beginning. 

The masterchain state implicitly defines a map transforming the hash of the first block of each “vertical” blockchain into the hash of its latest version. This enables a client to identify and locate any vertical blockchain by the hash of its very first (and usually the only) block.

#### 2.1.18. TON coins and multi-currency workchains. 

The TON Blockchain supports up to 2^32 different “cryptocurrencies”, “coins”, or “tokens”, distinguished by a 32-bit `currency_id`. New cryptocurrencies can be added by special transactions in the masterchain. Each workchain has a basic cryptocurrency, and can have several additional cryptocurrencies. 

There is one special cryptocurrency with `currency_id = 0`, namely, the TON coin, also known as the Gram (cf. Appendix A). It is the basic cryptocurrency of Workchain Zero. It is also used for transaction fees and validator stakes. 

In principle, other workchains may collect transaction fees in other tokens. In this case, some smart contract for automated conversion of these transaction fees into Grams should be provided. 

#### 2.1.19. Messaging and value transfer. 

Shardchains belonging to the same or different workchains may send *messages* to each other. While the exact form of the messages allowed depends on the receiving workchain and receiving account (smart contract), there are some common fields making inter-workchain messaging possible. In particular, each message may have some *value* attached, in the form of a certain amount of Grams (TON coins) and/or other registered cryptocurrencies, provided they are declared as acceptable cryptocurrencies by the receiving workchain. 

The simplest form of such messaging is a value transfer from one (usually not a smart-contract) account to another. 

#### 2.1.20. TON Virtual Machine. 

The TON Virtual Machine, also abbreviated as TON VM or TVM , is the virtual machine used to execute smart-contract code in the masterchain and in the basic workchain. Other workchains may use other virtual machines alongside or instead of the TVM. Here we list some of its features. They are discussed further in [2.3.12](https://zeroheight.com/86757ecb2/p/822e19/t/24fea2), [2.3.14](https://zeroheight.com/86757ecb2/p/822e19/t/01c684) and elsewhere. 

- TVM represents all data as a collection of (TVM) cells (cf. [2.3.14](https://zeroheight.com/86757ecb2/p/822e19/t/01c684)). Each cell contains up to 128 data bytes and up to 4 references to other cells. As a consequence of the “everything is a bag of cells” philosophy (cf. [2.5.14](https://zeroheight.com/86757ecb2/p/822e19/t/02317b)), this enables TVM to work with all data related to the TON Blockchain, including blocks and blockchain global state if necessary. 
- TVM can work with values of arbitrary algebraic data types (cf. [2.3.12](https://zeroheight.com/86757ecb2/p/822e19/t/24fea2)), represented as trees or directed acyclic graphs of TVM cells. However, it is agnostic towards the existence of algebraic data types; it just works with cells. 
- TVM has built-in support for hashmaps (cf. [2.3.7](https://zeroheight.com/86757ecb2/p/822e19/t/571bbe)). 
- TVM is a stack machine. Its stack keeps either 64-bit integers or cell references. 
- 64-bit, 128-bit and 256-bit arithmetic is supported. All n-bit arithmetic operations come in three flavors: for unsigned integers, for signed integers and for integers modulo 2^n (no automatic overflow checks in the latter case). 
- TVM has unsigned and signed integer conversion from n-bit to m-bit, for all 0 ≤ m, n ≤ 256, with overflow checks. 
- All arithmetic operations perform overflow checks by default, greatly simplifying the development of smart contracts. 
- TVM has “multiply-then-shift” and “shift-then-divide” arithmetic operations with intermediate values computed in a larger integer type; this simplifies implementing fixed-point arithmetic. 
- TVM offers support for bit strings and byte strings. 
- Support for 256-bit Elliptic Curve Cryptography (ECC) for some predefined curves, including Curve25519, is present. 
- Support for Weil pairings on some elliptic curves, useful for fast implementation of zk-SNARKs, is also present. 
- Support for popular hash functions, including sha256, is present. 
- TVM can work with Merkle proofs (cf. [5.1.9](https://zeroheight.com/86757ecb2/p/36f099/t/97550c)). 
- TVM offers support for “large” or “global” smart contracts. Such smart contracts must be aware of sharding (cf. [2.3.18](https://zeroheight.com/86757ecb2/p/822e19/t/870211) and [2.3.16](https://zeroheight.com/86757ecb2/p/822e19/t/7232ba)). Usual (local) smart contracts can be sharding-agnostic. 
- TVM supports closures. 
- A “spineless tagless G-machine” [[13](https://zeroheight.com/86757ecb2/p/724d6e/t/8599f7)] can be easily implemented inside TVM. 

Several high-level languages can be designed for TVM, in addition to the “TVM assembly”. All these languages will have static types and will support algebraic data types. We envision the following possibilities: 

- A Java-like imperative language, with each smart contract resembling a separate class. 
- A lazy functional language (think of Haskell). 
- An eager functional language (think of ML). 

#### 2.1.21. Configurable parameters. 

An important feature of the TON Blockchain is that many of its parameters are configurable. This means that they are part of the masterchain state, and can be changed by certain special proposal/vote/result transactions in the masterchain, without any need for hard forks. Changing such parameters will require collecting two-thirds of validator votes and more than half of the votes of all other participants who would care to take part in the voting process in favor of the proposal. 

### 2.2 Generalities on Blockchains

#### 2.2.1. General blockchain definition. 

In general, any *(true) blockchain* is a sequence of *blocks*, each *block B* containing a reference `blk-prev(B)` to the previous block (usually by including the hash of the previous block into the header of the current block), and a list of transactions. Each transaction describes some transformation of the *global blockchain state*; the transactions listed in a block are applied sequentially to compute the new state starting from the old state, which is the resulting state after the evaluation of the previous block.

#### 2.2.2. Relevance for the TON Blockchain. 

Recall that the TON Blockchain is not a true blockchain, but a collection of 2-blockchains (i.e., of blockchains of blockchains; cf. [2.1.1](https://zeroheight.com/86757ecb2/p/822e19/t/765115)), so the above is not directly applicable to it. However, we start with these generalities on true blockchains to use them as building blocks for our more sophisticated constructions. 

#### 2.2.3. Blockchain instance and blockchain type. 

One often uses the word *blockchain* to denote both a general *blockchain* type and its specific blockchain instances, defined as sequences of blocks satisfying certain conditions. For example, [2.2.1](https://zeroheight.com/86757ecb2/p/822e19/t/66b4bb) refers to blockchain instances. 

In this way, a blockchain type is usually a “subtype” of the type Block∗ of lists (i.e., finite sequences) of blocks, consisting of those sequences of blocks that satisfy certain compatibility and validity conditions: 

​                                        ` Blockchain ⊂ Block*`                                         (1)

A better way to define Blockchain would be to say that *Blockchain is a dependent couple type*, consisting of couples (B, v), with first component B : Block∗ being of type Block∗ (i.e., a list of blocks), and the second component v : isValidBc(B) being a proof or a witness of the validity of B. In this way, 

​                             `Blockchain ≡ Σ*(B:Block∗ )* isValidBc(B)  `             (2) 

We use here the notation for dependent sums of types borrowed from [[16](https://zeroheight.com/86757ecb2/p/724d6e/t/57d103)].

#### 2.2.4. Dependent type theory, Coq and TL. 

Note that we are using (Martin-Löf) dependent type theory here, similar to that used in the Coq* proof assistant. A simplified version of dependent type theory is also used in TL *(Type Language)*,** which will be used in the formal specification of the TON Blockchain to describe the serialization of all data structures and the layouts of blocks, transactions, and the like. 

In fact, dependent type theory gives a useful formalization of what a proof is, and such formal proofs (or their serializations) might become handy when one needs to provide proof of invalidity for some block, for example. 

> *[https://coq.inria.fr](https://coq.inria.fr/)

> **<https://core.telegram.org/mtproto/TL>

#### 2.2.5. TL, or the Type Language. 

Since TL (Type Language) will be used in the formal specifications of TON blocks, transactions, and network datagrams, it warrants a brief discussion. 

TL is a language suitable for description of dependent algebraic types,which are allowed to have numeric (natural) and type parameters. Each type is described by means of several constructors. Each constructor has a (human-readable) identifier and a name, which is a bit string (32-bit integer by default). Apart from that, the definition of a constructor contains a list of fields along with their types.

A collection of constructor and type definitions is called a TL-scheme. It is usually kept in one or several files with the suffix .tl. 

An important feature of TL-schemes is that they determine an unambiguous way of serializing and deserializing values (or objects) of algebraic types defined. Namely, when a value needs to be serialized into a stream of bytes, first the name of the constructor used for this value is serialized. Recursively computed serializations of each field follow.

The description of a previous version of TL, suitable for serializing arbitrary objects into sequences of 32-bit integers, is available at <https://core.telegram.org/mtproto/>TL. A new version of TL, called TL-B, is being developed for the purpose of describing the serialization of objects used by the TON Project. This new version can serialize objects into streams of bytes and even bits (not just 32-bit integers), and offers support for serialization into a tree of TVM cells (cf. [2.3.14](https://zeroheight.com/86757ecb2/p/822e19/t/01c684)). A description of TL-B will be a part of the formal specification of the TON Blockchain.

#### 2.2.6. Blocks and transactions as state transformation operators. 

Normally, any blockchain (type) *Blockchain* has an associated global state (type) State, and a transaction (type) Transaction. The semantics of a blockchain are to a large extent determined by the transaction application function: 

​                             `ev_trans0 : Transaction × State → State^?`                (3) 

Here X^? denotes Maybe X, the result of applying the Maybe monad to type X. This is similar to our use of X^∗ for `List X`. Essentially, a value of type `X^?` is either a value of type X or a special value ⊥ indicating the absence of an actual value (think about a null pointer). In our case, we use `State^?`instead of `State` as the result type because a transaction may be invalid if invoked from certain original states (think about attempting to withdraw from an account more money than it is actually there). We might prefer a curried version of ev_trans' : 

​                              `ev_trans : Transaction → State → State^?`                  (4)

Because a block is essentially a list of transactions, the block evaluation function 

​                              `ev_block : Block → State → State?    `                        (5) 

can be derived from ev_trans. It takes a block `*B : Block*` and the previous blockchain state` *s : State*` (which might include the hash of the previous block) and computes the next blockchain state s' = ev_block(B)(s) : *State*, which is either a true state or a special value ⊥ indicating that the next state cannot be computed (i.e., that the block is invalid if evaluated from the starting state given—for example, the block includes a transaction trying to debit an empty account.) 

#### 2.2.7. Block sequence numbers. 

Each block B in the blockchain can be referred to by its *sequence number* `blk-seqno(B)`, starting from zero for the very first block, and incremented by one whenever passing to the next block. More formally, 

​                             `blk-seqno(B) = blk-seqno (blk-prev(B)) + 1  `          (6) 

Notice that the sequence number does not identify a block uniquely in the presence of forks. 

#### 2.2.8. Block hashes. 

Another way of referring to a *block B* is by its hash `*blk-hash(B)*`, which is actually the hash of the *header* of block *B* (however, the header of the block usually contains hashes that depend on all content of block *B*). Assuming that there are no collisions for the hash function used (or at least that they are very improbable), a block is uniquely identified by its hash. 

#### 2.2.9. Hash assumption. 

During formal analysis of blockchain algorithms, we assume that there are no collisions for the k-bit hash function Hash : Bytes* → 2^k used: 

​             `Hash(s) = Hash(s') ⇒ s = s' for any s, s' ∈ Bytes*`             (7) 

Here Bytes = {0 . . . 255} = **2**^8 is the type of bytes, or the set of all byte values, and Bytes∗ is the type or set of arbitrary (finite) lists of bytes; while **2** = {0, 1} is the bit type, and **2^**k is the set (or actually the type) of all k-bit sequences (i.e., of k-bit numbers). 

Of course, (7) is impossible mathematically, because a map from an infinite set to a finite set cannot be injective. A more rigorous assumption would be 

​             ` ∀s, s' : s =/= s' , P (Hash(s) = Hash(s')) = 2^−k `            (8) 

However, this is not so convenient for the proofs. If (8) is used at most N times in a proof with 2 ^−k*N < ∈ for some small ∈ (say, ∈ = 10^−18), we can reason as if (7) were true, provided we accept a failure probability ∈ (i.e., the final conclusions will be true with probability at least 1 − ∈). 

Final remark: in order to make the probability statement of (8) really rigorous, one must introduce a probability distribution on the set *Bytes** of all byte sequences. A way of doing this is by assuming all byte sequences of the same length l equiprobable, and setting the probability of observing a sequence of length l equal to p^l − p^l+1 for some p → 1−. Then (8) should be understood as a limit of conditional probability `P (Hash(s) = Hash(s')|s =/= s') `when p tends to one from below. 

#### 2.2.10. Hash used for the TON Blockchain. 

We are using the 256-bit sha256 hash for the TON Blockchain for the time being. If it turns out to be weaker than expected, it can be replaced by another hash function in the future. The choice of the hash function is a configurable parameter of the protocol, so it can be changed without hard forks as explained in [2.1.21](https://zeroheight.com/86757ecb2/p/822e19/t/41c2fb)

### 2.3 Blockchain State, Accounts and Hashmaps

 We have noted above that any blockchain defines a certain global state, and each block and each transaction defines a transformation of this global state. Here we describe the global state used by TON blockchain

#### 2.3.1. Account IDs. 

The basic account IDs used by TON blockchains— or at least by its masterchain and Workchain Zero—are 256-bit integers, assumed to be public keys for 256-bit Elliptic Curve Cryptography (ECC) for a specific elliptic curve. In this way,

​                          `account_id : Account = uint256 = 2^256`                      (9) 

Here *Account* is the account type, while *account_id* : Account is a specific variable of type *Account*. 

Other workchains can use other account ID formats, 256-bit or otherwise. For example, one can use Bitcoin-style account IDs, equal to sha256 of an ECC public key. 

However, the bit length l of an account ID must be fixed during the creation of the workchain (in the masterchain), and it must be at least 64, because the first 64 bits of *account_id* are used for sharding and message routing.

#### 2.3.2. Main component: Hashmaps. 

The principal component of the TON blockchain state is a hashmap. In some cases we consider (partially defined) “maps” h : **2**^n --> **2**^m. More generally, we might be interested in hashmaps h : **2**^n --> X for a composite type X. However, the source (or index) type is almost always **2**^n . 

Sometimes, we have a “default value” empty : X, and the hashmap h : 2^n → X is “initialized” by its “default value” *i* → empty. 

#### 2.3.3. Example: TON account balances. 

An important example is given by TON account balances. It is a hashmap balance : 

​                                          `Account → uint*128*`                                              (10) 

mapping Account = 2^256 into a Gram (TON coin) balance of type uint128 = 2^128. This hashmap has a default value of zero, meaning that initially (before the first block is processed) the balance of all accounts is zero. 

#### 2.3.4. Example: smart-contract persistent storage. 

Another example is given by smart-contract persistent storage, which can be (very approximately) represented as a hashmap storage : 

​                                             ` 2^256 --> 2^256`                                              (11) 

This hashmap also has a default value of zero, meaning that uninitialized cells of persistent storage are assumed to be zero. 

#### 2.3.5. Example: persistent storage of all smart contracts. 

Because we have more than one smart contract, distinguished by *account_id*, each having its separate persistent storage, we must actually have a hashmap 

​                          `Storage : Account --> (2^256 -->  2^256)`                  (12) 

mapping *account_id* of a smart contract into its persistent storage.

#### 2.3.6. Hashmap type. 

The hashmap is not just an abstract (partially defined) function 2^n --> X; it has a specific representation. Therefore, we suppose that we have a special hashmap type 

​                                                `*Hashmap(n, X) : Type*`                                        (13) 

corresponding to a data structure encoding a (partial) map 2^n-->X. We can also write 

​                                      `Hashmap(n : nat)(X : Type) : Type`                         (14) 

or 

​                                           `Hashmap : nat → Type → Type`                                (15) 

We can always transform h : Hashmap(n, X) into a map hget(h) : 2^n → X^? . Henceforth, we usually write *h[i]* instead of hget(h)(i): 

​      `h[i] :≡ hget(h)(i) : X^? for any i : **2**^n , h : Hashmap(n, X)`    (16) 



#### 2.3.7. Definition of hashmap type as a Patricia tree. 

Logically, one might define *Hashmap(n, X)* as an (incomplete) binary tree of depth n with edge labels 0 and 1 and with values of type X in the leaves. Another way to describe the same structure would be as a *(bitwise) trie* for binary strings of length equal to n.

In practice, we prefer to use a compact representation of this trie, by compressing each vertex having only one child with its parent. The resulting representation is known as a Patricia tree or a binary radix tree. Each intermediate vertex now has exactly two children, labeled by two non-empty binary strings, beginning with zero for the left child and with one for the right child. 

In other words, there are two types of (non-root) nodes in a Patricia tree: 

- Leaf(x), containing value x of type X. 
- Node(l, sl , r, sr), where l is the (reference to the) left child or subtree, sl is the bitstring labeling the edge connecting this vertex to its left child (always beginning with 0), r is the right subtree, and sr is the bitstring labeling the edge to the right child (always beginning with 1). 

A third type of node, to be used only once at the root of the Patricia tree, is also necessary: 

- Root(n, s0, t), where n is the common length of index bitstrings of Hashmap(n, X), s0 is the common prefix of all index bitstrings, and t is a reference to a Leaf or a Node. 

If we want to allow the Patricia tree to be empty, a fourth type of (root) node would be used: 

- EmptyRoot(n), where n is the common length of all index bitstrings.

We define the height of a Patricia tree by height

`(Leaf(x))= 0`                                                                                           (17) 

`HEIGHT Node(*l, sl , r, sr*)= height(*l*) + len(*sl*) = height(*r*) + len(*sr*)`     (18) 

`HEIGHT Root(*n, s0, t*)= len(*s0*) + height(*t*) = *n*` (19) 

The last two expressions in each of the last two formulas must be equal. We use Patricia trees of height n to represent values of type *Hashmap(n, X)*. 

If there are N leaves in the tree (i.e., our hashmap contains *N* values), then there are exactly *N* − 1 intermediate vertices. Inserting a new value always involves splitting an existing edge by inserting a new vertex in the middle and adding a new leaf as the other child of this new vertex. Deleting a value from a hashmap does the opposite: a leaf and its parent are deleted, and the parent’s parent and its other child become directly linked

#### 2.3.8. Merkle-Patricia trees. 

When working with blockchains, we want to be able to compare Patricia trees (i.e., hash maps) and their subtrees, by reducing them to a single hash value. The classical way of achieving this is given by the Merkle tree. Essentially, we want to describe a way of hashing objects h of type Hashmap(n, X) with the aid of a hash function Hash defined for binary strings, provided we know how to compute hashes Hash(x) of objects x : X (e.g., by applying the hash function Hash to a binary serialization of object x). One might define Hash(h) recursively as follows: 

`Hash Leaf(x) := Hash(x)`                                                                              (20) 

`Hash Node(*l, sl , r, sr*) := Hash Hash(*l*). Hash(*r*). code(*sl*). code(*sr*)`  (21)

`Hash Root(*n, s0, t*) := Hash code(*n*). code(*s0*). Hash(*t*) `                (22) 

Here *s.t* denotes the concatenation of (bit) strings *s* and *t*, and code(*s*) is a prefix code for all bit strings *s*. For example, one might encode 0 by 10, 1 by 11, and the end of the string by 0.* 

We will see later (cf. [2.3.12](https://zeroheight.com/86757ecb2/p/822e19/t/24fea2) and [2.3.14](https://zeroheight.com/86757ecb2/p/822e19/t/01c684)) that this is a (slightly tweaked) version of recursively defined hashes for values of arbitrary (dependent) algebraic types.

> *One can show that this encoding is optimal for approximately half of all edge labels of a Patricia tree with random or consecutive indices. Remaining edge labels are likely to be long (i.e., almost 256 bits long). Therefore, a nearly optimal encoding for edge labels is to use the above code with prefix 0 for “short” bit strings, and encode 1, then nine bits containing length l = |s| of bitstring s, and then the l bits of s for “long” bitstrings (with l ≥ 10). 

#### 2.3.9. Recomputing Merkle tree hashes. 

This way of recursively defining Hash(h), called a Merkle tree hash, has the advantage that, if one explicitly stores Hash(h') along with each node h' (resulting in a structure called a *Merkle tree*, or, in our case, a *Merkle–Patricia tree*), one needs to recompute only at most *n* hashes when an element is added to, deleted from or changed in the hashmap. In this way, if one represents the global blockchain state by a suitable Merkle tree hash, it is easy to recompute this state hash after each transaction. 

#### 2.3.10. Merkle proofs. 

Under the assumption (7) of “injectivity” of the chosen hash function Hash, one can construct a proof that, for a given value z of Hash(h), h : Hashmap(n, X), one has` hget(h)(i) = x` for some i : 2^n and x : X. Such a proof will consist of the path in the Merkle–Patricia tree from the leaf corresponding to i to the root, augmented by the hashes of all siblings of all nodes occurring on this path. In this way, a light node* knowing only the value of Hash(h) for some hashmap h (e.g., smart-contract persistent storage or global blockchain state) might request from a full node** not only the value x = h[i] = hget(h)(i), but such a value along with a Merkle proof starting from the already known value Hash(h). Then, under assumption (7), the light node can check for itself that x is indeed the correct value of h[i]. 

In some cases, the client may want to obtain the value `y = Hash(x) = Hash(h[i])` instead—for example, if x itself is very large (e.g., a hashmap itself). Then a Merkle proof for (i, y) can be provided instead. If x is a hashmap as well, then a second Merkle proof starting from `y = Hash(x)` may be obtained from a full node, to provide a value x[j] = h[i][j] or just its hash. 

> *A light node is a node that does not keep track of the full state of a shardchain; instead, it keeps minimal information such as the hashes of the several most recent blocks, and relies on information obtained from full nodes when it becomes necessary to inspect some parts of the full state. 

> **A full node is a node keeping track of the complete up-to-date state of the shardchain in question. 

#### 2.3.11. Importance of Merkle proofs for a multi-chain system such as TON. 

Notice that a node normally cannot be a full node for all shardchains existing in the TON environment. It usually is a full node only for some shardchains—for instance, those containing its own account, a smart contract it is interested in, or those that this node has been assigned to be a validator of. For other shardchains, it must be a light node—otherwise the storage, computing and network bandwidth requirements would be prohibitive. This means that such a node cannot directly check assertions about the state of other shardchains; it must rely on Merkle proofs obtained from full nodes for those shardchains, which is as safe as checking by itself unless (7) fails (i.e., a hash collision is found).

#### 2.3.12. Peculiarities of TON VM. 

The TON VM or TVM (Telegram Virtual Machine), used to run smart contracts in the masterchain and Workchain Zero, is considerably different from customary designs inspired by the EVM (Ethereum Virtual Machine): it works not just with 256-bit integers, but actually with (almost) arbitrary “records”, “structures”, or “sum-product types”, making it more suitable to execute code written in high-level (especially functional) languages. Essentially, TVM uses tagged data types, not unlike those used in implementations of Prolog or Erlang. 

One might imagine first that the state of a TVM smart contract is not just a hashmap **2**^256 → **2**^256, or Hashmap(256, **2**^256), but (as a first step) Hashmap(256, X), where X is a type with several constructors, enabling it to store, apart from 256-bit integers, other data structures, including other hashmaps Hashmap(256, X) in particular. This would mean that a cell of TVM (persistent or temporary) storage—or a variable or an element of an array in a TVM smart-contract code—may contain not only an integer, but a whole new hashmap. Of course, this would mean that a cell contains not just 256 bits, but also, say, an 8-bit tag, describing how these 256 bits should be interpreted. 

In fact, values do not need to be precisely 256-bit. The value format used by TVM consists of a sequence of raw bytes and references to other structures, mixed in arbitrary order, with some descriptor bytes inserted in suitable locations to be able to distinguish pointers from raw data (e.g., strings or integers); cf. [2.3.14](https://zeroheight.com/86757ecb2/p/822e19/t/01c684). 

This raw value format may be used to implement arbitrary sum-product algebraic types. In this case, the value would contain a raw byte first, describing the “constructor” being used (from the perspective of a high-level language), and then other “fields” or “constructor arguments”, consisting of raw bytes and references to other structures depending on the constructor chosen (cf. [2.2.5](https://zeroheight.com/86757ecb2/p/822e19/t/174cd2)). 

However, TVM does not know anything about the correspondence between constructors and their arguments; the mixture of bytes and references is explicitly described by certain descriptor bytes.* The Merkle tree hashing is extended to arbitrary such structures: to compute the hash of such a structure, all references are recursively replaced by hashes of objects referred to, and then the hash of the resulting byte string (descriptor bytes included) is computed. 

In this way, the Merkle tree hashing for hashmaps, described in 2.3.8, is just a special case of hashing for arbitrary (dependent) algebraic data types, applied to type Hashmap(n, X) with two constructors.**

> *These two descriptor bytes, present in any TVM cell, describe only the total number of references and the total number of raw bytes; references are kept together either before or after all raw bytes. 

> **Actually, Leaf and Node are constructors of an auxiliary type, HashmapAux(n, X). Type Hashmap(n, X) has constructors Root and EmptyRoot, with Root containing a value of type HashmapAux(n, X).

#### 2.3.13. Persistent storage of TON smart contracts. 

Persistent storage of a TON smart contract essentially consists of its “global variables”, preserved between calls to the smart contract. As such, it is just a “product”, “tuple”, or “record” type, consisting of fields of the correct types, corresponding to one global variable each. If there are too many global variables, they cannot fit into one TON cell because of the global restriction on TON cell size. In such a case, they are split into several records and organized into a tree, essentially becoming a “product of products” or “product of products of products” type instead of just a product type. 

#### 2.3.14. TVM Cells. 

Ultimately, the TON VM keeps all data in a collection of (TVM) cells. Each cell contains two descriptor bytes first, indicating how many bytes of raw data are present in this cell (up to 128) and how many references to other cells are present (up to four). Then these raw data bytes and references follow. Each cell is referenced exactly once, so we might have included in each cell a reference to its “parent” (the only cell referencing this one). However, this reference need not be explicit. In this way, the persistent data storage cells of a TON smart contract are organized into a tree,* with a reference to the root of this tree kept in the smart-contract description. If necessary, a Merkle tree hash of this entire persistent storage is recursively computed, starting from the leaves and then simply replacing all references in a cell with the recursively computed hashes of the referenced cells, and subsequently computing the hash of the byte string thus obtained.

> *Logically; the “bag of cells” representation described in [2.5.5](https://zeroheight.com/86757ecb2/p/822e19/t/160b07) identifies all duplicate cells, transforming this tree into a directed acyclic graph (dag) when serialized.

#### 2.3.15. Generalized Merkle proofs for values of arbitrary algebraic types. 

Because the TON VM represents a value of arbitrary algebraic type by means of a tree consisting of (TVM) cells, and each cell has a well-defined (recursively computed) Merkle hash, depending in fact on the whole subtree rooted in this cell, we can provide “generalized Merkle proofs” for (parts of) values of arbitrary algebraic types, intended to prove that a certain subtree of a tree with a known Merkle hash takes a specific value or a value with a specific hash. This generalizes the approach of [2.3.10](https://zeroheight.com/86757ecb2/p/822e19/t/972551), where only Merkle proofs for x[i] = y have been considered.

#### 2.3.16. Support for sharding in TON VM data structures. 

We have just outlined how the TON VM, without being overly complicated, supports arbitrary (dependent) algebraic data types in high-level smart-contract languages. However, sharding of large (or global) smart contracts requires special support on the level of TON VM. To this end, a special version of the hashmap type has been added to the system, amounting to a “map” Account 9 --> X. This “map” might seem equivalent to Hashmap(m, X), where Account = 2^m. However, when a shard is split in two, or two shards are merged, such hashmaps are automatically split in two, or merged back, so as to keep only those keys that belong to the corresponding shard. 

#### 2.3.17. Payment for persistent storage. 

A noteworthy feature of the TON Blockchain is the payment exacted from smart contracts for storing their persistent data (i.e., for enlarging the total state of the blockchain). It works as follows: 

Each block declares two rates, nominated in the principal currency of the blockchain (usually the Gram): the price for keeping one cell in the persistent storage, and the price for keeping one raw byte in some cell of the persistent storage. Statistics on the total numbers of cells and bytes used by each account are stored as part of its state, so by multiplying these numbers by the two rates declared in the block header, we can compute the payment to be deducted from the account balance for keeping its data between the previous block and the current one. 

However, payment for persistent storage usage is not exacted for every account and smart contract in each block; instead, the sequence number of the block where this payment was last exacted is stored in the account data, and when any action is done with the account (e.g., a value transfer or a message is received and processed by a smart contract), the storage usage payment for all blocks since the previous such payment is deducted from the account balance before performing any further actions. If the account’s balance would become negative after this, the account is destroyed. 

A workchain may declare some number of raw data bytes per account to be “free” (i.e., not participating in the persistent storage payments) in order to make “simple” accounts, which keep only their balance in one or two cryptocurrencies, exempt from these constant payments. 

Notice that, if nobody sends any messages to an account, its persistent storage payments are not collected, and it can exist indefinitely. However, anybody can send, for instance, an empty message to destroy such an account. A small incentive, collected from part of the original balance of the account to be destroyed, can be given to the sender of such a message. We expect, however, that the validators would destroy such insolvent accounts for free, simply to decrease the global blockchain state size and to avoid keeping large amounts of data without compensation. 

Payments collected for keeping persistent data are distributed among the validators of the shardchain or the masterchain (proportionally to their stakes in the latter case).

#### 2.3.18. Local and global smart contracts; smart-contract instances. 

A smart contract normally resides just in one shard, selected according to the smart contract’s account_id, similarly to an “ordinary” account. This is usually sufficient for most applications. However, some “high-load” smart contracts may want to have an “instance” in each shardchain of some workchain. To achieve this, they must propagate their creating transaction into all shardchains, for instance, by committing this transaction into the “root” shardchain (w, ∅)* of the workchain w and paying a large commission.**

This action effectively creates instances of the smart contract in each shard, with separate balances. Originally, the balance transferred in the creating transaction is distributed simply by giving the instance in shard (w, s) the 2^−|s| part of the total balance. When a shard splits into two child shards, balances of all instances of global smart contracts are split in half; when two shards merge, balances are added together. In some cases, splitting/merging instances of global smart contracts may involve (delayed) execution of special methods of these smart contracts. By default, the balances are split and merged as described above, and some special “account-indexed” hashmaps are also automatically split and merged (cf. 2.3.16).

> *A more expensive alternative is to publish such a “global” smart contract in the masterchain. 

> **This is a sort of “broadcast” feature for all shards, and as such, it must be quite expensive.

#### 2.3.19. Limiting splitting of smart contracts. 

A global smart contract may limit its splitting depth *d* upon its creation, in order to make persistent storage expenses more predictable. This means that, if shardchain (w, s) with `|s| ≥ d` splits in two, only one of two new shardchains inherits an instance of the smart contract. This shardchain is chosen deterministically: each global smart contract has some *“account_id”*, which is essentially the hash of its creating transaction, and its instances have the same *account_id* with the first ≤ d bits replaced by suitable values needed to fall into the correct shard. This *account_id* selects which shard will inherit the smart-contract instance after splitting.

#### 2.3.20. Account/Smart-contract state. 

We can summarize all of the above to conclude that an account or smart-contract state consists of the following: 

- A balance in the principal currency of the blockchain 
- A balance in other currencies of the blockchain 
- Smart-contract code (or its hash)
- Smart-contract persistent data (or its Merkle hash) 
- Statistics on the number of persistent storage cells and raw bytes used
- The last time (actually, the masterchain block number) when payment for smart-contract persistent storage was collected 
- The public key needed to transfer currency and send messages from this account (optional; by default equal to *account_id* itself). In some cases, more sophisticated signature checking code may be located here, similar to what is done for Bitcoin transaction outputs; then the *account_id* will be equal to the hash of this code.

We also need to keep somewhere, either in the account state or in some other account-indexed hashmap, the following data:

- The output message queue of the account (cf. [2.4.17](https://zeroheight.com/86757ecb2/p/822e19/t/943eb5)) 
- The collection of (hashes of) recently delivered messages (cf. [2.4.23](https://zeroheight.com/86757ecb2/p/822e19/t/32f39c))

Not all of these are really required for every account; for example, smartcontract code is needed only for smart contracts, but not for “simple” accounts. Furthermore, while any account must have a non-zero balance in the principal currency (e.g., Grams for the masterchain and shardchains of the basic workchain), it may have balances of zero in other currencies. In order to avoid keeping unused data, a sum-product type (depending on the workchain) is defined (during the workchain’s creation), which uses different tag bytes (e.g., TL constructors; cf. [2.2.5](https://zeroheight.com/86757ecb2/p/822e19/t/174cd2)) to distinguish between different “constructors” used. Ultimately, the account state is itself kept as a collection of cells of the TVM persistent storage.

### 2.4 Messages Between Shardchains

An important component of the TON Blockchain is the *messaging system* between blockchains. These blockchains may be shardchains of the same workchain, or of different workchains. 

#### 2.4.1. Messages, accounts and transactions: a bird’s eye view of the system.

 Messages are sent from one account to another. Each *transaction* consists of an account receiving one message, changing its state according to certain rules, and generating several (maybe one or zero) new messages to other accounts. Each message is generated and received (delivered) exactly once. 

This means that messages play a fundamental role in the system, comparable to that of accounts (smart contracts). From the perspective of the Infinite Sharding Paradigm (cf. [2.1.2](https://zeroheight.com/86757ecb2/p/822e19/t/92b218)), each account resides in its separate “account-chain”, and the only way it can affect the state of some other account is by sending a message.

#### 2.4.2. Accounts as processes or actors; Actor model. 

One might think about accounts (and smart contracts) as “processes”, or “actors”, that are able to process incoming messages, change their internal state and generate some outbound messages as a result. This is closely related to the so-called *Actor model*, used in languages such as Erlang (however, actors in Erlang are usually called “processes”). Since new actors (i.e., smart contracts) are also allowed to be created by existing actors as a result of processing an inbound message, the correspondence with the Actor model is essentially complete. 

#### 2.4.3. Message recipient. 

Any message has its recipient, characterized by the *target workchain identifier w* (assumed by default to be the same as that of the originating shardchain), and the recipient account *account_id* . The exact format (i.e., number of bits) of *account_id* depends on *w*; however, the shard is always determined by its first (most significant) 64 bits. 

#### 2.4.4. Message sender. 

In most cases, a message has a *sender*, characterized again by a (*w' , account_id'* ) pair. If present, it is located after the message recipient and message value. Sometimes, the sender is unimportant or it is somebody outside the blockchain (i.e., not a smart contract), in which case this field is absent. 

Notice that the Actor model does not require the messages to have an implicit sender. Instead, messages may contain a reference to the Actor to which an answer to the request should be sent; usually it coincides with the sender. However, it is useful to have an explicit unforgeable sender field in a message in a cryptocurrency (Byzantine) environment.

#### 2.4.5. Message value. 

Another important characteristic of a message is its attached value, in one or several cryptocurrencies supported both by the source and by the target workchain. The value of the message is indicated at its very beginning immediately after the message recipient; it is essentially a list of (*currency_id, value*) pairs. 

Notice that “simple” value transfers between “simple” accounts are just empty (no-op) messages with some value attached to them. On the other hand, a slightly more complicated message body might contain a simple text or binary comment (e.g., about the purpose of the payment). 

#### 2.4.6. External messages, or “messages from nowhere”. 

Some messages arrive into the system “from nowhere”—that is, they are not generated by an account (smart contract or not) residing in the blockchain. The most typical example arises when a user wants to transfer some funds from an account controlled by her to some other account. In this case, the user sends a “message from nowhere” to her own account, requesting it to generate a message to the receiving account, carrying the specified value. If this message is correctly signed, her account receives it and generates the required outbound messages. 

In fact, one might consider a “simple” account as a special case of a smart contract with predefined code. This smart contract receives only one type of message. Such an inbound message must contain a list of outbound messages to be generated as a result of delivering (processing) the inbound message, along with a signature. The smart contract checks the signature, and, if it is correct, generates the required messages. 

Of course, there is a difference between “messages from nowhere” and normal messages, because the “messages from nowhere” cannot bear value, so they cannot pay for their “gas” (i.e., their processing) themselves. Instead, they are tentatively executed with a small gas limit before even being suggested for inclusion in a new shardchain block; if the execution fails (the signature is incorrect), the “message from nowhere” is deemed incorrect and is discarded. If the execution does not fail within the small gas limit, the message may be included in a new shardchain block and processed completely, with the payment for the gas (processing capacity) consumed exacted from the receiver’s account. “Messages from nowhere” can also define some transaction fee which is deducted from the receiver’s account on top of the gas payment for redistribution to the validators. 

In this sense, “messages from nowhere” or “external messages” take the role of transaction candidates used in other blockchain systems (e.g., Bitcoin and Ethereum). 

#### 2.4.7. Log messages, or “messages to nowhere”. 

Similarly, sometimes a special message can be generated and routed to a specific shardchain not to be delivered to its recipient, but to be logged in order to be easily observable by anybody receiving updates about the shard in question. These logged messages may be output in a user’s console, or trigger an execution of some script on an off-chain server. In this sense, they represent the external “output” of the “blockchain supercomputer”, just as the “messages from nowhere” represent the external “input” of the “blockchain supercomputer”.

#### 2.4.8. Interaction with off-chain services and external blockchains. 

These external input and output messages can be used for interacting with off-chain services and other (external) blockchains, such as Bitcoin or Ethereum. One might create tokens or cryptocurrencies inside the TON Blockchain pegged to Bitcoins, Ethers or any ERC-20 tokens defined in the Ethereum blockchain, and use “messages from nowhere” and “messages to nowhere”, generated and processed by scripts residing on some third-party off-chain servers, to implement the necessary interaction between the TON Blockchain and these external blockchains. 

#### 2.4.9 Message body

The message body is simply a sequence of bytes, the meaning of which is determined only by the receiving workchain and/or smart contract. For blockchains using TON VM, this could be the serialization of any TVM cell, generated automatically via the Send() operation. Such a serialization is obtained simply by recursively replacing all references in a TON VM cell with the cells referred to. Ultimately, a string of raw bytes appears, which is usually prepended by a 4-byte “message type” or “message constructor”, used to select the correct method of the receiving smart contract. Another option would be to use TL-serialized objects (cf. [2.2.5](https://zeroheight.com/86757ecb2/p/822e19/t/174cd2)) as message bodies. This might be especially useful for communication between different workchains, one or both of which are not necessarily using the TON VM.

Sometimes a message needs to carry information about the gas limit, the gas price, transaction fees and similar values that depend on the receiving workchain and are relevant only for the receiving workchain, but not necessarily for the originating workchain. Such parameters are included in or before the message body, sometimes (depending on the workchain) with special 4- byte prefixes indicating their presence (which can be defined by a TL-scheme; cf. [2.2.5](https://zeroheight.com/86757ecb2/p/822e19/t/174cd2)).

#### 2.4.10. Gas limit and other workchain/VM-specific parameters. 

Sometimes a message needs to carry information about the gas limit, the gas price, transaction fees and similar values that depend on the receiving workchain and are relevant only for the receiving workchain, but not necessarily for the originating workchain. Such parameters are included in or before the message body, sometimes (depending on the workchain) with special 4- byte prefixes indicating their presence (which can be defined by a TL-scheme; cf. [2.2.5](https://zeroheight.com/86757ecb2/p/822e19/t/174cd2)).

#### 2.4.11. Creating messages: smart contracts and transactions. 

There are two sources of new messages. Most messages are created during smartcontract execution (via the `Send()` operation in TON VM), when some smart contract is invoked to process an incoming message. Alternatively, messages may come from the outside as “external messages” or “messages from nowhere” (cf. [2.4.6](https://zeroheight.com/86757ecb2/p/822e19/t/09a14c)).*

> *The above needs to be literally true only for the basic workchain and its shardchains; other workchains may provide other ways of creating messages.

#### 2.4.12. Delivering messages. 

When a message reaches the shardchain containing its destination account,* it is “delivered” to its destination account. What happens next depends on the workchain; from an outside perspective, it is important that such a message can never be forwarded further from this shardchain. 

For shardchains of the basic workchain, delivery consists in adding the message value (minus any gas payments) to the balance of the receiving account, and possibly in invoking a message-dependent method of the receiving smart contract afterwards, if the receiving account is a smart contract. In fact, a smart contract has only one entry point for processing all incoming messages, and it must distinguish between different types of messages by looking at their first few bytes (e.g., the first four bytes containing a TL constructor; cf. [2.2.5](https://zeroheight.com/86757ecb2/p/822e19/t/174cd2)). 

> *As a degenerate case, this shardchain may coincide with the originating shardchain— for example, if we are working inside a workchain which has not yet been split. 

#### 2.4.13. Delivery of a message is a transaction. 

Because the delivery of a message changes the state of an account or smart contract, it is a special transaction in the receiving shardchain, and is explicitly registered as such. Essentially, all TON Blockchain transactions consist in the delivery of one inbound message to its receiving account (smart contract), neglecting some minor technical details. 

#### 2.4.14. Messages between instances of the same smart contract. 

Recall that a smart contract may be local (i.e., residing in one shardchain as any ordinary account does) or global (i.e., having instances in all shards, or at least in all shards up to some known depth d; cf. [2.3.18](https://zeroheight.com/86757ecb2/p/822e19/t/870211)). Instances of a global smart contract may exchange special messages to transfer information and value between each other if required. In this case, the (unforgeable) sender account_id becomes important (cf. [2.4.4](https://zeroheight.com/86757ecb2/p/822e19/t/21e3e2)).

#### 2.4.15. Messages to any instance of a smart contract; wildcard addresses. 

Sometimes a message (e.g., a client request) needs be delivered to any instance of a global smart contract, usually the closest one (if there is one residing in the same shardchain as the sender, it is the obvious candidate). One way of doing this is by using a “wildcard recipient address”, with the first d bits of the destination *account_id* allowed to take arbitrary values. In practice, one will usually set these d bits to the same values as in the sender’s *account_id*. 

#### 2.4.16. Input queue is absent. 

All messages received by a blockchain (usually a shardchain; sometimes the masterchain)—or, essentially, by an “account-chain” residing inside some shardchain—are immediately delivered (i.e., processed by the receiving account). Therefore, there is no “input queue” as such. Instead, if not all messages destined for a specific shardchain can be processed because of limitations on the total size of blocks and gas usage, some messages are simply left to accumulate in the output queues of the originating shardchains.

#### 2.4.17. Output queues. 

From the perspective of the Infinite Sharding Paradigm (cf. [2.1.2](https://zeroheight.com/86757ecb2/p/822e19/t/92b218)), each account-chain (i.e., each account) has its own output queue, consisting of all messages it has generated, but not yet delivered to their recipients. Of course, account-chains have only a virtual existence; they are grouped into shardchains, and a shardchain has an output “queue”, consisting of the union of the output queues of all accounts belonging to the shardchain. This shardchain output “queue” imposes only partial order on its member messages. Namely, a message generated in a preceding block must be delivered before any message generated in a subsequent block, and any messages generated by the same account and having the same destination must be delivered in the order of their generation.  

#### 2.4.18. Reliable and fast inter-chain messaging. 

It is of paramount importance for a scalable multi-blockchain project such as TON to be able to forward and deliver messages between different shardchains (cf. [2.1.3](https://zeroheight.com/86757ecb2/p/822e19/t/976318)), even if there are millions of them in the system. The messages should be delivered reliably (i.e., messages should not be lost or delivered more than once) and quickly. The TON Blockchain achieves this goal by using a combination of two “message routing” mechanisms. 

#### 2.4.19. Hypercube routing: “slow path” for messages with assured delivery. 

The TON Blockchain uses “hypercube routing” as a slow, but safe and reliable way of delivering messages from one shardchain to another, using several intermediate shardchains for transit if necessary. Otherwise, the validators of any given shardchain would need to keep track of the state of (the output queues of) all other shardchains, which would require prohibitive amounts of computing power and network bandwidth as the total quantity of shardchains grows, thus limiting the scalability of the system. Therefore, it is not possible to deliver messages directly from any shard to every other. Instead, each shard is “connected” only to shards differing in exactly one hexadecimal digit of their (w, s) shard identifiers (cf. [2.1.8](https://zeroheight.com/86757ecb2/p/822e19/t/26b537)). In this way, all shardchains constitute a “hypercube” graph, and messages travel along the edges of this hypercube. 

If a message is sent to a shard different from the current one, one of the hexadecimal digits (chosen deterministically) of the current shard identifier is replaced by the corresponding digit of the target shard, and the resulting identifier is used as the proximate target to forward the message to.* 

> *This is not necessarily the final version of the algorithm used to compute the next hop for hypercube routing. In particular, hexadecimal digits may be replaced by r-bit groups, with r a configurable parameter, not necessarily equal to four. 

The main advantage of hypercube routing is that the block validity conditions imply that validators creating blocks of a shardchain must collect and process messages from the output queues of “neighboring” shardchains, on pain of losing their stakes. In this way, any message can be expected to reach its final destination sooner or later; a message cannot be lost in transit or delivered twice. 

Notice that hypercube routing introduces some additional delays and expenses, because of the necessity to forward messages through several intermediate shardchains. However, the number of these intermediate shardchains grows very slowly, as the logarithm log N (more precisely, dlog`16` Ne − 1) of the total number of shardchains N. For example, if N ≈ 250, there will be at most one intermediate hop; and for N ≈ 4000 shardchains, at most two. With four intermediate hops, we can support up to one million shardchains. We think this is a very small price to pay for the essentially unlimited scalability of the system. In fact, it is not necessary to pay even this price: 

#### 2.4.20. Instant Hypercube Routing: “fast path” for messages. 

A novel feature of the TON Blockchain is that it introduces a “fast path” for forwarding messages from one shardchain to any other, allowing in most cases to bypass the “slow” hypercube routing of [2.4.19](https://zeroheight.com/86757ecb2/p/822e19/t/79694a) altogether and deliver the message into the very next block of the final destination shardchain. 

The idea is as follows. During the “slow” hypercube routing, the message travels (in the network) along the edges of the hypercube, but it is delayed (for approximately five seconds) at each intermediate vertex to be committed into the corresponding shardchain before continuing its voyage. 

To avoid unnecessary delays, one might instead relay the message along with a suitable Merkle proof along the edges of the hypercube, without waiting to commit it into the intermediate shardchains. In fact, the network message should be forwarded from the validators of the “task group” (cf. [2.6.8](https://zeroheight.com/86757ecb2/p/822e19/t/891cb0)) of the original shard to the designated block producer (cf. [2.6.9](https://zeroheight.com/86757ecb2/p/822e19/t/90fba1)) of the “task group” of the destination shard; this might be done directly without going along the edges of the hypercube. When this message with the Merkle proof reaches the validators (more precisely, the collators; cf. [2.6.5](https://zeroheight.com/86757ecb2/p/822e19/t/12a0aa)) of the destination shardchain, they can commit it into a new block immediately, without waiting for the message to complete its travel along the “slow path”. Then a confirmation of delivery along with a suitable Merkle proof is sent back along the hypercube edges, and it may be used to stop the travel of the message along the “slow path”, by committing a special transaction. 

Note that this “instant delivery” mechanism does not replace the “slow” but failproof mechanism described in [2.4.19](https://zeroheight.com/86757ecb2/p/822e19/t/79694a). The “slow path” is still needed because the validators cannot be punished for losing or simply deciding not to commit the “fast path” messages into new blocks of their blockchains.* 

Therefore, both message forwarding methods are run in parallel, and the “slow” mechanism is aborted only if a proof of success of the “fast” mechanism is committed into an intermediate shardchain.**

> *However, the validators have some incentive to do so as soon as possible, because they will be able to collect all forwarding fees associated with the message that have not yet been consumed along the slow path. 

> **In fact, one might temporarily or permanently disable the “instant delivery” mechanism altogether, and the system would continue working, albeit more slowly

#### 2.4.21. Collecting input messages from output queues of neighboring shardchains. 

When a new block for a shardchain is proposed, some of the output messages of the neighboring (in the sense of the routing hypercube of [2.4.19](https://zeroheight.com/86757ecb2/p/822e19/t/79694a)) shardchains are included in the new block as “input” messages and immediately delivered (i.e., processed). There are certain rules as to the order in which these neighbors’ output messages must be processed. Essentially, an “older” message (coming from a shardchain block referring to an older masterchain block) must be delivered before any “newer” message; and for messages coming from the same neighboring shardchain, the partial order of the output queue described in [2.4.17](https://zeroheight.com/86757ecb2/p/822e19/t/943eb5) must be observed. 

#### 2.4.22. Deleting messages from output queues. 

Once an output queue message is observed as having been delivered by a neighboring shardchain, it is explicitly deleted from the output queue by a special transaction. 

#### 2.4.23. Preventing double delivery of messages. 

To prevent double delivery of messages taken from the output queues of the neighboring shardchains, each shardchain (more precisely, each account-chain inside it) keeps the collection of recently delivered messages (or just their hashes) as part of its state. When a delivered message is observed to be deleted from the output queue by its originating neighboring shardchain (cf. [2.4.22](https://zeroheight.com/86757ecb2/p/822e19/t/6556d8)), it is deleted from the collection of recently delivered messages as well.

#### 2.4.24. Forwarding messages intended for other shardchains.

 Hypercube routing (cf. [2.4.19](https://zeroheight.com/86757ecb2/p/822e19/t/79694a)) means that sometimes outbound messages are delivered not to the shardchain containing the intended recipient, but to a neighboring shardchain lying on the hypercube path to the destination. In this case, “delivery” consists in moving the inbound message to the outbound queue. This is reflected explicitly in the block as a special forwarding transaction, containing the message itself. Essentially, this looks as if the message had been received by somebody inside the shardchain, and one identical message had been generated as result. 

#### 2.4.25. Payment for forwarding and keeping a message. 

The forwarding transaction actually spends some gas (depending on the size of the message being forwarded), so a gas payment is deducted from the value of the message being forwarded on behalf of the validators of this shardchain. This forwarding payment is normally considerably smaller than the gas payment exacted when the message is finally delivered to its recipient, even if the message has been forwarded several times because of hypercube routing. Furthermore, as long as a message is kept in the output queue of some shardchain, it is part of the shardchain’s global state, so a payment for keeping global data for a long time may be also collected by special transactions.

#### 2.4.26. Messages to and from the masterchain. 

Messages can be sent directly from any shardchain to the masterchain, and vice versa. However, gas prices for sending messages to and for processing messages in the masterchain are quite high, so this ability will be used only when truly necessary— for example, by the validators to deposit their stakes. In some cases, a minimal deposit (attached value) for messages sent to the masterchain may be defined, which is returned only if the message is deemed “valid” by the receiving party. 

Messages cannot be automatically routed through the masterchain. A message with *workchain_id* =/= −1 (−1 being the special *workchain_id* indicating the masterchain) cannot be delivered to the masterchain. 

In principle, one can create a message-forwarding smart contract inside the masterchain, but the price of using it would be prohibitive. 

#### 2.4.27. Messages between accounts in the same shardchain. 

In some cases, a message is generated by an account belonging to some shardchain, destined to another account in the same shardchain. For example, this happens in a new workchain which has not yet split into several shardchains because the load is manageable. 

Such messages might be accumulated in the output queue of the shardchain and then processed as incoming messages in subsequent blocks (any shard is considered a neighbor of itself for this purpose). However, in most cases it is possible to deliver these messages within the originating block itself. 

In order to achieve this, a partial order is imposed on all transactions included in a shardchain block, and the transactions (each consisting in the delivery of a message to some account) are processed respecting this partial order. 

In particular, a transaction is allowed to process some output message of a preceding transaction with respect to this partial order. In this case, the message body is not copied twice. Instead, the originating and the processing transactions refer to a shared copy of the message.

### 2.5 Global Shardchain State. “Bag of Cells” Philosophy.

Now we are ready to describe the global state of a TON blockchain, or at least of a shardchain of the basic workchain. We start with a “high-level” or “logical” description, which consists in saying that the global state is a value of algebraic type *ShardchainState*. 

#### 2.5.1. Shardchain state as a collection of account-chain states. 

According to the Infinite Sharding Paradigm (cf. 2.1.2), any shardchain is just a (temporary) collection of virtual “account-chains”, containing exactly one account each. This means that, essentially, the global shardchain state must be a hashmap 

​                         ` ShardchainState := (Account 99K AccountState) `        (23) 

where all account_id appearing as indices of this hashmap must begin with prefix s, if we are discussing the state of shard (w, s) (cf. [2.1.8](https://zeroheight.com/86757ecb2/p/822e19/t/34d916)).

 In practice, we might want to split *AccountState* into several parts (e.g., keep the account output message queue separate to simplify its examination by the neighboring shardchains), and have several hashmaps (Account --> *AccountStatePart i*) inside the *ShardchainState*. We might also add a small number of “global” or “integral” parameters to the ShardchainState, (e.g., the total balance of all accounts belonging to this shard, or the total number of messages in all output queues). 

However, (23) is a good first approximation of what the shardchain global state looks like, at least from a “logical” (“high-level”) perspective. The formal description of algebraic types *AccountState* and *ShardchainState* can be done with the aid of a TL-scheme (cf. [2.2.5](https://zeroheight.com/86757ecb2/p/822e19/t/174cd2)), to be provided elsewhere. 

#### 2.5.2. Splitting and merging shardchain states. 

Notice that the Infinite Sharding Paradigm description of the shardchain state (23) shows how this state should be processed when shards are split or merged. In fact, these state transformations turn out to be very simple operations with hashmaps. 

#### 2.5.3. Account-chain state. 

The (virtual) account-chain state is just the state of one account, described by type AccountState. Usually it has all or some of the fields listed in [2.3.20](https://zeroheight.com/86757ecb2/p/822e19/t/63e227), depending on the specific constructor used. 

#### 2.5.4. Global workchain state. 

Similarly to (23), we may define the global workchain state by the same formula, but with *account_id*’s allowed to take any values, not just those belonging to one shard. Remarks similar to those made in [2.5.1](https://zeroheight.com/86757ecb2/p/822e19/t/58a4b3) apply in this case as well: we might want to split this hashmap into several hashmaps, and we might want to add some “integral” parameters such as the total balance. Essentially, the global workchain state must be given by the same type ShardchainState as the shardchain state, because it is the shardchain state we would obtain if all existing shardchains of this workchain suddenly merged into one. 

#### 2.5.5. Low-level perspective: “bag of cells”. 

There is a “low-level” description of the account-chain or shardchain state as well, complementary to the “high-level” description given above. This description is quite important, because it turns out to be pretty universal, providing a common basis for representing, storing, serializing and transferring by network almost all data used by the TON Blockchain (blocks, shardchain states, smart-contract storage, Merkle proofs, etc.). At the same time, such a universal “low-level” description, once understood and implemented, allows us to concentrate our attention on the “high-level” considerations only. 

Recall that the TVM represents values of arbitrary algebraic types (including, for instance, *ShardchainState* of (23)) by means of a tree of TVM cells, or cells for short (cf. [2.3.14](https://zeroheight.com/86757ecb2/p/822e19/t/01c684) and [2.2.5](https://zeroheight.com/86757ecb2/p/822e19/t/174cd2)). 

Any such cell consists of two *descriptor bytes*, defining certain flags and values 0 ≤ b ≤ 128, the quantity of raw bytes, and 0 ≤ c ≤ 4, the quantity of references to other cells. Then *b* raw bytes and c cell references follow.* The exact format of cell references depends on the implementation and on whether the cell is located in RAM, on disk, in a network packet, in a block, and so on. A useful abstract model consists in imagining that all cells are kept in content-addressable memory, with the address of a cell equal to its (sha256) hash. Recall that the (Merkle) hash of a cell is computed exactly by replacing the references to its child cells by their (recursively computed) hashes and hashing the resulting byte string. 

In this way, if we use cell hashes to reference cells (e.g., inside descriptions of other cells), the system simplifies somewhat, and the hash of a cell starts to coincide with the hash of the byte string representing it. 

*Now we see that any object representable by TVM, the global shardchain state included*, can be represented as a *“bag of cells”*—i.e., a collection of cells along with a “root” reference to one of them (e.g., by hash). Notice that duplicate cells are removed from this description (the “bag of cells” is a set of cells, not a multiset of cells), so the abstract tree representation might actually become a directed acyclic graph (dag) representation. 

One might even keep this state on disk in a *B-* or *B+*-tree, containing all cells in question (maybe with some additional data, such as subtree height or reference counter), indexed by cell hash. However, a naive implementation of this idea would result in the state of one smart contract being scattered among distant parts of the disk file, something we would rather avoid.**

Now we are going to explain in some detail how almost all objects used by the TON Blockchain can be represented as “bags of cells”, thus demonstrating the universality of this approach.

> *One can show that, if Merkle proofs for all data stored in a tree of cells are needed equally often, one should use cells with b+ch ≈ 2(h+r) to minimize average Merkle proof size, where h = 32 is the hash size in bytes, and r ≈ 4 is the “byte size” of a cell reference. In other words, a cell should contain either two references and a few raw bytes, or one reference and about 36 raw bytes, or no references at all with 72 raw bytes. 

> **A better implementation would be to keep the state of the smart contract as a serialized string, if it is small, or in a separate B-tree, if it is large; then the top-level structure representing the state of a blockchain would be a B-tree, whose leaves are allowed to contain references to other B-trees. 

#### 2.5.6. Shardchain block as a “bag of cells”. 

A shardchain block itself can be also described by an algebraic type, and stored as a “bag of cells”. Then a naive binary representation of the block may be obtained simply by concatenating the byte strings representing each of the cells in the “bag of cells”, in arbitrary order. This representation might be improved and optimized, for instance, by providing a list of offsets of all cells at the beginning of the block, and replacing hash references to other cells with 32-bit indices in this list whenever possible. However, one should imagine that a block is essentially a “bag of cells”, and all other technical details are just minor optimization and implementation issues. 

#### 2.5.7. Update to an object as a “bag of cells”. 

Imagine that we have an old version of some object represented as a “bag of cells”, and that we want to represent a new version of the same object, supposedly not too different from the previous one. One might simply represent the new state as another “bag of cells” with its own root, a*nd remove from it all cells occurring in the old version*. The remaining “bag of cells” is essentially an *update* to the object. Everybody who has the old version of this object and the update can compute the new version, simply by uniting the two bags of cells, and removing the old root (decreasing its reference counter and de-allocating the cell if the reference counter becomes zero). 

#### 2.5.8. Updates to the state of an account. 

In particular, updates to the state of an account, or to the global state of a shardchain, or to any hashmap can be represented using the idea described in [2.5.7](https://zeroheight.com/86757ecb2/p/822e19/t/054154). This means that when we receive a new shardchain block (which is a “bag of cells”), we interpret this “bag of cells” not just by itself, but by uniting it first with the “bag of cells” representing the previous state of the shardchain. In this sense each block may “contain” the whole state of the blockchain. 

#### 2.5.9. Updates to a block. 

Recall that a block itself is a “bag of cells”, so, if it becomes necessary to edit a block, one can similarly define a “block update” as a “bag of cells”, interpreted in the presence of the “bag of cells” which is the previous version of this block. This is roughly the idea behind the “vertical blocks” discussed in [2.1.17](https://zeroheight.com/86757ecb2/p/822e19/t/83c8ba). 

#### 2.5.10. Merkle proof as a “bag of cells”. 

Notice that a (generalized) Merkle proof—for example, one asserting that *x[i] = y* starting from a known value of `Hash(x) = h` (cf. [2.3.10](https://zeroheight.com/86757ecb2/p/822e19/t/972551) and 2.3.15)—may also be represented as a “bag of cells”. Namely, one simply needs to provide a subset of cells corresponding to a path from the root of                `x : Hashmap(n, X)` to its desired leaf with index `i : 2^n` and value y : X. References to children of these cells not lying on this path will be left “unresolved” in this proof, represented by cell hashes. One can also provide a simultaneous Merkle proof of, say, `*x[i] = y*` and `*x[i'] = y'*` , by including in the “bag of cells” the cells lying on the union of the two paths from the root of *x* to leaves corresponding to indices *i* and *i'*. 

#### 2.5.11. Merkle proofs as query responses from full nodes. 

In essence, a full node with a complete copy of a shardchain (or account-chain) state can provide a Merkle proof when requested by a light node (e.g., a network node running a light version of the TON Blockchain client), enabling the receiver to perform some simple queries without external help, using only the cells provided in this Merkle proof. The light node can send its queries in a serialized format to the full node, and receive the correct answers with Merkle proofs—or just the Merkle proofs, because the requester should be able to compute the answers using only the cells included in the Merkle proof. This Merkle proof would consist simply of a “bag of cells”, containing only those cells belonging to the shardchain’s state that have been accessed by the full node while executing the light node’s query. This approach can be used in particular for executing “get queries” of smart contracts (cf. [4.3.12](https://zeroheight.com/86757ecb2/p/88e36d/t/851a8b)).

#### 2.5.12. Augmented update, or state update with Merkle proof of validity. 

Recall (cf.[ 2.5.7](https://zeroheight.com/86757ecb2/p/822e19/t/054154)) that we can describe the changes in an object state from an old value *x : X* to a new value *x' : X* by means of an “update”, which is simply a “bag of cells”, containing those cells that lie in the subtree representing new value *x',* but not in the subtree representing old value x, because the receiver is assumed to have a copy of the old value x and all its cells.

 However, if the receiver does not have a full copy of *x*, but knows only its (Merkle) hash `h = Hash(x)`, it will not be able to check the validity of the update (i.e., that all “dangling” cell references in the update do refer to cells present in the tree of x). One would like to have “verifiable” updates, augmented by Merkle proofs of existence of all referred cells in the old state. Then anybody knowing only `h = Hash(x)` would be able to check the validity of the update and compute the new `h'= Hash(x')` by itself. 

Because our Merkle proofs are “bags of cells” themselves (cf. [2.5.10](https://zeroheight.com/86757ecb2/p/822e19/t/440240)), one can construct such an augmented update as a “bag of cells”, containing the old root of x, some of its descendants along with paths from the root of *x* to them, and the new root of *x'* and all its descendants that are not part of x. 

#### 2.5.13. Account state updates in a shardchain block. 

In particular, account state updates in a shardchain block should be augmented as discussed in [2.5.12](https://zeroheight.com/86757ecb2/p/822e19/t/88a63b). Otherwise, somebody might commit a block containing an invalid state update, referring to a cell absent in the old state; proving the invalidity of such a block would be problematic (how is the challenger to prove that a cell is *not* part of the previous state?). Now, if all state updates included in a block are augmented, their validity is easily checked, and their invalidity is also easily shown as a violation of the recursive defining property of (generalized) Merkle hashes. 

#### 2.5.14. “Everything is a bag of cells” philosophy. 

Previous considerations show that everything we need to store or transfer, either in the TON Blockchain or in the network, is representable as a “bag of cells”. This is an important part of the TON Blockchain design philosophy. Once the “bag of cells” approach is explained and some “low-level” serializations of “bags of cells” are defined, one can simply define everything (block format, shardchain and account state, etc.) on the high level of abstract (dependent) algebraic data types. The unifying effect of the “everything is a bag of cells” philosophy considerably simplifies the implementation of seemingly unrelated services; cf. [5.1.9](https://zeroheight.com/86757ecb2/p/36f099/t/97550c) for an example involving payment channels. 

#### 2.5.15. Block “headers” for TON blockchains.

Usually, a block in a blockchain begins with a small header, containing the hash of the previous block, its creation time, the Merkle hash of the tree of all transactions contained in the block, and so on. Then the block hash is defined to be the hash of this small block header. Because the block header ultimately depends on all data included in the block, one cannot alter the block without changing its hash. 

In the “bag of cells” approach used by the blocks of TON blockchains, there is no designated block header. Instead, the block hash is defined as the (Merkle) hash of the root cell of the block. Therefore, the top (root) cell of the block might be considered a small “header” of this block. 

However, the root cell might not contain all the data usually expected from such a header. Essentially, one wants the header to contain some of the fields defined in the *Block* datatype. Normally, these fields will be contained in several cells, including the root. These are the cells that together constitute a “Merkle proof” for the values of the fields in question. One might insist that a block contain these “header cells” in the very beginning, before any other cells. Then one would need to download only the first several bytes of a block serialization in order to obtain all of the “header cells”, and to learn all of the expected fields. 

### 2.6 Creating and Validating New Blocks 

The TON Blockchain ultimately consists of shardchain and masterchain blocks. These blocks must be created, validated and propagated through the network to all parties concerned, in order for the system to function smoothly and correctly.

#### 2.6.1. Validators. 

New blocks are created and validated by special designated nodes, called *validators*. Essentially, any node wishing to become a validator may become one, provided it can deposit a sufficiently large stake (in TON coins, i.e., Grams; cf. Appendix [A](https://zeroheight.com/86757ecb2/p/851751/t/021e4c)) into the masterchain. Validators obtain some “rewards” for good work, namely, the transaction, storage and gas fees from all transactions (messages) committed into newly generated blocks, and some newly minted coins, reflecting the “gratitude” of the whole community to the validators for keeping the TON Blockchain working. This income is distributed among all participating validators proportionally to their stakes. 

However, being a validator is a high responsibility. If a validator signs an invalid block, it can be punished by losing part or all of its stake, and by being temporarily or permanently excluded from the set of validators. If a validator does not participate in creating a block, it does not receive its share of the reward associated with that block. If a validator abstains from creating new blocks for a long time, it may lose part of its stake and be suspended or permanently excluded from the set of validators.

All this means that the validator does not get its money “for nothing”. Indeed, it must keep track of the states of all or some shardchains (each validator is responsible for validating and creating new blocks in a certain subset of shardchains), perform all computations requested by smart contracts in these shardchains, receive updates about other shardchains and so on. This activity requires considerable disk space, computing power and network bandwidth.

#### 2.6.2. Validators instead of miners. 

Recall that the TON Blockchain uses the Proof-of-Stake approach, instead of the Proof-of-Work approach adopted by Bitcoin, the current version of Ethereum, and most other cryptocurrencies. This means that one cannot “mine” a new block by presenting some proof-ofwork (computing a lot of otherwise useless hashes) and obtain some new coins as a result. Instead, one must become a validator and spend one’s computing resources to store and process TON Blockchain requests and data. In short, one must be a validator to mine new coins. In this respect, validators are the new miners. However, there are some other ways to earn coins apart from being a validator. 

#### 2.6.3. Nominators and “mining pools”. 

To become a validator, one would normally need to buy and install several high-performance servers and acquire a good Internet connection for them. This is not so expensive as the ASIC equipment currently required to mine Bitcoins. However, one definitely cannot mine new TON coins on a home computer, let alone a smartphone. In the Bitcoin, Ethereum and other Proof-of-Work cryptocurrency mining communities there is a notion of mining pools, where a lot of nodes, having insufficient computing power to mine new blocks by themselves, combine their efforts and share the reward afterwards. A corresponding notion in the Proof-of-Stake world is that of a nominator. Essentially, this is a node lending its money to help a validator increase its stake; the validator then distributes the corresponding share of its reward (or some previously agreed fraction of it—say, 50%) to the nominator.

In this way, a nominator can also take part in the “mining” and obtain some reward proportional to the amount of money it is willing to deposit for this purpose. It receives only a fraction of the corresponding share of the validator’s reward, because it provides only the “capital”, but does not need to buy computing power, storage and network bandwidth. However, if the validator loses its stake because of invalid behavior, the nominator loses its share of the stake as well. In this sense the nominator shares the risk. It must choose its nominated validator wisely, otherwise it can lose money. In this sense, nominators make a weighted decision and “vote” for certain validators with their funds. On the other hand, this nominating or lending system enables one to become a validator without investing a large amount of money into Grams (TON coins) first. In other words, it prevents those keeping large amounts of Grams from monopolizing the supply of validators.

#### 2.6.4. Fishermen: obtaining money by pointing out others’ mistakes. 

Another way to obtain some rewards without being a validator is by becoming a fisherman. Essentially, any node can become a fisherman by making a small deposit in the masterchain. Then it can use special masterchain transactions to publish (Merkle) invalidity proofs of some (usually shardchain) blocks previously signed and published by validators. If other validators agree with this invalidity proof, the offending validators are punished (by losing part of their stake), and the fisherman obtains some reward (a fraction of coins confiscated from the offending validators). Afterwards, the invalid (shardchain) block must be corrected as outlined in [2.1.17](https://zeroheight.com/86757ecb2/p/822e19/t/83c8ba). Correcting invalid masterchain blocks may involve creating “vertical” blocks on top of previously committed masterchain blocks (cf. [2.1.17](https://zeroheight.com/86757ecb2/p/822e19/t/83c8ba)); there is no need to create a fork of the masterchain.

Normally, a fisherman would need to become a full node for at least some shardchains, and spend some computing resources by running the code of at least some smart contracts. While a fisherman does not need to have as much computing power as a validator, we think that a natural candidate to become a fisherman is a would-be validator that is ready to process new blocks, but has not yet been elected as a validator (e.g., because of a failure to deposit a sufficiently large stake). 

#### 2.6.5. Collators: obtaining money by suggesting new blocks to validators. 

Yet another way to obtain some rewards without being a validator is by becoming a *collator*. This is a node that prepares and suggests to a validator new shardchain block candidates, complemented (collated) with data taken from the state of this shardchain and from other (usually neighboring) shardchains, along with suitable Merkle proofs. (This is necessary, for example, when some messages need to be forwarded from neighboring shardchains.) Then a validator can easily check the proposed block candidate for validity, without having to download the complete state of this or other shardchains. 

Because a validator needs to submit new (collated) block candidates to obtain some (“mining”) rewards, it makes sense to pay some part of the reward to a collator willing to provide suitable block candidates. In this way, a validator may free itself from the necessity of watching the state of the neighboring shardchains, by outsourcing it to a collator. 

However, we expect that during the system’s initial deployment phase there will be no separate designated collators, because all validators will be able to act as collators for themselves.

#### 2.6.6. Collators or validators: obtaining money for including user transactions. 

Users can open micropayment channels to some collators or validators and pay small amounts of coins in exchange for the inclusion of their transactions in the shardchain

#### 2.6.7. Global validator set election. 

The “global” set of validators is elected once each month (actually, every 2^19 masterchain blocks). This set is determined and universally known one month in advance. In order to become a validator, a node must transfer some TON coins (Grams) into the masterchain, and then send them to a special smart contract as its suggested stake s. Another parameter, sent along with the stake, is `l ≥ 1`, the maximum validating load this node is willing to accept relative to the minimal possible. There is also a global upper bound (another configurable parameter) L on l, equal to, say, 10. Then the global set of validators is elected by this smart contract, simply by selecting up to T candidates with maximal suggested stakes and publishing their identities. Originally, the total number of validators is T = 100; we expect it to grow to 1000 as the load increases. It is a configurable parameter (cf. [2.1.21](https://zeroheight.com/86757ecb2/p/822e19/t/41c2fb)). The actual stake of each validator is computed as follows: If the top T proposed stakes are `s1 ≥ s2 ≥ · · · ≥ sT` , the actual stake of i-th validator is set to `s ' i := min(si , li · sT )`. In this way, `s'i /s'T ≤ li `, so the i-th validator does not obtain more than li ≤ L times the load of the weakest validator (because the load is ultimately proportional to the stake). 

Then elected validators may withdraw the unused part of their stake, si−s'i. Unsuccessful validator candidates may withdraw all of their proposed stake. Each validator publishes its public signing key, not necessarily equal to the public key of the account the stake came from.*

The stakes of the validators are frozen until the end of the period for which they have been elected, and one month more, in case new disputes arise (i.e., an invalid block signed by one of these validators is found). After that, the stake is returned, along with the validator’s share of coins minted and fees from transactions processed during this time. 

> *It makes sense to generate and use a new key pair for every validator election.

#### 2.6.8. Election of validator “task groups”. 

The whole global set of validators (where each validator is considered present with multiplicity equal to its stake—otherwise a validator might be tempted to assume several identities and split its stake among them) is used only to validate new masterchain blocks. The shardchain blocks are validated only by specially selected subsets of validators, taken from the global set of validators chosen as described in [2.6.7](https://zeroheight.com/86757ecb2/p/822e19/t/33f0f6). 

These validator “subsets” or “task groups”, defined for every shard, are rotated each hour (actually, every 2^10 masterchain blocks), and they are known one hour in advance, so that every validator knows which shards it will need to validate, and can prepare for that (e.g., by downloading missing shardchain data). 

The algorithm used to select validator task groups for each shard (w, s) is deterministic pseudorandom. It uses pseudorandom numbers embedded by validators into each masterchain block (generated by a consensus using threshold signatures) to create a random seed, and then computes for example `Hash(code(w). code(s).validator_id.rand_seed)` for each validator. Then validators are sorted by the value of this hash, and the first several are selected, so as to have at least 20/T of the total validator stakes and consist of at least 5 validators. This selection could be done by a special smart contract. In that case, the selection algorithm would easily be upgradable without hard forks by the voting mechanism mentioned in [2.1.21](https://zeroheight.com/86757ecb2/p/822e19/t/41c2fb). All other “constants” mentioned so far (such as 2^19 , 2^10 , T, 20, and 5) are also configurable parameters.

#### 2.6.9. Rotating priority order on each task group. 

There is a certain “priority” order imposed on the members of a shard task group, depending on the hash of the previous masterchain block and (shardchain) block sequence number. This order is determined by generating and sorting some hashes as described above. When a new shardchain block needs to be generated, the shard task group validator selected to create this block is normally the first one with respect to this rotating “priority” order. If it fails to create the block, the second or third validator may do it. Essentially, all of them may suggest their block candidates, but the candidate suggested by the validator having the highest priority should win as the result of Byzantine Fault Tolerant (BFT) consensus protocol.

#### 2.6.10. Propagation of shardchain block candidates. 

Because shardchain task group membership is known one hour in advance, their members can use that time to build a dedicated “shard validators multicast overlay network”, using the general mechanisms of the TON Network (cf. [3.3](https://zeroheight.com/86757ecb2/p/48c113/t/529b3a)). When a new shardchain block needs to be generated—normally one or two seconds after the most recent masterchain block has been propagated—everybody knows who has the highest priority to generate the next block (cf. 2.6.9). This validator will create a new collated block candidate, either by itself or with the aid of a collator (cf. 2.6.5). The validator must check (validate) this block candidate (especially if it has been prepared by some collator) and sign it with its (validator) private key. Then the block candidate is propagated to the remainder of the task group using the prearranged multicast overlay network (the task group creates its own private overlay network as explained in [3.3](https://zeroheight.com/86757ecb2/p/48c113/t/529b3a), and then uses a version of the streaming multicast protocol described in 3.3.15 to propagate block candidates). 

A truly BFT way of doing this would be to use a Byzantine multicast protocol, such as the one used in Honey Badger BFT [[11](https://zeroheight.com/86757ecb2/p/284c8c/t/49104f)]: encode the block candidate by an (N, 2N/3)-erasure code, send 1/N of the resulting data directly to each member of the group, and expect them to multicast directly their part of the data to all other members of the group. 

However, a faster and more straightforward way of doing this (cf. also [3.3.15](https://zeroheight.com/86757ecb2/p/48c113/t/89e62d)) is to split the block candidate into a sequence of signed one-kilobyte blocks (“chunks”), augment their sequence by a Reed–Solomon or a fountain code (such as the RaptorQ code [[9](https://zeroheight.com/86757ecb2/p/284c8c/t/239f9b)] [[14](https://zeroheight.com/86757ecb2/p/284c8c/t/55de6e)]), and start transmitting chunks to the neighbors in the “multicast mesh” (i.e., the overlay network), expecting them to propagate these chunks further. Once a validator obtains enough chunks to reconstruct the block candidate from them, it signs a confirmation receipt and propagates it through its neighbors to the whole of the group. Then its neighbors stop sending new chunks to it, but may continue to send the (original) signatures of these chunks, believing that this node can generate the subsequent chunks by applying the Reed–Solomon or fountain code by itself (having all data necessary), combine them with signatures, and propagate to its neighbors that are not yet ready. If the “multicast mesh” (overlay network) remains connected after removing all “bad” nodes (recall that up to one-third of nodes are allowed to be bad in a Byzantine way, i.e., behave in arbitrary malicious fashion), this algorithm will propagate the block candidate as quickly as possible. Not only the designated high-priority block creator may multicast its block candidate to the whole of the group. The second and third validator by priority may start multicasting their block candidates, either immediately or after failing to receive a block candidate from the top priority validator. However, normally only the block candidate with maximal priority will be signed by all (actually, by at least two-thirds of the task group) validators and committed as a new shardchain block. 

#### 2.6.12. Election of the next block candidate. 

Once a block candidate collects at least two-thirds (by stake) of the validity signatures of validators in the task group, it is eligible to be committed as the next shardchain block. A BFT protocol is run to achieve consensus on the block candidate chosen (there may be more than one proposed), with all “good” validators preferring the block candidate with the highest priority for this round. As a result of running this protocol, the block is augmented by signatures of at least two-thirds of the validators (by stake). These signatures testify not only to the validity of the block in question, but also to its being elected by the BFT protocol. After that, the block (without collated data) is combined with these signatures, serialized in a deterministic way, and propagated through the network to all parties concerned.

#### 2.6.15. Generation of new masterchain blocks. 

After all (or almost all) new shardchain blocks have been generated, a new masterchain block may be generated. The procedure is essentially the same as for shardchain blocks (cf. [2.6.12](https://zeroheight.com/86757ecb2/p/822e19/t/8343cb)), with the difference that *all* validators (or at least two-thirds of them) must participate in this process. Because the headers and signatures of new shardchain blocks are propagated to all validators, hashes of the newest blocks in each shardchain can and must be included in the new masterchain block. Once these hashes are committed into the masterchain block, outside observers and other shardchains may consider the new shardchain blocks committed and immutable (cf. [2.1.13](https://zeroheight.com/86757ecb2/p/822e19/t/866ed7)).

### 2.7 Splitting and Merging Shardchains  

#### 2.7.6. Determining the necessity of split operations. 

The split operation for a shardchain is triggered by certain formal conditions (e.g., if for 64 consecutive blocks the shardchain blocks are at least 90% full). These conditions are monitored by the shardchain task group. If they are met, first a “split preparation” flag is included in the header of a new shardchain block (and propagated to the masterchain block referring to this shardchain block). Then, several blocks afterwards, the “split commit” flag is included in the header of the shardchain block (and propagated to the next masterchain block).

#### 2.7.8. Determining the necessity of merge operations. 

The necessity of shard merge operations is also detected by certain formal conditions (e.g., if for 64 consecutive blocks the sum of the sizes of the two blocks of sibling shardchains does not exceed 60% of maximal block size). These formal conditions should also take into account the total gas spent by these blocks and compare it to the current block gas limit, otherwise the blocks may happen to be small because there are some computation-intensive transactions that prevent the inclusion of more transactions. 

These conditions are monitored by validator task groups of both sibling shards (w, s.0) and (w, s.1). Notice that siblings are necessarily neighbors with respect to hypercube routing (cf. [2.4.19](https://zeroheight.com/86757ecb2/p/822e19/t/79694a)), so validators from the task group of any shard will be monitoring the sibling shard to some extent anyways. 

When these conditions are met, either one of the validator subgroups can suggest to the other that they merge by sending a special message. Then they combine into a provisional “merged task group”, with combined membership, capable of running BFT consensus algorithms and of propagating block updates and block candidates if necessary. 

If they reach consensus on the necessity and readiness of merging, “merge prepare” flags are committed into the headers of some blocks of each shardchain, along with the signatures of at least two-thirds of the validators of the sibling’s task group (and are propagated to the next masterchain blocks, so that everybody can get ready for the imminent reconfiguration). However, they continue to create separate shardchain blocks for some predefined number of blocks.

### 2.8 Classification of Blockchain Projects

#### 2.8.4. Variants of Proof-of-Stake. DPOS vs. BFT. 

While Proof-of-Work algorithms are very similar to each other and differ mostly in the hash functions that must be computed for mining new blocks, there are more possibilities for Proof-of-Stake algorithms. They merit a sub-classification of their own. Essentially, one must answer the following questions about a Proof-ofStake algorithm: 

- Who can produce (“mine”) a new block—any full node, or only a member of a (relatively) small subset of validators? (Most PoS systems require new blocks to be generated and signed by one of several designated validators.) 
- Do validators guarantee the validity of the blocks by their signatures, or are all full nodes expected to validate all blocks by themselves? (Scalable PoS systems must rely on validator signatures instead of requiring all nodes to validate all blocks of all blockchains.) 
- Is there a designated producer for the next blockchain block, known in advance, such that nobody else can produce that block instead? 
- Is a newly-created block originally signed by only one validator (its producer), or must it collect a majority of validator signatures from the very beginning?

While there seem to be 2^4 possible classes of PoS algorithms depending on the answers to these questions, the distinction in practice boils down to two major approaches to PoS. In fact, most modern PoS algorithms, designed to be used in scalable multi-chain systems, answer the first two questions in the same fashion: only validators can produce new blocks, and they guarantee block validity without requiring all full nodes to check the validity of all blocks by themselves. 

As to the two last questions, their answers turn out to be highly correlated, leaving essentially only two basic options: 

- *Delegated Proof-of-Stake (DPOS):* There is a universally known designated producer for every block; no one else can produce that block; the new block is originally signed only by its producing validator. 
- *Byzantine Fault Tolerant (BFT) PoS algorithms:* There is a known subset of validators, any of which can suggest a new block; the choice of the actual next block among several suggested candidates, which must be validated and signed by a majority of validators before being released to the other nodes, is achieved by a version of Byzantine Fault Tolerant consensus protocol.

#### 2.8.5. Comparison of DPOS and BFT PoS. 

The BFT approach has the advantage that a newly-produced block has from the very beginning the signatures of a majority of validators testifying to its validity. Another advantage is that, if a majority of validators executes the BFT consensus protocol correctly, no forks can appear at all. On the other hand, BFT algorithms tend to be quite convoluted and require more time for the subset of validators to reach consensus. Therefore, blocks cannot be generated too often. This is why we expect the TON Blockchain (which is a BFT project from the perspective of this classification) to produce a block only once every five seconds. In practice, this interval might be decreased to 2–3 seconds (though we do not promise this), but not further, if validators are spread across the globe. 

The DPOS algorithm has the advantage of being quite simple and straightforward. It can generate new blocks quite often—say, once every two seconds, or maybe even once every second,* because of its reliance on designated block producers known in advance. 

However, DPOS requires all nodes—or at least all validators—to validate all blocks received, because a validator producing and signing a new block confirms not only the relative validity of this block, but also the validity of the previous block it refers to, and all the blocks further back in the chain (maybe up to the beginning of the period of responsibility of the current subset of validators). There is a predetermined order on the current subset of validators, so that for each block there is a designated producer (i.e., validator expected to generate that block); these designated producers are rotated in a round-robin fashion. In this way, a block is at first signed only by its producing validator; then, when the next block is mined, and its producer chooses to refer to this block and not to one of its predecessors (otherwise its block would lie in a shorter chain, which might lose the “longest fork” competition in the future), the signature of the next block is essentially an additional signature on the previous block as well. In this way, a new block gradually collects the signatures of more validators—say, twenty signatures in the time needed to generate the next twenty blocks. A full node will either need to wait for these twenty signatures, or validate the block by itself, starting from a sufficiently confirmed block (say, twenty blocks back), which might be not so easy. 

The obvious disadvantage of the DPOS algorithm is that a new block (and transactions committed into it) achieves the same level of trust (“recursive reliability” as discussed in 2.6.28) only after twenty more blocks are mined, compared to the BFT algorithms, which deliver this level of trust (say, twenty signatures) immediately. Another disadvantage is that DPOS uses the “longest fork wins” approach for switching to other forks; this makes forks quite probable if at least some producers fail to produce subsequent blocks after the one we are interested in (or we fail to observe these blocks because of a network partition or a sophisticated attack). 

We believe that the BFT approach, while more sophisticated to implement and requiring longer time intervals between blocks than DPOS, is better adapted to “tightly-coupled” (cf. 2.8.14) multichain systems, because other blockchains can start acting almost immediately after seeing a committed transaction (e.g., generating a message intended for them) in a new block, without waiting for twenty confirmations of validity (i.e., the next twenty blocks), or waiting for the next six blocks to be sure that no forks appear and verifying the new block by themselves (verifying blocks of other blockchains may become prohibitive in a scalable multi-chain system). Thus they can achieve scalability while preserving high reliability and availability (cf. [2.8.12](https://zeroheight.com/86757ecb2/p/822e19/t/143f6e)). 

On the other hand, DPOS might be a good choice for a “loosely-coupled” multi-chain system, where fast interaction between blockchains is not required – e.g., if each blockchain (“workchain”) represents a separate distributed exchange, and inter-blockchain interaction is limited to rare transfers of tokens from one workchain into another (or, rather, trading one altcoin residing in one workchain for another at a rate approaching 1 : 1). This is what is actually done in the BitShares project, which uses DPOS quite successfully. 

To summarize, while DPOS can generate new blocks and include transactions into them faster (with smaller intervals between blocks), these transactions reach the level of trust required to use them in other blockchains and off-chain applications as “committed” and “immutable” much more slowly than in the BFT systems—say, in thirty seconds** instead of five. Faster transaction inclusion does not mean faster transaction commitment. This could become a huge problem if fast inter-blockchain interaction is required. In that case, one must abandon DPOS and opt for BFT PoS instead.

> *Some people even claim DPOS block generation times of half a second, which does not seem realistic if validators are scattered across several continents. 

> **For instance, EOS, one of the best DPOS projects proposed up to this date, promises a 45-second confirmation and inter-blockchain interaction delay (cf. [[5](https://zeroheight.com/86757ecb2/p/284c8c/t/118757)], “Transaction Confirmation” and “Latency of Interchain Communication” sections).

#### 2.8.8. Blockchain types: homogeneous and heterogeneous systems. 

In a multi-chain system, all blockchains may be essentially of the same type and have the same rules (i.e., use the same format of transactions, the same virtual machine for executing smart-contract code, share the same cryptocurrency, and so on), and this similarity is explicitly exploited, but with different data in each blockchain. In this case, we say that the system is *homogeneous*. Otherwise, different blockchains (which will usually be called workchains inthis case) can have different “rules”. Then we say that the system is *heterogeneous*.

#### 2.8.10. Heterogeneous systems with several workchains having the same rules, or *confederations*. 

In some cases, several blockchains (workchains) with the same rules can be present in a heterogeneous system, but the interaction between them is the same as between blockchains with different rules (i.e., their similarity is not exploited explicitly). Even if they appear to use “the same” cryptocurrency, they in fact use different “altcoins” (independent incarnations of the cryptocurrency). Sometimes one can even have certain mechanisms to convert these altcoins at a rate near to 1 : 1. However, this does not make the system homogeneous in our view; it remains heterogeneous. We say that such a heterogeneous collection of workchains with the same rules is a *confederation*. 

While making a heterogeneous system that allows one to create several workchains with the same rules (i.e., a confederation) may seem a cheap way of building a scalable system, this approach has a lot of drawbacks, too. Essentially, if someone hosts a large project in many workchains with the same rules, she does not obtain a large project, but rather a lot of small instances of this project. This is like having a chat application (or a game) that allows having at most 50 members in any chat (or game) room, but “scales” by creating new rooms to accommodate more users when necessary. As a result, a lot of users can participate in the chats or in the game, but can we say that such a system is truly scalable? 

#### 2.8.12. Sharding support. 

Some blockchain projects (or systems) have native support for sharding, meaning that several (necessarily homogeneous; cf.[ 2.8.8](https://zeroheight.com/86757ecb2/p/822e19/t/100750)) blockchains are thought of as shards of a single (from a high-level perspective) virtual blockchain. For example, one can create 256 shard blockchains (“shardchains”) with the same rules, and keep the state of an account in exactly one shard selected depending on the first byte of its account_id. 

Sharding is a natural approach to scaling blockchain systems, because, if it is properly implemented, users and smart contracts in the system need not be aware of the existence of sharding at all. In fact, one often wants to add sharding to an existing single-chain project (such as Ethereum) when the load becomes too high. 

An alternative approach to scaling would be to use a “confederation” of heterogeneous workchains as described in [2.8.10](https://zeroheight.com/86757ecb2/p/822e19/t/29c3e2), allowing each user to keep her account in one or several workchains of her choice, and transfer funds from her account in one workchain to another workchain when necessary, essentially performing a 1 : 1 altcoin exchange operation. The drawbacks of this approach have already been discussed in [2.8.10](https://zeroheight.com/86757ecb2/p/822e19/t/29c3e2). 

However, sharding is not so easy to implement in a fast and reliable fashion, because it implies a lot of messages between different shardchains. For example, if accounts are evenly distributed between N shards, and the only transactions are simple fund transfers from one account to another, then only a small fraction (1/N) of all transactions will be performed within a single blockchain; almost all (1 − 1/N) transactions will involve two blockchains, requiring inter-blockchain communication. If we want these transactions to be fast, we need a fast system for transferring messages between shardchains. In other words, the blockchain project needs to be “tightly-coupled” in the sense described in [2.8.14](https://zeroheight.com/86757ecb2/p/822e19/t/23633d). 

#### 2.8.13. Dynamic and static sharding. 

Sharding might be dynamic (if additional shards are automatically created when necessary) or static (when there is a predefined number of shards, which is changeable only through a hard fork at best). Most sharding proposals are static; the TON Blockchain uses dynamic sharding (cf. [2.7](https://zeroheight.com/86757ecb2/p/822e19/t/50ec0c)).

#### 2.8.14. Interaction between blockchains: loosely-coupled and tightly-coupled systems. 

Multi-blockchain projects can be classified according to the supported level of interaction between the constituent blockchains. 

The least level of support is the absence of any interaction between different blockchains whatsoever. We do not consider this case here, because we would rather say that these blockchains are not parts of one blockchain system, but just separate instances of the same blockchain protocol. 

The next level of support is the absence of any specific support for messaging between blockchains, making interaction possible in principle, but awkward. We call such systems “loosely-coupled”; in them one must send messages and transfer value between blockchains as if they had been blockchains belonging to completely separate blockchain projects (e.g., Bitcoin and Ethereum; imagine two parties want to exchange some Bitcoins, kept in the Bitcoin blockchain, into Ethers, kept in the Ethereum blockchain). In other words, one must include the outbound message (or its generating transaction) in a block of the source blockchain. Then she (or some other party) must wait for enough confirmations (e.g., a given number of subsequent blocks) to consider the originating transaction to be “committed” and “immutable”, so as to be able to perform external actions based on its existence. Only then may a transaction relaying the message into the target blockchain (perhaps along with a reference and a Merkle proof of existence for the originating transaction) be committed. 

If one does not wait long enough before transferring the message, or if a fork happens anyway for some other reason, the joined state of the two blockchains turns out to be inconsistent: a message is delivered into the second blockchain that has never been generated in (the ultimately chosen fork of) the first blockchain. 

Sometimes partial support for messaging is added, by standardizing the format of messages and the location of input and output message queues in the blocks of all workchains (this is especially useful in heterogeneous systems). While this facilitates messaging to a certain extent, it is conceptually not too different from the previous case, so such systems are still “loosely-coupled”.

By contrast, “tightly-coupled” systems include special mechanisms to provide fast messaging between all blockchains. The desired behavior is to be able to deliver a message into another workchain immediately after it has been generated in a block of the originating blockchain. On the other hand, “tightly-coupled” systems are also expected to maintain overall consistency in the case of forks. While these two requirements appear to be contradictory at first glance, we believe that the mechanisms used by the TON Blockchain (the inclusion of shardchain block hashes into masterchain blocks; the use of “vertical” blockchains for fixing invalid blocks, cf. [2.1.17](https://zeroheight.com/86757ecb2/p/822e19/t/83c8ba); hypercube routing, cf. [2.4.19](https://zeroheight.com/86757ecb2/p/822e19/t/79694a); Instant Hypercube Routing, cf. [2.4.20](https://zeroheight.com/86757ecb2/p/822e19/t/730dc0)) enable it to be a “tightly-coupled” system, perhaps the only one so far. 

Of course, building a “loosely-coupled” system is much simpler; however, fast and efficient sharding (cf. [2.8.12](https://zeroheight.com/86757ecb2/p/822e19/t/143f6e)) requires the system to be “tightlycoupled”. 

#### 2.8.16. Complications of changing the “genome” of a blockchain project. 

The above classification defines the “genome” of a blockchain project. This genome is quite “rigid”: it is almost impossible to change it once the project is deployed and is used by a lot of people. One would need a series of hard forks (which would require the approval of the majority of the community), and even then the changes would need to be very conservative in order to preserve backward compatibility (e.g., changing the semantics of the virtual machine might break existing smart contracts). An alternative would be to create new “sidechains” with their different rules, and bind them somehow to the blockchain (or the blockchains) of the original project. One might use the blockchain of the existing single-blockchain project as an external masterchain for an essentially new and separate project.* Our conclusion is that the genome of a project is very hard to change once it has been deployed. Even starting with PoW and planning to replace it with PoS in the future is quite complicated.** Adding shards to a project originally designed without support for them seems almost impossible.***In fact, adding support for smart contracts into a project (namely, Bitcoin) originally designed without support for such features has been deemed impossible (or at least undesirable by the majority of the Bitcoin community) and eventually led to the creation of a new blockchain project, Ethereum.

> *For example, the Plasma project plans to use the Ethereum blockchain as its (external) masterchain; it does not interact much with Ethereum otherwise, and it could have been suggested and implemented by a team unrelated to the Ethereum project. 

> **As of 2017, Ethereum is still struggling to transition from PoW to a combined PoW+PoS system; we hope it will become a truly PoS system someday. 

> ***There are sharding proposals for Ethereum dating back to 2015; it is unclear how they might be implemented and deployed without disrupting Ethereum or creating an essentially independent parallel project. 

### 2.9 Comparsion with other blockchain projects

#### 2.9.7. EOS [5];[https://eos.io.](https://eos.io./)

EOS (2018 or later) is a proposed heterogeneous multi-blockchain DPoS system with smart contract support and with some minimal support for messaging (still loosely-coupled in the sense described in 2.8.14). It is an attempt by the same team that has previously successfully created the BitShares and SteemIt projects, demonstrating the strong points of the DPoS consensus algorithm. Scalability will be achieved by creating specialized workchains for projects that need it (e.g., a distributed exchange might use a workchain supporting a special set of optimized transactions, similarly to what BitShares did) and by creating multiple workchains with the same rules (confederations in the sense described in 2.8.10). The drawbacks and limitations of this approach to scalability have been discussed in loc. cit. Cf. also 2.8.5, 2.8.12, and 2.8.14 for a more detailed discussion of DPoS, sharding, interaction between workchains and their implications for the scalability of a blockchain system.

At the same time, even if one will not be able to “create a Facebook inside a blockchain” (cf. 2.9.13), EOS or otherwise, we think that EOS might become a convenient platform for some highly-specialized weakly interacting distributed applications, similar to BitShares (decentralized exchange) and SteemIt (decentralized blog platform).

#### 2.9.8. PolkaDot [17]; [https://polkadot.io/.](https://polkadot.io/) 

PolkaDot (2019 or later) is one of the best thought-out and most detailed proposed multichain Proofof-Stake projects; its development is led by one of the Ethereum co-founders. This project is one of the closest projects to the TON Blockchain on our map. (In fact, we are indebted for our terminology for “fishermen” and “nominators” to the PolkaDot project.)

PolkaDot is a heterogeneous loosely-coupled multichain Proof-of-Stake project, with Byzantine Fault Tolerant (BFT) consensus for generation of new blocks and a masterchain (which might be external—e.g., the Ethereum blockchain). It also uses hypercube routing, somewhat like (the slow version of) TON’s as described in [2.4.19](https://zeroheight.com/86757ecb2/p/822e19/t/79694a).

Its unique feature is its ability to create not only public, but also private blockchains. These private blockchains would also be able to interact with other public blockchains, PolkaDot or otherwise. 

As such, PolkaDot might become a platform for large-scale private blockchains, which might be used, for example, by bank consortiums to quickly transfer funds to each other, or for any other uses a large corporation might have for private blockchain technology. 

However, PolkaDot has no sharding support and is not tightly-coupled. This somewhat hampers its scalability, which is similar to that of EOS. (Perhaps a bit better, because PolkaDot uses BFT PoS instead of DPoS.)

#### 2.9.13. Is it possible to “upload Facebook into a blockchain”? 

Sometimes people claim that it will be possible to implement a social network on the scale of Facebook as a distributed application residing in a blockchain. Usually a favorite blockchain project is cited as a possible “host” for such an application. We cannot say that this is a technical impossibility. Of course, one needs a tightly-coupled blockchain project with true sharding (i.e., TON) in order for such a large application not to work too slowly (e.g., deliver messages and updates from users residing in one shardchain to their friends residing in another shardchain with reasonable delays). However, we think that this is not needed and will never be done, because the price would be prohibitive. 

Let us consider “uploading Facebook into a blockchain” as a thought experiment; any other project of similar scale might serve as an example as well. Once Facebook is uploaded into a blockchain, all operations currently done by Facebook’s servers will be serialized as transactions in certain blockchains (e.g., TON’s shardchains), and will be performed by all validators of these blockchains. Each operation will have to be performed, say, at least twenty times, if we expect every block to collect at least twenty validator signatures (immediately or eventually, as in DPOS systems). Similarly, all data kept by Facebook’s servers on their disks will be kept on the disks of all validators for the corresponding shardchain (i.e., in at least twenty copies). 

Because the validators are essentially the same servers (or perhaps clusters of servers, but this does not affect the validity of this argument) as those currently used by Facebook, we see that the total hardware expenses associated with running Facebook in a blockchain are at least twenty times higher than if it were implemented in the conventional way. 

In fact, the expenses would be much higher still, because the blockchain’s virtual machine is slower than the “bare CPU” running optimized compiled code, and its storage is not optimized for Facebook-specific problems. One might partially mitigate this problem by crafting a specific workchain with some special transactions adapted for Facebook; this is the approach of BitShares and EOS to achieving high performance, available in the TON Blockchain as well. However, the general blockchain design would still impose some additional restrictions by itself, such as the necessity to register all operations as transactions in a block, to organize these transactions in a Merkle tree, to compute and check their Merkle hashes, to propagate this block further, and so on. 

Therefore, a conservative estimate is that one would need 100 times more servers of the same performance as those used by Facebook now in order to validate a blockchain project hosting a social network of that scale. Somebody will have to pay for these servers, either the company owning the distributed application (imagine seeing 700 ads on each Facebook page instead of 7) or its users. Either way, this does not seem economically viable. 

We believe that it is not true that everything should be uploaded into the blockchain. For example, it is not necessary to keep user photographs in the blockchain; registering the hashes of these photographs in the blockchain and keeping the photographs in a distributed off-chain storage (such as FileCoin or TON Storage) would be a better idea. This is the reason why TON is not just a blockchain project, but a collection of several components (TON P2P Network, TON Storage, TON Services) centered around the TON Blockchain as outlined in Chapters [1](https://zeroheight.com/86757ecb2/p/33136a) and [4](https://zeroheight.com/86757ecb2/p/88e36d).

## 3. TON Networking

Any blockchain project requires not only a specification of block format and blockchain validation rules, but also a network protocol used to propagate new blocks, send and collect transaction candidates and so on. In other words, a specialized peer-to-peer network must be set up by every blockchain project. This network must be peer-to-peer, because blockchain projects are normally expected to be decentralized, so one cannot rely on a centralized group of servers and use conventional client-server architecture, as, for instance, classical online banking applications do. Even light clients (e.g., light cryptocurrency wallet smartphone applications), which must connect to full nodes in a client-server–like fashion, are actually free to connect to another full node if their previous peer goes down, provided the protocol used to connect to full nodes is standardized enough. While the networking demands of single-blockchain projects, such as Bitcoin or Ethereum, can be met quite easily (one essentially needs to construct a “random” peer-to-peer overlay network, and propagate all new blocks and transaction candidates by a gossip protocol), multi-blockchain projects, such as the TON Blockchain, are much more demanding (e.g., one must be able to subscribe to updates of only some shardchains, not necessarily all of them). Therefore, the networking part of the TON Blockchain and the TON Project as a whole merits at least a brief discussion. On the other hand, once the more sophisticated network protocols needed to support the TON Blockchain are in place, it turns out that they can easily be used for purposes not necessarily related to the immediate demands of the TON Blockchain, thus providing more possibilities and flexibility for creating new services in the TON ecosystem. 

### 3.1 Abstract Datagram Network Layer 

The cornerstone in building the TON networking protocols is the (TON) Abstract (Datagram) Network Layer. It enables all nodes to assume certain “network identities”, represented by 256-bit “abstract network addresses”, and communicate (send datagrams to each other, as a first step) using only these 256-bit network addresses to identify the sender and the receiver. In particular, one does not need to worry about IPv4 or IPv6 addresses, UDP port numbers, and the like; they are hidden by the Abstract Network Layer. 

#### 3.1.1. Abstract network addresses. 

An abstract network address, or an abstract address, or just address for short, is a 256-bit integer, essentially equal to a 256-bit ECC public key. This public key can be generated arbitrarily, thus creating as many different network identities as the node likes. However, one must know the corresponding private key in order to receive (and decrypt) messages intended for such an address. In fact, the address is not the public key itself; instead, it is a 256-bit hash (Hash = sha256) of a serialized TL-object (cf. [2.2.5](https://zeroheight.com/86757ecb2/p/822e19/t/174cd2)) that can describe several types of public keys and addresses depending on its constructor (first four bytes). In the simplest case, this serialized TL-object consists just of a 4-byte magic number and a 256-bit elliptic curve cryptography (ECC) public key; in this case, the address will equal the hash of this 36-byte structure. One might use, however, 2048-bit RSA keys, or any other scheme of publickey cryptography instead. When a node learns another node’s abstract address, it must also receive its “preimage” (i.e., the serialized TL-object, the hash of which equals that abstract address) or else it will not be able to encrypt and send datagrams to that address. 

#### 3.1.2. Lower-level networks. 

UDP implementation. From the perspective of almost all TON Networking components, the only thing that exists is a network (the Abstract Datagram Networking Layer) able to (unreliably) send datagrams from one abstract address to another. In principle, the Abstract Datagram Networking Layer (ADNL) can be implemented over different existing network technologies. However, we are going to implement it over UDP in IPv4/IPv6 networks (such as the Internet or intranets), with an optional TCP fallback if UDP is not available. 

#### 3.1.3. Simplest case of ADNL over UDP. 

The simplest case of sending a datagram from a sender’s abstract address to any other abstract address (with known preimage) can be implemented as follows. Suppose that the sender somehow knows the IP-address and the UDP port of the receiver who owns the destination abstract address, and that both the receiver and the sender use abstract addresses derived from 256-bit ECC public keys. In this case, the sender simply augments the datagram to be sent by its ECC signature (done with its private key) and its source address (or the preimage of the source address, if the receiver is not known to know that preimage yet). The result is encrypted with the recipient’s public key, embedded into a UDP datagram and sent to the known IP and port of the recipient. Because the first 256 bits of the UDP datagram contain the recipient’s abstract address, the recipient can identify which private key should be used to decrypt the remainder of the datagram. Only after that is the sender’s identity revealed. 

#### 3.1.4. Less secure way, with the sender’s address in plaintext.

Sometimes a less secure scheme is sufficient, when the recipient’s and the sender’s addresses are kept in plaintext in the UDP datagram; the sender’s private key and the recipient’s public key are combined together using ECDH (Elliptic Curve Diffie–Hellman) to generate a 256-bit shared secret, which is used afterwards, along with a random 256-bit nonce also included in the unencrypted part, to derive AES keys used for encryption. The integrity may be provided, for instance, by concatenating the hash of the original plaintext data to the plaintext before encryption. This approach has the advantage that, if more than one datagram is expected to be exchanged between the two addresses, the shared secret can be computed only once and then cached; then slower elliptic curve operations will no longer be required for encrypting or decrypting the next datagrams. 

#### 3.1.5. Channels and channel identifiers.

In the simplest case, the first 256 bits of a UDP datagram carrying an embedded TON ADNL datagram will be equal to the recipient’s address. However, in general they constitute a channel identifier. There are different types of channels. Some of them are point-to-point; they are created by two parties who wish to exchange a lot of data in the future and generate a shared secret by exchanging several packets encrypted as described in [3.1.3](https://zeroheight.com/86757ecb2/p/48c113/t/0526e0) or [3.1.4](https://zeroheight.com/86757ecb2/p/48c113/t/088b9f), by running classical or elliptic curve Diffie–Hellman (if extra security is required), or simply by one party generating a random shared secret and sending it to the other party. After that, a channel identifier is derived from the shared secret combined with some additional data (such as the sender’s and recipient’s addresses), for instance by hashing, and that identifier is used as the first 256 bits of UDP datagrams carrying data encrypted with the aid of that shared secret. 

#### 3.1.6. Channel as a tunnel identifier. 

In general, a “channel”, or “channel identifier” simply selects a way of processing an inbound UDP datagram, known to the receiver. If the channel is the receiver’s abstract address, the processing is done as outlined in [3.1.3](https://zeroheight.com/86757ecb2/p/48c113/t/0526e0) or [3.1.4](https://zeroheight.com/86757ecb2/p/48c113/t/088b9f); if the channel is an estabished point-to-point channel discussed in [3.1.5](https://zeroheight.com/86757ecb2/p/48c113/t/81e284), the processing consists in decrypting the datagram with the aid of the shared secret as explained in loc. cit., and so on. 

In particular, a channel identifier can actually select a “tunnel”, when the immediate recipient simply forwards the received message to somebody else—the actual recipient or another proxy. Some encryption or decryption steps (reminiscent of “onion routing”  [[6](https://zeroheight.com/86757ecb2/p/724d6e/t/2389eb)] or even “garlic routing” *) might be done along the way, and another channel identifier might be used for reencrypted forwarded packets (for example, a peer-to-peer channel could be employed to forward the packet to the next recipient on the path). 

In this way, some support for “tunneling” and “proxying”—somewhat similar to that provided by the TOR or I2P projects—can be added on the level of the TON Abstract Datagram Network Layer, without affecting the functionality of all higher-level TON network protocols, which would be agnostic of such an addition. This opportunity is exploited by the TON Proxy service (cf. [4.1.11](https://zeroheight.com/86757ecb2/p/88e36d/t/437958)).

> *<https://geti2p.net/en/docs/how/garlic-routing>

#### 3.1.7. Zero channel and the bootstrap problem. 

Normally, a TON ADNL node will have some “neighbor table”, containing information about other known nodes, such as their abstract addresses and their preimages (i.e., public keys) and their IP addresses and UDP ports. Then it will gradually extend this table by using information learned from these known nodes as answers to special queries, and sometimes prune obsolete records. 

However, when a TON ADNL node just starts up, it may happen that it does not know any other node, and can learn only the IP address and UDP port of a node, but not its abstract address. This happens, for example, if a light client is not able to access any of the previously cached nodes and any nodes hardcoded into the software, and must ask the user to enter an IP address or a DNS domain of a node, to be resolved through DNS. 

In this case, the node will send packets to a special “zero channel” of the node in question. This does not require knowledge of the recipient’s public key (but the message should still contain the sender’s identity and signature), so the message is transferred without encryption. It should be normally used only to obtain an identity (maybe a one-time identity created especially for this purpose) of the receiver, and then to start communicating in a safer way. Once at least one node is known, it is easy to populate the “neighbor table” and “routing table” by more entries, learning them from answers to special queries sent to the already known nodes. 

Not all nodes are required to process datagrams sent to the zero channel, but those used to bootstrap light clients should support this feature. 

#### 3.1.8. TCP-like stream protocol over ADNL. 

The ADNL, being an unreliable (small-size) datagram protocol based on 256-bit abstract addresses, can be used as a base for more sophisticated network protocols. One can build, for example, a TCP-like stream protocol, using ADNL as an abstract replacement for IP. However, most components of the TON Project do not need such a stream protocol. 

#### 3.1.9. RLDP, or Reliable Large Datagram Protocol over ADNL. 

A reliable arbitrary-size datagram protocol built upon the ADNL, called RLDP, is used instead of a TCP-like protocol. This reliable datagram protocol can be employed, for instance, to send RPC queries to remote hosts and receive answers from them (cf. [4.1.5](https://zeroheight.com/86757ecb2/p/88e36d/t/2288e2)).

### 3.2 TON DHT: Kademlia-like Distributed Hash Table 

The TON Distributed Hash Table (DHT) plays a crucial role in the networking part of the TON Project, being used to locate other nodes in the network. For example, a client wanting to commit a transaction into a shardchain might want to find a validator or a collator of that shardchain, or at least some node that might relay the client’s transaction to a collator. This can be done by looking up a special key in the TON DHT. Another important application of the TON DHT is that it can be used to quickly populate a new node’s neighbor table (cf. [3.1.7](https://zeroheight.com/86757ecb2/p/48c113/t/678893)), simply by looking up a random key, or the new node’s address. If a node uses proxying and tunneling for its inbound datagrams, it publishes the tunnel identifier and its entry point (e.g., IP address and UDP port) in the TON DHT; then all nodes wishing to send datagrams to that node will obtain this contact information from the DHT first. 

The TON DHT is a member of the family of Kademlia-like distributed hash tables [[10](https://zeroheight.com/86757ecb2/p/724d6e/t/57e4bb)].

#### 3.2.10. Distributed “torrent trackers” and “network interest groups” in TON DHT. 

Yet another interesting case is when the value contains a list of nodes—perhaps with their IP addresses and ports, or just with their abstract addresses—and the “update rule” consists in including the requester in this list, provided she can confirm her identity. This mechanism might be used to create a distributed “torrent tracker”, where all nodes interested in a certain “torrent” (i.e., a certain file) can find other nodes that are interested in the same torrent, or already have a copy. TON Storage (cf. [4.1.8](https://zeroheight.com/86757ecb2/p/88e36d/t/57af26)) uses this technology to find the nodes that have a copy of a required file (e.g., a snapshot of the state of a shardchain, or an old block). However, its more important use is to create “overlay multicast subnetworks” and “network interest groups” (cf. [3.3](https://zeroheight.com/86757ecb2/p/48c113/t/529b3a)). The idea is that only some nodes are interested in the updates of a specific shardchain. If the number of shardchains becomes very large, finding even one node interested in the same shard may become complicated. This “distributed torrent tracker” provides a convenient way to find some of these nodes. Another option would be to request them from a validator, but this would not be a scalable approach, and validators might choose not to respond to such queries coming from arbitrary unknown nodes.

#### 3.2.12. Locating services. 

Some services, located in the TON Network and available through the (higher-level protocols built upon the) TON ADNL described in [3.1](https://zeroheight.com/86757ecb2/p/48c113/t/70e58c), may want to publish their abstract addresses somewhere, so that their clients would know where to find them. However, publishing the service’s abstract address in the TON Blockchain may not be the best approach, because the abstract address might need to be changed quite often, and because it could make sense to provide several addresses, for reliability or load balancing purposes. An alternative is to publish a public key into the TON Blockchain, and use a special DHT key indicating that public key as its “owner” in the TL description string (cf. [2.2.5](https://zeroheight.com/86757ecb2/p/822e19/t/174cd2)) to publish an up-to-date list of the service’s abstract addresses. This is one of the approaches exploited by TON Services.

#### 3.2.14. Locating abstract addresses. 

Notice that the TON DHT, while being implemented over TON ADNL, is itself used by the TON ADNL for several purposes. 

The most important of them is to locate a node or its contact data starting from its 256-bit abstract address. This is necessary because the TON ADNL should be able to send datagrams to arbitrary 256-bit abstract addresses, even if no additional information is provided. 

To this end, the 256-bit abstract address is simply looked up as a key in the DHT. Either a node with this address (i.e., using this address as a public semi-persistent DHT address) is found, in which case its IP address and port can be learned; or, an input tunnel description may be retrieved as the value of the key in question, signed by the correct private key, in which case this tunnel description would be used to send ADNL datagrams to the intended recipient. 

Notice that in order to make an abstract address “public” (reachable from any nodes in the network), its owner must either use it as a semi-permanent DHT address, or publish (in the DHT key equal to the abstract address under consideration) an input tunnel description with another of its public abstract addresses (e.g., the semi-permanent address) as the tunnel’s entry point. Another option would be to simply publish its IP address and UDP port.

### 3.3 Overlay Networks and Multicasting 

Messages In a multi-blockchain system like the TON Blockchain, even full nodes would normally be interested in obtaining updates (i.e., new blocks) only about some shardchains. To this end, a special overlay (sub)network must be built inside the TON Network, on top of the ADNL protocol discussed in [3.1](https://zeroheight.com/86757ecb2/p/48c113/t/70e58c), one for each shardchain. Therefore, the need to build arbitrary overlay subnetworks, open to any nodes willing to participate, arises. Special gossip protocols, built upon ADNL, will be run in these overlay networks. In particular, these gossip protocols may be used to propagate (broadcast) arbitrary data inside such a subnetwork.

#### 3.3.10. TON overlay networks are optimized for lower latency.

TON overlay networks optimize the “random” network graph generated by the previous method as follows. Every node tries to retain at least three neighbors with the minimal round-trip time, changing this list of “fast neighbors” very rarely. At the same time, it also has at least three other “slow neighbors” that are chosen completely randomly, so that the overlay network graph would always contain a random subgraph. This is required to maintain connectivity and prevent splitting of the overlay network into several unconnected regional subnetworks. At least three “intermediate neighbors”, which have intermediate round-trip times, bounded by a certain constant (actually, a function of the round-trip times of the fast and the slow neighbors), are also chosen and retained. 

In this way, the graph of an overlay network still maintains enough randomness to be connected, but is optimized for lower latency and higher throughput.

#### 3.3.15. Streaming broadcast protocol. 

Finally, there is a streaming broadcast protocol for TON overlay networks, used, for example, to propagate block candidates among validators of some shardchain (“shardchain task group”), who, of course, create a private overlay network for that purpose. The same protocol can be used to propagate new shardchain blocks to all full nodes for that shardchain. 

This protocol has already been outlined in [2.6.10](https://zeroheight.com/86757ecb2/p/822e19/t/054d1f): the new (large) broadcast message is split into, say, *N* one-kilobyte chunks; the sequence of these chunks is augmented to *M ≥ N* chunks by means of an erasure code such as the Reed–Solomon or a fountain code (e.g., the RaptorQ code [[9](https://zeroheight.com/86757ecb2/p/284c8c/t/239f9b)] [[14](https://zeroheight.com/86757ecb2/p/284c8c/t/55de6e)]), and these *M* chunks are streamed to all neighbors in ascending chunk number order. The participating nodes collect these chunks until they can recover the original large message (one would have to successfully receive at least *N* of the chunks for this), and then instruct their neighbors to stop sending new chunks of the stream, because now these nodes can generate the subsequent chunks on their own, having a copy of the original message. Such nodes continue to generate the subsequent chunks of the stream and send them to their neighbors, unless the neighbors in turn indicate that this is no longer necessary. 

In this way, a node does not need to download a large message in its entirety before propagating it further. This minimizes broadcast latency, especially when combined with the optimizations described in [3.3.10](https://zeroheight.com/86757ecb2/p/48c113/t/487221).

## 4. TON Services and Applications

We have discussed the TON Blockchain and TON Networking technologies at some length. Now we explain some ways in which they can be combined to create a wide range of services and applications, and discuss some of the services that will be provided by the TON Project itself, either from the very beginning or at a later time.

### 4.1 TON Service Implementation Strategies 

We start with a discussion of how different blockchain and network-related applications and services may be implemented inside the TON ecosystem. First of all, a simple classification is in order: 

#### 4.1.1. Applications and services. 

We will use the words “application” and “service” interchangeably. However, there is a subtle and somewhat vague distinction: an application usually provides some services directly to human users, while a service is usually exploited by other applications and services. For example, TON Storage is a service, because it is designed to keep files on behalf of other applications and services, even though a human user might use it directly as well. A hypothetical “Facebook in a blockchain” (cf. [2.9.13](https://zeroheight.com/86757ecb2/p/822e19/t/54e298)) or Telegram messenger, if made available through the TON Network (i.e., implemented as a “ton-service”; cf. [4.1.6](https://zeroheight.com/86757ecb2/p/88e36d/t/806f71)), would rather be an application, even though some “bots” might access it automatically without human intervention. 

### 4.1.2. Location of the application: on-chain, off-chain or mixed. 

A service or an application designed for the TON ecosystem needs to keep its data and process that data somewhere. This leads to the following classification of applications (and services): 

- On-chain applications (cf. [4.1.4](https://zeroheight.com/86757ecb2/p/88e36d/t/4479ca)): All data and processing are in the TON Blockchain. 
- Off-chain applications (cf. [4.1.5](https://zeroheight.com/86757ecb2/p/88e36d/t/2288e2)): All data and processing are outside the TON Blockchain, on servers available through the TON Network. 
- Mixed applications (cf. [4.1.7](https://zeroheight.com/86757ecb2/p/88e36d/t/2089f0)): Some, but not all, data and processing are in the TON Blockchain; the rest are on off-chain servers available through the TON Network. 

### 4.1.3. Centralization: centralized and decentralized, or distributed, applications. 

Another classification criterion is whether the application (or service) relies on a centralized server cluster, or is really “distributed” (cf. 4.1.9). 

All on-chain applications are automatically decentralized and distributed. Off-chain and mixed applications may exhibit different degrees of centralization. Now let us consider the above possibilities in more detail. 

### 4.1.4. Pure “on-chain” applications: distributed applications, or “dapps”, residing in the blockchain. 

One of the possible approaches, mentioned in 4.1.2, is to deploy a “distributed application” (commonly abbreviated as “dapp”) completely in the TON Blockchain, as one smart contract or a collection of smart contracts. All data will be kept as part of the permanent state of these smart contracts, and all interaction with the project will be done by means of (TON Blockchain) messages sent to or received from these smart contracts. 

We have already discussed in 2.9.13 that this approach has its drawbacks and limitations. It has its advantages, too: such a distributed application needs no servers to run on or to store its data (it runs “in the blockchain”— i.e., on the validators’ hardware), and enjoys the blockchain’s extremely high (Byzantine) reliability and accessibility. 

The developer of such a distributed application does not need to buy or rent any hardware; all she needs to do is develop some software (i.e., the code for the smart contracts). After that, she will effectively rent the computing power from the validators, and will pay for it in Grams, either herself or by putting this burden on the shoulders of her users. 

### 4.1.5. Pure network services: “ton-sites” and “ton-services”. 

Another extreme option is to deploy the service on some servers and make it available to the users through the ADNL protocol described in 3.1, and maybe some higher level protocol such as the RLDP discussed in 3.1.9, which can be used to send RPC queries to the service in any custom format and obtain answers to these queries. 

In this way, the service will be totally off-chain, and will reside in the TON Network, almost without using the TON Blockchain. The TON Blockchain might be used only to locate the abstract address or addresses of the service, as outlined in 3.2.12, perhaps with the aid of a service such as the TON DNS (cf. 4.3.1) to facilitate translation of domainlike human-readable strings into abstract addresses.

To the extent the ADNL network (i.e., the TON Network) is similar to the Invisible Internet Project (I 2P), such (almost) purely network services are analogous to the so-called “eep-services” (i.e., services that have an I 2Paddress as their entry point, and are available to clients through the I 2P network). We will say that such purely network services residing in the TON Network are “ton-services”. 

An “eep-service” may implement HTTP as its client-server protocol; in the TON Network context, a “ton-service” might simply use RLDP (cf. 3.1.9) datagrams to transfer HTTP queries and responses to them. If it uses the TON DNS to allow its abstract address to be looked up by a human-readable domain name, the analogy to a web site becomes almost perfect. One might even write a specialized browser, or a special proxy (“ton-proxy”) that is run locally on a user’s machine, accepts arbitrary HTTP queries from an ordinary web browser the user employs (once the local IP address and the TCP port of the proxy are entered into the browser’s configuration), and forwards these queries through the TON Network to the abstract address of the service. Then the user would have a browsing experience similar to that of the World Wide Web (WWW). 

In the I 2P ecosystem, such “eep-services” are called “eep-sites”. One can easily create “ton-sites” in the TON ecosystem as well. This is facilitated somewhat by the existence of services such as the TON DNS, which exploit the TON Blockchain and the TON DHT to translate (TON) domain names into abstract addresses. 

### 4.1.6. Telegram Messenger as a ton-service; MTProto over RLDP. 

We would like to mention in passing that the MTProto protocol,* used by Telegram Messenger** for client-server interaction, can be easily embedded into the RLDP protocol discussed in 3.1.9, thus effectively transforming Telegram into a ton-service. Because the TON Proxy technology can be switched on transparently for the end user of a ton-site or a ton-service, being implemented on a lower level than the RLDP and ADNL protocols (cf. 3.1.6), this would make Telegram effectively unblockable. Of course, other messaging and social networking services might benefit from this technology as well.

> *<https://core.telegram.org/mtproto>

> **<https://telegram.org/>

### 4.1.7. Mixed services: partly off-chain, partly on-chain. 

Some services might use a mixed approach: do most of the processing off-chain, but also have some on-chain part (for example, to register their obligations towards their users, and vice versa). In this way, part of the state would still be kept in the TON Blockchain (i.e., an immutable public ledger), and any misbehavior of the service or of its users could be punished by smart contracts.

### 4.1.8. Example: keeping files off-chain; TON Storage. 

An example of such a service is given by TON Storage. In its simplest form, it allows users to store files off-chain, by keeping on-chain only a hash of the file to be stored, and possibly a smart contract where some other parties agree to keep the file in question for a given period of time for a pre-negotiated fee. In fact, the file may be subdivided into chunks of some small size (e.g., 1 kilobyte), augmented by an erasure code such as a Reed–Solomon or a fountain code, a Merkle tree hash may be constructed for the augmented sequence of chunks, and this Merkle tree hash might be published in the smart contract instead of or along with the usual hash of the file. 

This is somewhat reminiscent of the way files are stored in a torrent. An even simpler form of storing files is completely off-chain: one might essentially create a “torrent” for a new file, and use TON DHT as a “distributed torrent tracker” for this torrent (cf. 3.2.10). This might actually work pretty well for popular files. However, one does not get any availability guarantees. For example, a hypothetical “blockchain Facebook” (cf. 2.9.13), which would opt to keep the profile photographs of its users completely off-chain in such “torrents”, might risk losing photographs of ordinary (not especially popular) users, or at least risk being unable to present these photographs for prolonged periods. The TON Storage technology, which is mostly off-chain, but uses an on-chain smart contract to enforce availability of the stored files, might be a better match for this task. 

### 4.1.9. Decentralized mixed services, or “fog services”. 

So far, we have discussed centralized mixed services and applications. While their on-chain component is processed in a decentralized and distributed fashion, being located in the blockchain, their off-chain component relies on some servers controlled by the service provider in the usual centralized fashion. Instead of using some dedicated servers, computing power might be rented from a cloud computing service offered by one of the large companies. However, this would not lead to decentralization of the off-chain component of the service. 

A decentralized approach to implementing the off-chain component of a service consists in creating a market, where anybody possessing the required hardware and willing to rent their computing power or disk space would offer their services to those needing them. For example, there might exist a registry (which might also be called a “market” or an “exchange”) where all nodes interested in keeping files of other users publish their contact information, along with their available storage capacity, availability policy, and prices. 

Those needing these services might look them up there, and, if the other party agrees, create smart contracts in the blockchain and upload files for off-chain storage. In this way a service like TON Storage becomes truly decentralized, because it does not need to rely on any centralized cluster of servers for storing files. 

### 4.1.10. Example: “fog computing” platforms as decentralized mixed services. 

Another example of such a decentralized mixed application arises when one wants to perform some specific computations (e.g., 3D rendering or training neural networks), often requiring specific and expensive hardware. Then those having such equipment might offer their services through a similar “exchange”, and those needing such services would rent them, with the obligations of the sides registered by means of smart contracts. This is similar to what “fog computing” platforms, such as Golem (<https://golem.network/)>or SONM (<https://sonm.io/),> promise to deliver.

### 4.1.11. Example: TON Proxy is a fog service. 

TON Proxy provides yet another example of a fog service, where nodes wishing to offer their services (with or without compensation) as tunnels for ADNL network traffic might register, and those needing them might choose one of these nodes depending on the price, latency and bandwidth offered. Afterwards, one might use payment channels provided by TON Payments for processing micropayments for the services of those proxies, with payments collected, for instance, for every 128 KiB transferred. 

### 4.1.12. Example: TON Payments is a fog service.

 The TON Payments platform (cf. 5) is also an example of such a decentralized mixed application

## 4.3 Accessing TON Services 

We have discussed in 4.1 the different approaches one might employ for creating new services and applications residing in the TON ecosystem. Now we discuss how these services might be accessed, and some of the “helper services” that will be provided by TON, including TON DNS and TON Storage. 

### 4.3.1. TON DNS: a mostly on-chain hierarchical domain name service. 

The TON DNS is a predefined service, which uses a collection of smart contracts to keep a map from human-readable domain names to (256-bit) addresses of ADNL network nodes and TON Blockchain accounts and smart contracts. 

While anybody might in principle implement such a service using the TON Blockchain, it is useful to have such a predefined service with a wellknown interface, to be used by default whenever an application or a service wants to translate human-readable identifiers into addresses.

### 4.3.6. Retrieving data from a DNS smart contract. 

In principle, any full node for the masterchain or shardchain containing a DNS smart contract might be able to look up any subdomain in the database of that smart contract, if the structure and the location of the hashmap inside the persistent storage of the smart contract are known. However, this approach would work only for certain DNS smart contracts. It would fail miserably if a non-standard DNS smart contract were used. Instead, an approach based on general smart contract interfaces and get methods (cf. 4.3.11) is used. Any DNS smart contract must define a “get method” with a “known signature”, which is invoked to look up a key. Since this approach makes sense for other smart contracts as well, especially those providing on-chain and mixed services, we explain it in some detail in 4.3.11.

### 4.3.11. “Get methods” of smart contracts. 

A better way would be to define some get methods in the smart contract, that is, some types of inbound messages that do not affect the state of the smart contract when delivered, but generate one or more output messages containing the “result” of the get method. In this way, one can obtain data from a smart contract, knowing only that it implements a get method with a known signature (i.e., a known format of the inbound message to be sent and outbound messages to be received as a result). 

This way is much more elegant and in line with object oriented programming (OOP). However, it has an obvious defect so far: one must actually commit a transaction into the blockchain (sending the get message to the smart contract), wait until it is committed and processed by the validators, extract the answer from a new block, and pay for gas (i.e., for executing the get method on the validators’ hardware). This is a waste of resources: get methods do not change the state of the smart contract anyways, so they need not be executed in the blockchain.

### 4.3.12. Tentative execution of get methods of smart contracts. 

We have already remarked (cf. 2.4.6) that any full node can tentatively execute any method of any smart contract (i.e., deliver any message to a smart contract), starting from a given state of the smart contract, without actually committing the corresponding transaction. The full node can simply load the code of the smart contract under consideration into the TON VM, initialize its persistent storage from the global state of the shardchain (known to all full nodes of the shardchain), and execute the smart-contract code with the inbound message as its input parameter. The output messages created will yield the result of this computation. 

In this way, any full node can evaluate arbitrary get methods of arbitrary smart contracts, provided their signature (i.e., the format of inbound and outbound messages) is known. The node may keep track of the cells of the shardchain state accessed during this evaluation, and create a Merkle proof of the validity of the computation performed, for the benefit of a light node that might have asked the full node to do so (cf. 2.5.11). 

### 4.3.14. Public interfaces of a smart contract. 

Notice that a formalized smart-contract interface, either in form of a TL-scheme (represented as a TL source file; cf. 2.2.5) or in serialized form,* can be published—for example, in a special field in the smart-contract account description, stored in the blockchain, or separately, if this interface will be referred to many times. In the latter case a hash of the supported public interface may be incorporated into the smart-contract description instead of the interface description itself. An example of such a public interface is that of a DNS smart contract, which is supposed to implement at least one standard get method for looking up subdomains (cf. 4.3.6). A standard method for registering new subdomains can be also included in the standard public interface of DNS smart contracts.

> *TL-schemes can be TL-serialized themselves; cf. <https://core.telegram.org/mtproto/TL-tl>

### 4.3.17. Locating user interfaces via TON DNS. 

The TON DNS record containing an abstract address of a ton-service or a smart-contract account identifier might also contain an optional field describing the public (user) interface of that entity, or several supported interfaces. Then the client application (be it a wallet, a ton-browser or a ton-proxy) will be able to download the interface and interact with the entity in question (be it a smart contract or a ton-service) in a uniform way.

### 4.3.19. A light wallet and TON entity explorer can be built into Telegram Messenger clients. 

An interesting opportunity arises at this point. A light wallet and TON entity explorer, implementing the above functionality, can be embedded into the Telegram Messenger smartphone client application, thus bringing the technology to more than 200 million people. Users would be able to send hyperlinks to TON entities and resources by including TON URIs (cf. 4.3.22) in messages; such hyperlinks, if selected, will be opened internally by the Telegram client application of the receiving party, and interaction with the chosen entity will begin.

### 4.3.22. Hyperlink URLs may specify some parameters. 

The hyperlink URLs may contain not only a (TON) DNS domain or an abstract address of the service in question, but also the name of the method to be invoked and some or all of its parameters. A possible URI scheme for this might look as follows: 

​          `ton://<domain>/<method>?<field1>=<value 1>&<field2>=. . .`

 When the user selects such a link in a ton-browser, either the action is performed immediately (especially if it is a get method of a smart contract,invoked anonymously), or a partially filled form is displayed, to be explicitly confirmed and submitted by the user (this may be required for payment forms).

### 4.3.23. POST actions. 

A ton-site may embed into the HTML pages it returns some usual-looking POST forms, with POST actions referring either to ton-sites, ton-services or smart contracts by means of suitable (TON) URLs. In that case, once the user fills and submits that custom form, the corresponding action is taken, either immediately or after an explicit confirmation. 

### 4.3.24. TON WWW. 

All of the above will lead to the creation of a whole web of cross-referencing entities, residing in the TON Network, which would be accessible to the end user through a ton-browser, providing the user with a WWW-like browsing experience. For end users, this will finally make blockchain applications fundamentally similar to the web sites they are already accustomed to. 

#### 4.3.25. Advantages of TON WWW. 

This “TON WWW” of on-chain and off-chain services has some advantages over its conventional counterpart. For example, payments are inherently integrated in the system. User identity can be always presented to the services (by means of automatically generated signatures on the transactions and RPC requests generated), or hidden at will. Services would not need to check and re-check user credentials; these credentials can be published in the blockchain once and for all. User network anonymity can be easily preserved by means of TON Proxy, and all services will be effectively unblockable. Micropayments are also possible and easy, because ton-browsers can be integrated with the TON Payments system.

## TON Payments

The last component of the TON Project we will briefly discuss in this text is TON Payments, the platform for (micro)payment channels and “lightning network” value transfers. It would enable “instant” payments, without the need to commit all transactions into the blockchain, pay the associated transaction fees (e.g., for the gas consumed), and wait five seconds until the block containing the transactions in question is confirmed. The overall overhead of such instant payments is so small that one can use them for micropayments. 

For example, a TON file-storing service might charge the user for every 128 KiB of downloaded data, or a paid TON Proxy might require some tiny micropayment for every 128 KiB of traffic relayed. While TON Payments is likely to be released later than the core components of the TON Project, some considerations need to be made at the very beginning. For example, the TON Virtual Machine (TON VM; cf. 2.1.20), used to execute the code of TON Blockchain smart contracts, must support some special operations with Merkle proofs. If such support is not present in the original design, adding it at a later stage might become problematic (cf. 2.8.16). We will see, however, that the TON VM comes with natural support for “smart” payment channels (cf. 5.1.9) out of the box. 

### 5.1 Payment Channels 

We start with a discussion of point-to-point payment channels, and how they can be implemented in the TON Blockchain. 

#### 5.1.1. The idea of a payment channel. 

Suppose two parties, A and B, know that they will need to make a lot of payments to each other in the future. Instead of committing each payment as a transaction in the blockchain, they create a shared “money pool” (or perhaps a small private bank with exactly two accounts), and contribute some funds to it: A contributes a coins, and B contributes b coins. This is achieved by creating a special smart contract in the blockchain, and sending the money to it. Before creating the “money pool”, the two sides agree to a certain protocol. They will keep track of the state of the pool—that is, of their balances in the shared pool. Originally, the state is (a, b), meaning that a coins actually belong to A, and b coins belong to B. Then, if A wants to pay d coins to B, they can simply agree that the new state is (a', b' ) = (a − d, b + d).

Afterwards, if, say, B wants to pay d 0 coins to A, the state will become (a '', b'') = (a' + d' , b' − d' ), and so on. All this updating of balances inside the pool is done completely off-chain. When the two parties decide to withdraw their due funds from the pool, they do so according to the final state of the pool. This is achieved by sending a special message to the smart contract, containing the agreed-upon final state (a ∗ , b∗ ) along with the signatures of both A and B. Then the smart contract sends a ∗ coins to A, b ∗ coins to B and self-destructs. 

This smart contract, along with the network protocol used by A and B to update the state of the pool, is a simple payment channel between A and B. According to the classification described in 4.1.2, it is a mixed service: part of its state resides in the blockchain (the smart contract), but most of its state updates are performed off-chain (by the network protocol). If everything goes well, the two parties will be able to perform as many payments to each other as they want (with the only restriction being that the “capacity” of the channel is not overrun—i.e., their balances in the payment channel both remain non-negative), committing only two transactions into the blockchain: one to open (create) the payment channel (smart contract), and another to close (destroy) it. 

#### 5.1.8. Challenges for the sophisticated payment channel smart contracts. 

Notice that, while the final state of a sophisticated payment channel is still small, and the “clean” finalization is simple (if the two sides have agreed on their amounts due, and both have signed their agreement, nothing else remains to be done), the unilateral finalization method and the method for punishing fraudulent behavior need to be more complex. Indeed, they must be able to accept Merkle proofs of misbehavior, and to check whether the more sophisticated transactions of the payment channel blockchain have been processed correctly. 

In other words, the payment channel smart contract must be able to work with Merkle proofs, to check their “hash validity”, and must contain an implementation of ev_trans and ev_block functions (cf. 2.2.6) for the payment channel (virtual) blockchain.

#### 5.1.9. TON VM support for “smart” payment channels. 

The TON VM, used to run the code of TON Blockchain smart contracts, is up to the challenge of executing the smart contracts required for “smart”, or sophisticated, payment channels (cf. 5.1.8). 

At this point the “everything is a bag of cells” paradigm (cf. 2.5.14) becomes extremely convenient. Since all blocks (including the blocks of the ephemeral payment channel blockchain) are represented as bags of cells (and described by some algebraic data types), and the same holds for messages and Merkle proofs as well, a Merkle proof can easily be embedded into aninbound message sent to the payment channel smart contract. The “hash condition” of the Merkle proof will be checked automatically, and when the smart contract accesses the “Merkle proof” presented, it will work with it as if it were a value of the corresponding algebraic data type—albeit incomplete, with some subtrees of the tree replaced by special nodes containing the Merkle hash of the omitted subtree. Then the smart contract will work with that value, which might represent, for instance, a block of the payment channel (virtual) blockchain along with its state, and will evaluate the *ev_block* function (cf. 2.2.6) of that blockchain on this block and the previous state. Then either the computation finishes, and the final state can be compared with that asserted in the block, or an “absent node” exception is thrown while attempting to access an absent subtree, indicating that the Merkle proof is invalid. 

In this way, the implementation of the verification code for smart payment channel blockchains turns out to be quite straightforward using TON Blockchain smart contracts. One might say that *the TON Virtual Machine comes with built-in support for checking the validity of other simple blockchains*. The only limiting factor is the size of the Merkle proof to be incorporated into the inbound message to the smart contract (i.e., into the transaction). 

## Appendix

## References

[1] K. Birman, Reliable Distributed Systems: Technologies, Web Services

and Applications, Springer, 2005.

[2] V. Buterin, Ethereum: A next-generation smart contract and decentralized application platform, <https://github.com/ethereum/wiki/>

wiki/White-Paper, 2013.

[3] M. Ben-Or, B. Kelmer, T. Rabin, Asynchronous secure computations with optimal resilience, in Proceedings of the thirteenth annual ACM

symposium on Principles of distributed computing, p. 183–192. ACM,

1994.

[4] M. Castro, B. Liskov, et al., Practical byzantine fault tolerance,

Proceedings of the Third Symposium on Operating Systems Design and

Implementation (1999), p. 173–186, available at [http://pmg.csail.mit.](http://pmg.csail.mit./)

edu/papers/osdi99.pdf.

[5] EOS.IO, EOS.IO technical white paper, <https://github.com/EOSIO/>

Documentation/blob/master/TechnicalWhitePaper.md, 2017.

[6] D. Goldschlag, M. Reed, P. Syverson, Onion Routing for Anonymous and Private Internet Connections, Communications of the ACM,

42, num. 2 (1999), <http://www.onion-router.net/Publications/>

CACM-1999.pdf.

[7] L. Lamport, R. Shostak, M. Pease, The byzantine generals problem,

ACM Transactions on Programming Languages and Systems, 4/3 (1982),

p. 382–401.

[8] S. Larimer, The history of BitShares, <https://docs.bitshares.org/>

bitshares/history.html, 2013.

[9] M. Luby, A. Shokrollahi, et al., RaptorQ forward error correction

scheme for object delivery, IETF RFC 6330, <https://tools.ietf.org/>

html/rfc6330, 2011.

[10] P. Maymounkov, D. Mazières, Kademlia: A peer-to-peer information system based on the XOR metric, in IPTPS ’01 revised papers from the First International Workshop on Peer-to-Peer Systems, p. 53–65, available at <http://pdos.csail.mit.edu/~petar/papers/>

maymounkov-kademlia-lncs.pdf, 2002.

[11] A. Miller, Yu Xia, et al., The honey badger of BFT protocols,

Cryptology e-print archive 2016/99, <https://eprint.iacr.org/2016/>

199.pdf, 2016.

[12] S. Nakamoto, Bitcoin: A peer-to-peer electronic cash system, https:

//bitcoin.org/bitcoin.pdf, 2008.

[13] S. Peyton Jones, Implementing lazy functional languages on stock

hardware: the Spineless Tagless G-machine, Journal of Functional Programming 2 (2), p. 127–202, 1992.

[14] A. Shokrollahi, M. Luby, Raptor Codes, IEEE Transactions on

Information Theory 6, no. 3–4 (2006), p. 212–322.

[15] M. van Steen, A. Tanenbaum, Distributed Systems, 3rd ed., 2017.
[16] The Univalent Foundations Program, Homotopy Type Theory:
Univalent Foundations of Mathematics, Institute for Advanced Study,

2013, available at <https://homotopytypetheory.org/book.>

[17] G. Wood, PolkaDot: vision for a heterogeneous multi-chain framework, draft 1, <https://github.com/w3f/polkadot-white-paper/raw/>

master/PolkaDotPaper.pdf, 2016.

