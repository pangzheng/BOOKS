# 固定 JSON
	/pinning/pinJSONToIPFS
此端点允许发件人将他们希望的任何 JSON 对象添加并固定到 Pinata 的 IPFS 节点。此端点经过专门优化，仅处理 JSON 内容。服务器收到 JSON 后，会将其转换为 JSON 文件并固定到我们的 IPFS 存储节点。
## 上传和固定 JSON
每次上传都可以选择包含文件以外的其他信息。`pinataMetadata ` 和 `pinataOptions` 都可以包含在请求中。他们的格式.

- `POST https://api.pinata.cloud/pinning/pinJSONToIPFS`
	- 参数
		- Header 

			- `Authorization*` 		 PINATA-JWT
		- Body
			- `pinataContent*` 	要固定的 JSON 内容
			- `pinataOptions`	可选的 Pinata 选项对象
			- `pinataMetadata`	可选的 Pinata 元数据对象
	- 响应
		- 200

				{
				    IpfsHash: 这是为您的内容提供的IPFS多散列,
				    PinSize: 这是您刚刚固定的内容的大小(以字节为单位),
				    Timestamp: 这是用于内容固定的时间戳(以ISO 8601格式表示)。
				}

### 测试
- Curl

		curl --location --request POST 'https://api.pinata.cloud/pinning/pinJSONToIPFS' \
		--header 'Content-Type: application/json' \
		--header 'Authorization: Bearer PINATA JWT' \
		--data-raw '{
		    "pinataOptions": {
		        "cidVersion": 1
		    },
		    "pinataMetadata": {
		        "name": "testing",
		        "keyvalues": {
		            "customKey": "customValue",
		            "customKey2": "customValue2"
		        }
		    },
		    "pinataContent": {
		        "somekey":"somevalue"
		    }
		}'
- Node.js

		var axios = require('axios');
		var data = JSON.stringify({
		  "pinataOptions": {
		    "cidVersion": 1
		  },
		  "pinataMetadata": {
		    "name": "testing",
		    "keyvalues": {
		      "customKey": "customValue",
		      "customKey2": "customValue2"
		    }
		  },
		  "pinataContent": {
		    "somekey": "somevalue"
		  }
		});
		
		var config = {
		  method: 'post',
		  url: 'https://api.pinata.cloud/pinning/pinJSONToIPFS',
		  headers: { 
		    'Content-Type': 'application/json', 
		    'Authorization': 'Bearer PINATA JWT'
		  },
		  data : data
		};
		
		const res = await axios(config);
		
		console.log(res.data);
- Python

		import requests
		import json
		
		url = "https://api.pinata.cloud/pinning/pinJSONToIPFS"
		
		payload = json.dumps({
		  "pinataOptions": {
		    "cidVersion": 1
		  },
		  "pinataMetadata": {
		    "name": "testing",
		    "keyvalues": {
		      "customKey": "customValue",
		      "customKey2": "customValue2"
		    }
		  },
		  "pinataContent": {
		    "somekey": "somevalue"
		  }
		})
		headers = {pyth
		  'Content-Type': 'application/json',
		  'Authorization': 'Bearer PINATA JWT'
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
		
		  url := "https://api.pinata.cloud/pinning/pinJSONToIPFS"
		  method := "POST"
		
		  payload := strings.NewReader(`{
		    "pinataOptions": {
		        "cidVersion": 1
		    },
		    "pinataMetadata": {
		        "name": "testing",
		        "keyvalues": {
		            "customKey": "customValue",
		            "customKey2": "customValue2"
		        }
		    },
		    "pinataContent": {
		        "somekey":"somevalue"
		    }
		}`)
		
		  client := &http.Client {
		  }
		  req, err := http.NewRequest(method, url, payload)
		
		  if err != nil {
		    fmt.Println(err)
		    return
		  }
		 
		  req.Header.Add("Content-Type", "application/json")
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
