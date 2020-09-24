# neo-relayer

[English](./How_to_become_neo_relayer_en.md) | 中文

## 架构

一方面，neo-relayer实现了对Neo网络的监听，会持续扫描每个Neo区块并将切换了下一轮共识地址的关键区块头同步到中继链上，同时能识别并转发跨链交易，向中继链提交区块的状态根和交易的梅克尔证明。  
另一方面，neo-relayer也实现了对中继链的监听，会持续扫描每个中继链区块并将切换了共识地址的关键区块头同步到Neo链上，中继链收到其他链的跨链信息会构造到Neo链的返回交易，neo-relayer会将这些交易所在的中继链区块头同步到Neo链上因为其中包含了状态根，同时转发这些交易的梅克尔证明及跨链信息。  
只要存在一个neo-relayer在工作，那么整个Neo跨链生态就可以持续运作。

## 准备

1. 申请中继链钱包  
申请一个中继链的钱包，如果有本体的钱包可以直接使用，二者钱包是通用的。

2. 注册Relayer  
然后向中继链注册自己的地址为Relayer。

## 启动Relayer

更多关于如何配置、部署和启动一个neo-relayer，参考[neo-relayer]([./How_to_become_neo_relayer_en.md](https://github.com/polynetwork/neo-relayer/blob/master/README.md))。
