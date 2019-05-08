# AMO Blockchain Console Manual

## Installation
See [AMO ABCI app](https://github.com/amolabs/amoabci) to install `amocli` command line executable.

## General
`amocli` binary takes commands and subcommands:
```bash
amocli <command> <subcommand> <args> <flags>
```
For example:
```bash
amocli key list
```

`amocli` takes two global flags:
- `-j, --json`: output as JSON, if not given output as human-friendly format
	- default off
- `-r, --rpc`: designate remote RPC node, if not given use a node running in localhost
	- default `0.0.0.0:26657`

`--json` flag is used mainly in scripts. Each command and subcommand may take its own flags.

## Version
`version` command displays current version information.

## Key
`key` command manages local keyring.

`key` command takes the following flags:
- `-e, --encrypt`: whether store the key in encrypted form or not

## Status
`status` command displays status of the currently connected RPC node.

## Query
`query` command queries various AMO blockchain data.

## Tx
`tx` command sends signed transaction to an RPC node.

`tx` command takes the following flags:
- `-u, --user`: one of usernames(a.k.a. *nickname*) in a local keyring
- `-p, --pass`: passphrase for the given username

## Parcel
`parcel` command interacts with AMO storage services.

