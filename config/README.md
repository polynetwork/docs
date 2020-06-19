# TetNet | [DevNet](README_DevNet.md) 

This's cross chain asset contract info, it's used to bind asset mapping in different chain, if the name ends with (s) on behalf of the asset is mapping.

## Ethereum

Type | Contract Hash | Desc
---|---|---
CCMP | 0xC8042579D6b60E0e35161F228827E3Fa0F51d5B6 | Cross Chain Manager Proxy contract hash 
Lock Proxy | 0x75ED27ee68F0D6bdd4e41E38388C5A9028Fb6707 | The lock proxy bridge contract hash for asset not implementing the "lock" and "unlock" logic to do cross chain tx
ERC20 | 0xD1cb2BDA2146C0878b41b5c0164E4420aef72584 | ERC20 template contract hash in Ethereum chain
OEP4x | 0x63692d2BA64a5869114068b7B08DffED94F378D8 | OEP4x template contract hash in Ethereum chain
ONGx | 0xA8177Ee8a6E496c701CfeC0cBD8f723cC851153D | ONGx contract hash in Ethereum chain
ONTx | 0x514092ef689EBAe8EebBCa97fD6987e94B033cCb | ONTx contract hash in Ethereum chain
ETH | 0x0000000000000000000000000000000000000000 | The asset hash that we treat as the Ether asset
BTCx | 0xbbE0dA0f3D5132A5C245D7760d2700E2192fBa39 | Btcx contract hash corresponding with unique btc redeem script
NEOx |  | Neox contract hash in Ethereum chain

## Ontology
#### Please donot send from or to Ontology network during upgrade of Ontology testnet
Type | Contract Hash | Desc
---|---|---
Lock Proxy | B: </br> L:  | The bridge contract hash for asset not implementing the "lock" and "unlock" logic to do cross chain tx
ERC20x | B:  </br> L:  | ERC20 template contract hash in Ontology chain
OEP4 | B:  </br> L:  | OEP4 template contract hash in Ontology chain
ONG | B:  </br> L:  | ONG asset hash in Ontology chain
ONT | B:  </br> L:  | ONT asset hash in Ontology chain
ETHx | B:  </br> L:  | Ethx asset hash in Ontology chain
BTCx | B:  </br> L:  |  Btcx contract hash in Ontology chain
CNEOx | B:  </br> L:  |  CNEOx contract hash in Ontology chain
CGASx | B:  </br> L:  | CGasx contract hash in Ontology chain
## Neo

Type | Contract Hash | Desc
---|---|---
CNEO-TEST | B: 0x47e1b8aee4b8ea01a8868d86ed61be50e8f647bb </br> L: bb47f6e850be61ed868d86a801eab8e4aeb8e147 | 
CGAS | B: 0x74f2dc36a68fdc4682034178eb2220729231db76 </br> L: 76db3192722022eb7841038246dc8fa636dcf274 |
ETHx | B: 0x74fac41ad5ad23921a3400e953e1cafb41240d08 </br> L: 080d2441fbcae153e900341a9223add51ac4fa74 | Eth asset hash in Neo chain
ONTx | B: 0xffd33fc3e0c5f3574ef6a0fd028971a7ff7d3da6 </br> L: a63d7dffa7718902fda0f64e57f3c5e0c33fd3ff | ONT asset hash in Neo chain
CCMC | B: 0x02d9290db5ff0ce5242727fbdbdf01aacc6656f5 </br> L: f55666ccaa01dfdbfb272724e50cffb50d29d902 | Cross Chain Manager Contract
Lock Proxy | B: 0xa8ed3b1e0337230f1b90028ac3650e318bbec1ad </br> L: adc1be8b310e65c38a02901b0f2337031e3beda8 | The lock proxy bridge contract hash for asset not implementing the "lock" and "unlock" logic to do cross chain tx


## Note 
`B` means big-endian, we can search the contract transaction history in corresponding explorer.

`L` means little-endian, we usually use it as the asset hash input when we do binding asset hash operation.

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
Cosmos| stake | 7374616b65 | not including currently

## Router And ChainId
Type | Router Number | ChainId
:-:|:-:|:-:
Bitcoin | 1 | 1
Ethereum | 2 | 2
Ontology | 3 | 3
NEO | 4 | 4
Cosmos-gaia | 5 | 5
Switcheo | 5 | release soon
