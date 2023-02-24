# DAG-JOSE
	状态：描述性 - 草稿

JOSE 是一种用于对 JSON 对象进行签名和加密的标准。[JOSE 的各种规范可以在IETF datatracker](https://datatracker.ietf.org/wg/jose/documents/)中找到。
## 格式
有两种 JOSE 对象：

- [JWS](https://datatracker.ietf.org/doc/rfc7515/?include_text=1)（JSON 网络签名）
- 和 [JWE](https://datatracker.ietf.org/doc/rfc7516/?include_text=1)（JSON 网络加密）

这两个对象是 JOSE 中的原语，可用于创建 JWT 和 JWM 对象等。IETF RFC 指定了 JOSE 对象的 JSON 编码。此规范将 JSON 编码映射到 CBOR。在遇到 dag-jose 多格式实现时，可以确定该块包含与我们在下面指定的 IPLD 模式匹配的 dag-cbor 编码数据。

## 从 JOSE 通用 JSON 序列化到 dag-jose 序列化的映射
JWS 和 JWE 都支持三种不同的序列化格式：

- Compact Serialization,
- Flattened JSON Serialization, 
- 和General JSON Serialization. 

前两个更简洁，但它们只允许一个收件人。因此，DAG JOSE 总是使用 `General Serialization` 确保最大兼容性和最小歧义的。实现序列化的库应该接受所有 JOSE 格式，包括 `Decoded Representation`（见下文）并在必要时转换它们。

要将一般的 JSON 序列化映射到 CBOR，我们执行以下操作：

-  任何表示为 `base64url(<data>)` 的字段直接映射到 Bytes。对于 `header` 和 `protected` 这样被指定为 `base64url(ascii(<some json>))` 的字段，这意味着该值是 `ascii(<some json>)` 字节。
- 对于 JWS，我们指定有效载荷属性必须是 CID，并将编码的JOSE对象的有效载荷设置为包含CID字节的Bytes。对于不希望使用额外的网络请求检索链接内容的应用程序，则应该使用标识多散列。
- 对于 JWE 对象，密文必须解密为明文，即 CID 的字节。这与有效负载是 CID 的原因相同，并且可以使用相同的使用身份 multihash 的方法，而且很可能是保留数据机密性的唯一方法。

下面我们将展示一个表示已编码的 JOSE 对象的 IPLD 模式。注意，有两种 IPLD 模式，Encoded JWE 和 Encoded JWS。实际的 wire 格式是一个单独的结构体，它包含了来自 EncodedJWE 和 EncodedJWS 结构体的所有键，实实者应该遵循 JWE 规范的第9节，通过检查负载属性是否存在来区分这两个分支，因此你有一个 JWS ;或密文属性，因此你有一个JWE。

编码

	type EncodedSignature struct {
	  header optional {String:Any}
	  protected optional Bytes
	  signature Bytes
	}
	
	type EncodedRecipient struct {
	  encrypted_key optional Bytes
	  header optional {String:Any}
	}
	
	type EncodedJWE struct {
	  aad optional Bytes
	  ciphertext Bytes
	  iv optional Bytes
	  protected optional Bytes
	  recipients [EncodedRecipient]
	  tag optional Bytes
	  unprotected optional {String:Any}
	}
	
	type EncodedJWS struct {
	  payload optional Bytes
	  signatures [EncodedSignature]
	}
	
## 加密有效负载
应用程序可能需要在加密时有效负载明文以避免泄露明文的大小。这就提出了一个问题，即应用程序如何知道解密的明文的哪一部分是有效负载的。在这种情况下，我们使用明文必须是有效 CID 的事实，实现应该将明文解析为 CID 并丢弃超出多散列摘要大小的任何内容——我们假设它是有效负载。
## Decoded JOSE
通常实现希望将这种格式解码为对应用程序更有用的格式。具体是什么样子取决于实现的语言，这里我们使用 IPLD 模式语言来给出一种与语言无关的描述，说明解码后的表示在运行时可能是什么样子。请注意，在 JOSE 规范中指定为 `base64url(ascii(<some JSON>))` 的所有内容——以及我们以有线格式编码为 `Bytes` 的内容——在这里都被解码为字符串。我们还向 `DecodedJWS` 添加了 `link: &Any` 属性，它允许应用程序轻松检索经过身份验证的内容。

还要注意，与编码表示一样，有两种不同的表示; `DecodedJWE` 和 `DecodedJWS`。应用程序可以用与上面描述的 `Encoded` 表示相同的方式区分这两个分支。

	type DecodedSignature struct {
	  header optional {String:Any}
	  protected optional String
	  signature String
	}
	
	type DecodedJWS struct {
	  payload String
	  signatures [DecodedSignature]
	  link: &Any
	}
	
	type DecodedRecipient struct {
	  encrypted_key optional String
	  header optional {String:Any}
	}
	
	type DecodedJWE struct {
	  aad optional String
	  ciphertext String
	  iv String
	  protected String
	  recipients [DecodedRecipient]
	  tag String
	  unprotected optional {String:Any}
	}
## 实现
- [Javascript](https://github.com/oed/js-dag-jose)
- [go](https://github.com/alexjg/go-dag-jose)
- 