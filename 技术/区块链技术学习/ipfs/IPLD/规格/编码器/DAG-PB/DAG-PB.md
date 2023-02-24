# DAG-PB规范
## 状态：描述性 - 最终
DAG-PB 是一种 IPLD 编解码器，它使用[协议缓冲区](https://developers.google.com/protocol-buffers/)来描述可以对字节数组和关联的链接列表进行编码的二进制格式。它是[IPFS](https://ipfs.io/)编码结构化文件数据的主要手段，是 [UnixFS](https://docs.ipfs.io/concepts/file-systems/#unix-file-system-unixfs) 的编码数据载体。

DAG-PB 不支持完整的 [IPLD 数据模型](https://ipld.io/docs/data-model/)

## 实现
- JavaScript
	- [@ipld/dag-pb](https://github.com/ipld/js-dag-pb)
	
		与[多格式](https://github.com/multiformats/js-multiformats)兼容
	- [ipld-dag-pb](https://github.com/ipld/js-ipld-dag-pb)

		遗留实施
- go
	- [go-codec-dagpb](https://github.com/ipld/go-codec-dagpb)

		与 [go-ipld-prime](https://github.com/ipld/go-ipld-prime) 一起使用
	- [go-merkledag/pb](https://github.com/ipfs/go-merkledag/tree/master/pb) 

		遗留实现
	- [go-ipld-prime-proto](https://github.com/ipld/go-ipld-prime-proto)
		
		go-merkledag/pb 通过 [go-ipld-prime](https://github.com/ipld/go-ipld-prime)的只读接口

## 串行格式
DAG-PB IPLD 串行格式使用单个 protobuf 进行描述：

	message PBLink {
	  // 目标对象的二进制CID(没有多基前缀)
	  optional bytes Hash = 1;
	
	  // UTF-8字符串名称
	  optional string Name = 2;
	
	  // 目标对象的累积大小
	  optional uint64 Tsize = 3;
	}
	
	message PBNode {
	  // 引用其他对象
	  repeated PBLink Links = 2;
	
	  // 不透明的用户数据
	  optional bytes Data = 1;
	}
### Protobuf 严格性
DAG-PB 旨在为任何给定的数据集提供规范形式。因此，除了标准的 Protobuf 解析规则外，DAG-PB 解码器还应强制实施额外的约束以确保规范形式（在可能的情况下）：

- 消息中的字段 `PBLink` 必须按照上面 Protobuf 模式定义的顺序出现，在字段编号之后。具有无序字段的块 `PBLink` 应该被拒绝。（请注意，Protobuf 解码器通常会接受无序字段条目，这意味着 DAG-PB 规范比其他基于 Protobuf 的格式可能被视为典型规范要严格一些。）
- 消息中的字段 `PBNode` 必须按照上述 Protobuf 模式定义的顺序进行编码。请注意，此顺序不跟在字段编号之后。解码器应该接受任何一种顺序，因为 IPFS 数据以两种形式存在。
- 二进制形式的重复条目无效；具有重复字段值的块应该被拒绝。（请注意，Protobuf 解码器通常会接受二进制数据中的重复字段值，并将它们解释为对已设置字段的更新；DAG-PB 比这更严格。）
- 除了上面的 Protobuf 模式中出现的字段和线路类型之外，其他字段和线路类型都是无效的，包含这些的块应该被拒绝。（请注意，Protobuf 解码器通常会跳过与架构中的字段不匹配的每种消息类型中的数据。）

## 逻辑格式
当我们在数据模型级别处理 DAG-PB 内容时，我们将这些对象视为地图。

这种布局可以用IPLD 模式表示为：

	type PBNode struct {
	  Links [PBLink]
	  Data optional Bytes
	}
	
	type PBLink struct {
	  Hash Link
	  Name optional String
	  Tsize optional Int
	}
### 约束条件
- DAG-PB 数据块中的第一个节点将匹配该 `PBNode` 类型。
- `Data` 可以省略或长度为零或更多的字节数组。
- `Links`
	- 必须存在，即使是空的；二进制形式不区分空数组和省略值，在数据模型中我们总是实例化一个数组。
	- 编码时，元素必须按其 `Name` 值按升序排序 ，这些值按字节而不是字符串进行比较（另请参阅下面[有关排序链接的注释](https://ipld.io/specs/codecs/dag-pb/spec/#link-sorting)）。
		- `Names` 必须是唯一的或被省略。
- Hash
	- 即使 Hash 在 `optional` Protobuf 编码中，在创建新块或解码现有块时不应将其视为可选，省略 Hash 应被解释为坏块
	- 编码格式中的字节被解释为 CID 的字节，如果字节不能转换为 CID 则应将其视为坏块。
	- 数据以二进制形式编码为字节数组，因此解码器可能读取正确的二进制形式但无法将 `CID` 转换为 `Hash`，因此将其视为坏块。
	- 创建数据时，您可以使用标准数据模型概念创建 map ，只要它们恰好具有这些字段即可。如果存在其他字段，DAG-PB 编解码器将出错，因为无法对它们进行编码。
- 最新的 [JavaScript](https://github.com/ipld/js-dag-pb) 和 [Go](https://github.com/ipld/go-codec-dagpb) 实现都通过数据模型严格公开了这种逻辑格式，并且不支持像传统实现那样通过命名链接解析路径的替代方法（见下文）。

## 替代（传统）路径
虽然[逻辑格式](https://ipld.io/specs/codecs/dag-pb/spec/#logical-format)隐含地描述了一组机制，用于以严格的数据模型形式遍历和通过 DAG-PB 数据，但遗留实现提供了一种通过赋予 `Name` 在链接特权来解析路径的方法。

此替代路径作为此描述性规范的一部分在此处进行了介绍，但它是独立于数据模型开发的，因此没有得到很好的标准化。替代路径机制在实现之间有所不同，并且已从较新的实现中完全删除。

遗留的 [Go](https://github.com/ipfs/go-merkledag/tree/master/pb)和 [JavaScript](https://github.com/ipld/js-ipld-dag-pb)实现都支持使用链接名称的路径：/<name1>/<name2>/….

在传统的 Go 实现中，这是唯一的方法，这意味着不可能通过不命名其链接的节点。此外，数据部分和链接部分 `/` 元数据都无法通过路径访问。

在遗留的 JavaScript 实现中，还有一种通过数据路径的附加方法。它完全基于对象的结构，即

	/Links/<index>/Hash/…. 
这样您就可以直接访问 `Data、Links` 和 `size` 字段，例如 `/Links/<index>/Hash/Data`。

这两种路径方式可以结合使用，

- 因此您可以通过 `Data` 访问例如命名链接的字段 `/<name/Data`。
- 您还可以在单​​个路径中使用这两种方法，例如 `/<name1>/Links/0/Hash/Data` 或 `/Links/<index>/Hash/<name>/Data` 。

在 js-ipfs 中使用 DAG API 时，结构上的路径具有优先权，因此您将无法在名为 `Links `的命名链接上使用命名路径，您需要使用链接的索引。

最新的 [JavaScript](https://github.com/ipld/js-dag-pb) 和 [Go](https://github.com/ipld/go-codec-dagpb)实现都没有公开新的路径机制，而是严格遵守上述逻辑格式模式中描述的 IPLD 数据模型。

## 零长度块
零长度 DAG-PB 块是有效的，并且将被解码为具有 `nullData` 和空 `Links` 数组。

使用 SHA2-256 多重哈希，此块的 CID 为：

- CIDv1:

		bafybeihdwdcefgh4dqkjv67uzcmw7ojee6xedzdetojuzjevtenxquvyku
- CIDv0:

		QmdfTbBqBPQ7VNxZEYEj14VmRuZBkqFbiwReogJgS1zR1n

## 链接排序
DAG-PB 编码形式的列表 `Links` 必须按其 `Name` 值按升序排序，这些值按字节而不是字符串进行比较（另请参阅下面[有关排序链接的注释](https://ipld.io/specs/codecs/dag-pb/spec/#link-sorting)）。缺失值或 `Name` 空值被视为空字符串。排序应该是稳定的，以原来的顺序留下重复的 `Name`（或多个缺失或 `Name` 空值）。

排序不应应用于 DAG-PB 块的解码。在 DAG-PB 块中找到的链接顺序是它们以二进制形式出现的顺序，因此遍历这些链接遵循该顺序。

解码表单链接顺序的任何差异都会影响需要稳定顺序的遍历。

### go-merkledag 中的链接排序
v0.4.0 之前的 DAG-PB 块的旧版 [go-merkledag](https://github.com/ipfs/go-merkledag) 接口在通过和 API 读取[DecodeProtobuf()](https://pkg.go.dev/github.com/ipfs/go-merkledag#DecodeProtobuf) 和 [DecodeProtobufBlock()](https://pkg.go.dev/github.com/ipfs/go-merkledag#DecodeProtobufBlock) 时对解码块应用排序。

从 v0.4.0 到 v0.7.0 的 go-merkledag 版本将在执行某些操作（例如 `Size(),RawData()`和其他一些）时将反序列化块的链接与未排序的链接进行排序。

go-merkledag v0.7.0 及更高版本保持未排序的反序列化块的链接与未排序的链接，直到节点以某种方式发生变异，此时链接被自动排序。

有关详细信息，请参阅[此请求](https://github.com/ipfs/go-merkledag/pull/87)。