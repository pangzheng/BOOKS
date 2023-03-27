# 1 ipfs-pinning-service-yaml
	openapi: 3.0.3
	info:
	  version: "1.0.0"
	  title: 'IPFS Pinning Service API'
	  x-logo:
	    url: "https://bafybeidehxarrk54mkgyl5yxbgjzqilp6tkaz2or36jhq24n3rdtuven54.ipfs.dweb.link/?filename=ipfs-pinning-service.svg"
	  contact:
	    name: IPFS
	    url: https://github.com/ipfs/pinning-services-api-spec
	  description: "
  
  
## 1.1 关于本规范
IPFS Pinning Service API 旨在成为一个与实现无关的 API:

- 供固定服务提供商使用和实施
- 供 IPFS 节点和基于 GUI 的应用程序在客户端模式下使用

### 1.1.1  文档范围和目标受众
本文档的目标读者是 **IPFS 开发人员** 构建与此 OpenAPI 规范兼容的固定服务客户端或服务器。
在我们开发此 API 规范时，欢迎您提出宝贵的意见和反馈。请在 [github.com/ipfs/pinning-services-api-spec](https://github.com/ipfs/pinning-services-api-spec)加入设计讨论.

**IPFS 用户** 应该在 [docs.ipfs.io/how-to/work-with-pinning-services](https://docs.ipfs.io/how-to/work-with-pinning-services/) 代替.
### 1.1.2 相关资源
本规范的最新版本和其他资源可在以下位置找到

- 规范: https://github.com/ipfs/pinning-services-api-spec/raw/main/ipfs-pinning-service.yaml
- 文档: https://ipfs.github.io/pinning-services-api-spec/
- 客户和服务: https://github.com/ipfs/pinning-services-api-spec#adoption

# 2 模式
本节描述最重要的对象类型和约定。

字段和 `模式` 的完整列表可以在 [YAML file](https://github.com/ipfs/pinning-services-api-spec/blob/master/ipfs-pinning-service.yaml).
## 2.1 身份标识
### 2.1.1 cid
[Content Identifier (CID)](https://docs.ipfs.io/concepts/content-addressing/) 指向递归固定的 DAG 的根
### 2.1.2 requestid
pin 请求的唯一标识符。

创建 pin 后，服务会以唯一的 `requestid` 进行响应，稍后可用于删除 pin。当再次固定相同的 `cid` 时，将返回不同的 `requestid` 以区分这些固定请求。

服务实现应使用 UUID, `hash(accessToken,Pin,PinStatus.created)`, 或任何其他提供同样强大的竞争条件保护的不透明标识符.
## 2.2 对象
### 2.2.1 Pin 对象
![pin 对象](https://bafybeideck2fchyxna4wqwc2mo67yriokehw3yujboc5redjdaajrk2fjq.ipfs.dweb.link/pin.png)

`Pin`对象是 pin 请求的表示

它包括要 pin 的数据的 `cid`、`name`、`origins`和 `meta`中的可选元数据。

`origins` 列表中提供的地址仅在初始固定期间相关，不需要由固定服务保留。
### 2.2.2 Pin 状态响应
![pin 状态响应对象](https://bafybeideck2fchyxna4wqwc2mo67yriokehw3yujboc5redjdaajrk2fjq.ipfs.dweb.link/pinstatus.png)

`PinStatus` 对象表示 pin 操作的当前状态.

它包括来自原始 `Pin` 对象的值以及整个 pin 请求的当前 `status` 和全局唯一的 `requestid`，可用于将来的状态检查和管理。
`delegates` 数组中的地址是由 pinning 服务指定的对等点，它将通过 bitswap 接收 pin 数据（更多细节见 [Provider hints](#section/Provider-hints) 部分). 任何其他特定于供应商的信息都在可选的 `信息` 中返回 

# 3 pin 生命周期
![pinning service objects and lifecycle](https://bafybeideck2fchyxna4wqwc2mo67yriokehw3yujboc5redjdaajrk2fjq.ipfs.dweb.link/lifecycle.png)
## 3.1 创建一个新的 pin 对象
用户将 `Pin` 对象发送到 `POST /pins` 并收到 `PinStatus` 响应：

- `PinStatus` 中的 `requestid` 是 pin 操作的标识符，可用于检查状态并在将来移除 pin
- `PinStatus` 中的 `status` 指示 pin 的当前状态

## 3.2 检查正在进行的 pin 状态
`status`（在 `PinStatus` 中）可能表示两种挂起状态之一：`queued` 或 `pinning`：

- `queued` 是被动的

	pin 已添加到队列中，但服务尚未消耗任何资源来检索它。
- `pinning` 处于活动状态

	pinning 服务正在尝试通过查找所有相关 CID 的提供者来检索 CID，连接到这些提供者并从中下载数据。

当创建一个新的 pin 对象时，它通常以 `queued` 状态开始。一旦固定服务主动寻求检索文件，它就会更改为 `pinning`。`pinning` 通常意味着在 pinning 服务上找不到 `Pin.cid` 背后的数据，并且正在从整个 IPFS 网络中获取数据，这可能需要一些时间。

在任何一种情况下，用户都可以通过 `GET /pins/{requestid}` 定期检查固定进度，直到固定成功或者用户决定删除待处理的固定。

## 3.3 替换现有的 pin 对象
用户可以通过 `POST /pins/{requestid}` 替换现有的 pin 对象。这是删除由 `requestid` 标识的 pin 对象并在单个 API 调用中创建新对象的快捷方式以防止对两个 pin 共有的块进行不需要的垃圾收集。在更新表示大多数块未更改的巨大数据集的 pin 时很有用。新的 pin 对象 `requestid` 在 `PinStatus` 响应中返回。旧的 pin 对象被自动删除。
## 3.4 删除 pin 对象
可以通过 `DELETE /pins/{requestid}`删除 pin 对象

# 4 提供者提示
提供者提示采用两个 [multiaddr](https://docs.ipfs.io/concepts/glossary/#multiaddr) 列表的形式：

-  `Pin.origins`

	已知数据来源（提供者）的列表。由客户端在 pin 请求中发送。

	Pinning 服务将尝试连接到它们以加速数据传输。
-  `PinStatus.delegates`

	数据的临时目的地（检索器）列表。通过 pin 服务在 pin 请求的响应中返回。

	这些对等点由 pin 服务提供，目的是获取要 pin 的数据。

## 4.1 优化速度和连接性
两端应尝试相互预连接：

- 代表应始终预先连接到 `origins `
- 发起 pin 请求并且在自己的本地数据存储中也有 Pin 数据的客户应该预先连接到 `delegates`

**注意:** 连接到 ` origins` 和 `delegate` 数组中的 multiaddrs 应尽量尝试，拨号失败不应导致 pin 操作失败。当无法对显式的提供者提示进行操作时，DHT 和其他发现方法应该被 pin 服务用作备用方法。


## 4.2 理由
pin 服务将使用 DHT 和其他发现方法来定位 pin 内容; 但是，如果唯一的提供者，它可能无法检索到数据没有可公开拨打的地址（例如，在限制性 NAT/防火墙后面的桌面对等点）。

利用提供商提示可以减轻潜在的连接问题并加快内容路由阶段。

如果客户在自己的数据存储中拥有数据或已经知道其他提供商，则传输将立即开始。

最常见的场景是客户端将自己的 IPFS 节点的多地址放入 `Pin.origins`，然后尝试连接到返回的每个多地址在 `PinStatus.delegates` 中 pin 服务以启动传输。同时，pin 服务将尝试连接在 `Pin.origins` 中到客户端提供的多地址。

这确保数据传输立即开始（无需等待提供者通过 DHT 发现），以及客户端和服务之间的相互直接拨号解决限制性网络拓扑中的对等路由问题，例如 NAT、防火墙等

**注意：** 所有多地址必须以 `/p2p/{peerID}` 结尾并且应该完整已解决并确认可从公共互联网拨号。应避免发送来自本地网络的地址。
# 5 自定义元数据
鼓励 pin 服务通过利用可选的 `Pin.meta` 和 `PinStatus.info` 字段来添加对其他功能的支持。

虽然这些属性可以是特定于应用程序或特定于供应商的，但我们鼓励整个社区将这些属性用作沙箱，以提出可能成为此 API 未来修订的一部分的约定。
## 5.1 Pin 元数据
在 `Pin.meta` 中传递的字符串键和值与 pin 对象一起保存。

这是一个可选功能：客户端可以忽略或忽略这些可选属性，这样做不会影响基本固定功

潜在用途：

- `Pin.meta[app_id]`：将唯一标识符附加到应用程序创建的 pin ，启用每个应用程序的元过滤品
- `Pin.meta[vendor_policy]`：特定于供应商的策略（例如：使用哪个区域，保留多少副本）

### 5.1.1 基于元数据过滤
`Pin.meta` 的内容可以用作高级搜索过滤器，用于通过 `name` 和 `cid` 搜索不够的情况。

元数据键匹配规则为 `AND`：

- 查找返回具有 `meta` 的引脚，所有键值对与传递的值匹配
- pin 元数据可能有更多键，但只有在查询中传递的键才会用于过滤

用作查询参数时，`meta` 的有线格式是 [URL 转义](https://en.wikipedia.org/wiki/Percent-encoding) 字符串化 JSON 对象。

具有 `meta` 键值对 `{\"app_id\":\"UUID\"}` 的 pin 的查找示例是：

- `GET /pins?meta=%7B%22app_id%22%3A%22UUID%22%7D`

## 5.2 Pin 状态信息
pin 服务可以返回额外的 `PinStatus.info`。

潜在用途：

- `PinStatus.info[status_details]`：关于当前状态的更多信息（队列位置、传输数据的百分比、数据存储位置的摘要等）；

	什么时候 `PinStatus.status=failed`，它可以提供一个 pin 操作的原因失败（例如缺乏资金、DAG 太大等）
- `PinStatus.info[dag_size]`：pin 数据的大小以及 DAG 开销
- `PinStatus.info[raw_size]`：没有 DAG 开销的数据大小（例如 unixfs）
- `PinStatus.info[pinned_until]`：如果供应商支持时间限制的 pin，这可能表明 pin 何时到期

# 分页和过滤
可以通过使用可选参数执行 `GET /pins` 来列出 Pin 对象

- 当没有提供过滤器时，端点将返回一小批 10 个最近创建的项目，从最新到最旧。
- 可以使用 `limit` 参数调整返回项目的数量（隐式默认值为 10）。
- 如果 `PinResults.count` 中的值大于 `PinResults.results`，客户端可以推断还有更多的结果可以查询。
- 要阅读更多项目，请传递带有时间戳的 `before` 过滤器 `PinStatus.created` 在当前批次结果中最旧的项目中找到。重复阅读所有结果。
- 可以通过应用可选的 `after`、`cid`、`name`、`status` 或 `meta` 过滤器来微调返回的结果。

> **注意**：按 `created` 时间戳分页要求每个值都是独一无二。任何未来考虑添加对批量创建的支持必须考虑到这一点。


	"
	servers:
	  - url: https://pinning-service.example.com
	
	tags:
	  - name: pins
	
	paths:
	  /pins:  //接口 A
	    get:
	      operationId: getPins
	      summary: 列出 pin 对象
	      description: 列出所有 pin 对象，匹配可选过滤器;当没有提供过滤器时，只返回成功的 pin
	      tags:
	        - pins
	      parameters:
	        - $ref: '#/components/parameters/cid'
	        - $ref: '#/components/parameters/name'
	        - $ref: '#/components/parameters/match'
	        - $ref: '#/components/parameters/status'
	        - $ref: '#/components/parameters/before'
	        - $ref: '#/components/parameters/after'
	        - $ref: '#/components/parameters/limit'
	        - $ref: '#/components/parameters/meta'
	      responses:
	        '200':
	          description: 成功响应(PinResults对象)
	          content:
	            application/json:
	              schema:
	                $ref: '#/components/schemas/PinResults'
	        '400':
	          $ref: '#/components/responses/BadRequest'
	        '401':
	          $ref: '#/components/responses/Unauthorized'
	        '404':
	          $ref: '#/components/responses/NotFound'
	        '409':
	          $ref: '#/components/responses/InsufficientFunds'
	        '4XX':
	          $ref: '#/components/responses/CustomServiceError'
	        '5XX':
	          $ref: '#/components/responses/InternalServerError'
	          
	    post:
	      operationId: addPin
	      summary: 添加 pin 对象
	      description: 为当前访问 token 添加一个新的 pin 对象
	      tags:
	        - pins
	      requestBody:
	        required: true
	        content:
	          application/json:
	            schema:
	              $ref: '#/components/schemas/Pin'
	      responses:
	        '202':
	          description:  成功响应(PinResults对象)
	          content:
	            application/json:
	              schema:
	                $ref: '#/components/schemas/PinStatus'
	        '400':
	          $ref: '#/components/responses/BadRequest'
	        '401':
	          $ref: '#/components/responses/Unauthorized'
	        '404':
	          $ref: '#/components/responses/NotFound'
	        '409':
	          $ref: '#/components/responses/InsufficientFunds'
	        '4XX':
	          $ref: '#/components/responses/CustomServiceError'
	        '5XX':
	          $ref: '#/components/responses/InternalServerError'
	
	  /pins/{requestid}: //接口 B
	    parameters:
	      - name: requestid
	        in: path
	        required: true
	        schema:
	          type: string
	    get:
	      operationId: getPinByRequestId
	      summary: 获取 pin 对象
	      description: 获取一个 pin 对象及其状态
	      tags:
	        - pins
	      responses:
	        '200':
	          description: 成功响应(PinStatus对象)
	          content:
	            application/json:
	              schema:
	                $ref: '#/components/schemas/PinStatus'
	        '400':
	          $ref: '#/components/responses/BadRequest'
	        '401':
	          $ref: '#/components/responses/Unauthorized'
	        '404':
	          $ref: '#/components/responses/NotFound'
	        '409':
	          $ref: '#/components/responses/InsufficientFunds'
	        '4XX':
	          $ref: '#/components/responses/CustomServiceError'
	        '5XX':
	          $ref: '#/components/responses/InternalServerError'
	    post:
	      operationId: replacePinByRequestId
	      summary: 替换 pin 对象
	      description: 替换现有 pin 对象(在一步中执行删除和添加操作的快捷方式，以避免对两个递归 pin 中存在的块进行不必要的垃圾收集)
	      tags:
	        - pins
	      requestBody:
	        required: true
	        content:
	          application/json:
	            schema:
	              $ref: '#/components/schemas/Pin'
	      responses:
	        '202':
	          description: 成功响应(PinStatus对象)
	          content:
	            application/json:
	              schema:
	                $ref: '#/components/schemas/PinStatus'
	        '400':
	          $ref: '#/components/responses/BadRequest'
	        '401':
	          $ref: '#/components/responses/Unauthorized'
	        '404':
	          $ref: '#/components/responses/NotFound'
	        '409':
	          $ref: '#/components/responses/InsufficientFunds'
	        '4XX':
	          $ref: '#/components/responses/CustomServiceError'
	        '5XX':
	          $ref: '#/components/responses/InternalServerError'
	    delete:
	      operationId: deletePinByRequestId
	      summary: 删除 pin 对象
	      description: 移除一个 pin 对象
	      tags:
	        - pins
	      responses:
	        '202':
	          description: 响应成功(no body, ）
	        '400':
	          $ref: '#/components/responses/BadRequest'
	        '401':
	          $ref: '#/components/responses/Unauthorized'
	        '404':
	          $ref: '#/components/responses/NotFound'
	        '409':
	          $ref: '#/components/responses/InsufficientFunds'
	        '4XX':
	          $ref: '#/components/responses/CustomServiceError'
	        '5XX':
	          $ref: '#/components/responses/InternalServerError'
	
	components:
	  schemas:
	
	    PinResults:
	      description: 用于列出 pin 对象匹配请求的响应
	      type: object
	      required:
	        - count
	        - results
	      properties:
	        count:
	          description: 为传递的查询过滤器存在的 pin 对象的总数
	          type: integer
	          format: int32
	          minimum: 0
	          example: 1
	        results:
	          description:PinStatus 结果数组 
	          type: array
	          items:
	            $ref: '#/components/schemas/PinStatus'
	          uniqueItems: true
	          minItems: 0
	          maxItems: 1000
	
	    PinStatus:
	      description: 带状态的 pin 对象
	      type: object
	      required:
	        - requestid
	        - status
	        - created
	        - pin
	        - delegates
	      properties:
	        requestid:
	          description: pin 请求的全局唯一标识符;可以用来检查状态正在进行 pin 或 删除	          type: string
	          example: "UniqueIdOfPinRequest"
	        status:
	          $ref: '#/components/schemas/Status'
	        created:
	          description: 不可变的时间戳，指示 pin 请求何时进入 pin 服务;是否可以用于过滤结果和分页
	          type: string
	          format: date-time  # RFC 3339, section 5.6
	          example: "2020-07-27T17:32:28.276Z"
	        pin:
	          $ref: '#/components/schemas/Pin'
	        delegates:
	          $ref: '#/components/schemas/Delegates'
	        info:
	          $ref: '#/components/schemas/StatusInfo'
	
	    Pin:
	      description: Pin 对象
	      type: object
	      required:
	        - cid
	      properties:
	        cid:
	          description: Content Identifier (CID) 递归 pinned
	          type: string
	          example: "QmCIDToBePinned"
	        name:
	          description: pin 数据的可选名称;可以用于以后的查找
	          type: string
	          maxLength: 255
	          example: "PreciousData.pdf"
	        origins:
	          $ref: '#/components/schemas/Origins'
	        meta:
	          $ref: '#/components/schemas/PinMeta'
	
	    Status:
	      description: 一个 pin 对象在一个 pin 服务上可以拥有的状态
	      type: string
	      enum:
	        - queued     # pin 操作在队列中等待;附加信息可以在 info 中返回[status_details]      
	        - pinning    # 正在 pinning 中;附加信息可以在 info 中返回 [status_details]
	        - pinned     # pinned 成功
	        - failed     # pin 服务无法完成 pin 操作;更多信息可以在 info 中找到 [status_details]
	
	    Delegates:
	      description: 由接收 pin 数据的 pinning 服务指定的多地址列表;请参阅文档中的提供者提示
	      type: array
	      items:
	        type: string
	      uniqueItems: true
	      minItems: 1
	      maxItems: 20
	      example: ['/ip4/203.0.113.1/tcp/4001/p2p/QmServicePeerId']
	
	    Origins:
	      description: 已知提供数据的 multiaddr 的可选列表;请参阅文档中的提供者提示
	      type: array
	      items:
	        type: string
	      uniqueItems: true
	      minItems: 0
	      maxItems: 20
	      example: ['/ip4/203.0.113.142/tcp/4001/p2p/QmSourcePeerId', '/ip4/203.0.113.114/udp/4001/quic/p2p/QmSourcePeerId']
	
	    PinMeta:
	      description: pin 对象的可选元数据
	      type: object
	      additionalProperties:
	        type: string
	        minProperties: 0
	        maxProperties: 1000
	      example:
	        app_id: "99986338-1113-4706-8302-4420da6158aa" # Pin.meta[app_id], 有用的过滤针每个应用程序
	
	    StatusInfo:
	      description: PinStatus 响应可选信息
	      type: object
	      additionalProperties:
	        type: string
	        minProperties: 0
	        maxProperties: 1000
	      example:
	        status_details: "Queue position: 7 of 9" # PinStatus.info[status_details], 当 status=queued
	
	    TextMatchingStrategy:
	      description: 替代文本匹配策略
	      type: string
	      default: exact
	      enum:
	        - exact    # 完全匹配，区分大小写(隐式默认)
	        - iexact   # 完全匹配，不区分大小写
	        - partial  # 部分匹配，区分大小写
	        - ipartial # 部分匹配，不区分大小写
	
	    Failure:
	      description: 对失败请求的响应
	      type: object
	      required:
	        - error
	      properties:
	        error:
	          type: object
	          required:
	            - reason
	          properties:
	            reason:
	              type: string
	              description: 返回 string 标识错误类型
	              example: "ERROR_CODE_FOR_MACHINES"
	            details:
	              type: string
	              description: 可选，更长的错误描述;可以包括 UUID 的交易支持，链接到文件等
	              example: "Optional explanation for humans with more details"
	
	  parameters:
	
	    before:
	      description: 返回在提供时间戳之前创建的结果(排队)
	      name: before
	      in: query
	      required: false
	      schema:
	        type: string
	        format: date-time  # RFC 3339, section 5.6
	      example: "2020-07-27T17:32:28.276Z"
	
	    after:
	      description: 返回在提供时间戳后创建的结果(排队)
	      name: after
	      in: query
	      required: false
	      schema:
	        type: string
	        format: date-time  # RFC 3339, section 5.6
	      example: "2020-07-27T17:32:28.276Z"
	
	    limit:
	      description: 返回最大记录
	      name: limit
	      in: query
	      required: false
	      schema:
	        type: integer
	        format: int32
	        minimum: 1
	        maximum: 1000
	        default: 10
	
	    cid:
	      description: 返回负责 pin 指定 CID 的 pin 对象;请注意，使用更长的哈希函数会进一步限制浏览器上下文中每个 URL 不能超过 2000 个字符的 cid 数量
	      name: cid
	      in: query
	      required: false
	      schema:
	        type: array
	        items:
	          type: string
	        uniqueItems: true
	        minItems: 1
	        maxItems: 10
	      style: form # ?cid=Qm1,Qm2,bafy3
	      explode: false
	      example: ["Qm1","Qm2","bafy3"]
	
	    name:
	      description: 返回指定名称的 pin 对象 (默认情况下是区分大小写的精确匹配)
	      name: name
	      in: query
	      required: false
	      schema:
	        type: string
	        maxLength: 255
	      example: "PreciousData.pdf"
	
	    match:
	      description: 自定义当名称过滤器存在时应用的文本匹配策略;Exact(默认值)是区分大小写的精确匹配，partial 匹配名称中的任何位置，iexact 和 ipartial 是 Exact 和 partial 策略的不区分大小写版本
	      name: match
	      in: query
	      required: false
	      schema:
	        $ref: '#/components/schemas/TextMatchingStrategy'
	      example: exact
	
	    status:
	      description: 为指定状态的 pin 返回 pin 对象(当缺少 pin 时，服务默认仅为 pin )
	      name: status
	      in: query
	      required: false
	      schema:
	        type: array
	        items:
	          $ref: '#/components/schemas/Status'
	        uniqueItems: true
	        minItems: 1
	      style: form # ?status=queued,pinning
	      explode: false
	      example: ["queued","pinning"]
	
	    meta:
	      description: 返回与作为 JSON 对象的字符串表示传递的指定元数据键匹配的 pin 对象;在实现客户端库时，确保参数是 url 编码的，以确保安全传输.
	      name: meta
	      in: query
	      required: false
	      content:
	        application/json: # ?meta={"foo":"bar"}
	          schema:
	            $ref: '#/components/schemas/PinMeta'
	
	  responses:
	  
	    BadRequest:
	      description: 错误响应(错误请求)
	      content:
	        application/json:
	          schema:
	            $ref: '#/components/schemas/Failure'
	          examples:
	            BadRequestExample:
	              $ref: '#/components/examples/BadRequestExample'
	
	    Unauthorized:
	      description: 错误响应(未经授权;访问令牌缺失或无效)
	      content:
	        application/json:
	          schema:
	            $ref: '#/components/schemas/Failure'
	          examples:
	            UnauthorizedExample:
	              $ref: '#/components/examples/UnauthorizedExample'
	
	    NotFound:
	      description: 错误响应(未找到指定的资源)
	      content:
	        application/json:
	          schema:
	            $ref: '#/components/schemas/Failure'
	          examples:
	            NotFoundExample:
	              $ref: '#/components/examples/NotFoundExample'
	
	    InsufficientFunds:
	      description: 错误响应(资金不足)
	      content:
	        application/json:
	          schema:
	            $ref: '#/components/schemas/Failure'
	          examples:
	            InsufficientFundsExample:
	              $ref: '#/components/examples/InsufficientFundsExample'
	
	    CustomServiceError:
	      description: 错误响应(自定义服务错误)
	      content:
	        application/json:
	          schema:
	            $ref: '#/components/schemas/Failure'
	          examples:
	            CustomServiceErrorExample:
	              $ref: '#/components/examples/CustomServiceErrorExample'
	
	    InternalServerError:
	      description: 错误响应(意外的内部服务器错误)
	      content:
	        application/json:
	          schema:
	            $ref: '#/components/schemas/Failure'
	          examples:
	            InternalServerErrorExample:
	              $ref: '#/components/examples/InternalServerErrorExample'
	
	  examples:
	
	    BadRequestExample:
	      value:
	        error:
	          reason: "BAD_REQUEST"
	          details: "为人类提供更多细节的解释"
	      summary: 对错误请求的示例响应;原因会有所不同
	
	    UnauthorizedExample:
	      value:
	        error:
	          reason: "UNAUTHORIZED"
	          details: "访问 token 缺失或无效"
	      summary: 对未授权请求的响应
	
	    NotFoundExample:
	      value:
	        error:
	          reason: "NOT_FOUND"
	          details: "没有找到指定的资源"
	      summary: 响应对不存在的资源的请求
	
	    InsufficientFundsExample:
	      value:
	        error:
	          reason: "INSUFFICIENT_FUNDS"
	          details: "由于缺乏资金，无法处理请求"
	      summary: 访问 token 耗尽资金时响应
	
	    CustomServiceErrorExample:
	      value:
	        error:
	          reason: "CUSTOM_ERROR_CODE_FOR_MACHINES"
	          details: "可选的解释人类更多的细节"
	      summary: 自定义错误发生时的响应
	
	    InternalServerErrorExample:
	      value:
	        error:
	          reason: "INTERNAL_SERVER_ERROR"
	          details: "为人类提供更多细节的解释"
	      summary: 发生意外错误时的响应
	
	  securitySchemes:
	    accessToken:
	      description: " HTTP头中的每个请求都需要发送一个不透明令牌:
	
				- `Authorization: Bearer <access-token>`
			每个设备都应该生成“访问 Token”，用户应该能够分别撤销每个Token。"
	      
	      type: http
	      scheme: bearer
	security:
	  - accessToken: []