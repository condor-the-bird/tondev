Make sure to create a local folder for all your test projects.

## Creating a contract

1. Create the "hello" folder and place the "hello.sol" contract source code into it:

```javascript
pragma solidity >=0.5.0 <0.6.0;
contract HelloTON {
    uint32 deployTime;
  constructor() public {
        deployTime = uint32(now);
    }
    function sayHello() public view returns (uint32) {
        return deployTime;
    }
}   
```

2. Call `cd hello` to navigate to the new folder.

3. Run the TON Labs Solidity compiler:

```shell
tondev sol hello
```

## Creating an app

To get a functional playground, we need blockchain infrastructure for contract testing and debugging. We suggest using our Node SE. 

Make sure that you ran a Node SE instant according to the guidelines provided in the [**Installation**](/SDK/Installation/) section.

```shell
tondev start            
```

> **Note:** To create an application according to this procedure, you have to install the newest Rust compiler.

1. Initialize a Rust application in the hello directory:

```shell
~/ton-dev/hello$ cargo init
Created binary (application) package  
```

2. Open the newly created `Cargo.toml` file in your preferred editor and add following line to the `[dependencies]` section:

```shell
 ton-client-rs = "0.11.1"            
```

It enables using TON SDK Client package in your `hello` project.

3. Connect to the local node (Node SE) or to TON Labs testnet. In Rust Client Libraries the `TONClient` class is used to connect to TON Blockchain nodes that can work with SDK. 

If you use Node SE, specify `new_with_base_url("<http://0.0.0.0>")` (local URL).

> **Important**: for Windows use <http://127.0.0.1/> or [http://localhost](http://localhost/).

If you use the testnet, specify its URL. For test purposes, use Ton Labs test net URL `new_with_base_url('<https://testnet.ton.dev>')`

Go to the [main.rs](http://main.rs/) file in hello/src and paste the following code into it:

```shell
extern crate ton_client_rs;

use ton_client_rs::TonClient;

fn main() {
    let ton = TonClient::new_with_base_url("NodeSE/testnet URL").expect("Couldn't create TonClient");

    println!("Hello TON Done");
}
```

4. Run the client app from `hello` directory:

```shell
~/ton-dev/hello$ cargo run
        ...
     Running `target\\debug\\hello.exe`
Hello TON Done
~/ton-dev/hello$
```

5. Generate keys to sign the deploy transaction using `ton_client.crypto.generate_ed25519_keys()`function:

```rust
extern crate ton_client_rs;

use ton_client_rs::TonClient;

fn main() {
    let ton = TonClient::new_with_base_url("http://0.0.0.0").expect("Couldn't create TonClient");

    play_with_ton(ton);

    println!("Hello TON Done");
}

fn play_with_ton(ton_client: TonClient) {
    let keys = ton_client.crypto.generate_ed25519_keys().expect("Couldn't create key pair");
    println!("Generated keys: {}:{}", keys.secret, keys.public);
}
```

## Deployment

Before a contract can be deployed, the application needs retrieve: 

a compatible TVM code and an ABI structure. 

Both elements were obtained at the compilation stage and stored in the `hello.tvc` and `hello.abi.json`. 

> **Tip**: For more details on the ABI, see the [specification](https://docs.ton.dev/86757ecb2/p/70c253).

Below is the final contract deployment code:

```rust
...

fn play_with_ton(ton_client: TonClient) {
    let keys = ton_client.crypto.generate_ed25519_keys().expect("Couldn't create key pair");
    println!("Generated keys:\n{}\n{}", keys.secret, keys.public);

    let code = std::fs::read("hello.tvc").expect("Couldn't read code file");
    let abi = std::fs::read_to_string("hello.abi.json").expect("Couldn't read ABI file");

    let address = ton_client.contracts.deploy(&abi, &code, "{}".into(), &keys)
        .expect("Couldn't deploy contract");
    println!("Hello contract was deployed at address: {}", address);
}

...
```

Run the contract to see how it works.

```shell
~/ton-dev/hello$ cargo run
        ...
Running `target\\debug\\hello.exe`
Generated keys:
4236d45d544eccd16b16dea85f8aa201a7edfee06bb7f3e307c0ec02f9cb35ef
02154a29510c8bf29c4b4998e6510f60f224ab566eefaeaa9df22d7a90297b7e
Hello contract was deployed at address: b2a0d72b81d2cfae8bd3e7dc60c20a9f478570b8bea749318ff84fa0bd46d6bd
Hello TON Done
~/ton-dev/hello$   
```

## Running a contract

1. Run your contract; it is also quite simple:

```rust
...

fn play_with_ton(ton_client: TonClient) {
    let keys = ton_client.crypto.generate_ed25519_keys().expect("Couldn't create key pair");
    println!("Generated keys:\\n{}\\n{}", keys.secret, keys.public);

    let code = std::fs::read("hello.tvc").expect("Couldn't read code file");
    let abi = std::fs::read_to_string("hello.abi.json").expect("Couldn't read ABI file");

    let address = ton_client.contracts.deploy(&abi, &code, "{}".into(), &keys)
        .expect("Couldn't deploy contract");
    println!("Hello contract was deployed at address: {}", address);

    let response = ton_client.contracts.run(&address, &abi, "sayHello", "{}".into(), Some(&keys))
        .expect("Couldn't run contract");
    println!("Hello contract was responded to sayHello: {}", response);
}

...   
```

2. Then run the app:

```shell
~/ton-dev/hello$ cargo run
    ...
Running `target\\debug\\hello.exe`
Generated keys:
a7571d6041d45c261759caa04f73034396bf1a3aa35f092d5eb83a407f8284f8
1fcf08942290cf190287659f078199b0901e9403d852142f241c630e9b1ca6b3
Hello contract was deployed at address: d8634d6164b02c3a6c1447c505147ca1dde6149ba02b61f9eb21915d62467fde
Hello contract was responded to sayHello: {"value0":"0x5d78a4c7"}
Hello TON Done
~/ton-dev/hello$ 
```

> You can find more information about deploying and running in the [Contracts](/SDK/Contracts/) section.

## Querying blockchain

Each node server is equipped with a database that tracks the relevant blockchain.

This database is accessible through a GraphQL based protocol for querying blockchain.

The Client library contains the Query Module designed to perform GraphQL queries over a blockchain. The simplest way to query a blockchain is using the following `query` method:

```javascript
const TRANSACTION_FIELDS: &str = r#"
    id 
    now 
    status
"#;

let query_result = ton.queries.transactions.query(
        &json!({}).to_string(),
        TRANSACTION_FIELDS).unwrap();
println(query_result);
```

Then all transactions in the relevant blockchain are displayed (the first 50, to be precise). 50  is the default number used when no limit is specified or when it exceeds 50.

For each transaction, we have three result fields: **id**, **now** and **status**.

We have several options to filter the results:

```javascript
const TRANSACTION_FIELDS: &str = r#"
    id 
    now 
    status
"#;
let query_result = ton.queries.transactions.query(
        &json!({
            "now": {
                "eq": 1567601735
            }
        }).to_string(),
        TRANSACTION_FIELDS).unwrap(); 
```

The example gets all transactions with **now** equals to 1567601735. 

> Find more information about a filtering in the [**Queries**](/SDK/Client Libraries/Library Modules/Queries/) section.


