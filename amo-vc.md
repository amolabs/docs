# AMO Verifiable Credential Registry

## Introduction

A verifiable credential is a tamper-evident credential that has authorship taht
can be cryptographically verified. Typically, a verifiable credential is likely
to include a sensitive information about individuals, thus it is natural to
store a verifiable credential in a private storage, and hand it to a verifier
in a limited circumstances. However, some publicly known entities may find it
convenient to publish their verifiable credentials, so that everyone in the
domain can read and verify the credential at any time. Those examples may
include government agencies and public officials. AMO verifiable credential
registry safely stores and serves those public verifiable credentials.

## Identifiers

According to [Verifiable Credentials Data Model
1.0](https://www.w3.org/TR/vc-data-model/), a verifiable credential may have an
optional `id` property. In the context of AMO verifiable credential registry,
every verifiable credential MUST have a unique `id` to be stored in the
registry. This `id` property shall be used to uniquely determine a undisputable
location of the verifiable credential, thus it must be globally unique. For
this purpose, an `id` property of a verifiable credential MUST have the
following ABNF form:

```
amo-vc-id = "amo" ":" "cred" ":" idstring
idstring = 64idchar
idchar = DIGIT / "A" / "B" / "C" / "D" / "E" / "F"
```

`idstring` can be constructed from an arbitrary byte string of length 32
(256-bit binary string), which is converted to an hexadecimal representation.

A verifiable credential is called a _named_ verifiable credential in the
context of AMO verifiable credential registry, or more generally, in AMO
ecosystem.

## Data Model

In order to safely store a named verifiable credential, the following
properties MUST be present in the credential:

- `id`
- `issuer`
- `issued`

There may be other properties required to properly handle the credential
([Verifiable Credential Data Model 1.0](https://www.w3.org/TR/vc-data-model/)),
but AMO verifiable credential registry does not care about the validity of
those properties and store and server them _as is_.

### `id`

An `id` property 

### `issuer`

should be a DID as defined in [AMO DID Method](amo-did.md), whose document is
stored in AMO blockchain

### `issued`

ISO datetime string, which should be newer than the old one when replacing
existing VC

## CRUD (Operations)

### Create (Register)

Transmit a `did.issue` tx to the AMO blockchain mainnet. Form a tx body as
follows:

```json
{
  "target": _vc_id_,
  "credential": _credential_body_
}
```

`_vc_id_` is a VC id for a newly created verifiable credential
`_credential_body_`.

See [`did.issue` tx](protocol_v6.md#transaction-payload) for more detail.

See [Transmitting tx](#transmitting-tx) for information about sending a signed
tx to AMO blockchain.

### Read (Query)

Perform ABCI query for `vc` store to the AMO blockchain mainnet. Form query
message as follows:

```json
{
  "path": "/vc",
  "data": _query_data_
}
```

`_query_data_` is formed as follows:

1. construct double-quoted string containing a VC id<br/>
   `"amo:cred:70EAD5B53B11DFE78EC8CF131D7960F097D48D70"` for
   `amo:cred:70EAD5B53B11DFE78EC8CF131D7960F097D48D70`
1. convert the result of step 1 using hex conversion<br/>
   `22616D6F3A637265643A3730454144354235334231314446453738454338434631333144373936304630393744343844373022`

See [ABCI query](#abci-query) for information about sending an ABCI query to
AMO blockchain.

### Update (Replace)

Transmit a `did.issue` tx to the AMO blockchain mainnet. Form a tx body as
follows:

```json
{
  "target": _vc_id_,
  "credential": _credential_body_
}
```

`_vc_id_` is a VC id for an existing VC, which is replaced by a new verifiable
credential `_credential_body_`.

See [`did.issue` tx](protocol_v6.md#transaction-payload) for more detail.

See [Transmitting tx](#transmitting-tx) for information about sending a signed
tx to AMO blockchain.

### Delete (Revoke)

Transmit a `did.revoke` tx to the AMO blockchain mainnet. Form a tx body as
follows:

```json
{
  "target": _vc_id_
```

`_vc_id_` is a VC id for an existing VC to be deleted.

See [`did.revoke` tx](protocol_v6.md#transaction-payload) for more detail.

See [Transmitting tx](#transmitting-tx) for information about sending a signed
tx to AMO blockchain.

## Blockchain Interaction

### Accessing AMO Blockchain

In order to interact with AMO blockchain, one needs to find a RPC node of the
chain, and form an appropriate RPC message and send it to the node. One could
set up its own AMO blockchain node with the RPC feature on. However, there is a
public RPC node open to anybody in the world: https://rpc.amolabs.io.

*NOTE: rpc.amolabs.io serves on 443 (https) port instead of default 26657 port
of tendermint.*

The chain ID for AMO blockchain mainnet is `amo-cherryblossom-01`.

### Transmitting tx

Send signed tx to one of AMO blockchain RPC nodes. See [AMO blockchain
protocol](protocol_v6.md#transaction) for more detail.

### ABCI query

Send ABCI query message to one of AMO blockchain RPC nodes. See [AMO client RPC
spec](https://github.com/amolabs/docs/blob/master/rpc.md) for more detail.

### AMO Blockchain Explorer

Instead of querying a credential to an AMO blockchain RPC node, it is
possible to query it via AMO blockchain explorer. For example, in order to get
VC with an id `amo:cred:70EAD5B53B11DFE78EC8CF131D7960F097D48D70`, send
an HTTP request to the following URL.

```
https://api.explorer.amolabs.io/chain/amo-cherryblossom-01/vcs/70EAD5B53B11DFE78EC8CF131D7960F097D48D70
```

Since the public AMO blockchain explorer server has a persistent DNS name, it
is also possible to use the above URL as an alias for the verifiable credential
`amo:cred:70EAD5B53B11DFE78EC8CF131D7960F097D48D70`. In this case, VC
resolution is a simple HTTP request and the response is the VC for
the requested VC id.

## Notes on Status List Credentials

A special type of verifiable credential is used to manage the revocation status of multiple verifiable credentials.
