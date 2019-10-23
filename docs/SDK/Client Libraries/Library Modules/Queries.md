Use the SDK to make queries to blockchain objects. A *live blockchain snapshot* is accessible through an ArangoDB instance in the local node.

To query it, GraphQL protocol with subscription options is implemented (get the schema at <http://127.0.0.1/graphql>). All data in the database are divided into following collections:

- *accounts*: blockchain account data;
- *transactions*: transactions related to accounts;
- *messages:* input and output messages related to transactions;
- *blocks:* blockchain blocks.

The structure of each collection item matches that on the TON blochchain. So, for additional details on specific fields refer to official TON documentation.

The `queries` module of the Client library defines four objects to access each collection: `accounts`, `transactions`, `messages` and `blocks`. 

###### Sample query to an account that returns account balance for an address.

```SQL
  query {
  accounts(
    filter: {
      id: {eq: "a46af093b38fcae390e9af5104a93e22e82c29bcb35bf88160e4478417028884"}
    }
  ) {
    id
    storage{
      balance{
        Grams
      }
    }
  }
}
```

Each object has a set of query methods: 

- `query`: filters collection items according to the requested condition. The available filtration functionality covers a wide range of tasks. If none of the collection items matches the request, an empty set is returned.
- `waitFor`: same as above, but this method never returns until the requested item appears (i.e. it actually waits).
- `subscribe`: starts monitoring the relevant blockchain for items matching the requests. The subscription monitors all insert and update operations.

## Making queries

To perform a query over the relevant blockchain, choose a collection and then specify a filter, result projection, sorting order and the maximum number of items in the results list.

```javascript
const transactions = await client.queries.transactions.query({
    now: { eq: 1567601735 }
}, 'id now status');
```

The example above demonstrates a query to the `transactions` collection with the following parameters:

- `filter`: a JSON object matching the internal collection structure. It supports additional conditions for particular fields. In the example above, the `now` field of a transaction must be equal to 1567601735.
- `result`: is a result projection that deter structural subset used for returning items. In the example above the request is limited to three fields: `id`, `now` and `status`. Note that results have to follow GraphQL rules.

Given that the `now` field is unique in the example, the `transactions` array is either empty or contains one item.

The `waitFor` method can be used to obtain items that are not yet written to the blockchain but are expected to appear.

```javascript
const transactions = await client.queries.transactions.waitFor({
    now: { eq: 1567601735 }
}, 'id now status');
```

The signature of the `waitFor` is exactly the same as for the `query`. The only difference is behavior: if there is no transaction with the specified `now` in the requested blockchain, this method waits indefinitely until the transaction appears in the blockchain.

## Subscriptions

Some applications monitor blockchain for a specific data set or for potential updates. The subscribe method (included in collection object) handles these scenarios:

```javascript
const subscription = client.queries.blocks.subscribe({}, 'id', (e, doc) => {
    console.log('On block have created: ', doc);
});

setTimeout(() => {
    subscription.unsubscribe();
    resolve();
}, 10*60*1000);
```

In this example, we start a subscription and react whenever a block is inserted or updated in the relevant blockchain.

The `filter` and `result` parameters are the same as in the `query` method. The `filter` parameter narrows the action down to a subset of monitored items. In this case, the filter is empty: all items are included into monitoring.

The last parameter is an event handler. This event is triggered every time the monitored block is inserted or updated in the relevant blockchain.

The return value of the `subscribe` method is a subscription object with one available method:`unsubscribe`. The subscription remains active until it is called. In the example above the subscription is cancelled within 10 min.

## Filtration and sorting

Filters applied to querying functions are data structures matching collection items with several extra features:

- The value for scalar fields (e.g. strings, numbers etc.) is a structure with the scalar filter.
- The value for array fields is a structure with an array filter.
- The value for nested structures is a filter for nested structure.

### Scalar filters

Scalar filter is a structure with one or more predefined fields. Each field defines a specific scalar operation and a reference value:

- `eq`: item value must be equal to the specified value;
- `ne` : item value must not be equal to the specified value;
- `gt`: item value must be greater than the specified value;
- `lt`: item value must be less than specified value;
- `ge`: item value must be greater than or equal to the specified value;
- `le`: item value must be less than or equal to the specified value;
- `in`: item value must be contained in the specified array of values;
- `notIn`: item value must not be contained within the specified array of values.

Filter example:

```javascript
{
    id: { eq: 'e19948d53c4fc8d405fbb8bde4af83039f37ce6bc9d0fc07bbd47a1cf59a8465'},
    status: { in: ["Preliminary", "Proposed", "Finalized"] }
}
```



> Note that when a scalar filter for a field contains multiple operators, the`AND` logical operator is used to combine all the conditions:

```javascript
{
    now: { gt: 1563449, lt: 2063449 }
} 
```

The logic from the above snippet can be expressed in the following way:

```javascript
(transaction.now > 1563449) && (transaction.now < 2063449)
```

### Array filters

Array filters are used for array (list) fields. Each has to contain at least one of the predefined operators:

- `any` : used when at least one array item matches the nested filter;
- `all`: used when all items matches the nested filter.

The `any` or `all` must contain a nested filter for an array item.

Array operators are mutually exclusive and can not be combined. For empty arrays, the array filter is assumed to be false.

### Structure filters

If an item is a structure, then a filter has to contain fields named as fields of this item. Each nested filter field contains a condition for the appropriate field of an item. The `AND` operator is used to combine conditions for several fields.

### Joins

The NoSQL database contains additional fields that work as cross-references for related collections. For example, the *transactions* collection has the `in_message` field that stores the relevant message item. The message item exists in *messages* collection and has the `id` value equal to the `in_msg` value in *transactions*.

Joined items are represented as nested structures in a filter and in the result projection.

### Sorting and limiting

By default, retrieval order for several items is not defined. To specify it, use the `orderBy` parameter of `query` method.

The sort order is represented by an array or sort descriptors. These structures contain two fields: `path` and `direction`:

- `path` specifies a path from a root item of the collection to the field that determines the order of return items. The path includes field names separated by dot.
- `direction` specifies the sorting order: `ASC` or `DESC` (ascending and descending).

You can specify more than one field to define an order. If two items have equal values for the first sort descriptor, then second descriptor is used for comparison, etc. If values of sorting fields are the same for several items, then the order of these items is not defined.

The `limit` parameter determines the maximum number of items returned. This parameter has a default value of 50 and can not exceed it. If specified limit exceeds 50, 50 is used.

## Special fields

Each items in each collection has a unique key stored in the `id` field. This ID is the same as the item blockchain identifier.

## Variability and Nested Levels 

GraphQL is designed to search for entities similar to documents. In our case these entities are represented by the above mentioned collections.

Obviously, each entity may have a set of fields. A field can be used as a filter. But, some complex fields (e.g. a message header) also include fields (values) and whole structures. These enclosed fields are not consistent even within a single collection. Therefore, you cannot make a query that is filtered by, say, `header` field alone. 

You have to build a complex query that drills down to а particular scalar field or field with primitive value (string, number or boolean) at the bottom level of the whole nested structure. 

As mentioned before, field structures may depend on document type or other conditions. In this case we have to use the GraphQL 'union' type which means that a value can have one of alternative types. For example the message `header` field depends on message type: internal, external inbound or external outbound, so we have three alternative variants for message header structure with three fields in the header (`MsgInt`, `MsgExtIn` or` MsgExtOut`).

In the projection part of GraphQl queries we can use '`...on`' syntax to specify a result set for every variant at any level. Thus, the sample implies two nested levels and the number is unlimited. you can drill down as deep, as needed.

```
...on field_name {
   field_name {
      filter_value
      }
  }
```

Once you type `...on`, autocomplete hints appear. 

See the usage example below.

```sql
query{
  messages(filter: { 
    header: {
      IntMsgInfo: {
        created_lt: { gt: 281 }
      },
    }
  }, orderBy: [{path:"header.IntMsgInfo.created_lt", direction: ASC}])
  { 
    id 
    header {
      ...on MessageHeaderIntMsgInfoVariant {
        IntMsgInfo {
          created_lt
        }
      }
    }
  }
}
```

We see that the existing approach and schema are not perfect. There are improvement options and wee seek to implement them.







