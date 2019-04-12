# AMO Testnet
The purpose of the AMO testnet is demonstrating functions of AMO blockchain without spending real-world monetary assets. Beside this key difference, there is intended to be no functional differences between mainnet and testnet.
However, there must be notable differences including:
- AMO coin in a testnet has no real-world monetary value.
- All the data in AMO testnet shall be periodically purged and the blockchain shall be started freshly from a new genesis state.
- Any user can receive *free* AMO coins through minimal registration process.
- There shall be a fixed number of validators and cannot be changed during the lifetime of a testnet. ( *This policy may change in the future.* )

## Test guide
### Block explorer
You can visit [block explorer]() web site to explore the internals of AMO blockchain testnet: list of blocks, list transactions in a block, individual transaction, and various internal data of AMO blockchain state.

### Client connection
In order to participate as an AMO client, you need to download AMO client source code from AMOLabs' github [repository](https://github.com/amolabs/amoabci). For more information about downloading and building AMO client CLI, see [README.md](https://github.com/amolabs/amoabci/blob/master/README.md).

After you get an AMO CLI program(amocli), you can connect any AMO blockchain node as follows:
```bash
amocli --rpc <rpc_node_ip:port> <command> <args>
```
You can ommit `--rpc` option in order to connect to a node on a localhost.

### Validator node
In order to participate as a validator node, you need to download AMO blockchain node source code from AMOLabs' github [repository](https://github.com/amolabs/amoabci). For more information about downloading and building AMO blockchain node, see [README.md](https://github.com/amolabs/amoabci/blob/master/README.md).

- Prepare a data directory
- Generate a validator key pair
- Obtain enough AMO coins
- Obtain a stake with the AMO coins and a validator public key
- Run a _stable_ AMO blockchain node (config)

### Non-validator ndoe
- Run an AMO blockchain node (config)

## Testnet 190415
This testnet is scheduled to launch on 2019-04-15 12:00 KST. This testnet nodes will run software version `v1.0-alpha1`.

### Testnet configuration
- number of validators: 3 (subject to change)

### AMO blockchain nodes
- [genesis file](files/testnet_190415/genesis.json)
- seed validator node
    - node0
        - amo p2p connection: ` id_to_be_announced@139.162.116.176:26656 `
        - pdb p2p connection: ` id_to_be_announced@139.162.116.176:26659 `
        - amo rpc connection: http://139.162.116.176/amo/
        - pdb rpc connection: http://139.162.116.176/pdb/
- non-seed validator nodes
    - node1
        - amo p2p connection: ` id_to_be_announced@ip_to_be_announced:26656 `
        - pdb p2p connection: ` id_to_be_announced@ip_to_be_announced:26659 `
        - amo rpc connection: http://ip_to_be_announced/amo/
        - pdb rpc connection: http://ip_to_be_announced/pdb/
    - node2
        - amo p2p connection: ` id_to_be_announced@ip_to_be_announced:26656 `
        - pdb p2p connection: ` id_to_be_announced@ip_to_be_announced:26659 `
        - amo rpc connection: http://ip_to_be_announced/amo/
        - pdb rpc connection: http://ip_to_be_announced/pdb/
- block explorer
    - http://ip_to_be_announced

### AMO storage nodes
PAUST-DB will be used as a native AMO storage service. Each AMO blockchain node will run its own storage node.

### Additional nodes
In case of software failure in the seed node, the blockchain shall freshly start using the same genesis state.
