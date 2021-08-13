# AMO DID Method (Working Draft)

## Introduction

AMO is a platform on which various parties including corporations and
individual drivers can trade automotive data. AMO blockchain is a blockchain
technology supporting AMO platform and AMO ecosystem, specialized on uploading
and trading data parcels or access rights on the data. AMO DID method is a
subset of AMO blockchain feature set, which provides a concrete specification
for AMO-specific DID and appropriate operations. AMO DID method is valid for
AMO blockchain mainnet only. Thus it is necessary to set up a separate DID
resolution method to use any AMO blockchain testnet.

## AMO DID Scheme

ABNF notation for DID used in AMO DID Method is as follows:
```
amo-did = "did" ":" "amo" ":" address
address = 40addrchar
addrchar = DIGIT / "A" / "B" / "C" / "D" / "E" / "F"
```

Method name for AMO DID Method is `amo`. `address` part corresponds to an
account address in AMO blockchain, which is written in 40 _uppercase_
hexadecimal digits.

### Address Generation

Since AMO blockchain is based on its own account system, AMO DID method is
intended to use the same account system. However, as far as the DID format is
valid, it is possible to generate any DID address without considering actual
AMO blockchain account. Hence, in its extreme case, it is possible to generate
a random 160-bit binary stream and convert it to an uppercase hexadecimal
representation with 40 characters.

When generating a new DID based on an AMO blockchain account, one would execute
either of the following procedures:

1. Generate a new AMO blockchain address according to [Account
   address](https://github.com/amolabs/docs/blob/master/protocol_v5.md#account-address)
   described in [AMO blockchain
   protocol](https://github.com/amolabs/docs/blob/master/protocol_v5.md). Grab
   the account address `address` and form a DID as a concatenation of
   `did:amo:` and `address`.
2. Obtain an account address `address` from existing AMO blockchain account.
   Form a DID as a concatenation of `did:amo:` and `address`.

### Examples

The followings are valid AMO DIDs:

```
did:amo:70EAD5B53B11DFE78EC8CF131D7960F097D48D70
did:amo:0000000000000000000000000000000000000000
```

The followings are not valid AMO DIDs:

```
did:amo:70ead5b53b11dfe78ec8cf131d7960f097d48d70 (lower case)
did:amo:70EAD5B53B11DFE78EC8CF131D7960F097D48D (length error)
```

## DID Document

### Representation

In AMO DID method, a DID document is represented in the JSON-LD format. When
represented in JSON-LD format, `@context` property must have the following
entry:

```json
"@context": "https://www.w3.org/ns/did/v1"
```

### Properties

An AMO DID document must have the following top-level properties:

- `id`
- `verificationMethod`
- `authentication`

There can be following optional top-level properties.

- `controller`
- `assertionMethod`

#### Identifier

Identifier, i.e. DID, which references to this DID document is represented by a
`id` property, which is described in [AMO DID Scheme](#amo-did-scheme).

#### Verification Method

Verification method is represented by a `verificationMethod` property. Since an
AMO blockchain account has a pair of private and public keys,
`verificationMethod` property must have at least one entry as the first one.
`id` of the first verification method must be a concatenation of `id` of the
DID subject and `#keys-1`. As described in [AMO blockchain
protocol](https://github.com/amolabs/docs/blob/master/protocol_v5.md#account-key), AMO
blockchain uses ECDSA over NIST P256 curve. A corresponding public key is
described `publicKeyJwk` property. Properties of `publicKeyJwk` is as described
in [JSON Web Key
2020](https://w3c-ccg.github.io/lds-jws2020/#json-web-key-2020).
`controller` property must be the same as the DID subject.

```json
"verificationMethod": [{
  "id": "did:amo:addressinhex#keys-1",
  "type": "JsonWebKey2020",
  "controller": "did:amo:addressinhex",
  "publicKeyJwk": {
    "kty": "EC",
    "crv": "P-256",
    "x": ...,
    "y": ...
  }
}]
```

In AMO blockchain protocol, the public key of an account is represented by [an
uncompressed form](https://datatracker.ietf.org/doc/html/rfc5480#section-2.2)
with `04` prefix. `x` and `y` properties of `publicKeyJwk` are extracted from this
uncompressed form as follows:

- x = base64(pubkey[1:33])
- y = base64(pubkey[33:65])

*NOTE: This property may be changed to an optional property.*

*NOTE: Currently, there is no mechanism for key rotating and revocation.*

#### Authentication

Authentication relation of a verification method is represented by an
`authentication` property. `authentication` property must have a reference to
the first verification method as described in [Verification
Method](#verification-method). For example, `did:amo:addressinhex#keys-1`.

*NOTE: This property may be changed to an optional property.*

#### Assertion

Assertion relation of a verification method is represented by an
`assertionMethod` property. `assertionMethod` property must have a reference to
the first verification method as described in [Verification
Method](#verification-method). For example, `did:amo:addressinhex#keys-1`.

*NOTE: This property may be changed to an optional property.*

#### Controller

`controller` property is optional. 

### Examples

```json
{
  "@context": "https://www.w3.org/ns/did/v1",
  "id": "did:amo:70EAD5B53B11DFE78EC8CF131D7960F097D48D70",
  "controller": "did:amo:0687D766FF0563B86BFF078B7F560AFC070C81AD",
  "verificationMethod": [{
    "id": "did:amo:70EAD5B53B11DFE78EC8CF131D7960F097D48D70#keys-1",
    "type": "JsonWebKey2020",
    "controller": "did:amo:70EAD5B53B11DFE78EC8CF131D7960F097D48D70",
    "publicKeyJwk": {
      "kty": "EC",
      "crv": "P-256",
      "x": ...,
      "y": ...
    }
  }],
  "authentication": [
    "did:amo:70EAD5B53B11DFE78EC8CF131D7960F097D48D70#keys-1"
  ],
  "assertionMethod": [
    "did:amo:70EAD5B53B11DFE78EC8CF131D7960F097D48D70#keys-1"
  ],
}
```

## CRUD (Operations)

### Create (Register)

Transmit a `claim` tx to the AMO blockchain mainnet. Form a tx body as follows:
```json
{
  "target": _amo_did_,
  "document": _amo_did_document_
}
```
`_amo_did_` is a DID for a newly created DID document `_amo_did_document_`.

See [`claim` tx](https://github.com/amolabs/docs/blob/master/protocol_v5.md#transaction-payload)
for more detail.

See [Transmitting tx](#transmitting-tx) for information about sending a signed
tx to AMO blockchain.

### Read (Resolve)

Perform ABCI query for `did` store to the AMO blockchain mainnet. Form a query
message as follows:
```json
{
  "path": "/did",
  "data": _query_data_
}
```
`_query_data_` is formed as follows:
1. construct a double-quoted string containing an AMO
   DID<br/>
   `"did:amo:70EAD5B53B11DFE78EC8CF131D7960F097D48D70"` for
   `did:amo:70EAD5B53B11DFE78EC8CF131D7960F097D48D70`
1. convert the result of step 1 using hex conversion<br/>
   `226469643A616D6F3A0A3730454144354235334231314446453738454338434631333144373936304630393744343844373022`

See [ABCI query](#abci-query) for information about sending an ABCI query to
AMO blockchain.

### Update (Replace)

Transmit a `claim` tx to the AMO blockchain mainnet. Form a tx body as follows:
```json
{
  "target": _amo_did_,
  "document": _amo_did_document_
}
```
`_amo_did_` is a DID for an existing DID document, which is replaced by a new
document `_amo_did_document_`.

See [`claim` tx](https://github.com/amolabs/docs/blob/master/protocol_v5.md#transaction-payload)
for more detail.

See [Transmitting tx](#transmitting-tx) for information about sending a signed
tx to AMO blockchain.

### Deactivate (Revoke)

Transmit a `dismiss` tx to the AMO blockchain mainnet. Form a tx body as
follows:
```json
{
  "target": _amo_did
}
```
`_amo_did` is a DID for an existing DID document to be deactivated.

See [`dismiss` tx](https://github.com/amolabs/docs/blob/master/protocol_v5.md#transaction-payload)
for more detail.

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
protocol](https://github.com/amolabs/docs/blob/master/protocol_v5.md#transaction)
for more detail.

### ABCI query

Send ABCI query message to one of AMO blockchain RPC nodes. See [AMO client RPC
spec](https://github.com/amolabs/docs/blob/master/rpc.md) for more detail.

### AMO Blockchain Explorer

Instead of querying a AMO DID document to an AMO blockchain RPC node, it is
possible to query it via AMO blockchain explorer. For example, in order to get
DID document for a DID `did:amo:70EAD5B53B11DFE78EC8CF131D7960F097D48D70`, send
an HTTP request to the following URL.

```
https://api.explorer.amolabs.io/chain/amo-cherryblossom-01/dids/70EAD5B53B11DFE78EC8CF131D7960F097D48D70
```

Since the public AMO blockchain explorer server has a persistent DNS name, it
is also possible to use the above URL as an alias for the DID
`did:amo:70EAD5B53B11DFE78EC8CF131D7960F097D48D70`. In this case, DID
resolution is a simple HTTP request and the response is the DID document for
the requested DID.

## Security and Privacy Considerations

### Sybil Attack

An attacker must solve a computationally hard problem to forge a private key of
the corresponding account in order to register a DID document for an arbitrary
DID.

### Unauthorized Update

An attacker must solve a computationally hard problem to forge a private key of
the corresponding account in order to update or deactivate a DID document not
controlled by the attacker.

### Replay Attack

An attacker must subverts a [Replay
Preventer](https://github.com/amolabs/docs/blob/master/protocol_v5.md#replay-attack)
of AMO blockchain in order to replay any tx previously sent to the blockchain
by a legitimate user.

### Exposure of Account Address

It is inherent that an AMO blockchain account has a fixed address for a
lifetime of the account. It is inevitable to expose the account address while
interacting with AMO blockchain (or many other blockchains). Thus, this should
not be considered to be an additional privacy problem raised by this DID
method.

Since there is no published information about the real-world owner of an AMO
blockchain account, a user's real world identity remains safe even if the
account address is exposed.

### Linkability

Anyone who can read a public AMO blockchain data can track activies of an AMO
blockchain account. Thus all activies can be linkable to a specific AMO
blockchain account. However, this is an inherent property of AMO blockchain (or
many other blockchains). Thus, this should not be considered to be an
additional privacy problem raised by this DID method.

## References

