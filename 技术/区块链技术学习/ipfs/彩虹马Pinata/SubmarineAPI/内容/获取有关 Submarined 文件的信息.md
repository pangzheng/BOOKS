# 获取有关 Submarined 文件的信息
	/content/:id
此端点允许您获取有关 Submarined 的单个文件的信息。它不返回实际文件。这是由您的[网关](https://docs.pinata.cloud/gateways)处理的。
## 获取有关Submarined文件的信息
- `Get https://managed.mypinata.cloud/api/v1/content/:id`
	- 参数
		- Header 
			- `x-api-key*`      SUBMARINE KEY
		- Path
			- `id` 		 		Submarined 内容的ID(不是CID)
	- 响应
		- 200

				{
				  "status": 200,
				
				  "totalItems": 0,
				  "item": {
				      "id": "string",
				      "createdAt": "2022-05-20T19:33:36.340Z",
				      "cid": "string",
				      "name": "string",
				      "mimeType": "string",
				      "originalname": "string",
				      "size": 0,
				      "metadata": {},
				      "pinToIPFS": false,
				      "isDuplicate": false
				  }
				}

## 测试
- curl

		curl --location --request GET 'https://managed.mypinata.cloud/api/v1/content/95c904a0-4d61-4598-a4c8-fb5f0793c7ab' \
		--header 'x-api-key: SUBMARINE KEY'
- Node.js

		var axios = require('axios');
		
		var config = {
		  method: 'get',
		  url: 'https://managed.mypinata.cloud/api/v1/content/95c904a0-4d61-4598-a4c8-fb5f0793c7ab',
		  headers: { 
		    'x-api-key': 'SUBMARINE KEY'
		  }
		};
		
		const res = await axios(config);
		
		console.log(res.data);
- Python

		import requests
		
		url = "https://managed.mypinata.cloud/api/v1/content/95c904a0-4d61-4598-a4c8-fb5f0793c7ab"
		
		payload={}
		headers = {
		  'x-api-key': 'SUBMARINE KEY'
		}
		
		response = requests.request("GET", url, headers=headers, data=payload)
		
		print(response.text)
- GO

		package main
		
		import (
		  "fmt"
		  "net/http"
		  "io/ioutil"
		)
		
		func main() {
		
		  url := "https://managed.mypinata.cloud/api/v1/content/95c904a0-4d61-4598-a4c8-fb5f0793c7ab"
		  method := "GET"
		
		  client := &http.Client {
		  }
		  req, err := http.NewRequest(method, url, nil)
		
		  if err != nil {
		    fmt.Println(err)
		    return
		  }
		  req.Header.Add("x-api-key", "SUBMARINE KEY")
		
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
