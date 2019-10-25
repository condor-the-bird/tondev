# TON Blockchain

## 1.1 Everything is a bag of cells

### 1.1.4. The sha256 hash of a cell. 

The sha256 hash of a cell c is recursively defined as the sha256 of the standard representation CellRepr(c) of the cell in question:

​                         `Hash(c) := sha256(c) := sha256 CellRepr(c) `

Because cyclic cell references are not allowed (the relationships among all cells must constitute a directed acyclic graph, or DAG), the sha256 hash of a cell is always well-defined. Furthermore, because sha256 is tacitly assumed to be collision-resistant, we assume that all the cells that we encounter are completely determine by their hashes. In particular, the cell references of a cell c are completely determined by the hashes of the referenced cells, contained in the standard representation `CellRepr(c)`. 

Messages, message descriptors, and queues

Under construction. First we add chapters we have to refer to from our product docs.

## 1.1.8. Transparent hash and representation hash of a cell.

In this way, sha256(c) =Hash[(c)is the transparent hash of a cell (c) or the tree of cells rooted in (c). However, sometimes we need to reason about the exact representation of a tree of cells present in a block. To this end, are *presentation hash* Hash(c) is defined, which is not transparent with respect to reference cells and other exotic types of cells. We often say that the representation hash of *c* is “the”hash of c, because it is the most frequently used hash of a cell





## 3.1 Address, currency, and message layout 

### 3.1.7. Message layout. 

A message consists of its header followed by its body, or payload. The body is essentially arbitrary, to be interpreted by the destination smart contract. The message header is standard and is organized as follows:

 `int_msg_info$0 ihr_disabled:Bool bounce:Bool src:MsgAddressInt dest:MsgAddressInt value:CurrencyCollection ihr_fee:Grams fwd_fee:Grams created_lt:uint64 created_at:uint32 = CommonMsgInfo; ext_in_msg_info$10 src:MsgAddressExt dest:MsgAddressInt import_fee:Grams = CommonMsgInfo; ext_out_msg_info$11 src:MsgAddressInt dest:MsgAddressExt created_lt:uint64 created_at:uint32 = CommonMsgInfo; tick_tock$_ tick:Bool tock:Bool = TickTock; _ split_depth:(Maybe (## 5)) special:(Maybe TickTock) code:(Maybe ^Cell) data:(Maybe ^Cell) library:(Maybe ^Cell) = StateInit; 56 3.1. Address, currency, and message layout message$_ {X:Type} info:CommonMsgInfo init:(Maybe (Either StateInit ^StateInit)) body:(Either X ^X) = Message X; `

The meaning of this scheme is as follows. 

Type Message X describes a message with the body (or payload) of type X. Its serialization starts with info of type CommonMsgInfo, which comes in three flavors: for internal messages, inbound external messages, and outbound external messages, respectively. All of them have a source address src and destination address dest, which are external or internal according to the chosen constructor. Apart from that, an internal message may bear some value in Grams and other defined currencies, and all messages generated inside the TON Blockchain have a logical creation time created_lt (cf. 1.4.6) and creation unixtime created_at, both automatically set by the generating transaction. The creation unixtime equals the creation unixtime of the block containing the generating transaction.