# 基于 EVM 的链
源数据位于 _data/chains 中。每个链都有自己的文件，文件名是 [CAIP-2](https://github.com/ChainAgnostic/CAIPs/blob/master/CAIPs/caip-2.md) 表示的名称和 `.json` 扩展名。
## 例子
	{
	  "name": "Ethereum Mainnet",
	  "chain": "ETH",
	  "network": "mainnet",
	  "rpc": [
	    "https://mainnet.infura.io/v3/${INFURA_API_KEY}",
	    "https://api.mycryptoapi.com/eth"
	  ],
	  "faucets": [],
	  "nativeCurrency": {
	    "name": "Ether",
	    "symbol": "ETH",
	    "decimals": 18
	  },
	  "infoURL": "https://ethereum.org",
	  "shortName": "eth",
	  "chainId": 1,
	  "networkId": 1,
	  "icon": "ethereum",
	  "explorers": [{
	    "name": "etherscan",
	    "url": "https://etherscan.io",
	    "icon": "etherscan",
	    "standard": "EIP3091"
	  }]
	}
当在网络或资源管理器中使用图标时，_data/icons 中必须有一个带有所用名称的 json（例如，在上面的示例中，其中必须有 `ethereum.json` 和 `etherscan.json`） - 图标 json 如下所示

	[
	    {
	      "url": "ipfs://QmdwQDr6vmBtXmK2TmknkEuZNoaDqTasFdZdu3DRw8b2wt",
	      "width": 1000,
	      "height": 1628,
	      "format": "png"
	    }
	]

- URL 必须是可公开解析的 IPFS url
- 宽度和高度选填参数，但是当一个存在时，另一个也必须存在
- 格式为“png”、“jpg”或“svg”

如果链是 L2 或另一个链的分片，您可以将其链接到父链，如下所示

	{
	  ...
	  "parent": {
	   "type" : "L2",
	   "chain": "eip155-1",
	   "bridges": [ {"url":"https://bridge.arbitrum.io"} ]
	  }
	}
需要在其中指定类型 2 和对现有父级的引用。关于桥的字段是可选的。

## 聚合
还有自动组装所有链的聚合 json 文件

- https://chainid.network/chains.json
- https://chainid.network/chains_mini.json（小型化 - 较小文件大小的字段较少）

## 碰撞管理
如果不同的链具有相同的链 ID，我们会列出具有最旧起源的链
## 使用者
- chainlist.org or networklist-org.vercel.app as a staging version with a more up-to-date list
- chainid.network
- WallETH
- TREZOR
- networks.vercel.app
- eth-chains
- EVM-BOX
- FaucETH
- Sourcify playground
- chaindirectory.xyz
- chain-list.org
- DefiLlama's chainlist
- chainlist.network
- evmchainlist.org
- evmchainlist.com
- thechainlist.io
- chainlist.info
- chainmap.io
- chainlist.in
- chainz.me
- Otterscan

## 参考
[EVM-based Chains](https://github.com/ethereum-lists/chains)		
	