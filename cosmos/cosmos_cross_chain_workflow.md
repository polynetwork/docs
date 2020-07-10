<h1 align="center">Demonstration of Cross Chain Flow</h1>
<h4 align="center">Version 1.0 </h4>

English | [中文](cosmos_cross_chain_workflow_CN.md)

## Cross Chain Introduction
Cross chain, in a nutshell, is transferring asset from some account on public chain A to another account in public chain B. Through Poly chain, we can do cross chain transaction between both homogeneous public chains and heterogeneous public chains. Next, let us take a look at some additional concepts to help us understand the mechanism more conveniently.

#### Source Chain and Target Chain
Source chain and target chain are relative to a specific asset. A public chain is the source chain for a specific asset if the asset firstly appear in this public chain. Relatively, if we want to cross this asset into another public chain, and we issue a new token on this public chain, then this public chain is the relative target chain. For instance:
```markdown
For the ether asset, Ethereum is the source chain. When we want to issue a new OEP4 token in 
Ontology network that represents the ether, Ontology chain will be the target chain for 
the ether asset.
Vice versa, For the ONT token, Ontology chain is the source chain. When we want to issue
a new ERC20 token in Ethereum chain, Ethereum would be the target chain for Ont network.
```
#### Asset Hash on Cosmos subchain

There is no virtual machine in Cosmos subchain, however, we can issue new coins through a specific interface in [cosmos-poly-module](https://github.com/polynetwork/cosmos-poly-module.git). To name it, `MsgCreateCoin` in `btcx` module, `MsgCreateAndDelegateToProxy` in `lockproxy` module and `MsgCreateDenom` in `ft` module.

Before we introduce these interfaces and usage, let us know more detail about the asset hash on Cosmos subchain. Different from the asset contract hash on Ethereum, Ontology, to differentiate from coins in Cosmos subchain, there is a filed named `Denom` in cosmos-sdk `Coin` struct. For each account, his or her balance of all assets are recorded within `Coins` struct, containing all the asset the account owns in Cosmos subchain. Another field of `Coin` struct denotes how many coins the owner owns. In the flow of cross chain transaction through Poly chain, the asset hash of a specific coin is the hex format of bytes directly derived from the `Denom` string.

## Cosmos Cross Chain Specification

The following table explain the details amount different interfaces to create new denom.

Which Interface used to create coin | Initial Supply | who has initial supply | Need `lockproxy` module  | Asset Hash | BindAssetHash | How to cross 
--------- | --- | ----------- | -------------- | ------- | ------- | ---------
`btcx.MsgCreateDenom`  | Zero | None |  No | [Hex format](https://github.com/polynetwork/cosmos-poly-module/blob/master/btcx/internal/keeper/keeper.go#L104) of `Denom` | `btcx.BindAssetHash`  | `btcx.MsgLock`
`lockproxy.MsgCreateAndDelegateCoinToProxy` | Non-Zero | `lockproxy` module | Yes  | Hex format of `Denom` | `lockproxy.BindAssetHash` | `lockproxy.MsgLock` 
`ft.MsgCreateDenom` | Zero | None | Yes  | Hex format of `Denom` | `ft.BindAssetHash` | `ft.MsgLock`
`ft.MsgCreateCoins` | Non-Zero | Coin Creator | `bank.MsgSend`  | Hex format of `Denom` | No need | No need 

As for the usage and scenario of each message:
- `btcx.MsgCreateDenom`: This message serves for cross chain asset on target chain, like BTC asset in Cosmos subchain, the functionality is to issue a new coin with zero initial supply.
- `lockproxy.MsgCreateAndDelegateCoinToProxy`：This message serves for `lockproxy` module, the functionality is to issue a new type of coin with non-zero initial supply and transfer all the coins to the `lockproxy` module account. And the reason we transfer all the coins to the `lockproxy` module account is that this coin acts as the target chain asset, and needs to be unlocked from the module account to the receiver when the cosmos relayer submits the cross chain transaction proof and Poly chain header.
- `ft.MsgCreateDenom`：This message serves for cross chain transaction for those who do not trust `lockproxy` and wish to bypass the `lockproxy` and implement the cross chain logic within asset contract. Under this circumstance, we can issue a new coin with zero initial supply and map the current asset hash with the source chain asset contract hash through `ft.MsgBindAssetHash` interface.
- `ft.MsgCreateCoins`：This message serves for users who wish to issue some coins with non-zero initial supply. Once the coins are issued, they act as independently from the `cosmos-poly-module`. We can choose to do cross chain transaction through `lockproxy` or just transfer between cosmos subchain accounts utilizing `bank` module in cosmos-sdk.

It should be noted that once a denom or coin named "mst" is created, nobody within current Cosmos subchain can create a new denom or coin with the same name. The reason behind that is the `supply` module records all the global coins, and the additional asset cannot have the same denom with existing coins.

## Cross Chain Flow and Sequence
#### Send Btc from BTC BlockChain to Cosmos subchain
* The flow of crossing BTC asset from BTC chain to Cosmos subchain
<div align=center><img width="1093" height="894" src="pic/btc2cosmos_flow.png"/></div>

#### Send Ont from Ontology to Cosmos subchain
* The flow of crossing ONT token from Ontology chain to Cosmos subchain
<div align=center><img width="1158" height="1093" src="pic/ont2cosmos_flow.png"/></div>

## License
The Poly Network library is licensed under the GNU Lesser General Public License v3.0. Please refer to the LICENSE file in the root directory of the project for details.