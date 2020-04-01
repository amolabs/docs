# Quick Start Guide for Validator Node
This document is available in [Korean](qs_val.ko.md) also.

## Introduction
This quick start guide describes how to prepare and launch validator node in
proper way. This guide will use the method described in [amoabci
README](https://github.com/amolabs/amoabci/README.md).

## Monetary asset
Without stake, your blockchain node could not participate in consensus process.
So, it is essential to acquire some AMO coins to run a validator node. For a
testnet, you can contact AMO Labs via various channels to acquire enough AMO
coins for a validator node. But, for the mainnet, it is up to you to acquire
necessary AMO coins. This guide will not describe how to acquire AMO coins for
the mainnet. For both of testnet and mainnet, you need to acquire AMO coins
before you send a `stake` transaction (the last step in this guide).

## Prepare server environment
### Server machine
In order to run a validator node, you need a physical server with a stable
internet connection or a virtual machine on a cloud service(Amazon AWS, Google
cloud, Microsoft Azure or similar services). In this guide, we assume typical
Ubuntu Linux or MacOS is installed on the host machine.

### Install necessary packages
Connect to a terminal of the server machine and install `git`, `curl` and `jq`
as root:
```bash
sudo apt install git curl jq
```

### Install `amod` daemon
See [Getting Started](https://github.com/amolabs/amoabci#getting-started)
section in amoabci document to install `amod` from either pre-built binary or
source.

### Install setup script 
To use the script which helps to setup a validator node, connect to a terminal
of the server machine and execute the following commands:
```bash
cd $HOME
git clone https://github.com/amolabs/testnet
cd testnet
```
This script prepares data root directory, generates `config.toml`, registers
`amod.service` in `systemd` and copies necessary files to data root directory.
If you already have `node_key.json` and `priv_validator_key.json`, place them
below `testnet` directory. They are going to be copied into data root directory
automatically. Otherwise, the script would generate them for you.

## Launch on Mainnet/Testnet 
### Node info

| | `node_id` | `node_ip_addr` | `node_p2p_port` | `node_rpc_port` |
|-|-|-|-|-|
| mainnet | `f5123e0f663fe8e0662b82de8f6a1d843a9d4fbd` | `172.104.88.12` | `26656` | `26657` |
| testnet | `a944a1fa8259e19a9bac2c2b41d050f04ce50e51` | `172.105.213.114` | `26656` | `26657` |

### Get `genesis.json`
To download `genesis.json` file, execute the following commands:
```bash
cd testnet
curl <node_ip_addr>:<node_rpc_port>/genesis | jq '.result.genesis' > genesis.json
```
Specify proper `node_ip_addr` and `node_rpc_port` depending on which network
you would like to connect to, by referring to [Node info](#node-info) section.

For example, to download `genesis.json` file for **mainnet**, execute the
following commands:
```bash
cd testnet
curl 172.104.88.12:26657/genesis | jq '.result.genesis' > genesis.json
```

### Setup
First, select data directory location, figure out current server's external ip
address and seed node's information. Then, execute the following command as
root:
```bash
sudo ./setup.sh -e <ext_ip_addr> <data_root> <moniker> <node_id>@<node_ip_addr>:<node_p2p_port>
```
Specify proper `ext_ip_addr`, `data_root`, `moniker`(node name) and `node_*`
information, depending on which network you would like to connect to by
referring to [Node info](#node-info) section.

For example, if current server's external ip address is `123.456.789.0`(not
avaiable ip address), data root directory is `/mynode`, node name is
`mynodename` and you'd like to connect to mainnet, then execute the following
commands:
```bash
sudo ./setup.sh -e 123.456.789.0 /mynode mynodename f5123e0f663fe8e0662b82de8f6a1d843a9d4fbd@172.104.88.12:26656
```

### Backup key
Backup `/mynode/amo/config/priv_validator_key.json` file to a
safe location.

### Run node 
To run validator node, execute the following commands as root:
```bash
sudo systemctl start amod
```

### Check node status
To check validator node status, execute the following commands as root:
```bash
sudo systemctl status amod
```

### Stop node
To stop validator node, execute the following commands as root:
```bash
sudo systemctl stop amod
```

### Gather information
Execute the following command and write down the validator address and public
key somewhere convenient.
```bash
curl localhost:26657/status
```
<p align="center"><img src="images/node_status.png"/></p>

Check `"sync_info"` in the output to see if our node is syncing properly with
the other nodes in the blockchain network. If `"catching_up"` is `true`, then
it still behind the tip of the blockchain. So, wait until the syncing process
is complete.

## Stake coins
**NOTE:** For the mainnet, you must use more controlled steps as the
followings, but for the testnet you may visit <a
href="http://explorer.amolabs.io/wallet">AMO blockchain explorer</a> and follow
the guide there.

You need `amocli`(AMO client) to stake coins. See
[Installation](https://github.com/amolabs/amo-client-go#installation) section
in amo-client-go document to install `amocli` in proper way.

We assume you possess the account key (`myval` for amocli username,
`D2CC7F160874AF06027A09DC0E8DC67E85E6D704` for address) and necessary AMO
coins. Now, you need to send a `stake` transaction to the blockchain. For the
`stake` transaction you need a validator public key. This process is equivalent
to announce to the world that you take the control over a validator node which
is running with the validator public key. To find out the validator public key
of a node which is launched, execute the following command:
```bash
amod --home <data_root>/amo tendermint show_validator
```
Specify proper `data_root`.

For example, if data root directory is `/mynode`, execute the following
command: 
```bash
amod --home /mynode/amo tendermint show_validator
```

We assume the validator public key is as follows:
```json
{
  "address": "9A8F09C644941B5A526B19641A3D7C8805E312B9",
  "pub_key": {
    "type": "tendermint/PubKeyEd25519",
    "value": "+4jvv6ZCP+TxC0CwBQRr31ieZzj7KMZL3iwribL3czM="
  }
}
```

Now, you have a validator public key to announce. To stake 1000000 AMO along
with the public key, execute the following command:
```bash
amocli tx --user myval stake '+4jvv6ZCP+TxC0CwBQRr31ieZzj7KMZL3iwribL3czM=' 1000000000000000000000000 
```

To check if 1000000 AMO is properly staked, execute the following command:
```bash
amocli query stake 'D2CC7F160874AF06027A09DC0E8DC67E85E6D704'
```

You can inspect the list of validators and their voting power by executing the
following command:
```bash
curl localhost:26657/validators
```
