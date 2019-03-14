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
- `/balance`: list of account AMO coin balances
- `/parcel`: list of data parcel registrations
- `/request`: list of data parcel requests
- `/usage`: list of granted data parcel usages

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

### Balance query
When querying an account's _balance_, `data` is a `hex` conversion of the double-quoted _address_ of the account.

### Parcel query
When querying a _parcel_, `data` is a `hex` conversion of the double-quoted _data parcel ID_.

### Request query
When querying a _request_ for specific _parcel_, `data` is a `hex` conversion of a JSON representation of (`buyer address`, `data parcel ID`), where this JSON representation is as the following example:
```json
{ "buyer" : "_address_", "parcel" : "_parcel_ID_" }
```

### Usage query
When querying a _usage_ for specific _parcel_, `data` is a `hex` conversion of a JSON representation of (`buyer address`, `data parcel ID`), where this JSON representqtion is as the following example:
```json
{ "buyer" : "_address_", "parcel" : "_parcel_ID_" }
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

Full `message` body of `transfer`:
```json
{
    "command":"transfer_coin",
    "signer":"_address_sender_",
    "signing_pub_key":"_pubkey_sender_",
    "signature":"_signature_",
    "payload":{
        "to":"_address_receiver",
        "amount":"_amount_"
    },
    "nonce":"_nonce_"
}
```

## Operations

### Transfer coin
Transfer AMO coin the amount of `amount` to the address `recipient`. This command causes a chage in the state of the `store/balance`.

- command : `transfer`
- affected store : `balance`
- `owner_address`  *(implicit)*

```json
{ "recipient" : "_recipient_address_", "amount" : "_amount_" }
```

### Upload Data (PDB operation)
### Register Data

Register `parcel` with `extra_info`( price, description, expired_date, etc... ). This command causes a chage in the state of the `store/parcel`.

- command : `register`
- affected store : `parcel`
- `owner_address`  *(implicit)*

```json
{ "key_custody" : "_parcel_encryption_key_", "extra_info" : "_any_of_additional_info_may_comes_here_" }
```

### Request Data

Request `parcel` to purchase with `payment` as offer amount and `extra_info` ( expired_data, etc...). This is not the end of purchase process but the amount of `payment` will be *locked*. The transaction will be stored in `store/request` and waits to be granted by seller. 

- command : `request`
- affected store : `request`
- `buyer_address`  *(implicit)*

```json
{ "target" : "_parcel_id_", "extra_info" : "_any_of_additional_info_may_comes_here_" }
``` 

### Cancel Request

Cancel the request of `parcel` in `store/request`. It deletes the previous `request_data` of `myself_address` in `store/request` and releases the amount of `payment` which was locked. Since 

- command : `cancel`
- affected store : `request`
- `myself_address`  *(implicit)*

```json
{ "target" : "_parcel_id_" }
```

### Grant Data Usage

Grant the request of `parcel` in `store/request` by *data owner*. Specify `grantee` to avoid confusion with other purchasers of the same `parcel`.

- command : `grant`
- affected store : `request`
- `owner_address`  *(implicit)*

```json
{ "target" : "_parcel_id_", "grantee" : "_buyer_address_", "key_custody" : "_parcel_encryption_key_" }
```
### Revoke Data Usage
### Discard Data

Discard the registered data in `store/parcel`. After `discard` is recorded in AMO blockchain, delete `parcel` corresponding (`parcel_id`, `owner_address`, `key_custody`, `extra_info`). 

- command : `discard`
- affected store : `parcel`
- `owner_address`  *(implicit)*

```json
{ "target" : "_parcel_id_" }
```

### Retrieve Data (PDB operation)
### Delete Data (PDB operation)
