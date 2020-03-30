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
Connect to a terminal of the server machine and install git as root:
```bash
sudo apt install git
```

## Launch on testnet
### Prepare
First, select data directory location. In this guide, we assume you selected
`/testnet/mynode`. Select a node name. In this guide, we assume you selected
`mynodename`. Connect to a terminal of the server and execute the following
commands:
```bash
cd $HOME
git clone https://github.com/amolabs/testnet
cd testnet
./setup.sh /testnet/mynode mynodename f5123e0f663fe8e0662b82de8f6a1d843a9d4fbd@172.104.88.12:26656
```

### Backup key
Backup `/testnet/mynode/amo/config/priv_validator_key.json` file to a
safe location.

### Execution 
Connect to a terminal of the server and execute the following commands:
```bash
sudo systemctl start amod
```

### Gather information
Install `curl` package in the server's terminal:
```bash
sudo apt install curl
```
If the program ask something, input `y` and press `enter` key.

Execute the following command and write down the validator address and public
key somewhere convenient.
```bash
curl localhost:26657/status
```
<p align="center"><img src="images/node_status.png"/></p>

## Launch on mainnet
### Install
See [Getting Started](https://github.com/amolabs/amoabci#getting-started)
section in amoabci document to install `amod` from either pre-built binary or
source.

### Prepare
See [Prepare for launch](https://github.com/amolabs/amoabci#prepare-for-launch)
section in amoabci document to get information and prepare data directory. We
will assume the following:
- data root at `/mynode`
- `config.toml` at `/mynode/amo/config/config.toml`
- `genesis.json` at `/mynode/amo/config/genesis.json`

### Prepare validator key
Run the following command to generate a validator key:
```bash
amod --home <dataroot>/amo tendermint init
```
Here, `<dataroot>` is a data directory prepared previously. It will generate
missing keys: node key and validator key. Copy
`/mynode/amo/config/priv_validator_key.json` to some secure place for a backup.
We assume the validator public key is as follows:
```
{
	"address": "9A8F09C644941B5A526B19641A3D7C8805E312B9",
	"pub_key": {
		"type": "tendermint/PubKeyEd25519",
		"value": "+4jvv6ZCP+TxC0CwBQRr31ieZzj7KMZL3iwribL3czM="
	}
}
```

### Run node
Execute the following command to run the `amod` daemon:
```bash
amod --home <dataroot>/amo run
```
To run the daemon in background mode, use `amod --home <dataroot>/amo run &`.
`amod` will open port 26656 for incoming P2P connection and port 26657 for
incoming RPC connection.

Check the node status by running the following command:
```bash
apt install curl
curl localhost:26657/status
```
Check `"sync_info"` in the output to see if our node is syncing properly with
the other nodes in the blockchain network. If `"catching_up"` is `true`, then
it still behind the tip of the blockchain. So, wait until the syncing process
is complete.

## Stake coins
**NOTE:** For the mainnet, you must use more controlled steps as the
followings, but for the testnet you may visit <a
href="http://explorer.amolabs.io/wallet">AMO blockchain explorer</a> and follow
the guide there. (*Available on 6th Sep.*)

You need `amocli`(AMO client) to stake coins. See
[Installation](https://github.com/amolabs/amo-client-go#installation) section
in amo-client-go document to install `amocli` in proper way.

We assume you possess the account key (`myval` for amocli username) and
necessary AMO coins. Now, you need to send a `stake` transaction to the
blockchain. For the `stake` transaction you need a validator public key. This
process is equivalent to announce to the world that you take the control over a
validator node which is running with the validator public key. To find out the
validator public key of a node which is launched via docker, you may execute
the following command:
```bash
amod --home <dataroot>/amo tendermint show_validator
```
This will print the validator public key in the same format as seen in [Prepare
validator key](#prepare-validator-key) section. In order to feed this output to
some kind of automated script, you may do this:
```bash
amod --home <dataroot>/amo tendermint show_validator | python -c "import sys, json; print json.load(sys.stdin)['value']"
```

Now, you have a validator public key to announce. To stake 100 AMO along with
the public key, run the following command:
```bash
amocli tx stake --user myval '+4jvv6ZCP+TxC0CwBQRr31ieZzj7KMZL3iwribL3czM=' 100000000000000000000
```

You can inspect the list of validators and their voting power by executing the
following command:
```bash
curl localhost:26657/validators
```
