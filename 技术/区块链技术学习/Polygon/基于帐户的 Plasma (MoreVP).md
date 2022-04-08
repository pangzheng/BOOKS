# 基于帐户的 Plasma (MoreVP)
Matic Plasma 遵循类似于 Plasma MoreVP 的模型，但与其他基于 UTXO 的实现相比，它是基于账户的实现。侧链与 EVM 兼容。使用 MoreVP 结构，我们还消除了对确认签名的需要。
## PoS 层和检查点
Matic 网络使用检查点层的权益证明和块生产者层的块生产者的双重策略来实现更快的出块时间，并使用检查点和欺诈证明在主链上实现最终性。

在 Matic Network 的 checkpointing 层上，对于 Matic Network 的块层上的每几个块，一个（充分绑定的）验证器将在验证块层上的所有块并创建块的 Merkle 树后在主链上创建一个检查点自上一个检查点以来的哈希值。

除了在主链上提供最终确定性之外，检查点还在提款中发挥作用，因为它们包含在用户提款时的代币燃烧证明（提款）。它使用户能够使用 Patricia Merkle 证明和头块证明在根合约上证明他们剩余的代币。请注意，为了证明剩余的代币，头​​块必须通过 PoS（利益相关者）提交到根链。提款过程将像往常一样产生以太坊 GAS。我们在退出游戏中大量利用检查站。
## 类 UTXO 事件日志
对于 ERC20/ERC721 传输，这是通过使用类似 UTXO 的事件日志数据结构来实现的。下面是一个 LogTransfer 事件供参考。

	event LogTransfer(
	    address indexed token,
	    address indexed from,
	    address indexed to,
	    uint256 amountOrTokenId,
	    uint256 input1, // 发件人以前的帐户余额
	    uint256 input2, // 收款人以前的账户余额
	    uint256 output1, // 发件人的新帐户余额
	    uint256 output2 // 接收方的新帐户余额
	);
因此，基本上每个 ERC20/ERC721 传输都会发出此事件，

- 发送方和接收方的先前余额（input1和input2）成为 tx 的输入（类似 UTXO）
- 新余额成为输出（output1和output2）。

通过整理所有相关 LogTransfer 事件来跟踪转移。

### 退出游戏
由于区块是由单个区块生产者（或极少数）生产的，因此它暴露了欺诈的表面。我们将简要讨论攻击场景，然后讨论 Plasma 保证如何保护用户。
#### 攻击向量
- A. 恶意运营商(矿工)

	下面讨论运营商可能变得恶意并试图作弊的场景。

	- 无处不在的代币/双花/畸形收据

		欺诈性地增加（对于运营商控制的账户）/减少（对于用户）代币余额。
	- 数据不可用

		用户发送 tx 后，假设运营商将 tx 包含在了 Plasma 块中，但使链数据对用户不可用。在这种情况下，如果用户从较旧的 tx 开始退出，那么他们可以通过展示他们最近的 tx 在链上受到挑战。很容易伤害用户。
	- 错误的检查点

		在最坏的情况下，操作员可以执行 A.1 和（或）A.2 并与验证者勾结，将那些无效的状态转换提交给根链。
	- 停止侧链

		运营商停止生产区块，侧链停止。如果检查点在指定的时间内没有提交，则可以将侧链标记为在根链上停止。之后就不能再提交检查点了。

	由于上述或其他原因，如果等离子链变得流氓，用户需要开始大规模退出，我们渴望在根链上提供用户可以利用的退出结构，如果时机成熟。
- B. 恶意用户
	- 用户开始退出已提交的交易，但继续在侧链上花费代币。类似于双花，但跨 2 条链。

#### 解决问题
我们建立在[MoreVp 的理念之上](https://ethresear.ch/t/more-viable-plasma/2160) .简而言之，MoreVP 引入了一种计算退出优先级的新方法，称为“最小输入”优先级。moreVP 不是按输出的年龄排序退出，而是按最年轻输入的年龄排序退出。这具有输出退出的效果，即使它们在“不知从何而来”交易之后被包含在保留的块中，只要它们仅来自有效输入，它们就会被正确处理。我们定义 `getAge` 哪个将年龄分配给包含的 tx。这是在[最小存活 plasma](https://ethresear.ch/t/minimal-viable-plasma/426)中定义的 

	function getAge(receipt) {
	  const { headerNumber, plasmaBlockNum, txindex, oindex } = receipt
	  return f(headerNumber, plasmaBlockNum, txindex, oindex) //乘以它们各自的权重
	}
在我们继续讨论退出场景之前介绍一些术语。

- 术语
	- 提款人：想要退出 Plasma 链的用户。
	- Committed tx：已包含在 matic 链块中并已在根链上设置检查点的 tx。
	- Spend tx：响应用户签名的操作（不包括传入的代币转移）更改用户代币余额的交易。这可能是用户发起的转账、销毁 tx 等
	- 参考 tx：在该特定用户和令牌的退出 tx 之前的 Txs。正如我们在基于账户余额的 UTXO 方案中所定义的那样，参考 tx 的输出成为要退出的 tx 的输入。
	- MoreVP 退出优先级：特定 tx 的最年轻输入（在参考 txs 中）的年龄。它最常用于计算退出优先级。
- 退出场景
	- A. 销毁代币

		要退出侧链，用户将在 Plasma 链上发起一次取款，也就是销毁代币 tx。此 tx 将发出一个 Withdraw 事件。

			event Withdraw(
			    address indexed token,
			    address indexed from,
			    uint256 amountOrTokenId,
			    uint256 input1,
			    uint256 output1
			);
		这里 input1 表示用户之前对相关代币的余额，并 output1 表示侧链上剩余的代币数量。这种结构与我们基于帐户的 UTXO 方案是一致的。用户将出示此提款交易的收据以提取主链上的代币。在引用此收据时，用户还必须提供以下内容：

		- 侧链区块中包含收据的默克尔证明 ( receiptsRoot)
		- 交易包含在侧链区块中的 Merkle 证明 ( transactionsRoot)
		- 证明在根链上的检查点中包含侧链块头

				startExit(withdrawTx, proofOfInclusion /* 在检查点的取款机 */) {
				  Verify inclusion of withdrawTx in checkpoint using proofOfInclusion
				  Verify msg.sender == ecrecover(withdrawTx)
				
				  uint age = getAge(withdrawTx)
				  // 将exit添加到优先级Q
				  PlasmaExit exit = ({owner, age, amount, token})
				  addExitToQueue(exit)
				}
		每当用户希望退出 Plasma 链时，他们（或被他们的客户端应用程序（即钱包）抽象出来）应该在侧链上烧掉代币，等待它获得检查点，然后从检查点提取 tx 中开始退出。

		挑战期

		- 用户在侧链上烧掉了所有的代币，没有再进行任何交易，那么就没有什么可挑战的了，退出将看到它的适当时机。
		- 用户可以选择部分烧掉他们的余额，或者可能已经收到了他们可能在侧链上花费的代币。只要运营商不是恶意的，EVM 就可以保证用户在侧链上花费的余额不会超过剩余的余额。
		- 如果运营商自己创建了格式错误的 txs 并试图花费比可用代币更多的代币，唯一的选择是开始大规模退出，并且使用 MoreVP 构造退出将节省用户。有关详细信息，请参阅接下来的 2 个场景。
	- B. 退出最后一次 ERC20/721 转账（MoreVP）

		考虑这个场景，用户在侧链上进行了 ERC20 转账。操作员在用户转移之前添加了一个无处不在的 tx，并与验证者勾结以检查该块。在这种情况下，更一般地说，在上面讨论的攻击向量 A1 到 A3 中，用户可能没有机会在包含恶意 tx 之前烧掉他们的代币，因此需要从最后一个检查点 tx 开始退出根链——出于这个原因，除了销毁退出之外，我们还需要支持各种 tx 的退出，例如 ERC20/721 转账等。基于此攻击向量并分解两种情况：

		- 传出转账

			我向用户转账了一些代币，但是我注意到运营商在包含我的转账 tx 之前在块/检查点中包含了恶意 tx。我需要开始退出链条。我将开始从传输交易中退出。正如 MoreVP 中所定义的，我需要提供一个参考 tx（输入 UTXO）来定义出口的优先级。因此，我将引用一个更新了我的代币余额的 tx，它就在传出传输 tx 之前。

				startExit(referenceTx, proofOfInclusion, exitTx) {
				  Verify inclusion of referenceTx in checkpoint using proofOfInclusion
				  Verify token balance for the user after the input tx was executed >= tokens being transferred in the exitTx
				  Verify msg.sender == ecrecover(exitTx)
				
				  uint age = getAge(referenceTx)
				  // add exit to priority Q
				  PlasmaExit exit = ({owner, age, amount, token})
				  addExitToQueue(exit)
				}
		- 传入传输

			我注意到运营商在包含我的传入传输 tx 之前在块/检查点中包含了一个恶意 tx。我将在引用交易对手的余额时从传入的传输 tx 开始退出 - 因为这里输入的 UTXO是交易对手的代币余额。

				startExit(referenceTx, proofOfInclusion, exitTx) {
				  Verify inclusion of referenceTx in checkpoint using proofOfInclusion
				  Verify token balance for the counterparty after the input tx was executed >= tokens being transferred in the exitTx
				  Verify input.sender == ecrecover(exitTx) && input.receiver == msg.sender
				
				  uint age = getAge(referenceTx)
				  // add exit to priority Q
				  PlasmaExit exit = ({owner, age, amount, token})
				  addExitToQueue(exit)
				}
		- 挑战期

			如果用户开始从特定状态退出但继续在侧链上花费代币，他们将受到挑战。为了挑战，挑战者将提供用户在参考 tx 之后按时间顺序显示的任何 tx。
	- C. 退出进行中的交易（MoreVP）

		这个场景是为了对抗数据不可用场景。假设我做了一笔交易，但由于数据不可用，我不知道该交易是否已包含在内。我可以通过引用最后一个检查点的 tx 来从这个飞行中的 tx 中退出。用户在开始 MoreVP 风格的退出时应注意不要进行任何 tx，否则将受到挑战。
	- 注释

		当退出 moreVP 风格的构造时，用户可以通过提供参考 txs、exit tx 并放置一个小的 exit bond. 对于任何退出，
		
		- 如果退出挑战成功，退出将被取消，退出保证金将被没收。

## 限制
- 大证明：包含交易的 Merkle 证明和包含在检查点中的块（包含该交易）的 merkle 证明。
- 批量退出：如果运营商出现恶意，用户需要开始批量退出。

该规范处于初期阶段，如果此结构被彻底破坏，我们将不胜感激任何有助于我们改进或重新设计的反馈。我们的[合约](https://github.com/maticnetwork/contracts)正在进行中的实施工作 46存储库。

## 参考
- [account-based-plasma-morevp](https://ethresear.ch/t/account-based-plasma-morevp/5480)
- [more-viable-plasma](https://ethresear.ch/t/more-viable-plasma/2160)