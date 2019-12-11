# AMO client RPC specification
Tendermint provides various RPC methods to interact with Tendermint base and ABCI app through it. See https://tendermint.com/rpc/ for details. Since AMO is a blockchain application (ABCI app) built upon Tendermint, it provides all the RPC methods which are provided by Tendermint itself. Among these methods, `broadcast_tx_*` and `abci_query` are used to interact with AMO ABCI app: sending transactions and querying app state.

Tendermint RPC operates over three possible communication channels: URI/HTTP, JSONRPC/HTTP, JSONRPC/websocket. This document assumes JSONRPC/HTTP, but descriptions in this document can be applied to other communication channels also.

(Since we assume JSONRPC/HTTP)
RPC request data will be transmitted as HTTP POST request body with content-type as `application/json`. The default RPC endpoint is http://localhost:26657, but this may change according to the server configuration. Request body is in JSON format and consists of RPC preamble(e.g. `"jsonrpc":"2.0","id":"non-empty"`), rpc method, and rpc parameters. For example:
```json
{"jsonrpc":"2.0","id":"non-empty","method":"abci_query","params":{"path":"AMO specific path","data":"AMO specific data","height":"0","prove":"false"}}
{"jsonrpc":"2.0","id":"non-empty","method":"broadcast_tx_sync","params":{"tx":"AMO specific transaction"}}
```
This document defines various parameters to `abci_query` and `broadcast_tx_*` methods.

NOTES:
- JSONRPC doesn't support number expression(e.g. 300), so a number should be represented as double-quoted string like "300".
- Binary data such as byte-strings should be encoded by base64.

## ABCI Query parameters
Ignore `height` and `prove` for now.

`path` decides which data store the app should look for:
- `/app_config`: config of AMO abci app
- `/balance`: list of accounts' AMO coin balances
- `/stake`: list of accounts' stake values
- `/delegate`: list of accounts' delegated stake values
- `/validator`: address of validator's stake holder
- `/parcel`: list of data parcel registrations
- `/request`: list of data parcel requests
- `/usage`: list of granted data parcel usages
- `/inc_block`: list of accounts' incentive history of given block height
- `/inc_address`: list of accounts' incentive history of given address
- `/inc`: list of acoounts' incentive history of given block height and address

`data` specifies the key to find a match in the data store. `data` must be a `hex` conversion of a proper JSON object.

In full RPC request body:
```json
{
    "jsonrpc":"2.0",
    "id":"non-empty",
    "method":"abci_query",
    "params": {
        "path": "/balance",
        "data": "hex conversion of \"12FF....\""
    }
}
```
### App Config Query
When querying AMO abci app's *app_config*,`data` is null

### Balance query
When querying an account's _balance_, `data` is a `hex` conversion of the double-quoted _address_ of the account.
```json
"_address_"
```

### Stake query
When querying an account's _stake_, `data` is a `hex` conversion of the double-quoted _address_ of the account.
```json
"_address_"
```

### Delegate query
When querying an account's _delegated stake_, `data` is a `hex` conversion of the double-quoted _address_ of the account.
```json
"_address_"
```

### Validator query
When querying a _validator_'s holder and list of appointed _delegated stakes_,
`data` is a `hex` conversion of the double-quoted _address_ of the validator.
```json
"_address_"
```

### Parcel query
When querying a _parcel_, `data` is a `hex` conversion of the double-quoted _data parcel ID_.
```json
"_parcel_ID_"
```

### Request query
When querying a _request_ for specific _parcel_, `data` is a `hex` conversion of a JSON representation of (`buyer address`, `data parcel ID`), where this JSON representation is as the following example:
```json
{ "buyer" : "_address_", "target" : "_parcel_ID_" }
```

### Usage query
When querying a _usage_ for specific _parcel_, `data` is a `hex` conversion of a JSON representation of (`buyer address`, `data parcel ID`), where this JSON representqtion is as the following example:
```json
{ "buyer" : "_address_", "target" : "_parcel_ID_" }
```

### Incentive Block query
When querying a *incentive_info* of a specific block height, `data` is a `string`-formatted block height.
```json
"_height_"
```

### Incentive Address query
When querying a *incentive_info* of a specific stake or delegate holder, `data` is a `hex` conversion of the double-quoted address of the account.
```json
"_address_"
```

### Incentive query
When query a *incentive_info* of a specific block height and a specific stake or delegate holder, `data` is a `hex` conversion of a JSON representation of (`block height`, `holder address`), where this JSON representation is as the following example:
```json
{ "height": "_height_", "address": "_address_" }
```

## Broadcast tx parameters
Broadcast the transaction. There are three options exist in `broadcast_tx_*`.
- `broadcast_tx_async` returns right away, with no reponse.
- `broadcast_tx_sync` waits for `CheckTx` response and returns it.
- `broadcast_tx_commit` does `CheckTx` && `DeliverTx` to `abci`.
This method returns an error only if _mempool.CheckTx()_ errs or timeout for waiting `tx` commit. When `CheckTx` || `DeliverTx` failed, it returns result containing _non-OK_ `abci` code.

Full rpc request body of `broadcast_tx_commit`:
```json
{
    "jsonrpc":"2.0",
    "id":"non-empty",
    "method":"broadcast_tx_commit",
    "params": {
        "tx": "0x_the_hex_string_byte_code_for_abci_code"
    }
}
```
`tx` is a serialized `message` which is byte code for `abci` command execution.
The `message` is also JSON style and an example of body is shown below.
An indication of *(implicit*) means that it can be considered as a `_sender_address_`.

Full `message` body of `transfer`:

```json
{
    "type": "_tx_type_",
    "sender": "_sender_address_",
    "nonce": "_HEX-encoded_nonce_bytes_",
    "payload": {
		"to": "_recipient_address_",
		"amount": "_currency_"
	},
    "signature": {
        "pubkey": "_HEX-encoded_public_key_bytes_",
        "sig_bytes": "_HEX-encoded_signature_bytes_"
    }
}
```
## Operations

### Transfer coin
Transfer AMO coin the amount of `amount` to the address `to`. This command causes a chage in the state of the `store/balance`.

- tx type : `transfer`
- affected store : `balance`
- `owner_address`  *(implicit)*

```json
{ "to" : "_recipient_address_", "amount" : "_currency_" }
```

### Stake coin
Lock AMO coin as a `stake` of the coin holder, or lock additional coin and increase `stake` value.

- tx type: `stake`

```json
{ "validator": "_validator_pubkey_", "amount": "_currency_" }
```

### Withdraw coin
Withdraw all or part of the AMO coin locked as a `stake`.

- tx type: `withdraw`

```json
{ "amount": "_currency_" }
```

### Delegate stake
Lock sender's AMO coin as a **delegated** `stake` appointed to a delegatee, or lock additional coin and increase delegated `stake` value.

- tx type: `delegate`

```json
{ "to": "_delegatee_address_", "amount": "_currency_" }
```

### Retract stake
Retract all or part of the AMO coin locked as a delegated stake.

- tx type: `retract`

```json
"amount": "_currency_"
```

### Register Data

Register `parcel` with `extra`( price, description, expired_date, etc... ). This command causes a chage in the state of the `store/parcel`.

- tx type : `register`
- affected store : `parcel`
- `owner_address`  *(implicit)*

```json
{ "target" : "_parcel_id_", "key_custody" : "_owner_custody_", "extra" : "_extra_info_" }
```

### Request Data

Request `parcel` to purchase with `payment` as offer amount and `extra` ( expired_data, etc...). This is not the end of purchasing process, but the amount of `payment` will be *locked*. The transaction will be stored in `store/request` and waits to be granted by *owner*. 

- tx type : `request`
- affected store : `request`
- `buyer_address`  *(implicit)*

```json
{ "target" : "_parcel_id_", "payment" : "_currency_", "extra" : "_extra_info_" }
``` 

### Grant Data Usage

Grant the `request` of `parcel` in `store/request` by *data owner*. Specify `grantee` to avoid confusion with other purchasers of the same `parcel`. After `grant` is recorded in AMO blockchain, *locked* AMO coin will be added in owner's balance. The `request` in `store/request` will be deleted and added in `store/usage` as (`buyer_address`, `parcel_id`, `key_custody`, `exp_condition`).

- tx type : `grant`
- affected store : `request`, `balance`, `usage`.
- `owner_address`  *(implicit)*

```json
{ "target" : "_parcel_id_", "grantee" : "_buyer_address_", "key_custody" : "_buyer_custody_" }
```

### Discard Data

Discard the registered data in `store/parcel`. After `discard` is recorded in AMO blockchain, delete `parcel` corresponding (`parcel_id`, `owner_address`, `key_custody`, `extra_info`). 

- tx type : `discard`
- affected store : `parcel`
- `owner_address`  *(implicit)*

```json
{ "target" : "_parcel_id_" }
```

### Cancel Request

Cancel the `request` of `parcel` in `store/request`. It deletes the previous `request_data` of `myself_address` in `store/request` and releases the amount of `payment` which was *locked*.

- tx type : `cancel`
- affected store : `request`
- `myself_address`  *(implicit)*

```json
{ "target" : "_parcel_id_" }
```

### Revoke Data Usage

Delete the `usage` of `parcel` in `store/usage`.

- tx type : `revoke`
- affected store : `usage`
- `owner_address`  *(implicit)*

```json
{ "target" : "_parcel_id_", "grantee" : "_buyer_address_" }
```

### Upload Data (PDB operation)
### Retrieve Data (PDB operation)
### Delete Data (PDB operation)
