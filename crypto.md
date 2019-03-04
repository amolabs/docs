# Notes on Cryptography

## General
AMO-compliant blockchain clients use industry-standard cryptographic algorithms instead of non-standard ones such as secp256k1 or Keccak. This is because the typical target environment of AMO is rather conservative environments such as cars or embedded devices. This decision has pros and cons:

##### Pros
- Can use CMVP-certified cryptographic modules
- Has much larger variety of choices for cryptographic implementations

##### Cons
- Cannot use ready-made cryptographic implementations from blockchain community
- Cannot use market-leading crypto-wallets without add-ons or modifications

While AMO blockchain uses Tendermint as its base, Tendermint uses another non-standard Edward curve along with several non-standard hash algorithms. In order to remedy this situation, AMO blockchain developers forked the original Tendermint code and modify it to use P256 curve as its default ECDSA parameter.

## Algorithms
CMVP-approved algorithms and related standards:
- SHA256, SHA3-256, SHA3-512
    - Secure Hash Standard
- AES128-CTR
    - Advanced Encryption Standard
- ECDSA with NIST P256 curve
    - Digital Signature Standard
- ECDH with NIST P256 curve
    - Digital Signature Standard
    - Pair-Wise Key-Establishment Schemes Using Discrete Logarithm Cryptography
- PBKDF
    - Password-Based Key Derivation
