### DAO 治理参考
新部署的治理合约 Bravo 可通过 [Etherscan](https://etherscan.io/address/0x408ED6354d4973f66138C91495F2f2FCbd8724C3) 获得更新参考，以下一些参考材料可能已过时。

Uniswap 协议由 UNI Token持有者管理和升级，使用三个不同的组件；

- UNI Token
- 治理模块
- 时间锁

这些合约共同允许社区

- 提出
	- 任何拥有超过 250 万个 UNI 的地址都可以提出治理行动，其中包含已完成的可执行代码。
	- 创建提案后，社区可以在 3 天的投票期内投票。
- 投票
	- 如果该提案获得多数票且至少有 400 万票就可以通过 
- 实施
	- 则该提案将在 Timelock 中排队，并可能在至少 2 天内执行。 	
对 uniswap 协议的更改。

#### Timelock
Timelock 合约可以“延时、退出”升级模式修改系统参数、逻辑和合约。Timelock 的硬编码最短延迟时间为 2 天，这是治理操作的最短通知时间。每项提议的行动将在公告之日起至少 2 天后公布。重大升级（例如更改风险系统）可能会有长达 30 天的延迟。Timelock 由治理模块控制；可以在 Timelock Dashboard 上监控未决和已完成的治理操作。

![](./pic/v2protocol7.png)

#### 关键事件
- 委托变更

	当帐户更改其委托时发出

		DelegateChanged(address indexed delegator, address indexed fromDelegate, address indexed toDelegate)
- 代表投票已更改

	当委托账户的投票余额发生变化时发出
	
		DelegateVotesChanged(address indexed delegate, uint previousBalance, uint newBalance)
- 提案已创建

	创建新提案时发出
	
		ProposalCreated(uint id, address proposer, address[] targets, uint[] values, string[] signatures, bytes[] calldatas, uint startBlock, uint endBlock, string description)
- 投票

	当对提案进行投票时发出。
	
		VoteCast(address voter, uint proposalId, bool support, uint votes)
- 提案取消

	当提案被取消时发出
	
		ProposalCanceled(uint id)	
- 提案排队

	当提案已在 Timelock 中排队时发出
	
		ProposalQueued(uint id, uint eta)	
- 提案执行

	在 Timelock 中执行提案时发出
	
		ProposalExecuted(uint id)	

#### 只读函数：UNI
- Get Current Votes

	返回当前块的帐户的投票余额。
		
		function getCurrentVotes(address account) returns (uint96)
	名称|类型|描述
	---|---|---
	account|`address`|要检索票数的帐户地址
- Get Prior Votes

	返回特定区块号的帐户的先前投票数。传递的块号必须是最终块，否则函数将恢复。
	
		function getPriorVotes(address account, uint blockNumber) returns (uint96)
	名称|类型|描述
	---|---|---
	account|`address `|要检索先前投票数的帐户的地址
	blocknumber|`uint `|检索先前投票数的块号。
	unnamed|`uint96 `|先前投票数

#### 状态改变函数：UNI
- Delegate

	将发件人的投票委托给受托人。用户一次可以委托1个地址，被委托人的投票数加到的票数相当于用户账户中UNI的余额。投票从当前区块开始委托，直到发送者再次委托或转移他们的 UNI。
	
		function delegate(address delegatee)	
	名称|类型|描述
	---|---|---
	delegatee|`address `|msg.sender 希望将投票委托给的地址。
- Delegate By Signature

	将发件人的投票委托给受托人。用户一次可以委托1个地址，被委托人的投票数加到的票数相当于用户账户中UNI的余额。投票从当前区块开始委托，直到发送者再次委托或转移他们的 UNI。
	
		function delegateBySig(address delegatee, uint nonce, uint expiry, uint8 v, bytes32 r, bytes32 s)
	名称|类型|描述
	---|---|---
	delegatee|`address `|msg.sender 希望将投票委托给的地址
	nonce|`uint `|与签名匹配所需的合同状态。这可以从合约的公共随机数映射中检索到
	expiry|`uint `|签名到期的时间。自 unix 纪元以来的区块时间戳（以秒为单位）。
	v|`uint `|签名的恢复字节。
	r|`bytes32 `|ECDSA 签名对的一半。
	s|`bytes32 `|	ECDSA 签名对的一半。

#### 只读函数：Governor Alpha
- Quorum Votes

	返回提案成功所需的最小投票数。
	
		function quorumVotes() public pure returns (uint)
- Proposal Threshold

	返回帐户创建提案所需的最小投票数
	
		function proposalThreshold() returns (uint)	
- Proposal Max Operations

	返回提案中可以包含的最大操作数。操作是在提案成功并执行时将进行的函数调用。
	
		function proposalMaxOperations() returns (uint)
- Voting Delay

	返回开始对提案进行投票之前要等待的块数。创建提案时，此值将添加到当前区块编号。
	
		function votingDelay() returns (uint)
- Voting Period

	返回对提案进行投票的持续时间，以块为单位。	
	
		function votingPeriod() returns (uint)
- Get Actions

	获取选定提案的操作。传递一个提案 ID 并获取该提案的目标、值、签名和调用数据。
	
		function getActions(uint proposalId) returns (uint proposalId) public view returns (address[] memory targets, uint[] memory values, string[] memory signatures, bytes[] memory calldatas)
	名称|类型|描述
	---|---|---
	proposalId|`uint `|ID of the proposal
	
	- 返回
		- 提案调用的合约地址数组
		- 提案用作值的无符号整数数组
		- 提案签名的字符串数组
		- 提案的 calldata 字节数组
- Get Receipt

	返回给定选民的提案选票收据。
	
		function getReceipt(uint proposalId, address voter) returns (Receipt memory)
	名称|类型|描述
	---|---|---
	proposalId|`uint `|获取选民选票收据的提案的 ID。
	voter|`address `|提案投票人帐户的地址。
	Receipt|	`struct `|选民地址选票的收据结构。
- State

	返回 ProposalState 类型的枚举，
	
		function state(uint proposalId) returns (ProposalState)
	可能的类型有： 
	
	- -Pending 
	- -Active 
	- -Canceled 
	- -Defeated 
	- -Succeeded 
	- -Queued 
	- -Expired 
	- -andExecuted

#### 状态改变函数：Governor Alpha
- Propose

	创建一个更改协议的提案。提案将由受委托的选民投票表决。如果在投票期结束前获得足够支持，该提案将自动通过。已制定的提案在 Timelock 合约中排队并执行。

	发送者必须持有比前一个块的当前提议阈值 (`proposalThreshold()`) 更多的 UNI。基于 `proposalMaxOperations()` 提案最多可以有 10 个操作）。

	如果提议者当前有待处理或活动的提议，则提议者无法创建另一个提议。不可能在同一个块中排队两个相同的动作（由于时间锁的限制），因此单个提案中的动作必须是唯一的，并且共享相同动作的唯一提案必须在不同的块中排队。
	
		function propose(address[] memory targets, uint[] memory values, string[] memory signatures, bytes[] memory calldatas, string memory description) returns (uint)
	名称|类型|描述
	---|---|---
	targets|`address `|在提案执行期间要进行调用的目标地址的有序列表。此数组必须与此函数中的所有其他数组参数长度相同。
	values|`uint `|要传递给提案执行期间进行的调用的有序值列表（即 msg.value）。此数组必须与此函数中的所有其他数组参数长度相同
	signatures|`string `|要在执行期间传递的函数签名的有序列表。此数组必须与此函数中的所有其他数组参数长度相同。
	calldatas|`bytes `|在提案执行期间要传递给每个单独的函数调用的有序数据列表。此数组必须与此函数中的所有其他数组参数长度相同。
	description|`string `|对提案及其将进行的更改的人类可读描述
	Unnamed|`uint `|返回新提案的 ID
- Queue

	提案成功后，任何地址都可以调用 queue 方法将提案移入 Timelock 队列。只有成功的提案才能排队。
	
		function queue(uint proposalId)
	名称|类型|描述
	---|---|---
	proposalId|`uint `|给定成功提案的 ID
- Execute

	在 Timelock 延迟期之后，任何账户都可以调用 execute 方法将提案中的更改应用到目标合约。这将调用提案中描述的每个操作。此功能是应付的，因此 Timelock 合约可以调用在提案中选择的应付功能。
	
		function execute(uint proposalId) payable
	名称|类型|描述
	---|---|---
	proposalId|`uint `|给定成功提案的 ID
- Cancel

	取消尚未执行的提案。监护人是唯一可以执行此操作的人，除非提案人不维护创建提案所需的代表。如果提议者没有超过提议阈值的代表，任何人都可以取消该提议。

		function cancel(uint proposalId)
	名称|类型|描述
	---|---|---
	proposalId|`uint `|给定成功提案的 ID
- Cast Vote

	对提案进行投票。帐户的投票权重由提案生效时的委托投票数决定。
	
		function castVote(uint proposalId, bool support)
	名称|类型|描述
	---|---|---
	proposalId|`uint `|给定成功提案的 ID
	support|`bool`|在提案投票中，布尔值 true 表示“是”或 false 表示“否”
- Cast Vote By Signature

	对提案进行投票。帐户的投票权重由提案生效时的委托投票数决定。这种方式与 Cast Vote 的目的相同，只是让离线签名能够参与治理投票。有关如何创建离线签名的更多详细信息，请查看 EIP-712。
	
		function castVoteBySig(uint proposalId, bool support, uint8 v, bytes32 r, bytes32 s)
	名称|类型|描述
	---|---|---
	proposalId|`uint `|给定成功提案的 ID
	support|`bool`|在提案投票中，布尔值 true 表示“是”或 false 表示“否”
	expiry|`uint `|签名到期的时间。自 unix 纪元以来的区块时间戳（以秒为单位）。
	v|`uint `|签名的恢复字节。
	r|`bytes32 `|ECDSA 签名对的一半。
	s|`bytes32 `|	ECDSA 签名对的一半。