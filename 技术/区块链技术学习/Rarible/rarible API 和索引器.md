## rarible API 和索引器
### 概述
API 和索引器的主要功能

- 关注区块链
- 处理读取请求
- 进程创建请求

#### 架构
Rarible Protocol 以太坊索引器由以下四个子索引器组成：

- [NFT 索引器](https://github.com/rarible/ethereum-indexer/blob/master/nft) 聚合 NFT 数据
- [ERC-20 索引器](https://github.com/rarible/ethereum-indexer/blob/master/erc20) 聚合有关 ERC-20 代币和余额的数据
- [订单索引器](https://github.com/rarible/ethereum-indexer/blob/master/order):聚合来自不同平台的订单数据
- [NFT-order](https://github.com/rarible/ethereum-indexer/blob/master/nft-order) 将 nft 和 order 索引器连接在一起

每个子索引器都会监听以太坊区块链的`特定部分`。索引器可用于请求有关区块链状态的数据。索引器工作是在区块链状态更改时生成事件。它们是用 Spring Framework 开发的，并使用这些外部服务：

- MongoDB:主要数据存储
- Apache Kafka:事件处理

![](./pic/index.png)

#### 控制器
1. 创建或修改 NFT 并搜索有关它们的信息
	- nft-交易控制器
	- nft-惰性铸币控制器
	- nft-状态控制器
	- nft-所有权控制器
	- nft-实例控制器
	- nft-合约集合控制器
- 创建或修改订单并搜索有关订单的信息：
	- 订单签名控制器
	- 顺序编码控制器
	- 订单控制器
	- 订单交易控制器
	- 订单活动控制器
	- 订单聚合控制器
	- nft-订单-所有权-控制器
	- nft-order-item-controller
	- nft-order-activity-controller
	- nft-order-collection-controller
- 附加控制器
	- 网关控制器
	- erc20-余额控制器
	- erc20-token-控制器
	- 锁控制器

### API 使用示例
#### 使用惰性铸造
要使用 Lazy Minting 铸造 NFT，请使用 [nft-lazy-mint-controller](https://ethereum-api.rarible.org/v0.1/doc#tag/nft-lazy-mint-controller) 中的 mintNftAsset 方法。

- [惰性铸造 Nft 资产](https://ethereum-api.rarible.org/v0.1/doc#operation/mintNftAsset)

	创建一个 Lazy Minted NFT 代币

		https://ethereum-api-staging.rarible.org/v0.1/nft/mints

	- 示例请求（暂存）


			curl --request POST 'https://ethereum-api-staging.rarible.org/v0.1/nft/mints' \
			  --header 'content-type: application/json' \
			  --data-raw '{
			  "contract":"0x6ede7f3c26975aad32a475e1021d8f6f39c89d82",
			  "uri":"/ipfs/QmWeVMxWPbz2AdGhrZdi3ZQXVdppzPPakXfdavtu9syyA7",
			  "royalties":[{"account":"0xb7d1f311ef648f0a9d9eff2b47f5bbb53e583dca","value":1000}],
			  "creators":[{"account":"0xb7d1f311ef648f0a9d9eff2b47f5bbb53e583dca","value":10000}],
			  "tokenId":"83144199935168800027256855471009236590928205711995533202494108027144764391425",
			  "@type":"ERC721","
			  signatures":["0xf91e21cda6fc519f4b3ea77086393482a65f715b3e981da8d31c753bf5c5060018b4460cc9935f397808d14365e64214c18d0fc46441b55e0858a398ff7ffc8c1b"]
			  }'
	- 参数说明
		- `@typerequired` — 代币类型 ERC721 或 ERC1155
		- `supply` — Mint 的代币总数。仅适用于 ERC-1155
		- `contract` — 智能合约的地址
		- `tokenId` — 代币 ID
		- `uri`  — 代币 URI 的后缀。前缀通常是ipfs:/
		- `creators` — 作者地址数组
		- `royalties` — 版税数组
		- `signatures` — 数字签名数组。每个创建者都必须有一个签名，唯一的例外是当创建者发送一个铸币交易时。
	- 响应示例(状态 200)

			{
			  "id": "0x6ede7f3c26975aad32a475e1021d8f6f39c89d82:83144199935168800027256855471009236590928205711995533202494108027144764391425",
			  "contract": "0x6ede7f3c26975aad32a475e1021d8f6f39c89d82",
			  "tokenId": "83144199935168800027256855471009236590928205711995533202494108027144764391425",
			  "creators": [
			    {
			      "account": "0xb7d1f311ef648f0a9d9eff2b47f5bbb53e583dca",
			      "value": 10000
			    }
			  ],
			  "supply": "1",
			  "lazySupply": "1",
			  "owners": [
			    "0xb7d1f311ef648f0a9d9eff2b47f5bbb53e583dca"
			  ],
			  "royalties": [
			    {
			      "account": "0xb7d1f311ef648f0a9d9eff2b47f5bbb53e583dca",
			      "value": 1000
			    }
			  ],
			  "date": "2021-11-23T15:37:03.610Z",
			  "pending": [],
			  "deleted": false,
			  "meta": {
			    "name": "Violence Token",
			    "description": "",
			    "attributes": [],
			    "image": {
			      "url": {
			        "ORIGINAL": "ipfs://ipfs/QmbnW9qS5dcmXavo55yqzmF79wWT6qTcYKBAAaS3gDgrDa/image.jpeg"
			      },
			      "meta": {
			        "ORIGINAL": {
			          "type": "image/jpeg",
			          "width": 400,
			          "height": 400
			        }
			      }
			    }
			  }
			}
	- 响应参数
		- `id` — 项目 ID，具有格式 `${contract}:${tokenId}`
		- `contract` — 智能合约的地址
		- `tokenId` — 代币 ID
		- `creators` — 有关创建者的信息数组
		- `supply `— 创建的代币数量, 721 默认是1
		- `lazysupply` — 创建的 Lazy Minting 代币的数量.# 这里还可能有同时惰性铸造和非惰性铸造？
		- `owner` — 有关代币所有者的信息数组
		- `royalties ` ——关于版税的一系列信息
		- `date` — 代币创建日期
		- `pending ` - 项目是否不完整。例如，它处于状态 `TRANSFER`
		- `deleted` — 是否已删除订单
		- `meta` — 关于项目的元信息
		
#### 搜索藏品
使用藏品的主要请求与 [nft-item-controller](https://ethereum-api.rarible.org/v0.1/doc#tag/nft-item-controller) 相关。让我们看一下 `getNftAllItems` 的例子。

- [获取NftAllItems](https://ethereum-api.rarible.org/v0.1/doc#operation/getNftAllItems)

	它将返回所有 NFT 项目。

	- 示例请求
		
			curl --request GET 'https://ethereum-api.rarible.org/v0.1/nft/items/all?size=1'
	- 请求参数
		- `size` — 要返回的项目数
		- `showDeleted` — 是否显示已删除的项目
		- `lastUpdatedFrom` — 返回自该日期以来已更新的项目（以毫秒为单位的时间戳）
		- `lastUpdatedTo` — 返回已更新到此日期的项目（以毫秒为单位的时间戳）
		- `continuation` — 上一个响应的延续代币

	- 响应示例（状态 200）：

			{
			    "total": 1,
			    "continuation": "1637677864204_0xf6793da657495ffeff9ee6350824910abc21356c:0x8108800667cb3859020c77f7643cedb794b4455700000000000000000000000e",
			    "items": [
			        {
			            "id": "0xf6793da657495ffeff9ee6350824910abc21356c:58363375839982426315252321964399886024230569048144758096248518895130164330510",
			            "contract": "0xf6793da657495ffeff9ee6350824910abc21356c",
			            "tokenId": "58363375839982426315252321964399886024230569048144758096248518895130164330510",
			            "creators": [
			                {
			                    "account": "0x8108800667cb3859020c77f7643cedb794b44557",
			                    "value": 10000
			                }
			            ],
			            "supply": "1",
			            "lazySupply": "1",
			            "owners": [
			                "0x8108800667cb3859020c77f7643cedb794b44557"
			            ],
			            "royalties": [
			                {
			                    "account": "0x8108800667cb3859020c77f7643cedb794b44557",
			                    "value": 1000
			                }
			            ],
			            "date": "2021-11-23T14:31:04.204Z",
			            "pending": [],
			            "deleted": false,
			            "meta": {
			                "name": "Rampows the Daltons",
			                "description": "",
			                "attributes": [],
			                "image": {
			                    "url": {
			                        "ORIGINAL": "ipfs://ipfs/QmWXtdxkrR33rNxoNAbUDELsv2YBjajE4nhHGrgwT7UBHQ/image.jpeg"
			                    },
			                    "meta": {
			                        "ORIGINAL": {
			                            "type": "image/jpeg",
			                            "width": 1792,
			                            "height": 1401
			                        }
			                    }
			                }
			            }
			        }
			    ]
			}
	- 响应参数说明
		- `total ` — 根据请求返回的项目数
		- `continuation` — 上一个响应的延续代币
		- `items` — 找到的项目列表以及关于它们的基本信息

#### 创建订单
要创建订单，请使用 [order-controller](https://ethereum-api.rarible.org/v0.1/doc#tag/order-controller) 中的 upsertOrder 方法。

- [插入订单](https://ethereum-api.rarible.org/v0.1/doc#operation/upsertOrder)

	创建或更新订单。

		https://ethereum-api.rarible.org/v0.1/order/orders
	
	- 示例请求（暂存）：

			curl --request POST 'https://ethereum-api-staging.rarible.org/v0.1/order/orders' \
			  --header 'content-type: application/json' \
			  --data-raw '{"maker":"0xb7d1f311ef648f0a9d9eff2b47f5bbb53e583dca",
			  "type":"RARIBLE_V2",
			  "data":{"dataType":"RARIBLE_V2_DATA_V1",
			  "payouts":[],
			  "originFees":[{"account":"0x76c5855e93bd498b6331652854c4549d34bc3a30","value":250}]},
			  "salt":"86881339267547208763979074509610437060593762063071194041427040496432360352297",
			  "signature":"0xd3147a72aa61e2ff970c05dac9b87c4a7187a5d0d6dc125f187e40612cea73d637608995704da1d35dc0f92c153811167860a85889de9b96c9c80e9b960d80681c",
			  "make":{"assetType":{"@type":"ERC721",
			  "contract":"0x6ede7f3c26975aad32a475e1021d8f6f39c89d82",
			  "tokenId":"83144199935168800027256855471009236590928205711995533202494108027144764391425",
			  "uri":"/ipfs/QmWeVMxWPbz2AdGhrZdi3ZQXVdppzPPakXfdavtu9syyA7",
			  "creators":[{"account":"0xb7d1f311ef648f0a9d9eff2b47f5bbb53e583dca","value":10000}],
			  "royalties":[{"account":"0xb7d1f311ef648f0a9d9eff2b47f5bbb53e583dca","value":1000}],
			  "signatures":["0xf91e21cda6fc519f4b3ea77086393482a65f715b3e981da8d31c753bf5c5060018b4460cc9935f397808d14365e64214c18d0fc46441b55e0858a398ff7ffc8c1b"],
			  "assetClass":"ERC721_LAZY"},"value":"1"},
			  "take":{"assetType":{"assetClass":"ETH"},
			  "value":"10000000000000000"}}'
	- 请求参数
		- `type` — 订单类型 `RARIBLE_V1` 或 `RARIBLE_V2`
		- `data` — 用于创建或更新订单的数据。必填字段取决于订单类型
		- `maker` — 订单创建者的地址
		- `taker` — 订单接收者的地址（可选）
		- `make` — 制作订单的一边。创造者有什么
		- `take` — 另一边。创作者想用制作方换取什么
		- `salt` — 与输入数据数组一起传递给散列函数以计算散列的数据字符串
		- `start` — 下单的开始日期，买方可以从该日期开始出价（可选） 使用 `ps unix` 时间
		- `end` — 下单的结束日期，在此之前买家可以出价（可选） 使用 `ps unix` 时间
		- `signature ` — 订单创建者的数字签名
	- 响应示例（状态 200）：

			{
			  "type": "RARIBLE_V2",
			  "maker": "0xb7d1f311ef648f0a9d9eff2b47f5bbb53e583dca",
			  "make": {
			    "assetType": {
			      "assetClass": "ERC721_LAZY",
			      "contract": "0x6ede7f3c26975aad32a475e1021d8f6f39c89d82",
			      "tokenId": "83144199935168800027256855471009236590928205711995533202494108027144764391425",
			      "uri": "/ipfs/QmWeVMxWPbz2AdGhrZdi3ZQXVdppzPPakXfdavtu9syyA7",
			      "creators": [
			        {
			          "account": "0xb7d1f311ef648f0a9d9eff2b47f5bbb53e583dca",
			          "value": 10000
			        }
			      ],
			      "royalties": [
			        {
			          "account": "0xb7d1f311ef648f0a9d9eff2b47f5bbb53e583dca",
			          "value": 1000
			        }
			      ],
			      "signatures": [
			        "0xf91e21cda6fc519f4b3ea77086393482a65f715b3e981da8d31c753bf5c5060018b4460cc9935f397808d14365e64214c18d0fc46441b55e0858a398ff7ffc8c1b"
			      ]
			    },
			    "value": "1",
			    "valueDecimal": 1
			  },
			  "take": {
			    "assetType": {
			      "assetClass": "ETH"
			    },
			    "value": "10000000000000000",
			    "valueDecimal": 0.01
			  },
			  "fill": "0",
			  "fillValue": 0,
			  "makeStock": "1",
			  "makeStockValue": 1,
			  "cancelled": false,
			  "salt": "0xc015186be955e586fb9d3bc1ddbbf43470a40d18b4ccdc750af405155eee6a29",
			  "signature": "0xd3147a72aa61e2ff970c05dac9b87c4a7187a5d0d6dc125f187e40612cea73d637608995704da1d35dc0f92c153811167860a85889de9b96c9c80e9b960d80681c",
			  "createdAt": "2021-11-23T15:37:13.158Z",
			  "lastUpdateAt": "2021-11-23T15:37:13.158Z",
			  "pending": [],
			  "hash": "0x279863b844f7bae2db201bff5abd9467f380eb029399e1339671d9d51f9a0280",
			  "makeBalance": "0",
			  "makePrice": 0.01,
			  "makePriceUsd": 40.193574609729986,
			  "priceHistory": [
			    {
			      "date": "2021-11-23T15:37:13.158Z",
			      "makeValue": 1,
			      "takeValue": 0.01
			    }
			  ],
			  "status": "ACTIVE",
			  "data": {
			    "dataType": "RARIBLE_V2_DATA_V1",
			    "payouts": [],
			    "originFees": [
			      {
			        "account": "0x76c5855e93bd498b6331652854c4549d34bc3a30",
			        "value": 250
			      }
			    ]
			  }
			}
	- 响应参数
		- `type` — 订单类型 `RARIBLE_V1`, `RARIBLE_V2`,`OPEN_SEA_V1` 或 `CRYPTO_PUNK`
		- `data` — 有关订单类型、费用等的信息。
		- `maker` — 订单创建者的地址
		- `taker` — 订单接收者的地址
		- `make` — 制作订单方。创造者有什么
		- `take` — 另一边创作者想用制作方换取什么
		- `fill` — 匹配时填充数据
		- `fillvalue` — 数据填充值
		- `start` — 下订单的开始日期，买方可以从该日期开始出价
		- `end` — 下订单的结束日期，在此之前买方可以出价
		- `makestock` — 有多少可供出售
		- `makestockvalue` — 可供出售的价值
		- `cancelled` - 订单是否取消
		- `salt` — 与输入数据数组一起传递给散列函数以计算散列的数据字符串
		- `signature` — 订单创建者的数字签名
		- `createdAt` — 订单创建日期
		- `lastupdateat` — 订单更新日期
		- `pending ` — 订单是否不完整。例如，它处于 `CANCEL，ORDER_SIDE_MATCH` 或 `ON_CHAIN_ORDER`
		- `hash` — 订单的哈希值
		- `makebalance` — 订单创建者的余额
		- `makeprice` — 订单价格
		- `Takeprice` — 买方的建议价格
		- `makepriceusd` — 以美元为单位的订单价格
		- `takepriceusd` — 买方建议的美元价格
		- `pricehistory` — 价格变化的历史
		- `status` — 订单状态ACTIVE, FILLED, HISTORICAL,INACTIVE或CANCELLED
		
#### 搜索订单
使用 Orders 的主要请求是指 [order-controller](https://ethereum-api.rarible.org/v0.1/doc#tag/order-controller)。让我们看一下 getOrdersAllByStatus 的例子。

- [getOrdersAllByStatus](https://ethereum-api.rarible.org/v0.1/doc#operation/getOrdersAllByStatus)

	返回按状态排序的所有订单。

	- 示例请求

			curl --request GET 'https://ethereum-api.rarible.org/v0.1/order/orders/all/byStatus?sort=LAST_UPDATE_DESC&size=1&status=ACTIVE'
	- 请求参数
		- `sort` — 按最近更新的订单排序 `LAST_UPDATE_ASC`，`LAST_UPDATE_DESC`
		- `size` — 要返回的订单数量
		- `continuation` — 上一个响应的延续代币
		- `status` — 订单状态 `ACTIVE, FILLED, HISTORICAL, INACTIVE,CANCELLED`
	- 响应示例（状态 200）：


			{
			    "orders": [
			        {
			            "type": "RARIBLE_V2",
			            "maker": "0x1a68bab3ebe18ffe95508b2d4e1362addc2cbdd6",
			            "make": {
			                "assetType": {
			                    "assetClass": "ERC20",
			                    "contract": "0xc02aaa39b223fe8d0a0e5c4f27ead9083c756cc2"
			                },
			                "value": "260000000000000000",
			                "valueDecimal": 0.260000000000000000
			            },
			            "take": {
			                "assetType": {
			                    "assetClass": "ERC721",
			                    "contract": "0x918f677b3ab4b9290ca96a95430fd228b2d84817",
			                    "tokenId": "260"
			                },
			                "value": "1",
			                "valueDecimal": 1
			            },
			            "fill": "0",
			            "fillValue": 0,
			            "makeStock": "260000000000000000",
			            "makeStockValue": 0.260000000000000000,
			            "cancelled": false,
			            "salt": "0xf7f9ca62eceb7ee9021a89ac2a77976287721cc56a28f9d8d12c1332898e12e4",
			            "signature": "0x382bd104f4497119c75b36e5f0954a03692ab1c3b995266b20623bc718a58f423ff68e2caacb93668811e52b6a099260b3147ded35dc3ad5d65970efde9587761c",
			            "createdAt": "2021-11-23T15:08:20.925Z",
			            "lastUpdateAt": "2021-11-23T15:08:20.925Z",
			            "pending": [],
			            "hash": "0x3e23919b0ad64895792c49bb40a551a8e60d70081eb940fa3d1a56c17436eedf",
			            "makeBalance": "0",
			            "takePrice": 0.260000000000000000,
			            "takePriceUsd": 1089.887807206807960000000000000000,
			            "priceHistory": [
			                {
			                    "date": "2021-11-23T15:08:20.925Z",
			                    "makeValue": 0.260000000000000000,
			                    "takeValue": 1
			                }
			            ],
			            "status": "ACTIVE",
			            "data": {
			                "dataType": "RARIBLE_V2_DATA_V1",
			                "payouts": [],
			                "originFees": [
			                    {
			                        "account": "0x1cf0df2a5a20cd61d68d4489eebbf85b8d39e18a",
			                        "value": 250
			                    }
			                ]
			            }
			        }
			    ],
			    "continuation": "1637680100925_3e23919b0ad64895792c49bb40a551a8e60d70081eb940fa3d1a56c17436eedf"
			}
	- 响应参数：
		- `orders` — 找到的订单列表及其基本信息
		- `continuation` — 上一个响应的延续代币

## 元数据
### 以太坊元数据
提供资产元数据允许应用程序提取数字资产的数据并将其显示在应用程序中。URI 通常代表智能合约中的数字资产。元数据允许资产具有附加属性，例如名称、描述和图像。

- 代币 URI

	要获取 ERC-721 和 ERC-1155 的元数据，需要返回 URI。为此，请使用以下函数：

	- `tokenURI` 在 [ERC-721](https://github.com/rarible/protocol-contracts/blob/master/tokens/contracts/erc-721/ERC721Upgradeable.sol)

			/**
		     * @dev See {IERC721Metadata-tokenURI}.
		     */
		    function tokenURI(uint256 tokenId) public view virtual override returns (string memory) {
		        require(_exists(tokenId), "ERC721Metadata: URI query for nonexistent token");
		
		        string memory _tokenURI = _tokenURIs[tokenId];
		        string memory base = baseURI();
		
		        //如果没有 baseURI，则返回代币 URI。
		        if (bytes(base).length == 0) {
		            return _tokenURI;
		        }
		        // 如果两者都设置了，则连接 baseURI 和 tokenURI(通过abi. encodepack)。
		        if (bytes(_tokenURI).length > 0) {
		            return LibURI.checkPrefix(base, _tokenURI);
		        }
		        // 如果有一个 baseURI 但没有 tokenURI，则将 tokenID 连接到 baseURI。
		        return string(abi.encodePacked(base, tokenId.toString()));
		    }
	- `uri` 在 [ERC-1155](https://github.com/rarible/protocol-contracts/blob/master/tokens/contracts/erc-1155/ERC1155Upgradeable.sol) 

		    /**
		     * @dev See {IERC1155MetadataURI-uri}.
		     *
		     * 这个实现为*所有*代币类型返回相同的URI
		     * 它依赖于代币类型 ID 替换机制 https://eips.ethereum.org/EIPS/eip-1155#metadata[在EIP中定义]
		     *
		     * 调用此函数的客户端必须用实际的代币类型id替换' \{id\} '子字符串
		     */
		    function uri(uint256) external view virtual override returns (string memory) {
		        return _uri;
		    }

该 `tokenURI` 或 `uri` 函数返回一个 HTTP 或 IPFS URL。请求 URL 时，应返回带有代币元数据的 JSON。

- 元数据结构

	Rarible Ethereum 协议支持符合 [EIP-721](https://eips.ethereum.org/EIPS/eip-721) 和 [EIP-1155](https://eips.ethereum.org/EIPS/eip-1155) 标准的元数据结构。

	ERC-1155 NFT 的元数据结构示例：

		{
		    "name": "CryptoParrot#17",
		    "description": "CryptoParrot 是 10,000 个独特的 Parrot nft 的集合",
		    "attributes": [
		        {
		            "key": "/background",
		            "value": "rust"
		        },
		        {
		            "key": "/body",
		            "value": "lavender down"
		        },
		        {
		            "key": "/color",
		            "value": "gray"
		        },
		        {
		            "key": "/eye",
		            "value": "small 1"
		        },
		        {
		            "key": "/head",
		            "value": "navy blue nightcap"
		        },
		        {
		            "key": "/mouth",
		            "value": "yellow 6"
		        }
		    ],
		    "image": {
		        "url": {
		            "ORIGINAL": "ipfs://ipfs/QmUoc2LDDnHxHsesLXtpxTLupzVuyfVkJomWWHmvKNCjrL/image.png"
		        },
		        "meta": {
		            "ORIGINAL": {
		                "type": "image/png",
		                "width": 999,
		                "height": 999
		            }
		        }
		    }
		}
		
	- 属性说明

		名称|参数|描述
		---|---|---
		description||项目的人类可读描述
		attributes|key, value|这些是物品的属性
		image|url|这是物品图片的网址
		||meta|这是关于媒体的元信息。包括类型、宽度和高度
		animation|url|这是项目动画的 URL
		||meta|这是关于媒体的元信息。包括类型、宽度和高度
		
		对于 Rarible Ethereum 协议，NFT 的元数据放置在哪里并不重要。请参阅使用 [IPFS](https://ipfs.io/)[上传](https://docs.rarible.org/ethereum/metadata/ipfs-example/)和使用元数据的[示例](https://docs.rarible.org/ethereum/metadata/ipfs-example/)。

### 使用 IPFS 上传和使用元数据的示例
- 上传图片到 IPFS

	要将图像上传到 IPFS，将使用 [Pinata](https://www.pinata.cloud/) 服务。
	
	在这里，您可以看到使用 Node JS 使用 Pinata API 上传图像的示例。
	
		const axios = require("axios");
		const fs = require("fs");
		const FormData = require("form-data");
		
		export const pinFileToIPFS = (pinataApiKey, pinataSecretApiKey) => {
		  const url = `https://api.pinata.cloud/pinning/pinJSONToIPFS`;
		  let data = new FormData();
		
		  data.append("file", fs.createReadStream("./yourfile.png"));
		
		  return axios.post(url, data, {
		      headers: {
		        "Content-Type": `multipart/form-data; boundary= ${data._boundary}`,
		        pinata_api_key: pinataApiKey,
		        pinata_secret_api_key: pinataSecretApiKey,
		      },
		    })
		    .then(function (response) {
		      console.log(repsonse.IpfsHash);
		    })
		    .catch(function (error) {
		      console.log(error)
		    });
		};
	对请求的响应：
	
		{
		    IpfsHash: // 这是为内容提供的 IPFS 哈希
		    PinSize: // 这是锁定的内容的大小(以字节为单位)
		    Timestamp: // 这是内容固定的时间戳(以ISO 8601格式表示)
		}
- 为 NFT 创建元数据文件

	使用 `IpfsHash`，我们可以创建元数据文件。它将连接到区块链网络内的 NFT。
	
		{
		   "name": /* NFT名称-这必须是一个字符串 */,
		   "description": /* NFT的描述-这必须是一个字符串 */,
		   "image": /*  内容的 IPFS 哈希，这必须以 "ipfs://ipfs/{{ IPFS_HASH ))" - 必须是一个字符串 */,
		   "external_url": /* 这是我们目前没有的 Rarible 链接，我们可以在短时间内填充 */,
		   "animation_url": /* IPFS 哈希就像图像字段，但它允许任何类型的多媒体文件。像mp3, mp4等*/,
		   // 下面的部分可以不要.
		   "attributes": [
		      {
		         "key": /* Key name - 必须是一个字符串 */,
		         "trait_type": /* Trait name - 必须是一个字符串 */,
		         "value": /* Key Value - 必须是一个字符串 */
		      }
		   ]
		}
- 将生成的元数据添加到 IPFS
	1. `external_url` 以格式指定 `${contractAddress}:${tokenId}`，例如：
	
			"external_url": "https://app.rarible.com/0x60f80121c31a0d46b5279700f9df786054aa5ee5:123913"
	- 将元数据发布到 IPFS：
	
			var axios = require('axios');
			var data = JSON.stringify({"name":"Test NFT","description":"Test NFT","image":"ipfs://ipfs/QmW4P1Mgoka8NRCsFAaJt5AaR6XKF6Az97uCiVtGmg1FuG/image.png","external_url":"https://app.rarible.com/0x60f80121c31a0d46b5279700f9df786054aa5ee5:123913","attributes":[{"key":"Test","trait_type":"Test","value":"Test"}]});
			
			var config = {
			  method: 'post',
			  url: 'https://api.pinata.cloud/pinning/pinFileToIPFS',
			  headers: { 
			    'pinata_api_key': // KEY_HERE, 
			    'pinata_secret_api_key': // SECRET_KEY_HERE, 
			    'Content-Type': 'application/json'
			  },
			  data: data
			};
			
			axios(config).then(function (response) {
			  console.log(JSON.stringify(response.data));
			}).catch(function (error) {
			  console.log(error);
			});
		- 响应示例：
	
				{
				    "IpfsHash": "QmNybufJtuvWCZ355HGejvKfUXK8VeLcPA5G7CxT9MXJJp",
				    "PinSize": 290,
				    "Timestamp": "2021-02-10T14:06:09.255Z"
				}
	- 将新的附加 `IpfsHash` 到您的 NFT

## 以太坊 SDK
[Rarible 协议 Ethereum SDK](https://github.com/rarible/protocol-ethereum-sdk) 使应用程序能够轻松地与 Rarible 协议交互。使用案例 [React 的示例应用程序](https://github.com/rarible/protocol-ethereum-sdk) 以快速开始。
### 安装
- npm 安装
	
		npm install -D @rarible/protocol-ethereum-sdk
- 或使用 web3 实例将包注入到您的网页中

		<script src="https://unpkg.com/@rarible/web3-ethereum@0.10.0/umd/rarible-web3-ethereum.js" type="text/javascript"></script>
		<script src="https://unpkg.com/@rarible/protocol-ethereum-sdk@0.10.0/umd/rarible-ethereum-sdk.js" type="text/javascript"></script>
		<script src="https://unpkg.com/web3@1.6.0/dist/web3.min.js" type="text/javascript"></script>

### 与 web3.js 一起使用
- 配置和创建 Rarible SDK 对象

		import { createRaribleSdk } from "@rarible/protocol-ethereum-sdk"

		const sdk = createRaribleSdk(web3, env, { fetchApi: fetch })

	- `web3` - 使用您的提供商 web3js 客户端配置
	- `ENV` -环境配置的名称，应该接受这些值中的一个 `ropsten`，`rinkeby`，`mainnet` 或者 `e2e`
- 在浏览器中配置 Rarible SDK

		const web = new Web3(ethereum)
		const web3Ethereum = new window.raribleWeb3Ethereum.Web3Ethereum({ web3: web })
		const env = "mainnet" // "e2e" | "ropsten" | "rinkeby" | "mainnet"
		const raribleSdk = new window.raribleEthereumSdk.createRaribleSdk(web3Ethereum, env)
	- `ethereum` - metamask 浏览器实例 (window.ethereum)
	
有关使用 Rarible 协议以太坊 SDK 的更多信息，请参阅 GitHub 上的协议以太坊 SDK[页面](https://github.com/rarible/protocol-ethereum-sdk)。

					