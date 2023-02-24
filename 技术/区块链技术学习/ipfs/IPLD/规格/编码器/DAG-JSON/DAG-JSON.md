# DAG-JSON
状态：描述性 - 最终

DAG-JSON 支持完整的 IPLD [数据模型](https://ipld.io/docs/data-model/)，某些 map 布局有特定和小的例外，这些布局保留用于划分[links](https://ipld.io/docs/data-model/kinds/#link-kind)和[bytes](https://ipld.io/docs/data-model/kinds/#bytes-kind)。

DAG-JSON 使用 [JavaScript Object Notation (JSON)] 数据格式，由[RFC 8259](https://tools.ietf.org/html/rfc8259)定义。

## 格式
本机 JSON IPLD 格式称为 DAG-JSON 以消除它与常规 JSON 的歧义。大多数简单的 JSON 对象都是有效的 DAG-JSON。主要区别是：

- 通过特殊使用单键 ( "/") map 来支持字节和链接。
- "/" 在有限的情况下，不允许使用用于编码字节和链接的密钥以外的密钥进行 map。
- map 按键排序。

### 连载
编解码器实现 `MUST` 在编码数据时执行以下操作，以确保哈希一致地匹配相同的块数据。

- 按 (UTF-8) 编码表示对对象键进行排序，即使用字节比较
- 去除空白

这会产生最紧凑和一致的表示，这将确保产生相同数据的两个编解码器最终具有匹配的块哈希。

编解码器实施者在解码数据时不应强制执行此严格性以支持历史数据和非严格编码器生成的数据。但是，它们可能会为往返确定性是理想功能且不需要与旧的、非严格数据向后兼容的系统提供选择加入。

### 支持种类
JSON 本机支持除 Bytes 和 Link 之外的所有 [IPLD 数据模型种类](https://ipld.io/docs/data-model/kinds/)。

Bytes 和 Link 使用特定于 DAG-JSON 的扩展。它们以 map 的形式实现，其中单个键是斜杠 ( "/")，值包含种类的数据。

- 数字

	JSON 只有一种数字类型。许多动态类型编程语言（例如 `Python、Ruby、PHP`）在解析 JSON 时会区分整数和浮点数。JavaScript 没有，因为所有数字在内部都表示为 `IEEE 754` 浮点数。由可选前导符号 (-) 和仅数字组成的 JSON 数字被解析为整数，如果它包含小数点，则被解析为浮点数。对于 DAG-JSON，同样的方法用于表示整数和浮点数。

	没有小数部分的数据模型浮点数应该用小数点编码，因此在往返过程中可以与整数区分开来。（请注意，由于 JavaScript 仍然无法区分浮点数和没有小数部分的整数，因此此规则不会影响 JavaScript 编码或解码）。

	与流行的看法相反，JSON 作为一种格式支持大整数。只有 JavaScript 本身对它们有问题。`2^53-1`这意味着如果大于需要支持的整数，DAG-JSON 的 JS 实现不能使用本机 JSON 解析器和序列化器。

`Infinity,NaN` 和  `-Infinity` 不受 JSON 原生支持并且不受 IPLD 数据模型支持。

请参阅有关[数据模型中浮点](https://ipld.io/docs/data-model/kinds/#float-kind)的进一步讨论，包括在生成和使用内容寻址数据时尽可能避免浮点的建议。

- 字节

	Bytes 类型表示为一个对象，以 `"bytes"` key 和 Base64 编码的字符串为 value。Base64 编码是 [RFC 4648 第 4 节](https://tools.ietf.org/html/rfc4648#section-4)中描述的编码，没有填充。

	请注意，此规范的先前版本和某些实现对字节使用了 [Multibase](https://github.com/multiformats/multibase)前缀，这已从规范中删除，不应为 Base64 编码的字节添加前缀 `m`。

		{"/": { "bytes": String /* Base64 encoded binary */ }}
- 链接

	链接种类表示为基本编码的 CID。CIDv0 和 CIDv1 的编码方式不同。

	- CIDv1

		表示为 Multibase `Base32` 编码字符串。Base32 编码是 [RFC 4648 第 6 节](https://tools.ietf.org/html/rfc4648#section-6) 中描述的编码，没有填充，因此 Multibase 前缀是 `b`.
	- CIDv0

		以其唯一可能的 `Base58` 编码表示。Base58 编码是 [Base58 draft](https://tools.ietf.org/html/draft-msporny-base58)中描述的编码。

			{"/": String /* Base58 encoded CIDv0 or Multibase Base32 encoded CIDv1 */}

### 保留命名空间
具有第一个键 "/" 的 map 被视为 DAG-JSON 中的`保留命名空间`，因为它们用于表示字节和链接。有一些特殊规则限制某些数据形式在 DAG-JSON 中被正确编码。这些规则允许字节和链接的干净表示以及标记化解码器的高效操作。在检测到未正确编码链接或 map 的字节时，标记化解码器不需要缓冲和回溯超过 4 个标记。

保留命名空间中使用的两种形式是：

- CID

	单键为 “/” 的 map，值为字符串，必须包含有效的 Base58 字符串形式的 CIDv0 或 Base32 字符串形式的 CIDv1 。这样的 map，其字符串不能正确地表示这样的 CID，应该作为无效的 DAG-JSON 被拒绝。
- Bytes

	一个具有单键 “/” 的 map，其值是一个具有单键 “Bytes” 的 map，其值是一个字符串，必须包含一个有效的 Base64 编码字节数组。这样的结构，其字符串不能正确地表示 Base64 编码的字节数组，应作为无效的 DAG-JSON 拒绝。

### 在保留的名称空间中解析拒绝模式
以下格式的数据严格来说是无效的 DAG-JSON，编码器和解码器应该拒绝它们:

- 具有多个键的 map，其中第一个键是“/”，其值是字符串。

	例如
	
		{"/":"foo","bar":"baz"}

	- 如果存在 “/” 之前排序的键，则 map 是有效的

		例如
				
			{"0bar":"baz"，"/":"foo"}。
	- 当"/"项的值不是字符串时，map 是有效的

		例如
			
			{"/":true，"bar":"baz"}。
- map 其中第一个键是 “/”，其值是一个具有多个键的 map，其中内部 map 的第一个键是 “bytes”，其值是字符串。

	例如
	
		 {"/":{"bytes":"foo","bar":"baz"}}
	- 如果内部 map 中存在一个键在 "bytes" 之前排序，则该 map 是有效的
	
		例如
		
			{"/":{"abar":"baz"，"bytes":"foo"}}。
	- 当内部 map 的"bytes"项的值不是字符串时，map 是有效的

		例如
		
			{"/":{"bytes":true}，"bar":"baz"}。
- 具有多个键的 map，其中第一个键是“/”，其值是一个 map，其中内部 map 的第一个键是值为字符串的“bytes”。

	例如
	
		{":{“字节”:“foo”},“酒吧”:“记者”}

	- 如果存在一个键在 “/” 之前排序，map 是有效的

		例如
		
			{"0bar":"baz"，"/":{"bytes":"foo"}}。
	- 当内部 map 中的 "bytes" 项的值不是字符串时，map 是有效的

		例如
		
			{"/":{"bytes":true}，"bar":"baz"}。

没有任何机制可以转义采用这些形式的有效JSON数据。因此，建议在可能使用DAG-JSON的数据模型map中避免使用“/”键，以避免此类冲突。

## 实现
- JavaScript

	[@ipld/dag-json](https://github.com/ipld/js-dag-json)，用于[多格式遵守](https://github.com/multiformats/js-multiformats)此规范。

	旧版 [ipld-dag-json](https://github.com/ipld/js-ipld-dag-json) 实现遵循此规范，但有以下注意事项：

	- 上面保留的命名空间规则没有严格应用。解码具有字节和链接形式但在内部或外部 map 中具有附加条目的 map 将成功解码为字节或链接，但无关条目将被忽略。
	- 根据本规范的先前版本，字节使用其 Multibase Base64 前缀 `m` 进行编码。
- go

	[go-ipld-prime](https://github.com/ipld/go-ipld-prime) 遵守此规范。