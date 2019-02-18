# AMO client RPC specification
You can check rpc endpoints via http://localhost:26657

[https://github.com/tendermint/tendermint/wiki/RPC](https://github.com/tendermint/tendermint/wiki/RPC)

Arguments which expect strings or byte arrays may be passed as quoted strings, like "abc" or as 0x-prefixed strings, like 0x616263.

JSONRPC doesn't support number expression. e.g. 300 → "300"

Data in tags are encoded by base64.

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
