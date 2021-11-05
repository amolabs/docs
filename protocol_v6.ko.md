# AMO Blockchain 프로토콜 사양

AMO protocol version 6 (DRAFT).

## 소개

현재 구현된 AMO 블록체인은 텐더민트에 크게 의존적이지만 AMO 블록체인의 프로토콜 자체는 텐더민트에 독립적입니다. 이는 각 블록체인 노드의 추상화 된 내부 데이터베이스에서 여러 프로토콜 메시지와 해당 상태 전환으로 설명이 됩니다. While the protocol messages
are concretely defined(meaning and format), abstract internal database of a
blockchain node is implementation-specific. But, note that every AMO blockchain
node **MUST** incorporate a kind of database storing all kinds of data items
described in [Blockchain Data](#blockchain-data) section.

Some notes related to Tendermint will be tagged with **TM**.

## Data Format
### Account Key
AMO blockchain uses ECDSA key pair to sign and verify various transactions and
messages. AMO blockchain uses NIST P256 curve(aka. secp256r1) and SHA256 as its
default ECDSA domain parameter.

A key pair in AMO blockchain is a pair of a private key and a public key. A
private key is a sequence 32 bytes, and a public key is a sequence of 65
bytes(uncompressed form with 0x04 prefix). These byte sequences are represented
by HEX encoding when transmitted over a communication channel or stored as a
marshaled form, while they may reside as other format in a program's internal
memory space.

A private key should **NEVER** be transmitted via a network communication
channel. A public key must be HEX-encoded in a protocol message.

The following types are used in this document.
- `_HEX_encoded_public_key_bytes_`

### Signature
A signature field has the following form:
```json
{
  "pubkey": "_HEX_encoded_public_key_bytes_",
  "sig_bytes": "_HEX_encoded_signature_bytes_"
}
```
`pubkey` is the signer's public key, and `sig_bytes` is HEX-encoded ECDSA
signature bytes, which is a concatenation of `r` and `s`. `(r, s) =
ECDSA(privkey, sb)` is output of ECDSA signature algorithm, where `privkey` is
the signer's private key.

### Validator Key
A validator key pair is a ed25519 key pair and handled by Tendermint, but
validator's public key is carried in a AMO blockchain protocol message when
staking AMO coin to acquire stakes. In this case, a validator's public key must
be HEX-encoded.

The following types are used in this document.
- `_HEX_encoded_ed25519_pubkey_`

### Key custody
A key custody is a special form of key transfer medium. It is recommended to be
a public-key encryption of a data encryption key `PKEnc(PK, DEK)`, where
`PKEnc` is a sort of a hybrid encryption (combination of public key encryption
and symmetric key encryption).  For `PKEnc`, we use a combination of ECDH
ephemeral mode and AES-256. For ECDH ephemeral key generation, we reuse ECDSA
key generation algorithm. `PK` is a public key of a recipient and `DEK` is a
data encryption key of an encrypted *data parcel*.

The following types are used in this document.
- `_HEX_encoded_key_custody_`

### Account address
An address is a human-readable character string which is a hex-encoding of a
byte sequence with the length of 20 bytes (=160-bit). Hence, the opaque form of
an address is a 40-byte character string which consists of `[0-9]` and `[A-F]`
only.

An account address is derived from the public key of an account. First, take 32
bytes by applying SHA256 on the public key bytes. Next, take 20 bytes by
truncating the first 20 bytes from the 32-byte SHA256 output: `addr_bin =
trunc_20(SHA256(PK))`. For the last step, convert this `addr_bin` by
HEX-encoding. An AMO-compliant program may utilize this `addr_bin` for its
internal purpose, but it should apply hex-encoding before sending to other
protocol party or storing to other medium outside the program.

**NOTE:** In Bitcoin, they use `addr_bin = RIPEMD160(SHA256(PK))`, but we
cannot use RIPEMD160. See [Notes on Cryptography](crypto.md) for more details
and reasons.

The following types are used in this document.
- `_account_address_` = `addr_bin`
- `_HEX_encoded_account_address_` = HEX encoding of `_account_address_`
- `"_HEX_encoded_account_address_"` as a JSON string

### Currency
As in other popular blockchain systems, AMO coin amount is expressed as an
integer value, which is a multiple of the smallest transferable unit. In AMO
blockchain the unit is called a *mote*. And *one AMO* is 10<sup>18</sup>
*motes*, where the number 18 is often called a *decimals* in the cryptocurrency
community. In all protocol messages, AMO coin amount is expressed in mote unit.

Amount of AMO coin or user-defined coin must be expressed as a decimal number
enclosed in double-quotes when included in a JSON-format message, i.e. all
protocol messages. However it may be expressed in other formats in blockchain
node's *internal* memory.

The following types are used in this document.
- `_currency_`
- `"_currency_"` as a JSON string

### Draft ID
A draft ID is a 32-bit unsigned integer. It is represented as a double-quoted
decimal number without redundant leading zeroes when used in JSON, e.g. in
protocol messages. However, it is represented as a 4-byte big-endian integer
including leading zeroes when it is used to composite another identifier.

The following types are used in this document.
- `_draft_id_` = alias of `_decimal_number_`
- `_draft_id_` as a JSON number, e.g. `1234` not `"1234"`

### Storage ID
A storage ID is a 32-bit unsigned integer. It is represented as a double-quoted
decimal number without redundant leading zeroes when used in JSON, e.g. in
protocol messages. However, it is represented as a 4-byte big-endian integer
when it is used to composite another identifier.

The following types are used in this document.
- `_storage_id_` = alias of `_decimal_number_`
- `_storage_id_` as a JSON number, e.g. `1234` not `"1234"`

### Parcel ID
A parcel ID is a concatenation of a storage ID and in-storage ID. In-storage ID
is a 32-byte(256-bit) binary sequence. See [AMO Storage
Specification](storage.md) for more detail. Be aware that a storage ID itself
is a 32-bit unsigned integer. But when forming a parcel ID, this storage ID is
converted as a four-byte binary sequence (big-endian integer). For example,
suppose a parcel in a storage with the id of `123456789` has the in-storage id
of `12ABEF23...`. This parcel has a parcel id `075BCD1512ABEF23...` in a HEX
encoding, where `075BCD15` is a HEX encoding of the integer `123456789` using
big-endian byte order.

The following types are used in this document.
- `_parcel_id_`
- `_HEX_encoded_parcel_id_` = HEX encoding of `_parcel_id_`
- `"_HEX_encoded_parcel_id_"` as a JSON string

### Extra info
`register`, `request` and `grant` tx may carry extra information. It must be a
JSON object, but its internal structure is application-specific. Internal DB of
a blockchain node must store extra information from the previous steps also,
i.e. `parcel` stores extra from register tx, `request` stores extra from
register tx and request tx, `usage` stores extra from register tx, request tx
and grant tx.

extra in `parcel` store
```json
{
  "register": {} // application-specific JSON object, optional
}
```
extra in `request` store
```json
{
  "register": {}, // application-specific JSON object, optional
  "request": {} // application-specific JSON object, optional
}
```
extra in `usage` store
```json
{
  "register": {}, // application-specific JSON object, optional
  "request": {}, // application-specific JSON object, optional
  "grant": {} // application-specific JSON object, optional
}
```
Since a JSON object must be enclosed by braces(`{` and `}`), it cannot be a
single JSON value. Each extra info must be an empty object(`{}`) or a proper
JSON object with members.

```json
{
  "register": "boo", // wrong
  "request": {"some":"value"}, // ok
  "grant": {} // ok
}
```
Each member is marked as *optional*, so an empty object(`{}`) is valid extra
information for all of three state stores.

### UDC(User-Defined Coin) ID

A UDC ID is a 32-bit unsigned integer. It is represented as a double-quoted
decimal number without redundant leading zeroes when used in JSON, e.g. in
protocol messages. However, it is represented as a 4-byte big-endian integer
when it is used to composite another identifier.

The following types are used in this document.
- `_udc_id_` = alias of `_decimal_number_`
- `_udc_id_` as a JSON number, e.g. `1234` not `"1234"`

### DID (Decentralized Identifier)

When used in AMO mainnet, a DID is a concatenation of the string `"did:amo:"`
and an account address represented as an uppercase hexadecimal string. When
used in networks other than AMO mainnet, the prefix would be something like
`"did:" + method_name + ":"`, where `method_name` is other than `"amo"`.

### DID Document

`did.claim` tx and `did.dismiss` tx operate on a DID document, which is used in
[AMO DID Method](amo-did.md). A DID document is a JSON document with an
additional top-level `@context` property, which is called a JSON-LD
representation. While an AMO blockchain node does not care about the value of
`@context` property, this property must exist and its value must be of a
*string* type.

### VC (Verifiable Credential) ID

When used in AMO mainnet, a VC ID is a concatenation of the string
`"amo:cred:"` and an uppercase hexadecimal string of maximum length of 64
characters, i.e. a representation of a byte string of length 32 or less. When
used in networks other than AMO mainnet, the prefix would be something like
`network_name + ":cred:"`, where `network_name` is other than `"amo"`.

### Verifiable Credential

`did.issue` tx and `did.revoke` tx operate on a VC (Verifiable Credential), which is used in [AMO Verifiable Credential Registry](amo-vc.md). A
VC is a JSON document with an additional top-level
`@context` property, which is called a JSON-LD representation. While an AMO
blockchain node does not care about the value of `@context` property, this
property must exist and its value must be of a string type.

## Message Format
### Transaction
A transaction is a description of the state change in a blockchain node's
internal database(i.e. blockchain state). In other words, sending a transaction
to a blockchain node is the only way to trigger the change in a blockchain
state. When a transaction is received by a node and eventually included in a
*block*, a blockchain node shall modify the internal database according to each
transaction type.

A transaction is represented by a JSON document which has the following
context:
```json
{
  "type": "_tx_type_",
  "sender": "_HEX_encoded_account_address_",
  "fee": "_currency_",
  "last_height": "_decimal_number",
  "payload": {}, // tx-specific JSON object
  "signature": {
    "pubkey": "_HEX_encoded_public_key_bytes_",
    "sig_bytes": "_HEX_encoded_signature_bytes_"
  }
}
```
It is irrelevant whether it is in compact or pretty form.

`type` identifies a transaction type. The value `_tx_type_` is one of the
following:
- coins and stakes
    - `transfer`
    - `stake`
    - `withdraw`
    - `delegate`
    - `retract`
- governance
    - `propose`
    - `vote`
- storage
    - `setup`
    - `close`
- parcels
    - `register`
    - `request`
    - `grant`
    - `discard`
    - `cancel`
    - `revoke`
- did
    - `did.claim`
    - `did.dismiss`
    - `did.issue`
    - `did.revoke`
- user-defined coin
    - `issue`
    - `burn`
    - `lock`

`sender` identifies the sender or originator of this transaction. `fee` is
amount of AMO coin expected to get transferred to a block proposer after the
transaction is committed to a block. `last_height` is the last height of AMO
blockchain at the time creating the transaction. `payload` is a JSON object,
which is specific for each transaction type.

`signature` is an ECDSA signature of the sender on the _compact_ JSON
representation of a transaction with all the HEX-encoded string in **upper
case** as the following:

```json
{"type":"transfer","sender":"662E3DD1C6470CFE12C8EDBCE5F44C08E2763753","fee":"0","last_height":"4052","payload":{"to":"614A9F2FC4E6B119D7612C35BC150E33CB38BB40","amount":"100"}}
```

A signed transaction is as the following:
```json
{"type":"transfer","sender":"662E3DD1C6470CFE12C8EDBCE5F44C08E2763753","fee":"0","last_height":"4052","payload":{"to":"614A9F2FC4E6B119D7612C35BC150E33CB38BB40","amount":"100"},"signature":{"pubkey":"04DBCEC2C0F52018606F588713305E1DA49367037281B960F51C46BE64E3144977009A811A865B3CB3331B788147C03853C7920C4C8FB6FFB5B0D435DAEB3F59A4","sig_bytes":"50A8307AAFF6611AE67ADD09EA813F37668072A214230DF375CFA25FB368B0EBD861943661EC690AE0E5D789E738B3C4518F78D768E5E006C9EB53E81821671D"}}
```

**TM:** Tendermint receives transactions via tendermint-specific RPC channel.
For the exact RPC message format, see [AMO Client RPC Specification](rpc.md).

### Transaction payload
A payload format for each transaction type is as the following.

- `transfer` payload:
  ```json
  {
    "to": "_HEX_encoded_account_address_",
    "udc": _udc_id_, // optional
    "amount": "_currency_", // optional
    "parcel": "_HEX_encoded_parcel_id_" // optional
  }
  ```
  where `to` is the recipient of the transfer and mandatory, `amount` is amount
  of AMO coin or user-defined coin, and `parcel` is ID of a parcel to be
  transferred. Either one of `amount` or `parcel` must be present, but not
  both. If `amount` is present there can be another optional field `udc`, which
  is an identifier of a user-defined coin. If both of `amount` and `udc` are
  present, the transfer tx is for transferring user-defined coin. If `amount`
  is present but `udc` is not, the transfer tx is for transferring AMO coin.
  `_udc_id_` must be one of registered user-defined coin ID. `_currency_` is a
  string representation of a decimal number.

- `stake` payload:
  ```json
  {
    "validator": "_HEX_encoded_ed25519_pubkey_",
    "amount": "_currency_"
  }
  ```
  where `validator` is the only public key type other than P256 public key used
  in AMO blockchain protocol. It must be obtained from underlying Tendermint
  node, but in HEX encoding, not Base64 encoding. `amount` is amount of AMO
  coin to be locked as stake.

- `withdraw` payload:
  ```json
  {
    "amount": "_currency_"
  }
  ```
  where `amount` is amount of AMO coin to be withdrawn from stake.

- `delegate` payload:
  ```json
  {
    "to": "_HEX_encoded_account_address_",
    "amount": "_currency_"
  }
  ```
  where `to` is an address of an account which has stakes already and `amount`
  is amount of AMO coin to be delegated.

- `retract` payload:
  ```json
  {
    "amount": "_currency_"
  }
  ```
  where `amount` is amount of AMO coin to be retracted from delegated stake.

- `propose` payload
  ```json
  {
    "draft_id": "_draft_id_",
    "config": {}, // application-specific JSON object
    "desc": "human-readable string describing this draft"
  }
  ```
  where `config` is an optional field which is necessary for a proposal of
  applying of new configuration on-chain. 

- `vote` payload
  ```json
  {
    "draft_id": "_draft_id_",
    "approve": true // boolean
  }
  ```
  where `approve` indicates `sender`'s opinion on `draft_id`; `true` for
  approval or `false` for rejection.

- `setup` payload
  ```json
  {
    "storage": _storage_id_, // integer
    "url": "_url_",
    "registration_fee": "_currency_",
    "hosting_fee": "_currency_"
  }
  ```

- `close` payload
  ```json
  {
    "storage": _storage_id_ // integer
  }
  ```

- `register` payload:
  ```json
  {
    "target": "_HEX_encoded_parcel_id_",
    "custody": "_HEX_encoded_key_custody_",
    "proxy_account": "_HEX_encoded_account_address_", // optional
    "extra": {} // application-specific JSON object, optional
  }
  ```
  where `target` is the id of a parcel currently being registered, `custody` is
  a encrypted key material used to encrypt the data parcel body, and the key
  material is encrypted by the owner(seller)'s public key.

- `request` payload:
  ```json
  {
    "target": "_HEX_encoded_parcel_id_",
    "payment": "_currency_",
    "recipient": "_HEX_encoded_account_address_", // optional
    "dealer": "_HEX_encoded_account_address_", // optional
    "dealer_fee": "_currency_", // optional
    "extra": {} // application-specific JSON object, optional
  }
  ```
  where `target` is the id of a parcel for which the sender wants usage grant,
  `payment` is amount of AMO coin to be collected by the seller, `recipient` is
  the address of a recipient explictly designated to get granted a usage on the
  parcel. In order for `dealer_fee` to work, both of `dealer` and `dealer_fee`
  must be valid.

- `grant` payload
  ```json
  {
    "target": "_HEX_encoded_parcel_id_",
    "recipient": "_HEX_encoded_account_address_",
    "custody": "_HEX_encoded_key_custody_",
    "extra": {} // application-specific JSON object, optional
  }
  ```
  where `target` is the id of a parcel currently being granted, `recipient` is
  the address of a recipient, `custody` is a encrypted key material used to
  encrypt the data parcel body, and the key material is encrypted by the
  buyer's public key.

- `discard` payload
  ```json
  {
    "target": "_HEX_encoded_parcel_id_"
  }
  ```
  where `target` is the id of a parcel currently being discarded.

- `cancel` payload
  ```json
  {
    "target": "_HEX_encoded_parcel_id_",
    "recipient": "_HEX_encoded_account_address_" // optional
  }
  ```
  where `target` is the id of a parcel which the sender requested previously,
  `recipient` is the address of a recipient designated to get granted a usage
  on the parcel.

- `revoke` payload
  ```json
  {
    "target": "_HEX_encoded_parcel_id_",
    "recipient": "_HEX_encoded_account_address_"
  }
  ```
  where `target` is the id of a parcel currently being revoked, `recipient` is
  the address of a buyer which is previously granted a usage on the parcel.

- `did.claim` payload
  ```json
  {
    "target": "_string_",
    "document": {} // DID document in the form of JSON object
  }
  ```
  where `target` is a JSON string conforming to `idchar` in DID syntax.
  `document` value will be stored as a compact representation. When `did.claim`
  tx is received on a previously claimed `target`, `document` will be replaced
  with a new one.

- `did.dismiss` payload
  ```json
  {
    "target": "_string_",
  }
  ```
  effectively removes the DID document from the DID registry.

- `did.issue` payload
  ```json
  {
    "target": "_string_",
    "credential": {} // verifiable credential in the form of JSON object
  }
  ```
  where `target` is a JSON string representing a VC id.
  `credential` value will be stored as a compact representation. When
  `did.issue` tx is received on a previously issued `target`, `credential`
  will replace the old one.

- `did.revoke` payload
  ```json
  {
    "target": "_string_",
  }
  ```
  effectively removes the VC from the VC registry.

- `issue` payload
  ```json
  {
    "udc": _udc_id_,
    "desc": "human-readable string describing this user-defined coin",
    "operators": [
      "_HEX_encoded_account_address_",
      ...
    ],
    "amount": "_currency_"
  }
  ```
  where `operators` is an optional list of operator addresses, and `amount` is
  the amount of UDC balance to be created.

- `lock` payload
  ```json
  {
    "udc": _udc_id_,
    "holder": "_HEX_encoded_account_address_",
    "amount": "_currency_"
  }
  ```
  where `udc` is an identifier of a user-defined coin, and `amount` is the
  amount of UDC coin to be locked.

- `burn` payload
  ```json
  {
    "udc": _udc_id_,
    "amount": "_currency_"
  }
  ```
  where `udc` is an identifier of a user-defined coin, and `amount` is the
  amount of UDC balance to burn.

## Blockchain Data

### State DB
AMO blockchain state is an exact snapshot of all the active data items.
Typically, this state is stored as a key-value database, but the exact method
for managing this database may be different for each implementation. One
obligation is that every blockchain node must be able to calculate the same
`app_hash` for a given block height, and all kinds of implementation must have
the same semantic meaning. This section describes the data format suitable for
calculating `app_hash`.

The state database stores data items in separate logical *state stores*
according to the data type. To distinguish between logical state stores, each
state store has unique prefix for the database key. A prefix is a
human-readable ASCII string, but it is treated as a byte array when
concatenating with the in-store data item key.

### Top-level data
There is a top-level data item not associated with any logical state store. This
item has the key as `config` and the value is a JSON marshaled blockchain
configuration.
```json
{
  "max_validators": 100,
  "weight_validator": 2,
  "weight_delegator": 1,
  "min_staking_unit": "_currency_",
  "blk_reward": "_currency_",
  "tx_reward": "_currency_",
  "penalty_ratio_m": 0.1,
  "penalty_ratio_l": 0.01,
  "laziness_window": 100,
  "laziness_threshold": 90,
  "hibernate_threshold": 10,
  "hibernate_period": 1000,
  "block_binding_window": 100,
  "lockup_period": 3600,
  "draft_open_count": 500000,
  "draft_close_count": 100000,
  "draft_apply_count": 500000,
  "draft_deposit": "_currency_",
  "draft_quorum_rate": 0.1,
  "draft_pass_rate": 0.7,
  "draft_refund_rate": 0.2,
  "upgrade_protocol_height": 1,
  "upgrade_protocol_version": 1
}
```
#### Key's value type and constraint 
| key | value type | value constraint |
|-|-|:-:|
| `max_validators` | uint64 | `> 0` |
| `weight_validator` | float64 | `> 0` |
| `weight_delegator` | float64 | `> 0` |
| `min_staking_unit` | currency | `> 0` |
| `blk_reward` | currency | `>= 0` |
| `tx_reward` | currency | `>= 0` |
| `penalty_ratio_m` | float64 | `> 0` |
| `penalty_ratio_l` | float64 | `> 0` |
| `laziness_window` | int64 | `>= 10000` |
| `laziness_threshold` | int64 | `> 0` |
| `hibernate_threshold` | int64 | `>= 10000` |
| `hibernate_period` | int64 | `> 0` |
| `block_binding_window` | int64 | `>= 10000` |
| `lockup_period` | int64 | `>= 10000` |
| `draft_open_count` | int64 | `>= 10000` |
| `draft_close_count` | int64 | `>= 10000` |
| `draft_apply_count` | int64 | `>= 10000` |
| `draft_deposit` | currency | `>= 0` |
| `draft_quorum_rate` | float64 | `> 0` |
| `draft_pass_rate` | float64 | `> 0` |
| `draft_refund_rate` | float64 | `> 0` |
| `upgrade_protocol_height` | int64 | `> app.state.Height + draft_open_count + draft_close_count + draft_apply_count` |
| `upgrade_protocol_version` | uint64 | `== app.state.ProtocolVersion + 1` |

It is mandatory to restrict proper type and value of configurations in order to
make AMO blockchain protocol keep operating as it has to, even after modifying
their values since genesis block. The currency-related configurations' type is
restricted to `string` as it can store values without limit. Even though it is
highly recommended to use `uint64` on configurations for its better space
availability than `int64`, `laziness_window`, `block_binding_window`,
`lockup_period`, `draft_*_count`, and `upgrade_protocol_height` have to use
`int64` as it is an tendermint-dependant configuration.

### State stores
There are 12 default state stores and optional UDC(user-defined coin) balance
and balance lock stores.

| tier | category | store | prefix |
|-|-|-|-|
| 0 | fungible asset | AMO coin balance | `balance:` |
| 0 | fungible asset | stake | `stake:` |
| 0 | fungible asset | delegate | `delegate:` |
| 0 | maintenance | hibernate | `hibernate:` |
| 1 | governance | draft | `draft:` |
| 1 | governance | vote | `vote:` |
| 2 | maintenance | storage | `storage:` |
| 2 | non-fungible asset | parcel | `parcel:` |
| 2 | non-fungible asset | request | `request:` |
| 2 | non-fungible asset | usage | `usage:` |
| 2 | non-fungible asset | did | `did:` |
| 3 | maintenance | UDC | `udc:` |
| 3 | maintenance | UDC balance lock | `udclock:<udc_id>:` |
| 3 | fungible asset | UDC balance | `balance:<udc_id>:` |

Tier 0 items are essential for the operations of a DPoS-based blockchain. Tier
1 items are important as much as the tier 0 items, but the chain may be still
called a functional blockchain without them. Tier 2 items defines the core
business data items, while tier 3 items are pretty much optional.

- default coin balance
    - key: `_account_address_`
    - value: JSON string `"_currency_"`
    - key is the owner of a coin balance
- stake
    - key: `_account_address_`
    - value: compact representation of a JSON object
      ```json
      {
        "validator": "_HEX_encoded_ed25519_pubkey_",
        "amount": "_currency"
      }
      ```
    - key is the sender of a stake tx
- delegate
    - key: `_account_address_`
    - value: compact representation of a JSON object
      ```json
      {
        "delegatee": "_HEX_encoded_accont_address_",
        "amount": "_currency_"
      }
      ```
    - key is the sender of a delegate tx
    - **NOTE:** For delegate store, a key to the database is just
      `_account_address_`, instead of a concatenation of holder address and
      delegatee address. This means that a user can have only one delegated
      stake. In other words, a user cannot delegate his/her stakes to
      **multiple** delegatees. While an AMO-compliant node can freely choose
      the actual database implementation, this constraint must be enforced in
      any way. An implementor may choose to keep this `_account_address_` as a
      unique key, or use more loose database implementation with an application
      code or a wrapper layer to keep this constraint on top of it.
- hibernate
    - key: `_validator_address_`
    - value: compact representation of a JSON object
      ```json
      {
        "start": _block_height_,
        "end": _block_height_
      }
      ```
- draft
    - key: `_draft_id_`(big-endian)
    - value: compact representation of a JSON object
      ```json
      {
        "proposer": "_HEX_encoded_account_address_",
        "config": {},
        "desc": "_human_readable_string_describing_this_draft_",
        "open_count": "_decimal_number_",
        "close_count": "_decimal_number_",
        "apply_count": "_decimal_number_",
        "deposit": "_currency_",
        "tally_approve": "_currency_",
        "tally_reject": "_currency_"
      }
      ```
    - `config` keys should be a subset of the top-level `config` item. The
      values may be omitted if they should remain the same. There should be no
      multiple live drafts having config change items conflicting with each
      other.
    - `*_count` control overall voting process until the draft being passed and
      applied to the blockchain configuration. They are initialized according
      to the configuration at the time of being proposed. `open_count` is
      decremented at each block progress, and when it reaches zero
      `close_count` is decremented afterwards. When `close_count` reaches zero
      and the vote summary is _approval_, then `apply_count` is decremented
      until the new configuration is applied.
    - `tally_*` fields count votes cast upon this draft. `tally_approve` and
      `tally_reject` are as the names imply.
- vote
    - key: `_draft_id_`(big-endian) + `_account_address_`
    - value: compact representation of a JSON object
      ```json
      {
        "approve": true // boolean 
      }
      ```
- storage
    - key: `_storage_id_`(big-endian)
    - value: compact representation of a JSON object
      ```json
      {
        "owner": "_HEX_encoded_account_address_",
        "url": "_url_",
        "registration_fee": "_currency_",
        "hosting_fee": "_currency_",
        "active": _bool_
      }
      ```
- parcel
    - key: `_parcel_id_`
    - value: compact representation of a JSON object
      ```json
      {
        "owner": "_HEX_encoded_account_address_",
        "custody": "_HEX_encoded_key_custody_",
        "proxy_account": "_HEX_encoded_account_address_",
        "extra": {} // application-specific JSON object
      }
      ```
    - key is the `target` of a register tx
    - `owner` is the sender of a register tx
- request
    - key: `_account_address_` + `_parcel_id_`
    - value: compact representation of a JSON object
      ```json
      {
        "payment": "_currency_",
        "agency": "_HEX_encoded_account_address_", // optional
        "dealer": "_HEX_encoded_account_address_", // optional
        "dealer_fee": "_currency_", // optional
        "extra": {} // application-specific JSON object
      }
      ```
    - key is a concatenation of (sender or `recipient`) and `target` of a
      request tx
- usage
    - key: `_account_address_` + `_parcel_id_`
    - value: compact representation of a JSON object
      ```json
      {
        "custody": "_HEX_encoded_key_custody_",
        "extra": {} // application-specific JSON object
      }
      ```
    - key is a concatenation of `recipient` and `target` of a grant tx
- did
    - key: `_did_address_`
    - value: compact representation of a JSON object
    ```json
    {
      "owner": "_HEX_encoded_account_address_",
      "document": {} // DID documentn in the form of JSON object
    }
    ```
- udc(user-defined coin)
    - key: `_udc_id_`
    - value: compact representation of a JSON object
      ```json
      {
        "owner": "_HEX_encoded_account_address_",
        "desc": "human-readable string describing this user-defined coin",
        "operators": [
          "_HEX_encoded_account_address_",
          ...
        ],
        "total": "_currency_"
      }
      ```
    - key is `id` of an issue tx
    - `owner` is the sender of an *initial* issue tx
- udc balance
    - key: `_account_address_`
    - value: JSON string `"_currency_"`
- udc balance lock
    - key: `_account_address_`
    - value: JSON string `"_currency_"`
    - UDC balance of an account cannot be lowered under this value via transfer
      tx. This lock value may be higher than the UDC balance of an account at
      the time of processing lock tx

### Merkle tree and app hash
Although the internal state DB is composed of top-level data items and several
logical state stores, its actual form is a linear key-value database. In
viewpoint of state management, it suffices to manage this database in any form
as long as the contents are equivalent. However, in order to interact with the
underlying Tendermint consensus engine, we need to calculate *app hash* from
the state DB contents. Every blockchain node must be able to calculate the same
app hash from the equivalent state DB contents. To calculate this app hash, we
assume that the database is stored as a Merkle tree.

Every data item in the database is stored as a leaf node in a Merkle tree with
the key as the concatenation of the prefix and in-store key. The leaf nodes are
sorted by the key and they are labeled with a hash derived from `hash(key +
value)`. A pair of leaf nodes generates a one-level higher inner node labeled
with `hash(ln1_hash + ln2_hash)`. In the similar way, another one-level higher
inner node is added to the merkle tree with the label of `hash(in1_hash +
in2_hash)`. The above process is repeated until only one single root node
appears at the top of the merkle tree. The resulting app hash is the hash label
of the root node.

**TM:** This app hash is calculated every time a new block is committed and
stored as *app hash* in the *next* block. App hash is to provide an evidence
that every blockchain node hash the same state DB contents for a given block
height.

## Operations
This section describes how the AMO blockchain state is changed when a
transaction is included in a block or a block is completed. There shall be no
other state change than described in this section.

In following subsections, `blk.incentive` is accumulated from the beginning of
a block until the end of a block. When completing a block, this incentive is
distributed among the validator who produced a block and the users who
delegated stakes to the validator.

**TM:** These operations are implemented by `DeliverTx` and `EndBlock` method
in the ABCI application.

### Transferring coin
Upon receiving a `transfer` transaction from an account with an asset type of
AMO coin or UDC coin, an AMO blockchain node performs a validity check and
transfers coins from sender's balance to recipient's balance when the
transaction is valid.

NOTE: When the asset type is `parcel`, see
[Transferring data ownership](#transferring-data-ownership).

1. validity check
    1. `tx.amount` > `0`
    1. `sender.balance` &ge; `tx.amount` + `tx.fee`
1. state change
    1. `sender.balance` &larr; `sender.balance` - `tx.amount` - `tx.fee`
    1. `to.balance` &larr; `to.balance` + `tx.amount`
    1. `blk.incentive` &larr; `blk.incentive` + `tx.fee`

When optional parameter `udc` is given, the operation is changed as follows.
1. validity check
    1. UDC id `<udc>` is registered
    1. `tx.amount` > `0`
    1. `<udc>.sender.balance` &ge; `<udc>.sender.lock` + `tx.amount`
    1. `sender.balance` &ge; `tx.fee`
1. state change
    1. `<udc>.sender.balance` &larr; `<udc>.sender.balance` - `tx.amount`
    1. `<udc>.to.balance` &larr; `<udc>.to.balance` + `tx.amount`
    1. `sender.balance` &larr; `sender.balance` - `tx.fee`
    1. `blk.incentive` &larr; `blk.incentive` + `tx.fee`

### Staking coin
Upon receiving a `stake` transaction from an account, an AMO blockchain node
performs a validity check and locks requested coins to stake store and
decreases the sender's balance when the transaction is valid.

1. validity check
    1. `tx.amount` > `0`
    1. `tx.amount % config.minimum_staking_unit` == `0` (check staking unit 
       restriction)
    1. `sender.balance` &ge; `tx.amount` + `tx.fee`
    1. There is no other account `holder` having `holder.stake.validator` ==
       `tx.validator`
    1. `sender.stake.validator` == `tx.validator` if `sender.stake` exists
1. state change
    1. `sender.balance` &larr; `sender.balance` - `tx.amount` - `tx.fee`
    1. `sender.stake.amount` &larr; `sender.stake.amount` + `tx.amount`
    1. `sender.stake.locked_height` &larr; `config.lockup_period`
    1. `blk.incentive` &larr; `blk.incentive` + `tx.fee`

Upon receiving a `withdraw` transaction from an account, an AMO blockchain node
performs a validity check and relieves requested coins from `stake` store and
increases the account's balance when the transaction is valid.

1. validity check
    1. `tx.amount` > `0`
    1. `sender.balance` &ge; `tx.fee`
    1. `sender.stake.unlocked` == `true`
    1. `sender.stake.amount` &ge; `tx.amount`
    1. `sender.stake.amount` &gt; `tx.amount` if this account is a delegatee
       for any of delegated stakes
    1. `sender.stake.delegate` is empty
1. state change
    1. `sender.stake.amount` &larr; `sender.stake.amount` - `tx.amount`
    1. `sender.balance` &larr; `sender.balance` - `tx.fee` + `tx.amount`
    1. `blk.incentive` &larr; `blk.incentive` + `tx.fee`

**TODO:** need rounding? or currency to stake ratio?

**Stake Lock-up**

This feature locks a newly added stake for a certain period of time. The time
is measured in terms of number of blocks. If a stake is set at the block height
`h`, the stake can be withdrawn after the block height reaches `h + l`, where
`l` is the pre-configured lock-up period.

Upon receiving a `stake` transaction from an account, an AMO blockchain node
records the stake in `LockedStake` with `l`. Then, the stake's `l` decreases by
1 block height per block creation. When `l` becomes `0`, the stake gets removed
from `LockedStake` and put into `UnlockedStake`.

**Block Progress(Creation) Condition**

As tendermint's `create_empty_blocks` config is set to `false` on an AMO
blockchain node, the block is progressed only if there is a change of
`appHash`, the root hash value of `State` merkle tree. The conditions in which
the `appHash` can change are as follows:

- Successfully delivered(processed) transactions
- Stakes in `LockedStake` 

Even though there is no transaction to process on an AMO blockchain node, the
`appHash` can change. The lock-period `l` of locked stakes decrease by 1 block
height and it is written in `State`, resulting in the change of `appHash`.

### Delegating stake
There may be users who have the intention to participate in the block
production but don't have enough stake value or computing power to competent in
the validator selection race. In this case, a user can delegate his/her stake
to a more competent validator.

Upon receiving a `delegate` transaction from an account, an AMO blockchain node
performs a validity check and locks requested coins to `delegate` store and
decreases the account's balance when the transaction is valid.

1. validity check
    1. `tx.amount` > `0`
    1. `tx.amount` % `config.minimum_staking_unit` == `0` (check staking unit
       restriction)
    1. `sender.balance` &ge; `tx.fee` + `tx.amount`
    1. `tx.to` address already has a positive stake in `stake` store
    1. the `sender` has no previous delegatee or `tx.to` is the same as the
       previous delegatee
1. state change
    1. `sender.balance` &larr; `sender.balance` - `tx.fee` - `tx.amount`
    1. `blk.incentive` &larr; `blk.incentive` + `tx.fee`
    1. `sender.delegate.amount` &larr; `sender.delegate.amount` + `tx.amount`

Upon receiving a `retract` transaction from an account, an AMO blockchain node
performs a validity check and relieves requested coins from `delegate` store
and increases the account's balance when the transaction is valid.

1. validity check
    1. `tx.amount` > `0`
    1. `sender.balance` &ge; `tx.fee`
    1. `sender.delegate.amount` &ge; `tx.amount`
1. state change
    1. `sender.delegate.amount` &larr; `sender.delegate.amount` - `tx.amount`
    1. `sender.balance` &larr; `sender.balance` - `tx.fee` + `tx.amount`
    1. `blk.incentive` &larr; `blk.incentive` + `tx.fee`

**NOTE:** `sender.delegate` is a `stake` value in the `delegate` store where
the `address` is the sender account.

### Proposing and voting
When it is necessary to modify the configuration of AMO blockchain without
hard-forking the chain, one of the validators can propose a draft containing
the configuration to get applied with its description and deposit. Then, the
validators vote for or against it. For the draft to get processed further after
the vote is closed, the draft must have a quorum for voting. If not, the votes
for draft are ignored no matter what the final result of votes is. On the other
hand, if quorum is met, the draft would get applied or not, according to its
final result. Also, if turnout of voters is below refund rate, the draft
deposit is distributed among the validators who participate in voting.
Otherwise, it is returned to the proposer.

Upon receiving a `propose` transaction from an account, an AMO blockchain node
performs a validity check and add a record in `draft` store.

1. validity check
    1. there is no other draft in progress
    1. there is no record having `tx.draft_id` as a key in `draft` store
    1. `sender` is one of `blk.validators`
    1. `sender.balance` &ge; `config.draft_deposit` + `tx.fee`
    1. `tx.draft_id` == `state.latest_draft_id` + 1
1. state change
    1. add new record having `tx.draft_id` as a key in `draft` store
    1. `sender.balance` &larr; `sender.balance` - `config.draft_deposit` -
       `tx.fee`
    1. `blk.incentive` &larr; `blk.incentive` + `tx.fee`

Upon receiving a `vote` transaction from an account, an AMO blockchain node
performs a validity check and add a record in `vote` store.

1. validity check
    1. `tx.draft_id` is in progress
    1. `sender` != `draft.proposer`
    1. there is no record having `tx.draft_id`+`sender` as a key in `vote`
       store
    1. `sender` is one of `blk.validators`
    1. `sender.balance` &ge; `tx.fee`
1. state change
    1. add new record having `tx.draft_id`+`sender` as a key in `vote` store 
    1. `sender.balance` &larr; `sender.balance` - `tx.fee`
    1. `blk.incentive` &larr; `blk.incentive` + `tx.fee`

Upon `close_count` reaches zero and the vote gets closed, an AMO blockchain
node calculates and updates `draft.tally_*` values.

1. state change
    1. for `validator` in `validators`: `draft.tally_quorum` &larr;
       `draft.tally_quorum` + `validator.effective_stake`
    1. `draft.tally_quorum` &larr; `draft.tally_quorum` *
       `config.draft_quorum_rate`
    1. `draft.tally_approve` &larr; `draft.tally_approve` +
       `draft.proposer.effective_stake`
    1. for `vote` in `draft.votes`: `draft.tally_approve` &larr;
       `draft.tally_approve` + `vote.voter.effective_stake`, if `vote.approve`
       is `true`
    1. for `vote` in `draft.votes`: `draft.tally_reject` &larr;
       `draft.tally_reject` + `vote.voter.effective_stake`, if `vote.approve`
       is `false`

### Registering storage
In order to register a data parcel in AMO blockchain, there must be an already
registered data storage in the blockchain.

Upon receiving a `setup` transaction from an account, an AMO blockchain node
performs a validity check and add or update an item in `storage` store.

1. validity check
    1. `sender.balance` &ge; `tx.fee`
    1. `prev.owner` == `tx.sender` if `prev` with `prev.id` == `tx.storage`
       exists in `storage` store
1. state change
    1. add new record or update existing record having `tx.storage` as a key
       in `storage` store
    1. `sender.balance` &larr; `sender.balance` - `tx.fee`
    1. `blk.incentive` &larr; `blk.incentive` + `tx.fee`

Upon receiving a `close` transaction from an account, an AMO blockchain node
performs a validity check and remove a record from `store` store.

1. validity check
    1. `sender.balance` &ge; `tx.fee`
    1. `prev` with `prev.id` == `tx.storage` exists in `storage` store
    1. `prev.owner` == `tx.sender`
1. state change
    1. remove record having `tx.storage` as a key from `storage` store
    1. `sender.balance` &larr; `sender.balance` - `tx.fee`
    1. `blk.incentive` &larr; `blk.incentive` + `tx.fee`

### Registering data
Upon receiving a `register` transaction from an account, an AMO blockchain node
performs a validity check and add a new record with its extra information in
`parcel` store.

1. validity check
    1. extract storage ID `storage` from `tx.target`
    1. `storage` should exist in storage store and `storage.active` should be
       true
    1. `tx.target` should NOT exist in `parcel` store
    1. `sender.balance` &ge; `storage.registration_fee` + `tx.fee`
1. state change
    1. add new record having `tx.target` as a key in `parcel` store
    1. `sender.balance` &larr; `sender.balance` - `storage.registration_fee` -
       `tx.fee`
    1. `storage.owner.balance` &larr; `storage.owner.balance` +
       `storage.registration_fee`
    1. `blk.incentive` &larr; `blk.incentive` + `tx.fee`

Upon receiving a `discard` transaction from an account, an AMO blockchain node
performs a validity check and remove record in `parcel` store.

1. validity check
    1. `tx.target` should exist in `parcel` store
    1. `sender.balance` &ge; `tx.fee`
    1. `sender` == `tx.target.owner` or `sender` == `tx.target.proxy_account`
1. state change
    1. remove record having `tx.target` as a key in `parcel` store
    1. `sender.balance` &larr; `sender.balance` - `tx.fee`
    1. `blk.incentive` &larr; `blk.incentive` + `tx.fee`

**NOTE:** `proxy_account` refers to an account which has a `owner`-equivalent
permission to control over `tx.target` record.

### Requesting data
Upon receiving a `request` transaction from an account, an AMO blockchain node
performs a validity check and add a new record with its extra information in
`request` store.

1. validity check
    1. `tx.target` should exist in `parcel` store
    1. if `tx.recipient` is valid, then<br/>
       form request ID `request` from `tx.recipient`+`tx.target`
    1. else<br/>
       form request ID `request` from `sender`+`tx.target`
    1. `request` should NOT exist in `request` store
    1. `request` should NOT exist in `usage` store
    1. `tx.recipient` &ne; `tx.target.owner`
    1. if `tx.dealer` and `tx.dealer_fee` is valid, then<br/>
       `sender.balance` &ge; `tx.fee` + `tx.payment` + `tx.dealer_fee`
    1. else<br/>
       `sender.balance` &ge; `tx.fee` + `tx.payment`
1. state change
    1. if `tx.recipient` is valid, then<br/>
       add new record having `tx.recipient`+`tx.target` as a key in `request`
       store where `request.agency` is `sender`
    1. else<br/>
       add new record having `sender`+`tx.target` as a key in `request` store
       where `request.agency` is left empty
    1. if `tx.dealer` and `tx.dealer_fee` is valid, then<br/>
       `sender.balance` &larr; `sender.balance` - `tx.fee` - `tx.payment` -
       `tx.dealer_fee`
    1. else<br/>
       `sender.balance` &larr; `sender.balance` - `tx.fee` - `tx.payment`
    1. `blk.incentive` &larr; `blk.incentive` + `tx.fee`

Upon receiving a `cancel` transaction from an account, an AMO blockchain node
performs a validity check and remove record in `request` store.

1. validity check
    1. `tx.target` should exist in `parcel` store
    1. if `tx.recipient` is valid, then<br/>
       form request ID `request` from `tx.recipient`+`tx.target`
    1. else<br/>
       form request ID `request` from `sender`+`tx.target`
    1. `request` should exist in `request` store
    1. if `tx.recipient` is valid, then<br/>
       `sender` == `request.agency`
    1. `sender.balance` &ge; `tx.fee`
1. state change
    1. if `tx.recipient` is valid, then<br/>
       delete record having id as `tx.recipient` + `tx.target` in `request`
       store
    1. else<br/>
       delete record having id as `sender` + `tx.target` in `request` store
    1. if `tx.dealer` and `tx.dealer_fee` is valid, then<br/> `sender.balance`
       &larr; `sender.balance` - `tx.fee` + `request.payment` +
       `request.dealer_fee`
    1. else<br/> `sender.balance` &larr; `sender.balance` - `tx.fee` +
       `tx.target.payment`
    1. `blk.incentive` &larr; `blk.incentive` + `tx.fee`

**NOTE:** `payment` refers to the amount of coins `sender` is willing to pay
for `tx.target` to `tx.target.owner`.

### Granting data
Upon receiving a `grant` transaction from an account, an AMO blockchain node
performs a validity check and add a new record with its extra information in
`usage` store.

1. validity check
    1. `tx.target` should exist in `parcel` store
    1. `tx.target` should exist in `request` store
    1. `tx.target` should NOT exist in `usage` store
    1. `sender` == `tx.target.owner` or `sender` == `tx.target.proxy_account`
    1. extract storage ID `storage` from `tx.target`
    1. `storage` should exist in storage store and `storage.active` should be
       true
    1. find `request` having id as `tx.recipient` + `tx.target` in `request`
       store
    1. if `sender` == `tx.target.owner`, then <br/>
       `sender.balance` + `request.payment` &ge; `storage.hosting_fee` +
       `tx.fee`
    1. if `sender` &ne; `tx.target.owner`, then <br/>
       `sender.balance` &ge; `tx.fee` and <br/>
       `tx.target.owner.balance` + `request.payment` &ge; `storage.hosting_fee`
1. state change
    1. delete record having id as `tx.recipient` + `tx.target` in `request`
       store
    1. add new record having id as `tx.recipient` + `tx.target` in `usage`
       store
    1. `tx.target.owner.balance` &larr; `tx.target.owner.balance` +
       `tx.target.payment` - `storage.hosting_fee`
    1. `storage.owner.balance` &larr; `storage.owner.balance` +
       `storage.hosting_fee`
    1. `request.dealer.balance` &larr; `request.dealer.balance` +
       `request.dealer_fee`
    1. `sender.balance` &larr; `sender.balance` - `tx.fee`
    1. `blk.incentive` &larr; `blk.incentive` + `tx.fee`

Upon receiving a `revoke` transaction from an account, an AMO blockchain node
performs a validity check and remove record in `usage` store.

1. validity check
    1. `tx.target` should exist in `parcel` store
    1. `tx.target` should exist in `usage` store
    1. `sender` == `tx.target.owner` or `sender` == `tx.target.proxy_account`
    1. `sender.balance` &ge; `tx.fee`
1. state change
    1. delete record having id as `tx.recipient` + `tx.target` in `usage` store
    1. `sender.balance` &larr; `sender.balance` - `tx.fee`
    1. `blk.incentive` &larr; `blk.incentive` + `tx.fee`

### Transferring data parcel ownership

Upon receiving a `transfer` transaction from an account with an asset type of `parcel`, an AMO blockchain node performs a validity check and transfers the ownership of the parcel to the recipient when the transaction is valid.

NOTE: When the asset type is AMO coin or UDC coin, see
[Transferring coin](#transferring-coin).

1. validity check
    1. `tx.parcel.owner` == `tx.sender`
    1. `sender.balance` &ge; `tx.fee`
1. state change
    1. `tx.parcel.owner` &larr; `tx.to`
    1. `sender.balance` &larr; `sender.balance` - `tx.fee`
    1. `blk.incentive` &larr; `blk.incentive` + `tx.fee`

### Creating (Claiming) DID document

*NOTE: `claim` and `dismiss` tx in protocol v5 are deprecated.*

Upon receiving a `did.claim` transaction from an account, an AMO blockchain
node performs a validity check and add a new record or update an existing
record in `did` store.

1. validity check
    1. if `tx.target` already exists in `did` store as `doc`, perform
       permission and validity check:
        1. `tx.target` must be the same as `doc.id`.
        1. concatenation of `did:amo:` and `tx.sender` must be either `doc.id`
           or `doc.controller`.
        1. `tx.document` and `doc` must have the same `id`,
           `verificationMethod` and `authentication` properties.
           (`assertionMethod` may change.)
    1. if `tx.target` does not exist in `did` store, perform a validity check:
        1. `tx.target` must be the same as `tx.document.id`.
        1. concatenation of `did:amo:` and `tx.sender` must be
           `tx.document.id`.
        1. `tx.document.verificationMethod[0].id` must be `tx.document.id` +
           `#keys-1`.
        1. `tx.document.verificationMethod[0]` must be of a `JsonWebKey2020`
           format, and a derived address from
           `tx.document.verificationMethod[0]` must be `tx.document.id`.
    1. `sender.balance` &ge; `tx.fee`
1. state change
    1. add new record or replace the record with key `tx.target`
    1. `sender.balance` &larr; `sender.balance` - `tx.fee`
    1. `blk.incentive` &larr; `blk.incentive` + `tx.fee`

Upon receiving a `did.dismiss` transaction from an account, an AMO blockchain
node performs a validity check and remove record from `did` store.

1. validity check
    1. `tx.target` should exist in `did` store as `doc`.
    1. concatenation of `did:amo:` and `tx.sender` must be either `doc.id` or
       `doc.controller`.
    1. `sender.balance` &ge; `tx.fee`
1. state change
    1. remove record with key `tx.target` from `did` store
    1. `sender.balance` &larr; `sender.balance` - `tx.fee`
    1. `blk.incentive` &larr; `blk.incentive` + `tx.fee`

### Creating (Issuing) VC

*NOTE: In the context of verifiable credential, an **issuer** is supposed to
create, in other word, **issue** a verifiable credential.*

Upon receiving a `did.issue` transaction from an account, an AMO blockchain
node performs a validity check and add a new record or update an existing
record in `vc` store.

1. validity check
    1. if `tx.target` already exists in `vc` store as `cred`, perform
       permission and validity check:
        1. `tx.target` must be the same as `cred.id`.
        1. concatenation of `did:amo:` and `tx.sender` must be `cred.issuer`.
        1. `tx.credential` and `cred` must have the same `id` and `issuer`
           properties.
        1. `tx.credential.issued` must be more recent than `cred.issued`.
    1. if `tx.target` does not exist in `vc` store, perform a validity check:
        1. `tx.target` must be the same as `tx.credential.id`.
        1. concatenation of `did:amo:` and `tx.sender` must be
           `tx.credential.issuer`.
    1. `sender.balance` &ge; `tx.fee`
1. state change
    1. add new record or replace the record with the key `tx.garget`.
    1. `sender.balance` &larr; `sender.balance` - `tx.fee`
    1. `blk.incentive` &larr; `blk.incentive` + `tx.fee`

Upon receiving a `did.revoke` transaction from an account, an AMO blockchain node performs a validity check and remove a record from `vc` store.

1. validity check
    1. `tx.target` should exist in `vc` store as `cred.
    1. concatenation of `did:amo:` and `tx.sender` must be `cred.issuer`.
    1. `sender.balance` &ge; `tx.fee`
1. state change
    1. remove record with key `tx.target` from `vc` store
    1. `sender.balance` &larr; `sender.balance` - `tx.fee`
    1. `blk.incentive` &larr; `blk.incentive` + `tx.fee`

### Managing user-defined coin
In order to transfer user-defined coin balance in AMO blockchain, there must be
an user-defined coin registry in the blockchain.

Upon receiving an `issue` transaction from an account, an AMO blockchain node
performs a validity check and add a new record in `udc` store.

1. validity check
    1. if `udc` exists having `tx.udc` as a key in udc store, then
        1. `sender` == `udc.owner` or `sender` should be one of `udc.operators`
    1. else
        1. `sender` is one of `blk.validators`
    1. `sender.balance` &ge; `tx.fee`
1. state change
    1. if `udc` exists in udc store, then
        1. `udc.operators` &larr; `tx.operators`
        1. `udc.desc` &larr; `tx.desc`
        1. `udc.total` &larr; `udc.total` + `tx.amount`
    1. else add a new record `udc` with the following
        1. `udc.owner` &larr; `sender`
        1. `udc.operators` &larr; `tx.operators`
        1. `udc.desc` &larr; `tx.desc`
        1. `udc.total` &larr; `tx.amount`
    1. `<udc>.sender.balance` &larr; `<udc>.sender.balance` + `tx.amount`
    1. `sender.balance` &larr; `sender.balance` - `tx.fee`
    1. `blk.incentive` &larr; `blk.incentive` + `tx.fee`

Upon receiving a `lock` transaction from an account, an AMO blockchain node
performs a validity check and add or update a record in udc balance lock store.

1. validity check
    1. `udc` exists having `tx.udc` as a key in udc store
    1. `sender` == `udc.owner` or `sender` should be one of `udc.operators`
    1. `tx.amount` > 0
    1. `sender.balance` &ge; `tx.fee`
1. state change
    1. if `tx.amount` > 0, then<br/>
       add new `<udc>.holder.lock` or update existing `<udc>.holder.lock` in
       udc balance lock store
    1. else<br/>
       delete  existing `<udc>.holder.lock` from udc balance lock store
    1. `sender.balance` &larr; `sender.balance` - `tx.fee`
    1. `blk.incentive` &larr; `blk.incentive` + `tx.fee`

Upon receiving a `burn` transaction from an account, an AMO blockchain node
performs a validity check and reduce sender's designated UDC balance.

1. validity check
    1. `udc` exists having `tx.udc` as a key in udc store
    1. `tx.amount` > 0
    1. `<udc>.sender.balance` &ge; `<udc>.sender.lock` + `tx.amount`
    1. `sender.balance` &ge; `tx.fee`
1. state change
    1. `<udc>.sender.balance` &larr; `<udc>.sender.balance` - `tx.amount`
    1. `sender.balance` &larr; `sender.balance` - `tx.fee`
    1. `blk.incentive` &larr; `blk.incentive` + `tx.fee`

### Completing Block
After processing state changes triggered by users' transactions in
`DeliverTx()`, the nodes complete a block in `EndBlock()` by applying
additional state changes described in this section.

#### Distributing Incentive

`tx.fee` is collected while transactions are processed and it gets included in
`blk.incentive`. Then, `blk.incentive` is distributed among the stakers and the
delegators at the end of block creation. The process is explained in [incentive
distribution](#distribution) section, in more detail.

#### Validator hibernation

A lazy validator is a validator who is in the validator set until a block, but
did not vote in the consensus process for the block. We say the validator
missed the block. If a validator missed **consecutive** `blks_lazy` blocks and
`blks_lazy >= hibernate_threshold`, it is removed from the validator set for
the next `hibernate_period` blocks. New record is created in `hibernate` store
with `start` to be the current height, and `end` current height plug
`hibernate_period`. We say the validator enters hibernation state. If a
validator missed consecutive `blks_lazy` blocks and `blks_lazy <
hibernate_threshold` but wakes up and comes back to the consensus process,
`blks_lazy` resets to zero.

If a validator stayed hibernation state for `blks_hib` blocks and `blks_hib >=
hibernate_period`, i.e. `hibernate.address.end = current_height`, then
`hibernate.address` is removed and the validator leaves the hibernation state
and may be included in the validator set again.

#### Validator Updates

TM validator set may change along with the progress of the chain. There are two
reasons for the change:

- Validator hibernation state change
- Stake or effective stake change

In order to select new set of validators, first top `n_val` accounts with
highest *effective stakes* are selected excluding accounts associated to the
validators in hibernation state. Extract validator public keys for the
accounts, and inform the list of validator public keys to Tendermint layer.

**NOTE:** Effective stake value is the sum of his/her own stake in the `stake`
store and all items in the `delegate` store having the same `delegatee` field
as the account address in question

**NOTE:** `n_val` is a global parameter fixed across nodes and blocks (and so
the time). So, it shall be set at the genesis time.

**TM:** New list of validator pubkeys shall be transferred to the Tendermint
daemon via `EndBlock` response. Each validator has the voting power in
proportion to the effective stake value.

**TM:** According to the official documentation of tendermint and several
experimental results, to maintain a blockchain network, it is mandatory for
over 2/3 validator(MUST-ONLINE) nodes to be online. Also, the voting power of a
validator node matters to the ratio of **MUST-ONLINE** nodes. That is, stopping
validator nodes of which the sum of voting power is over 1/3 breaks the
consensus algorithm of tendermint and results in the interruption of generating
blocks on the chain.

##### Voting power calculation

**TM:** In tendermint, a **voting power** has a similar role as a stake in PoS
or DPoS consensus mechanism. One limitation is that sum of voting powers of all
validators must not exceed the value `MaxTotalVotingPower`, which is 2^60 - 1.
When we use one-to-one relation between stake value and voting power, exceeding
this max limit is not very likely, but possible anyway. So, the validator set
update mechanism must adjust voting power of each validator, so that total sum
of voting power does not exceed `MaxTotalVotingPower`:

1. For each validator `Val_i`, set voting power `vp_i` to be `stake` of
   `Val_i`.
1. Calculate `TotalVotingPower`, which is the sum of `vp_i`s of all validators
   in the new validator set.
1. `adjFactor` &larr; 0 (use this as a persistent factor)
1. While `TotalVotingPower` > `MaxTotalVotingPower`
    1. `adjFactor` &larr; `adjFactor` + 1
    1. `TotalVotingPower` &larr; `TotalVotingPower` / 2
    <br/>(implemented as right-shift)
    1. For each validator `Val_i`, `vp_i` &larr; `vp_i` / 2
    <br/>(implemented as right-shift)

**NOTE:** When `vp_i` reaches to zero, then `Val_i` shall be removed from the
new validator set.

#### Penalizing convicts
At the beginning of block creation `BeginBlock()`, AMO ABCI app receives a list
of convicts from tendermint. The convicts get penalized in `EndBlock()` for its
malicious attempts to harm the blockchain network. The detailed penalization
process is explained in [penalty](#penalty) section.

## Incentive 
**TM:** Tendermint provides a block information, in `BeginBlock()` method which
is called at the beginning of a block creation, including a block proposer
address. This address is derived from the validator pubkey who proposes the
block. In AMO ABCI app, we can look up the original stake holder in the `stake`
store having the same validator pubkey.

Incentive refers to the sum of a block reward and transaction fees. The fees of
transactions which are successfully verified(delivered) by the block proposer
are accumulated and then transferred to the stake holder at the end of a block
creation in `EndBlock()`.

### Calculation
A stake holder who proposes a block receives an incentive. This is the only
step in which there is a state change in `balance` store without involving any
transaction:

`R` &larr; `b_reward` + `n_delivered_txs` \* `tx_reward`<br/>
`I` &larr; `R` + `acc_fee`

where `R` is the final block reward, `b_reward` a block reward rate,
`n_delivered_txs` the number of delivered transactions in the block,
`tx_reward` a transaction reward rate, `I` the final incentive and `acc_fee`
the accumulated fee.

### Distribution
When the incentive is `I`, this incentive shall be distributed among the stake
holder and the delegated stake holders. The distribution mechanism is as the
following:

1. `wStakes` &larr; `w_val` \* `stake_0` (stake of the proposer)
1. For each delegated stake `stake_i`, `wStakes` &larr; `wStakes` + `w_ds` \*
   `stake_i`
1. Calculate the incentive for the proposer `I_0` &larr; `I` \* `w_val` \*
   `stake_0` / `wStakes`.
1. For each delegated stake holder, calculate the incentive for `i`-th
   delegated stake, `I_i` &larr; `I` \* `w_ds` \* `stake_i` / `wStakes`.

where `w_val` is the validator stake weight, and `w_ds` is the delegated stake
weight.

**TODO:** Eliminate ambiguity in float number arithmetic.

**TODO:** Take care of overflow situation.

## Penalty
To maintain the DPoS blockchain as healthy as possible, it is essential to
encourage block validators to participate in creating and verifying blocks with
incentive, but also to impose responsibilities on their misbehavior with
penalty. The penalty shall be distributed among the stake holder and the
delegated stake holders according to the distribution mechanism presented in
[Incentive Distribution](#distribution).

The types of abnormal behavior and parameters are defined as follows:

- Convict
  - Malicious Validator: `PenaltyRatioM`
  - Lazy Validator: `PenaltyRatioL`

### Evidence
**TM:** The evidence of validators' misbehavior is provided by Tendermint in
`BeginBlock()` method which is called at the beginning of a block creation.
Tendermint supports currently only a single type of evidence, the
`DuplicateVoteEvidence`.

The relevant validators pay the price for misbehavior by burning the specific
amount of coins staked and delegated to them, immediately at the moment when
their misbehavior is caught. The penalty shall be distributed amount the stake
holder and the delegated stake holders according to the distribution mechanism
presented in [Incentive Distribution](#distribution). 

#### parameters
- `PenaltyRatioM`

### Downtime
If a validator missed `n_blks` blocks within last `laziness_window`
blocks and `n_blks >= laziness_threshold`, then this validator gets penalized
accordingly. The penalty is calculated by `penalty_ratio_l * eff_stake`.


## On-chain Protocol Upgrade
To enhance the stability of AMO's overall system, it is required to upgrade its
application protocol consistently. To apply a new protocol on alive blockchain,
'hard-fork' is an inevitable process necessary to be done externally. AMO
provides a feature which helps 'hard-fork' get processed more smoothly.  

### Node startup

### Marking version
AMO ABCI app's protocol version is recorded as `ProtocolVersion` in app's
`state`. `ProtocolVersion` gets initiated with the value of `genesis.json`. If
not specified, it is set with current app's hard-coded protocol version.

### Upgrade decision
The specific time when a new protocol gets applied and its version is decided
among validators through [proposing and voting](#proposing-and-voting) draft.
The time is recorded as `UpgradeProtocolHeight` and the version as
`UpgradeProtocolVersion` in app's `config`.

### Upgrade execution
At the beginning of block creation `BeginBlock()`, AMO ABCI app checks
conditions and processes operations as follows:

#### Protocol version update
- if `blk.height` == `app.config.UpgradeProtocolHeight`
  - `app.state.ProtocolVersion` &larr; `app.config.UpgradeProtocolVersion` 

#### Block process 
- if `sw.ProtocolVersion` != `app.state.ProtocolVersion`
  - abort and exit current sw

#### State migration
- if `blk.height` == `app.config.UpgradeProtocolHeight`
  - execute `app.MigrateToX()` (`X` refers to `sw.ProtocolVersion`)

## Genesis App State
Initial state of the app (_genesis app state_) is defined by genesis document
(genesis.json file in tendermint config directory, typically
$HOME/.tendermint/config/genesis.json). Initial app state is described in
`app_state` field in a genesis document. For example:
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
**TM:** In order to reset and apply new genesis state, run the following
command in command line:
```bash
amod tendermint unsafe_reset_all
```
An AMO-compliant blockchain node should have some mechanisms to modify internal
database for this operation.

## Further Notes
### Replay Attack
In order to prevent [replay
attack](https://en.wikipedia.org/wiki/Replay_attack) (in some sense,
double-spending), every AMO transaction is checked for whether it is already
introduced or processed in previous blocks. Basic idea is that when a
blockchain node sees a transaction that is already presented in the blockchain
network, it immediately discards the transaction. Here, every transaction has a
`tx hash` in Tendermint context. This `tx hash` is a hash of whole byte
sequence representing the transaction. Since we incorporated ECDSA signature to
authenticate the sender's identity, this gives randomness to the transaction,
and it can prevent replay attacks. However, AMO blockchain protocol itself is
independent of Tendermint. Moreover a future version AMO blockchain may not use
Tendermint as a base platform. So, in order to provide some generic
countermeasure against replay attacks, we use `ReplayPreventer`, a module which
monitors every incoming transaction to prevent its replay attacks by checking
its existence in the blockchain network.

If a user wants to send the same amount of coin to the same recipient again,
then the user must put into the transaction a signature different from the one
used for the previous transaction. If so, the transaction would have a
different `tx hash` and be treated as a different one, passing the transaction
check process of `ReplayPreventer` successfully.
