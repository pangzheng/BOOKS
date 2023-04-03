# 使用 IPFS pin 文件
Pinning 是 IPFS 中一个非常重要的概念。IPFS 语义试图让它感觉每个对象都是本地的——没有“从远程服务器为我检索这个文件”，只是 `ipfs cat`, `ipfs get` 无论实际对象位于何处，它们的行为都是一样的。

虽然这很好，但有时您希望能够控制随身携带的物品。Pinning 是一种机制，它允许您告诉 IPFS 始终将给定对象保留在某个地方——默认情况下是您的本地节点，但如果您使用[第三方远程 pinning 服务](https://docs.ipfs.tech/how-to/work-with-pinning-services/)，这可能会有所不同。

IPFS 有一个相当积极的缓存机制，在你对它执行任何 IPFS 操作后，它会在短时间内将对象保留在本地，但这些对象可能会定期被垃圾收集。要防止垃圾收集，只需 pin 您关心的 CID 或将其添加到 MFS 即可。通过  `ipfs add`  添加的对象默认情况下递归 `pin`。默认情况下，MFS 中的内容不会 `pin`，但会受到垃圾收集保护，因此可以将 MFS 视为一种隐式 `pin` 机制。

让我们看一下这个例子，更深入地探索pin到本地 IPFS 节点：

1. 首先创建一个名为 `foo` 内容的文件 `ipfs rocks`：

		echo "ipfs rocks" > foo    
	使用输出重定向指令时，该 `echo` 命令输出  `>` 。 

2. 接下来，`foo` 使用 `ipfs add` 命令添加到 IPFS：

		ipfs add foo               
		
		> added QmRTV3h1jLcACW4FRfdisokkQAk4E4qDhUzGpgdrd4JAFy foo
3. 接下来，生成已 `pin` 到本地存储的所有对象的列表。

		ipfs pin ls --type=all     
		
		
		> QmQPeNsJPyVWPFDVHb77w8G42Fvo15z4bG2X8D2GhfbSXc recursive
		> QmRTV3h1jLcACW4FRfdisokkQAk4E4qDhUzGpgdrd4JAFy recursive
		> QmQy6xmJhrcC5QLboAcGFcAE1tC8CrwDVkrHdEYJkLscrQ indirect
		> ...
	该 `pin ls` 命令将列出所有 `pin` 到本地存储的对象，`--type=all` 意味着它将列出所有不同类型的 `pin`（递归、间接等）。
4. 接下来，使用 `pin rm <foo hash>` 命令取消 `pin` 目标对象。

		ipfs pin rm <foo hash>     
		
		> unpinned QmRTV3h1jLcACW4FRfdisokkQAk4E4qDhUzGpgdrd4JAFy
	`foo hash` 这将从您 `pin` 的项目中删除该对象。
5. 接下来，尝试运行与上一步相同的命令。

		ipfs pin rm <foo hash>     
		
		> Error: not pinned or pinned indirectly
	在这里尝试再次运行ipfs pin rm <foo hash>会返回错误，这是因为上一步已经从<foo hash>.
6. 接下来，生成所有 pin 对象的新列表。

		ipfs pin ls --type=all    
	
		> QmQPeNsJPyVWPFDVHb77w8G42Fvo15z4bG2X8D2GhfbSXc recursive
		> QmQy6xmJhrcC5QLboAcGFcAE1tC8CrwDVkrHdEYJkLscrQ indirect
		> ...
	您会注意到此列表与上面的前一个列表完全相同，只是 `<foo hash>` 不再列出。

## 三种 pin
正如您在上面的示例中可能已经注意到的那样，第一个 `ipfs pin rm` 命令不起作用 — 它应该警告您给定的散列是递归 `pin` 的。这是什么意思？IPFS 世界中存在三种类型的 `pin`：

- Direct pins

	它只 pin 一个块没有其他相关的块。
- Recursive pins

	它 pin 给定的块及其所有子块。
- Indirect pins

	这是给定块的父级被递归 pin 的结果

pin 对象不能被垃圾回收——试试这个来证明：

1. 首先， `foo`  添加到 IPFS。

		ipfs add foo           
		
		> added QmRTV3h1jLcACW4FRfdisokkQAk4E4qDhUzGpgdrd4JAFy foo
		> 11 B / 11 B [===================] 100.00%

	这会将文件 `foo` 添加到 IPFS。
2. 接下来，使用 `repo gc` 命令对存储库进行垃圾回收。

		ipfs repo gc
		
		> removed QmVoSaWpZYicoLSAcdwxDPt2Gk4WVFCfBFTBtwwY2ASD9P
		> removed QmcpK3cSDyPGiNriMZrZTNu8YCPBSiMAApuvMqXaJVyuWr
		> removed QmdpczDhBmrkxerCUWkEcRExcTHFcA4EcDCeYNdXcV5iqE
		> ...
	该 `repo gc` 命令将从 IPFS 中删除所有未 pin 的对象。

3. 接下来，使用命令 `cat <foo hash>` 显示的 foo 内容。

		ipfs cat <foo hash>    
		
		> ipfs rocks
	该命令将 `ipfs rocks` 在 `<foo hash>` pin 后返回，因此无法将其删除。

但是，如果 `foo` 以某种方式变得无法 pin......

1. 使用 `pin rm` 命令取消 `pin<foo hash>`。

		ipfs pin rm <foo hash>    
		
		> unpinned QmRTV3h1jLcACW4FRfdisokkQAk4E4qDhUzGpgdrd4JAFy
	这将 foo 取消 pin。
2. 现在，再次运行垃圾收集命令。

		ipfs repo gc              
		
		> removed QmPPjksRv8SqiibAy6bSAXBnnfcBf3QnTnApxWjcFUTTkZ                                                
		> removed QmS3wrDNoaRtRi84K7Hf8jzh5sBZcwrQFj7CZLmmmacS2U                                                
		> removed QmRTV3h1jLcACW4FRfdisokkQAk4E4qDhUzGpgdrd4JAFy
	就像以前一样，这将从 IPFS 中删除所有未 pin 的对象。

	最后再次运行 `cat <foo hash>` 看是否返回了 foo.

		ipfs cat <foo hash>       
		
		> ipfs rocks
	您会注意到它仍然返回正确的响应，这是因为虽然 `<foo hash>` 已从本地存储中删除，但数据仍然存在于 IPFS 上。

## 本地与远程 pin
上面的所有信息都假定您在本地 pin 项目——也就是说，pin 到您的本地 IPFS 节点。这是 IPFS 的默认行为，但也可以将文件 pin 到远程 pin 服务。这些第三方服务让您有机会不将文件 pin 到您自己的本地节点，而是 pin 到它们运行的​​节点。您无需担心自己节点的可用磁盘空间或正常运行时间。

虽然您可以使用远程 pin 服务自己的 GUI、CLI 或其他开发工具来管理 pin 到其服务的 IPFS 文件，但您也可以使用本地 IPFS 安装直接使用pin 服务——这意味着您不需要学习 pin 服务的独特 API 或其他工具。

- [IPFS Pinning Service API](https://ipfs.github.io/pinning-services-api-spec/) （打开新窗口）提供一个规范，使开发人员能够集成任何支持该规范的 pin 服务或创建他们自己的服务。得益于 OpenAPI 规范格式，可以[生成](https://github.com/ipfs/pinning-services-api-spec#code-generation)客户端和服务器 （打开新窗口）来自 YAML 规范文件。
- 如果您从命令行使用 Kubo 0.8+，您可以访问 `ipfs pin remote` 充当客户端的命令以与 pin 服务 API 进行交互。直接从 CLI 添加您最喜欢的 `pin` 服务、以人类可读的名称 `pin` CID、获取 `pin` 状态等。[了解](https://docs.ipfs.tech/how-to/work-with-pinning-services/)
- [IPFS桌面](https://github.com/ipfs-shipyard/ipfs-desktop) （打开新窗口）及其等效的浏览器内 IPFS Web 界面，[IPFS Web UI](https://github.com/ipfs-shipyard/ipfs-webui) （打开新窗口），两者都支持远程pin服务，因此您可以直接从 UI pin到您最喜欢的pin服务。[了解](https://docs.ipfs.tech/how-to/work-with-pinning-services/)
