# 创作 IPLD 模式
IPLD 模式可以用紧凑的、人性化的 [DSL](https://en.wikipedia.org/wiki/Domain_specific_language) 表示。IPLD 模式也可以自然地表示为 IPLD 节点图，通常以 JSON 形式呈现。人性化的 DSL 编译成这种 IPLD 原生格式。
## 基本
### 记录：`type` 和 `advanced`
IPLD 模式通常包含一组可选的相互依赖的类型。每个类型定义都以一行 `type` 开头的前缀开头，后面是类型的名称，然后是它的定义。另一种记录样式可选地存在于 IPLD 模式中，即高级数据布局。这些将 `type` 关键字替换为 `advanced` 并对其内容具有特定规则。更多内容请见下文。
## 换行符和空格
DSL 将换行符视为重要的，它们用于分解记录（`type` 和 `advanced` ）以及记录中的描述符。换行符的使用方式与;用重要换行符代替 C 风格换行符的编程语言类似。

多个换行符在解析过程中折叠成一个，因此换行符可用于适当的格式化和文档目的。也没有必要通过特定数量的换行符来分隔记录，尽管一个空行是典型的。

空白字符（制表符和空格）在解析过程中也被折叠成一个空格，因此可以在适当的时候用于格式化和文档目的。大多数不需要以换行符开头的标记应至少由一个换行符分隔。还有其他标记并不严格需要换行符（例如，`{String:Int}` 对于 Map 定义，其中 5 个标记可以连接，但也可以分开，`{ String : Int }`）。缩进不是记录组件描述符的严格要求，但很典型，因为它们可用于表达意图。

	type Foo struct {
	  a   Int
	  b   Int
	  msg Message
	}
	
	type Message string
在这个例子中：

- 非标点符号之间的空格是必需的（`typeMessagestring` 这是胡说八道！）
- 记录的每个组成部分之间至少 Foo 需要一个换行符，这样 `a, b` 和`msg`都在单独的行上。
- `a、b` 和的缩进 `msg` 是可选的，但有助于将这些项目的所有权表达给父记录。
- `a` 和 `b` 它们的类型描述符之间的额外空格 `Int` 是可选的，并用作格式的精确度以在 `Struct` 中排列类型。
- `Foo` 和之间的空行 `Message` 是可选的，但旨在帮助提高可读性。
- `{换行符和空格规则}`不严格，但惯例是使用本例中的位置和间距。

## 注释
在解析过程中忽略字符后一行中的所有字符。这允许全行注释和注释尾随 `Schema` DSL 令牌：

	#
	# This is a (pseudo)block comment
	#
	
	type Foo struct {
	  a Int # An inline comment
	  b Int
	  msg Message
	}
	
	# Another full-line comment
	type Message string
## 模式种类
有关此主题的更多信息，请参阅 [IPLD 模式类型](https://ipld.io/docs/schemas/features/typekinds/)。

模式种类具有匹配的标记，这些标记出现在整个 IPLD 模式中。根据上下文，标记可以是小写（例如 `int` ）或标题大写（例如 `Int`），也可以完全省略，因为它们可以可靠地推断出来。随着我们的继续，这将变得清晰。

- Null

	`null`  可能显示为 typedef，但有关于更改 `Null` 在模式中使用方式的语义的可能性的讨论。`nullable` 在 `Struct` 字段之外，它通常没有用。
- Boolean

	`Bool` 可能作为组件说明符或 `bool` typedef 出现。
- Integer

	 `Int`  可能作为组件说明符或 `int` typedef 出现。没有额外的整数大小或符号说明符（尽管这可能在未来作为代码生成的附件出现）。
- Float

	`Float` 可能作为组件说明符或 `float` typedef 出现。没有额外的大小或字节表示说明符（尽管这可能会在未来作为代码生成的附件出现）。
- String

	 `String`  可能作为组件说明符或 `string` typedef 出现。数据模型采用 `unicode`。特定的字符串编码也以表示形式出现，见下文。Union `stringprefix` `Representations Strategies`是一种特殊情况，指示前缀字符串指示后续字符的类型，请参见下文。
- Bytes

	`Bytes` 可能作为组件说明符或 `byte` stypedef 出现。字节数组长度没有额外的说明符，也无法指定单个字节。Union `bytesprefix` 表示类型是一种特殊情况，表示一个或多个字节的前缀字符串指示后续字节的类型，请参见下文。
- list

	`[Type]` 由 typedef 和 inline 组件规范的简写推断。Schema DSL 中未使用标记 “List”，所有列表都必须指定值类型（尽管联合在此提供了很大的灵活性）。
- map

	`{KeyType:ValueType}` 由 typedef 和 inline 组件规范的简写推断。Schema DSL 中不使用令牌 “Map”，所有 Maps 都必须指定键和值类型（尽管 Union 在这里提供了很大的灵活性）。
- link

	`&` 作为类型前缀的令牌用作链接的简写。指向无类型资源的通用链接使用特殊的 `&Any`，而可以找到预期类型的​​链接使用该类型名称作为提示机制，`&Foo`. 有关更多信息，请参见下文和 [IPLD 模式中的链接](https://ipld.io/docs/schemas/features/links/)。
- Union

	`union` 出现如下 `type` 和 `Union` 的类型名称。
- Struct

	`struct` 出现如下 `type` 和结构的类型名称。
- Enum

	`enum` 出现如下 `type` 和枚举的类型名称。
- Copy

	使用简写 `=` 表示复制类型，如 `type Foo = Bar` . 令牌 “Copy” 不会直接出现在 Schema DSL 中。

## 命名类型
类型名称只能包含字母数字 ASCII 字符和下划线。第一个字符必须是大写字母。应避免使用多个连接的下划线（它们应保留用于代码生成目的）。类型名称的严格正则表达式为：`[a-zA-Z][a-zA-Z0-9_]*`. 遵循约定的正则表达式是：`（[A-Z][a-zA-Z0-9_]*` 为简单起见，忽略多下划线规则）。

推荐首字母大写的驼峰式大小写。`_` 应谨慎使用下划线。` ThisIsRecommend, This_Not_So_Much, Thisisnotrecommended, neitherIsThis.`

类型名称在 Schema 中是唯一的，并且理想情况下在相关 Schema 文档中是唯一的；重叠名称通常不适合用于文档目的。某些形式的 Schema 种类标识符是被禁止的，那些不被禁止的形式应该避免，以避免混淆以用于文档目的。即 `Null, Boolean, Int, Float, String, Bytes` 严格不允许作为类型名称（它们已经是隐式类型名称），并且应避免使用它们的小写对应物和其他模式类型。

`类型名称应该用作文档工具`。如果长名称更有利于描述，则它们不需要很短。
## 命名标量类型 (typedefs)
非递归（标量）模式类型（布尔值、整数、浮点数、字符串、字节、链接）可能都显示为 typedef 类型。也就是说，可以为一种类型分配一个唯一的名称，并且该名称可以用于稍后在模式中代替该类型。多个唯一类型名称可能共享同一种类。

	type Foo string
	type Bar int
	type Boom {Foo:Bar}
在数据布局方面，这相当于：

	type Boom {String:Int}
（请注意，即使数据模型仅允许 map 的字符串键， `Foo`  也允许通过类型进行间接访问，因为它具有字符串表示形式。）

typedef 标量模式种类有很多原因：

- 文档

	在模式 DSL 中可以更轻松地记录独立类型。如果存在围绕 DSL 中无法表达的类型的附加规则，但模式的读者可能需要注意，这可能会有所帮助。您会在[schema-schema](https://ipld.io/specs/schemas/schema-schema.ipldsch) 中找到很多 typedef 这样的 。
- 突出重用

	在特定模式类型的重用值得注意的地方，命名它可能有助于表达意图。
- 代码生成

	命名类型的使用将对代码生成工具产生影响。从模式生成的代码可能需要在某些位置具有可识别的类型名称。

## 链接
IPLD 模式中的链接是一种特例。数据模型种类 “链接” 由以字符为  `&` 前缀的标记表示。令牌的其余部分应该是 `Any` 或类型的名称。

链接可以是 typedef'，`type Foo &Bar` 也可以 inline 显示：`type Baz {String:&Bang}`。

此外，类型名称不是可以直接针对底层数据进行测试的严格断言，它只是关于在架构链接指示的位置跟随 CID 标识的链接时应该找到什么的提示。当解析链接和解码节点时，可以在模式验证层之上的层应用这种预期类型的​​严格断言。

有关架构中的链接的更多信息，请参阅 [IPLD 架构中的链接](https://ipld.io/docs/schemas/features/links/)。

## inline 递归类型
标量类型（Boolean、Integer、Float、String、Bytes、Link）可能以 inline 形式出现或被类型定义。此外，Map 和 Link 类型都可以 inline 显示并作为它们自己的类型出现。其他模式类型（Struct、Enum、Union、Cop​​y）没有内联变体。

	type IntList [Int]
	
	type MapOfIntLists {String:IntList}
	
	type Foo struct {
	  id Int
	  data MapOfIntLists
	}
相当于：

	type Foo struct {
	  id Int
	  data {String:[Int]}
	}
与 typedef 的标量种类一样，这对代码生成和其他 API 与模式类型的交互有影响。自动生成的名称可以应用于在该 Map 中找到的 List 节点 `MapOfIntLists` 的类型，而不是具有显式名称和 `Foo->data` 。（例如，` Foo__dataType, Foo__data__valueType`，）。

提供 inline 工具是为了方便，但始终建议明确性高于权宜之计，包括这种情况，以提高模式的文档作用。通过命名 Map 和 List 元素，作者可以向用户表达意图并通过使用模式的工具提供清晰度。

## 表示
“表示”的概念是 IPLD 模式的关键组成部分，应该理解它以便创建和阅读有效的 IPLD 模式。

在数据模型中只有 9 种（`Null、Boolean、Integer、Float、String、Bytes、List、Map & Link`）。Schema 层增加了 4 个（`Union, Struct, Enum & Copy`）。这些不存在于数据模型中，并且对序列化格式不透明。相反，它们必须 “表示” 为基本数据模型种类。因此，Schema 层的每种数据类型都有一个“表示类型”。标量种类在数据模型层表示为相同种类（高级数据布局除外，见下文）。

在序列化和反序列化时，

- Struct 默认表示为 Map。Struct 添加了对键应用额外约束的能力，使用 Map 的值节点时发现的类型，某些键是否必须存在以及当它们不存在时要做什么。
- Enum 也有默认表示；如果未指定，则假定它们在序列化或反序列化时表示为字符串，但对枚举出现的节点的有效字符串有限制。
- Copy 类型是一种特殊情况，它复制复制类型的所有属性，但不包括其名称，包括表示形式。
- Union 没有默认表示，因为它们表达的概念通常以多种方式表示，因此在定义联合类型时必须提供表示。

某些模式类型具有替代表示“表示”，这些“表示”指示如何以序列化形式表示类型。这些表示中的大多数改变了类型的表示类型，但有些表示保留了相同的类型，只是改变了类型在该类型中的编码方式。可用于 Struct 类型的 `stringjoin` 和表示都将 Struct 的表示类型从默认的 Map 更改为 String。`stringpairs` 编码为单个 String 的方法对于两者都是不同的。表示 `stringjoin` 按顺序附加由分隔符（例如"v1,v2"）分隔的字段，而 `stringpairs` 表示包括字段名称，需要分隔的字段以及分隔的条目（例如 `"f1=v1,f2=v2"`）。同样，`listpairs` 和 `tuple` 结构表示都使用 List 表示类型，但使用不同的表示在 List 中进行编码。

要指定类型的表示，在主要类型定义之后提供关键字，然后是对该类型有效的表示名称。

例如，考虑这个结构：

	type Foo struct {
	  fieldOne nullable String
	  fieldTwo Bool
	}
我们可以将以下 JSON（使用 DAG-JSON 编解码器）解码为一种 `Foo` 类型：

	{
	  "fieldOne": "This is field one of Foo",
	  "fieldTwo": false
	}
Struct 也可以具有显式表达的默认表示：

	type Foo struct {
	  fieldOne nullable String
	  fieldTwo Bool
	} representation map
当未提供表示时，这两个描述符 `Foo` 在解析时是相同的，因为 `representation map` 对于结构来说是隐式的。

当我们提供表示时，`Struct` 也可以表示为 `List tuple`：

	type Foo struct {
	  fieldOne nullable String
	  fieldTwo Bool
	} representation tuple
当在数据层遇到预期 `Foo` 此变体的 Map 时，将发生错误或验证失败。相反，这个 Struct 的数据是一个包含两个元素的简单列表，第一个是 String，第二个是 Bool。在 JSON 中，这可能看起来像：

	[ "This is field one of Foo", false ]
可以在 [Representations Strategies](https://ipld.io/docs/schemas/features/representation-strategies/) 中找到可用 Representations Strategies 及其类型的完整列表，这些表示可以为各种 Schema 类型提供。

### 表示参数
一些 `Representations Strategies` 具有可以提供的附加参数，而一些具有必需的参数，这些参数是正确塑造类型表示所必需的。提供表示参数的方法有两种：在`representation` 块内用于通用参数和在括号中与类型字段相邻的内联，其中表示参数特定于字段。

一般表示参数
我们的 `Foo` 具有表示的结构 `tuple` 可以通过提供通用 `fieldOrder` 参数以替代字段顺序序列化 ：

	type Foo struct {
	  fieldOne nullable String
	  fieldTwo Bool
	} representation tuple {
	  fieldOrder ["fieldTwo", "fieldOne"]
	}
JSON 中此类类型的序列化可能显示为：

	[ false, "This is field one of Foo" ]
Structs 的表示 `stringjoin` 有一个必需的参数，join.  此参数没有默认值，因此在 stringjoin 没有它的情况下指定 Struct 的 Schema 是无效的：

	type Foo struct {
	  fieldOne nullable String
	  fieldTwo Bool
	} representation stringjoin {
	  join ":"
	}
此表示形式 `Foo` 将序列化为单个 String 节点：

	"This is field one of Foo:false"
Structs 的这种表示有局限性，因为连接字符没有转义机制，因此应谨慎使用。类似的限制适用于 `stringpairs` map 表示。有关此类限制的更多详细信息，请参阅[Representations Strategies](https://ipld.io/docs/schemas/features/representation-strategies/)。
### 字段特定的表示参数
主 `type` 声明块中的内容（在 opening {和 closing之间}）旨在将类型表示为面向用户的概念，包括字段的基数。但是，显示在各个字段旁边的括号 ((, ))中的内容是此规则的例外。此内容是特定于字段的表示参数。也就是说，这些括号内的参数通常属于块下方，representation 因为它涉及与序列化形式的交互。它出现在字段旁边，主要是为了避免重复重新声明块中的字段 representation。

Structs 的两个常见的特定于字段的表示参数是 `implicit` 和 `rename` ：

	type Foo struct {
	  fieldOne nullable String (rename "one")
	  fieldTwo Bool (rename "two" implicit "false")
	}
将类型声明与序列化形式表示详细信息分开的更清晰的声明可能将其呈现为：

	# 这不是有效的 IPLD 模式，但它是为了说明如何避免额外的冗长
	
	type Foo struct {
	  fieldOne nullable String
	  fieldTwo Bool
	} representation map {
	  fields {
	    fieldOne rename "one"
	    fieldTwo rename "two" implicit "false"
	  }
	}
在我们的示例中，我们可以看到`nullable` 与 `rename` 和 `implicit` 相比，这是字段的不同参数。这是因为 `nullable` 影响面向用户的 API 的形状`Foo`，`rename` 和`implicit` 仅影响 `Foo ` 的序列化（表示）对用户有效隐藏。

有关此类问题的讨论以及[对值基数](https://ipld.io/docs/schemas/features/typekinds/#value-type-modifiers)的影响，请参阅值类型修饰符。

参数 `rename` 指定在序列化和反序列化时，字段具有与架构中存在的名称不同的替代名称。`implicit` 指定，当序列化形式中不存在时，该字段应具有特定值。

回想一下我们的原始序列化表格 `Foo`：

	{
	  "fieldOne": "This is field one of Foo",
	  "fieldTwo": false
	}
使用上面的 `rename` 和 `implicit` 参数，相同的数据将被序列化为：

	{
	  "one": "This is field one of Foo"
	}
有关 [fields-with-implicit-values](https://ipld.io/docs/schemas/features/typekinds/#fields-with-implicit-values)的更多信息，请参见具有隐式值的字段`implicit`。在同一文档中，您还将找到有关组合`nullable` , `optional` 及`implicit `其限制的讨论。

每当一个值出现在表示参数中时，无论类型如何，它都必须被引用。在我们上面的示例中，`implicit "false"` 引用了一个 Bool 参数。这将根据上下文进行适当的解释，在这种情况下，很明显引用值的类型应该是 Bool。

字段参数的另一个示例是 `int`  枚举的表示，其中字段参数是必需的：

	type Status enum {
	  | Nope  ("0")
	  | Yep   ("1")
	  | Maybe ("100")
	} representation int
在这种情况下，我们将序列化形式的 `Int` 值 map 到三个 `Enum` 值。另请注意，这些值再次被引用，但将被适当地解释为整数，因为上下文清楚地表明了这一点。

## structs
Struct 的基本 DSL 形式具有以下结构：

	type TypeName struct {
	  field1Name Field1Type
	  field2Name Field2Type
	  ... etc.
	}
其中 `TypeName` 是该类型的唯一名称，并遵循上述命名规则。字段名称遵循与类型命名相同的规则，除了允许使用小写首字符并鼓励使用常规形式。所有字段都有一个类型，该类型应该是现有的隐式模式类型之一（`Int, String`等），或者作为命名类型出现在文档的其他地方。字段类型可以是递归的，因为它们可以引用父类型，表示嵌套数据结构（显然，这样的嵌套数据结构必须具有可空或可选元素，以防止它必然无限递归）。

结构必须始终有一个主体，由 `{, }` 包围。字段必须以换行符分隔，并且为了清楚起见应该缩进。

`representation` Structs 的表示是默认 map 的，所以可以省略。可以在有关 [Representations Strategies](https://ipld.io/docs/schemas/features/representation-strategies/) 的功能详细信息页面中找到更多详细信息。

字段表示参数在存在时显示在括号中，需要附加通用参数的表示显示在 `representation` 由`{,}`括起来的单独块中。例如，同时具有字段表示参数和一般表示参数的 Struct：

	type Foo struct {
	  fieldOne nullable String (rename "one")
	  fieldTwo Bool (rename "two" implicit "false")
	} representation stringpairs {
	  innerDelim "="
	  entryDelim ","
	}
导致序列化形式，例如：

	"one=This is field one of Foo,two=true"
有关更多详细信息，请参见下文，以及有关[Representations Strategies](https://ipld.io/docs/schemas/features/representation-strategies/) 的stringpairs功能详细信息页面。

Structs 的有效 [Representations Strategies](https://ipld.io/docs/schemas/features/representation-strategies/) 是：

- map
- tuple
- stringpairs
- stringjoin
- listpairs

有关这些 [Representations Strategies](https://ipld.io/docs/schemas/features/representation-strategies/) 的更多详细信息，包括 map 到的数据模型类型及其各种参数，请扫描有关[Representations Strategies](https://ipld.io/docs/schemas/features/representation-strategies/)的功能详细信息页面。

## Enums
枚举用于指示不同的、固定的值列表。IPLD 模式中的枚举具有字符串表示类型，默认情况下使用值标记作为序列化值。

	type Status enum {
	  | Nope
	  | Yep
	  | Maybe
	}
	
	type Response struct {
	  timestamp Int
	  status Status
	}
在这个例子中， 这里的 `Status` 被用作 `status` 中的字段 `Response` Struct ，我们希望找到一个序列化形式的字符串，它是 `"Nope","Yep"` 或 `"Maybe"` 之一。此字符串值不是通过通过此模式交互的 API 呈现的，而是通过特殊标记呈现的 `Nope，Yep` 并且 `Maybe` 可能会被使用。Codegen 会将这些值呈现为不同的类型，这些类型可以在与 `status` 字段交互时传递给实现 `Response` 的结构`/`类。

序列化的字符串可能与值不同：

	type Status enum {
	  | Nope ("Nay")
	  | Yep  ("Yay")
	  | Maybe
	}
在数据模型层的字符串和 API 可能在架构层使用的令牌之间创建差异。

可以指定枚举的 `int ` 替代`Representations Strategies`。使用 `int Representations Strategies`，值被序列化和反序列化为数据模型 Int，但枚举值标记在模式层呈现：

	type Status enum {
	  | Nope  ("0")
	  | Yep   ("1")
	  | Maybe ("100")
	} representation int
再次注意 Int 值在字段表示括号中被引用，在解析时它们将被解释和验证为整数，因为 `Representations Strategies` 的上下文清楚地表明了 `int` 这一点。

可以在有关 [Representations Strategies](https://ipld.io/docs/schemas/features/representation-strategies/)  的功能详细信息页面中找到更多详细信息。

## Unions
## Unions 简介：Kinded Unions
IPLD 模式 Unions 描述了节点的各种方式，这些节点可能是多种类型或形式之一。考虑包含以下数据的节点，可能作为信号协议的一部分：

	{
	  "msg": "Something bad happened",
	  "payload": "ERROR"
	}
还有一种也可以接受但表示不同状态和含义的替代形式：

	{
	  "msg": "All good",
	  "payload": {
	    "percent": 0.6,
	    "last": "61626378797a"
	  }
	}
在此示例中，我们有一个可以表示为 Struct 的 Map，因为它只有两个字段，但该字段 `payload` 没有稳定的种类，因此我们不能使用任何现有的 Schema 类型来表示该字段类型。相反，我们可以引入一个 Union 并且可以根据不同的可接受形式采用不同的形式。

IPLD 模式旨在提高效率，因此区分 Unions 类型的能力仅限于我们在当前节点上可以找到的内容。也就是说，我们无法检查节点是否具有采用特定形式的子节点并将其用作鉴别器（例如检查 Map 的键或值）。模式必须能够在数据与预期形式不匹配的被检查节点处验证失败。

在我们的示例中，找到的类型的鉴别符 `payload` 是存在的节点类型。它是 String 类型或 Map 类型。我们可以根据这条信息立即判断类型。

我们的数据模式可以写成：

	type Message struct {
	  msg String
	  payload Payload
	}
	
	type Payload union {
	  | Error string
	  | Progress map
	} representation kinded
	
	type Error string
	
	type Progress struct {
	  percent Float
	  last String
	}
我们的 `Payload` Union 可以理解为 “其中之一Error 或 Progress”，如果 "payload" 可以采用不同的形式，则可以包含其他元素。所有 Union 都需要声明一个`Representations Strategies`，没有默认表示。在这种情况下，我们指定了 `kinded` 表示，因此我们选择通过检查数据模型层中存在的种类来区分类型。如果我们在数据模型层找到一个字符串，那么我们可以安全地假设它是一个 `Error`. 如果我们找到一个 Map 那么我们假设它是一个 `Progress` 类型但我们必须继续对其进行验证`Progress `并检查 Map 是否具有所需的两个元素，但此时验证工作 `Payload` 已完成，它只需要检查是否存在字符串或map 。

## Union 歧视的局限性
在 IPLD 模式中创作 Union 有助于揭示快速验证允许变化的数据的一些局限性。如果我们扩展我们的示例并引入另一种可接受的形式，"payload" 我们可以看到这种快速区分的能力如何失效并需要进行子内容检查以进行区分：

	{
	  "msg": "Ping",
	  "payload": {
	    "ts": 1572935564043,
	    "nonce": "424f524b"
	  }
	}
我们引入了一种新的消息类型，但由于我们的新类型也是 Map，因此失去了基于种类进行区分的能力。适应这种额外有效载荷类型的模式是可能的，但会迫使歧视的负担和数据的消费者以及一些额外的验证负担：

	type Message struct {
	  msg String
	  payload Payload
	}
	
	type Payload union {
	  | Error string
	  | ProgressOrPing map
	} representation kinded
	
	type Error string
	
	type ProgressOrPing struct {
	  percent optional Float
	  last optional String
	  ts optional Int
	  nonce optional String
	}
现在，此类模式的用户必须进行自己的现场检查，以确定是 `Progress` 或 `Ping`进度消息还是 `ping`。此外，确保 `percent ` 和 `last ` 都存在或  `ts` 和 `nonce` 都存在的负担留给了用户架构层在这里无能为力。此场景中存在的权衡是通过检查节点的子节点来验证节点。这种类型的数据在现实世界中很常见，但 IPLD 模式鼓励更好的数据形状设计，以便通过明确区分存在此类差异的地方进行快速验证。

## 替代歧视表示
如果我们正在为我们的示例协议设计数据布局（而不是消耗我们无法控制其设计的东西），我们可以选择一种允许更有效区分的替代表示。union 允许五种不同的代表表示，允许不同类型的歧视。

- 键控 Keyed

	通过让我们的 "payload" 对象包含一个特定的键来区分有效负载的类型，我们可以使用一个 `keyed` Union：

		{
		  "msg": "Something bad happened",
		  "payload": {
		    "error": "ERROR"
		  }
		}
		{
		  "msg": "All good",
		  "payload": {
		    "progress": {
		      "percent": 0.6,
		      "last": "61626378797a"
		    }
		  }
		}
		{
		  "msg": "Ping",
		  "payload": {
		    "ping": {
		      "ts": 1572935564043,
		      "nonce": "424f524b"
		    }
		  }
		}
	我们现在可以使用以下模式轻松处理此数据：

		type Message struct {
		  msg String
		  payload Payload
		}
		
		type Payload union {
		  | Error "error"
		  | Progress "progress"
		  | Ping "ping"
		} representation keyed
		
		type Error string
		
		type Progress struct {
		  percent Float
		  last String
		}
		
		type Ping struct {
		  ts Int
		  nonce String
	}
	我们的 `Payload` union 现在有了 `keyed` 代表表示。此表示意味着 `Payload` 将具有 Map 表示类型并且该 map 将需要恰好具有用于区分存在的类型的各种键之一。在 Schema DSL 的语法上，`Payload` 现在在类型旁边列出引号字符串键而不是以前 `kinded` 的 Union 类型——这些是将在映射中看到的区别值。

	此类数据的验证现在可以检查这些键中的每一个是否存在，恰好存在其中一个，然后将验证移交给在该键的值中找到的节点处的预期类型。如果 "error" 找到一个键，它将继续进行验证， Error 假设该节点是一个字符串。如果 "progress "找到一个键，它将继续验证它是否在值节点上找到了一个 Map 并且它是否与类型匹配Progress，等等。

- Envelope

	一种类似于 `keyed ` 的表示，但更明确并允许保留节点 "payload" 是 `envelope` `Representations Strategies`。使用此表示，我们希望类型将作为 Map ( "payload") 的固定键的值出现，但我们可以通过检查 Map 中另一个键的值来区分要找到的数据类型：

		{
		  "msg": "Something bad happened",
		  "envelope": {
		    "tag": "error",
		    "payload": "ERROR"
		  }
		}
		{
		  "msg": "All good",
		  "envelope": {
		    "tag": "progress",
		    "payload": {
		      "percent": 0.6,
		      "last": "61626378797a"
		    }
		  }
		}
		{
		  "msg": "Ping",
		  "envelope": {
		    "tag": "ping",
		    "payload": {
		      "ts": 1572935564043,
		      "nonce": "424f524b"
		    }
		  }
		}
	该表示导致有效负载数据位于文档中的可预测位置，以及鉴别器值位于文档中的可预测位置，但文档有效负载部分的结构有所不同。

	我们的模式现在可以采用以下形式：

		type Message struct {
		  msg String
		  envelope Payload
		}
		
		type Payload union {
		  | Error "error"
		  | Progress "progress"
		  | Ping "ping"
		} representation envelope {
		  discriminantKey "tag"
		  contentKey "payload"
		}
		
		type Error string
		
		type Progress struct {
		  percent Float
		  last String
		}
		
		type Ping struct {
		  ts Int
		  nonce String
	}
	此 `envelope` `Representations Strategies`需要参数 `discriminantKey` 和 `contentKey` 。告诉 `discriminantKeySchema` 鉴别器值的键，而鉴别器值列在 Union 的类型旁边（在这种情况下，与我们在上面的 Union 示例中使用的值相同keyed）。

- inline

	inline`Representations Strategies`将嵌套结构拉入当前节点，而不是向下导航到子节点以按照先前的联合`Representations Strategies`解释构成类型。类型之间的区分使用 ` discriminantKey` , 也在当前节点中。这必然意味着当前节点必须是地图表示类型，并且 Union 的组成类型也必须具有地图表示类型。

	我们的示例必须扩展，以便 `Error` 可以从 map 表示中提取类型：

		{
		  "msg": "Something bad happened",
		  "union": {
		    "tag": "error",
		    "message": "ERROR"
		  }
		}
		{
		  "msg": "All good",
		  "union": {
		    "tag": "progress",
		    "percent": 0.6,
		    "last": "61626378797a"
		  }
		}
		{
		  "msg": "Ping",
		  "union": {
		    "tag": "ping",
		    "ts": 1572935564043,
		    "nonce": "424f524b"
		  }
		}
	对于联合中只有一个字段的结构类型（如上面的第一个示例数据），这看起来与信封联合非常相似......除了注意我们联合的表示定义中没有 - 所以另一个的`contentKey` 字符串该示例中的映射键来自结构的字段名称！随着包含的类型获得更多字段，union 的行为 `inline` 变得更加清晰：标签字段总是紧挨着其他 map 键。

		type Message struct {
		  msg String
		  union Payload
		}
		
		type Payload union {
		  | Error "error"
		  | Progress "progress"
		  | Ping "ping"
		} representation inline {
		  discriminantKey "tag"
		}
		
		type Error struct {
		  message String
		}
		
		type Progress struct {
		  percent Float
		  last String
		}
		
		type Ping struct {
		  ts Int
		  nonce String
		}
	与之前的 Union 相比，此 Schema 呈现的接口有所调整，Error 现在是一个带有字段的 Struct message。

### 字符串的字符串前缀联合
存在用于处理 String 类型的特殊情况联合。在节点包含字符串的情况下，我们可能希望在应用程序层区分该字符串的两种不同用途。例如，此类前缀鉴别器在配置选项方案中很常见。Stringprefix unions 重新解释一个字符串，去掉匹配的前缀，只向应用层呈现匹配的类型和剩余的字符串类型。高级用法还可能涉及更高级别的类型，这些类型使用在匹配类型（例如 map ）之上分层的字符串`Representations Strategies`stringjoin。

	type Username string
	
	type Credentials struct {
	  credType String
	  credToken String
	} representation stringjoin {
	  join ":"
	}
	
	type Authorization union {
		| Username "user:"
		| Credentials "auth:"
	} representation stringprefix
通过声明 `stringprefix` unions，我们指定匹配节点的字符串的第一个字符 `Authorization` 将区分哪个 `type` 公钥。那些第一个字符将被切掉并预期为 `user:` 或 `auth:`，然后字符串的其余部分将被提取并封装在 either 中 `Username` 或根据鉴别器前缀进一步解码为 `Credentials` 类型（通过进一步拆分）。
### 字节的字节前缀联合
存在用于处理字节类型的特殊情况联合。在节点包含字节数组（字节类）的情况下，我们可能希望在应用程序层区分该字节数组的两种不同用途。
	
例如，考虑两种不同的编码方案，我们在其中存储对于每种编码方案不同的“关键”字段。出于实际目的，它们都是字节数组，但在应用程序层，将它们分成不同的形式会有所帮助，也许这样我们就可以针对给定的编码方案获取预期的密钥类型做出简单的断言。提取不同的形式并在可能影响此类决策的模式中命名它们还有其他文档清晰度的好处。
	
	type Authorization struct {
	  key PublicKey
	  keySize Int
	}
	
	type PublicKey union {
	  | RsaPubkey "00"
	  | Ed25519Pubkey "01"
	} representation bytesprefix
	
	type RsaPubkey bytes
	type Ed25519Pubkey bytes
通过声明一个 `bytesprefix` 联合，我们指定在`Authorization` 的关键节点上找到的字节数组的字节将区分哪个公钥是。那些第一个字节将被期望为 `0x00` 或 `0x01`，然后字节数组的其余部分将被提取并封装在 RsaPubkey 或 Ed25519Pubkey 中，具体取决于鉴别器字节。
	
鉴别符必须至少有一个字节长并且不能冲突。它们表示为格式正确的十六进制字符串，仅使用大写字符。
## Copy
Copy Schema 类型是一种特殊情况，它提供了一种将一个命名类型的定义复制到一个新名称中的机制。它=在新类型的名称之后使用令牌，后跟要复制的类型的名称。无法复制未命名（匿名）类型。
	
	type Ping struct {
	  ts Int
	  nonce String
	}
	
	type Pong = Ping
这个例子在 Schema 层之上的交互方面严格等价于下面的例子：
	
	type Ping struct {
	  ts Int
	  nonce String
	}
	
	type Pong struct {
	  ts Int
	  nonce String
	}
Schema 工具和 Schema 的具体化形式保留了一种 copy 标记，但使用 Schemas 的工具应将此标记视为对正在复制的命名类型的间接访问，并将该类型的整个定义复制到新名称。

Copy 类型是为了方便而提供的，并且在指出类型之间的关系方面也应该被证明是有益的。

## 高级数据布局
高级数据布局 (ADL) 是一种将模式处理分解为自定义逻辑的机制，在这种情况下，此类逻辑无法在模式中表达，但与模式种类的连接可能会有所帮助。

ADL 在 Schema 意义上不被视为types，相反，它们伪装成类型，或者更具体地说，在某些条件下使用时具有伪装成 Schema 种类的能力。

ADL 的声明类似于声明 type 但只需要一个名称：

	advanced ROT13
一旦在模式中声明为实体，名称（`ROT13` 在本例中）可以用作模式中其他地方的表示。我们这样做是在 `representation advanced` 后面跟上名字：

	type MyString string representation advanced ROT13
结合这个 type 和 advanced 定义，我们声明在 Schema 层之上存在一些标记 ROT13 为能够代表数据模型层交互的逻辑 `MyString`，并为此提供标准的 String 类接口。

ADL 逻辑如何连接到模式工具将是特定于语言和工具的。出于模式创作的目的，advanced 可以将定义和用法视为一种机制，以打破所执行的标准数据模型到模式处理，而是在该流程中为所讨论的特定节点插入自定义逻辑这样它就变成了 `Data-model-to-ADL-to-Schema`。

与数据模型的交互也留给了 ADL，因此它不限于使用特定节点。相反，它可以使用任意数量的节点（或没有节点！），甚至可以以不透明的方式遍历链接。ADL 示例的另一个示例提供了这方面的示例。在这种情况下，我们声明了一个分片 Map 类型，它可用于扩展到非常大的 Map，因此包括多个独立的块：

	advanced ShardedMap
	
	type MyMap { String : &Any } representation advanced ShardedMap
MyMap 在这种情况下，为了 Schema 的其余部分，我们声明了一种被视为 Map 类型的类型，并在 Schema 层之上呈现。同时，我们插入了自定义逻辑，标记为ShardedMap，负责处理向此类模式的用户呈现标准 Map 类型所需的解码/编码和遍历。

`representation advanced 目前仅适用于 Map、List 和 Bytes 种类`。将来可能会考虑其他用例（例如上面假设的 String 类型）。

有关[高级数据布局](https://ipld.io/docs/advanced-data-layouts)的更多详细信息，请参阅高级布局文档；有关在使用 IPLD 模式时如何使用它们的更多详细信息，请参阅[模式中的指示 ADL ](https://ipld.io/docs/schemas/features/indicating-adls/)。
## Markdown 中的模式
IPLD 模式旨在充当文档角色以及编程声明角色。在此文档角色中，内联注释 ( #) 有助于扩展带有解释的声明，但扩展此文档形式以将 IPLD 模式嵌入到可消费的 Markdown 中也是可能的。当在具有正确语言标记的代码块中嵌入 Markdown 时，IPLD Schema 工具可以接受 Markdown 文件并仅提取它找到的那些 IPLD Schema 部分，代替独立的 Schema 文件。

在 Markdown 中嵌入 IPLD Schema 声明时，使用带有语言标记的代码块 ipldsch，即：


	```ipldsch
	type Foo struct {
	  a   Int
	  b   Int
	  msg Message
	}
	
	type Message string
	```
在 Markdown 文档中找到的任何此类块都将被提取并拼接在一起以形成单个架构文档。

此外，对于足够复杂的 Schema 声明，还可以跨多个 Markdown 文档执行此过程。当 IPLD Schema 工具提供 Markdown 文件列表时，它将提取ipldsch块并将它们全部拼接在一起，并假设它们包含一个独立的 Schema 文档。