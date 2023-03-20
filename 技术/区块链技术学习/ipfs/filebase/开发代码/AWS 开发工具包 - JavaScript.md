# [AWS 开发工具包](https://github.com/aws/aws-sdk-js-v3) - JavaScript
	了解如何将适用于 JavaScript 的 AWS 开发工具包与 Filebase 结合使用。
## 什么是 AWS 开发工具包 - JavaScript？
AWS SDK（软件开发工具包）通过支持和提供与 S3 兼容服务一起使用的代码对象，帮助简化编码和应用程​​序开发。有多种不同的 AWS 开发工具包，每种都适用于不同的编码语言。本指南涵盖 AWS 开发工具包 - JavaScript。

适用于 JavaScript 的 AWS 开发工具包支持三种运行时：

- JavaScript
- node.js
- 用于移动开发的 React Native

AWS SDK - JavaScript 还支持跨运行时。Cross-runtime 是一个服务客户端包，可以在浏览器、Node.js 和 React-Native 上运行，无需更改任何代码。

- 先决条件：
	- 如果使用 CLI，请在您的环境中下载并安装 npm
	- 下载安装适用于 Javascript 的 AWS 开发工具包。
	- 注册一个免费的 Filebase 帐户。
	- 拥有您的文件库访问权限和密钥。了解如何查看您的访问密钥。

本指南是使用 Ubuntu 20.04 编写的，并使用命令行界面进行了测试。如果您使用的是集成开发环境 (IDE)，则依赖项和模块的安装会有所不同。

## 声明 S3 客户端
以下代码示例定义了 S3 客户端，并且不返回任何输出。替换代码中的以下值以匹配您的配置：

- `accessKeyId` 您的文件库访问密钥
- `secretAccessKey` 您的文件库密钥

code

	const AWS = require('aws-sdk');
	
	const s3 = new AWS.S3({
	  apiVersion: '2006-03-01',
	  accessKeyId: 'Filebase-Access-Key',
	  secretAccessKey: 'Filebase-Secret-Key',
	  endpoint: 'https://s3.filebase.com',
	  region: 'us-east-1',
	  s3ForcePathStyle: true
	});
	
### 检索桶
以下代码示例返回您的 Filebase 帐户下的存储桶列表。替换代码中的以下值以匹配您的配置：

- `accessKeyId` 您的文件库访问密钥
- `secretAccessKey` 您的文件库密钥

code

	const AWS = require('aws-sdk');
	
	const s3 = new AWS.S3({
	  apiVersion: '2006-03-01',
	  accessKeyId: 'Filebase-Access-Key',
	  secretAccessKey: 'Filebase-Secret-Key',
	  endpoint: 'https://s3.filebase.com',
	  region: 'us-east-1',
	  s3ForcePathStyle: true
	});
	
	s3.listBuckets(function (err, data) {
	  if (err) {
	    console.log("Error when listing buckets", err);
	  } else {
	    console.log("Success when listing buckets", data);
	  }
	});

### 创建桶
以下代码示例创建一个新的 Filebase 存储桶。替换代码中的以下值以匹配您的配置：

- `accessKeyId`：您的文件库访问密钥
- `secretAccessKey`：您的文件库密钥
- `Bucket` ：要创建的文件库存储桶名称

存储桶名称在所有 Filebase 用户中必须是唯一的，长度在 3 到 63 个字符之间，并且只能包含小写字符、数字和破折号。
通过这种方法创建的桶将自动创建在 IPFS 网络上。

	const AWS = require('aws-sdk');
	
	const s3 = new AWS.S3({
	  apiVersion: '2006-03-01',
	  accessKeyId: 'Filebase-Access-Key',
	  secretAccessKey: 'Filebase-Secret-Key',
	  endpoint: 'https://s3.filebase.com',
	  region: 'us-east-1',
	  s3ForcePathStyle: true
	});
	
	const params = {
	    Bucket: "New-Filebase-Bucket"
	}
	
	s3.createBucket(params, (err, data)=>{
	    if(err){
	        console.log(err)
	    } else {
	        console.log("Bucket created", data)
	    }
	})
	
### 列出存储桶中的文件
以下代码示例返回所提供存储桶名称中的对象列表。替换代码中的以下值以匹配您的配置：

- `accessKeyId` 您的文件库访问密钥
- `secretAccessKey` 您的文件库密钥
- `Bucket` 您的文件库存储桶名称

code

	const AWS = require('aws-sdk');
	
	const s3 = new AWS.S3({
	  apiVersion: '2006-03-01',
	  accessKeyId: 'Filebase-Access-Key',
	  secretAccessKey: 'Filebase-Secret-Key',
	  endpoint: 'https://s3.filebase.com',
	  region: 'us-east-1',
	  s3ForcePathStyle: true
	});
	
	const params = {
	  Bucket: "filebase-bucket",
	  MaxKeys: 20
	};
	
	s3.listObjectsV2(params, function (err, data) {
	  if (err) {
	    console.log("Error when listing objects", err);
	  } else {
	    console.log("Success when listing objects", data);
	  }
	});

### 上传对象
以下代码示例将对象上传到提供的存储桶名称。替换代码中的以下值以匹配您的配置：

- accessKeyId：您的文件库访问密钥
- secretAccessKey：您的文件库密钥
- Bucket ：您的文件库存储桶名称
- Key : 待上传对象的本地路径
- Content Type：上传对象的类型

code

	const AWS = require('aws-sdk');
	
	const s3 = new AWS.S3({
	  apiVersion: '2006-03-01',
	  accessKeyId: 'Filebase-Access-Key',
	  secretAccessKey: 'Filebase-Secret-Key',
	  endpoint: 'https://s3.filebase.com',
	  region: 'us-east-1',
	  s3ForcePathStyle: true
	});
	
	const params = {
	  Bucket: 'filebase-bucket',
	  Key: '/path/to/file/filename.png',
	  ContentType: 'image/png',
	  Body: myPictureFile,
	  ACL: 'public-read',
	};
	
	const request = s3.putObject(params);
	request.send();