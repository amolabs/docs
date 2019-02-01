# AMO blockchain protocol specification

## Data Types
### Transaction
```go
type Transaction struct {
    Sender      string  `json:"sender"`
    Recipient   string  `json:"recipient"`
    Amount      uint128  `json:"amount"`
}
```

## Operations
### AMO coin transfer
1. AMO user creates a signed transaction and sends to one of AMO nodes.
1. AMO node processes the transaction.
