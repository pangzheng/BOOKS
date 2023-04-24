# Submarine API
Submarine 是上传保持私有且脱离公共 IPFS 网络的内容的过程。你可以这里了解[方案详情](https://www.pinata.cloud/blog/introducing-submarining-what-it-is-why-you-need-it). 使用 Pinata Submarine，您可以将内容保密并以编程方式生成访问 Token，只要您指定就可以持续。

此 API 可用于上传文件和文件夹、列出内容以及生成访问 Token以查看和下载内容。

查看 Submarine 内容需要专用网关. 通过专用网关查看 Submarined 内容时，您必须包含有效的访问 Token。可以使用生成访问 Token. 这是你应该如何构建一个链接来查看 Submarined 内容：

- 单个文件

		GATEWAY_URL/ipfs/FILE_CID?accessToken=TOKEN
- 文件夹内的文件

		GATEWAYURL/ipfs/FOLDER_CID/FILE_NAME?accessToken=TOKEN
		
## 验证
要连接到 Pinata Submarine API，您需要生成一个 Submarine API 密钥。访问页面以生成新密钥。

当点击“新建密钥”时，会生成一个新的密钥，可以直接从表中复制密钥。

单击 Revoke 按钮可以撤销任何密钥的访问权限。密钥一旦被撤销，就不能再用于任何目的。
### 连接到 Submarine API
Pinata 请求的基本 URL 是：https://managed.mypinata.cloud/api/v1

身份验证通过客户标头进行：

	x-api-key: YOUR SUBMARINE KEY
## 授权
auth 端点允许您为 Submarined 文件和文件夹生成访问Token。

### 生成访问 Token
	/auth/content/jwt
此端点允许您创建一个访问 Token，该 Token 将用于通过专用网关查看Submarined内容。Token 是有时间限制的，你通过这个端点来控制时间。

- `POST https://managed.mypinata.cloud/api/v2/auth/content/jwt`

	生成访问 Token
	
	- 参数
		- Header 
			- `x-api-key*`      SUBMARINE KEY
		- Body
			- `timeoutSeconds*`		访问 Token 应该有效的秒数
			- `contentIds*`     文件的字符串化id数组(不是cid)
		- 响应
			- 200
	
					"TOKEN"
					
### 测试
- Curl

		curl --location --request POST 'https://managed.mypinata.cloud/api/v1/auth/content/jwt' \
		--header 'x-api-key: SUBMARINE KEY' \
		--header 'Content-Type: application/json' \
		--data-raw '{
		    "timeoutSeconds": 3600,
		    "contentIds": ["95c904a0-4d61-4598-a4c8-fb5f0793c7ab"]
		}'
- Node.js

		var axios = require('axios');
		var data = JSON.stringify({
		  "timeoutSeconds": 3600,
		  "contentIds": [
		    "95c904a0-4d61-4598-a4c8-fb5f0793c7ab"
		  ]
		});
		
		var config = {
		  method: 'post',
		  url: 'https://managed.mypinata.cloud/api/v1/auth/content/jwt',
		  headers: { 
		    'x-api-key': 'SUBMARINE KEY', 
		    'Content-Type': 'application/json'
		  },
		  data: data
		};
		
		const res = await axios(config);
		
		console.log(res.data);
- Python

		import requests
		import json
		
		url = "https://managed.mypinata.cloud/api/v1/auth/content/jwt"
		
		payload = json.dumps({
		  "timeoutSeconds": 3600,
		  "contentIds": [
		    "95c904a0-4d61-4598-a4c8-fb5f0793c7ab"
		  ]
		})
		headers = {
		  'x-api-key': 'SUBMARINE KEY',
		  'Content-Type': 'application/json'
		}
		
		response = requests.request("POST", url, headers=headers, data=payload)
		
		print(response.text)
- Go

		package main
		
		import (
		  "fmt"
		  "strings"
		  "net/http"
		  "io/ioutil"
		)
		
		func main() {
		
		  url := "https://managed.mypinata.cloud/api/v1/auth/content/jwt"
		  method := "POST"
		
		  payload := strings.NewReader(`{
		    "timeoutSeconds": 3600,
		    "contentIds": ["95c904a0-4d61-4598-a4c8-fb5f0793c7ab"]
		}`)
		
		  client := &http.Client {
		  }
		  req, err := http.NewRequest(method, url, payload)
		
		  if err != nil {
		    fmt.Println(err)
		    return
		  }
		  req.Header.Add("x-api-key", "SUBMARINE KEY")
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
	
	
	