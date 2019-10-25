## Analyze & Test

For the time being, there is not way to analyze contract return values: the contract sends external messages, and the instrument which can parse them is yet to be developed.

Still, there are two ways to visualize contract execution results: peeking into persistent memory and transferring funds to other accounts.

For the issues below, we consider that you already sent all messages to contracts and now are analyzing new state of the blockchain.

## Persistent Memory

This method requires your contract to write values into persistent memory.

1. Open a terminal window.
2. Change your directory to <project-name>/build_contract_<contract-name>.c .
3. Start test-lite-client: **test-lite-client -С ton-global.json.**
4. Use the **getaccount** **0:<account address>** command to retrieve the contact data (expect a long list of values).
5. Find the '**data:**' string. It contains raw contents of the persistent memory specified after the string.
6. Close the terminal when finished.

## Transferring Funds 

This method requires your contract to send funds to another account.

1. Open a new terminal window.
2. Change your directory to` <project-name>/build_contract_<contract-name>.c`
3. Start test-lite-client: `**test-lite-client -С ton-global.json.**`
4. Use the `**getaccount** **0:<another account address**`**>** command to retrieve the contact data (expect a long list of values).
5. Find` **storage/balance/grams/amount**` field (it is close to the top of the account info) and check its value.
6. Close the terminal when finished.


  