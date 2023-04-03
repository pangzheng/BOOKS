# 配置节点
IPFS 通过一个 json 格式的文本文件配置，默认位于 `~/.ipfs/config`. 特定于实施的信息可以在 [Kubo 中找到](https://github.com/ipfs/kubo/blob/master/docs/config.md) （打开新窗口）和 [js-ipfs](https://github.com/ipfs/js-ipfs/blob/master/docs/CONFIG.md) （打开新窗口）存储库。它在节点实例化时读取一次，用于离线命令或启动守护程序时。在运行的守护进程上执行的命令不会在运行时读取配置文件。
# 修改 bootstrap peers 列表
IPFS 引导列表是一个对等点列表，IPFS 守护进程通过它了解网络上的其他对等点。IPFS 带有一个默认的可信对等点列表，但您可以自由修改该列表以满足您的需要。自定义引导列表的一种流行用途是创建个人 IPFS 网络。

首先，让我们列出您节点的引导程序列表：

	ipfs bootstrap list
	> /dnsaddr/bootstrap.libp2p.io/p2p/QmNnooDu7bfjPFoTZYxMNLWUQJyrVwtbZg5gBMjTezGAJN
	> /dnsaddr/bootstrap.libp2p.io/p2p/QmQCU2EcMqAqQPR2i9bChDtGNJchTbq5TbXJJ16u19uLTa
	> /dnsaddr/bootstrap.libp2p.io/p2p/QmbLHAnMoJPWSCR5Zhtx6BHJX9KiKNN6tpvbUcqanj75Nb
	> /dnsaddr/bootstrap.libp2p.io/p2p/QmcZf59bWwK5XFi76CZX8cbJ4BhTzzA3gU1ZjYZcYW3dwt
	> /ip4/104.131.131.82/tcp/4001/p2p/QmaCpDMGvV2BGHeYERUEnRQAwe3N8SzbUtfsmvsqQLuvuJ
上面列出的行是默认 IPFS 引导节点的地址——它们由 IPFS 开发团队运行。列出的地址已在 [multiaddr](https://github.com/multiformats/multiaddr) 中完全解析和指定 （打开新窗口）格式，这使得每个协议都是明确的。通过这种方式，您的节点确切地知道从哪里到达引导程序节点——位置是明确的。

不要更改此列表，除非您了解这样做的意义。Bootstrapping 是分布式系统中一个重要的安全故障点：恶意 bootstrap 节点只能将您介绍给其他恶意节点。建议保留 IPFS 开发团队提供的默认列表或者——在设置专用网络的情况下——你控制的节点列表。不要将您不信任的同行添加到此列表。

在这里，我们将一个新的对等点添加到引导列表：

	> ipfs bootstrap add /ip4/25.196.147.100/tcp/4001/p2p/QmaMqSwWShsPg2RbredZtoneFjXhim7AQkqbLxib45Lx4S
在这里，我们从引导列表中删除一个节点：

	> ipfs bootstrap rm /ip4/104.131.131.82/tcp/4001/p2p/QmaCpDMGvV2BGHeYERUEnRQAwe3N8SzbUtfsmvsqQLuvuJ
假设我们要创建新引导列表的备份。我们可以通过将 stdout 重定向ipfs bootstrap list到一个文件来轻松地做到这一点：

	> ipfs bootstrap list >save
如果我们想从头开始，我们可以一次删除整个引导程序列表：

	> ipfs bootstrap rm --all
使用空列表，我们可以恢复默认的引导程序列表：

	> ipfs bootstrap add --default
再次删除整个引导程序列表，并通过将保存文件的内容通过管道传输到  `ipfs bootstrap add` 以下内容来恢复我们保存的列表：

	> ipfs bootstrap rm --all
	> cat save | ipfs bootstrap add

# 自定义 IPFS 存储
[custom-ipfs-repo](https://github.com/ipfs-examples/js-ipfs-examples/tree/master/examples/custom-ipfs-repo)

# 在 Kubo 中使用垃圾回收
## 自动清理
在 IPFS Kubo 中，IPFS [垃圾收集器在 Kubo 配置文件](https://github.com/ipfs/kubo/blob/master/docs/config.md) `Datastore` 的部分中配置 （打开新窗口）. 垃圾收集器相关的重要设置有：

- `StorageGCWatermarkStorageMax`

	如果守护程序在启用自动垃圾收集的情况下运行，则自动触发垃圾收集的值的百分比。默认值为 90`。
- `GCPeriod`

	指定垃圾收集运行的频率。仅在启用自动垃圾收集时使用。默认值为 1 小时。

## 手动清理
要手动启动垃圾收集，请运行 [ipfs repo gc](https://docs.ipfs.tech/reference/kubo/cli/#ipfs-repo-gc)：

	ipfs repo gc
	
	> removed QmPZhyTu8D7NqR5NvgkgNYsSYD4CNjnyuFejB8i23itJvA
	> removed QmSYQFVAZgEnpa6NxiW5agyj3XU9VR4CbERShXiLhuPPPE
	> removed QmS6SJXApoi59hqD8Naktgakc6UNHK1XDhqhtMg9sBhY8g
`--enable-gc` 在启动 IPFS 守护进程时启用自动垃圾收集：

	ipfs daemon --enable-gc
	
	> Initializing daemon...
	> Kubo version: 0.9.0
	> Repo version: 10
	> ...

提示

如果您使用 IPFS 桌面，您可以通过单击 IPFS 桌面应用程序的任务栏图标并选择 `Advanced → Run Garbage Collector`来触发垃圾收集。