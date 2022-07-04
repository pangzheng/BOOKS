# Openzepplin API-ERC 20
## ERC 20
这套接口、合约和实用程序都与[ERC20 通证标准](https://eips.ethereum.org/EIPS/eip-20)相关。有关 ERC20 Token的概述以及如何创建Token合约的演练，请阅读 [ERC20 指南](https://docs.openzeppelin.com/contracts/4.x/erc20)。

有一些核心合约实现了 EIP 中指定的行为：

- [IERC20](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#IERC20)

	所有 ERC20 实现都应该遵循的接口。
- [IERC20Metadata](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#IERC20Metadata):

	扩展的 ERC20 接口，包括
	
	- [name](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#ERC20-name)
	- [symbol](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#ERC20-symbol)
	- 和 [decimals](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#ERC20-decimals) 功能。
- [ERC20](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#ERC20)

	ERC20 接口的实现，包括基本接口的 name,symbol 和 decimals 可选标准扩展。

此外，还有多个自定义扩展，包括：

- [ERC20Burnable](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#ERC20Burnable)

	销毁自己的Token。
- [ERC20Capped](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#ERC20Capped)

	在铸造Token时对总供应量实施上限。
- [ERC20Pausable](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#ERC20Pausable)

	暂停Token传输的能力。
- [ERC20Snapshot](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#ERC20Snapshot)

	有效存储过去的Token余额，以便以后随时查询。
- [ERC20Permit](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#ERC20Permit)

	Token的无气体授权（标准化为 ERC2612）。
- [ERC20FlashMint](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#ERC20FlashMint)

	通过铸造和销毁临时Token（标准化为 ERC3156）对闪电贷款的Token级支持。
- [ERC20Votes](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#ERC20Votes)

	支持投票和投票委托。
- [ERC20VotesComp](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#ERC20VotesComp)

	支持投票和投票委托（兼容 Compound 的 token，有 uint96 限制）。
- [ERC20Wrapper](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#ERC20Wrapper)

	用于创建由另一个 ERC20 支持的 ERC20 的包装器，具有存款和取款方法。与 [ERC20Votes](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#ERC20Votes) 结合使用很有用。

最后，还有一些实用程序可以以各种方式与 ERC20 合约进行交互。

- [SafeERC20](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#SafeERC20)

	接口的包装器，无需处理布尔返回值。
- [TokenTimelock](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#TokenTimelock)

	为受益人持有Token直到指定时间。

还处于 EIP 草案状态的功能。

- [ERC20Permit](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#ERC20Permit)

这套核心合约被设计为无主见，允许开发人员访问 ERC20 中的内部函数（例如 [_mint](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#ERC20-_mint-address-uint256-)），并以他们喜欢的方式将它们公开为外部函数。另一方面，[ERC20 预设](https://docs.openzeppelin.com/contracts/4.x/erc20#Presets)（例如[ERC20PresetMinterPauser](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#token/ERC20/presets.md#ERC20PresetMinterPauser)）是使用固有的模式设计的，为开发人员提供即用型、可部署的合约。

### 核心
#### IERC20 [github](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/v4.6.0/contracts/token/ERC20/IERC20.sol)
	import "@openzeppelin/contracts/token/ERC20/IERC20.sol";
EIP 中定义的 ERC20 标准的接口。

- 功能
	- [totalSupply()](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#IERC20-totalSupply--)

		外部方法，返回存在的Token数量。
		
		- 返回值类型 unit256
	- [balanceOf(account)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#IERC20-balanceOf-address-)

		外部方法，返回 account 拥有的Token数量。
		
		- 返回值类型 unit256
	- [transfer(to, amount)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#IERC20-transfer-address-uint256-)

		外部方法，将  `amount`  Token从调用方的帐户移至 `to` .返回一个布尔值，指示操作是否成功。

		发出 [Transfer](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#IERC20-Transfer-address-address-uint256-)事件。
		
		- 返回值类型 bool
	- [allowance(owner, spender)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#IERC20-allowance-address-address-)

		
		
		外部方法，通过 [transferFrom](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#IERC20-transferFrom-address-address-uint256-)返回 `spender` 将被允许代表 `own` 消费的 token 剩余数量。默认情况下为零。

		当[approve](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#IERC20-approve-address-uint256-)或 [transferFrom](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#IERC20-transferFrom-address-address-uint256-)被调用时，这个值会改变。
		
		- 返回值类型 unit256
	- [approve(spender, amount)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#IERC20-approve-address-uint256-)

		外部方法，设置 `amount` 为 `spender` 调用方Token的余量。返回一个布尔值，指示操作是否成功。

		请注意，使用这种方法更改配额会带来风险，即有人可能通过堵塞的交易排序同时使用旧配额和新配额。缓解这种竞争条件的一种可能解决方案是首先将支出者的津贴减少到 0，然后设置所需的值： 
	
			https://github.com/ethereum/EIPs/issues/20#issuecomment-263524729
		发出一个 [Approval](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#IERC20-Approval-address-address-uint256-) 事件。
		
		- 返回值类型 bool
	- [transferFrom(from, to, amount)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#IERC20-transferFrom-address-address-uint256-)

		使用限额(`allowance`) 机制将 `amount ` Token从(`from`)一个移动到 (`to`) 另一个。 `amount ` 将从调用方的限额(`allowance`)中扣除。返回一个布尔值，指示操作是否成功。

		发出 [Transfer](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#IERC20-Transfer-address-address-uint256-)事件。
		
		- 返回值类型 bool
- 事件
	- [Transfer(from, to, value)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#IERC20-Transfer-address-address-uint256-)

		当  `value` Token从(`from`)一个帐户移动到 (`to`) 另一个帐户时发出。

		请注意，`value` 可能为零。
	- [Approval(owner, spender, value)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#IERC20-Approval-address-address-uint256-)


		当调用一个 `spender` 为一个 `owner` 设置允许时发出 [approve](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#IERC20-approve-address-uint256-)。value 是新的限额(`allowance`)

#### IERC20Metadata [github](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/v4.6.0/contracts/token/ERC20/extensions/IERC20Metadata.sol)
	import "@openzeppelin/contracts/token/ERC20/extensions/IERC20Metadata.sol";
来自 ERC20 标准的可选元数据功能的接口。

从 v4.1 开始可用。

- 功能
	- [name()](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#IERC20Metadata-name--)
		
		外部方法，返回Token的名称。
		
		- 返回值类型 string 
	- [symbol()](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#IERC20Metadata-symbol--)

		外部方法，返回Token的符号。
		
		- 返回值类型 string 
	- [decimals()](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#IERC20Metadata-decimals--)

		外部方法，返回Token的小数位。
		
		- 返回值类型 string 
	- IERC20功能相同
		- [totalSupply()](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#IERC20-totalSupply--)
		- [balanceOf(account)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#IERC20-balanceOf-address-)
		- [transfer(to, amount)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#IERC20-transfer-address-uint256-)
		- [allowance(owner, spender)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#IERC20-allowance-address-address-)
		- [approve(spender, amount)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#IERC20-approve-address-uint256-)
		- [transferFrom(from, to, amount)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#IERC20-transferFrom-address-address-uint256-)
	- 事件
		- IERC20功能相同
			- [Transfer(from, to, value)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#IERC20-Transfer-address-address-uint256-)
			- [Approval(owner, spender, value)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#IERC20-Approval-address-address-uint256-)

#### ERC20 [github](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/v4.6.0/contracts/token/ERC20/ERC20.sol)
	import "@openzeppelin/contracts/token/ERC20/ERC20.sol";
接口的实现 [IERC20](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#IERC20)。

此实现与创建Token的方式无关。这意味着必须在派生合约中添加供应机制，使用 [_mint](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#ERC20-_mint-address-uint256-). 有关通用机制，请参阅 [ERC20PresetMinterPauser](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#token/ERC20/presets.md#ERC20PresetMinterPauser)。

有关详细的文章，请参阅我们的指南 [如何实施供应机制](https://forum.zeppelin.solutions/t/how-to-implement-erc20-supply-mechanisms/226)。

我们遵循了一般的 OpenZeppelin Contracts 指南：函数 `false` 在失败时恢复而不是返回。这种行为仍然是传统的，并且与 ERC20 应用程序的期望不冲突。

此外，[Approval](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#IERC20-Approval-address-address-uint256-) 在调用 [transferFrom](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#ERC20-transferFrom-address-address-uint256-). 这允许应用程序仅通过收听所述事件来重建所有帐户的限额。EIP 的其他实现可能不会发出这些事件，因为规范没有要求。

最后，添加了非标准 [decreaseAllowance](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#ERC20-decreaseAllowance-address-uint256-) 和 [increaseAllowance ](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#ERC20-increaseAllowance-address-uint256-) 功能，以缓解围绕设置 限额(`allowance`) 的众所周知的问题。见[IERC20.approve](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#IERC20-approve-address-uint256-)。

- 功能
	- [constructor(name_, symbol_)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#ERC20-constructor-string-string-)

		公共方法，[name](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#ERC20-name--) 设置和的值 [symbol](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#ERC20-symbol--)。

		默认值为 [decimals](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#ERC20-decimals--) 18。要为 [decimals](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#ERC20-decimals--) 选择不同的值，应重载它。

		所有这两个值都是不可变的：它们只能在构造过程中设置一次。
	- [name()](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#ERC20-name--)

		公共方法，返回Token的名称。
		
		- 返回值类型 string
	- [symbol()](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#ERC20-symbol--)

		公共方法，返回Token的符号，通常是名称的较短版本。
		
		- 返回值类型 string
	- [decimals()](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#ERC20-decimals--)

		公共方法，返回用于获取其用户表示的小数位数。例如，如果 `decimals` 等于2 ，用户显示 505 的代余额。

			5.05 (505 / 10 ** 2).
		Token通常选择 18 的值，模仿 Ether 和 Wei 之间的关系。这是 [ERC20](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#ERC20) 使用的值，除非此函数被覆盖；
		
		此信息仅用于显示目的：它绝不会影响合约的任何计算，包括 [IERC20.balanceOf](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#IERC20-balanceOf-address-) 和 [IERC20.Transfer](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#IERC20-Transfer-address-address-uint256-)。
		
		- 返回值类型 uint8
	- [totalSupply()](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#ERC20-totalSupply--)

		公共方法，见 [IERC20.totalSupply](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#IERC20-totalSupply--)。
		
		- 返回值类型 uint256
	- [balanceOf(account)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#ERC20-balanceOf-address-)

		公共方法，见 [IERC20.balanceOf](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#IERC20-balanceOf-address-)
		
		- 返回值类型 uint256
	- [transfer(to, amount)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#ERC20-transfer-address-uint256-)

		公共方法，见 [IERC20.Transfer](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#IERC20-Transfer-address-address-uint256-)。

		- 要求：
			- `to` 不能是零地址。
			- 调用方的余额必须至少为`amount`
		- 返回值类型 bool
	- [allowance(owner, spender)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#ERC20-allowance-address-address-)

		 公共方法，见 [IERC20.allowance](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#IERC20-allowance-address-address-)。
		 
		 - 返回值类型 bool
	- [approve(spender, amount)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#ERC20-approve-address-uint256-)

		公共方法，见 [IERC20.approve](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#IERC20-approve-address-uint256-)。

		如果 `amount` 是最大的 `uint256`，则限额(`allowance`)  `transferFrom` 不更新 。这在语义上等同于无限授权。
		
		- 要求
			- `spender` 不能是零地址
		- 返回值类型 bool
	- [transferFrom(from, to, amount)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#ERC20-transferFrom-address-address-uint256-)

		公共方法，见 [IERC20.transferFrom](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#IERC20-transferFrom-address-address-uint256-)。

		发出一个 [Approval](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#IERC20-Approval-address-address-uint256-) 指示更新的容差的事件。EIP 不需要这样做。见开头的注释 [ERC20](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#ERC20)。
		
		如果当前限额为最大值，则不更新限额(`allowance`)  `uint256`。
		
		- 要求：
			- `from` 和 `to`不能是零地址。
			- `from` 必须有余额 `amount`。
			- 调用方 `from` 必须从Token扣除 `amount`.
		- 返回值类型 bool
	- [increaseAllowance(spender, addedValue)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#ERC20-increaseAllowance-address-uint256-)

		公共方法，`spender` 原子性增加调用方授予的限额(`allowance`) 。这是一种替代[approve](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#ERC20-approve-address-uint256-)方法 ，可用于缓解 [IERC20.approve](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#IERC20-approve-address-uint256-)中描述的问题。

		发出一个[Approval](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#IERC20-Approval-address-address-uint256-)指示更新限额(`allowance`) 事件。

		- 要求
			- `spender` 不能是零地址。
		- 返回值类型 bool
	- [decreaseAllowance(spender, subtractedValue)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#ERC20-decreaseAllowance-address-uint256-)

		公共方法，`spender` 原子性减少调用方授予的限额(`allowance`) 。这是一种替代[approve](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#ERC20-approve-address-uint256-)方法 ，可用于缓解 [IERC20.approve](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#IERC20-approve-address-uint256-)中描述的问题。

		发出一个[Approval](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#IERC20-Approval-address-address-uint256-)指示更新限额(`allowance`) 事件。

		- 要求
			- `spender` 不能是零地址。
		- 返回值类型 bool
	- [_transfer(from, to, amount)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#ERC20-_transfer-address-address-uint256-)

		内部方法，标记 `amount`  从 `from` 到`to`的移动。这个内部函数相当于 [transfer](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#ERC20-transfer-address-uint256-)，可用于例如实现自动Token费用、slashing 机制等。

		发出 [transfer](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#ERC20-transfer-address-uint256-)事件。

		- 要求
			- `from` 不能是零地址。
			- `to` 不能是零地址。
			- `from` 必须至少有`amount` 余额。
	- [_mint(account, amount)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#ERC20-_mint-address-uint256-)

		内部方法，创建 `amount` Token并将它们分配给 `account`，增加总供应量。

		发出一个设置 `from` 为零地址的 [transfer](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#ERC20-transfer-address-uint256-)事件。

		- 要求
			- `account` 不能是零地址。
	- [_burn(account, amount)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#ERC20-_burn-address-uint256-)

		内部方法，从 `account` 销毁 `amount` Token，减少总供应量。发出一个 `to` 设置为零地址的 `transfer` 事件。

		- 要求
			- `account` 不能是零地址。
			- `account` 至少有 `amount` Token
	- [_approve(owner, spender, amount)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#ERC20-_approve-address-address-uint256-)

		内部方法， `spender`  设置 `amount` 为 owners Token的消费限额。此内部函数等效于 `approve`，可用于例如为某些子系统设置自动容限等。

		发出一个 [Approval](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#IERC20-Approval-address-address-uint256-)事件。

		- 要求
			- `owner` 不能是零地址。
			- `spender` 不能是零地址。
	- [_spendAllowance(owner, spender, amount)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#ERC20-_spendAllowance-address-address-uint256-)

		内部方法，根据`spender` `amount` 更新 `owner` 限额。

		在无限限额的情况下不更新限额金额。如果没有足够的余量则恢复。

		可能会发出一个 [Approval](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#IERC20-Approval-address-address-uint256-) 事件
	- [_beforeTokenTransfer(from, to, amount)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#ERC20-_beforeTokenTransfer-address-address-uint256-)

		内部方法，在任何Token转移之前调用的挂钩。这包括铸造和销毁。

		调用条件
		
		- 当 `from` 和 `to` 都非零时，`amount` 的 `from` Token将转移到 `to`。
		- 当 `from` 为零时，`amount` 将为`to`铸造Token 。
		- 当`to`为零时， `from` 持有 `amoun` 的Token将被销毁。
		- `from` 和 `to` 永远都不为零。

		要了解有关钩子的更多信息，请前往[使用钩子](https://docs.openzeppelin.com/contracts/4.x/extending-contracts#using-hooks)。
	- [_afterTokenTransfer(from, to, amount)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#ERC20-_afterTokenTransfer-address-address-uint256-)

		内部方法，在任何Token转移后调用的挂钩。这包括铸造和销毁。
		
		调用条件：
		
		- 当 `from` 和 `to` 都非零时，`amount` 的 `from` Token将转移到 `to`。
		- 当 `from` 为零时，`amount` 将为`to`铸造Token 。
		- 当`to`为零时， `from` 持有 `amoun` 的Token将被销毁。
		- `from` 和 `to` 永远都不为零。
		
		要了解有关钩子的更多信息，请前往[使用钩子](https://docs.openzeppelin.com/contracts/4.x/extending-contracts#using-hooks)
- 事件
	- IERC20功能相同
		- [Transfer(from, to, value)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#IERC20-Transfer-address-address-uint256-)
		- [Approval(owner, spender, value)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#IERC20-Approval-address-address-uint256-)

### 扩展
#### ERC20Burnable [github](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/v4.6.0/contracts/token/ERC20/extensions/ERC20Burnable.sol)
	import "@openzeppelin/contracts/token/ERC20/extensions/ERC20Burnable.sol";
对此的扩展 [ERC20](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#ERC20) 允许Token持有者以一种可以在链下识别的方式（通过事件分析）销毁他们自己的Token和他们授权的Token 

- 功能
	- [burn(amount)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#ERC20Burnable-burn-uint256-)

		公共方法，销毁amount调用方的Token。

		见 [ERC20._burn](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#ERC20-_burn-address-uint256-)。
	- [burnFrom(account, amount)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#ERC20Burnable-burnFrom-address-uint256-)

		公共f昂发，从 `account` 中销毁 `amount` Token，从调用方的限额中扣除。

		见 [ERC20._burn](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#ERC20-_burn-address-uint256-) 和 [ERC20.allowance](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#ERC20-allowance-address-address-)。

		- 要求
			- 调用方 `accounts` 必须有 `amount` 数量Token.
	- ERC20功能相同
		- [constructor(name_, symbol_)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#ERC20-constructor-string-string-)
		- [name()](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#ERC20-name--)
		- [symbol()](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#ERC20-symbol--)
		- [decimals()](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#ERC20-decimals--)
		- [totalSupply()](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#ERC20-totalSupply--)
		- [balanceOf(account)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#ERC20-balanceOf-address-)
		- [transfer(to, amount)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#ERC20-transfer-address-uint256-)
		- [allowance(owner, spender)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#ERC20-allowance-address-address-)
		- [approve(spender, amount)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#ERC20-approve-address-uint256-)
		- [transferFrom(from, to, amount)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#ERC20-transferFrom-address-address-uint256-)
		- [increaseAllowance(spender, addedValue)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#ERC20-increaseAllowance-address-uint256-)
		- [decreaseAllowance(spender, subtractedValue)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#ERC20-decreaseAllowance-address-uint256-)
		- [_transfer(from, to, amount)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#ERC20-_transfer-address-address-uint256-)
		- [_mint(account, amount)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#ERC20-_mint-address-uint256-)
		- [_burn(account, amount)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#ERC20-_burn-address-uint256-)
		- [_approve(owner, spender, amount)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#ERC20-_approve-address-address-uint256-)
		- [_spendAllowance(owner, spender, amount)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#ERC20-_spendAllowance-address-address-uint256-)
		- [_beforeTokenTransfer(from, to, amount)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#ERC20-_beforeTokenTransfer-address-address-uint256-)
		- [_afterTokenTransfer(from, to, amount)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#ERC20-_afterTokenTransfer-address-address-uint256-)
- 事件
	- IERC20功能相同
		- [Transfer(from, to, value)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#IERC20-Transfer-address-address-uint256-)
		- [Approval(owner, spender, value)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#IERC20-Approval-address-address-uint256-)

#### ERC20Capped [github](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/v4.6.0/contracts/token/ERC20/extensions/ERC20Capped.sol)
	import "@openzeppelin/contracts/token/ERC20/extensions/ERC20Capped.sol";
对此的扩展 [ERC20](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#ERC20) 增加了Token供应的上限。

- 功能
	- [constructor(cap_)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#ERC20Capped-constructor-uint256-)

		内部方法，设置 `cap` 的值。该值是不可变的，在构建过程中只能设置一次。
	- [cap()](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#ERC20Capped-cap--)

		公共方法，返回Token总供应量的上限。
	- [_mint(account, amount)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#ERC20Capped-_mint-address-uint256-)
	
		内部方法，见 [ERC20._mint](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#ERC20-_mint-address-uint256-)。
	- ERC20功能相同
		- [name()](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#ERC20-name--)
		- [symbol()](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#ERC20-symbol--)
		- [decimals()](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#ERC20-decimals--)
		- [totalSupply()](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#ERC20-totalSupply--)
		- [balanceOf(account)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#ERC20-balanceOf-address-)
		- [transfer(to, amount)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#ERC20-transfer-address-uint256-)
		- [allowance(owner, spender)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#ERC20-allowance-address-address-)
		- [approve(spender, amount)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#ERC20-approve-address-uint256-)
		- [transferFrom(from, to, amount)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#ERC20-transferFrom-address-address-uint256-)
		- [increaseAllowance(spender, addedValue)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#ERC20-increaseAllowance-address-uint256-)
		- [decreaseAllowance(spender, subtractedValue)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#ERC20-decreaseAllowance-address-uint256-)
		- [_transfer(from, to, amount)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#ERC20-_transfer-address-address-uint256-)
		- [_burn(account, amount)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#ERC20-_burn-address-uint256-)
		- [_approve(owner, spender, amount)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#ERC20-_approve-address-address-uint256-)
		- [_spendAllowance(owner, spender, amount)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#ERC20-_spendAllowance-address-address-uint256-)
		- [_beforeTokenTransfer(from, to, amount)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#ERC20-_beforeTokenTransfer-address-address-uint256-)
		- [_afterTokenTransfer(from, to, amount)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#ERC20-_afterTokenTransfer-address-address-uint256-)
- 事件
	- IERC20功能相同
		- [Transfer(from, to, value)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#IERC20-Transfer-address-address-uint256-)
		- [Approval(owner, spender, value)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#IERC20-Approval-address-address-uint256-)
		
#### ERC20Pausable [github](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/v4.6.0/contracts/token/ERC20/extensions/ERC20Pausable.sol)
	import "@openzeppelin/contracts/token/ERC20/extensions/ERC20Pausable.sol";
ERC20 Token具有可暂停的Token转移、铸造和销毁。

对于在评估期结束之前，阻止交易或在出现大错误时冻结所有Token转移的紧急开关等场景很有用。

- 功能
	- [_beforeTokenTransfer(from, to, amount)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#ERC20Pausable-_beforeTokenTransfer-address-address-uint256-)
	
		内部方法，见 [ERC20._beforeTokenTransfer](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#ERC20-_beforeTokenTransfer-address-address-uint256-)

		- 要求
			- 不得暂停合约。
	- PAUSABLE 功能相同
		- [constructor()](https://docs.openzeppelin.com/contracts/4.x/api/security#Pausable-constructor--)
		- [paused()](https://docs.openzeppelin.com/contracts/4.x/api/security#Pausable-paused--)
		- [_pause()](https://docs.openzeppelin.com/contracts/4.x/api/security#Pausable-_pause--)
		- [_unpause()](https://docs.openzeppelin.com/contracts/4.x/api/security#Pausable-_unpause--)
	- ERC20功能相同
		- [name()](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#ERC20-name--)
		- [symbol()](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#ERC20-symbol--)
		- [decimals()](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#ERC20-decimals--)
		- [totalSupply()](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#ERC20-totalSupply--)
		- [balanceOf(account)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#ERC20-balanceOf-address-)
		- [transfer(to, amount)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#ERC20-transfer-address-uint256-)
		- [allowance(owner, spender)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#ERC20-allowance-address-address-)
		- [approve(spender, amount)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#ERC20-approve-address-uint256-)
		- [transferFrom(from, to, amount)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#ERC20-transferFrom-address-address-uint256-)
		- [increaseAllowance(spender, addedValue)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#ERC20-increaseAllowance-address-uint256-)
		- [decreaseAllowance(spender, subtractedValue)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#ERC20-decreaseAllowance-address-uint256-)
		- [_transfer(from, to, amount)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#ERC20-_transfer-address-address-uint256-)
		- [_mint(account, amount)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#ERC20-_mint-address-uint256-)
		- [_burn(account, amount)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#ERC20-_burn-address-uint256-)
		- [_approve(owner, spender, amount)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#ERC20-_approve-address-address-uint256-)
		- [_spendAllowance(owner, spender, amount)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#ERC20-_spendAllowance-address-address-uint256-)
		- [_afterTokenTransfer(from, to, amount)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#ERC20-_afterTokenTransfer-address-address-uint256-)  
	- 事件
		- PAUSABLE 功能相同
			- [Paused(account)](https://docs.openzeppelin.com/contracts/4.x/api/security#Pausable-Paused-address-)
			- [Unpaused(account)](https://docs.openzeppelin.com/contracts/4.x/api/security#Pausable-Unpaused-address-)
		- IERC20功能相同
			- [Transfer(from, to, value)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#IERC20-Transfer-address-address-uint256-)
			- [Approval(owner, spender, value)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#IERC20-Approval-address-address-uint256-)
	#### ERC20Snapshot [github](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/v4.6.0/contracts/token/ERC20/extensions/ERC20Snapshot.sol)
	import "@openzeppelin/contracts/token/ERC20/extensions/ERC20Snapshot.sol";
该合约使用快照机制扩展了 ERC20 Token。创建快照时会记录当时的余额和总供应量以供以后访问。

这可用于安全地创建基于Token余额的机制，例如无需信任的股息或加权投票。在乐观的实现中，可以通过重用来自不同账户的相同余额来执行“双重支付”攻击。通过使用快照来计算股息或投票权，这些攻击不再适用。它还可用于创建高效的 ERC20 分叉机制。

快照由内部 [_snapshot](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#ERC20Snapshot-_snapshot--) 函数创建，该函数将发出 [Snapshot](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#ERC20Snapshot-Snapshot-uint256-)事件并返回快照 ID。要获取快照时的总供应量，请使用 [totalSupplyAt](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#ERC20Snapshot-totalSupplyAt-uint256-) 快照 ID 调用函数。要获取快照时帐户的余额，请使用 [balanceOfAt](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#ERC20Snapshot-balanceOfAt-address-uint256-) 快照 id 和帐户地址调用该函数。

可以通过覆盖 [_getCurrentSnapshotId](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#ERC20Snapshot-_getCurrentSnapshotId--) 方法来自定义快照策略。例如，让它返回 `block.number` 将触发在每个新块开始时创建快照。覆盖此函数时，请注意其结果的单调性。非单调的快照 id 会破坏合约。

使用这种方法为每个块实现快照将产生大量的 gas 成本。对于节省气体的替代方案，请考虑 [ERC20Votes](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#ERC20Votes)。

- Gas 成本

	快照是有效的。快照创建是O(1)。在已创建的快照数量中，从快照中检索余额或总供应量是O(log n) ，尽管特定帐户的 n 通常会小得多，因为后续快照中的相同余额被存储为单个条目。

	由于额外的快照簿记，正常的 ERC20 传输存在恒定的开销。此开销仅对于紧随特定帐户快照之后的第一次传输而言很重要。后续传输将具有正常成本，直到下一个快照，依此类推。
- 功能
	- [_snapshot()](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#ERC20Snapshot-_snapshot--)

		内部方法，创建一个新快照并返回其快照 ID。发出一个 [Snapshot](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#ERC20Snapshot-Snapshot-uint256-) 包含相同 id 的事件。

		[_snapshot](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#ERC20Snapshot-_snapshot--) 是 `internal`，您必须决定如何将其暴露在外部。它的使用可能使用 [AccessControl](https://docs.openzeppelin.com/contracts/4.x/api/access#AccessControl) 仅限于一组帐户，或它可能对公众开放。
		
		
		`_snapshot` 虽然某些信任最小化机制（例如分叉）需要开放的调用方式，但您必须考虑到攻击者可能以两种方式使用它。

		- 首先，它可以用来增加从快照中检索值的成本，尽管它会以对数方式增长，从而使这种攻击在长期内无效。
		- 其次，它可以用于针对特定账户并增加他们的 ERC20 转账成本，方式在上面的 Gas Costs 部分中指定。
		
		我们还没有测量实际数字；如果您对此感兴趣，请与我们联系。
		
		- 值参数 unit256
	- [_getCurrentSnapshotId()](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#ERC20Snapshot-_getCurrentSnapshotId--)

		内部方法，获取当前的快照 ID
		
		- 值参数 unit256
	- [balanceOfAt(account, snapshotId)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#ERC20Snapshot-balanceOfAt-address-uint256-)

		公开方法，`account` 检索创建 `snapshotId` 时的余额。
		
		- 值参数 unit256
	- [totalSupplyAt(snapshotId)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#ERC20Snapshot-totalSupplyAt-uint256-)

		公开方法，检索创建时的总供应量 snapshotId。
		
		- 值参数 unit256
	- [_beforeTokenTransfer(from, to, amount)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#ERC20Snapshot-_beforeTokenTransfer-address-address-uint256-)
		
		内部方法 
	- ERC20功能相同
		- [constructor(name_, symbol_)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#ERC20-constructor-string-string-)
		- [name()](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#ERC20-name--)
		- [symbol()](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#ERC20-symbol--)
		- [decimals()](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#ERC20-decimals--)
		- [totalSupply()](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#ERC20-totalSupply--)
		- [balanceOf(account)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#ERC20-balanceOf-address-)
		- [transfer(to, amount)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#ERC20-transfer-address-uint256-)
		- [allowance(owner, spender)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#ERC20-allowance-address-address-)
		- [approve(spender, amount)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#ERC20-approve-address-uint256-)
		- [transferFrom(from, to, amount)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#ERC20-transferFrom-address-address-uint256-)
		- [increaseAllowance(spender, addedValue)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#ERC20-increaseAllowance-address-uint256-)
		- [decreaseAllowance(spender, subtractedValue)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#ERC20-decreaseAllowance-address-uint256-)
		- [_transfer(from, to, amount)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#ERC20-_transfer-address-address-uint256-)
		- [_mint(account, amount)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#ERC20-_mint-address-uint256-)
		- [_burn(account, amount)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#ERC20-_burn-address-uint256-)
		- [_approve(owner, spender, amount)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#ERC20-_approve-address-address-uint256-)
		- [_spendAllowance(owner, spender, amount)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#ERC20-_spendAllowance-address-address-uint256-)
		- [_afterTokenTransfer(from, to, amount)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#ERC20-_afterTokenTransfer-address-address-uint256-) 
- 事件
	- [Snapshot(id)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#ERC20Snapshot-Snapshot-uint256-) 

		由创建 [_snapshot](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#ERC20Snapshot-_snapshot--) 标识的快照 id 时发出。
	- IERC20功能相同
		- [Transfer(from, to, value)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#IERC20-Transfer-address-address-uint256-)
		- [Approval(owner, spender, value)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#IERC20-Approval-address-address-uint256-)

#### ERC20Votes [github](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/v4.6.0/contracts/token/ERC20/extensions/ERC20Votes.sol)
	import "@openzeppelin/contracts/token/ERC20/extensions/ERC20Votes.sol";
扩展 ERC20 以支持类似 Compound 的投票和委托。这个版本比 Compound 更通用，最多支持 2 <sup>224</sup> - 1 的Token供应，而 COMP 限制为 2 <sup>96</sup> - 1。

如果需要精确的 COMP 兼容性，请使用[ERC20VotesComp](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#ERC20VotesComp)此模块的变体。

此扩展保留每个帐户投票权的历史记录（检查点）。投票权可以通过 [delegate](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#ERC20Votes-delegate-address-) 直接调用函数或通过提供与 [delegateBySig](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#ERC20Votes-delegateBySig-address-uint256-uint256-uint8-bytes32-bytes32-). 投票权可以通过 public accessors [getVotes](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#ERC20Votes-getVotes-address-)和 [getPastVotes](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#ERC20Votes-getPastVotes-address-uint256-).
 
默认情况下，Token余额不考虑投票权。这使得转账更便宜。缺点是它需要用户委托给自己以激活检查点并跟踪他们的投票权。

从 v4.2 开始可用。

- 功能
	- [checkpoints(account, pos)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#ERC20Votes-checkpoints-address-uint32-)

		公共方法，获取 `account` 的 `pos-th` 检查点 。
		
		- 返回值类型 struct
	- [numCheckpoints(account)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#ERC20Votes-numCheckpoints-address-)

		公共方法，获取 `account ` 检查点的数量。
		
		- 返回值类型 uint32
	- [delegates(account)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#ERC20Votes-delegates-address-)

		公共方法, 获取 `account` 当前委托的地址。
		
		- 返回值类型 address
	- [getVotes(account)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#ERC20Votes-getVotes-address-)

		公共方法，获取当前 account 投票余额
		 
		-  返回值类型 uint256
	- [getPastVotes(account, blockNumber)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#ERC20Votes-getPastVotes-address-uint256-)

		公共方法，`account` 检索结束时的投票数 `blockNumber`。

		- 要求
			- blockNumber 一定已经被开采了
		- 返回值类型 uint256
	- [getPastTotalSupply(blockNumber)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#ERC20Votes-getPastTotalSupply-uint256-)

		公共方法，在 `blockNumber` 的末尾检索 `totalSupply`。注意，这个值是所有余额的总和。它是但不是所有委托投票的总和!

		- 要求
			- blockNumber 一定已经被开采了
		- 返回值类型 uint256
	- [delegate(delegatee)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#ERC20Votes-delegate-address-)

		公共方法，将发件人(`sender`)的投票委托给委托人 (`delegatee`).
	- [delegateBySig(delegatee, nonce, expiry, v, r, s)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#ERC20Votes-delegateBySig-address-uint256-uint256-uint8-bytes32-bytes32-)

		公共方法，将签名者的投票委托给委托人 (`delegatee`)
	- [_maxSupply()](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#ERC20Votes-_maxSupply--)

		内部方法，最大Token供应。默认为 type(uint <sup>224</sup>).max(2 <sup>224</sup> - 1)
		
		- 返回值类型 uint224
	- [_mint(account, amount)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#ERC20Votes-_mint-address-uint256-)

		内部方法，在增加后进行快照的总数
	- [_burn(account, amount)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#ERC20Votes-_burn-address-uint256-)

		内部方法，在减少后进行快照的总数
	- [_afterTokenTransfer(from, to, amount)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#ERC20Votes-_afterTokenTransfer-address-address-uint256-)

		内部方法，转移Token时移动投票权。
		
		发出 `{DelegateVotesChanged} ` 事件。
	- [_delegate(delegator, delegatee)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#ERC20Votes-_delegate-address-address-)

		内部方法，改变委派方式，让委派者 （`delegator`）进行委派 （ `delegatee`）。

		发出事件 `{DelegateChanged}` 和` {DelegateVotesChanged}`。
	- ERC20 Permit 功能相同
		- [constructor(name)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#ERC20Permit-constructor-string-)
		- [permit(owner, spender, value, deadline, v, r, s)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#ERC20Permit-permit-address-address-uint256-uint256-uint8-bytes32-bytes32-)
		- [nonces(owner)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#ERC20Permit-nonces-address-)
		- [DOMAIN_SEPARATOR()](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#ERC20Permit-DOMAIN_SEPARATOR--)
		- [_useNonce(owner)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#ERC20Permit-_useNonce-address-)
	- EIP712功能相同
		- [_domainSeparatorV4()](https://docs.openzeppelin.com/contracts/4.x/api/utils#EIP712-_domainSeparatorV4--)
		- [_hashTypedDataV4(structHash)](https://docs.openzeppelin.com/contracts/4.x/api/utils#EIP712-_hashTypedDataV4-bytes32-)
	- ERC20功能相同
		- [name()](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#ERC20-name--)
		- [symbol()](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#ERC20-symbol--)
		- [decimals()](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#ERC20-decimals--)
		- [totalSupply()](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#ERC20-totalSupply--)
		- [balanceOf(account)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#ERC20-balanceOf-address-)
		- [transfer(to, amount)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#ERC20-transfer-address-uint256-)
		- [allowance(owner, spender)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#ERC20-allowance-address-address-)
		- [approve(spender, amount)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#ERC20-approve-address-uint256-)
		- [transferFrom(from, to, amount)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#ERC20-transferFrom-address-address-uint256-)
		- [increaseAllowance(spender, addedValue)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#ERC20-increaseAllowance-address-uint256-)
		- [decreaseAllowance(spender, subtractedValue)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#ERC20-decreaseAllowance-address-uint256-)
		- [_transfer(from, to, amount)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#ERC20-_transfer-address-address-uint256-)
		- [_approve(owner, spender, amount)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#ERC20-_approve-address-address-uint256-)
		- [_spendAllowance(owner, spender, amount)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#ERC20-_spendAllowance-address-address-uint256-)
		- [_beforeTokenTransfer(from, to, amount)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#ERC20-_beforeTokenTransfer-address-address-uint256-)
- 事件
	- IERC20功能相同
		- [Transfer(from, to, value)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#IERC20-Transfer-address-address-uint256-)
		- [Approval(owner, spender, value)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#IERC20-Approval-address-address-uint256-)
	- vote功能相同
		- [DelegateChanged(delegator, fromDelegate, toDelegate)](https://docs.openzeppelin.com/contracts/4.x/api/governance#IVotes-DelegateChanged-address-address-address-)		- [DelegateVotesChanged(delegate, previousBalance, newBalance) ](https://docs.openzeppelin.com/contracts/4.x/api/governance#IVotes-DelegateVotesChanged-address-uint256-uint256-)

#### ERC20VotesComp [github](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/v4.6.0/contracts/token/ERC20/extensions/ERC20VotesComp.sol)
	import "@openzeppelin/contracts/token/ERC20/extensions/ERC20VotesComp.sol";
扩展 ERC20 以支持 Compound 的投票和授权。此版本与 Compound 的界面完全匹配，缺点是仅支持最多 (2 <sup>96</sup> - 1) 的供应。

如果您需要与 COMP 完全兼容（例如，为了将您的Token与 Governor Alpha 或 Bravo 一起使用）并且您确定 2 <sup>96</sup> 的供应上限对您来说足够，您应该使用此合约。否则，请使用 [ERC20Votes](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#ERC20Votes) 此模块的变体。

此扩展保留每个帐户投票权的历史记录（检查点）。投票权可以通过 [delegate](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#ERC20Votes-delegate-address-) 直接调用函数或通过提供与 [delegateBySig](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#ERC20Votes-delegateBySig-address-uint256-uint256-uint8-bytes32-bytes32-)一起使用的签名来委托 . 

投票权可以通过公共接口 [getCurrentVotes](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#ERC20VotesComp-getCurrentVotes-address-)和 [getPriorVotes](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#ERC20VotesComp-getPriorVotes-address-uint256-) 进行查询

默认情况下，Token余额不考虑投票权。这使得转账更便宜。缺点是它需要用户委托给自己以激活检查点并跟踪他们的投票权。

从 v4.2 开始可用。

- 功能
	- [getCurrentVotes(account)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#ERC20VotesComp-getCurrentVotes-address-)

		外部方法，访问器的 Comp 版本 [getVotes](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#ERC20Votes-getVotes-address-)
		
		-   返回类型 `uint96`
	- [getPriorVotes(account, blockNumber)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#ERC20VotesComp-getPriorVotes-address-uint256-)

		外部方法，访问器的 Comp 版本 [getPastVotes](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#ERC20Votes-getPastVotes-address-uint256-)
	
		-   返回类型 `uint96`
	- [_maxSupply()](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#ERC20VotesComp-_maxSupply--)

		最大Token供应。减少到 type(uint96).max(2 <sup>96</sup> - 1) 以适应 COMP 接口。
		
		-   返回类型 `uint224`
	- ERC20 vote
		- [checkpoints(account, pos)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#ERC20Votes-checkpoints-address-uint32-)
		- [numCheckpoints(account)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#ERC20Votes-numCheckpoints-address-)
		- [delegates(account)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#ERC20Votes-delegates-address-)
		- [getVotes(account)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#ERC20Votes-getVotes-address-)
		- [getPastVotes(account, blockNumber)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#ERC20Votes-getPastVotes-address-uint256-)
		- [getPastTotalSupply(blockNumber)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#ERC20Votes-getPastTotalSupply-uint256-)
		- [delegate(delegatee)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#ERC20Votes-delegate-address-)
		- [delegateBySig(delegatee, nonce, expiry, v, r, s)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#ERC20Votes-delegateBySig-address-uint256-uint256-uint8-bytes32-bytes32-)
		- [_mint(account, amount)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#ERC20Votes-_mint-address-uint256-)
		- [_burn(account, amount)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#ERC20Votes-_burn-address-uint256-)
		- [_afterTokenTransfer(from, to, amount)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#ERC20Votes-_afterTokenTransfer-address-address-uint256-)
		- [_delegate(delegator, delegatee)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#ERC20Votes-_delegate-address-address-)
	- ERC20 Permit 功能相同
		- [constructor(name)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#ERC20Permit-constructor-string-)
		- [permit(owner, spender, value, deadline, v, r, s)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#ERC20Permit-permit-address-address-uint256-uint256-uint8-bytes32-bytes32-)
		- [nonces(owner)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#ERC20Permit-nonces-address-)
		- [DOMAIN_SEPARATOR()](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#ERC20Permit-DOMAIN_SEPARATOR--)
		- [_useNonce(owner)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#ERC20Permit-_useNonce-address-)
	- EIP712功能相同
		- [_domainSeparatorV4()](https://docs.openzeppelin.com/contracts/4.x/api/utils#EIP712-_domainSeparatorV4--)
		- [_hashTypedDataV4(structHash)](https://docs.openzeppelin.com/contracts/4.x/api/utils#EIP712-_hashTypedDataV4-bytes32-)
	- ERC20
		- [name()](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#ERC20-name--)
		- [symbol()](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#ERC20-symbol--)
		- [decimals()](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#ERC20-decimals--)
		- [totalSupply()](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#ERC20-totalSupply--)
		- [balanceOf(account)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#ERC20-balanceOf-address-)
		- [transfer(to, amount)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#ERC20-transfer-address-uint256-)
		- [allowance(owner, spender)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#ERC20-allowance-address-address-)
		- [approve(spender, amount)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#ERC20-approve-address-uint256-)
		- [transferFrom(from, to, amount)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#ERC20-transferFrom-address-address-uint256-)
		- [increaseAllowance(spender, addedValue)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#ERC20-increaseAllowance-address-uint256-)
		- [decreaseAllowance(spender, subtractedValue)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#ERC20-decreaseAllowance-address-uint256-)
		- [_transfer(from, to, amount)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#ERC20-_transfer-address-address-uint256-)
		- [_approve(owner, spender, amount)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#ERC20-_approve-address-address-uint256-)
		- [_spendAllowance(owner, spender, amount)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#ERC20-_spendAllowance-address-address-uint256-)
		- [_beforeTokenTransfer(from, to, amount)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#ERC20-_beforeTokenTransfer-address-address-uint256-)
- 事件
	- IERC20功能相同
		- [Transfer(from, to, value)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#IERC20-Transfer-address-address-uint256-)
		- [Approval(owner, spender, value)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#IERC20-Approval-address-address-uint256-)
	- vote功能相同
		- [DelegateChanged(delegator, fromDelegate, toDelegate)](https://docs.openzeppelin.com/contracts/4.x/api/governance#IVotes-DelegateChanged-address-address-address-)		- [DelegateVotesChanged(delegate, previousBalance, newBalance) ](https://docs.openzeppelin.com/contracts/4.x/api/governance#IVotes-DelegateVotesChanged-address-uint256-uint256-)

#### ERC20Wrapper [github](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/v4.6.0/contracts/token/ERC20/extensions/ERC20Wrapper.sol)
	import "@openzeppelin/contracts/token/ERC20/extensions/ERC20Wrapper.sol";
扩展 ERC20 Token合约以支持Token包装。

用户可以存入和提取“底层Token”并获得匹配数量的“包装Token”。这在与其他模块结合使用时很有用。例如，将这种包装机制与 [ERC20Votes](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#ERC20Votes) 将允许将现有的 “基本” ERC20 包装成治理Token。

从 v4.2 开始可用。

- 功能
	- [constructor(underlyingToken)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#ERC20Wrapper-constructor-contract-IERC20-)

		内部方法
	- [decimals()](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#ERC20Wrapper-decimals--)
	
		公共方法，见 [ERC20.decimals](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#ERC20-decimals--)。
		
		- 返回值类型  uint8
	- [depositFor(account, amount)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#ERC20Wrapper-depositFor-address-uint256-)

		公共方法，允许用户存入底层Token并铸造相应数量的包装Token。
		
		- 返回值类型  bool
	- [withdrawTo(account, amount)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#ERC20Wrapper-withdrawTo-address-uint256-)

		公共方法，允许用户烧掉一定数量的打包Token并提取相应数量的底层Token。
		
		- 返回值类型  bool
	- [_recover(account)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#ERC20Wrapper-_recover-address-)

		内部方法，Mint 包装的Token以覆盖任何可能被错误转移的底层Token。如果需要，可以通过访问控制公开的内部功能。
		
		- 返回值类型  uint256
	- ERC20 功能相同
		- [name()](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#ERC20-name--)
		- [symbol()](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#ERC20-symbol--)
		- [totalSupply()](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#ERC20-totalSupply--)
		- [balanceOf(account)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#ERC20-balanceOf-address-)
		- [transfer(to, amount)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#ERC20-transfer-address-uint256-)
		- [allowance(owner, spender)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#ERC20-allowance-address-address-)
		- [approve(spender, amount)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#ERC20-approve-address-uint256-)
		- [transferFrom(from, to, amount)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#ERC20-transferFrom-address-address-uint256-)
		- [increaseAllowance(spender, addedValue)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#ERC20-increaseAllowance-address-uint256-)
		- [decreaseAllowance(spender, subtractedValue)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#ERC20-decreaseAllowance-address-uint256-)
		- [_transfer(from, to, amount)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#ERC20-_transfer-address-address-uint256-)
		- [_mint(account, amount)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#ERC20-_mint-address-uint256-)
		- [_burn(account, amount)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#ERC20-_burn-address-uint256-)
		- [_approve(owner, spender, amount)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#ERC20-_approve-address-address-uint256-)
		- [_spendAllowance(owner, spender, amount)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#ERC20-_spendAllowance-address-address-uint256-)
		- [_beforeTokenTransfer(from, to, amount)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#ERC20-_beforeTokenTransfer-address-address-uint256-)
		- [_afterTokenTransfer(from, to, amount)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#ERC20-_afterTokenTransfer-address-address-uint256-)
- 事件
	- IERC20功能相同
		- [Transfer(from, to, value)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#IERC20-Transfer-address-address-uint256-)
		- [Approval(owner, spender, value)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#IERC20-Approval-address-address-uint256-)

#### ERC20FlashMint [github](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/v4.6.0/contracts/token/ERC20/extensions/ERC20FlashMint.sol)
	import "@openzeppelin/contracts/token/ERC20/extensions/ERC20FlashMint.sol";
实施 ERC3156 快速贷款延期，如 [ERC-3156](https://eips.ethereum.org/EIPS/eip-3156) 中所定义。

添加  [flashLoan](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#ERC20FlashMint-flashLoan-contract-IERC3156FlashBorrower-address-uint256-bytes-) 在Token级别提供闪贷支持的方法。默认情况下不收费，但可以通过覆盖来更改 [flashFee](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#ERC20FlashMint-flashFee-address-uint256-)。

从 v4.1 开始可用。

- 功能
	- [maxFlashLoan(token)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#ERC20FlashMint-maxFlashLoan-address-)

		公共方法，返回可借出的最大Token数量。
		
		- 返回值类型  uint256
	- [flashFee(token, amount)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#ERC20FlashMint-flashFee-address-uint256-)

		公共方法，返回进行快速贷款时应用的费用。默认情况下，此实现的费用为 0。这个函数可以重载，使闪贷机制通货紧缩。
		
		- 返回值类型  uint256
	- [flashLoan(receiver, token, amount, data)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#ERC20FlashMint-flashLoan-contract-IERC3156FlashBorrower-address-uint256-bytes-)

		公共方法，执行闪贷。新Token被铸造并发送给 receiver 需要实现 [IERC3156FlashBorrower](https://docs.openzeppelin.com/contracts/4.x/api/interfaces#IERC3156FlashBorrower) 接口的 。到闪电贷结束时，接收方预计将拥有金额 + 费用Token，并将它们授权回Token合约本身，以便它们可以被销毁。
		
		- 返回值类型  bool
	- ERC20 功能相同
		- [constructor(name_, symbol_)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#ERC20-constructor-string-string-)
		- [name()](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#ERC20-name--)
		- [symbol()](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#ERC20-symbol--)
		- [decimals()](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#ERC20-decimals--)
		- [totalSupply()](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#ERC20-totalSupply--)
		- [balanceOf(account)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#ERC20-balanceOf-address-)
		- [transfer(to, amount)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#ERC20-transfer-address-uint256-)
		- [allowance(owner, spender)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#ERC20-allowance-address-address-)
		- [approve(spender, amount)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#ERC20-approve-address-uint256-)
		- [transferFrom(from, to, amount)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#ERC20-transferFrom-address-address-uint256-)
		- [increaseAllowance(spender, addedValue)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#ERC20-increaseAllowance-address-uint256-)
		- [decreaseAllowance(spender, subtractedValue)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#ERC20-decreaseAllowance-address-uint256-)
		- [_transfer(from, to, amount)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#ERC20-_transfer-address-address-uint256-)
		- [_mint(account, amount)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#ERC20-_mint-address-uint256-)
		- [_burn(account, amount)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#ERC20-_burn-address-uint256-)
		- [_approve(owner, spender, amount)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#ERC20-_approve-address-address-uint256-)
		- [_spendAllowance(owner, spender, amount)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#ERC20-_spendAllowance-address-address-uint256-)
		- [_beforeTokenTransfer(from, to, amount)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#ERC20-_beforeTokenTransfer-address-address-uint256-)
		- [_afterTokenTransfer(from, to, amount)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#ERC20-_afterTokenTransfer-address-address-uint256-)
- 事件
	- IERC20功能相同
		- [Transfer(from, to, value)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#IERC20-Transfer-address-address-uint256-)
		- [Approval(owner, spender, value)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#IERC20-Approval-address-address-uint256-)

### EIP 草案
以下 EIP 仍处于 Draft 状态。由于它们是草稿的性质，这些合约的细节可能会发生变化，我们不能保证它们的[稳定性](https://docs.openzeppelin.com/contracts/4.x/releases-stability)。OpenZeppelin 合约的次要版本可能包含此目录中合约的重大更改，这些更改将在[更改日志](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/CHANGELOG.md)中正式公布。此处包含的 EIP 被生产中的项目使用，这可能会降低它们发生重大变化的可能性。

#### ERC20Permit [github](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/v4.6.0/contracts/token/ERC20/extensions/draft-ERC20Permit.sol)
	import "@openzeppelin/contracts/token/ERC20/extensions/draft-ERC20Permit.sol";
实施 ERC20 许可证扩展，允许通过签名进行授权，如 [EIP-2612](https://eips.ethereum.org/EIPS/eip-2612) 中所定义。

添加 [permit](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#ERC20Permit-permit-address-address-uint256-uint256-uint8-bytes32-bytes32-) 方法，该方法可用于 [IERC20.allowance](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#IERC20-allowance-address-address-) 通过显示由帐户签名的消息来更改帐户的 ERC20 配额（请参阅 ）。通过不依赖 [IERC20.approve](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#IERC20-approve-address-uint256-)，Token持有者账户不需要发送交易，因此根本不需要持有 Ether。

从 v3.4 开始可用。

- 功能
	- [constructor(name)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#ERC20Permit-constructor-string-)

		内部方法，使用参数初始化 [EIP712](https://docs.openzeppelin.com/contracts/4.x/api/utils#EIP712) 域分隔符 `name`，并设置 `version` 为 `"1"`。

		`name` 使用定义为 ERC20 Token名称的名称是个好主意。
	- [permit(owner, spender, value, deadline, v, r, s)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#ERC20Permit-permit-address-address-uint256-uint256-uint8-bytes32-bytes32-)

		公共方法，见 [IERC20Permit.permit](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#IERC20Permit-permit-address-address-uint256-uint256-uint8-bytes32-bytes32-)。
	- [nonces(owner)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#ERC20Permit-nonces-address-)

		公共方法，见 [IERC20Permit.nonces](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#IERC20Permit-nonces-address-)
		
		- 返回值类型 uint256
	- [DOMAIN_SEPARATOR()](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#ERC20Permit-DOMAIN_SEPARATOR--)

		外部方法，见 [IERC20Permit.DOMAIN_SEPARATOR](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#IERC20Permit-DOMAIN_SEPARATOR--)。
		
		- 返回值类型 bytes32
	- [_useNonce(owner)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#ERC20Permit-DOMAIN_SEPARATOR--)

		“消耗一个随机数”：返回当前值和增量。

		从 v4.1 开始可用。
		
		- 返回值类型 当前的 unit 256
	- EIP712功能相同
		- [_domainSeparatorV4()](https://docs.openzeppelin.com/contracts/4.x/api/utils#EIP712-_domainSeparatorV4--)
		- [_hashTypedDataV4(structHash)](https://docs.openzeppelin.com/contracts/4.x/api/utils#EIP712-_hashTypedDataV4-bytes32-)
	- ERC20
		- [name()](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#ERC20-name--)
		- [symbol()](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#ERC20-symbol--)
		- [decimals()](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#ERC20-decimals--)
		- [totalSupply()](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#ERC20-totalSupply--)
		- [balanceOf(account)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#ERC20-balanceOf-address-)
		- [transfer(to, amount)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#ERC20-transfer-address-uint256-)
		- [allowance(owner, spender)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#ERC20-allowance-address-address-)
		- [approve(spender, amount)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#ERC20-approve-address-uint256-)
		- [transferFrom(from, to, amount)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#ERC20-transferFrom-address-address-uint256-)
		- [increaseAllowance(spender, addedValue)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#ERC20-increaseAllowance-address-uint256-)
		- [decreaseAllowance(spender, subtractedValue)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#ERC20-decreaseAllowance-address-uint256-)
		- [_transfer(from, to, amount)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#ERC20-_transfer-address-address-uint256-)
		- [_mint(account, amount)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#ERC20-_mint-address-uint256-)
		- [_burn(account, amount)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#ERC20-_burn-address-uint256-)
		- [_approve(owner, spender, amount)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#ERC20-_approve-address-address-uint256-)
		- [_spendAllowance(owner, spender, amount)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#ERC20-_spendAllowance-address-address-uint256-)
		- [_beforeTokenTransfer(from, to, amount)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#ERC20-_beforeTokenTransfer-address-address-uint256-)
- 事件
	- IERC20功能相同
		- [Transfer(from, to, value)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#IERC20-Transfer-address-address-uint256-)
		- [Approval(owner, spender, value)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#IERC20-Approval-address-address-uint256-)

### 预设
这些合约是上述功能的预配置组合。它们可以通过继承或作为模型来复制和粘贴其源代码
### 实用程序
#### SafeERC20 [github](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/v4.6.0/contracts/token/ERC20/utils/SafeERC20.sol)
	import "@openzeppelin/contracts/token/ERC20/utils/SafeERC20.sol";
围绕引发失败的 ERC20 操作的包装器（当Token合约返回 false 时）。还支持不返回值（而是还原或抛出失败）的Token，假定非还原调用是成功的。要使用这个库，你可以使用 IERC20 的  SafeERC20 ; 

在你的合约中添加一个语句，它允许你调用安全操作 `token.safeTransfer(…​)`等。

- 功能
	- [safeTransfer(token, to, value)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#SafeERC20-safeTransfer-contract-IERC20-address-uint256-)

		内部方法
	- [safeTransferFrom(token, from, to, value)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#SafeERC20-safeTransferFrom-contract-IERC20-address-address-uint256-)

		内部方法
	- [safeApprove(token, spender, value)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#SafeERC20-safeApprove-contract-IERC20-address-uint256-)

		已弃用。
	- [safeIncreaseAllowance(token, spender, value)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#SafeERC20-safeIncreaseAllowance-contract-IERC20-address-uint256-)
		
		内部方法		
	- [safeDecreaseAllowance(token, spender, value)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#SafeERC20-safeDecreaseAllowance-contract-IERC20-address-uint256-)

		内部方法

#### TokenTimelock [github](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/v4.6.0/contracts/token/ERC20/utils/TokenTimelock.sol)
	import "@openzeppelin/contracts/token/ERC20/utils/TokenTimelock.sol";
Token持有者合约，允许受益人在给定的发行时间后提取Token。

对于简单的归属时间表很有用，例如“顾问在 1 年后获得所有Token”。

- 功能
	- [constructor(token_, beneficiary_, releaseTime_)](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#TokenTimelock-constructor-contract-IERC20-address-uint256-)

		公共方法，部署一个时间锁实例，该实例能够保存指定的Token，并且只会 `releasereleaseTime_` 在之后调用 [release](https://docs.openzeppelin.com/contracts/4.x/api/token/`beneficiary_`。发布时间指定为 Unix 时间戳（以秒为单位）。
	- [token()](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#TokenTimelock-token--)

		公共方法，返回持有的Token。
		
		- 返回值类型 合约 IERC20
	- [beneficiary()](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#TokenTimelock-beneficiary--)
	
		公共方法，返回将收到Token的受益人。
		
		- 返回值类型 address
	- [releaseTime()](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#TokenTimelock-releaseTime--)

		公共方法，返回自 Unix 纪元（即 Unix 时间戳）以来Token释放的时间（以秒为单位）。
		
		- 返回值类型 uint256
	- [release()](https://docs.openzeppelin.com/contracts/4.x/api/token/erc20#TokenTimelock-release--)

		公共方法，将时间锁持有的Token转移给受益人。只有在发布时间之后调用才会成功。