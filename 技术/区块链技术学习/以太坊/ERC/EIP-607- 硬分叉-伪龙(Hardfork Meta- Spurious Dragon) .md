# EIP-607: 硬分叉-伪龙(Hardfork Meta: Spurious Dragon) 
## 抽象
这指定了名为 Spurious Dragon 的硬分叉中包含的更改。
### 为什么我们建议对网络进行硬分叉？
“伪龙”是 9 月和 10 月以太坊网络 DoS 攻击的双轮硬分叉响应中的第二次硬分叉。之前的硬分叉（又名“Tangerine Whistle”）解决了由于攻击导致的直接网络健康问题。即将到来的硬分叉解决了重要但不那么紧迫的问题，例如进一步调整操作码定价以防止未来对网络的攻击，启用区块链状态的“debloat”，并添加重放攻击保护。
## 规格
- 代码名: Spurious Dragon
- 别名: State-clearing
- 激活点:
	- Block >= 2,675,000 on Mainnet
	- Block >= 1,885,000 on Morden
- 包含 EIPs:
	- [EIP-155](https://eips.ethereum.org/EIPS/eip-155) (简单重放攻击保护)

		防止来自一条以太坊链的交易在另一条链上重播。例如：如果从 Morden 测试网向某人发送 150 个测试以太币，则无法在以太坊主链上重播相同的交易。重要提示： EIP 155向后兼容，因此以 “pre-Spurious-Dragon” 格式生成的交易仍将被接受。
		
		但是，为了确保您免受重放攻击，您仍然需要使用实现 EIP 155 的钱包解决方案。请注意，这种向后兼容性还意味着从尚未实现 EIP 155 的替代基于以太坊的区块链创建的交易（例如以太坊经典）仍然可以在以太坊主链上重播。
	- [EIP-160](https://eips.ethereum.org/EIPS/eip-160) (EXP成本增加)

		EXP 成本增加- 调整 `EXP` 操作码的价格，以平衡 `EXP` 的价格与操作的计算复杂性，本质上通过计算成本高昂的合约操作来降低网络速度变得更加困难。
	- [EIP-161](https://eips.ethereum.org/EIPS/eip-161) (状态树清除)

		可以以非常低的成本删除由于早期 DoS 攻击而被放入状态的大量空账户。使用此 EIP，只要 `触及`另一笔交易，`空`帐户就会从状态中删除。删除空账户大大减少了区块链状态的大小，这将提供客户端优化，例如更快的同步时间。实际的删除过程将在分叉后开始，系统地对攻击创建的空账户执行“CALL”。
	- [EIP-170](https://eips.ethereum.org/EIPS/eip-170) (合约代码大小限制)

		更改区块链上的合约可以拥有的最大代码大小。此更新可防止以固定 gas 成本重复访问大量帐户代码的攻击场景。最大大小已设置为 24576 字节，比当前部署的任何合约都大。
 
## 参考
[eip-607](https://eips.ethereum.org/EIPS/eip-607)
- https://blog.ethereum.org/2016/11/18/hard-fork-no-4-spurious-dragon/







