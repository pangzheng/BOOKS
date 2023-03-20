# Tableland
	了解如何使用 Tableland SQL 跟踪存储在 IPFS 文件库存储桶中的对象的对象名称、类型和 IPFS CID。
## 什么是 Tableland？
Tableland 是一个 SQL 数据库和网络协议，专为在以太坊网络上使用而构建。Tableland 可用于存储从 tokenURI 元数据到 NFT 扩展或 DAO 工具的任何内容。使用 Tableland 创建和存储的表使用用户的以太坊钱包地址进行身份验证，因此不需要用户名、密码或访问密钥。

Tableland 可用于记录和跟踪存储在 Filebase IPFS 存储桶中的对象的元数据。存储在 IPFS 网络上的对象有一个关联的 IPFS CID，它对每个文件都是唯一的，但不像典型的文件 ID 或存储路径那样容易识别。为了解决这个问题，您可以使用 Tableland 创建一个数据库来跟踪您的对象及其关联的文件名、类型和 IPFS CID，这样您就可以在需要快速引用对象时查询它。

阅读下文以了解如何使用 Tableland SQL 跟踪存储在 IPFS 文件库存储桶中的对象的对象名称、类型和 IPFS CID。

- 先决条件：
	- 使用他们的注册 Tableland 抢先体验指示。此过程涉及将您的以太坊公共钱包地址列入白名单以用于 Tableland API。
	- 下载并安装 Tableland CLI 工具：npm i -g @textile/tableland-cli
	- 注册的免费的 Filebase 帐户。
	- 拥有您的文件库访问权限和密钥。了解如何查看您的访问密钥.
	- 在 IPFS 网络上创建一个 filebase 存储桶。有关如何在 IPFS 网络上创建存储桶的说明，请参阅.

## 步骤
1. 一旦您的 ETH 钱包地址被列入白名单以用于 Tableland，您将需要获取您的私钥字符串以与以太坊交互以创建一个新表。

	你可以关注[说明](https://metamask.zendesk.com/hc/en-us/articles/360015289632-How-to-Export-an-Account-Private-Key)从 Metamask 获取您的私钥字符串。
2. 将您的私钥字符串导出为环境变量，这样您就不必一直粘贴它了。

		export TBL_PRIVATE_KEY=[PRIVATE_KEY_STRING]
	替换 `PRIVATE_KEY_STRING` 为您的私钥字符串。
3. 使用以下 CLI 命令在 Tableland 上创建一个名为 AssetTracker 的表。

		tableland create "CREATE TABLE AssetTracker (id INT PRIMARY KEY, name TEXT, type TEXT, cid TEXT, provider TEXT, url TEXT);" --description="Filebase Asset Tracker"
	该查询的响应是您的新表名！您稍后需要引用它来更新和查询表。它应该类似于以下内容：

		{
		  "name": "assettracker_192
		}
	如果您有自己的 Infura、Alchemy 等 API 密钥，则可以避免使用上述命令打印的警告消息。详情请见。`tableland create —help`
4. 使用查询 CLI 命令插入一些要查询的数据。

		tableland query "INSERT INTO assettracker_192 VALUES (0, 'filebase_robot.png', 'PNG', 'bafybeict7kegxaugjue5rcys65islddi2rnzmj2hh2wfq3wynf7t772hy4', 'filebase.com', '<https://bafybeict7kegxaugjue5rcys65islddi2rnzmj2hh2wfq3wynf7t772hy4.ipfs.dweb.link>');"

	替换 `assettracker_192` 为您在上一步中收到的值。下表包含一些您可能想要包括的附加对象。

	示例数据库表：

	id|name|type|cid|provider|url
	---|---|---|---|---|---|
	1|filebase_robot.png|bafybeict7kegxaugjue5rcys65islddi2rnzmj2hh2wfq3wynf7t772hy4|https://ipfs.filebase.io/ipfs/bafybeict7kegxaugjue5rcys65islddi2rnzmj2hh2wfq3wynf7t772hy4
	2|filebase_logo.png|bafkreighyv7jppuyen6kvdw3lhnleydibj44wej3ejq2j7ndwd3hsa7oam|https://ipfs.filebase.io/ipfs/bafkreighyv7jppuyen6kvdw3lhnleydibj44wej3ejq2j7ndwd3hsa7oam
5. 要查询数据库，请使用以下 CLI 命令：

		tableland query "SELECT * FROM assettracker_192;"
	此示例查询数据库中的所有条目。您可以修改它以反映您对单个对象或特定对象条件的所需查询。

有关 Tableland 的更多信息，请查看他们的[文档](https://docs.tableland.xyz/).