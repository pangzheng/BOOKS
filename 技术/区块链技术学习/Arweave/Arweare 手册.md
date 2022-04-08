# Arweare 手册
## 采矿指南
### 注意
Arweave 核心开发人员已获悉，中国大陆至少有一个挖矿节点已被政府扣押。节点运营商应了解，Arweave 网络存储和提供大量与中国政府活动相关的材料。Arweave 协议不要求任何矿工存储他们认为不合适的数据。阅读有关我们内容政策的更多信息。
### 安装矿工
`.tar.gz` 下载适用于您的操作系统的存档。
### 准备：文件描述符限制
可用文件描述符的数量会影响节点处理数据的速率。由于在大多数操作系统上分配给用户进程的默认限制通常很低，我们建议增加它。

可以通过执行来检查当前限制 `ulimit -n`

- 在 Linux 上，要设置更大的全局限制

	请打开 `/etc/sysctl.conf` 并添加以下行：

		fs.file-max=100000000
	执行sysctl -p以使更改生效。
- 为特定用户设置适当的限制。

	要设置用户级别限制，请打开 `/etc/security/limits.conf` 并添加以下行：

		<您的操作系统用户> soft nofile 10000000
	- 打开一个新的终端会话。要确保更改生效并增加限制，请键入 `ulimit -n`. 
	- 还可以通过以下方式更改当前会话的限制 `ulimit -n 10000000`
- 如果上述方法不起作用，请设置

		DefaultLimitNOFILE=10000000
	在两者 `/etc/systemd/user.conf中/etc/systemd/system.conf`

### 运行矿工
现在您已准备好使用 Arweave 目录中的以下命令开始挖掘过程：

	./bin/start mine mine_addr YOUR-MINING-ADDRESS peer 188.166.200.45 peer 188.166.192.169 peer 163.47.11.64 peer 139.59.51.59 peer 138.197.232.192

注意

	请将 YOUR-MINING-ADDRESS 替换为你的钱包地址！
如果您想查看矿工活动的日志，可以 `./bin/logs -f ` 在不同终端的 Arweave 目录中运行。

挖矿控制台最终应该是这样的：

	[Stage 1/3] Starting to hash
	Miner spora rate: 1545 h/s, recall bytes computed/s: 3129, MiB/s read: 386, the round lasted 145 seconds.
	[Stage 1/3] Starting to hash
	Skipping hashrate report, the round lasted less than 10 seconds.
	[Stage 1/3] Starting to hash
	Miner spora rate: 1545 h/s, recall bytes computed/s: 3182, MiB/s read: 386, the round lasted 135 seconds.
	[Stage 1/3] Starting to hash
	Miner spora rate: 1637 h/s, recall bytes computed/s: 3292, MiB/s read: 409, the round lasted 245 seconds.
	[Stage 1/3] Starting to hash
当你挖掘一个块时，控制台会显示：

	[Stage 2/3] Produced candidate block ... and dispatched to network.
大约 20 分钟后，您应该会看到

	[Stage 3/3] Your block ... was accepted by the network
请注意，有时您的区块不会被确认（链选择不同的分叉）。

要停止矿工，运行 `./bin/stop` 或终止操作系统进程（`kill -sigterm <pid>或pkill <name>`）。不建议发送 SIGKILL ( `kill -9`) 。

### 调整矿工
#### 优化矿工 SPoRA 率
决定矿工效率的三个关键因素是磁盘吞吐量 (GiB/s)、同步数据量和处理器能力。我们建议您拥有 32 GiB 的 RAM，而最低要求是 8 GiB。

该节点在控制台中报告其哈希率  `-Miner spora rate: 1546 h/s` 并记录 `-miner_sporas_per_second` 。请注意，在没有数据的情况下启动矿机时为 0，并随着更多数据同步而缓慢增加。数量稳定后，可以在输入社区成员 @tiama t慷慨创建的挖矿计算器，查看预期收益。

要提前估算哈希率，您需要了解或测量您的 CPU 性能、磁盘吞吐量以及您将为挖掘分配的磁盘空间量。

- CPU 评估

	要对 CPU 进行基准测试，您可以运行打包的 `randomx-benchmark` 脚本。`./bin/randomx-benchmark --mine --init 32 --threads 32 --jit --largePages.` 将 32 替换为 CPU 线程数。请注意，减少线程数可能会改善结果。如果您尚未配置它们，请不要指定。`--largePages` 作为参考，
	
	- 一个 32 线程 AMD Ryzen 3950x 可以做大约 10000 h/s
	- 一个 32 线程 AMD EPYC 7502P - 24000 h/s
	- 一个 12 线程 Intel Xeon E-2276G CPU - 2500 h/s
	- 一个 2 - 云中的线程 Intel Xeon CPU E5-2650 机器 - 600 h/s。
- 磁盘评估

	如果您不知道磁盘的吞吐量，请运行 `hdparm -t /dev/sda` . 替换 `/dev/sda` 为 `df -h`. 为了具有竞争力，请考虑使用每秒几 GiB 甚至更多的快速 NVMe SSD。

	最后，要查看设置的哈希率上限，运行 `./bin/hashrate-upper-limit 2500 1 3` 其中 2500 是 RandomX 哈希率，
	
	- 1 是磁盘每秒读取的 GiB 数
	- 3 是编织的 1/复制份额。

	例如，具有 1 GiB/s SSD 的 12 核 Intel Xeon 具有三分之一的编织速度上限为 540 h/s。在实践中，性能通常为上限的 0.7 - 0.9 左右。
#### 更改挖矿配置
我们尽最大努力选择合理的默认值；但是，更改以下某些参数可能会提高矿工的效率：（

- `stage_one_hashing_threads` (介于 1 和 CPU 线程数之间）
- `stage_two_hashing_threads`
- `io_threads`
- `randomx_bulk_hashing_iterations`

例如

	./bin/start stage_one_hashing_threads 32 stage_two_hashing_threads 32 io_threads 50 randomx_bulk_hashing_iterations 64 data_dir /your/dir mine sync_jobs 80 mining_addr YOUR-MINING-ADDRESS peer 188.166.200.45 peer 188.166.192.169 peer 163.47.11.64 peer 139.59.51.59 peer 138.197.232.192
	
- `recall bytes computed/s` 应该大致等于 `Miner spora rate` 除以你的编织份额。
	- 如果不是，请考虑增加 `io_threads` 和减少 `stage_one_hashing_threads`。
- `chunk_storage` 您可以通过将文件夹 (`du -sh /path/to/data/dir/chunk_storage`) 的来了解节点迄今为止同步的编织的份额。
	- 增加 `randomx_bulk_hashing_iterations` 到128 或更大可能会对强大的机器产生很大的影响。

#### 同步编织
Arweave 矿工在没有数据的情况下不会开采。对于每个新块，为了挖掘它，需要读取和检查过去数据的大量随机块。从对等点下载数据需要时间，所以不要指望在第一次启动后挖矿会非常密集。例如果你有 10% 的总编织大小，你正在以 10% 的效率挖掘整个数据集的类似设置。请注意，不需要下载完整的数据集。如果您只有 1 TiB 的空间用于 `chunk_storage` 和 `rocksdb` 文件夹，则该节点将填满它，并且您的矿工可能仍然具有竞争力，前提是磁盘和处理器具有足够的性能。

要加快引导，请为 `sync_jobs` 配置参数使用更高的（默认为 20）值，如下所示：

	./bin/start mine sync_jobs 80 mining_addr YOUR-MINING-ADDRESS peer 188.166.200.45 peer 188.166.192.169 peer 163.47.11.64 peer 139.59.51.59 peer 138.197.232.192
	
同步历史数据后，您可以将 sync_jobs 设置到 2。关闭矿工（不要设置 `mine` 标志）以进一步加快同步。
#### 配置大内存页面
要获得额外的性能提升，请考虑在您的操作系统中配置大内存页面。

在 Ubuntu 上，要查看当前值，请执行：`cat /proc/meminfo | grep HugePages`. 

- 要设置值，请运行 `sudo sysctl -w vm.nr_hugepages=1000`. 
- 要使配置在重新启动后仍然有效，请创建 `/etc/sysctl.d/local.conf` 并放置 `vm.nr_hugepages=1000` 。

输出 `cat /proc/meminfo | grep HugePages` 应如下所示：

	The output of cat /proc/meminfo | grep HugePages should then look like this:
	AnonHugePages: 0 kB
	ShmemHugePages: 0 kB 
	HugePages_Total: 1000 
	HugePages_Free: 1000 
	HugePages_Rsvd: 0 
	HugePages_Surp: 0
如果没有或启动时出现 “erl_drv_rwlock_destroy” 错误，请重新启动机器。

最后，通过在启动时指定来告诉矿工它可以使用大页面：`enable randomx_large_pages`

	./bin/start mine enable randomx_large_pages mining_addr YOUR-MINING-ADDRESS peer 188.166.200.45 peer 188.166.192.169 peer 163.47.11.64 peer 139.59.51.59 peer 138.197.232.192

#### 使用多个磁盘
最简单的方法是将所有内容存储在一个磁盘中。如果您对此感到满意，请跳过此部分。但是，您可以将未用于挖掘的元数据存储在更便宜且速度较慢的介质上，例如 HDD 磁盘。

将快速设备挂载到 `chunk_storage` 和 `rocksdb` 文件夹：

	sudo mount /dev/nvme1n1 /your/dir/chunk_storage
	sudo mount /dev/nvme1n2 /your/dir/rocksdb
	sudo mount /dev/hdd1 /your/dir

输出 `df -h` 应如下所示：

	/dev/hdd1 5720650792 344328088 5087947920 7% /your/dir /dev/nvme1n1 104857600 2097152 102760448 2% /your/dir/chunk_storage /dev/nvme1n2 104857600 2097152 102760448 2% /your/dir/rocksdb
将 `/dev/nvme1n1、/dev/nvme1n2、/dev/hdd1` 替换为您拥有的文件系统，替换 `/your/dir` 为您在启动时指定的目录：

	./bin/start data_dir /your/dir mine sync_jobs 80 mining_addr YOUR-MINING-ADDRESS peer 188.166.200.45 peer 188.166.192.169 peer 163.47.11.64 peer 139.59.51.59 peer 138.197.232.192
故障排除
确保您的节点可在 Internet 上访问
采矿过程的一个重要部分是发现其他矿工开采的区块。您的节点需要可从 Internet 上的任何位置访问，以便您的对等方可以与您连接并共享他们的块。
要检查您的节点是否可公开访问，请浏览至http://[Your Internet IP]:1984. 您可以在此处或通过运行。如果您在启动矿工时指定了不同的端口，请将这些说明中的任何位置的“1984”替换为您的端口。如果您无法访问该节点，您需要设置 TCP 端口转发，将传入的 HTTP 请求通过端口 1984 发送到您的 Internet IP 地址到您的矿机上的选定端口。有关如何设置端口转发的更多详细信息，请咨询您的 ISP 或云提供商。curl ifconfig.me/ip
如果节点无法在 Internet 上访问，则矿工可以运行，但效率会大大降低。
将数据复制到另一台机器
如果您想在另一台机器上引导另一个矿工，您可以从第一个矿工复制下载的数据，以使其加速。请按照以下步骤操作：
停止第一个 Arweave 矿工，并确保第二个矿工也没有运行。
将整个data_dir文件夹复制到新机器。请注意，该chunk_storage文件夹包含，因此以通常的方式复制它们将花费大量时间，并且目标文件夹的大小会太大。要复制此文件夹，请使用带有-aS标志的 rsync 或tar -Scf在复制之前将其存档。
启动两个矿工。
在 Windows 上运行矿工
我们不建议使用 Windows 进行挖矿，因为根据我们的经验，它的效率和可靠性较低。尽管如此，在 Windows 上进行挖掘是可能的。
您可以在 Windows Subsystem for Linux (WSL) 中运行 Arweave 矿工。请注意，WSL 依赖的默认 TCP 配置比典型的 Linux 配置更具限制性。WSL 配置为建立 TCP 连接提供了大约一半的 TCP 端口和两倍长的套接字重用超时，这显着减少了矿工每秒可以向其他节点发出的并发请求数。
因此，您可能会在矿工控制台中看到以下错误：
=ERROR REPORT====...=== Socket connection error: exit badarg, [{gen_tcp,connect,4, [{file,"gen_tcp.erl"},{line,149}]}
Windows 事件日志预计会有以下警告：
TCP/IP failed to establish an outgoing connection because the selected local endpoint was recently used to connect to the same remote endpoint. This error typically occurs when outgoing connections are opened and closed at a high rate, causing all available local ports to be used and forcing TCP/IP to reuse a local port for an outgoing connection. To minimize the risk of data corruption, the TCP/IP standard requires a minimum time period to elapse between successive connections from a given local endpoint to a given remote endpoint.
保持最新
加入我们的服务器
加入我们的采矿列表
一旦您在 Arweave 上成功挖掘，您将需要及时了解新版本。以接收电子邮件，通知您新的更新已发布，以及您需要采取的步骤以保持最新状态 - 特别是需要您在特定时间段内执行操作才能保持更新的更新与网络同步。请留意这些邮件，如果可能，请确保将添加到您的电子邮件提供商的受信任发件人列表中！