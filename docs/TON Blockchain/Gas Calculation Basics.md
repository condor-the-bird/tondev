# Gas Calculation Basics

## TON Gas Implementation

### **Specification Overview**

The entire state of TVM consists of the five components:

- stack
- control registers
- current continuation
- current codepage
- gas limits

Collectively these are called SCCCG.

The **Gas** component limits gas usage and сontains four signed 64-bit integers:

- the remaining gas: **gr**
- the current gas limit: **gl**
- the maximal gas limit: **gm**
- the gas credit: **gc**.

The following is always true:

**0 ≤ gl ≤ gm, gc ≥ 0, and gr ≤ gl + gc**

**gc** is initialized by zero for internal messages, **gr** is initialized by **gl + gc** and gradually decreases, as the TVM runs. When **gr** becomes negative or if contract terminates with **gc > 0**, an **out of gas** exception is triggered.

According to the original TON, for most primitives gas is calculated according to the following formula:

**Pb := 10 + b**

where **b** is the instruction length in bits. The same is true for TON Labs implementation.

Apart from integer constants, the following expressions may appear:

- The total price of loading cells. Currently it is 100 gas units per cell.
- The total price of creating new Cells from Builders. Currently it is 500 gas units.
- Exception throwing. 50 gas units per exception.
- Tuple gas price. 1 gas unit for every tuple element.

The usage of these additional integers remains unclear now. Research is underway.

### Global gas limits

Global gas limits are values stored in the masterchain configuration contract. Global values are standard and do not change at contract deployment. Only validator consensus can modify them.

The following values are now used for shardchains:

- Global_gas_price = 1000
- Global_gas_limit = 1000000
- Global_gas_credit = 10000
- Global_block_gas_limit = 10000000

### Gas-related TVM primitives

These is the list of official TVM primitives used for gas-related operations:

- F800 — ACCEPT, sets current gas limit gl to its maximal allowed value gm, and resets the gas credit gc to zero, decreasing the value of gr by gc in the process. In other words, the current smart contract agrees to buy some gas to finish the current transaction. This action is required to process external messages, which bring no value (hence no gas) with themselves.
- F801 — SETGASLIMIT (g – ), sets current gas limit gl to the minimum of g and gm, and resets the gas credit gc to zero. If the gas consumed so far (including the present instruction) exceeds the resulting value of gl , an (unhandled) out of gas exception is thrown before setting new gas limits. Notice that SETGASLIMIT with an argument g ≥ 2 63 − 1 is equivalent to ACCEPT.
- F802 — BUYGAS (x – ), computes the amount of gas that can be bought for x nanograms, and sets gl accordingly in the same way as SETGASLIMIT.
- F804 — GRAMTOGAS (x – g), computes the amount of gas that can be bought for x nanograms. If x is negative, returns 0. If g exceeds 2 63−1, it is replaced with this value.
- F805 — GASTOGRAM (g – x), computes the price of g gas in nanograms.
- F806–F80F — Reserved for gas-related primitives. These are yet to be released.

All of the above are operational in the TON TVM implementation.

## TON Labs Implementation

The general gas formula is the same as specified by TON specifications. Overall, TON Labs nodes operate in compliance with the specification.

For every executed primitive, the amount of gas is added to the virtual machine according to the specification formula. Gas value for every primitive is based on **gr**.

### Gas initialization types

**1. Calling contract from another contract**

An internal message with a balance value is received. In this case, the following formulas are applied to determine limits:

- gm = min(account balance / gas price, global_gas_limit)
- gl = min(message value / gas price, global_gas_limit)
- gc = 0
- gr = gc + gl

By default gas costs are allocated to the caller contract that triggers the transaction with a message. Accepting is also available for internal contracts. If ACCEPT is not called, gas is taken from the caller contract according to the message value. In other words, the message value defines the current limit. The message value determines the starting TVM gas limit.

So, to put it plain, if ACCEPT is not called, the message pays, if ACCEPT is used, additional gas can be bought by the target contract. This approach enables flexible contract design where either total gas is paid by the caller contract (but in this case it has to have enough gas at any moment of time) or the target contract also incurs costs.

**2. Offchain contract call**

External messages do not carry balance values. In this case, the values are calculated according to the following formulas:

- gm = min(account balance / gas price, global_gas_limit)
- gl = 0
- gc = min(gm, global_gas_credit)
- gr = gc + gl

As external messages have no gas value, gas is credited to execute it. Target contracts have to cover costs by calling Accept to buy gas.

If a contract returns an exception before the credit is given, no gas fee applies

As the public code for node has just been released this documentation is likely to be updated.

# Storage Fee Calculation

Every transaction in TON has a storage phase that implies a certain storage fee charged on an account balance. This fee is charged for the period between transactions and is calculated according to the following formula:

```
Storage fee =(account.bits*global_bit_price+account.cells*global_cell_price)*period
```

where:

- `account.bits` and `account.cells` stand for a number of bits and cells in the Account structure represented as tree of cells (including code and data).
- `global_bit_price` is a global configuration parameter; price for storing one bit. Currently in the testnet it is 1 nanogram.
- `global_cell_price` another global configuration parameter; price for storing one cell. Currently in the testnet it is 500 nanogram.
- period - number of seconds since previous storage fee payment.

If the account balance is less than the due storage fee, the account is frozen and its balance is subtracted from storage fee and reduced to zero. Remaining storage fee is stored in account as debt

