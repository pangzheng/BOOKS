# web3.js
web3.js 是一组用来和本地或远程以太坊节点进行交互的 js 库，它可以使用 HTTP 或 IPC 建立与以太坊节点旳连接。

本文档是 web3.js 1.0 的API参考手册，其中每个 API 都包含有示例代码。

## web3
web3 是顶层包，它包含了所有以太坊相关的模块。

	var Web3 = require('web3');
	
	> Web3.utils
	> Web3.version
	> Web3.modules
	
	// 在支持以太坊的浏览器中，Web3.providers.givenProvider 将被自动设置
	var web3 = new Web3(Web3.givenProvider || 'ws://some.local-or-remote.node:8546');
	
	> web3.eth
	> web3.shh
	> web3.bzz
	> web3.utils
	> web3.version
### web3.eth
web3-eth 这个包用来和以太坊区块链与智能合约交互。

	var Eth = require('web3-eth');
	
	// "Eth.providers.givenProvider" 将在以太坊支持的浏览器中设置。
	var eth = new Eth(Eth.givenProvider || 'ws://some.local-or-remote.node:8546');
	
	
	// 或者使用 web3 雨伞套装
	var Web3 = require('web3');
	var web3 = new Web3(Web3.givenProvider || 'ws://some.local-or-remote.node:8546');
	
	// -> web3.eth
- web3.eth.subscribe

	web3.eth.subscribe 函数系列的作用是订阅区块链上的事件。
- web3.eth.Contract

	web3.eth.Contract 对象简化了与以太坊区块链上合约的交互。 合约对象的 json 接口与相应的智能合约相匹配，web3 会自动帮你将所有的调用转化为底层的基于 RPC 的 ABI 调用。

	这使得与智能合约的交互和操作 JavaScript 对象一样简单。
- web3.eth.accounts
	
	web3.eth.accounts 包含了与以太坊账户创建、交易和数据签名相关的函数。

	可以单独使用这个包：

		var Accounts = require('web3-eth-accounts');
		
		//  for accounts.signTransaction() 传入 eth 或 web3 包是必要的，以允许自动检索 chainId, gasPrice 和 nonce
		var accounts = new Accounts('ws://localhost:8546');
- web3.eth.personal

	web3-eth-personal 包的作用是与以太坊节点上的账户进行交互。

		var Personal = require('web3-eth-personal');
		
		// 在支持以太坊的浏览器中，`Personal.providers.givenProvider`将被自动设置
		var personal = new Personal(Personal.givenProvider || 'ws://some.local-or-remote.node:8546');
		
		// 也可以通过 web3 顶层包使用
		
		var Web3 = require('web3');
		var web3 = new Web3(Web3.givenProvider || 'ws://some.local-or-remote.node:8546');
		
		// -> web3.eth.personal
- web3.eth.Iban

	web3.eth.Iban 函数系列用于在以太坊地址、IBAN/BBAN 地址之间进行转换。
- web3.eth.abi

	web3.eth.abi 函数系列用于将函数调用编码为 ABI 格式，或者反之。

### web3.*.net
web3-net 包用来访问以太坊节点旳网络属性。

	var Net = require('web3-net');
	
	// 在支持以太坊的浏览器中，Personal.providers.givenProvider 被自动赋值
	var net = new Net(Net.givenProvider || 'ws://some.local-or-remote.node:8546');
	
	// 也可以使用web3顶层报
	
	var Web3 = require('web3');
	var web3 = new Web3(Web3.givenProvider || 'ws://some.local-or-remote.node:8546');
	
	// -> web3.eth.net
	// -> web3.bzz.net
	// -> web3.shh.net
### web3.bzz
web3-bzz 包用来与去中心化文件存储系统 swarm 进行交互。

	var Bzz = require('web3-bzz');
	
	// 可以自动检测 ethereum 对象是否存在，将连接到本地节点或 swarm-gateways.net
	// 可选地，可以指定一个服务提供器 URL，当没有指定服务提供器 UR 时，将使用 http://swarm-gateways.net
	var bzz = new Bzz(Bzz.givenProvider || 'http://swarm-gateways.net');
	
	// 也可以通过 web3 顶层包使用
	
	var Web3 = require('web3');
	var web3 = new Web3(Web3.givenProvider || 'ws://some.local-or-remote.node:8546');
	
	// -> web3.bzz.currentProvider 
	// 如果 Web3.givenProvider 时一个以太坊服务提供器，该属性将被赋值为 http://localhost:8500 ，
	//否则被赋值为 http://swarm-gateways.net
	
	// 必要时可以手工设置服务提供器
	web3.bzz.setProvider("http://localhost:8500");
### web3.shh
web3-shh 包用来与 whisper 协议进行交互，以便进行广播。

	var Shh = require('web3-shh');
	
	// `Shh.providers.givenProvider`在支持以太坊的浏览器中将被自动赋值
	var shh = new Shh(Shh.givenProvider || 'ws://some.local-or-remote.node:8546');
	
	// 也可以通过web3顶层包使用
	
	var Web3 = require('web3');
	var web3 = new Web3(Web3.givenProvider || 'ws://some.local-or-remote.node:8546');
	
	// -> web3.shh
### web3.utils
这个包提供以太坊 Dapp 开发相关的工具函数等。
### web3.version – 版本信息
web3.version属性记录了web3容器对象的版本

- 调用方法：

		Web3.version
		web3.version
- 返回值：

	String: 当前版本字符串
- 示例代码：

		web3.version;
		> "1.0.0"

### web3.modules – 查询子模块集合对象
web3.modules 属性返回一个包含所有子模块类的对象，可以用来手工实例化这些子模块类。

- 调用方法：

		Web3.modules
		web3.modules
- 返回值：

	Object: 子模块列表:

	- Eth – Function: Eth模块类，用来与以太坊网络进行交互。参见web3.eth。
	- Net – Function: Net模块类，用来与网络属性进行交互。参见web3.eth.net。
	- Personal – Function: Personal模块类，用来与以太坊账户进行交互。参见web3.eth.personal。
	- Shh – Function: Shh模块类，用来与whisper协议交互。参见web3.shh。
	- Bzz – Function: Bzz模块类，用来与swarm网络交互。参见web3.bzz。
- 示例代码：

		web3.modules
		> {
		    Eth: Eth function(provider),
		    Net: Net function(provider),
		    Personal: Personal function(provider),
		    Shh: Shh function(provider),
		    Bzz: Bzz function(provider),
		}

### web3.setProvider – 设置服务提供器
web3.setProvider() 方法用来修改指定模块的底层通讯服务提供器。

- 调用：

		web3.setProvider(myProvider)
		web3.eth.setProvider(myProvider)
		web3.shh.setProvider(myProvider)
		web3.bzz.setProvider(myProvider)
		...

	注意：当在 web3 上直接调用 setProvider() 方法时，将为所有其他子模块设置服务提供器，例如 web3.eth 和 web3.shh。但 web3.bzz 不受影响，因为该子模块始终使用独立的服务提供器。
- 参数
	
		Object – myProvider: 有效的服务提供器对象。
- 返回值
	
		Boolean
- 示例代码：

		var Web3 = require('web3');
		var web3 = new Web3('http://localhost:8545');
		// 或者
		var web3 = new Web3(new Web3.providers.HttpProvider('http://localhost:8545'));
		
		// 改变服务提供器
		web3.setProvider('ws://localhost:8546');
		// 或者
		web3.setProvider(new Web3.providers.WebsocketProvider('ws://localhost:8546'));
		
		// 在 node.js 中使用 IPC 服务提供器
		var net = require('net');
		var web3 = new Web3('/Users/myuser/Library/Ethereum/geth.ipc', net); // mac os 路径
		
		// 或者
		
		var web3 = new Web3(new Web3.providers.IpcProvider('/Users/myuser/Library/Ethereum/geth.ipc', net)); // mac os path
		// 在windows下的路径是: "\\\\.\\pipe\\geth.ipc"
		// 在linux下的路径是: "/users/myuser/.ethereum/geth.ipc"

### web3.providers – 服务提供器集合对象
返回当前有效的通信服务提供器。

- 调用：

		web3.providers
		web3.eth.providers
		web3.shh.providers
		web3.bzz.providers
		...
- 返回值：

	Object， 参见以下服务提供器对象:

	- Object – HttpProvider: HTTP服务提供器已经被弃用，因为它不支持订阅。
	- Object – WebsocketProvider: Websocket服务提供器是用于传统的浏览器中的标准方法。
	- Object – IpcProvider: 当运行一个本地节点时，IPC服务提供器用于node.js下的DApp环境，该方法提供最安全的连接。
- 示例代码：

		var Web3 = require('web3');
		// 使用指定的服务提供器（例如在Mist中）或实例化一个新的websocket提供器
		var web3 = new Web3(Web3.givenProvider || 'ws://remotenode.com:8546');
		// 或者
		var web3 = new Web3(Web3.givenProvider || new Web3.providers.WebsocketProvider('ws://remotenode.com:8546'));
		
		// 在node.js中使用IPC服务提供器
		var net = require('net');
		
		var web3 = new Web3('/Users/myuser/Library/Ethereum/geth.ipc', net); // mac os 路径
		// 或者
		var web3 = new Web3(new Web3.providers.IpcProvider('/Users/myuser/Library/Ethereum/geth.ipc', net)); // mac os 路径
		// windows路径是: "\\\\.\\pipe\\geth.ipc"
		// linux路径是: "/users/myuser/.ethereum/geth.ipc"

### web3.givenProvider – 原生服务提供器
在以太坊兼容的浏览器中使用 web3.js 时，web3.givenProvider 属性将返回浏览器设置的原生服务提供器，否则返回 null。

- 调用：

		web3.givenProvider
		web3.eth.givenProvider
		web3.shh.givenProvider
		web3.bzz.givenProvider
		...
- 返回值：

	Object: 浏览器设置好的提供器，或者null;

### web3.currentProvider – 当前服务提供器
web3.currentProvider属性返回当前在用的通信服务提供器，如果没有的话则返回null。

- 调用：

		web3.currentProvider
		web3.eth.currentProvider
		web3.shh.currentProvider
		web3.bzz.currentProvider
		...
- 返回值：

	Object： 当前在用的服务提供器，或者 null

### web3.BatchRequest – 批量请求
web3.BatchRequest 类用来创建并执行批请求。

- 调用：

		new web3.BatchRequest()
		new web3.eth.BatchRequest()
		new web3.shh.BatchRequest()
		new web3.bzz.BatchRequest()
- 参数：

	无
- 返回值：

	一个对象，具有如下方法：

	- add(request): 将请求对象添加到批调用中
	- execute(): 执行批请求
- 示例代码：

		var contract = new web3.eth.Contract(abi, address);
		
		var batch = new web3.BatchRequest();
		batch.add(web3.eth.getBalance.request('0x0000000000000000000000000000000000000000', 'latest', callback));
		batch.add(contract.methods.balance(address).call.request({from: '0x0000000000000000000000000000000000000000'}, callback2));
		batch.execute();

### web3.extend – 模块继承
使用 web3.extend() 方法来继承扩展 web3 的模块类。

- 调用：

		web3.extend(methods)
		web3.eth.extend(methods)
		web3.shh.extend(methods)
		web3.bzz.extend(methods)
		...
- 参数：
	- methods – Object: 扩展对象，包含一组如下的方法描述对象：
		- property – String: 可选，要添加到模块上的属性名称。 如果没有设置属性，则直接添加到模块上。
		- methods – Array: 方法描述对象数组，每个描述对象包含以下字段：
			- name – String: 要添加的方法名称
			- call – String: RPC方法名称
			- params – Number: 可选，方法的参数个数，默认值为0
			- inputFormatter – Array: 可选，输入格式化函数数组，每个成员对应一个函数参数，或者使用null来对应不需要进行格式化处理的参数
			- outputFormatter – Function: 可选，用来格式化输出
- 返回值:

	Object: 扩展后的模块
- 示例代码：

		web3.extend({
		    property: 'myModule',
		    methods: [{
		        name: 'getBalance',
		        call: 'eth_getBalance',
		        params: 2,
		        inputFormatter: [web3.extend.formatters.inputAddressFormatter, web3.extend.formatters.inputDefaultBlockNumberFormatter],
		        outputFormatter: web3.utils.hexToNumberString
		    },{
		        name: 'getGasPriceSuperFunction',
		        call: 'eth_gasPriceSuper',
		        params: 2,
		        inputFormatter: [null, web3.utils.numberToHex]
		    }]
		});
		
		web3.extend({
		    methods: [{
		        name: 'directCall',
		        call: 'eth_callForFun',
		    }]
		});
		
		console.log(web3);
		> Web3 {
		    myModule: {
		        getBalance: function(){},
		        getGasPriceSuperFunction: function(){}
		    },
		    directCall: function(){},
		    eth: Eth {...},
		    bzz: Bzz {...},
		    ...
		}
### web3.eth 下参数
#### web3.eth.defaultAccount – 默认账户
web3.eth.defaultAccount 属性记录了默认地址，在以下方法中如果没有指定 from 属性，将使用该属性的值作为默认的 from 属性值。

		web3.eth.sendTransaction()
		web3.eth.call()
		new web3.eth.Contract() -> myContract.methods.myMethod().call()
		new web3.eth.Contract() -> myContract.methods.myMethod().send()
- 调用

		web3.eth.defaultAccount
- 属性
	- String – 20 Bytes: 以太坊地址，你应当在节点或keystore中存有该地址的私钥。默认值为undefined
- 示例代码

		web3.eth.defaultAccount;
		> undefined
		
		// set the default account
		web3.eth.defaultAccount = '0x11f4d0A3c12e86B4b5F39B213F7E19D048276DAe';

#### web3.eth.defaultBlock – 默认块
web3.eth.defaultBlock 属性记录默认块，默认值为 latest。该属性值用于以下方法调用：

	web3.eth.getBalance()
	web3.eth.getCode()
	web3.eth.getTransactionCount()
	web3.eth.getStorageAt()
	web3.eth.call()
	new web3.eth.Contract() -> myContract.methods.myMethod().call()
在以上方法中，可以传入 defaultBlock 参数来覆盖该属性值。

- 调用：

		web3.eth.defaultBlock
- 属性

	默认块参数的值为以下列表中之一：

	- Number: 一个具体的块编号
	- "genesis" – String: 创世块
	- "latest" – String: 最后一个块，即当前的链头块
	- "pending" – String: 当前挖掘中的块，其中包含挂起的交易
	
	默认值为"latest"
- 示例代码：

		web3.eth.defaultBlock;
		> "latest"
		
		// 设置默认块属性
		web3.eth.defaultBlock = 231;

#### web3.eth.getProtocolVersion – 返回协议版本信息
返回节点旳以太坊协议版本

- 调用

		web3.eth.getProtocolVersion([callback])
- 返回值

	一个 Promise 对象，其解析值为协议版本字符串
- 示例代码：

		web3.eth.getProtocolVersion().then(console.log);
		> "63"

#### web3.eth.isSyncing – 检查节点是否同步
web3.eth.isSyncing() 方法用来检查节点当前是否已经与网络同步。

- 调用

		web3.eth.isSyncing([callback])
- 返回值

	一个 Promise 对象，其解析值为 Object 或 Boolean。
	
	- 如果节点尚未与网络同步，则返回 false
	- 否则返回一个同步对象，具有以下属性：
		- startingBlock – Number: 同步起始块编号
		- currentBlock – Number: 当前已同步块编号
		- highestBlock – Number: 预估的目标同步块编号
		- knownStates – Number: 预估的要下载的状态
		- pulledStates – Number: 已经下载的状态
- 示例代码：

		web3.eth.isSyncing().then(console.log);
		> {
		    startingBlock: 100,
		    currentBlock: 312,
		    highestBlock: 512,
		    knownStates: 234566,
		    pulledStates: 123455
		}
		
#### web3.eth.getCoinbase – 返回挖矿收益账户
使用 web3.eth.getCoinbase() 方法获取当前接收挖矿奖励的账户地址。

- 调用

		web3.eth.getCoinbase([callback])
- 返回值

		一个 Promise 对象，其解析值为接收挖矿奖励的账户地址字符串，20字节长。
- 示例代码

		web3.eth.getCoinbase().then(console.log);
		> "0x11f4d0A3c12e86B4b5F39B213F7E19D048276DAe"

#### web3.eth.isMining – 检查节点是否在挖矿
调用 web3.eth.isMining() 方法来检查节点是否在进行挖矿

- 调用：

		web3.eth.isMining([callback])
- 返回值：

	一个Promise对象，挖矿时其解析值为true，否则为false。
- 示例代码：

		web3.eth.isMining().then(console.log);
		> true						

#### web3.eth.getHashrate – 返回节点旳哈希计算速度
web3.eth.getHashrate() 方法用来读取当前挖矿节点的每秒钟哈希值算出数量。

- 调用：

		web3.eth.getHashrate([callback])
- 返回值：

	一个Promise对象，其解析值为Number类型，表示每秒哈希值算出数量。
- 示例代码：

		web3.eth.getHashrate().then(console.log);
		> 493736

#### web3.eth.getGasPrice – 返回当前 gas 价格
web3.eth.getGasPrice() 方法用来获取当前 gas 价格，该价格由最近的若干块的 gas 价格中值决定。

- 调用

		web3.eth.getGasPrice([callback])
- 返回值

	一个Promise对象，其解析值为表示当前gas价格的字符串，单位为wei。
- 示例代码

		web3.eth.getGasPrice().then(console.log);
		> "20000000000"	

#### web3.eth.getAccounts – 返回账户列表
web3.eth.getAccounts() 方法返回当前节点控制的账户列表。

- 调用

		web3.eth.getAccounts([callback])
- 返回值

	一个Promise对象，其解析值为账户地址数组。
- 示例代码

		web3.eth.getAccounts().then(console.log);
		> ["0x11f4d0A3c12e86B4b5F39B213F7E19D048276DAe", "0xDCc6960376d6C6dEa93647383FfB245C

#### web3.eth.getBlockNumber – 返回当前块编号
web3.eth.getBlockNumber() 方法返回当前块编号。

- 调用：

		web3.eth.getBlockNumber([callback])
- 返回值：

	一个Promise对象，其解析值为最近一个块的编号，Number类型。
- 示例代码：

		web3.eth.getBlockNumber().then(console.log);
		> 2744

#### web3.eth.getBalance – 返回指定账户余额
web3.eth.getBalance() 方法用来获取指定块中特定账户地址的余额。

- 调用：

		web3.eth.getBalance(address [, defaultBlock] [, callback])
- 参数：
	- address：String – 要检查余额的账户地址
	- defaultBlock：Number|String – 可选，使用该参数覆盖 web3.eth.defaultBlock 属性值
	- callback：Function – 可选的回调函数，该回调的第一个参数为 error 对象，第二个参数为结果值
- 返回值：

	一个Promise对象，其解析值为指定账户地址的余额字符串，以wei为单位。
- 示例代码：

		web3.eth.getBalance("0x407d73d8a49eeb85d32cf465507dd71d507100c1")
		.then(console.log);
		> "1000000000000"

#### web3.eth.getStorageAt – 返回指定地址存储内容
web3.eth.getStorageAt() 方法返回一个以太坊地址的指定位置存储内容。

- 调用：

		web3.eth.getStorageAt(address, position [, defaultBlock] [, callback])
- 参数：
	- address：String – 要读取的地址
	- position：Number – 存储中的索引编号
	- defaultBlock：Number|String – 可选，使用该参数覆盖web3.eth.defaultBlock属性值
	- callback：Function – 可选的回调函数, 其第一个参数为错误对象，第二个参数为结果。
- 返回值：

	一个Promise对象，其解析值为存储中指定位置的内容。
- 示例代码：

		web3.eth.getStorageAt("0x407d73d8a49eeb85d32cf465507dd71d507100c1", 0)
		.then(console.log);
		> "0x033456732123ffff2342342dd12342434324234234fd234fd23fd4f23d4234"
#### web3.eth.getCode – 返回指定地址的代码
web3.eth.getCode() 方法返回指定以太坊地址处的代码。

- 调用：

		web3.eth.getCode(address [, defaultBlock] [, callback])
- 参数：
	- address：String – 要读取代码的地址
	- defaultBlock：Number|String – 可选，使用该参数覆盖web3.eth.defaultBlock属性值
	- callback：Function – 可选的回调函数，其第一个参数为错误对象，第二个参数为结果
- 返回值：

	一个Promise对象，其解析值为指定地址处的代码字符串。
- 示例代码：

		web3.eth.getCode("0xd5677cf67b5aa051bb40496e68ad359eb97cfbf8")
		.then(console.log);
		> "0x600160008035811a818181146012578301005b601b6001356025565b8060005260206000f25b6

#### web3.eth.getBlock – 返回指定块
web3.eth.getBlock() 方法返回指定块编号或块哈希对应的块。

- 调用：

		web3.eth.getBlock(blockHashOrBlockNumber [, returnTransactionObjects] [, callback])
- 参数：
	- blockHashOrBlockNumber：String|Number – 块编号或块哈希值，或者使用以下字符串："genesis"、"latest" 或 "pending" 。
	- returnTransactionObjects：Boolean – 可选，默认值为false。当设置为true时,返回块中将包括所有交易详情，否则仅返回交易哈希。
	- callback：Function – 可选的回调函数，其第一个参数为错误对象，第二个参数为结果。
- 返回值：

	一个Promise对象，其解析值为满足搜索条件的块对象，具有以下字段：

	- number – Number: 块编号，处于pending状态的块为null
	- hash 32 Bytes – String: 块哈希，处于pending状态的块为null
	- parentHash 32 Bytes – String: 父块哈希
	- nonce 8 Bytes – String: 生成的proof-of-work的哈希，处于pending状态的块为null
	- sha3Uncles 32 Bytes – String: 块中叔伯数据的SHA3值
	- logsBloom 256 Bytes – String: 块中日志的bloom filter，处于pending状态的块为null
	- transactionsRoot 32 Bytes – String: 块中的交易树根节点
	- stateRoot 32 Bytes – String: 块中的最终状态树根节点
	- miner – String: 接收奖励的矿工地址
	- difficulty – String: 该块的难度值
	- totalDifficulty – String: 截至该块的全链总难度值
	- extraData – String: 块 “extra data” 字段
	- size – Number: 字节为单位的块大小
	- gasLimit – Number: 该块允许的最大gas值
	- gasUsed – Number: 该块中所有交易使用的gas总量
	- timestamp – Number: 出块的unix时间戳
	- transactions – Array: 交易对象数组，或者32字节长的交易哈希值，取决于returnTransactionObjects的设置
	- uncles – Array: 叔伯块哈希值数组
- 示例代码：

		web3.eth.getBlock(3150)
		.then(console.log);
		
		> {
		    "number": 3,
		    "hash": "0xef95f2f1ed3ca60b048b4bf67cde2195961e0bba6f70bcbea9a2c4e133e34b46",
		    "parentHash": "0x2302e1c0b972d00932deb5dab9eb2982f570597d9d42504c05d9c2147eaf9c88",
		    "nonce": "0xfb6e1a62d119228b",
		    "sha3Uncles": "0x1dcc4de8dec75d7aab85b567b6ccd41ad312451b948a7413f0a142fd40d49347",
		    "logsBloom": "0x00000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000",
		    "transactionsRoot": "0x3a1b03875115b79539e5bd33fb00d8f7b7cd61929d5a3c574f507b8acf415bee",
		    "stateRoot": "0xf1133199d44695dfa8fd1bcfe424d82854b5cebef75bddd7e40ea94cda515bcb",
		    "miner": "0x8888f1f195afa192cfee860698584c030f4c9db1",
		    "difficulty": '21345678965432',
		    "totalDifficulty": '324567845321',
		    "size": 616,
		    "extraData": "0x",
		    "gasLimit": 3141592,
		    "gasUsed": 21662,
		    "timestamp": 1429287689,
		    "transactions": [
		        "0x9fc76417374aa880d4449a1f7f31ec597f00b1f6f3dd2d66f4c9c6c445836d8b"
		    ],
		    "uncles": []
		}

#### web3.eth.getBlockTransactionCount – 返回指定块中的交易数量
web3.eth.getBlockTransactionCount() 方法返回指定块中的交易数量。

- 调用：

		web3.eth.getBlockTransactionCount(blockHashOrBlockNumber [, callback])
- 参数：
	- blockHashOrBlockNumber：String|Number – 块编号或块的哈希值，或者使用以下字符串：
		- "genesis"
		- "latest"
		- "pending" 
	- callback：Function – 可选的回调函数，其第一个参数为错误对象，第二个参数为执行结果。
- 返回值：

	一个 Promise 对象，其解析值为指定块中的交易数量，Number 类型。
- 示例代码：

		web3.eth.getBlockTransactionCount("0x407d73d8a49eeb85d32cf465507dd71d507100c1")
		.then(console.log);
		> 1

#### web3.eth.getUncle – 返回指定叔伯块
web3.eth.getUncle()方法返回指定索引位置的叔伯块。

- 调用：

		web3.eth.getUncle(blockHashOrBlockNumber, uncleIndex [, returnTransactionObjects] [, callback])
- 参数：
	- blockHashOrBlockNumber：String|Number – 块编号或块的哈希值，或者使用以下字符串
		- "genesis"
		- "latest"
		- "pending" 
	- uncleIndex：Number – 叔伯块的索引位置。
	- returnTransactionObjects：Boolean – 可选，默认值为false。如果设置为true，那么返回的块信息中将包含全部交易的完整信息，否则仅包含交易的哈希值。
	- callback：Function – 可选的回调函数，其第一个参数为错误对象，第二个参数为结果。
- 返回值：

	一个 Promise 对象，其解析值为叔伯块对象，具体内容参见 web3.eth.getBlock()
- 注意

	叔伯块中不包含单独的交易信息。
- 示例代码：

		web3.eth.getUncle(500, 0)
		.then(console.log);
		> // 块对象字段说明参见  web3.eth.getBlock

#### web3.eth.getTransaction – 返回指定交易对象
web3.eth.getTransaction()方法返回具有指定哈希值的交易对象。

- 调用：

		web3.eth.getTransaction(transactionHash [, callback])
- 参数：
	- transactionHash：String – 交易的哈希值
	- callback：Function – 可选的回调函数，其第一个参数为错误对象，第二个参数为返回结果。
- 返回值：

	一个Promise对象，其解析值为具有给定哈希值的交易对象，该对象具有如下字段：
	
	- hash 32 Bytes – String: 交易的哈希值
	- nonce – Number: 交易发送方在此交易之前产生的交易数量
	- blockHash 32 Bytes – String: 交易所在块的哈希值。如果交易处于pending状态，则该值为null
	- blockNumber – Number: 交易所在块的编号，如果交易处于pending状态，则该值为null
	- transactionIndex – Number: 交易在块中的索引位置，如果交易处于pending状态，则该值为null
	- from – String: 交易发送方的地址
	- to – String: 交易接收方的地址。对于创建合约的交易，该值为null
	- value – String: 以wei为单位的转账金额
	- gasPrice – String: 发送方承诺的gas价格，以wei为单位
	- gas – Number: 发送方提供的gas用量
	- input – String: 随交易发送的数据
- 示例代码

		web3.eth.getTransaction('0x9fc76417374aa880d4449a1f7f31ec597f00b1f6f3dd2d66f4c9c6c445836d8b§234')
		.then(console.log);
		> {
		    "hash": "0x9fc76417374aa880d4449a1f7f31ec597f00b1f6f3dd2d66f4c9c6c445836d8b",
		    "nonce": 2,
		    "blockHash": "0xef95f2f1ed3ca60b048b4bf67cde2195961e0bba6f70bcbea9a2c4e133e34b46",
		    "blockNumber": 3,
		    "transactionIndex": 0,
		    "from": "0xa94f5374fce5edbc8e2a8697c15331677e6ebf0b",
		    "to": "0x6295ee1b4f6dd65047762f924ecd367c17eabf8f",
		    "value": '123450000000000000',
		    "gas": 314159,
		    "gasPrice": '2000000000000',
		    "input": "0x57cb2fc4"
		}

#### web3.eth.getTransactionFromBlock – 返回块中指定交易对象
web3.eth.getTransactionFromBlock() 方法返回指定块中特定索引号的交易对象。

- 调用：

		getTransactionFromBlock(hashStringOrNumber, indexNumber [, callback])
- 参数：
	- hashStringOrNumber：String – 块编号或块的哈希值，或者使用以下字符串："genesis、"latest" 或 "pending" 来指定块
	- indexNumber：Number – 交易索引位置
	- callback：Function – 可选的回调函数，其第一个参数为错误对象，第二个参数为结果交易对象
- 返回值：

	一个Promise对象，其解析值为交易对象，该对象具体内容描述参见web3.eth.getTransaction()
- 示例代码：

		var transaction = web3.eth.getTransactionFromBlock('0x4534534534', 2)
		.then(console.log);
		> // see web3.eth.getTransaction

#### web3.eth.getTransactionReceipt – 返回指定交易的收据
web3.eth.getTransactionReceipt() 方法返回指定交易的收据对象。如果交易处于pending状态，则返回null。

- 调用：

		web3.eth.getTransactionReceipt(hash [, callback])
- 参数：
	- hash：String – 交易的哈希值
	- callback：Function – 可选的回调函数，其第一个参数为错误对象，第二个参数为结果
- 返回值：

	一个Promise对象，其解析值为交易的收据对象或者null。收据对象具有如下字段：
	
	- status – Boolean: 成功的交易返回true，如果EVM回滚了该交易则返回false
	- blockHash 32 Bytes – String: 交易所在块的哈希值
	- blockNumber – Number: 交易所在块的编号
	- transactionHash 32 Bytes – String: 交易的哈希值
	- transactionIndex – Number: 交易在块中的索引位置
	- from – String: 交易发送方的地址
	- to – String: 交易接收方的地址，对于创建合约的交易，该值为null
	- contractAddress – String: 对于创建合约的交易，该值为创建的合约地址，否则为null
	- cumulativeGasUsed – Number: 该交易执行时所在块的gas累计总用量
	- gasUsed– Number: 该交易的gas总量
	- logs – Array: 该交易产生的日志对象数组
- 示例代码：

		var receipt = web3.eth.getTransactionReceipt('0x9fc76417374aa880d4449a1f7f31ec597f00b1f6f3dd2d66f4c9c6c445836d8b')
		.then(console.log);
		> {
		  "status": true,
		  "transactionHash": "0x9fc76417374aa880d4449a1f7f31ec597f00b1f6f3dd2d66f4c9c6c445836d8b",
		  "transactionIndex": 0,
		  "blockHash": "0xef95f2f1ed3ca60b048b4bf67cde2195961e0bba6f70bcbea9a2c4e133e34b46",
		  "blockNumber": 3,
		  "contractAddress": "0x11f4d0A3c12e86B4b5F39B213F7E19D048276DAe",
		  "cumulativeGasUsed": 314159,
		  "gasUsed": 30234,
		  "logs": [{
		         // logs as returned by getPastLogs, etc.
		     }, ...]
		}

#### web3.eth.getTransactionCount – 返回指定地址发生的交易数量
web3.eth.getTransactionCount() 方法返回指定地址发出的交易数量。

- 调用：

		web3.eth.getTransactionCount(address [, defaultBlock] [, callback])
- 参数：
	- address：String – 要查询的账户地址
	- defaultBlock：Number|String – 可选，设置该参数来覆盖web3.eth.defaultBlock属性值
	- callback：Function – 可选的回调函数，其第一个参数为错误对象，第二个参数为结果
- 返回值：

	一个Promise对象，其解析值为指定地址发出的交易数量。
- 示例代码：

		web3.eth.getTransactionCount("0x11f4d0A3c12e86B4b5F39B213F7E19D048276DAe")
		.then(console.log);
		> 1										

#### web3.eth.sendTransaction – 发送交易
web3.eth.sendTransaction()方法向以太坊网络提交一个交易。

- 调用：

		web3.eth.sendTransaction(transactionObject [, callback])
- 参数
	- transactionObject：Object – 要发送的交易对象，包含以下字段：
		- from – String|Number: 交易发送方账户地址，不设置该字段的话，则使用web3.eth.defaultAccount属性值。可设置为一个地址或本地钱包web3.eth.accounts.wallet中的索引序号
		- to – String: 可选，消息的目标地址，对于合约创建交易该字段为null
		- value – Number|String|BN|BigNumber: (optional) The value transferred for the transaction in wei, also the endowment if it’s a contract-creation transaction.
		- gas – Number: 可选，默认值：待定，用于交易的gas总量，未用完的gas会退还
		- gasPrice – Number|String|BN|BigNumber: 可选，该交易的gas价格，单位为wei，默认值为web3.eth.gasPrice属性值
		- data – String: 可选，可以是包含合约方法数据的ABI字符串，或者是合约创建交易中的初始化代码
		- nonce – Number: 可选，使用该字段覆盖使用相同nonce值的挂起交易
	- callback – Function: 可选的回调函数，其第一个参数为错误对象，第二个参数为结果
- 返回值：

	web3.eth.sendTransaction() 方法的返回值是32字节长的交易哈希值。

	PromiEvent: 一个整合事件发生器的 Promise 对象，将在收到交易收据后得到解析。

	- "transactionHash" 返回String: 在交易发出并得到有效的交易哈希值后立刻触发
	- "receipt" 返回Object: 当交易收据有效后立刻触发
	- "confirmation" 返回Number, Object: 在每次确认后立刻触发，最多12次确认。确认编号为第一个参数，收据为第二个参数。从0号确认开始触发
	- "error" 返回Error对象: 在发送交易的过程中如果出现错误则立刻触发。如果是out of gas错误，则传入第二个参数为交易收据
- 示例代码：

		// 使用 https://remix.ethereum.org 编译的稳固源代码
		var code = "603d80600c6000396000f3007c01000000000000000000000000000000000000000000000000000000006000350463c6888fa18114602d57005b6007600435028060005260206000f3";
		
		// 使用回调函数
		web3.eth.sendTransaction({
		    from: '0xde0B295669a9FD93d5F28D9Ec85E40f4cb697BAe',
		    data: code // 部署一个合约
		}, function(error, hash){
		    ...
		});
		
		// 使用 promise
		web3.eth.sendTransaction({
		    from: '0xde0B295669a9FD93d5F28D9Ec85E40f4cb697BAe',
		    to: '0x11f4d0A3c12e86B4b5F39B213F7E19D048276DAe',
		    value: '1000000000000000'
		})
		.then(function(receipt){
		    ...
		});
		
		
		// 使用事件发生器
		web3.eth.sendTransaction({
		    from: '0xde0B295669a9FD93d5F28D9Ec85E40f4cb697BAe',
		    to: '0x11f4d0A3c12e86B4b5F39B213F7E19D048276DAe',
		    value: '1000000000000000'
		})
		.on('transactionHash', function(hash){
		    ...
		})
		.on('receipt', function(receipt){
		    ...
		})
		.on('confirmation', function(confirmationNumber, receipt){ ... })
		.on('error', console.error); //如果出气错误，第二个参数是收据。

#### web3.eth.sendSignedTransaction – 发送已签名交易
web3.eth.sendSignedTransaction() 方法用来发送已经签名的交易，例如，可以使用 web3.eth.accounts.signTransaction() 方法进行签名。

- 调用：

		web3.eth.sendSignedTransaction(signedTransactionData [, callback])
- 参数：
	- signedTransactionData：String – 16进制格式的签名交易数据
	- callback：Function – 可选的回调函数，其第一个参数为错误对象，第二个参数为结果
- 返回值：

	PromiEvent: 一个整合了事件发生器的Promise对象。当交易收据生效后得到解析。
- 示例代码：

		var Tx = require('ethereumjs-tx');
		var privateKey = new Buffer('e331b6d69882b4cb4ea581d88e0b604039a3de5967688d3dcffdd2270c0fd109', 'hex')
		
		var rawTx = {
		  nonce: '0x00',
		  gasPrice: '0x09184e72a000',
		  gasLimit: '0x2710',
		  to: '0x0000000000000000000000000000000000000000',
		  value: '0x00',
		  data: '0x7f7465737432000000000000000000000000000000000000000000000000000000600057'
		}
		
		var tx = new Tx(rawTx);
		tx.sign(privateKey);
		
		var serializedTx = tx.serialize();
		
		// console.log(serializedTx.toString('hex'));
		// 0xf889808609184e72a00082271094000000000000000000000000000000000000000080a47f74657374320000000000000000000000000000000000000000000000000000006000571ca08a8bbf888cfa37bbf0bb965423625641fc956967b81d12e23709cead01446075a01ce999b56a8a88504be365442ea61239198e23d1fce7d00fcfc5cd3b44b7215f
		
		web3.eth.sendSignedTransaction('0x' + serializedTx.toString('hex'))
		.on('receipt', console.log);
		
		> // see eth.getTransactionReceipt() for details

#### web3.eth.sign – 为数据生成签名
web3.eth.sign() 方法使用指定的账户对数据进行签名，该账户必须先解锁。

- 调用：

	web3.eth.sign(dataToSign, address [, callback])
- 参数：
	- dataToSign：String – 待签名的数据。对于字符串将首先使用web3.utils.utf8ToHex()方法将其转换为16进制
	- address：String|Number – 用来签名的账户地址。或者本地钱包web3.eth.accounts.wallet中的地址或其序号
	- callback：Function – 可选的回调函数，其第一个参数为错误对象，第二个参数为结果
- 返回值：

	一个Promise对象，其解析值为签名结果字符串。
- 示例代码：

		web3.eth.sign("Hello world", "0x11f4d0A3c12e86B4b5F39B213F7E19D048276DAe")
		.then(console.log);
		> "0x30755ed65396facf86c53e6217c52b4daebe72aa4941d89635409de4c9c7f9466d4e9aaec7977f05e923889b33c0d0dd27d7226b6e6f56ce737465c5cfd04be400"
		
		// 相同
		web3.eth.sign(web3.utils.utf8ToHex("Hello world"), "0x11f4d0A3c12e86B4b5F39B213F7E19D048276DAe")
		.then(console.log);
		> "0x30755ed65396facf86c53e6217c52b4daebe72aa4941d89635409de4c9c7f9466d4e9aaec7977f05e923889b33c0d0dd27d7226b6e6f56ce737465c5cfd04be400"
		
#### web3.eth.signTransaction – 为交易生成签名
使用 web3.eth.signTransaction() 方法对交易进行签名，用来签名的账户地址需要首先解锁。

- 调用：

		web3.eth.signTransaction(transactionObject, address [, callback])
- 参数：
	- transactionObject：Object – 要签名的交易数据
	- address：String – 用于签名的账户地址
	- callback：Function – 可选的回调函数，其第一个参数为错误对象，第二个参数为结果
- 返回值：

	一个Promise对象，其解析值为RLP编码的交易对象。该对象的raw属性可以用来通过 web3.eth.sendSignedTransaction() 方法来发送交易。

- 示例代码：

		web3.eth.signTransaction({
		    from: "0xEB014f8c8B418Db6b45774c326A0E64C78914dC0",
		    gasPrice: "20000000000",
		    gas: "21000",
		    to: '0x3535353535353535353535353535353535353535',
		    value: "1000000000000000000",
		    data: ""
		}).then(console.log);
		> {
		    raw: '0xf86c808504a817c800825208943535353535353535353535353535353535353535880de0b6b3a76400008025a04f4c17305743700648bc4f6cd3038ec6f6af0df73e31757007b7f59df7bee88da07e1941b264348e80c78c4027afc65a87b0a5e43e86742b8ca0823584c6788fd0',
		    tx: {
		        nonce: '0x0',
		        gasPrice: '0x4a817c800',
		        gas: '0x5208',
		        to: '0x3535353535353535353535353535353535353535',
		        value: '0xde0b6b3a7640000',
		        input: '0x',
		        v: '0x25',
		        r: '0x4f4c17305743700648bc4f6cd3038ec6f6af0df73e31757007b7f59df7bee88d',
		        s: '0x7e1941b264348e80c78c4027afc65a87b0a5e43e86742b8ca0823584c6788fd0',
		        hash: '0xda3be87732110de6c1354c83770aae630ede9ac308d9f7b399ecfba23d923384'
		    }
		}										

#### web3.eth.call – 执行消息调用
使用 web3.eth.call() 方法执行一个消息调用交易，消息调用交易直接在节点旳 VM 中执行而不需要通过区块链的挖矿来执行。

- 调用：

		web3.eth.call(callObject [, defaultBlock] [, callback])
- 参数：
	- callObject：Object – 交易对象，消息调用交易的from属性可选
	- defaultBlock：Number|String – 可选，使用该参数来覆盖默认的web3.eth.defaultBlock属性值
	- callback：Function – 可选的回调函数，其第一个参数为错误对象，第二个参数为结果
- 返回值：

	一个Promise对象，其解析值为调用方法的返回数据字符串
- 示例代码：

		web3.eth.call({
		    to: "0x11f4d0A3c12e86B4b5F39B213F7E19D048276DAe", // contract address
		    data: "0xc6888fa10000000000000000000000000000000000000000000000000000000000000003"
		})
		.then(console.log);
		> "0x000000000000000000000000000000000000000000000000000000000000000a"
		
#### web3.eth.estimateGas – 估算gas用量
web3.eth.estimateGas() 方法通过执行一个消息调用来估算交易的 gas 用量。

- 调用：

		web3.eth.estimateGas(callObject [, callback])
- 参数：
	- callObject：Object – 交易对象，其from属性可选
	- callback：Function – 可选的回调函数，其第一个参数为错误对象，第二个参数为结果
- 返回值：

	一个Promise对象，其解析值为模拟调用的gas用量。
- 示例代码：

		web3.eth.estimateGas({
		    to: "0x11f4d0A3c12e86B4b5F39B213F7E19D048276DAe",
		    data: "0xc6888fa10000000000000000000000000000000000000000000000000000000000000003"
		})
		.then(console.log);
		> "0x0000000000000000000000000000000000000000000000000000000000000015"

#### web3.eth.getPastLogs – 返回历史日志
web3.eth.getPastLogs() 方法根据指定的选项返回历史日志。

- 调用：

		web3.eth.getPastLogs(options [, callback])
- 参数：
	- options：Object – 过滤器对象，包含如下字段：
		- romBlock – Number|String: 最新的块的编号(“latest”可以表示最近的和“pending”的当前挖掘块)。默认情况下“latest”。
		- toBlock – Number|String: 最新块的编号(“latest”可以表示最近的和“pending”的正在挖掘的块)。默认情况下“最新”。
		- address – String|Array: 一个地址或地址列表，仅用于从特定帐户获取日志。
		- topics – Array: 一组值，每个值必须出现在日志项中。顺序是很重要的，如果你想要保留主题使用 null，例如 [null， '0x12…']。您也可以为每个主题传递一个数组，该主题的选项，如 [null， ['option1'， 'option2']]
- 返回值：

	一个 Promise 对象，其解析值为日志对象数组。数组中的事件对象结构如下：

	- address – String: 事件发生源地址
	- data – String: 包含未索引的日志参数
	- topics – Array: 包含最多4个32字节主题的数组，主题1-3包含日志的索引参数
	- logIndex – Number: 事件在块中的索引位置
	- transactionIndex – Number: 包含事件的交易的索引位置
	- transactionHash 32 Bytes – String: 包含事件的交易的哈希值
	- blockHash 32 Bytes – String: 包含事件的块的哈希值，如果处于pending状态，则为null
	- blockNumber – Number: 包含事件的块编号，处于pending状态时该字段为null
- 示例代码：

		web3.eth.getPastLogs({
		    address: "0x11f4d0A3c12e86B4b5F39B213F7E19D048276DAe",
		    topics: ["0x033456732123ffff2342342dd12342434324234234fd234fd23fd4f23d4234"]
		})
		.then(console.log);
		
		> [{
		    data: '0x7f9fade1c0d57a7af66ab4ead79fade1c0d57a7af66ab4ead7c2c2eb7b11a91385',
		    topics: ['0xfd43ade1c09fade1c0d57a7af66ab4ead7c2c2eb7b11a91ffdd57a7af66ab4ead7', '0x7f9fade1c0d57a7af66ab4ead79fade1c0d57a7af66ab4ead7c2c2eb7b11a91385']
		    logIndex: 0,
		    transactionIndex: 0,
		    transactionHash: '0x7f9fade1c0d57a7af66ab4ead79fade1c0d57a7af66ab4ead7c2c2eb7b11a91385',
		    blockHash: '0xfd43ade1c09fade1c0d57a7af66ab4ead7c2c2eb7b11a91ffdd57a7af66ab4ead7',
		    blockNumber: 1234,
		    address: '0xde0B295669a9FD93d5F28D9Ec85E40f4cb697BAe'
		},{...}]
		
#### web3.eth.getCompilers – 返回可用编译器清单
web3.eth.getCompilers() 方法返回可用编译器的列表。

- 调用：

		web3.eth.getCompilers([callback])
- 参数：

		callback：Function – 可选的回调函数，其第一个参数为错误对象，第二个参数为结果。
- 返回值：

	一个Promise对象，其解析值为可用编译器名称字符串数组。
- 示例代码：

		web3.eth.getCompilers()
		.then(console.log);
		> ["lll", "solidity", "serpent"]				

#### web3.eth.compile.solidity – 编译solidity代码
web3.eth.compile.solidity() 方法用来编译使用 solidity 语言编写的合约源代码。

- 调用：

		web3.eth.compile.solidity(sourceCode [, callback])
- 参数：
	- sourceCode：String – solidity 源代码字符串
	- callback：Function – 可选的回调函数，其第一个参数为错误对象，第二个参数为结果
- 返回值：

	一个 Promise 对象，其解析值为编译结果信息对象。
- 示例代码：

		var source = "" +
		    "contract test {\n" +
		    "   function multiply(uint a) returns(uint d) {\n" +
		    "       return a * 7;\n" +
		    "   }\n" +
		    "}\n";
		
		web3.eth.compile.solidity(source)
		.then(console.log);
		
		> {
		  "test": {
		    "code": "0x605280600c6000396000f3006000357c010000000000000000000000000000000000000000000000000000000090048063c6888fa114602e57005b60376004356041565b8060005260206000f35b6000600782029050604d565b91905056",
		    "info": {
		      "source": "contract test {\n\tfunction multiply(uint a) returns(uint d) {\n\t\treturn a * 7;\n\t}\n}\n",
		      "language": "Solidity",
		      "languageVersion": "0",
		      "compilerVersion": "0.8.2",
		      "abiDefinition": [
		        {
		          "constant": false,
		          "inputs": [
		            {
		              "name": "a",
		              "type": "uint256"
		            }
		          ],
		          "name": "multiply",
		          "outputs": [
		            {
		              "name": "d",
		              "type": "uint256"
		            }
		          ],
		          "type": "function"
		        }
		      ],
		      "userDoc": {
		        "methods": {}
		      },
		      "developerDoc": {
		        "methods": {}
		      }
		    }
		  }
		}

#### web3.eth.compile.lll – 编译lll代码
web3.eth.compile.lll() 方法用来编译使用 LLL 语言编写的合约源代码。

- 调用：

		web3. eth.compile.lll(sourceCode [, callback])
- 参数：
	- sourceCode：String – LLL源代码
	- callback：Function – 可选的回调函数，其第一个参数为错误对象，第二个参数为结果
- 返回值：

	一个Promise对象，其解析值为编译得到的16进制字符串。
- 示例代码：

		var source = "...";
		
		web3.eth.compile.lll(source)
		.then(console.log);
		> "0x603880600c6000396000f3006001600060e060020a600035048063c6888fa114601857005b60216
		
#### web3.eth.compile.serpent – 编译 serpent 代码
web3.eth.compile.serpent() 方法用来编译使用 serpent 语言编写的合约源代码。

- 调用：

		web3.eth.compile.serpent(sourceCode [, callback])
- 参数：
	- sourceCode：String – serpent 源代码
	- callback：Function – 可选的回调函数，其第一个参数为错误对象，第二个参数为结果
- 返回值：

	一个Promise对象，其解析值为编译得到的16进制字符串。
- 示例代码：

		var source = "...";
		
		var code = web3.eth.compile.serpent(source)
		.then(console.log);
		> "0x603880600c6000396000f3006001600060e060020a600035048063c6888fa114601857005b60216

#### web3.eth.getWork – 返回当前挖矿工作情况
web3.eth.getWork() 方法返回矿工的工作内容，包括当前块的哈希值、种子哈希值和要满足的边界条件。

- 调用：

		web3.eth.getWork([callback])
- 参数：
	- callback：Function – 可选的回调函数，其第一个参数为错误对象，第二个参数为结果
- 返回值：

	一个Promise对象，其解析值为具有以下结构的数组：

	- String 32 Bytes – 0#成员: 当前块头的 pow-hash
	- String 32 Bytes – 1#成员: 用于 DAG 算法的种子哈希
	- String 32 Bytes – 2#成员: 目标边界条件, 2^256 / difficulty.
- 示例代码：

		web3.eth.getWork()
		.then(console.log);
		> [
		  "0x1234567890abcdef1234567890abcdef1234567890abcdef1234567890abcdef",
		  "0x5EED00000000000000000000000000005EED0000000000000000000000000000",
		  "0xd1ff1c01710000000000000000000000d1ff1c01710000000000000000000000"
		]

#### web3.eth.submitWork – 提交POW解决方案
web3.eth.submitWork() 方法用来提交工作量证明（POW）解决方案。

- 调用：

		web3.eth.submitWork(nonce, powHash, digest, [callback])
- 参数：
	- nonce：String 8 Bytes: 找到的nonce(64 bits)
	- powHash：String 32 Bytes: 区块头pow-hash (256 bits)
	- digest：String 32 Bytes: 混合摘要 (256 bits)
	- callback：Function – 可选的回调函数，其第一个参数为错误对象，第二个参数为结果
- 返回值：

	一个Promise对象，当提交的解决方案有效时解析为true，否则解析为false。
- 示例代码：

		web3.eth.submitWork([
		    "0x0000000000000001",
		    "0x1234567890abcdef1234567890abcdef1234567890abcdef1234567890abcdef",
		    "0xD1FE5700000000000000000000000000D1FE5700000000000000000000000000"
		])
		.then(console.log);
		> true

#### web3.eth.subscribe – 订阅链上事件
使用 web3.eth.subscribe() 方法来订阅区块链上的指定事件。

- 调用：

		web3.eth.subscribe(type [, options] [, callback]);
- 参数：
	- type：String – 订阅类型
	- options：Mixed – 可选的额外参数，依赖于订阅类型
	- callback：Function – 可选的回调函数，其第一个参数为错误对象，第二个参数为结果
- 返回值：

	EventEmitter – 订阅实例对象，具有以下字段：
	
	- subscription.id: 订阅id编号，用于标识一个订阅以及进行后续的取消订阅操作
	- subscription.subscribe([callback]): 可用于使用相同的参数进行再次订阅
	- subscription.unsubscribe([callback]): 取消订阅，如果成功取消的话，在回调函数中返回true
	- subscription.arguments: 订阅参数，当重新订阅时使用
	- on("data") 返回 Object: 每次有新的日志时都触发该事件，参数为日志对象
	- on("changed") 返回 Object: 每次有日志从区块链上移除时触发该事件，被移除的日志对象将添加额外的属性："removed: true"
	- on("error") 返回 Object: 当订阅中发生错误时，触发此事件
- 通知返回值：

	Mixed – 取决于具体的订阅类型
- 示例代码：

		var subscription = web3.eth.subscribe('logs', {
		    address: '0x123456..',
		    topics: ['0x12345...']
		}, function(error, result){
		    if (!error)
		        console.log(log);
		});
		
		// unsubscribes the subscription
		subscription.unsubscribe(function(error, success){
		    if(success)
		        console.log('Successfully unsubscribed!');
		});

#### web3.eth.clearSubscriptions – 复位订阅状态
web3.eth.clearSubscriptions() 方法用来复位订阅状态。注意该方法不能复位其他包的订阅，例如 web3-shh，因为这些包有自己的请求管理器。

- 调用：

		web3.eth.clearSubscriptions(flag)
- 参数:
	- flag：Boolean – 值为true则表示保持同步订阅
- 返回值
	- Boolean：复位成功返回true，否则返回false
- 示例代码：

		web3.eth.subscribe('logs', {} ,function(){ ... });

		...
		
		web3.eth.clearSubscriptions();	

#### web3.eth.subscribe('pendingTransactions') – 订阅 pending 交易事件
参数 pendingTransactions 表示订阅处于 pending 状态的交易。

- 调用：

		web3.eth.subscribe('pendingTransactions' [, callback]);
- 参数：
	- type：String – "pendingTransactions"，订阅类型
	- callback：Function – 可选的回调函数，其第一个参数为错误对象，第二个参数为结果
- 返回值：
	
	EventEmitter: 订阅实例对象，是一个事件发生器，定义有以下事件：
	
	- "data" 返回 Object: 当接收到pending状态的交易时触发
	- "error" 返回 Object: 当订阅中发生错误时触发

	返回对象的结构，参见web3.eth.getTransaction()的返回值。

	通知返回值：

	- Object|Null – 第一个参数是一个错误对象，如果订阅成功则为null
	- Object – 块头对象
- 示例代码：

		var subscription = web3.eth.subscribe('pendingTransactions', function(error, result){
		    if (!error)
		        console.log(result);
		})
		.on("data", function(transaction){
		    console.log(transaction);
		});
		
		// 订阅取消
		subscription.unsubscribe(function(error, success){
		    if(success)
		        console.log('Successfully unsubscribed!');
		});			

#### web3.eth.subscribe('newBlockHeaders') – 订阅区块头生成事件
使用 newBlockHeaders 参数订阅新的区块头生成事件。可用做检查区块链上变化的计时器。

- 调用：

		web3.eth.subscribe('newBlockHeaders' [, callback]);
- 参数
	- type：String – "newBlockHeaders", 订阅类型
	- callback：Function – 可选的回调函数，其第一个参数为错误对象，第二个参数为结果
- 返回值：

	EventEmitter: 订阅对象实例，是一个事件发生器，定义有如下事件：
	
	- "data" 返回 Object: 当收到新的区块头时触发
	- "error" 返回 Object: 当订阅中出现错误时触发

	返回的区块头对象结构如下：
	
	- number – Number: 区块编号，对于pending的块该值为null
	- hash 32 Bytes – String: 块的哈希值，挂起的块该值为null
	- parentHash 32 Bytes – String: 父区块的哈希值
	- nonce 8 Bytes – String: 生成的proof-of-work的哈希值。挂起块该值为null、
	- sha3Uncles 32 Bytes – String: 块中叔伯数据的SHA3值
	- logsBloom 256 Bytes – String: 块日志的bloom filter，块处于挂起状态时该值为null
	- transactionsRoot 32 Bytes – String: 块交易树的根节点
	- stateRoot 32 Bytes – String: 块状态树的根节点
	- receiptRoot 32 Bytes – String: 收据根节点
	- miner – String: 接收挖矿奖励的矿工地址
	- extraData – String: 区块的额外数据字段
	- gasLimit – Number: 该块允许的最大gas用量
	- gasUsed – Number: 该块中所有交易使用的gas总用量
	- timestamp – Number: 出块的unix时间戳
- 通知返回值：
	- Object|Null – 如果订阅失败，则该参数为错误对象，否则为null
	- Object – 区块头对象
- 示例代码：

		var subscription = web3.eth.subscribe('newBlockHeaders', function(error, result){
		    if (error)
		        console.log(error);
		})
		.on("data", function(blockHeader){
		});
		
		// 卸载订阅
		subscription.unsubscribe(function(error, success){
		    if(success)
		        console.log('Successfully unsubscribed!');
		});

#### web3.eth.subscribe('syncing') – 订阅同步事件
使用 syncing 参数订阅同步事件。当节点同步时将返回一个同步对象，否则返回 false。

- 调用：

		web3.eth.subscribe('syncing' [, callback]);
- 参数：
	- type：String – "syncing", 订阅类型
	- callback：Function – 可选的回调函数，其第一个参数为错误对象，第二个参数为结果
- 返回值：

	EventEmitter: 订阅对象实例，是一个事件发生器，定义有如下事件：
	
	- "data" 返回 Object: 收到同步对象时触发
	- "changed" 返回 Object: 当节点从同步状态转换为非同步状态时触发
	- "error" 返回 Object: 当订阅中出现错误时触发

	要了解返回的事件对象的结构，可以查看web3.eth.isSyncing()方法的返回值。

	通知返回值：

	- Object|Null – 当订阅失败时，该值为错误对象，否则为null
	- Object|Boolean – 同步对象
- 示例代码：

		var subscription = web3.eth.subscribe('syncing', function(error, sync){
		    if (!error)
		        console.log(sync);
		})
		.on("data", function(sync){
		    // 显示一些同步数据
		})
		.on("changed", function(isSyncing){
		    if(isSyncing) {
		        // 停止应用程序操作
		    } else {
		        // 恢复应用程序操作
		    }
		});
		
		// 停止订阅
		subscription.unsubscribe(function(error, success){
		    if(success)
		        console.log('Successfully unsubscribed!');
		});

#### web3.eth.subscribe('logs') – 订阅日志
使用 logs 参数订阅日志，并且可以指定条件进行过滤。

- 调用：

		web3.eth.subscribe('logs', options [, callback]);
- 参数：
	- "logs" ：String, 订阅类型
	- options：Object – 订阅选项，该对象包含如下字段：
		- fromBlock – Number: 最早块的编号，默认值为null
		- address – String|Array: 地址或地址列表，仅订阅来自这些账户地址的日志
		- topics – Array: 主题数组，仅订阅日志中包含这些主题的日志。
	- callback – Function: 可选的回调函数，其第一个参数为错误对象，第二个参数为结果
- 返回值：

	EventEmitter: 订阅实例对象，是一个事件发生器，定义有如下事件：

	- "data" 返回 Object: 接收到新日志时触发，参数为日志对象
	- "changed" 返回 Object: 日志从链上移除时触发，该日志同时添加属性 "removed: true"
	- "error" 返回 Object: 当订阅中出现错误时触发

	要了解返回的事件对象的结果，可查阅web3.eth.getPastEvents()方法的返回值。

- 通知返回值：
	- Object|Null – 订阅失败时该值为错误对象，否则为null
	- Object – 日志对象，类似于web3.eth.getPastEvents()方法的返回值
- 示例代码：

		var subscription = web3.eth.subscribe('logs', {
		    address: '0x123456..',
		    topics: ['0x12345...']
		}, function(error, result){
		    if (!error)
		        console.log(result);
		})
		.on("data", function(log){
		    console.log(log);
		})
		.on("changed", function(log){
		});
		
		// 取消订阅
		subscription.unsubscribe(function(error, success){
		    if(success)
		        console.log('Successfully unsubscribed!');
		});
							
## 合约相关
#### web3.eth.Contract – 合约构造函数
web3.eth.Contract 类简化了与以太坊区块链上智能合约的交互。创建合约对象时，只需指定相应智能合约的 json 接口，web3 就可以自动地将所有的调用转换为底层基于 RPC 的 ABI 调用。

通过 web3 的封装，与智能合约的交互就像与 JavaScript 对象一样简单。

实例化一个新的合约对象：

	new web3.eth.Contract(jsonInterface[, address][, options])

- 参数：
	- jsonInterface – Object: 要实例化的合约的json接口
	- address – String: 可选，要调用的合约的地址，也可以在之后使用 myContract.options.address = '0x1234..' 来指定该地址
	- options – Object : 可选，合约的配置对象，其中某些字段用作调用和交易的回调：
		- from – String: 交易发送方地址
		- gasPrice – String: 用于交易的gas价格，单位：wei
		- gas – Number: 交易可用的最大gas量，即gas limit
		- data – String: 合约的字节码，部署合约时需要
- 返回值：

	Object: 合约实例及其所有方法和事件。
- 示例代码：

		var myContract = new web3.eth.Contract([...], '0xde0B295669a9FD93d5F28D9Ec85E40f4cb697BAe', {
		    from: '0x1234567890123456789012345678901234567891', // 默认 from 地址
		    gasPrice: '20000000000' // 默认的 gas price 价格，这个例子里是 20 gwei
		});		
		
### options – 合约配置对象	
合约实例的可选配置对象。当发送交易时，其 from、gas 和 gasPrice 被用作回调值。

- 调用

		myContract.options
- options 属性对象具有以下字段
	- address – String: 合约的部署地址
	- jsonInterface – Array: 合约的json接口
	- data – String: 合约的字节码，合约部署时会用到
	- from – String: 合约发送方账户地址
	- gasPrice – String: 用于交易的gas价格，单位：wei
	- gas – Number: 交易的gas用量上限，即gas limit
- 示例代码：

		myContract.options;
		> {
		    address: '0x1234567890123456789012345678901234567891',
		    jsonInterface: [...],
		    from: '0xde0B295669a9FD93d5F28D9Ec85E40f4cb697BAe',
		    gasPrice: '10000000000000',
		    gas: 1000000
		}
		
		myContract.options.from = '0x1234567890123456789012345678901234567891'; // 默认地址
		myContract.options.gasPrice = '20000000000000'; // 默认 gas price 单位 wei
		myContract.options.gas = 5000000; // 提供作为后备总是5M天然气

### options.address – 合约地址
用于合约实例的地址，保存。web3.js 从这个合约生成的所有交易，其 to 字段的值都是该地址。当设置该选项时，web3.js 将以小写形式保存。

- 调用

		myContract.options.address
- 属性
	- address – String|null: 合约实例对象的地址，如果未设置的话则为null
- 示例代码：

		myContract.options.address;
		> '0xde0b295669a9fd93d5f28d9ec85e40f4cb697bae'
		
		// 设置一个新地址
		myContract.options.address = '0x1234FFDD...';

### options.jsonInterface – 合约JSON接口
jsonInterface配置项基于合约的ABI信息生成。

- 调用

		myContract.options.jsonInterface
- 属性
	- jsonInterface – Array: 合约的json接口。重新设置该属性将重新生成合约实例的方法和事件。
- 示例代码

		myContract.options.jsonInterface;
		> [{
		    "type":"function",
		    "name":"foo",
		    "inputs": [{"name":"a","type":"uint256"}],
		    "outputs": [{"name":"b","type":"address"}]
		},{
		    "type":"event",
		    "name":"Event"
		    "inputs": [{"name":"a","type":"uint256","indexed":true},{"name":"b","type":"bytes32","indexed":false}],
		}]
		
		// 设置一个新的接口
		myContract.options.jsonInterface = [...];

### clone – 克隆合约
克隆当前的合约实例对象。

- 调用

		myContract.clone()
- 参数

	无
- 返回值：

	Object: 克隆得到的新合约实例
- 示例代码：

		var contract1 = new eth.Contract(abi, address, {gasPrice: '12345678', from: fromAddress});
		
		var contract2 = contract1.clone();
		contract2.options.address = address2;
		
		(contract1.options.address !== contract2.options.address);
		> true

### deploy – 部署合约
调用合约的 deploy() 方法将其部署到区块链上。其返回的 Promise 对象将在成功部署后解析为新的合约实例。

- 调用

		myContract.deploy(options)
- 参数
	- options – Object: 用于部署的配置选项，包含以下字段：
		- data – String: 合约的字节码
		- arguments – Array : 可选，在部署时将传入合约的构造函数
- 返回值：

	Object: 交易对象，包含以下字段：

	- arguments: Array – 之前传入方法的参数，可修改
	- send: Function – 用来部署合约，其返回的promise对象将解析为新的合约实例，而非交易收据！
	- estimateGas: Function – 用来估算用于部署的gas用量
	- encodeABI: Function – 用来编码部署的ABI数据，即合约数据 + 构造函数参数
- 示例代码：

		myContract.deploy({
		    data: '0x12345...',
		    arguments: [123, 'My String']
		})
		.send({
		    from: '0x1234567890123456789012345678901234567891',
		    gas: 1500000,
		    gasPrice: '30000000000000'
		}, function(error, transactionHash){ ... })
		.on('error', function(error){ ... })
		.on('transactionHash', function(transactionHash){ ... })
		.on('receipt', function(receipt){
		   console.log(receipt.contractAddress) // 收据中包含了新的合约地址
		})
		.on('confirmation', function(confirmationNumber, receipt){ ... })
		.then(function(newContractInstance){
		    console.log(newContractInstance.options.address) // 新地址的合约实例
		});
		
		// data是合约自身的一个可选配置项
		myContract.options.data = '0x12345...';
		
		myContract.deploy({
		    arguments: [123, 'My String']
		})
		.send({
		    from: '0x1234567890123456789012345678901234567891',
		    gas: 1500000,
		    gasPrice: '30000000000000'
		})
		.then(function(newContractInstance){
		    console.log(newContractInstance.options.address) // 实例的新合同地址
		});
		
		// 编码
		myContract.deploy({
		    data: '0x12345...',
		    arguments: [123, 'My String']
		})
		.encodeABI();
		> '0x12345...0000012345678765432'
		
		
		// 估算gas
		myContract.deploy({
		    data: '0x12345...',
		    arguments: [123, 'My String']
		})
		.estimateGas(function(err, gas){
		    console.log(gas);
		});

### methods – 为合约方法创建交易
为指定的合约方法创建一个交易对象，以便使用该交易对象进行调用、发送或估算 gas。

- 调用

		myContract.methods.myMethod([param1[, param2[, ...]]])

可以使用以下语法获得指定方法的交易对象：

- 名称: myContract.methods.myMethod(http://cw.hubwiz.com/card/c/web3.js-1.0/1/4/7/123)
- 带参名称: myContract.methods'myMethod(uint256)'
- 签名: myContract.methods'0x58cf5f10'

这样就可以支持从 javascript 合约对象调用同名但参数不同的合约方法。

- 参数

	参数取决于在JSON接口中定义的合约方法
- 返回值

	Object: 交易对象，包含以下字段
	
	- arguments: Array – 之前传入方法的参数，可修改
	- call: Function – 用来调用只读的合约方法，在EVM直接执行而不必发出交易，因此不会改变合约的状态
	- send: Function – 用来向合约发送交易并执行方法，因此可以改变合约的状态
	- estimateGas: Function – 用来估算方法在链上执行时的gas用量
	- encodeABI: Function – 用来为合约方法进行ABI编码。
- 示例代码：

		// 调用合约方法
		myContract.methods.myMethod(http://cw.hubwiz.com/card/c/web3.js-1.0/1/4/7/123).call({from: '0xde0B295669a9FD93d5F28D9Ec85E40f4cb697BAe'}, function(error, result){
		    ...
		});
		
		// 发送交易，使用Promise对象获取返回结果
		myContract.methods.myMethod(http://cw.hubwiz.com/card/c/web3.js-1.0/1/4/7/123).send({from: '0xde0B295669a9FD93d5F28D9Ec85E40f4cb697BAe'})
		.then(function(receipt){
		    // receipt can also be a new contract instance, when coming from a "contract.deploy({...}).send()"
		});
		
		// 发送交易，使用事件获取返回结果
		myContract.methods.myMethod(http://cw.hubwiz.com/card/c/web3.js-1.0/1/4/7/123).send({from: '0xde0B295669a9FD93d5F28D9Ec85E40f4cb697BAe'})
		.on('transactionHash', function(hash){
		    ...
		})
		.on('receipt', function(receipt){
		    ...
		})
		.on('confirmation', function(confirmationNumber, receipt){
		    ...
		})
		.on('error', console.error);
		
### call – 调用合约方法
调用合约的只读方法，并在 EVM 中直接执行方法，不需要发送任何交易。因此不会改变合约的状态。

- 调用：

		myContract.methods.myMethod([param1[, param2[, ...]]]).call(options[, callback])
- 参数：
	- options – Object : 选项，包含如下字段：
		- from – String (optional): The address the call “transaction” should be made from.
		- gasPrice – String (optional): The gas price in wei to use for this call “transaction”.
		- gas – Number (optional): The maximum gas provided for this call “transaction” (gas limit).
	- callback – Function : 可选的回调函数，其第二个参数为合约方法的执行结果，第一个参数为错误对象
- 返回值：

	一个Promise对象，其解析值为合约方法的返回值，Mixed类型。如果合约方法返回多个值，则解析值为一个对象。
- 示例代码：

		// 使用回调函数接收合约方法执行结果
		myContract.methods.myMethod(123).call({from: '0xde0B295669a9FD93d5F28D9Ec85E40f4cb697BAe'}, function(error, result){
		    ...
		});
		
		// 使用Promise接收合约方法执行结果
		myContract.methods.myMethod(123).call({from: '0xde0B295669a9FD93d5F28D9Ec85E40f4cb697BAe'})
		.then(function(result){
		    ...
		});
	下面合约中的方法返回多个值：

		// Solidity
		contract MyContract {
		    function myFunction() returns(uint256 myNumber, string myString) {
		        return (23456, "Hello!%");
		    }
		}
	那么在 web3.js 中，call 方法的返回值将是一个对象，即使用返回变量名作为键，也使用索引作为键：

		// web3.js
		var MyContract = new web3.eth.contract(abi, address);
		MyContract.methods.myFunction().call()
		.then(console.log);
		> Result {
		    myNumber: '23456',
		    myString: 'Hello!%',
		    0: '23456', // these are here as fallbacks if the name is not know or given
		    1: 'Hello!%'
		}
	下面合约中的方法返回单个值：

		// Solidity
		contract MyContract {
		    function myFunction() returns(string myString) {
		        return "Hello!%";
		    }
		}
	那么在web3.js中，call方法也将返回单个值：

		// web3.js
		var MyContract = new web3.eth.contract(abi, address);
		MyContract.methods.myFunction().call()
		.then(console.log);
		> "Hello!%"

### send – 发送合约方法交易
向合约发送交易来执行指定方法，将改变合约的状态。

- 调用：

		myContract.methods.myMethod([param1[, param2[, ...]]]).send(options[, callback])
- 参数：
	- options – Object: 选项，包含如下字段：
		- from – String: 交易发送方地址
		- gasPrice – String : 可选，用于本次交易的gas价格，单位：wei
		- gas – Number : 可选，本次交易的gas用量上限，即gas limit
		- value – Number|String|BN|BigNumber: 可选，交易转账金额，单位：wei
	- callback – Function: 可选的回调参数，其参数为交易哈希值和错误对象
- 返回值：

	回调函数中将返回32字节长的交易哈希值。

	PromiEvent: 一个Promise对象，当交易收据有效时或者发送交易时解析为新的合约实例。它同时也是一个事件发生器，声明有以下事件：
	
	- "transactionHash" 返回 String: 交易发送后得到有效交易哈希值时触发
	- "receipt" 返回 Object: 交易收据有效时触发。
	- "confirmation" 返回 Number, Object: 收到确认时触发
	- "error" 返回 Error: 交易发送过程中如果出现错误则触发此事件。对于out of gas错误，其第二个参数为交易收据
- 示例代码：

		// 使用回调
		myContract.methods.myMethod(123).send({from: '0xde0B295669a9FD93d5F28D9Ec85E40f4cb697BAe'}, function(error, transactionHash){
		    ...
		});
		
		//使用 promise
		myContract.methods.myMethod(123).send({from: '0xde0B295669a9FD93d5F28D9Ec85E40f4cb697BAe'})
		.then(function(receipt){
		    // receipt can also be a new contract instance, when coming from a "contract.deploy({...}).send()"
		});
		
		
		// 使用事件发射器
		myContract.methods.myMethod(123).send({from: '0xde0B295669a9FD93d5F28D9Ec85E40f4cb697BAe'})
		.on('transactionHash', function(hash){
		    ...
		})
		.on('confirmation', function(confirmationNumber, receipt){
		    ...
		})
		.on('receipt', function(receipt){
		    // 收据的例子
		    console.log(receipt);
		    > {
		        "transactionHash": "0x9fc76417374aa880d4449a1f7f31ec597f00b1f6f3dd2d66f4c9c6c445836d8b",
		        "transactionIndex": 0,
		        "blockHash": "0xef95f2f1ed3ca60b048b4bf67cde2195961e0bba6f70bcbea9a2c4e133e34b46",
		        "blockNumber": 3,
		        "contractAddress": "0x11f4d0A3c12e86B4b5F39B213F7E19D048276DAe",
		        "cumulativeGasUsed": 314159,
		        "gasUsed": 30234,
		        "events": {
		            "MyEvent": {
		                returnValues: {
		                    myIndexedParam: 20,
		                    myOtherIndexedParam: '0x123456789...',
		                    myNonIndexParam: 'My String'
		                },
		                raw: {
		                    data: '0x7f9fade1c0d57a7af66ab4ead79fade1c0d57a7af66ab4ead7c2c2eb7b11a91385',
		                    topics: ['0xfd43ade1c09fade1c0d57a7af66ab4ead7c2c2eb7b11a91ffdd57a7af66ab4ead7', '0x7f9fade1c0d57a7af66ab4ead79fade1c0d57a7af66ab4ead7c2c2eb7b11a91385']
		                },
		                event: 'MyEvent',
		                signature: '0xfd43ade1c09fade1c0d57a7af66ab4ead7c2c2eb7b11a91ffdd57a7af66ab4ead7',
		                logIndex: 0,
		                transactionIndex: 0,
		                transactionHash: '0x7f9fade1c0d57a7af66ab4ead79fade1c0d57a7af66ab4ead7c2c2eb7b11a91385',
		                blockHash: '0xfd43ade1c09fade1c0d57a7af66ab4ead7c2c2eb7b11a91ffdd57a7af66ab4ead7',
		                blockNumber: 1234,
		                address: '0xde0B295669a9FD93d5F28D9Ec85E40f4cb697BAe'
		            },
		            "MyOtherEvent": {
		                ...
		            },
		            "MyMultipleEvent":[{...}, {...}] // 如果有多个相同的事件，它们将在一个数组中
		        }
		    }
		})
		.on('error', console.error); // 如果有一个没有 gas 的错误，第二个参数是收据。

### estimateGas – 估算合约方法gas用量
通过在 EVM 中执行方法来估算链上执行是需要的 gas 用量。得到的估算值可能与之后实际发送交易的 gas 用量有差异，因为合约的状态可能在两个时刻存在差异。

- 调用：

		myContract.methods.myMethod([param1[, param2[, ...]]]).estimateGas(options[, callback])
- 参数
	- options – Object : 选项，包括以下字段：
		- from – String : 可选，交易发送方地址
		- gas – Number : 可选，本次交易gas用量上限
		- value – Number|String|BN|BigNumber: 可选，交易转账金额，单位：wei
		- callback – Function : 可选的回调函数，触发时其第二个参数为gas估算量，第一个参数为错误对象。
- 返回值：

	一个Promise对象，其解析值为估算的 gas 用量。
- 示例代码：

		// 使用回调函数
		myContract.methods.myMethod(123).estimateGas({gas: 5000000}, function(error, gasAmount){
		    if(gasAmount == 5000000)
		        console.log('Method ran out of gas');
		});
		
		// 使用promise
		myContract.methods.myMethod(123).estimateGas({from: '0xde0B295669a9FD93d5F28D9Ec85E40f4cb697BAe'})
		.then(function(gasAmount){
		    ...
		})
		.catch(function(error){
		    ...
		});	

### encodeABI – ABI编码合约方法
为指定的合约方法进行ABI编码，可用于发送交易、调用方法或向另一个合约方法传递参数。

- 调用：

		myContract.methods.myMethod([param1[, param2[, ...]]]).encodeABI()
- 参数：

	无
- 返回值：

	String: 编码后的ABI字节码，可用于交易发送或方法调用
- 示例代码：

		myContract.methods.myMethod(123).encodeABI();
		> '0x58cf5f1000000000000000000000000000000000000000000000000000000000000007B'

## 订阅相关		
### once – 单次订阅合约事件
订阅合约指定的事件，并在第一次触发后或发生错误后立即取消订阅。一个事件仅触发一次。

- 调用：

		myContract.once(event[, options], callback)
- 参数：
	- event – String: 要订阅的合约事件名，或者使用"allEvents" 订阅所有事件
	- options – Object: 可选，用于部署的选项，包含以下字段：
		- filter – Object : 可选，用于根据索引参数过滤事件。例如 {filter: {myNumber: [12,13]}} 表示“myNumber” 为12或13的所有事件
		- topics – Array : 可选，用于手动设置事件过滤器的主题。如果制订了filter属性和事件签名，将不会自动设置(topic[0])
		- callback – Function: 该回调函数触发时，其第一个参数为错误对象，第二个参数为事件对象
- 返回值：

	未定义
- 示例代码：

		myContract.once('MyEvent', {
		    filter: {myIndexedParam: [20,23], myOtherIndexedParam: '0x123456789...'}, // Using an array means OR: e.g. 20 or 23
		    fromBlock: 0
		}, function(error, event){ console.log(event); });
		
		// 事件输出示例
		> {
		    returnValues: {
		        myIndexedParam: 20,
		        myOtherIndexedParam: '0x123456789...',
		        myNonIndexParam: 'My String'
		    },
		    raw: {
		        data: '0x7f9fade1c0d57a7af66ab4ead79fade1c0d57a7af66ab4ead7c2c2eb7b11a91385',
		        topics: ['0xfd43ade1c09fade1c0d57a7af66ab4ead7c2c2eb7b11a91ffdd57a7af66ab4ead7', '0x7f9fade1c0d57a7af66ab4ead79fade1c0d57a7af66ab4ead7c2c2eb7b11a91385']
		    },
		    event: 'MyEvent',
		    signature: '0xfd43ade1c09fade1c0d57a7af66ab4ead7c2c2eb7b11a91ffdd57a7af66ab4ead7',
		    logIndex: 0,
		    transactionIndex: 0,
		    transactionHash: '0x7f9fade1c0d57a7af66ab4ead79fade1c0d57a7af66ab4ead7c2c2eb7b11a91385',
		    blockHash: '0xfd43ade1c09fade1c0d57a7af66ab4ead7c2c2eb7b11a91ffdd57a7af66ab4ead7',
		    blockNumber: 1234,
		    address: '0xde0B295669a9FD93d5F28D9Ec85E40f4cb697BAe'
		}
		
### events – 订阅合约事件
订阅指定的合约事件。

- 调用：

		myContract.events.MyEvent([options][, callback])
- 参数
	- options – Object: 可选，用于部署的选项，包含以下字段：
		- filter – Object : 可选，按索引参数过滤事件。例如 {filter: {myNumber: [12,13]}} 表示 “myNumber” 为12或13的所有事件
		- fromBlock – Number: 可选，仅监听该选项指定编号的块中发生的事件
		- topics – Array : 可选，用来手动为事件过滤器设定主题。如果设置过filter属性和事件签名，那么(topic[0])将不会自动设置
	- callback – Function: 可选，该回调函数触发时，其第二给参数为事件对象，第一个参数为错误对象
- 返回值：
	
	 EventEmitter: 事件发生器，声明有以下事件:
	 
	 - "data" 返回 Object: 接收到新的事件时触发，参数为事件对象
	 - "changed" 返回 Object: 当事件从区块链上移除时触发，该事件对象将被添加额外的属性"removed: true"
	 - "error" 返回 Object: 当发生错误时触发

	返回的事件对象结构如下：
	
	- event – String: 事件名称
	- signature – String|Null: 事件签名，如果是匿名事件，则为null
	- address – String: 事件源地址
	- returnValues – Object: 事件返回值，例如 {myVar: 1, myVar2: '0x234…'}.
	- logIndex – Number: 事件在块中的索引位置
	- transactionIndex – Number: 事件在交易中的索引位置
	- transactionHash 32 Bytes – String: 事件所在交易的哈希值
	- blockHash 32 Bytes – String: 事件所在块的哈希值，pending的块该值为 null
	- blockNumber – Number: 事件所在块的编号，pending的块该值为null
	- raw.data – String: 该字段包含未索引的日志参数
	- raw.topics – Array: 最多可保存4个32字节长的主题字符串数组。主题1-3 包含事件的索引参数
- 示例代码：

		myContract.events.MyEvent({
		    filter: {myIndexedParam: [20,23], myOtherIndexedParam: '0x123456789...'}, // Using an array means OR: e.g. 20 or 23
		    fromBlock: 0
		}, function(error, event){ console.log(event); })
		.on('data', function(event){
		    console.log(event); // same results as the optional callback above
		})
		.on('changed', function(event){
		    // remove event from local database
		})
		.on('error', console.error);
		
		// event output example
		> {
		    returnValues: {
		        myIndexedParam: 20,
		        myOtherIndexedParam: '0x123456789...',
		        myNonIndexParam: 'My String'
		    },
		    raw: {
		        data: '0x7f9fade1c0d57a7af66ab4ead79fade1c0d57a7af66ab4ead7c2c2eb7b11a91385',
		        topics: ['0xfd43ade1c09fade1c0d57a7af66ab4ead7c2c2eb7b11a91ffdd57a7af66ab4ead7', '0x7f9fade1c0d57a7af66ab4ead79fade1c0d57a7af66ab4ead7c2c2eb7b11a91385']
		    },
		    event: 'MyEvent',
		    signature: '0xfd43ade1c09fade1c0d57a7af66ab4ead7c2c2eb7b11a91ffdd57a7af66ab4ead7',
		    logIndex: 0,
		    transactionIndex: 0,
		    transactionHash: '0x7f9fade1c0d57a7af66ab4ead79fade1c0d57a7af66ab4ead7c2c2eb7b11a91385',
		    blockHash: '0xfd43ade1c09fade1c0d57a7af66ab4ead7c2c2eb7b11a91ffdd57a7af66ab4ead7',
		    blockNumber: 1234,
		    address: '0xde0B295669a9FD93d5F28D9Ec85E40f4cb697BAe'
		}
		
### events.allEvents – 订阅全部事件
- 调用：
	
		myContract.events.allEvents([options][, callback])
	与 events 相同，只是可以接收合约的全部事件。可以使用options.filter属性进行过滤
	
### getPastEvents – 读取历史事件
读取合约的历史事件。

- 调用:

		myContract.getPastEvents(event[, options][, callback])
- 参数
	- event – String: 事件名，或者使用 "allEvents" 来读取所有的事件
	- options – Object: 用于部署的选项，包含以下字段：
		- filter – Object : 可选，按索引参数过滤事件，例如 {filter: {myNumber: [12,13]}} 表示所有“myNumber” 为12 或 13的事件
		- fromBlock – Number : 可选，仅读取从该编号开始的块中的历史事件。
		- toBlock – Number : 可选，仅读取截止到该编号的块中的历史事件，默认值为"latest"
		- topics – Array : 可选，用来手动设置事件过滤器的主题。如果设置了filter属性和事件签名，那么(topic[0])将不会自动设置
	- callback – Function : 可选的回调参数，触发时其第一个参数为错误对象，第二个参数为历史事件数组
- 返回值

	一个Promise对象，其解析值为历史事件对象数组
- 示例代码

		myContract.getPastEvents('MyEvent', {
		    filter: {myIndexedParam: [20,23], myOtherIndexedParam: '0x123456789...'}, // 使用数组意味着或:例如20或23		    fromBlock: 0,
		    toBlock: 'latest'
		}, function(error, events){ console.log(events); })
		.then(function(events){
		    console.log(events) //结果与上面的可选回调相同
		});
		
		> [{
		    returnValues: {
		        myIndexedParam: 20,
		        myOtherIndexedParam: '0x123456789...',
		        myNonIndexParam: 'My String'
		    },
		    raw: {
		        data: '0x7f9fade1c0d57a7af66ab4ead79fade1c0d57a7af66ab4ead7c2c2eb7b11a91385',
		        topics: ['0xfd43ade1c09fade1c0d57a7af66ab4ead7c2c2eb7b11a91ffdd57a7af66ab4ead7', '0x7f9fade1c0d57a7af66ab4ead79fade1c0d57a7af66ab4ead7c2c2eb7b11a91385']
		    },
		    event: 'MyEvent',
		    signature: '0xfd43ade1c09fade1c0d57a7af66ab4ead7c2c2eb7b11a91ffdd57a7af66ab4ead7',
		    logIndex: 0,
		    transactionIndex: 0,
		    transactionHash: '0x7f9fade1c0d57a7af66ab4ead79fade1c0d57a7af66ab4ead7c2c2eb7b11a91385',
		    blockHash: '0xfd43ade1c09fade1c0d57a7af66ab4ead7c2c2eb7b11a91ffdd57a7af66ab4ead7',
		    blockNumber: 1234,
		    address: '0xde0B295669a9FD93d5F28D9Ec85E40f4cb697BAe'
		},{
		    ...
		}]

## 账户管理
### web3.eth.accounts – 账户管理与交易签名
web3.eth.accounts 包中的函数用于创建以太坊账户以及对数据和交易的签名。

这个包还没有进行审计，使用时有潜在的不安全因素。要注意正确清理内存、安全保存私钥，并且在上线生产环境之前进行测试，保证交易的发送和接收功能正常。

可以单独使用这个包：

	var Accounts = require('web3-eth-accounts');
	
	//传入 eth 或 web3 包是必要的，以允许自动检索 chainId, gasPrice 和 nonce 账户. signtransaction()。
	var accounts = new Accounts('ws://localhost:8546');

#### web3.eth.accounts.create – 创建账户
创建一个账户对象。

- 调用：

		web3.eth.accounts.create([entropy]);
- 参数：

	entropy – String : 可选，用于增加混乱度的随机字符串，至少32字符长。如果未设定将使用 randomhex 生成一个随机字符串
- 返回值：
	- Object – 账户对象，结构如下
		- address – string: 账户地址
		- privateKey – string: 账户私钥，绝不要共享私钥或者在本地不加密保存!同时确保在使用后清空内存
		- signTransaction(tx [, callback]) – Function: 用来对交易进行签名。
		- sign(data) – Function: 用来对数据进行签名。
- 示例代码：

		web3.eth.accounts.create();
		> {
		    address: "0xb8CE9ab6943e0eCED004cDe8e3bBed6568B2Fa01",
		    privateKey: "0x348ce564d427a3311b6536bbcff9390d69395b06ed6c486954e971d960fe8709",
		    signTransaction: function(tx){...},
		    sign: function(data){...},
		    encrypt: function(password){...}
		}
		
		web3.eth.accounts.create('2435@#@#@±±±±!!!!678543213456764321§34567543213456785432134567');
		> {
		    address: "0xF2CD2AA0c7926743B1D4310b2BC984a0a453c3d4",
		    privateKey: "0xd7325de5c2c1cf0009fac77d3d04a9c004b038883446b065871bc3e831dcd098",
		    signTransaction: function(tx){...},
		    sign: function(data){...},
		    encrypt: function(password){...}
		}
		
		web3.eth.accounts.create(web3.utils.randomHex(32));
		> {
		    address: "0xe78150FaCD36E8EB00291e251424a0515AA1FF05",
		    privateKey: "0xcc505ee6067fba3f6fc2050643379e190e087aeffe5d958ab9f2f3ed3800fa4e",
		    signTransaction: function(tx){...},
		    sign: function(data){...},
		    encrypt: function(password){...}
		}
		
### web3.eth.accounts.privateKeyToAccount – 使用指定私钥创建账户
使用指定的私钥创建一个账户对象。

- 调用

		web3.eth.accounts.privateKeyToAccount(privateKey);
- 参数

	privateKey – String: 私钥
- 返回值：

	Object – 账户对象。
- 示例代码：

		web3.eth.accounts.privateKeyToAccount('0x348ce564d427a3311b6536bbcff9390d69395b06ed6c486954e971d960fe8709');
		> {
		    address: '0xb8CE9ab6943e0eCED004cDe8e3bBed6568B2Fa01',
		    privateKey: '0x348ce564d427a3311b6536bbcff9390d69395b06ed6c486954e971d960fe8709',
		    signTransaction: function(tx){...},
		    sign: function(data){...},
		    encrypt: function(password){...}
		}
		
		web3.eth.accounts.privateKeyToAccount('0x348ce564d427a3311b6536bbcff9390d69395b06ed6c486954e971d960fe8709');
		> {
		    address: '0xb8CE9ab6943e0eCED004cDe8e3bBed6568B2Fa01',
		    privateKey: '0x348ce564d427a3311b6536bbcff9390d69395b06ed6c486954e971d960fe8709',
		    signTransaction: function(tx){...},
		    sign: function(data){...},
		    encrypt: function(password){...}
		}

### web3.eth.accounts.recoverTransaction – 提取交易的签名账户
从给定的RLP编码的交易中提取签名地址。

- 调用

		web3.eth.accounts.recoverTransaction(rawTransaction);
- 参数

	rawTransaction – String: RLP编码的交易
- 返回值

	String: 对该建议进行签名的以太坊地址
- 示例代码：

		web3.eth.accounts.recoverTransaction('0xf86180808401ef364594f0109fc8df283027b6285cc889f5aa624eac1f5580801ca031573280d608f75137e33fc14655f097867d691d5c4c44ebe5ae186070ac3d5ea0524410802cdc025034daefcdfa08e7d2ee3f0b9d9ae184b2001fe0aff07603d9');
		> "0xF0109fC8DF283027b6285cc889F5aA624EaC1F55"

### web3.eth.accounts.hashMessage – 计算消息的哈希
计算给定消息的哈希，用于 web3.eth.accounts.recover() 。

- 调用

		web3.eth.accounts.hashMessage(message);
- 参数

	message – String: 要进行哈希计算的消息，如果是16进制字符串，将首先进行UTF8解码.
- 返回值

	String: 哈希过的消息
- 示例代码：

		web3.eth.accounts.hashMessage("Hello World")
		> "0xa1de988600a42c4b4ab089b619297c17d53cffae5d5120d82d8a92d0bb3b78f2"

		// 下面的结果是相同的散列
		web3.eth.accounts.hashMessage(web3.utils.utf8ToHex("Hello World"))
		> "0xa1de988600a42c4b4ab089b619297c17d53cffae5d5120d82d8a92d0bb3b78f2"

### web3.eth.accounts.sign – 为数据生成签名
对任意数据进行签名。数据应当是 utf-8 并且 16 进制解码的，封装如下：

	"\x19Ethereum Signed Message:\n" + message.length + message

- 调用

		web3.eth.accounts.sign(data, privateKey);
- 参数
	- data – String: 要签名的数据
	- privateKey – String: 用来签名的私钥
- 返回值：

	String|Object: RLP编码的签名：
	
	- message – String: 指定的消息
	- messageHash – String: 消息的哈希
	- r – String: 签名的前32字节
	- s – String: 签名的后32字节
	- v – String: 恢复值 + 27
- 示例代码：

		web3.eth.accounts.sign('Some data', '0x4c0883a69102937d6231471b5dbb6204fe5129617082792ae468d01a3f362318');
		> {
		    message: 'Some data',
		    messageHash: '0x1da44b586eb0729ff70a73c326926f6ed5a25f5b056e7f47fbc6e58d86871655',
		    v: '0x1c',
		    r: '0xb91467e570a6466aa9e9876cbcd013baba02900b8979d43fe208a4a4f339f5fd',
		    s: '0x6007e74cd82e037b800186422fc2da167c747ef045e5d18a5f5d4300f8e1a029',
		    signature: '0xb91467e570a6466aa9e9876cbcd013baba02900b8979d43fe208a4a4f339f5fd6007e74cd82e037b800186422fc2da167c747ef045e5d18a5f5d4300f8e1a0291c'
		}

### web3.eth.accounts.recover – 提取数据的签名账户
从给定的已签名数据中回复用来进行签名的以太坊地址。

- 调用

		web3.eth.accounts.recover(signatureObject);
		web3.eth.accounts.recover(message, signature [, preFixed]);
		web3.eth.accounts.recover(message, v, r, s [, preFixed]);
- 参数
	- message|signatureObject – String|Object: 已签名消息或消息哈希，或者是如下的签名对象：
		- messageHash – String: 给定消息的哈希，已加前缀 "\x19Ethereum Signed Message:\n" + message.length + message.
		- r – String: 签名的前32字节
		- s – String: 签名的后32字节
		- v – String: 恢复值 + 27
	- signature – String: RLP编码的签名，或者2-4参数分别是 v, r, s
	- preFixed – Boolean ，可选，默认值为false。如果该值为true，则给定的消息不会自动添加前缀 "\x19Ethereum Signed Message:\n" + message.length + message
- 返回值：

	String: 用于签名的以太坊地址
- 示例代码：

		web3.eth.accounts.recover({
		    messageHash: '0x1da44b586eb0729ff70a73c326926f6ed5a25f5b056e7f47fbc6e58d86871655',
		    v: '0x1c',
		    r: '0xb91467e570a6466aa9e9876cbcd013baba02900b8979d43fe208a4a4f339f5fd',
		    s: '0x6007e74cd82e037b800186422fc2da167c747ef045e5d18a5f5d4300f8e1a029'
		})
		> "0x2c7536E3605D9C16a7a3D7b1898e529396a65c23"
		
		// hash signature
		web3.eth.accounts.recover('0x1da44b586eb0729ff70a73c326926f6ed5a25f5b056e7f47fbc6e58d86871655', '0xb91467e570a6466aa9e9876cbcd013baba02900b8979d43fe208a4a4f339f5fd6007e74cd82e037b800186422fc2da167c747ef045e5d18a5f5d4300f8e1a0291c');
		> "0x2c7536E3605D9C16a7a3D7b1898e529396a65c23"
		
		// hash, v, r, s
		web3.eth.accounts.recover('0x1da44b586eb0729ff70a73c326926f6ed5a25f5b056e7f47fbc6e58d86871655', '0x1c', '0xb91467e570a6466aa9e9876cbcd013baba02900b8979d43fe208a4a4f339f5fd', '0x6007e74cd82e037b800186422fc2da167c747ef045e5d18a5f5d4300f8e1a029');
		> "0x2c7536E3605D9C16a7a3D7b1898e529396a65c23"				
### web3.eth.accounts.encrypt – 加密指定私钥
将私钥加密变换为 keystore v3 标准格式。

- 调用：

		web3.eth.accounts.encrypt(privateKey, password);
- 参数：
	- privateKey – String: 要加密的私钥.
	- password – String: 用于加密的密码
- 返回值：

	Object: 加密后的 keystore v3 JSON.
- 示例代码：

		web3.eth.accounts.encrypt('0x4c0883a69102937d6231471b5dbb6204fe5129617082792ae468d01a3f362318', 'test!')
		> {
		    version: 3,
		    id: '04e9bcbb-96fa-497b-94d1-14df4cd20af6',
		    address: '2c7536e3605d9c16a7a3d7b1898e529396a65c23',
		    crypto: {
		        ciphertext: 'a1c25da3ecde4e6a24f3697251dd15d6208520efc84ad97397e906e6df24d251',
		        cipherparams: { iv: '2885df2b63f7ef247d753c82fa20038a' },
		        cipher: 'aes-128-ctr',
		        kdf: 'scrypt',
		        kdfparams: {
		            dklen: 32,
		            salt: '4531b3c174cc3ff32a6a7a85d6761b410db674807b2d216d022318ceee50be10',
		            n: 262144,
		            r: 8,
		            p: 1
		        },
		        mac: 'b8b010fff37f9ae5559a352a185e86f9b9c1d7f7a9f1bd4e82a5dd35468fc7f6'
		    }
		}

### web3.eth.accounts.decrypt – 解密 keystore 对象
解密给定的 keystore 对象，并创建账户

- 调用：

		web3.eth.accounts.decrypt(keystoreJsonV3, password);
- 参数：
	- encryptedPrivateKey – String: 要解密的keystore私钥
	- password – String: 用来解密的密码
- 返回值：

	Object: 解密的账户对象
- 示例代码：

		web3.eth.accounts.decrypt({
		    version: 3,
		    id: '04e9bcbb-96fa-497b-94d1-14df4cd20af6',
		    address: '2c7536e3605d9c16a7a3d7b1898e529396a65c23',
		    crypto: {
		        ciphertext: 'a1c25da3ecde4e6a24f3697251dd15d6208520efc84ad97397e906e6df24d251',
		        cipherparams: { iv: '2885df2b63f7ef247d753c82fa20038a' },
		        cipher: 'aes-128-ctr',
		        kdf: 'scrypt',
		        kdfparams: {
		            dklen: 32,
		            salt: '4531b3c174cc3ff32a6a7a85d6761b410db674807b2d216d022318ceee50be10',
		            n: 262144,
		            r: 8,
		            p: 1
		        },
		        mac: 'b8b010fff37f9ae5559a352a185e86f9b9c1d7f7a9f1bd4e82a5dd35468fc7f6'
		    }
		}, 'test!');
		> {
		    address: "0x2c7536E3605D9C16a7a3D7b1898e529396a65c23",
		    privateKey: "0x4c0883a69102937d6231471b5dbb6204fe5129617082792ae468d01a3f362318",
		    signTransaction: function(tx){...},
		    sign: function(data){...},
		    encrypt: function(password){...}
		}		

### web3.eth.accounts.wallet – 钱包对象
一个内存钱包，其中包含多个账户。这些账户可以用于web3.eth.sendTransaction()

- 调用：

		web3.eth.accounts.wallet;
- 示例代码：

		web3.eth.accounts.wallet;
		> Wallet {
		    0: {...}, // account by index
		    "0xF0109fC8DF283027b6285cc889F5aA624EaC1F55": {...},  // 相同的账户地址
		    "0xf0109fc8df283027b6285cc889f5aa624eac1f55": {...},  // 相同的账户地址小写
		    1: {...},
		    "0xD0122fC8DF283027b6285cc889F5aA624EaC1d23": {...},
		    "0xd0122fc8df283027b6285cc889f5aa624eac1d23": {...},
		
		    add: function(){},
		    remove: function(){},
		    save: function(){},
		    load: function(){},
		    clear: function(){},
		
		    length: 2,
		}

### web3.eth.accounts.wallet.create – 在钱包中创建账户
在钱包中创建一个或多个账户。不会覆盖已经存在的钱包。

- 调用

		web3.eth.accounts.wallet.create(numberOfAccounts [, entropy]);
- 参数
	- numberOfAccounts – Number: 要创建的账户数量。设为空值则创建空钱包
	- entropy – String:可选，创建账户时使用该随机字符串增加熵，至少32字符长度
- 返回值：

	Object: 钱包对象
- 示例代码：

		web3.eth.accounts.wallet.create(2, '54674321§3456764321§345674321§3453647544±±±§±±±!!!43534534534534');
		> Wallet {
		    0: {...},
		    "0xF0109fC8DF283027b6285cc889F5aA624EaC1F55": {...},
		    "0xf0109fc8df283027b6285cc889f5aa624eac1f55": {...},
		    ...
		}

### web3.eth.accounts.wallet.add – 向钱包添加已有账户
使用私钥或账户对象向钱包中添加一个账户。

- 调用：

		web3.eth.accounts.wallet.add(account);
- 参数：

	account – String|Object: 私钥，或者使用 web3.eth.accounts.create() 创建的账户对象
- 返回值：

	Object: 被添加的账户
- 示例代码：

		web3.eth.accounts.wallet.add('0x4c0883a69102937d6231471b5dbb6204fe5129617082792ae468d01a3f362318');
		> {
		    index: 0,
		    address: '0x2c7536E3605D9C16a7a3D7b1898e529396a65c23',
		    privateKey: '0x4c0883a69102937d6231471b5dbb6204fe5129617082792ae468d01a3f362318',
		    signTransaction: function(tx){...},
		    sign: function(data){...},
		    encrypt: function(password){...}
		}
		
		web3.eth.accounts.wallet.add({
		    privateKey: '0x348ce564d427a3311b6536bbcff9390d69395b06ed6c486954e971d960fe8709',
		    address: '0xb8CE9ab6943e0eCED004cDe8e3bBed6568B2Fa01'
		});
		> {
		    index: 0,
		    address: '0xb8CE9ab6943e0eCED004cDe8e3bBed6568B2Fa01',
		    privateKey: '0x348ce564d427a3311b6536bbcff9390d69395b06ed6c486954e971d960fe8709',
		    signTransaction: function(tx){...},
		    sign: function(data){...},
		    encrypt: function(password){...}
		}
		
### web3.eth.accounts.wallet.remove – 从钱包中移除指定账户
从钱包中移除一个账户。

- 调用：

		web3.eth.accounts.wallet.remove(account);
- 参数：

	account – String|Number: 账户地址，或在钱包中的索引
- 返回值：

	Boolean: 移除成功则返回true，否则返回false
- 示例代码：

		web3.eth.accounts.wallet;
		> Wallet {
		    0: {...},
		    "0xF0109fC8DF283027b6285cc889F5aA624EaC1F55": {...}
		    1: {...},
		    "0xb8CE9ab6943e0eCED004cDe8e3bBed6568B2Fa01": {...}
		    ...
		}
		
		web3.eth.accounts.wallet.remove('0xF0109fC8DF283027b6285cc889F5aA624EaC1F55');
		> true
		
		web3.eth.accounts.wallet.remove(3);
		> false

### web3.eth.accounts.wallet.clear – 清空钱包
安全地清空钱包并移除全部账户。

- 调用：

		web3.eth.accounts.wallet.clear();
- 返回值：

	Object: 钱包对象
- 示例代码：

		web3.eth.accounts.wallet.clear();
		> Wallet {
		    add: function(){},
		    remove: function(){},
		    save: function(){},
		    load: function(){},
		    clear: function(){},
		
		    length: 0
		}										

### web3.eth.accounts.wallet.encrypt – 加密钱包
加密所有的钱包账户为 keystore v3 对象。

- 调用：

		web3.eth.accounts.wallet.encrypt(password);
- 参数：

		password – String: The password which will be used for encryption.
- 返回值：

	Array: 加密后的keystore v3对象
- 示例代码：

		web3.eth.accounts.wallet.encrypt('test');
		> [ { version: 3,
		    id: 'dcf8ab05-a314-4e37-b972-bf9b86f91372',
		    address: '06f702337909c06c82b09b7a22f0a2f0855d1f68',
		    crypto:
		     { ciphertext: '0de804dc63940820f6b3334e5a4bfc8214e27fb30bb7e9b7b74b25cd7eb5c604',
		       cipherparams: [Object],
		       cipher: 'aes-128-ctr',
		       kdf: 'scrypt',
		       kdfparams: [Object],
		       mac: 'b2aac1485bd6ee1928665642bf8eae9ddfbc039c3a673658933d320bac6952e3' } },
		  { version: 3,
		    id: '9e1c7d24-b919-4428-b10e-0f3ef79f7cf0',
		    address: 'b5d89661b59a9af0b34f58d19138baa2de48baaf',
		    crypto:
		     { ciphertext: 'd705ebed2a136d9e4db7e5ae70ed1f69d6a57370d5fbe06281eb07615f404410',
		       cipherparams: [Object],
		       cipher: 'aes-128-ctr',
		       kdf: 'scrypt',
		       kdfparams: [Object],
		       mac: 'af9eca5eb01b0f70e909f824f0e7cdb90c350a802f04a9f6afe056602b92272b' } }
		]

### web3.eth.accounts.wallet.decrypt – 解密钱包
解密keystore v3对象。

- 调用：

		web3.eth.accounts.wallet.decrypt(keystoreArray, password);
- 参数：
	- keystoreArray – Array: 要解密的keystore v3对象
	- password – String: 用来解密的密码
- 返回值：

	Object: 钱包对象
- 示例代码:

		web3.eth.accounts.wallet.decrypt([
		  { version: 3,
		  id: '83191a81-aaca-451f-b63d-0c5f3b849289',
		  address: '06f702337909c06c82b09b7a22f0a2f0855d1f68',
		  crypto:
		   { ciphertext: '7d34deae112841fba86e3e6cf08f5398dda323a8e4d29332621534e2c4069e8d',
		     cipherparams: { iv: '497f4d26997a84d570778eae874b2333' },
		     cipher: 'aes-128-ctr',
		     kdf: 'scrypt',
		     kdfparams:
		      { dklen: 32,
		        salt: '208dd732a27aa4803bb760228dff18515d5313fd085bbce60594a3919ae2d88d',
		        n: 262144,
		        r: 8,
		        p: 1 },
		     mac: '0062a853de302513c57bfe3108ab493733034bf3cb313326f42cf26ea2619cf9' } },
		   { version: 3,
		  id: '7d6b91fa-3611-407b-b16b-396efb28f97e',
		  address: 'b5d89661b59a9af0b34f58d19138baa2de48baaf',
		  crypto:
		   { ciphertext: 'cb9712d1982ff89f571fa5dbef447f14b7e5f142232bd2a913aac833730eeb43',
		     cipherparams: { iv: '8cccb91cb84e435437f7282ec2ffd2db' },
		     cipher: 'aes-128-ctr',
		     kdf: 'scrypt',
		     kdfparams:
		      { dklen: 32,
		        salt: '08ba6736363c5586434cd5b895e6fe41ea7db4785bd9b901dedce77a1514e8b8',
		        n: 262144,
		        r: 8,
		        p: 1 },
		     mac: 'd2eb068b37e2df55f56fa97a2bf4f55e072bef0dd703bfd917717d9dc54510f0' } }
		], 'test');
		> Wallet {
		    0: {...},
		    1: {...},
		    "0xF0109fC8DF283027b6285cc889F5aA624EaC1F55": {...},
		    "0xD0122fC8DF283027b6285cc889F5aA624EaC1d23": {...}
		    ...
		}
		
### web3.eth.accounts.wallet.save – 保存钱包
### web3.eth.accounts.wallet.load – 载入钱包
从本地存储器载入钱包并解密。注意，仅在浏览器中有效。

- 调用：

		web3.eth.accounts.wallet.load(password [, keyName]);
- 参数：
	- password – String: 用来解密钱包的密码
	- keyName – String: 可选，在本地存储时的键，默认值为"web3js_wallet"
- 返回值：

	Object: 钱包对象
- 示例代码：

		web3.eth.accounts.wallet.load('test#!$', 'myWalletKey');
		> Wallet {
		    0: {...},
		    1: {...},
		    "0xF0109fC8DF283027b6285cc889F5aA624EaC1F55": {...},
		    "0xD0122fC8DF283027b6285cc889F5aA624EaC1d23": {...}
		    ...
		}

## 账户交互
### web3.eth.personal – 账户交互
使用 web3-eth-personal 包和以太坊节点账户进行交互。

	注意，这个包中的许多函数包含敏感信息，例如密码，因此不要在未加密的 websocket 或 http 服务提供器上调用这些函数，因为你的密码是明文发送的！

- 使用方法：

		var Personal = require('web3-eth-personal');
		
		// 在以太坊兼容浏览器中，"Personal.providers.givenProvider"将自动被设置
		var personal = new Personal(Personal.givenProvider || 'ws://some.local-or-remote.node:8546');
		
		
		// 也可以使用web3包
		
		var Web3 = require('web3');
		var web3 = new Web3(Web3.givenProvider || 'ws://some.local-or-remote.node:8546');
		
		// -> web3.eth.personal

### web3.eth.personal.newAccount – 创建新账户
创建一个新账户。

	注意，不要在未加密的 websocket 或 http 提供器上调用该函数，否则你的密码将泄漏！
- 调用：

		web3.eth.personal.newAccount(password, [callback])
- 参数：

	password – String: 用来加密账户的密码
- 返回值：

	一个Promise对象，其解析值为新创建的账户。
- 示例代码：

		web3.eth.personal.newAccount('!@superpassword')
		.then(console.log);
		> '0x1234567891011121314151617181920212223456'

### web3.eth.personal.sign – 为数据生成签名
使用指定账户对数据进行签名。

	注意，在未加密的HTTP连接上发送账户密码极其不安全！
- 调用：

		web3.eth.personal.sign(dataToSign, address, password [, callback])
- 参数：
	- dataToSign：String – 要签名的数据。会先使用web3.utils.utf8ToHex()将字符串转化为16进制
	- address：String – 用来签名的账户地址
	- password：String – 用来签名的账户密码
	- callback：Function – 可选的回调函数，其第一个参数为错误对象，第二个参数为结果签名
- 返回值：

	一个Promise对象，其解析值为签名结果字符串。
- 示例代码：

		web3.eth.personal.sign("Hello world", "0x11f4d0A3c12e86B4b5F39B213F7E19D048276DAe", "test password!")
		.then(console.log);
		> "0x30755ed65396facf86c53e6217c52b4daebe72aa4941d89635409de4c9c7f9466d4e9aaec7977f05e923889b33c0d0dd27d7226b6e6f56ce737465c5cfd04be400"
		
		// 下面代码实现同样功能
		web3.eth.personal.sign(web3.utils.utf8ToHex("Hello world"), "0x11f4d0A3c12e86B4b5F39B213F7E19D048276DAe", "test password!")
		.then(console.log);
		> "0x30755ed65396facf86c53e6217c52b4daebe72aa4941d89635409de4c9c7f9466d4e9aaec7977f05e923889b33c0d0dd27d7226b6e6f56ce737465c5cfd04be400"											
### web3.eth.personal.ecRecover – 提取数据的签名账户
从签名的数据中提取进行签名的账户地址。

- 调用

		web3.eth.personal.ecRecover(dataThatWasSigned, signature [, callback])
- 参数
	- dataThatWasSigned：String – 带有签名的数据，首先使用web3.utils.utf8ToHex()转化为16进制字符串
	- signature：String – 签名
	- Function – 可选的回调函数，其第一个参数为错误对象，第二个参数为结果
- 返回值

	一个Promise对象，其解析值为账户地址字符串
- 示例代码：

		web3.eth.personal.ecRecover("Hello world", "0x30755ed65396facf86c53e6217c52b4daebe72aa4941d89635409de4c9c7f9466d4e9aaec7977f05e923889b33c0d0dd27d7226b6e6f56ce737465c5cfd04be400").then(console.log);
		> "0x11f4d0A3c12e86B4b5F39B213F7E19D048276DAe"
		
### web3.eth.personal.signTransaction – 为交易生成签名
对交易进行签名，账户必须先解锁。

	注意：在未加密的HTTP连接上发送账户密码有巨大的风险！
- 调用

		web3.eth.personal.signTransaction(transaction, password [, callback])
- 参数
	- transaction：Object – 更多信息，请访问签署 web3.eth.sendTransaction() 的事务数据。
	- password：String – 从帐户的密码，签署交易。
	- callback：Function –(可选)可选的回调函数，第一个参数返回一个错误对象，第二个参数返回一个结果。
- 返回值：

	一个 Promise 对象，其解析值为 RLP 编码的交易对象，其 raw 属性可以使用 web3.eth.sendSignedTransaction() 发送交易。
- 示例代码：

		web3.eth.signTransaction({
		    from: "0xEB014f8c8B418Db6b45774c326A0E64C78914dC0",
		    gasPrice: "20000000000",
		    gas: "21000",
		    to: '0x3535353535353535353535353535353535353535',
		    value: "1000000000000000000",
		    data: ""
		}, 'MyPassword!').then(console.log);
		> {
		    raw: '0xf86c808504a817c800825208943535353535353535353535353535353535353535880de0b6b3a76400008025a04f4c17305743700648bc4f6cd3038ec6f6af0df73e31757007b7f59df7bee88da07e1941b264348e80c78c4027afc65a87b0a5e43e86742b8ca0823584c6788fd0',
		    tx: {
		        nonce: '0x0',
		        gasPrice: '0x4a817c800',
		        gas: '0x5208',
		        to: '0x3535353535353535353535353535353535353535',
		        value: '0xde0b6b3a7640000',
		        input: '0x',
		        v: '0x25',
		        r: '0x4f4c17305743700648bc4f6cd3038ec6f6af0df73e31757007b7f59df7bee88d',
		        s: '0x7e1941b264348e80c78c4027afc65a87b0a5e43e86742b8ca0823584c6788fd0',
		        hash: '0xda3be87732110de6c1354c83770aae630ede9ac308d9f7b399ecfba23d923384'
		    }
		}

## web3.eth.abi – ABI管理
web3.eth.abi 系列函数用来将参数编码为 ABI (Application Binary Interface)，或者从 ABI 解码回来。

以便在以太坊虚拟机 EVM 上执行函数函数调用。

函数列表：

- web3.eth.abi.encodeFunctionSignature
- web3.eth.abi.encodeEventSignature
- web3.eth.abi.encodeParameter
- web3.eth.abi.encodeParameters
- web3.eth.abi.encodeFunctionCall
- web3.eth.abi.decodeParameter
- web3.eth.abi.decodeParameters
- web3.eth.abi.decodeLog

### web3.eth.abi.encodeFunctionSignature – 函数编码
将函数名编码为 ABI 签名，方法是取函数名及参数类型的sha3哈希值的头4个字节。

- 调用

		web3.eth.abi.encodeFunctionSignature(functionName);
- 参数

	functionName – String|Object: 要编码的函数名字符串，或者函数的JSON接口对象。当采用字符串时，必须采用function(type,type,...)的格式，例如: myFunction(uint256,uint32[],bytes10,bytes)。
- 返回值

	String – 函数的ABI签名
- 示例代码：

		// 传入JSON接口对象
		web3.eth.abi.encodeFunctionSignature({
		    name: 'myMethod',
		    type: 'function',
		    inputs: [{
		        type: 'uint256',
		        name: 'myNumber'
		    },{
		        type: 'string',
		        name: 'myString'
		    }]
		})
		> 0x24ee0097
		
		// 传入字符串
		web3.eth.abi.encodeFunctionSignature('myMethod(uint256,string)')
		> '0x24ee0097'

### web3.eth.abi.encodeEventSignature – 事件编码
将事件名编码为ABI签名，方法是取事件名及其参数类型的sha3哈希值。

- 调用：

		web3.eth.abi.encodeEventSignature(eventName);
- 参数：
	- eventName – String|Object: 要编码的事件名字符串，或者事件的JSON接口对象。如果采用字符串参数，则需要符合格式event(type,type,...) ，例如myEvent(uint256,uint32[],bytes10,bytes)。
- 返回值：

	String – 事件的ABI签名
- 示例代码：

		// 使用字符串参数
		web3.eth.abi.encodeEventSignature('myEvent(uint256,bytes32)')
		> 0xf2eeb729e636a8cb783be044acf6b7b1e2c5863735b60d6daae84c366ee87d97
		
		// 使用json接口对象
		web3.eth.abi.encodeEventSignature({
		    name: 'myEvent',
		    type: 'event',
		    inputs: [{
		        type: 'uint256',
		        name: 'myNumber'
		    },{
		        type: 'bytes32',
		        name: 'myBytes'
		    }]
		})
		> 0xf2eeb729e636a8cb783be044acf6b7b1e2c5863735b60d6daae84c366ee87d97

### web3.eth.abi.encodeFunctionCall – 函数调用编码
将函数调用根据其 JSON 接口对象和给定的参数进行ABI编码。

- 调用

		web3.eth.abi.encodeFunctionCall(jsonInterface, parameters);
- 参数
	- jsonInterface – Object: 函数的JSON接口对象
	- parameters – Array: 参数值数组
- 返回值
	- String – ABI编码结果，包含函数签名和参数
- 示例代码

		web3.eth.abi.encodeFunctionCall({
		    name: 'myMethod',
		    type: 'function',
		    inputs: [{
		        type: 'uint256',
		        name: 'myNumber'
		    },{
		        type: 'string',
		        name: 'myString'
		    }]
		}, ['2345675643', 'Hello!%']);
		> "0x24ee0097000000000000000000000000000000000000000000000000000000008bd02b7b0000000000000000000000000000000000000000000000000000000000000040000000000000000000000000000000000000000000000000000000000000000748656c6c6f212500000000000000000000000000000000000000000000000000"
		
### web3.eth.abi.decodeParameter – 参数解码
将 ABI 编码过的参数解码为其 JavaScript 形式。

- 调用：

		web3.eth.abi.decodeParameter(type, hexString);
- 参数
	- type – String: 参数类型
	- hexString – String: 要解码的ABI字节码
- 返回值：

	Mixed – 解码结果
- 示例代码：

		web3.eth.abi.decodeParameter('uint256', '0x0000000000000000000000000000000000000000000000000000000000000010');
		> "16"
		
		web3.eth.abi.decodeParameter('string', '0x0000000000000000000000000000000000000000000000000000000000000020000000000000000000000000000000000000000000000000000000000000000848656c6c6f212521000000000000000000000000000000000000000000000000');
		> "Hello!%!"
		
### web3.eth.abi.decodeParameters – 参数组解码
将 ABI 编码的参数解码为其 JavaScript 形式。

- 调用：

		web3.eth.abi.decodeParameters(typesArray, hexString);
- 参数
	- typesArray – Array|Object: 参数类型数组，或JSON接口输出数组
	- hexString – String: 要解码的ABI字节码
- 返回值：

	Object – 包含解码后参数值的对象
- 示例代码：

		web3.eth.abi.decodeParameters(['string', 'uint256'], '0x000000000000000000000000000000000000000000000000000000000000004000000000000000000000000000000000000000000000000000000000000000ea000000000000000000000000000000000000000000000000000000000000000848656c6c6f212521000000000000000000000000000000000000000000000000');
		> Result { '0': 'Hello!%!', '1': '234' }
		
		web3.eth.abi.decodeParameters([{
		    type: 'string',
		    name: 'myString'
		},{
		    type: 'uint256',
		    name: 'myNumber'
		}], '0x000000000000000000000000000000000000000000000000000000000000004000000000000000000000000000000000000000000000000000000000000000ea000000000000000000000000000000000000000000000000000000000000000848656c6c6f212521000000000000000000000000000000000000000000000000');
		> Result {
		    '0': 'Hello!%!',
		    '1': '234',
		    myString: 'Hello!%!',
		    myNumber: '234'
		}

### web3.eth.abi.decodeLog – 日志解码
对ABI编码后的日志数据和索引的主题数据进行解码。

- 调用

		web3.eth.abi.decodeLog(inputs, hexString, topics);
- 参数
	- inputs – Object: JSON 接口输入数组。有关类型的列表，请参阅solid文档。
	- hexString – String: 日志数据字段中的ABI字节码。
	- topics – Array: 一个索引参数为日志主题的数组，如果是一个非匿名事件则不带主题[0]，否则带主题[0]。
- 返回值：

	Object – 包含解码后参数的对象
- 示例代码：

		web3.eth.abi.decodeLog([{
		    type: 'string',
		    name: 'myString'
		},{
		    type: 'uint256',
		    name: 'myNumber',
		    indexed: true
		},{
		    type: 'uint8',
		    name: 'mySmallNumber',
		    indexed: true
		}],
		'0x0000000000000000000000000000000000000000000000000000000000000020000000000000000000000000000000000000000000000000000000000000000748656c6c6f252100000000000000000000000000000000000000000000000000',
		['0x000000000000000000000000000000000000000000000000000000000000f310', '0x0000000000000000000000000000000000000000000000000000000000000010']);
		> Result {
		    '0': 'Hello%!',
		    '1': '62224',
		    '2': '16',
		    myString: 'Hello%!',
		    myNumber: '62224',
		    mySmallNumber: '16'
		}

## web3.utils – 辅助工具函数集
web3.utils 属性包含一组辅助函数集。

- 调用方法
	- Web3.utils
	- web3.utils

### web3.utils.randomHex – 生成伪随机16进制字符串
生成指定长度的密码学强伪随机16进制字符串。

- 调用：

		web3.utils.randomHex(size)
- 参数：
	- size – Number: 生成长度，例如32表示要生成32字节长的16进制字符串，即64个字符以及前缀“0x”
- 返回值：

	String: 生成的随机16进制字符串
- 示例代码：

		web3.utils.randomHex(32)
		> "0xa5b9d60f32436310afebcfda832817a68921beb782fabf7915cc0460b443116a"
		
		web3.utils.randomHex(4)
		> "0x6892ffc6"
		
		web3.utils.randomHex(2)
		> "0x99d6"
		
		web3.utils.randomHex(1)
		> "0x9a"
		
		web3.utils.randomHex(0)
		> "0x" 
		
### web3.utils._ – underscore接口
web3.utils._提供了访问 underscore 库的接口。underscore 是一个非常流行的 javascript 工具库，提供了很多方便的 js 函数。

请参考 underscore 的API文档查询更多细节信息。

- 调用：

		web3.utils._
- 示例代码：

		var _ = web3.utils._;
		
		_.union([1,2],[3]);
		> [1,2,3]
		
		_.each({my: 'object'}, function(value, key){ ... })

### web3.utils.BN – BN.js接口
web3.utils.BN 提供了 BN.js 库的访问接口。BN.js 库用来处理大数的计算。

请参考 BN.js 的文档获取更详细的信息。

	注意，为了安全地进行类型转换，请使用 utils.toBN。

- 调用：

		web3.utils.BN(mixed)
- 参数：

	mixed – String|Number: 数值字符串或16进制字符串
- 返回值：

	Object: BN.js实例对象
- 实例：

		var BN = web3.utils.BN;
		
		new BN(1234).toString();
		> "1234"
		
		new BN('1234').add(new BN('1')).toString();
		> "1235"
		
		new BN('0xea').toString();
		> "234"									
		
### web3.utils.isBN – 检查给定参数是否BN对象
web3.utils.isBN() 方法用来检查给定的参数是否是一个 BN.js 实例对象。

- 调用：

		web3.utils.isBN(bn)
- 参数：

	bn – Object: 要检查的对象
- 返回值

	Boolean：如果参数为BN对象则返回true，否则返回false
- 实例代码：

		var number = new BN(10);
		
		web3.utils.isBN(number);
		> true

### web3.utils.isBigNumber – 检查给定参数是否为BigNumber对象
使用 web3.utils.isBigNumber() 方法检查给定的参数是否是一个 BigNumber.js 对象表示的大数。

- 调用：

		web3.utils.isBigNumber(bignumber)
- 参数：
	- bignumber – Object: 要检查的对象
- 返回值：
	- Boolean：如果参数是BigNumber.js对象则返回true，否则返回false
- 实例代码：

		var number = new BigNumber(10);
		
		web3.utils.isBigNumber(number);
		> true

### web3.utils.sha3 – 计算sha3哈希值
使用web3.utils.sha3()方法计算给定字符串的sha3哈希值。

	注意，如果要模拟solidity中的sha3，请使用soliditySha3函数。
- 调用：

		web3.utils.sha3(string)
		web3.utils.keccak256(string) // ALIAS
- 参数：
	- string – String: 要计算sha3哈希值的字符串
- 返回值：
	- String: 计算结果哈希值
- 实例代码：

		web3.utils.sha3('234'); // 字符串参数
		> "0xc1912fee45d61c87cc5ea59dae311904cd86b84fee17cc96966216f811ce6a79"
		
		web3.utils.sha3(new BN('234')); // BN对象参数
		> "0xbc36789e7a1e281436464229828f817d6612f7b477d66591ff96a9e064bcc98a"
		
		web3.utils.sha3(234); 
		> null // 不能计算数值类型的哈希值
		
		web3.utils.sha3(0xea); // 同上，也不能计算16进制表示的数值
		> null
		
		web3.utils.sha3('0xea'); // 首先转化为字节数组，然后再计算哈希值
		> "0x2f20677459120677484f7104c76deb6846a2c071f9b3152c103bb12cd54d1a4a"

### web3.utils.soliditySha3 – solidity 方式计算 sha3 哈希
采用和 solidity 同样的方式计算给定参数的 sha3 哈希值，也就是说，

在计算哈希之前，需要首先对参数进行ABI编码，并进行字节紧凑化处理。

调用：

web3.utils.soliditySha3(param1 [, param2, ...])
参数：

paramX – Mixed: 任意类型，或是一个具有如下结构的对象：{type: 'uint', value: '123456'} 或 {t: 'bytes', v: '0xfff456'}。
可以自动识别基本数据类型并解析为对应的solidity类型：

String 非数值的UTF-8字符串将解释为string
String|Number|BN|HEX 正数将解释为uint256
String|Number|BN 负数将解释为int256
Boolean 解释为bool.
String HEX 具有前缀0x的字符串将解释为bytes
HEX 16进制表示的数值将解释为uint256.
返回值：

String: 结果哈希值

示例代码：

web3.utils.soliditySha3('234564535', '0xfff23243', true, -10);
// 自动检测:        uint256,      bytes,     bool,   int256
> "0x3e27a893dc40ef8a7f0841d96639de2f58a132be5ae466d40087a2cfa83b7179"


web3.utils.soliditySha3('Hello!%'); // 自动检测: string
> "0x661136a4267dba9ccdf6bfddb7c00e714de936674c4bdb065a531cf1cb15c7fc"


web3.utils.soliditySha3('234'); // 自动检测: uint256
> "0x61c831beab28d67d1bb40b5ae1a11e2757fa842f031a2d0bc94a7867bc5d26c2"

web3.utils.soliditySha3(0xea); // 同上
> "0x61c831beab28d67d1bb40b5ae1a11e2757fa842f031a2d0bc94a7867bc5d26c2"

web3.utils.soliditySha3(new BN('234')); // 同上
> "0x61c831beab28d67d1bb40b5ae1a11e2757fa842f031a2d0bc94a7867bc5d26c2"

web3.utils.soliditySha3({type: 'uint256', value: '234'})); // 同上
> "0x61c831beab28d67d1bb40b5ae1a11e2757fa842f031a2d0bc94a7867bc5d26c2"

web3.utils.soliditySha3({t: 'uint', v: new BN('234')})); // 同上
> "0x61c831beab28d67d1bb40b5ae1a11e2757fa842f031a2d0bc94a7867bc5d26c2"


web3.utils.soliditySha3('0x407D73d8a49eeb85D32Cf465507dd71d507100c1');
> "0x4e8ebbefa452077428f93c9520d3edd60594ff452a29ac7d2ccc11d47f3ab95b"

web3.utils.soliditySha3({t: 'bytes', v: '0x407D73d8a49eeb85D32Cf465507dd71d507100c1'});
> "0x4e8ebbefa452077428f93c9520d3edd60594ff452a29ac7d2ccc11d47f3ab95b" // 结果同上


web3.utils.soliditySha3({t: 'address', v: '0x407D73d8a49eeb85D32Cf465507dd71d507100c1'});
> "0x4e8ebbefa452077428f93c9520d3edd60594ff452a29ac7d2ccc11d47f3ab95b" // 同上，但首先会检查校验和


web3.utils.soliditySha3({t: 'bytes32', v: '0x407D73d8a49eeb85D32Cf465507dd71d507100c1'});
> "0x3c69a194aaf415ba5d6afca734660d0a3d45acdc05d54cd1ca89a8988e7625b4" // 不同的类型得到不同的哈希值


web3.utils.soliditySha3({t: 'string', v: 'Hello!%'}, {t: 'int8', v:-23}, {t: 'address', v: '0x85F43D8a49eeB85d32Cf465507DD71d507100C1d'});
> "0xa13b31627c1ed7aaded5aecec71baf02fe123797fffd45e662eac8e06fbe4955"																	
						