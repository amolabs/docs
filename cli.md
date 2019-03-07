# AMO blockchain CLI specification

## Management

`amod` is basically a daemon that runs the AMO ABCI. In addition, it fully
supports basic [tendermint RPC functions](https://tendermint.com/rpc) such as `BlockchainInfo` that querying AMO blockchain metadata.

You can use `amod` to run the `abci` node with CLI.

```Bash
amod run

I[2019-03-05|12:27:11.322] Starting ABCIServer                          module=abci-server impl=ABCIServer
I[2019-03-05|12:27:11.324] Waiting for new connection...                module=abci-server
```
