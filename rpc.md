# AMO client RPC specification
Tendermint provides various RPC methods to interact with Tendermint base and ABCI app through it. See https://tendermint.com/rpc/ for details. Since AMO is a blockchain application (ABCI app) built upon Tendermint, it provides all the RPC methods which are provided by Tendermint itself. Among these methods, `broadcast_tx_*` and `abci_query` are used to interact with AMO ABCI app: sending transactions and querying app state.

Tendermint RPC operates over three possible communication channels: URI/HTTP, JSONRPC/HTTP, JSONRPC/websocket. This document assumes JSONRPC/HTTP, but descriptions in this document can be applied to other communication channels also.

(Since we assume JSONRPC/HTTP)
RPC request data will be transmitted as HTTP POST request body with content-type as `application/json`. The default RPC endpoint is http://localhost:26657, but this may change according to the server configuration. Request body is in JSON format and consists of RPC preamble(e.g. `"jsonrpc":"2.0","id":""`), rpc method, and rpc parameters. For example:
```json
{"jsonrpc":"2.0","id":"","method":"abci_query","params":{"path":"AMO specific path","data":"AMO specific data","height":"0","prove":"false"}}
{"jsonrpc":"2.0","id":"","method":"broadcast_tx_sync","params":{"tx":"AMO specific transaction"}}
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

`data` specifies the key to find a match in the data store. `data` must be a `base64` conversion of a proper JSON object.

In full RPC request body:
```json
{
    "jsonrpc":"2.0",
    "id":"",
    "method":"abci_query",
    "params": {
        "path": "/balance",
        "data": "base64 conversion of \"12FF....\""
    }
}
```

### Balance query
When querying an account's _balance_, `data` is a `base64` conversion of the double-quoted _address_ of the account.

### Parcel query
When querying a _parcel_, `data` is a `base64` conversion of the double-quoted _data parcel ID_.

### Request query
When querying a _request_ for specific _parcel_, `data` is a `base64` conversion of a JSON representation of (`buyer address`, `data parcel ID`), where this JSON representation is as the following example:
```json
{ "buyer" : "_address_", "parcel" : "_parcel_ID_" }
```

### Usage query
When querying a _usage_ for specific _parcel_, `data` is a `base64` conversion of a JSON representation of (`buyer address`, `data parcel ID`), where this JSON representqtion is as the following example:
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
    "id":"",
    "method":"broadcast_tx_commit",
    "params": {
        "tx": "0x_the_hex_string_byte_code_for_abci_code"
    }
}
```


## Rewrite or delete followings

## Blockchain State

### get_balance

1. Parameters

```JSON
        params : {
          "address": "aH2JdDUP5NoFmeEQEqDREZnkmCh8V7co7y"
        }
```

2. Returns

`uint256` The balance of account of given address.

3. Example

```Shell
    // Request
    curl -X POST --data '{"jsonrpc":"2.0","id":"dontcare","method":"get_balance","params":{"address":"aH2JdDUP5NoFmeEQEqDREZnkmCh8V7co7y"}}' localhost:26657

    // Result
    {
      "jsonrpc": "2.0",
      "id": "0",
      "result": {
        "check_tx": {},
        "deliver_tx": {
          "tags": [
            {
              "key": "aH2JdDUP5NoFmeEQEqDREZnkmCh8V7co7y" # given address
              "value": "3000", # 3000
            }
          ]
        }
      }
    }
```

### upload_data

1. Parameters

```JSON
        params: {
          "dataID": "F7CB0A457DC12C4DBE51B0F158E22CBD52B10F0D25AE3BBB7E668CBE261D4F5F",
          "ownerID": "aH2JdDUP5NoFmeEQEqDREZnkmCh8V7co7y",
          "key": "",
          "info":{
            "description": "Total mileage of all taxis in Newyork by 2018.",
            "price":"3000",
            "expCondition":{
              "due_date": "2020-12-31 23:59",
              "due_count": "10",
              "current_count": "0"
            }
          }
        }
```

2. Returns

`uint8` http response code

3. Example

```Shell
    // Request
    curl -X POST --data '{"jsonrpc":"2.0","id":"dontcare","method":"upload_data","params":{"dataID":"F7CB0A457DC12C4DBE51B0F158E22CBD52B10F0D25AE3BBB7E668CBE261D4F5F","ownerID": "aH2JdDUP5NoFmeEQEqDREZnkmCh8V7co7y",          "key": "","info": ["description":"Total mileage of all taxis in Newyork by 2018.","price":"3000","expCondition":["due_date":"2020-12-31 23:59","due_count":"10","current_count":"0"]]}}}' localhost:26657

    // Result
    {
      "jsonrpc": "2.0",
      "id": "0",
      "result": {
        "check_tx": {},
        "deliver_tx": {
          "tags": [
            {
              "key": "F7CB0A457DC12C4DBE51B0F158E22CBD52B10F0D25AE3BBB7E668CBE261D4F5F=", # aaaaa
              "value": "200" # http response code
            }
          ]
        },
        "hash": "905998809DCBB5EA078A1A488DA448554A5B8F9E2CC97C68D295BCA852E65A47",
        "height": "3"
      }
    }
```

### show_parcel_info_by_id

1. Parameters

```JSON
        params : {
          "dataID": "F7CB0A457DC12C4DBE51B0F158E22CBD52B10F0D25AE3BBB7E668CBE261D4F5F"
        }
```

2. Returns

`address`   owner address.
`key`       key custody needed.
`string`    extra info.

3. Example

```Shell
    // Request
    curl -X POST --data '{"jsonrpc":"2.0","id":"dontcare","method":"show_parcel_info_by_id","params":{"dataID":"F7CB0A457DC12C4DBE51B0F158E22CBD52B10F0D25AE3BBB7E668CBE261D4F5F"}}' localhost:26657

    // Result
    {
      "jsonrpc": "2.0",
      "id": "0",
      "result": {
        "check_tx": {},
        "deliver_tx": {
          "tags": [
            {
              "key": "F7CB0A457DC12C4DBE51B0F158E22CBD52B10F0D25AE3BBB7E668CBE261D4F5F=", # aaaaa
              "value": {
                "ownerID": "aH2JdDUP5NoFmeEQEqDREZnkmCh8V7co7y",
                "key": "",
                "info": {
                  "description":"Total mileage of all taxis in Newyork by 2018.",
                  "price":"3000",
                  "expCondition": {
                    "due_date":"2020-12-31 23:59",
                    "due_count":"10",
                    "current_count":"2"
                  }
                }
              }
            }
          ]
        },
        "hash": "905998809DCBB5EA078A1A488DA448554A5B8F9E2CC97C68D295BCA852E65A47",
        "height": "3"
      }
    }
```

### request_parcel_purchase_by_id

1. Parameters

```JSON
        params : {
          "address": "aZJ5UP5NdDeEQEq54Eoh8V7coFmZnkmC7z",
          "dataID": "F7CB0A457DC12C4DBE51B0F158E22CBD52B10F0D25AE3BBB7E668CBE261D4F5F"
        }
```

2. Returns

`state`: request grant state. holding, granted ...

3. Example

```Shell
    // Request
    curl -X POST --data '{"jsonrpc":"2.0","id":"dontcare","method":"request_parcel_purchase_by_id","params":{"address": "aZJ5UP5NdDeEQEq54Eoh8V7coFmZnkmC7z","dataID":"F7CB0A457DC12C4DBE51B0F158E22CBD52B10F0D25AE3BBB7E668CBE261D4F5F"}}' localhost:26657

    // Result
    {
      "jsonrpc": "2.0",
      "id": "0",
      "result": {
        "check_tx": {},
        "deliver_tx": {
          "tags": [
            {
              "key": {
                "address": "aH2JdDUP5NoFmeEQEqDREZnkmCh8V7co7y",
                "dataID": "F7CB0A457DC12C4DBE51B0F158E22CBD52B10F0D25AE3BBB7E668CBE261D4F5F=", # aaaaa
              },
              "value": "holding"
            }
          ]
        },
        "hash": "905998809DCBB5EA078A1A488DA448554A5B8F9E2CC97C68D295BCA852E65A47",
        "height": "3"
      }
    }
```

### discard_parcel_by_id

Verification key is needed which can prove that the user who trying discard is owner.

1. Parameters

```JSON
        params : {
          "address": "aZJ5UP5NdDeEQEq54Eoh8V7coFmZnkmC7z",
          "dataID": "F7CB0A457DC12C4DBE51B0F158E22CBD52B10F0D25AE3BBB7E668CBE261D4F5F",
          "key": "52B10F0D25AE3BBB7E668CBE261D4F5F"  
        }
```

2. Returns

`uint8` http response code

3. Example

```Shell
    // Request
    curl -X POST --data '{"jsonrpc":"2.0","id":"dontcare","method":"discard_parcel_by_id","params":{"address": "aH2JdDUP5NoFmeEQEqDREZnkmCh8V7co7y","dataID":"F7CB0A457DC12C4DBE51B0F158E22CBD52B10F0D25AE3BBB7E668CBE261D4F5F","key":"52B10F0D25AE3BBB7E668CBE261D4F5F"}}' localhost:26657

    // Result
    {
      "jsonrpc": "2.0",
      "id": "0",
      "result": {
        "check_tx": {},
        "deliver_tx": {
          "tags": [
            {
              "key": {
                "address": "aZJ5UP5NdDeEQEq54Eoh8V7coFmZnkmC7z",
                "dataID": "F7CB0A457DC12C4DBE51B0F158E22CBD52B10F0D25AE3BBB7E668CBE261D4F5F=", # aaaaa
              },
              "value": "200" # http response code
            }
          ]
        },
        "hash": "905998809DCBB5EA078A1A488DA448554A5B8F9E2CC97C68D295BCA852E65A47",
        "height": "3"
      }
    }
```

### show_usage

1. Parameters

```JSON
        params : {
          "address": "aZJ5UP5NdDeEQEq54Eoh8V7coFmZnkmC7z",
          "dataID": "F7CB0A457DC12C4DBE51B0F158E22CBD52B10F0D25AE3BBB7E668CBE261D4F5F"
        }
```

2. Returns

`address`   owner address.
`key`       key custody needed.
`string`    extra info.

3. Example

```Shell
    // Request
    curl -X POST --data '{"jsonrpc":"2.0","id":"dontcare","method":"show_usage","params":{"address": "aZJ5UP5NdDeEQEq54Eoh8V7coFmZnkmC7z","dataID":"F7CB0A457DC12C4DBE51B0F158E22CBD52B10F0D25AE3BBB7E668CBE261D4F5F"}}' localhost:26657

    // Result
    {
      "jsonrpc": "2.0",
      "id": "0",
      "result": {
        "check_tx": {},
        "deliver_tx": {
          "tags": [
            {
              "key": {
                "address": "aZJ5UP5NdDeEQEq54Eoh8V7coFmZnkmC7z",
                "dataID": "F7CB0A457DC12C4DBE51B0F158E22CBD52B10F0D25AE3BBB7E668CBE261D4F5F=", # aaaaa
              },
              "value": {
                "ownerID": "aH2JdDUP5NoFmeEQEqDREZnkmCh8V7co7y",
                "key": "",
                "info": {
                  "description":"Total mileage of all taxis in Newyork by 2018.",
                  "price":"3000",
                  "expCondition": {
                    "due_date":"2020-12-31 23:59",
                    "due_count":"10",
                    "current_count":"2"
                  }
                }
              }
            }
          ]
        },
        "hash": "905998809DCBB5EA078A1A488DA448554A5B8F9E2CC97C68D295BCA852E65A47",
        "height": "3"
      }
    }
```

## Blockchain Operations
`To-do`

### transfer

1. Message Type

    transfer

2. Parameters

```JSON
        params : {
          "from": "aaaaa",
          "to": "bbbbb",
          "amount": "300"
        }
```

3. Returns
4. Example

```Shell
    // Request
    curl -X POST --data '{"jsonrpc":"2.0","id":"dontcare","method":"transfer","params":{"from":"aaaaa","to":"bbbbb","amount":"500"}}' localhost:26657

    // Result
    {
      "jsonrpc": "2.0",
      "id": "0",
      "result": {
        "check_tx": {},
        "deliver_tx": {
          "tags": [
            {
              "key": "YWFhYWE=", # aaaaa
              "value": "MjUwMA==" # 2500
            },
            {
              "key": "YmJiYmI=", # bbbbb
              "value": "NTAw" # 500
            }
          ]
        },
        "hash": "905998809DCBB5EA078A1A488DA448554A5B8F9E2CC97C68D295BCA852E65A47",
        "height": "3"
      }
    }
```

### purchase

1. Message Type

    purchase

2. Parameters

```JSON
        params : {
          "from": "aaaaa",
          "file_hash": "b94d27b9934d3e08a52e52d7da7dabfac484efe37a5380ee9088f7ace2efcde9"
        }
```

3. Returns
4. Example

```Shell
    // Request
    curl -X POST --data '{"jsonrpc":"2.0","id":"dontcare","method":"purchase","params":{"from":"aaaaa","file_hash":"b94d27b9934d3e08a52e52d7da7dabfac484efe37a5380ee9088f7ace2efcde9"}}' localhost:26657

    // Result
    {
      "jsonrpc": "2.0",
      "id": "dontcare",
      "result": {
        "check_tx": {},
        "deliver_tx": {
          "tags": [
            {
              "key": "Yjk0ZDI3Yjk5MzRkM2UwOGE1MmU1MmQ3ZGE3ZGFiZmFjNDg0ZWZlMzdhNTM4MGVlOTA4OGY3YWNlMmVmY2RlOQ==", # hash of data
              "value": "eyJmaWxlX2hhc2giOiJiOTRkMjdiOTkzNGQzZTA4YTUyZTUyZDdkYTdkYWJmYWM0ODRlZmUzN2E1MzgwZWU5MDg4ZjdhY2UyZWZjZGU5IiwicHJpY2UiOjEwMCwib3duZXIiOiJhYWFhYSJ9" # metadata
            },
            {
              "key": "YmJiYmI=", # bbbbb
              "value": "NDAw" # 400
            }
          ]
        },
        "hash": "282DB15B6C934E821FBBE5AA1E76C9AFBC1EFA8E84442C9A412E0BAC516145EE",
        "height": "5"
      }
    }
```
