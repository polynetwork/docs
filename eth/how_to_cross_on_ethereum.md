<h1 align="center">Cross Chain Transactions on Ethereum</h1>
<h4 align="center">Version 1.0 </h4>

English | [中文](how_to_cross_on_ethereum_CN.md)

In this document, we will be discussing cross chain transactions from the Ethereum network. We will mainly be focusing on how transfer the cross chain BTC from Ethereum back to Bitcoin network and how to send ETH and other **ERC-20** tokens to the Ontology network.

## Sending Cross Chain BTC back to Bitcoin Network

### Setup

The BTC cross chain service contract deployed on Ethereum handles the cross chain asset exchange between the Bitcoin network and Ethereum. Ensure the contract is deployed on Ethereum before carrying out BTC cross chain transactions. Transferring the cross chain BTC back to the Bitcoin network would require the user to a have a certain amount of cross chain BTC balance in their account on Ethereum.

### Transfer

The `lock` method of the service contract is invoked to send the cross chain BTC back to Bitcoin network. The method is structured as follows:

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

| Parameter  | Description                                           |
| ---------- | ----------------------------------------------------- |
| toChainId  | Target chain ID in the consortium chain, Bitcoin is 1 |
| toUserAddr | Receiver `Base58` format address on Bitcoin network   |
| amount     | BTC amount to be transferred, in satoshis             |


The transfer can also be carried out using the `go-ethereum` SDK `SendTransaction` method. The sample code is as follows:

```go
func InvokeCross2Btc(cfg *config.ServiceConfig) bool {
	ethclient, err := ethclient.Dial(cfg.RestURL)
	if err != nil {
		fmt.Errorf("InvokeCross2Btc - cannot dial sync node, err: %s", err)
		return false
	}
	contractabi, err := abi.JSON(strings.NewReader(contractbtc.BTCTokensABI))
	if err != nil {
		fmt.Errorf("InvokeCross2Btc - err:" + err.Error())
		return false
	}
	txData, err := contractabi.Pack("lock", uint64(cfg.BindBtcChainId), []byte(cfg.Cross2BtcAddress), uint64(cfg.Cross2BtcAmount))
	if err != nil {
		fmt.Errorf("InvokeCross2Btc - err:" + err.Error())
		return false
	}
	contractAddr := common.HexToAddress(cfg.BtcContractAddress)
	nonce := nonceManager.GetAddressNonce(common.HexToAddress(cfg.Signer))
	tx := types.NewTransaction(nonce, contractAddr, big.NewInt(0), uint64(5000000), big.NewInt(30000000000), txData)
	bf := new(bytes.Buffer)
	rlp.Encode(bf, tx)
	ethsigner := tools.NewETHSigner(cfg)
	rawtx := hexutil.Encode(bf.Bytes())
	signedtx, err := ethsigner.SignRawTx(rawtx)
	if err != nil {
		fmt.Errorf("InvokeCross2Btc - sign raw tx error: %s", err.Error())
		return false
	}
	err = ethclient.SendTransaction(context.Background(), signedtx)
	if err != nil {
		fmt.Errorf("InvokeCross2Btc - send transaction error:%s\n", err.Error())
		return false
	}
	return true
}
```

The user may invoke the `lock` method using **remix** to send the BTC back to the Bitcoin chain. However, the user may first need to setup the remix environment to be able to do so. The following sample code illustrates the invocation of `lock` using remix.

![](pic/eth_cross_2_btc.png)

## Sending ETH to Ontology

### Setup

The proxy contract deployed on Ethereum handles the cross chain requests for transfers to other chains, and that is why it is necessary to ensure that the proxy contract is deployed correctly and functioning. Also, confirm the chain ID. Here, Ontology's chain ID is 3.

### Sending a Transaction

Sending ETH from Ethereum to other chains is carried out by invoking the `lock` method of the proxy contract. The method declaration in the proxy contract is as follows:

```js
/* @notice                  This function is meant to be invoked by the user,
*                           a certin amount teokens will be locked in the proxy contract the invoker/msg.sender immediately.
*                           Then the same amount of tokens will be unloked from target chain proxy contract at the target chain with chainId later.
*  @param sourceAssetHash   The asset hash in current chain
*  @param toChainId         The target chain id
*
*  @param toAddress         The address in bytes format to receive same amount of tokens in target chain
*  @param amount            The amount of tokens to be crossed from ethereum to the chain with chainId
*/
function lock(address sourceAssetHash, uint64 toChainId, bytes memory toAddress, uint256 amount)
```

| Parameter       | Description                                                  |
| --------------- | ------------------------------------------------------------ |
| sourceAssetHash | Transfer asset hash, For ETH set to 0000000000000000000000000000000000000000 |
| toChainId       | Target chain ID, for e.g. Ontology is 3                      |
| toAddress       | Receiving account address on the target chain                |
| amount          | Transfer amount of the transaction                           |


The same process can be carried out using the **go-ethereum** SDK.

```go
func sendEthCrossOnt(index int) bool {
	gasPrice, err := ethclient.SuggestGasPrice(context.Background())
	if err != nil {
		fmt.Errorf("sendEthCrossOnt - get suggest sas price failed error: %s", err.Error())
		return false
	}
	contractabi, err := abi.JSON(strings.NewReader(lock_proxy_abi.LockProxyABI))
	if err != nil {
		fmt.Errorf("sendEthCrossOnt - err:" + err.Error())
		return false
	}
	cross2OntAddress := "ARiwjLzjzLKZy8V43vm6yUcRG9b56DnZtY"
	if err != nil {
		fmt.Errorf("sendEthCrossOnt - err:" + err.Error())
		return false
	}
	assetaddress := common.HexToAddress("0000000000000000000000000000000000000000")
	txData, err := contractabi.Pack("lock", assetaddress, uint64(config.ONT_CHAIN_ID), cross2OntAddress[:], big.NewInt(int64(cfg.Cross2OntAmount)))
	if err != nil {
		fmt.Errorf("sendEthCrossOnt - err:" + err.Error())
		return false
	}
	contractAddr := common.HexToAddress(cfg.ProxyContract)
	signerAccount := ethSigner.GetAccount(index)
	callMsg := ethereum.CallMsg{
		From: signerAccount.Address, To: &contractAddr, Gas: 0, GasPrice: gasPrice,
		Value: big.NewInt(int64(cfg.Cross2OntAmount)), Data: txData,
	}
	gasLimit, err := ethclient.EstimateGas(context.Background(), callMsg)
	if err != nil {
		fmt.Errorf("sendEthCrossOnt - estimate gas limit error: %s", err.Error())
		return false
	}
	nonce := nonceManager.GetAddressNonce(signerAccount.Address)
	tx := types.NewTransaction(nonce, contractAddr, big.NewInt(int64(cfg.Cross2OntAmount)), gasLimit, gasPrice, txData)
	bf := new(bytes.Buffer)
	rlp.Encode(bf, tx)
	rawtx := hexutil.Encode(bf.Bytes())
	signedtx, err := ethSigner.SignRawTx(rawtx, index)
	if err != nil {
		fmt.Errorf("sendEthCrossOnt - sign raw tx error: %s", err.Error())
		return false
	}
	err = ethclient.SendTransaction(context.Background(), signedtx)
	if err != nil {
		ftm.Errorf("sendEthCrossOnt - send transaction error:%s", err.Error())
		return false
	}
	return true
}
```

The process remains the same when sending a transaction by invoking `lock` method **remix**.

## Sending ERC20 Tokens to Ontology

### Initial Setup

The proxy contract deployed on Ethereum handles the cross chain requests for transfers to other chains. ERC20 transfer request are also processed by the proxy contract. Before proceeding, ensure that the ERC20 asset contract adheres to the cross chain protocol and has been configured and deployed on the Ethereum chain.

Please refer to the sample code below for sending ERC20 tokens to Ontology (Chain ID 3).

### Sending a Transaction

Sending ERC20 tokens to another chain involves two steps:

1. `approve` the ERC20 tokens to the proxy contract

The ERC20 amount that is sent to the proxy contract using `approve` can be transferred to other chains. The `approve` method of an ERC20 token that follows the cross chain protocol is defined in the following manner:

```js
/**
 * @dev See {IERC20-approve}.
 *
 * Requirements:
 *
 * - `spender` cannot be the zero address.
 */
function approve(address spender, uint256 amount)
```

| Parameter | Description            |
| --------- | ---------------------- |
| address   | Proxy contract address |
| amount    | Transfer amount        |


2. Invoke the `lock` method of the proxy contract to send the transaction

The `lock` method of the proxy contract used to send ERC20 resources to other chain is defined as follows:

```js
/* @notice                  This function is meant to be invoked by the user,
*                           a certin amount teokens will be locked in the proxy contract the invoker/msg.sender immediately.
*                           Then the same amount of tokens will be unloked from target chain proxy contract at the target chain with chainId later.
*  @param sourceAssetHash   The asset hash in current chain
*  @param toChainId         The target chain id
*
*  @param toAddress         The address in bytes format to receive same amount of tokens in target chain
*  @param amount            The amount of tokens to be crossed from ethereum to the chain with chainId
*/
function lock(address sourceAssetHash, uint64 toChainId, bytes memory toAddress, uint256 amount)
```
| Parameter       | Description                           |
| --------------- | ------------------------------------- |
| sourceAssetHash | ERC20 asset hash                      |
| toChainId       | Target chain ID, Ontology is 3        |
| toAddress       | Receiving address on the target chain |
| amount          | Transfer amount                       |

## License

The Poly Network library is licensed under the GNU Lesser General Public License v3.0. Please refer to the LICENSE file in the root directory of the project for details.
