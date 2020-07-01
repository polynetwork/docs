<h1 align="center">Ethereum Relayer Manual</h1>
<h4 align="center">Version 1.0 </h4>

[English](ethereum_relayer_manual.md) | 中文

以太跨链到其他链或者其他链跨链到以太都是通过中继链进行，以太和中继链之间交互需要relayer来完成区块头同步和跨链交易提交，包括从以太到中继链的区块头同步和跨链交易提交，从中继链到以太的区块头同步和跨链交易的提交。

## 准备

一个以太账户，有一定的以太币，提交交易需要GAS，同时需要在中继链注册该地址才能向中继链同步区块头和提交跨链交易。

## 下载

从指定位置下载正确的ethereum relayer package。

## 配置

以下是ethereum relayer的全部配置
```
{
  "MultiChainConfig":{
    "RestURL":"http://172.168.3.73:40336",
    "EntranceContractAddress":"0300000000000000000000000000000000000000",
    "WalletFile":"./wallet.dat",
    "WalletPwd":"123456",
    "GasPrice":0,
    "GasLimit":2000000
  },
  "ETHConfig":{
    "RestURL":"https://ropsten.infura.io/v3/1ba5f3635395470e9a3f19ba7a852144",
    "LockerContractAddress":"0x3C3e443F4076aff3ded4C6a65217F9796BF3e6a7",
    "PrivateKey":"30E292FD40E645AB65FD3376689E6E5AC8C74E275867C4E7A04B88FA5CB4D252",
    "Signer":"0xB7Ee265D94446F465dba65002A9960D4bef9dca7",
    "AccountWalletPath":"./ethwallet.dat",
    "CapitalOwnersPath": "./capital-owners",
    "CapitalPassword": "123456"
  }
}
```
配置包括两部分，中继链相关配置和以太相关配置。

中继链配置项有：
```
RestURL                                  -           中继链的节点URL
EntranceContractAddress                  -           中继链的跨链管理合约
WalletFile&WalletPwd                     -           中继链的钱包以及账户
GasPrice&GasLimit                        -           中继链的gas配置
```

以太配置项有：
```
RestURL                                  -           以太节点的URL
LockerContractAddress                    -           以太跨链管理合约
PrivateKey&Signer                        -           以上准备好的以太账户
```

## 运行

启动参数主要包括两项：
```
--ethereum             -    指定从以太的哪个高度开始，一般指定为以太当前的最新高度
--Poly             -    指定从中继链的哪个高度开始，一般指定为0，如果不是0，指定的区块头需要包括验证节点信息
```

启动：
```
./eth_relayer --ethereum 7318671 --Poly 0
```

也可以从后台启动或者使用启动脚本启动： (但需要修改启动脚本中启动参数)
```
./eth_relayer_start.sh
```

## 许可证

Poly Network遵守GNU Lesser General Public License, 版本3.0。 详细信息请查看项目根目录下的LICENSE文件。
