# 查询 Submarined 内容
	/content
此端点返回有关您通过 Pinata Submarined 的内容的数据。

可以使用下面将讨论的多个查询参数来过滤此调用的结果。
## 查询海底文件
- `GET https://managed.mypinata.cloud/api/v1/content`
	- 参数
		- Header 
			- `x-api-key*`      SUBMARINE KEY
		- Query
			- `cidContains`
			
				您正在搜索的文件的 CID
			- `createdAtStart`

				ISO_8601 格式日期，按开始日期过滤文件被 Submarined 的时间
			- `createdAtEnd`

				ISO_8601 格式日期按结束日期过滤文件被 Submarined 的时间
			- `originalname`

				来自文件系统的原始文件名
			- `pinSizeMin`

				要返回的文件的最小大小（以字节为单位）
			- `pinSizeMax`

				要返回的文件的最大大小（以字节为单位）
			- `status`
			
				选择填写  "all", "pinned", or "unpinned"
			- `limit`

				要返回的最大文件数。默认为 10，最大为 1000
			- `offset`

				用于对文件进行分页，默认为 0
			- `metadata`

				请参阅下面的元数据查询部分
			- `order`

				用于对结果进行排序的属性（上述任何查询参数）
	- 响应
		- 200

				{
				  "status": 200,
				
				  "totalItems": 0,
				  "items": [
				    {
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
				  ]
				}

## 元数据查询
您还可以选择在通过 PinFileToIPFS 或 PinJSONToIPFS 端点上传内容时，在相关元数据上查询您的内容，这些元数据可能包含在内容中。

这些查询看起来与默认参数非常相似，但稍微复杂一些。

这里有几个简单的例子，后面有补充说明。

- 要查询您为 pin 提供的名称，您的查询将采用以下形式：

		?metadata[name]=exampleName
		
	（这将匹配包含作为值提供的字符串的名称。为了增加特异性，请包括您要查找的全名）。
- 查询元数据键值属性：

		?metadata[keyvalues]={"exampleKey":{"value":"exampleValue","op":"exampleOp"}}
	或者：

		?metadata[keyvalues][exampleKey]={"value":"exampleValue","op":"exampleOp"}
- 查询元数据名称和多个 key-value 属性：

		?metadata[name]=exampleName&metadata[keyvalues]={"exampleKey":{"value":"exampleValue","op":"exampleOp"},"exampleKey2":{"value":"exampleValue2","op":"exampleOp2"}}

	- 解释“value”和“op”键/值

		如上所示，每个对自定义值的查询都采用带有“value”键和“op”键的对象形式。

		“value”相当简单。这只是您希望应用查询操作的值。这些值可以是：
		
		- 字符串
		- 数字（整数或小数）
		- 日期（以 ISO_8601 格式提供）

		“op”是将应用于您提供的值的查询操作。以下操作码可用于查询：
		
		- “gt”-（大于）
		- “gte”-（大于或等于）
		- “lt”-（小于）
		- “lte”-（小于或等于）
		- “ne”-（不等于）
		- "eq" - （等于）
		- “between”——（使用“between”操作进行查询时，您需要提供一个“secondValue”以供查询操作使用）

		例如：

			?metadata[keyvalues]={"exampleKey":{"value":"2018-01-01 00:00:00.000+00","secondValue":"2018-02-01 00:00:00.000+00","op":"between"}}

		- “notBetween”-（使用“notBetween”操作进行查询时，您需要提供一个“secondValue”以供查询操作使用）
		
		例如：

			?metadata[keyvalues]={"exampleKey":{"value":4.00,"secondValue":5.50,"op":"notBetween"}}
			
		- “like”-（您可以使用它来查找与您传入的值相似的值）

		例如，此查询将查找以“testValue”开头的所有条目。在您的值之前的A%表示任何内容都可以在它之前，而一个%符号在它之后意味着任何字符都可以在它之后。所以%testValue%将包含所有包含字符“testValue”的条目。

			?metadata[keyvalues]={"exampleKey":{"value":"testValue%","op":"like"}}
			
		- “notLike”——（您可以使用它来查找不包含您传入的字符串的值）
		- “iLike”——（“喜欢”操作码的不区分大小写的版本）
		- “notILike”——（“notLike”操作码的不区分大小写的版本）
		- "regexp" - （正则表达式匹配）
		- "iRegexp" - （不区分大小写的正则表达式匹配）

## 测试
- curl

		curl --location --request GET 'https://managed.mypinata.cloud/api/v1/content?createdAtStart=2022-01-01T06:00:00.000Z&pinSizeMin=100' \
		--header 'x-api-key: SUBMARINE KEY'
- Node.js

		var axios = require('axios');
		
		var config = {
		  method: 'get',
		  url: 'https://managed.mypinata.cloud/api/v1/content?createdAtStart=2022-01-01T06:00:00.000Z&pinSizeMin=100',
		  headers: { 
		    'x-api-key': 'SUBMARINE KEY'
		  }
		};
		
		const res = await axios(config);
		
		console.log(res.data);
- python

		import requests
		
		url = "https://managed.mypinata.cloud/api/v1/content?createdAtStart=2022-01-01T06:00:00.000Z&pinSizeMin=100"
		
		payload={}
		headers = {
		  'x-api-key': 'SUBMARINE KEY'
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
		
		  url := "https://managed.mypinata.cloud/api/v1/content?createdAtStart=2022-01-01T06:00:00.000Z&pinSizeMin=100"
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