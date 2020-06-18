# TetNet | [DevNet](README_DevNet.md) 

This's cross chain asset contract info, it's used to bind asset mapping in different chain, if the name ends with (s) on behalf of the asset is mapping.

## Ethereum

Type | Contract Hash | Desc
---|---|---
CCMP | 0xC8042579D6b60E0e35161F228827E3Fa0F51d5B6 | Cross Chain Manager Proxy contract hash 
Lock Proxy | 0xfD22e3EAD9f7B9334dF52f79b607f99C3dBAb37c | The lock proxy bridge contract hash for asset not implementing the "lock" and "unlock" logic to do cross chain tx
ERC20 | 0x6C57893347Ff46F0aa3CA935512199421E52DBb3 | ERC20 template contract hash in Ethereum chain
OEP4x | 0xBdf64af92dbbF6494bB19AaF85069099a2DDFE64 | OEP4x template contract hash in Ethereum chain
ONGx | 0xA0006a6cAafDa851E7706C4919E4F3BC3E058209 | ONGx contract hash in Ethereum chain
ONTx | 0xD2B45C29E66f08937753DE346E78243D949BaC65 | ONTx contract hash in Ethereum chain
ETH | 0x0000000000000000000000000000000000000000 | The asset hash that we treat as the Ether asset
BTCx | 0x700CA49ccA3803316124D2A8a44498ABB3E9cF51 | Btcx contract hash corresponding with unique btc redeem script
NEOx | 0x20f307ea523E69d195b3a370fe6496Eb50ce281a | Neox contract hash in Ethereum chain

## Ontology

Type | Contract Hash | Desc
---|---|---
Lock Proxy | B:dd43e07db99728b6188335b84bf176da758b4750 </br> L: 50478b75da76f14bb8358318b62897b97de043dd | The bridge contract hash for asset not implementing the "lock" and "unlock" logic to do cross chain tx
ERC20x | B: 35e123f9816d05917cf79ef07eb17908ff970c7e </br> L: 7e0c97ff0879b17ef09ef77c91056d81f923e135 | ERC20 template contract hash in Ontology chain
OEP4 | B: 25ed838b45b0dc19ee453fb68e55df85741b9899 </br> L: 99981b7485df558eb63f45ee19dcb0458b83ed25 | OEP4 template contract hash in Ontology chain
ONG | B: 0200000000000000000000000000000000000000 </br> L: 0000000000000000000000000000000000000002 | ONG asset hash in Ontology chain
ONT | B: 0100000000000000000000000000000000000000 </br> L: 0000000000000000000000000000000000000001 | ONT asset hash in Ontology chain
ETHx | B: 79e111161de47207f888f4d4ecba7cad16450108 </br> L: 08014516ad7cbaecd4f488f80772e41d1611e179 | Ethx asset hash in Ontology chain
BTCx | B: 837d0a836b3bee3ebad985d61dde64167198f3b7 </br> L: b7f398711664de1dd685d9ba3eee3b6b830a7d83 |  Btcx contract hash in Ontology chain
CNEOx | B: 4f8d1df75afcab1ed5a361ab45c6376d2129d12b </br> L: 2bd129216d37c645ab61a3d51eabfc5af71d8d4f |  CNEOx contract hash in Ontology chain
CGASx | B: 7b71a5b519c96b403db233631579ab3ef32c311c </br> L: 1c312cf33eab79156333b23d406bc919b5a5717b | CGasx contract hash in Ontology chain
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


## Router And ChainId
Type | Router Number | ChainId
---|---|---
Bitcoin | 1 | 1
Ethereum | 2 | 2
Ontology | 3 | 3
NEO | 4 | 4
Switcheo | 6 | release soon
