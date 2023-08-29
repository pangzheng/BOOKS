# Rss.app 官方 API 参考
欢迎使用 RSS.app API 文档！RSS.app API 是一项 [RESTful](http://en.wikipedia.org/wiki/Representational_State_Transfer) Web 服务，专为想要以编程方式与 RSS.app 进行交互的开发人员而设计。此 API 允许您创建、删除和检索 RSS feed

API 使用标准 HTTP 方法（例如 GET、POST、PUT 和 DELETE）来发送请求和接收响应。它需要 [JSON 编码](http://www.json.org/)的请求主体并返回 [JSON 编码](http://www.json.org/) 的响应，并附有适当的 HTTP 状态代码。

在本文档中，您将找到有关可用端点、所需参数和示例代码片段的详细信息，以帮助您开始使用 RSS.app API。

	基本网址
	
	https://api.rss.app
## 验证
RSS.app 使用 API 密钥来验证请求，确保只有授权用户才能访问 API。要查看和管理您的 API 密钥，请访问 [RSS.app 仪表板](https://rss.app/account)。

保持 API 密钥的私密性和安全性至关重要。不要与任何人共享它们，并避免将它们包含在任何可公开访问的资feed中，例如客户端代码或 GitHub 存储库。

请注意，所有 API 请求都必须通过 HTTPS 发出。通过纯 HTTP 进行的任何调用都会失败。此外，未经正确身份验证的 API 请求也将被拒绝。

	您的 API 密钥
	
	本文档中的所有示例都提供了示例测试 API 密钥，使您可以立即测试任何示例。
## 错误
RSS.app API 使用传统的 HTTP 响应代码来指示请求的成功或失败。

`200 范围` 内的代码表示成功。

`400 范围`内的代码表示给定信息有错误。

`500 范围` 内的代码表示 RSS.app 的服务器出现错误（这种情况很少见）。

通过检查响应和包含的错误属性，您可以诊断 API 请求的问题并实施适当的错误处理。

发生错误时，API 返回包含以下属性的 JSON 对象。

- `Attributes` 属性
	- `message` string

		一条人类可读的消息，提供有关错误的更多详细信息。
	- `statusCode` number

		与错误关联的 HTTP 状态代码。
	- `errors` array

		所有错误元素的容器，每个元素包含：
		
		- `title` string

			提供有关特定错误的更多详细信息的消息。
		- `code` string
		
			指示错误代码的短字符串。

## 分页
某些 API 资feed（例如“Feed List”）提供获取多个项目并支持分页的功能。

RSS.app API 利用基于偏移量的分页以及 `$offset` 和 `$limit` 参数对结果进行分页。`$offset` 和 `$limit` 参数都接受整数值并按时间倒序返回对象。

- `Parameters` 参数
	- `$limit` 可选，默认为 10

		单个请求中要获取的最大项目数。值在 1 到 100 之间。
	- `$offset` 可选，默认为 0

		获取项目的起始位置。
- 列出响应格式
	- `data` array

		包含实际响应元素的数组，通过任何请求参数进行分页。
	- `total` number

		元素总数。
	- `offset` number

		确定起点。
	- `limit` number

		对象数量的限制。

### RESPONSE 例子
	{
	    "total": 372,
	    "offset": 1,
	    "limit": 1,
	    "data": [
	        {
	            "id": "cYVBYcpUEbgXfg9v",
	            "title": "BBC - Homepage",
	            "source_url": "http://bbc.com",
	            "rss_feed_url": "https://rss.app/feeds/cYVBYcpUEbgXfg9v.xml",
	            "description": "Breaking news, sport, TV, radio and a whole lot more.
	        The BBC informs, educates and entertains - wherever you are, whatever your age.",
	            "icon": "https://gn-web-assets.api.bbc.com/wwhp/20220322-0833-37491ec2b6e5b4c43bda3673e521e8164a789b87/responsive/img/apple-touch/apple-touch-180.jpg"
	        }
	    ]
	}
## Feeds
Feeds 代表您的 RSS.app 帐户中的新闻feed。使用 RSS.app API，您可以通过执行各种操作（例如创建、检索、删除或列出与您的帐户关联的所有feed）来轻松管理新闻 feed。

每个 feed 都封装了一系列项目，包括标题、图像和各个帖子的简短描述。通过利用 API，您可以有效地组织和管理您的新闻feed，确保您及时了解来自您喜爱的来feed的最新内容。

- feed 对象
	- 属性
		- `id` string
	
			对象的唯一标识符。
		- `title` string
	
			feed 标题。这可以从 RSS.app 仪表板进行编辑。
		- `source_url` string
	
			来feed网址。创建 feed 的 URL。
		- `rss_feed_url` string
	
			RSS feed URL。用于检索 RSS feed的 URL。
		- `description` string
		
			feed描述。这可以从 RSS.app 仪表板进行编辑。
		- `icon` string
		
			feed 图标的 URL。
		- `items` array
		
			表示附加到 feed 对象的文章的列表。
		
			- `url` string
			
				项目描述的资feed的 URL。
			- `title`string
			
				标题
			- `description_text` string
			
				纯文本格式的描述
			- `description_html` string
			
				HTML 格式的描述。如果没有描述，该字段将为空。
			- `thumbnail` string
			
				缩略图网址
			- `author` sarray
	
				作为对象的作者数组。如果没有作者，将返回一个空数组。
			- `authors` object
	
				对象代表一个作者
			
				- `name` string
		
					作者姓名
			- `date_published` date
	
				发表日期
- ENDPOINTS

		POST /v1/feeds
		GET /v1/feeds/:id
		GET /v1/feeds
		DELETE /v1/feeds/:id
- feed 对象例子

		{
		    "id": "cYVBYcpUEbgXfg9v",
		    "title": "BBC - Homepage",
		    "rss_feed_url": "https://rss.app/feeds/cYVBYcpUEbgXfg9v.xml",
		    "source_url": "https://bbc.com",
		    "description": "Breaking news, sport, TV, radio and a whole lot more. The BBC informs, educates and entertains - wherever you are, whatever your age.",
		    "items": [
		        {
		            "url": "https://www.bbc.com/news/av/world-us-canada-65118507",
		            "title": "Why this iconic spider sculpture faces removal",
		            "description_text": "The Vancouver sculpture is made from recycled waste but is tangled up in a web of bureaucracy.",
		            "description_html": "<div><div style="left: 0; width: 100%; height: 0; position: relative; padding-bottom: 56.25%;"><iframe src="https://www.bbc.com/news/av-embeds/65118507" style="border: 0; top: 0; left: 0; width: 100%; height: 100%; position: absolute;" allowfullscreen scrolling="no" allow="encrypted-media"></iframe></div></div>",
		            "thumbnail": "https://ichef.bbci.co.uk/news/1024/branded_news/8C30/production/_129188853_p0fcpll9.jpg",
		            "date_published": "2023-04-08T01:09:36.000Z",
		            "authors": []
		        },
		        {
		            "url": "https://www.bbc.com/travel/article/20230407-the-real-way-to-whip-cream",
		            "title": "The 'real' way to whip cream",
		            "description_text": "Aptly named after its place of origin, this sweet, thick whipped cream is arguably the best of its kind the "crème de la crème" so to speak.",
		            "thumbnail": "https://ychef.files.bbci.co.uk/live/624x351/p0fcxv79.jpg",
		            "date_published": "2023-04-08T12:50:28.000Z",
		            "authors": [{
		                "name": "Angela Dansby"
		            }]
		        },
		        {
		            "url": "https://www.bbc.com/news/world-africa-65221385",
		            "title": "Thabo Bester: South African murderer who faked death arrested in Tanzania",
		            "description_text": "Thabo Bester, known as the "Facebook rapist", escaped from South African prison in May last year.",
		            "thumbnail": "https://ichef.bbci.co.uk/news/1024/branded_news/1806B/production/_129311489_gettythumbnails-129143970.jpg",
		            "date_published": "2023-04-08T15:47:06.000Z",
		            "authors": []
		        }
		    ]
		}

### 创建 feed
使用以下参数从网站创建feed：

- 参数
	- `url` 必须填写
	
		需要有效的网站 URL（例如：https://bbc.com）
- 返回

	返回包含帖子的 feed。否则，将返回错误。

- `Post /v1/feed` 请求例子

	请求支持多种格式，包括 curl、nodejs、 php、java、go、python ，以下请求例子以 go 为说明，其他例子查看页面 [rssapp.api docs](https://rss.app/docs/api/feeds/create)
	
		package main
		import (
		  "fmt"
		  "strings"
		  "net/http"
		  "io/ioutil"
		)
		
		func main() {
		
		  url := "api.https://rss.app/v1/feeds"
		  method := "POST"
		
		  payload := strings.NewReader(`{
		    "url": "https://bbc.com"
		}`)
		
		  client := &http.Client {
		  }
		  req, err := http.NewRequest(method, url, payload)
		
		  if err != nil {
		    fmt.Println(err)
		    return
		  }
		  req.Header.Add("Authorization", "Bearer c_ciwogLkIa8rJQ9:s_i12EVmZ4D5d3ZZ7Q8OLQ21")
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
- 响应

		{
		    "id": "cYVBYcpUEbgXfg9v",
		    "title": "BBC - Homepage",
		    "rss_feed_url": "https://rss.app/feeds/cYVBYcpUEbgXfg9v.xml",
		    "source_url": "https://bbc.com",
		    "description": "Breaking news, sport, TV, radio and a whole lot more. The BBC informs, educates and entertains - wherever you are, whatever your age.",
		    "items": [
		        {
		            "url": "https://www.bbc.com/news/av/world-us-canada-65118507",
		            "title": "Why this iconic spider sculpture faces removal",
		            "description_text": "The Vancouver sculpture is made from recycled waste but is tangled up in a web of bureaucracy.",
		            "description_html": "<div><div style="left: 0; width: 100%; height: 0; position: relative; padding-bottom: 56.25%;"><iframe src="https://www.bbc.com/news/av-embeds/65118507" style="border: 0; top: 0; left: 0; width: 100%; height: 100%; position: absolute;" allowfullscreen scrolling="no" allow="encrypted-media"></iframe></div></div>",
		            "thumbnail": "https://ichef.bbci.co.uk/news/1024/branded_news/8C30/production/_129188853_p0fcpll9.jpg",
		            "date_published": "2023-04-08T01:09:36.000Z",
		            "authors": []
		        },
		        {
		            "url": "https://www.bbc.com/travel/article/20230407-the-real-way-to-whip-cream",
		            "title": "The 'real' way to whip cream",
		            "description_text": "Aptly named after its place of origin, this sweet, thick whipped cream is arguably the best of its kind the "crème de la crème" so to speak.",
		            "thumbnail": "https://ychef.files.bbci.co.uk/live/624x351/p0fcxv79.jpg",
		            "date_published": "2023-04-08T12:50:28.000Z",
		            "authors": [{
		                "name": "Angela Dansby"
		            }]
		        },
		        {
		            "url": "https://www.bbc.com/news/world-africa-65221385",
		            "title": "Thabo Bester: South African murderer who faked death arrested in Tanzania",
		            "description_text": "Thabo Bester, known as the "Facebook rapist", escaped from South African prison in May last year.",
		            "thumbnail": "https://ichef.bbci.co.uk/news/1024/branded_news/1806B/production/_129311489_gettythumbnails-129143970.jpg",
		            "date_published": "2023-04-08T15:47:06.000Z",
		            "authors": []
		        }
		    ]
		}

### 检索 feed
通过提供现有 feed 的唯一 feed ID 来检索其详细信息。

feed ID 可以从“创建 feed”请求或“feed 列表”请求中找到。

- 参数

	无参数
- 返回

	返回 feed。否则，将返回错误。


- `GET /v1/feed/:id` 请求例子

	请求支持多种格式，包括 curl、nodejs、 php、java、go、python ，以下请求例子以 go 为说明，其他例子查看页面 [rssapp.api docs](https://rss.app/docs/api/feeds/create)
	
		package main
	    
	      import (
	        "fmt"
	        "net/http"
	        "io/ioutil"
	      )
	      
	      func main() {
	      
	        url := "api.https://rss.app/v1/feeds/:id"
	        method := "GET"
	      
	        client := &http.Client {
	        }
	        req, err := http.NewRequest(method, url, nil)
	      
	        if err != nil {
	          fmt.Println(err)
	          return
	        }
	        req.Header.Add("Authorization", "Bearer YOUR_API_KEY:YOUR_API_SECRET")
	      
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
- 响应

		{
		    "id": "cYVBYcpUEbgXfg9v",
		    "title": "BBC - Homepage",
		    "rss_feed_url": "https://rss.app/feeds/cYVBYcpUEbgXfg9v.xml",
		    "source_url": "https://bbc.com",
		    "description": "Breaking news, sport, TV, radio and a whole lot more. The BBC informs, educates and entertains - wherever you are, whatever your age.",
		    "items": [
		        {
		            "url": "https://www.bbc.com/news/av/world-us-canada-65118507",
		            "title": "Why this iconic spider sculpture faces removal",
		            "description_text": "The Vancouver sculpture is made from recycled waste but is tangled up in a web of bureaucracy.",
		            "description_html": "<div><div style="left: 0; width: 100%; height: 0; position: relative; padding-bottom: 56.25%;"><iframe src="https://www.bbc.com/news/av-embeds/65118507" style="border: 0; top: 0; left: 0; width: 100%; height: 100%; position: absolute;" allowfullscreen scrolling="no" allow="encrypted-media"></iframe></div></div>",
		            "thumbnail": "https://ichef.bbci.co.uk/news/1024/branded_news/8C30/production/_129188853_p0fcpll9.jpg",
		            "date_published": "2023-04-08T01:09:36.000Z",
		            "authors": []
		        },
		        {
		            "url": "https://www.bbc.com/travel/article/20230407-the-real-way-to-whip-cream",
		            "title": "The 'real' way to whip cream",
		            "description_text": "Aptly named after its place of origin, this sweet, thick whipped cream is arguably the best of its kind the "crème de la crème" so to speak.",
		            "thumbnail": "https://ychef.files.bbci.co.uk/live/624x351/p0fcxv79.jpg",
		            "date_published": "2023-04-08T12:50:28.000Z",
		            "authors": [{
		                "name": "Angela Dansby"
		            }]
		        },
		        {
		            "url": "https://www.bbc.com/news/world-africa-65221385",
		            "title": "Thabo Bester: South African murderer who faked death arrested in Tanzania",
		            "description_text": "Thabo Bester, known as the "Facebook rapist", escaped from South African prison in May last year.",
		            "thumbnail": "https://ichef.bbci.co.uk/news/1024/branded_news/1806B/production/_129311489_gettythumbnails-129143970.jpg",
		            "date_published": "2023-04-08T15:47:06.000Z",
		            "authors": []
		        }
		    ]
		}

### 列出所有 feed
查看帐户中创建的 feed 列表。

- 参数
	- `$limit` 可选，默认为 10

		返回的 Feed 数量限制在 1 到 100 之间
	- `$offset` 可选，默认为 0
		
		确定起点
- 响应

	返回帐户中的 feed 列表。

	数组中的每个条目都是一个单独的 Feed 对象。如果没有更多可用的feed，则生成的数组为空。
- `GET /v1/feed/` 请求例子

	请求支持多种格式，包括 curl、nodejs、 php、java、go、python ，以下请求例子以 go 为说明，其他例子查看页面 [rssapp.api docs](https://rss.app/docs/api/feeds/create)

		package main
		import (
		  "fmt"
		  "net/http"
		  "io/ioutil"
		)
		
		func main() {
		
		  url := "api.https://rss.app/v1/feeds?limit=10"
		  method := "GET"
		
		  client := &http.Client {
		  }
		  req, err := http.NewRequest(method, url, nil)
		
		  if err != nil {
		    fmt.Println(err)
		    return
		  }
		  req.Header.Add("Authorization", "Bearer YOUR_API_KEY:YOUR_API_SECRET")
		
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
- 响应

		{
		    "total": 373,
		    "offset": 0,
		    "limit": 10,
		    "data": [
		        {
		            "id": "cYVBYcpUEbgXfg9v",
		            "title": "BBC - Homepage",
		            "source_url": "http://bbc.com",
		            "rss_feed_url": "https://rss.app/feeds/cYVBYcpUEbgXfg9v.xml",
		            "description": "Breaking news, sport, TV, radio and a whole lot more.
		        The BBC informs, educates and entertains - wherever you are, whatever your age.",
		            "url": "https://gn-web-assets.api.bbc.com/wwhp/20220322-0833-37491ec2b6e5b4c43bda3673e521e8164a789b87/responsive/img/apple-touch/apple-touch-180.jpg"
		        },
		        {
		            "id": "tRB1VRwysSuwnHlJ",
		            "title": "Bitcoin",
		            "source_url": "https://rss.app/rss-feed?keyword=vanadzor"
		            "rss_feed_url": "https://rss.app/feeds/tRB1VRwysSuwnHlJ.xml",
		            "description": "#Bitcoin generated by RSS.app"
		        },
		        {
		            "id": "tq7X9v2dKgkTre59",
		            "source_url": "https://rss.app/rss-feed?topicId=technology"
		            "rss_feed_url": "https://rss.app/feeds/tq7X9v2dKgkTre59.xml",
		            "title": "Technology",
		            "description": "#Technology generated by RSS.app"
		        },
		        {...}
		    ]
		}

### 删除 feed
您可以通过 RSS.app 仪表板的 feed 管理页面删除 feed。删除的 feed 将从捆绑中删除。还可以通过 API 删除 Feed。

- Parameters 参数

	无参数
- 返回

	返回 feed ID 和删除标志。如果 Feed 已被删除，则会返回一条错误消息，指出“Feed 已被删除”。
- `DELETE /v1/feed/:id` 请求例子

	请求支持多种格式，包括 curl、nodejs、 php、java、go、python ，以下请求例子以 go 为说明，其他例子查看页面 [rssapp.api docs](https://rss.app/docs/api/feeds/create)
	
		package main
		import (
		  "fmt"
		  "net/http"
		  "io/ioutil"
		)
		
		func main() {
		
		  url := "api.https://rss.app/v1/feeds/%7B%7BfeedId%7D%7D"
		  method := "DELETE"
		
		  client := &http.Client {
		  }
		  req, err := http.NewRequest(method, url, nil)
		
		  if err != nil {
		    fmt.Println(err)
		    return
		  }
		  req.Header.Add("Authorization", "Bearer YOUR_API_KEY:YOUR_API_SECRET")
		
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
- 响应

		{
		    "id": "zScdlc0QIdfuBNA6",
		    "deleted": true
		}	
	
## 参考
https://rss.app/docs/api