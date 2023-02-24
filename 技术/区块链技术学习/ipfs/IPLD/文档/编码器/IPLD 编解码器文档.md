# IPLD 编解码器文档
IPLD 编解码器是将 IPLD 数据模型转换为序列化字节的函数，因此您可以发送和共享数据，并将序列化字节转换回 IPLD 数据模型以便您可以使用它。

有很多受支持的 IPLD 编解码器，您可以编写自己的编解码器。

与 IPLD 编解码器相关，有一个称为 “多编解码器” 的规范，它提供了一些标识符代码。这些多编解码器标识符代码可用于支持多个 IPLD 编解码器的应用程序和协议。通常，这些标识符代码与序列化数据相关联，因此很清楚如何反序列化它。（您可能希望查看 CID 以获得典型的完整使用案例，其中包括多编解码器的典型使用方式。）

## 已知编解码器
此处记录了 IPLD 中广为人知的一些编解码器。

（请记住，这不是编解码器的详尽列表。但它是我们使用最广泛的一些编解码器！）

- [DAG-CBOR](https://ipld.io/docs/codecs/known/dag-cbor/)

	二进制格式，支持完整的 IPLD 数据模型，性能优良，适用于任何工作。
- [DAG-JSON](https://ipld.io/docs/codecs/known/dag-json/)	
	一种人类可读的格式，支持几乎完整的 IPLD 数据模型，非常便于互操作性、开发和调试。
- [DAG-PB](https://ipld.io/docs/codecs/known/dag-pb/)

	一种用于特定有限数据结构的二进制格式，在 IPFS 和 unixfsv1 中被广泛使用。
	
[这里](https://ipld.io/specs/codecs/)还有几个编解码器需要短页。

- 这些页面中的每一个都应该包含对我们推荐或不推荐该编解码器的原因的人道描述。
- 这些页面中的每一个都应该链接到同一编解码器的规范页面。

## 代码中的编解码器
以下是一些 IPLD 库中如何实现编解码器的关键部分的指针：

- go：
	- 对功能接口描述编解码器
		- [ipld.Encoder (godoc)](https://pkg.go.dev/github.com/ipld/go-ipld-prime#Encoder)
		- 和 [ipld.Decoder (godoc)](https://pkg.go.dev/github.com/ipld/go-ipld-prime#Decoder)