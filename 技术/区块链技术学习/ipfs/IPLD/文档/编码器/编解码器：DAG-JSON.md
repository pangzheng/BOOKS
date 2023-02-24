# 编解码器：DAG-JSON
DAG-JSON 是一种将 IPLD 数据模型实现为 JSON 的编解码器，加上一些额外的编码链接约定，它通过声明 map 的某些特定结构并为其分配此含义来实现。DAG-CBOR 还使用 CBOR 标签添加了 “链接” 类型，使其与 IPLD [数据模型](https://ipld.io/glossary/#data-model) 保持一致。

DAG-JSON 是一种非常方便使用的编解码器，因为它与 JSON 具有互操作性，并且对读写都非常友好。我们有条件地推荐 DAG-JSON。不过，在使用它之前请注意一些注意事项：请参阅下面的 [一般性说明](https://ipld.io/docs/codecs/known/dag-json/#generality)。

DAG-JSON 作为调试格式特别方便。主要将协议设计为 [DAG-CBOR](https://ipld.io/docs/codecs/known/dag-cbor/)，然后使用 DAG-JSON 作为调试和开发格式，可以成为最终产品效率和开发人员生产力的高效组合。

有关详细信息，请参阅 [DAG-JSON 规范](https://ipld.io/specs/codecs/dag-json/)。
## 快速比较
- 概论

	DAG-JSON 支持大多数 IPLD [数据模型](https://ipld.io/glossary/#data-model)，但有几个重要的警告。
	
	不能安全地编码或解码带有键的map"/"——因为该空间是为链接编码保留的。
	具有非 unicode 字节的字符串可能无法表示——因为 JSON 没有指定除 unicode 之外的转义机制。
	有关实现应如何处理这些极端情况的详细信息，请参阅 [DAG-JSON 规范](https://ipld.io/specs/codecs/dag-json/)。
- 互操作性

	DAG-JSON 基于 JSON，而 JSON 库已经无处不在，可以在多种语言中找到。如果您使用的语言还没有 IPLD 库，您可以轻松地使用另一个 JSON 库。
- 表现

	（标准性能警告：实现细节总是很重要；性能总是视情况而定；在假定序列化性能是关键瓶颈之前总是做基准测试；等等）

	DAG-JSON 的解析和发送效率与 JSON 差不多……也就是说，它还不错，但它也将比任何二进制协议慢得多。
	
	DAG-JSON 相对人类可读：没有长度前缀，因此，字符串需要转义。这些特征意味着对性能的上限限制。
	
	（在长度前缀格式中，字符串可以在不转义的情况下快速扫描，在不转义的情况下进行编码；可以预先进行分配；等等；所有这些都有助于提高速度和效率。在没有此类信息的格式中，如 DAG-JSON，信息必须即时重新计算，这本来就很昂贵。）
- 人性化

	DAG-JSON 与 JSON 一样对人类友好。

	您可以轻松地手动编写或编辑 DAG-JSON。这与编辑常规 JSON 完全一样。

	与常规 JSON 的唯一区别是，如果你想对一个链接进行编码，你需要 [CID](https://ipld.io/glossary/#cid)，将其编码为 base58（或 base32，对于 CIDv0），然后在带有密钥的map中将其侧接"/"——所以它看起来像这样：{"/":"Qmfoo"}.
- 密度

	DAG-JSON 的密度与 JSON 大致相当。例如，如果一系列map重复了相同的键，那么这些键将以 DAG-JSON 序列形式重复。
	
	DAG-JSON 在编码字节时会遭受相当大的膨胀，因为它必须对它们进行 base64 编码。对于大量内容为字节的协议，不建议使用 DAG-JSON。考虑在这种情况下使用 [DAG-CBOR ，它可以更有效地编码字节](https://ipld.io/docs/codecs/known/dag-cbor/)。