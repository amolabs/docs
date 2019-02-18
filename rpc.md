# AMO client RPC specification
You can check rpc endpoints via http://localhost:26657

[https://github.com/tendermint/tendermint/wiki/RPC](https://github.com/tendermint/tendermint/wiki/RPC)

Arguments which expect strings or byte arrays may be passed as quoted strings, like "abc" or as 0x-prefixed strings, like 0x616263.

JSONRPC doesn't support number expression. e.g. 300 → "300"

Data in tags are encoded by base64.

## Blockchain State

### getBalance

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
    curl -X POST --data '{"jsonrpc":"2.0","id":"dontcare","method":"getBalance","params":{"address":"aH2JdDUP5NoFmeEQEqDREZnkmCh8V7co7y"}}' localhost:26657
    
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

### uploadData

1. Parameters

```JSON
        params : {
          "dataID": "F7CB0A457DC12C4DBE51B0F158E22CBD52B10F0D25AE3BBB7E668CBE261D4F5F",
          "price": "3000",
          "key": "",
          "info": "Total mileage of all taxis in Newyork by 2018."
        }
```

2. Returns

`uint8` http response code

3. Example

```Shell
    // Request
    curl -X POST --data '{"jsonrpc":"2.0","id":"dontcare","method":"uploadData","params":{"dataID":"F7CB0A457DC12C4DBE51B0F158E22CBD52B10F0D25AE3BBB7E668CBE261D4F5F","info":"Total mileage of all taxis in Newyork by 2018."}}' localhost:26657
    
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

### showDataInfoById

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
    curl -X POST --data '{"jsonrpc":"2.0","id":"dontcare","method":"getDataInfoById","params":{"dataID":"F7CB0A457DC12C4DBE51B0F158E22CBD52B10F0D25AE3BBB7E668CBE261D4F5F"}}' localhost:26657
    
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
              "value": [
                "address": "aH2JdDUP5NoFmeEQEqDREZnkmCh8V7co7y",  # owner address
                "price": "3000",
                "key": "",
                "info": "Total mileage of all taxis in Newyork by 2018."
            }
          ]
        },
        "hash": "905998809DCBB5EA078A1A488DA448554A5B8F9E2CC97C68D295BCA852E65A47",
        "height": "3"
      }
    }
```

### requestDataPurchaseById

1. Parameters

```JSON
        params : {
          "address": "aH2JdDUP5NoFmeEQEqDREZnkmCh8V7co7y"  
          "dataID": "F7CB0A457DC12C4DBE51B0F158E22CBD52B10F0D25AE3BBB7E668CBE261D4F5F"
        }
```

2. Returns

`state`: request grant state.

3. Example

```Shell
    // Request
    curl -X POST --data '{"jsonrpc":"2.0","id":"dontcare","method":"requestDataPurchaseById","params":{"address": "aH2JdDUP5NoFmeEQEqDREZnkmCh8V7co7y","dataID":"F7CB0A457DC12C4DBE51B0F158E22CBD52B10F0D25AE3BBB7E668CBE261D4F5F"}}' localhost:26657
    
    // Result
    {
      "jsonrpc": "2.0",
      "id": "0",
      "result": {
        "check_tx": {},
        "deliver_tx": {
          "tags": [
            {
              "key": [
                "address": "aH2JdDUP5NoFmeEQEqDREZnkmCh8V7co7y",
                "dataID": "F7CB0A457DC12C4DBE51B0F158E22CBD52B10F0D25AE3BBB7E668CBE261D4F5F=", # aaaaa
              ],
              "value": [
                "state": "granted"
              ]
            }
          ]
        },
        "hash": "905998809DCBB5EA078A1A488DA448554A5B8F9E2CC97C68D295BCA852E65A47",
        "height": "3"
      }
    }
```

`To-do`
### revertDataPurchaseById
### grantDataPurchaseById
### grantDataPurchaseByAddress
### showDataUsageById


## Blockchain Operations
`To-do`

### transfer

1. Message Type

    transfer

2. Parameters

```JSON
        params : {
        	from: "aaaaa",
        	to: "bbbbb",
        	amount: 300
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
        	from: "aaaaa",
        	file_hash: "b94d27b9934d3e08a52e52d7da7dabfac484efe37a5380ee9088f7ace2efcde9"
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
