# EIP-225: Clique 权威证明共识协议(Clique proof-of-authority consensus protocol)
## 抽象的
Clique 是一种权威证明共识协议。它隐藏了以太坊主网的设计，因此可以毫不费力地将其添加到任何客户端。
## 动机
以太坊的第一个官方测试网是 Morden。它从 2015 年 7 月持续到 2016 年 11 月左右，当时由于累积的垃圾以及 Geth 和 Parity 之间的一些测试网共识问题，它最终被搁置以支持测试网重启。

Ropsten 就这样诞生了，清除了所有垃圾并从头开始。这种情况一直持续到 2017 年 2 月底，当时恶意行为者决定滥用低 PoW 并逐渐将区块 gas 限制提高到 90 亿（从正常的 470 万），此时发送的巨额交易使整个网络瘫痪。甚至在此之前，攻击者就尝试了多次极长的重组，导致不同客户端甚至不同版本之间的网络分裂。

这些攻击的根本原因是 PoW 网络的安全性取决于其背后的计算能力。从零重新启动一个新的测试网并不能解决任何问题，因为攻击者可以一遍又一遍地发起相同的攻击。Parity 团队决定采用紧急解决方案，回滚大量区块，并制定软分叉规则，禁止气体限制超过特定阈值。

虽然此解决方案可能在短期内起作用：

- 这并不优雅：以太坊应该有动态块限制
- 不可移植：其他客户端需要自己实现新的分叉逻辑
- 它与同步模式不兼容：快速和轻型客户端都不走运
- 这只是延长了攻击时间：垃圾仍然可以无限期地稳步推进

Parity 的解决方案虽然不完美，但仍然可行。我想提出一个更长期的替代解决方案，它涉及更多，但应该足够简单，可以在合理的时间内推出。

### 标准化的权威证明
如上所述，工作量证明不能在没有价值的网络中安全地工作。以太坊的长期目标是基于 Casper 的权益证明，但这是一项繁重的研究，因此我们不能很快依靠它来解决今天的问题。然而，一种解决方案很容易实现，但足以有效地修复测试网，即权威证明方案。

此处描述的 PoA 协议的主要设计目标是实现并嵌入到任何现有的以太坊客户端应该非常简单，同时允许使用现有的同步技术（快速、轻量、扭曲），而无需客户端开发人员添加关键软件的自定义逻辑。

## 设计约束
通常有两种同步区块链的方法：

- 经典的方法是采用创世块并一一处理所有交易。这是经过尝试和证明的，但在以太坊复杂网络中，很快就会证明计算成本非常高。
- 另一种方法是只下载区块头链并验证它们的有效性，然后可以从网络下载任意最近的状态并检查最近的头。

PoA 方案基于区块只能由受信任的签名者铸造的想法。因此，客户端看到的每个块（或标头）都可以与受信任的签名者列表进行匹配。

这里的挑战是如何维护一个可以及时更改的授权签名者列表？

显而易见的答案（将其存储在以太坊合约中）也是错误的答案,因为快速、轻量和扭曲同步在同步期间无法访问状态。

### 维护授权签名者列表的协议必须完全包含在区块头中。
这个想法是改变区块头的结构，从而放弃 PoW 的概念，并引入新的字段来满足投票机制的需求。这也是错误的答案：在多个实现中更改这样一个核心数据结构将是一场噩梦般的开发、维护和安全明智的选择。
### 维护授权签名者列表的协议必须完全适合当前的数据模型。
因此，根据上述内容，我们不能使用 EVM 进行投票，而必须求助于 headers。并且我们无法更改标题字段，而必须求助于当前可用的字段。没有多少回旋余地。

### 重新利用标头字段以进行签名和投票
当前仅用作有趣元数据的最明显字段是块头中的 32 字节额外数据部分。矿工通常将他们的客户端和版本放在那里，但有些人会用替代的“消息”填充它。该协议将扩展这个领域到具有 65 个字节，用于 secp256k1 矿工签名。这将允许任何获得块的人根据授权签名者列表对其进行验证。它还使区块头中的矿工部分过时（因为地址可以从签名中获得）。

请注意，更改标头字段的长度是一种非侵入性操作，因为所有代码（例如 RLP 编码、散列）都与此无关，因此客户端不需要自定义逻辑。

以上足以验证链，但我们如何更新签名者的动态列表。答案是我们可以重新利用新废弃的  `miner` 字段和 PoA 废弃的 `nonce` 字段来创建投票协议：

- 在常规块期间，这两个字段都将设置为零。
- 如果签名者希望更改授权签名者列表，它将：
	- 将 `miner ` 设置为它希望投票的签名者
	- 将 `nonce` 设置为 0 或 `0xff...f` 投票赞成添加或踢出
	
任何同步链的客户端都可以在区块处理期间“统计”投票，并通过大众投票维护动态变化的授权签名者列表。

为了避免有一个无限的窗口来统计投票，并允许定期刷新陈旧的提案，我们可以重用来自 `ethash` 的 `epoch` 的概念，其中每个 `epoch` 转换都会刷新所有未决的投票。此外，这些纪元转换还可以充当无状态检查点，其中包含标头额外数据中的当前授权签名者列表。这允许客户端仅基于检查点散列进行同步，而不必重播在该点之前在链上完成的所有投票。它还允许创世头完全定义链，包含初始签名者列表。

### 攻击媒介：恶意签名者
可能会发生恶意用户被添加到签名者列表中或者签名者密钥/机器被泄露的情况。在这种情况下，协议需要能够抵御重组和垃圾邮件。建议的解决方案是，给定 N 个授权签名者的列表，任何签名者只能从每个 K 中铸造 1 个区块。这确保了损害是有限的，并且其余的矿工可以投票排除恶意用户。
### 攻击媒介：审查签名者
另一个有趣的攻击媒介是如果签名者（或签名者组）试图审查投票赞成将其从授权列表中删除的区块。为了解决这个问题，我们将签名者的允许铸造频率限制为 N/2 中的 1 个。这确保了恶意签名者需要控制至少 51% 的签名帐户，在这种情况下无论如何游戏都结束了。
### 攻击媒介：垃圾邮件签名者
最后一个小攻击向量是恶意签名者在他们铸造的每个区块中注入新的投票提案。由于节点需要统计所有投票以创建授权签名者的实际列表，因此它们需要随时间跟踪所有投票。如果不对投票窗口设置限制，这可能会缓慢但无限制地增长。解决方法是放置一个移动 W 块的窗口，之后投票被认为是陈旧的。 一个正常的窗口可能是 1-2 个 epoch。 我们称之为一个时代。
### 攻击向量：并发块
如果授权签名者的数量为 N，并且我们允许每个签名者从 K 中铸造 1 个区块，那么在任何时间点都允许 N-K+1 个矿工进行铸造。为了避免争夺区块，每个签名者都会在发布新区块的时间添加一个小的随机“偏移量”。这确保了小分叉很少见，但偶尔仍会发生（如在主网上）。如果签名者被发现滥用职权并造成混乱，则可以将其投票否决。
## 规格
我们定义了以下常量：

- `EPOCH_LENGTH`

	在此之后检查点并重置未决投票的块数。
	
	- 建议 30000 测试网与主网 ethash 时代保持类似。
- `BLOCK_PERIOD`

	两个连续块的时间戳之间的最小差异。
	
	- 建议 15s 测试网与主网 ethash 目标保持类似。
- `EXTRA_VANITY`

	为签名者虚荣保留的固定数量的额外数据前缀字节。
	
	- 建议 32 bytes 保留当前的额外数据限额和/或使用。
- `EXTRA_SEAL`

	为签名者印章保留的固定数量的额外数据后缀字节。
	
	- 65 bytes 固定，因为签名基于标准 secp256k1 曲线。
	- 在创世块上填充零。
- `NONCE_AUTH`

	`0xffffffffffffffff` 用于投票添加新签名者的魔术随机数。
- `NONCE_DROP`

	`0x0000000000000000` 用于投票删除签名者的魔术随机数。
- `UNCLE_HASH`

	总是 Keccak256(RLP([])) 因为叔叔在 PoW 之外毫无意义。
- `DIFF_NOTURN`

	包含乱序签名的块的块得分（难度）。

	- 建议，`1` 因为它只需要是一个任意的基线常数。
- `DIFF_INTURN`

	包含依次签名的块的块得分（难度）。

	- 建议 `2` 表现出对失序签名的轻微偏好。

我们还定义了以下每个块的常量：

- `BLOCK_NUMBER`
	
	链中的区块高度，其中创世的高度为 block 0。
- `SIGNER_COUNT`

	在链中特定实例有效的授权签名者的数量。
- `SIGNER_INDEX`

	当前授权签名者排序列表中区块签名者的从零开始的索引。
- `SIGNER_LIMIT`

	签名者只能签署一个的连续块数。

	- 必须是 `floor(SIGNER_COUNT/2) + 1` 在链上强制执行多数共识。

我们重新调整 `ethash` 标题字段的用途如下：

- `beneficiary/miner`

	建议修改授权签名者列表的地址

	- 通常应该用零填充，仅在投票时修改。
	- 尽管如此，还是允许使用任意值（甚至是无意义的值，例如投票出非签名者），以避免在围绕投票机制的实现中产生额外的复杂性。
	- 必须在检查点（即纪元转换）块上填充零。
	- 交易执行必须使用实际的区块签名者（参见 `extraData`）作为 `COINBASE` 操作码，并且交易费用必须归属于签名者帐户。
- `nonce`

	关于 `beneficiary` 字段定义的帐户的签名者提案。

	- 应该是 `NONCE_DROP` 提议取消授权 `beneficiary` 作为现有签名者。
	- 应该是 `NONCE_AUTH` 提议授权 `beneficiary` 为新的签名者。
	- 必须在检查点（即纪元转换）块上填充零。
	- 必须不占用除了两种以上的任何其它值（现在）。
- `extraData`

	签名者虚荣、检查点和签名者签名的组合字段。

	- 第一个 `EXTRA_VANITY` 字节（固定）可能包含任意签名者虚荣数据。
	- 最后一个 `EXTRA_SEAL` 字节（固定）是签名者的签名，用于密封标头。
	- 检查点块之间必须包含签名者列表 (`N*20 bytes`)，否则省略。
	- 检查点块额外数据部分中的签名者列表必须按字节升序排序。
- `mixHash`

	保留 fork 保护逻辑，类似于 DAO 期间的 `extra-data`。
	
	- 在正常操作期间必须用零填充。
- `ommersHash`

	一定是 `UNCLE_HASH` 因为叔叔在 PoW 之外毫无意义。
- `timestamp`

	必须至少是父时间戳 + `BLOCK_PERIOD`。
- `difficulty`

	包含区块的独立分数以推导出链的质量。
	
	- 必须是 `DIFF_NOTURN` 如果 `BLOCK_NUMBER % SIGNER_COUNT != SIGNER_INDEX`
	- 必须是 `DIFF_INTURN` 如果 `BLOCK_NUMBER % SIGNER_COUNT == SIGNER_INDEX`

### 授权区块
要为网络授权一个区块，签名者需要签署包含 `除签名本身之外的所有内容` 的区块的 sighash 。这意味着该散列包含标头的每个字段（包含的 `nonce` 和 `mixDigest` ），以及 `extraData` 65 字节签名后缀除外。这些字段按照它们在黄纸中的定义顺序进行散列。请注意，这个 `sighash` 不同于最终的区块哈希，后者也包括签名。

使用标准 `secp256k1` 曲线对 sighash 进行签名，并将生成的 65 字节签名（R, S, V, 其中V是0或1）`extraData` 作为尾随的 65 字节后缀嵌入。

为确保恶意签名者（或丢失签名密钥）不会在网络中造成严重破坏，每个授权节点最多可以在 `SIGNER_LIMIT` 连续区块中签名一个。顺序不是固定的，但轮转签名 `DIFF_INTURN` 比轮转 一（`DIFF_NOTURN`）更重要。
####  授权策略
只要签名者符合上述规范就可以按照认为合适的方式授权和分发区块。但是，以下建议的策略将减少网络流量和小分支，因此这是一个建议的功能：

- 如果签名者被允许签署一个区块（在授权名单上并且最近没有签名）。
	- 计算下一个区块（父+ BLOCK_PERIOD）的最佳签名时间。
	- 如果签名者轮流，等待准确时间到达，立即签名并广播。
	- 如果签名者不合时宜，则延迟签名 `rand(SIGNER_COUNT * 500ms)`。

这个小策略将确保轮内签名者（谁的区块权重更大）与轮外签名者相比，在签名和传播方面略有优势。此外，该方案允许随着签名者数量的增加而进行一些扩展。

### 对签名者进行投票
每个纪元转换（包括创世块）都充当无状态检查点，有能力的客户端应该能够从中同步而无需任何先前的状态。这意味着纪元头不能包含投票，所有未解决的投票都将被丢弃，并且从头开始计数。

对于所有非时代过渡块：

- 签名者可以为每个区块投一票以提议更改授权列表。
- 只有每个目标受益人的最新提案才会被单一签名者保留。
- 随着链的进展，投票会实时统计（允许并发提案）。
- 达成多数共识的提案 `SIGNER_LIMIT` 立即生效。
- 无效的提案不会因为客户端实现的简单性而受到惩罚。

`一项生效的提案需要放弃对该提案的所有未决投票（支持和反对），并从头开始。`

#### 级联投票
在签名者取消授权期间可能会出现复杂的极端情况。当先前授权的签名者被删除时，批准提案所需的签名者数量可能会减少一个。这可能会导致一个或多个待决提案达成多数共识，其执行可能会进一步导致新提案的通过。

当多个相互冲突的提案同时通过时（例如添加新签名者与删除现有签名者），处理这种情况并不明显，其中评估顺序可能会彻底改变最终授权列表的结果。由于签名者可能会在他们铸造的每个区块中颠倒他们自己的投票，因此哪个提案将是“第一”并不那么明显。

为了避免级联执行所带来的陷阱，Clique 提案明确禁止级联效应。换句话说：`只有 beneficiary 当前标题/投票可以添加到授权列表中/从授权列表中删除。如果这导致其他提案达成共识，则这些提案将在其各自的受益人再次被“触及”时执行（假设多数共识在那时仍然成立）。`
#### 投票策略
由于区块链可以进行小规模重组，“一劳永逸”的幼稚投票机制可能不是最佳的，因为包含单例投票的区块可能不会最终出现在最终链上。

一个简单但有效的策略是

- 允许用户在签名者上配置 “建议”（例如“添加 0x...”、“删除 0x...”）。
- 然后签名代码可以为其签名的每个块选择一个随机提议并注入它。这可确保最终在链上注意到多个并发提案以及重组。

此列表可能会在经过一定数量的区块/时期后过期，但重要的是要意识到“看到”提案通过并不意味着它不会被重组，因此不应在提案通过时立即删除它。

## 测试用例
	// block represents a single block signed by a parcitular account, where
	// the account may or may not have cast a Clique vote.
	// 区块代表由特定帐户签名的单个区块，其中该帐户可能投了也可能没有投了 Clique 投票。
	type block struct {
	  signer     string   // Account that signed this particular block // 签署这个特定区块的账户
	  voted      string   // Optional value if the signer voted on adding/removing someone // 可选值，如果签名者对添加/删除某人进行投票
	  auth       bool     // Whether the vote was to authorize (or deauthorize) // 投票是否授权（或取消授权）
	  checkpoint []string // List of authorized signers if this is an epoch block // 授权签名者列表，如果这是一个纪元块
	}
	
	// Define the various voting scenarios to test
	// 定义要测试的各种投票场景
	tests := []struct {
	  epoch   uint64   // Number of blocks in an epoch (unset = 30000) // 一个 epoch 中的块数 (unset = 30000)
	  signers []string // Initial list of authorized signers in the genesis // 创世中授权签名者的初始列表
	  blocks  []block  // Chain of signed blocks, potentially influencing auths // 签名区块链，可能影响身份验证
	  results []string // Final list of authorized signers after all blocks // 所有区块之后的最终授权签名者列表
	  failure error    // Failure if some block is invalid according to the rules // 如果某些块根据规则无效则失败
	}{
	  {
	    // Single signer, no votes cast // 单人签名，不投票
	    signers: []string{"A"},
	    blocks:  []block{
	      {signer: "A"}
	    },
	    results: []string{"A"},
	  }, {
	    // Single signer, voting to add two others (only accept first, second needs 2 votes) // 单签，投票添加另外两个（只接受第一个，第二个需要2票）
	    signers: []string{"A"},
	    blocks:  []block{
	      {signer: "A", voted: "B", auth: true},
	      {signer: "B"},
	      {signer: "A", voted: "C", auth: true},
	    },
	    results: []string{"A", "B"},
	  }, {
	    // Two signers, voting to add three others (only accept first two, third needs 3 votes already) // 两个签名者，投票添加另外三个（只接受前两个，第三个已经需要3票了）
	    signers: []string{"A", "B"},
	    blocks:  []block{
	      {signer: "A", voted: "C", auth: true},
	      {signer: "B", voted: "C", auth: true},
	      {signer: "A", voted: "D", auth: true},
	      {signer: "B", voted: "D", auth: true},
	      {signer: "C"},
	      {signer: "A", voted: "E", auth: true},
	      {signer: "B", voted: "E", auth: true},
	    },
	    results: []string{"A", "B", "C", "D"},
	  }, {
	    // Single signer, dropping itself (weird, but one less cornercase by explicitly allowing this) // 单一签名者，放弃自己（奇怪，但通过明确允许这样做减少了一个角落情况）
	    signers: []string{"A"},
	    blocks:  []block{
	      {signer: "A", voted: "A", auth: false},
	    },
	    results: []string{},
	  }, {
	    // Two signers, actually needing mutual consent to drop either of them (not fulfilled) // 两个签名者，实际上需要双方同意才能删除其中一个（未完成）
	    signers: []string{"A", "B"},
	    blocks:  []block{
	      {signer: "A", voted: "B", auth: false},
	    },
	    results: []string{"A", "B"},
	  }, {
	    // Two signers, actually needing mutual consent to drop either of them (fulfilled) // 两个签名者，实际上需要双方同意才能删除其中一个（已完成）
	    signers: []string{"A", "B"},
	    blocks:  []block{
	      {signer: "A", voted: "B", auth: false},
	      {signer: "B", voted: "B", auth: false},
	    },
	    results: []string{"A"},
	  }, {
	    // Three signers, two of them deciding to drop the third // 三个签名者，其中两个决定放弃第三个
	    signers: []string{"A", "B", "C"},
	    blocks:  []block{
	      {signer: "A", voted: "C", auth: false},
	      {signer: "B", voted: "C", auth: false},
	    },
	    results: []string{"A", "B"},
	  }, {
	    // Four signers, consensus of two not being enough to drop anyone // 四个签名者，两个人的共识不足以放弃任何人
	    signers: []string{"A", "B", "C", "D"},
	    blocks:  []block{
	      {signer: "A", voted: "C", auth: false},
	      {signer: "B", voted: "C", auth: false},
	    },
	    results: []string{"A", "B", "C", "D"},
	  }, {
	    // Four signers, consensus of three already being enough to drop someone // 四个签名者，三个人的共识已经足以放弃某人
	    signers: []string{"A", "B", "C", "D"},
	    blocks:  []block{
	      {signer: "A", voted: "D", auth: false},
	      {signer: "B", voted: "D", auth: false},
	      {signer: "C", voted: "D", auth: false},
	    },
	    results: []string{"A", "B", "C"},
	  }, {
	    // Authorizations are counted once per signer per target // 每个目标每个签名者的授权计数一次
	    signers: []string{"A", "B"},
	    blocks:  []block{
	      {signer: "A", voted: "C", auth: true},
	      {signer: "B"},
	      {signer: "A", voted: "C", auth: true},
	      {signer: "B"},
	      {signer: "A", voted: "C", auth: true},
	    },
	    results: []string{"A", "B"},
	  }, {
	    // Authorizing multiple accounts concurrently is permitted // 允许同时授权多个账户
	    signers: []string{"A", "B"},
	    blocks:  []block{
	      {signer: "A", voted: "C", auth: true},
	      {signer: "B"},
	      {signer: "A", voted: "D", auth: true},
	      {signer: "B"},
	      {signer: "A"},
	      {signer: "B", voted: "D", auth: true},
	      {signer: "A"},
	      {signer: "B", voted: "C", auth: true},
	    },
	    results: []string{"A", "B", "C", "D"},
	  }, {
	    // Deauthorizations are counted once per signer per target // 每个目标每个签名者计算一次取消授权
	    signers: []string{"A", "B"},
	    blocks:  []block{
	      {signer: "A", voted: "B", auth: false},
	      {signer: "B"},
	      {signer: "A", voted: "B", auth: false},
	      {signer: "B"},
	      {signer: "A", voted: "B", auth: false},
	    },
	    results: []string{"A", "B"},
	  }, {
	    // Deauthorizing multiple accounts concurrently is permitted // 允许同时取消多个账户的授权
	    signers: []string{"A", "B", "C", "D"},
	    blocks:  []block{
	      {signer: "A", voted: "C", auth: false},
	      {signer: "B"},
	      {signer: "C"},
	      {signer: "A", voted: "D", auth: false},
	      {signer: "B"},
	      {signer: "C"},
	      {signer: "A"},
	      {signer: "B", voted: "D", auth: false},
	      {signer: "C", voted: "D", auth: false},
	      {signer: "A"},
	      {signer: "B", voted: "C", auth: false},
	    },
	    results: []string{"A", "B"},
	  }, {
	    // Votes from deauthorized signers are discarded immediately (deauth votes) // 来自被取消授权的签名者的投票会被立即丢弃（deauth 投票）
	    signers: []string{"A", "B", "C"},
	    blocks:  []block{
	      {signer: "C", voted: "B", auth: false},
	      {signer: "A", voted: "C", auth: false},
	      {signer: "B", voted: "C", auth: false},
	      {signer: "A", voted: "B", auth: false},
	    },
	    results: []string{"A", "B"},
	  }, {
	    // Votes from deauthorized signers are discarded immediately (auth votes) // 来自未授权签名者的投票会立即被丢弃（授权投票）
	    signers: []string{"A", "B", "C"},
	    blocks:  []block{
	      {signer: "C", voted: "D", auth: true},
	      {signer: "A", voted: "C", auth: false},
	      {signer: "B", voted: "C", auth: false},
	      {signer: "A", voted: "D", auth: true},
	    },
	    results: []string{"A", "B"},
	  }, {
	    // Cascading changes are not allowed, only the account being voted on may change // 不允许级联更改，只有被投票的账户可以更改
	    signers: []string{"A", "B", "C", "D"},
	    blocks:  []block{
	      {signer: "A", voted: "C", auth: false},
	      {signer: "B"},
	      {signer: "C"},
	      {signer: "A", voted: "D", auth: false},
	      {signer: "B", voted: "C", auth: false},
	      {signer: "C"},
	      {signer: "A"},
	      {signer: "B", voted: "D", auth: false},
	      {signer: "C", voted: "D", auth: false},
	    },
	    results: []string{"A", "B", "C"},
	  }, {
	    // Changes reaching consensus out of bounds (via a deauth) execute on touch // 越界达成共识的更改（通过 deauth）在触摸时执行
	    signers: []string{"A", "B", "C", "D"},
	    blocks:  []block{
	      {signer: "A", voted: "C", auth: false},
	      {signer: "B"},
	      {signer: "C"},
	      {signer: "A", voted: "D", auth: false},
	      {signer: "B", voted: "C", auth: false},
	      {signer: "C"},
	      {signer: "A"},
	      {signer: "B", voted: "D", auth: false},
	      {signer: "C", voted: "D", auth: false},
	      {signer: "A"},
	      {signer: "C", voted: "C", auth: true},
	    },
	    results: []string{"A", "B"},
	  }, {
	    // Changes reaching consensus out of bounds (via a deauth) may go out of consensus on first touch // 越界达成共识的更改（通过 deauth）可能会在第一次接触时超出共识
	    signers: []string{"A", "B", "C", "D"},
	    blocks:  []block{
	      {signer: "A", voted: "C", auth: false},
	      {signer: "B"},
	      {signer: "C"},
	      {signer: "A", voted: "D", auth: false},
	      {signer: "B", voted: "C", auth: false},
	      {signer: "C"},
	      {signer: "A"},
	      {signer: "B", voted: "D", auth: false},
	      {signer: "C", voted: "D", auth: false},
	      {signer: "A"},
	      {signer: "B", voted: "C", auth: true},
	    },
	    results: []string{"A", "B", "C"},
	  }, {
	    // Ensure that pending votes don't survive authorization status changes. This
	    // corner case can only appear if a signer is quickly added, removed and then
	    // readded (or the inverse), while one of the original voters dropped. If a
	    // past vote is left cached in the system somewhere, this will interfere with
	    // the final signer outcome.
	    
	    // 确保未决投票不会在授权状态更改后继续存在。 这个只有在快速添加、删除签名者然后已读（或相反），
	    // 而原始选民之一下降。 如果一个过去的投票被缓存在系统某处，这会干扰最终签名者结果。

	    signers: []string{"A", "B", "C", "D", "E"},
	    blocks:  []block{
	      {signer: "A", voted: "F", auth: true}, // Authorize F, 3 votes needed // 授权F，需要3票
	      {signer: "B", voted: "F", auth: true},
	      {signer: "C", voted: "F", auth: true},
	      {signer: "D", voted: "F", auth: false}, // Deauthorize F, 4 votes needed (leave A's previous vote "unchanged") // 取消对 F 的授权，需要 4 票（保持 A 之前的投票“不变”）
	      {signer: "E", voted: "F", auth: false},
	      {signer: "B", voted: "F", auth: false},
	      {signer: "C", voted: "F", auth: false},
	      {signer: "D", voted: "F", auth: true}, // Almost authorize F, 2/3 votes needed // 差不多授权F，需要2/3票
	      {signer: "E", voted: "F", auth: true},
	      {signer: "B", voted: "A", auth: false}, // Deauthorize A, 3 votes needed // 取消授权 A，需要 3 票
	      {signer: "C", voted: "A", auth: false},
	      {signer: "D", voted: "A", auth: false},
	      {signer: "B", voted: "F", auth: true}, // Finish authorizing F, 3/3 votes needed // 完成授权F，需要3/3票
	    },
	    results: []string{"B", "C", "D", "E", "F"},
	  }, {
	    // Epoch transitions reset all votes to allow chain checkpointing // Epoch 转换重置所有投票以允许链检查点
	    epoch:   3,
	    signers: []string{"A", "B"},
	    blocks:  []block{
	      {signer: "A", voted: "C", auth: true},
	      {signer: "B"},
	      {signer: "A", checkpoint: []string{"A", "B"}},
	      {signer: "B", voted: "C", auth: true},
	    },
	    results: []string{"A", "B"},
	  }, {
	    // An unauthorized signer should not be able to sign blocks // 未经授权的签名者不应该能够对区块进行签名
	    signers: []string{"A"},
	    blocks:  []block{
	      {signer: "B"},
	    },
	    failure: errUnauthorizedSigner,
	  }, {
	    // An authorized signer that signed recenty should not be able to sign again // 近期签署过的授权签署人应该不能再次签署
	    signers: []string{"A", "B"},
	  blocks []block{
	      {signer: "A"},
	      {signer: "A"},
	    },
	    failure: errRecentlySigned,
	  }, {
	    // Recent signatures should not reset on checkpoint blocks imported in a batch // 最近的签名不应在批量导入的检查点块上重置
	    epoch:   3,
	    signers: []string{"A", "B", "C"},
	    blocks:  []block{
	      {signer: "A"},
	      {signer: "B"},
	      {signer: "A", checkpoint: []string{"A", "B", "C"}},
	      {signer: "A"},
	    },
	    failure: errRecentlySigned,
	  },,
	}
	
## 参考
- [issue 地址](https://github.com/ethereum/EIPs/issues/225) 
- [EIP-225: Clique proof-of-authority consensus protocol](https://eips.ethereum.org/EIPS/eip-225)