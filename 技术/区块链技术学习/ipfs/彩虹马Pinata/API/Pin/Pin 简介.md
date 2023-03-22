# Pin
Pinning 端点适用于所有涉及通过 Pinata 固定到 IPFS 的文件的操作。这些端点通常应在服务器环境中使用，以确保您的 API 密钥受到保护。

对于导致文件被上传或添加到帐户的所有操作，可以包含可选的数据片段。

- `pinataOptions `

	pinata 选项 
	
	Pinata 支持在固定文件时添加其他选项。此端点当前支持以下选项：

	- `cidVersion`

		IPFS 在为您的内容创建哈希时将使用的 CID 版本。有效选项是：“0”(CIDv0) 或“1”(CIDv1)
	- `wrapWithDirectory`

		添加到 IPFS 时将您的内容包装在目录中。这允许用户通过文件名而不是哈希来检索内容。有关更详细的解释，请参阅[这里](https://flyingzumwalt.gitbooks.io/decentralized-web-primer/content/files-on-ipfs/lessons/wrap-directories-around-content.html)。有效选项是：true 或 false

	这是要创建 `pinataOptions` 的对象的示例 ：

		pinataOptions: {
		    cidVersion: 0, 
		    wrapWithDirectory: false
		}
- `pinataMetadata`

	Pinata 原数据

	除了将文件固定到 Pinata 之外，您还可以选择包含元数据供 Pinata 存储。
此元数据稍后可用于轻松查询您通过我们 [userPinList](https://docs.pinata.cloud/pinata-api/pinning)的请求固定的内容。

	可选的元数据对象采用以下形式：

		{
		    name: A custom name. If you don't provide this value, it will automatically be populated by the original name of the file you've uploaded,
		    keyvalues: {
		        customKey: customValue,
		        customKey2: customValue2
		    }
		}

	键值对象允许用户为上传的文件提供他们想要的任何自定义键/值对。这些值可以是：

	- 字符串
	- 数字（整数或小数）
	- 日期（以 ISO_8601 格式提供）

	截至目前，这仅限于 10 个键/值对，但如果这对您来说有问题，请告诉我们！

	- 真实世界的 JSON 示例

		例如，假设您正在为律师构建一项服务并希望使用律师的 ID、律师客户的 ID、收费代码和所提供服务的成本来标记每个 IPFS 上传。
您的 `pinataMetadata` 对象可能如下所示：

			{
			    name: 'ExampleNameOfDocument.pdf'
			    keyvalues: {
			        LawyerName: 'Lawyer001',
			        ClientID: 'Client002',
			        ChargeCode: 'Charge003'
			        Cost: 100.00
			    }
			}
