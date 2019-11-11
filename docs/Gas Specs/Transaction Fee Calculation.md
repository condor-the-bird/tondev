Transaction fees consist of a few types of different fees connected to the execution of a single transaction. Transactions itself are complex processes, and fees are paid relative to different stages of executing them.

In this section, we explain how the fees are calculated.

We shall define `transaction_fee` is a sum of all fees for a single transaction.

`transaction_fee = trans.total_fees + N*msg.header.fwd_fee`;

where N is the number for outbound messages included in the transaction.

Here, `trans.total_fees` is a sum of all fees that are not dependent on the number of messages in the transaction meaning it excludes the outbound message commission. Outbound message fees might be absent if no outbound messages are sent in the process of completing the transaction.

It is calculated according to the following formula:

```
trans.total_fees = in_fwd_fee + trans.total_action_fees + trans.gas_fees + trans.storage_fee
```

`trans.gas_fees` include all gas fees associated with the transaction. You can find more info in the Gas Calculation Basics document.

`trans.total_action_fees` - fees for performing actions. TON transactions currently have three types of actions that add a fee:

1. set.code - contract code update;
2. send.message - sending a message;
3. reserve.balance - saving balance.

`trans.storage_fees` - storage costs since the moment of the last transaction. At this time there's no sufficient data on how exactly these fees calculate and apply. Update pending.

```
in_fwd_fee` - external inbound message broadcast forwarding fee. It is not directly shown in the transaction data, yet it is calculated as a part of the fees. It is calculated according to the following formula: `msg_fwd_fees = (lump_price + ceil((bit_price * msg.bits + cell_price * msg.cells)/2^16)) nanograms
```

This is a complete formula, but not all of the fees are present in every transaction. For example, the `trans.gas_fees` can be skipped if the TVM compute phase is not initialized in a transaction. The action fee might be absent as well if no actions described above are performed during the transaction.

`trans.total_fwd_fees` is a separate way to calculate total forwarding and action fees.

```
trans.total_fwd_fees = trans.total_action_fees + msg.header.fwd_fee;
```

`msg.header.fwd_fee` - outbound message forwarding fee. There can be a few according to the number of forwarded messages as well as it can be absent if no messages are forwarded during the transaction. It is defined in the message header.