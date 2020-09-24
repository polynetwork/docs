



<h1 align=center>How to Transfer ETH to Ontology</h1>

## Smart Contracts on Both Side

When you need to cross-chain ETH to Ontology, you need prepare smart contracts on both side. We suggest you are using **Poly testnet**.

- **On Ethereum side**: The contract `LockProxy` on Ethereum is necessary. Check demo [here](https://github.com/polynetwork/eth-contracts/blob/master/contracts/core/lock_proxy/LockProxyPip1.sol)  and read the [document](../eth/how_to_deploy_crosschain_on_ethereum.md) to understand the contract. After deploy, you must call `SetManagerProxy` to set ECCMP to your lockproxy. You can find the testnet contracts setting [here](../config/README.md). 
- **On Ontology side**: First deploy your lockproxy on Ontology and here is [demo](https://github.com/polynetwork/ont-contracts/blob/master/contracts/core/lockproxy/lock_proxy.py). And there must be a [contract](https://github.com/polynetwork/ont-contracts/blob/master/contracts/core/assets/ethx.py) on Ontology to receive ETH. You can deploy contracts according to these documents [ont](../ont/How_to_new_cross_chain_asset.md). Before deploying, you need to change the `Operator` to yourself in contracts. Call `delegateToProxy` of ETHX to delegate enough ETHX for cross-chain test. And call `bindAssetHash` of lockproxy on Ontology to bind ETH with ETHX. 
- **Setup contracts**: 
  - Call `bindProxyHash` of lockproxy on Ontology with parameters `{2, raw_hash_bytes_of_eth_lockproxy}`. 
  - Call `bindProxyHash` of lockproxy on Ethereum with parameters `{3, raw_hash_bytes_of_ont_lockproxy}`.
  - Call `bindAssetHash` of lockproxy on Ethereum with parameters `{ETHAssetHash, 3, ont_ethx_raw_hash}`.

Of course you can using our lock proxy `0xE44dDbAb7a9Aa11b9D0cA8b686F9E0299DB735d1` on Ethereum and `34a593c5ccfb8590e9a4bc7018529016aa4a55ad` on Ontology. ETHX on Ontology is `ec2e25c4c12371ca37b129477a23a152e62ff103`. No need to setup.

Of course you need to prepare ontology account and ethereum account yourself.

## Lock ETH to LockProxy

You need to lock some ETH to contract LockProxy. And then the Poly cross-chain Ecosystem will do the rest work. 

The follow code shows you how to lock your ETH to `LockProxy` on Ethereum Ropsten testnet. 

```go
import (
	"bytes"
	"context"
	"fmt"
	"github.com/ethereum/go-ethereum"
	"github.com/ethereum/go-ethereum/accounts/abi"
	ethcommon "github.com/ethereum/go-ethereum/common"
	"github.com/ethereum/go-ethereum/common/hexutil"
	"github.com/ethereum/go-ethereum/core/types"
	"github.com/ethereum/go-ethereum/crypto"
	"github.com/ethereum/go-ethereum/ethclient"
	"github.com/ethereum/go-ethereum/rlp"
	"github.com/ontio/ontology/common"
	"github.com/polynetwork/eth-contracts/go_abi/lock_proxy_abi"
	"math/big"
	"strings"
	"time"
)

func SendEthCrossOnt(amount int64) error {
	ethclient, err := ethclient.Dial("ethereum_rpc_address")
	if err != nil {
		return err
	}
	gasPrice, err := ethclient.SuggestGasPrice(context.Background())
	if err != nil {
		return fmt.Errorf("SendEthCrossOnt, get suggest gas price failed error: %s", err.Error())
	}
	gasPrice = gasPrice.Mul(gasPrice, big.NewInt(5))

	contractabi, err := abi.JSON(strings.NewReader(lock_proxy_abi.LockProxyABI))
	if err != nil {
		return fmt.Errorf("SendEthCrossOnt, abi.JSON error:" + err.Error())
	}

	ontAddr, err := common.AddressFromBase58("your_ont_address")
	if err != nil {
		return err
	}
	assetaddress := ethcommon.HexToAddress("0000000000000000000000000000000000000000")
	txData, err := contractabi.Pack("lock", assetaddress, 3, ontAddr[:], big.NewInt(amount))
	if err != nil {
		return fmt.Errorf("SendEthCrossOnt, contractabi.Pack error:" + err.Error())
	}

	contractAddr := ethcommon.HexToAddress("your_lockproxy")
	// prepare eth account
	priKey, err := crypto.HexToECDSA("eth_private_key")
	if err != nil {
		return fmt.Errorf("SendEOntCrossOnt, cannot decode private key")
	}
	address := crypto.PubkeyToAddress(priKey.PublicKey)
	nonce, err := ethclient.PendingNonceAt(context.Background(), address)
	if err != nil {
		return err
	}
	callMsg := ethereum.CallMsg{
		From: address, To: &contractAddr, Gas: 0, GasPrice: gasPrice,
		Value: big.NewInt(amount), Data: txData,
	}
	gasLimit, err := ethclient.EstimateGas(context.Background(), callMsg)
	if err != nil {
		return fmt.Errorf("SendEthCrossOnt, estimate gas limit error: %s", err.Error())
	}

	tx := types.NewTransaction(nonce, contractAddr, big.NewInt(amount), gasLimit, gasPrice, txData)
	bf := new(bytes.Buffer)
	rlp.Encode(bf, tx)

	rawtx := hexutil.Encode(bf.Bytes())
	unsignedTx, err := DeserializeTx(rawtx)
	if err != nil {
		return fmt.Errorf("SendEthCrossOnt, eth.DeserializeTx error: %s", err.Error())
	}
	signedtx, err := types.SignTx(unsignedTx, types.HomesteadSigner{}, priKey)
	if err != nil {
		return fmt.Errorf("SendEthCrossOnt, types.SignTx error: %s", err.Error())
	}

	err = ethclient.SendTransaction(context.Background(), signedtx)
	if err != nil {
		return fmt.Errorf("SendEthCrossOnt, send transaction error:%s", err.Error())
	}

	WaitTransactionConfirm(signedtx.Hash(), ethclient)
	fmt.Printf("txhash is %s\n", signedtx.Hash().String())
	return nil
}

func WaitTransactionConfirm(hash ethcommon.Hash, ethclient *ethclient.Client) {
	for {
		time.Sleep(time.Second * 1)
		_, ispending, err := ethclient.TransactionByHash(context.Background(), hash)
		if err != nil {
			continue
		}
		if ispending == true {
			continue
		} else {
			break
		}
	}
}

func DeserializeTx(rawTx string) (*types.Transaction, error) {
	txData := ethcommon.FromHex(rawTx)
	tx := &types.Transaction{}
	err := rlp.DecodeBytes(txData, tx)
	if err != nil {
		return nil, fmt.Errorf("deserializeTx: err: %s", err)
	}
	return tx, nil
}


```

After running the function, you would get the transaction hash which can checked on poly [explorer](http://explorer.poly.network/testnet/) for the cross-chain status  of your ETH.

Wait for some time and you will receive ETH as ETHX token on Ontology.

## Unlock ETH from LockProxy

There is a lock proxy on Ontology and you need to lock your ETHX to it. After locking and wait some time, you will find the lockproxy on EThereum just unlock your ETH back to you.

Use the follow function to send ETH back to Ethereum.

```go
import (
	"fmt"
	common2 "github.com/ethereum/go-ethereum/common"
	"github.com/ontio/ontology-go-sdk"
	"github.com/ontio/ontology/common"
	"time"
)

func SendEthoCrossEth(amount uint64) error {
	ont := ontology_go_sdk.NewOntologySdk()
	ont.NewRpcClient().SetAddress("http://polaris4.ont.io:20336")

	wallet, err := ont.OpenWallet("/path/to/your/ontology/wallet.dat")
	if err != nil {
		return fmt.Errorf("open wallet error: %v", err)
	}
	acc, err := wallet.GetDefaultAccount([]byte("pwd"))
	if err != nil {
		return fmt.Errorf("getDefaultAccount error: %v", err)
	}

	proxyContractAddress, err := common.AddressFromHexString("lockproxy on Ontology")
	if err != nil {
		return fmt.Errorf("SendEthoCrossEth, ontcommon.AddressFromHexString error: %s", err)
	}

	assetaddress, err := common.AddressFromHexString("ethx on ontology")
	if err != nil {
		return fmt.Errorf("SendEthoCrossEth, ontcommon.AddressFromHexString error: %s", err)
	}

	txHash, err := ont.NeoVM.InvokeNeoVMContract(500, 200000,
		acc,
		acc,
		assetaddress,
		[]interface{}{"approve", []interface{}{acc.Address, proxyContractAddress, amount}})
	if err != nil {
		return fmt.Errorf("SendEthoCrossEth, approve error: %v", err)
	}
	WaitOntTx(txHash, ont)

	to := common2.HexToAddress("0xyour_ethereum_address")
	txHash, err = ont.NeoVM.InvokeNeoVMContract(500, 200000,
		acc,
		acc,
		proxyContractAddress,
		[]interface{}{"lock", []interface{}{assetaddress, acc.Address[:], 2, to.Bytes(), amount}})
	if err != nil {
		return fmt.Errorf("SendEthoCrossEth, ctx.Ont.NeoVM.InvokeNeoVMContract error: %s", err)
	}
	WaitOntTx(txHash, ont)
	return nil
}

func WaitOntTx(txhash common.Uint256, ont *ontology_go_sdk.OntologySdk) {
	tick := time.NewTicker(100 * time.Millisecond)
	var h uint32
	startTime := time.Now()
	for range tick.C {
		h, _ = ont.GetBlockHeightByTxHash(txhash.ToHexString())
		curr, _ := ont.GetCurrentBlockHeight()
		if h > 0 && curr > h {
			break
		}

		if startTime.Add(100 * time.Millisecond); startTime.Second() > 300 {
			panic(fmt.Errorf("tx( %s ) is not confirm for a long time ( over %d sec )",
				txhash.ToHexString(), 300))
		}
	}
}
```

## More

Except Ontology, you can transfer ETH to any chain supporting Poly cross-chain protocol. 

Other assets like ERC20 can cross-chain transfer to other chain like ETH.