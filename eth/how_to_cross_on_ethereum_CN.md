<h1 align="center">How to Cross on Ethereum</h1>
<h4 align="center">Version 1.0 </h4>

[English](how_to_cross_on_ethereum.md) | 中文

这里介绍怎么使用以太的跨链平台，包括三个部分，如何将跨链到以太的btc发送回BTC链，如何将eth发送到ontology，如何发送erc20到ontology。

## 将跨链到以太的btc发送回BTC链

### 准备

开始前需要具备以下几个条件：

- 确保以太上成功部署了BTC跨链业务合约，以及合约中BTC的余额；
- 用户的以太账户中有足够的ETH，用来调用智能合约，返回BTC；
- 准备好比特币地址来接收返回的BTC，跨链返回的金额最好不要太小，避免无法覆盖手续费的情况；

### 发送

在以太上的BTC都存储在一本智能合约中，用户调用合约的lock接口，进而调用以太的跨链管理合约，返回跨链事件，跨链生态会把事件中的信息提炼出来，组成BTC的解锁交易，这是比特币网络的交易，需要供应商完成多签签名，然后这笔交易会被中继链返回，进而广播到比特币网络，最终BTC回到用户手里，不过这笔交易的手续费需要用户支付。

lock接口如下：

```
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
```
toChainId           - BTC为1
toUserAddr          - BTC上账户地址，base58格式的btc地址
amount              - BTC金额，以聪为单位
```

下面是go语言版本的调用示例：

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
用户也可以在remix上调用lock来发送btc到BTC链，前提是准备好remix环境，下面是remix上调用lock合约的示例：

 ![](pic/eth_cross_2_btc.png)

## 将eth发送到ontology

### 准备

跨链条件如下：

- 确保以太的跨链代理合约已经部署到以太；
- 确保本体和以太上部署了ETH的代理合约；
- 准备好用来接收ETH的本体地址；

### 发送

用户调用ETH代理合约的lock方法并转入一定量的ETH，这些ETH将被锁定并开始跨链，以太的跨链管理合约会返回包含跨链信息的事件，跨链生态会转发这些信息到本体链的跨链管理合约，在本体的ETH代理合约上为用户增发ETH，这样锁定的ETH就到达了本体链。

代理合约的lock接口如下：

```
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
```
sourceAssetHash              -    eth的资产hash指定为0000000000000000000000000000000000000000
toChainId                    -    目标链的id，跨链到ontology为例，id为3
toAddress                    -    目标链账户的地址
amount                       -    发送的金额
```

下面是go语言版本的调用示例：
```
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
用户也可以通过remix调用lock来完成交易发送，和上面类似，不在示例。

## 如何发送erc20到ontology

### 准备

开始之前需要具备以下条件：

- 本体和以太上分别部署了ERC20的跨链代理合约，并完成了相关配置；
- 准备好本体的地址，用来接收跨链的ERC20；

### 发送

发送erc20到其他链与发送ETH类似，主要不同的是，用户需要先调用ERC20合约先Approve代理合约使用一定量的ERC20，然后则可以调用代理合约的lock接口，锁定ERC20并调用以太的跨链管理合约，最终会在本体上的ERC20代理合约上增发代币。

1. approve erc20到代理合约

approve到代理合约的erc20可以被代理合约跨链发送到其他链，遵循跨链协议的erc20合约的approve接口如下：

```
/**
 * @dev See {IERC20-approve}.
 *
 * Requirements:
 *
 * - `spender` cannot be the zero address.
 */
function approve(address spender, uint256 amount)
```
```
address                      -    代理合约地址
amount                       -    金额
```

调用示例和上面类似，不再示例。

2. 调用代理合约lock完成发送

发送erc20到其他链，如ontology，是通过调用代理合约的lock方法来完成。代理合约的lock接口如下：

```
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
```
sourceAssetHash              -    erc20的资产hash
toChainId                    -    目标链的id，跨链到ontology为例，id为3
toAddress                    -    目标链账户的地址
amount                       -    发送的金额
```
调用示例和上面类似，不再示例。

## 许可证

Poly Network遵守GNU Lesser General Public License, 版本3.0。 详细信息请查看项目根目录下的LICENSE文件。
