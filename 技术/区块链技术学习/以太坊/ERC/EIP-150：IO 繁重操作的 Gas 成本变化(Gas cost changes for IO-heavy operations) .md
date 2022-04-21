# EIP-150：IO 繁重操作的 Gas 成本算法变化(Gas cost changes for IO-heavy operations) 
## 协议参考
[eip-608](https://eips.ethereum.org/EIPS/eip-608)
## 参数
FORK_BLKNUM|CHAIN_ID|CHAIN_NAME
---|---|---
2,463,000|1|主网
## 规格
如果 `block.number >= FORK_BLKNUM` ，那么：

- 将 EXTCODESIZE 的 gas 成本增加到 700（从 20）。
- 将 EXTCODECOPY 的基本气体成本提高到 700（从 20）。
- 将 BALANCE 的 gas 成本从 20 增加到 400。
- 将 SLOAD 的 gas 成本从 50 增加到 200。
- 将 CALL、DELEGATECALL、CALLCODE 的 gas 成本从 40 增加到 700。
- 将 SELFDESTRUCT 的 gas 消耗增加到 5000（从 0）。
- 如果 SELFDESTRUCT 命中一个新创建的账户，它会触发 25000 的额外 gas 成本（类似于 CALL）。
- 将推荐的气体限制目标提高到 550 万。
- 将“除 64 分之一之外”定义N为 `N - floor(N/64)`。
- 如果一个调用请求的 gas 超过了最大允许量（即减去调用的 gas 成本和内存扩展后剩余的总 gas 量），不要返回 OOG 错误；相反，如果调用要求的 gas 超过最大允许量的 64 分之一，则调用最大允许量的 64 分之一（这相当于 EIP-90 1加上 EIP-114 2的版本） ）。CREATE 仅向子调用提供所有父 gas 的 64 分之一。

也就是说，替换：

        extra_gas = (not ext.account_exists(to)) * opcodes.GCALLNEWACCOUNT + \
            (value > 0) * opcodes.GCALLVALUETRANSFER
        if compustate.gas < gas + extra_gas:
            return vm_exception('OUT OF GAS', needed=gas+extra_gas)
        submsg_gas = gas + opcodes.GSTIPEND * (value > 0)
和：

        def max_call_gas(gas):
          return gas - (gas // 64)

        extra_gas = (not ext.account_exists(to)) * opcodes.GCALLNEWACCOUNT + \
            (value > 0) * opcodes.GCALLVALUETRANSFER
        if compustate.gas < extra_gas:
            return vm_exception('OUT OF GAS', needed=extra_gas)
        if compustate.gas < gas + extra_gas:
            gas = min(gas, max_call_gas(compustate.gas - extra_gas))
        submsg_gas = gas + opcodes.GSTIPEND * (value > 0)

## 基本原理
最近的拒绝服务攻击表明，读取状态树的操作码相对于其他操作码而言价格偏低。已经进行、正在进行并且可以进行软件更改以缓解这种情况；然而，事实仍然是这样的操作码将在很大程度上成为最容易通过交易垃圾邮件降低网络性能的已知机制。之所以出现这种担忧，是因为从磁盘读取需要很长时间，而且对未来的分片提议也是一个风险，因为迄今为止在降低网络性能方面最成功的“攻击交易”也需要数十兆字节来提供 Merkle 证明为了。

这个 EIP 增加了存储读取操作码的成本来解决这个问题。成本来自用于生成 1.0 天然气成本的计算表的更新版本：

	https://docs.google.com/spreadsheets/d/15wghZr-Z6sRSMdmRmhls9dVXTOpxKy8Y64oy9MvDZEQ/edit#gid=0；
这些规则尝试将需要读取的数据限制为 8 MB 以处理一个块，并包括估计 500 个字节用于 SLOAD 的 Merkle 证明和 1000 个用于帐户。

此 EIP 旨在简单，并在此表中计算的成本之上增加了 300 gas 的固定惩罚，以考虑加载代码的成本（在最坏的情况下约为 17-21 kb）。

引入 EIP 90 gas 机制是因为没有它，所有当前调用的合约都将停止工作，因为它们使用如下表达式 `msg.gas - 40` 确定拨打电话需要多少汽油，取决于通话的汽油成本为 40。此外，引入 EIP 114 是因为，鉴于我们正在使通话成本更高且更难以预测，我们有机会做它对当前可用的保证没有额外的成本，因此我们还实现了将调用堆栈深度限制替换为“更软”的基于气体的限制的好处，从而消除了调用堆栈深度攻击作为合约开发人员必须进行的一类攻击担心并因此增加合约编程的安全性。请注意，在给定参数的情况下，事实上的最大调用堆栈深度限制为 ~340（低于 ~1024），从而减轻了任何进一步潜在的依赖于调用的二次复杂度 DoS 攻击所造成的危害。

建议增加 gas limit，以保持系统对平均合约的实际每秒交易处理能力。

## 参考
- [EIP-90](https://github.com/ethereum/EIPs/issues/90)
- [EIP-114](https://github.com/ethereum/EIPs/issues/114)
- [EIP-150](https://eips.ethereum.org/EIPS/eip-150)