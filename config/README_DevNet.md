# [TestNet](README.md) | DevNet

This's cross chain asset contract info ONLY in development mode, it's used to bind asset mapping in different chain, if the name ends with (s) on behalf of the asset is mapping.

## Btc

BTCx Redeem Script: 
- 522103c4564b837674de2482961a8d5f2a24a7e11e8a97aac5e92ac2e64500219144512102ccc07d3df7da58bb6fa5cfe5d7be415ff9463171b2600c93c080fcd0d49576a721036ec6299c1b14e57b45f1ad85eecbc48ad5447a05158a1bfb2ffb689ad69490d353ae

BTCx Redeem Key: 
- c737784e5fdae8860fb461d4d30ffd0b34701d5a

## Ethereum

Type | Contract Hash | Desc
---|---|---
CCMP | 0xDff4A9B177fa89394a10506b9f99AadA31B635f8 |
Lock Proxy | 0x20eFad169D803D8d3Ca04f5a71C67A52de687c26 |
ERC20 | 0xE4dc8fa991a3e0BC41f5E736BAc69b8186a1386D |
OEP4x | 0x15b55ca5C3082c95Bd4707651494384D2d2A6b50 |
ONGx | 0x1ed03F921425883ceF5eE59ff129192Ba87DaC48 |
ONTx | 0x1666970b08DD77D67aB43441E519381d6bA9c49c |
ETH | 0x0000000000000000000000000000000000000000 |
BTCx | 0x8940fa5B2bb6e961D0bF3E8C3FBd9757ffCDDAf1 | 
NEOx |  | not including in the testing framework

## Ontology

Type | Contract Hash | Desc
---|---|---
Lock Proxy | B: ebad45b887c6bf7cc4c1df8f72da156bc91b04b7 </br> L: b7041bc96b15da728fdfc1c47cbfc687b845adeb |
ERC20x | B: 2bd086d90c282a26d021d01039b84a968ed57444 </br> L: 4474d58e964ab83910d021d0262a280cd986d02b |
OEP4 | B: d6967fa9b11b0836ec9dc8572f4bab6f65d710f9 </br> L: f910d7656fab4b2f57c89dec36081bb1a97f96d6 |
ONG | B: 0200000000000000000000000000000000000000 </br> L: 0000000000000000000000000000000000000002 |
ONT | B: 0100000000000000000000000000000000000000 </br> L: 0000000000000000000000000000000000000001 |
ETHx | B: d6b4cd930377e7e81d1ace85bbb9ce59f4dd9410 </br> L: 1094ddf459ceb9bb85ce1a1de8e7770393cdb4d6 |
BTCx | B: d177d904456479dd592d4ce368e5d97ec8ffbcee </br> L: eebcffc87ed9e568e34c2d59dd79644504d977d1 |
NEOx | B:  </br> L: | not including in the testing framework
## Neo

Type | Contract Hash | Desc
---|---|---
CNEO-TEST | B:  </br> L:  |
CGAS | B:  </br> L:  |
ETHx | B:  0x74fac41ad5ad23921a3400e953e1cafb41240d08 L: 080d2441fbcae153e900341a9223add51ac4fa74 |
ONTx | B:  </br> L:  |

BTCx | B: 0x658c0e461174aa02a382b16ffe938569ab72db1d </br> L: 1ddb72ab698593fe6fb182a302aa7411460e8c65 |
COSMOSx | B:  </br> L:  |
CCMC | B: 0x27c140d51208fed049a1388c7778517546a83bde  </br> L: de3ba846755178778c38a149d0fe0812d540c127 |
Lock Proxy | B: 0x429871e1d088457081b3a121f19248a8a8a46285 </br> L: 8562a4a8a84892f121a1b381704588d0e1719842 |


Receiver: A: AZPXxnzAMZ58uaETnSkaiiMtvQAwoySBM1 B: 0x2d7e0d3d0ca3347a39863b54852d8ac25cae38c1 L: c138ae5cc28a2d85543b86397a34a30c3d0d7e2d

## Note 
`B` means big-endian, we can search the contract transaction history in corresponding explorer.

`L` means little-endian, we usually use it as the asset hash input when we do binding asset hash operation.

## Cosmos-Gaia

Type | Denom (coin name) | Asset/Contract Hash | Desc
:-:|:-:|:-:|:-:

Lock Proxy | | f71b55ef55cedc91fd007f7a9ba386ec978f3aa8 |

ERC20 | erc20x | 657263323078 |
OEP4x | oep4x | 6f65703478 |
ONGx | ongx | 6f6e6778 |
ONTx | ontx | 6f6e7478 |
ETHx | ethx | 65746878 |
BTCx | btcx | 62746378 |
NEOx | neox | 6e656f78 | not including in the testing framework
GASx | gasx | 67617378 | not including in the testing framework
Cosmos| stake | 7374616b65 | not including in the testing framework



## Router And ChainId
Type | Router Number | ChainId
:-:|:-:|:-:
Bitcoin | 1 | 1
Ethereum | 2 | 2
Ontology | 3 | 3
NEO | 4 | 4
Cosmos-gaia | 6 | 6
Switcheo | 6 | 170
