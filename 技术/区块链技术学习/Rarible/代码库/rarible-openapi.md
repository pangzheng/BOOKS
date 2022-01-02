# [rarible-openapi](https://ethereum-api.rarible.org/v0.1/doc)
## gateway-controller 网关控制器
### 方法
- [createGatewayPendingTransactions](](https://ethereum-api.rarible.org/v0.1/doc#operation/createGatewayPendingTransactions)) // 用网关创建交易
	- 接口

			https://ethereum-api.rarible.org/v0.1/transactions
	- 请求
		- 方法 post		 
		- 参数说明

				{
					"hash": "string",
					"from": "0x60f80121c31a0d46b5279700f9df786054aa5ee5",
					"nonce": 0,
					"to": "0x60f80121c31a0d46b5279700f9df786054aa5ee5",
					"input": "string"
				}
	- 返回

			{
			  "hash": "string",
			  "from": "0x60f80121c31a0d46b5279700f9df786054aa5ee5",
			  "nonce": 0,
			  "to": "0x60f80121c31a0d46b5279700f9df786054aa5ee5",
			  "input": "string"
			}
		
## erc20-balance-controller // ERC20 余额控制器
### 方法
- [getErc20Balance](https://ethereum-api.rarible.org/v0.1/doc#operation/getErc20Balance) // erc 20 拥有者余额查询 
	- 接口

			https://ethereum-api.rarible.org/v0.1/erc20/balances/{contract}/{owner}
	- 请求
		- 方法 get		 
		- 参数说明

				curl --request get \
				--url 'https://ethereum-api.rarible.org/v0.1/erc20/balances/0x60f80121c31a0d46b5279700f9df786054aa5ee5/0x60f80121c31a0d46b5279700f9df786054aa5ee5'
	- 返回
	
			{
			"contract": "0x60f80121c31a0d46b5279700f9df786054aa5ee5",
			"owner": "0x60f80121c31a0d46b5279700f9df786054aa5ee5",
			"balance": 717802,
			"decimalBalance": 717802.342336
			}

## erc20-token-controller // ERC 20 代币控制器 
### 方法
- [getErc20TokenById](https://ethereum-api.rarible.org/v0.1/doc#tag/erc20-token-controller)  // 根据合约 ID 查询 ERC20 代币
	- 接口

			https://ethereum-api.rarible.org/v0.1/erc20/tokens/{contract}
	- 请求
		- 方法 get
		- 参数说明
			- contract //合约地址
	- 返回

			{
			"id": "0x60f80121c31a0d46b5279700f9df786054aa5ee5",
			"name": "string",
			"symbol": "string"
			} 

## nft-transaction-controller // NFT 交易控制器
### 方法
- [createNftPendingTransaction](https://ethereum-api.rarible.org/v0.1/doc#operation/createNftPendingTransaction) // 创建 NFT 赠予异步交易
	- 接口

			https://ethereum-api.rarible.org/v0.1/nft/transactions
	- 请求
		- 方法  post
		- 参数说明

				{
				  "hash": "string",
				  "from": "0x60f80121c31a0d46b5279700f9df786054aa5ee5",
				  "nonce": 0,
				  "to": "0x60f80121c31a0d46b5279700f9df786054aa5ee5",
				  "input": "string"
				}
	- 返回
		
			{
			"transactionHash": "string",
			"status": "PENDING",
			"address": "0x60f80121c31a0d46b5279700f9df786054aa5ee5",
			"topic": "string"
			}	

## nft-lazy-mint-controller // 惰性铸币控制器
### 方法
- [mintNftAsset](https://ethereum-api.rarible.org/v0.1/doc#operation/mintNftAsset)  // NFT 惰性铸造
	- 接口

			https://ethereum-api.rarible.org/v0.1/nft/mints
	- 请求
		- 方法 post
		- 参数说明
			- 721

					{
					  "@type": "ERC721",
					  "contract": "0x60f80121c31a0d46b5279700f9df786054aa5ee5",
					  "tokenId": 717802,
					  "uri": "string",
					  "creators": [
					    {
					      "account": "0x60f80121c31a0d46b5279700f9df786054aa5ee5",
					      "value": 0
					    }
					  ],
					  "royalties": [
					    {
					      "account": "0x60f80121c31a0d46b5279700f9df786054aa5ee5",
					      "value": 0
					    }
					  ],
					  "signatures": [
					    "string"
					  ]
					}
			- 1155

					{
					  "@type": "ERC1155",
					  "supply": 717802,
					  "contract": "0x60f80121c31a0d46b5279700f9df786054aa5ee5",
					  "tokenId": 717802,
					  "uri": "string",
					  "creators": [
					    {
					      "account": "0x60f80121c31a0d46b5279700f9df786054aa5ee5",
					      "value": 0
					    }
					  ],
					  "royalties": [
					    {
					      "account": "0x60f80121c31a0d46b5279700f9df786054aa5ee5",
					      "value": 0
					    }
					  ],
					  "signatures": [
					    "string"
					  ]
					} 
		- 返回

				{
				  "id": "string",
				  "contract": "0x60f80121c31a0d46b5279700f9df786054aa5ee5",
				  "tokenId": 717802,
				  "creators": [
				    {
				      "account": "0x60f80121c31a0d46b5279700f9df786054aa5ee5",
				      "value": 0
				    }
				  ],
				  "supply": 1, // 铸造数量
				  "lazySupply": 1, // 铸造数量
				  "owners": [
				    "0x60f80121c31a0d46b5279700f9df786054aa5ee5"
				  ],
				  "royalties": [
				    {
				      "account": "0x60f80121c31a0d46b5279700f9df786054aa5ee5",
				      "value": 0
				    }
				  ],
				  "date": "2019-08-24T14:15:22Z",  //  铸造时间？
				  "pending": [
				    {
				      "type": "TRANSFER",
				      "from": "0x60f80121c31a0d46b5279700f9df786054aa5ee5"
				    }
				  ],
				  "deleted": true, // 是否可删除？
				  "meta": {  //  NFT 元信息
				    "name": "string",
				    "description": "string",
				    "attributes": [
					      {
					        "key": "string",
					        "value": "string",
					        "type": "string",
					        "format": "string"
					      }
					    ],
				    "image": {
					      "url": {
					        "property1": "string",
					        "property2": "string"
					      },
					      "meta": {
						        "property1": {
						          "type": "string",
						          "width": 0,
						          "height": 0
						        },
						        "property2": {
						          "type": "string",
						          "width": 0,
						          "height": 0
						        }
				      		}
				    },
				    "animation": {
					      "url": {
					        "property1": "string",
					        "property2": "string"
					      },
					      "meta": {
					        "property1": {
					          "type": "string",
					          "width": 0,
					          "height": 0
					        },
					        "property2": {
					          "type": "string",
					          "width": 0,
					          "height": 0
					        }
				      }
				    }
				  }
				}

## nft-activity-controller	// NFT 当前状态控制器
### 方法
- [getNftActivities](https://ethereum-api.rarible.org/v0.1/doc#operation/getNftActivities) //查询所有NFT当前状态
	- 接口

			https://ethereum-api.rarible.org/v0.1/nft/activities/search
	- 请求
		- 方法 post
		- 参数说明
			- continuation // 来自前一个响应的令牌
			- size // 搜索多少个
			- sort // 排序，从前向后，还是由后向前
		- 过滤模式 
			- NftActivityFilterAll // 过滤所有
				- @type // 总过滤类型
					- all 
				- types // NFT 当前状态
					 - "TRANSFER" //交易
					 - "MINT" // 铸造
					 - "BURN" // 销毁
			- NftActivityFilterByUser // 按用户过滤过滤
				- @type // 总过滤类型
					- by_user 
				- user // 用户列表
				- types // NFT 当前状态
					 - "TRANSFER_FROM" //从那里来
					 - "TRANSFER_TO" //去那里来
					 - "MINT" // 铸造
					 - "BURN" // 销毁
				- form //地址，默认0
				- to // 地址  ，默认0
			- NftActivityFilterByItem
				- @type // 总过滤类型
					- by_item 
				- contract // NFT 所属合约地址
				- tokenId // NFT 实例具体 ID
				- types // NFT 当前状态
					 - "TRANSFER" //交易
					 - "MINT" // 铸造
					 - "BURN" // 销毁
			- NftActivityFilterByCollection
				- @type // 总过滤类型
					- by_collection 
				- contract // NFT 所属合约地址
				- types // NFT 当前状态
					 - "TRANSFER" //交易
					 - "MINT" // 铸造
					 - "BURN" // 销毁
	- 返回

			{
			  "continuation": "string",
			  "items": [
			    {
			      "@type": "mint"
			    }
			  ]
			}

## nft-ownership-controller // NFT 所有权控制器
### 方法
- [getNftAllOwnerships](https://ethereum-api.rarible.org/v0.1/doc#operation/getNftAllOwnerships) //查询所有 NFT 所有权信息	- 接口
	
			https://ethereum-api.rarible.org/v0.1/nft/ownerships/all
	- 请求
		- 方法 get
		- 参数说明
			-  continuation
			-  size // 请求返回数量
	- 返回

			{
			  "total": 0,
			  "continuation": "string", //？
			  "ownerships": [ //所有权信息
			    {
			      "id": "string", //所有权 id
			      "contract": "0x60f80121c31a0d46b5279700f9df786054aa5ee5", //合约
			      "tokenId": 717802, //tokenid
			      "owner": "0x60f80121c31a0d46b5279700f9df786054aa5ee5", //拥有者
			      "creators": [
			        {
			          "account": "0x60f80121c31a0d46b5279700f9df786054aa5ee5", //创建者
			          "value": 0 //?
			        }
			      ],
			      "value": 717802, //tokenid
			      "lazyValue": 717802, // tokenid
			      "date": "2019-08-24T14:15:22Z", //创建时间
			      "pending": [  // 状态？
			        {
			          "type": "ROYALTY", //税率
			          "royalties": [
			            {
			              "account": "0x60f80121c31a0d46b5279700f9df786054aa5ee5",
			              "value": 0
			            }
			          ]
			        }
			      ]
			    }
			  ]
			} 	
- [getNftOwnershipById](https://ethereum-api.rarible.org/v0.1/doc#operation/getNftOwnershipById) //根据所有权 ID 查询的 NFT 所有权信息
	- 接口

			https://ethereum-api.rarible.org/v0.1/nft/ownerships/{ownershipId}
	- 请求
		- 方法 get
		- 参数说明
			- ownershipId // 所有权ID格式 `$contractAddress:$tokenId:$ownerAddress`
	- 返回

			{
			  "id": "string",
			  "contract": "0x60f80121c31a0d46b5279700f9df786054aa5ee5",
			  "tokenId": 717802,
			  "owner": "0x60f80121c31a0d46b5279700f9df786054aa5ee5",
			  "creators": [
			    {
			      "account": "0x60f80121c31a0d46b5279700f9df786054aa5ee5",
			      "value": 0
			    }
			  ],
			  "value": 717802, //tokenid
			  "lazyValue": 717802, // tokenid
			  "date": "2019-08-24T14:15:22Z", //创建时间
			  "pending": [
			    {
			      "type": "ROYALTY", //平台类型？
			      "royalties": [ // 税率
			        {
			          "account": "0x60f80121c31a0d46b5279700f9df786054aa5ee5",
			          "value": 0
			        }
			      ]
			    }
			  ]
			}
- [getNftOwnershipsByItem](https://ethereum-api.rarible.org/v0.1/doc#operation/getNftOwnershipsByItem) //根据 NFT 实例查询所有权信息
	- 接口
	
			https://ethereum-api.rarible.org/v0.1/nft/ownerships/byItem
	- 请求
		- 方法 get
		- 参数说明
			- contract //合约地址
			- tokenId // tokenid
			- continuation // 来自前一个响应的延续令牌 ？
			- size //请求数量
	- 返回
	
		同 `getNftAllOwnerships` 返回格式 

## nft-item-controller	// NFT 实例控制器
### 方法
- [getNftItemMetaById](https://ethereum-api.rarible.org/v0.1/doc#operation/getNftItemMetaById) //根据 itemId 查询 NFT 元数据
	- 接口

			https://ethereum-api.rarible.org/v0.1/nft/items/{itemId}/meta
	- 请求
		- 方法 get
		- 参数说明
			- itemId //实例ID，格式 `${contract}:${tokenId}` 
		- 例

				curl --request get \
				--url 'https://ethereum-api.rarible.org/v0.1/nft/items/0x60f80121c31a0d46b5279700f9df786054aa5ee5:717802/meta'
	- 返回

			{
			  "name": "string",  // NFT 名称
			  "description": "string", // NFT 说明
			  "attributes": [ // 属性
			    {
			      "key": "string",
			      "value": "string",
			      "type": "string",
			      "format": "string"
			    }
			  ],
			  "image": {  // 图片相关
			    "url": {
			      "property1": "string",
			      "property2": "string"
			    },
			    "meta": {  // 图片元数据
			      "property1": {  // 图片属性1
			        "type": "string",
			        "width": 0,
			        "height": 0
			      },
			      "property2": { // 图片属性2
			        "type": "string",
			        "width": 0,
			        "height": 0
			      }
			    }
			  },
			  "animation": { //动画
			    "url": {
			      "property1": "string",
			      "property2": "string"
			    },
			    "meta": { // 动画元数据
			      "property1": { //动画属性1
			        "type": "string",
			        "width": 0,
			        "height": 0
			      },
			      "property2": {//动画属性2
			        "type": "string",
			        "width": 0,
			        "height": 0
			      }
			    }
			  }
			}		
- [resetNftItemMetaById](https://ethereum-api.rarible.org/v0.1/doc#operation/resetNftItemMetaById) // 按 itemId 删除 NFT实例元数据
	- 接口

			https://ethereum-api.rarible.org/v0.1/nft/items/{itemId}/resetMeta
	- 请求
		- 方法 DELETE
		- 参数说明
			- itemId \\ 对象 ID ，格式 `${contract}:${tokenId}` 
		- 例

				curl --request DELETE \
				--url 'https://ethereum-api.rarible.org/v0.1/nft/items/0x60f80121c31a0d46b5279700f9df786054aa5ee5:717802/resetMeta'
	- 返回
		
			200 
- [getNftLazyItemById](https://ethereum-api.rarible.org/v0.1/doc#operation/getNftLazyItemById) // 根据 itemId 返回NFT惰性铸币信息
	- 接口

			https://ethereum-api.rarible.org/v0.1/nft/items/{itemId}/lazy
	- 请求
		- 方法 get
		- 参数说明
			- itemId \\ 对象 ID ，格式 `${contract}:${tokenId}` 
		- 例
		
				curl --request get \
				--url 'https://ethereum-api.rarible.org/v0.1/nft/items/0x60f80121c31a0d46b5279700f9df786054aa5ee5:717802/lazy'
	- 返回
		- 721

				{
				  "@type": "ERC721",  // 协议类型
				  "contract": "0x60f80121c31a0d46b5279700f9df786054aa5ee5",  //合约地址
				  "tokenId": 717802, // tokenid
				  "uri": "string", // 元数据 url
				  "creators": [ //创建者信息
				    {
				      "account": "0x60f80121c31a0d46b5279700f9df786054aa5ee5",
				      "value": 0
				    }
				  ],
				  "royalties": [ //税率
				    {
				      "account": "0x60f80121c31a0d46b5279700f9df786054aa5ee5",
				      "value": 0
				    }
				  ],
				  "signatures": [ //签名
				    "string"
				  ]
				}
		- 1155
		
				{
				  "@type": "ERC1155", // 协议类型
				  "supply": 717802, //供给数量，这里怀疑写错了
				  "contract": "0x60f80121c31a0d46b5279700f9df786054aa5ee5", //合约地址
				  "tokenId": 717802, // tokenid
				  "uri": "string", // 元数据 url
				  "creators": [ //创建者信息
				    {
				      "account": "0x60f80121c31a0d46b5279700f9df786054aa5ee5",
				      "value": 0
				    }
				  ],
				  "royalties": [ //税率
				    {
				      "account": "0x60f80121c31a0d46b5279700f9df786054aa5ee5",
				      "value": 0
				    }
				  ],
				  "signatures": [ //签名
				    "string"
				  ]
				}
- [deleteLazyMintNftAsset](https://ethereum-api.rarible.org/v0.1/doc#operation/deleteLazyMintNftAsset) // 删除惰性铸币但还没出售的 NFT
	- 接口

			https://ethereum-api.rarible.org/v0.1/nft/items/{itemId}/lazy/delete
	- 请求
		- 方法 post
		- 参数说明
			- itemId \\ 对象 ID ，格式 `${contract}:${tokenId}` 
			- creators \\创建者地址
			- signatures \\ 创建者数字签名
		- 例
		
				{
				  "creators": [
				    "0x60f80121c31a0d46b5279700f9df786054aa5ee5"
				  ],
				  "signatures": [
				    "string"
				  ]
				}	
		- 返回 
		
				204
- [getNftItemById](https://ethereum-api.rarible.org/v0.1/doc#operation/getNftItemById) //根据 itemId 返回 NFT 数据
	- 接口

			https://ethereum-api.rarible.org/v0.1/nft/items/{itemId}
	- 请求
		- 方法 get
		- 参数说明
			- itemId \\ 对象 ID ，格式 `${contract}:${tokenId}` 
		- 例
		
				curl --request get \
				--url 'https://ethereum-api.rarible.org/v0.1/nft/items/0x60f80121c31a0d46b5279700f9df786054aa5ee5:717802'
		- 返回

				{
				  "id": "string", \\ itemId 				 			  "contract": "0x60f80121c31a0d46b5279700f9df786054aa5ee5", \\ 合约地址
				  "tokenId": 717802, // tokenid 
				  "creators": [ //创建者
				    {
				      "account": "0x60f80121c31a0d46b5279700f9df786054aa5ee5",
				      "value": 0
				    }
				  ],
				  "supply": 717802, // 供给量
				  "lazySupply": 717802, // 惰性铸币供给量
				  "owners": [ //拥有者
				    "0x60f80121c31a0d46b5279700f9df786054aa5ee5"
				  ],
				  "royalties": [ // 税率
				    {
				      "account": "0x60f80121c31a0d46b5279700f9df786054aa5ee5",
				      "value": 0
				    }
				  ],
				  "date": "2019-08-24T14:15:22Z", // 创建时间
				  "pending": [ //交易状态？
				    {
				      "type": "TRANSFER",
				      "from": "0x60f80121c31a0d46b5279700f9df786054aa5ee5"
				    }
				  ],
				  "deleted": true, //可删除？
				  "meta": { // NFT 元数据
				    "name": "string",
				    "description": "string",
				    "attributes": [
				      {
				        "key": "string",
				        "value": "string",
				        "type": "string",
				        "format": "string"
				      }
				    ],
				    "image": {
				      "url": {
				        "property1": "string",
				        "property2": "string"
				      },
				      "meta": {
				        "property1": {
				          "type": "string",
				          "width": 0,
				          "height": 0
				        },
				        "property2": {
				          "type": "string",
				          "width": 0,
				          "height": 0
				        }
				      }
				    },
				    "animation": {
				      "url": {
				        "property1": "string",
				        "property2": "string"
				      },
				      "meta": {
				        "property1": {
				          "type": "string",
				          "width": 0,
				          "height": 0
				        },
				        "property2": {
				          "type": "string",
				          "width": 0,
				          "height": 0
				        }
				      }
				    }
				  }
				}
- [getNftItemsByOwner](https://ethereum-api.rarible.org/v0.1/doc#operation/getNftItemsByOwner) //按 NFT 所有者返回 NFT 数据
	- 接口

			https://ethereum-api.rarible.org/v0.1/nft/items/byOwner
	- 请求
		- 方法 get
		- 参数说明
			- owner \\所有者地址
			- continuation \\continuation=1631782042000_0x85d39cea74b0baba54d7fd0df42dd3e6e39b1625:0x000000000000000000000000000000000000000000000000000000000000209a
			- size \\ 请求返回数量
 
		- 例
		
				curl --request get \
				--url 'https://ethereum-api.rarible.org/v0.1/nft/items/byOwner?owner=0x60f80121c31a0d46b5279700f9df786054aa5ee5&size=10'

		- 返回

				{
				  "total": 0,
				  "continuation": "string",
				  "items": [ \\返回 NFT 列表
				    {
				      "id": "string", \\ itemid1
				      ...每个 nft item 数据等同于 getNftItemById 返回数据...
				      }
				     ]
				    }
- [getNftItemsByCreator](https://ethereum-api.rarible.org/v0.1/doc#operation/getNftItemsByCreator) // 根据创建者查询 NFT NFT实例
	- 接口

			https://ethereum-api.rarible.org/v0.1/nft/items/byCreator
	- 请求
		- 方法 get
		- 参数说明
			- creator \\创建者地址
			- continuation \\?
			- size \\ 请求返回数量
		- 例
		
				curl --request get \
				--url 'https://ethereum-api.rarible.org/v0.1/nft/items/byCreator?creator=0x60f80121c31a0d46b5279700f9df786054aa5ee5&size=10'
		- 返回

				{
				  "total": 0,
				  "continuation": "string",
				  "items": [
				    {
				      "id": "string",
				         ...每个 nft item 数据等同于 getNftItemById 返回数据...
				      }
				     ]
				    } 
- [getNftItemsByCollection](https://ethereum-api.rarible.org/v0.1/doc#operation/getNftItemsByCollection) // 按集合(收藏)返回NFT数据
	- 接口

			https://ethereum-api.rarible.org/v0.1/nft/items/byCollection
	- 请求
		- 方法 get
		- 参数说明
			- collection \\收藏集合 ，格式例子 `collection=0x60f80121c31a0d46b5279700f9df786054aa5ee5`
			- owner \\ 所有者
			- continuation \\?
			- size \\ 请求返回数量
		- 例
		
				curl --request get \
				--url 'https://ethereum-api.rarible.org/v0.1/nft/items/byCollection?collection=0x60f80121c31a0d46b5279700f9df786054aa5ee5&size=10'
		- 返回

				{
				  "total": 0,
				  "continuation": "string",
				  "items": [
				    {
				      "id": "string",
				       ...每个 nft item 数据等同于 getNftItemById 返回数据...
				    }
				  ]
				}
- [getNftAllItems](https://ethereum-api.rarible.org/v0.1/doc#operation/getNftAllItems) // 返回所有 NFT实例
	- 接口

			https://ethereum-api.rarible.org/v0.1/nft/items/all
	- 请求
		- 方法 get
		- 参数说明
			- continuation \\?
			- size \\ 请求返回数量
			- showDeleted \\ 是否包含删除项目
			- lastUpdatedFrom \\ 筛选条件，以只返回在此日期之后更新的项(unix格式) `lastUpdatedFrom=1631653245`
			- lastUpdatedTo \\ 筛选条件，以只返回在此日期之前更新的项(unix格式) `lastUpdatedTo=1631653245`
		- 例
		
				curl --request get \
				--url 'https://ethereum-api.rarible.org/v0.1/nft/items/all?size=100&lastUpdatedFrom=1631653245'
		- 返回

				{
				  "total": 0,
				  "continuation": "string",
				  "items": [
				    {
				      "id": "string",
				       ...每个 nft item 数据等同于 getNftItemById 返回数据...
				    }
				  ]
				}
- [getNftItemRoyaltyById](https://ethereum-api.rarible.org/v0.1/doc#operation/getNftItemRoyaltyById) //按 itemId 返回NFT实例税费
	- 接口

			https://ethereum-api.rarible.org/v0.1/nft/items/{itemId}/royalty
	- 请求
		- 方法 get
		- 参数说明
			- itemId \\ 对象 ID ，格式 `${contract}:${tokenId}` 
		- 例
		
				curl --request get \
				--url 'https://ethereum-api.rarible.org/v0.1/nft/items/0x60f80121c31a0d46b5279700f9df786054aa5ee5:717802/royalty'
		- 返回

				{
				  "royalty": [
				    {
				      "account": "0x60f80121c31a0d46b5279700f9df786054aa5ee5",
				      "value": 0
				    }
				  ]
				}

## nft-collection-controller \\ 收藏控制器(其他合约控制器)
### 方法
- [generateNftTokenId](https://ethereum-api.rarible.org/v0.1/doc#operation/generateNftTokenId) // 生成铸币者(minter)下一个可用的 tokenId
	- 接口

			https://ethereum-api.rarible.org/v0.1/nft/collections/{collection}/generate_token_id
	- 请求
		- 方法 get
		- 参数说明
			- collection \\ 收藏地址 ，格式 `0x60f80121c31a0d46b5279700f9df786054aa5ee5 ` 
			- minter \\ 铸币人地址，也就是合约地址
		- 例
		
				curl --request get \
				--url 'https://ethereum-api.rarible.org/v0.1/nft/collections/0x60f80121c31a0d46b5279700f9df786054aa5ee5/generate_token_id?minter=0x60f80121c31a0d46b5279700f9df786054aa5ee5'
		- 返回

				{
				  "tokenId": 717802,
				  "signature": {
				    "v": 0,
				    "r": "string",
				    "s": "string"
				  }
				}
- [getNftCollectionById](https://ethereum-api.rarible.org/v0.1/doc#operation/getNftCollectionById) // 按合约(收藏)地址返回合约信息
	- 接口

			https://ethereum-api.rarible.org/v0.1/nft/collections/{collection}
	- 请求
		- 方法 get
		- 参数说明
			- collection \\ 收藏地址 ，格式 `0x60f80121c31a0d46b5279700f9df786054aa5ee5 ` 
		- 例
		
				curl --request get \
				--url 'https://ethereum-api.rarible.org/v0.1/nft/collections/0x60f80121c31a0d46b5279700f9df786054aa5ee5'
		- 返回

				{
				  "id": "0x60f80121c31a0d46b5279700f9df786054aa5ee5", \\收藏地址
				  "type": "ERC721",  \\ 合集 NFT 类型 
				  "owner": "0x60f80121c31a0d46b5279700f9df786054aa5ee5", \\拥有者地址
				  "name": "string", \\ 收藏地址
				  "symbol": "string", \\ 代币名称
				  "features": [ \\功能，授权
				    "APPROVE_FOR_ALL"
				  ],
				  "supportsLazyMint": true, \\ 是否支持惰性铸币
				  "minters": [ \\ 铸币者
				    "0x60f80121c31a0d46b5279700f9df786054aa5ee5"
				  ]
				}
- [searchNftCollectionsByOwner](https://ethereum-api.rarible.org/v0.1/doc#operation/searchNftCollectionsByOwner) // 由所有者返回合约信息(收藏)
	- 接口

			https://ethereum-api.rarible.org/v0.1/nft/collections/byOwner
	- 请求
		- 方法 get
		- 参数说明
			- owner \\ 拥有者
			- continuation \\
			- size \\ 返回请求数量
		- 例
		
				curl --request get \
				--url 'https://ethereum-api.rarible.org/v0.1/nft/collections/byOwner?owner=0x488728a8a3e51f7c34410164c4469c71fce1084a'
		- 返回
		
				{
				  "total": 0,
				  "continuation": "string",
				  "collections": [
				    {
				      "id": "0x60f80121c31a0d46b5279700f9df786054aa5ee5",
				      "type": "ERC721",
				      ... 合约信息同 getNftCollectionById 返回结果...
				    }
				  ]
				}
- [searchNftAllCollections](https://ethereum-api.rarible.org/v0.1/doc#operation/searchNftAllCollections) // 查询返回所有合约(收藏)
	- 接口

			https://ethereum-api.rarible.org/v0.1/nft/collections/all
	- 请求
		- 方法 get
		- 参数说明
			- continuation \\
			- size \\ 返回请求数量
		- 例
		
				curl --request get \
				--url 'https://ethereum-api.rarible.org/v0.1/nft/collections/all?size=100'
		- 返回

				{
				  "total": 0,
				  "continuation": "string",
				  "collections": [
				    {
				      "id": "0x60f80121c31a0d46b5279700f9df786054aa5ee5",
				      "type": "ERC721",
				         ... 合约信息同 getNftCollectionById 返回结果...				    }
				  ]
				}

## order-signature-controller  \\ 订单签名控制器
### 方法
- [validate](https://ethereum-api.rarible.org/v0.1/doc#operation/validate) // 验证订单签名
	- 接口

			https://ethereum-api.rarible.org/v0.1/order/signature/validate
	- 请求
		- 方法 post
		- 参数说明
			- signer \\ 签名人地址
			- message \\ 被签名信息
			- signature \\ 签名认证信息  
		- 例
		
				{
				  "signer": "0x60f80121c31a0d46b5279700f9df786054aa5ee5",
				  "message": "string",
				  "signature": "string"
				}
		- 返回

				true

## order-encode-controller \\ 订单编码控制器
### 方法
- [encodeOrderData](https://ethereum-api.rarible.org/v0.1/doc#operation/encodeOrderData) // 编码订单数据
	- 接口

			https://ethereum-api.rarible.org/v0.1/order/encoder/data
	- 请求
		- 方法 post
		- 模式	 
			- OrderRaribleV2Data // Rarible 订单数据
			
					{
					  "dataType": "RARIBLE_V2_DATA_V1", \\ 订单类型，如 "RARIBLE_V2_DATA_V1"
					  "payouts": [ \\ 订单支付信息
					    {
					      "account": "0x60f80121c31a0d46b5279700f9df786054aa5ee5", \\  支付地址
					      "value": 0 \\ 支付金额 
					    }
					  ],
					  "originFees": [ \\ 订单原始税费信息
					    {
					      "account": "0x60f80121c31a0d46b5279700f9df786054aa5ee5", \\  支付地址
					      "value": 0 \\ 支付金额
					    }
					  ]
					} 		
			- OrderDataLegacy // LEGACY 订单数据，支持平台税费？
			
					{
					  "dataType": "LEGACY", \\ 订单数据类型
					  "fee": 0 \\ operation 费用金额
					}
			- OrderOpenSeaV1DataV1 //opensea v1 订单数据

					{
					  "dataType": "OPEN_SEA_V1_DATA_V1", \\ 订单数据类型
					  "exchange": "0x60f80121c31a0d46b5279700f9df786054aa5ee5", \\交换合约地址，也就是我们自己合约地址
					  "makerRelayerFee": 717802, \\ 制造商税费
					  "takerRelayerFee": 717802, \\ 购买者税费
					  "makerProtocolFee": 717802, \\ 制造商协议税费
					  "takerProtocolFee": 717802, \\ 购买者协议税费
					  "feeRecipient": "0x60f80121c31a0d46b5279700f9df786054aa5ee5", \\ 税收人
					  "feeMethod": "PROTOCOL_FEE", \\ 税收模式,支持协议税费(PROTOCOL_FEE)和分割税费(SPLIT_FEE)
					  "side": "BUY", \\ 交税方,支持买方(BUY)和卖方（SELL）
					  "saleKind": "FIXED_PRICE", \\销售方式，支持固定价格(FIXED_PRICE)和荷兰拍卖(DUTCH_AUCTION)
					  "howToCall": "CALL", \\ 调用方式，支持直接调用(CALL)和委托调用(DELEGATE_CALL)
					  "callData": "string", \\ 调用数据
					  "replacementPattern": "string", \\ 替换模式？
					  "staticTarget": "0x60f80121c31a0d46b5279700f9df786054aa5ee5", \\静态目标
					  "staticExtraData": "string", \\ 静态额外数据
					  "extra": 717802 \\ 额外数据
					}
			- OrderCryptoPunksData // 赛博朋克订单数据，也就是不支持税费

					{
					  "dataType": "CRYPTO_PUNKS_DATA" \\ 订单数据类型
					}					
		- 返回

				{
				  "type": "string",
				  "data": "string"
				}
- [encodeOrderAssetType](https://ethereum-api.rarible.org/v0.1/doc#operation/encodeOrderAssetType) // 编码订单资产类型？
	- 接口

			https://ethereum-api.rarible.org/v0.1/order/encoder/assetType
	- 请求
		- 方法 post
		- 模式
			- EthAssetType // eth 资产类型
			
					{
					  "assetClass": "ETH" \\ 资产类型
					} 
			- Erc20AssetType // erc 20 资产类型
	
					{
					  "assetClass": "ERC20", \\ 资产类型
					  "contract": "0x60f80121c31a0d46b5279700f9df786054aa5ee5" \\ 资产合约地址
					}
			- Erc721AssetType // erc 721 资产类型

					{
					  "assetClass": "ERC721", \\ 资产类型
					  "contract": "0x60f80121c31a0d46b5279700f9df786054aa5ee5", \\ 资产合约地址
					  "tokenId": 717802\\ 资产tokenid
					}
			- Erc1155AssetType // erc 1155 资产类型 

					{
					  "assetClass": "ERC1155", \\ 资产类型
					  "contract": "0x60f80121c31a0d46b5279700f9df786054aa5ee5", \\ 资产合约地址
					  "tokenId": 717802 \\ 资产tokenid
					}
			- Erc721LazyAssetType // erc 721 惰性铸币资产类型

					{
					  "assetClass": "ERC721_LAZY", \\ 资产类型
					  "contract": "0x60f80121c31a0d46b5279700f9df786054aa5ee5", \\ 资产合约地址
					  "tokenId": 717802, \\ 资产tokenid
					  "uri": "string", \\ uri 地址
					  "creators": [ \\创建者信息
					    {
					      "account": "0x60f80121c31a0d46b5279700f9df786054aa5ee5", \\ 创建者账户
					      "value": 0 \\ 税收金额
					    }
					  ],
					  "royalties": [ \\税收信息
					    {
					      "account": "0x60f80121c31a0d46b5279700f9df786054aa5ee5", \\ 收税人地址
					      "value": 0 \\ 税收金额
					    }
					  ],
					  "signatures": [ \\ 签名数据
					    "string"
					  ]
					}
			- Erc1155LazyAssetType // erc 1155 惰性铸币资产类型
				
					{
					  "assetClass": "ERC1155_LAZY", \\ 资产类型
					  "contract": "0x60f80121c31a0d46b5279700f9df786054aa5ee5", \\ 资产合约地址
					  "tokenId": 717802, \\ 资产tokenid
					  "uri": "string",  \\ uri 地址
					  "supply": 717802,  \\ 供给数量
					  "creators": [ \\创建者信息
					  ...同721部分..
					  ]
					}
			- CryptoPunksAssetType // 加密朋克资产类型

					{
					  "assetClass": "CRYPTO_PUNKS",  \\ 资产类型
					  "contract": "0x60f80121c31a0d46b5279700f9df786054aa5ee5", \\ 资产合约地址
					  "tokenId": 0 \\ 资产id
					}
	- 返回
	
			{
			  "type": "string",
			  "data": "string"
			}

## order-controller \\ 订单控制器
### 方法
- [upsertOrder](https://ethereum-api.rarible.org/v0.1/doc#operation/upsertOrder) // 创建和更新订单，支持多平台
	- 接口

			https://ethereum-api.rarible.org/v0.1/order/orders
	- 请求
		- 方法 post
		- body 参数
			- LegacyOrderForm // 从传奇统计的资产

					{
					  "type": "RARIBLE_V1", // 订单发送平台类型
					  "data": {  //资产数据类型
					    "dataType": "LEGACY",
					    "fee": 0
					  },
					  "maker": "0x60f80121c31a0d46b5279700f9df786054aa5ee5", // 创建者
					  "taker": "0x60f80121c31a0d46b5279700f9df786054aa5ee5", // 接收者
					  "make": { // 创建者信息
					    "assetType": { // 资产类型
					      "assetClass": "ETH" // 支持 EthAssetType (object) or Erc20AssetType (object) or Erc721AssetType (object) or Erc1155AssetType (object) or Erc721LazyAssetType (object) or Erc1155LazyAssetType (object) or CryptoPunksAssetType (object) or CollectionAssetType (object) or GenerativeArtAssetType (object) (AssetType)
					    },
					    "value": 717802 //持有金额
					  },
					  "take": { //接收者
					    "assetType": { // 资产类型
					      "assetClass": "ETH" // 支持 EthAssetType (object) or Erc20AssetType (object) or Erc721AssetType (object) or Erc1155AssetType (object) or Erc721LazyAssetType (object) or Erc1155LazyAssetType (object) or CryptoPunksAssetType (object) or CollectionAssetType (object) or GenerativeArtAssetType (object) (AssetType)

					    },
					    "value": 717802 //持有金额
					  },
					  "salt": 717802, //盐
					  "start": 0,
					  "end": 0,
					  "signature": "string" \\ 签名数据
					}
			- RaribleV2OrderForm // 从传奇Rarible统计的资产

					{
					  "type": "RARIBLE_V2", // 订单发送平台类型
					  "data": { //资产数据类型
					    "dataType": "RARIBLE_V2_DATA_V1",
					    "payouts": [ \\ 支出
					      {
					        "account": "0x60f80121c31a0d46b5279700f9df786054aa5ee5",
					        "value": 0
					      }
					    ],
					    "originFees": [  \\ 原始税费
					      {
					        "account": "0x60f80121c31a0d46b5279700f9df786054aa5ee5",
					        "value": 0
					      }
					    ]
					  },
					  ..... 其他部分同 LegacyOrderForm.....
					}
	- 返回
		-  LegacyOrder // 传奇订单

				{
				  "type": "RARIBLE_V1",  // 订单发送平台类型
				  "data": { //资产数据类型
				    "dataType": "LEGACY",
				    "fee": 0
				  },
				  "maker": "0x60f80121c31a0d46b5279700f9df786054aa5ee5", //创作者
				  "taker": "0x60f80121c31a0d46b5279700f9df786054aa5ee5", // 接收者
				  "make": { //创建者信息
				    "assetType": { 
				      "assetClass": "ETH" 
				    },
				    "value": 717802, // 资产持有数量
				    "valueDecimal": 717802.342336 // 资产持有精确数量
				  },
				  "take": { // 接收者信息
				    "assetType": {
				      "assetClass": "ETH"
				    },
				    "value": 717802,
				    "valueDecimal": 717802.342336
				  },
				  "fill": 717802, \\ 总量信息
				  "fillValue": 717802.342336, \\ 精确总量信息
				  "start": 0, // ？
				  "end": 0, // ？
				  "makeStock": 717802, //？
				  "makeStockValue": 717802.342336, //?
				  "cancelled": true, // 是否可取消
				  "salt": "string", // 盐
				  "signature": "string", // 签名
				  "createdAt": "2019-08-24T14:15:22Z", // 创建订单时间
				  "lastUpdateAt": "2019-08-24T14:15:22Z", // 最后一次更新时间
				  "pending": [
				    {
				      "type": "CANCEL",  // 状态类型
				      "owner": "0x60f80121c31a0d46b5279700f9df786054aa5ee5" // 拥有者
				    }
				  ],
				  "hash": "string",
				  "makeBalance": 717802,  // 创作者余额
				  "makePrice": 717802.342336, // 创作者价格
				  "takePrice": 717802.342336, //接收者价格
				  "makePriceUsd": 717802.342336,// 创建者美元价格
				  "takePriceUsd": 717802.342336, // 接收者美元价格
				  "priceHistory": [], // 价格历史
				  "status": "ACTIVE" // 订单状态
				}
		- RaribleV2Order // 默认订单

				{
				  "type": "RARIBLE_V2",
				  "data": {
				    "dataType": "RARIBLE_V2_DATA_V1",
				    "payouts": [
				      {
				        "account": "0x60f80121c31a0d46b5279700f9df786054aa5ee5",
				        "value": 0
				      }
				    ],
				    "originFees": [
				      {
				        "account": "0x60f80121c31a0d46b5279700f9df786054aa5ee5",
				        "value": 0
				      }
				    ]
				  },
				  "maker": "0x60f80121c31a0d46b5279700f9df786054aa5ee5",
				  ....其他和 LegacyOrder 返回数据一致....				}
		- OpenSeaV1Order // opensea 订单

				{
				  "type": "OPEN_SEA_V1",
				  "data": {
				    "dataType": "OPEN_SEA_V1_DATA_V1",
				    "exchange": "0x60f80121c31a0d46b5279700f9df786054aa5ee5",
				    "makerRelayerFee": 717802,
				    "takerRelayerFee": 717802,
				    "makerProtocolFee": 717802,
				    "takerProtocolFee": 717802,
				    "feeRecipient": "0x60f80121c31a0d46b5279700f9df786054aa5ee5",
				    "feeMethod": "PROTOCOL_FEE",
				    "side": "BUY",
				    "saleKind": "FIXED_PRICE",
				    "howToCall": "CALL",
				    "callData": "string",
				    "replacementPattern": "string",
				    "staticTarget": "0x60f80121c31a0d46b5279700f9df786054aa5ee5",
				    "staticExtraData": "string",
				    "extra": 717802
				  },
				  "maker": "0x60f80121c31a0d46b5279700f9df786054aa5ee5",
				 ....其他和 LegacyOrder 返回数据一致....
				}
		- CryptoPunkOrder // 加密朋克订单

				{
				  "type": "CRYPTO_PUNK",
				  "data": {
				    "dataType": "CRYPTO_PUNKS_DATA"
				  },
				  "maker": "0x60f80121c31a0d46b5279700f9df786054aa5ee5",
				   ....其他和 LegacyOrder 返回数据一致....
				}	 
- [prepareOrderCancelTransaction](https://ethereum-api.rarible.org/v0.1/doc#operation/prepareOrderCancelTransaction)  // 取消预付费订单交易
	- 接口

			https://ethereum-api.rarible.org/v0.1/order/orders/{hash}/prepareCancelTx
	- 请求
		- 参数说明
			-  hash //tx订单hash
		- 方法 post
		
				curl --request post \
				--url 'https://ethereum-api.rarible.org/v0.1/order/orders/0x38a2616dcff042fa157b2d5389171fe6f1d92c9dc3c85b7465a7e60948bf3ba0/prepareCancelTx'
	- 返回

			{
			  "to": "0x60f80121c31a0d46b5279700f9df786054aa5ee5",
			  "data": "string"
			}
- [getOrderByHash](https://ethereum-api.rarible.org/v0.1/doc#operation/getOrderByHash) // 根据TX订单 hash 返回订单数据
	- 接口

			https://ethereum-api.rarible.org/v0.1/order/orders/{hash}
	- 请求
		- 参数说明
			-  hash //tx订单hash
		- 方法 get
		
				curl --request get \
				--url 'https://ethereum-api.rarible.org/v0.1/order/orders/0x38a2616dcff042fa157b2d5389171fe6f1d92c9dc3c85b7465a7e60948bf3ba0'
	- 返回
		- LegacyOrder
		- RaribleV2Order
		- OpenSeaV1Order
		- CryptoPunkOrder
		
		以上4个返回模式同 `upsertOrder` 返回模式
- [updateOrderMakeStock](https://ethereum-api.rarible.org/v0.1/doc#operation/updateOrderMakeStock) // 根据 TX 订单 hash 更新 Stock 数据
	- 接口

			https://ethereum-api.rarible.org/v0.1/order/orders/{hash}/updateMakeStock
	- 请求
		- 参数说明
			-  hash //tx订单hash
		- 方法 get
		
				curl --request get \
				--url 'https://ethereum-api.rarible.org/v0.1/order/orders/0x38a2616dcff042fa157b2d5389171fe6f1d92c9dc3c85b7465a7e60948bf3ba0/updateMakeStock'
	- 返回
		- LegacyOrder
		- RaribleV2Order
		- OpenSeaV1Order
		- CryptoPunkOrder

		以上4个返回模式同 `upsertOrder` 返回模式
- [buyerFeeSignature](https://ethereum-api.rarible.org/v0.1/doc#operation/buyerFeeSignature) // 买方签字费用收据
	- 接口

			https://ethereum-api.rarible.org/v0.1/order/orders
	- 请求
		- 方法 post
		- body 参数
			- LegacyOrderForm

					{
					  "type": "RARIBLE_V1",
					  "data": {
					    "dataType": "LEGACY",
					    "fee": 0
					  },
					  "maker": "0x60f80121c31a0d46b5279700f9df786054aa5ee5",
					  "taker": "0x60f80121c31a0d46b5279700f9df786054aa5ee5",
					  "make": {
					    "assetType": {
					      "assetClass": "ETH"
					    },
					    "value": 717802
					  },
					  "take": {
					    "assetType": {
					      "assetClass": "ETH"
					    },
					    "value": 717802
					  },
					  "salt": 717802,
					  "start": 0,
					  "end": 0,
					  "signature": "string"
					}
			- RaribleV2OrderForm

					{
					  "type": "RARIBLE_V2",
					  "data": {
					    "dataType": "RARIBLE_V2_DATA_V1",
					    "payouts": [
					      {
					        "account": "0x60f80121c31a0d46b5279700f9df786054aa5ee5",
					        "value": 0
					      }
					    ],
					    "originFees": [
					      {
					        "account": "0x60f80121c31a0d46b5279700f9df786054aa5ee5",
					        "value": 0
					      }
					    ]
					  },
					  "maker": "0x60f80121c31a0d46b5279700f9df786054aa5ee5",
					  .. 其余信息同 LegacyOrderForm...
					}	
	- 返回

			string
- [getOrdersByIds](https://ethereum-api.rarible.org/v0.1/doc#operation/getOrdersByIds) // 根据tx订单 hash 返回所有订单
	- 接口

			https://ethereum-api.rarible.org/v0.1/order/orders/byIds
	- 请求
		- 方法 post
		- body 参数

				{
				  "ids": [ // 订单hash 列表
				    "string"
				  ]
				}
	- 返回

			  {
			    "type": "RARIBLE_V1", \\ 平台类型
			    "data": {
			      "dataType": "LEGACY", \\数据类型
			      "fee": 0 \\税费
			    }	
- [getSellOrdersByMakerAndByStatus](https://ethereum-api.rarible.org/v0.1/doc#operation/getSellOrdersByMakerAndByStatus) //根据制造商和订单状态返回所有销售订单
	- 接口
		
			https://ethereum-api.rarible.org/v0.1/order/orders/sell/byMakerAndByStatus
	- 请求
		- 方法 get
		- 参数说明
			- maker \\ 创作者地址
			- origin \\ 接收订单委托合约地址
			- platform \\创建订单的平台
				- "ALL"
				- "RARIBLE"
				- "OPEN_SEA"
				- "CRYPTO_PUNKS"
			- continuation
			- size // 返回多少
			- status //状态类型
				- "ACTIVE" \\ 活跃
				- "FILLED" \\ 填满
				- "HISTORICAL"\\历史
				- "INACTIVE" \\不活跃
				- "CANCELLED" \\ 取消   		
	- 返回

			{
			  "orders": [
			    {
			      "type": "RARIBLE_V1",
			      "data": {
			        "dataType": "LEGACY",
			        "fee": 0
			      }
			    }
			  ],
			  "continuation": "string"
			}		
- [getSellOrdersByItemAndByStatus](https://ethereum-api.rarible.org/v0.1/doc#operation/getSellOrdersByItemAndByStatus) // 根据NFT实例和订单状态返回所有在售中的订单
	- 接口
		
			https://ethereum-api.rarible.org/v0.1/order/orders/sell/byItemAndByStatus
	- 请求
		- 方法 get
		- 参数说明
			- contract
			- tokenId
			- maker
			- origin
			- platform
				- "ALL"
				- "RARIBLE"
				- "OPEN_SEA"
				- "CRYPTO_PUNKS"
			- continuation
			- size
			- status
				- "ACTIVE" \\ 活跃
				- "FILLED" \\ 填满
				- "HISTORICAL"\\历史
				- "INACTIVE" \\不活跃
				- "CANCELLED" \\ 取消   
			- currencyId		 
	- 返回

			{
			  "orders": [
			    {
			      "type": "RARIBLE_V1",
			      "data": {
			        "dataType": "LEGACY",
			        "fee": 0
			      }
			    }
			  ],
			  "continuation": "string"
			}
- [getCurrenciesBySellOrdersOfItem](https://ethereum-api.rarible.org/v0.1/doc#operation/getCurrenciesBySellOrdersOfItem)  // 从所有出售的 NFT 订单中返回作为支付的代币
	- 接口

			https://ethereum-api.rarible.org/v0.1/order/orders/currencies/bySellOrdersOfItem
	- 请求
		- 方法 get
		- 参数说明
			- contract
			- tokenId 
	- 返回

			{
			  "orderType": "SELL",
			  "currencies": [
			    {
			      "assetClass": "ETH"
			    }
			  ]
			} 
- [getCurrenciesByBidOrdersOfItem](https://ethereum-api.rarible.org/v0.1/doc#operation/getCurrenciesByBidOrdersOfItem) // 从所有出价的 NFT 订单中返回作为支付的代币
	- 接口
	
			https://ethereum-api.rarible.org/v0.1/order/orders/currencies/byBidOrdersOfItem
	- 请求
		- 方法 get
		- 参数说明
			- contract
			- tokenId 
	- 返回
	
			{
			  "orderType": "BID",
			  "currencies": [
			    {
			      "assetClass": "ETH"
			    }
			  ]
			}
- [getSellOrdersByCollectionAndByStatus](https://ethereum-api.rarible.org/v0.1/doc#operation/getSellOrdersByCollectionAndByStatus) //根据合约(收藏)和订单状态返回所有在售订单
	- 接口

			https://ethereum-api.rarible.org/v0.1/order/orders/sell/byCollectionAndByStatus
	- 请求
		- 方法 get
		- 参数说明
			- collection \\ 合约地址
			- origin  \\客户接受订单委托的合约地址
			- platform
				- "ALL"
				- "RARIBLE"
				- "OPEN_SEA"
				- "CRYPTO_PUNKS"
			- continuation
			- size
			- status
				- "ACTIVE" \\ 活跃
				- "FILLED" \\ 填满
				- "HISTORICAL"\\历史
				- "INACTIVE" \\不活跃
				- "CANCELLED" \\ 取消  
	- 返回

			{
			  "orders": [
			    {
			      "type": "RARIBLE_V1",
			      "data": {
			        "dataType": "LEGACY",
			        "fee": 0
			      }
			    }
			  ],
			  "continuation": "string"
			}
- [getSellOrdersByStatus](https://ethereum-api.rarible.org/v0.1/doc#operation/getSellOrdersByStatus) // 根据订单状态返回所有在售订单
	- 接口

			https://ethereum-api.rarible.org/v0.1/order/orders/sellByStatus
	- 请求
		- 方法 get
		- 参数说明
			- origin  \\客户接受订单委托的合约地址
			- platform
				- "ALL"
				- "RARIBLE"
				- "OPEN_SEA"
				- "CRYPTO_PUNKS"
			- continuation
			- size
			- status
				- "ACTIVE" \\ 活跃
				- "FILLED" \\ 填满
				- "HISTORICAL"\\历史
				- "INACTIVE" \\不活跃
				- "CANCELLED" \\ 取消 
			- sort
				- "LAST_UPDATE_ASC"
				- "LAST_UPDATE_DESC"  
	- 返回
	
		返回同 getSellOrdersByCollectionAndByStatus
- [getOrderBidsByMakerAndByStatus](https://ethereum-api.rarible.org/v0.1/doc#operation/getOrderBidsByMakerAndByStatus) // 根据创建者和订单状态，返回所有出价订单
	- 接口

			https://ethereum-api.rarible.org/v0.1/order/orders/sellByStatus
	- 请求
		- 方法 get
		- 参数说明
			- maker  \\订单创建者地址
			- origin \\ 客户接受订单委托的地址
			- platform
				- "ALL"
				- "RARIBLE"
				- "OPEN_SEA"
				- "CRYPTO_PUNKS"
			- continuation
			- size
			- status
				- "ACTIVE" \\ 活跃
				- "FILLED" \\ 填满
				- "HISTORICAL"\\历史
				- "INACTIVE" \\不活跃
				- "CANCELLED" \\ 取消 
			- startDate \\ 开始计算的数据的时间边界(unix格式)
			- endDate \\	结束计算的数据的时间边界(unix格式)
	- 返回

		返回同 getSellOrdersByCollectionAndByStatus
- [getOrderBidsByItemAndByStatus](https://ethereum-api.rarible.org/v0.1/doc#operation/getOrderBidsByItemAndByStatus) // 根据NFT实例和订单状态返回所有出价订单
	- 接口

			https://ethereum-api.rarible.org/v0.1/order/orders/bids/byItemAndByStatus
	- 请求
		- 方法 get
		- 参数说明
			-  contract \\ 合约地址
			-  tokenid \\ tokenid
			-  maker \\ 合约创建者地址
			-  origin \\ 客户接受订单委托的地址
			- platform
				- "ALL"
				- "RARIBLE"
				- "OPEN_SEA"
				- "CRYPTO_PUNKS"
			- continuation
			- size
			- status
				- "ACTIVE" \\ 活跃
				- "FILLED" \\ 填满
				- "HISTORICAL"\\历史
				- "INACTIVE" \\不活跃
				- "CANCELLED" \\ 取消 
			- currencyId \\ 当前ID？
			- startDate \\ 开始计算的数据的时间边界(unix格式)
			- endDate \\	结束计算的数据的时间边界(unix格式)
	- 返回

		返回同 getSellOrdersByCollectionAndByStatus
- [getOrdersAllByStatus](https://ethereum-api.rarible.org/v0.1/doc#operation/getOrdersAllByStatus) // 所有订单根据状态排序
	- 接口

			https://ethereum-api.rarible.org/v0.1/order/orders/all/byStatus
	- 请求
		- 方法 get
		- 参数说明
			- sort \\排序
				- LAST_UPDATE_ASC
				- LAST_UPDATE_DESC
			- continuation
			- size
			- status
				- "ACTIVE" \\ 活跃
				- "FILLED" \\ 填满
				- "HISTORICAL"\\历史
				- "INACTIVE" \\不活跃
				- "CANCELLED" \\ 取消     
	- 返回

			{
			  "orders": [
			    {
			      "type": "RARIBLE_V1",
			      "data": {
			        "dataType": "LEGACY",
			        "fee": 0
			      }
			    }
			  ],
			  "continuation": "string"
			}
			
## order-transaction-controller // 订单交易控制器
### 方法
- [createOrderPendingTransaction](https://ethereum-api.rarible.org/v0.1/doc#operation/createOrderPendingTransaction) //创建暂挂事务的订单
	-  接口

			https://ethereum-api.rarible.org/v0.1/order/transactions
	- 请求
		- 方法 post
		- body 参数

				{
				  "hash": "string",
				  "from": "0x60f80121c31a0d46b5279700f9df786054aa5ee5",
				  "nonce": 0,
				  "to": "0x60f80121c31a0d46b5279700f9df786054aa5ee5",
				  "input": "string"
				}
	- 返回

			[
			  {
			    "transactionHash": "string",
			    "status": "PENDING",
			    "address": "0x60f80121c31a0d46b5279700f9df786054aa5ee5",
			    "topic": "string"
			  }
			]	

## order-activity-controller //订单状态控制器
### 方法
- [getOrderActivities](https://ethereum-api.rarible.org/v0.1/doc#operation/getOrderActivities) //根据订单返回历史事件
	-  接口

			https://ethereum-api.rarible.org/v0.1/order/activities/search
	- 请求
		- 方法 post
		- 参数说明
			- continuation
			- size
			- sort
				-  LATEST_FIRST
				-  EARLIEST_FIRST
		- 过滤模式
			- OrderActivityFilterAll  // 全部
	
					{
					  "@type": "all", // 过滤方法
					  "types": [ // 根据订单状态过滤 ，包括 "BID" "LIST" "MATCH" "CANCEL_BID" "CANCEL_LIST"
					    "BID"
					  ]
					}
			- OrderActivityFilterByUser // 根据用户过滤

					{
					  "@type": "by_user", // 过滤方法
					  "users": [ //用户列表
					    "0x60f80121c31a0d46b5279700f9df786054aa5ee5"
					  ],
					  "types": [ // 根据订单状态过滤 ，包括 "MAKE_BID" "GET_BID" "LIST" "BUY" "SELL" "CANCEL_BID" "CANCEL_LIST"
					    "MAKE_BID"
					  ],
					  "from": 0, //?
					  "to": 0 //?
					}
			-  OrderActivityFilterByItem  // 根据 NFT 实例过滤

					{
					  "@type": "by_item", // 过滤方法
					  "contract": "0x60f80121c31a0d46b5279700f9df786054aa5ee5",
					  "tokenId": 717802,
					  "types": [ // 根据订单状态过滤 ，包括 "BID" "LIST" "MATCH" "CANCEL_BID" "CANCEL_LIST"
					    "BID"
					  ]
					}
			- OrderActivityFilterByCollection //根据合约收藏过滤
	
					{
					  "@type": "by_collection", // 过滤方法
					  "contract": "0x60f80121c31a0d46b5279700f9df786054aa5ee5",
					  "types": [// 根据订单状态过滤 ，包括 "BID" "LIST" "MATCH" "CANCEL_BID" "CANCEL_LIST"

					    "BID"
					  ]
					}							
	- 返回

			{
			  "continuation": "string",
			  "items": [
			    {
			      "@type": "match",
			      "left": { //左订单
			        "maker": "0x60f80121c31a0d46b5279700f9df786054aa5ee5",
			        "hash": "string",
			        "asset": {
			          "assetType": {
			            "assetClass": "ETH"
			          },
			          "value": 717802,
			          "valueDecimal": 717802.342336
			        },
			        "type": "SELL"
			      },
			      "right": { //右订单
			        "maker": "0x60f80121c31a0d46b5279700f9df786054aa5ee5",
			        "hash": "string",
			        "asset": {
			          "assetType": {
			            "assetClass": "ETH"
			          },
			          "value": 717802,
			          "valueDecimal": 717802.342336
			        },
			        "type": "SELL"
			      },
			      "price": 717802.342336,
			      "priceUsd": 717802.342336,
			      "transactionHash": "string",
			      "blockHash": "string",
			      "blockNumber": 0,
			      "logIndex": 0,
			      "type": "SELL"
			    }
			  ]
			}

## order-aggregation-controller \\订单聚合控制器
### 方法
- [aggregateNftSellByMaker](https://ethereum-api.rarible.org/v0.1/doc#operation/aggregateNftSellByMaker) //按制造商返回 NFT 卖出订单聚合
	- 接口

			https://ethereum-api.rarible.org/v0.1/order/aggregations/nftSellByMaker
	- 请求
		- 方法 get
		- 参数说明
			- startDate \\ 开始搜索数据边界时间
			- endDate \\ 结束搜索数据边界时间
			- size \\ 打印数量
			- source \\ 订单来源
				- "ALL"
				- "RARIBLE"
				- "OPEN_SEA"
				- "CRYPTO_PUNKS" 
	- 返回

			[
			  {
			    "address": "0x60f80121c31a0d46b5279700f9df786054aa5ee5",  // 地址
			    "sum": 717802.342336, // 聚合结果
			    "count": 0 // 出售数量
			  }
			]						 
- [aggregateNftPurchaseByTaker](https://ethereum-api.rarible.org/v0.1/doc#operation/aggregateNftPurchaseByTaker) //根据接收方返回 NFT 购买订单聚合
	-  接口

			https://ethereum-api.rarible.org/v0.1/order/aggregations/nftPurchaseByTaker
	- 请求
		- 方法 get
		- 参数说明
			- startDate \\ 开始搜索数据边界时间
			- endDate \\ 结束搜索数据边界时间
			- size \\ 打印数量
			- source \\ 订单来源
				- "ALL"
				- "RARIBLE"
				- "OPEN_SEA"
				- "CRYPTO_PUNKS" 
	- 返回

			[
			  {
			    "address": "0x60f80121c31a0d46b5279700f9df786054aa5ee5", \\ 地址
			    "sum": 717802.342336, \\ 聚合结果
			    "count": 0 \\ 数量
			  }
			]
- [aggregateNftPurchaseByCollection](https://ethereum-api.rarible.org/v0.1/doc#operation/aggregateNftPurchaseByCollection) //根据合约(收藏)返回 NFT 购买订单聚合
	-  接口

			https://ethereum-api.rarible.org/v0.1/order/aggregations/nftPurchaseByCollection
	- 请求
		- 方法 get
		- 参数说明
			- startDate \\ 开始搜索数据边界时间
			- endDate \\ 结束搜索数据边界时间
			- size \\ 打印数量
			- source \\ 订单来源
				- "ALL"
				- "RARIBLE"
				- "OPEN_SEA"
				- "CRYPTO_PUNKS" 
	- 返回

			  {
			    "address": "0x60f80121c31a0d46b5279700f9df786054aa5ee5", \\ 地址
			    "sum": 717802.342336, \\ 聚合结果
			    "count": 0
			  }
			  
## auction-controller \\ 拍卖控制器
### 方法
- [getAuctionByHash](https://ethereum-api.rarible.org/v0.1/doc#operation/getAuctionByHash) //根据TX 交易 hash 返回拍卖信息
	-  接口

			https://ethereum-api.rarible.org/v0.1/order/auctions/{hash}
	- 请求
		- 方法 get
		- 参数说明
			- hash \\交易hash  
		- 例

				curl --request get \
				--url 'https://ethereum-api.rarible.org/v0.1/auction/auctions/0x38a2616dcff042fa157b2d5389171fe6f1d92c9dc3c85b7465a7e60948bf3ba0'
	- 返回

				{
				  "type": "RARIBLE_AUCTION_V1", \\ 类型
				  "auctionId": 717802, \\ 拍卖 id
				  "lastBid": { \\ 最后一次出价
				    "type": "RARIBLE_AUCTION_V1_BID_V1", \\ 出价接口类型
				    "data": {
				      "dataType": "RARIBLE_AUCTION_V1_BID_DATA_V1", \\数据类型
				      "originFees": [ \\原始税率
				        {
				          "account": "0x60f80121c31a0d46b5279700f9df786054aa5ee5",
				          "value": 0
				        }
				      ],
				      "payouts": [ \\支出
				        {
				          "account": "0x60f80121c31a0d46b5279700f9df786054aa5ee5",
				          "value": 0
				        }
				      ]
				    }
				  },
				  "data": { \\??
				    "dataType": "RARIBLE_AUCTION_V1_DATA_V1", \\数据类型
				    "originFees": [ \\原始税率
				      {
				        "account": "0x60f80121c31a0d46b5279700f9df786054aa5ee5",
				        "value": 0
				      }
				    ],
				    "payouts": [ \\支出
				      {
				        "account": "0x60f80121c31a0d46b5279700f9df786054aa5ee5",
				        "value": 0
				      }
				    ],
				    "startTime": "2019-08-24T14:15:22Z", \\ 拍卖开始时间
				    "duration": 717802, \\ 持续时间
				    "buyOutPrice": 717802.342336 // 收购价格
				  },
				  "seller": "0x60f80121c31a0d46b5279700f9df786054aa5ee5", \\ 出售方地址
				  "sell": { 
				    "assetType": {
				      "assetClass": "ETH" \\ 出售方认可购买代币
				    },
				    "value": 717802, \\ 金额
				    "valueDecimal": 717802.342336 \\ 精确金额
				  },
				  "buy": {
				    "assetClass": "ETH" \\ 收购方认可购买代币
				  },
				  "endTime": "2019-08-24T14:15:22Z", \\ 结束时间
				  "minimalStep": 717802.342336,  \\ 最小一次出价单位
				  "minimalPrice": 717802.342336, \\ 最小价格
				  "createdAt": "2019-08-24T14:15:22Z", \\ 创建时间
				  "lastUpdateAt": "2019-08-24T14:15:22Z", \\ 最后更新时间
				  "buyPrice": 717802.342336, \\ 购买价格
				  "buyPriceUsd": 717802.342336, \\ 购买美金价格
				  "pending": [
				    {
				      "hash": "string" \\交易hash？
				    }
				  ],
				  "status": "ACTIVE", \\订单状态
				  "hash": "string" \交易hash？
				}
- [getAuctionBidsByHash](https://ethereum-api.rarible.org/v0.1/doc#operation/getAuctionBidsByHash) // 根据拍卖 hash 返回出价订单
	- 接口

			https://ethereum-api.rarible.org/v0.1/order/auctions/{hash}/bids
	- 请求
		- 方法 get
		- 参数说明
			- hash \\拍卖交易 hash 
			- continuation
			- size
		- 例

				curl --request get \
				--url 'https://ethereum-api.rarible.org/v0.1/auction/auctions/0x38a2616dcff042fa157b2d5389171fe6f1d92c9dc3c85b7465a7e60948bf3ba0/bids'
	- 返回	

			{
			  "bids": [ \\出价信息
			    {
			      "type": "RARIBLE_AUCTION_V1_BID_V1", \\ 订单类型
			      "data": {
			        "dataType": "RARIBLE_AUCTION_V1_BID_DATA_V1", \\ 数据类型
			        "originFees": [ \\初始税率
			          {
			            "account": "0x60f80121c31a0d46b5279700f9df786054aa5ee5",
			            "value": 0
			          }
			        ],
			        "payouts": [ \\ 出售额
			          {
			            "account": "0x60f80121c31a0d46b5279700f9df786054aa5ee5",
			            "value": 0
			          }
			        ]
			      }
			    }
			  ],
			  "continuation": "string"
			}
- [getAuctionsByIds](https://ethereum-api.rarible.org/v0.1/doc#operation/getAuctionsByIds) // 根据拍卖 hash 返回所有拍卖
	- 接口

			https://ethereum-api.rarible.org/v0.1/order/auctions/byIds
	- 请求
		- 方法 post
		- 参数说明

				{
				  "ids": [ //拍卖 hash 列表
				    "string"
				  ]
				}
	- 返回

			auctions[
			  {
			    "type": "RARIBLE_AUCTION_V1", // 拍卖类型
			    "auctionId": 717802, // 拍卖订单id
			    "lastBid": { // 最后出价
			      "type": "RARIBLE_AUCTION_V1_BID_V1",
			      "data": {
			        "dataType": "RARIBLE_AUCTION_V1_BID_DATA_V1",
			        "originFees": [
			          {
			            "account": "0x60f80121c31a0d46b5279700f9df786054aa5ee5",
			            "value": 0
			          }
			        ],
			        "payouts": [
			          {
			            "account": "0x60f80121c31a0d46b5279700f9df786054aa5ee5",
			            "value": 0
			          }
			        ]
			      }
			    },
			    "data": { // 售价信息？
			      "dataType": "RARIBLE_AUCTION_V1_DATA_V1",
			      "originFees": [
			        {
			          "account": "0x60f80121c31a0d46b5279700f9df786054aa5ee5",
			          "value": 0
			        }
			      ],
			      "payouts": [
			        {
			          "account": "0x60f80121c31a0d46b5279700f9df786054aa5ee5",
			          "value": 0
			        }
			      ],
			      "startTime": "2019-08-24T14:15:22Z", // 开始时间
			      "duration": 717802, //持续时间
			      "buyOutPrice": 717802.342336 //直卖价格？
			    }
			  }
			]
- [getAuctionsBySeller](https://ethereum-api.rarible.org/v0.1/doc#operation/getAuctionsBySeller) // 根据卖家返回所有拍卖订单
	-  接口

			https://ethereum-api.rarible.org/v0.1/order/auctions/bySeller
	- 请求
		- 方法 get
		- 参数说明
			- seller
			- status
				- "ACTIVE"
				- "CANCELLED"
				- "FINISHED"
			- origin
			- platform
				- "ALL"
				- "RARIBLE"
				- "OPEN_SEA"
				- "CRYPTO_PUNKS" 
			- continuation
			- size 
		- 返回

			返回同 getAuctionsByIds
- [getAuctionsByItem](https://ethereum-api.rarible.org/v0.1/doc#operation/getAuctionsBySeller) // 根据NFT实例返回所有拍卖
	-  接口

			https://ethereum-api.rarible.org/v0.1/order/auctions/byItem
	- 请求
		- 方法 get
		- 参数说明
			- contract
			- tokenId
			- seller
			- sort
				- LAST_UPDATE_ASC
				- LAST_UPDATE_DESC
				- BUY_PRICE_ASC 
			- origin
			- status
				- "ACTIVE"
				- "CANCELLED"
				- "FINISHED"
			- currencyId
			- platform
				- "ALL"
				- "RARIBLE"
				- "OPEN_SEA"
				- "CRYPTO_PUNKS" 
			- continuation
			- size
	- 返回

		返回同 getAuctionsByIds
- [getAuctionsByCollection](https://ethereum-api.rarible.org/v0.1/doc#operation/getAuctionsByCollection)// 根据合约(收藏)返回所有拍卖
	- 接口

			https://ethereum-api.rarible.org/v0.1/order/auctions/byCollection
	- 请求
		- 方法 get
		- 参数说明
			- contract \\ 收藏合约
			- seller
			- origin
			- status
				- "ACTIVE"
				- "CANCELLED"
				- "FINISHED"
			- platform
				- "ALL"
				- "RARIBLE"
				- "OPEN_SEA"
				- "CRYPTO_PUNKS" 
			- continuation
			- size
	- 返回

		返回同 getAuctionsByIds

## lock-controller // 锁控制器
### 方法
- [createLock](https://ethereum-api.rarible.org/v0.1/doc#operation/createLock) // 创建锁
	- 接口

			https://ethereum-api.rarible.org/v0.1/unlockable/item/{itemId}/lock
	- 请求
		- 方法 post
		- 参数说明
			- itemId \\格式 `${contract}:${tokenId}`
			- signature
			- content
		- 例

				{
				  "signature": "string",
				  "content": "string"
				}
	- 返回

			{
			  "id": "string", \\ 锁id
			  "itemId": "string", \\ itemid
			  "content": "string", \\ 内容
			  "author": "0x60f80121c31a0d46b5279700f9df786054aa5ee5", \\ 作者
			  "signature": "string", \\签名
			  "unlockDate": "2019-08-24T14:15:22Z", \\ 解锁时间
			  "version": 0 \\ 版本
			 }
- [getLockContent](https://ethereum-api.rarible.org/v0.1/doc#operation/getLockContent) //检查 NFT 实体锁信息
	- 接口

			https://ethereum-api.rarible.org/v0.1/unlockable/item/{itemId}/content
	- 请求
		- 方法 post
		- 参数说明
			- itemId \\格式 `${contract}:${tokenId}`
			- signature
		- 返回  
	
				{
				  "signature": "string"
				}
	- 返回

			string
- [isUnlockable](https://ethereum-api.rarible.org/v0.1/doc#operation/isUnlockable) // 解锁
	- 接口

			https://ethereum-api.rarible.org/v0.1/unlockable/item/{itemId}/isUnlockable
	- 请求
		- 方法 get
		- 参数说明
			- itemId \\格式 `${contract}:${tokenId}`
	- 返回

			true

## 注销接口
### nft-order-collection-controller // NFT 收藏集订单合控制器
- generateNftOrderTokenId // 根据铸币者返回下一个可用 tokenid
- getNftOrderCollectionById // 根据地址返回收藏集合
- searchNftOrderCollectionsByOwner //根据拥有者返回收藏集合
- searchNftOrderAllCollections // 返回所有收藏集合

### nft-order-activity-controller // NFT 活跃订单控制器
- getNftOrderActivitiesByUser // 根据用户返回订单历史事件
- getNftOrderActivitiesByItem // 根据 NFT 实例返回订单历史事件
- getNftOrderActivitiesByCollection // 根据收藏集合返回订单历史事件
- getNftOrderAllActivities // 返回所有订单的历史事件

### nft-order-item-controller // NFT 订单实体控制器
- getNftOrderItemById // 根据 NFT 实体标识返回订单实体
- getNftOrderItemMetaById // 根据 NFT 实体标识返回订单实体元数据
- getNftOrderLazyItemById // 根据 NFT 实体标识返回惰性铸币订单实体
- getNftOrderItemsByOwner // 根据拥有者返回订单实体
- getNftOrderItemsByCreator //根据创建者返回订单实体
- getNftOrderItemsByCollection // 根据收藏集合返回订单实体
- getNftOrderAllItems // 返回所有订单 

### nft-order-lazy-mint-controller //NFT 订单惰性铸币控制器
- mintNftOrderAsset // 惰性铸币(NFT)

### nft-order-ownership-controller // NFT 订单所有权控制器
- getNftOrderOwnershipById // 根据所有权 ID 返回订单所有权和相关数据 // 所有权 ID 格式 `$contractAddress:$tokenId:$ownerAddress`
- getNftOrderOwnershipsByItem //  根据 NFT 实例具体信息返回订单所有权和相关数据
- getNftOrderAllOwnerships // 返回具有所有权的所有订单

### auction-controller //拍卖控制器
- getAuctionsAll // 按照指定参数返回拍卖所有订单

### order-bid-controller // 出价订单控制器
- getBidsByItem // 根据NFT 实体返回出价订单

### order-controller // 订单控制器
- prepareOrderTransaction // 预订单交易
- getOrdersAll // 根据特殊参数返回所有订单
- 在售订单
	- getSellOrdersByMaker // 根据订单创建者返回所有在售订单
	- getSellOrdersByItem // 根据 NFT 实体返回所有在售订单
	- getSellOrdersByCollection // 根据收藏集合返回所有在售订单
	- getSellOrders // 根据特殊参数返回所有在售订单
- 出价订单
	- getOrderBidsByMaker // 根据常见者返回所有出价订单
	- getOrderBidsByItem // 根据 NFT 实体返回所有出价订单

### order-encode-controller // 订单编码控制器
- encodeOrder // 编码订单？给其他平台用？





		

				 


				

 