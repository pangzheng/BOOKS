# Polygon 验证机制
## Polygon 概述
Polygon 是公共区块链的扩展解决方案。[基于 Plasma 框架 (Plasma MoreVP)](https://ethresear.ch/t/account-based-plasma-morevp/5480) 的改进实现——基于账户的实现，Polygon 支持所有现有的以太坊工具以及更快、更便宜的交易。
### 验证者和委托者角色#
在 Polygon 网络上，您可以是验证者或委托者。看：

- [谁是验证者](https://docs.polygon.technology/docs/validate/polygon-basics/who-is-validator)
- [谁是委托人](https://docs.polygon.technology/docs/validate/polygon-basics/who-is-delegator)

### 架构#
如果您的目标是成为验证者，那么了解 Polygon 架构至关重要。

请参阅 [Polygon 架构](https://docs.polygon.technology/docs/validate/validator/architecture)。

### 组件#
要详细了解 Polygon 架构，请参阅核心组件：

- [Heimdall](https://docs.polygon.technology/docs/contribute/heimdall/overview)
- [Bor](https://docs.polygon.technology/docs/contribute/bor/overview)
- [Contracts](https://docs.polygon.technology/docs/contribute/contracts/stakingmanager)

### 代码库#
要详细了解核心组件，请参阅代码库：

- [Heimdall](https://github.com/maticnetwork/heimdall)
- [Bor](https://github.com/maticnetwork/bor)
- [Contracts](https://github.com/maticnetwork/contracts)

### 如何做#
#### 节点设置#
- [使用 Ansible 运行验证器节点](https://docs.polygon.technology/docs/validate/validate/run-validator-ansible)
- [从二进制文件运行验证器节点](https://docs.polygon.technology/docs/validate/validate/run-validator-binaries)

#### 质押操作#
- [验证人质押操作](https://docs.polygon.technology/docs/validate/docs/validate/validate/validator-staking-operations)
- [代表](https://docs.polygon.technology/docs/validate/delegate)

## Polygon 基础
### 什么是 Polygon
Polygon是第 2 层扩展解决方案，它通过利用侧链进行链下计算来实现扩展，同时使用 Plasma 框架和分散的权益证明 (PoS) 验证器网络确保资产安全。

[Polygon](https://polygon.technology/) 致力于解决可扩展性和可用性问题，同时不影响去中心化并利用现有的开发者社区和生态系统。Polygon 是一种适用于现有平台的离线/侧链扩展解决方案，可为 DApp 和用户功能提供可扩展性和卓越的用户体验。
#### 主要特点和亮点#
- 可扩展性：Polygon 侧链上的快速、低成本和安全的交易，最终在主链和以太坊上实现，作为第一个兼容的第 1 层基础链。
- 高吞吐量：在内部测试网的单个侧链上实现了高达 10,000 TPS；要添加多个链以进行水平缩放。
- 用户体验：流畅的用户体验和从主链到 polygon 链的开发者抽象；支持 WalletConnect 的原生移动应用和 SDK。
- 安全性：polygon 链运营商（矿工？）本身就是 PoS 系统中的质押者。
- 公共侧链：polygon 侧链本质上是公共的（相对于单个 DApp 链），无需许可且能够支持多种协议。
- Polygon 系统经过精心设计，可支持支持 EVM 的 Polygon 侧链上的任意状态转换。

### 谁是验证者
验证者是网络中的参与者，他将代币锁定在网络中并运行[验证](https://docs.polygon.technology/docs/validate/glossary#validator)者节点以运行和保护网络。

验证者具有以下职责：

- 质押网络代币并运行验证者节点以作为验证者加入系统。
- 通过验证区块链上的状态转换获得质押奖励。
- 因双重签名、验证者停机等问题受到处罚/削减。

区块链验证者是负责验证区块链内交易的人。对于 Polygon Network，任何参与者都可以通过运行一个完整的节点来获得奖励并收取交易费用，从而有资格成为 Polygon 的验证者。为了确保验证者的良好参与，他们将部分 MATIC 代币锁定为生态系统中的股份。

Polygon Network 中的验证者是通过定期发生的链上拍卖过程选择的。这些选定的验证者作为块生产者和验证者参与。一旦参与者验证了[检查点](https://docs.polygon.technology/docs/validate/glossary#checkpoint-transaction)，就会在父链（以太坊主网）上进行更新，根据他们在网络中的股份为验证者释放奖励。

那些对保护网络感兴趣但没有运行完整节点的人可以作为[委托人](https://docs.polygon.technology/docs/validate/glossary#delegator)参与。

也可以看看：

- [验证人职责](https://docs.polygon.technology/docs/validate/validate/validator-responsibilities)
- [证实](https://docs.polygon.technology/docs/validate/validate/getting-started)
- [验证器常见问题解答](https://docs.polygon.technology/docs/validate/validator-faq)

### 谁是委托人
委托人是不能或不想自己运行[验证](https://docs.polygon.technology/docs/validate/glossary#validator)节点的代币持有者。

委托人通过与验证人质押来保护网络。为了与验证者进行质押，委托人向他们选择的验证者支付[佣金](https://docs.polygon.technology/docs/validate/glossary#commission)。

委托人将质押代币委托给验证人，并获得部分奖励作为交换。因为委托人与他们的验证人分享奖励，委托人也分担风险。如果验证者行为不端，他们的每个委托人都将按其委托权益的比例被部分削减。委托人在系统中扮演着至关重要的角色，因为他们负责选择验证人。

也可以看看：

- [代表](https://docs.polygon.technology/docs/validate/delegate)
- [委托人常见问题](https://docs.polygon.technology/docs/validate/delegator-faq)

### 什么是权益证明(POS)
权益证明 (PoS) 是公共区块链的一类共识算法，它依赖于验证者在网络中的经济[权益](https://docs.polygon.technology/docs/validate/glossary#staking)。

- PoW

	在基于工作量证明 (PoW) 的公共区块链中，该算法奖励解决密码难题以验证交易并创建新区块的参与者。PoW 区块链示例：比特币、当前的以太坊。
- PoS

	在基于 PoS 的公共区块链中，一组验证者轮流提议和投票下一个区块。每个验证者投票的权重取决于其存款的大小—[权益](https://docs.polygon.technology/docs/validate/glossary#staking)。PoS 的显着优势包括安全性、降低集中化风险和能源效率。PoS 区块链示例：Eth2、Polygon。

通常，PoS 算法如下所示。区块链跟踪一组验证者，任何持有区块链基础加密货币（在以太坊的情况下为以太币）的人都可以通过发送一种特殊类型的交易来成为验证者，该交易将他们的以太币锁定在存款中。然后通过所有当前验证者都可以参与的共识算法来完成创建和同意新块的过程。

共识算法有很多种，给参与共识算法的验证者分配奖励的方式也很多，所以权益证明有很多“味道”。从算法的角度来看，主要有两种类型：

- 基于链式的 PoS 

	在 `基于链的权益证明` 中，算法在每个时间段（例如，每 10 秒可能是一个时间段）伪随机选择一个验证者，并分配该验证者创建单个块的权利，并且该块必须指向某个先前的块（通​​常是先前最长链末端的块），因此随着时间的推移，大多数块会汇聚成一条不断增长的链。
- [BFT](https://en.wikipedia.org/wiki/Byzantine_fault_tolerance) 式的 PoS。

	在 `BFT 风格的权益证明` 中，验证者被`随机分配` 提出区块的权利 ，但是通过多轮过程来确定哪个区块是规范的，每个验证者在每一轮中为某个特定区块发送“投票”，并且在该过程结束时，所有（诚实的和在线的）验证者永久同意任何给定的块是否是链的一部分。请注意，块可能仍然链接在一起；关键区别在于，对一个区块的共识可以在一个区块内进行，并且不依赖于它之后的链的长度或大小。

有关更多详细信息，请参阅 [https://github.com/ethereum/wiki/wiki/Proof-of-Stake-FAQ](https://github.com/ethereum/wiki/wiki/Proof-of-Stake-FAQ)。

也可以看看：

- [委托人](https://docs.polygon.technology/docs/validate/glossary#delegator)
- [验证器](https://docs.polygon.technology/docs/validate/glossary#validator)

### 经济学
有关多边形和权益证明算法的介绍，请参阅：

- [什么是多边形](https://docs.polygon.technology/docs/validate/polygon-basics/what-is-polygon)
- [什么是权益证明](https://docs.polygon.technology/docs/validate/polygon-basics/what-is-proof-of-stake)

在 Polygon 中，验证者将他们的 MATIC 代币作为抵押品，以维护网络的安全，并以他们的服务换取奖励。

#### 什么是激励？#
Polygon 将其 100 亿代币总供应量的 12% 分配给 Staking 奖励。这是为了确保网络播种足够好，直到交易费用获得牵引力。这些奖励主要是为了启动网络。而从长远来看，该协议旨在基于交易费用维持自身。

	验证人奖励 = 质押奖励 + 交易费用
这是以确保权益奖励逐渐脱离验证者奖励的主要组成部分的方式分配的。

年|目标股份(循环供应量的30%)|30%绑定率|奖池
---|---|---|---
第一年|1,977,909,431|20%|312,917,369
第二年|2,556,580,023|12%|275,625,675
第三年|2,890,642,855|9%|246,933,140
第四年|2,951,934,048|7%|204,303,976
第五年|2,996,518,749|5%|148,615,670 + 11,604,170

以下是前 5 年预期年度奖励的示例快照，考虑到 5% 到 40% 的质押供应，间隔为 5%

质押流通供给量百分比|5%|10%|15%|20%|25%|30%|35%|40%
---|---|---|---|---|---|---|---|---
年度奖励|
第一年|120%|60%|40%|30%|24%|20%|17.14%|15%
第二年|72%|36%|24%|18%|14.4%|12%|10.29%|9%
第三年|54%|27%|18%|13.5%|10.8%|9%|7.71%|6.75%
第四年|42%|21%|14%|10.5%|8.4%|7%|6%|5.25%
第五年|30%|15%|10%|7.5%|6%|5%|4.29%|3.75%

#### 谁得到激励？#
运行验证者节点的质押者和将他们的代币委托给他们喜欢的[验证者的质押者](https://docs.polygon.technology/docs/validate/glossary#validator)。

[验证人可以选择对委托人](https://docs.polygon.technology/docs/validate/glossary#delegator)获得的奖励收取佣金。

属于所有质押者的资金被锁定在部署在以太坊主网上的合约中。

没有验证人保管委托人代币。
#### 职责#
设置验证者节点是一项责任重大的工作。

请参阅[验证者职责](https://docs.polygon.technology/docs/validate/validate/validator-responsibilities)。
#### 质押奖励#
年度激励是绝对的— —无论网络中的总股份或目标绑定率如何，激励金额都会定期作为奖励发放给所有签名者。

在 Polygon 中，还有一个额外的元素是向以太坊主网提交定期[检查点](https://docs.polygon.technology/docs/validate/glossary#checkpoint-transaction)。这是验证者职责的主要部分，他们被激励执行此活动。这构成了验证器的成本，这是第 2 层解决方案（例如 Polygon）所独有的。我们努力在验证者质押奖励支付机制中容纳这一成本，作为支付给负责提交检查点的提议者的奖金。奖励减去奖金将在所有质押者之间共享；提议者和签名者，按比例。

鼓励提议者包括所有签名#
要完全利用奖金，提议者必须在检查点中包含所有签名。因为协议要求总权益的 ⅔ +1 权重，所以即使有 80% 的选票，检查点也会被接受。但是，在这种情况下，提议者仅获得计算奖金的 80%。

交易费用#
Bor的每个区块生产者都会获得每个区块中收取的交易费用的一定比例。任何给定跨度的生产者选择也取决于验证者在总权益中的比例。剩余的交易费用与奖励通过相同的渠道流动，奖励在Heimdall层工作的所有验证者之间共享。
成为验证者#
要成为验证者，您需要运行完整的验证者节点并质押 MATIC。

请参阅验证。

成为委托人#
要成为委托人，您只需将 MATIC 委托给验证人。

请参阅委托。





