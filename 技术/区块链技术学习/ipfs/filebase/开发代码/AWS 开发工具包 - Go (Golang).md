# [AWS SDK - Go](https://github.com/aws/aws-sdk-go) (Golang)
	了解如何将适用于 Go (Golang) 的 AWS SDK与 Filebase 结合使用。
## 什么是 AWS SDK - Go？
AWS SDK（软件SDK）通过支持和提供与 S3 兼容服务一起使用的代码对象，帮助简化编码和应用程​​序开发。有多种不同的 AWS SDK，每种都适用于不同的编码语言。本指南涵盖 AWS SDK - Go。

阅读下文以了解如何将适用于 Go 的 AWS SDK与 Filebase 结合使用。

- 先决条件：
	- 在您的环境中下载安装 go
	- 下载安装适用于 Go 的 AWS SDK。
	- 注册一个免费的 Filebase 帐户。
	- 拥有您的文件库访问权限和密钥。了解如何查看您的访问密钥。

本指南是使用 Ubuntu 20.04 编写的，并使用命令行界面进行了测试。如果您使用的是集成开发环境 (IDE)，则依赖项和模块的安装会有所不同。

## 创建一个桶
以下代码示例创建一个新的 Filebase 存储桶。替换代码中的以下值以匹配您的配置：

- `filebase-access-key` 您的文件库访问密钥
- `filebase-secret-key` 你的文件库密钥
- `new-filebase-bucket` 预期的新存储桶名称

存储桶名称在所有 Filebase 用户中必须是唯一的，长度在 3 到 63 个字符之间，并且只能包含小写字符、数字和破折号。

通过这种方法创建的桶将自动创建在 IPFS 网络上。

	package main
	
	import (
		"fmt"
		"github.com/aws/aws-sdk-go/aws"
		"github.com/aws/aws-sdk-go/aws/session"
		"github.com/aws/aws-sdk-go/aws/credentials"
		"github.com/aws/aws-sdk-go/service/s3"
	)
	
	func main() {
	//// create a configuration
		s3Config := aws.Config{
		Credentials:      credentials.NewStaticCredentials("filebase-access-key", "filebase-secret-key", ""),
		Endpoint:         aws.String("https://s3.filebase.com"),
		Region:           aws.String("us-east-1"),
		S3ForcePathStyle: aws.Bool(true),
	}
	
	goSession, err := session.NewSessionWithOptions(session.Options{
		Config:  s3Config,
		Profile: "filebase",
	})
	
	// check if the session was created correctly.
	
	if err != nil {
		fmt.Println(err)
	}
	
	// create a s3 client session
	s3Client := s3.New(goSession)
	
	// set parameter for bucket name
	bucket := aws.String("new-filebase-bucket")
	
	// create a bucket
	_, err = s3Client.CreateBucket(&s3.CreateBucketInput{
		Bucket: bucket,
	})
	
	// print if there is an error
	if err != nil {
		fmt.Println(err.Error())
	return
	}
	}

## 上传对象
以下代码示例将对象上传到指定的存储桶。替换代码中的以下值以匹配您的配置：

- `filebase-access-key` 您的文件库访问密钥
- `filebase-secret-key` 你的文件库密钥
- `bucket-name` 您的文件库存储桶名称
- `/path/to/object/to/upload` 要上传的文件的路径
- `object-name` 要上传的对象的名称

上传

	package main
	
	import (
		"fmt"
		"github.com/aws/aws-sdk-go/aws"
		"github.com/aws/aws-sdk-go/aws/session"
		"github.com/aws/aws-sdk-go/aws/credentials"
		"github.com/aws/aws-sdk-go/service/s3"
		"os"
	)
	
	func main() {
	//// create a configuration
		s3Config := aws.Config{
		Credentials:      credentials.NewStaticCredentials("filebase-access-key", "filebase-secret-key", ""),
		Endpoint:         aws.String("https://s3.filebase.com"),
		Region:           aws.String("us-east-1"),
		S3ForcePathStyle: aws.Bool(true),
	}
	
	// create a new session using the config above and profile
	goSession, err := session.NewSessionWithOptions(session.Options{
		Config:  s3Config,
		Profile: "filebase",
	})
	
	// check if the session was created correctly.
	
	if err != nil {
		fmt.Println(err)
	}
	
	// create a s3 client session
	s3Client := s3.New(goSession)
	
	//set the file path to upload
	file, err := os.Open("/path/to/object/to/upload")
	if err != nil {
		fmt.Println(err.Error())
	return
	}
	
	defer file.Close()
	// create put object input
		putObjectInput := &s3.PutObjectInput{
		Body:   file,
		Bucket: aws.String("bucket-name"),
		Key:    aws.String("object-name"),
	}
	
	// upload file
	_, err = s3Client.PutObject(putObjectInput)
	
	// print if there is an error
	if err != nil {
		fmt.Println(err.Error())
	return
	}
	}


## 下载对象
以下代码示例从指定的存储桶下载一个对象。替换代码中的以下值以匹配您的配置：

- `filebase-access-key` 您的文件库访问密钥
- `filebase-secret-key` 你的文件库密钥
- `bucket-name` 您的文件库存储桶名称
- `object-name` 要下载的对象的名称

下载

	package main
	
	import (
	"fmt"
	"github.com/aws/aws-sdk-go/aws"
	"github.com/aws/aws-sdk-go/aws/session"
	"github.com/aws/aws-sdk-go/aws/credentials"
	"github.com/aws/aws-sdk-go/service/s3"
	)
	
	func main() {
	//// create a configuration
		s3Config := aws.Config{
		Credentials:      credentials.NewStaticCredentials("filebase-access-key", "filebase-secret-key", ""),
		Endpoint:         aws.String("https://s3.filebase.com"),
		Region:           aws.String("us-east-1"),
		S3ForcePathStyle: aws.Bool(true),
	}
	
	// create a new session using the config above and profile
	goSession, err := session.NewSessionWithOptions(session.Options{
		Config:  s3Config,
		Profile: "filebase",
	})
	
	// check if the session was created correctly.
	if err != nil {
		fmt.Println(err)
	}
	
	// create a s3 client session
	s3Client := s3.New(goSession)
	
	// create put object input
	getObjectInput := &s3.GetObjectInput{
		Bucket: aws.String("bucket-name"),
		Key:    aws.String("object-name"),
	}
	
	// get file
	_, err = s3Client.GetObject(getObjectInput)
	
	// print if there is an error
	if err != nil 
		fmt.Println(err.Error())
	return
	}
	}


## 删除对象
以下代码示例从指定的存储桶中删除一个对象。替换代码中的以下值以匹配您的配置：

- `filebase-access-key`：您的文件库访问密钥
- `filebase-secret-key`：你的文件库密钥
- `bucket-name`：您的文件库存储桶名称
- `object-name`：要删除的对象的名称

删除

	package main
	
	import (
		"fmt"
		"github.com/aws/aws-sdk-go/aws"
		"github.com/aws/aws-sdk-go/aws/session"
		"github.com/aws/aws-sdk-go/aws/credentials"
		"github.com/aws/aws-sdk-go/service/s3"
	)
	
	func main() {
	//// create a configuration
		s3Config := aws.Config{
		Credentials:      credentials.NewStaticCredentials("filebase-access-key", "filebase-secret-key", ""),
		Endpoint:         aws.String("https://s3.filebase.com"),
		Region:           aws.String("us-east-1"),
		S3ForcePathStyle: aws.Bool(true),
	}
	
	// create a new session using the config above and profile
	goSession, err := session.NewSessionWithOptions(session.Options{
		Config:  s3Config,
		Profile: "filebase",
	})
	
	// check if the session was created correctly.
	if err != nil {
		fmt.Println(err)
	}
	
	// create a s3 client session
	s3Client := s3.New(goSession)
	
	// create put object input
	deleteObjectInput := &s3.DeleteObjectInput{
		Bucket: aws.String("bucket-name"),
		Key:    aws.String("object-name"),
	}
	
	// get file
	_, err = s3Client.DeleteObject(deleteObjectInput)
	
	// print if there is an error
	if err != nil {
		fmt.Println(err.Error())
	return
	}
	}