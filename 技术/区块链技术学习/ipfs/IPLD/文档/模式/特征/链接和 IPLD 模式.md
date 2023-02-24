# 链接和 IPLD 模式
IPLD 模式旨在描述以块为界的数据。值得注意的是，IPLD 模式有意与块边界无关——这对于包含验证的复杂性是必要的：您应该能够将模式应用于块并有足够的信息来进行快速、完整的验证。

种类 `link` 是模式中的一种 [类型](https://ipld.io/docs/schemas/features/typekinds/)。它相当直接地 map 到数据模型链接类型 （就像 `int` 类型类型本质上与 数据模型 `int`类型相同）。然而，模式链接也可以添加更多信息——我们稍后将在 链接目标类型提示部分看到。

链接定义了有效的块边界。例如：

	type Foo struct {
	  baz Int
	  boom Link
	}
这告诉我们存在一个 map（默认 struct 表示）包含一个 baz 整数属性和一个 boom 链接属性。通过这种方式，我们可以以与描述所有其他数据模型类型的位置相同的方式明确地说明期望链接的位置。

链接可能出现在 structS 中，作为 listS 的元素和 mapS 的值。unionS 如果它们也被分配给一个类型，它们也可能出现在中：

	type MyKeyedUnion union {
	  | Foo "foo"
	  | Bar "bar"
	} representation keyed
	
	type Foo struct {
	  froz Bool
	}
	type Bar link
这适用于和适用于 `envelope` unionS，也适用于`kinded` union，因为 `link` 它是一种可以区分的类型。`inline` unionS 然而，目前描述的是复杂类型（structS），因此不能直接描述 linkS，尽管包含 structS 的可能包含 linkS。

## 链接目标类型提示
在许多情况下，描述跨块边界发生的事情是有帮助的，即使这不能通过模式在每个块的使用中验证。出于 codegen 的目的并作为文档工具，我们可以在链接的另一侧提供有关数据形状的“提示”（以模式术语表示）。

我们使用&运算符将​​复合体转换 `type` 为链接，其中，出于块内验证的目的，我们希望它描述的内容是 link，但出于跨块遍历的目的，我们可以建议其他内容的一侧 link 是特定的 type。

在我们原来的 struct 例子中，我们可以建议  boom 的属性，Foo 当遵循时，将产生一个 Grop：

	type Foo struct {
	  baz Int
	  boom &Grop
	}
	
	type Grop struct {
	  description String
	  x Float
	  y Float
	  data [ Int ]
	}
出于验证的目的，它的工作 Foo 方式与我们最初的示例完全相同；我们仍然希望 boom 成为一个链接，并且无论如果点击该链接可能会发现什么，这种验证都可能成功。但我们现在提供了一个“提示”，我们希望在点击链接时，boom 我们应该找到一个可以用 Grop 描述的对象。

同样，keyed union 如果我们点击链接，我们可能会更复杂地描述我们期望找到的内容：

	type MyKeyedUnion union {
	  | Foo "foo"
	  | &Grop "bar"
	} representation keyed
	
	type Foo struct {
	  froz Bool
	}
IPLD 模式中没有用于隐式描述“可能是链接，也可能是 inline”数据结构的工具。然而，我们可以使用 kinded unions 明确地做到这一点，尽管对于当我们点击一​​个链接时我们可能会发现什么没有强有力的保证：

	type HashMapNode struct {
	  map Bytes
	  data [ Element ]
	}
	
	type Element union {
	  | HashMapNode map
	  | &HashMapNode link
	  | Bucket list
	} representation kinded

	# ... “Bucket”的进一步定义
在此示例中，根据 [HashMap](https://github.com/ipld/specs/blob/master/data-structures/hashmap.md) 规范，我们描述了一个 map 名为 的 `HashMapNode`，它包含一个 data 属性，该属性是 Elements 的列表。这些 Elements 可能是以下三种情况之一：

- amap 包含另一个 HashMapNode（树数据结构的内联子级）
- alink 假定导致一个 HashMapNode（链接的子级）
- 或一个 Bucket（此处未描述）包含 a list。

我们的验证器可以扫描 data 列表并检查每个元素是否是以下之一：`map,link` 和 `list`。`link` 我们没有对实际产生做出强有力的断言，但在加载和验证 `HashMapNode` 链接时，此类断言可能会内置到从此模式生成的代码中。

# 在模式中使用 ADL
	本文档需要核心贡献者的审查。使用的语言已经过时并且可能会造成混淆。
“高级数据布局”[(ADL)](https://ipld.io/docs/advanced-data-layouts/) 是 IPLD 模式中的一种特殊类型，它表示抽象节点，当与模式层以上交互时，它可以充当标准 IPLD 数据模型类型之一，但以完全不同的数据[种类](https://ipld.io/docs/data-model/kinds/)表示在数据模型层。高级布局允许我们使用转换逻辑为比数据模型允许的复杂得多的数据或比合理的块大小允许的数据大得多的数据提供一致的接口。

诸如 map、列表或字节数组之类的数据结构可以使用数据模型以新颖的形式实现，甚至可以透明地跨越块边界以呈现比标准数据模型种类可能更大和更复杂的数据，同时提供标准数据模型各自种类的 API 交互方式。

高级布局的示例用途包括：

- 非常大的字节数组，可能跨越许多块边界，但仍然提供标准 `bytes` 类型的接口。
- map 可以跨多个块进行分片，允许通过标准 `map` 类型接口进行高效访问，同时允许超大存储空间。
- 加密块机制能够在数据模型层（可能在 `byte` 或 `string` 形式中）编码和解码数据，同时在通过模式层访问时呈现为未加密类型。
- 新颖的表示格式，例如高精度或大数字或替代字符串编码格式。尽管语言层对数字种类的大小限制可能对大型或高精度数字格式提出挑战。

高级布局必然涉及一种逻辑形式来执行其数据转换。许多用例甚至需要与块加载和存储机制进行交互，例如遍历 `bytes` 跨块链存储的大型数组。目前对于此逻辑驻留在何处、如何加载或如何与 IPLD 模式定义关联没有正式要求。这个话题会继续发展。未来这种逻辑有可能通过 WebAssembly 嵌入到 IPLD 块中，这样高级布局由数据和读取它所需的逻辑来表示。有关其中一些主题的早期探索，请参阅 [#130](https://github.com/ipld/specs/issues/130) 。

## 基本模式定义和使用
在最基本的情况下，高级布局不需要任何属性。一个可以简单地用关键字 `advanced` 以类似的方式定义 `type`：

	advanced ROT13
然后它可以用作一个 `advanced representationfor a type`，我们可以从中推断出 `kind` 我们期望 。

	type MyString string representation advanced ROT13
类似地，实现分片的高级布局 `map` 可以定义并用作：

	advanced ShardedMap
	
	type MyMap { String : &Any } representation advanced ShardedMap
从这种用法中，我们可以推断 `ShardedMap` 可以（1）呈现一个熟悉的 `map` 种类接口和（2）将 `links` 存储为值（没有特定的 `“expectedType”` ），由标准数据模型 `strings` 引用。其他操作模式 `ShardedMap` 也可能是可能的（它可能能够存储其他值类型，或者它甚至可以作为一种类型，`bytes` 尽管它的名称！）。