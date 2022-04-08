# chainhammer(区块链压测工具) v59
开源工具“chainhammer”将大量智能合约交易提交到基于以太坊的区块链，然后“chainreader”读取整个链，并生成 TPS、blocktime、gasUsed 和 gasLimit 以及块大小的图表。

可以测试 parity aura, geth clique, quorum, tobalaba 等链客户端的 tps。适用于任何以太坊链，专注 PoA 共识
## 测试结果摘要
chainhammer摘要目录路由:

	hammer --> reader --> diagrams
- 例

	AWS t2.xlarge 上的 geth clique

	[geth.md](https://github.com/drandreaskrueger/chainhammer/blob/master/results/geth.md) = geth（GO以太坊客户端），“Clique”共识。

	50,000 次交易到 Amazon t2.xlarge 机器。

	有趣的神器，在大约 14k 事务之后，速度显着下降 - 但又恢复了。[geth 报告](https://github.com/ethereum/go-ethereum/issues/17447#issuecomment-431629285)。
	
	![](./pic/geth-clique-50kTx_t2xlarge_tps-bt-bs-gas_blks12-98.png)
	
	[其他共识方案测试](https://github.com/drandreaskrueger/chainhammer#geth-clique-on-aws-t2xlarge)
	

## 目录说明
- hammer
	- 提交许多交易，同时查看最近的区块
- reader
	- 读取块；可视化 TPS、阻塞时间、gas、字节 - 参见 [reader/README.md](https://github.com/drandreaskrueger/chainhammer/blob/master/reader/README.md)
- docs
	- 见特别 copy.md、cloud.md、FAQ.md、新： azure.md
- results
	- 为每个客户端提供一个md文件年表；`results/runs/`- 自动生成的页面
- logs
	- 如果有问题，请先检查
- networks
	- 通过安装脚本的网络启动器和外部存储库，见下文
- scripts
	- 安装程序和其他 iseful bash 脚本
- env
	- Python virtualenv，通过安装脚本创建，见下文
- tests
	- 通过启动整个集成测试套件 `./pytest.sh`

## 年表
查看 [results](https://github.com/drandreaskrueger/chainhammer/blob/master/results) 文件夹：

- [log.md](https://github.com/drandreaskrueger/chainhammer/blob/master/results/log.md)
	- 初始步骤；还尝试了 Quorum 的私人交易
- [quorum.md](https://github.com/drandreaskrueger/chainhammer/blob/master/results/quorum.md) 
	- raft 共识，quorum 是一个 geth 分叉
- [tobalaba.md](https://github.com/drandreaskrueger/chainhammer/blob/master/results/tobalaba.md) 
	- EnergyWebFoundation 的平价分叉
- [quorum-IBFT.md](https://github.com/drandreaskrueger/chainhammer/blob/master/results/quorum-IBFT.md)
	- IstanbulBFT，仲裁中的第二个共识算法
- [geth.md](https://github.com/drandreaskrueger/chainhammer/blob/master/results/geth.md) 
	- geth clique PoA 算法
- [parity.md](https://github.com/drandreaskrueger/chainhammer/blob/master/results/parity.md) 
	- parity aura PoA 算法，多次尝试加速
- [eos.md](https://github.com/drandreaskrueger/chainhammer/blob/master/results/eos.md)：尚未开始
- [substrate.md](https://github.com/drandreaskrueger/chainhammer/blob/master/results/substrate.md)：尚未开始

## 总结表
在 2018 年秋季手动运行每个实验的过时表；很快就完全重新完成，使用下面的自动化。所以请现在联系我，如果你知道如何加速这些客户：

共识类型|硬件|node数量|配置|峰值 TPS|最终 TPS
---|---|---|---|---|---
parity aura|t2.micro|4|(D)|45.5|44.3
 |t2.large|4|(D)|53.5|52.9
 |t2.xlarge|4|(J)|57.1|56.4
 |t2.2xlarge|4|(D)|57.6|57.6
 parity instantseal|t2.micro|1|(G)|42.3|42.3
  |t2.xlarge|1|(J)|48.1|48.1
 geth clique|t2.2xlarge|3+1 +2|(B)|	421.6|	400.0
  |t2.xlarge|3+1 +2|(B)|	386.1|321.5
  |t2.xlarge|3+1|(K)|372.6|325.3
  | t2.large|3+1 +2|(B)|	170|169.4
  |t2.small|3+1 +2|(B)|96.8|96.5
  |t2.micro|3+1 |(H)|124.3|122.4
 quorum crux IBFT|t2.micro SWAP|4|(I) SWAP!|98.1|98.1
  | t2.large |4|(F)|207.7|199.9
  | t2.xlarge |4|(F)|439.5|395.7
  | t2.xlarge |4|(L)|389.1|338.9
  |t2.2xlarge|4|(F)|435.4|423.1
  |c5.4xlarge|4|(F) test_getNearestEntry()|536.4|524.3

轻松[重现这些结果](https://github.com/drandreaskrueger/chainhammer/blob/master/docs/reproduce.md)；对于 `config` 列，请参见那里。使用我的[Amazon AMI 现成镜像](https://github.com/drandreaskrueger/chainhammer/blob/master/docs/reproduce.md#readymade-amazon-ami)进行最快的复制。并查看 [parity.md](https://github.com/drandreaskrueger/chainhammer/blob/master/results/parity.md) 和 [geth.md](https://github.com/drandreaskrueger/chainhammer/blob/master/results/geth.md) 和 [quorum-IBFT.md](https://github.com/drandreaskrueger/chainhammer/blob/master/results/quorum-IBFT.md) 的底部以了解最新的运行、问题和其他详细信息。  
  
## 未来
- 我最初是如何在 `Quorum` 上逐步加快速度的，请阅读第一本日志 [log.md](https://github.com/drandreaskrueger/chainhammer/blob/master/results/log.md)
- 然后我对每个客户端进行了改进，请参见上面年表
- （可能的[TODO](https://github.com/drandreaskrueger/chainhammer/blob/master/docs/TODO.md) - 还有其他想法吗？）

但不需要更多=当前版本已经完全自动化。用它！愿它帮助您提高以太坊客户端的速度！

## 使用
使用 chainhammer 或类似的项目将自己添加到 [other-projects.md](https://github.com/drandreaskrueger/chainhammer/blob/master/docs/other-projects.md) 。

(特别是如果您在其中一个开发团队工作，您最了解您的客户端代码 - ）请尝试改进上述结果，例如通过改变启动节点的 CLI 参数；我不认为这是我的工作，你会更成功。

请在您完成其他/新测量时报告时，请参阅 

- parity [PE#9393](https://github.com/paritytech/parity-ethereum/issues/9393)
- parity [SE#58521](https://ethereum.stackexchange.com/questions/58521/parity-tps-optimization-please-help)
- geth [GE#17447](https://github.com/ethereum/go-ethereum/issues/17447)
- quorum [Q#479](https://github.com/jpmorganchase/quorum/issues/479#issuecomment-413603316)

## 安装并运行
所有这些都是在 Debian、本地和 AWS 云中开发和测试的。新：现在也支持 Ubuntu，见下文。
### 快速开始
	注意：最好在一次性云或虚拟机上执行此操作；因为安装会产生持久的变化并且需要 sudo！
- 解压缩下载的 repo 的 ZIP 后或通过

		git clone https://github.com/drandreaskrueger/chainhammer drandreaskrueger_chainhammer
		ln -s drandreaskrueger_chainhammer CH
		cd CH
- 现在只需要这两行来准备和运行第一个实验！

		scripts/install.sh
		CH_TXS=1000 CH_THREADING="sequential" ./run.sh $HOSTNAME-TestRPC testrpc
- 然后，您将有一个图表，以及关于此运行的 HTML 和 MD 页面！

（在Ubuntu上代替 `scripts/install.sh docker ubuntu`：）

#### 激活 docker 
打开一个新终端，因为上面的 `scripts/install.sh` 可能已经为这个用户第一次启用了 docker。然后：

- 请运行每个客户端片刻：

		export CH_MACHINE=yourChoice
		./run-all_small.sh
有关详细说明，请参阅 [docs](https://github.com/drandreaskrueger/chainhammer/blob/master/docs)，尤其是。[reproduce.md](https://github.com/drandreaskrueger/chainhammer/blob/master/docs/reproduce.md)，并用于解决 [FAQ.md](https://github.com/drandreaskrueger/chainhammer/blob/master/docs/FAQ.md)和 [github 问题](https://github.com/drandreaskrueger/chainhammer/issues)。

#### 对远程节点进行基准测试
现在可以将 Chainhammer 剥离为纯粹的基准测试能力，即无需安装 docker，也无需三个本地网络启动器（parity-deploy、geth-dev、quorum-crux）。它已成功用于对 Microsoft Azure 区块链即服务产品进行基准测试。本质区别是使用开关开始安装 `nodocker`：

	scripts/install.sh nodocker
因此，如果您只想对现有的以太坊节点或网络进行基准测试，请查看手册 [docs/azure.md](https://github.com/drandreaskrueger/chainhammer/blob/master/docs/azure.md)。
## 单元测试
	./pytest.sh
- 启用 virtualenv，然后在后台在 `http://localhost:8545` `testrpc-py` 上启动以太坊模拟器，登录；
- 然后运行；
- 最后运行所有单元测试，同时登录 `.tests/logs/./deploy.py andtests tests/logs/`

（而不是 testrpc-py）如果你想用另一个节点运行测试，只需启动它；并 `pytest` 手动运行：

	source env/bin/activate
	py.test -v --cov

1 月 23 日进行了 98 次测试，所有 98 次通过（请参阅此[日志文件](https://github.com/drandreaskrueger/chainhammer/blob/master/tests/logs/tests-with_testrpc-py.log.ansi) --> `cat tests/logs/*.ansi` 因为颜色）在这些不同的以太坊提供商上：

- testrpc instantseal (`testrpc-py`) 13 秒
- geth Clique (`geth-dev`) 63 秒
- quorum IBFT ( `blk-io/crux`) 59 秒
- parity instantseal ( `parity-deploy`) 8 秒
- parity aura ( `parity-deploy`) 72 秒

## 参考
- [chainhammer](https://github.com/drandreaskrueger/chainhammer)
- [v55 版本视频演示](https://www.youtube.com/watch?v=xTYnsfs5U7I)
- [geth Clique PoA benchmarking](https://github.com/drandreaskrueger/chainhammer/blob/master/results/geth.md)