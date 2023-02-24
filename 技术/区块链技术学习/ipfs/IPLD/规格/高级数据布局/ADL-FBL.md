# ADL-FBL
- FBL ADL

	灵活字节布局的缩写——提供[字节](https://ipld.io/docs/data-model/kinds/#bytes-kind)接口，同时在内部对数据进行分片
- FBL ADL 

	可用于在 IPLD 中存储非常大的二进制 blob，同时将它们分解为易于传输和单独校验和的小块
- FBL ADL

	包括足够的元数据和结构来支持合理有效地查找操作
- FBL ADL 

	不是对称的：它指定了如何查找和读取其组成数据，但没有规定数据是如何拆分的。

## 规范：灵活的字节布局
### 状态：规定 - 草案
Flexible Byte Layout 是一种用于表示二进制数据的高级布局。

它足够灵活，可以支持非常小和非常大（多块）的二进制数据。

	type FlexibleByteLayout union {
	  | Bytes bytes
	  | NestedByteList list
	  | &FlexibleByteLayout link
	} representation kinded
	
	type NestedByteList [ NestedByte ]
	
	type NestedByte union {
	  | Bytes bytes
	  | NestedFBL list
	} representation kinded
	
	type NestedFBL struct {
	  length Int
	  part FlexibleByteLayout
	} representation tuple
`FlexibleByteLayout` 是数据结构的入口点/根，并使用潜在的递归联合类型。这允许您通过 `NestedByteList` 构建非常大的嵌套dag，这些 `dag` 本身可以包含额外的 `NestedByteList` 或实际字节(通过 `NestedByte` 中的 `FlexibleByteLayout`链接)。

实现必须为二进制数据的读取范围定义一个自定义函数，但一旦实现，无论使用哪种布局算法，都可以读取数据。

由于读者只需要关心 read 方法的实现，他们不需要理解用于生成布局的算法。这在将来提供了很大的灵活性，可以根据需要定义新的布局算法，而无需担心更新先前的实现。

`length` 属性必须用适当的字节长度进行编码。如果编码不当，读者将无法正确阅读。但是，该属性并不安全，恶意编码器可以将其写成他们喜欢的任何形式。因此，当根据配额或任何类似的计算计算编码器可能有动机改变长度时，不应该依赖于它。
