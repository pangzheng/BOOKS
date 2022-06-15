# NFT.Storage-概念
## 去中心化存储简介
NFT.Storage 依靠分散式存储在网络上免费存储您的文件。去中心化存储是一种存储数据的技术，它代替传统的服务器，使用分布式网络，许多参与者提供存储容量。该模型固有地构建了冗余，可提供针对故障和攻击的弹性，以及由于大型分布式存储提供商网络提供的地理定位而增强的性能。虽然您不需要了解去中心化存储就能将 NFT.Storage 整合到您的应用程序和服务中，但如果您对幕后发生的事情感到好奇，请继续阅读。
### 内容寻址
从广义上讲，当今的大部分网络都使用所谓的位置寻址方式运行。位置寻址从网络上的特定位置检索在线信息——即从 URL 后面。

	https://example.com/page-one.html
然而，这种方法存在一些关键问题。位置寻址是集中的，这意味着控制该位置的人控制内容。控制器可以更改内容，完全替换它，或者直接拿走它。这意味着基于位置的地址容易受到攻击、利用和丢失。

分散这种传统的网络运营方式的方法的一部分需要实施一种新的寻址方式：内容寻址。内容寻址是一种向每条数据发出内容标识符 (CID)的技术，它是使用始终为相同内容生成相同密钥的算法直接从文件内容派生的令牌。使用内容寻址，可以根据文件的内容而不是位置来查询和检索文件——这是使网络摆脱集中式内容控制的主要因素。

NFT.Storage 使用 [IPFS](https://ipfs.io/)（星际文件系统）生成的 CID，为所有存储的数据启用内容寻址。此计算通常在用户（客户端）端完成，这意味着用户不必信任 NFT.Storage 即可知道他们拥有正确的 CID。只要 CID 用于引用 NFT 中的链下数据，就不会对 NFT 所指的内容（艺术、视频等）存在争议——您只需根据 CID 验证内容！

然而，内容寻址只是解决方案的一部分。IPFS 通过其 CID 引用数据，并确保只要至少一个副本被广播到网络，您就可以取回内容。但是仅仅因为一个文件有一个 CID 并不意味着该文件就可以保证永远存在。在一个运行良好的去中心化系统中，参与者都需要同意成为优秀的参与者并提供可靠的存储容量。为此，NFT.Storage 使用 [Filecoin](https://filecoin.io/) 网络。Filecoin 网络及其同名代币 FIL（或简称 ⨎）的创建是为了激励网络上的存储提供商同意存储交易，并提供去信任的证明，证明存储提供商实际上是按照承诺存储数据的。这些交易规定将在约定的时间段内提供一定数量的存储容量，以确保解决方案的第二部分：内容持久性。
### 用于可验证内容持久性的 Filecoin 
利用 Filecoin 网络存储使用 NFT.Storage 存储的数据，确保内容可供检索，从而确保 IPFS 提供的基于内容的寻址随着时间的推移保持弹性。Filecoin 使用多种方法来实现这一使命，包括[新颖的密码学、共识协议和博弈论激励措施](https://filecoin.io/blog/posts/filecoin-features-verifiable-storage/)——但其中最重要的可能是 Filecoin 独特的存储验证方法。

Filecoin 的存储验证系统解决了去中心化存储以前难以解决的问题：存储提供商如何证明他们确实存储了他们所说的一段时间内的数据？Filecoin 的[证明算法](https://filecoin.io/blog/posts/what-sets-us-apart-filecoin-s-proof-system/)负责此验证：

- [复制证明](https://proto.school/verifying-storage-on-filecoin/03)证明给定的存储提供商正在存储客户原始数据的唯一副本。
- [时空证明](https://proto.school/verifying-storage-on-filecoin/04)证明客户的数据随着时间的推移而连续存储。

由于这些证明最终在链上，任何人都可以查看 Filecoin 区块链并验证这些存储提供商在这段时间内是否存储了给定的内容。

除了这个证明系统之外，Filecoin 网络还依靠博弈论激励来阻止恶意或疏忽活动。为了成为 Filecoin 存储提供商，所有潜在的提供商在同意存储交易时必须以 FIL 的形式提供抵押品。此外，任何未能通过时空证明检查的存储提供商都会受到处罚，失去一部分抵押品，并最终无法再次向客户提供存储。

### 使用 NFT.Storage 进行存储和监控
您如何在上传到 NFT.Storage 的文件中看到这些原则？使用 [NFT.Storage JavaScript 客户端库](https://nftstorage.github.io/nft.storage/client/)很容易。

当您将文件上传到 NFT.Storage 时，您将获得该文件的 CID 作为回报。然后将该文件放入一个队列，供地理上分布的 Filecoin 网络存储提供商使用，这些提供商已被选为性能和可用性。这些提供商竞标存储队列中的文件（包括您的文件）的权利，并同意存储交易。

您可以通过调用 [check()](https://nftstorage.github.io/nft.storage/client/classes/lib.NFTStorage.html#check) 并提供文件的 CID 来监控上传到 NFT.Storage 的文件的此活动。这将返回查询时已完成的交易列表。以下是在 JavaScript 项目中包含此调用的方法：

	const cid = 'bafybeiggzq4ryi7hscq5hzvzcnk4urnxt3asp37dhgvnjilf7exskximla'
	
	// 根据CID检查状态
	const info = await client.check(rootCid)
	
	// 查询显示结果
	console.log(`${info.cid}`)
	for (const deal of info.deals) {
	  console.log(deal)
	}
而且您始终可以使用像 Filfox 这样的 Filecoin 区块浏览器来验证这些数据！

### 完全去中心化的 NFT.Storage 用于多代存储
保留链下 NFT 数据可能会成为“公地悲剧”的牺牲品。最终，NFT 的所有者有责任确保其 NFT 的链下组件存在且可访问，并且这种文化在 NFT 空间中不断发展——但实际上，确保链下可能会有些复杂数据始终保持不变。这就是为什么 NFT.Storage 的使命是[将所有 NFT 数据存储为公共物品](https://nft.storage/blog/post/2022-01-20-decentralizing-nft-storage/)。但这将如何运作？

单个 Filecoin 交易是任何去中心化存储系统的基石。Filecoin 链无需信任地显示特定内容是持久的、该内容在网络上的副本数量以及与谁一起存储。因此，您可以使用智能合约来监听链并执行诸如长期冗余之类的事情。如今，NFT.Storage 正在手动更新单个 Filecoin 交易，但很快，它将使用 [FVM](https://fvm.filecoin.io/) 来实现多代、去中心化的持久性。目前的计划是创建一个“数据 DAO”，为智能合约提供资金，以永久确保上传到 NFT.Storage 的数据存在许多副本，并在存储交易到期或副本消失时创建新的存储交易。

但是，通过使用 IPFS CID 来处理数据，用户不需要仅仅依靠 NFT.Storage 的持久性模型来保证他们的 NFT 安全。我们鼓励用户将他们的 NFT 保存在尽可能多的地方让他们感到舒服（我们称之为“[存储层最大化](https://nft.storage/blog/post/2021-12-14-storage-layer-maximalism/)”）。拥有这种文化也有助于 NFT 作为公共产品的去中心化存储！
### 总结
使用位置寻址来引用 Web 上的文件的传统方法存在许多严重的干扰、利用和丢失漏洞。因为谁控制了位置，谁就控制了内容，这些定位地址的中央存储网络也让网络暴露在大型企业中央存储平台的异想天开之下。

通过将 IPFS 内容寻址（去中心化如何指定资源）与 Filecoin（去中心化如何获取存储容量）配对，可以创建一个完整的解决方案来定位、存储和获取数据——一个不仅可以抵御这些漏洞，但也奖励网络参与者。
### 了解更多
想要深入了解去中心化存储、它的工作原理以及它为何如此重要？查看 [ProtoSchool](https://proto.school/verifying-storage-on-filecoin/) 以深入了解验证 Filecoin 上的存储，以及有关 DWeb 概念、协议和工具的大量其他交互式教程。
## 使用内容档案
当您使用[客户端库](https://nftstorage.github.io/nft.storage/client/)将文件上传到 NFT.Storage 时，您的数据将转换为数据结构图，然后将其打包成一种称为内容存档 (CAR) 的格式，然后再发送到 NFT.Storage 服务。

对于许多用例，您永远不需要了解此过程，因为在使用客户端库时转换发生在幕后。但是，如果您使用的是 [HTTP API](https://nft.storage/api-docs/) 或者您想要更多地控制 IPFS 数据图的结构，您可能希望直接使用 Content Archives。

本操作指南将解释 [Content Archives](https://nft.storage/docs/concepts/car-files/#what-is-a-content-archive) 的基础知识以及 [NFT.Storage API](https://nft.storage/docs/concepts/car-files/#car-files-and-nft-storage) 如何使用它们。

我们还将看到使用 [命令行工具](https://nft.storage/docs/concepts/car-files/#command-line-tools) 创建和操作内容档案的几种方法以及可以在应用程序代码中使用的库的概述。
### 什么是内容存档？
[CAR 内容存档格式](https://ipld.io/specs/transport/car/)是一种将[内容寻址数据] [概念-内容-寻址] 打包成可以轻松存储和传输的存档文件的方式。您可以将它们想象为设计用于存储内容寻址数据集合的 [TAR 文件](https://en.wikipedia.org/wiki/Tar_(computing))。

存储在 CAR 中的数据类型由 IPLD 或星际链接数据定义。IPLD 是结构化数据类型的规范和一组实现，可以使用基于散列的内容标识符 (CID) 相互链接。以这种方式链接的数据形成有向无环图或 DAG，您可能会在 IPLD 和 IPFS 的文档中看到对 DAG 的一些引用。

IPFS 文件是 IPLD 数据的一个示例，但 IPLD 也可用于访问来自以太坊、Git 和其他哈希寻址系统的数据。您还可以将 IPLD 用作结构化数据的通用格式，有点像 Web3 风格的 JSON。有关详细信息，请参阅下面的 [高级 IPLD 格式](https://nft.storage/docs/concepts/car-files/#advanced-ipld-formats)。
### CAR 和 NFT.Storage
当 NFT.Storage 客户端将常规文件打包到 CAR 中以存储在 IPFS 上时，CAR 包含以 IPFS 在使用命令行或其他 IPFS API 导入文件时使用的相同格式编码的数据。

这种格式使用名为 IPLD“编解码器” [dag-pb](https://ipld.io/docs/codecs/known/dag-pb/)，它使用[protocol-buffers](https://developers.google.com/protocol-buffers ) 对对象图进行编码。图中是描述文件及其内容的 [UnixFS 对象](https://docs.ipfs.io/concepts/file-systems/#unix-file-system-unixfs)。

尽管 [HTTP API](https://nft.storage/api-docs/) 也允许您上传常规文件，但出于一些原因，客户端更喜欢发送 CAR。

首先，格式化客户端上的所有内容允许我们在将任何数据发送到远程服务之前计算您正在上传的数据的根内容标识符。这意味着您可以将 NFT.Storage 服务返回的 CID 与您在本地计算的 CID 进行比较，并且您不必相信该服务会做正确的事情。

使用 CAR 的另一个原因是支持大文件，否则会达到 NFT.Storage 后端平台的大小限制。CAR中 的数据已经被分块成小块，这使得 CAR 很容易拆分成可以批量上传的小块。

### 命令行工具
有几种方法可以从命令行创建 CAR 文件并与之交互。
#### ipfs-car
[ipfs-car](https://github.com/web3-storage/ipfs-car) JavaScript 包包括一个命令行工具，用于轻松创建、解包和验证 CAR 文件。

要安装它，您需要[Node.js](https://nodejs.org/) - 我们推荐最新的稳定版本。

您可以全局安装命令：

	npm install -g ipfs-car
`npx` 或者在不将其安装到您的 PATH 的情况下运行该命令：

	npx ipfs-car --help
该 `--pack` 标志将从一组输入文件中创建一个新的 CAR 文件：

	ipfs-car --pack path/to/files --output path/to/write/a.car
或从 CAR 中提取文件 `--unpack`：

	ipfs-car --unpack path/to/my.car --output /path/to/unpack/files/to
您还可以使用以下命令列出 CAR 的内容 `--list`：

	ipfs-car --list path/to/my.car
有关更多使用信息，请运行 `ipfs-car --help`.

### goipfs 
[go-ipfs](https://docs.ipfs.io/install/command-line/) 是IPFS协议的参考实现。在许多其他功能中，`go-ipfs` 支持将任何 IPFS 对象图导出到 CAR 文件中，并将数据从 CAR 文件导入到本地 IPFS 存储库。

该 [ipfs dag export命](https://docs.ipfs.io/reference/cli/#ipfs-dag-export) 令将通过其内容 ID (CID) 获取 IPFS 对象图，将 CAR 数据流写入标准输出。

要使用 go-ipfs 创建 CAR 文件，您可以将输出重定向 `ipfs dag export` 到文件：

	cid="bafybeigdmvh2wgmryq5ovlfu4bd3yiljokhzdep7abpe4c4lrf6rukkx4m"
	ipfs dag export $cid > path/to/output.car
请注意，您应该将 `cid` 引号内的值替换为您要导出的 CID。

如果您的本地 IPFS 存储库中没有 CID，该 `dag export` 命令将尝试通过 IPFS 网络获取它。

要将 CAR 文件的内容添加到本地 IPFS 存储库，可以使用 `ipfs dag import`：

	ipfs dag import path/to/input.car
### 应用程序开发人员的库
- JavaScript 

	有两个 JavaScript 包可用于在应用程序中操作 CAR。

	- ipfs-car
	
		该 `ipfs-car` 包包括使用 IPFS UnixFs 数据模型将文件打包和解包到 CAR 的库函数。`ipfs-car` 该库包含与上述命令行实用程序相同的功能。
	
		有关 API 文档和使用示例，请参阅 [ipfs-car README](https://github.com/web3-storage/ipfs-car#api)。
	- @ipld/car
	
		该 [@ipld/car](https://github.com/ipld/js-car) 包包含 CAR 规范的主要 JavaScript 实现，并在后台使用 `ipfs-car`。如果你想使用 [高级IPLD格式](https://nft.storage/docs/concepts/car-files/#advanced-ipld-formats) 存储非文件数据，你应该 `@ipld/car` 直接使用。
	
		`@ipld/car` 还提供了 `CarReader` NFT.Storage 客户端 [storeCar](https://nftstorage.github.io/nft.storage/client/classes/lib.NFTStorage.html#storeCar) 方法使用的接口。
	
		下面是一个从 Node.js 流中加载 CAR 文件并将其存储在 NFT.Storage 中的简单示例：
	
			import { createReadStream } from 'fs'
			import { CarReader } from '@ipld/car'
			
			async function storeCarFile(filename) {
			  const inStream = createReadStream(filename)
			  const car = await CarReader.fromIterable(inStream)
			  
			  const client = makeStorageClient()
			  const cid = await client.putCar(car)
			  console.log('Stored CAR file! CID:', cid)
			}
		`CarReader.fromIterable` 接受任何可迭代的 `Uint8Array` 数据，包括 Node.js 流。如果您已经将所有 CAR 数据放在一个文件 `Uint8Array` 中，则可以[CarReader.fromBytes](https://github.com/ipld/js-car#CarReader__fromBytes)改用。
	
		上面显示的 `CarReader` 类型会将 CAR 的全部内容读入内存，这可能会导致大文件出现问题。在 Node.js 上，您可以使用 [CarIndexedReader](https://github.com/ipld/js-car#carindexedreader)，它直接从磁盘读取 CAR 数据，并且使用的内存比 `CarReader`.
- GO

	该 [go-car 模块](https://github.com/ipld/go-car) 提供了 CAR 规范的主要 Golang 实现。我们建议使用 `v2` 模块版本，它支持最新版本的 CAR 规范。

	有关更多信息，请参阅 [API 参考文档](https://pkg.go.dev/github.com/ipld/go-car/v2)。

### 拆分 CAR 以上传到 NFT.Storage 
NFT.Storage [HTTP API](https://nft.storage/api-docs/) 接受最大 100 MB 的 CAR 上传，但 JavaScript 客户端使用 HTTP API 上传任意大小的文件。客户端通过将 CAR 拆分为每个小于 100 MB 的块并分别上传每个块来设法做到这一点。

可用于拆分和加入 CAR 的主要工具称为 `carbites`，它在 JavaScript 和 Go 中实现。JavaScript 实现包括一个命令行版本，允许您从终端或喜欢的脚本语言拆分和加入 CAR。

本节将演示几种以 NFT.Storage 服务可接受的方式拆分 CAR 的方法，使用命令行工具，以及JavaScript 和 Go 中的库以编程方式使用 `carbites` 。
#### 使用 carbites-cli 工具
JavaScript [carbites 库](https://github.com/nftstorage/carbites) 包含一个名为的包 `carbites-cli`，可以从命令行拆分和连接 CAR。您需要安装最新版本的[Node.js](https://nodejs.org/)，最好是最新的稳定版本。

您可以使用以下命令全局安装该工具 `npm`：

	npm install -g carbites-cli

	added 71 packages, and audited 72 packages in 846ms
	20 packages are looking for funding
	  run `npm fund` for details
	found 0 vulnerabilities
这将向 `carbites` 您的 shell 环境添加一个命令：

	carbites --help
	
	  CLI tool for splitting a single CAR into multiple CARs from the comfort of your terminal.
	  Usage
	    $ carbites <command>
	    Commands
	      split
	      join
您可以运行该命令，而无需使用 Node.js 附带的命令 `carbites` 进行全局安装：`npx`

	npx carbites-cli --help
第一次，它会询问您是否要安装软件包：

	Need to install the following packages:
	  carbites-cli
	Ok to proceed? (y)
之后，您可以使用 `npx carbites-cli` 代替 `carbites` 以下任何命令！

#### 拆分 CAR 
该 `carbites split` 命令将 CAR 文件作为输入并将其拆分为多个较小的 CAR。

该 `--size` 标志设置输出 CAR 文件的最大大小。对于上传到 NFT.Storage，`--size` 必须小于 `100MB`.

另一个重要的标志是 `--strategy` ，它决定了 CAR 文件的拆分方式。对于 NFT.Storage 上传，我们需要使用该 `treewalk` 策略以便我们所有的 CAR 共享相同的根 CID。这将允许 NFT.Storage 服务在它们全部上传后再次将它们拼凑在一起。

这是一个示例，使用一个名为的输入 car 文件 `my-video.car` ，其重量为 455MB：

	carbites split --size 100MB --strategy treewalk my-video.car
这将在与输入文件相同的目录中创建五个新文件，命名 `my-video-0.car` 为 `my-video-4.car` . 如果列出它们的大小，您可以看到所有分块的汽车都小于或等于 100 MB：

	ls -lh my-video*
	
	-rw-r--r--  1 user  staff   100M Sep 15 13:56 my-video-1.car
	-rw-r--r--  1 user  staff   100M Sep 15 13:56 my-video-0.car
	-rw-r--r--  1 user  staff   100M Sep 15 13:56 my-video-2.car
	-rw-r--r--  1 user  staff   100M Sep 15 13:56 my-video-3.car
	-rw-r--r--  1 user  staff    56M Sep 15 13:56 my-video-4.car
	-rw-r--r--  1 user  staff   455M Sep 15 13:52 my-video.car
#### 合并 CAR
要合并之前拆分的 CAR，可以使用以下 `carbites join` 命令：

	carbites join my-video-*.car --output my-video-joined.car
### 使用 JavaScript
[carbites 库](https://github.com/nftstorage/carbites) 提供了一个用于拆分 CAR 的接口，该接口可以从您的应用程序代码中调用。

	你可能不需要这个！
	
	如果您使用 JavaScript，您可以 [使用 NFT.Storage 客户端][howto-store] 上传您的数据，让客户端为您处理 CAR 拆分。如果您确定要自己从 JavaScript 中分离 CAR，请继续阅读！

要从 JavaScript 代码中拆分 CAR，请安装 `carbites` 包：

	npm install carbites
并将 `TreewalkCarSplitter` 类导入您的代码：

	import { TreewalkCarSplitter } from 'carbites/treewalk'
可以通过为输出 CAR `TreewalkCarSplitter` 传入 `aCarReader` 和  `targetSize` 字节来创建。有关更多信息，请参阅 `@ipld/car` 部分 `CarReader`。现在，我们假设 `loadLargeCar` 函数返回 `CarReader`，我们将使用 `TreewalkCarSplitter` 来创建拆分 CAR：

	import { TreewalkCarSplitter } from 'carbites/treewalk'
	async function splitCars() {
	  const largeCar = await loadLargeCar()
	  const targetSize = 100000000
	  const splitter = new TreewalkCarSplitter(largeCar, targetSize)
	  for await (const smallCar of splitter.cars()) {
	    // 每个小的 CAR 是一个 AsyncIterable<Uint8Array> 的 car 数据
	    for await (const chunk of smallCar) {
		//处理 CAR 数据…
		//例如，你可以上传它到 NFT.storage HTTP API
	      // https://nft.storage/api-docs
	    }
	    // 您还可以使用 getRoots 方法获取每个小型 CAR 的根 CID
	    const roots = await smallCar.getRoots()
	    console.log('root cids', roots)
	    // Since we're using TreewalkCarSpliter, all the smaller CARs should have the
	    // same root CID as the large input CAR.
	    //因为我们使用 TreewalkCarSpliter，所有较小的 car 应该有与大输入 CAR 相同的根 CID。
	  }
	}
### 使用 Go
[go-carbites](https://github.com/alanshaw/go-carbites) 模块可用于从您的 Go 应用程序中拆分大型 CAR 。

安装模块 `go get`：

	go get github.com/alanshaw/go-carbites
该 [carbites.Split](https://pkg.go.dev/github.com/alanshaw/go-carbites#Split) 函数返回一个 [carbites.Splitter](https://pkg.go.dev/github.com/alanshaw/go-carbites#Splitter) 将确保输出 CAR 都具有相同的根 CID，这在上传到 NFT.Storage 时很重要。

	package main
	
	import (
	  "io"
	  "os"
	  "github.com/alanshaw/go-carbites"
	)
	
	func main() {
	  bigCar, _ := os.Open("big.car")
	  targetSize := 1024 * 1024 // 1MiB chunks
	  strategy := carbites.Treewalk
	  spltr, _ := carbites.Split(bigCar, targetSize, strategy)
	
	  var i int
	  for {
	  	car, err := spltr.Next()
	  	if err != nil {
	  		if err == io.EOF {
	  			break
	  		}
	  		panic(err)
	  	}
	  	b, _ := ioutil.ReadAll(car)
	  	ioutil.WriteFile(fmt.Sprintf("chunk-%d.car", i), b, 0644)
	  	i++
	  }
	}
您也可以使用 [NewTreewalkSplitterFromPath](https://pkg.go.dev/github.com/alanshaw/go-carbites#NewTreewalkSplitterFromPath)，它采用本地文件路径而不是 `io.Reader`.

### 高级 IPLD 格式
IPLD 也可以用作通用数据格式，如 JSON。事实上，您可以直接将 JSON 用作 IPLD，只需使用特殊约定链接到其他 IPLD 对象。此约定在 [dag-json“编解码器”](https://ipld.io/docs/codecs/known/dag-json/) 中定义。

这是一个 `dag-json` 对象的示例：

	{
	  "name": "Have you seen this dog?",
	  "description": "I have now...",
	  "image": { "/": "bafybeihkqv2ukwgpgzkwsuz7whmvneztvxglkljbs3zosewgku2cfluvba" }
	}
该 `image` 字段使用特殊的“链接类型”来引用另一个 IPLD 对象。该链接只是一个常规 JSON 对象，具有一个名为 的键 `/`，其值为内容标识符。

虽然 `dag-json` 熟悉且易于使用，但我们建议使用类似的 [dag-cbor 编解码器](https://ipld.io/docs/codecs/known/dag-cbor/)。`dag-cbor` 使用[简洁二进制对象](https://cbor.io/) 表示来更有效地编码数据，尤其是在使用 `dag-json`.

你可能已经在 `dag-cbor` 不知不觉中使用了！当使用客户端库的 `store` 方法或 HTTP API 的 `/store` 端点时，NTF.Storage 会创建一个 `dag-cbor` “包”，其中包含 NFT 元数据以及指向 NFT 中引用的所有文件的本地 IPLD 链接，包括用于兼容性的元数据的 JSON 版本

## 架构注意事项
在构建与 NFT.Storage 交互的工具和服务时，在规划项目时需要牢记一些事项。此页面收集了与 NFT.Storage 交互的一些常见模式，包括如何代表您平台的用户对请求进行身份验证。

### 认证和授权
目前有两种验证上传到 NFT.Storage 的方法，您选择哪种方法将取决于您要定位的区块链平台，以及您的目标和整体服务架构。
#### 第一种 API 密钥
最常见的身份验证方法是使用 API 密钥，可以在 NFT.Storage 网站上的[帐户管理页面](https://nft.storage/manage/)上生成。生成密钥后，您可以使用它来验证 [JavaScript 客户端](https://nftstorage.github.io/nft.storage/client) 或将密钥附加到对 [HTTP API](https://nft.storage/api-docs/) Authentication的请求中的标头。

虽然 API 密钥易于使用，但它们需要受到保护并保密，以避免访问您的 NFT.Storage 帐户。因此，我们强烈建议不要将您的 API 密钥包含在可能会暴露并可能提取和滥用密钥的 Web 应用程序或其他客户端应用程序中。

目前有几种方法可以在不暴露 API 密钥的情况下从前端代码提供对 NFT.Storage 的访问。

- 如果您有后端服务，您可以通过端点代理上传请求，该端点在执行您自己的身份验证后将您的 API 密钥附加到请求。
- 鼓励您的最终用户注册一个免费的 NFT.Storage 帐户，并提示他们输入他们的 API 密钥。您可以将密钥存储在浏览器的本地存储或其他持久位置以供将来使用，并将其附加到对 NFT.Storage 服务的请求。
- 如果您在 Solana 区块链上进行铸造，请参阅有关 Solana 钱包身份验证的部分，看看它是否适合您的用例。

#### 第二种 Solana 钱包认证
[metaplex-auth 库](https://github.com/nftstorage/metaplex-auth) 提供了一种身份验证机制，用于使用来自 [Solana](https://solana.com/) 私钥或浏览器钱包的签名将 NFT 数据上传到 NFT.Storage 。

虽然最初是为 [Metaplex](https://www.metaplex.com/) 设计的，但身份验证系统适用于任何 Solana 程序。使用库时，只需将 `mintingAgent` 选项设置为您的平台或工具的名称。

身份验证机制通过将您的上传文件打包到 [CAR 文件](https://nft.storage/docs/concepts/car-files/) 并计算根 CID 来工作，该 CID 包含在签名令牌有效负载中。因此，每次上传都需要用户私钥或钱包的签名。如果你想一次上传多个 NFT，你可以通过将它们打包到一个 CAR 中来减少用户摩擦，这样只需要一个签名。有关使用 CAR 的更多信息，请参阅 [CAR 文件指南](https://nft.storage/docs/concepts/car-files/)。

尽管该库是用 TypeScript 编写的，但您不需要使用 TypeScript 或 JavaScript 来利用身份验证机制，只要您愿意编写一点代码来生成正确的令牌。

请参阅 [规范文档](https://github.com/nftstorage/metaplex-auth/blob/main/SPEC.md) 以了解如何构建 metaplex-auth 库使用的自签名令牌。如果您打算这样做，请打开一个[问题](https://github.com/nftstorage/metaplex-auth/issues)让我们知道您正在做什么，我们可以帮助回答您可能遇到的任何问题。

### 即将推出：使用 UCAN 对所有人进行公钥认证
上述基于 Solana 钱包的身份验证被有意设计为特定于一个区块链，以限制我们探索此问题空间的范围并避免构建过于通用的解决方案。

同时，我们将继续为所有用户开发一种灵活的基于公钥的身份验证机制，无论他们打算在哪个链上铸造。这项工作目前处于设计阶段，以社区驱动的[用户控制授权网络 (UCAN)标准](https://whitepaper.fission.codes/access-control/ucan)为基础，并结合了我们在构建 Solana 集成时学到的一些东西。

您可以通过关注“Signed uploads w/UCAN”GitHub [issue](https://github.com/nftstorage/nft.storage/issues/851) 来跟踪这项工作的进度。如果您对设计有任何反馈，请对该问题发表评论并告诉我们。
### 为最终用户检索数据
对于需要代表他人检索 NFT 数据的 NFT 平台，您可以采取一些措施来确保高质量的用户体验。

在显示使用 IPFS 链接到链下数据的 NFT 时，您有多种检索选项。
#### HTTP 网关
IPFS 是围绕点对点内容共享协议构建的，该协议允许任何计算机向任何请求它的人提供内容。这提供了[许多好处][概念-分散存储]，但并非所有计算环境都可以完全支持对等网络范式。

虽然像 [Brave](https://brave.com/) 等一些浏览器提供 [原生 IPFS 支持](https://brave.com/ipfs-support/)，但其他浏览器仅限于 HTTP 及其相关协议，这使得直接使用点对点协议变得更加困难。

作为平台运营商，您可以使用 IPFS 网关通过 HTTP 为您的用户提供 NFT 服务，从而支持所有浏览器。由于 NFT.Storage 和其他遵循 [IPFS 上 NFT 数据的最佳实践][ipfs-docs-nft-best-practices] 的服务使用 `ipfs://` URL 而不是 `https://`，因此您的 Web 应用程序应该支持重写 IPFS URL 以针对您选择的网关.

请参阅 [最佳实践文档](https://docs.ipfs.io/how-to/best-practices-for-nft-data/#http-gateway-url) [ipfs-docs-nft-best-practices] 中有关HTTP 网关 URL的部分，以了解网关 URL 的结构以及如何从 `ipfs://` URL 创建它们。
#### 网关和集中化
重要的是要注意网关是一个[集中点](https://docs.ipfs.io/concepts/ipfs-gateway/#centralization)，受网关运营商的控制。这就是我们强烈建议将与网关无关的 `ipfs://` URL 存储在 NFT 元数据中的原因之一，尤其是在链上记录中，这样链接就不会“绑定”到单个提供商。

作为平台运营商，您可以将用户引导至任何 IPFS 网关，包括由 [Protocol Labs](https://protocol.ai/) 和 IPFS 生态系统中的其他组织运行的[公共网关](https://ipfs.github.io/public-gateway-checker/)。

由于公共网关是共享资源，您可能更愿意运行自己的 [专用网关](https://blog.stacktical.com/ipfs/gateway/dapp/2019/09/21/ipfs-server-google-cloud-platform.html) 以提供更一致的用户体验。由于内容寻址的灵活性，您可以将平台用户引导到您自己的应用程序层网关。任何不使用您的平台的人仍然可以从任何公共网关获取数据，但您的用户将获得改进的延迟和可靠性。

另一个潜在的优化是将 NFT 数据的冗余副本存储在传统存储服务上，并通过 HTTP 传递它们。这完全避开了 IPFS 检索，并且在某些情况下可能性能更高，但您需要做一些簿记以跟踪每条数据的 HTTP 位置。在采用此选项时，我们强烈建议向您的用户公开原始 `ipfs://` URL，以便他们有多个检索选项，并且不依赖于您的自定义 HTTP 服务。

## IPFS 网关
NFT.Storage 使用 [星际文件系统 (IPFS)](https://ipfs.io/) 作为其[存储和检索基础架构](https://nft.storage/docs/concepts/decentralized-storage/) 的关键部分。

IPFS 网络是一个点对点的计算机网络，它们共享资源以有效地向任何请求它的人提供内容。加入 IPFS 网络的计算机使用一种称为 [Bitswap](https://docs.ipfs.io/concepts/bitswap/) 的协议来请求和提供数据块，使用基于哈希的内容标识符 (CID) 来唯一标识每个块。

为了使 IPFS 数据可以在点对点网络之外访问，称为“网关”的[特殊 IPFS 节点](https://docs.ipfs.io/concepts/ipfs-gateway/)充当所有 Web 浏览器都理解的 HTTP 协议和点对点 Bitswap 协议之间的桥梁。

随着 [Brave](https://brave.com/ipfs-support/)和 [Opera](https://blogs.opera.com/tips-and-tricks/2021/02/opera-crypto-files-for-keeps-ipfs-unstoppable-domains/) 等越来越多的浏览器采用原生 IPFS 支持，对网关的需求自然会随着时间的推移而减少。今天，您可以通过在 Web 应用程序中使用 HTTP 网关来接触最广泛的受众，但同时 `ipfs://` 显示您的内容的原始 URI 是一个好主意，以便 IPFS 原生浏览器可以直接通过 Bitswap 访问内容。

本指南收集了一些有关网关的有用信息，包括如何以及为何使用 [NFT.Storage 网关](https://nft.storage/docs/concepts/gateways/#the-nft-storage-gateway)，网址为 `nftstorage.link`。

有关从网关获取内容的更多信息，请参阅我们的[数据检索指南](https://nft.storage/docs/how-to/retrieve)。
### NFT.Storage 网关
在网络上提供出色的检索体验是 NFT.Storage 使命的关键部分，因为我们致力于使[去中心化数据存储](https://nft.storage/docs/concepts/decentralized-storage/) 成为 NFT 创建者的默认选择。

为了进一步实现这一目标，我们创建了一个新的 HTTP 网关，它使用现有的公共 IPFS 基础设施和云原生缓存策略来提供以 NFT 为重点的高性能、基于 CID 的 HTTP 检索解决方案。
#### 架构
NFT.Storage 网关实际上是一个缓存层，它位于其他几个公共 IPFS 网关的前面，并相互“竞争”它们，以提供对最终用户请求的最快响应。

与直接使用“下游”网关相比，这种分层架构有几个好处。通过使用 CloudFlare Workers 在边缘缓存请求，网关可以快速提供流行内容，同时减轻公共 IPFS 网关和点对点网络的负载。

通过向多个公共 IPFS 网关发出并行请求并返回第一个有效响应来处理对未缓存内容的请求。这为您提供了最快的下游网关的性能，由于网络条件和网关自身本地缓存的状态，可能会不时发生变化。

发送多个请求还可以减少对任何单个网关的依赖，并在网关因维护而关闭或出现连接问题时保护您的应用不会遇到服务质量下降的问题。

由于 NFT.Storage 还知道服务中存储了哪些内容，因此 NFT.Storage 网关可以进行优化，并且对于存储在 NFT.Storage 上的数据而言性能最高。
#### 使用网关
NFT.Storage 网关可通过 URL 访问 `https://nftstorage.link`，您只需通过创建 URL `nftstorage.link` 作为主机即可使用它。

我们强烈建议在您的链上 NFT 记录中存储与网关无关的 `ipfs://` URI，因此无需更改该层的任何内容即可采用 NFT.Storage 网关。在获取和显示 NFT 的 Web 应用程序和库中，您可以将 `ipfs://` URI 重写为“路径样式”网关链接，方法是替换 `ipfs://` 为 `https://nftstorage.link/ipfs/`.

网关同时支持[“路径样式”](https://nft.storage/docs/concepts/gateways/#path-style-urls)和 [“子域样式”](https://nft.storage/docs/concepts/gateways/#subdomain-style-urls) URL，但您必须确保您的 HTTP 客户端在使用“路径样式”URL 时遵循重定向。例如，使用 `curl` 时 ，请确保添加 `-L` 标志以跟随 `Location` 标头重定向：

`curl -L https://nftstorage.link/ipfs/bafybeid4gwmvbza257a7rx52bheeplwlaogshu4rgse3eaudfkfm7tx2my/hi-gateway.txt`
网关始终将路径样式的 URL 重定向到子域样式的 URL，以便通过网关提供的 Web 内容可以使用[同源策略](https://en.wikipedia.org/wiki/Same-origin_policy)隔离。

在某些情况下，这可能会导致 CI​​D 被重新编码为与子域寻址兼容的格式。特别是以 base32 字符串编码开头的“版本 0”CID `Qm` 将被编码为 CID 版本 1 ( bafy...)。尽管编码不同，但出于内容识别和验证的目的，CID 仍然是等效的，您可以在不丢失任何信息的情况下转换回 CIDv0。

您可以通过直接创建子域样式 URL来避免从路径重定向到子域 URL ，但您需要确保您只使用 CIDv1，因为 CIDv0 的区分大小写的编码与子域 URL 不兼容。NFT.Storage API 始终返回 CIDv1，但如果您有其他 IPFS CID 来源，您可以在构建网关 URL 之前自行将 CIDv0 转换为 v1 。
#### 速率限制
IPFS 网关是一种公共的共享资源，它们的需求量通常很大。为了向所有用户提供公平的访问权限，NFT.Storage 网关对大流量来源施加了请求限制。

NFT.Storage 网关当前对给定 IP 地址的速率限制为每分钟 200 个请求。如果出现速率限制，IP 将被阻止 30 秒。
#### 网关类型
有关网关的 IPFS [官方文档](https://docs.ipfs.io/concepts/ipfs-gateway/) 有助于了解 IPFS 生态系统中的网关类型以及它们的使用方式。

就我们的目的而言，要了解的关键事项之一是可用于使用网关 URL 获取内容的不同[分辨率样式](https://docs.ipfs.io/concepts/ipfs-gateway/#resolution-style)。

如果您查看[公共网关列表](https://ipfs.github.io/public-gateway-checker/)，您会看到一些支持“子域”样式的 URL，而另一些只支持路径样式的 URL。下面简要回顾一下这两种风格之间的区别。有关更多示例，请参阅我们的 [从 IPFS 检索 NFT 数据的指南](https://nft.storage/docs/how-to/retrieve)。

#### 路径样式 URL 
“路径样式” URL 将 IPFS CID 放入网关 URL 的路径部分，如下所示：

[https://nftstorage.link/ipfs/bafkreied5tvfci25k5td56w4zgj3owxypjgvmpwj5n7cvzgp5t4ittatfy](https://nftstorage.link/ipfs/bafkreied5tvfci25k5td56w4zgj3owxypjgvmpwj5n7cvzgp5t4ittatfy)

如果 CID 指向目录列表，您可以在目录中附加文件名以获取文件：

[https://nftstorage.link/ipfs/bafybeid4gwmvbza257a7rx52bheeplwlaogshu4rgse3eaudfkfm7tx2my/hi-gateway.txt](https://nftstorage.link/ipfs/bafybeid4gwmvbza257a7rx52bheeplwlaogshu4rgse3eaudfkfm7tx2my/hi-gateway.txt)

#### 子域样式 URL 
“子域样式”网关 URL 将 CID 放入 URL 的主机部分，作为网关主机的子域，如下所示：

[https://bafkreied5tvfci25k5td56w4zgj3owxypjgvmpwj5n7cvzgp5t4ittatfy.ipfs.nftstorage.link](https://bafkreied5tvfci25k5td56w4zgj3owxypjgvmpwj5n7cvzgp5t4ittatfy.ipfs.nftstorage.link/)

如果 CID 指向目录列表，您可以使用 URL 的路径部分来指定文件名：

[https://bafybeid4gwmvbza257a7rx52bheeplwlaogshu4rgse3eaudfkfm7tx2my.ipfs.nftstorage.link/hi-gateway.txt](https://bafybeid4gwmvbza257a7rx52bheeplwlaogshu4rgse3eaudfkfm7tx2my.ipfs.nftstorage.link/hi-gateway.txt)

这是通过 HTTP 网关提供 Web 资产的首选样式，因为 Web 浏览器在每个域的基础上提供安全隔离。使用子域样式，每个 CID 都有自己的“命名空间”，用于 cookie 和本地存储等内容，从而将内容与存储在 IPFS 上的其他 Web 内容隔离开来。