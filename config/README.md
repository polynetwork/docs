# TestNet | [DevNet](README_DevNet.md) 

This's cross chain asset contract info, it's used to bind asset mapping in different chain, if the name ends with (s) on behalf of the asset is mapping.

## Ethereum

Type | Contract Hash | Desc
---|---|---
CCMP | 0x87FdA0ceD9aEb973E7e8655584DeEC77658546cC | Cross Chain Manager Proxy contract hash 
Lock Proxy | 0x388Ed8B73bd707A78034E1d157fA08Da24095c18 | The lock proxy bridge contract hash for asset not implementing the "lock" and "unlock" logic to do cross chain tx
ERC20 | 0x8142d8936A3571F6F4777716D37b1Dd4c1Ca1d93 | ERC20 template contract hash in Ethereum chain
OEP4x | 0x27A9206bbF288e469AaCC2168E621e503c2bc846 | OEP4x template contract hash in Ethereum chain
ONGx | 0xFb37c160CFBd8BD4Ba6df6f70e2449b6EB83fc26 | ONGx contract hash in Ethereum chain
ONTx | 0x147CB77cFd0D3A0932D859820d1A3DB9686396F9 | ONTx contract hash in Ethereum chain
ETH | 0x0000000000000000000000000000000000000000 | The asset hash that we treat as the Ether asset
BTCx | 0xA427C5994C2e00272516140836D21BdC3A538c79 | Btcx contract hash corresponding with unique btc redeem script
NEOx | 0x3DB9D2FF5A878A70f049cFD5Cb3D884EA7EF539F |

## Ontology
#### Please donot send from or to Ontology network during upgrade of Ontology testnet

Type | Contract Hash | Desc
---|---|---
Lock Proxy | B: f4b57dee05beb0be928cb57b17f2db96d5187053 </br> L: 537018d596dbf2177bb58c92beb0be05ee7db5f4 | The bridge contract hash for asset not implementing the "lock" and "unlock" logic to do cross chain tx
ERC20x | B:8c8257c1f58533e906f6e24ae06d06be0fc16432  </br> L: 3264c10fbe066de04ae2f606e93385f5c157828c  | ERC20 template contract hash in Ontology chain
OEP4 | B: 34661f6e90f3d590bbe0281a76935f94bd459532 </br> L: 329545bd945f93761a28e0bb90d5f3906e1f6634 | OEP4 template contract hash in Ontology chain
ONG | B: 0200000000000000000000000000000000000000 </br> L: 0000000000000000000000000000000000000002 | ONG asset hash in Ontology chain
ONT | B: 0100000000000000000000000000000000000000 </br> L: 0000000000000000000000000000000000000002 | ONT asset hash in Ontology chain
ETHx | B: 1fd76f19e621816de34c5c74643ca69aeeab4d42 </br> L: 424dabee9aa63c64745c4ce36d8121e6196fd71f | Ethx asset hash in Ontology chain
BTCx | B: aa0e7aede9c0d5b599b9908237f3df84d7260be5 </br> L: e50b26d784dff3378290b999b5d5c0e9ed7a0eaa |  Btcx contract hash in Ontology chain
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
ATOMx | B: 0xffd33fc3e0c5f3574ef6a0fd028971a7ff7d3da6 </br> L: a63d7dffa7718902fda0f64e57f3c5e0c33fd3ff | ATOM asset hash in Neo chain
CCMC | B: 0x978286951e0011221de3fffe6a9e6dd160925837 </br> L: 37589260d16d9e6afeffe31d2211001e95868297 | Cross Chain Manager Contract
Lock Proxy | B: 0x87220d44c14178e5e57850bbddc4bc69cedbfdb0 </br> L: b0fddbce69bcc4ddbb5078e5e57841c1440d2287 | The lock proxy bridge contract hash for asset not implementing the "lock" and "unlock" logic to do cross chain tx


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
