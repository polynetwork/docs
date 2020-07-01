<h1 align="center">How to Deploy Crosschain on Cosmos</h1>
<h4 align="center">Version 1.0 </h4>

English | [中文](how_to_cross_on_cosmos_CN.md)

This article mainly introduces how to import, set up and employ the available [modules](https://github.com/polynetwork/cosmos-poly-module.git) in your cosmos application, including how to initialize different keepers of different modules and configure existing asset or non-existing asset in order to carry out cross chain operations through Poly chain Poly chain.

## How to integrate these cross chain modules for developers to implement cross chain functionality?

Firstly, add these created module keepers to your decentralized application.
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
## What method, or interface or message can be used in each module?

### Messages in `headersync` module
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
### Messages in `ccm` module
Here, `ccm` means "cross chain manager" or "cross chain management". It contains two functions:

- 1. It provides interface for Cosmos relayer to submit cross chain transaction coming from Poly chain. While Cosmos relayer monitoring the poly chain, once it detects a Poly chain native event aiming for current Cosmos subchain, it will get the proof from ledger store of Poly chain and construct a `MsgProcessCrossChainTx` to request for the completion of cross chain transaction on the Cosmos subchain side. Within `ccm` module, the submitted header will be verified first. The `CrossStateRoot` within verified Poly chain header will be used to verify the proof. Then the actual cross chain message or transaction can be obtained. The `ccm` module then checks which module should be responsible for processing the cross chain message and execute the "unlock" logic. Currently, there are three modules can process "unlock" logic, including `lockproxy`, `btcx` and `ft`, which will be introduced later.
- 2. It provides the `CreateCrossChainTx` interface for `lockproxy`, `btcx` and `ft` modules to create cross chain transaction from current Cosmos subchain to another public chains, say, Ethereum, BTCX, Ontology, or NEO. Within this interface, there will be a unique key and immutable value consisted of current cross chain transaction stored in the ledger to represent the cross chain transaction existing in Cosmos subchain side are completed successfully.

Now we can easily see, `MsgProcessCrossChainTx` is for Cosmos relayer to relay cross chain tx to Cosmos subchain. And `CreateCrossChainTx` function is for other modules to construct cross chain tx from Cosmos subchain. 
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

### Messages in `btcx` module
This module supports a specific group of asset like BTC, this group of asset usually exist in a POW pulic chain without virtual machine and smart contract. When the asset crossed from its original public chain to Cosmos subchain, the native asset like BTC is usually locked within a multi-signature account in the original public chain. The asset is managed through multiple signature, and the hash of redeem script (called redeem key or redeem hash) can be interpreted as the contract hash. The amount of asset like BTC locked through this specific redeem key is that crossed through the specific multi-signed account address.

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

### Messages in `ft` module

Here, `ft` denotes the `fungible token`, like the asset follows [OEP4 protocol](https://github.com/ontio/OEPs/blob/master/OEPS/OEP-4.mediawiki) or [ERC20 protocol](https://eips.ethereum.org/EIPS/eip-20). This module contains many interfaces, which can be used by the cross chain service providers, users or some project dev team wishing issuing new tokens.
As cross chain service provider, the process to use `ft` module is as follow.

- Create zero initial supply through `MsgCreateDenom` interface, such that the created token can bypass `lockproxy` module to realize cross chain functionality. In detail,
  - 1. The denom creator or token creator can bind current denom with another asset contract hash existing on another public chain through `MsgBindAssetHash`, the notation of each element of this struct is introduced later. To be noticed, in the target public chain, it's better to implement "lock" and "unlock" logic and "bindAssetHash" within asset contract rather than map asset hash with the Cosmos denom hash through `lockproxy' contract.
  - 2. Once the mapping between current asset hash and the target asset hash is established on both Cosmos subchain and other chains, users can receive designated asset (denom or token) after cosmos relayer submits cross chain transaction from Poly chain to current Cosmos subchain. In addition, users can do transfer between any users within Cosmos subchain through the native module `bank`. Meanwhile, any user holding non-zero tokens can cross his or her tokens from current Cosmos subchain to any other public chains supported by Poly chain through `MsgLock` in `ft` module. And of course, any body can check the total supply of this token within current cosmos subchain through the native `supply` module. Isn't it very convenient?

- Besides, the new coins (tokens) issuer can create non-zero initial supply through `MsgCreateCoins` interface within `ft` module, and the initial owner of these coins will be the creator. The issuer can transfer these tokens through `bank` module, or do cross chain transaction throught `lockproxy`.
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

This module contains common interface for cross chain request through `lockproxy` module. There are two types of invokers separated by the roles.

As the service provider or lockproxy provider, the instruction to use `lockproxy` module are the followings.
  1. The `lockproxy` creator (service provider) invoke `MsgCreateLockProxy` to create the globally unique in current cosmos subchain and his or her own lockproxy contract.
  2. The denom creator can invoke `MsgCreateAndDelegateCoinToProxy` to create non-zero initial supply tokens and delegate them to the reliable (or his own `lockproxy` contract created as the above step). Here "reliable" also including trusting the `lockproxy` contracts in all the other public chains issued by current `lockproxy` creator and their operation of binding or mapping the asset hashes.
  3. The `lockproxy` creator sets up the map through `MsgBindProxyHash` interface from current `lockproxy` contract hash (same as the creator address in hex format) to the correlated `lockproxy` contract hash in another public chain if it has not been set up before.
  4. The `lockproxy` creator sets up the map through `MsgBindAssetHash` interface from current asset hash (hex format of bytes from denom) to the same asset hash deployed in another public chain. Note that the creator should also do bindAssetHash in another side of public chain.
  5. Condition check:
     1. Coin creator request the reliable `lockproxy` creator to confirm the `BindProxyHash` operation is completed. And anybody can check if the mapping relation between `lockproxy` contract hash is correct through rest client.
     2. Coin creator request the reliable `lockproxy` creator to confirm the `BindAssetHash` operation is completed. And anybody can check if the mapping relation between asset contract hash is correct through rest client.
  6. After the configuration of cross chain environment is completed correctly, users can receive coins unlocked from `lockproxy` module account when the cosmos relayer submit the cross chain proof generated on Poly chain and Poly chain block header containing the state root of that proof. Then any user holding non-zero coins can transfer their coins to others through `bank` module within cosmos-sdk or do cross chain transaction to other public chains through `MsgLock` within `lockproxy` module.

As users employing the cross chain service, they can do cross chain transaction through `lockproxy.MsgLock` directly, before which it should be noted that they should have enough coins, which can be either those transferred from other accounts or those unlocked from `lockkproxy` module account due to the cross chain transaction submission by cosmos relayer.

Next we explain each part of the struct in detail.


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

### Cosmos Cross Chain Workflow

[Here](cosmos_cross_chain_workflow.md) explains the cross chain operation in detail.

## License

The Poly Network library is licensed under the GNU Lesser General Public License v3.0. Please refer to the LICENSE file in the root directory of the project for details.
