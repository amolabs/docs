# AMO blockchain protocol specification

## Data Structure
### Transaction
payload to Tendermint `DeliverTx` and `CheckTx` methods

ex)
```go
type Transaction struct {
    Sender      string  `json:"sender"`
    Recipient   string  `json:"recipient"`
    Amount      uint128 `json:"amount"`
}
```

### KV Stores
* key-value store of (address, balance)
* key-value store of (parcel id, owner address)

## Operations
### AMO coin transfer
1. AMO user creates a signed transaction and sends to one of AMO nodes.
1. AMO node processes the transaction.

### Register data parcel

### Grant permission
* "grant `read` permission on a data parcel `A` to an AMO user `B`"
* "revoke `read` permission on a data parcel `A` from an AMO user `B`"
