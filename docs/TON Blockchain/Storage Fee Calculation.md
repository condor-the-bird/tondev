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

