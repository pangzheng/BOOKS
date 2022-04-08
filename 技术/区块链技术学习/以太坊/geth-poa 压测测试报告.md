# geth-poa 性能测试报告
## 测试目的
- 在一定的硬件基础上，得到 geth-poa 的性能基准
- 通过调节 eth 的参数，尝试进一步增加 geth 的 tps 和稳定性

## 压力测试工具
[chainhammer](https://github.com/drandreaskrueger/chainhammer)
## 基础数据推导
- 测试数据

	![](./pic/geth-performance-nullbolck.png)

	从上图数据可以推导数据如下

	- 单空块大小为 609 字节
	- 如果2秒出一个块
		- 25MB/24小时
		- 8GB/365 天 
		- 80GB/10年 
		- 400GB/50年
	- 如果5秒出一个块
		- 10MB/24小时
		- 3.56GB/365 天 
		- 35.6GB/10年 
		- 178GB/50年 
	
	单节点区块链节点存储盘为2TB比较合理	
- 压测 tx 合约数据基准
	- 测试合约
		- tx
			- 26,455/gasuse
			- 1134tx/块(数据预测)
				- 2秒/块
					- 567/tps 
				- 5秒/块
					- 226/tps
			- max 1131tx/块(数据实测)
				- 2秒/块
					- 401/tps 
				- 5秒/块
					- 232/tps  
		    
- 基金会 tx 合约数据基准
	- 721
		- [mint/铸币](https://test-sctscan.netwarps.com/tx/0xed82ffa7c0b11f0197e81413d9912b0b135c3ccfdd6dc13dc47b7c90c136f937/token-transfers)
			- 222,283/gasuse
			- 144 tx/块
				- 2秒1块
					- 72/tps
				- 5秒1块
					- 28/tps
		- [gift/赠予](https://test-sctscan.netwarps.com/tx/0x8c268f2b5e16cebaa842057f0d2ab73eb11f23f101649c4653849c8858781f7d/token-transfers)   
			- 55,810/gasuse
			- 537 tx/块
				- 2秒1块
					- 268/tps
				- 5秒1块
					- 107/tps  
	- 1155
		- [mint/铸币](https://test-sctscan.netwarps.com/tx/0x15233ad7c7d3df527bd385fce5c0a78f80e9e58b979b140e236f65bd3ac80962/token-transfers)
			- 223,305/gasuse
			- 134 tx/块
				- 2秒1块
					- 67/tps
				- 5秒1块
					- 26/tps
		- [gift/赠予](https://test-sctscan.netwarps.com/tx/0x6bbe1c375a837ff06399dd0ffd7cef398b556978b250f5d1ea81b33217de48d5/token-transfers)   
			- 59,841/gasuse
			- 501 tx/块
				- 2秒1块
					- 250/tps
				- 5秒1块
					- 100/tps
		- [Burning/销毁](https://test-sctscan.netwarps.com/tx/0xb7e947fa3e6af9e75d4b0ef9314e944bf353d09e29b8f0d5d5a204e3298067ad/token-transfers)   
			- 36,825/gasuse
			- 814 tx/块
				- 2秒1块
					- 407/tps
				- 5秒1块
					- 162/tps    

#### geth 设置参数参考
-  poa 公链
	- 5 秒一个块
	- 每个块 3000万 gaslimit
-  polycon 公链
	- 2 秒一个块
	- 每个块 3000万 gaslimit

## 单机压力
### 测试硬件条件
- geth 共识节点
	- 硬件
		- cpu*4
		- 内存*8
		- 磁盘*100/ucloud rssd
		- 千兆网络+hostport 
	- 操作系统
		- Rocky Linux 8.5
	- 软件版本
		- Geth/v1.10.17-stable-25c9b49f/linux-amd64/go1.18
- 客户端节点
	- 硬件
		- cpu*2
		- 内存*4
		- 磁盘*20/ucloud rssd
		- 千兆网络+hostport 
	-  操作系统
		-  ubuntu 20.04
	- 软件版本
		- commit  ee5a31e2ec7d7ef7eb37bba5c378168120a77d25 
		
### 测试第一轮
#### eth 参数
- 基础设置参考 poa 公链
	- 5秒一个块
	- 每个块 3000万 gaslimit
- 详细参数
	- 命令行
		
			command: ["geth"]
			  args:
			    - --config
			    - /root/config/config.custom.toml
			    - --http
			    - --ws
			    - --allow-insecure-unlock
			    - --rpc.allow-unprotected-txs
			    - --gcmode=archive
			    - --unlock
			    - 631cb5db2476e55e5886e0a7d1cdc02701c4a581
			    - --password
			    - /root/.ethereum/password
			    - --mine
			    - --nodiscover
			    - --metrics
			    - --metrics.addr=0.0.0.0
			    - --miner.gaslimit=30000000
	- 配置	
		
			config: |
			  [Eth]
			  NetworkId = 65533
			  SyncMode = "full"
			
			  [Node]
			  HTTPHost = "0.0.0.0"
			  HTTPModules = ["eth", "net", "web3", "personal", "txpool"]
			
			  [Metrics]
			  HTTP = "0.0.0.0"
			
			  [Node.P2P]
			  StaticNodes = []
			  TrustedNodes = []
	- 创始块配置	
		
		创始块配置按照每5秒一个块，每个块 3000w gaslimit 的 poa 公链设置
		
			genesis: '{"config":{"chainId": 65533,"homesteadBlock":0,"eip150Block":0,"eip150Hash":"0x0000000000000000000000000000000000000000000000000000000000000000","eip155Block":0,"eip158Block":0,"byzantiumBlock":0,"constantinopleBlock":0,"petersburgBlock":0,"istanbulBlock":0,"clique":{"period":5,"epoch":30000}},"extradata":"0x0000000000000000000000000000000000000000000000000000000000000000631cb5db2476e55e5886e0a7d1cdc02701c4a5810000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000","gasLimit":"30000000","coinbase":"0x0000000000000000000000000000000000000000","difficulty":"0x1","mixHash":"0x0000000000000000000000000000000000000000000000000000000000000000","alloc":{"0000000000000000000000000000000000000000":{"balance":"0x1"},"631cb5db2476e55e5886e0a7d1cdc02701c4a581":{"balance":"0x200000000000000000000000000000000000000000000000000000000000000"}},"nonce":"0x0000000000000123","number":"0x0","gasUsed":"0x0","parentHash":"0x0000000000000000000000000000000000000000000000000000000000000000","baseFeePerGas":null}'
			key: '{"address":"631cb5db2476e55e5886e0a7d1cdc02701c4a581","crypto":{"cipher":"aes-128-ctr","ciphertext":"67b2fe8cf0acb5b8221883a1219e5cc46b9a5345bddad481a1775f4bdc2d37f6","cipherparams":{"iv":"f351d571f83ff30c25e3ded259efaa67"},"kdf":"scrypt","kdfparams":{"dklen":32,"n":262144,"p":1,"r":8,"salt":"c99e8936a62de9719501d9e3f38c08a896820d5dd9c7689195fe84a079b42250"},"mac":"9ccf8654d97369bd9acb7bec5eb72f1e56e84d781aa86501b9407270e3b1eeeb"},"id":"18a30473-c74e-4f6b-a2e2-d11949c70d03","version":3}'      

#### 测试结果
- 从软硬件测试结果看
	- 硬件

		从资源监控看，硬件除了 CPU 使用率高、内存较高外，其他硬件均较为正常
	- 软件
		- 总共测试交易 19440000 
		- 测试区块
			- 2241 是创建测试合约 
			- 2242-19360，但实际是19430(问题有待查询),约等于24小时
		- 最终平均是 232/s (注意这个测试使用1个账户测试，多用户可能不同) 
		- 失败交易待查？？
- 测试中掺杂真实转账交易
	
	中间测试过程掺杂了真实转账交易，会进入pending状态，必须等压测完成重新激活才可以发布，这样证明该压力情况下，正常业务已经被堵塞。但是压力过后会马上恢复
- 实际情况可能性能相对有所下降需要测试

	当前测试使用合约和调用均是简单调用，只能测试共识节点 tx 打包能力，以下情况可能影响测试结果
	
	- 多共识节点
	- 只读业务节点接收交易
	- 共识节点与业务节点跨公网
	- 合约复杂度提升	
	
#### 共识节点主机硬件监控
- cpu
	- use
	
		![](./pic/geth-performance-geth-cpu.png)
	- load

		![](./pic/geth-performance-geth-cpuload.png)
- mem

	![](./pic/geth-performance-geth-mem.png)	
- network

	![](./pic/geth-performance-geth-network.png)
- other

	![](./pic/geth-performance-geth-other.png)
	
#### [游览器](http://106.75.120.70:4000/)查询数据
- 抽取块数据
	- 块信息 
	
		![](./pic/geth-performance.png)
	- 块交易列表

		![](./pic/geth-performance-1.png)
- 抽取单条交易数据
	
	 ![](./pic/geth-performance-2.png)

#### 测试客户端结果
	top - 16:52:05 up 8 days,  5:08,  2 users,  load average: 0.43, 0.42, 0.23
	Tasks: 120 total,   1 running,  62 sleeping,   0 stopped,   0 zombie
	%Cpu(s): 24.4 us,  1.7 sy,  0.0 ni, 72.4 id,  0.0 wa,  0.5 hi,  1.0 si,  0.0 st
	KiB Mem :  3832676 total,  2774356 free,   220268 used,   838052 buff/cache
	KiB Swap:        0 total,        0 free,        0 used.  3358528 avail Mem
	
	  PID USER      PR  NI    VIRT    RES    SHR S  %CPU %MEM     TIME+ COMMAND
	14785 root      20   0  190060  76604  10792 S  51.8  2.0   5:11.16 python3
	14772 root      20   0  155452  41924  10624 S   0.3  1.1   0:05.90 python3
#### 服务端结果
- 图形

	![](./pic/geth-performance-clent.png)
- 文本

		(geth-tps) Geth v1.10.17 with 19440000 txs: 0.0 TPS
		information:
		NODE: Geth/v1.10.17-stable-25c9b49f/linux-amd64/go1.18 on http://172.17.57.222:8545
		      consensus=clique chain_name=poa chain_id=65533 network_id=65533
		SEND: 19440000 transactions in blocks 2242-2242 with 10 empty blocks following.
		      A sample of transactions looked as if they: failed (at least partially).
		TPS:  The stopclock watcher measured a final TPS of 232.0 since contract deploy,
		      and in between saw values as high as 227.1 TPS.
		DIAG: The whole experiment was prefixed 'geth-tps'.
		      The diagrams were saved into 'no-diagram-created-because-less-than-2-filled-blocks'.
		      Looking only at the experiment block-timestamps, the overall TPS was ~0.0.
		log:
		versions: web3 4.8.2, py-solc: 3.2.0, solc 0.4.25+commit.59dbf8f1.Linux.gpp, testrpc 1.3.5, python 3.6.9 (default, Dec  8 2021, 21:08:43) [GCC 8.4.0]
		web3 connection established, blockNumber = 2240, node version string =  Geth/v1.10.17-stable-25c9b49f/linux-amd64/go1.18
		first account of node is 0x631Cb5DB2476e55E5886e0a7d1Cdc02701C4A581, balance is 904625697166532776746648320380374280103671755200316906558.262375061821325312 Ether
		nodeName: Geth, nodeType: Geth, nodeVersion: v1.10.17-stable-25c9b49f, consensus: clique, network: 65533, chainName: poa, chainId: 65533
		
		Block  2240  - waiting for something to happen
		(filedate 1648723741) last contract address: 0x51E32B11B9322352Ee00Cca3cd554E346cA56799
		(filedate 1648724021) new contract address: 0x655317DC9af9354221F28bc229F1B1E8d95259f7
		
		blocknumber_start_here = 2241
		starting timer, at block 2241 which has  1  transactions; at epochtime 1648724021.198591
		block 2242 | new #TX 1131 / 5000 ms = 226.2 TPS_current | total: #TX 1132 /  4.9 s = 232.0 TPS_average (peak  is 232.0 TPS_average)
		block 2243 | new #TX 1132 / 5000 ms = 226.4 TPS_current | total: #TX 2264 / 10.1 s = 224.9 TPS_average (peak  is 224.9 TPS_average)
		block 2244 | new #TX 1132 / 5000 ms = 226.4 TPS_current | total: #TX 3396 / 14.9 s = 227.2 TPS_average (peak  is 227.2 TPS_average)
		block 2245 | new #TX 1132 / 5000 ms = 226.4 TPS_current | total: #TX 4528 / 20.1 s = 224.9 TPS_average (peak  is 224.9 TPS_average)
		block 2246 | new #TX 1132 / 5000 ms = 226.4 TPS_current | total: #TX 5660 / 25.0 s = 226.2 TPS_average (peak  is 226.2 TPS_average)
		block 2247 | new #TX 1132 / 5000 ms = 226.4 TPS_current | total: #TX 6792 / 29.9 s = 227.1 TPS_average (peak  is 227.1 TPS_average)
		block 2248 | new #TX 1132 / 5000 ms = 226.4 TPS_current | total: #TX 7924 / 35.1 s = 225.8 TPS_average (peak was 227.1 TPS_average)
		block 2249 | new #TX 1132 / 5000 ms = 226.4 TPS_current | total: #TX 9056 / 40.0 s = 226.6 TPS_average (peak was 227.1 TPS_average)
		block 2250 | new #TX 1132 / 5000 ms = 226.4 TPS_current | total: #TX 10188 / 45.1 s = 225.7 TPS_average (peak was 227.1 TPS_average)
		.....
		#TX 19352599 / 85554.9 s = 226.2 TPS_average (peak was 227.1 TPS_average)
		block 19353 | new #TX 1131 / 5000 ms = 226.2 TPS_current | total: #TX 19353730 / 85560.1 s = 226.2 TPS_average (peak was 227.1 TPS_average)
		block 19354 | new #TX 1131 / 5000 ms = 226.2 TPS_current | total: #TX 19354861 / 85565.0 s = 226.2 TPS_average (peak was 227.1 TPS_average)
		block 19355 | new #TX 1131 / 5000 ms = 226.2 TPS_current | total: #TX 19355992 / 85569.9 s = 226.2 TPS_average (peak was 227.1 TPS_average)
		block 19356 | new #TX 1131 / 5000 ms = 226.2 TPS_current | total: #TX 19357123 / 85575.1 s = 226.2 TPS_average (peak was 227.1 TPS_average)
		block 19357 | new #TX 1131 / 5000 ms = 226.2 TPS_current | total: #TX 19358254 / 85580.0 s = 226.2 TPS_average (peak was 227.1 TPS_average)
		block 19358 | new #TX 1131 / 5000 ms = 226.2 TPS_current | total: #TX 19359385 / 85584.9 s = 226.2 TPS_average (peak was 227.1 TPS_average)
		block 19359 | new #TX 1131 / 5000 ms = 226.2 TPS_current | total: #TX 19360516 / 85590.1 s = 226.2 TPS_average (peak was 227.1 TPS_average)
		block 19360 | new #TX 1131 / 5000 ms = 226.2 TPS_current | total: #TX 19361647 / 85595.0 s = 226.2 TPS_average (peak was 227.1 TPS_average)
		Received signal from send.py = updated INFOFILE.
		Experiment ended! Current blocknumber = 19360
		Updated info file: last-experiment.json THE END.
		....	 
		info raw
		{'diagrams': {'blocktimestampsTpsAv': 0, 'filename': 'no-diagram-created-because-less-than-2-filled-blocks', 'prefix': 'geth-tps'}, 'node': {'chain_id': 65533, 'chain_name': 'poa', 'consensus': 'clique', 'name': 'Geth', 'network_id': 65533, 'rpc_address': 'http://172.17.57.222:8545', 'type': 'Geth', 'version': 'v1.10.17-stable-25c9b49f', 'web3.version.node': 'Geth/v1.10.17-stable-25c9b49f/linux-amd64/go1.18'}, 'send': {'block_first': 2242, 'block_last': 2242, 'empty_blocks': 10, 'num_txs': 19440000, 'sample_txs_successful': False}, 'tps': {'finalTpsAv': 232.00721653254368, 'peakTpsAv': 227.0949998952504, 'start_epochtime': 1648724021.198591}}
		
### 测试第二轮
#### 硬件修改指标
- 客户端节点(因为之前资源测试不足，所以增加到8c16)
	- 硬件
		- cpu*8
		- 内存*16

#### eth 参数
其他参数不变只变更出块时间，从5秒一个块到2 秒一个块，其他参数不变测试
#### 测试结果
- 从软硬件测试结果看
	- 硬件

		从资源监控看，硬件除了 CPU 使用率高、内存较高外，其他硬件均较为正常
	- 软件
		- 总共测试交易 47520000 
		- 测试区块
			- 74532 是创建测试合约 
			- 74534-133724，但实际是19430(问题有待查询),约等于32小时
		- 最终平均是 401/s (注意这个测试使用1个账户测试，多用户可能不同) 
		- 失败交易待查？？
- 测试中掺杂真实转账交易
	
	中间测试过程掺杂了真实转账交易，因为块的 gaslimit 在串行压力情况下没有100%块吃满，所以使用钱包交易的时候可以正常进行业务处理。没有堵塞
	
	- 解决问题：
	
		因为跨了2次初始化 geth ，所以导致我使用的 metamsk 钱包维护的 nonce 与链不相符，所以发送交易在区块链上无法执行，必须补齐之前的 nonce 才可以执行。 
- 实际情况可能性能相对有所下降需要测试

	当前测试使用合约和调用均是简单调用，只能测试共识节点 tx 打包能力，以下情况可能影响测试结果
	
	- 多共识节点
	- 只读业务节点接收交易
	- 共识节点与业务节点跨公网
	- 合约复杂度提升	

- 测试中遇到的问题
	- 客户端崩溃
		- 因为硬件不足，性能瓶颈解决，如果是测试2s/3000万limit 最少4c16g
	- 游览器崩溃
		- 同样是因为硬件不足，所以增扩，性能瓶颈解决，如果是测试2s/3000万limit 最少 32c32g2TB  	
#### 共识节点主机硬件监控
- cpu
	- use
	
		![](./pic/geth-performance-geth-cpu-2s.png)
	- load

		![](./pic/geth-performance-geth-cpuload-2s.png)
- mem

	![](./pic/geth-performance-geth-mem-2s.png)	
- network

	![](./pic/geth-performance-geth-network-2s.png)
- other

	![](./pic/geth-performance-geth-other-2s.png)
	
#### [游览器](http://106.75.120.70:4000/)查询数据
- 抽取块数据
	- [创建测试合约](http://106.75.120.70:4000/block/74532/transactions)
		
		![](./pic/geth-performance0.png)
	- 负载块信息 
	
		![](./pic/geth-performance-2s.png)
	- 块交易列表

		![](./pic/geth-performance-1-2s.png)
- 抽取单条交易数据
	
	 ![](./pic/geth-performance-2-2s.png)

#### 测试客户端结果	
- 图形

	![](./pic/geth-performance-clent-2s.png)
- 文本

		(geth-tps) Geth v1.10.17 with 47520000 txs: 401.4 TPS
		information:
		NODE: Geth/v1.10.17-stable-25c9b49f/linux-amd64/go1.18 on http://172.17.57.222:8545
		      consensus=clique chain_name=poa chain_id=65533 network_id=65533
		SEND: 47520000 transactions in blocks 74534-133724 with 10 empty blocks following.
		      A sample of transactions looked as if they: succeeded.
		TPS:  The stopclock watcher measured a final TPS of 401.4 since contract deploy,
		      and in between saw values as high as 459.5 TPS.
		DIAG: The whole experiment was prefixed 'geth-tps'.
		      The diagrams were saved into 'img/geth-tps-20220404-1040_blks74534-133724.png'.
		      Looking only at the experiment block-timestamps, the overall TPS was ~401.4.
		log:
		versions: web3 4.8.2, py-solc: 3.2.0, solc 0.4.25+commit.59dbf8f1.Linux.gpp, testrpc 1.3.5, python 3.6.9 (default, Dec  8 2021, 21:08:43) [GCC 8.4.0]
		web3 connection established, blockNumber = 74529, node version string =  Geth/v1.10.17-stable-25c9b49f/linux-amd64/go1.18
		first account of node is 0x631Cb5DB2476e55E5886e0a7d1Cdc02701C4A581, balance is 904625697166532776746648320380374280103671755200316806556.262375061821325312 Ether
		nodeName: Geth, nodeType: Geth, nodeVersion: v1.10.17-stable-25c9b49f, consensus: clique, network: 65533, chainName: poa, chainId: 65533
		
		Block  74529  - waiting for something to happen
		(filedate 1648895489) last contract address: 0x056463c88e8DEEe00A4990678fC649449DcC5124
		(filedate 1649040053) new contract address: 0xA1e8eFcC25a48EC4C5c7D0AE2cB10FB1722F6e10
		
		blocknumber_start_here = 74532
		starting timer, at block 74532 which has  1  transactions; at epochtime 1649040053.051406
		block 74533 | new #TX   0 / 2000 ms =   0.0 TPS_current | total: #TX    1 /  2.1 s =   0.5 TPS_average (peak  is   0.5 TPS_average)
		block 74534 | new #TX 749 / 2000 ms = 374.5 TPS_current | total: #TX  750 /  4.3 s = 175.7 TPS_average (peak  is 175.7 TPS_average)
		block 74535 | new #TX 1007 / 2000 ms = 503.5 TPS_current | total: #TX 1757 /  6.1 s = 287.0 TPS_average (peak  is 287.0 TPS_average)
		block 74536 | new #TX 1007 / 2000 ms = 503.5 TPS_current | total: #TX 2764 /  8.3 s = 333.8 TPS_average (peak  is 333.8 TPS_average)
		block 74537 | new #TX 1002 / 2000 ms = 501.0 TPS_current | total: #TX 3766 / 10.1 s = 371.7 TPS_average (peak  is 371.7 TPS_average)
		block 74538 | new #TX 1035 / 2000 ms = 517.5 TPS_current | total: #TX 4801 / 12.3 s = 390.6 TPS_average (peak  is 390.6 TPS_average)
		block 74539 | new #TX 1020 / 2000 ms = 510.0 TPS_current | total: #TX 5821 / 14.1 s = 411.6 TPS_average (peak  is 411.6 TPS_average)
		block 74540 | new #TX 1008 / 2000 ms = 504.0 TPS_current | total: #TX 6829 / 16.0 s = 426.8 TPS_average (peak  is 426.8 TPS_average)
		block 74541 | new #TX 1013 / 2000 ms = 506.5 TPS_current | total: #TX 7842 / 18.2 s = 431.9 TPS_average (peak  is 431.9 TPS_average)
		block 74542 | new #TX 1017 / 2000 ms = 508.5 TPS_current | total: #TX 8859 / 20.0 s = 442.5 TPS_average (peak  is 442.5 TPS_average)
		block 74543 | new #TX 978 / 2000 ms = 489.0 TPS_current | total: #TX 9837 / 22.2 s = 443.6 TPS_average (peak  is 443.6 TPS_average)
		block 74544 | new #TX 980 / 2000 ms = 490.0 TPS_current | total: #TX 10817 / 24.0 s = 450.2 TPS_average (peak  is 450.2 TPS_average)
		block 74545 | new #TX 1021 / 2000 ms = 510.5 TPS_current | total: #TX 11838 / 26.2 s = 452.1 TPS_average (peak  is 452.1 TPS_average)
		block 74546 | new #TX 1023 / 2000 ms = 511.5 TPS_current | total: #TX 12861 / 28.0 s = 458.7 TPS_average (peak  is 458.7 TPS_average)
		block 74547 | new #TX 1010 / 2000 ms = 505.0 TPS_current | total: #TX 13871 / 30.2 s = 459.5 TPS_average (peak  is 459.5 TPS_average)
		...
		block 133709 | new #TX 629 / 2000 ms = 314.5 TPS_current | total: #TX 47508820 / 118354.1 s = 401.4 TPS_average (peak was 459.5 TPS_average)
		block 133710 | new #TX 927 / 2000 ms = 463.5 TPS_current | total: #TX 47509747 / 118356.3 s = 401.4 TPS_average (peak was 459.5 TPS_average)
		block 133711 | new #TX 645 / 2000 ms = 322.5 TPS_current | total: #TX 47510392 / 118358.1 s = 401.4 TPS_average (peak was 459.5 TPS_average)
		block 133712 | new #TX 889 / 2000 ms = 444.5 TPS_current | total: #TX 47511281 / 118360.2 s = 401.4 TPS_average (peak was 459.5 TPS_average)
		block 133713 | new #TX 740 / 2000 ms = 370.0 TPS_current | total: #TX 47512021 / 118362.1 s = 401.4 TPS_average (peak was 459.5 TPS_average)
		block 133714 | new #TX 895 / 2000 ms = 447.5 TPS_current | total: #TX 47512916 / 118364.2 s = 401.4 TPS_average (peak was 459.5 TPS_average)
		block 133715 | new #TX 642 / 2000 ms = 321.0 TPS_current | total: #TX 47513558 / 118366.1 s = 401.4 TPS_average (peak was 459.5 TPS_average)
		block 133716 | new #TX 867 / 2000 ms = 433.5 TPS_current | total: #TX 47514425 / 118368.2 s = 401.4 TPS_average (peak was 459.5 TPS_average)
		block 133717 | new #TX 678 / 2000 ms = 339.0 TPS_current | total: #TX 47515103 / 118370.1 s = 401.4 TPS_average (peak was 459.5 TPS_average)
		block 133718 | new #TX 864 / 2000 ms = 432.0 TPS_current | total: #TX 47515967 / 118372.2 s = 401.4 TPS_average (peak was 459.5 TPS_average)
		block 133719 | new #TX 687 / 2000 ms = 343.5 TPS_current | total: #TX 47516654 / 118374.0 s = 401.4 TPS_average (peak was 459.5 TPS_average)
		block 133720 | new #TX 838 / 2000 ms = 419.0 TPS_current | total: #TX 47517492 / 118376.2 s = 401.4 TPS_average (peak was 459.5 TPS_average)
		block 133721 | new #TX 735 / 2000 ms = 367.5 TPS_current | total: #TX 47518227 / 118378.0 s = 401.4 TPS_average (peak was 459.5 TPS_average)
		block 133722 | new #TX 828 / 2000 ms = 414.0 TPS_current | total: #TX 47519055 / 118380.2 s = 401.4 TPS_average (peak was 459.5 TPS_average)
		block 133723 | new #TX 592 / 2000 ms = 296.0 TPS_current | total: #TX 47519647 / 118382.0 s = 401.4 TPS_average (peak was 459.5 TPS_average)
		block 133724 | new #TX 354 / 2000 ms = 177.0 TPS_current | total: #TX 47520001 / 118384.2 s = 401.4 TPS_average (peak was 459.5 TPS_average)
		block 133725 | new #TX   0 / 2000 ms =   0.0 TPS_current | total: #TX 47520001 / 118386.0 s = 401.4 TPS_average (peak was 459.5 TPS_average)
		block 133726 | new #TX   0 / 2000 ms =   0.0 TPS_current | total: #TX 47520001 / 118388.1 s = 401.4 TPS_average (peak was 459.5 TPS_average)
		block 133727 | new #TX   0 / 2000 ms =   0.0 TPS_current | total: #TX 47520001 / 118390.2 s = 401.4 TPS_average (peak was 459.5 TPS_average)
		block 133728 | new #TX   0 / 2000 ms =   0.0 TPS_current | total: #TX 47520001 / 118392.1 s = 401.4 TPS_average (peak was 459.5 TPS_average)
		block 133729 | new #TX   0 / 2000 ms =   0.0 TPS_current | total: #TX 47520001 / 118394.2 s = 401.4 TPS_average (peak was 459.5 TPS_average)
		block 133730 | new #TX   0 / 2000 ms =   0.0 TPS_current | total: #TX 47520001 / 118396.0 s = 401.4 TPS_average (peak was 459.5 TPS_average)
		block 133731 | new #TX   0 / 2000 ms =   0.0 TPS_current | total: #TX 47520001 / 118398.1 s = 401.4 TPS_average (peak was 459.5 TPS_average)
		block 133732 | new #TX   0 / 2000 ms =   0.0 TPS_current | total: #TX 47520001 / 118400.0 s = 401.4 TPS_average (peak was 459.5 TPS_average)
		block 133733 | new #TX   0 / 2000 ms =   0.0 TPS_current | total: #TX 47520001 / 118402.1 s = 401.3 TPS_average (peak was 459.5 TPS_average)
		block 133734 | new #TX   0 / 2000 ms =   0.0 TPS_current | total: #TX 47520001 / 118404.2 s = 401.3 TPS_average (peak was 459.5 TPS_average)
		Received signal from send.py = updated INFOFILE.
		Experiment ended! Current blocknumber = 133734
		Updated info file: last-experiment.json THE END.
		...
		info raw
		{'diagrams': {'blocktimestampsTpsAv': 401.41283155938504, 'filename': 'img/geth-tps-20220404-1040_blks74534-133724.png', 'prefix': 'geth-tps'}, 'node': {'chain_id': 65533, 'chain_name': 'poa', 'consensus': 'clique', 'name': 'Geth', 'network_id': 65533, 'rpc_address': 'http://172.17.57.222:8545', 'type': 'Geth', 'version': 'v1.10.17-stable-25c9b49f', 'web3.version.node': 'Geth/v1.10.17-stable-25c9b49f/linux-amd64/go1.18'}, 'send': {'block_first': 74534, 'block_last': 133724, 'empty_blocks': 10, 'num_txs': 47520000, 'sample_txs_successful': True}, 'tps': {'finalTpsAv': 401.40501631801055, 'peakTpsAv': 459.45822222379286, 'start_epochtime': 1649040053.051406}}




		

