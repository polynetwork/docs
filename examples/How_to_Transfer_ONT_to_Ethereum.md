



<h1 align=center>How to Transfer ONT to Ethereum</h1>

## Smart Contracts on Both Side

When you need to cross-chain ONT to Ethereum, you need prepare smart contracts on both side. We suggest you are using **Poly testnet**.

- **On Ontology side**: The contract `LockProxy` on Ontology is necessary. Check demo [here](https://github.com/polynetwork/ont-contracts/blob/master/contracts/core/lockproxy/lock_proxy.py)  and read the [document](../ont/How_to_new_cross_chain_asset.md) to understand the contract. Before deploying, you need to change the `Operator` to yourself in contracts. 
- **On Ethereum side**: First deploy your lockproxy on Ethereum and here is [demo](https://github.com/polynetwork/eth-contracts/blob/master/contracts/core/lock_proxy/LockProxyPip1.sol). And there must be a [contract](https://github.com/polynetwork/eth-contracts/tree/master/contracts/core/assets/ont) on Ethereum to receive ONT. You can deploy contracts according to these documents [eth](../eth/how_to_deploy_crosschain_on_ethereum.md).  And call `bindAssetHash` of lockproxy on Ethereum to bind ONT with ONTX. 
- **Setup contracts**: 
  - Call `bindProxyHash` of lockproxy on Ontology with parameters `{2, raw_hash_bytes_of_eth_lockproxy}`. 
  - Call `bindProxyHash` of lockproxy on Ethereum with parameters `{3, raw_hash_bytes_of_ont_lockproxy}`.
  - Call `bindAssetHash` of lockproxy on Ethereum with parameters `{ONTAssetHash, 2, ethereum_ontx_raw_hash}`.

Of course you can using our lock proxy `0xE44dDbAb7a9Aa11b9D0cA8b686F9E0299DB735d1` on Ethereum and `34a593c5ccfb8590e9a4bc7018529016aa4a55ad` on Ontology. ONTX on Ethereum is `0x8cE92808118Bc43c2B8635d3eb6b1b67cD2fB9a0`. No need to setup.

Of course you need to prepare ontology account and ethereum account yourself.

## Lock ONT to LockProxy

You need to lock some ONT to contract LockProxy. And then the Poly cross-chain Ecosystem will do the rest work. 

First you need to learn sending a ontology transaction by using [Ontology-SDK](https://github.com/ontio/ontology-go-sdk).  The follow code shows you how to lock your ONT to `LockProxy` on Ontology testnet. Of course you need to prepare a ontology `wallet.dat` with ONT.

```go
import (
	"fmt"
	ethcommon "github.com/ethereum/go-ethereum/common"
	"github.com/ontio/ontology-go-sdk"
	ontcommon "github.com/ontio/ontology/common"
	nutils "github.com/ontio/ontology/smartcontract/service/native/utils"
)

func SendOntCrossEth(amount uint64) error {
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

	proxyContractAddress, err := ontcommon.AddressFromHexString("34a593c5ccfb8590e9a4bc7018529016aa4a55ad") // lockproxy on Ontology
	if err != nil {
		return fmt.Errorf("SendOntCrossEth, ontcommon.AddressFromHexString error: %s", err)
	}
	to := ethcommon.HexToAddress("0xyour_ethereum_address")
	txHash, err := ont.NeoVM.InvokeNeoVMContract(500, 2000000,
		acc,
		acc,
		proxyContractAddress,
		[]interface{}{"lock", []interface{}{nutils.OntContractAddress[:], acc.Address[:], 2, to, amount}})
	if err != nil {
		return fmt.Errorf("SendOntCrossEth, ctx.Ont.NeoVM.InvokeNeoVMContract error: %s", err)
	}
	fmt.Printf("your txhash is %s\n", txHash.ToHexString())

	return nil
}
```

After running the function, you would get the transaction hash which can checked on [explorer](https://explorer.ont.io/testnet). And Poly prepare a [explorer](http://explorer.poly.network/testnet/) for checking the cross-chain status  of your ONT.

Wait for some time and you will receive ONT as ONTX token on Ethereum.

## Unlock ONT from LockProxy

There is a lock proxy on Ethereum and you need to lock your ONTX to it. After locking and wait some time, you will find the lockproxy on Ontology just unlock your ONT back to you.

Before write code, you need to get the [ABI](https://github.com/polynetwork/eth-contracts/tree/master/go_abi) for contracts on Ethereum. Use the follow function to send ONT back to Ontology.

```go
import (
	"bytes"
	"context"
	"crypto/ecdsa"
	"fmt"
	"github.com/ethereum/go-ethereum"
	"github.com/ethereum/go-ethereum/accounts/abi"
	"github.com/ethereum/go-ethereum/accounts/abi/bind"
	ethcommon "github.com/ethereum/go-ethereum/common"
	"github.com/ethereum/go-ethereum/common/hexutil"
	"github.com/ethereum/go-ethereum/core/types"
	"github.com/ethereum/go-ethereum/crypto"
	"github.com/ethereum/go-ethereum/ethclient"
	"github.com/ethereum/go-ethereum/rlp"
	"github.com/ontio/ontology/common"
	"github.com/polynetwork/eth-contracts/go_abi/lock_proxy_abi"
	"github.com/polynetwork/eth-contracts/go_abi/ontx_abi"
	"math/big"
	"strings"
	"time"
)

func SendEOntCrossOnt(amount int64) error {
	ethclient, err := ethclient.Dial("ethereum_rpc_address")
	if err != nil {
		return err
	}
	contractabi, err := abi.JSON(strings.NewReader(lock_proxy_abi.LockProxyABI)) // import ABI code for this
	if err != nil {
		return fmt.Errorf("SendEOntCrossOnt, abi.JSON error: %v", err)
	}

	assetaddress := ethcommon.HexToAddress("0x8cE92808118Bc43c2B8635d3eb6b1b67cD2fB9a0") // ONTX
	contractAddr := ethcommon.HexToAddress("0xE44dDbAb7a9Aa11b9D0cA8b686F9E0299DB735d1") // lockproxy on Ethereum

	ontxContract, err := ontx_abi.NewONTX(assetaddress, ethclient) // import ABI code for this
	if err != nil {
		return fmt.Errorf("SendEOntCrossOnt, NewONTX error: %v", err)
	}

	// prepare eth account
	priKey, err := crypto.HexToECDSA("eth_private_key")
	if err != nil {
		return fmt.Errorf("SendEOntCrossOnt, cannot decode private key")
	}
	address := crypto.PubkeyToAddress(priKey.PublicKey)
	nonce, err := ethclient.PendingNonceAt(context.Background(), address)
	auth := MakeEthAuth(priKey, nonce, 5e9, 8000000)
	if err != nil {
		return fmt.Errorf("SendEOntCrossOnt, failed to get eth auth: %v", err)
	}
	txhash, err := ontxContract.Approve(auth, contractAddr, big.NewInt(amount)) // you need approving ONTX to lockproxy first
	if err != nil {
		return fmt.Errorf("SendEOntCrossOnt, failed to approve: %v", err)
	}
	WaitTransactionConfirm(txhash.Hash(), ethclient)

	ontAddr, err := common.AddressFromBase58("your_ont_address")
	if err != nil {
		return err
	}
	txData, err := contractabi.Pack("lock", assetaddress, 3, ontAddr[:], big.NewInt(int64(amount)))
	if err != nil {
		return fmt.Errorf("SendEOntCrossOnt, contractabi.Pack error: %v", err)
	}

	callMsg := ethereum.CallMsg{
		From: address, To: &contractAddr, Gas: 0, GasPrice: big.NewInt(5e9),
		Value: big.NewInt(0), Data: txData,
	}
	gasLimit, err := ethclient.EstimateGas(context.Background(), callMsg)
	if err != nil {
		return fmt.Errorf("SendEOntCrossOnt, estimate gas limit error: %s", err.Error())
	}

	nonce++
	tx := types.NewTransaction(nonce, contractAddr, big.NewInt(amount), gasLimit, big.NewInt(5e9), txData)
	bf := new(bytes.Buffer)
	rlp.Encode(bf, tx)

	rawtx := hexutil.Encode(bf.Bytes())
	unsignedTx, err := DeserializeTx(rawtx)
	if err != nil {
		return fmt.Errorf("SendEOntCrossOnt, eth.DeserializeTx error: %s", err.Error())
	}
	signedtx, err := types.SignTx(unsignedTx, types.HomesteadSigner{}, priKey)
	if err != nil {
		return fmt.Errorf("SendEOntCrossOnt, types.SignTx error: %s", err.Error())
	}

	err = ethclient.SendTransaction(context.Background(), signedtx)
	if err != nil {
		return fmt.Errorf("SendEOntCrossOnt, send transaction error:%s", err.Error())
	}

	WaitTransactionConfirm(signedtx.Hash(), ethclient)
	return nil
}

func MakeEthAuth(signer *ecdsa.PrivateKey, nonce, gasPrice, gasLimit uint64) *bind.TransactOpts {
	auth := bind.NewKeyedTransactor(signer)
	auth.Nonce = big.NewInt(int64(nonce))
	auth.Value = big.NewInt(int64(0)) // in wei
	auth.GasLimit = gasLimit          // in units
	auth.GasPrice = big.NewInt(int64(gasPrice))

	return auth
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

## More

Except Ethereum, you can transfer ONT to any chain supporting Poly cross-chain protocol. 

Other assets like ONG, OEP4. etc can cross-chain transfer to other chain like ONT.