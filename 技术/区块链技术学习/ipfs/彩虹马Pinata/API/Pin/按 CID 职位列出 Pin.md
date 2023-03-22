# 按 CID 列出 Pin
	/pinning/pinJobs
使用 `pinByHash` 端点时，您可能希望以编程方式检查您请求固定到您帐户的 CID 的状态。该端点允许您这样做。
## 按 CID 列出 Pin
所有可能的过滤器都包含在下面的 API 参考中，但这些是可能的“状态”过滤器：

- “status” - 按固定队列中作业的状态过滤（请参阅下面的潜在状态）
	- “prechecking” - Pinata 正在对您的 pin 请求进行初步验证。
	- “searching” — Pinata 正在 IPFS 网络上积极搜索您的内容。如果您的内容是孤立的，这可能需要一些时间。
	- “retrieving” - Pinata 已找到您的内容，现在正在检索它。
	- “expired” — Pinata 在搜索 IPFS 网络一天后无法找到您的内容。在尝试再次固定之前，请确保您的内容托管在 IPFS 网络上。
	- “over\_free\_limit” - 固定此对象会使您超出免费层级限制。请添加信用卡以继续固定内容。
	- “over\_max\_size”- 此对象太大而无法固定。如果您看到这种情况，请联系我们以获得更个性化的解决方案。
	- “invalid\_object” - IPFS 节点无法读取您尝试固定的对象。如果您收到此邮件，请联系我们，因为我们希望更好地了解您尝试固定的内容。
	- “bad\_host\_node”- 您提供的主机节点无效或无法访问。请确保所有提供的主机节点都在线且可访问。

- `Get https://api.pinata.cloud/pinning/pinJobs` 
	- 参数
		- Query
			- `sort`				取值包括:ASC或DESC
			- `status`			请参阅上面的可用值
			- `ipfs_pin_hash`		要固定的文件的CID
			- `limit`				返回的作业数。默认值为5，最大值为1000
			- `offset` 			为返回的记录提供记录偏移量。这就是如何检索其他页面上的记录(默认值为0)
		- Header 

			- `Authorization*` 		 PINATA-JWT
	- 响应
		- 200

				{
				    count: (这是您传入的查询过滤器所存在的引脚作业记录的总数),
				    rows: [
				        {
				            id: (pin 作业记录的id),
				            ipfs_pin_hash: (你固定的内容的IPFS多重哈希),
				            date_queued: (此散列最初排队以钉住的日期—以ISO 8601格式表示),
				            status: (pin 作业的当前状态),
				            name: (如果您为您的散列传递了一个名称，它将在这里列出),
				            keyvalues: (如果您为您的散列传递了键值，它们将在这里列出),
				            host_nodes: (如果您为您的散列提供了主机节点，它们将在这里列出),
				            pin_policy: 一旦找到了这个内容，这就是用于复制的 pin 策略
				        },
				        {
				           与上面相同的记录格式
				        }
				        .
				        .
				        .
				    ]
				}

- 测试
	- Curl

			curl --location --request GET 'https://api.pinata.cloud/pinning/pinJobs?status=retrieving&sort=ASC' \
			--header 'Authorization: Bearer PINATA JWT' 
	- Node.js

			var axios = require('axios');
			
			var config = {
			  method: 'get',
			  url: 'https://api.pinata.cloud/pinning/pinJobs?status=retrieving&sort=ASC',
			  headers: { 
			    'Authorization': 'Bearer PINATA JWT'
			  }
			};
			
			const res = await axios(config);
			
			console.log(res.data);
	- Python

			var axios = require('axios');
			
			var config = {
			  method: 'get',
			  url: 'https://api.pinata.cloud/pinning/pinJobs?status=retrieving&sort=ASC',
			  headers: { 
			    'Authorization': 'Bearer PINATA JWT'
			  }
			};
			
			const res = await axios(config);
			
			console.log(res.data);
	- GO

			package main
			
			import (
			  "fmt"
			  "net/http"
			  "io/ioutil"
			)
			
			func main() {
			
			  url := "https://api.pinata.cloud/pinning/pinJobs?status=retrieving&sort=ASC"
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
