# AMO Testnet
The purpose of the AMO testnet is demonstrating functions of AMO blockchain without spending real-world monetary assets. Beside this key difference, there is intended to be no functional differences between mainnet and testnet.
However, there must be notable differences including:
- AMO coin in a testnet has no real-world monetary value.
- All the data in AMO testnet shall be periodically purged and the blockchain shall be started freshly from a new genesis state.
- Any user can receive *free* AMO coins through minimal registration process.
- There shall be a fixed number of validators and cannot be changed during the lifetime of a testnet. ( *This policy may change in the future.* )

## Test guide
### Client connection
### Block explorer

## Testnet 190415
This testnet is scheduled to launch on 2019-04-15 12:00 KST. This testnet nodes will run software version `pre-release1`.

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
