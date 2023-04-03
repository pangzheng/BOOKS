# 初始化 Kubo 节点并与 IPFS 网络交互
在本教程中，您将初始化一个 IPFS Kubo 节点存储库，使您的节点在线，与 IPFS 网络交互，并在您的本地节点上查看 Web 控制台。如果您在遵循本指南时遇到任何问题，请参阅故障排除。
## 先决条件
如果您尚未安装 Kubo，请按照 Kubo 安装指南进行操作。
## 初始化存储库
ipfs 将其所有设置和内部数据存储在称为存储库的目录中。在第一次使用 Kubo 之前，您需要初始化存储库。

- 提示
	- 如果您在数据中心运行 Kubo 节点，则应使用 server 配置文件初始化 IPFS。这样做将阻止 IPFS 创建试图发现本地节点的数据中心内部流量：
	
			ipfs init --profile server
	- 在 Unix 平台（包括 macOS）上使用请小心 sudo！运行将为用户而不是本地用户 `root` 帐户 `sudo ipfs init` 创建存储库。Kubo 不需要 root 权限，因此最好 `ipfs` 以普通用户身份运行所有命令！

1. 打开一个终端窗口。
2. 使用命令 `ipfs init`  初始化存储库

		ipfs init

	输出类似于以下显示：
	
		> initializing ipfs node at /Users/jbenet/.ipfs
		> generating 2048-bit RSA keypair...done
		> peer identity: Qmcpo2iLBikrdf1d6QU6vXuNb6P7hwrbNPW9kLAH8eG67z
		> to get started, enter:
		>
		>   ipfs cat /ipfs/QmYwAPJzv5CZsnA625s3Xf2nemtYgPpHdWEz79ojWnPbdG/readme
	提示
	
	旁边的散列 `peer identity` 是您节点的 ID，将与上面输出中显示的不同。网络上的其他节点用于 `peer identity` 查找并连接到您。
	
	如果需要 `peer identity`，运行 `ipfs id` 以显示。
3. 现在 `ipfs init` ，尝试运行在(ie )的输出中向您建议的命令 `ipfs cat /ipfs/QmYwAPJzv5CZsnA625s3Xf2nemtYgPpHdWEz79ojWnPbdG/readme`：

		ipfs cat /ipfs/QmYwAPJzv5CZsnA625s3Xf2nemtYgPpHdWEz79ojWnPbdG/readme
	你应该看到这样的东西：

		Hello and Welcome to IPFS!
		
		██╗██████╗ ███████╗███████╗
		██║██╔══██╗██╔════╝██╔════╝
		██║██████╔╝█████╗  ███████╗
		██║██╔═══╝ ██╔══╝  ╚════██║
		██║██║     ██║     ███████║
		╚═╝╚═╝     ╚═╝     ╚══════╝
		
		If you see this, you have successfully installed
		IPFS and are now interfacing with the ipfs merkledag!
		
		-------------------------------------------------------
		| Warning:                                              |
		|   This is alpha software. use at your own discretion! |
		|   Much is missing or lacking polish. There are bugs.  |
		|   Not yet secure. Read the security notes for more.   |
		-------------------------------------------------------
		
		Check out some of the other files in this directory:
		
		  ./about
		  ./help
		  ./quick-start     <-- usage examples
		  ./readme          <-- this file
		  ./security-notes
4. 该 `quick-start` 目录显示了其他要尝试的示例命令。要显示 `quick-start` 的内容，请运行：

		ipfs cat /ipfs/QmYwAPJzv5CZsnA625s3Xf2nemtYgPpHdWEz79ojWnPbdG/quick-start
	提示

	您可以设置许多其他配置选项——请参阅完整参考 （打开新窗口）更多。

## 让你的节点上线
接下来，让您的节点在线并与 IPFS 网络交互：

1. 打开另一个终端窗口。
2. 在新的终端窗口中启动 IPFS 守护进程：

		ipfs daemon
	片刻之后，输出如下所示，您的节点已准备就绪：

		> Initializing daemon...
		> API server listening on /ip4/127.0.0.1/tcp/5001
		> Gateway server listening on /ip4/127.0.0.1/tcp/8080
	记下输出中的 TCP 端口。如果它们不同，请在下面的命令中使用您的。

	切勿将 RPC API 暴露于公共互联网

	API 端口（默认情况下 `5001`）提供对您的 Kubo IPFS 节点的管理员级别访问。有关详细信息，请参阅 [RPC API v0 文档](https://docs.ipfs.tech/reference/kubo/rpc/)。
3. 切换回原来的终端窗口。
4.  如果您已连接到网络，请运行 `ipfs swarm peers` 以查看对等方的 IPFS 地址：

		ipfs swarm peers
	输出类似于以下显示：

		> /ip4/104.131.131.82/tcp/4001/p2p/QmaCpDMGvV2BGHeYERUEnRQAwe3N8SzbUtfsmvsqQLuvuJ
		> /ip4/104.236.151.122/tcp/4001/p2p/QmSoLju6m7xTh3DuokvT3886QRYqxAzb1kShaanJgW36yx
		> /ip4/134.121.64.93/tcp/1035/p2p/QmWHyrPWQnsz1wxHR219ooJDYTvxJPyZuDUPSDpdsAovN5
		> /ip4/178.62.8.190/tcp/4002/p2p/QmdXzZ25cyzSF99csCQmmPZ1NTbWTe8qtKFaZKpZQPdTFB
	显示的地址由
	
	- 一个 `<transport address>(ie /ip4/104.131.131.82/tcp/4001)` 
	- 和一个 `<hash-of-public-key>(ie QmSoLju6m7xTh3DuokvT3886QRYqxAzb1kShaanJgW36yx)` 组成
	- 因此地址格式为 `<transport address>/p2p/<hash-of-public-key>`。
5. 现在，使用 `ipfs cat` 命令从网络中获取宇宙飞船发射的精美图片：

		ipfs cat /ipfs/QmSgvgwxZGaBLqkGyWemEDqikCqU52XxsYLKtdy3vGZ8uq > ~/Desktop/spaceship-launch.jpg
	当上述命令运行时，Kubo 在 IPFS 网络中搜索指定的 CID ( QmSgv...) 并将数据写入名为 `spaceship-launch.jpg`.
6. 确认名为 `spaceship-launch.jpg` 的宇宙飞船发射照片位于您的~/Desktop.
7. 接下来，创建一个文件以添加到您的节点：

		echo "meow" > meow.txt
8. 添加 `meow.txt` 使用 `ipfs add`：

		ipfs add meow.txt
	输出类似于以下显示：

		> added QmabZ1pL9npKXJg8JGdMwQMJo2NCVy9yDVYjhiHK4LTJQH meow.txt
	记下 CID（即QmabZ1..），因为您将在下一步中需要它。
9. <CID> 通过指定上一步返回的 CID 查看对象：

	提示

	下面的示例使用的 curl 是浏览器，但您可以在其他浏览器中打开 IPFS 地址。根据网络状态，curl 由于公共网关过载或难以联系到您，可能需要一段时间。

		curl "https://ipfs.io/ipfs/<CID>"
	输出如下所示：

		> meow
	在此步骤中，网关从您的计算机提供了一个文件。网关查询分布式哈希表（DHT），找到你的机器，请求文件，你的计算机发送给网关，网关发送给你的浏览器。
10. 查看自己本地网关上的对象：

		curl "http://127.0.0.1:8080/ipfs/<CID>"
		
		> meow
	默认情况下，您的网关不会向外界公开。它只在本地有效。

## 使用 Web 控制台与节点交互
您可以通过导航到查看本地节点的 Web 控制台 `localhost:5001/webui`。

![Web 控制台连接视图](./pic/webui-connection.png)

Web 控制台显示 [可变文件系统 (MFS)](https://docs.ipfs.tech/concepts/file-systems/#mutable-file-system-mfs)中的文件。MFS 是一种内置于 Web 控制台的工具，可帮助您以与基于名称的标准文件系统相同的方式浏览 IPFS 文件。

当您使用 [CLI](https://docs.ipfs.tech/reference/kubo/cli/#ipfs-add) 命令 `ipfs add ...` 添加文件时，这些文件不会自动在 MFS 中可用。要在 IPFS 桌面中查看您使用 CLI 添加的文件，您必须将文件复制到 MFS：

1. 进入 `localhost:5001/webui` 您的浏览器以查看 Web 控制台。
2. 在左侧边栏菜单中，单击文件。将显示一个空目录以及以下消息：

		No files here yet! Add files to your local IPFS node by clicking the Import button above.
3. 导航回原来的终端窗口。
4. 使用在上一步中添加<CID>  meow.txt   到您的节点时获得的 CID ，将文件复制到 MFS。

		ipfs files cp /ipfs/<CID> /meow.txt
	例如，如果 `<CID> meow.txt` 是 `QmabZ1pL9npKXJg8JGdMwQMJo2NCVy9yDVYjhiHK4LTJQH`，它将被复制到 MFS：

		ipfs files cp /ipfs/QmabZ1pL9npKXJg8JGdMwQMJo2NCVy9yDVYjhiHK4LTJQH /meow.txt
5. 在您的浏览器中，刷新“文件”页面。文件列表显示 `meow.txt`。

## 将 IPFS Companion 与 Kubo 结合使用
您可以使用 IPFS Companion，这是一种浏览器扩展，可简化对 IPFS 资源的访问并添加对 IPFS 协议的支持，自动将 IPFS 网关请求重定向到本地守护程序，这样您就不会依赖远程网关。

有关 IPFS Companion 的更多信息，包括如何安装它，请参阅 [IPFS Companion 快速入门](https://docs.ipfs.tech/install/ipfs-companion/)。

## 故障排除
### 检查你的 Go 版本
IPFS 适用于 Go 1.12.0 或更高版本。要检查您安装的版本，请输入 `go version`：

	go version
	
	> go version go1.12.2 linux/amd64
如果您需要更新，我们建议您从 [规范的 Go 包中安装](https://go.dev/doc/install) （打开新窗口）. 包管理器通常包含过时的 Go 包。
### 检查是否安装了 FUSE
您需要安装和设置 FUSE 才能挂载文件系统。有关设置 FUSE 的更多详细信息，请参阅 [github.com/ipfs/kubo/blob/master/docs/fuse.md](github.com/ipfs/kubo/blob/master/docs/fuse.md)（打开新窗口）

