# Blockscout
Blockscout 是一个功能齐全的开源区块链浏览器
## 特点
- 开源
- 实时交易跟踪

	无需刷新，无限滚动
- 智能合约交互

	可以读取和验证 solidty 合约并访问预先存在合约以快速开发
- 代币支持

	支持 ERC 20/721
- 解码视图

	合约方法调用和日志被解码以直接从 UI 提供附加信息
- 功能齐全的 API 

	GraphQL 提供直接从Web 界面测试 API 调用的机会，并且 RPC 方法有据可查
- 自定义界面

	用户可以轻松的在网络上部署并自定义 Bootstrap 界面
- 以太坊生态系统支持

	支持 POA 网络和 EVM 分叉，如经典以太坊等
	

## 信息和设置	
### 部署前准备
![](./pic/Prerequisites.png)
### 环境变量
- ansible 部署变量查看[这里](https://docs.blockscout.com/for-developers/ansible-deployment)
- 弃用变量查看[这里](https://docs.blockscout.com/for-developers/information-and-settings/deprecated-env-variables)
- 使用 CLI 设置变量，请使用 `export` 命令
	
		$ export ETHEREUM_JSONRPC_VARIANT=parity
		$ export COIN=POA
		$ export NETWORK=POA

#### 表格
|变量|必选参数|描述|默认|版本|需要重新编译
---|---|---|---|---|---
|ETHEREUM_JSONRPC_VARIANT|	v|告诉游览器正在使用哪一种 RPC 客户端(即 Geth、parity、besu..)[更多](https://docs.blockscout.com/for-developers/information-and-settings/client-settings-parity-geth-ganache)|ganache|all|
|ETHEREUM_JSONRPC_HTTP_URL|v|获取区块链信息 RPC URL|localhost:8545|all|
|DATABASE_URL|v|数据库 URL|空|all
|ETHEREUM_JSONRPC_TRACE_URL||`trace_block` 和 `trace_replayTransaction` 使用的 Geth、parity、besu.等专用 RPC |localhost:8545|all|
|Network||主EVM网络环境变量，如 Ethereum或Poa|POA|all
|SUBNETWORK||子网的环境变量，如core或sokol network ，这将在链列表中下拉选择|POA Sokol|all
|LOGO||标题logo图像位置变量,可在[这里](https://github.com/poanetwork/blockscout/tree/master/apps/block_scout_web/assets/static/images)找到不同的logo|`/images/blockscout_logo.svg`|all
|LOGO_FOOTER||页脚logo图像位置，这里找到[更多](https://github.com/poanetwork/blockscout/tree/master/apps/block_scout_web/assets/static/images)|`/images/blockscout_logo.svg`|
|ETHEREUM_JSONRPC_WS_URL||websocket RPC 地址，用于 `newHeads` 订阅提醒所引起获取新块订阅|ws://localhost:8546|all
|ETHEREUM_JSONRPC_TRANSPORT||制定游览器连接到区块链节点的传输，可用的 `http` 和 `ipc`。如果选择 `ipc` ，需要设置 `IPC_PATH` |http|v3.1.0+
|IPC_PATH||如果制定 ipc 传输，设置区块链节点 IPC 文件路径|空|v2.1.1+|
|NETWORK_PATH||用于设置与根目录中显示的内容不同于网络路径。例，添加 `/eth/mainnet/` 到根目录|/|all
|API_PATH||API 文档页面的 API URL中的 Path|/|v3.1.0+|
|SOCKET_ROOT||自定义 websocket 路径|空|v3.1.0+|
|BLOCKSCOUT_HOST||API 端点例主机|localhost|v2.1.0+|
|BLOCKSCOUT_PROTOCOL||blockscount 的 URL 方案|在 prod 环境中使用 `https`,dev 环境中使用 `http`|v2.1.0+|
|SECRET_KEY_BASE||使用 ` mix phx.gen.secre` 生成新的 `Secret Key Base`  字符串以保护资产||all
|CHECK_ORIGIN||当原始标头存在时，用于检查请求的来源。如果默认为 `false`. 如果为 `true` 将检查主机值| false|all
|PORT||应用端口|4000|all
|COIN||此处的coin通过 coinGecko API 检查获取与美金的汇率价格|POA|all
|COIN_NAME||将链添加到 Metamask 的 coin 名称|空|v4.1.2+|
|COINGECKO_COIN_ID||显示设置 CoinGecko 币ID|空|v3.1.2+|
|METADATA_CONTRACT||POA Network 专门使用此环境变量来获取 Validator 信息已显示在 UI 中|空|all
|VALIDATORS_CONTRACT||POA Network 专门使用此环境变量设置获取当前验证者列表|空|all|
|KEYS_MANAGER_CONTRACT||POA Network 专门使用这个环境变量设置 KeyManager 代理合约，以便通过挖掘密钥获取支付密钥，这需要确定对验证者的分布式奖励|空|v3.1.2+|
|REWARDS_CONTRACT||奖励合约地址。此环境变来功能仅在 `EMISSION_FORMAT` 设置为时使用 `POA` | 0xeca443e8e1ab29971a45a9c57a6a9875701698a5|v2.0.4+
|TOKEN_BRIDGE_CONTRACT||代币桥代理合约,用于 `TokenBridge` 供应模块|0x7301CFA0e1756B71869E93d4e4Dca5c7d0eb0AA6|v1.3.2+
|EMISSION_FORMAT||如果有与 `POA` 网络相同的块发布，则设置该变量为 `CHAIN_SPEC_PATH`|`DEFAULT`|v2.0.4+
|CHAIN_SPEC_PATH||用于导入区块链释放的奖励范围和创世账户余额的链规范路径，支持 Geth 和 OpenEthereum 风格规范|空|v2.0.4+
|SUPPLY_MODULE|| xDaiChain/RSK 使用此环境变量来告诉应用程序如何计算链的总供应量，可用值为`TokenBridge`,`RSK`|空|all
|SOURCE_MODULE||该环境变量用于计算汇率，专门提供 xDaiChain 使用，可为 TokenBridge|空|all
|POOL_SIZE||定义允许数据库连接数的变量，不包括只读请求|50|all
|POOL_SIZE_API||用于定义只读 API 请求允许的数据库连接数变量|50|all
|ECTO_USE_SSL||在 Ecto 查询上使用 SSL 的生产环境变量|true|all
|DATADOG_HOST||[Datadog](https://docs.datadoghq.com/integrations/) 集成的主机配置|空|all
|DATADOG_PORT||[Datadog](https://docs.datadoghq.com/integrations/) 集成的端口配置|空|all
|SPANDEX_BATCH_SIZE||[Spandex](https://github.com/spandex-project/spandex) 和 Datadog 配置设置|空|all
|SPANDEX_SYNC_THRESHOLD||[Spandex](https://github.com/spandex-project/spandex) 和 Datadog 配置设置|空|all
|HEART_BEAT_TIMEOUT||生产环境变量，用于发生崩溃时重启程序|30|all
|HEART_COMMAND||生产环境变量，用于发生崩溃时重启程序|空|all
|BLOCKSCOUT_VERSION|| 添加的页脚以表示当前的 BlockScout 版本|空|v1.3.4+
|RELEASE_LINK||页脚指向 Blockscout 发行版本说明的连接|https: //github.com/poanetwork/blockscout/releases/tag/${BLOCKSCOUT_VERSION}|v1.3.5+|
|ELIXIR_VERSION||在 Blockscout 部署之前要在节点上安装 Elixir 版本，用于 ansible 部署脚本中的 bash 脚本|空|all
|BLOCK_TRANSFORMER||块 transformer:base or clique|base|v1.3.4+|
|GRAPHIQL_TRANSACTION||对 GraphiQL 的查询中默认事务的哈希|空|v1.2.0+
|FIRST_BLOCK||块号 catchup fetcher 导入块的开始位置|0|v1.3.8+
|LAST_BLOCK||块号 catchup fetcher 停止导入块的位置|空|v2.0.3+
|TRACE_FIRST_BLOCK||块号，内部交易的索引开始的地方|0|v4.1.0+
|TRACE_LAST_BLOCK||块号，内部交易食物索引停止的地方|空|v4.1.0+
|LINK_TO_OTHER_EXPLORERS||`true or false` ，如果为 true ，则在页脚中添加其他资源管理器的链接|空|v1.3.0+
|OTHER_EXPLORERS||替代资源管理器列表，此环境变量时在 PR [#3414](https://github.com/poanetwork/blockscout/pull/3414) 中引入|空|v3.4.0+
|SUPPORTED_CHAINS||在页脚和链下拉列表中显示的一系列支持的链。这个 var 是在这个 PR [#1900](https://github.com/poanetwork/blockscout/pull/1900) 中引入的，看起来像一个 json 的对象数组|空|v2.0.0+
|CACHE_BLOCK_COUNT_PERIOD||具有共识计数缓存的块的生存时间(以秒为单位)。这个变量是在 [#1876](https://github.com/poanetwork/blockscout/pull/1876) 中引入的|2小时|v4.1.3+
|CACHE_TXS_COUNT_PERIOD||重新启动任务的间隔(以秒为单位)，它计算总txs计数|2小时|v4.1.3+
|CACHE_ADDRESS_COUNT_PERIOD||缓存的生存时间(以秒为单位)。这个变量在 [#2822](https://github.com/poanetwork/blockscout/pull/2822) 中引入|2小时|v4.1.3+
|CACHE_ADDRESS_SUM_PERIOD||地址总和(烧录地址除外)缓存的生存时间(以秒为单位)。这个变量是在 [#2862](https://github.com/poanetwork/blockscout/pull/2862)中引入|1小时|v4.1.3+
|CACHE_TOTAL_GAS_USAGE_PERIOD||以秒为单位重新启动任务的间隔，它计算总气体使用量|2个小时|v4.1.3+
|CACHE_ADDRESS_TRANSACTIONS_GAS_USAGE_COUNTER_PERIOD||以秒为单位重新启动任务的间隔时间，该任务计算地址处的gas使用量|30分钟|v4.1.3+
|CACHE_TOKEN_HOLDERS_COUNTER_PERIOD||以秒为单位重新启动任务的时间间隔，该任务计算代币的持有者数量|1个小时|v4.1.3+
|CACHE_TOKEN_TRANSFERS_COUNTER_PERIOD||以秒为单位重新启动任务的间隔，它计算token的传输计数|1小时|v4.1.3+
|CACHE_ADDRESS_WITH_BALANCES_UPDATE_INTERVAL||以秒为单位重新启动任务的间隔，该任务计算带有余额的地址|30分钟|v4.1.3+
|TOKEN_METADATA_UPDATE_INTERVAL||以秒为单位重启更新 token 原数据的任务时间间隔| `60*60*24*2`|v2.0.1+
|CACHE_AVERAGE_BLOCK_PERIOD||平均块周期缓存的更新，以秒为单位|30分钟|v4.1.3+
|CACHE_MARKET_HISTORY_PERIOD||更新市场历史缓存，以秒为单位|6小时|v4.1.3+
|TOKEN_BALANCE_ON_DEMAND_FETCHER_THRESHOLD_MINUTES||使 token 余额缓存失效的阈值。在 [#3870](https://github.com/blockscout/blockscout/pull/3870)|60分钟|v3.7.1+
|COIN_BALANCE_ON_DEMAND_FETCHER_THRESHOLD_MINUTES||使 coin 余额缓存无效的阈值。在 [#5240](https://github.com/blockscout/blockscout/pull/5240)|60分钟|v4.1.2+
|ALLOWED_EVM_VERSIONS||用于合约验证允许 EVM 版本的逗号分隔列表，这个变量在 [#1964](https://github.com/poanetwork/blockscout/pull/1964) 引入|"homestead, tangerineWhistle, spuriousDragon, byzantium, constantinople, petersburg,istanbul"|v2.0.0+
|UNCLES_IN_AVERAGE_BLOCK_TIME||在平均区块时间计算中，包括或排除非共识区块，排除如果 `false`| `false` |v2.0.1+
|DISABLE_WEBAPP||如果为 true，则 webapp 的端点是隐藏的(编译时)。此外，启用它会使同志通过 `db_notify`| `false`|v2.0.3+|v|
|DISABLE_READ_API||如果 true，则 api 的只读端点被隐藏(编译时)|`false`|v2.0.3+|v|
|DISABLE_WRITE_API||如果 true,则写入 api 的端点事隐藏的(编译时)|`false`|v2.0.3+|v|
|DISABLE_INDEXER||如果 true，索引器应用程序不运行|`false`|v2.0.3+|v|
|INDEXER_DISABLE_PENDING_TRANSACTIONS_FETCHER||如果 true ，挂起事务提取器被禁用|`false`|v4.1.3+
|INDEXER_DISABLE_INTERNAL_TRANSACTIONS_FETCHER||如果 true ，挂起事务提取器被禁用|`false`|v4.1.2+
|INDEXER_DISABLE_BLOCK_REWARD_FETCHER||如果是 true，块奖励获取器被禁用|`false`|v4.1.3+
|INDEXER_DISABLE_ADDRESS_COIN_BALANCE_FETCHER||如果 true ，coin 余额提取器将被禁用|`false`|v4.1.3+
|INDEXER_DISABLE_CATALOGED_TOKEN_UPDATER_FETCHER||如果 true ，禁用遍历token原数据提取器|`false`|v4.1.3+
|INDEXER_MEMORY_LIMIT||索引器的内存软限制|1Gb||v4.1.3+
|INDEXER_EMPTY_BLOCKS_SANITIZER_BATCH_SIZE||空块清理器的批量大小(重新获取器)|100|v4.1.3+
|INDEXER_DISABLE_EMPTY_BLOCK_SANITIZER||如果是 true 空块清理被禁用|`false`|v4.1.3+
|WEBAPP_URL||链接到 web 应用程序的实例， 如: protocol://host/path|空|v2.0.3+
|API_URL||链接到 api 应用程序的实例， 如: protocol://host/path|空|v2.0.3+
|WOBSERVER_ENABLED||如果 true 启用 wobserver 接口|空|v3.3.2+|v|
|SHOW_ADDRESS_MARKETCAP_PERCENTAGE||在顶部账户页面上配置市值百分比列|`true`|v2.1.1+
|CHECKSUM_ADDRESS_HASHES||如果设置为 true,则重定向到地址哈嘻的校验和版本|`true`|v3.1.0+
|CHECKSUM_FUNCTION||定义校验和地址函数2个可用值，`rsk,eth`|eth|v2.0.1+
|DISABLE_EXCHANGE_RATES||禁用或者启用从 coingecko api 获取的 coin 价格| `false`|v3.1.2+|
|DISABLE_KNOWN_TOKENS||禁用或者启用已知合约的代币符号|`false`|v3.4.0+
|ENABLE_TXS_STATS||禁用或者启用每天的 txs 统计信息收集|`false`|v3.1.2+
|SHOW_PRICE_CHART||在主页禁用或者启用 coin图标的价格和市值|`false`|v3.1.2+
|SHOW_TXS_CHART||在主页禁用或者启用每日交易次数表|`false`|v3.1.2+
|HISTORY_FETCH_INTERVAL||以分钟为单位的间隔，请求每天的 txs 计数以在主页上显示每天 txs 计数图表的频率|60|v3.1.2+
|TXS_HISTORIAN_INIT_LAG||txs计数历史获取重的初始延迟(以分钟为单位)，以便在主页上显示每天的 txs 技术历史图表|0|v3.1.2+
|TXS_STATS_DAYS_TO_COMPILE_AT_INIT||获取每天 txs 计数历史的天数，以便将其显示在主页上的每天 txs 计数历史图表中|365|v3.1.2+
|COIN_BALANCE_HISTORY_DAYS||在 coin 余额历史图表中考虑天数|10|v3.1.3+
|APPS_MENU||如果为 treu，则会出现 apps 导航菜单|`false`|v3.3.1+
|EXTERNAL_APPS||在应用程序菜单项重显示的一组外部应用程序。此 var 在 pr [#3184](https://github.com/poanetwork/blockscout/pull/3184) 引入，看起来是像一个json对象数组|空|v3.3.1+|
|ETH_OMNI_BRIDGE_MEDIATOR||home omniBridge 的地址，用于桥接来自eth 的多个 token。提供此地址可启动桥接代币功能，桥接状态和链接到外部链中的原始代币|空|v3.6.0+
|BSC_OMNI_BRIDGE_MEDIATOR||home omniBridge 的地址，用于桥接来自eth 的多个 token。提供此地址可启动桥接代币功能，桥接状态和链接到外部链中的原始代币|空|v3.6.0+
|AMB_BRIDGE_MEDIATORS||逗号分隔的 AMB 扩展的中介地址哈希表，用于通过这些中介地址获取桥接 token|空|v3.3.3+
|GAS_PRICE||Gwei 的 GAS 价格，如果变量存在，gas price 将显示在主页上|空|v3.3.2+
|FOREIGN_JSON_RPC||JOSN RPC 到外链，以便通过 Omni-bridge token 取桥接的元数据。在 PR [#3282](https://github.com/poanetwork/blockscout/pull/3282) 中引入|空|v3.3.3+
|BRIDGE_MARKET_CAP_UPDATE_INTERVAL||TokenBridge 供应模块的市值更新间隔与 TokenBrdge 和 OmniBridge 一样，以秒为单位。PR [#3293](https://github.com/poanetwork/blockscout/pull/3293) | 30分钟|v3.3.3+
|RESTRICTED_LIST||以逗号分隔的地址列表，以启用对它们的受限访问|空|v3.3.3+
|RESTRICTED_LIST_KEY||访问 `RESTRICTED_LIST ` 变量列出地址的key,可以通过查询参数传递到页面的 URL:?key=...|空|v3.3.3+
|ADDRESS_TRANSACTIONS_CACHE_PERIOD||地址的交易计数器的生存时间(以秒为单位)。这个变来功能在 PR [3330](https://github.com/poanetwork/blockscout/pull/3330) 引入|1小时|v3.4.0+
|DISABLE_BRIDGE_MARKET_CAP_UPDATER||禁用来自 TokenBridge 和 AMB 扩展的 TokenBridge 市值重复合并|空|v3.3.3+
|POS_STAKING_CONTRACT||POSDAO 质押合约的地址。提供后，启用 StakingDapp .ValidatorSet 和 BlockReward 合约地址是使用相应的getter 获取的|空|v3.4.0+
|ENABLE_POS_STAKING_IN_MENU||在菜单中启用 Staking dapp| `false`|v3.6.0+
|TOKEN_EXCHANGE_RATE_CACHE_PERIOD|| 管理 token 汇率的缓存失效|1小时|v3.5.0+
|ADDRESS_TOKENS_USD_SUM_CACHE_PERIOD||管理每个代币持有者地址的代币美元价值总和的缓存失效期|空|v3.5.0+
|SHOW_MAINTENANCE_ALERT||在资源管理器顶部禁用/启用公告|错误|v3.6.0+
|MAINTENANCE_ALERT_MESSAGE||资源管理器顶部公告的消息文本|空|v3.6.0+
|SHOW_STAKING_WARNING||在质押 dapp 中禁用/启用公告| `false`|v3.6.0+
|STAKING_WARNING_MESSAGE||staking dapp 餧公告的消息文本|空|v3.6.0+
|CUSTOM_CONTRACT_ADDRESSES_TEST_TOKEN||测试代币的地址列表，将应用测试标签，这些代币将被排除在全桥市值计算之外|空|v3.6.0+
|ENABLE_SOURCIFY_INTEGRATION||通过 sourcify 启用和禁用合同验证| `false`|v3.7.0+
|SOURCIFY_SERVER_URL||sourcify 后段的 url|https://sourcify.dev/server|v3.7.0+
|SOURCIFY_REPO_URL||具有完全验证合同的 Sourcify 存储库 url |https://repo.sourcify.dev/contracts/|v3.7.0+
|CHAIN_ID||网络的链ID 。例如，在 xdai 链的情况下为100|空|v3.7.0+
|MAX_SIZE_UNLESS_HIDE_ARRAY||在智能合约中隐藏长数组 [#4218](https://github.com/blockscout/blockscout/pull/4218)|50|v3.7.1+
|HIDE_BLOCK_MINER||如果值为 `true` 则在块页面和切片隐藏矿工/验证者/排序者 [#4611](https://github.com/blockscout/blockscout/pull/4611) |空|v4.0.0+
|DISPLAY_TOKEN_ICONS||如果为 true，则显示来自 trustwallet 资产存储库 token 图标，[#4596](https://github.com/blockscout/blockscout/pull/4596)|空|v4.0.0+
|SHOW_TENDERLY_LINK||如果 true，则交易页面会显示 `tenderly `开盘按钮，[#4656](https://github.com/blockscout/blockscout/pull/4656)|空|v4.0.0+
|TENDERLY_CHAIN_PATH|| tenderly 中交易的链式路径，例如对于 xdai 中的交易，tenderly 链看起来是 https://dashboard.tenderly.co/tx/xdai/0x... 然后 `TENDERLY_CHAIN_PATH =/xdai. ` [#4656](https://github.com/blockscout/blockscout/pull/4656)|空|v4.0.0+
|MAX_STRING_LENGTH_WITHOUT_TRIMMING||隐藏长合约方法数据，更多详情[#4667](https://github.com/blockscout/blockscout/pull/4667)|2040|v4.0.0+
|RE_CAPTCHA_SECRET_KEY||google reCAPTCHA 密钥。由高级 csv 导出使用 [#4747](https://github.com/blockscout/blockscout/pull/4747)|空|v4.0.0+
|RE_CAPTCHA_CLIENT_KEY||google reCAPTCHA 客户端密钥，由高级 csv 导出使用。[#4747](https://github.com/blockscout/blockscout/pull/4747)|空|v4.0.0+
|ADDRESS_TOKEN_TRANSFERS_COUNTER_CACHE_PERIOD||以秒为单位重新启动任务的间隔，计算该地址的token 传输次数[#4699](https://github.com/blockscout/docs/blob/master/for-developers/information-and-settings/broken-reference/README.md)|空|v4.0.0+
|JSON_RPC|| walletconnect 集成链 JsonRPC [#4931](https://github.com/blockscout/blockscout/pull/4931)|空|v4.0.0+
|API_RATE_LIMIT|| 全局API 速率限制，所有用户每秒的请求数 [#5030](https://github.com/blockscout/blockscout/pull/5030)|50|v4.1.1+
|API_RATE_LIMIT_BY_KEY||静态 API 密钥的专用 API 速率限制 [#5080](https://github.com/blockscout/blockscout/pull/5080)|50|v4.1.1+
|API_RATE_LIMIT_BY_IP||白名单IP的专用 API 速率限制，[#5090](https://github.com/blockscout/blockscout/pull/5090)|50|v4.1.1+
|API_RATE_LIMIT_WHITELISTED_IPS||具有专用速率限制的逗号分隔的白名单 IP 列表。[#5090](https://github.com/blockscout/blockscout/pull/5090)|空|v4.1.1+
|API_RATE_LIMIT_STATIC_API_KEY||具有专用 API 速率限制的静态 API 密钥，[#5080](https://github.com/blockscout/blockscout/pull/5080)|空|v4.1.1+
|ETHEREUM_JSONRPC_DEBUG_TRACE_TRANSACTION_TIMEOUT||存档节点的 `debug_traceTransaction ` Json rpc 方法请求超时。 geth 支持的[时间格式](https://pkg.go.dev/time#ParseDuration) [#5500](https://github.com/blockscout/blockscout/pull/5505)|5s|v4.1.3+
|ADMIN_PANEL_ENABLED||如果 true， admin/ 路由可用 [#5208](https://github.com/blockscout/blockscout/pull/5208)|空|v4.1.2+


### 弃用环境变量
[查看](https://docs.blockscout.com/for-developers/information-and-settings/deprecated-env-variables)
### 数据库存储要求
配置变量 `db_storage` 可用于定义分配给 RDS 实例存储量。这些数字是估值，并且不断增加

|链|存储(GIB)|
---|---
POA Core|90
POA Sokol|80
Ethereum Classic|250
Ethereum Mainnet|5000
Kovan Testnet|250
Ropsten Testnet|6000
Gnosis Chain|100 


### 雨伞项目组织
 Elixir 的[雨伞项目](https://elixir-lang.org/getting-started/mix-otp/dependencies-and-umbrella-projects.html)。下的每个目录 `apps/` 都是一个单独的 [mix](https://hexdocs.pm/mix/Mix.html) 项目和 [OTP](https://hexdocs.pm/elixir/Application.html)应用程序，但是这些项目可以在它们的 `mix.exs` 查看，每个 OTP 应用程序都有一个受限域
 
 目录|一次性密码申请|命名空间|目的
---|---|---|---|---
`apps/ethereum_jsonrpc`|`:ethereum_jsonrpc`|`EthereumJSONRPC`| eth json rpc 客户端，允许知道 `explorer` 的参数格式，但不能直接以来 `explorer`
`apps/explorer`|`:explorer`|`Explorer`|索引链的存储，可以读取和写入后备存储。必须能够在独立于运行时以只读模式启动 `:indexer`,因此不能依赖 `:indexer` 会开始 `:indexer` 索引
`apps/block_scout_web`|`:block_scout_web`|`BlockScoutWeb`|凤凰接口到 `:explorer`,允许web访问的最小接口在`:block_scout_web`.任何业务规则或接口不直接绑定 `phoenix` 或 `plug` 不应该进入`:explorer` 必须能够在独立于运行时以只读模式启动 `:indexer` 因此不会依赖 `:indexer` 它会开始 `:indexer` 索引
`apps/indexer`|`:indexer`|`Indexer`|用于: ethereum_jsonrpc 索引链和批量导入数据到 `:explorer` 任何自动从链中读取并写入的进程 `task` 或都应该在这个限制自动写入，并且只读模式可以通过不运行来实现 

### 客户端设置（Parity、OpenEthereum、Hyperledger Besu、Geth、Ganache）
#### 支持的客户
BlockScout 目前支持 Parity、OpenEthereum、Geth、Nethermind、Hyperledger Besu 和 Ganache 客户端。要定义节点变体，建议定义`ETHEREUM_JSONRPC_VARIANT` 环境变量。正确的值包括：

- `parity` (Parity、OpenEthereum 和 Nethermind 相同（默认）)
- `geth`
- `besu`
- `ganache`

	BlockScout 目前需要一个全的存档节点，以便导入目标网络上每个地址的每个状态更改。

#### 开发环境
- [Explorer](https://github.com/poanetwork/blockscout/blob/master/apps/explorer/config/dev.exs)
- [indexer](https://github.com/poanetwork/blockscout/blob/master/apps/indexer/config/dev.exs)

		variant =
		  if is_nil(System.get_env("ETHEREUM_JSONRPC_VARIANT")) do
		    "ganache"
		  else
		    System.get_env("ETHEREUM_JSONRPC_VARIANT")
		    |> String.split(".")
		    |> List.last()
		    |> String.downcase()
		  end

### 生产环境
- [Explorer](https://github.com/poanetwork/blockscout/blob/master/apps/explorer/config/prod.exs)
- [indexer](https://github.com/poanetwork/blockscout/blob/master/apps/indexer/config/prod.exs)

		variant =
		  if is_nil(System.get_env("ETHEREUM_JSONRPC_VARIANT")) do
		    "parity"
		  else
		    System.get_env("ETHEREUM_JSONRPC_VARIANT")
		    |> String.split(".")
		    |> List.last()
		    |> String.downcase()
		  end

### OpenEthereum Client
	--jsonrpc-interface all --jsonrpc-apis web3,eth,net,parity,pubsub,traces --ws-interface all --fat-db=on --pruning=archive --ws-apis all --ws-origins all --ws-hosts all
	
名称|环境变量|默认值|描述
---|---|---|---
http endpoint| `ETHEREUM_JSONRPC_HTTP_URL` | `http://localhost:8545`|用于获取 `blocks,transactions,receipts,coin/token balances`
tracing endpoint|`ETHEREUM_JSONRPC_TRACE_URL`| `http://localhost:8545`|用于获取 `internal transactions` 和 `block traces` 在大多数情况下，设置于上面相同
websockets endpoint|`ETHEREUM_JSONRPC_WS_URL`| `ws://localhost:8546`|订阅 `newHeads` 通知索引器从订阅获取新块

#### 开发环境
- [Explorer](https://github.com/poanetwork/blockscout/blob/59d8423e7ca3f608dbea411d4a0dc9bb4662a891/apps/explorer/config/dev/parity.exs#L8-L22)
- [indexer](https://github.com/poanetwork/blockscout/blob/59d8423e7ca3f608dbea411d4a0dc9bb4662a891/apps/indexer/config/dev/parity.exs#L9-L23)

### 生产环境
- [Explorer](https://github.com/poanetwork/blockscout/blob/59d8423e7ca3f608dbea411d4a0dc9bb4662a891/apps/explorer/config/prod/parity.exs#L8-L22)
- [indexer](https://github.com/poanetwork/blockscout/blob/59d8423e7ca3f608dbea411d4a0dc9bb4662a891/apps/indexer/config/prod/parity.exs#L9-L23)

### Geth Client
	sudo /usr/bin/geth --http --http.addr 0.0.0.0 --port 30303 --http.port 8545 --http.api debug,net,eth,shh,web3,txpool --wsapi "eth,net,web3,network,debug,txpool" --ws --ws.addr 0.0.0.0 --ws.port 8546 --wsorigins "*" --rinkeby --datadir=/rinkeby --syncmode=full --gcmode=archive --http.vhosts=* 

跟踪和修剪:默认情况下，最后 128 个块的状态保存在内存中，大多数状态都是垃圾收集。如果正在运行区块链游览器或其他依赖于事务跟踪而没有归档节点的服务(`--gcmode=archive`),则需要在此窗口内进行跟踪或指定 `reexec` 跟踪器选项以允许重新生成历史状态。理想状态下，切换到链式跟踪，在所有跟踪的块中分摊开销

名称|环境变量|默认值|描述
---|---|---|---
http endpoint| `ETHEREUM_JSONRPC_HTTP_URL` | `http://localhost:8545`|用于获取 `blocks,transactions,receipts,coin/token balances`
websockets endpoint|`ETHEREUM_JSONRPC_WS_URL`| `ws://localhost:8546`|订阅 `newHeads` 通知索引器从订阅获取新块

#### 开发环境
- [Explorer](https://github.com/poanetwork/blockscout/blob/59d8423e7ca3f608dbea411d4a0dc9bb4662a891/apps/explorer/config/dev/geth.exs#L8-L17)
- [indexer](https://github.com/poanetwork/blockscout/blob/59d8423e7ca3f608dbea411d4a0dc9bb4662a891/apps/indexer/config/dev/geth.exs#L9-L18)

### 生产环境
- [Explorer](https://github.com/poanetwork/blockscout/blob/59d8423e7ca3f608dbea411d4a0dc9bb4662a891/apps/explorer/config/prod/geth.exs#L8-L17)
- [indexer](https://github.com/poanetwork/blockscout/blob/59d8423e7ca3f608dbea411d4a0dc9bb4662a891/apps/indexer/config/prod/geth.exs#L9-L18)	
	
### 链之间的差异配置
所有链必须配置

环境变量|默认
---|---
BLOCKSCOUT_VERSION|`unknown`
COIN|`POA`
DB_HOST|-
DB_PASSWORD|-
DB_PORT|-
DB_USERNAME|-
ETHEREUM_JSONRPC_HTTP_URL|`http://localhost:8545`
ETHEREUM_JSONRPC_TRACE_URL|`http://localhost:8545`
ETHEREUM_JSONRPC_WS_URL|`ws://localhost:8546`
ETHEREUM_JSONRPC_VARIANT|`parity`
HEART_BEAT_TIMEOUT|`30`
HEART_COMMAND|`sudo systemctl restart explorer.service`
LOGO|`/images/blockscout_logo.svg`
NETWORK|`POA Network`
SUBNETWORK|`Sokol Testnet`
NETWORK_ICON|`_test_network_icon.html`
LINK_TO_OTHER_EXPLORERS|`true`

### 无法使用 env 更改的配置
- [更新 验证者、矿工、签名者](https://github.com/poanetwork/blockscout/blob/12aa15671142af00b35ff05aeac107c2c686c4c8/apps/block_scout_web/priv/gettext/default.pot)
- [更新使用内存](https://github.com/poanetwork/blockscout/blob/12aa15671142af00b35ff05aeac107c2c686c4c8/apps/indexer/config/config.exs#L36)
- [更新主题](https://github.com/poanetwork/blockscout/blob/12aa15671142af00b35ff05aeac107c2c686c4c8/apps/block_scout_web/assets/css/theme/_variables.scss#L1)
- [更新币名](https://github.com/poanetwork/blockscout/blob/5b5a0b3cfe47fcbb3631b82e58aeb2c7c9c48504/apps/block_scout_web/priv/gettext/en/LC_MESSAGES/default.po#L398)
- [更新其他资源](https://github.com/poanetwork/blockscout/blob/12aa15671142af00b35ff05aeac107c2c686c4c8/apps/block_scout_web/config/config.exs#L22-L26)

#### POA Core
[查看](https://docs.blockscout.com/for-developers/information-and-settings/deployment-differences-between-chains#poa-core)
#### POA Sokol
[查看](https://docs.blockscout.com/for-developers/information-and-settings/deployment-differences-between-chains#poa-core)
#### xDAI
[查看](https://docs.blockscout.com/for-developers/information-and-settings/deployment-differences-between-chains#poa-core)
#### Rinkeby,Goerli
[查看](https://docs.blockscout.com/for-developers/information-and-settings/deployment-differences-between-chains#poa-core)

### 添加验证者元数据
[查看](https://docs.blockscout.com/for-developers/information-and-settings/add-validator-metadata)
### 共享锁
ShareLock 是 PostgreSQL 内部使用的行级别锁定机制
[查看](https://docs.blockscout.com/for-developers/information-and-settings/sharelock)
### Blockscout.com 的 haproxy 设置
Blockscout.com 对所有 BlockScout 托管和外部实例使用 haproxy。

如果想将您的实例添加到 Blockscout.com,请进行必要的更改并提交 PR 到 [https://github.com/blockscout/haproxy-config](https://github.com/blockscout/haproxy-config)

Blockscout 使用由两部分组成网络 url 路径名

- 第一部分 (org) 指定网络的组织/类型
- 第二部分(name)网络命名空间

例

- eth 主网

		https://blockscout.com/eth/mainnet
		(org=eth, netname=mainnet)
- eth 经典

		https://blockscout.com/etc/mainnet 
		(org=etc, netname=mainnet)
- Kovan testnet

		https://blockscout.com/eth/kovan 
		(org=eth, netname=kovan)

### 1 为你的 blockscout 实例的 url 添加一个新的 org
- 在 cfg 的 `acl` 的 `#Check for network type` 添加一个新的

		#Check for network type
		...
		acl is_myorg path_beg -i /myorg 
- 在 `#default redirect` 部分添加 `!is_myorg` 对应的异常

		# default redirect
		redirect prefix /poa/xdai if !is_websocket !is_poa !is_etc !is_eth !is_lukso !is_rsk !is_myorg !is_cookie
- 添加 `!is_myorg` 到 `redirect for networks` 中

		#redirect for networks
		redirect prefix /poa/core if !is_websocket !is_poa !is_etc !is_eth !is_lukso !is_rsk !is_myorg is_cookie_core
		redirect prefix /poa/sokol if !is_websocket !is_poa !is_etc !is_eth !is_lukso !is_rsk !is_myorg is_cookie_sokol
		# do this for every line

### 2 用你的网络名添加一个新的 ACL
在 `#Check for network name` 添加一个 `acl` 与 `netname` 现有名称相似的心名称

	#Check for network name
	...
	acl is_mynet path_beg -i /myorg/mynet
### 3 设置 `rpc/websockets` 服务器的 cookie 和名字

	#Check for cookies
	...
	acl is_cookie_mynet hdr_sub(cookie) network=mynet
	
	#Default backends
	...
	use_backend mynet if is_mynet
	
	#WebSocket backends
	...
	use_backend mynet_ws if is_cookie_mynet is_websocket
### 4 在文件末尾添加新的后端部分	
在文件末尾添加新的 `backend_mynet` 和 `backend_mynet_ws` 部分，并提供 blockscout 实例的 dns 名称

例如

	backend mynet
	    #Proxy mode
	    mode http
	
	    #Backend queries should not have network prefix, so we delete it
	    acl is_mynet path_beg /myorg/mynet
	    reqirep ^([^\ ].*)myorg/mynet[/]?(.*) \1\2 if is_mynet
	
	    #Setting headers to mask our query
	    http-request set-header X-Forwarded-Host %[req.hdr(Host)]
	    http-request set-header X-Client-IP %[src]
	    http-request set-header Host my-blockscout-instance.com
	
	    #Specifying cookie to insert
	    cookie network insert
	
	    #Server list. Format:
	    #server <name> <address>:<port> cookie <network_name> check inter 60000 fastinter 1000 fall 3 rise 3 ssl verify none
	    server mynet_server my-blockscout-instance.com:443 cookie mynet check inter 60000 fastinter 1000 fall 3 rise 3 ssl verify none
	
	backend mynet_ws
	    #Check if WebSocket request is correct
	    acl hdr_websocket_key      hdr_cnt(Sec-WebSocket-Key)      eq 1
	    acl hdr_websocket_version  hdr_cnt(Sec-WebSocket-Version)  eq 1
	    http-request deny if ! hdr_websocket_key ! hdr_websocket_version
	
	    #WebSockets should be forwarded, not http proxied
	    option forwardfor
	
	    #Setting headers to mask our query
	    http-request set-header Host my-blockscout-instance.com
	    http-request set-header Origin https://my-blockscout-instance.com
	
	    #Specifying cookie to insert
	    cookie network insert
	
	    #Server list. Format:
	    #server <name> <address>:<port> cookie <network_name> check inter 60000 fastinter 1000 fall 3 rise 3 check inter 60000 fastinter 1000 fall 3 rise 3 ssl verify none maxconn 30000
	    server mynet_server my-blockscout-instance.com:443 cookie mynet check inter 60000 fastinter 1000 fall 3 rise 3 ssl verify none maxconn 30000
	    
	    
- backend mynet:
	- acl is_...
	- reqirep ...
	- http-request set-header Host ...
	- server mynet_server... //不要忘记这里指定端口 443
- backend mynet_ws:
	- http-request set-header Host...
	- http-request set-header Origin ... //不要忘记这里 https
	- server mynet_server...  //不要忘记这里指定端口 443	
### Docker 集成
#### 用法
Blocksout 需要 PostgreSQL 服务器，这是通过启动脚本提供的(创建一个名为 postgres 的新 docker 实例)

- 启动命令

	设置并在 docker 容器中启动 Blockscout 。将其链接到本地的环境，请使用 `make start` [环境变量](https://docs.blockscout.com/for-developers/information-and-settings/docker-integration-local-use-only#env-variables) 如下描述

	链接到 mac/win 上端口运行本地实例为  `ganace 2000`
	
		COIN=DAI \
		ETHEREUM_JSONRPC_VARIANT=ganache \ 
		ETHEREUM_JSONRPC_HTTP_URL=http://host.docker.internal:2000 \
		ETHEREUM_JSONRPC_WS_URL=ws://host.docker.internal:2000 \
		make start
	Blockscout 在 `localhost:4000`
		
	- mac/win 

		docker 提供了一个特殊的 url - host.docker.interanl 在容器中可用路由到本级  
	- linux

		docker 开始使用 `--nerwork=host` 所有服务可用 localhost
- 迁移

	默认情况下，创建 `Makefile` 完成迁移 `postgresql`，还可以使用 `make migrate` ，命令手动运行迁移
	
	注意：迁移将清理本地数据库

#### 环境变来那个
BlockScout 支持3种不同的 Json RPC 变体，可以使用 `ETHEREUM_JSONRPC_VARIANT` 环境变来功能配置变体

- 例子

		ETHEREUM_JSONRPC_VARIANT=ganache \
		make start
	注意，请不要使用 sudo with make start 命令，因为基本上，每当使用该命令时，都会导致[无法与节点通讯](https://github.com/poanetwork/blockscout/issues/2795#issuecomment-546076853)。
- 可用选项
	- `parity`(默认)
	- `geth`
	- `besu` 
	- `ganache`

变量|描述|默认值
---|---|---
ETHEREUM_JSONRPC_VARIANT|JSON RPC 服务的变体，填写上面可选4个选项|parity
ETHEREUM_JSONRPC_HTTP_URL| HTTP JSON RPC URL 仅用于 geth 或 ganache |基于 JSONRPC
ETHEREUM_JSONRPC_WS_URL|WS JSON RPC |基于 JSONRPC
ETHEREUM_JSONRPC_TRACE_URL|用于跟踪 URL 仅用于 parity |`http://localhost:8545`
COIN|默认硬币|POA
LOGO|默认硬币logo|空
NETWORK|网络|空
SUBNETWORK|子网|空
NETWORK_ICON|网络图标|空
NETWORK_PATH|网络路径|`/`
	
#### JSONRPC 变体默认值
- ETHEREUM_JSONRPC_HTTP_URL `默认值`
	- `parity` - `http://localhost:8545`
	- `geth` - `https://mainnet.infura.io/8lTvJTKmHPCHazkneJsY`
	- `besu` - `http://localhost:8545`
	- `ganache` - `http://localhost:7545`
- ETHEREUM_JSONRPC_WS_URL `默认值`		
	- `parity` - `ws://localhost:8546`
	- `geth` - `wss://mainnet.infura.io/8lTvJTKmHPCHazkneJsY/ws`
	- `besu` - `ws://localhost:8546`
	- `ganache` - `ws://localhost:7545`

## ansible 部署(aws)
### [概述](https://docs.blockscout.com/for-developers/ansible-deployment/overview)
### [预先准备](https://docs.blockscout.com/for-developers/ansible-deployment/prerequisites)
### [AWS 权限设置](https://docs.blockscout.com/for-developers/ansible-deployment/aws-permissions)
### [变量](https://docs.blockscout.com/for-developers/ansible-deployment/variables)
### [部署 Blockscout 基础设施](https://docs.blockscout.com/for-developers/ansible-deployment/deploying-the-blockscout-infrastructure)
### [部署 Blockscont](https://docs.blockscout.com/for-developers/ansible-deployment/deploying-blockscout) 
### [销毁基础设施](https://docs.blockscout.com/for-developers/ansible-deployment/destroying-provisioned-infrastructure)
### [常见附加任务](https://docs.blockscout.com/for-developers/ansible-deployment/common-additional-tasks)
### [常见错误和问题](https://docs.blockscout.com/for-developers/ansible-deployment/common-errors-and-questions)			
### [AWS Marketplace](https://docs.blockscout.com/for-developers/ansible-deployment/aws-marketplace)

## 手动部署
### 检查
- 检查环境是否部署了[合格的数据库](https://docs.blockscout.com/for-developers/information-and-settings/database-storage-requirements)
- Blockscout 需要一个[全节点](https://docs.blockscout.com/for-developers/information-and-settings/client-settings-parity-geth-ganache)

### 部署步骤
1. `git clone https://github.com/poanetwork/blockscout`
- `cd blockscout`
- 提供数据库 URL:

	`export DATABASE_URL=postgresql://user:password@localhost:5432/blockscout `

	- Linux: 更新数据库用户名和密码
	- Mac: 使用登陆用户名和空密码
	- 可选: 更改 `apps/explorer/config/test.exs ` 测试环境变量
- 安装 Mix 依赖和编译

		mix do deps.get, local.rebar --force, deps.compile 
- 通过设置相应的环境变量为数据库生成一个新的 `secret_key_base`

		export SECRET_KEY_BASE=VTIB3uHDNbvrY0+60ZWgUoUBKDn9ppLR8MI4CpRz4/qLyEFs54ktJfaNT6Z221No
	使用 `mix phy.gen.secret` 生成 `secret_key_base`
- 如果之前部署过，清理环境执行 `mix phx.digest.clean`
-  根据需求设置其他环境变量，例如

		export ETHEREUM_JSONRPC_VARIANT=parity
		export ETHEREUM_JSONRPC_HTTP_URL=http://localhost:8545
		export DATABASE_URL=postgresql://...
		export COIN=DAI
		export ...
- 编译

		mix compile		
- 如果未运行，请启动 postgres

		pg_ctl -D /usr/local/var/postgres start	 
- 检查 [数据库状态](https://www.postgresql.org/docs/9.6/app-pg-isready.html) 是否是 pg_isready
- 创建和迁移数据库

		mix do ecto.create,ecto.migrate
	如果开发环境中使用了不同的区块链运行了应用程序，请删除以前的数据库以前
		
		mix do ecto.drop, ecto.create, ecto.migrate
	不想丢失数据的话，请不要在生产数据运行
- 安装 Node.js 依赖
	- `cd apps/block_scout_web/assets; npm install && node_modules/webpack/bin/webpack.js --mode production; cd -`
	- `cd apps/explorer && npm install; cd -	`	
- 部署构建静态数据

		mix phx.digest 
- 开发中启动 https。 凤凰服务器仅使用 https
	- `cd apps/block_scout_web; mix phx.gen.cert blockscout blockscout.local; cd -`
	- 将 `blockscout` 和 `blockscout.local` 添加到 /etc/hosts

			   127.0.0.1       localhost blockscout blockscout.local
			
			   255.255.255.255 broadcasthost
			
			   ::1             localhost blockscout blockscout.local
- 如果使用 chrome ,请启动 chrome://flags/#allow-insecure-localhost
- 回到根目录，启动

		mix phx.server

### 清除实例
为了删除构建数据，执行脚本 `./rel/commands/clear_build.sh`，它将删除文件夹

- 删除 `_build` 和 `deps` 目录
- 删除位于节点的模块
	- `app/block_scout_web/assets/node_modues` 目录
	- `app/explorer/node_modules` 目录
- 删除 `logs/dev` 目录
- 如果还需要删除静态数据(`apps/block_scout_web/priv/static`) 例如更改前端，执行命令增加 `-f` 参数

		./rel/commands/clear_build.sh -f
		
## 配置选项
### CSS 配置和预设
- CSS

	更新导入指令 `apps/block_scout_web/assets/css/theme/_variables.scss` 以选项设置的 css 文件。这反应在 `production-{chain}` 的每个实例分支中，例如在 `production-xdai` 分支中
	
	- 注释掉 `@import "neutral_variables"` 
	- 并取消注释 `@import "dai-variables"`
- ENV

	更新[环境变量](https://docs.blockscout.com/for-developers/information-and-settings/env-variables)
	
当前的 CSS 预设

	@import "theme/base_variables";
	@import "neutral_variables";
	// @import "dai_variables";
	// @import "ethereum_classic_variables";
	// @import "ethereum_variables";
	// @import "ether1_variables";
	// @import "expanse_variables";
	// @import "gochain_variables";
	// @import "goerli_variables";
	// @import "kovan_variables";
	// @import "lukso_variables";
	// @import "musicoin_variables";
	// @import "pirl_variables";
	// @import "poa_variables";
	// @import "posdao_variables";
	// @import "rinkeby_variables";
	// @import "ropsten_variables";
	// @import "social_variables";
	// @import "sokol_variables";
	// @import "tobalaba_variables";
	// @import "tomochain_variables";
	// @import "rsk_variables";

### 品牌配置
注意: 如果部署过 blockscout ，请确保在更新前，删除静态资产(使用 `mix phx.digest.clean` 或手动删除位于 `apps/block_scout_web/priv/static ` 的资产)和重新重建/启动 blockscout 

为了重建新前端运行

- `cd apps/block_scout_web/assets; npm install && node_modules/webpack/bin/webpack.js --mode production; cd -`
- `mix phx.digest`

#### 可设置内容
- 主题颜色和其他基于 css 的属性

	参阅 [CSS 配置和预设](https://docs.blockscout.com/for-developers/configuration-options/css-configuration-and-presets)以将你的实例设置为您要使用的样式表。
- logo

	使用 LOGO 和 LOGO_FOOTER [环境变量](https://docs.blockscout.com/for-developers/information-and-settings/env-variables) 指向上传的 logo。logo和其他资产应位于 `apps/block_scout_web/assets/static/images`
- Coin 

	Coin 和 Coingecko_coin_id 环境变量[自定义 coin 符号](https://docs.blockscout.com/for-developers/developer-faqs/how-can-i-customize-the-coin-symbol)
- MetaData

	MetaTags 可以为模版区域种的各个页面进行调整。对于主页，标签数据从这里呈现 `/apps/block_scout_web/lib/block_scout_web/templates/chain/_metatags.html.eex`
- 标题和字幕

	游览器选项卡显示来自 2 SUBNETWORK+NETWORK [环境变量](https://docs.blockscout.com/for-developers/information-and-settings/env-variables)的标题
- Favicon

	替换 `apps/block_scout_web/assets/static/images` 文件夹种当前 favicon。 
- Menus

	使用 APPS_MENU [环境变量](https://docs.blockscout.com/for-developers/information-and-settings/env-variables) 来包含应用程序菜单和 EXTERNAL_APPS 来填充菜单
- 顶部导航栏

	数据从这里渲染  `/apps/block_scout_web/lib/block_scout_web/templates/layout/_topnav.html.eex`
- 页脚

	数据从这里渲染 `/apps/block_scout_web/lib/block_scout_web/templates/layout/_footer.html.eex`
- 自定义主题

	Blockscout 团队提供[自定义主题](https://docs.blockscout.com/for-projects/premium-features/custom-branded-themes)

### 自动撑起
默认情况下 Blockscout ，如果它崩溃不会自动重启。要启动自动重启，请将[环境变量](https://docs.blockscout.com/for-developers/information-and-settings/env-variables)设置为自动重启。配置  `HEART_COMMAND` 心跳时，应该考虑无响应的等待时间。

它将 kill 当前实例，并执行  `HEART_COMMAND`。默认请夸过你相爱，设置 `ERL_CRASH_DUMP_SECONDS` 为正整数，详情[查看](http://erlang.org/doc/man/heart.html)

### 前端配置文件
- javascript

	所有js文件都位于 [apps/block_scout_web/assets/js](https://github.com/poanetwork/blockscout/tree/master/apps/block_scout_web/assets/js). 主文件是 [app.js](https://github.com/poanetwork/blockscout/blob/master/apps/block_scout_web/assets/js/app.js)。此文件导入应用程序中所有的 js 文件。如果要创建新的 js 文件，需要考虑在 [/js/pages](https://github.com/poanetwork/blockscout/tree/master/apps/block_scout_web/assets/js/pages) 或 [/js/lib](https://github.com/poanetwork/blockscout/tree/master/apps/block_scout_web/assets/js/lib) 创建。
- js/lib

	此文件夹包含可用于任何页面或作为某些组建的帮助程序的所有脚本
- js/pages

	此文件夹包含特定于页面的脚本
- redux

	本项目使用 redux 来控制某些页面中的状态。有使用 phoenix 频道的实时事件页面，例如地址页面。页面状态经常根据它监控的时间改变。 redux 也用于异步加载一些内容，参见 [async_listing_load.js](https://github.com/poanetwork/blockscout/blob/master/apps/block_scout_web/assets/js/lib/async_listing_load.js) 。
	
	要了解如何构建需要的 redux 页面，查看 [redux_helper.js](https://github.com/poanetwork/blockscout/blob/master/apps/block_scout_web/assets/js/lib/redux_helpers.js)

### CI 更新
监控构建状态，轻使用 url 配置本地 [CCMenu](http://ccmenu.org/)

	https://circleci.com/gh/poanetwork/blockscout.cc.xml?circle-token=f8823a3d0090407c11f87028c73015a331dbf604
### 国际化
该一个你哟功能程序目前已经国际化，它仅本地化为英语，翻译新的字符串

- 设置翻译文件

		cd apps/block_scout_web; mix gettext.extract --merge; cd -
- 编辑新字符串

		apps/block_scout_web/priv/gettext/en/LC_MESSAGES/default.po

### 监控指标
#### 观察者
[webserver]()配置为在 web 姐买那种姑娘显示来自 api 的数据。 `/metrics` 要查看，请转到 `/wobserver` 查看链。例如

	https://blockscout.com/eth/mainnet/wobserver
#### prometheus
Blocksout 设置将 prometheus 指标导出到 `/metrics`

- 安装 prometheus

		brew install prometheus
- 启动网络服务

		iex -S mix phx.server
- 启动 prometheus 

		prometheus --config.file=prometheus.yml

#### grafana
grafana 仪表盘用于指标显示

- 安装 grafana

		brew install grafana					
- 安装图形插件

		grafana-cli plugins install grafana-piechart-panel
- 启动 grafana

		brew services start grafana		
- 将 prometheus 添加数据源
	- open http://localhost:3000/datasources
	- `+ Add data source`
	- 输入名称
	- 选择 `type` prometheus
	- 设置 URL `http://localhost:9090`
	- 设置 `scrape interval` 到 `10s`
- 从 `https://github.com/deadtrickster/beam-dashboards` 添加仪表盘，对于 repo 中每个 `*.json` 文件
	- `open http://localhost:3000/dashboard/import`
	- 将 json 文件的内容复制
	- 点击加载
- 查看仪表盘

### 跟踪
Blockscout 支持通过 [Spandex](https://github.com/spandex-project/spandex) 进行跟踪，每个应用程序都有自己内部配置的跟踪器。要启用跟踪，请访问每个应用程序 `config/<env>.ex.` 更改 `disabled?:true` 为 `disabled?:false`。

目前仅支持 [datadog](https://www.datadoghq.com/) 作为后端跟踪
#### Datadog
需要启用 `Spandex` 设置 `DATADOG_HOST` 和 `DATADOG_PORT` 环境变来功能，更多[查看](https://docs.datadoghq.com/)
### 其他
如果想使用不同后端，请删除 from 并按照提供的任何说明设置后端。

`SpandexDatadog.ApiServer` `Supervisor.child_spec` `Explorer.Application` `Spandex`

### 日志配置
调整文件数和最大大小以设置磁盘空间

每个 [雨伞应用程序](https://docs.blockscout.com/for-developers/information-and-settings/untitled) (索引器、资源管理器、block_scout_web、ethereum_json_rpc) 在 `prod.exs` 文件中都包含一个日志 `rotate` 配置

例子

	https://github.com/blockscout/blockscout/blob/master/apps/explorer/config/prod.exs#L36-L51.

配置

	config :logger, :explorer,
	  level: :info,
	  path: Path.absname("logs/prod/explorer.log"),
	  rotate: %{max_bytes: 52_428_800, keep: 19}
	
	config :logger, :reading_token_functions,
	  level: :debug,
	  path: Path.absname("logs/prod/explorer/tokens/reading_functions.log"),
	  metadata_filter: [fetcher: :token_functions],
	  rotate: %{max_bytes: 52_428_800, keep: 19}	
`config:logger` 定义了几个变量，包含

- `level` (info,debug,error) 
- `rotate` 
	- 最大字节:每个文件的最大字节大小 `52,428,800`
	- 保持:`19` 时要存储的最大文件数

如果遇到磁盘空间问题，请修改级别、默认文件数和最大字节，以减少日志占用空间。

`ecto.log`(数据库交互包装器)的附加日志 rotate 配置，`error.log` 位于通用配置文件

	https://github.com/blockscout/blockscout/blob/master/config/config.exs##L47-L61

### 内存使用
用于构建所有块、余额(coin和token)和内部交易的索引的工作队列可能会变得非常大，默认情况下，软限制为 1 GIB，可以在一下位置更改 `apps/indexer/config/config.exs`

	config :indexer, memory_limit: 1 <<< 30
每分钟检查一次内存使用情况。如果达到软限制，可收缩工作队列来减少一半的负载。卸载的负载将从数据库中恢复，与重新启动服务器时相同，因此重建队列会更慢，但使用更少内存。

如果所有队列都处于最小大小，则无法回收更多内存并且将记录错误。

## 索引
Blockscout 可能需要一些时间来完全索引一个链，更大的链则需要更多时间。索引从链的头部(当前块)开始，然后向后回溯到创世区块。创世区块时在此过程中索引的最终块
### 索引期间的消息		
- `n% Blocks Indexed`

	现在正在索引这个链。有些计数可能不准确。这意味着 Blocksout 仍在收集和处理块。一旦创始块被索引，该消息应该消失
- `Indexing tokens `

	现在正在索引这个链。有些计数可能不准确。这意味着 Blocksout 已经收集了所有区块，但仍在使用归档节点跟踪 API 索引内部事务。
	
### 监控索引过程
- 块数应接近链中的块数。 

	使用 `SELECT COUNT(1) FROM blocks;` 应该约等于链中的区块数量
- 可以使用查询监控内部事务获取 `SELECT COUNT(1) FROM pending_block_operations;` 在内部事务处理期间它应该趋于零

## [测试](https://docs.blockscout.com/for-developers/testing)
## 开发者常见问题
- [如何加速托管的 Blockscout 实例的速度](https://docs.blockscout.com/for-developers/developer-faqs/how-do-i-speed-up-my-hosted-blockscout-instance)
- 如何自定义 coin 符号

	Blockscout 利用 `Coin` 环境变量从 Coinmarketcap.com API 或 Coin API 中提取相关的市场数据，以在整个程序中提供定价数据
	
	- 可以手动设置
	
			https://github.com/poanetwork/blockscout/blob/03061be2f710a1fc84c6ce555f676e8bb9dfa54d/apps/explorer/config/config.exs#L11
	-  显示的 coin 符号

		Blockscout 用于 `gettext` 在整个应用程序中创建显示的 coin 符号。gettext 文档[获得](https://hexdocs.pm/gettext/Gettext.html) 。在Blockscout 中，雨伞应用程序中 `po` 和 `pot` 文件。`BlockScoutWeb` 
		
		默认文本为 [实例](https://github.com/poanetwork/blockscout/blob/03061be2f710a1fc84c6ce555f676e8bb9dfa54d/apps/block_scout_web/priv/gettext/en/LC_MESSAGES/default.po#L405)
		
		- msgstr "ETH"
		- msgstr "POA"

		要在整个应用程序中编辑 POA 符号，请将 `msgstr` 以下文件中的值替换成你的目标值
		
		- Po 文件

				https://github.com/poanetwork/blockscout/blob/03061be2f710a1fc84c6ce555f676e8bb9dfa54d/apps/block_scout_web/priv/gettext/en/LC_MESSAGES/default.po
		- Pot 文件

				https://github.com/poanetwork/blockscout/blob/03061be2f710a1fc84c6ce555f676e8bb9dfa54d/apps/block_scout_web/priv/gettext/default.pot		
		内容移动 https://forum.poa.network/t/faq-how-can-i-customize-coin-symbol/1875
- [如何使用 aws codedeploy 管理部署](https://docs.blockscout.com/for-developers/developer-faqs/how-do-i-manage-deployment-with-aws-codedeploy)
- 如何在我的BlockScout 部署中替换缺失的资产和版本号
	- 资产版本
		- 查询对应 blockscout 的的公网ip
		- 通过 ssh 连入主机
		- 进入 `cd /opt/app/apps/block_scout_web/priv/static`
		- 根据缺少的内容，将缺少的资产添加到 `./images` 文件夹中，再刷新页面。
			- 如 `./images` 文件夹缺少 `favicon.ico`,只需要从根目录 `cp favicon.ico ./images/` 复制。现在应该可以查看到丢失的资产内容 
	- 页脚缺少版本
		- 查询对应 blockscout 的的公网ip
		- 通过 ssh 连入主机
		- 进入 `cd /opt/app/apps/block_scout_web/lib/block_scout_web/templates/layout`  
		- 打开文件 `_footer_html.eex` 页脚模版。
			-  例如 `nano ./_footer.html.eex` 
		-  用硬编码的新版本修复 `<% version = version() %>`
			- 例如 `<% version = 'v1.3.3-beta' %>`
		- 重启 blocksout 

				sudo systemctl restart explorer.service   
	内容移动：https://forum.poa.network/t/faq-missing-assets-version-number-in-blockscout-deployment/2459
- 如何修复 gettext.error

	对特定 blockscout 应用程序进行更改后，可能收到此错误
	
		Gettext.Error) translation with msgid '...<msg_here>...' has a non-empty msgstr
	这提示要更新的 gettext ，更改方法如下
	
	- 跳转到更改的目录 `./apps/{name_of_app} `
	- 运行 `mix gettext.extract —merge`
	- 根据需要对其他应用目录做重复操作

	有关 gettext 更多[信息](https://hexdocs.pm/gettext/Mix.Tasks.Gettext.Extract.html)
	
	内容移动 https://forum.poa.network/t/faq-fixing-gettext-error-in-blockscout/2444
	
- 如何修复索引超时

	BlockScout 使用两个独立索引器来索引网络历史并和上传新块。由于这个过程，节点可能会变得过载，并在制定的超时期限内无法响应 BlockScout 的 RPC 请求。
	
	可以调整索引器应用程序已满足节点大小和响应能力。
	
	提取器|描述|默认值
	---|---|---
	[Catchup Block Fetcher](https://github.com/poanetwork/blockscout/blob/1475e3bfd002d9397efd0f0cc29c20f39a70d023/apps/indexer/lib/indexer/block/catchup/fetcher.ex#L24)|这个提取器索引块、交易、收据，从链的顶端开始向后工作到创世块|批量大小10,并发10
	[Internal Transaction Fetcher](https://github.com/poanetwork/blockscout/blob/1475e3bfd002d9397efd0f0cc29c20f39a70d023/apps/indexer/lib/indexer/internal_transaction/fetcher.ex#L20)|这个提取器索引内部事务，对于实时提取器，此过程是同步完成的，而对于追赶提取器，此过程是异步完成的。|批量大小10,并发4
	[Coin Balance Fether](https://github.com/poanetwork/blockscout/blob/1475e3bfd002d9397efd0f0cc29c20f39a70d023/apps/indexer/lib/indexer/coin_balance/fetcher.ex#L22)|该提取器在地址与交互的块高度索引每个coin 余额|批量大小500,并发4
	[Uncle Block Fetcher](https://github.com/poanetwork/blockscout/blob/1475e3bfd002d9397efd0f0cc29c20f39a70d023/apps/indexer/lib/indexer/block/uncle/fetcher.ex#L22)|这个提取器索引非共识块、交易和数据|批量大小10,并发10
	[Token Balance Fetcher](https://github.com/poanetwork/blockscout/blob/1475e3bfd002d9397efd0f0cc29c20f39a70d023/apps/indexer/lib/indexer/token_balance/fetcher.ex#L29)|这个提取器索引代币余额。由于智能合约格式错误或其他不允许获取余额的功能，可能会遇到一些无法获取的代币余额|批量大小100,并发10
	[TokenFetcher](https://github.com/poanetwork/blockscout/blob/1475e3bfd002d9397efd0f0cc29c20f39a70d023/apps/indexer/lib/indexer/token/fetcher.ex#L18)|此索引获取代币合约的元数据|批量大小1:并发0
	
	- 了解错误和超时
	
			application=indexer fetcher=coin_balance count=500 error_count=500 [error] failed to fetch: :timeout
		在项目提供的错误中，可以看出 `索引器应用程序`由于 `超时` 错误而未能获取 `500 coin` 。通常，当索引器开始接收超时时，也会发生更多来自其他提取器的超时。
	- 解决超时

		当 fetcher 开始收到超时时，最好采取的措施是降低一些 fetcher 的批量大小和并发，这将减少对节点的压力。
		
		对节点造成最大压力的2个索引器是块索引器和内部事务提取器。建议这些 fetcher 值减半并重新启动应用程序。可能还需要重启节点。
	- 其他行为

		BlockScout 配置了 `n` 在检测到超时时自动停止发送请求几秒钟的功能。可以通过设置不同的值来调整这些设置 `wait_per_timeout`,[查看](https://github.com/poanetwork/blockscout/blob/1475e3bfd002d9397efd0f0cc29c20f39a70d023/apps/ethereum_jsonrpc/config/config.exs#L3-L9)
- 如何更新内存消耗以修复索引器内存错误

	在索引阶段，由于尝试一次获取所有块内容的负载，许多获取过程是异步运行的。这些进程存储在内存中，以在每个异步提取器中定义的设定间隔进行提取。
	
	`Indexer.Memory.Monitor` 检查 BEAM 内存使用是否超过设定的限制(默认 1GIB),如果超过，他会要求注册为可收缩的内存最多的进程进行收缩。
	
	每分钟检查一次内存使用情况。如果到达软限制，可以收缩工作队列将减少一半的负载。卸载的负载将从数据库中恢复，与重新启动服务器时相同，因此重建工作队列更慢，但使用内存更少。
	
	如果所有队列处于最小大小，则无法回收更多内存并记录错误。
	
	- 未来工作

		如上述，未来的工作被输入到内存中以供销后处理。这些相同的进程被导入数据库以在服务器重新启动时进行检查，并重新输入到内存中进行处理。
	- 更新内存

		默认内存限制为 1GIB，在下面进行编辑
		
		https://github.com/poanetwork/blockscout/blob/b444ac26bd7738c1ccee0eb8c382ef2b34b7ef24/apps/indexer/config/config.exs#L12
		
		此配置使用 Bitwise,在例子中，它计算算数左移结果。推荐的用于索引以太坊主网最小内存为 30GIB
	- 左位移换算表

		要执行左位移位转换，请打开交互 shell 
		
		- 1. `iex` 
		- 2. `import Bitwise` 
		- 3. `1 <<< 3 //1073741824`

		左位移|字节|GIB
		---|---|---
		1 <<< 30|1073741824|1
		5 <<< 30|5368709120|5.3
		10 <<< 30|10737418240|10.7
		15 <<< 30|16106127360|16.1
		20 <<< 30|21474836480|21.4
		25 <<< 30|26843545600|26.8
		30 <<< 30|32212254720|32.2
		35 <<< 30|37580963840|37.6
		40 <<< 30|42949672960|43
		45 <<< 30|48318382080|48.3	
		50 <<< 30|53687091200|53.7
- 如何更新页眉页脚中的菜单链接

	在 block_scout_web 目录中的 config.exs 文件中更新菜单。更新会延续到页眉页脚链接

	https://github.com/poanetwork/blockscout/blob/d164c35e02a1c75b94d93129422485245d22120d/apps/block_scout_web/config/config.exs#L24-L71
- 如果禁用汇率？

	将 Exploer.ExchangeRates 变量从更改 enabled:true 改为 enbled:false
	
	- 配置 explorer Explorer.ExchangeRates
		- enabled:false
		- store:ets

	https://github.com/poanetwork/blockscout/blob/7aeecb2a04838cda0289a7fe432db74481cf575a/apps/explorer/config/config.exs#L32
	
## 高级功能
### 自定义/品牌主题
自定义主题创建一致的品牌，帮助客户游览器交易和其他链上信息，并让客户与应用程序保持互动。

- [黑色森林](https://blockscout.com/xdai/mainnet/0x678ACb78948Be7F354B28DaAb79B1ABD81574c1B/transactions)
- [圈子 UBI](https://blockscout.com/xdai/mainnet/tx/0x497970f09839f31a0e98ef24cad93df06afe6d264eea68380675e357358da600/token-transfers)

### BlockScout.com 上的托管实例
在 BlockScout.com 上托管您的链

在 [BlockScout.com](http://blockscout.com/) 	上托管的链对每天访问该网站的用户来说是高度可见的。此外，在上面托管的链可以获得专门的技术支持，包括

- 品牌 ui 设置
- 稳定版本的自动更新
- 特定链参数定制，例如市场、验证者名称等
- Blockscout 核心团队的专门支持

### moonbeam(月光) 支持
在 Polkadot 上支持与以太坊兼容的合约平行链
以下高级定制

- moonbeam 平行链 Blockscout 实例的自定义样式
- 支持 moonbeam 的 Ethereum RPC 

### 导出 CSV
导出 CSV 是一项高级功能，为用户提供了一种轻松下载和分析区块链数据的方法，可用于地址页面上的选项卡，包括交易、内部交易、token和日志

注意, 3.5.1 之前的版本简化了 csv 导出，仅限交易和传输，无法定义时段。从 3.5.2后，增强了高级功能

在交易的 tracsactions 选项卡相关实例列表下可以找到 csv 按钮

注意，在 token 选项导出 token 转移列表

单击按钮后，您将被转到给定地址和导出交易类型（交易、内部交易、token转移、日志）的 csv 导出页面
		
				
		


			