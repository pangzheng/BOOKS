# IPFS 与 Sia 区别
## 应该使用 IPFS 还是 Sia？
了解 IPFS 和 Sia 网络之间的区别，以及哪个网络最适合您的项目工作流程。

Filebase 支持为 IPFS 和 Sia 网络上的数据存储创建桶。您应该为项目的工作流程使用哪个网络，每个网络都有哪些好处？
### IPFS
要了解 IPFS 的工作原理，请参阅以下文档：

- [IPFS](https://docs.filebase.com/what-is-filebase/storage-networks)
- [IPFS 固定](https://docs.filebase.com/ipfs/ipfs-pinning)
- [IPFS CID](https://docs.filebase.com/ipfs/ipfs-cids)
- [IPFS 网关](https://docs.filebase.com/ipfs/ipfs-gateways)
- [Filebase IPFS 白皮书](https://docs.filebase.com/knowledge-base/whitepapers#filebase-ipfs-whitepaper)

IPFS 或星际文件系统，本身并不是一个分散的存储网络——它是一种通信协议，为利用内容寻址而不是位置寻址的对等文件存储网络提供了基础。

#### 内容寻址
内容寻址不易受到所谓的“链接失效”的影响，而使用位置寻址的 URL 容易受到这种影响。链接失效是指当 HTTP URL 由于内容被删除或域变得不活动而不再可访问或有效时。由于内容寻址不依赖于位置寻址 URL，而是依赖于可以通过数百个不同的 IPFS HTTP 网关引用的唯一内容标识符，因此 IPFS 网络上的内容标识符 (CID) 对于长期数据持久性是可靠的。
#### 公开
上传到 IPFS 的所有数据都可以公开访问。世界上任何人都可以查看和下载它，无需使用 IPFS 本机 URL 或 IPFS HTTP 网关以及文件的 IPFS 内容标识符 (CID) 进行身份验证。

用户可以在上传之前对数据进行加密，以便为敏感文档或信息提供一定程度的数据隐私。用户必须在上传之前和之后维护加密。文件仅在数据传输期间在网络上加密，而不是在数据处于静止状态时加密。

存储在私有 IPFS 存储桶中的数据不是私有的。存储在私有 IPFS 存储桶和公共 IPFS 存储桶中的数据之间的唯一区别是私有 IPFS 存储桶的数据无法通过 Filebase S3 URL 访问。

	所有数据仍然可以通过文件的 CID 和 IPFS 网关或 URL 访问。
#### 多功能性
上传到 IPFS 的数据被赋予一个唯一的内容标识符（CID），用于识别和访问文件。对于本机不支持 IPFS 的平台，可以通过 IPFS 本机 URL 或 IPFS HTTP 网关在不进行身份验证的情况下访问 CID。这允许在各种代码库、SDK 或其他应用程序中引用和使用数据。只要您的 IPFS CID 固定到网络，此 CID 就永远不会过期或更改。
#### Filebase 的 Geo-Redundant IPFS Pinning
通过 Filebase 上传到 IPFS 的所有数据都固定在整个 Filebase IPFS 基础设施中的 3 倍冗余。每个文件在美国、伦敦和德国存储 3 个副本，以实现最大的数据可靠性和可用性。

- IPFS 的好处
	- 公共可用性允许轻松访问，而无需维护权限和身份验证访问。
	- 分散式和分布式基础架构，可在没有单一可用性的情况下提供数据持久性。
	- 专为长期存储和内容托管而设计的数据弹性。
	- 与传统的数据存储或网络托管选项相比价格低廉。
	- 高效且高性能，可为数据检索提供可持续的带宽节省。
- IPFS的缺点
	- 所有数据都是公开的，必须加密以保持私密性。
	- 节点参与网络没有货币激励。

#### 何时使用 IPFS
IPFS 应用于以下项目：

- 公共数据共享
- NFT收藏品
- 为元界平台存储和托管数字资产和世界物品
- 视频直播
- 并行数据分析
- 托管数据集
- 对文档和项目进行同行评审或协作
- 分散的静态网站托管
- 软件容器托管，例如 Docker 容器
- 分布式选美管理
- 内容分发网络配置
- 去中心化数据复制集群
- 数字身份
- 音乐串流
- 去中心化 DNS 服务
- 抗审查内容
- 去中心化社交媒体、聊天应用程序或其他社交网络平台
- 使用 IPFS 存储提案和链下投票结果的 DAO 治理或投票
- 分布式操作系统
- 不受审查限制的维权协调

### Sia
要了解有关 Sia 的更多信息，请参阅以下文档：

- [sia](https://docs.filebase.com/what-is-filebase/storage-networks)

Sia 是一个开源的、公共的去中心化存储区块链网络。Filebase 直接与 Sia 作为节点运行，这意味着 Filebase 平台代表 Filebase 用户管理所有 Sia 存储合约。

#### 隐私
默认情况下，存储在 Sia 网络上的所有数据都是私有的。如果您的 Filebase 存储桶设置为私有，则其他任何人都无法访问您的数据，并且存储在您的存储桶中的文件也无法被其他人访问。

存储在 Sia 上的数据使用 Threefish 加密算法进行加密，该算法已针对相关密钥攻击和旁路攻击进行了强化。
#### Reed-Solomon 纠删码
Sia 使用 Reed-Solomon 擦除编码来保护和存储跨多个节点的数据。在 Sia 上，数据被分成 30 个单独的部分，其中一些部分充当奇偶校验块。每件作品都存储在全球不同的节点上。在这 30 件中，只有 10 件需要可访问，以便为要检索的文件编译文件。这样最多可以有20个节点离线或者不可访问，文件还是可以访问的。
#### 最大文件大小
Sia 上可以存储的最大单个对象大小为 300GB。

- Sia 的好处
	- 默认情况下，数据是私有的，无需维护加密。
	- 高度冗余。
	- 通过 Siacoin 激励节点参与网络，从而减少节点流失。
- Sia 的缺点
	- 数据共享需要公共存储桶或身份验证。
	- 托管分散的应用程序、网站或其他面向公众的平台需要使用域名和内容交付机制。

#### 何时使用 Sia
- 存储高度敏感的私人数据——个人身份信息、身份证复印件等
- 备份私人机密文件
- 存储用户数据，例如地址、信用卡号或其他必须满足某些安全合规性要求的信息
- 存储私人应用程序数据，例如用户的消息日志或购买历史记录

## IPFS 和 Sia 有什么区别？
	此页面详细介绍了 Filebase 使用的当前支持的网络之间的差异。
Filebase 目前使用 IPFS 和 Sia 去中心化网络。这些网络中的每一个都使用纠删码技术对存储在其上的所有数据进行加密，并具有本地地理冗余。但是，根据其底层框架和技术，每个网络都比其他网络更适合某些用例。
### IPFS
星际文件系统（IPFS）是一个分布式和去中心化的存储网络，用于存储和访问文件、网站、数据和应用程序。IPFS 使用点对点网络技术连接一系列位于世界各地的节点，这些节点构成了 IPFS 网络。

本节简要概述 IPFS 及其工作原理。有关 IPFS 的详细技术说明，请查看我们的 IPFS 白皮书：

- [白皮书](https://docs.filebase.com/knowledge-base/whitepapers#filebase-ipfs-whitepaper)

IPFS特有的IPFS分为三块：

- 通过内容寻址的唯一数据标识

	存储在 IPFS 上的数据是通过其内容地址而不是其物理位置来定位的。当数据存储在 IPFS 上时，它存储在一系列加密的片段中，每个片段都- 有自己唯一的内容标识符或哈希。此唯一标识符称为 CID。此散列用作标识符并将片段链接到该数据的所有其他片段。
- 通过有向无环图 (DAG) 进行内容链接

	DAG 是一种分层数据结构，其中 IPFS 对等网络上的每个节点和存储对象都有一个唯一标识符，该标识符是节点内容的哈希值。此唯一标识符称为 CID。
- 通过分布式哈希表 (DHT) 进行内容发现

	DHT 是键和值的数据库，分布在分布式网络上的所有对等点上。要查找内容，您可以询问网络上的对等点，它会返回一个 DHT，告诉您哪些对等点正在存储构成您请求的数据对象的哪些内容块。

可以使用 IPFS 网关访问存储在 IPFS 上的内容。网关用于为本机不支持 IPFS 的应用程序提供解决方法。IPFS 网关可以是本地的、私有的或公共的，并使用 IPFS 内容 ID 提供内容的 URL 链接以访问存储的内容。

#### 本机 IPFS URL
本机支持 IPFS 内容寻址的应用程序可以引用存储在 IPFS 上的内容，格式如下：

	ipfs://{CID}/{optional path to resource}
此格式不适用于依赖 HTTP 的应用程序或工具，例如 Curl 或 Wget。对于这些工具，您需要使用 IPFS 网关。
#### IPFS 网关
可以使用 IPFS 网关访问存储在 IPFS 上的内容。网关用于为本机不支持 IPFS 的应用程序提供解决方法。IPFS 网关可以是本地的、私有的或公共的，并使用 IPFS 内容 ID 提供内容的 URL 链接以访问存储的内容。

Filebase 的原生 IPFS 网关如下：

	https://ipfs.filebase.io/ipfs/{CID}
通过 Filebase 存储在 IPFS 上的所有内容都可以通过 Filebase 网关访问，响应时间比通过任何其他网关访问内容更快。这是因为 Filebase 网关与我们的 IPFS 节点对等。Filebase 网关还与其他固定服务的 IPFS 网关对等。

#### IPFS 子域网关
子域网关的格式如下：

	https://{CID}.ipfs.dweb.link
使用子域网关作为传统网关的直接替代品，无需进行 CID 版本转换，因为打开旧的 CIDv0 资源和 CIDv1 资源是有区别的。当使用传统网关访问 CIDv0 时，域将返回 HTTP 301 重定向到子域，将 CIDv0 CID 转换为 CIDv1 CID。

例如，使用传统网关在以下位置打开 CIDv0 资源：

	https://dweb.link/ipfs/QmbWqxBEKC3P8tqsKc98xmWNzrzDtRLMiMPL8wBuTGsMnR
返回到 CIDv1 表示的重定向：

	https://bafkreidgvpkjawlxz6sffxzwgooowe5yt7i6wsyg236mfoks77nywkptdq.ipfs.dweb.link/
通过最初使用子域网关，不需要进行转换步骤。

#### 什么是 IPFS 的“固定”数据？
当数据存储在 Filebase 存储桶中时，Filebase 的原生边缘缓存技术会将数据缓存在网络的边缘位置，以便再次快速访问该数据。然后，当达到 IPFS 垃圾收集存储限制（IPFS 桌面客户端和 Brave 之间有所不同）时，缓存数据将被清除以为其他数据腾出空间。这种未固定数据的清除称为 IPFS 垃圾收集过程。

如果您希望数据长期缓存，您可以固定数据。这将使数据无限期地缓存在 IPFS 网络上，因为在垃圾收集过程中会跳过固定数据，从而加快数据检索时间。

默认情况下，上传到 Filebase 上的 IPFS 的文件会自动固定到 IPFS 并通过 3 倍复制存储在 Filebase 基础设施中，您无需支付额外费用。这意味着您的数据在发生灾难或中断时可访问且可靠，并且不会受到 IPFS 垃圾收集过程的影响。

固定可以通过 [控制台](https://docs.filebase.com/ipfs/ipfs-pinning#uploading-a-file-to-ipfs-through-the-filebase-web-dashboard) 或者 [S3 兼容 API ](https://docs.filebase.com/ipfs/ipfs-pinning#uploading-a-file-to-ipfs-using-the-s3-compatible-api).

固定数据是 IPFS 网络的一个独特功能，不为在其他网络上创建的桶提供。

IPFS 网络没有最大文件大小，但必须使用与 Filebase S3 兼容的 API 上传大于 5GB 的文件才能使用的

IPFS 桶非常适合：

- 无需管理身份验证或权限即可共享数据。
- 托管流媒体视频。
- 托管用于数据分析的数据集。
- 托管 NFT 集合并存储相关的 NFT 元数据和文件。
- 为分散式应用程序存储脚本、代码和内容。

更多关于 IPFS 的信息，请参考 [IPFS 文档](https://docs.ipfs.io/)


### Sia
Sia 是一个开源去中心化存储网络，它利用区块链技术创建一个安全且冗余的云存储平台。

Filebase 作为节点运营商直接与 Sia 合作，这意味着 Filebase 代表 Filebase 用户管理所有存储合约。上传到 Filebase Sia 存储桶的每个对象都使用[EC](https://en.wikipedia.org/wiki/Reed%E2%80%93Solomon_error_correction) 具有 30 分之 10 的算法。对象被分成 30 个部分，然后在地理上分布到世界各地的 Sia 主机服务器。30 个文件中只有 10 个可用才能处理下载请求。这意味着对象的 20 个部分可以被销毁、脱机或以其他方式不可用，从而创建本机冗余和高可用性。

Sia 使用 Threefish 算法实现高性能和安全加密。Threefish 特别加强了对相关密钥攻击和边信道攻击的防御。

Sia 上可以存储的最大单个对象大小为 300GB。

默认情况下，上传到 Sia 存储桶的所有数据都是私有的。

Sia 桶非常适合：

- 增加数据隐私。

有关 Sia 的更多信息，请参阅[sia docs](https://sia.tech/docs/) 
