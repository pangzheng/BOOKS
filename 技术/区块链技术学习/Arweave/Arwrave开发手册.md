# 黑客马拉松信息表
Arweave 网络本身是一个全球性的永久硬盘驱动器，而 permaweb 是一个建立在 Arweave 之上的分散的、不可变的网络。
Permaweb 应用程序使用普通的 Web 技术（HTML、CSS 和 Javascript）构建，但部署到 Arweave 的链上存储系统，使其永久化并以快速、分散的方式可用。 使用 Arweave Deploy，您可以在 2 分钟或更短的时间内部署一个 permaweb 应用程序！
## HTTP API
Arweave HTTP API 概述。
### 介绍
Arweave 协议基于 HTTP，因此任何现有的 http 客户端/库都可用于与网络接口，例如 JavaScript、PHP 等。默认端口为 1984。

请求和查询可以直接使用其 IP 地址发送到任何 Arweave 节点，例如 .[http://159.65.213.43:1984/info](http://159.65.213.43:1984/info)

如果配置了 DNS，也可以使用主机名，例如 .

例子请求

- 请求类型

		cURL
	- 语句
	
			curl --request GET \
			  --url 'https://arweave.net/info'
	
- 请求类型

		JS
	- 语句
	
			fetch('https://arweave.net/info')
			.then(response => response.json())
			.then(data => {
			  console.log('Arweave network height is: ' + data.height);
			})
			.catch(error => {
			  console.error(error);
			});
- 请求类型

		NodeJS
	- 语句

			let request = require("request");
			
			let options = {
			 method: 'GET',
			 url: 'https://arweave.net/info'
			};
			
			request(options, function (error, response, body) { 
			  if (error){
			   console.error(error); 
			  }
			  console.log('Arweave network height is: ' + JSON.parse(body).height);
			});
			
### 集成
Arweave 特定的包装器和客户端目前正在开发中，以简化常见操作和 API 交互，目前集成了 Go、PHP、Scala（也可以与 Java 和 C# 一起使用）和 [JavaScript/TypeScript/NodeJS](https://github.com/ArweaveTeam/arweave-js)。

## 架构
解释了常见的数据结构、格式和流程
### 块格式					
- 高度小于 269510

		{
		    "nonce": "AAEBAAABAQAAAQAAAQEBAAEAAAABAQABAQABAAEAAAEBAAAAAQAAAAAAAQAAAQEBAAEBAAEBAQEBAQEAAQEBAAABAQEAAQAAAQABAAABAAAAAAEBAQEBAAABAQEAAAAAAAABAQAAAQAAAQEAAQABAQABAQEAAAABAAABAQABAQEAAAEBAQABAQEBAQEBAAABAQEAAAABAQABAAABAAEAAQEBAQAAAAABAQABAQAAAAAAAAABAQABAAEBAAEAAQABAQABAAEBAQEBAAEAAQABAAABAQEBAQAAAQABAQEBAAEBAQAAAQEBAQABAAEBAQEBAAAAAAABAAEAAAEAAAEAAAEBAAAAAAEAAQABAAAAAAABAQABAQAAAAEBAQAAAAABAAABAAEBAQEAAAAAAQAAAQABAQABAAEAAQABAQAAAAEBAQAAAQAAAAEBAAEBAAEBAQEAAAEBAQAAAQAAAAABAAEAAQEAAQ",
		    "previous_block": "V6YjG8G3he0JIIwRtzTccX39rS0jH-jOqUJy6rxrVAHY0RT0AVhG8K22wCDxy1A0",
		    "timestamp": 1528500720,
		    "last_retarget": 1528500720,
		    "diff": 31,
		    "height": 100,
		    "hash": "AAAAANsEvzGbICpfAj3NN41_ox--2cNxkEhAo0aggpDPkY7zru29g24uMWUP9hTa",
		    "indep_hash": "",
		    "txs": [
		        "7BoxcxiJIjTwUp3JXp0xRJQXf6hZtyJj1kjGNiEl5A8"
		    ],
		    "wallet_list": "ph2FDDuQjNbca34tz7vP9X5Xve2EGJi2ZgFqhMITAdw",
		    "reward_addr": "em8MfGRInwWEAQnE6b50ENaFOf-0to4Pbygng1ilWGQ",
		    "tags": [],
		    "reward_pool": 60770606104,
		    "weave_size": 599058,
		    "block_size": 0
		}
- 高度大于 269510  小于 422250

		{
		    "nonce": "O3IQWXYmxLN_b0w7QyT2GTruaVIGsl-Ybhc6Pl2V20U",
		    "previous_block": "VRVYubqppWUVAeCWlzHR-38dQoWcFAKbGculkVZThfj-hNMX4QVZjqkC6-PkiNGE",
		    "timestamp": 1567052949,
		    "last_retarget": 1567052114,
		    "diff": "115792088374597902074750511579343425068641803109251942518159264612597601665024",
		    "height": 269512,
		    "hash": "____47liyh_OZdYUP4EzBoLl7JOPge9VsWPQ3b5kiU8",
		    "indep_hash": "5H-hJycMS_PnPOpobXu2CNobRlgqmw4yEMQSc5LeBfS7We63l8HjS-Ek3QaxK8ug",
		    "txs": ["tqDWYT-qdoCeSWGpV2Ig48lpswOxccbBpyxf0GQjs2U", "y0bIjxLaXu1gEjpRlyPUh0Uz0c5XrhIOs6z4lerXo8w"],
		    "wallet_list": "6haahtRP5WVchxPbqtLCqDsFWidhebYJpU5PVB4zQhE",
		    "reward_addr": "aE1AjkBoXBfF-PRP2dzRrbYY8cY2OYzeH551nSPRU5M",
		    "tags": [],
		    "reward_pool": 0,
		    "weave_size": 21080508475,
		    "block_size": 991723,
		    "cumulative_diff": "616416144",
		    "hash_list_merkle": "1QVbbLwZHpNMJd8ZghRb13HZfrRu-aIIfzY29r64_yBJAcYv-Kfblv_c2pfKbQBP"
		}
- 高度大于 422250

		{
		    "nonce": "W3Jy4wp2LVbDFhGX_hUjRQZCkTdEbKxz45E5OVe52Lo",
		    "previous_block": "YuTyalVBTNB9t5KhuRezcIgxVz9PbQsbrcY4Tpkiu8XBPgglGM_Yql5qZd0c9PVG",
		    "timestamp": 1586440919,
		    "last_retarget": 1586440919,
		    "diff": "115792089039110416381168389782714091630053560834545856346499935466490404274176",
		    "height": 422250,
		    "hash": "_____8422fLZnBsEsxtwEdpi8GZDHVT-aFlqroQDG44",
		    "indep_hash":"5VTARz7bwDO4GqviCSI9JXm8_JOtoQwF-QCZm0Gt2gVgwdzSY3brOtOD46bjMz09",
		    "txs":["IRPCjc_ws7aS5GWp4mwR2k-HuQy-zT_GWrgR6kRdbmI"],
		    "tx_root": "lsoo-p3Tj7oblZ-54WVPHoVguqgw5rA9Jf3lLH6H8zY",
		    "tx_tree":[],
		    "wallet_list":"N5NJtXhgH9bPmXoSopehcr_zqwyPjjg3igel0V8G1DdLk_BYdoRVIBsqjVA9JmFc","reward_addr":"Oox7m4HIcVhUtMd6AUuGtlaOoSCmREUNPyyKQCbz4d4",
		    "tags":[],
		    "reward_pool":3026104059201252,
		    "weave_size": 407672420044,
		    "block_size": 937455,
		    "cumulative_diff": "99416580392277",
		    "hash_list_merkle": "akSjDrBKPuepJMOhO_S9C-iFp5zn9Glv57HGdN_WPqEToWC0Ukb37Gzs4PDA7oLU",
		    "poa": {
		        "option":"1",
		        "tx_path": "xZ6vhVXw_0BlD-Xkv3KtfnJeLXykjkjUrwcPsXw2JUnie021At7I-fMZkt5EF_xOHtcdq4RIqXto1gwFAM5eZgAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAfDSbuKpWzKZ9HP_N2I4gX6cUujNsJtelJULjHmbZp0XzmkBljlK4S1PMlSrTePIjfJdRfqvFNE8idpnj69X1P0zAfwAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAn4ybxD6lgdArqnPJzs7t8bU-7KfEb1YqpAOvbr6q3vmP-MWnCTWZJKTL90azeYZmHrTMx-iutuT6bP6CUC7zgHAfGgAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAmTpFIGvz18gKl5rZ6p2Ve4yVeRzWNwibyVTKz80HSBYprfIpVJk9oRG3E5q1xRn5wErqyH2vFLbsLxDqKcR0vLunBwAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAfDwBRWXT_vDxcaBxGmihJwlU_n_PFBCOsP-Lx3hSG6H6UGesIMAEYMmd2c5QixR-fCimhm_9S582cLzSUffsrAHliQAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAmP-RTrBhY9xCC1yywyehB7X6EmlBjyQBqm0y1L9Ex_dkswkf50rG-LE29UJP4st0bzFthHukfHvvWZY3bgIiog3L7gAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAfD3YxQguhfH8daMBAQrveQq3MMp4iKB3khk5mbU34Ckl1q8AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAJj_kQ",
		        "data_path": "bTVpffiN3SSDeqBEJpKiXegQGKKnprS_AFMh6zz4QRIU-8dJuvFzyKxqjkDHQvtKl0Eajfm18yZsjaAJkNhbAwAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAEAAAOH0cuoLq1CTbSelF9C59C-fcO3a3ywoceaNxRl4nQQH1BuwcpiNdDdZvEz6Pfk5wKbnsF_VwVIgrfcLZgsxoKwAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAACAAAefOoaNyW7ORmrzbZ5O7midzLByHooxjM5oEMJfZbQsY9mKS14G9fUEFmFaCPPJX6EXVGrUwROzDIWfHf8oHErAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAADAAAktmxYyC7BSV-MULrjzgdJJYfJY7lDFcKe3mo_EX19xoAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAABAAAA",
		        "chunk":"....$忽略...    }
		}
		
### 交易格式
节点和网关通过 `POST /tx` 端点接受交易。 正文必须是交易的 JSON 编码。 [winstons](https://docs.arweave.org/developers/server/http-api#ar-and-winston)为单位。

当通过 HTTP 发送或放入 URL 或显示在块浏览器中时，交易和块标识符和钱包地址以及其他一些字段被编码为 Base64URL 字符串。
	
- Base64URL 区分大小写，例子
		
		 T414mkfW-EQWEwPtk__LMJAgawNdxZfdjxhGPQKMwDQ, t414mkfW-EQWEwPtk__LMJAgawNdxZfdjxhGPQKMwDQ, andt414mkfw-eqwewptk__lmjagawndxzfdjxhgpqkmwdq are three different addresses. It is impossible to recover tokens sent to a different case of the same address.		
交易可用于上传数据、传输代币或两者兼而有之。

有关完整[示例](https://docs.arweave.org/developers/server/http-api#sample-transactions)，请参阅下面的示例交易。

### 字段定义
名称|必要|序列化格式|值
---|---|---|---
format|是|integer|当前支持的格式是 1 和 2（通常分别称为 v1 和 v2）。 v1 格式已弃用
id|是|Base64URL string| 1个[交易签名](https://docs.arweave.org/developers/server/http-api#transaction-signing)的SHA-256哈希。
last_tx|是|Base64URL string|锚点 - 防止重放攻击。 它可以是最后 50 个区块之一的哈希值，也可以是发送钱包的最后一个传出交易 ID。 如果这是来自钱包的第一笔交易，则可以使用空字符串。 推荐的方法是使用 `GET /tx_anchor` 返回的值。 如果使用块哈希，则两个不同的交易可以具有相同的 `last_tx`。
owner|是|Base64URL string|发送钱包的完整 RSA 模值。 模数是来自 `JWK` 的 n 值。 RSA 公钥。
tags|否|list|名称-值对的列表，每对被序列化为 `{"name": "a BaseURL string", "value":" a Base64URL string"}` 如果没有使用标签，则使用空数组`[]`。 名称和值的总大小不得超过 2048 字节。 标签可能有助于将消息附加到发送到另一个钱包的交易中，例如参考号或标识符以帮助说明交易。
target|否|Base64URL string|将代币发送到的目标地址（如果需要）。 如果没有代币被转移到另一个钱包，则使用空字符串。 请注意，不支持将代笔发送到所有者地址。 该地址是 RSA 公钥的 SHA-256 哈希。
quantity|否| [一个 winston 字符串](https://docs.arweave.org/developers/server/http-api#ar-and-winston)|从所有者钱包转移到目标钱包地址的金额（如果需要）
data_root|否|Base64URL string|仅用于 v2 事务。 交易数据的默克尔根。 如果没有数据，则使用空字符串
data_size|否|string|仅用于 v2 事务。 交易数据的字节大小。 如果没有数据，请使用“0”。 数字的字符串表示形式不得超过 21 个字节。
data|否|Base64URL string|要提交的数据。 如果没有提交数据，则使用空字符串。 对于 v2 事务，即使有数据（意味着 data_size > 0 且 data_root 不为空），也无需使用此字段，尽管可以。 在 v1 事务中，数据不能大于 10 MiB。 在 v2 交易中，限制由节点决定。 在撰写本文时，网络中的所有节点都通过该字段接受最多 12 MiB 的数据。
reward|是|一个 winston 字符串|交易费。 有关更多信息，请参阅[价格端点文档](https://docs.arweave.org/developers/server/http-api#get-transaction-price)。
signature|是|Base64URL string|交易字段的 SHA-384 散列的默克尔根的 RSA 签名（id 除外，它是签名的散列）。 有关更多信息，请参阅[交易签名](https://docs.arweave.org/developers/server/http-api#transaction-signing)。

- 交易例子
	- 数据交易

			{
			  "format": 2,
			  "id": "BNttzDav3jHVnNiV7nYbQv-GY0HQ-4XXsdkE5K9ylHQ",
			  "last_tx": "jUcuEDZQy2fC6T3fHnGfYsw0D0Zl4NfuaXfwBOLiQtA",
			  "owner": "posmE...psEok",
			  "tags": [],
			  "target": "",
			  "quantity": "0",
			  "data_root": "PGh0b...RtbD4",
			  "data": "",
			  "data_size": "1234235",
			  "reward": "124145681682",
			  "signature": "HZRG_...jRGB-M"
			} 
	- A R 钱包交易

			{
			  "format": 2,
			  "id": "UEVFNJVXSu7GodYbZoldRHGi_tjzNtNcYjeSkxKCpiE",
			  "last_tx": "knQ5gf4Z_3i-NQ6_jFT2a9zShUOHh4lDZoAUzsWMxMQ",
			  "owner": "1nPKv...LjJMc",
			  "tags": [{"name": "BBBB", "value": "AAAA"}],
			  "target": "WxLW1MWiSWcuwxmvzokahENCbWurzvwcsukFTGrqwdw",
			  "quantity": "46598403314697200",
			  "data": "",
			  "data_root": "",
			  "data_size": "0",
			  "reward": "321179212",
			  "signature": "OYIJU...j9Mxc"
			}
	- AR 和数据交易

			{
			  "format": 2,
			  "id": "3pXpj43Tk8QzDAoERjHE3ED7oEKLKephjnVakvkiHF8",
			  "last_tx": "NpeIbi93igKhE5lKUMhH5dFmyEsNGC0fb2Qysggd-kM",
			  "owner": "posmE...psEok",
			  "tags": [],
			  "target": "pEbU_SLfRzEseum0_hMB1Ie-hqvpeHWypRhZiPoioDI",
			  "quantity": "10000000000",
			  "data_root": "PGh0b...RtbD4",
			  "data_size": "234234",
			  "data": "VGVzdA",
			  "reward": "321579659",
			  "signature": "fjL0N...f2UMk"
			}

### 交易签名
交易签名是通过计算交易字段的 SHA-384 哈希的默克尔根生成的：`format, owner, target, data_root, data_size, quantity, reward, last_tx, tags,`，然后对哈希进行签名。

签名是使用 `SHA-256` 作为散列函数的 `RSA-PSS`。
#### 密钥格式
Arweave 使用带有 4096 长度的 RSA-PSS 密钥的 JSON Web 密钥 (JWK) 格式 (RFC 7517)。 此 JWK 格式允许将加密密钥表示为 JSON 对象，其中每个属性都表示底层加密密钥的一个属性。

它受到大多数流行语言的库的广泛支持。 可以将 JWK 转换为 PEM 文件或其他加密密钥文件格式，对此的支持会因语言而异。

如果您手动生成自己的密钥，则`公共指数 (e) 必须为 65537`。如果使用任何其他值，则由这些密钥签名的交易将无效并被拒绝。
#### 地址
n 值是公共模数，用作交易[所有者字段](https://docs.arweave.org/developers/server/http-api#transaction-format)，钱包地址是来自 JWK 的 `n` 值的 Base64URL 编码的 SHA-256 哈希。	
#### 简单的 JWK
这个钱包的地址是` GRQ7swQO1AMyFgnuAPI7AvGQlW3lzuQuwlJbIpWV7xk`

	{
	    "kty": "RSA",
	    "e": "AQAB",
	    "n": "ovFF6EbOtXeg7VnojIgtChgxfU6GZ16JjVj5JFHh6NGHJnq4p059BnMphcDx1mqb3yxM73FxhEszSFLcJiPzway6eIDiXuYiT-Sf_0Wl6_wDLvEmlz43psp7WYJumwpaSyiI_1FWmOVQnTnoAIKaOYKVqzUlteiECQj7XjJl0MZH16RlEfVqVpJ_8Ier4_QXIJ8Y3pe2KF3Lg9UANFU97nuvEM94CSzX-0WIju6Lykt3DBb2YtFFg4bJjOFv3T38nCZmDh8lYjm25_1qILalsB0XRoDxQy9FLxWb4zd09JsDhL0EYAQ_hNfOnQFVOBtYEHVYMCHYH6GoTcNgxmUkZPk4AfpAqZmjDzKfVJrw4Fr68pPTEQOQEzBcIWp61P21BSkhqO4QuFinkQsSH6NdTB_3FpbhYf34Hjf-iH7hdpdWo4aoRLb8eZeZcqBRZoRmlhQnOD-PVxQR_vb9rjXSjGkCWwRbsurVLWdBh_FQn0S9Q6EHqiV8nbW-R0Rk2E76JwgMFkqGUtZj8DeEqXJ2jlAvuzp56fXeAViPEtvUj1HheO8O3LxdVYCiapWWKq4qQVoRzdiyvydYSmbztgFUhekvmjNkxLNKOh71i3hFtoXycegqZ6izrUGoF2oD24lsTKsV5lV5pwfmUjVvxtHZm54bJIMfUDYbOV6yeDjYBb8",
	    "d": "EePSrJeFn4f0a8rozPEwnMCeQmdKO3Q2PwYrSJES8Ch9IbzspDXqZThksTJHeya2WXD4O3vlnkRRa5npYOimnTeVO6DO-eNjlgkAhhsEBh5jzRYeChIDMzVdCK1Y7n3a_xCCxiGMk_nteW2_qrqsKy9KtoL90nSmdoV9b9CxvBPhFGyQykF7POkV0fdbaIpGtcayCNJ4ZgMyUpWi0ZwgUhxTUtGsmLlLN2Phg-vt_jZ96h5lS-E1NCUq4ORpj018fDp9DwTdamTyz5LTwaa8F1OCWDPVCW7Ztjs1o-NVXHvejYbhQZeFz9SP804PqLrb1ubDWXmFzKdHns4aRH4bWivh9L8HwSJUl5UEXprJUpYilT0tb3VauI7Cih2LBfhU3fUIDJFYm_j9etgNcPlqt64T7_TI8elgj7-sciXa1XEqIje9Mn8spxT6lpn4nhxJ9qelERCJwiWbuPnW2VsJHeqXZTly52KQEP_UBC4z8a0tDm7HIQw7WQ-OAuNUOu8ongOHaOexkqKYIcF3f812sOIVEJufoBXUUTIvJk-buH0ytgtTjkrO64zZeIvFHa1MFU-6UXh8jipSZ617znNR2Pc1-l3s7pACdbXvy2-5VWE3psRr1L5HM4KNwm6Rs5BXXqBSifzfiJ5qNGqKabfXvPXI8wYyl3mhUQtHW6sUUl0",
	    "p": "0q_DP_FzSi8JEd-NNXoIaeL5MOxmNiXmDHGNxP3noKPyr-N6h3CrK5G59Rj2vWAJMhKToz1eSQ1p0-X0Ku2DvdT5LQOGIXVPtojw0OcOI8G8SoqMGAGehaLsnV3vexwtwjLfIM99XccKAxWMA1SMuL48nuBpMUhO0MlagbrL5vfpKB9kL7XCQqspAnN_vBmQZGWYczQmBgfC6v6xGQV3xHJmL--dn-qF2XU9pKuqd0J-cKYcdLPrccdJtGLid4nrSOTDfEbr77IUI5VGWV8CFJ-n8Vki-GwUxUkJpIoRyp5DxnYtSJb7cV-xOf7kBTCEUFn5B8fb2q-d8011cgnp5Q",
	    "q": "xfzB-Yf4fa2y2q4ubJCJA5H-IG9-mr7fVRTUbj-gTqVL-I7MCDIImdAPbA-3EoIR5H70GVbAFGQJyYDq6eDeTbNs1zfnU0JPurASE3fKbOpoRdLwXwaSdRJRP9qnqUe-BzuloIzWc-dI-6TJxmHUSA1X9CtHvIdfNdKPCVFKUMrb1bv5arAI8tRbNRfy3tnbiw4wfKhYEQ1e6RPpxAR5F4We9RJ81-sIlfAy7WfliwmcGmgcPNdUinGR299CiVYKf5ktoqGFQ9n6K-v4gNZV23f33-tuD8pMVxyc3xG34j4frH57bsbm7v8Qz-92ZxHWzOUgxIVhGgSaa4E51d9m0w",
	    "dp": "yArepo4I230BbZkHKKlv56n81PkAq5UccuA2rb4u-ZXxThP9OTA_NiUtnYxQassOsB53U91m8pHr06hZR5ExL0NSO-1Go-oQ_83SaWeZQ1YmA9i83-ZZr6VcaKbSReAhimxm825PKIVd-kOxJ1BWNOtb_7Yv6v0u6IrmhproE6t8E_6KT8qSYl7Fl3A27lCPiuPz9h6jo8Imzp15ZbqNV1cPs6Ad18MDx8_L8diVCJt4FlmCV0Sl3uhMERx6zumDHzkma4-jYXmCKa8Ilr7g6NgWy8_Ipnto1VFd-H6oGexficaXhH7my2UCj4B23H6OgwSKsVqQY3mvzV3Uj6zeCQ",
	    "dq": "a0_ey6OZWnWFleYHH60PtrGw7l_AXZvLbVBG_CLcfwQ1M1oi2OZVpxkQ4t95uTxq-lCdegZ9QhAfBessaOwLUk5IVjbk2Un98RByG784JuS-8-mrg7YKOA5fn56idax_IWiBE46Cxnu8ITlmbHKmHw-sdpnm3hb50jB4evJmt3fcw_KI8_zKPORBM3vxljy7NJnSSh7s7QE0Sl0Svb427Drut6L3rAimtK5mzCseTcg9pkp707ZbClcYWfafF9VdB2A9TgMCOo6xfJEANsT18GkMH4B6PXDHBAhsNrRh2O0XOeWsfZStoyj5Mdt3b9JJfPFMW3h38yQ_lrmKYZQfJQ",
	    "qi": "aDsPYxE-JBYsYhCYXSU7WsCrnFxNsRpFMcYXdmdryYIdQUpeemChDGzVJXLnJhE4cAS9TtLcNg82xZSKZvHrnkbFpRfSJxzEnvIXW4V0LHkxkxbmM0e9B7UrpYm6LKtvEY6I7L8wHFpHdOwV6NjY925oULEV156X0r55V7N0XF-jy3rbm71DCWRh6IDRghhCZQ3aNgJxE-OtnABqasaY6CQnTDRXLkGE0kq9GCx85-92fQLHMzvrMhr9m_2MHYJ_gZehL4j95CQzhD3Zh602D0YYYwRSsU4h5HGjlmN52pe-rfTLgwCJq5295s7qUP8TTMzbZAOM_hehksHpAaFghA"
	}
#### AR和 winston
Winston 是 AR 的最小可能单位，类似于比特币中的 satoshi 或以太坊中的 wei。

	1 AR = 1000000000000 winston（12 个零）和 1 winston = 0.000000000001 AR。
HTTP API 将返回所有金额作为 winston 字符串，这是为了在不支持任意精度算术的环境之间实现轻松的互操作性。

例如，JavaScript 将所有数字存储为双精度浮点值，因此无法原生表达 winston 的整数。 将这些值作为字符串提供允许它们直接加载到大多数“bignum”库中。

## 连接 API
### 交易
用于与事务和相关资源交互的端点。
#### 交易信息 GET
- 通过交易 ID 查询
	- 请求
		
			https://arweave.net/tx/{id}
		
		数量和奖励值始终表示为 winston 字符串。 [什么和为什么？](https://docs.arweave.org/developers/server/http-api#winston-and-ar)
	- 响应
		- 200 

				{
				  "format": 2,
				  "id": "BNttzDav3jHVnNiV7nYbQv-GY0HQ-4XXsdkE5K9ylHQ",
				  "last_tx": "jUcuEDZQy2fC6T3fHnGfYsw0D0Zl4NfuaXfwBOLiQtA",
				  "owner": "posmE...psEok",
				  "tags": [],
				  "target": "",
				  "quantity": "0",
				  "data_root": "PGh0b...RtbD4",
				  "data_size": "123",
				  "reward": "124145681682",
				  "signature": "HZRG_...jRGB-M"
				}
		- 202

				pending
		- 400
		- 404
		
		有关[交易结构](https://docs.arweave.org/developers/server/http-api#transaction-format)和内容的详细信息，请参阅章节，并附有示例。
- 通过交易ID 查询交易状态
	- 请求
	
			https://arweave.net/tx/{id}/status
	- 响应
		- 200

				{
				  "block_height": 641606,
				  "block_indep_hash": "akLaom7XAKYvIW7HPCtCqSCgYTGAa0zjer6FXvF8lX0pAPzcHMZj4XnQq0jaedT6",
				  "number_of_confirmations": 12
				} 
		- 404
- 通过交易 ID 和字段从交易中获取单个字段
	- 请求

			https://arweave.net/tx/{id}/{field}	
	- 200

			jUcuEDZQy2fC6T3fHnGfYsw0D0Zl4NfuaXfwBOLiQtA
	- 202

			Pending
	- 400
	- 404
- 从交易中获取解码数据。

	`Content-Type` 将默认为 `Content-Type` 标记中指定的类型。
	
	- 请求

			https://arweave.net/{id}
- 您还可以通过执行以下操作获取具有不同 `Content-Type` 响应的数据：
	- 请求

			http://arweave.net:1984/tx/{id}/data.{extension}

交易ID

可以根据客户端用例指定任何扩展。 可以使用 `data.html` 请求网页

	<html lang="en-GB" class="b-pw-1280 no-touch orb-js id-svg bbcdotcom ads-enabled bbcdotcom-init bbcdotcom-responsive bbcdotcom-async bbcdotcom-ads-enabled orb-more-loaded  bbcdotcom-group-5" id="responsive-news">
	<meta charset="utf-8">
	<meta name="viewport" content="width=device-width, initial-scale=1, user-scalable=1">
	...
可以通过交易提交 `Content-Type` 标签，然后在提供数据响应时，该标签将用作 `Content-Type` 标头，这允许您提交像图像这样的二进制文件，并通过 HTTP 为它们提供正确的内容类型标头 .

默认的 `Content-Type` 是 `text/html`。


- 交易查询其他结果
	- Pending

		交易处理中
	- Invalid hash.

		提供的交易 ID 无效或字段名称无效。
	- Not Found

		找不到具有给定 ID 的交易。

- 获取交易价格

	此端点用于计算特定大小的交易的最低费用（奖励），并可能针对特定地址。此端点应始终用于计算尽可能接近提交时间的交易费用。 定价是动态的，由网络决定，因此并非总是可以离线或提前准确计算价格。`费用过低的交易将被拒绝`。

		https://arweave.net/price/{bytes}/{target} 	
	- 200

发送到新钱包地址的第一笔交易需要支付额外费用。 这是故意的，可以阻止钱包垃圾邮件。

- 例子
	- 要获得向地址为 abc 的钱包发送 10 个 AR 的费用，请咨询 `/price/0/abc`。
	- 要上传 `123` 字节而不传输代币，请咨询 `/price/123`。
	- 发送一些AR到“abc”钱包并上传`123`字节数据查询`/price/123/abc`

#### 交易信息 POST
- 提交一个交易

	向网络提交新交易。请求正文应为 JSON 对象，具有交易格式中描述的属性。

	- 请求		
		
			https://arweave.net/tx
	- 响应
		- 200
			
				ok
		- 208

				Transaction already processed.	交易已处理
		- 400
		- 429
		- 503

在[交易格式](https://docs.arweave.org/developers/server/http-api#transaction-format)部分中找到有关这些字段和示例的更多信息。

### 钱包地址查询 GET
用于获取钱包信息的端点。

- 获取余额

	获取给定钱包的余额。 未知的钱包地址将简单地返回 0
	
	- 请求
		
			https://arweave.net/wallet/{address}/balance
	- 响应
		- 200

				20000000
		- 400
	
	注意 ：钱包余额始终表示为 winston 字符串。 [什么和为什么？](https://docs.arweave.org/developers/server/http-api#winston-and-ar)
- 获取最后一笔交易的 ID

	获取给定钱包地址的最后一笔传出交易。
	
	- 请求

			https://arweave.net/wallet/{address}/last_tx
	- 响应
		- 200

				7SRpf0dWDqN4hbnCMPkdg02u_tzyMBtqwjDBy3EU9dg
		- 400

### 块查询 GET
用于获取块和块数据的端点。

- 通过块 ID 获取块信息
	
	通过其哈希（idep_hash）获取一个块。
	
	- 请求	
	
			https://arweave.net/block/hash/{block:hash}
	- 响应
		- 200 

				{
				    "nonce": "W3Jy4wp2LVbDFhGX_hUjRQZCkTdEbKxz45E5OVe52Lo",
				    "previous_block": "YuTyalVBTNB9t5KhuRezcIgxVz9PbQsbrcY4Tpkiu8XBPgglGM_Yql5qZd0c9PVG",
				    "timestamp": 1586440919,
				    "last_retarget": 1586440919,
				    "diff": "115792089039110416381168389782714091630053560834545856346499935466490404274176",
				    "height": 422250,
				    "hash": "_____8422fLZnBsEsxtwEdpi8GZDHVT-aFlqroQDG44","indep_hash":"5VTARz7bwDO4GqviCSI9JXm8_JOtoQwF-QCZm0Gt2gVgwdzSY3brOtOD46bjMz09","txs":["IRPCjc_ws7aS5GWp4mwR2k-HuQy-zT_GWrgR6kRdbmI"],
				    "tx_root": "lsoo-p3Tj7oblZ-54WVPHoVguqgw5rA9Jf3lLH6H8zY",
				    "tx_tree":[],
				    "wallet_list":"N5NJtXhgH9bPmXoSopehcr_zqwyPjjg3igel0V8G1DdLk_BYdoRVIBsqjVA9JmFc","reward_addr":"Oox7m4HIcVhUtMd6AUuGtlaOoSCmREUNPyyKQCbz4d4",
				    "tags":[],
				    "reward_pool":3026104059201252,
				    "weave_size": 407672420044,
				    "block_size": 937455,
				    "cumulative_diff": "99416580392277",
				    "hash_list_merkle": "akSjDrBKPuepJMOhO_S9C-iFp5zn9Glv57HGdN_WPqEToWC0Ukb37Gzs4PDA7oLU",
				    "poa": {
				        "option":"1",
				        "tx_path": "xZ6vhVXw_0BlD-Xkv3KtfnJeLXykjkjUrwcPsXw2JUnie021At7I-fMZkt5EF_xOHtcdq4RIqXto1gwFAM5eZgAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAfDSbuKpWzKZ9HP_N2I4gX6cUujNsJtelJULjHmbZp0XzmkBljlK4S1PMlSrTePIjfJdRfqvFNE8idpnj69X1P0zAfwAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAn4ybxD6lgdArqnPJzs7t8bU-7KfEb1YqpAOvbr6q3vmP-MWnCTWZJKTL90azeYZmHrTMx-iutuT6bP6CUC7zgHAfGgAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAmTpFIGvz18gKl5rZ6p2Ve4yVeRzWNwibyVTKz80HSBYprfIpVJk9oRG3E5q1xRn5wErqyH2vFLbsLxDqKcR0vLunBwAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAfDwBRWXT_vDxcaBxGmihJwlU_n_PFBCOsP-Lx3hSG6H6UGesIMAEYMmd2c5QixR-fCimhm_9S582cLzSUffsrAHliQAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAmP-RTrBhY9xCC1yywyehB7X6EmlBjyQBqm0y1L9Ex_dkswkf50rG-LE29UJP4st0bzFthHukfHvvWZY3bgIiog3L7gAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAfD3YxQguhfH8daMBAQrveQq3MMp4iKB3khk5mbU34Ckl1q8AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAJj_kQ",
				        "data_path": "bTVpffiN3SSDeqBEJpKiXegQGKKnprS_AFMh6zz4QRIU-8dJuvFzyKxqjkDHQvtKl0Eajfm18yZsjaAJkNhbAwAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAEAAAOH0cuoLq1CTbSelF9C59C-fcO3a3ywoceaNxRl4nQQH1BuwcpiNdDdZvEz6Pfk5wKbnsF_VwVIgrfcLZgsxoKwAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAACAAAefOoaNyW7ORmrzbZ5O7midzLByHooxjM5oEMJfZbQsY9mKS14G9fUEFmFaCPPJX6EXVGrUwROzDIWfHf8oHErAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAADAAAktmxYyC7BSV-MULrjzgdJJYfJY7lDFcKe3mo_EX19xoAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAABAAAA",
				        "chunk":"-省略-
				    }
				}
		- 400
	
### 网络和节点状态 GET
用于获取有关当前网络和节点状态信息的端点。

- 网路信息

	获取当前网络信息，包括高度、当前区块和其他属性。
	
	- 请求
	
			https://arweave.net/info
	- 响应

			{
			  "network": "arweave.N.1",
			  "version": 5,
			  "release": 43,
			  "height": 551511,
			  "current": "XIDpYbc3b5iuiqclSl_Hrx263Sd4zzmrNja1cvFlqNWUGuyymhhGZYI4WMsID1K3",
			  "blocks": 97375,
			  "peers": 64,
			  "queue_length": 0,
			  "node_state_latency": 18
			}
- peer 列表

	从节点获取对等点列表。 节点只能与他们当前知道的对等节点进行响应，因此这不会是网络上节点的详尽或完整列表。
	
	- 请求

			https://arweave.net/peers			
	- 响应
		- 200

				[
				  "127.0.0.1:1984",
				  "0.0.0.0:1984"
				]

## chunk POST
- 上传 chunk 数据
	- 请求

			https://arweave.net/chunk
	- 响应
		- 200
		- 400

			不同类型的 400 响应
			
				//当 Chuck 大于 256 KiB 时。
				{"error": "chunk_too_big"}
				
				or
				
				// 当证明大于 256 KiB 时。
				{"error": "data_path_too_big"}
				
				or
				
				// 当偏移量大于 2 ^ 256 时。
				{"error": "offset_too_big"}
				
				or
				
				// 当数据大小大于 2 ^ 256 时。
				{"error": "data_size_too_big"}
				
				or
				
				/*
				 当 data_path 大于 chuck 时。
				注意：如果原始数据太小，不宜分块上传。
				此外，这不适用于作为其交易的唯一块的块和到每笔交易的最后一块。
				*/
				
				{"error": "chunk_proof_ratio_not_attractive"}
				
				or
				
				// 当节点还没有看到相应交易的头部时.
				{"error": "data_root_not_found"}
				
				or
				
				/*
				 相应的交易处于待处理状态，它是以下之一：
				
				- 该节点已经为此接受了 50 MiB 的块（数据根、数据大小）；
				- 该节点已接受价值 2 GiB 的所有待处理块。
				
				注意：以上数值为默认值，任何节点都可以配置更大的限制。
				
				*/
				{"error": "exceeds_disk_pool_size_limit"}
				
				or
				
				// 交易签名错误
				{"error": "invalid_proof"}
				
				// 当节点尚未加入网络时。
				{"error": "not_joined"}
				
				or
				
				// 交易超时
				{"error": "timeout"}
				
				{
				  "data_root": "<Base64URL 编码数据默克尔根>",
				  "data_size": "a number, 交易的大小（以字节为单位）",
				  "data_path": "<Base64URL 编码包含证明>",
				  "chunk": "<Base64URL",编码数据块>
				  "offset": "<[start_offset, start_offset + 块大小）中的一个数字，相对于其他块>"
				}
				
			注意：除了数据根之外，还需要 data_size，因为可能会提交具有不同交易大小的相同数据根。 为了避免块重叠，数据根总是与大小一起出现。

## Chunks 下载
- 通过交易 ID 下载数据			
	
	无论数据如何上传，端点都会提供数据
	
	- 请求

			https://arweave.net/tx/{id}/data
	- 响应
		- 200

				<Base64URL encoded data>
		- 400
		- 503
- 通过交易 ID 下载获取偏移量和大小

	获取交易的绝对结束偏移量和大小

	注意：客户端可以使用此信息来收集交易块。 从结束偏移开始并通过 `G0ET /chunk/<offset>` 获取一个块。 从事务大小中减去其大小 - 如果要获取更多块，则从偏移量中减去块的大小并获取下一个块。 
	
	- 请求

			https://arweave.net /tx/{id}/offset
	- 响应
		- 200

				{"offset": "...", "size": "..."}
		- 400
		- 503 
			
							 
			 
			
			
					
	

					
	
				 
		

	
		
	
					



						