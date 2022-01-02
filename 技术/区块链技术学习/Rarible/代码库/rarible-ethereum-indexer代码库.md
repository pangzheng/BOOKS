# rarible-ethereum-indexer代码库
## readme.md
Rarible Protocol 以太坊索引器由以下四个子索引器组成：

- [NFT 索引器](https://github.com/rarible/ethereum-indexer/blob/master/nft):聚合 NFT 数据
- [ERC-20 索引器](https://github.com/rarible/ethereum-indexer/blob/master/erc20):聚合有关 ERC-20 代币和余额的数据
- [order 索引器](https://github.com/rarible/ethereum-indexer/blob/master/order):聚合来自不同平台的订单数据
- [NFT-order](https://github.com/rarible/ethereum-indexer/blob/master/nft-order) 将 nft 和 order 索引器连接在一起

### 架构
每个子索引器都会监听以太坊区块链的`特定部分`。索引器可用于请求有关区块链状态的数据。索引器工作是在区块链状态更改时发出事件。它们是用 Spring Framework 开发的，并使用这些外部服务：

- MongoDB:主要数据存储
- Apache Kafka:事件处理

### OpenAPI
索引器使用 OpenAPI 来描述 API（和事件）。客户端（kotlin、typescript 等）和服务器控制器接口是使用 yaml OpenAPI 文件自动生成的。

在 [Rarible Etehreum 协议 OpenAPI](https://github.com/rarible/ethereum-openapi) 中查看更多信息。

OpenAPI 文档：[https://ethereum-api.rarible.org/v0.1/doc](https://ethereum-api.rarible.org/v0.1/doc)

## 代码目录结构
- `block-scanner` 块扫描器？
- `gateway` api 网关服务？
- erc20 索引器目录

	侦听基于 EVM 的区块链并重建 ERC-20 代币余额的状态。
	
	- [API](https://github.com/rarible/ethereum-indexer/blob/master/erc20/api)

		这个模块有控制器

		- 查询有关 ERC-20 余额的数据
	- [监听](https://github.com/rarible/ethereum-indexer/blob/master/erc20/listener)
		
		这个模块侦听事件并相应地更新数据。它基于[ethereum-core 日志侦听器](https://github.com/rarible/ethereum-core/tree/master/listener-log)
	- 数据模型
		- [Erc20TokenHistory](https://github.com/rarible/ethereum-indexer/blob/master/erc20/core/src/main/kotlin/com/rarible/protocol/erc20/core/model/Erc20TokenHistory.kt) 

			从区块链监听的事件
			
			- Erc20IncomeTransfer(入交易)  字段包括
				- owner // 所有者  //可覆盖
				- vlaue //进入金额 //可覆盖
				- token // 合约地址 //可覆盖
				- date //交易时间 //可覆盖
			- Erc20OutcomeTransfer(出交易) 字段包括
				- owner // 所有者  //可覆盖
				- vlaue //进入金额 //可覆盖
				- token // 合约地址  //可覆盖
				- date //交易时间 //可覆盖
			- Erc20Deposit(定金)  字段包括
				- owner // 所有者  //可覆盖
				- vlaue //进入金额 //可覆盖
				- token // 合约地址  //可覆盖
				- date //交易时间 //可覆盖
			- Erc20Withdrawal (退出) 字段包括
				- owner // 所有者  //可覆盖
				- vlaue //进入金额 //可覆盖
				- token // 合约地址  //可覆盖
				- date //交易时间 //可覆盖
			- Erc20TokenApproval(授权) 字段包括
				- owner // 所有者  //可覆盖
				- spender // 授权者
				- vlaue //进入金额 //可覆盖
				- token // 合约地址  //可覆盖
				- date //交易时间 //可覆盖
			   
		- [Erc20Balance](https://github.com/rarible/ethereum-indexer/blob/master/erc20/core/src/main/kotlin/com/rarible/protocol/erc20/core/model/Erc20Balance.kt) 

			特定用户/代币的余额
		
			- Erc20Balance(余额)  字段包括
				- token // 合约地址
				- owner // 所有者
				- balance //余额
				- version // 版本？  
- nft 索引器目录 
	- 简介
	
		侦听基于 EVM 的区块链并为区块链中找到的所有 NFT 重建状态。支持的 NFT 类型：
		
		- ERC-721
		- ERC-1155
		- 加密朋克（进行中）
	- [API](https://github.com/rarible/ethereum-indexer/blob/master/nft/api)

		这个模块有控制器
		
		- 查询有关 NFT 的数据（谁拥有什么，有多少 ERC-1155 地址）
		- 查询有关事件的数据（转移、造币厂、燃烧）
		- 创建懒惰的 NFT
		- 发布有关待处理交易的信息
	- [监听](https://github.com/rarible/ethereum-indexer/blob/master/nft/listener)

		该模块侦听事件并相应地更新数据。它基于[ethereum-core 日志侦听器](https://github.com/rarible/ethereum-core/tree/master/listener-log)
	- 数据模型
		- [Item](https://github.com/rarible/ethereum-indexer/blob/master/nft/core/src/main/kotlin/com/rarible/protocol/nft/core/model/Item.kt)

			代表一个 NFT。项目只能由一个地址拥有 (ERC-721) 或许多所有者可以拥有它 (ERC-1155)。
			
			- item 字段包括
				- 通用字段 
					- token // 合约地址
					- tokenid // tokenid
					- creator //创建者,字段类型 list，默认是空
					- creatorsFinal // ? ,字段类型:布尔，默认false
					- supply // 数量
					- lazySupply // 惰性数量
					- royalties // 税率
				- 所有者字段
					- owners // 所有者,类型 list
					- date //创建时间？
				- 状态字段
					- pending // 待处理列表
					- deleted // 销毁, 字段类型:布尔，默认false
					- lastLazyEventTimestamp //最后惰性事件时间戳
					- revertableEvents //字段类型 list //可覆盖
					- ownerships //  所有者,字段类型  map  
		- [Ownership](https://github.com/rarible/ethereum-indexer/blob/master/nft/core/src/main/kotlin/com/rarible/protocol/nft/core/model/Ownership.kt) 

			代表地址拥有的 NFT 的项目（所有者和项目之间的关系）
			
			- Ownership 表字段包括
				- 通用字段 
					- token // 合约地址
					- tokenid // tokenid
					- creator //创建者,字段类型 list，默认是空
					- owner // 所有者
					- value // ？
					- lazyValue //?
					- date //?字段类型:时间格式
				- 状态字段
					- pending // 待处理列表
					- deleted // 销毁, 字段类型:布尔，默认false
					- lastLazyEventTimestamp //最后惰性事件时间戳
					- revertableEvents //字段类型 list   //可覆盖
		- [ItemHistory](https://github.com/rarible/ethereum-indexer/blob/master/nft/core/src/main/kotlin/com/rarible/protocol/nft/core/model/ItemHistory.kt) 

			所有关于物品和所有权的事件（转让、铸造、销毁等）。侦听器从区块链中侦听这些事件，保存它们并重新创建项目和所有权的状态。
			
			- ItemTransfer  表字段包括
				- owner // 所有者  //可覆盖
				- token // 合约地址 //可覆盖
				- tokenid // tokenid //可覆盖
				- date // 字段类型:时间格式 //可覆盖
				- from //从哪里来
				- value //NFT 详情？
			- ItemRoyalty  表字段包括
				- token // 合约地址 //可覆盖
				- tokenid // tokenid //可覆盖
				- date // 字段类型:时间格式 //可覆盖
				- royalties // 税率 ，字段类型 list
				- owner // 所有者 //可覆盖
			- ItemCreators
				- token // 合约地址 //可覆盖
				- tokenid // tokenid //可覆盖
				- date // 字段类型:时间格式 //可覆盖
				- creator //创建者,字段类型 list
				- owner // 所有者 //可覆盖
			- ItemLazyMint
				- token // 合约地址 //可覆盖
				- tokenid // tokenid //可覆盖
				- value // NFT 详情？
				- date // 字段类型:时间格式 //可覆盖
				- uri // 元数据地址字段
				- standard // 标准
				- creator //创建者,字段类型 list
				- royalties // 税率 ，字段类型 list？
				- signatures // 签名，字段类型 llist 二进制
				- owner // 所有者 ,creators.first().account //可覆盖
			- BurnItemLazyMint
				- from // 从哪里来的
				- token // 合约地址  //可覆盖
				- tokenid // tokenid //可覆盖
				- value // NFT 详情？
				- date // 字段类型:时间格式 //可覆盖
				- owner // 所有者 //可覆盖         

		- [Token](https://github.com/rarible/ethereum-indexer/blob/master/nft/core/src/main/kotlin/com/rarible/protocol/nft/core/model/Token.kt)

			代表区块链中的智能合约（集合）。项目存在于这些智能合约中。
			
			- Token  表字段包括
				- id // 什么地址  //可覆盖  
				- owner // 所有者
				- name // token 名称
				- symbol // 币名称
				- status //状态
				- features //特性
				- lastEventId //最后事件id
				- standard // 标准
				- version // 更好的做法是引入 TokenVersion 枚举（RPN-1264）？
				- isRaribleContract // 是否是 rarible 合约，字段类型，布尔，默认false
				- deleted // 是否销毁 ，字段类型，布尔，默认false
				- revertableEvents //字段类型 list   //可覆盖
- order 订单索引器目录

	存储链外订单，侦听基于 evm 的区块链并为所有订单重建状态。
	
	- 支持以下订单类型:
		- Rarible V1 (链下)订单
		- Rarible V2 (链下)订单
		- Rarible V2 (链上)订单
		- Rarible 拍卖(WIP) ？
		- TBA ？
	
	结构
	
	- [API](https://github.com/rarible/ethereum-indexer/blob/master/order/api)

		该模块控制器功能

		- 查询订单数据(所有订单、卖出订单、出价、按项目、按集合等)
		- 创建或更新订单
		- 查询事件信息
		- 发布关于未决事务的信息
	- [监听](https://github.com/rarible/ethereum-indexer/blob/master/order/listener)

		该模块监听事件并相应地更新数据。它基于[日志侦听器](https://github.com/rarible/ethereum-core/tree/master/listener-log)
	- 数据模型
		- [Order](https://github.com/rarible/ethereum-indexer/blob/master/order/core/src/main/kotlin/com/rarible/protocol/order/core/model/Order.kt)

			表示用户将其资产转换为其他资产的意图
			
			- Order 表字段包括
				- 通用字段	 
					- maker //订单创作者
					- taker //订单接收者
					- make // 订单资产
					- take // 订单接收资产
					- type // 订单类型
					- fill // 填充信息
					- cancelled // 订单是否取消，字段类型布尔
					- makestock // ?
					- salt //?
					- start // 开始？
					- end // 结束
					- data // 订单时间
					- signature //签名
					- createAT // 创建订单时间
					- lastUpdateAT // 最后变更时间
				- 状态字段
					- pending // 待处理列表
				- 销售字段
					- makePriceUsd // 订单价格(汇率后的美金价格)
					- takePriceUsd // 订单购买价格 (汇率后的美金价格)
					- makePrice // 订单价格
					- takePrice // 订单购买价格
					- makeusd //?
					- takeusd //?
					- priceHistory // 订单价格历史列表，字段列表类型，默认空
					- status // 订单状态
					- platform // 订单平台
					- lastEventId //  订单最后时间 id
					- hash // 订单 hash id? //算法 `hashKey(maker, make.type, take.type, salt.value, data)`
					- version //版本？
		- [OrderExchangeHistory](https://github.com/rarible/ethereum-indexer/blob/master/order/core/src/main/kotlin/com/rarible/protocol/order/core/model/OrderExchangeHistory.kt)

			从区块链监听的事件。通常这些事件信号顺序被填充(可能是部分填充)或取消。

			订单索引器支持不同种类的资产
			
			- ERC-20
			- erc - 721
			- erc - 1155
			- ETH
			- 和其他

			支持的资产的完整列表可以在[这里](https://github.com/rarible/ethereum-indexer/blob/master/order/core/src/main/kotlin/com/rarible/protocol/order/core/model/AssetType.kt)查看
			
			- OrderSideMatch 表字段包括 // 匹配订单
				- hash // 订单hash , 验证用？
				- counterHash // ？
				- side // 买方还是卖方？
				- fill //？
				- maker // 订单创建者
				- taker //订单接收者
				- make // 创建者指定资产
				- take // 接收者指定资产
				- makeUsd // 创建者法币出价？
				- takeUsd // 接收者发币出价
				- makePriceUsd // 汇率价格？
				- takePriceUsd //汇率价格？
				- makeValue // 积分值
				- takeValue // 积分值
				- date // 时间
				- source // 源头？
				- externalOrderExecutedOnRarible //  外部订单是否在 rarible 上？
				- date // 订单时间
				- adhoc // ? 字段类型布尔
				- counterAdhoc // ？字段类型布尔
			- OrderCancel  表字段包括 // 取消订单
				- hash // 订单hash , 验证用？
				- maker // 订单创建者
				- make // 创建者指定资产
				- take // 接收者指定资产
				- date // 时间
				- source // 历史源头？
			- OnChainOrder   表字段包括 //在链上订单字段
				- maker // 订单创建者
				- taker //订单接收者
				- make // 创建者指定资产
				- take // 接收者指定资产
				- createAT // 创建订单时间
				- platform // 订单平台
				- salt //?
				- start // 开始？
				- end // 结束
				- data // 订单时间
				- signature //签名
				- PriceUsd // 价格
				- hash // 订单hash , 验证用？
				- date // 等于 createAT
				- source // 历史源头？
- nft-order 目录
	- 简介
	
		该服务连接 [NFT indexer](https://github.com/rarible/ethereum-indexer/blob/master/nft)和 [order indexer](https://github.com/rarible/ethereum-indexer/blob/master/order)。它基本上公开了与 NFT indexer 有相同的接口，但还为 `Item` 和 `Ownership` 添加了一些额外的字段。

	- 扩展域模型
		- [item](https://github.com/rarible/ethereum-indexer/blob/master/nft-order/core/src/main/kotlin/com/rarible/protocol/nftorder/core/model/Item.kt)

			扩展了下面2个字段
			
			- bestSellOrder：最佳销售订单，卖出订单最低的
			- bestBidOrder：最佳出价订单，买入订单最高的
			- 可解锁：如果物品可解锁
		
			字段说明
			
			- Item 表字段包括
				- 通用字段 
					- token // 合约地址
					- tokenid // tokenid
					- creator //创建者,字段类型 list，默认是空
					- supply // 数量
					- lazySupply // 惰性数量
					- royalties // 税率
				- 所有者字段
					- owners // 所有者,类型 list
					- date //创建时间？
				- 状态字段
					- pending // 待处理列表
				- 出售字段
					- sellers //字段类型整数，默认0
					- totalStock // 字段类型, 长整型
					- bestSellOrder // 最佳销售订单，卖出订单最低的
					- bestBidOrder  // 最佳出价订单，买入订单最高的
					- unlockable // 交易是否锁定，字段类型布尔
					- version //版本
		- [ownership](https://github.com/rarible/ethereum-indexer/blob/master/nft-order/core/src/main/kotlin/com/rarible/protocol/nftorder/core/model/Ownership.kt)
			
			扩展了下面1个字段
			
			- bestSellOrder：最佳销售订单，卖出订单最低的
			
			字段说明
			
			- ownership 表字段包括
				- 通用字段 
					- contract // 合约地址- 没有用 token很奇怪？
					- tokenid // tokenid
					- creator //创建者,字段类型 list，默认是空
					- owner // 所有者
					- value // ？
					- lazyValue //?
					- date //?字段类型:时间格式
				- 状态字段
					- pending // 待处理列表
				- 销售字段
					- bestSellOrder // 最佳销售订单，卖出订单最低的 
		
		本协议订单比其他平台的优先级更高。因此，如果在 Rarible 协议上下订单，则认为它比在其他平台上下订单更好。 
- unlockable 系统锁？

##  参考
[ethereum-indexer](https://github.com/rarible/ethereum-indexer)
