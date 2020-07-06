<h1 align="center">How to Deploy Crosschain on Cosmos</h1>
<h4 align="center">Version 1.0 </h4>

[English](how_to_cross_on_cosmos.md) | 中文

本文档介绍如何在Cosmos子链中添加、设置及使用[跨链模块](https://github.com/polynetwork/cosmos-poly-module.git)包括如何在Cosmos子链的App中初始化不同跨链模块的Keeper，及设置当前链已有资产的跨链配置或设置当前链初始时不存在的资产配置。

## 对于集成这些模块的Dapp开发者，如何在App中添加跨链模块呢？
首先在`app.go`中新建这些模块的Keeper并添加到App中。
```go
// make sure crosschain keepr has the permission to change supply
var (
    maccPerms = map[string][]string{
		auth.FeeCollectorName:     nil,
		distr.ModuleName:          nil,
		mint.ModuleName:           {supply.Minter},
		staking.BondedPoolName:    {supply.Burner, supply.Staking},
		staking.NotBondedPoolName: {supply.Burner, supply.Staking},
		gov.ModuleName:            {supply.Burner},
		btcx.ModuleName:           {supply.Burner, supply.Minter},
		ft.ModuleName:             {supply.Burner, supply.Minter},
		lockproxy.ModuleName:      {supply.Minter},
	}
)
// create store key of the correlated module 
keys := sdk.NewKVStoreKeys(
    bam.MainStoreKey, auth.StoreKey, staking.StoreKey,
    supply.StoreKey, mint.StoreKey, distr.StoreKey, slashing.StoreKey,
    gov.StoreKey, params.StoreKey,
    headersync.StoreKey, ccm.StoreKey,
    btcx.StoreKey, ft.StoreKey, lockproxy.StoreKey,
)
// allocate subspace used for crosschain keeper
app.subspaces[ccm.ModuleName] = app.ParamsKeeper.Subspace(ccm.DefaultParamspace)

// create the necessary account keeper
app.accountKeeper = auth.NewAccountKeeper(app.cdc, keys[auth.StoreKey], authSubspace, auth.ProtoBaseAccount)
app.bankKeeper = bank.NewBaseKeeper(app.accountKeeper, bankSubspace, bank.DefaultCodespace, app.ModuleAccountAddrs())
app.supplyKeeper = supply.NewKeeper(app.cdc, keys[supply.StoreKey], app.accountKeeper, app.bankKeeper, maccPerms)

// create the crosschain keeper
// for syncing poly chain vital block header into this cosmos chain
app.HeaderSyncKeeper = headersync.NewKeeper(app.cdc, keys[headersync.StoreKey])
// for processing crosschain msg from both cosmos chain and poly chain
app.CcmKeeper = ccm.NewKeeper(app.cdc, keys[ccm.StoreKey], app.subspaces[ccm.ModuleName], app.HeaderSyncKeeper, app.SupplyKeeper)
// for non-virtual machine blockchain asset to be able to be crossed into current cosmos chain
app.BtcxKeeper = btcx.NewKeeper(app.cdc, keys[btcx.StoreKey], app.AccountKeeper, app.BankKeeper, app.SupplyKeeper, app.CcmKeeper)
// for some organizations to be able to provide proxy service for some assets
app.LockProxyKeeper = lockproxy.NewKeeper(app.cdc, keys[lockproxy.StoreKey], app.AccountKeeper, app.SupplyKeeper, app.CcmKeeper)
// for creating fungible token to be crossed into/from another chain 
app.FtKeeper = ft.NewKeeper(app.cdc, keys[ft.StoreKey], app.AccountKeeper, app.BankKeeper, app.SupplyKeeper, app.CcmKeeper)
//  mount btcxKeeper, ftKeeper and lockproxyKeeper into ccm (cross chain manager) module 
app.CcmKeeper.MountUnlockKeeperMap(map[string]ccm.UnlockKeeper{
    btcx.StoreKey:      app.BtcxKeeper,
    ft.StoreKey:        app.FtKeeper,
    lockproxy.StoreKey: app.LockProxyKeeper,
})
```
## 那么每一个模块分别有哪些功能或接口可以使用呢？

### `headersync`模块中函数或消息
```go
// Msg for syncing poly chain genesis header into current cosmos chain
type MsgSyncGenesisParam struct {
	Syncer        sdk.AccAddress // submitter of poly chain header
	GenesisHeader []byte         // header bytes of poly genesis header, encoding role is the same as poly chain Header.Serialization() method
}

// Msg for syncing poly chain header into current cosmos chain
type MsgSyncHeadersParam struct {
	Syncer  sdk.AccAddress      // submitter address
	Headers [][]byte            // multiple headers of poly chain block header
}
```
### `ccm`模块中函数或消息
`ccm`代表了跨链管理模块，它主要含有两个函数：
- 1. 该模块提供了`MsgProcessCrossChainTx`接口供Cosmos relayer提交来自中继链的跨链交易。
- 2. 提供了`CreateCrossChainTx`接口来供其他模块如`lockproxy`, `btcx`和`ft`模块创建从当前Cosmos子链到其他公链的跨链交易。
其中第一个接口才会对外开放，由Cosmos relayer使用。
```go
// Msg for processing cross chain msg from poly chain to current cosmos chain
// this message is designed for the cosmos-relayer to invoke
type MsgProcessCrossChainTx struct {
    Submitter   sdk.AccAddress    // cosmos-relayer account address 
	FromChainId uint64            // which chain id this cross chain msg comes from
	Height      uint32            // which height the proof is generated at poly chain 
	Proof       string            // the proof of cross chain msg coming from poly chain for current cosmos chain to obtain the actual cross chain msg
	Header      []byte            // the block header of poly chain containing the CrossStateRoot to verify the provided proof
}
```

### `btcx`模块中函数或消息
该模块的存在是为了支持一些特殊的资产如BTC，这类资产的特征是某源链没有虚拟机，不支持智能合约，所以该资产从源链跨链到Cosmos链的过程中，其原生资产如BTC是被锁定在一个多签帐户中，通过多签的方式对跨链的原生资产进行管理，其多签脚本的哈希在某种程度上可以理解为源链如BTC链上的合约哈希，该合约哈希锁定的BTC是通过该多签帐户进行跨链操作的BTC资产。

```go
// Msg for some organizations or service provider to create btc asset contract for cross chain usage.
// `Createor` creates a kind of coin with name of `Denom` and redeem script of `RedeemScript`
// The initial supply if zero, make sure the coin with name of `Denom` has never been created before, otherwise, tx will fail
type MsgCreateCoin struct {
	Creator      sdk.AccAddress     // the btc asset contract creator
	Denom        string             // the name of the btc asset contract
    RedeemScript string             // the redeem script comes from the btc blockchain redeemscript of multiple accounts in order to 
                                    // process the cross chain transaction back to btc and ideneity the unique btc asset contract in cosmos chain 
}

// this interface is designed for the denom creator to bind asset of `sourceAssetDenom` with another asset contract with hash of `ToAssetHash` in another blockchain with chain id `ToChainId`
// only the `SourceAssetDenom` creator can make the transaction successfully
type MsgBindAssetHash struct {
	Creator          sdk.AccAddress // this parameter should be the same as `Creator` in `MsgCreateCoin` struct, indicates that only the creator can make bind asset hash transaction successfully
	SourceAssetDenom string         // this parameter indicates which denom in current cosmos chain will be bonded 
	ToChainId        uint64         // this parameter indicates the `SourceAssetDenom` will be bonded with another asset hash in which blockchain 
	ToAssetHash      []byte         // this parameter indicates the `SourceAssetDenom` will be bonded with which asset hash in `ToChainId` blockchain
}

// this interface is designed for the user to do "lock" operation, meaning user (`FromAddress`) will lock `Value` amount of `SourceAssetDenom` coins into btcx module, 
// so that the address `ToAddressBs` in another blockchain with chainId (`ToChainId`) can receive the same amount of the same asset (yet in different chain).
// before user invokes this method, he/she should make sure the service provider/organization is reliable in terms of their reputation, 
// correctness of paramters (say, `ToChainId` and the related `ToAssetHash` in previous message)
type MsgLock struct {
	FromAddress      sdk.AccAddress  // the user (locker/sender) who wants to do cross chain transaction from current cosmos chain to another blockchain
	SourceAssetDenom string          // the kind of denom (coin) the user wants to send `Value` amount of `SourceAssetDenom` to "btcx" module
	ToChainId        uint64          // into which chainId, the user wants to do cross chain transaction form current cosmos chain
	ToAddressBs      []byte          // in `ToChainId`, the receiver address is `ToAddressBs` in bytes format
    Value            *sdk.Int        // the amount of `SourceAssetDenom` the user intends to lock 
                                     //   (cross from current cosmos chain to another chain with `ToChainId`)
}
```

### `ft`模块中函数或消息
这里，`ft` 代表了`fungible token`，即同质化资产。该模块具有丰富多样的接口，可供跨链服务提供商、普通用户和发行代币的项目方使用。
作为服务提供商，使用`ft`模块的方式为：
- 通过调用`MsgCreateDenom`接口来创建初始总量为零的代币，这样后续可以不通过`lockproxy`模块来实现跨链功能；具体使用的流程如下：
  - 1、创建代币的人通过`ft`模块内置的方法，即`MsgBindAssetHash`来绑定当前资产与另一条链上相同资产的资产哈希，具体参数的含义下面会介绍到。这里需要注意的是：另一条链上的相同资产一般不能通过`lockproxy`与当前链的当前资产进行绑定映射。
  - 2、跨链环境设置完毕，用户可在当前Cosmos子链接收到来自`poly chain`的跨链请求后，收到对应资产代币，而且用户可通过cosmos-sdk的`bank`模块进行转账，同时任何拥有该资产的用户也可通过`ft`的`MsgLock`接口将资产转到其他任意支持的链上。也可以通过cosmos-sdk的`supply`模块来查询代币总量，是不是很方便？

- 此外，发行代币的项目方可通过调用`MsgCreateCoins`接口来创建初始总量不为零的代币，且这些初始总量都在创建者的个人帐户中。项目方可通过cosmos-sdk的`bank`模块进行转账。也可通过依赖`lockproxy`来进行跨链操作。
```go
// this message is for some service provider/organizations to create some fungible token with zero initial supply and is designed to 
// perform cross chain transaction within `ft` module with the following two message: `MsgBindAssetHash` and `MsgLock`. 
type MsgCreateDenom struct {
	Creator sdk.AccAddress          // `Denom` creator 
	Denom   string                  // coin name being created with initial zero supply
}

// this interface is designed for the denom creator to bind asset of `sourceAssetDenom` with another asset contract with hash of `ToAssetHash` in another blockchain with chain id `ToChainId`
// only the `SourceAssetDenom` creator can make the transaction successfully
type MsgBindAssetHash struct {
	Creator          sdk.AccAddress // this parameter should be the same as `Creator` in the above `MsgCreateDenom` struct, indicates that only the creator can make bind asset hash transaction successfully
	SourceAssetDenom string         // this parameter indicates which denom in current cosmos chain will be bonded 
	ToChainId        uint64         // this parameter indicates the `SourceAssetDenom` will be bonded with another asset hash in which blockchain 
	ToAssetHash      []byte         // this parameter indicates the `SourceAssetDenom` will be bonded with which asset hash in `ToChainId` blockchain
}

// this interface is designed for the user to do "lock" operation, meaning user (`FromAddress`) will burn `Value` amount of `SourceAssetDenom` coins to air, 
// so that the address `ToAddressBs` in another blockchain with chainId (`ToChainId`) can receive the same amount of the same asset (yet in different chain).
// before user invokes this method, he/she should make sure the service provider/organization is reliable in terms of their reputation, 
// correctness of paramters (say, `ToChainId` and the related `ToAssetHash` in previous message)
type MsgLock struct {
	FromAddress      sdk.AccAddress  // the user (locker/msg sender) who wants to do cross chain transaction from current cosmos chain to another blockchain
	SourceAssetDenom string          // the kind of denom (coin) the user wants to burn `Value` amount of `SourceAssetDenom`
	ToChainId        uint64          // into which chainId, the user wants to do cross chain transaction form current cosmos chain
	ToAddressBs      []byte          // in `ToChainId`, the receiver address is `ToAddressBs` in bytes format
    Value            *sdk.Int        // the amount of `SourceAssetDenom` the user intends to do cross chain
                                     //   (from current cosmos chain to another chain with `ToChainId`)
}

// this interface is designed for everyone to create multiple coins, which are available for both `bank` and `supply` modules in cosmos-sdk. The coins should have valid format 
// in terms of coin.Denom and coin.Amount. All the initialized coins are distributed to the `Creator`.
type MsgCreateCoins struct {
	Creator sdk.AccAddress           // coins' creator
	Coins   string                   // Coins should be the format of "100000MySimpleToken,2000000MST"
}
```

### `lockproxy`模块中函数或消息

该模块具有通过跨链代理合约进行跨链交易的通用接口，其调用对象有两种：

作为服务提供商即代理商，应该进行如下操作来使用`lockproxy`模块。

    
  1. 首先通过调用`MsgCreateLockProxy`创建属于Creator自己的lockproxy代理合约。
  2. 可通过调用`MsgCreateAndDelegateCoinToProxy`创建代币并委托给可信的(也可以是提供商自己创建的`lockproxy`合约，即步骤1创建的)代理合约。此处“信任”的含义也包括信任`lockproxy`创建者在其他所有链进行的所有`BindProxyHash`类似操作。
  3. 代理商通过调用`MsgBindProxyHash`在本代理合约里设置代理商在另一条链所布署的代理合约，注：这里需要代理商在两条链上进行双向绑定设置。
  4. 代理商通过调用`MsgBindAssetHash`设置本代理合约支持跨链的资产种类及另一条链相同资产的资产哈希，注：这里需要在另一条链上代理商的代理合约中也进行相似的绑定设置。
  5. 检查：
     1. 代币创建者请求其所信任的`lockproxy`合约的创建者，确认其信任的`lockproxy`合约内已经做过了`BindProxyHash`的操作，任何人都可以通过rest客户端来查询映射关系的正确性。
     2. 代币创建者请求其所信任的`lockproxy`合约的创建者，调用`BindAssetHash`操作且在其他链上也进行相似的操作来设置不同链之间，相同资产的映射。
  6. 跨链环境设置完毕，用户可在当前Cosmos子链接收到来自`poly chain`的跨链请求后，收到对应资产代币，而且用户可通过cosmos-sdk的`bank`模块进行转账，同时任何拥有该资产的用户也可通过`lockproxy`的`MsgLock`接口将资产转到其他任意支持的链上。

作为使用跨链服务的用户，可直接调用`MsgLock`来进行跨链请求。来使用`lockproxy`模块。需要注意的是要确保`MsgLock`的发送者(`MsgLock.FromAddress`)有足够的代币，而这些代币既可以来自Cosmos子链上其他人转过来的，也可以来自其他链资产通过跨链到当前Cosmos子链上发送者(`MsgLock.FromAddress`)地址而来。

下面就详细介绍一下每个接口作用及接口中每个参数的含义。


```go
// this message is for the cross chain service provider to invoke in order to create a new lockproxy contract.
// the lockproxy contract address is identified by the creator's account address. One account can only create one lockproxy
type MsgCreateLockProxy struct {
	Creator sdk.AccAddress
}

// this message is for some service provider/organizations to create some fungible token with non-zero initial supply 
// and delegate all the coins to `lockproxy` module, which will be introduced later. This coin is prepared for the target chain asset in cosmos chain.
// Actually, there is no much difference speaking of who is the coin creator since the critical part is that the `lockproxy` operator should do "BindProxyHash" and "BindAssetHash" correctly.
// for some service provider who wishes to do cross chain transaction through the `lockproxy` module below, you should invoke this transaction to create your token/coin.
type MsgCreateAndDelegateCoinToProxy struct {
	Creator sdk.AccAddress          // coin creator
	Coin    sdk.Coin                // coin should have the format of "100000MySimpleToken" with supply of 100000 and name of "MySimpleToken"
}

// this message is for lockproxy creator to bind current lockproxy contract with another lockproxy contract in another chain.
type MsgBindProxyHash struct {
	Operator         sdk.AccAddress     // `Operator` should be the lockproxy creator
	ToChainId        uint64             // `ToChainId` indicates another chain the current lockproxy is able to reach
    ToChainProxyHash []byte             // `ToChainProxyHash` indicates the lockproxy contract hash in another chain that
                                        //  can reach the current lockproxy contract.
}

// this message is for lockproxy creator to bind current asset denom `SourceAssetDenom` with the same asset 
// with hash of `ToAssetHash` yet in another chain with `ToChainId`. The initial amount prepared for cross chain transaction 
// is `InitialAmt`, meaning `InitialAmt` of `SourceAssetDenom` should already in `lockproxy` module account.
// Now we can see, this message should be invoked after `MsgCreateAndDelegateCoinToProxy` of `ft` module is invoked.
type MsgBindAssetHash struct {
	Operator         sdk.AccAddress  // `Operator` should be the lockproxy creator
	SourceAssetDenom string          // `SourceAssetDenom` is the coin kind correlated with the asset of `ToAssetHash` in `ToChainId` blockchain
	ToChainId        uint64          // another chain with `ToChainId` that holds the `ToAssetHash`, and are supported by the poly chain
	ToAssetHash      []byte          // the asset hash existing in `ToChainId` chain 
	InitialAmt       sdk.Int         // the initial amount of `SourceAssetDenom` that the `lockproxy` module account holds
}

// this interface is designed for the user to do "lock" operation, meaning user (`FromAddress`) will burn `Value` amount of `SourceAssetDenom` coins to air, 
// so that the address `ToAddressBs` in another blockchain with chainId (`ToChainId`) can receive the same amount of the same asset (yet in different chain).
// before user invokes this method, he/she should make sure the service provider/organization is reliable in terms of their reputation, 
// correctness of paramters (say, `ToChainId` and the related `ToAssetHash` in previous message)
type MsgLock struct {
    LockProxyHash    []byte          // current lockproxy the FromAddress wants to use to do cross chain transaction
	FromAddress      sdk.AccAddress  // the user (locker/msg sender) who wants to do cross chain transaction from current cosmos chain to another blockchain
	SourceAssetDenom string          // the kind of denom (coin) the user wants to burn `Value` amount of `SourceAssetDenom`
	ToChainId        uint64          // into which chainId, the user wants to do cross chain transaction form current cosmos chain
	ToAddressBs      []byte          // in `ToChainId`, the receiver address is `ToAddressBs` in bytes format
    Value            *sdk.Int        // the amount of `SourceAssetDenom` the user intends to do cross chain
                                     //   (from current cosmos chain to another chain with `ToChainId`)
}
```
### 跨链流程

[这里](cosmos_cross_chain_workflow_CN.md)是跨链操作的详细流程。

### 例子
有关资产跨链的协议例子可参考[PIP-1](https://github.com/polynetwork/PIPs/blob/master/PIPS/pip-1.md)。

## 许可证

Poly Network遵守GNU Lesser General Public License, 版本3.0。 详细信息请查看项目根目录下的LICENSE文件。
