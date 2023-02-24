# DAG-CBOR
状态：描述性 - 草稿

DAG-CBOR 支持完整的 [IPLD 数据模型](https://ipld.io/docs/data-model/)。

DAG-CBOR 使用 [简明二进制对象表示 (CBOR)](https://cbor.io/)数据格式，由 [RFC 8949](https://tools.ietf.org/html/rfc8949)（以前称为[RFC 7049](https://tools.ietf.org/html/rfc7049)）定义，它原生 [支持所有 IPLD 数据模型种类](https://ipld.io/docs/data-model/kinds/)。

## 格式
CBOR IPLD 格式称为 DAG-CBOR，以消除它与常规 CBOR 的歧义。大多数简单的 CBOR 对象都是有效的 DAG-CBOR。主要区别是：

- 标签 `42` 解释为 CID，不支持其他标签。
- map 只能由字符串键入。
- 应用额外的严格要求以确保规范的数据编码形式。请参阅下面的严格性。

## Links
在 DAG-CBOR 中，[link](https://ipld.io/docs/data-model/kinds/#link-kind) 是使用原始二进制身份 [Multibase](https://github.com/multiformats/multibase) 编码的 [CID](https://ipld.io/glossary/#cid) 的二进制形式。也就是说，Multibase 身份前缀 (0x0042) 被添加到 CID 的二进制形式之前，并且这个新的字节数组被编码为 CBOR 作为字节字符串（主要类型 2），并与 CBOR 标签相关联。

标签  42 在 [CBOR 标签注册表](https://www.iana.org/assignments/cbor-tags/cbor-tags.xhtml) 中关联为“[IPLD 内容标识符](https://github.com/ipld/cid-cbor/)”，并在 CBOR 中的 IPLD 内容标识符 (CID)中进一步定义。

包含 Multibase 前缀是出于历史原因，身份前缀不能省略。

## Map Keys
在 DAG-CBOR 中，Map Keys 必须是字符串，如 [IPLD 数据模型](https://ipld.io/docs/data-model/)所定义。不支持其他 map（例如 int），遇到时应拒绝。
## Strictness
DAG-CBOR 要求存在一种单一的、规范的方式来编码任何给定的数据集，并且编码形式不包含在往返解码/编码中可能被忽略或丢失的多余数据。

因此 DAG-CBOR 编解码器必须：

- 除了 CID 标签 (42) 之外，不使用其他标签。有效的 DAG-CBOR 编码器不得使用任何附加标签进行编码，并且有效的 DAG-CBOR 解码器必须拒绝包含附加标签的无效对象。
	- 这包括 [RFC 8949 第 3.4 节](https://tools.ietf.org/html/rfc8949#section-3.4) 中列出的任何明确定义的标记号，例如日期、大数字、大浮点数、URI、正则表达式和其他复杂或简单的值，无论它们是否 map 到 IPLD 数据模型。
- 使用 [RFC 8949 第 4.2 节](https://tools.ietf.org/html/rfc8949#section-4.2) 中定义的 “确定性编码 CBOR” 规则建议，mapkey 排序除外，它遵循 [RFC 7049 第 3.9 节](https://tools.ietf.org/html/rfc7049#section-3.9) 中定义的原始规则。因此，一个有效的 DAG-CBOR 编码器应该产生符合以下规则的编码形式，而一个有效的 DAG-CBOR 解码器应该拒绝不符合以下规则的编码形式：
	- 整数编码必须尽可能短。
	- 主要类型 2 到 5 中的长度表达式必须尽可能短。
	- 对于major type 6，标签号的表达式(只有42个)必须尽可能短。因此，对于有效的 DAG-CBOR，唯一可以出现的标记令牌是 `0xd82a`
		- 其中 `0xd8` 是“主类型6
		- 后面是8位整数”，`0x2a` 是数字 42。
	- 每个 map 中的 key 必须按字符串键的字节表示按长度优先排序，其中：
		- 如果两个键的长度不同，则较短的优先排序；
		- 如果两个键的长度相同，则（按字节）词法顺序中值较小的键排序较早。
	- 不支持不定长项，只能使用定长项。这包括字符串、字节、列表和映射。也不支持“break”标记。
- 唯一可用的主要类型 7 次要类型是用于编码 `Floats (minors 25, 26, 27)、True (minor 20)、False (minor 21)` 和 `Null (minor 22)` 的类型。
	- 不支持 True、False 和 Null 以外的[简单值](https://tools.ietf.org/html/rfc8949#section-2.1)。这包括使用 `True、False` 和 `Null` 以外的主要类型 7 编码的所有已注册或未注册的简单值。
	- 不支持未定义（次要23），因为它不是 IPLD [数据模型](https://ipld.io/docs/data-model/)的一部分。
- 浮点值必须始终以 64 位双精度形式编码，无论它们是否可以表示为半 (16) 或单 (32) 精度。
- [IEEE 754](https://en.wikipedia.org/wiki/Floating-point_arithmetic) 特殊值 `NaN，Infinity` 并且 `-Infinity` 不能被接受，因为它们没有出现在 [IPLD 数据模型](https://ipld.io/docs/data-model/) 中。因此，令牌 `0xf97c00( Infinity)、0xf97e00( NaN)` 和 `0xf9fc00( -Infinity)`、它们的 16 位、32 位和 64 位变体，以及任何其他被解释为这些值的 [IEEE 754](https://en.wikipedia.org/wiki/Floating-point_arithmetic) 字节布局，不应出现或被 DAG-CBOR 二进制形式。
- 编码和解码必须对单个顶级 CBOR 对象进行操作。正如 [RFC 8949 的第 5.1 节](https://tools.ietf.org/html/rfc8949#section-5.1) 针对流式应用程序所建议的那样，不允许或不支持背对背串联对象。编码的 DAG-CBOR 对象的所有字节必须解码为单个对象。IPLD 块中包含的无关字节，无论 CBOR 有效还是无效，都不得被接受为有效的 DAG-CBOR。

### 解码严格
由于历史数据的存在和积极使用，以及不合格编码器的存在和积极使用，DAG-CBOR 解码器可能会默认放宽严格性要求。可以为往返确定性是理想特性并且不需要与旧的、非严格数据向后兼容的系统提供严格选择加入。

特别是，为了允许与历史和其他松散编码数据的互操作性，可以放宽以下规则：

- map key 排序：可以按任何顺序接受 map 条目
- 整数编码不需要尽可能短
- 主要类型 2 到 5 的长度描述符不需要尽可能短
- 标签的表达42不必越短越好( 0xd82a)
- 浮点值可以表示为单精度和半精度

## 实现
### JavaScript
鼓励 JavaScript 用户使用 [@ipld/dag-cbor](https://github.com/ipld/js-dag-cbor/) 进行 DAG-CBOR 编码和解码。

[@ipld/dag-cbor](https://github.com/ipld/js-dag-cbor/)，用于[多格式](https://github.com/multiformats/js-multiformats/)遵守此规范，但有以下警告：

- 对解码强制执行一些严格性；整数编码和长度（主要类型 2 到 5）必须尽可能短，标签42只能表示为0xd82a. 但是，不强制执行映射键排序和浮点编码大小。
- [BigInt](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/BigInt) 与编码一起被接受 `Number`，但编码时遵循最小可能规则。当解码 JavaScript“安全整数”范围之外的整数时，BigInt将使用 a。

遗留 [ipld-dag-cbor](https://github.com/ipld/js-ipld-dag-cbor/) 实现遵循此规范，但有以下注意事项：

- 解码时不强制执行严格性；根据上面 decode-strictness 中列出的项目。
- 浮点值被编码为它们的最小形式，而不是总是 64 位。
- 目前接受数据模型之外的许多其他对象类型进行解码和编码，包括 `undefined`.
- [IEEE 754](https://en.wikipedia.org/wiki/Floating-point_arithmetic) 特殊值 `NaN,Infinity` 并被 `-Infinity` 解码和编码接受。
- JavaScript “安全整数” 范围之外的整数将使用第三方 [bignumber.js](https://github.com/MikeMcl/bignumber.js) 库来表示它们的值。

请注意，无法在 JavaScript 中清楚地区分整数和浮点数可能会导致浮点值的往返问题。请参阅 [IPLD 数据模型](https://ipld.io/docs/data-model/) 和下面关于[限制](https://ipld.io/specs/codecs/dag-cbor/spec/#limitations)的讨论，以进一步讨论 JavaScript 数字和有关使用浮点数的建议。

### go
鼓励 Go 用户使用 [go-ipld-prime](http://github.com/ipld/go-ipld-prime) 进行 DAG-CBOR 编码和解码。

- [go-ipld-cbor](https://github.com/ipfs/go-ipld-cbor)和[go-ipld-prime](http://github.com/ipld/go-ipld-prime)遵守此规范，但有以下注意事项：
	- 解码时不强制执行严格性；根据上面在[解码严格](https://ipld.io/specs/codecs/dag-cbor/spec/#decode-strictness)性中列出的项目。
	- [IEEE 754](https://en.wikipedia.org/wiki/Floating-point_arithmetic)特殊值 `NaN,Infinity` 并被 `-Infinity` 解码和编码接受。
- [cbor-gen](https://github.com/whyrusleeping/cbor-gen) 遵守此规范，但有以下注意事项：
	- 解码时不强制执行严格性；根据上面在[解码严格性](https://ipld.io/specs/codecs/dag-cbor/spec/#decode-strictness)中列出的项目。
	- 在编码时使用 [sort.Strings()](https://pkg.go.dev/sort#Strings) 按字母数字顺序对映射键进行排序。
### java
- [Peergos 的 Java IPLD](https://github.com/Peergos/Peergos/tree/master/src/peergos/shared/cbor) 遵守此规范，但有以下注意事项：
	- 解码时不强制执行严格性；根据上面在解码严格性中列出的项目。
	- 浮动被禁用。

## 限制
### JavaScript Numbers
DAG-CBOR 的用户如果希望他们的数据可能在某个时候被 JavaScript 使用或生成，则应该意识到该语言对其使用 DAG-CBOR 施加的限制，特别是关于数字的限制。

所有 JavaScript 数字，包括浮点数和整数，（使用 [Number](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Number) 原始包装器）在内部表示为 64 位 [IEEE 754](https://en.wikipedia.org/wiki/Floating-point_arithmetic) 浮点值（即双精度）。这种设计选择在 JavaScript 中的一些含义是：

- 纯整数类型和浮点数之间没有明确的区别，开发人员可能希望有这样的区别。
- 按照惯例，JavaScript 引擎和开发人员在表示整数时通常会省略小数点，以模拟整数，其中数字实际上并未存储为整数。 
- JavaScript 中可表示的最大和最小安全整数大小存在限制，比具有 64 位整数类型的语言的限制更大。`Number.MAX_SAFE_INTEGER (<sup>253</sup>- 1) `和 `Number.MIN_SAFE_INTEGER (-(<sup>253</sup> - 1))` 范围之外的数字无法安全地操作或检查，因为它们会产生 [IEEE 754](https://en.wikipedia.org/wiki/Floating-point_arithmetic) 表示法强加的舍入效应。
- 无法在 32 位范围之外执行对“整数”的本机位操作；较大的数字将被截断。

[@ipld/dag-cbor](https://github.com/ipld/js-dag-cbor/)支持 [BigInt](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/BigInt) 安全整数范围之外的值，而旧版 [ipld-dag-cbor](https://github.com/ipld/js-ipld-dag-cbor/) 使用第三方 [bignumber.js](https://github.com/MikeMcl/bignumber.js) 库来处理这些值。

这些限制对 DAG-CBOR 的影响是：

- 任何大于 `Number.MAX_SAFE_INTEGER` 或小于`Number.MIN_SAFE_INTEGER` JavaScript CBOR 解码器反序列化的整数都将作为 [BigInt](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/BigInt) 来自 [@ipld/dag-cbor](https://github.com/ipld/js-dag-cbor/) 或来自 [ipld-dag-cbor](https://github.com/ipld/js-ipld-dag-cbor/) 的 [bignumber.js](https://github.com/MikeMcl/bignumber.js) 包装器类型返回，这可能对用户来说是意想不到的，并且对下游代码。
- 由 JavaScript CBOR 编码器序列化的任何内容都 `Number` 依赖于整数检查（即 ` Number.isInteger(), roughly x % 1 === 0`）来确定它应该被编码为整数还是浮点数。
- 由 JavaScript CBOR 解码器反序列化的任何没有小数部分的浮点数对于 JavaScript 程序将无法与整数区分开来，并且如果最初由非 JavaScript 代码生成则可能不会往返到相同的字节。
- 任何 `Number` 大于 `Number.MAX_SAFE_INTEGER` 或小于 `Number.MIN_SAFE_INTEGER` 的值都无法正确检查其整数状态，因此无论它是整数还是具有小数部分，JavaScript CBOR 编码器都会将其编码为浮点数。在处理超出安全范围的整数时，[BigInt](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/BigInt) 应将其用于 [@ipld/dag-cbor](https://github.com/ipld/js-dag-cbor/) 以确保正确处理。