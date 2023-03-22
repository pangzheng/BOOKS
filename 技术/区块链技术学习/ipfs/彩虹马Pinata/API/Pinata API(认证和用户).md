# Pinata API
Pinata API 是使用 Pinata 的主要接口。它负责所有 IPFS 固定功能、列出内容、添加元数据等。
## 验证
要连接到 Pinata API，您需要生成 Pinata API 密钥。访问 [Pinata API](https://app.pinata.cloud/keys) 页面以生成新密钥。

当您单击“新 API 密钥”时，系统将提示您选择权限和您生成的密钥的使用次数。如您所料，管理员权限可以访问所有 API 端点。如果您想指定特定的端点，您可以通过扩展端点的父路由并打开权限来实现。

默认情况下，所有密钥都可以无限使用。但是，如果您想限制一个密钥的使用次数，您可以通过设置 Max Uses 字段来实现。

通过设置密钥名称，您将能够轻松识别密钥及其用途。

单击 Revoke 按钮可以撤销任何密钥的访问权限。密钥一旦被撤销，就不能再用于任何目的。

<iframe width="745" height="419" src="https://www.youtube.com/embed/l4vPAeBtdms" title="How to Create a Pinata API Key" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen></iframe>

### 重要的
当您生成密钥时，您将看到一个带有 Pinata API Key 、Pinata API Secret 和 JWT 的模式。

您的“Pinata API 密钥”充当我们 REST API 的公钥，您的“Pinata Secret API 密钥”充当您公钥的密码。JWT 是两者的编码组合。请务必将您的秘密密钥保密。

为了增加客户的安全性，这些密钥在 Pinata 端进行了加密，并且只会显示一次，因此请将它们记下来。如果您丢失了这些值，则需要撤销密钥并创建一个新密钥。
### 连接到 Pinata API
Pinata 请求的基本 URL 是：https://api.pinata.cloud

您有两种连接到 Pinata API 的方法：

- `Bearer Authentication`

	根据网络搜索结果，Bearer Authentication（也称为 Token 认证）是一种 HTTP 认证方案，涉及称为 bearer tokens 的安全 Token。Bearer Authentication 的名称可以理解为“给予此 Token 持有者访问权限”。`bearer token` 是一个神秘的字符串，通常由服务器在响应登录请求时生成。它是一种用于访问 OAuth 2.0 保护的资源的方法。
- `Custom Headers`

要使用不记名身份验证模型，您将需要在创建 API 密钥时生成的 JWT。此 Token 可用作 `Authorization` 以下格式的所有 API 请求的标头：

	"Authorization": "Bearer YOUR_JWT"

- `Get https://api.pinata.cloud/data/testAuthentication`

	测试您的 API 密钥和连接到 Pinata API 的能力。

	- 参数
		- Header 

			Authorization* 		 PINATA-JWT
	- 响应
		- 200

				{
				    message: "Congratulations! You are communicating with the Pinata API!"
				}	

### Custom Headers
如果 `Bearer Authentication` 验证，您的 API 请求将需要包含两个标头：

- `pinata_api_key` (把你的个人 pinata api 密钥放在这里） 
- `pinata_secret_api_key` （将您的个人 pinata 秘密 api 密钥放在这里） 

- Get `https://api.pinata.cloud/data/testAuthentication`

	测试您的 API 密钥和连接到 Pinata API 的能力。

	- 参数
		- Header 
			- `pinata_api_key` *    PINATA-API-密钥
			- `pinata_secret_api_key` *   PINATA 秘密 API 密钥
	- 响应
		- 200

				{
				    message: "Congratulations! You are communicating with the Pinata API!"
				}

展望未来，为简单起见，所有示例都将使用 `JWT Bearer Authentication` 格式。

#### 测试认证
让我们尝试通过 `data/testAuthentication` 端点连接到 Pinata API（此端点需要管理密钥才能调用）。

- curl

		curl --location --request GET 'https://api.pinata.cloud/data/testAuthentication' \
		--header 'Authorization: Bearer PINATA_JWT'
- Node.js

		var axios = require('axios');
		
		var config = {
		  method: 'get',
		  url: 'https://api.pinata.cloud/data/testAuthentication',
		  headers: { 
		    'Authorization': 'Bearer PINATA_JWT'
		  }
		};
		
		const res = await axios(config)
		
		console.log(res.data);
- python

		import requests
		url = "https://api.pinata.cloud/data/testAuthentication"
		payload={} headers = { 'Authorization': 'Bearer PINATA_JWT' }
		response = requests.request("GET", url, headers=headers, data=payload)
		print(response.text)		
- GO

		package main
		import ( "fmt" "net/http" "io/ioutil" )
		func main() {
		url := "https://api.pinata.cloud/data/testAuthentication" method := "GET"
		client := &http.Client { } req, err := http.NewRequest(method, url, nil)
		if err != nil { fmt.Println(err) return } req.Header.Add("Authorization", "Bearer PINATA_JWT")
		res, err := client.Do(req) if err != nil { fmt.Println(err) return } defer res.Body.Close()
		body, err := ioutil.ReadAll(res.Body) if err != nil { fmt.Println(err) return } fmt.Println(string(body)) }GoJavaScript with Axios example:

## 用户
用户端点允许您为 Pinata API 创建和管理 API 密钥。请记住，这些密钥与 Submarine API 是分开的。

这些端点对于创建作用域密钥和撤销密钥很有用。
### 生成 Pinata API 密钥
	/users/generateApiKey
此端点用于以编程方式生成 Pinata API 密钥。只能使用 “Admin” 键调用此端点。生成新密钥时，可以实施特定范围和限制。

该端点将返回三个值：API Key、API Secret 和 JWT Bearer Token。有关这些值的[更多信息](https://docs.pinata.cloud/pinata-api/authentication).

	确保记录 API Secret 和 JWT，因为它们将无法再次访问。
- 生成 API 密钥

	生成 Pinata API 密钥时的请求正文将如下所示：

		{
		    keyName: (一个便于引用的新密钥的名称 - 必须),
		    maxUses: (可以使用新api键的次数 - 选填),
		    permissions: {
		      admin: boolean,
		      endpoints: {
		        data: {
		          pinList: boolean,
		          userPinnedDataTotal: boolean
		        },
		        pinning: {
		          hashMetadata: boolean,
		          hashPinPolicy: boolean,
		          pinByHash: boolean,
		          pinFileToIPFS: boolean,
		          pinJSONToIPFS: boolean,
		          pinJobs: boolean,
		          unpin: boolean,
		          userPinPolicy: boolean
		        }
		      }
		    }
		}

	注意 `keyName` 是必需的。设置权限时，必须包括所有属性和子属性，除非您要创建管理密钥。如果您正在创建管理密钥，则 `endpoints` 可以省略属性和子属性。如果您要包含该 `endpoints` 属性，则不能包含该 `admin`属性。
的

	例如，这将是管理密钥生成的简化主体：

		{
		    keyName: "My admin key",
		    permissions: {
		      admin: true
		    }
		}

	- `POST https://api.pinata.cloud/users/generateApiKey`
		- 参数
			- Header 
				- `Authorization*`      Bearer PINATA-JWT
			- Body
				- `keyName*`
				- `permissions*`     Stringified 对象
				- `maxUses`

		- 响应
			- 200

					{
					    "pinata_api_key": "KEY",
					    "pinata_api_secret": "SECRET",
					    "JWT": "JWT"
					}

#### 测试验证
- CURL
	
		curl --location --request POST 'https://api.pinata.cloud/users/generateApiKey' \
		--header 'Authorization: Bearer PINATA JWT' \
		--header 'Content-Type: application/json' \
		--data-raw '{
		    "keyName": "My Key",    
		    "permissions": {
		      "endpoints": {
		        "data": {
		          "pinList": false,
		          "userPinnedDataTotal": false
		        },
		        "pinning": {
		          "hashMetadata": true,
		          "hashPinPolicy": false,
		          "pinByHash": true,
		          "pinFileToIPFS": true,
		          "pinJSONToIPFS": true,
		          "pinJobs": false,
		          "unpin": false,
		          "userPinPolicy": false
		        }
		      }
		    }
		}'
- Node.js

		var axios = require('axios');
		var data = JSON.stringify({
		  "keyName": "My Key",
		  "permissions": {
		    "endpoints": {
		      "data": {
		        "pinList": false,
		        "userPinnedDataTotal": false
		      },
		      "pinning": {
		        "hashMetadata": true,
		        "hashPinPolicy": false,
		        "pinByHash": true,
		        "pinFileToIPFS": true,
		        "pinJSONToIPFS": true,
		        "pinJobs": false,
		        "unpin": false,
		        "userPinPolicy": false
		      }
		    }
		  }
		});
		
		var config = {
		  method: 'post',
		  url: 'https://api.pinata.cloud/users/generateApiKey',
		  headers: { 
		    'Authorization': 'Bearer PINATA JWT', 
		    'Content-Type': 'application/json'
		  },
		  data : data
		};
		
		const res = await axios(config);
		
		console.log(res.data);
- Python

		import requests
		import json
		
		url = "https://api.pinata.cloud/users/generateApiKey"
		
		payload = json.dumps({
		  "keyName": "My Key",
		  "permissions": {
		    "endpoints": {
		      "data": {
		        "pinList": False,
		        "userPinnedDataTotal": False
		      },
		      "pinning": {
		        "hashMetadata": True,
		        "hashPinPolicy": False,
		        "pinByHash": True,
		        "pinFileToIPFS": True,
		        "pinJSONToIPFS": True,
		        "pinJobs": False,
		        "unpin": False,
		        "userPinPolicy": False
		      }
		    }
		  }
		})
		headers = {
		  'Authorization': 'Bearer PINATA JWT',
		  'Content-Type': 'application/json'
		}
		
		response = requests.request("POST", url, headers=headers, data=payload)
		
		print(response.text)
- GO

		package main
		
		import (
		  "fmt"
		  "strings"
		  "net/http"
		  "io/ioutil"
		)
		
		func main() {
		
		  url := "https://api.pinata.cloud/users/generateApiKey"
		  method := "POST"
		
		  payload := strings.NewReader(`{
		    "keyName": "My Key",    
		    "permissions": {
		      "endpoints": {
		        "data": {
		          "pinList": false,
		          "userPinnedDataTotal": false
		        },
		        "pinning": {
		          "hashMetadata": true,
		          "hashPinPolicy": false,
		          "pinByHash": true,
		          "pinFileToIPFS": true,
		          "pinJSONToIPFS": true,
		          "pinJobs": false,
		          "unpin": false,
		          "userPinPolicy": false
		        }
		      }
		    }
		}`)
		
		  client := &http.Client {
		  }
		  req, err := http.NewRequest(method, url, payload)
		
		  if err != nil {
		    fmt.Println(err)
		    return
		  }
		  req.Header.Add("Authorization", "Bearer PINATA JWT")
		  req.Header.Add("Content-Type", "application/json")
		
		  res, err := client.Do(req)
		  if err != nil {
		    fmt.Println(err)
		    return
		  }
		  defer res.Body.Close()
		
		  body, err := ioutil.ReadAll(res.Body)
		  if err != nil {
		    fmt.Println(err)
		    return
		  }
		  fmt.Println(string(body))
		}

### 列出 Pinata API 密钥
	/users/apiKeys
此端点用于以编程方式列出所有活动的 Pinata API 密钥。只能使用“Admin”键调用此端点。

此端点将一次返回包含 10 个 API 密钥的分页列表。您可以使用偏移量查询参数对键进行分页。
#### 列出 API 密钥
- `GET https://api.pinata.cloud/users/apiKeys`
	- 参数
		- Query
			- `offset`	开始偏移的键数(即10)		
		- Header 
			- `Authorization*`      Bearer PINATA-JWT
	- 响应
		- 200

				{
				    "keys": [
				        {
				            "id": "string",
				            "name": "string",
				            "key": "string",
				            "secret": "string",
				            "max_uses": number,
				            "uses": number,
				            "user_id": "string",
				            "scopes": {
				                "admin": true
				            },
				            "revoked": false,
				            "createdAt": "2022-06-01T16:25:08.473Z",
				            "updatedAt": "2022-06-01T16:25:08.473Z"
				        }
				    ],
				    "count": 1
				}

#### 测试验证
- Curl

		curl --location --request GET 'https://api.pinata.cloud/users/apiKeys' \
		--header 'Authorization: Bearer PINATA JWT'
- Node.js

		var axios = require('axios');
		
		var config = {
		  method: 'get',
		  url: 'https://api.pinata.cloud/users/apiKeys',
		  headers: { 
		    'Authorization': 'Bearer PINATA JWT'
		  }
		};
		
		const res = await axios(config);
		
		console.log(res.data);
- Python

		import requests
		
		url = "https://api.pinata.cloud/users/apiKeys"
		
		payload={}
		headers = {
		  'Authorization': 'Bearer PINATA JWT'
		}
		
		response = requests.request("GET", url, headers=headers, data=payload)
		
		print(response.text)
- Go

		package main
		
		import (
		  "fmt"
		  "net/http"
		  "io/ioutil"
		)
		
		func main() {
		
		  url := "https://api.pinata.cloud/users/apiKeys"
		  method := "GET"
		
		  client := &http.Client {
		  }
		  req, err := http.NewRequest(method, url, nil)
		
		  if err != nil {
		    fmt.Println(err)
		    return
		  }
		  req.Header.Add("Authorization", "Bearer PINATA JWT")
		
		  res, err := client.Do(req)
		  if err != nil {
		    fmt.Println(err)
		    return
		  }
		  defer res.Body.Close()
		
		  body, err := ioutil.ReadAll(res.Body)
		  if err != nil {
		    fmt.Println(err)
		    return
		  }
		  fmt.Println(string(body))
		}

### 撤销 Pinata API 密钥
	/users/revokeApiKey
此端点用于以编程方式撤销 Pinata API 密钥。只能使用“Admin”键调用此端点。
#### 撤销 API 密钥
撤销 Pinata API 密钥需要公共 API 密钥，而不是秘密或 JWT。

- `PUT https://api.pinata.cloud/users/revokeApiKey`
	- 参数
		- Header 
			- `Authorization*`      Bearer PINATA-JWT
		- Body
			- `apiKey` 			要撤销公共API密钥  
	- 响应
		- 200

				"Revoked"
		
#### 测试实验
- Curl

		curl --location --request PUT 'https://api.pinata.cloud/users/revokeApiKey' \
		--header 'Authorization: Bearer PINATA JWT' \
		--header 'Content-Type: application/json' \
		--data-raw '{
		    "apiKey": "API KEY TO REVOKE"
		}'
- Node.js

		var axios = require('axios');
		var data = JSON.stringify({
		  "apiKey": "API KEY TO REVOKE"
		});
		
		var config = {
		  method: 'put',
		  url: 'https://api.pinata.cloud/users/revokeApiKey',
		  headers: { 
		    'Authorization': 'Bearer PINATA JWT', 
		    'Content-Type': 'application/json'
		  },
		  data : data
		};
		
		const res = await axios(config);
		
		console.log(res.data);
- Python

		import requests
		import json
		
		url = "https://api.pinata.cloud/users/revokeApiKey"
		
		payload = json.dumps({
		  "apiKey": "API KEY TO REVOKE"
		})
		headers = {
		  'Authorization': 'Bearer PINATA JWT',
		  'Content-Type': 'application/json'
		}
		
		response = requests.request("PUT", url, headers=headers, data=payload)
		
		print(response.text)
- GO

		package main
		
		import (
		  "fmt"
		  "strings"
		  "net/http"
		  "io/ioutil"
		)
		
		func main() {
		
		  url := "https://api.pinata.cloud/users/revokeApiKey"
		  method := "PUT"
		
		  payload := strings.NewReader(`{
		    "apiKey": "API KEY TO REVOKE"
		}`)
		
		  client := &http.Client {
		  }
		  req, err := http.NewRequest(method, url, payload)
		
		  if err != nil {
		    fmt.Println(err)
		    return
		  }
		  req.Header.Add("Authorization", "Bearer PINATA JWT")
		  req.Header.Add("Content-Type", "application/json")
		
		  res, err := client.Do(req)
		  if err != nil {
		    fmt.Println(err)
		    return
		  }
		  defer res.Body.Close()
		
		  body, err := ioutil.ReadAll(res.Body)
		  if err != nil {
		    fmt.Println(err)
		    return
		  }
		  fmt.Println(string(body))
		}

## SDK
Pinata 已经构建了一些官方 SDK 来帮助使用 Pinata 进行更容易的开发。此处引用了这些 SDK 以及一些社区构建的 SDK。
### 官方 SDK
- Pinata API 节点 SDK

	官方 Node.js 开发工具包。这是开始使用 Pinata 进行开发的最简单方法。
	
	- [@pinata/sdk/npm](https://www.npmjs.com/package/@pinata/sdk)
- Pinata 上传 CLI

	这是一个专为上传内容而构建的命令行工具。它可用于上传和固定文件和目录，也可用于上传和 Submarine 文件和目录。
	
	- [pinata-upload-cli/npm](https://www.npmjs.com/package/pinata-upload-cli)
- Pinata Submarine SDK

	这是用于使用的官方 Submarine API SDK. 它简化了开发并以简单的方式实现了复杂的工作流程。
	
	- [pinata-submarine/npm](https://www.npmjs.com/package/pinata-submarine)
- IPFS 网关工具 SDK

	这是一个方便的 SDK，允许开发人员获取任何类型的 IPFS URL  并将其转换为他们选择的网关 URL。当您想通过自己的方式加载 IPFS 内容时，这尤其有用.
	
	- [ipfs-gateway-tools/npm](https://www.npmjs.com/package/@pinata/ipfs-gateway-tools)

### 社区 SDK
这些 SDK 由 Pinata 社区的成员编写。它们未经 Pinata 的任何人审计，我们也不能保证它们的安全性或可行性。

- Python	
	- [pinatapy](https://github.com/Vourhey/pinatapy)
	- [pinata-python-util](https://github.com/edmondyu/pinata-python-util)
- GO
	- [pinata-cli](https://gitlab.com/benoit74/pinata-cli) 
- RUST
	- [pinata-sdk](https://github.com/perfectmak/pinata-sdk)
- .NET
	- [Pinata.Client](https://www.nuget.org/packages/Pinata.Client/) 
- Elixir
	- [ex_pinata](https://github.com/m1ome/ex_pinata)
