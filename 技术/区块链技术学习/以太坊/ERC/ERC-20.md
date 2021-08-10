# EIP-20：ERC-20 代币标准
## 简单总结
令牌的标准接口
## ERC-20 抽象
以下标准允许在智能合约中为代币实施标准 API。该标准提供了转移代币的基本功能，并允许批准代币，以便其他链上第三方使用它们。
## 动机
标准接口允许以太坊上的任何代币被其他应用程序重用：从钱包到去中心化交易所。
## 规格
## 令牌
### 函数方法
- 注意事项
	- 以下规范使用 Solidity 0.4.17（或更高版本）的语法
	- 调用者必须 `false` 从 `returns (bool success)`. 调用者不能假设 `false` 永远不会返回！
- 名称

	返回令牌的名称 - 例如 "MyToken"。

	可选 - 此方法可用于提高可用性，但接口和其他合同不得期望这些值存在。

		function name() public view returns (string)
- 象征

	返回令牌的符号。例如“HIX”。

		function symbol() public view returns (string)
- 小数点

	返回令牌使用的小数位数 - 例如 `8`，表示将令牌数量除以 `100000000` 得到其用户表示。

	可选 - 此方法可用于提高可用性，但接口和其他合同不得期望这些值存在。

		function decimals() public view returns (uint8)
- 总供给

	返回总代币供应量。

		function totalSupply() public view returns (uint256)
- 余额

	返回地址为另一个账户的账户余额 `_owner`。

		function balanceOf(address _owner) public view returns (uint256 balance)
- 转移

	将 `_value` 一定数量的代币转移到 `address _to`，并且必须触发该 `Transfer` 事件。`throw` 如果消息调用者的账户余额没有足够的代币来消费，则该函数抛出异常。

	注意 0 值的传输必须被视为正常传输并触发 `Transfer` 事件。

		function transfer(address _to, uint256 _value) public returns (bool success)
- 从转移

	将 `_value` 一定数量的代币从 address 转移 `_from` 到 address `_to`，并且必须触发该 `Transfer` 事件。

	该 `transferFrom` 方法用于提款工作流程，允许合约代表您转移代币。例如，这可用于允许合约代表您转移代币和/或以子货币收取费用。`throw` 除非 `_from` 帐户通过某种机制故意授权消息的发送者，否则该函数抛出异常。

	注意0 值的传输必须被视为正常传输并触发 `Transfer` 事件。

		function transferFrom(address _from, address _to, uint256 _value) public returns (bool success)
- 批准

	允许 `_spender` 多次从您的账户中提款，最多提款 `_value` 金额。如果再次调用此函数，它将用覆盖当前限额 `_value`。

	注：为了防止攻击向量像一个[在这里描述](https://docs.google.com/document/d/1YLPtQxZu1UAvO9cZ1O2RPXBbT0mooh4DYKjA_jp-RLM/)和讨论[在这里](https://github.com/ethereum/EIPs/issues/20#issuecomment-263524729)，他们首先要设置限额的用户界面 0 将它设置为另一个值相同花钱之前。虽然合同本身不应该强制执行它，以允许与之前部署的合同向后兼容

		function approve(address _spender, uint256 _value) public returns (bool success)
- 津贴

	返回 `_spender` 仍允许从中提取的金额 `_owner`。

		function allowance(address _owner, address _spender) public view returns (uint256 remaining)

### 事件
- 转移

	必须在代币转移时触发，包括零值转移。

	创建新代币的代币合约应该触发一个 `Transfer` 事件，`_from` 地址设置为 `0x0` 创建代币时。

		event Transfer(address indexed _from, address indexed _to, uint256 _value)
- 赞同

	必须在对的任何  `approve(address _spender, uint256 _value)` 成功调用时触发。

		event Approval(address indexed _owner, address indexed _spender, uint256 _value)

## 执行
以太坊网络上已经部署了大量符合 ERC20 的代币。不同的团队编写了不同的实现，它们具有不同的权衡：从节省 gas 到提高安全性。

示例实现可在

- [OpenZeppelin 实现](https://github.com/OpenZeppelin/openzeppelin-solidity/blob/9b3710465583284b8c4c5d2245749246bb2e0094/contracts/token/ERC20/ERC20.sol)
- [ConsenSys 实施](https://github.com/ConsenSys/Tokens/blob/fdf687c69d998266a95f15216b1955a4965a0a6d/contracts/eip20/EIP20.sol)

## 历史
与本标准相关的历史链接：

Vitalik Buterin 的原始提案：https : //github.com/ethereum/wiki/wiki/Standardized_Contract_APIs/499c882f3ec123537fc2fccd57eaa29e6032fe4a
Reddit 讨论：https : //www.reddit.com/r/ethereum/comments/3n8fkn/lets_talk_about_the_coin_standard/
原始问题 #20：https : //github.com/ethereum/EIPs/issues/20
“EIP-20：ERC-20 代币标准”，以太坊改进提案，第 20，2015 年 11 月。 [在线连载]。可用：https://eips.ethereum.org/EIPS/eip-20。
## 参考
- [eip-20/git](https://github.com/ethereum/EIPs/blob/master/EIPS/eip-20.md)
- [eip-20/eth](https://eips.ethereum.org/EIPS/eip-20)