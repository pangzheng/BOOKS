# EIP-155: 简单的重放攻击保护(Simple replay attack protection)
## [硬分叉](https://eips.ethereum.org/EIPS/eip-607)
## 参数
- `FORK_BLKNUM`: 2,675,000
- `CHAIN_ID`：1（主网）

## 规格
如果 `block.number >= FORK_BLKNUM` 并且`CHAIN_ID` 是可用的，那么在计算交易的哈希以进行签名时，

- 不仅对六个 rlp 编码元素进行哈希处理 `(nonce, gasprice, startgas, to, value, data)`
- 而应对九个 rlp 编码元素进行哈希处理 `(nonce, gasprice, startgas, to, value, data, chainid, 0, 0)`

如果这样做，则必须将 `v` 签名的如果您选择仅散列 6 个值，则继续像以前一样设置。`{0,1} + CHAIN_ID * 2 + 35{0,1}yrv{0,1} + 27`

如果 `block.number >= FORK_BLKNUM` 和 `v = CHAIN_ID * 2 + 35` 或 `v = CHAIN_ID * 2 + 36`，那么当计算交易的哈希以进行恢复时

- 不是对六个 rlp 编码元素 `(nonce, gasprice, startgas, to, value, data)` 进行哈希处理
- 而是对 九个 rlp 编码元素`(nonce, gasprice, startgas, to, value, data, chainid, 0, 0)`进行哈希处理。

当前存在的签名方案使用 `v = 27` 和 `v = 28` 保持有效，并继续在与以前相同的规则下运行。

### 例子
考虑一个带有 `nonce = 9`, `gasprice = 20 * 10**9`, `startgas = 21000`, `to = 0x3535353535353535353535353535353535353535`, `value = 10**18`, `data=' '`（空）的交易。

- “签名数据”变为：

		0xec098504a817c800825208943535353535353535353535353535353535353535880de0b6b3a764000080018080
- “签名哈希”变为：

		0xdaf5a779ae972f972197303d7b574746c7ef83eadac0f2791ad23db92e4c8e53

如果交易使用私钥签名 `0x4646464646464646464646464646464646464646464646464646464646464646`，则 v,r,s 值变为：

	(37, 18515461264373351373200002665853028612451056578545711640558177340181847433846, 46948507304638947509940763649030358759909902576025900602547168820602576006531)
注意使用 37 而不是 27。签名的 tx 将变为：

	0xf86c098504a817c800825208943535353535353535353535353535353535353535880de0b6b3a76400008025a028ef61340bd939bc2195fe537567866003e1a15d3c71ff63e1590620aa636276a067cbe9d8997f761aecb703304b3800ccf555c9f3dc64214b297fb1966a3b6d83
	
## 基本原理
这将提供一种无需在 ETC 或 Morden 测试网上工作即可发送在以太坊上工作的交易的方法。鼓励 ETC 采用此 EIP 但替换 `CHAIN_ID` 为不同的值，并鼓励所有未来的测试网、联盟链和 alt-etherea 采用此 EIP 替换 `CHAIN_ID` 为唯一值。
## 链 ID 列表
CHAIN_ID|名称
---|---
1|Ethereum mainnet
2|Morden (disused), Expanse mainnet
3|Ropsten
4|Rinkeby
5|Goerli
42|Kovan
1337|Geth private chains (default)

在 [chainid.network](https://chainid.network/) 上查找更多链 ID并为 [ethereum-lists/chains](https://github.com/ethereum-lists/chains) 做出贡献。

## 参考
[eip-155:Simple replay attack protection](https://eips.ethereum.org/EIPS/eip-155)