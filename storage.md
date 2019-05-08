# AMO Storage Service Requirements
Any AMO storage service must meet the following requirements in order to participate in AMO environment as a AMO-compatible storage service.

## Storage Service Identification
A storage service must be registered as AMO-compatible storage service. Upon registration a storage service shall receives a unique identifier. AMO-compatible clients will use this identifier to locate the storage service and figure out how to connect and interact with the service nodes.

## Data Identification
A storage service must provide a form of an object storage service. Every data item in the storage must be able to be identified by a unique identifier. An typical example is Amazon S3 service. In AMO environment, all data items are identified, uploaded, and retrieved by the unit of *parcel*. A data *parcel* does not have any special internal structure. It is just a unit of addressing and merchandizing. Its content may be a single *byte* or a *bulk* of data with the size of several terabytes.

## Access Control
A service must implement a access control mechanism to meet the minimum data protection requirement. A public key signature scheme is used for user authentication, and AMO blockchain query is used for permission check.

### Overall flow
1. AMO client &rarr; storage node
    1. An AMO client or AMO-compatible client sends a request<sup>&dagger;</sup> to retrieve a data parcel.
    1. Storage node verifies the signature of the request<sup>&dagger;&dagger;</sup>.
    1. Storage node records a user address from the client request.
1. storage node &lrarr; AMO blockchain node
    1. Storage node sends a query to an AMO blockchain node via AMO client RPC<sup>&dagger;&dagger;&dagger;</sup> to check if the user has a *granted usage* to the target data *parcel*.
    1. AMO blockchain node replies to the RPC query.
1. AMO client &larr; storage node
    1. Storage node checks the permission
    1. Storage node either transfer the data parcel or reject the request.

&dagger; See [Data format](#data-format).<br/>
&dagger;&dagger; See [User authentication](#user-authentication).<br/>
&dagger;&dagger;&dagger; See [Usage query](rpc.md#usage-query).

### User authentication
AMO client must sends a signed request along with a public key and a time-stamp to a storage node. The storage node must verify the signature before any action. AMO storage node shall derive the user account address from the public key. See [AMO Blockchain Protocol Specification](protocol.md) for the address derivation method.

### Permission check
See [Usage query](rpc.md#usage-query) for how to query via RPC channel.<br/>
See [AMO Blockchain Protocol Specification](protocol.md) for how to interpret RPC response.

### Data format

##### AMO client &rarr; storage node
```json
{
    "address" : "_user_account_address_",
    "data_parcel": "_data_parcel_id_",
    "timestamp": "_time_of_the_request_",
    "signature": {
        "pubkey": "_hex_encoded_public_key_",
        "sig_bytes": "_hex_encoded_signature_"
    }
}
```

##### storage node &lrarr; AMO blockchain node
See [Usage query](rpc.md#usage-query).

##### storage node &rarr; AMO client
specific to each storage service type.
