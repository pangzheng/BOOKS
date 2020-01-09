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

	内射函数 `next: N -> N` 获取 `N` 的一个元素并返回下一个版本号 。`V` 和 `N` 都包含一个特定的元素 `\bot`，这是 `N` 的最底层元素的特例。最开始时所有的键都映射到 `(\bot,\bot)`。
	
	对于 `s(k)=(v,ver)`
	
	- `s(k).value` 代表 `V`
	- `s(k).version` 代表 `ver`

KV操作模型

- `put(k,v)`,对于 `k\ in K` 和 `v\ in V`。
- 处理区块链状态 `s`，将它变为`s'`，这样`s'(k)=(v,next(s(k).version))`，以及 `s'(k')=s(k')` 以保证所有的 `k'!=k`
- `get(k)` 返回 `s(k)`

状态由 peer 节点保持，而不是 order 节点和client。

- 状态划分

	KV 中的键能够通过它们的名字识别属于哪个特定的链码。从这点上说，只有某个链码的交易可以修改属于这个链码的键。原则上，任何链码能够读取属于其它链码的键。支持跨链交易，修改属于两个或更多链码的状态是 V1 后期的特征。

#### 1.2.2 账本（Ledger）
账本提供了在系统运行过程中发生的可验证的历史，它包含所有成功的状态更改（称为有效交易）和不成功的状态更改尝试（称为无效交易）。

账本是由 order 服务构建的一个全部有序的交易哈希链块（有效的或无效的）。哈希链强制将全部排序块置入账本，每个块包含一批全部排序交易。这个强制全部排序覆盖所有交易。

账本保存在所有 peer 节点，(可选，保存在排序者的一个子集)。在谈论排序时的账本是 `OrdererLedger`，而谈论 peer 节点时的账本是 `PeerLedger`。

`PeerLedger` 与 `OrdererLedger` 的区别是，peer 节点本地维护一个位掩码来表明隔离有效交易和无效交易。

Peer 节点可以按照 Section XX 修剪 `PeerLedger`内容 。Orderers 节点维护 `OrdererLedger`用于实现容错性和可用性(`PeerLedger`)，可以提前在 Order 服务的保留属性后，随时修改 `PeerLedger`

账本允许 peer 节点重演所有交易的历史和重建状态。

### 1.3. 节点(Nodes)
Node 节点是区块链的通信实体。一个 “node节点” 仅仅是一个逻辑函数，多个不同类型的 node 节点可以运行在同一台物理服务器上。关键在于 node 节点如何在 “信任域” 中分组和关联控制它们的逻辑实体。

有三种类型的 node 节点：

1. `client(Client)`或叫`提交client(submitting-client)`

	client提交实际交易调用到背书者，广播交易请求到 Order 节点。

2. `Peer`

	提交交易、维持状态和账本的拷贝。此外，peer 节点可以有一个特殊的背书角色。

3. `order 服务节点(Ordering-service-node)`或`排序者(Orderer)`

	运行通信服务实现交付保证，像原子或全域广播。

#### 1.3.1. client(Client)
client代表最终用户实体。它必须连接到一个 peer 节点以便与区块链交互。client可以选择连接任何 peer 节点。client创建并调用交易。

如第2节说明，client同时与 peer 节点和 order 服务通信。

#### 1.3.2. Peer
Peer 节点以块的形式从 order 服务接收有序状态更新，维护状态和账本。

Peer 节点还可以担任 `背书节点(endorsing peer)` 或 `背书者(endorser)`的特殊角色。背书节点的特殊功能是“在提交事物之前认可交易”这个特殊链码才发生作用。每个链码可以指定一个“引用一组背书节点”的背书策略。该策略为一个有效交易背书（典型的是一组背书者签名）定义了必要和充分的条件。在部署交易(安装链码)的特殊情况下，背书策略是由系统链码的背书策略指定。

#### 1.3.3. Orderer
排序者产生 order 服务，即一个提供交付保证的通信架构。order 服务能以不同的方式实现：

- 集中服务排序（例如，开发和测试）
- 指向不同的网络和节点故障模型的分布式协议

order 服务为client和 peer 节点提供共享的通信信道，为包含交易的消息提供广播服务。client连接到信道，并可以在这个信道上广播消息，随后传递消息给所有 peer 节点。信道支持所有消息的原子传递，意思是，总排序交付的消息通信和可靠性。信道将相同消息输出给所有连接的 peer 节点并且输出的消息具有同样的逻辑顺序。这个原子通信保证也称为总排序广播，原子广播，或是分布式系统中的共识。通信消息是包含在区块链状态中的申请交易。

- `分隔（order 服务信道）Partitioning (ordering service channels)`

	order 服务可以支持多个信道，类发布/订阅主题消息系统 (pub/sub)。client能够连接到一个给定的信道，然后能够发送消息和获得到达的消息。信道能够被认为是分区的(client连接到一个信道而无需知道其它信道的存在)，client也可以连接到多个信道。尽管一些 order 服务实现包括 Fabric v1 将支持多信道，为了阐述简单剩余部分，假定 order 服务包含一个单独的信道/主题(channel/topic)。
- `order 服务 API(Ordering service API)`

	peer 节点通过 order 服务提供的接口连接到 order 服务提供的信道。order 服务 API 包含两个基本操作（更多是异步事件-正在做：增加在client/peer节点指定序列号下取得特定块的 API。）：

	- broadcast(blob)
	
		client调用此函数来广播任意消息 blob 在信道散播。当发送一个请求到服务器时,在 BFT 环境下也称为 request(blob)。
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

	1. 安全性（一致性保证）

		只要 peer 节点连接到信道足够长的时间（它们能够断开或崩溃，但会重启和重新连接），它们会看到一系列相同的传递（`seqno,prevhash,blob`）消息。这意味着向所有 peer 节点输出（ `deliver() events`）相同排序，以及根据相同序列号携带同等内容（`blob` 和 `prevhash`）。
		
		注意这仅是一个逻辑顺序，在任何实时的关系中，任意 peer 节点上的 `deliver(seqno,prevhash,blob)` 是不需要与另外一个 peer 节点发生任何实时关系。换句话说，给定一个特定的 seqno，两个正确的 peer 节点交付相同的 prehash 或 blob 值。而且，除非某个 client 实际调用了广播(blob),否则不会传递 blob 值，最好每次广播 blob 只传递一次.
	
		此外，`deliver()` 事件包含之前 `deliver()` 事件`（prevhash）`的数据加密哈希。当 order 服务实现原子广播保证，`prevhash` 是从序列号为 `seqno-1` 的 `deliver()` 事件得到的参数的加密哈希，这在第4节和第5节讨论。对于第一个 `deliver()` 事件特例，`prevhash` 有一个缺省值。
	2. 活跃度（交付保证）

		order 服务的活跃度保证由 order 服务实现确定。准确的保证可以依赖于网络和节点故障模型。

原则上，如果提交client没有失败，order 服务应该保证每个连接到order 服务的正确peer节点终究交付每个提交交易。

概括地说，order 服务确保以下特性

1. 一致

	对于任何两个具有相同 seqno 的正确 peer 节点的事件 `deliver(seqno, prevhash0, blob0)`和`deliver(seqno, prevhash1, blob1)` , 则 `prevhash0==prevhash1`，以及 `blob0==blob1`;
2. 哈希链完整性

	对于任何在正确 peer 节点的两个事件 `deliver(seqno-1, prevhash0, blob0)` 和 `deliver(seqno, prevhash, blob)`, `prevhash = HASH(seqno-1||prevhash0||blob0)`.
3. 没有跳过

	如果 order 服务在正确 peer 节点 p 输出 `deliver(seqno, prevhash, blob)` , 这样的话`seqno>0`, 然后 p 已经交付事件 `deliver(seqno-1, prevhash0, blob0)`.
4. 没有创造

	任何在正确 peer 节点上的事件 `deliver(seqno, prevhash, blob)` 必须之前一定有一个`broadcast(blob)` 事件在一些(可能是不同的) peer 节点上;
5. 没有重复 (可选,但可取)

	对于任何两个事件 `broadcast(blob)` 和 `broadcast(blob’)` , 当两个事件 `deliver(seqno0, prevhash0, blob)` 和 `deliver(seqno1, prevhash1, blob’)` 发生在正确的节点和 `blob == blob’`, 那么 `seqno0==seqno1` 和 `prevhash0==prevhash1`.
6. 活跃性

	如果正确的client调用事件 `broadcast(blob)` 那么每个正确的 peer 节点“最终”发出事件 `deliver(, , blob)`，其中*表示任意值。

## 2 交易背书的基本工作流程(Basic workflow of transaction endorsement)
接下来概述一个交易的高级请求流程。

备注：注意下面的协议不假定所有的交易是确定的，即，它允许不确定交易。
### 2.1 client 创建交易和发送给它选择的背书 peer 节点（The client creates a transaction and sends it to endorsing peers of its choice）
调用交易，client向其选择的一组背书 peer 节点发送一个 `PROPOSE` 消息（可能不是同时发送，见第2.1.2节和第2.3节）。给定 `chaincodeID` 的背书 peer 节点集合通过 peer 节点提供给 client，peer 又从背书策略（见第3节）知道背书 peer 节点集合的设置。例如，可以将事物发送给给定 `chaincodeID` 的所有背书者.也就是说，一些背书者能够离线，其它人可能反对并不签署交易。提交 client 将尝试使用可用背书者来满足背书策略。

接下来，我们首先描述 `PROPOSE` 消息格式，然后讨论在提交 client 和背书者之间可能的互动模式。
#### 2.1.1 PROPOSE 消息格式(PROPOSE message format)
一个 PROPOSE 消息的格式是，其中 `tx` 是强制的，`anchor` 可选参数在下面列出

- tx=
	- clientID 是提交客户端的身份
	- chaincodeID 引用交易所有的链码
	- txPayload 是提交交易自身的有效负载
	- timestamp 是由 client 维护的一个单独递增(为每一笔新交易)整型值
	- clientSig 是tx的其它字段 cinet 签名

txPayload 的细节会在调用交易和部署交易之间有所不同（即，调用交易引用部署指定的系统链码）

调用交易，txPayload 会包含两个域

- txPayload=
	- operation 表示链码操作（函数）和参数
	- metadata 表示调用相关的属性

部署交易，txPayload会包含三个域

- txPayload=
	- source 表示链码的源码
	- metadata 表示链码和应用的相关属性
	- policies 包含所有 peer 节点可访问的链码的相关策略，像背书策略。注意背书策略在部署交易中不支持txPayload，但部署的 txPayload 包含背书策略ID和它的参数（见第3节）。

anchor 包含 `read version` 依赖，或更具体地说，键-版本对（即，anchor 是 KxN 的一个子集），它捆绑或“锚” PROPOSE 请求到指定 KVS 中 key 的版本（第1.2节）。如果 client 指定 anchor 参数，背书者背书交易的情况是，只基于读它本地 KVS 匹配 anchor 中的相应 KEY 的版本号（更详细内容见第2.2节）。

tx 加密哈希被所有 node 节点用作唯一的交易标识 tid（即，`tid=HASH(tx)`）。client 保存 tid 在内存中，等待背书 peer 节点的响应。

#### 2.1.2 消息模式(Message patterns)
clinet 决定与背书者互动的顺序。

例如，clinet 通常会发送（即，没有 anchor 参数）到一个单独的背书者，背书者随后产生版本依赖（anchor）,clinet 可以在晚些时候使用这个版本依赖（anchor）作为它的 PROPOSE 消息参数，发送给其它背书者。

另外的例子，clinet 能直接发送（没有 anchor）到它选择的所有背书者。可使用任意通信模式，clinet 在这方面是自由的（也见第2.3节）。

### 2.2. 背书 peer 节点模拟交易和产生背书签名(The endorsing peer simulates a transaction and produces an endorsement signature)
在从 client 接收消息时，背书 peer 节点 `epID` 首先校验 client 签名 `clientSig`，然后模拟一个交易。如果 client 指定了 anchor，那么背书 peer 节点模拟交易只基于在它本地 KVS 匹配的由 anchor 指定的版本号对应的 key 读版本号（即，下面定义的 readset）。

模拟一个交易涉及背书节点尝试执行一个交易(txPayload), 通过调用链码到交易引用（chaincodeID）和背书peer 节点本地持有的状态拷贝。

作为执行的结果，背书 peer 节点计算读版本依赖（readset）和状态更新（writeset），也在DB语言中称为MVCC+postimage info。

回顾状态包含键/值对。所有键/值对实体都是版本化的，那就是说，每个实体包含排序版本信息，它是在每次键的值更新时增加的。解释交易的 peer 节点记录了所有的被链码访问的键/值对，不管读或是写，peer 节点不会更新它的状态。更具体地说：

- 在背书节点执行一个交易前给定状态s，被交易读取的每个键k，键/值对 `(k,s(k).version)`被添加到 readset。
- 此外，对于每一个被交易编辑的键k到值v’，键/值对(k,v’)被添加到 writeset。或者，v’能成为新值与前值`(s(k).value)` 的增量。

如果 client 在 PROPOSE 消息中指定了 anchor，那么 client 指定的 anchor 在模拟交易时必须等于背书 peer 节点产生的 readset.

它的逻辑部分来背书交易，称为背书逻辑。缺省时，一个 peer 节点的背书逻辑接受交易提案并简单签署。无论如何，背书逻辑可以执行任意功能，例如，与原有系统交互交易提案和tx作为输入来得知是否背书交易。

如果背书逻辑决定背书一个交易，它发送消息到提交 client，其中:

- 交易提案(tran-proposal)

	`(epID,tid,chaincodeID,txContentBlob,readset,writeset)`, 其中 `txContentBlob` 是链码/交易专用信息。目的是让 `txContentBlob` 用作 tx 的一些陈述 (例如, `txContentBlob=tx.txPayload`).
- epSig 是背书 peer 节点的交易提案签名。

否则，假使背书逻辑拒绝背书交易，背书者可以发送消息(TRANSACTION-INVALID, tid, REJECTED)到提交客户端。

注意背书者在这一步不能改变它的状态(在背书没有影响状态的情况下交易模拟产生状态更新)。

### 2.3 提交 client 收集交易背书并通过排序服务广播它(The submitting client collects an endorsement for a transaction and broadcasts it through ordering service)
提交 client 一直等待直到它在(TRANSACTION-ENDORSED, tid, , )上收集到“足够”的消息和签名来推断出交易提案已背书。像在2.1.2节讨论的那样，这可能涉及一个或多个与背书者的往返。

“足够”的准确数字取决于链码背书策略（也见第3节）。如果背书策略是安全的，交易已经背书；注意它还没提交。签署 TRANSACTION-ENDORSED 消息的收集从背书 peer 节点来，背书 peer 节点建立了交易是背书的称为背书并以背书为名称。

如果提交 client 没有设法为交易提案收集背书，则放弃这个交易，稍后再试。

对于一个具有有效背书的交易，我们现在开始使用排序服务。提交 client 使用 broadcast(blob) 调用排序服务，其中 `blob=endorsement` .如果 client 没有能力直接调用排序服务，它可以通过它选择的 peer 节点代理广播。这样的 peer 节点必须被 client 信任不会从背书移除任何消息或其它可能被无效的交易。注意一点，无论如何，代理 peer 节点不可能制造有效背书。

### 2.4. 排序服务向 peer 节点提交交易(The ordering service delivers a transactions to the peers)
当一个事件 (seqno, prevhash, blob) 发生并且一个 peer 节点已为所有序列号低于 seqno 的 blosbs 更新状态，peer 节点执行如下流程：

- 它检查 `blob.endorsement` 是有效的，根据的是它引用的链码 `(blob.tran-proposal.chaincodeID)`。
- 在典型情况下，它也验证了依赖 `(blob.endorsement.tran-proposal.readset)` 在期间没有被违反。在更复杂的用例中，背书中的交易提案域可能不同，在这种情况下，背书策略（第3节）指定状态如何形成。

依赖的验证能以不同的方式实现，根据一致性属性或为状态更新选择的“孤立保证”。`Serializability` 是一个缺省的孤立保证，除非链码背书策略指定一个不同的。`Serializability` 能够通过在 readset 中的每个 key 关联的版本被提供，相当于 key 在状态中的版本，并拒绝不满足这个要求的交易。

- 如果所有这些检查通过，交易被视为有效或承诺。在这种情况下，peer 节点在 PeerLedger 用1标记交易，适用于 `blob.endorsement.tran-proposal.writeset` 区块链状态（如果交易提案是相同的，其它背书策略逻辑定义了函数处理 `blob.endorsement`）。
- 如果 `blob.endorsement` 背书策略验证失败，交易无效，并且 peer 节点在 PeerLedger 的位掩码用0标记交易。重要的是要注意无效交易不会改变状态。

注意，这里有足够的让所有（正确）peer 节点在处理一个给定序列号的 deliver 事件（块）之后具有同样的状态。即，通过排序服务的保证，所有正确的 peer 节点会收到相同的 `deliver(seqno, prevhash, blob)` 事件序列。当背书策略的评估和 readset 中版本依赖的评估是确定的，所有正确的 peer 节点也会得出相同的结论，关于包含在 blob 中的交易是否有效。因此，所有 peer 节点提交和应用同样交易序列并用同样的方式更新它们的状态。

![](./pic/flow-4.png)

上图是一种可能的交易流程说明（一般情况路径）

## 3 背书策略(Eorsement policies)
### 3.1. 背书策略规范(Endorsement policy specification)
背书策略，是背书一个交易的条件。区块链 peer 节点有一组预先确定的背书策略，它被安装特定链码的部署交易引用。背书策略能参数化，这些参数能被部署交易指定。

为了保证区块链和安全特性，背书策略组应该是一组验证过的策略，具有有限功能，为了保证有限的执行时间（终止），决定、性能和安全保证。

背书策略的动态添加（即，在链码部署时间由部署交易添加）是对背书评估时间限制（终止）、决定、性能和安全保证非常敏感的。因此，动态添加背书策略是不允许的，但将来能支持。
### 3.2. 针对背书策略的交易评估(Transaction evaluation against endorsement policy)
交易只有经过根据背书策略的背书才会宣布有效。对于链码的调用交易首先需要的到一个满足链码策略的背书，或不提交。这通过在提交 client 和背书 peer 节点之间的互动发生，在第2节解释。

正式的背书策略是以背书为基础，以及潜在的进一步评估为真假状态。对于部署交易，获得背书的依据是系统范围策略（例如，来自系统链码）。

背书策略断言引用一定的变量。潜在可能引用的是：

1. 与链码有关的钥匙或身份（在链码元数据中能发现），例如，一组背书者；
2. 链码进一步的元数据；
3. `endorsement and endorsement.tran-proposal` 的元素；
4. 其它更多。

上面的列表根据表现和复杂性排序，意思是说，它将会是相对简单的支持策略，只引用 node 节点的钥匙和身份。

背书策略断言的评估必须被确定。背书应当被每个 peer 节点本地评估，这样这个 peer 节点就不需要和其它 peer 节点在这件事情上交互，但所有正确的 peer 节点都以相同的方式评估背书策略。
### 3.3. 背书策略例子(Example endorsement policies)
断言可以包含逻辑表达式和评估真假。通常情况会对背书节点为链码发出的交易请求使用数字签名。

假定链码指定背书者集 `E={Alice, Bob, Charlie, Dave, Eve, Frank, George}`

一些例子策略如下：

- 一个有效签名来自全体E的成员的同样的交易提案
- 一个有效签名来自E的任一单个成员
- 从背书 peer 节点来的同一交易提案的有效签名条件是 `(Alice OR Bob) AND (any two of: Charlie, Dave, Eve, Frank, George)`.
- 同一提案的有效签名为7名背书者的任意5名。（更常用的，链码 n>3f 背书者，n名背书者有任意2f+1有效签名，或任意大于(n+f)/2背书者小组有效签名）
- 假定背书者有一个“股份”或“权重”的任务，像 `{Alice=49, Bob=15, Charlie=15, Dave=10, Eve=7, Frank=3, George=1}`, 其中全部股份是100：策略需要一组占大多数股份的有效签名（即，一组合并股份完全超过50），像{Alice, X}，X只要不是 George 的任何人，或{除去Alice以外的所有人}，等等。
- 假定前面例子中的股权条件是静态的（固定在链码的元数据中）或动态的（例如，取决于链码的状态和在执行中修改）。
- 交易提案1的有效签名来自 `(Alice OR Bob)` 和交易提案2有效签名来自`（Charlie, Dave, Eve, Frank, George中的任何两个）`，其中交易提案1和交易提案2的不同只在它们的背书 peer 节点和状态更新。

如何使用这些策略取决于应用、失败或恶意背书者的恢复能力和各种其它特性。

## 4 (post-v1). 证实账本和节点账本检查（修剪）(Validated ledger and PeerLedger checkpointing (pruning))
### 4.1. 验证账本（Validated ledger (VLedger)）
维护一个账本的抽象，只包含有效和提交交易（例如比特币的方案），peer 节点可以，除状态和账本外，维护证实账本（或VLedger）。这是一个哈希链，来自过滤掉无效交易的账本。

证实账本块的生成按如下顺序。

- 当节点账本块可能包含无效交易（即，交易的背书无效或版本依赖无效），这样的交易被 peer 节点在交易从块变为证实块之前过滤掉。
- 每个 peer 节点自身实现这点（例如，使用节点账本关联的位掩码）。证实块被定义为没有无效交易的块，是将过滤的块。这样证实块在大小上是动态的也可能是空的。证实块生成的说明在下图中给出。

![](./pic/blocks-3.png)

上图从节点账本块形成证实账本块

证实块被每个 peer 节点链接在一起形成一个哈希链。更具体地，证实账本的每个块包含

- 前证实块的哈希
- 证实块编号
- 从上一个证实块被计算出以来所有 peer 节点提交交易的排序列表（即，在相应块中的有效交易列表）
- 相应块的哈希（在节点账本中），来自得出的当前证实块

所有这些信息都被 peer 节点级联和哈希，产生证实账本中证实块的哈希。
### 4.2. 节点账本检查(PeerLedger Checkpointing)
账本包含的无效交易，没有必要永久记录。然而，一旦建立相应的证实块，peer 节点不能简单地通过丢弃节点账本块来修剪节点账本容量。即，在这种情况下，如果新的 peer 节点加入了网络，其它 peer 节点不能转移丢弃块（与节点账本有关的）到新加入的节点，也不能使新加入的 peer 节点承认它们的证实块。

为了便于节点账本修剪，这个文档描述一个检查点机制。这个机制建立了证实块的有效性，贯穿节点网络，允许检查点证实块替换丢弃的节点账本块。反过来，减少了存储空间，因为没有必要存储无效交易。它也减少了新加入的 peer 节点重构状态的工作量（当通过重演节点账本重构状态时，因为他们不需要建立有效的单个交易，但可以简单重演包含在节点账本中的状态更新。）

#### 4.2.1. 检查点协议(Checkpointing protocol)
检查点是由 peer 节点每个 CHK 块周期性地形成，这里 CHK 是一个可配置参数。开辟一个检查点，peer 节点广播（例如，传播）给其它 peer 节点 , 其中，`blockno` 是当前块编号，blocknohash 是各自的哈希，stateHash 是最新状态的哈希（产生于，例如 Merkle hash），基于确认的块编号，peerSig 是 peer 节点的对(CHECKPOINT,blocknohash,blockno,stateHash) 的签名，引用了证实账本。

peer 节点收集 CHECKPOINT 消息直到它得到匹配 blockno, blocknohash 和 stateHash 的足够正确的签名消息来建立一个有效的检查点。（见4.2.2节）

在为块编号 blockno 和 blocknohash 建立了有效的检查点的基础上，peer节点： 

- 如果 `blockno>latestValidCheckpoint.blockno`, 那么 peer 节点分配 `latestValidCheckpoint=(blocknohash,blockno)`
- 存储各 peer 节点的签名集，它构成了有效的检查点到集合 `latestValidCheckpointProof`
- 存储状态相应的 stateHash 到 `latestValidCheckpointedState`
- （可选的）修剪它的节点账本到块编blockno (包含).

#### 4.2.2. 有效检查点(Valid checkpoints)
显然，检查点协议增加了下面的问题：peer 节点什么时候能修剪它的 `PeerLedger` (节点账本)？多少 `CHECKPOINT` (检查点_消息是足够多的？这由检查点有效策略定义，要有（至少）两种可能的方法且也能合并：

- Local (peer-specific) checkpoint validity policy (LCVP)

	给定 peer 节点 p 上的本地策略可以确定一组 peer 节点，这一组 peer 节点是 p 信任的且它的CHECKPOINT 消息是足够建立一个有效的检查点。例如，在 peer 节点 Alice 上的 LCVP 可以定义本地（peer确定）检查点有效性策略（LCVP）
- Global checkpoint validity policy (GCVP) 全局检查点有效性策略（GCVP）。

	检查点有效策略可以确定为全局的。这类似于本地节点策略，不同之处在于它是按系统颗粒度而不是对等颗粒度规定。例如，GCVP可以指定：
	
	- 每个 peer 节点可以信任一个由 11 各不同 peer 节点确认的检查点。
	- 在一个特定部署中，每个排序者与 peer 节点配置在同一台机器上（即，信任域），最多可能有 f 个排序者可能（拜占庭）出现错误，每个 peer 节点可以信任一个检查点，如果经过 f+1 个排序者配置的不同的节点确认。

## 参考
[架构说明](https://hyperledgercn.github.io/hyperledgerDocs/arch-deep-dive_zh/)

[官方协议规范](https://github.com/hyperledger/fabric/blob/v0.6/docs/source/protocol-spec_zh.rst)