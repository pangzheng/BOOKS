# EIP-170: 合约代码大小限制(Contract code size limit)
## 参数
- MAX_CODE_SIZE: 0x6000( 2**14 + 2**13)
- 分叉高度 FORK_BLKNUM: 2,675,000
- CHAIN_ID: 1 (主网)

## 规格
如果 `block.number >= FORK_BLKNUM`，那么如果合约创建初始化返回长度超过 `MAX_CODE_SIZE` 字节的数据，合约创建将失败并出现气体不足错误。

## 基本原理
目前，以太坊仍然存在一个轻微的二次漏洞：

当调用合约时，即使调用消耗恒定量的 gas，调用也会触发 O(n) 成本，即从磁盘读取代码，对代码进行预处理虚拟机执行，并将 O(n) 数据添加到 Merkle 证明中以用于块的有效性证明。在当前的 gas 水平下，即使不是最理想的，这也是可以接受的。

在未来可能触发的更高 gas 水平下，可能很快就会因为动态 gas 限制规则，这将成为一个更大的问题——不像最近的拒绝服务攻击那么严重，但仍然不方便，特别是对于未来的轻客户端验证有效或无效的证明。解决方案是对可以保存到区块链的对象的大小设置硬上限，

## 参考
- EIP-170 问题和讨论：https://github.com/ethereum/EIPs/issues/170
- pyethereum 实现：https://github.com/ethereum/pyethereum/blob/5217294871283d8dc4fb3ca9d8a78c7d416490e8/ethereum/messages.py#L397
- [eip-170](https://eips.ethereum.org/EIPS/eip-170)