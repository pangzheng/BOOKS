# 什么是 FileBase？
Filebase 是第一个兼容 S3 的对象存储平台，它允许您以安全、冗余和高性能的方式跨多个去中心化存储网络存储数据
## 入门
要开始使用 Filebase，一个免费的 5GB 帐户，然后查看我们的入门指南：

- [开发人员快速入门指南](https://docs.filebase.com/getting-started/developer-quick-start-guide)
- [IPFS 入门指南](https://docs.filebase.com/getting-started/ipfs-getting-started-guide)
- [S3 API 入门指南](https://docs.filebase.com/getting-started/s3-api-getting-started-guide)
- [web 控制台入门指南](https://docs.filebase.com/getting-started/getting-started-guide)

有问题？答案很可能可以在我们的常见问题解答部分找到：

- [常见问题](https://docs.filebase.com/getting-started/faq)

### 去中心化存储
分散式存储提供了一种不同的方式来思考如何存储和访问您的信息。数据分布在地理分布的节点上，并通过对等网络连接。这与传统的云模型非常不同，传统的云模型将数据孤岛存储到容易发生中断的区域。

这些节点使用数据分片和 EC 来存储数据。分片和 EC 将对象分成称为分片的小块，对其进行加密，然后将这些分片分布在这些存储节点上。

每个节点在任何给定时间只能访问存储数据的一小部分。为了检索对象，这些对等网络只需要数据分片的一部分来组成要传输的数据。
#### 地理冗余
地理冗余是将作为分散式网络一部分的物理节点放置在不同地理位置的做法。这使得连接这些节点的对等网络能够灵活应对自然灾害、火灾或基础设施受损等灾难性事件，确保网络上的所有节点都不会被破坏。**存储在这些节点上的数据通过 EC 存储在分片中**。当这些网络上的服务器离线时，丢失的碎片会自动修复并上传到新节点，不会对您造成任何干扰。

当数据冗余度为 100% 时，Filebase 能够为每个对象实现 3 倍的冗余度。

要深入了解异地冗余，请[此处](https://docs.filebase.com/web3-education/deep-dives/deep-dive-geo-redundancy)查看我们的文档。
#### 易于使用
Filebase 代表您管理数据存储层的所有方面并消除与在任何支持的网络上存储数据相关的复杂性。其中一些限制包括最小文件/扇区大小问题、保留问题和检索延迟，以及需要使用特殊软件或技能组合才能使用这些网络。
#### [边缘缓存技术](https://docs.filebase.com/what-is-filebase/edge-caching-technology)
Filebase 使用独特的专有。如果从我们的服务下载了一个最近或经常访问的对象，这个对象很可能会缓存在我们的边缘层。如果数据缓存在我们的边缘，您将不需要为消耗的带宽付费。
### 文件库[定价模型](https://docs.filebase.com/getting-started/billing-and-pricing)
Filebase 是一个供所有用户免费使用的平台。所有用户都可以在 IPFS 网络上存储最多 5GB 的数据，最多 1,000 个单独的文件，无需信用卡。

Filebase 提供 3 层付费订阅计划：Starter、Pro 和 Business。

	这些订阅层适用于 IPFS 网络，不包括 Sia 网络上的存储。有关 Sia 定价，请参阅 Sia 定价。
- 初级：20 美元/月
	
	包括：
	
	- 无限文件
	- 200 GB 存储空间
	- 400 GB 带宽
	- 1 个专用 IPFS 网关
	- IPFS 固定服务 API
	- 按 CID 固定
	- 额外存储 `$0.15/GB`
	- 额外带宽 `$0.015/GB`
- 专业版：100 美元/月

	包括

	- 无限文件
	- 1,000 GB 存储空间
	- 2,000 GB 带宽
	- 3 个专用 IPFS 网关
	- IPFS 固定服务 API
	- 按 CID 固定
	- 额外存储 `$0.13/GB`
	- 额外带宽 `$0.015/GB`
- 商务版本：500/月
	- 包括
		- 无限文件
		- 5,000 GB 存储空间
		- 10,000 GB 带宽
		- 5 个专用 IPFS 网关
		- IPFS 固定服务 API
		- 按 CID 固定
		- 额外存储 `$0.12/GB`
		- 额外带宽 `$0.015/GB`
		
请[在此处](https://docs.filebase.com/getting-started/billing-and-pricing/pricing-model) 了解更详细地查看我们的定价模型。

关于计费的问题？查看我们的账单常见问题解答：

[账单常见问题](https://docs.filebase.com/getting-started/billing-and-pricing/billing-faq)

#### IPFS服务
默认情况下，上传到 Filebase 上的 IPFS 的文件会自动固定到 IPFS 并通过3 倍复制存储在 Filebase 基础设施中，您无需支付额外费用。这意味着您的数据在发生灾难或中断时可访问且可靠，并且不会受到 IPFS 垃圾收集过程的影响。

在此处了解有关 IPFS 的更多信息：

- [ipfs](https://docs.filebase.com/what-is-filebase/storage-networks/ipfs)
- [ipfs cid](https://docs.filebase.com/ipfs/ipfs-cids)
- [ipfs pin](https://docs.filebase.com/ipfs/ipfs-pinning) 

Filebase 提供公共 IPFS 网关或私有专用网关作为付费功能。通过 Filebase 存储在 IPFS 上的所有内容都可以通过 Filebase 网关访问，响应时间比通过任何其他网关访问内容更快。这是因为 Filebase 网关与我们的 IPFS 节点对等。Filebase 公共网关还与其他固定服务的 IPFS 网关对等。

[IPFS 网关](https://docs.filebase.com/ipfs/ipfs-gateways)

在我们的 IPFS 白皮书中了解有关 Filebase 的 IPFS 服务的更多信息：

[白皮书](https://docs.filebase.com/knowledge-base/whitepapers#filebase-ipfs-whitepaper)

#### [S3 兼容 API](https://docs.filebase.com/knowledge-base/whitepapers#filebase-ipfs-whitepaper)
由于 Filebase 与 S3 兼容，因此它可以与 S3 API 兼容的工具、SDK 和框架一起开箱即用。查看以下部分以查看有关各种不同类型的配置和用例的文档：

- [备份客户端配置](https://docs.filebase.com/third-party-tools-and-clients/backup-client-configurations)
- [命令行工具](https://docs.filebase.com/third-party-tools-and-clients/cli-tools)
- [代码开发](https://docs.filebase.com/code-development-+-sdks/code-development)
- [内容分发网络](https://docs.filebase.com/third-party-tools-and-clients/content-delivery-networks)
- [文件管理客户端配置](https://docs.filebase.com/third-party-tools-and-clients/file-management-client-configurations)
- [NAS 设备配置](https://docs.filebase.com/third-party-tools-and-clients/nas-device-configurations)

如果您有任何问题，请加入我们的，或发送电子邮件至hello@filebase.com

#### 通过 [FIlebase 学习教程](https://docs.filebase.com/web3-education/web3-tutorials)
通过遵循学习教程，了解如何使用各种 web3 生态系统、工具和工作流

- [web3 教程](https://docs.filebase.com/web3-education/web3-tutorials)
- 通过我们的 [DeepDive](https://docs.filebase.com/web3-education/deep-dives) 文档了解各种 web3 概念

#### 电子书、单页书、白皮书
- [电子书](https://docs.filebase.com/filebase-knowledge-base/ebooks)
- [单页书](https://docs.filebase.com/filebase-knowledge-base/one-pagers)
- [白皮书](https://docs.filebase.com/filebase-knowledge-base/whitepapers)


## 数据管理
通过我们的浏览器界面、S3 兼容 API 或 IPFS 固定服务 API，使用 Filebase 管理数据变得简单。

可以通过三种主要方法与 Filebase 上的数据进行交互：

- [浏览器界面](https://console.filebase.com/)

	‌Filebase 浏览器界面可用于
	
	- 创建、管理和删除存储桶
	- 它还可用于上传、下载、管理和删除存储在这些存储桶中的对象
	- 它还可用于查看和编辑有关存储对象的元数据

	通过浏览器界面，您还可以编辑帐户设置并查看您的 Filebase 帐户的帐单历史记录和帐单信息。

	您通过浏览器界面与之交互的存储后端与可通过 S3 兼容 API 访问的后端相同。使用 API 所做的任何更改都将在刷新网页时反映在浏览器界面中。
- [S3 兼容 API](https://s3.filebase.com/)

	Filebase 已经认证了一系列与 S3 兼容的 API 软件和工具以与 Filebase 一起使用。Filebase 认证包括通过测试程序对软件工具进行测试，以验证与 Filebase S3 兼容 API 的一致性。

	如果您没有看到您选择的工具列出 - 不要担心。这仅仅意味着我们还没有机会对其进行测试和认证。
- [IPFS 固定 API](https://docs.filebase.com/api-documentation/ipfs-pinning-service-api)

	IPFS Pinning Service API 是一个标准化的 API，可用于重新固定现有的 IPFS CID。它通常被称为“远程”固定或 IPFS“PSA”。
IPFS 开发团队评估每个支持 IPFS PSA 的提供商的合规性，以验证每个实施。Filebase 完全符合 IPFS PSA 标准。官方[合规报告](https://ipfs-shipyard.github.io/pinning-service-compliance/api.filebase.io.html)可见.

	Filebase 通过我们的支持 [IPFS 固定服务](https://docs.filebase.com/ipfs/ipfs-pinning#re-pinning-existing-ipfs-cids-from-the-filebase-web-console)。此 API  可用于查看固定对象、添加要固定的新对象或删除固定对象。

要了解如何开始使用这些方法中的任何一种，请查看我们的入门指南：

- [IPFS 入门指南](https://docs.filebase.com/getting-started/ipfs-getting-started-guide)
- [S3 API 入门指南](https://docs.filebase.com/getting-started/s3-api-getting-started-guide)
- [web 控制台入门指南](https://docs.filebase.com/getting-started/getting-started-guide)

## 计算合作伙伴
详细了解 Filebase 的计算合作伙伴

- [akash-network](https://docs.filebase.com/what-is-filebase/compute-partners/akash-network)

### akash-network
了解 Akash 网络及其与 Filebase 的合作关系。

去中心化网络（也称为 dWeb）的核心目标是帮助人们和组织重新控制他们的计算和存储。

为了打破构建 dWeb 的障碍，开源、去中心化的云 Akash Network 与 Filebase 合作。
### 什么是Akash？
Akash 是一个 [开源云平台](https://github.com/ovrclk/akash) ，可以使用[命令行](https://docs.akash.network/general-commands)以低于 AWS 的成本将 Docker 容器快速部署到您选择的任何云提供商。

Akash 的典型工作流程如下：

- 在 `deploy.yaml` 文件中定义 Docker 映像、CPU、内存和存储。
- 设置您的价格，在几秒钟内收到供应商的出价，然后选择最低价格。
- 无需设置、配置或管理服务器即可部署您的应用程序。
- 将您的应用程序从单个容器扩展到数百个部署。

### 这是什么意思？
	将分散式计算和存储作为一个整体访问。
通过 Akash 和 Filebase 之间的这种合作关系，开发人员和企业现在能够将 dWeb 应用程序与他们已经了解和喜爱的数千种现有备份工具、应用程序 SDK 和框架配对。通过提供开源 dWeb 计算和 PB 级对象存储，与传统云服务相比，Akash 和 Filebase 可以帮助将云成本降低多达 85%。
### Akash 的社区项目
- [Ruby akash-on-rails ](https://github.com/ovrclk/akash-on-rails) 
- [PostgreSQL 备份与恢复](https://github.com/ovrclk/akash-postgres-restore)
- [部署去中心化 wordpress 博客](https://medium.com/@zJ_/how-to-deploy-a-decentralized-blog-3a5a13a6a827)
- [托管 Cosmos 节点](https://github.com/ovrclk/cosmos-omnibus)
- 托管的 [Helium](https://github.com/tombeynon/helium-on-akash) 验证器

以下是一个简短的设置指南，可帮助您快速配置和设置 Akash 以取得成功。如有更多问题，请访问 [akash docs](http://docs.akash.network/)。

1. 设置阿卡什
	- [deploy.yml](https://github.com/ovrclk/akash-postgres-restore/blob/master/deploy.yml)中设置环境变量并部署到 Akash 上。
	- 使用 Akash 提供的 URL 和端口，使用您在环境变量中提供的凭据连接到 Postgres 服务器。例如 
	
			cluster.ewr1p0.mainnet.akashian.io:31234
2. 将 Akash 与应用容器一起使用

	您可以将自己的应用程序添加到 deploy.yml 并将数据库端口设置在应用程序，仅用于本地服务
		
	例 deploy 文件类似

		services:
		  app: 
		    image: myappimage:v1
		    depends_on: 
		      - service: postgres
		  cron:
		    image: ghcr.io/ovrclk/akash-postgres-restore:v0.0.4
		    env:
		      - POSTGRES_PASSWORD=password
		      ...
		    depends_on:
		      - service: postgres
		  postgres:
		    image: postgres:12.6
		    env:
		      - POSTGRES_PASSWORD=password
		    expose:
		      - port: 5432
		        to:
		          - service: app
		          - service: cron
	环境变量和定义

		POSTGRES_USER=postgres - 数据库用户名称
		POSTGRES_PASSWORD=password - 数据库密码
		POSTGRES_HOST=postgres - 数据库主机名 (this will be whatever you named it in deploy.yml)
		POSTGRES_PORT=5432 - 数据库端口, will be 5432 unless you aliased it in deploy.yml
		POSTGRES_DATABASE=akash_postgres - 数据库名称
		BACKUP_PATH=bucketname/path - Filebase 桶部署路径 (make sure directories exist first)
		BACKUP_KEY=key - Filebase api key
		BACKUP_SECRET=secret - Filebase secret key
		BACKUP_PASSPHRASE=secret - 加密短语
		BACKUP_HOST=https://s3.filebase.com - S3 备份主机，默认 Filebase,可以设置其他 S3 备份主机
		BACKUP_SCHEDULE = */15 * * * * -备份的cron计划（默认为每15分钟）
		BACKUP_RETAIN = 7 days - 保留备份的天数
3. 部署

	您可以使用 Docker Compose 在本地运行应用程序。
	将 `.env.sample` 文件复制到 `.env` 并填入`Run docker-compose up `构建和运行应用程序。

## 边缘缓存技术
了解 Filebase 的边缘缓存技术。

所有 Filebase 客户，无论是免费的还是付费的，都可以使用我们的 Filebase 边缘缓存技术。这项技术改进了上传和下载请求，使它们比直接从去中心化网络访问相同文件更快。

Filebase 通过其独特的边缘平台提供对去中心化存储的访问。数据检索费用由这些存储网络收取。因此，Filebase 必须在执行下载时向客户收取出站带宽费用。

如果一个对象被频繁或最近访问过，那么它会被缓存在边缘层以便更快地访问。

如果对象被缓存，则以下情况为真：

- 下载性能显着提高。
- 如果该对象被频繁访问，它会在缓存中保持活跃状态​​ 7 天。
- 如果边缘节点繁忙，则可能会驱逐休眠的对象。
- HTTP 标 `x-cache-status` 头将返回一个值 `HIT`。未缓存的对象将返回 `MISS`。

## 安全
了解存储在 Filebase 上的对象的数据安全性。

所有存储在 Filebase 上的数据都具有与生俱来的高级别数据安全性。对象存储在分散的网络中。目前，我们支持 IPFS 和 Sia 网络。有关这些网络的更多信息，请参阅[文档](https://docs.filebase.com/what-is-filebase/storage-networks/what-is-the-difference-between-ipfs-and-sia)。

去中心化网络由不同的对等网络组成。这些网络由遍布全球的存储节点组成。

一些去中心化网络使用 [EC](https://docs.filebase.com/web3-education/deep-dives/deep-dive-erasure-coding)，也称为 Reed-Solomon 编码，是一种数据存储算法，允许将对象存储在部分而不是整体中。存储在去中心化存储网络中的每个对象都被分成数十或数百个部分，也称为分片。每个分片在数据传输期间和数据静止时都会被加密。

虽然数据可能分布在多个节点上，但只需要其中的一个片段就可以重新组合它。存储在节点上的每一位数据本身都是毫无价值的。这确保数据是可靠的，即使任何单个节点被损坏或破坏。

要访问该对象，您需要向存储网络提供个人加密密钥或哈希。然后网络检索存储在不同节点上的文件片段，将它们重新组合成单​​个对象，并将它们发送给用户。该对象仅对拥有加密密钥的人可用。
## API 端点
Filebase 服务端点的简要概述。
### S3 兼容的 API 端点
要将 Filebase 与 S3 兼容的 API 一起使用，您将需要一个 Filebase 端点。

对于所有端点，Filebase 保持严格的 HTTPS-only 标准。这意味着对象和 API 调用仅通过 HTTPS 提供。目前无法禁用此功能。
### 区域端点
区域名称|位置代码|端点|协议
---|---|---|---
美国东部（弗吉尼亚州文特山）|us-east-1|https://s3.filebase.com|HTTPS

Filebase 也可以通过基于浏览器的图形用户界面访问，而不是通过与 S3 兼容的 API。Filebase 的基于浏览器的图形用户界面可以在[这里](https://console.filebase.com/)找到的

### IPFS 固定服务 API 端点
有关 IPFS 固定服务 API 的信息，请查看此[文档](https://docs.filebase.com/api-documentation/ipfs-pinning-service-api)：
### 文件库平台 API 端点
有关 Filebase Platform API 的信息，请查看此文档：

- [IPFS 网关管理 API](https://docs.filebase.com/api-documentation/filebase-ipfs-gateway-management-api)
- [平台 API](https://docs.filebase.com/api-documentation/filebase-platform-apis)

## 存储网络
详细了解每个存储网络

- [IPFS](https://docs.filebase.com/what-is-filebase/storage-networks/ipfs)
- [sia](https://docs.filebase.com/what-is-filebase/storage-networks/sia)
- [如何选择](https://docs.filebase.com/what-is-filebase/storage-networks/should-i-use-ipfs-or-sia)
- [它们的区别](https://docs.filebase.com/what-is-filebase/storage-networks/what-is-the-difference-between-ipfs-and-sia)


