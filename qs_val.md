# Quick Start Guide for Validator
This document is available in [Korean](qs_val.ko.md) also.

## Introduction
This quick start guide describes how to prepare and launch a validator node in
proper way, and also how to stake on a validator. This guide will use the
method described in [amoabci
README](https://github.com/amolabs/amoabci/README.md).

## Monetary asset
Acquire enough AMO coins to stake on your validator. This guide does not
explain how to acquire AMO coins.

## Prerequisite

#### Server machine
Prepare either a physical server or a virtual machine with a stable internet
connection and a fast storage device. Minimum spec is as follows:
* Intel i5 or equivalent with minimum 2 cores
* 8GB RAM
* SSD storage with minimum 500GB free space

Recommended spec is as follows:
* Intel i5 or better with minimum 4 cores
* 16GM RAM
* SSD storage with minimum 1TB free space

In this guide, we assume typical Ubuntu Linux is installed on the server
machine.

#### Install necessary packages
Connect to a terminal of the server machine and install necessary packages:
```bash
sudo apt install git curl jq docker.io
```

#### Prepare data directory
Prepare `data_root` directory on a device with enough free space. We assume thi
s directory is `/mynode`. Prepare `config` and `data` directories:
```bash
sudo mkdir -p /mynode/amo/config
sudo mkdir -p /mynode/amo/data
```

#### Download setup script and `genesis.json`
Download a setup script:
```bash
cd $HOME
git clone https://github.com/amolabs/testnet
```
Download `genesis.json` file:
```bash
cd $HOME/testnet
curl <rpc_addr>/genesis | jq '.result.genesis' > genesis.json
```

See the following table to find out correct `rpc_addr`.

| `chain` | `rpc_addr` |
|-|-|
| mainnet | `20.194.0.193:26657` |
| testnet | `172.105.213.114:26657` |

For example, to download `genesis.json` file for **mainnet**, execute the
following commands:
```bash
cd $HOME/testnet
curl 172.104.88.12:26657/genesis | jq '.result.genesis' > genesis.json
```

## Run using Docker

#### Pull `amolabs/amod` image
Pull the official `amod` image:
```bash
sudo docker pull amolabs/amod:1.8.3
```

#### Execute setup script
Figure out current server's external ip address(`<ext_ip_addr>`) and
seed node's `p2p_addr`. Decide `moniker`(your node's name). Execute the
following command:
```bash
sudo ./setup.sh -d 1.8.3 -e <ext_ip_addr> /mynode <moniker> <p2p_addr>
```

| `chain` | `p2p_addr` |
|-|-|
| mainnet | `fbd1cb0741e30308bf7aae562f65e3fd54359573@172.104.88.12:26656` |
| testnet | `a944a1fa8259e19a9bac2c2b41d050f04ce50e51@172.105.213.114:26656` |

#### Download the latest snapshot
Instead of syncing from the geneis block, download the latest snapshot of the
chain DB and start syncing from there. Here is the list of all snapshots:

| Block height | DB backend | Size (uncompressed) | Protocol | SW ver. | Download |
|-|-|-|-|-|-|
| 19538000 | rocksdb | 138GB (263GB) | v4 | v1.7.7, v1.8.3 | [link](http://us-east-1.linodeobjects.com/amo-archive/cherryblossom_v1.7.7_rocksdb_19538000.tar.bz2)([sha256](http://us-east-1.linodeobjects.com/amo-archive/cherryblossom_v1.7.7_rocksdb_19538000.tar.bz2.sha256)) |
| 11762421 | rocksdb | 101GB (200GB) | v4 | v1.7.7 | [link](http://us-east-1.linodeobjects.com/amo-archive/cherryblossom_v1.7.7_rocksdb_11762421.tar.bz2)([sha256](http://us-east-1.linodeobjects.com/amo-archive/cherryblossom_v1.7.7_rocksdb_11762421.tar.bz2.sha256)) |
| 7698783 | rocksdb | 70GB (141GB) | v4 | v1.7.5 | [link](http://us-east-1.linodeobjects.com/amo-archive/cherryblossom_v1.7.5_rocksdb_7698783.tar.bz2)([sha256](http://us-east-1.linodeobjects.com/amo-archive/cherryblossom_v1.7.5_rocksdb_7698783.tar.bz2.sha256)) |
| 6451392 | rocksdb | 56GB (116GB) | v4 | v1.7.5 | [link](http://us-east-1.linodeobjects.com/amo-archive/cherryblossom_v1.7.5_rocksdb_6451392.tar.bz2)([sha256](http://us-east-1.linodeobjects.com/amo-archive/cherryblossom_v1.7.5_rocksdb_6451392.tar.bz2.sha256)) |
| 2908399 | rocksdb | 21GB (50GB) | v3 | v1.6.5 | [link](http://us-east-1.linodeobjects.com/amo-archive/cherryblossom_v1.6.5_rocksdb_2908399.tar.bz2)([sha256](http://us-east-1.linodeobjects.com/amo-archive/cherryblossom_v1.6.5_rocksdb_2908399.tar.bz2.sha256)) |

Download the snapshot:
```bash
wget http://us-east-1.linodeobjects.com/amo-archive/cherryblossom_v1.7.7_rocksdb_19538000.tar.bz2
wget http://us-east-1.linodeobjects.com/amo-archive/cherryblossom_v1.7.7_rocksdb_19538000.tar.bz2.sha256
```
Verify the chesksum:
```bash
sha256sum cherryblossom_v1.7.7_rocksdb_19538000.tar.bz2
cat cherryblossom_v1.7.7_rocksdb_19538000.tar.bz2.sha256
```
Check two checksums are equal. Replace the chain DB:
```bash
tar jxf cherryblossom_v1.7.7_rocksdb_19538000.tar.bz2
sudo rm -rf /mynode/amo/data/
sudo mv amo-data/amo/data/ /mynode/amo/
```

**NOTE:** The directory structure of files extracted from compressed `*.tar.bz2`
file may differ from each one. Check out whether extracted `data/` directory is
properly placed under `<data_root>/amo/` directory.

#### Run container
To create and run a node:
```bash
./run.sh -d 1.8.3 -p -s /mynode
```

Check Docker container status:
```bash
docker ps
```
There should be a container item named `amod`. Check the node's status:
```bash
curl localhost:26657/status
```
Check if the `catching_up` is true. It may take hours or even days to complete
the sync. Wait until the `catching_up` becomes false.

#### Backup keys
Backup `priv_validator_key.json`, `node_key.json` files in `/mynode/amo/config`
directory to a safe location. If you want to run a full node but not a working
validator, the job is done here.

## Stake coins
### Install `amocli`
In order to stake coins, you need install `amocli`(AMO client). Follow
[Install](https://github.com/amolabs/amo-client-go#installation) document to
install `amocli`.

### Generate holder key
Generate a key for the owner account of your node (assume the name is `mynode`):
```bash
amocli key generate mynode
amocli key list
```
Note the account address with the name `mynode`.

### Note validator public key
```bash
curl localhost:26657/status
```
<p align="center"><img src="images/val_pubkey.png"/></p>

### Send stake transaction
Send `stake` transaction:
```bash
amocli tx --user mynode stake <val_pubkey> <amount>
```
`val_pubkey` is the value of `pub_key` you noted in the previous step. `amount`
must be multiple of `1000000000000000000000000 mote` (1,000,000 AMO).

Query your stake:
```bash
amocli query stake <address>
```
`address` is the account address with the name `mynode`.

