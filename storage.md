# AMO Storage Service
DRAFT

Any storage service must meet the following requirements in order to
participate in AMO environment as an AMO storage service.

There is a default AMO storage service, and its availability is guaranteed by
AMO Labs. See [Default AMO Storage Service](ceph.md) for more information.

## General Requirements
* object storage
* high availability
* long-term persistent storage
* updatable data body

## Data Parcel
A _data parcel_ is a generic term used to call any data item for sale in AMO
infrastructure. Each data parcel should have a globally unique identifier. A
data parcel has no intrinsic properties related to internal data structure or
contextual meaning. It can be a single byte or a tera byte-sized multi-media
data. One requirement is that anyone shall get an identical data from an AMO
storage service for the same data parcel identifier and for a given time.

_For a convention, empty data body means there is no data parcel registered
with a given data parcel ID._

## Identifiers

### Storage Service ID
TODO:
globally and uniquely identify storage service name and access point.
there must be some mechanism to make a consensus on a globally unique storage
service identifier.

### Data Parcel ID
TODO:
globally and uniquey identify data parcel throughout all storage services.
there must be some mechanism to generate new globally unique data parcel
identifier.

## AMO Storage Adapter
TODO:
define common API outfit

## Protocol Messages
### User Identity
```json
"_account_address_"
```
* *address* is arealdy a hex-encoded 16-byte binary sequence. _address_ is used
  to uniquely identify a user account in AMO ecosystem. See [AMO Blockchain
  Protocol Specification](protocol.md) for more information.

### Signature Appendix
```json
{
	"pubkey": "_hex-encoded_ECDSA_public_key_",
	"sig_bytes": "_hex-encoded_ECDSA_signature_"
}
```
* _pubkey_ is a hexadecimal representation of a compressed elliptic curve
  point. See [AMO Blockchain Protocol Specification](protocol.md) for more
  information.
* *sig_bytes*  is a hexadecimal representation of concatenated ECDSA signature
  bytes.

### Data Parcel Identifier
```json
"_hex-encoded_binary_sequence_"
```

### Data Parcel
```json
{
	"id": _data_parcel_ID_,
	"owner": _user_identity_,
	"metadata": _metadata_,
	"data": _hex-encoded_binary_sequence_
}
```
* *id* may be omitted.
* *metadata* is any JSON object and can be null.

### Initiation Message Type 1
```json
{
	"user": _user_identity_,
	"operation": _operation_description_,
	"signature": _signature_appendix_
}
```
* signing bytes for the *signature* is the initiation message itself without
  *signature* field.

### Initiation Message Type 2
```json
{
	"user": _user_identity_,
	"operation": _operation_description_
}
```

### Auth Challenge Message
```json
{
	"challenge": "_hex-encoded_256-bit_sequence_"
}
```
* _challenge_ is a hexadecimal representation of a 256-bit binary sequence

### Auth Response Message
```json
{
	"signature": _signature_appendix_
}
```
* signing bytes for the *signature* is the concatenation of initiation message
  and challenge message.

### Result Message
```json
{
	"code": _integer_,
	"data": _operation_specific_result_data_
}
```

## API Operations
list of tier 1 operations

### Upload
Operation description is as follows:
```json
{
	"name": "upload",
	"hash": "_hex-encoded_256-bit_hash_"
}
```
* *hash* is SHA256 output of the data body.

Operation steps are as follows:
* client &rArr; storage: initiation message type 1
* client &lArr; storage: continue or fail
* client &rArr; storage: data parcel
* client &lArr; storage: result message with no data field

Upon receiving *initiation message*, storage service shall perform signature
verification and response to the client with either *continue* or *fail*. No
access control than this.

Result data is a *data parcel ID*.
**need discussion**

### Inspect
Operation description is as follows:
```json
{
	"name": "inspect",
	"id": _data_parcel_ID_
}
```

Operation steps are as follows:
* client &rArr; storage: initiation message type 2
* client &lArr; storage: result message

No access control at this operation.

Result data is a data parcel metadata.

### Download
Operation description is as follows:
```json
{
	"name": "download",
	"id": _data_parcel_ID_
}
```

Operation steps are as follows:
* client &rArr; storage: initiation message type 2
* client &lArr; storage: auth challenge
* client &rArr; storage: auth response
* client &lArr; storage: result message

Upon receiving *auth response*, storage service shall access AMO blockchain to
perform AMO-based access control.

Result data is either empty or a *data parcel* depending on the access control
result.

### Remove
Operation description is as follows:
```json
{
	"name": "remove",
	"id": _data_parcel_ID_
}
```

Operation steps are as follows:
* client &rArr; storage: initiation message type 2
* client &lArr; storage: auth challenge
* client &rArr; storage: auth response
* client &lArr; storage: result message

Upon receiving *auth response*, storege service shall perform local identity
matching.

Result data is empty.

### *Append*
optional. tier 2 oepration
### *Replace*
optional. tier 2 oepration

## Access Control
### AMO-Based Access Control
TODO

See [Usage query](rpc.md#usage-query) for how to query via RPC channel.

### Local Identity Matching
It's a simple account address matching. Identity from the user is reliable
enough, and if it exactly matches the owner field in the saved data parcel the
access control result is success. Fail otherwise.
