# AMO Testnet
This document is available in [Korean](testnet.ko.md) also.

The purpose of the AMO testnet is demonstrating functions of AMO blockchain
without spending real-world monetary assets. There is intended to be no other
functional differences between mainnet and testnet than this key difference.
Notable differences are:
- AMO coin in a testnet has no real-world monetary value, since all the data in
  AMO testnet shall be periodically purged and the blockchain shall be started
  freshly from a new genesis state.
- Any user can receive small amount of *free* AMO coins from the *faucet
  server*.

## Test Guide
### AMO blockchain explorer
You can visit [block explorer](http://explorer.amolabs.io) web site to explore
the internals of the AMO blockchain testnet: list of blocks, list transactions
in a block, individual transaction, and internal state of AMO blockchain.

Also, there is a interactive [demo page](http://explorer.amolabs.io/demo) in
the explorer site. You can freely test the functionality of the testnet.

### Client program
You may want to interact with the AMO blockchain network than using controlled
environment such as AMO blockchain explorer's demo page. Any software program
or hardware device which conforms to the [AMO client RPC
specification](https://github.com/amolabs/docs/blob/master/rpc.md) is called an
AMO client. AMO blockchain explorer is a special kind of an AMO client. AMO
Labs provide a general-purpose CLI program,
[`amo-client-go`](https://github.com/amolabs/amo-client-go), and you can
interact with the AMO blockchain, including testnet, on your own way.

In order to participate as an AMO client using `amo-client-go`, you need to
download AMO client source code from AMOLabs' github
[repository](https://github.com/amolabs/amo-client-go). For more information
about downloading and building AMO client CLI, see
[README.md](https://github.com/amolabs/amo-client-go/blob/master/README.md).

After you get an AMO CLI program(`amocli`), you can connect any AMO blockchain
node as follows:
```bash
amocli --rpc <rpc_node_ip:port> <command> <args>
```
You can omit `--rpc` option in order to connect to a node on a localhost. For
more information about how to use `amocli`, see
[README.md](https://github.com/amolabs/amo-client-go/blob/master/README.md).

### Launching validator node
In order to participate as a validator, you need to prepare and run a validator
node. You may see a separate [quick start
guide](https://github.com/amolabs/docs/blob/master/qs_val.md).

### Launching non-validator node
In order to participate as a non-validator, you may see
[README.md](https://github.com/amolabs/testnet/blob/master/README.md).

## Network information
### AMO blockchain nodes
A testnet will launch with the initial 6 validator nodes and a non-validator
seed node (this may change in the future):
- seed: 172.104.88.12
- validator 1: 139.162.116.176 (genesis validator)
- validator 2: 96.126.125.11
- validator 3: 139.162.180.153
- validator 4: 172.104.82.146
- validator 5: 45.33.24.148
- validator 6: 172.105.64.192

Each node will open 2 ports: port 26656 for P2P connection, port 26657 for RPC
connection. It is recommended to connect to the seed node when you launch your
own blockchain node.

### AMO storage service
Currently, there is a running AMO default storage service.
- http://139.162.111.178:5000

### P2P connection
Every AMO blockchain node has a *node id*, and you need to specify this *node
id* when connecting to a node via P2P connection. For the testnet, a node id
for each initial nodes may change when starting a new chain. You can figure out
the *node id* of a node via RPC query as following:
```bash
curl http://<ip>:26657/status
```
Note `result.node_info.id`. Use a node address as `<node_id>@<ip>:26656` in the
`config.toml` file:
```toml
[p2p]
psersistent_peers = "node address"
```

### RPC connection
Every AMO client needs an RPC endpoint to interact with the AMO blockchain. Use
a rpc server address as `http://<ip>:26657`. For `amocli`, you need to specify
only `<ip>:26657` without `http://` prefix for `--rpc` flag.

### AMO storage connection
For `amocli`, you need to specify only `<ip>:5000` without `http://` prefix for
`--sto` flag.

### Genesis file
Any AMO blockchain node provide a genesis file via an RPC connection. You can
obtain `genesis.json` as follows:
```bash
curl http://<ip>:26657/genesis
```
Save the output as a file `genesis.json`.

