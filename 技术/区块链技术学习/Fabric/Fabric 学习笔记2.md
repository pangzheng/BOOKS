# Fabric 学习笔记2
## 1 系统架构
区块链是一个分布式系统，由许多相互通信的节点组成。区块链运行的程序称为链码(智能合约)，保存状态和账本数据、执行交易。链码是核心要素，交易操作在链码上调用。交易必须被“背书”，只有经过背书的交易才可以提交，并对状态产生影响。有可能存在一个或多个特定的链码用于管理功能和参数，统称为系统链码。
### 1.1. 交易(Transactions)
交易可以有两种类型

- 部署交易

	创建新的链码并设置一个程序作为参数。当一个部署执行成功后就是表明链码已被安装到区块链上。
- 调用交易

	之前已部署链码的情况下执行一个操作。调用交易引用链码提供的一个函数。当成功时，链码执行特定的函数-它可能涉及修改相应的状态，并返回一个输出。

如后所述，部署交易是调用交易的特例，部署交易创建新的链码，对应于系统链码的一个调用交易。
### 1.2. 区块链数据结构(Blockchain datastructures)
#### 1.2.1. State/状态
区块链的最新状态（简称为，状态）被建模为一个版本键/值存储（KV）。整体上由运行在区块链上的链码（智能合约）操控，通过 `put` 和 `get` KV 操作实现。状态持续存储并且状态的更新也被记录。

注意：KV 被采用为状态模型，可以使用任意的 KV 库方案，像 RDBMS。

更正式地，状态 `s` 建模为一个元素映射 `K -> (V X N)`，其中

- `K` 是一组键
- `V` 是一组值
- `N` 是一个无限有序的版本号集

	内射函数 `next: N -> N` 获取 `N` 的一个元素并返回下一个版本号

`V` 和 `N` 都包含一个特定的元素 `\bot`，这是 `N` 的最底层元素的特例。最开始时所有的键都映射到` (\bot,\bot)`。对于 `s(k)=(v,ver)`，用 `s(k).value` 代表 `v`， 用 `s(k).version` 代表 `ver`。

KV操作模型

- `put(k,v)`,对于 `k\in K` 和 `v\in V`。
- 处理区块链状态 `s`，将它变为`s'`，这样`s'(k)=(v,next(s(k).version))`，以及 `s'(k')=s(k')` 以保证所有的 `k'!=k`。
- `get(k)` 返回 `s(k)`。

状态由 peer 节点保持，而不是order节点和客户端。

- 状态划分

	KV 中的键能够通过它们的名字识别属于哪个特定的链码。从这点上说，只有某个链码的交易可以修改属于这个链码的键。原则上，任何链码能够读取属于其它链码的键。支持跨链交易，修改属于两个或更多链码的状态是 V1 后期的特征。

#### 1.2.2 账本（Ledger）
账本提供了在系统运行过程中发生的可验证的历史，它包含所有成功的状态更改（称为有效交易）和不成功的状态更改尝试（称为无效交易）。

账本是由order 服务构建的一个全部有序的交易哈希链块（有效的或无效的）。哈希链强制将全部排序块置入账本，每个块包含一批全部排序交易。这个强制全部排序覆盖所有交易。

账本保存在所有 peer 节点，可选，保存在排序者的一个子集。在谈论排序时的账本是 `OrdererLedger`，而谈论 peer 节点时的账本是 `PeerLedger`。`PeerLedger` 与 `OrdererLedger` 的区别是，peer 节点本地维护一个位掩码来表明隔离有效交易和无效交易。

Peer 节点可以按照 Section XX 修剪 `PeerLedger`内容 。Orderers 节点维护 `OrdererLedger`用于实现容错性和可用性(`PeerLedger`)，可以提前在 Order 服务的保留属性后，随时修改 `PeerLedger`

账本允许 peer 节点重演所有交易的历史和重建状态。

### 1.3. 节点(Nodes)
Node 节点是区块链的通信实体。一个 “node节点” 仅仅是一个逻辑函数，多个不同类型的 node 节点可以运行在同一台物理服务器上。关键在于 node 节点如何在 “信任域” 中分组和关联控制它们的逻辑实体。

有三种类型的node节点：

1. `客户端(Client)`或叫`提交客户端(submitting-client)`

	客户端提交实际交易调用到背书者，广播交易请求到 Order 节点。

2. `Peer`

	提交交易、维持状态和账本的拷贝。此外，peer 节点可以有一个特殊的背书角色。

3. `order 服务节点(Ordering-service-node)`或`排序者(Orderer)`

	运行通信服务实现交付保证，像原子或全域广播。

#### 1.3.1. 客户端(Client)
客户端代表最终用户实体。它必须连接到一个 peer 节点以便与区块链交互。客户端可以选择连接任何 peer 节点。客户端创建并调用交易。

如第2节说明，客户端同时与 peer 节点和order 服务通信。
#### 1.3.2. Peer
Peer 节点以块的形式从 order 服务接收有序状态更新，维护状态和账本。

Peer 节点还可以担任 `背书节点(endorsing peer)` 或 `背书者(endorser)`的特殊角色。背书节点的特殊功能是“在提交事物之前认可交易”这个特殊链码才发生作用。每个链码可以指定一个“引用一组背书节点”的背书策略。该策略为一个有效交易背书（典型的是一组背书者签名）定义了必要和充分的条件。在部署交易(安装链码)的特殊情况下，背书策略是由系统链码的背书策略指定。
#### 1.3.3. Orderer
排序者产生 order 服务，即一个提供交付保证的通信架构。order 服务能以不同的方式实现：

- 集中服务排序（例如，开发和测试）
- 指向不同的网络和节点故障模型的分布式协议

order 服务为客户端和 peer 节点提供共享的通信信道，为包含交易的消息提供广播服务。客户端连接到信道，并可以在这个信道上广播消息，随后传递消息给所有 peer 节点。信道支持所有消息的原子传递，意思是，总排序交付的消息通信和可靠性。信道将相同消息输出给所有连接的 peer 节点并且输出的消息具有同样的逻辑顺序。这个原子通信保证也称为总排序广播，原子广播，或是分布式系统中的共识。通信消息是包含在区块链状态中的申请交易。

- `分隔（order 服务信道）Partitioning (ordering service channels)`

	order 服务可以支持多个信道，类发布/订阅主题消息系统 (pub/sub)。客户端能够连接到一个给定的信道，然后能够发送消息和获得到达的消息。信道能够被认为是分区的(客户端连接到一个信道而无需知道其它信道的存在)，客户端也可以连接到多个信道。尽管一些 order 服务实现包括 Fabric v1 将支持多信道，为了阐述简单剩余部分，假定 order 服务包含一个单独的信道/主题(channel/topic)。
- `order 服务 API(Ordering service API)`

	peer 节点通过 order 服务提供的接口连接到 order 服务提供的信道。order 服务 API 包含两个基本操作（更多是异步事件）：

	正在做：增加在客户端/peer节点指定序列号下取得特定块的 API。

	- broadcast(blob)
	
		客户端调用此函数来广播任意消息 blob 在信道散播。当发送一个请求到服务器时,在 BFT 环境下也称为 request(blob)。
	- deliver(seqno, prevhash, blob)
	
		order 服务在 peer 节点传送带有非负整型序列号`(seqno)`的 blob 和最近的传输消息的 hash 值 `(prevhash)`。换言之，它是从 order 服务产生的输出事件。`deliver()` 有时在发布/订阅系统也称为 `notify()` 或在 BFT 系统中称为 `commit()`。
- `账本(Ledger)和块的构成(block formation)`

	账本包含了 order 服务输出的所有数据。它是一系列根据之前描述的 `prevhash`和 `deliver(seqno, prevhash, blob)` 事件计算形成的一个哈希链。	
	出于效率的原因，大多数情况下 order 服务会在出块时在一个单个交付事件中批量输出 blobs，代替输出单个交易（blobs）。在这种情况下，order 服务必须在每个块内实施和传递 blobs 的确定顺序。块内 blobs 的数量可以由 order 服务动态选择。
	
	接下来，定义 order 服务的属性和解释交易背书的工作流程
	
	- 假定每个 deliver 事件一个 blob
	- 假定一个块的 deliver 事件对应单个 blob 一系列的单独传递事件
	- 根据上述描述确定块内的 blobs 顺序

- `order 服务特性(Ordering service properties)`

	order 服务的保证（或原子广播信道）规定了广播消息的发生和交付消息之间存在什么关系。这些保证如下：

	1. 安全性（一致性保证）：只要peer节点连接到信道足够长的时间（它们能够断开或奔溃，但会重启和重新连接），它们会看到交付（seqno,prevhash,blob）消息的同等序列。这意味着向所有peer节点输出（deliver()events）相同排序，以及根据序列号和为相同序列号携带同等内容（blob和prevhash）。注意这仅是一个逻辑顺序，在一个peer节点上的deliver(seqno,prevhash,blob)是不需要与另外一个peer节点输出的deliver(seqno,prevhash,blob)发生任何实时关系。换句话说，给定一个特定的seqno，没有两个正确的peer节点交付不同的prehash或blob值。而且，没有值blob交付除非一些客户端（peer节点）实际是广播（blob），以及更好的，每个广播blob只交付一次.
	
		此外，deliver()事件包含之前deliver()事件（prevhash）的数据加密哈希。当order 服务实现原子广播保证，prevhash是从序列号为seqno-1的deliver()事件得到的参数的加密哈希，这在第4节和第5节讨论。对于第一个deliver()事件特例，prevhash有一个缺省值。
	2. 活跃度（交付保证）：order 服务的活跃度保证由order 服务实现确定。准确的保证可以依赖于网络和节点故障模型。

原则上，如果提交客户端没有失败，order 服务应该保证每个连接到order 服务的正确peer节点终究交付每个提交交易。

概括地说，order 服务确保以下特性

1. 一致. 对于任何两个具有相同seqno的正确peer节点的事件deliver(seqno, prevhash0, blob0)和deliver(seqno, prevhash1, blob1) , 则prevhash0==prevhash1，以及 blob0==blob1;
2. 哈希链完整性。对于任何在正确peer节点的两个事件deliver(seqno-1, prevhash0, blob0)和deliver(seqno, prevhash, blob), prevhash = HASH(seqno-1||prevhash0||blob0).
3. 没有跳过. 如果order 服务在正确peer节点p输出deliver(seqno, prevhash, blob) , 这样的话seqno>0, 然后p已经交付事件deliver(seqno-1, prevhash0, blob0).
4. 没有创造. 任何在正确peer节点上的事件deliver(seqno, prevhash, blob)必须之前一定有一个broadcast(blob)事件在一些(可能是不同的)peer节点上;
5. 没有重复 (可选,但可取). 对于任何两个事件broadcast(blob)和broadcast(blob’), 当两个事件deliver(seqno0, prevhash0, blob) 和 deliver(seqno1, prevhash1, blob’) 发生在正确的节点 和 blob == blob’, 那么 seqno0==seqno1 和 prevhash0==prevhash1.
6. 活跃性。如果正确的客户端调用事件broadcast(blob)那么每个正确的peer节点“最终”发出事件deliver(, , blob)，其中*表示任意值。





## 参考
[架构说明](https://hyperledgercn.github.io/hyperledgerDocs/arch-deep-dive_zh/)

[官方协议规范](https://github.com/hyperledger/fabric/blob/v0.6/docs/source/protocol-spec_zh.rst)