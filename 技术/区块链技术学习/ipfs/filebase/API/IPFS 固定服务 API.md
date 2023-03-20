# IPFS 固定服务 API
了解如何以编程方式将 IPFS 固定服务 API 与 Filebase 结合使用。
## 什么是 IPFS Pinning 服务 API？
IPFS Pinning Service API 是用于重新固定现有 IPFS CID 的标准化 API。它通常被称为“远程”固定或 IPFS“PSA”。IPFS PSA 由 IPFS CLI 和 IPFS 桌面应用程序使用。

IPFS 开发团队跟踪支持 IPFS PSA 的每个提供商的合规性，以验证每个实施。Filebase 完全符合 IPFS PSA 要求。[此处](https://ipfs-shipyard.github.io/pinning-service-compliance/api.filebase.io.html)查看官方合规报告。

[Filebase 通过我们的 IPFS 重新固定服务支持 IPFS 固定服务 API](https://docs.filebase.com/ipfs/ipfs-pinning#re-pinning-existing-ipfs-cids-from-the-filebase-web-console) 。此 API 可用于查看固定对象、添加要固定的新对象或删除固定对象。

	IPFS Pinning Service API 的有效速率限制为每个 IP 地址 100 RPS。
要与 Filebase IPFS Pinning Service API 交互，请使用以下信息。

### API端点
	https://api.filebase.io/v1/ipfs/pins
### 验证
要使用 IPFS 固定服务 API 进行身份验证，需要使用以下格式在 HTTP 标头中随每个请求一起发送 Token：

	Authorization: Bearer <access-token>
可以通过导航到 `access-token` [访问密钥页面](https://console.filebase.com/keys)，然后查看 IPFS 固定服务 API 端点来生成。单击“选择存储桶以生成 Token”的下拉菜单，然后选择您要使用的 IPFS 存储桶。

![](./pic/pinserver.webp)
然后复制生成的 Secret Access Token：

![](./pic/pinserver1.webp)

### 有效载荷
请求正文架构： application/json

要与 IPFS Pinning Service API 交互，您需要将 JSON 形式的有效负载发送到 API URL。以下是 POST 负载的示例：

	{
	  "cid": "bafybeihbfkn5afsjep2jd65gn5vedh7vpvp6i6avfu5za7fnnhsvfutzjy",
	  "name": "image3",
	  "meta": {
	    "key_name": "value_name"
	  }
	}

	以下 API 方法中列出的所有参数都是负载中的 JSON 字段。
- 列出固定对象 ` GET https://api.filebase.io/v1/ipfs/pins`

	列出所有固定对象，支持可选的过滤器，如果没有提供过滤器，则返回所有成功的 pin
	
	- Query 参数

		参数名|类型|说名
		---|---|---
		cid|Array|返回匹配制定过滤对象，游览器上下文每个 URL 的字符限制为2000个，例子 `cid=Qm1 , bafy2 , Qm3`
		name|String|返回具有特定名称的固定对象，默认匹配策略是区分大小写，完全匹配，例子`name=ExampleObject.pdf`
		match|String|自定义应用于名称过滤器的文本匹配策略。默认区分大小写，完全匹配。其他选项部分匹配、不区分大小写的完全匹配(iexact)和不区分大小写的部分(ipartial) 匹配,选项 `"exact" "iexact" "partial" "ipartial"` ,例子 `match=iexact`
		status|Array|返回匹配制定状态的固定对象。选项 `"queued" "pinning" "pinned" "failed"` ,例子`status=queued,failed`
		before|String|返回在提供的时间戳之前排队的固定对象。例子:`before=2022-02-09T12:12:02Z`
		after|String|返回在提供的时间戳之后排队的固定对象。例子:`after=2022-02-09T12:12:02Z`
		limit|Integer|返回的最大结果量|默认10，例子 `limit=20`
		meta|Object|返回于使用 JSON 对象的字符串表示形式传递的制定元数据值匹配的固定对象。如果使用客户端库，请对参数进行编码以确保安全传输。例子 `meta={"object_id": "99999"}`
	- 响应
		- 200
	
				{
				  "count": 1,
				  "results": [
				    {
				      "requestid": "UniqueIdOfPinRequest",
				      "status": "queued",
				      "created": "2022-02-09T12:12:02Z",
				      "pin": {
				        "cid": "QmWTqpfKyPJcGuWWg73beJJiL6FrCB5yX8qfcCF4bHvane",
				        "name": "nft-collection",
				        "origins": [
				          "/ip4/123.12.113.142/tcp/4001/p2p/SourcePeerId",
				          "/ip4/123.12.113.114/udp/4001/quic/p2p/SourcePeerId"
				        ],
				        "meta": {
				          "object_id": "99999"
				        }
				      },
				      "delegates": [
				        "/ip4/123.12.113.142/tcp/4001/p2p/QmServicePeerId"
				      ],
				      "info": {
				        "status_details": "Queue position: 1 of 1"
				      }
				    }
				  ]
				}
		- 其他状态

				https://docs.filebase.com/api-documentation/ipfs-pinning-service-api
- 固定对象 `POST https://api.filebase.io/v1/ipfs/pins`

	添加一个固定在 IPFS 上的新对象
	
	- Body 参数

		参数名|类型|说名
		---|---|---
		cid*|string|要固定的 CID
		name|string|与固定 CID 相关联的可选人类可读名称，小于或等于 255 个字符
		origins|Array|已知 CID 数据的多个地址列表
		meta|Object|与固定 CID 关联的可选元数据
	- 响应
		- 202

				 {
				      "requestid": "UniqueIdOfPinRequest",
				      "status": "queued",
				      "created": "2022-02-09T12:12:02Z",
				      "pin": {
				        "cid": "QmWTqpfKyPJcGuWWg73beJJiL6FrCB5yX8qfcCF4bHvane",
				        "name": "nft-collection",
				        "origins": [
				          "/ip4/123.12.113.142/tcp/4001/p2p/SourcePeerId",
				          "/ip4/123.12.113.114/udp/4001/quic/p2p/SourcePeerId"
				        ],
				        "meta": {
				          "object_id": "99999"
				        }
				      },
				      "delegates": [
				        "/ip4/123.12.113.142/tcp/4001/p2p/QmServicePeerId"
				      ],
				      "info": {
				        "status_details": "Queue position: 1 of 1"
				  }
		- 其他状态

				https://docs.filebase.com/api-documentation/ipfs-pinning-service-api
- 获取固定对象 `Get https://api.filebase.io/v1/ipfs/pins/{requestid}`

	获取固定对象的信息和状态
	
	- Path 参数

		参数名|类型|说名
		---|---|---
		requestid*|string
	- 响应
		- 200

				 {
				      "requestid": "UniqueIdOfPinRequest",
				      "status": "queued",
				      "created": "2022-02-09T12:12:02Z",
				      "pin": {
				        "cid": "QmWTqpfKyPJcGuWWg73beJJiL6FrCB5yX8qfcCF4bHvane",
				        "name": "nft-collection",
				        "origins": [
				          "/ip4/123.12.113.142/tcp/4001/p2p/SourcePeerId",
				          "/ip4/123.12.113.114/udp/4001/quic/p2p/SourcePeerId"
				        ],
				        "meta": {
				          "object_id": "99999"
				        }
				      },
				      "delegates": [
				        "/ip4/123.12.113.142/tcp/4001/p2p/QmServicePeerId"
				      ],
				      "info": {
				        "status_details": "Queue position: 1 of 1"
				  }
		- 其他状态

				https://docs.filebase.com/api-documentation/ipfs-pinning-service-api
- 替换固定对象 `POST https://api.filebase.io/v1/ipfs/pins/{requestid}`

	替换现有的固定对象，这个可以用作分别为递归引脚运行删除、添加操作的快捷方式
	
	- Path 参数

		参数名|类型|说名
		---|---|---
		requestid*|string
	- Body 参数

		参数名|类型|说名
		---|---|---
		cid*|string|要固定的 CID
		name|string|与固定 CID 相关联的可选人类可读名称，小于或等于 255 个字符
		origins|Array|已知 CID 数据的多个地址列表
		meta|Object|与固定 CID 关联的可选元数据
	- 响应
		- 202

				 {
				      "requestid": "UniqueIdOfPinRequest",
				      "status": "queued",
				      "created": "2022-02-09T12:12:02Z",
				      "pin": {
				        "cid": "QmWTqpfKyPJcGuWWg73beJJiL6FrCB5yX8qfcCF4bHvane",
				        "name": "nft-collection",
				        "origins": [
				          "/ip4/123.12.113.142/tcp/4001/p2p/SourcePeerId",
				          "/ip4/123.12.113.114/udp/4001/quic/p2p/SourcePeerId"
				        ],
				        "meta": {
				          "object_id": "99999"
				        }
				      },
				      "delegates": [
				        "/ip4/123.12.113.142/tcp/4001/p2p/QmServicePeerId"
				      ],
				      "info": {
				        "status_details": "Queue position: 1 of 1"
				  }
		- 其他状态

				https://docs.filebase.com/api-documentation/ipfs-pinning-service-api
- 删除固定对象 `DELETE https://api.filebase.io/v1/ipfs/pins/{requestid}`

	删除固定对象
	
	- Path 参数

		参数名|类型|说名
		---|---|---
		requestid*|string
	- 响应
		- 202
	- 其他状态

				https://docs.filebase.com/api-documentation/ipfs-pinning-service-api
		