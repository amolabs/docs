# AMO Storage Requirements
Any storage service node must meet the following requirements in order to participate in AMO environment as a AMO-compatible storage node.

## Storage Identification
TBA

## Data Identification
TBA

## Access Control
A node must implement a access control mechanism to meet the minimum data protection requirement.

### Overall flow
1. AMO client &rarr; AMO storage node
    1. An AMO client(see [AMO technical terms and definitinos](goole_drive_doc)) or AMO-compatible client sends a request(see [Data format](#data-format)) to retrieve a data parcel(see [AMO technical terms and definitinos](google_drive_doc)).
1. AMO storage node &rarr; AMO blockchain node
    1. An AMO storage node verifies the signature.
    1. The AMO storage node records a user address from the client request.
    1. The AMO storage node sends a query to an AMO blockchain node via AMO client RPC(see [Usage query](rpc.md#usage-query)) to check if the user has a permission to retrieve a data parcel.
1. AMO blockchain node &rarr; AMO storage node
    1. The AMO blockchain node replies to the RPC query.
1. AMO storage node &rarr; AMO client
    1. The AMO storage node checks the permission
    1. The AMO storage node either transfer the data parcel or reject the request.

### User authentication
AMO client must sends a signed request along with a public key. AMO storage node must verify the signature before any action. AMO storage node shall derive the user account address from the public key. See [AMO Blockchain Protocol Specification](protocol.md) for the address derivation method.

**TODO:** use challenge-response scheme to mitigate a replay attack.

### Permission check
See [Usage query](rpc.md#usage-query) for how to query via RPC channel.

See [AMO Blockchain Protocol Specification](protocol.md) for how to interpret RPC response.

### Data format
*Draft*

**AMO client request to AMO storage node**
```json
{
    "address" : "_user_account_address_",
    "data_parcel": "_data_parcel_id_",
    "signature": {
        "pubkey": "_hex_encoded_public_key_",
        "sig_bytes": "_hex_encoded_signature_"
    }
}
```

**AMO storage node to AMO blockchain node**
```json
see rpc doc
```

**AMO blockchain node to AMO storage node**
```json
see rpc and protocol doc
```

**AMO storage node to AMO client**
```json
TODO
```
