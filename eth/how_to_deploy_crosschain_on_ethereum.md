<h1 align="center">How to Deploy Crosschain on Ethereum</h1>
<h4 align="center">Version 1.0 </h4>

English | [中文](how_to_deploy_crosschain_on_ethereum_CN.md)

This document serves as a guide on how to deploy a cross chain ecosystem to transfer assets from Ethereum to other chains. The deployment process includes deploying a proxy contract, the existing ERC20 token contract being transferred, and two different methods of deploying new cross chain ERC20 token contract, i.e. implementing cross chain via the proxy contract, and directly.

## Proxy Contract Deployment and Configuration

Considering cross chain transfer from Ethereum to Ontology as an example, two proxy contracts that follow the cross chain protocol need to be deployed on both Ethereum and the Ontology chain. After deploying successfully the following configuration needs to be carried out.

The `bindProxyHash` method of the Ethereum cross chain proxy contract is used to bind the proxy contract on Ethereum to the one deployed on Ontology. The method declaration is as follows:

```js
/* @notice                  This function binds the target chain proxy contract.
*                           Only the binded proxy contract request is accept by this contract.
*  @param toChainId         The target chain id
*
*  @param targetProxyHash   The binded proxy contract address
*
*/
function bindProxyHash(uint64 toChainId, bytes memory targetProxyHash)
```

Please refer to the proxy contract deployment and configuration sections in the [Ontology guide]() for more details on how to link a proxy contract deployed on Ontology to the proxy contract on Ethereum.

## Setup Cross Chain for an Existing ERC20 Contract

The following directions describe how to setup the cross chain ecosystem for an existing ERC20 contract. The following procedure is aimed at linking to the Ontology chain as an example.

### Deploy and configure the asset contract on the target chain

First, you need to deploy an OEP-4 contract on the Ontology chain. This contract corresponds to the ERC20 asset contract on Ethereum.

After deploying the OEP-4 contract, the cross chain configuration and the link between this contract and the ERC20 contract needs to be specified in the proxy contract on Ontology. Please refer to the [OEP-4 cross chain guide]() for more details regarding the configuration.

### Configure ERC20

Define a cross chain link between ERC20 and the OEP4 contract in the proxy contract deployed on Ethereum.

This is carried out by invoking the `bindAssetHash` method of the Ethereum proxy contract.

```js
/* @notice                  This function binds the erc20 asset and eht other asset on other chain.
*                           Only the binded asset can cross chain.
*  @param sourceAssetHash         The erc20 asset hash
*
*  @param toChainId               The target chain id
*
*  @param targetAssetHash         The target asset hash on target chain
*
*  @param assetLimit              The limit of target asset used to cross chain
*
*  @param isTargetChainAsset      The indicate of sourceAssetHash, new erc20 or existing erc20
*
*/
function bindAssetHash(address sourceAssetHash, uint64 toChainId, bytes memory targetAssetHash, uint256 assetLimit, bool isTargetChainAsset)
```

## Deploy New Cross Chain ERC20 Contract Using Proxy Contract

When deploying a new cross chain ERC20 contract via a proxy contract, you can first deploy a standard ERC20 contract on Ethereum and then proceed to link and configure this contract based on the process described for an existing contract.

## Deploy a Cross Chain ERC20 Contract Directly

### Contract development

If the cross chain environment is not setup using a proxy contract, you can write an ERC20 contract that adheres to the ERC20 and the cross chain protocol. Please refer to [this]() link for a template. There are four main API methods that need to be implemented in this contract-

1. **setMinter**

The `setMinter` API method is used to set a **minter**, i.e. the management contract that has the exclusive privilege to invoke the `unlock` method. This method can only be invoked by the administrator of the ERC20 contract. The declaration is as follows:

```js
/* @notice                              Set the ETH cross chain contract as the minter such that the ETH cross chain contract
*                                       will be able to mint tokens to the designated account after a certain amount of tokens
*                                       are locked in the source chain
*  @param ethCrossChainContractAddr     The ETH cross chain management contract address
*/
function setMinter(address ethCrossChainContractAddr) onlyManager
```

Sample definition:

```js
function setMinter(address ethCrossChainContractAddr) onlyManager public {
    minter = ethCrossChainContractAddr;
    emit SetMinterEvent(ethCrossChainContractAddr);
}
```

2. **bindContractAddrWithChainId**

`bindContractAddrWithChainId` method can be invoked to link the asset contract on the target chain to this ERC20 contract. This method can only be invoked by the administrator of the ERC20 contract. The ERC20 only accepts requests from the linked contract on the target chain. The declaration is follows:

```js
/* @notice              Bind the target chain with the target chain id
*  @param chainId       The target chain id
*  @param contractAddr  The specific contract address in bytes format in the target chain
*/
function bindContractAddrWithChainId(uint64 chainId, bytes memory contractAddr) onlyManager
```

Sample definition:

```js
function bindContractAddrWithChainId(uint64 chainId, bytes memory contractAddr) onlyManager public {
	require(chainId >= 0, "chainId illegal!");
	contractAddrBindChainId[chainId] = contractAddr;
	emit BindContractAddrWithChainIdEvent(chainId, contractAddr);
}
```

3. **lock**

The `lock` method can be invoked to carry out transfers from the Ethereum chain to the target chain. The declaration is as follows:

```js
/* @notice                  This function is meant to be invoked by the user,
*                           a certin amount teokens will be burnt from the invoker/msg.sender immediately.
*                           Then the same amount of tokens will be mint at the target chain with chainId later.
*  @param toChainId         The target chain id
*
*  @param toUserAddr        The address in bytes format to receive same amount of tokens in target chain
*  @param amount            The amount of tokens to be crossed from ethereum to the chain with chainId
*/
function lock(uint64 toChainId, bytes memory toUserAddr, uint64 amount)
```

Sample definition:

```js
function lock(uint64 toChainId, bytes memory toUserAddr, uint256 amount) public returns (bool) {
    /*
	1. The parameters used to create the transaction. Sent to the unlock method of the linked asset contract on the target chain
	*/
	TxArgs memory txArgs = TxArgs({
		toAddress: toUserAddr,
		amount: amount
	});
	bytes memory txData = _serializeTxArgs(txArgs);
    /*
	2. Lock the ERC20 tokens in the contract
	*/
	require(transfer(amount), "Burn msg.sender ERC20 tokens failed");
	/*
	3. Invoke the management contract that generates a secure message that is sent to the target chain
	*/
	bytes memory method = bytes("unlock");
	EthCrossChain ecc = EthCrossChain(minter);
	require(ecc.crossChain(toChainId, contractAddrBindChainId[toChainId], method, txData), "EthCrossChain crossChain executed error!");
	/*
	4. Generate the lock event
	*/
	emit LockEvent(address(this), toChainId, contractAddrBindChainId[toChainId], txData);
	return true;
}
```

4. **unlock**

The `unlock` method is exclusively invoked by the cross chain management contract. The declaration is as follows:

```js
/* @notice                  This function is meant to be invoked by the ETH crosschain management contract,
*                           then mint a certin amount of tokens to the designated address since a certain amount
*                           was burnt from the source chain invoker.
*  @param argsBs            The argument bytes recevied by the ethereum business contract, need to be deserialized.
*                           based on the way of serialization in the source chain contract.
*  @param fromContractAddr  The source chain contract address
*  @param fromChainId       The source chain id
*/
function unlock(bytes memory argsBs, bytes memory fromContractAddr, uint64 fromChainId) onlyMinter
```

Sample definition:

```js
function unlock(bytes memory argsBs, bytes memory fromContractAddr, uint64 fromChainId) onlyMinter public returns (bool) {
    /*
	1. Arguments used to generate a cross chain transaction request
	*/
	TxArgs memory args = _deserializTxArgs(argsBs);
    /*
	2. Verify wheter the linked asset contract exists on the target chain
	*/
	require(Utils.equalStorage(contractAddrBindChainId[fromChainId], fromContractAddr), "From contract address error!");
	/*
	3. Send the locked ERC20 tokens to the specified address
	*/
	require(transfer(Utils.bytesToAddress(args.toAddress), args.amount), "mint ERCT in unlock method failed!");
	/*
	4. Generate the unlock event
	*/
	emit UnlockEvent(fromContractAddr, fromChainId, args.toAddress, args.amount);
	return true;
}
```

An OEP-4 contract that corresponds to this ERC20 contract needs to be deployed on the Ontology chain. Please refer to the Ontology cross chain OEP-4 contract development guide for more details.

### Contract configuration

After the ERC20 contract is deployed the administrator needs to invoke the `setMinter` and `bindContractAddrWithChainId` method to carry out the necessary configuration.

## License

The Poly Network library is licensed under the GNU Lesser General Public License v3.0, read the LICENSE file in the root directory of the project for details.

