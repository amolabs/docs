# AMO blockchain protocol specification

## Data Format
### Key
AMO blockchain uses ECDSA key pair to sign and verify various transactions and messages. AMO blockchain uses NIST P256 curve as its default ECDSA domain parameter.

A key pair in AMO blockchain is a pair of a private key and a public key. A private key is a sequence 32 bytes, and a public key is a sequence of 33 bytes(compressed form. TODO: give a reference). These byte sequences are represented by base64 encoding when transmitted over a communication channel or stored as a marshaled form, while they reside *as is* in a program's internal memory space.

### Key custody
A key custody is a special form of key transfer medium. It is a public-key encryption of a data encryption key: `PKEnc(PK, DEK)`, where `PKEnc` is a sort of a hybrid encryption (combination of public key encryption and symmetric key encryption). For `PKEnc`, we use a combination of ECDH ephemeral mode and AES. For ECDH ephemeral key generation, we use ECDSA key generation algorithm.`PK` is a public key of a recipient and `DEK` is a data encryption key of an encrypted *data parcel*. 

### Account address
An address is a human-readable character string which is a hex-encoding of a byte sequence with the length of 20 bytes (=160-bit). Hence, the opaque form of an address is a 40-byte character string which consists of `[0-9]` and `[a-f]` only.

An account address is derived from the public key of an account. First, take 32 bytes by applying SHA256 on the public key bytes. Next, take 20 bytes by truncating the first 20 bytes from the 32-byte SHA256 output: `addr_bin = trunc_20(SHA256(PK))`. For the last step, convert this `addr_bin` by hex-encoding. An AMO-compliant program may utilize this `addr_bin` for its internal purpose, but it should apply hex-encoding before sending to other protocol party or storing to other medium outside the program.

**NOTE:** In Bitcoin, they use `addr_bin = RIPEMD160(SHA256(PK))`, but we cannot use RIPEMD160. See [Notes on Cryptography](crypto.md) for more details and reasons.

### Transaction
payload to Tendermint `DeliverTx` and `CheckTx` methods

### Encoding

Any byte sequence other than `account address` is represented by a base64 encoding.

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
