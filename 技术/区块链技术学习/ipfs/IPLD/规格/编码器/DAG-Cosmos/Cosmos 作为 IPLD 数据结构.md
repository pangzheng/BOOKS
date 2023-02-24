# Cosmos 作为 IPLD 数据结构
注意：这仍在进行中！

Tendermint 和 Cosmos 可以作为 IPLD 数据读取和访问。

在这些文档中，模式按其序列化块分组。除了“基本类型”和“加密类型”中列出的那些类型之外，代码块中的每个模式类型分组都表示一个数据结构，该数据结构被序列化到一个具有自己的链接 (CID) 的 IPLD 块中。

这包括 Tendermint 区块链和 CosmosSDK 状态机的模式。

有关 IPLD 架构语言的更多信息，请参阅 [IPLD 架构文档](https://ipld.io/docs/schemas/)。

## 数据结构说明
- Tendermint 和 Cosmos 数据结构基本类型
- Tendermint 和 Cosmos加密类型
- Tendermint 链数据结构
- Cosmos 状态机数据结构

## 数据结构图
![](./tendermint_dag.png)