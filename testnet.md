# AMO Testnet
The purpose of the AMO testnet is demonstrating functions of AMO blockchain
without spending real-world monetary assets. Beside this key difference, there
is intended to be no functional differences between mainnet and testnet.
However, there must be notable differences including:
- AMO coin in a testnet has no real-world monetary value.
- All the data in AMO testnet shall be periodically purged and the blockchain
  shall be started freshly from a new genesis state.
- Any user can receive *free* AMO coins through minimal registration process.
- There shall be a fixed number of validators and cannot be changed during the
  lifetime of a testnet. ( *This policy may change in the future.* )

## Test guide
### Block explorer
You can visit [block explorer](http://explorer.amolabs.io) web site to explore the internals of AMO
blockchain testnet: list of blocks, list transactions in a block, individual
transaction, and various internal data of AMO blockchain state.

### Client connection
In order to participate as an AMO client, you need to download AMO client
source code from AMOLabs' github
[repository](https://github.com/amolabs/amoabci). For more information about
downloading and building AMO client CLI, see
[README.md](https://github.com/amolabs/amoabci/blob/master/README.md).

After you get an AMO CLI program(amocli), you can connect any AMO blockchain
node as follows:
```bash
amocli --rpc <rpc_node_ip:port> <command> <args>
```
You can ommit `--rpc` option in order to connect to a node on a localhost.

### Validator node
In order to participate as a validator node, you need to download AMO
blockchain node source code from AMOLabs' github
[repository](https://github.com/amolabs/amoabci). For more information about
downloading and building AMO blockchain node, see
[README.md](https://github.com/amolabs/amoabci/blob/master/README.md).

- Prepare a data directory
- Generate a validator key pair
- Obtain enough AMO coins
- Obtain a stake with the AMO coins and a validator public key
- Run a _stable_ AMO blockchain node (config)

### Non-validator ndoe
- Run an AMO blockchain node (config)

## Testnet 190423
This testnet is scheduled to launch on 2019-04-23 12:00 KST. This testnet nodes
will run software version `v1.0-alpha4`.

### Testnet configuration
- number of validators: 3 (subject to change)
- [genesis file](https://github.com/amolabs/testnet/blob/master/testnet_190423/genesis.json)

### AMO blockchain nodes
- seeder (validator 0)
	- p2p: ` a944a1fa8259e19a9bac2c2b41d050f04ce50e51@139.162.116.176:26656 `
	- rpc: http://139.162.116.176:26657
- validator 1
	- p2p: ` 85f77642d233fdff6cb16a73e3484bc7b2bf8e37@96.126.125.11:26656 `
	- rpc: http://96.126.125.11:26657
- validator 2
	- p2p: ` aaa9ac5fcf4aae14576c75e3d6c0b875f1fce377@139.162.180.153:26656 `
	- rpc: http://139.162.180.153:26657

### AMO blockchain explorer
- http://explorer.amolabs.io

### AMO storage nodes
Not included in this testnet.

### Additional nodes
In case of software failure in the seed node, the blockchain shall freshly
start using the same genesis state.

## Testnet 190415
This testnet is scheduled to launch on 2019-04-15 12:00 KST. This testnet nodes
will run software version `v1.0-alpha2`. (updated to `v1.0-alpha3`)

### Testnet configuration
- number of validators: 3 (subject to change)
- [genesis file](https://github.com/amolabs/testnet/blob/master/testnet_190415/genesis.json)

### AMO blockchain nodes
- seeder (validator 0)
	- p2p: ` 3e4f83cb37f5a8be3efdc7eaa3e40329b5370fc8@139.162.116.176:26656 `
	- rpc: http://139.162.116.176:26657
- validator 1
	- p2p: ` fa8c77684227c95f0e7f0d91196bfb7b3f2fa21c@96.126.125.11:26656 `
	- rpc: http://96.126.125.11:26657
- validator 2
	- p2p: ` 8b3a95b3432103c23c65e76c4d91970201ff29b0@139.162.180.153:26656 `
	- rpc: http://139.162.180.153:26657

### AMO blockchain explorer
- http://explorer.amolabs.io

### AMO storage nodes
Not included in this testnet.

### Additional nodes
In case of software failure in the seed node, the blockchain shall freshly
start using the same genesis state.