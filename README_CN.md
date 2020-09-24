<h1 align="center">Poly Network</h1>
<h4 align="center">Version 1.0 </h4>

[English](README.md) | 中文

欢迎来到Poly Network跨链技术中心！ 

区块技术经历10多年的发展，虽然已经涌现了众多的公链系统，但现有的区块链架构体系大多围绕单条区块链的可扩展、性能等方面的研究展开，而单条区块链由于架构体系和发展方向的制约，
很难满足所有的需求，因此，我们预计未来区块链生态一定是多条区块链并存的格局，不同的链可以包含不同的特性成为区块链基础设施中的一份子，但链与链之间依然自成体系，各自形成了自己的价值孤岛，
不同的链之间缺乏快捷的互操作性和方便的价值流通手段。为更好的打造下一代互联网基本设施，我们推出了全新的跨链技术Poly Network。

跨链技术是在现有的单条区块链架构设计的基础上通过跨链互操作协议打通链与链之间交互的一种新的技术手段。跨链互操作协议最早在 2016 年 9 月由 Vitalik Buterin 首次提出，他将跨链互操作协议分为三种模式: 公证人机制 (Notary schemes)、侧链/中继 (Sidechains/relays)、哈希时间锁 (Hash locking)。公证人机制是指在跨链交互的过程中通过一个信任的第三方来交互，主要通过” 单签、多签公证人机制” 等机制实现；侧链/中 继模式是现在使用比较多的协议，例如:Cosmos 和 Polkadot等跨链明星项目都是使用该协议，它是指不需要依赖可信的第三方进行跨链交易的验证，可以自行进行跨链交易验证的一种技术手段；哈希时间锁是最早闪电网络的底层技术，它将该协议分为时间锁和哈希锁，时间锁指交易双方约定在某个时间内提交才有效，超时则承诺方案失效（无论是提出方或接受方）。哈希锁指对一个哈希值H，如果提供原像R使得 Hash(R) = H，则承诺有效，否则失效。如果交易双方因为各种原因未能成功，时间锁能够让交易参与各方拿回自己资金，避免因欺诈或交易失败造成的损失。

Poly Network基于侧链/中继模式，采用双层结构设计，使用Poly中继链作为跨链协调器，多条异构链作为跨链事务执行器，Relayer作为跨链信息的搬运工，通过解决跨链信息的有效信、安全性和事务性等问题，实现了一套安全、易用、高效的跨链体系。

## 特性
* 轻量级跨链协议
* 接入简单方便
* 同时支持同构链和异构链
* 支持跨链事务强一致性和最终一致性
* 资产跨链超级，同时支持资产跨链和任意信息跨链
* 跨链协议安全可靠，以密码学为基石
* 支持异构链协议范围广（BTC/ETH/NEO/Ontology/Cosmos）
* 中继链是开放式准入的公链

## 教程文档
* [中继链介绍](poly/README_CN.md)
* [Bitcoin链介绍](btc/README_CN.md)
* [Ethereum链介绍](eth/README_CN.md)
* [NEO链介绍](neo/README_CN.md)
* [Ontology链介绍](ont/README_CN.md)
* [Cosmos链介绍](cosmos/README_CN.md)
* [链配置文件信息](config/README.md)

## 贡献代码

请您以签过名的commit发送pull request请求，我们期待您的加入！
您也可以通过邮件的方式发送你的代码到开发者邮件列表，欢迎加入Poly Network邮件列表和开发者论坛。

另外，在您想为Poly贡献代码时请提供详细的提交信息，格式参考如下：

  Header line: explain the commit in one line (use the imperative)

  Body of commit message is a few lines of text, explaining things
  in more detail, possibly giving some background about the issue
  being fixed, etc etc.

  The body of the commit message can be several paragraphs, and
  please do proper word-wrap and keep columns shorter than about
  74 characters or so. That way "git log" will show things
  nicely even when it's indented.

  Make sure you explain your solution and why you're doing what you're
  doing, as opposed to describing what you're doing. Reviewers and your
  future self can read the patch, but might not understand why a
  particular solution was implemented.

  Reported-by: whoever-reported-it
  Signed-off-by: Your Name <youremail@yourhost.com>

## 开源社区

## License
The Poly Network source code is available under the LGPL-3.0 license.

