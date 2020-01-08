# AMO client RPC specification
Tendermint provides various RPC methods to interact with Tendermint base and
ABCI app through it. See https://tendermint.com/rpc/ for details. Since AMO
blockchain is an application (ABCI app) built upon Tendermint, it provides all
the RPC methods which are provided by Tendermint itself. Among these methods,
`broadcast_tx_*` and `abci_query` are used to interact with AMO ABCI app:
sending transactions and querying blockchain data.

Tendermint RPC operates over three possible communication channels: URI/HTTP,
JSONRPC/HTTP, JSONRPC/websocket. This document assumes JSONRPC/HTTP, but
descriptions in this document can be applied to other communication channels
also.

Since we assume JSONRPC/HTTP RPC request data will be transmitted in HTTP POST
request body with content-type as `application/json`. The default RPC endpoint
is http://&lt;ip_addr&gt;:26657, but this may change according to the server
configuration. Request body is in JSON format and consists of RPC preamble(e.g.
`"jsonrpc":"2.0","id":"non-empty"`), rpc method, and rpc parameters. For
example:
```json
{
  "jsonrpc": "2.0",
  "id": "non-empty",
  "method": "abci_query",
  "params": {
    "path": "AMO specific path",
    "data": "AMO specific data",
  }
}
```
```json
{
  "jsonrpc": "2.0",
  "id": "non-empty",
  "method": "broadcast_tx_sync",
  "params": {
    "tx": "base64-encoded tx body"
  }
}
```
This document defines various parameters to `abci_query` and `broadcast_tx_*`
methods.

The response to the RPC request has the following form:
```json
{
  "jsonrpc": "2.0",
  "id": 0,
  "error": {}, // empty when no error
  "result": {} // empty when no result
}
```
`result` is different for each method.

## ABCI Query
ABCI query is used to query blockchain data stored in a blockchain node. In
Tendermint terms, `params` contains two fields, `path` and `data`.

Blockchain data are stored in several logical data stores. `path` decides which
data store to look for:
- `/config`: config of AMO abci app
- `/balance`: account's AMO coin balance
- `/stake`: account's stake
- `/delegate`: account's delegated stake
- `/validator`: validator's holder and list of appointed delegated stakes
- `/storage`: data storage registration
- `/parcel`: data parcel registration
- `/request`: data parcel request
- `/usage`: granted data parcel usage
- `/inc_block`: incentive history of given block height
- `/inc_address`: incentive history of given address
- `/inc`: incentive history of given block height and address

`data` specifies the key to find a match in a data store. `data` must be a
HEX conversion of a proper JSON object.

Full rpc request body of `abci_query`:
```json
{
  "jsonrpc": "2.0",
  "id": "non-empty",
  "method": "abci_query",
  "params": {
    "path": "/balance",
    "data": "HEX-conversion of a double-quoted HEX-encoded account address"
  }
}
```
Note that the data is HEX of a quoted HEX! This is due to the rule that the
`data` param should be a HEX-conversion of query data, and a query data should
be a proper JSON object. To query balance of an account
`85B5ED98D4BF06D6A2540087D16C39725B87C65E`, we need to set the query data to be
`223835423545443938443442463036443641323534303038374431364333393732354238374336354522`.
This is a HEX conversion of `"85B5ED98D4BF06D6A2540087D16C39725B87C65E"`. For
an opaque form of each data type, see [AMO Blockchain Protocol
Specification](protocol.md).

An RPC result for ABCI query has the following form:
```json
{
  "response": {
    "log": "optional log string from the app",
    "key": "data key used in the query request",
    "value": "base64 encoded response value"
  }
}
```
where `value` is different for each query path. We need first to decode a
base64-encoded `value` to get a meaningful JSON representation for the real
query result.

### Query app config
Since there is only one set of app config, `data` for this query is irrelevant
and should be set as JSON object `null`. When transmitted as `params` in ABCI
query message, it would be `6e756c6c`. The resulting query message is as the following:
```json
{
  "jsonrpc": "2.0",
  "id": 0,
  "method": "abci_query",
  "params": {
    "path": "/config",
    "data": "6e756c6c",
  }
}
```

The response value for this query has the following form:
```json
{
  "max_validators": 100,
  "weight_validator": 2,
  "weight_delegator": 1,
  "min_staking_unit", "_currency_",
  "blk_reward": 0,
  "tx_reward": 1e+17,
  "penalty_ratio_m": 0.1,
  "penalty_ratio_l": 0.01,
  "laziness_counter_window": 100,
  "laziness_threshold": 0.9,
  "block_bound_tx_grace_period": 100,
  "lockup_period": 3600,
  "draft_vote_open": 500000,
  "draft_vote_close": 100000,
  "draft_apply": 500000
}
```

### Query balance
When querying an account's balance, `data` is a HEX conversion of a JSON
representation of an account address.
```json
"_HEX_encoded_account_address_"
```

The response value for this query has the following form:
```json
"_decimal_number_"
```

### Query stake
When querying an account's stake, `data` is a HEX conversion of a JSON representation of an account address.
```json
"_HEX_encoded_account_address_"
```

### Query delegate
When querying an account's delegated stake, `data` is a HEX conversion of a JSON representation of an account address.
```json
"_HEX_encoded_account_address_"
```

### Query validator
When querying a validator's holder and list of appointed delegated stakes,
`data` is a HEX conversion of a JSON representation of a validator address.
```json
"_HEX_encoded_validator_address_"
```

TODO: reference to validator address

### Query draft
When querying a draft detail and voting progress, `data` is a HEX conversion of a JSON representation of a draft ID.
```json
"_draft_id_"
```

### Query storage
When querying a storage detail, `data` is a HEX conversion of a JSON representation of a storage ID.
```json
"_storage_id_"
```

### Query parcel
When querying a parcel, `data` is a HEX conversion of a JSON representation of
a parcel ID.
```json
"_HEX_encoded_parcel_id_"
```

### Query request
When querying a request for specific parcel, `data` is a HEX conversion of a
compact JSON representation of (`buyer address`, `data parcel ID`), where this
JSON representation is as the following:
```json
{"buyer":"_HEX_encoded_account_address_","target":"_HEX_encoded_parcel_id_"}
```

### Query usage
When querying a usage for specific parcel, `data` is a HEX conversion of a
compact JSON representation of (`buyer address`, `data parcel ID`), where this
JSON representation is as the following:
```json
{"buyer":"_HEX_encoded_account_address_","target":"_HEX_encoded_parcel_id_"}
```

### Query incentive for block
When querying an incentive record for a specific block height, `data` is a HEX
conversion of a _string-formatted_ block height.
```json
"_deciman_number_"
```

### Query incentive for account
When querying an incentive record for a specific stake holder or delegated stake
holder, `data` is a HEX conversion of a JSON representation of an account
address.
```json
"_HEX_encoded_account_address_"
```

### Query incentive for block and account
When querying an incentive record for a specific block height and a specific
stake holder or delegated stake holder, `data` is a HEX conversion of a compact
JSON representation of (`block height`, `holder address`), where this JSON
representation is as the following:
```json
{"height":"_decimal_number_","address":"_HEX_encoded_account_address_"}
```

## Broadcast Tx
Broadcast the transaction among the AMO blockchain nodes. There are three
types of broadcast methods in `broadcast_tx_*` family.
- `broadcast_tx_async`: returns right away, with no indication of success or
  failure.
- `broadcast_tx_sync`: waits for tx to be included in a _mempool_ and returns
  the result. The result includes basic validation result.
- `broadcast_tx_commit`: waits for tx to be included and committed in a _block_
  and returns the result. The result includes basic validation result and any
  error during delivering the tx contents according to the current context of
  the AMO blockchain state.

Full rpc request body of `broadcast_tx_commit`:
```json
{
  "jsonrpc": "2.0",
  "id": "non-empty",
  "method": "broadcast_tx_commit",
  "params": {
    "tx": "_base64_encoded_tx_body_"
  }
}
```
`tx` is a base64-encoded transaction body. (Although we are trying to use HEX
encoding for almost every situation when we need to encode a binary data, we
need to use base64 encoding for `broadcast_tx_*` methods. This is because
Tendermint wants base64 encoding here.) For detailed transaction body format
see [AMO Blockchain Protocol Specification](protocol.md#transaction).

An example RPC message to send `transfer` transaction is as the following (in
pretty format):
```json
{
  "jsonrpc": "2.0",
  "id": 0,
  "method": "broadcast_tx_sync",
  "params": {
    "tx": "eyJ0eXBlIjoidHJhbnNmZXIiLCJzZW5kZXIiOiI2NjJFM0REMUM2NDcwQ0ZFMTJDOEVEQkNFNUY0NEMwOEUyNzYzNzUzIiwiZmVlIjoiMCIsImxhc3RfaGVpZ2h0IjoiNDA1MiIsInBheWxvYWQiOnsidG8iOiI2MTRBOUYyRkM0RTZCMTE5RDc2MTJDMzVCQzE1MEUzM0NCMzhCQjQwIiwiYW1vdW50IjoiMTAwIn0sInNpZ25hdHVyZSI6eyJwdWJrZXkiOiIwNGRiY2VjMmMwZjUyMDE4NjA2ZjU4ODcxMzMwNWUxZGE0OTM2NzAzNzI4MWI5NjBmNTFjNDZiZTY0ZTMxNDQ5NzcwMDlhODExYTg2NWIzY2IzMzMxYjc4ODE0N2MwMzg1M2M3OTIwYzRjOGZiNmZmYjViMGQ0MzVkYWViM2Y1OWE0Iiwic2lnX2J5dGVzIjoiNTBhODMwN2FhZmY2NjExYWU2N2FkZDA5ZWE4MTNmMzc2NjgwNzJhMjE0MjMwZGYzNzVjZmEyNWZiMzY4YjBlYmQ4NjE5NDM2NjFlYzY5MGFlMGU1ZDc4OWU3MzhiM2M0NTE4Zjc4ZDc2OGU1ZTAwNmM5ZWI1M2U4MTgyMTY3MWQifX0="
  }
}
```
where `tx` is a base64 encoding of the following tx body:
```json
{"type":"transfer","sender":"662E3DD1C6470CFE12C8EDBCE5F44C08E2763753","fee":"0","last_height":"4052","payload":{"to":"614A9F2FC4E6B119D7612C35BC150E33CB38BB40","amount":"100"},"signature":{"pubkey":"04DBCEC2C0F52018606F588713305E1DA49367037281B960F51C46BE64E3144977009A811A865B3CB3331B788147C03853C7920C4C8FB6FFB5B0D435DAEB3F59A4","sig_bytes":"50A8307AAFF6611AE67ADD09EA813F37668072A214230DF375CFA25FB368B0EBD861943661EC690AE0E5D789E738B3C4518F78D768E5E006C9EB53E81821671D"}}
```

An RPC result for broadcasting tx hash the following form:
```json
{
  "check_tx": {},
  "deliver_tx": {},
  "hash": "_HEX_encoded_transaction_hash_",
  "height": "_decimal_number_"
}
```
Each field contains the following data:

| broadcast | `check_tx` | `deliver_tx` | `height` |
|-|-|-|-|
| async | `{}` | `{}` | `""` |
| sync  | ok / err | `{}` | `""` |
| commit | ok / err | ok / err | `"_decimal_number_"` / `""` |

`check_tx` and `deliver_tx` field contain `"info":"ok"` when success, `code`
and `info` when error. For `broadcast_tx_async` method, both of `check_tx` and
`deliver_tx` fields are empty. For `broadcast_tx_sync` method, only `check_tx`
field contains the result. For `broadcast_tx_commit` method, both of the fields
contain the result. `height` field is relevant only when the method is
`broadcast_tx_commit` and the tx is included successfully. It contains the
height number of the block in which the tx is included.

