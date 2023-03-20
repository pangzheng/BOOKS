# IPFS-CAR
	了解如何将文件和文件夹转换为内容可寻址存档 (.car) 文件并通过 Filebase 将它们上传到 IPFS。
## 什么是 .car 文件？
.car 文件是一种包含多个文件的压缩存档文件，类似于 ZIP 文件。.car 文件由 FileCoin 和 IPFS 网络使用，它们利用元数据字段将每个文件的 IPFS CID 包含在存档中。

Filebase 支持通过 PutObject 或 MultipartUpload 方法使用与 S3 兼容的 API 将 .car 文件上传到 IPFS。

阅读下文，了解如何将文件和文件夹转换为内容可寻址存档 (.car) 文件，并通过 Filebase 将它们上传到 IPFS。

- 先决条件：
	- 下载和安装的AWS命令行界面。
	- 配置 AWS CLI 用于您的 Filebase 帐户。
	- 报名免费的 Filebase 帐户。
	- 创建文件库存储桶。了解如何创建存储桶.

步骤

1. 使用以下命令下载 ipfs-car 包：

		git clone https://github.com/web3-storage/ipfs-car
2. 安装 ipfs-car 包：

		npx ipfs-car
3. 接下来，使用ipfs-car将一个文件夹的文件打包成一个.car文件。

	这已通过包含 10,000 个或更多文件的 .car 档案进行了测试。根据您所需的工作流程，使用以下任一命令：

	- 将 .car 文件写入当前工作目录

			ipfs-car --pack path/to/file/or/dir
	- 将 .car 文件写入具有指定名称的特定目录

			ipfs-car --pack path/to/files --output path/to/write/a.car
4. 可以通过以下命令查看.car文件的内容：

	- 列出 .car 存档中每个文件的 CID：

			ipfs-car --list-cids path/to/my.car
	- 列出 .car 存档中每个文件的文件名：

			ipfs-car --list path/to/my.car
5. 接下来，使用以下 AWS CLI 命令将您的 .car 文件上传到 Filebase：

		aws --endpoint https://s3.filebase.com s3 cp file-name.car s3://bucket-name --debug --metadata 'import=car'

	替换 `file-name` 为您的 .car 文件名，并 `bucket-name` 替换为您的 Filebase IPFS 存储桶名称。
	
	- 下面是一个例子：

			aws --endpoint https://s3.filebase.com s3 cp TEST_CAR.car s3://ipfs-test --debug --metadata 'import=car'

	上传 .car 文件时，您可以 `x-amz-meta-import: car` 随请求一起传递 S3 元数据标头。Filebase 然后将文件作为 .car 导入，并返回生成的 CID。

	使用指定的 `–debug` 标志，显示响应标头，其中显示 CID：

		2022-06-14 19:51:41,400 - ThreadPoolExecutor-0_2 - botocore.parsers - DEBUG - Response headers: 
		{
		    'Date': 'Tue, 14 Jun 2022 23:51:41 GMT', 
		    'Content-Type': 'application/xml', 
		    'Transfer-Encoding': 'chunked', 
		    'Connection': 'keep-alive', 
		    'x-amz-meta-cid': 'bafybeieurpeyzighqrvwjqyj3szuvbqvrbijm7cdair5a422ipf2d5qnlq', 
		    'ETag': 'W/"d8cad258a3d9bbe03a26a13a3ec43b21"',
		    'x-amz-request-id': '144e0415-8162-45cd-b071-e51dada956ae', 
		    'x-amz-id-2': 'ZmlsZWJhc2UtNmQ3ZjQ5OGZmZC14ejk1Mg==',
		    'Server': 'Filebase'
		}

	要了解有关 ipfs-car 的更多信息以及使用 ipfs-car API 的其他用例，请参阅 ipfs-car 文档.
