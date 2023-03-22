# 按 CID 固定
	/pinning/pinByHash
有时，您的本地机器或服务器上可能没有内容，但 IPFS 网络上已经有内容。在那些时候，Pin By CID API 端点很有用。CID（或内容标识符）是由 IPFS 协议生成的哈希值，代表一段内容。通过获取该 CID（或哈希）并使用此 API，您可以将内容导入您自己的 Pinata 帐户并将其固定在 Pinata 的存储节点上。
## 通过 CID 固定文件
固定的每个文件都可以选择包含文件以外的其他信息。`pinataMetadata` 和 `pinataOptions` 都可以包含在请求中。他们的格式.

此端点允许对象中的附加属性 `pinataOptions` 来帮助我们的 IPFS 节点找到您想要固定的内容。

`hostNodes` - 已存储您的内容的节点的多地址。 

您最多可以将“多地址”传入您的内容已驻留的五个主机节点。

要查找您自己节点的多地址，只需在节点的命令行上运行以下命令：

	ipfs id
在响应中，您需要关注返回的“地址”数组。在这里你会找到你的节点的多地址。这些多地址是其他 IPFS 节点用来连接到您的节点的。

在“地址”数组中，记下包含您的外部 IP 地址的多地址。不是本地 ipv4“127.0.0.1”地址或本地 ipv6“::1”地址。

这是一个完整的外部 ipv4 多地址示例（您的 IP 地址和节点 ID 会有所不同）：

	/ip4/123.456.78.90/tcp/4001/ipfs/QmAbCdEfGhIjKlMnOpQrStUvWxYzAbCdEfGhIjKlMnOpQr
一旦你为每个已经固定内容的节点获取了完整的多地址，只需将 “host_nodes” 属性添加到你的 `pinataOptions` 对象中，如下所示：
	
	{
	    hashToPin: (ExampleHash),
	    pinataOptions: {
	        hostNodes: [
	            /ip4/hostNode1ExternalIP/tcp/4001/ipfs/hostNode1PeerId,
	            /ip4/hostNode2ExternalIP/tcp/4001/ipfs/hostNode2PeerId
	            .
	            .
	            .
	        ]
	    }
	}

- `POST https://api.pinata.cloud/pinning/pinByHash`

	通过 CID 固定文件

	- 参数
		- Header 

			- `Authorization*` 		 PINATA-JWT
		- Body
			- `hashToPin`
			- `pinataOptions`	可选的字符串化对象
			- `pinataMetadata` 	可选的字符串化对象
	- 响应
		- 200

				{
				    // Response
				}


## 测试
- Curl

		curl --location --request POST 'https://api.pinata.cloud/pinning/pinByHash' \
		--header 'Authorization: Bearer PINATA JWT' \
		--header 'Content-Type: application/json' \
		--data-raw '{
		    "hashToPin": "CID TO PIN",
		    "pinataMetadata": {
		        "name": "MyCustomName",
		        "keyvalues": {
		            "customKey": "customValue",
		            "customKey2": "customValue2"
		        }
		    }
		}'
- Node.js

		var axios = require('axios');
		var data = JSON.stringify({
		  "hashToPin": "CID TO PIN",
		  "pinataMetadata": {
		    "name": "MyCustomName",
		    "keyvalues": {
		      "customKey": "customValue",
		      "customKey2": "customValue2"
		    }
		  }
		});
		
		var config = {
		  method: 'post',
		  url: 'https://api.pinata.cloud/pinning/pinByHash',
		  headers: { 
		    'Authorization': 'Bearer PINATA JWT', 
		    'Content-Type': 'application/json'
		  },
		  data: data
		};
		
		const res = await axios(config);
		
		console.log(res.data);
- Python

		import requests
		import json
		
		url = "https://api.pinata.cloud/pinning/pinByHash"
		
		payload = json.dumps({
		  "hashToPin": "CID TO PIN",
		  "pinataMetadata": {
		    "name": "MyCustomName",
		    "keyvalues": {
		      "customKey": "customValue",
		      "customKey2": "customValue2"
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
		
		  url := "https://api.pinata.cloud/pinning/pinByHash"
		  method := "POST"
		
		  payload := strings.NewReader(`{
		    "hashToPin": "CID TO PIN",
		    "pinataMetadata": {
		        "name": "MyCustomName",
		        "keyvalues": {
		            "customKey": "customValue",
		            "customKey2": "customValue2"
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
