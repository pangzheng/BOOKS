# 第三方NFT合约分析
分析其他 NFT 使用合约的方法，来参考。以下数据获取于 opensea 默认合约和第三方合约
## Bored Ape Kennel Club
介绍

开始是猩猩，后来是狗，有路线图，但总体走宠物路线

- 合约地址

	0xBC4CA0EdA7647A8aB7C2061c2E118A18a936f13D
- 一些链接
	- [etherscan.io 合约地址](https://etherscan.io/address/0xBC4CA0EdA7647A8aB7C2061c2E118A18a936f13D) 
		- [合约代码](https://etherscan.io/address/0xBC4CA0EdA7647A8aB7C2061c2E118A18a936f13D#code) 
	- [官网地址](http://boredapeyachtclub.com/#/kennel-club)
- 合约操作(参数)
	- Transfer From (转移)
		- form
		- to
		- tokenid
	- Set Approval For ALL(授权运营商)
		-  运营商地址
		-  是否为真
	-  Approved
		- to
		- tokenid
	- 早期 
		- reserveApes(同时铸造多币)
			- 内部方法  reserveApes
		- Set Base URI
			- baseURI
				- https://us-central1-bayc-metadata.cloudfunctions.net/api/tokens/
		- Flip Sale State
			- 内部函数(flipSaleState) 
		- Mint Ape (铸造)
			-  numberOfTokens
		- Set Starting Index(设置开始索引)
			- 内部函数 setStartingIndex             	