# Quick Start Guide for CryptoExchange
[//]: # "This docuemtn is available in [Korean](qs_crex.ko.md) also."

## Introduction
This quick start guide describes how to prepare AMO client program and connect
to an AMO blockchain node. This guide assumes the following:
- you use a golang package in `https://github.com/amolabs/amo-client-go/lib`
- you want to transfer AMO coins
- you want to delegate and retract coins to and from a validator

## Development Environment
First, you need to setup golang development environment:
* [git](https://git-scm.com)
* [make](https://www.gnu.org/software/make/)
  * For Debian or Ubuntu linux, you can install `build-essential` package.
  * For MacOS, you can use `make` from Xcode, or install GNU Make via
    [Homebrew](https://brew.sh).
* [golang](https://golang.org/dl/) v1.11 or later (for Go Modules)
  * In some cases, you need to set `GOPATH` and `GOBIN` environment variable
    manually. Check these variables are set before you proceed.

## Using library
Create a golang program using the following `import` statement:
```go
import (
    "github.com/amolabs/amo-client-go/lib/rpc"
)

func main() {
}
```

### Setup Connection
You need to set a pakcage-global `rpc.RpcRemote` variable to setup connections. Currently, there is a testnet running. So you may connect to one of the testnet RPC nodes:
- `172.104.88.12:26657`
- `139.162.116.176:26657`
- `96.126.125.11:26657`
- `139.162.180.153:26657`

```go
import (
    "github.com/amolabs/amo-client-go/lib/rpc"
)

func main() {
    rpc.RpcRemote = "http://172.104.88.12:26657"
}
```

There are other nodes, but their addresses may change in near future without
notice:
- `172.104.82.146:26657`
- `45.33.24.148:26657`
- `172.105.64.192:26657`

### Transfer coin
You may use `rpc.Transfer()` function to transfer AMO coins from one to another
account.
```go
import (
    "github.com/amolabs/amo-client-go/lib/rpc"
)

func main() {
    rpc.RpcRemote = "http://172.104.88.12:26657"

    // recp_addr: string such as "85B5ED98D4BF06D6A2540087D16C39725B87C65E"
    // transfer_amount, fee_amount: string such as "1000000000000000000" (1 AMO)
    // key_to_sign: see next tutorial
    // last_height: see next tutorial
    rpc.Transfer(recp_addr, transfer_amount, key_to_sign, fee_amount, last_height)
}
```

### Key
Transaction functions such as `rpc.Transfer()` uses `keys.KeyEntry` structure
to handle keys when signing a transaction. You may use this structure by
importing `https://github.com/amolabs/amo-client-go/lib/keys` package.
```go
import (
    "github.com/amolabs/amo-client-go/lib/rpc"
    "github.com/amolabs/amo-client-go/lib/keys"
)

func main() {
    rpc.RpcRemote = "http://172.104.88.12:26657"

    // send_addr: HEX string such as "F5C92D4E4E4DF4E03B9F98BAF6D564D7B8183911"
    // pub_key: public key bytes using NIST P256 curve, uncompressed form
    // priv_key: private key bytes
    key_to_sign = keys.KeyEntry{
        Address: send_addr,
        PubKey: pub_key,
        PrivKey: priv_key,
        Encrypted: false,
    }

    // recp_addr: HEX string such as "85B5ED98D4BF06D6A2540087D16C39725B87C65E"
    // transfer_amount, fee_amount: string such as "1000000000000000000" (1 AMO)
    // last_height: see next tutorial
    rpc.Transfer(recp_addr, transfer_amount, key_to_sign, fee_amount, last_height)
}
```

### Height
AMO blockchain uses block-bound transactions. In order for a transaction to be
valid, it must have a reference to the last block it has seen. This is called a
*last seen height*. If this value is greater than the current height, or too
small compared to the current height, this transaction will be discarded by AMO
blockchain nodes. So you need to query the chain to get the current height.
```go
import (
    "github.com/amolabs/amo-client-go/lib/rpc"
    "github.com/amolabs/amo-client-go/lib/keys"
)

func main() {
    rpc.RpcRemote = "http://172.104.88.12:26657"

    // send_addr: HEX string such as "F5C92D4E4E4DF4E03B9F98BAF6D564D7B8183911"
    // pub_key: public key bytes using NIST P256 curve, uncompressed form
    // priv_key: private key bytes
    key_to_sign = keys.KeyEntry{
        Address: send_addr,
        PubKey: pub_key,
        PrivKey: priv_key,
        Encrypted: false,
    }

    // recp_addr: HEX string such as "85B5ED98D4BF06D6A2540087D16C39725B87C65E"
    // transfer_amount, fee_amount: string such as "1000000000000000000" (1 AMO)
    // last_height: up-to-date height of the chain
    rpc.Transfer(recp_addr, transfer_amount, key_to_sign, fee_amount, last_height)
}
```

## Using CLI
TBA
