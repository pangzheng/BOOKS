# Submarine JSON
	/content/json
此端点允许您添加和潜入任何您想要的 JSON 对象。此端点经过专门优化，仅处理 JSON 内容。服务器收到 JSON 后，会将其转换为 JSON 文件并进行 Submarined。
## Submarine JSON
每次上传都可以选择包含文件以外的其他信息。可以包含 JSON 对象形式的元数据。元数据必须采用键值对的形式。目前无法嵌套键值对。

- `GET https://managed.mypinata.cloud/api/v1/content/json`
	- 参数
		- Header 
			- `x-api-key*`      SUBMARINE KEY
		- Body
			- `name`			JSON 文件的名称
			- `metadata`     可选的元数据对象
			- `content*`		要固定的 JSON 内容
			- `pinToIPFS`	 	False to Submarine
			- `cidVersion`	0 或 1
			- `name`			True 将 JSON 文件放入文件夹中
		- 响应
			- 200

## 测试
- curl'

		curl --location --request POST 'https://managed.mypinata.cloud/api/v1/content/json' \
		--header 'x-api-key: Submarine Key' \
		--header 'Content-Type: application/json' \
		--data-raw '{
		    "content": {
		        "example": "content", 
		        "more": "examples"
		    }, 
		    "name": "My JSON",
		    "metadata": {
		        "keyvalues": {
		            "one": "two"
		        }
		    }
		}'
- Node.js

		var axios = require('axios');
		var data = JSON.stringify({
		  "content": {
		    "example": "content",
		    "more": "examples"
		  },
		  "name": "My JSON",
		  "metadata": {
		    "keyvalues": {
		      "one": "two"
		    }
		  }
		});
		
		var config = {
		  method: 'post',
		  url: 'https://managed.mypinata.cloud/api/v1/content/json',
		  headers: { 
		    'x-api-key': 'Submarine Key', 
		    'Content-Type': 'application/json'
		  },
		  data : data
		};
		
		const res = await axios(config);
		
		console.log(res.data);
- python

		import requests
		import json
		
		url = "https://managed.mypinata.cloud/api/v1/content/json"
		
		payload = json.dumps({
		  "content": {
		    "example": "content",
		    "more": "examples"
		  },
		  "name": "My JSON",
		  "metadata": {
		    "keyvalues": {
		      "one": "two"
		    }
		  }
		})
		headers = {
		  'x-api-key': 'Submarine Key',
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
		
		  url := "https://managed.mypinata.cloud/api/v1/content/json"
		  method := "POST"
		
		  payload := strings.NewReader(`{
		    "content": {
		        "example": "content", 
		        "more": "examples"
		    }, 
		    "name": "My JSON",
		    "metadata": {
		        "keyvalues": {
		            "one": "two"
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
		  req.Header.Add("x-api-key", "Submarine Key")
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