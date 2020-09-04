# [MainNet](README.md) | TestNet | [DevNet](README_DevNet.md) 

This's cross chain asset contract info on <strong>TESTNET</strong>, it's used to bind asset mapping in different chain, if the name ends with (s) on behalf of the asset is mapping.

## Decentralized Application TestNet Nodes
Chain | IP | Rpc Port
---|---|---
Poly | ```http://beta1.poly.network``` | 21336
Neo | ```http://seed9.ngd.network``` | 20332



## Ethereum

Type | Contract Hash | Desc
---|---|---
CCMP | 0xb600c8a2e8852832B75DB9Da1A3A1c173eAb28d8 | Cross Chain Manager Proxy contract hash 
ECCD | 0xA38366d552672556CE82426Da5031E2Ae0598dcD | Ethereum Cross Chain Data contract hash
ECCM | 0x726532586C50ec9f4080B71f906a3d9779bbd64F | Ethereum Cross Chain Manager contract hash
Lock Proxy | 0xD8aE73e06552E270340b63A8bcAbf9277a1aac99 | The lock proxy bridge contract hash for asset not implementing the "lock" and "unlock" logic to do cross chain tx
ERC20 | 0x276788aF4a803781267c84692416311DE1F761f9 | ERC20 template contract hash in Ethereum chain
OEP4x | 0x3105A14F7956D33a51F12eF3AE50A3f1eF161Dff | OEP4x template contract hash in Ethereum chain
ONGx | 0x42d9feF0Cbd9c3000CECe9764d99A4a6fE9E1B34 | ONGx contract hash in Ethereum chain
ONTx | 0x530aae4C0859894023906e28467f2a7F111B6ff3 | ONTx contract hash in Ethereum chain
ETH | 0x0000000000000000000000000000000000000000 | The asset hash that we treat as the Ether asset
USDT | 0xad3f96ae966ad60347f31845b7e4b333104c52fb | USD Tether 
WBTC | 0x557563dc4ed3fd256eBA55B9622f53331ab97c2f | WBTC 
DAI | 0x8Cad2301F7348DFc10C65778197028F432d51e76 | DAI 
BTCx |  | Btcx contract hash corresponding with unique btc redeem script
NEOx |  |

## Ontology
#### Please donot send from or to Ontology network during upgrade of Ontology testnet

Type | Contract Hash | Desc
---|---|---
Lock Proxy | B: 33c439c502cb4b6ac5a1e8057a65fe1fa7c300e2 </br> L: e200c3a71ffe657a05e8a1c56a4bcb02c539c433 | The bridge contract hash for asset not implementing the "lock" and "unlock" logic to do cross chain tx
ERC20x | B: e930755b130dccb25dc3cfee2b2e30d9370c1a75  </br> L: 751a0c37d9302e2beecfc35db2cc0d135b7530e9  | ERC20 template contract hash in Ontology chain
OEP4 | B: 969850e009b5e2a061694f3479ec8e44bc68bcd3 </br> L: d3bc68bc448eec79344f6961a0e2b509e0509896 | OEP4 template contract hash in Ontology chain
ONG | B: 0200000000000000000000000000000000000000 </br> L: 0000000000000000000000000000000000000002 | ONG asset hash in Ontology chain
ONT | B: 0100000000000000000000000000000000000000 </br> L: 0000000000000000000000000000000000000002 | ONT asset hash in Ontology chain
ETHx | B: 7448f5f18088c566a3e78c207ea0f06b0aea58b6 </br> L: b658ea0a6bf0a07e208ce7a366c58880f1f54874 | Ethx asset hash in Ontology chain
USDTx | B: 11c60400f54c17df0d8e9c4a38c333b66c1f1c54 </br> L: 541c1f6cb633c3384a9c8e0ddf174cf50004c611 | USDT on Ontology 
WBTCx | B:  c984a8fa8d3b4c9399e36b5e257d9e150d0480a1</br> L: a180040d159e7d255e6be399934c3b8dfaa884c9 | WBTC 
DAI | B: 3249446364433365f075793e298ee3327c1fcb58</br> L: 58cb1f7c32e38e293e7975f06533436463444932 | DAI 
BTCx | B:  </br> L: |  Btcx contract hash in Ontology chain
CNEOx | B:  </br> L:  |  CNEOx contract hash in Ontology chain
CGASx | B:  </br> L:  | CGasx contract hash in Ontology chain

## Neo

Type | Contract Hash | Desc
---|---|---
CNEO-TEST | B: 0x47e1b8aee4b8ea01a8868d86ed61be50e8f647bb </br> L: bb47f6e850be61ed868d86a801eab8e4aeb8e147 | 
CGAS | B: 0x74f2dc36a68fdc4682034178eb2220729231db76 </br> L: 76db3192722022eb7841038246dc8fa636dcf274 |
ETHx | B: 0xd7b32de37ad906df80805c2419ff5560d20f9cbf </br> L: bf9c0fd26055ff19245c8080df06d97ae32db3d7 | Eth asset hash in Neo chain
BTCx | B: 0x3ee29d5cc82771e91383f9ba09c6f5c5878f3f24 </br> L: 243f8f87c5f5c609baf98313e97127c85c9de23e | BTC asset hash in Neo chain
ONTx | B: 0xffd33fc3e0c5f3574ef6a0fd028971a7ff7d3da6 </br> L: a63d7dffa7718902fda0f64e57f3c5e0c33fd3ff | ONT asset hash in Neo chain
ATOMx | B: 0xb25b51d684f7945f7aab43496cb0e87138abdb35 </br> L: 35dbab3871e8b06c4943ab7a5f94f784d6515bb2 | ATOM asset hash in Neo chain
CCMC | B: 0x82a3401fb9a60db42c6fa2ea2b6d62e872d6257f </br> L: 7f25d672e8626d2beaa26f2cb40da6b91f40a382 | Cross Chain Manager Contract
Lock Proxy | B: 0x777d0dcc388dbd0fcb41488811d1486b3056fa16 </br> L: 16fa56306b48d111884841cb0fbd8d38cc0d7d77 | The lock proxy bridge contract hash for asset not implementing the "lock" and "unlock" logic to do cross chain tx


## Note 
`B` means big-endian, we can search the contract transaction history in corresponding explorer.

`L` means little-endian, we usually use it as the asset hash input when we do binding asset hash operation.


## Cosmos

Type | Denom (coin name) | Asset/Contract Hash | Desc
:-:|:-:|:-:|:-:
Lock Proxy | | f71b55ef55cedc91fd007f7a9ba386ec978f3aa8 |
ERC20 | erc20x | 657263323078 |
OEP4x | oep4x | 6f65703478 |
ONGx | ongx | 6f6e6778 |
ONTx | ontx | 6f6e7478 |
ETHx | ethx | 65746878 |
BTCx | btcx | 62746378 |
NEOx | neox | 6e656f78 | not including currently
GASx | gasx | 67617378 | not including currently
ATOM | stake | 7374616b65 | not including currently

## Router And ChainId
Type | Router Number | ChainId
:-:|:-:|:-:
Bitcoin | 1 | 1
Ethereum | 2 | 2
Ontology | 3 | 3
NEO | 4 | 4
Cosmos-gaia | 5 | 5
Switcheo | 5 | release soon
