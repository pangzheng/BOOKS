# 编解码器：DAG-PB
DAG-PB 是一种编解码器，它在特定的 Protobuf 消息集中实现了 IPLD 数据模型的一个非常小的子集。

DAG-PB 可以由 Protobuf 库解析，并且可以由 Protobuf 库发出（有一些警告—— DAG-PB 实际上在某些细节上比 Protobuf 更严格）。

DAG-PB 只实现了 [IPLD 数据模型](https://ipld.io/glossary/#data-model) 的一小部分。您不应该期望能够获取任意 IPLD 文档并将它们作为 DAG-PB 发出，也不应该期望能够从其他编解码器（例如 [DAG-CBOR](https://ipld.io/docs/codecs/known/dag-cbor/)）中获取文档并将它们音译为 DAG-PB——DAG-PB 不是, 没那么灵活。

DAG-PB 主要出现在 IPFS 中，它是 Unixfsv1 数据序列化方式的支柱。

有关详细信息，请参阅 [DAG-PB 规范](https://ipld.io/specs/codecs/dag-pb/)。
## 快速比较
- 概论

	DAG-PB 不是通用的。只有非常特定的数据结构才能在 DAG-PB 中编码。[数据模型](https://ipld.io/glossary/#data-model) 中的非结构化数据不太可能在 DAG-PB 中进行编码，除非它具有与 DAG-PB 完全匹配的已知结构。

	请记住，[如互操作性部分所述](https://ipld.io/docs/codecs/known/dag-pb/#interoperability)，DAG-PB 是Protocol Buffers 的子集：它是 Protobuf 中的特定模式，不支持一般用户指定的 Protobuf。

	有关可在 DAG-PB 中编码的特定结构的详细信息，请参阅 [DAG- PB 规范：逻辑格式](https://ipld.io/specs/codecs/dag-pb/spec/#logical-format)。
- 互操作性

	DAG-PB 基于 Protocol Buffers，Protobuf 库已经无处不在，可以在多种语言中找到。如果您使用的语言还没有 IPLD 库，您可以使用其他 Protobuf 库编写 DAG-PB 交互，尽管这可能需要一些小心。

	DAG-PB 是 Protocol Buffers 的一个子集：它是 Protobuf 中的一个特定模式，并且还施加了一些并非所有 Protobuf 系统都会执行的约束。因此，如果使用其他 Protobuf 库制作您自己的 DAG-PB 交互，则有必要谨慎行事。
- 表现

	（标准性能警告：实现细节总是很重要；性能总是视情况而定；在假定序列化性能是关键瓶颈之前总是做基准测试；等等）

	DAG-PB 通常被认为是快速的。它是一种二进制的、长度为前缀的格式。这些特征通常与良好的表现相关。

	（在长度前缀格式中，字符串可以在不转义的情况下快速扫描，在不转义的情况下进行编码；可以预先进行分配；等等；所有这些都有助于提高速度和效率。）
- 人性化

	DAG-PB 不是很人性化。它是一种二进制的、长度为前缀的格式。虽然这些特性有助于其性能，但它们并不容易编辑。

	您通常不能手动编写或编辑 DAG-PB。
- 密度

	与以 DAG-JSON 或 DAG-CBOR 编码的相同数据相比，DAG-PB 的密度略高，因为它不为其主要结构编码密钥，否则这通常是冗余的。

	然而，这样做的代价是 DAG-PB 缺乏通用性，也缺乏自我描述。赋予 DAG-PB 这种密度的相同设计选择本质上意味着它可以用于更窄范围的数据结构。

	此密钥省略也不适用于存储在 DAG-PB 中的任何用户内容。DAG-PB 没有任何压缩功能。