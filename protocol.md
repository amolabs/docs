# AMO blockchain protocol specification

**TODO:** staking and validator update

## Introduction
Although the current implementation of AMO blockchain depends ***heavily*** on Tendermint, AMO blockchain protocol itself is independent of Tendermint. It is described by several protocol messages and corresponding state transition in abstract internal database of each blockchain node. While the protocol messages are concretely defined(meaning and format), abstract internal database of a blockchain node is implementation-specific. But, note that every AMO blockchain node **MUST** incorporate a kind of database storing all kinds of data items described in [Internal Data](#internal-data) section.

Some notes related to Tendermint will be tagged with **TM**.

## Data Format
### Key
AMO blockchain uses ECDSA key pair to sign and verify various transactions and messages. AMO blockchain uses NIST P256 curve as its default ECDSA domain parameter.

A key pair in AMO blockchain is a pair of a private key and a public key. A private key is a sequence 32 bytes, and a public key is a sequence of 33 bytes(compressed form. TODO: give a reference). These byte sequences are represented by HEX encoding when transmitted over a communication channel or stored as a marshaled form, while they reside *as is* in a program's internal memory space.

### Key custody
A key custody is a special form of key transfer medium. It is a public-key encryption of a data encryption key: `PKEnc(PK, DEK)`, where `PKEnc` is a sort of a hybrid encryption (combination of public key encryption and symmetric key encryption). For `PKEnc`, we use a combination of ECDH ephemeral mode and AES. For ECDH ephemeral key generation, we use ECDSA key generation algorithm.`PK` is a public key of a recipient and `DEK` is a data encryption key of an encrypted *data parcel*.

### Account address
An address is a human-readable character string which is a hex-encoding of a byte sequence with the length of 20 bytes (=160-bit). Hence, the opaque form of an address is a 40-byte character string which consists of `[0-9]` and `[a-f]` only.

An account address is derived from the public key of an account. First, take 32 bytes by applying SHA256 on the public key bytes. Next, take 20 bytes by truncating the first 20 bytes from the 32-byte SHA256 output: `addr_bin = trunc_20(SHA256(PK))`. For the last step, convert this `addr_bin` by hex-encoding. An AMO-compliant program may utilize this `addr_bin` for its internal purpose, but it should apply hex-encoding before sending to other protocol party or storing to other medium outside the program.

**NOTE:** In Bitcoin, they use `addr_bin = RIPEMD160(SHA256(PK))`, but we cannot use RIPEMD160. See [Notes on Cryptography](crypto.md) for more details and reasons.

## Message Format
### Transaction
A transaction is a description of the state change in a blockchain node's internal database(i.e. blockchain state). In other words, sending a transaction to a blockchain node is the only way to change blockchain state. When a transaction is received by a node and eventually included in a _block_, a blockchain node shall modify the internal database according to each transaction type.

A transaction is represented by a JSON document which has the following format:
```json
{
    "type": "_tx_type_",
    "sender": "_sender_address_",
    "nonce": "_HEX-encoded_nonce_bytes_",
    "params": "_HEX-encoded_JSON_object_",
    "signature": {
        "pubkey": "_HEX-encoded_public_key_bytes_",
        "sig_bytes": "_HEX-encoded_signature_bytes_"
    }
}
```
`"_tx_type_"` is one of :
- `"transfer"`
- `"register"`
- `"request"`
- `"grant"`
- `"discard"`
- `"cancel"`
- `"revoke"`

`_sender_address_` identifies the sender or originator of this transaction. `"params"` is a HEX-encoded JSON object, which is different for each transaction type.

- transfer body:
```json
{
    "to": "_recipient_address_",
    "amount": "_currency_"
}
```
- register body:
```json
{
    "target": "_parcel_id_",
    "key_custody": "_owner_custody_",
    "extra": "_extra_info_"
}
```
- request body:
```json
{
    "target": "_parcel_id_",
    "payment": "_currency_",
    "extra": "_extra_info_"
}
```
- cancel body
```json
{
    "target": "_parcel_id_"
}
```
- grant body
```json
{
    "target": "_parcel_id_",
    "grantee": "_buyer_address_",
    "key_custody": "_buyer_custody_"
}
```
- revoke body
```json
{
    "target": "_parcel_id_",
    "grantee": "_buyer_address_"
}
```
- discard body
```json
{
    "target": "_parcel_id_"
}
```

`"pubkey"` is of type P256, neither of ed25519 or secp256k1. It is a HEX-encoding of a compressed elliptic curve point with the length of 33 bytes.

`"sig_bytes"` is HEX-encoded ECDSA signature, which is a concatenation of `r` and `s`. This is `(r, s) = ECDSA(privkey, sb)`, where `privkey` is the corresponding private key, and `sb` is a concatenation of values of `"type"`, `"sender"`, `"nonce"` and `"params"`.

### Transaction Result
**TM:** TBA

![protocol_transaction](./images/protocol_transaction.svg)

**TM:** Tendermint core sends a **Transaction** to AMO app, and the app replies with the response **ResponseDeliverTx**. This **ResponseDeliverTx** is defined in Tendermint, but is not part of AMO blockchain protocol. However, this is an important reply to the users indicating whether the transmitted transaction was successfully processed or not. This reply is described in [RPC](rpc.md) document. Moreover, [CLI](https://github.com/amolabs/amoabci/tree/master/cmd/amocli) may process this reply and prompt the user with a more human-friendly output.

## Internal Data

* internal persistent database of (address, balance)
* internal persistent database of (parcel id, owner address, key custody, extra info)

## Operations
### AMO coin transfer
1. AMO user creates a signed transaction and sends to one of AMO nodes.
1. AMO node processes the transaction.

### Register data parcel

### Grant permission
* "grant `read` permission on a data parcel `A` to an AMO user `B`"
* "revoke `read` permission on a data parcel `A` from an AMO user `B`"

## Genesis App State
Initial state of the app (_genesis app state_) is defined by genesis document (genesis.json file in tendermint config directory, typically $HOME/.tendermint/config/genesis.json). Initial app state is described in `app_state` field in a genesis document. For example:
```json
"app_state": {
    "balances": [
        {
            "owner": "7CECB223B976F27D77B0E03E95602DABCC28D876",
            "amount": "100"
        }
    ]
}
```
**NOTE:** In order to reset and apply new genesis state, run the following command in command line:
```bash
tendermint unsafe_reset_all
```
An AMO-compliant blockchain node should have some mechanisms to modify internal database for this operation.