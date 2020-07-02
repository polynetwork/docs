<h1 align="center">COSMOS Relayer</h1>
<h4 align="center">Version 1.0 </h4>

[English]() | 中文

## 引言

在将COSMOS上的Coin转到其他链的时候，需要在中继链和COSMOS中间启动一个Relayer。Relayer会将COSMOS上的跨链请求及其证明搬运到中继链，中继链可以验证这个跨链请求确实在COSMOS上执行成功了，然后再生成证明，表示这个跨链请求是验证通过的，其他链的Relayer会将这个来自COSMOS的跨链请求搬运到下一条链，最后在目标链为指定账户增发资产。

Relayer搬运的信息是请求内容、证明和区块头。请求内容包含了目标链ID、账户地址、跨链金额，目标链ID是目标区块链网络在跨链生态中唯一的标识，比如比特币是1，账户地址是在目标链接收资产的地址，比如跨链转1BTC，那么在目标链账户地址上将有1BTCX（BTCX是BTC在目标链的映射，可视为一种凭证，它与BTC是一一对应的）；”证明“可以证明跨链请求交易在COSMOS链上且已经成功执行，中继链需要这个证明来确认跨链请求的合法性；”区块头“是COSMOS的区块头，是取证明高度的下一个区块头，用来验证”证明“的正确性，即计算”证明“的根，验证区块头中的AppStateRoot与根是否相等，类似于MerkleProof的证明过程，若相等，则跨链请求合法。

同样地，Relayer也会把到COSMOS的跨链请求搬运到COSMOS，crosschain模块可以验证并处理跨链请求。

## 架构

Relayer监听COSMOS每个高度的事件，在高度h发现跨链交易Tx1、Tx2，从大于h的高度H获取交易的Proof，这个Proof由crosschain模块产生，以及H+1高度的区块头，首先向中继链发送包含COSMOS区块头的交易，等到该交易在中继链落账之后，Relayer并发发送交易及其Proof，中继链就可以通过区块头验证Proof。

<div align=center><img width="500" height="440" src="./pic/cosmos2poly.png"/></div>

类似地，Relayer会把中继链上的跨链请求转发到COSMOS上。Relayer监听中继链的事件，如果发现事件中包含跨链到COSMOS的事件，则获取事件中的Proof和下一个高度的区块头，组装成COSMOS的交易，由Relayer发到COSMOS，Relayer需要为此支付手续费，COSMOS的crosschain模块会验证Proof并增发对应的代币，实现其他虚拟货币跨链到COSMOS链。

<div align=center><img width="500" height="440" src="./pic/poly2cosmos.png"/></div>

## 使用

下载代码编译，配置文件如下：

```json
{
  "cosmos_rpc_addr": "http://ip:port", //COSMOS的RPC地址
  "cosmos_wallet": "/path/to/cosmos_key", // cosmos的钱包文件，使用gaiacli导出
  "cosmos_wallet_pwd": "pwd", // 对应的密码
  "cosmos_start_height": 0, // Relayer扫描开始的COSMOS高度
  "cosmos_listen_interval": 1, // 扫描间隔（秒）
  "cosmos_chain_id": "testing", // COSMOS启动的网络ID
  "cosmos_gas_price": "0.00001stake", // 发送COSMOS交易的gas price
  "cosmos_gas": 200000, // 一笔交易的gas上限

  "poly_rpc_addr": "http://172.168.3.73:40336", // 中继链的RPC地址
  "poly_wallet": "wallet.dat", // 中继链钱包
  "poly_wallet_pwd": "pwd", // 中继链钱包密码
  "poly_start_height": 0, // 扫描的起始高度
  "poly_listen_interval": 1, // 扫描间隔时间（秒）
  "poly_to_cosmos_key": "makeProof", // 跨链事件关键字

  "db_path": "cosmos-relayer/db", // DB
  "log_level": 2 // 0: TRACE, 1: DEBUG:, 2: INFO, 3: WARN, 4: ERROR
}
```

准备好自己的配置文件，就可以运行Relayer了。如果没有配置钱包密码，启动的时候需要输入对应钱包的密码。

```go
go build -o run_cosmos_relayer cmd/run.go
./run_cosmos_relayer -conf=conf.json
```