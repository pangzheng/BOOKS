# 部署 Opensea Creature 合约笔记
- 查看文档

	
- 只需获取[仓库代码](https://github.com/ProjectOpenSea/opensea-creatures)

		git clone https://github.com/ProjectOpenSea/opensea-creatures
- 获得以太坊 API 代理调用权限(免费的) [Alchemy API 密钥](https://dashboard.alchemyapi.io/signup?referral=affiliate:e535c3c3-9bc4-428f-8e27-4b70aa2e8ca5)

- 安装 npm
	
	参考精通以太坊
- 安装 truffle
	
	参考精通以太坊
- 安装 yarn
	
	brew install yarn
- 然后使用 Truffle 进行部署
	
			yarn install
			export ALCHEMY_KEY="<your_alchemy_project_id>"
			export MNEMONIC="<metamask>"
			export NETWORK="rinkeby"
			truffle deploy --network rinkeby	
	
	如果您已经在使用 Infura API，则也可以使用 `INFURA_KEY` 环境变量代替 `ALCHEMY_KEY`
	
	
	
## 问题
- opensea 上的代理账户是什么？为什么只能是一个？
- 什么是 `Alchemy ` 或 `Infura `

	这2个算是以太坊的 API 基础设施提供商，提供商用级别的以太坊 API 调用后台，当然还体统更多服务，具体查看对应文档