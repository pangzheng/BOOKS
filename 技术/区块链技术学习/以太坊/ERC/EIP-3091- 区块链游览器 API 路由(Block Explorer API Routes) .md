# EIP-3091: 区块链游览器 API 路由(Block Explorer API Routes) 
## 简单总结
区块链浏览器的标准 API 路由
## 抽象
在链接交易、区块、账户和代币时，该提案带来了区块浏览器 API 路由之间的标准化。
## 动机
目前钱包将交易和账户链接到区块浏览器网页，但随着链的多样性和第二层解决方案的增长，保持一致的用户体验变得更加困难。鉴于这些端点不一致，添加新链或第二层解决方案变得更加困难。

标准化这些链接的 API 路由可以提高钱包和区块浏览器之间的互操作性。这个 EIP 使 [EIP-2015](https://eips.ethereum.org/EIPS/eip-2015) 这样的 RPC 端点更加可行。
## 规格
区块浏览器将针对以下数据相应地路由他们的网页：

- 块

		<BLOCK_EXPORER_URL>/block/<BLOCK_HASH_OR_HEIGHT>
- 交易

		<BLOCK_EXPORER_URL>/tx/<TX_HASH>
- 账户

		<BLOCK_EXPORER_URL>/address/<ACCOUNT_ADDRESS>
- ERC-20 代币

		<BLOCK_EXPORER_URL>/token/<TOKEN_ADDRESS>

## 向后兼容性
此 EIP 在设计时考虑了现有的 API 路由，以减少中断。不兼容的区块浏览器应包含 301 重定向到其现有 API 路由以匹配此 EIP。