# IPFS CID
	了解 IPFS 内容标识符 (CID)。
## 什么是 IPFS CID？
内容标识符，也称为 CID，是用于标识存储在 IPFS 网络上的文件的唯一值。此 CID 值不指示值的存储位置，而是指基于内容本身的地址。CID 是根据文件或文件夹的加密哈希生成的，这意味着：

- 使用相同设置和参数添加到两个独立 IPFS 节点的相同文件或文件夹将产生相同的 CID。
- 内容的任何差异（例如元数据差异）都会产生不同的 CID。

无论文件大小或内容如何，​​每个文件或文件夹的 CID 长度都相同。

## 客户 ID 格式
目前有两个不同版本的 IPFS CID——版本 0 (0v) 和版本 1 (v1)。

- 版本 0 (v0)

	CIDv0 CID 是 IPFS CID 的第一种格式，它是 IPFS 初始设计的一部分。此版本的 IPFS CID 使用基本 58 位编码的多重哈希来创建内容标识符值。这种方法很简单，但不像最新的 CID 版本 CIDv1 那样通用。CIDv0 仍然是各种 IPFS 应用程序和操作的默认版本，建议您在任何开发环境中支持 CIDv0。

	CIDv0 CID 的长度为 46 个字符，始终以字符“Qm”开头。
- 版本 1 (v1)

	CIDv1 旨在提供向前兼容性并支持可与未来版本的 IPFS CID 一起使用的不同格式。CIDv1 CID 包括：
	
	- 一个多基前缀，指定用于 CID 其余部分的编码方法和格式。
	- 指定 CID 版本的 CID 版本标识符。
	- 多编解码器标识符，指示 CID 旨在识别的内容格式。一旦从 IPFS 节点获取内容，这有助于软件和程序解释内容。

	CIDv1 CID 的前几个字节可用于解释 CID 的其余部分，并提供有关从 IPFS 获取内容后如何解码内容的信息。

### 何时使用 CIDv0 与 CIDV1
目前，建议新项目在开发中使用 CIDv1，因为这样可以使项目面向未来，并且 IPFS 计划在未来将默认 CID 版本设置为 CIDv1 作为新的默认版本。CIDv1 CID 也可以在浏览器上下文中安全使用，例如通过 IPFS 子域网关。
### CID转换
当使用传统网关访问 CIDv0 时，域将返回 HTTP 301 重定向到子域，将 CIDv0 CID 转换为 CIDv1 CID。

例如

- 使用传统网关在以下位置打开 CIDv0 资源：

		https://ipfs.filebase.io/ipfs/QmbWqxBEKC3P8tqsKc98xmWNzrzDtRLMiMPL8wBuTGsMnR
- 返回到 CIDv1 表示的重定向：

		https://bafkreidgvpkjawlxz6sffxzwgooowe5yt7i6wsyg236mfoks77nywkptdq.ipfs.dweb.link/

## 如何使用 CID？
一旦文件或文件夹上传到 IPFS 并返回 CID，该 CID 可用于通过 IPFS 网关访问存储的数据。默认情况下，上传到 IPFS 的所有数据都是公开的，因为访问它所需的只是关联的 CID。没有与 IPFS CID 相关的权限、用户帐户或其他安全设置。

网关用于为本身不支持 IPFS 的应用程序提供解决方法，也可以用作数据的共享链接。IPFS 网关可以是本地的、私有的或公共的，并使用 IPFS CID 提供内容的 URL 链接以访问存储的内容。

Filebase的原生IPFS网关如下：

	https://ipfs.filebase.io/ipfs/{CID}
通过 Filebase 存储在 IPFS 上的所有内容都可以通过 Filebase 网关访问，响应时间比通过任何其他网关访问内容更快。这是因为 Filebase 网关与我们的 IPFS 节点对等。Filebase 网关还与其他固定服务的 IPFS 网关对等。

如果您通过网关访问 IPFS 文件夹 CID，则该文件夹的内容将如下所示列出：

![](./pic/ipfs.png)

	您可以一个免费的注册 Filebase 帐户，立即开始使用 IPFS。