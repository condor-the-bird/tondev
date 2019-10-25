# Smart Contracts

***Q**: Based on your documentation and the white paper, there is a gas limit on transactions/transfers but this is not set by the sender but is a function of the transfer amount or contract balance.  Do senders/contracts have any ability to set an upper limit on the gas consumed by a transaction

**A**: contracts can set upper gas limit in `msg.value` (in nanograms), but receiver contract can increase this limit buying more gas (accept cmd). We cannot set gas limit  for external messages; it is automatically calculated as minimal contract balance or global gas limit per transaction.

------

**Q**: Do you have any insight into the various send modes for `SENDRAWMSG`?

I'm trying to understand  failure scenarios and if that call fails when `mode=0`, then the internal contract state fails to update and we risk the failed message being replayed until the account is drained.

`mode=2` is supposed to "ignore errors" such that the contract state will get updated even if the msg fails to send.  But by changing the mode to 2 I'm now seeing some additional fees taken out of my transfer amount (specifically my transfer amt is reduced by 0.001 Grams which is the `total_fwd_fee`)  Do you have any insight here?

**A**: Forward fees reduce transfer amount even if mode=0; these are always present in internal messages.

