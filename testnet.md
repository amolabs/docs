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

## Testnet 190412
This testnet is scheduled to launch on 2019-04-12 18:00 KST. This testnet nodes will run software version `pre-release1`.

### AMO blockchain nodes
- [genesis file](files/testnet_190412/genesis.json)
- seed validator node
    - node0
        - p2p connection: ` id_to_be_announced@139.162.116.176 `
        - rpc connection: https://139.162.116.176/
- non-seed validator nodes
    - node1
        - p2p connection: ` id_to_be_announced@ip_to_be_announced `
        - rpc connection: TBA
    - node2
        - p2p connection:` id_to_be_announced@ip_to_be_announced `
        - rpc connection: TBA
- block explorer
    - https://ip_to_be_announced

### Testnet configuration
- number of validators: 3 (subject to change)

### AMO storage nodes
TBA

### Additional nodes
In case of software failure in the seed node, the blockchain shall freshly start using the same genesis state.
