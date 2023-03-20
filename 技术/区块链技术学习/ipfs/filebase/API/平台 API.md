# 平台 API
了解如何使用 Filebase Platform API。
## 文件库平台 API
Filebase Platform API 可用于监控和了解用户的数据存储、最近的带宽使用情况以及最近的 IPFS 网关带宽使用情况。
### 总存储量和最近带宽（过去 24 小时）使用 API
以下 API 可用于生成有关所有存储桶中用户当前存储（以字节为单位）的报告，而不管存储桶的网络如何。此 API 还返回有关用户在过去 24 小时内的带宽使用情况的信息。
#### 授权
	Authorization: Bearer <access-token>
- 要生成 `access-token`
	- 首先检索您 [访问密钥和秘密访问密钥对](https://docs.filebase.com/getting-started/getting-started-guides/getting-started-guide#access-keys)。
	- 然后，导航 [base64encode](https://www.base64encode.org/) 到并输入以下信息：
	
			ACCESS-KEY:SECRET-KEY
	- 然后选择“编码”并复制结果

		![](./pic/GatewayAPI.png)
		
#### 有效载荷
请求正文架构： application/json

- 列出所有存储桶中的当前存储以及过去 24 小时内的所有带宽使用情况 `Get https://api.filebase.io/v1/usage`

	此 API 用于生成报告，提供有关用户当前所在的存储桶中的存储以及过去 24小事内所有带宽使用情况。
	
	- 参数

		无
	- 响应
		- 200

				{
				    "storage": {
				        "bytes": 4977622511155
				    },
				    "bandwidth": {
				        "bytes": 0
				    }
				}

### 特定存储桶 API 的总存储量
#### 授权
	Authorization: Bearer <access-token>
- 要生成 `access-token`
	- 首先检索您 [访问密钥和秘密访问密钥对](https://docs.filebase.com/getting-started/getting-started-guides/getting-started-guide#access-keys)。
	- 然后，导航 [base64encode](https://www.base64encode.org/) 到并输入以下信息：
	
			ACCESS-KEY:SECRET-KEY
	- 然后选择“编码”并复制结果

		![](./pic/GatewayAPI.png)
		
#### 有效载荷
请求正文架构： application/json

- 列出特定存储桶的当前存储使用情况（以字节为单位）`GET https://api.filebase.io/v1/usage/storage/<bucket-name>`。

	提供用户当前特定的桶的存储信息
	
	- path 参数

		参数名|类型|说名
		---|---|---
		bucket-name*|string|一个桶名字
	- 响应
		- 200
		
				{
				    "storage": {
				        "bytes": 1251576869
				    }
				}		
### IPFS 专用网关 API 的带宽使用
#### 授权
	Authorization: Bearer <access-token>
- 要生成 `access-token`
	- 首先检索您 [访问密钥和秘密访问密钥对](https://docs.filebase.com/getting-started/getting-started-guides/getting-started-guide#access-keys)。
	- 然后，导航 [base64encode](https://www.base64encode.org/) 到并输入以下信息：
	
			ACCESS-KEY:SECRET-KEY
	- 然后选择“编码”并复制结果

		![](./pic/GatewayAPI.png)
		
#### 有效载荷
请求正文架构： application/json

- 列出特定 IPFS 专用网关在过去 24 小时内的带宽使用情况。`GET https://api.filebase.io/v1/usage/gateway/<gateway-name>`

	列出特定 IPFS 专用网关在过去 24 小时内的带宽使用情况
	
	- path 参数

		参数名|类型|说名
		---|---|---
		gateway-name*|string|一个 IPFS 网关名称
	- 响应
		- 200

				{
				    "gateway": {
				        "bytes": 620424
				    }
				}