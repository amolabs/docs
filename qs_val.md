# Quick Start Guide for Validator and Delegator
This document is available in [Korean](qs_val.ko.md) also.

## Introduction
This quick start guide describes how to prepare and launch a validator node in
proper way, and also how to stake and delegate on a validator. This guide will
use the method described in [amoabci
README](https://github.com/amolabs/amoabci/README.md).

## Monetary asset
Without stake, your blockchain node could not participate in consensus process.
So, it is essential to acquire some AMO coins to run a validator node. For a
testnet, you can contact AMO Labs via various channels to acquire enough AMO
coins for a validator node. But, for the mainnet, it is up to you to acquire
necessary AMO coins. This guide will not describe how to acquire AMO coins for
the mainnet. For both of testnet and mainnet, you need to acquire AMO coins
before you send a `stake` transaction.

## Prerequisite

### Server machine
In order to run a validator node, you need a physical server with a stable
internet connection or a virtual machine on a cloud service(Amazon AWS, Google
cloud, Microsoft Azure or similar services). In this guide, we assume typical
Ubuntu Linux or macOS is installed on the host machine.

### Install necessary packages
Connect to a terminal of the server machine and install `git`, `curl` and `jq`:
```bash
sudo apt install git curl jq
```

### Prepare data directory
There needs a directory `data_root` in which all of AMO related data are stored
on the server machine. To make the directory, execute the following commands:
```bash
sudo mkdir -p <data_root>/amo/config
sudo mkdir -p <data_root>/amo/data
```

As we assume the `data_root` is `/mynode`, execute the following commands:  
```bash
sudo mkdir -p /mynode/amo/config
sudo mkdir -p /mynode/amo/data
```

### Download setup script
To use the script which helps to setup a validator node, connect to a terminal
of the server machine and execute the following commands:
```bash
cd $HOME
git clone https://github.com/amolabs/testnet
```

This script prepares data root directory, generates `config.toml`, registers
`amod.service` in `systemd` and copies necessary files to data root directory.
If you already have `genesis.json`, `node_key.json` and
`priv_validator_key.json`, place them below `testnet` directory. They are going
to be copied into data root directory automatically. Otherwise, the script
would generate them for you.

### Get `genesis.json`
To download `genesis.json` file, execute the following commands:
```bash
cd testnet
curl <node_ip_addr>:<node_rpc_port>/genesis | jq '.result.genesis' > genesis.json
```

Specify proper `data_root`, `node_ip_addr` and `node_rpc_port` depending on the
network you connect to, by referring to [Chain info](#chain-info) section.

For example, to download `genesis.json` file for **mainnet**, execute the
following commands:
```bash
cd testnet
curl 172.104.88.12:26657/genesis | jq '.result.genesis' > genesis.json
```

### Chain Info
| `chain` | `node_id` | `node_ip_addr` | `node_p2p_port` | `node_rpc_port` |
|-|-|-|-|-|
| mainnet | `fbd1cb0741e30308bf7aae562f65e3fd54359573` | `172.104.88.12` | `26656` | `26657` |
| testnet | `a944a1fa8259e19a9bac2c2b41d050f04ce50e51` | `172.105.213.114` | `26656` | `26657` |

## Run using pre-compiled binary

### Install `amod` daemon
Refer to [Getting Started](https://github.com/amolabs/amoabci#getting-started)
section in amoabci document to install `amod` from either pre-built binary or
source.

### Execute setup script
First, figure out current server's external ip address and seed node's
information. Then, execute the following command as root:
```bash
sudo ./setup.sh -e <ext_ip_addr> <data_root> <moniker> <node_id>@<node_ip_addr>:<node_p2p_port>
```

Specify proper `ext_ip_addr`, `data_root`, `moniker`(node name) and `node_*`
information, depending on which network you would like to connect to by
referring to [Chain info](#chain-info) section.

For example, if current server's external ip address is `123.456.789.0`(not
avaiable ip address), data root is `/mynode`, node name is `mynodename` and
you'd like to connect to mainnet, then execute the following commands:
```bash
sudo ./setup.sh -e 123.456.789.0 /mynode mynodename fbd1cb0741e30308bf7aae562f65e3fd54359573@172.104.88.12:26656
```

### Run service
To run validator node, execute the following commands as root:
```bash
sudo systemctl start amod
```

To check validator node status, execute the following commands as root:
```bash
sudo systemctl status amod
```

To stop validator node, execute the following commands as root:
```bash
sudo systemctl stop amod
```

## Run using Docker

### Install `docker`
Refer to [Get Docker](https://docs.docker.com/get-docker/) in Docker's official
document to install `docker` from either pre-built binary or source.

### Pull `amolabs/amod` image
To pull the official `amod` image of amolabs, execute the following commands:
```bash
sudo docker pull amolabs/amod:<tag>
```

Specify proper `tag` which indicates a specific version of `amod` image. To
pull the latest image, `tag` should be `latest` or can be omitted. For example,
if you would like to pull `1.6.5`, then execute the following commands: 
```bash
sudo docker pull amolabs/amod:1.6.5
```

### Execute setup script
First, figure out current server's external ip address and seed node's
information. Then, execute the following command as root:
```bash
sudo ./setup.sh -d -e <ext_ip_addr> <data_root> <moniker> <node_id>@<node_ip_addr>:<node_p2p_port>
```

Specify proper `ext_ip_addr`, `data_root`, `moniker`(node name) and `node_*`
information, depending on which network you would like to connect to by
referring to [Chain info](#chain-info) section.

For example, if current server's external ip address is `123.456.789.0`(not
avaiable ip address), data root is `/mynode`, node name is `mynodename` and
you'd like to connect to mainnet, then execute the following commands:
```bash
sudo ./setup.sh -d -e 123.456.789.0 /mynode mynodename fbd1cb0741e30308bf7aae562f65e3fd54359573@172.104.88.12:26656
```

### Run container
To run(create & start) validator node, execute the following commands as root:
```bash
sudo docker run -d --name amod -v <data_root>:/amo amolabs/amod
```

To start validator node, execute the following commands as root:
```bash
sudo docker start amod
```

To check validator node status, execute the following commands as root:
```bash
sudo docker stats amod
```

To stop validator node, execute the following commands as root:
```bash
sudo docker stop amod
```

## Postrequisite

### Backup keys
Backup `priv_validator_key.json`, `node_key.json` files found below
`/mynode/amo/config` directory to a safe location.

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

We assume you possess the account key (`myval` for amocli key username,
`D2CC7F160874AF06027A09DC0E8DC67E85E6D704` for address) and necessary AMO
coins. Now, you need to send a `stake` transaction to the blockchain. For the
`stake` transaction you need a validator public key. This process is equivalent
to announce to the world that you take the control over a validator node which
is running with the validator public key. 

### Inspect validator public key
To find out the validator public key of a node which is launched, connect to a
terminal of the server machine and execute the following command:
```bash
amod --home <data_root>/amo tendermint show_validator
# or
docker run -it --rm -v <data_root>/amo:/amo amolabs/amod amo tendermint show_validator
```
Specify proper `data_root`.

For example, if data root directory is `/mynode`, execute the following
command: 
```bash
amod --home /mynode/amo tendermint show_validator
# or
docker run -it --rm -v /mynode/amo:/amo amolabs/amod amo tendermint show_validator
```

We assume the validator public key is as follows:
```json
{
  "pub_key": {
    "type": "tendermint/PubKeyEd25519",
    "value": "+4jvv6ZCP+TxC0CwBQRr31ieZzj7KMZL3iwribL3czM="
  }
}
```
Now, you have a validator public key to announce.

### Send stake transaction
To stake certain amount of AMO coins on a validator, execute the following
command:
```bash
amocli tx --user <key_username> stake <validator_pub_key> <amount>
```
Specify proper `key_username`, `validator_pub_key` and `amount`.

For example, to stake 1000000 AMO along with the validator public key, execute
the following command:
```bash
amocli tx --user myval stake +4jvv6ZCP+TxC0CwBQRr31ieZzj7KMZL3iwribL3czM= 1000000000000000000000000 
```

### Query stake
To check if certain amount of AMO coins are properly staked, execute the
following command:
```bash
amocli query stake <key_address>
```
Specify proper `key_address`.

For example, to check if 1000000 AMO coins are properly staked on `myval` along
with the validator public key, execute the following command:
```bash
amocli query stake D2CC7F160874AF06027A09DC0E8DC67E85E6D704
```

### Inspect validators
To inspect the list of validators and their voting power, execute the following
command:
```bash
curl localhost:26657/validators
```

### Send withdraw transaction
To withdraw all or part of AMO coins locked as a stake, execute the following
command:
```bash
amocli tx --user <key_username> withdraw <amount>
```
Specify proper `key_username` and `amount`.

For example, to withdraw 100 AMO from `myval`'s stakes, exectue the following
command:
```bash
amocli tx --user myval 100000000000000000000
```

## Delegate coins
We assume you possess the account key (`mydel` for amocli key username
`4BFCD048B837135C1F23B6302000E096D48F99B8` for key address) and necessary AMO
coins. Now, you need to send a `delegate` transaction to the blockchain. For
the `delegate` transaction you need a validator account address to which you
would like to delegate specific amount of AMO coins. In this guide, we assume
the degelatee's validator account address is
`D2CC7F160874AF06027A09DC0E8DC67E85E6D704`.

### Send delegate transaction
To delegate certain amount of AMO coins to a validator, execute the following
command:
```bash
amocli tx --user <key_username> delegate <validator_account_address> <amount>
```
Specify proper `key_username`, `validator_account_address` and `amount`.

For example, to delegate 100 AMO to the validator `myval`'s account address,
execute the following command:
```bash
amocli tx --user mydel delegate D2CC7F160874AF06027A09DC0E8DC67E85E6D704 100000000000000000000
```

### Query delegate
To check if certain amount of AMO coins are properly delegated, execute the
following command:
```bash
amocli query delegate <key_address>
```
Specify proper `key_address`.

For example, to check if 100 AMO coins are properly delegated, execute the
following command:
```bash
amocli query delegate 4BFCD048B837135C1F23B6302000E096D48F99B8
```

### Send retract transaction
To retract all or part of the AMO coins locked as a delegated stake, execute
the following command:
```bash
amocli tx --user <key_username> retract <amount>
```
Specify proper `key_username` and `amount`.

For example, to retract 1 AMO from `mydel`'s delegate stakes, exectue the
following command:
```bash
amocli tx --user mydel 1000000000000000000
```
