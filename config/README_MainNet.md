# MainNet | [TestNet](README.md) | [DevNet](README_DevNet.md) 

This's cross chain asset contract info ONLY in <strong>MAINNET</strong> mode, it's used to bind asset mapping in different chain, if the name ends with (s) on behalf of the asset is mapping.


## Decentralized Application MainNet Nodes
Chain | IP | Rpc Port
---|---|---
Poly | ```http://seed1.poly.network``` | 20336
Neo | ```http://seed9.ngd.network``` | 10332



## Btc


## Ethereum

Type | Contract Hash | Desc
---|---|---
CCMP | 0x5a51e2ebf8d136926b9ca7b59b60464e7c44d2eb | Cross Chain Manager Proxy contract hash 
ECCD | 0xcf2afe102057ba5c16f899271045a0a37fcb10f2 | Ethereum Cross Chain Data contract hash
ECCM | 0x838bf9e95cb12dd76a54c9f9d2e3082eaf928270 | Ethereum Cross Chain Manager contract hash

## Ontology

## Neo
Type | Contract Hash | Desc
---|---|---
ETHx | B: 0x8918cf19674db1f7e02b9a68ff7cf72ff280af27 </br> L: 27af80f22ff77cff689a2be0f7b14d6719cf1889 | Eth asset hash in Neo chain
ONTx | B: 0x62331ab464a9091fc3f5fb6fc1b58f10208c58ca </br> L: ca588c20108fb5c16ffbf5c31f09a964b41a3362 | ONT asset hash in Neo chain
CCMC | B: 0x82a3401fb9a60db42c6fa2ea2b6d62e872d6257f </br> L: 7f25d672e8626d2beaa26f2cb40da6b91f40a382 | Cross Chain Manager Contract
Lock Proxy | B: 0x02a8554f5a7d0714d7c31c46424b1a712eb01443 </br> L: 4314b02e711a4b42461cc3d714077d5a4f55a802 | The lock proxy bridge contract hash for asset not implementing the "lock" and "unlock" logic to do cross chain tx

## Note 
`B` means big-endian, we can search the contract transaction history in corresponding explorer.

`L` means little-endian, we usually use it as the asset hash input when we do binding asset hash operation.

## Switcheo

Type | Denom (coin name) | Asset/Contract Hash | Desc
:-:|:-:|:-:|:-:


## Router And ChainId
Type | Router Number | ChainId
:-:|:-:|:-:
Bitcoin | 1 | 1
Ethereum | 2 | 2
Ontology | 3 | 3
NEO | 4 | 4
Switcheo | 5 | 5
