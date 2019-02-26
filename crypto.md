# Notes on Cryptography

AMO blockchain uses industry-standard cryptographic algorithms instead of non-standard ones such as secp256k1 or Keccak. This is because the typical target environment of AMO is embedded devices and rather conservative environments such as cars. This decision has pros and cons:

##### Pros
- Can use CMVP-certified cryptographic modules
- Has much larger variety of choices for cryptographic implementations

##### Cons
- Cannot use ready-made cryptographic implementations from blockchain community
- Cannot use market-leading crypto-wallets without add-ons or modifications

While AMO blockchain uses Tendermint as its base, Tendermint uses another non-standard Edward curve along with several non-standard hash algorithms. In order to remedy this situation, AMO blockchain developers forked the original Tendermint code and modify it to use P256 curve as its default ECDSA parameter.