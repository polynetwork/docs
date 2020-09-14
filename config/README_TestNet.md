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
ECCM | 0x726532586C50ec9f4080B71f906a3d9779bbd64F   | Ethereum Cross Chain Manager contract hash
Lock Proxy | 0xD8aE73e06552E270340b63A8bcAbf9277a1aac99 | The lock proxy bridge contract hash for asset not implementing the "lock" and "unlock" logic to do cross chain tx
ERC20 | 0x276788aF4a803781267c84692416311DE1F761f9 | ERC20 template contract hash in Ethereum chain
EOEP4 | 0x3105A14F7956D33a51F12eF3AE50A3f1eF161Dff | OEP4x template contract hash in Ethereum chain
EONG | 0x42d9feF0Cbd9c3000CECe9764d99A4a6fE9E1B34 | ONGx contract hash in Ethereum chain
EONT | 0x530aae4C0859894023906e28467f2a7F111B6ff3 | ONTx contract hash in Ethereum chain
EONTD | 0x76130c293AA35bf7B3e5fED1E9aE1E5DF12C6A92 | ONTD ONT with decimals 
ETH | 0x0000000000000000000000000000000000000000 | The asset hash that we treat as the Ether asset
USDT | 0xad3f96ae966ad60347f31845b7e4b333104c52fb | USD Tether 
USDC | 0x0d9c8723b343a8368bebe0b5e89273ff8d712e3c | USDC 
WBTC | 0x557563dc4ed3fd256eBA55B9622f53331ab97c2f | WBTC 
DAI | 0x8Cad2301F7348DFc10C65778197028F432d51e76 | DAI 
EBTC | 0x92705a16815A3d1AEC3cE9Cc273C5aa302961FcC | Btcx contract hash corresponding with unique btc redeem script
ECNEO | 0x7E269f2f33A97C64192e9889FAeEC72A6fcdB397 |
RENBTC | 0x239100e629a9Ca8e0BF45C7892b0fc72d78AA97A |

## Ontology

#### Please donot send from or to Ontology network during upgrade of Ontology testnet

Type | Contract Hash | Desc
---|---|---
Lock Proxy | B: 33c439c502cb4b6ac5a1e8057a65fe1fa7c300e2 </br> L: e200c3a71ffe657a05e8a1c56a4bcb02c539c433 | The bridge contract hash for asset not implementing the "lock" and "unlock" logic to do cross chain tx
OERC20 | B: e930755b130dccb25dc3cfee2b2e30d9370c1a75  </br> L: 751a0c37d9302e2beecfc35db2cc0d135b7530e9  | ERC20 template contract hash in Ontology chain
OEP4 | B: 969850e009b5e2a061694f3479ec8e44bc68bcd3 </br> L: d3bc68bc448eec79344f6961a0e2b509e0509896 | OEP4 template contract hash in Ontology chain
ONG | B: 0200000000000000000000000000000000000000 </br> L: 0000000000000000000000000000000000000002 | ONG asset hash in Ontology chain
ONT | B: 0100000000000000000000000000000000000000 </br> L: 0000000000000000000000000000000000000001 | ONT asset hash in Ontology chain
OUSDC      | B:07a12c0a6bdce4df04ef4b2045d1b0fd63a56e25</br> L:256ea563fdb0d145204bef04dfe4dc6b0a2ca107 |                                                              
ONTd | B: 33ae7eae016193ba0fe238b223623bc78faac158</br> L:58c1aa8fc73b6223b238e20fba936101ae7eae33 | ONT with 9 decimal contract hash 
oUSDT | B: c6f91c11d740d39943b99a6b1c6fd2b5f476e2a3 </br>L: a3e276f4b5d26f1c6b9ab94399d340d7111cf9c6 | Ontology wrapped USDT (Ethereum) Cross Chain asset contract hash
OWBTC | B: aede525f05065306423a5522bfcd31b5847ffa52</br> L: 52fa7f84b531cdbf22553a42065306055f52deae | Ontology wrapped WBTC (Ethereum) Cross Chain asset contract hash
ODAI | B: 96cf88356123592835a2fa75068a242260be1791</br> L: 9117be6022248a0675faa235285923613588cf96 | 
OETH | B: 7009a2f7c8a2e45fa386a6078c7bfeaf518be487</br> L: 87e48b51affe7b8c07a686a35fe4a2c8f7a20970 | 
ONEO | B: 13eef3e184d878038317d806796b3af2d9f9b36d</br> L: 6db3f9d9f23a6b7906d817830378d884e1f3ee13 | 
ORENBTC | B: ec547bc4486dea97cb659f1fe73407922f9e63c8</br> L: c8639e2f920734e71f9f65cb97ea6d48c47b54ec | 

## Neo

Type | Contract Hash | Desc
---|---|---
CNEO | B: 0xc074a05e9dcf0141cbe6b4b3475dd67baf4dcb60 </br> L: 60cb4daf7bd65d47b3b4e6cb4101cf9d5ea074c0 | 
CGAS | B: 0x74f2dc36a68fdc4682034178eb2220729231db76 </br> L: 76db3192722022eb7841038246dc8fa636dcf274 |
ETHx | B: 0xd7b32de37ad906df80805c2419ff5560d20f9cbf </br> L: bf9c0fd26055ff19245c8080df06d97ae32db3d7 | Eth asset hash in Neo chain
BTCx | B: 0x3ee29d5cc82771e91383f9ba09c6f5c5878f3f24 </br> L: 243f8f87c5f5c609baf98313e97127c85c9de23e | BTC asset hash in Neo chain
ONTx | B: 0x5a9222225f1bdb135123b74354c7248200c440aa </br> L: aa40c4008224c75443b7235113db1b5f2222925a | ONT asset hash in Neo chain
ATOMx | B: 0xb25b51d684f7945f7aab43496cb0e87138abdb35 </br> L: 35dbab3871e8b06c4943ab7a5f94f784d6515bb2 | ATOM asset hash in Neo chain
CCMC | B: 0xe1695b1314a1331e3935481620417ed835669407 </br> L: 07946635d87e4120164835391e33a114135b69e1 | Cross Chain Manager Contract
Lock Proxy | B: 0x229e8fe772d5f0cbb408d58b593e42a1d1dfa3a9 </br> L: a9a3dfd1a1423e598bd508b4cbf0d572e78f9e22 | The lock proxy bridge contract hash for asset not implementing the "lock" and "unlock" logic to do cross chain tx


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
