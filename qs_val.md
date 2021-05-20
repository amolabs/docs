# Quick Start Guide for Validator
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

## Run node

### Prerequisite

#### Server machine
In order to run a validator node, you need a physical server with a stable
internet connection or a virtual machine on a cloud service(Amazon AWS, Google
cloud, Microsoft Azure or similar services). In this guide, we assume typical
Ubuntu Linux is installed on the server machine.

#### Install necessary packages
Connect to a terminal of the server machine and install `git`, `curl` and `jq`:
```bash
sudo apt install git curl jq
```

#### Prepare data directory
There needs a directory `data_root` in which all of AMO related data are stored
on the server machine. To make the directory, execute the following commands:
```bash
sudo mkdir -p <data_root>/amo/config
sudo mkdir -p <data_root>/amo/data
```

Assuming the `data_root` is `/mynode`, execute the following commands:  
```bash
sudo mkdir -p /mynode/amo/config
sudo mkdir -p /mynode/amo/data
```

#### Download setup script
Execute the following command to download the validator setup script:
```bash
cd $HOME
git clone https://github.com/amolabs/testnet
```

#### Download `genesis.json`
Following the previous command, execute the following command to download
`genesis.json` file:
```bash
cd testnet
curl <node_ip_addr>:<node_rpc_port>/genesis | jq '.result.genesis' > genesis.json
```

See the following table to find out correct `node_ip_addr` and `node_rpc_port`
for your network.

| `chain` | `node_id` | `node_ip_addr` | `node_p2p_port` | `node_rpc_port` |
|-|-|-|-|-|
| mainnet | `fbd1cb0741e30308bf7aae562f65e3fd54359573` | `172.104.88.12` | `26656` | `26657` |
| testnet | `a944a1fa8259e19a9bac2c2b41d050f04ce50e51` | `172.105.213.114` | `26656` | `26657` |

For example, to download `genesis.json` file for **mainnet**, execute the
following commands:
```bash
cd testnet
curl 172.104.88.12:26657/genesis | jq '.result.genesis' > genesis.json
```

### Run using Docker

#### Install `docker`
Refer to [Get Docker](https://docs.docker.com/get-docker/) in Docker's official
document to install `docker` from either pre-built binary or source. For Ubuntu
Linux, execute the following command to install docker:
```bash
sudo apt install docker.io
```

#### Pull `amolabs/amod` image
To pull the official `amod` image from amolabs, execute the following commands:
```bash
sudo docker pull amolabs/amod:<tag>
```

Specify proper `tag` which indicates a specific version of `amod` image. To
pull the latest image, `tag` should be `latest` or can be omitted. For example,
if you would like to pull `1.6.6`, then execute the following commands: 
```bash
sudo docker pull amolabs/amod:1.6.6
```

#### Execute setup script
Figure out current server's external ip address(`<ext_ip_addr>`) and
seed node's information(`<node_id>@<node_ip_addr>:<node_p2p_port>`), and then
execute the following command:
```bash
sudo ./setup.sh -d -e <ext_ip_addr> <data_root> <moniker> <node_id>@<node_ip_addr>:<node_p2p_port>
```

`moniker` is the name of your node. So, choose anything you want it to be the
unique name. If current server's external ip address is `111.111.111.111`, data
root is `/mynode`, node name is `mynodename` and you'd like to connect to
mainnet, then execute the following commands:
```bash
sudo ./setup.sh -d -e 111.111.111.111 /mynode mynodename fbd1cb0741e30308bf7aae562f65e3fd54359573@172.104.88.12:26656
```

#### Sync from snapshot (optional)
Before running a node, there are two available options to sync blocks; sync
from genesis block or sync from snapshot. If you'd like to sync from genesis
block, you can skip this step.

As syncing from genesis block consumes lots of physical time, we offer snapshot
of blocks taken at certain block height. The offerings are as follows:
| chain id | `preset` | `version` | `db_backend` | `block_height` | size</br>(comp/raw) | download |
|-|-|-|-|-|-|-|
| `amo-cherryblossom-01` | `cherryblossom` | `v1.7.7` | `rocksdb` | `11762421` | 101GB / 200GB | [file](http://us-east-1.linodeobjects.com/amo-archive/cherryblossom_v1.7.7_rocksdb_11762421.tar.bz2)([sha256](http://us-east-1.linodeobjects.com/amo-archive/cherryblossom_v1.7.7_rocksdb_11762421.tar.bz2.sha256)) |
| `amo-cherryblossom-01` | `cherryblossom` | `v1.7.5` | `rocksdb` | `7698783` | 70GB / 141GB | [file](http://us-east-1.linodeobjects.com/amo-archive/cherryblossom_v1.7.5_rocksdb_7698783.tar.bz2)([sha256](http://us-east-1.linodeobjects.com/amo-archive/cherryblossom_v1.7.5_rocksdb_7698783.tar.bz2.sha256)) |
| `amo-cherryblossom-01` | `cherryblossom` | `v1.7.5` | `rocksdb` | `6451392` | 56GB / 116GB | [file](http://us-east-1.linodeobjects.com/amo-archive/cherryblossom_v1.7.5_rocksdb_6451392.tar.bz2)([sha256](http://us-east-1.linodeobjects.com/amo-archive/cherryblossom_v1.7.5_rocksdb_6451392.tar.bz2.sha256)) |
| `amo-cherryblossom-01` | `cherryblossom` | `v1.6.5` | `rocksdb` | `2908399` | 21GB / 50GB | [file](http://us-east-1.linodeobjects.com/amo-archive/cherryblossom_v1.6.5_rocksdb_2908399.tar.bz2)([sha256](http://us-east-1.linodeobjects.com/amo-archive/cherryblossom_v1.6.5_rocksdb_2908399.tar.bz2.sha256)) |

**NOTE:** Mainnet's chain id is `amo-cherryblossom-01`.

To download and setup the snapshot, execute the following commands:
```bash
sudo wget http://us-east-1.linodeobjects.com/amo-archive/<preset>_<version>_<db_backend>_<block_height>.tar.bz2
sudo tar -xjf <preset>_<version>_<db_backend>_<block_height>.tar.bz2
sudo rm -rf <data_root>/amo/data/
sudo mv amo-data/amo/data/ <data_root>/amo/
```

**NOTE:** The directory structure of files extracted from compressed `*.tar.bz2`
file may differ from each one. Check out whether extracted `data/` directory is
properly placed under `<data_root>/amo/` directory.

For example, if chain id is `amo-cherryblossom-01`, version is `v1.7.5`, db
backend is `rocksdb`, block height is `7698783`, and data root is `/mynode,
then execute the following commands:
```bash
sudo wget http://us-east-1.linodeobjects.com/amo-archive/cherryblossom_v1.7.5_rocksdb_7698783.tar.bz2
sudo tar -xjf cherryblossom_v1.7.5_rocksdb_7698783.tar.bz2
sudo rm -rf /mynode/amo/data/
sudo mv amo-data/amo/data/ /mynode/amo/
```

That' pretty much it. Counduct the following steps.

#### Run container
To create and run a node, execute the following command:
```bash
sudo docker run -d --name amod -v <data_root>/amo:/amo amolabs/amod
```

To start a node, execute the following command:
```bash
sudo docker start amod
```

To check node status, execute the following command:
```bash
sudo docker stats amod
```

When you need to stop the node later, execute the following command:
```bash
sudo docker stop amod
```

### Additional works

#### Backup keys
Backup `priv_validator_key.json`, `node_key.json` files in `/mynode/amo/config`
directory to a safe location.

#### Gather information
Execute the following command and write down the validator address and public
key somewhere convenient.
```bash
curl localhost:26657/status
```

<p align="center"><img src="images/node_status.png"/></p>

Check `"sync_info"` in the output to see if our node is syncing properly with
the other nodes in the blockchain network. If `"catching_up"` is `true`, then
it still behind the tip of the blockchain. So, wait until the syncing process
is complete. When there are a lot of blocks to sync, you may need to wait
several hours or days.

## Upgrade node
AMO blockchain node S/W would be upgraded from time to time, or periodically.
When new node S/W is released, you need to upgrade your node as soon as
possible. Especially when your node is a validator node, and the upgrade
includes a protocol change, you are **required** to prepare before the upgrade
height and execute the upgrade **on time**.

When the blockhain reaches a designated height where the protocol changes,
`amod` service will halt with a error message `protocol mismatch`. When this
happens, prepare new `amod` docker image and restart the service as soon as
possible.

Refer to [Protocol](protocol.md#on-chain-protocol-upgrade) document section to
learn how 'On-chain Protocol Upgrade' works in more detail.

### Using Docker

#### Pull latest image
To pull latest `amolabs/amod` docker image, execute the following command:
```bash
sudo docker pull amolabs/amod:latest
```

#### Restart container 
To restart `amod` docker container, execute the following commands:
```bash
sudo docker stop amod && sudo docker rm amod
sudo docker run -d --name amod -v <data_root>:/amo amolabs/amod:latest
```

## Stake coins
**NOTE:** For the testnet you may visit <a
href="http://explorer.amolabs.io/wallet">AMO blockchain explorer</a> and follow
the guide there.

### Install `amocli`
In order to stake or delegate coins, you need install `amocli`(AMO client). Consult [Install](https://github.com/amolabs/amo-client-go#installation) section of the amo-client-go document to install `amocli`.

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
