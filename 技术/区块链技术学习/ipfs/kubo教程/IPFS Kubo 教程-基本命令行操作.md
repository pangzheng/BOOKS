# IPFS Kubo 教程-基本命令行操作
本简短指南旨在引导您了解通过 Kubo CLI 使用 IPFS 的基础知识。Kubo 是多个 IPFS 实现之一。它是最古老的 IPFS 实现并公开了一个 CLI（除其他外）。

您将学习如何在 CLI 中添加、检索、读取和删除文件。如果您不确定某些术语的含义，可以查看词汇表。

此处显示的所有说明和示例均在 M1 Mac 上执行和测试。但是，IPFS 命令在 Linux、macOS 和 Windows 上是相同的。您将需要知道如何从 CLI 中导航计算机的目录。如果您不确定如何使用 CLI，我们建议您先了解如何使用，然后再继续阅读本指南。
## 安装
接下来，我们需要为命令行安装 Kubo。我们有一个很好的指南，将引导您完成如何使用CLI 安装 Kubo。

安装 Kubo 后，我们需要启动并运行我们的节点。如果这是您第一次使用 Kubo，您首先需要初始化配置文件：

	ipfs init
这将输出如下内容：

	initializing ipfs node at /Users/<user>/.ipfs
	generating 2048-bit RSA keypair...done
	peer identity: Qm...
	to get started, enter:
	
		 ipfs cat /ipfs/QmYwAPJzv5CZsnA625s3Xf2nemtYgPpHdWEz79ojWnPbdG/readme
我们现在准备启动 IPFS 守护进程以使节点联机。运行 `ipfs daemon` 命令：

	ipfs daemon
这将输出如下内容：

	Initializing daemon...
	Kubo version: 0.12.0
	Repo version: 12
	System version: arm64/darwin
	[...]
	Daemon is ready
该命令将一直运行，直到您告诉它停止；不要这样做！只需打开一个新的 CLI 实例并继续本指南！

	警告
	不要关闭用于初始化守护程序的 CLI。仅当您想让 IPFS 节点脱机时才终止守护进程。

## 添加文件
现在我们已经启动并运行了 Kubo IPFS 节点，我们已准备好将文件添加到 IPFS。

1. 在 CLI 中，导航到包含您要共享的文件或文件夹的目录。在此示例中，我们将导航到 `~/Documents` 目录：

		cd ~/Documents
2. 进入后，列出目录的内容以确保我们在正确的位置：

		ls
3. 这将输出该文件夹的内容：

		hello-ipfs.txt
4. 接下来，我们将使用ipfs add命令将文件添加到 IPFS。请务必将文件扩展名添加到文件名的末尾：

		ipfs add hello-ipfs.txt
5. 这将输出如下内容：

		added QmRgR7Bpa9xDMUNGiKaARvFL9MmnoFyd86rF817EZyfdGE hello-ipfs.txt
		6 B / 6 B [==========================================================] 100.00

我们现在已经向 IPFS 添加了一个文件，并准备好与网络上的同行共享！

## 检索文件
在上一节中，我们介绍了如何将本地文件添加到 IPFS。我们将介绍如何从 IPFS 检索远程文件并将它们保存到您的计算机。对于此示例，我们将检索包含单个文本文件的文件夹。

1. 在 CLI 中，导航到您希望保存文件夹的目录。IPFS 会将文件夹保存到您所在的任何目录。在这种情况下，我们将把文件夹保存在目录中~/Documents：

		cd ~/Documents
2. 要通过 IPFS 获取内容，我们需要告诉我们ipfs daemon想要哪个 CID。在这种情况下，我们想要

		bafybeif2ewg3nqa33mjokpxii36jj2ywfqjpy3urdh7v6vqyfjoocvgy3a.
3. 使用命令 `ipfs get`，结合我们想要的 CID，检索文件夹：

		ipfs get bafybeif2ewg3nqa33mjokpxii36jj2ywfqjpy3urdh7v6vqyfjoocvgy3a
	这将输出如下内容：

		Saving file(s) to bafybeif2ewg3nqa33mjokpxii36jj2ywfqjpy3urdh7v6vqyfjoocvgy3a
		1.76 KiB / 1.76 KiB [==============================================] 100.00% 0s

	您的 IPFS 节点可能需要一些时间才能找到我们正在寻找的数据。

我们现在已经通过 IPFS 检索到该文件夹​​，并且它的副本已保存到您计算机的本地存储中！您现在还托管该文件夹及其内容以供其他人检索。

## 查看文件
您可以使用命令从 CLI 中查看文件的内容 `ipfs cat`。我们将使用此命令来查看我们刚刚检索到的文件夹的内容，但您也可以使用 `ipfs cat` 本地没有的文件。

	ipfs cat bafybeif2ewg3nqa33mjokpxii36jj2ywfqjpy3urdh7v6vqyfjoocvgy3a
这将输出如下内容：

	Error: this dag node is a directory
尝试从上面的 CID 上运行 `ipfs cat` 会返回错误！CID 指向目录，而不是文件。要查看目录内容，我们将运行 `ipfs refs <CID>.`

	ipfs refs bafybeif2ewg3nqa33mjokpxii36jj2ywfqjpy3urdh7v6vqyfjoocvgy3a
这将输出如下内容：

	bafkreig24ijzqxj3cxdp6yh6ia2ysxzvxsfnd6rzahxxjv6ofcuix52wtq
该 `ref` 命令返回指向目录中文件的 CID。现在我们可以使用 `ipfs cat` 这个新的 CID 来查看它：

	ipfs cat bafkreig24ijzqxj3cxdp6yh6ia2ysxzvxsfnd6rzahxxjv6ofcuix52wtq
这将输出如下内容：

	MMMMMMMMMMN0xo;';ox0NMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMM
	MMMMMMWXOdoloxkkkxolodOXWMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMM
	MMMN0xdoodkOOOOOOOOOkdoodx0NMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMM
	MKo;;oxOOOOOOOOOOOOOOOOOxo;;oKMMMMMXOKMMMN0kkkkkO0NWMMXOkkkkkkOXMMWKkddddk0NMMMM
	Wd...':okOOOOOOOOOOOOOko:'...dWMMMWd.lWMM0,.;ccc:;,oXWd.'clllco0WO:,:loolcckWMMM
	Wdckdc,..;lxOOOOOOOxl;..';lo;dWMMMWo.lWMM0''0MMMWK:.oNo.lWMMMMMMX:.dWMMMMMWWMMMM
	WdcOKK0ko;'.,:c:c:,..,:oxxxd;dWMMMWo.lWMM0''0MMMMNc.oNo.lNWWWWWMNl.;x0XNWMMMMMMM
	WdcOKKKKKKOd:.   .,cdxxxxxxd;dWMMMWo.lWMM0'.coool;'cKWo..clllldXMNkl:;;;:lxXMMMM
	WdcOKKKKKKKK0l. .:dxxxxxxxxd;dWMMMWo.lWMM0'.cooodkKWMWo.:0K000KWMMMMWNK0xc.,0MMM
	WdcOKKKKKKKKK0: ,dxxxxxxxxxd:dWMMMWo.lWMM0''0MMMMMMMMWo.lWMMMMMMMMMMMMMMMX:.dWMM
	Wd;xKKKKKKKKK0: ,dxxxxxxxxxl,dWMMMWo.lWMM0''0MMMMMMMMWo.lWMMMMMMWOdk0KXXKd.,0MMM
	Mk',d0KKKKKKK0: ,dxxxxxxxdc.'kMMMMMO:xWMMKllXMMMMMMMMWk:kWMMMMMMWOlcccccccdKWMMM
	MWKkdooxOKKKK0: ,dxxxxdlclokKWMMMMMWWWMMMMWWMMMMMMMMMMMWMMMMMMMMMMMWWNNNNWMMMMMM
	MMMMWNOxdodk00: ,ddoccldONWMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMM
	MMMMMMMMWKkdol' .:lokKWMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMM
	MMMMMMMMMMMWKd'.'dKWMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMMM
重要的是要注意，`ipfs cat` 它只适用于明文文件。如上所示，尝试访问 `cat` 目录将返回错误。如果您尝试使用 `cat` 非纯文本的图像文件或文本文件，您的终端将充满不可读的行。
## pin 文件
我们可以将要保存的数据固定到我们的 IPFS 节点，以确保我们不会丢失这些数据。

1. 使用 `ipfs pin add` 命令：

		ipfs pin add bafybeif2ewg3nqa33mjokpxii36jj2ywfqjpy3urdh7v6vqyfjoocvgy3a
	这将输出如下内容：

		pinned bafybeif2ewg3nqa33mjokpxii36jj2ywfqjpy3urdh7v6vqyfjoocvgy3a recursively

默认情况下，您通过 IPFS 检索的对象不会固定到您的节点。如果您希望防止文件被垃圾收集，您需要固定它们。您会注意到您刚刚添加的 pin 是https://docs.ipfs.tech/concepts/persistence/一个 `recursive` pin，这意味着它是一个包含其他对象的目录。查看 [Pinning 页面以了解有关其工作原理的更多信息](https://docs.ipfs.tech/concepts/persistence/)。
## 删除文件
如果我们决定不再托管文件，我们所要做的就是删除 pin。

1. 首先，我们需要获取要取消固定的文件或文件夹的 CID。要查看您已 pin 的内容列表，请运行 `ipfs pin ls`：

		ipfs pin ls
	这将输出如下内容：

		QmPZ9gcCEpqKTo6aq61g2nXGUhM4iCL3ewB6LDXZCtioEB indirect
		QmQGiYLVAdSHJQKYFRTJZMG4BXBHqKperaZtyKGmCRLmsF indirect
		QmU5k7ter3RdjZXu3sHghsga1UQtrztnQxmTL22nPnsu3g indirect
		QmUNLLsPACCz1vLxQVkXqqLX5R1X345qqfHbsf67hvA3Nn recursive
		QmYCvbfNbCwFR45HiNP45rwJgvatpiW38D961L5qAhUM5Y indirect
		QmejvEPop4D7YUadeGqYWmZxHhLc4JBUCzJJHWMzdcMe2y indirect
		QmQ5vhrL7uv6tuoN9KeVBwd4PwfQkXdVVmDLUZuTNxqgvm indirect
		QmQPeNsJPyVWPFDVHb77w8G42Fvo15z4bG2X8D2GhfbSXc recursive
		QmQy6xmJhrcC5QLboAcGFcAE1tC8CrwDVkrHdEYJkLscrQ indirect
2. 如果您确切知道要删除哪个 CID，那就太好了！但是，如果您不确定是哪个 CID，可以 `ipfs add` 在要删除的文件或文件夹上再次使用该命令进行查找。我们将取消固定 `hello-ipfs.txt` 我们之前使用的文件。

		cd ~/Documents
		ipfs add hello-ipfs.txt

	这将输出如下内容：

		added QmRgR7Bpa9xDMUNGiKaARvFL9MmnoFyd86rF817EZyfdGE hello-ipfs.txt
		6 B / 6 B [==========================================================] 100.00
	尽管 IPFS 只是给我们提供与以前相同的输出，但我们实际上并没有重新添加文件。不过，我们可以从这个输出中获取 CID。

	现在，我们可以使用ipfs pin rm命令取消固定文件：

		ipfs pin rm QmRgR7Bpa9xDMUNGiKaARvFL9MmnoFyd86rF817EZyfdGE
	这将输出如下内容：

		unpinned QmRgR7Bpa9xDMUNGiKaARvFL9MmnoFyd86rF817EZyfdGE
	该 `hello-ipfs.txt` 文件现在已取消固定，但尚未从我们的节点中完全删除。要完全删除它，我们需要运行垃圾回收。该命令将从您的节点中删除没有 pin 的所有内容：

		ipfs repo gc
	这将输出如下内容：

		removed bafybeif2ewg3nqa33mjokpxii36jj2ywfqjpy3urdh7v6vqyfjoocvgy3a
		removed bafkreieceevgg2auxo4u3rjgeiqfr4ccxh6ylkgxt2ss6k2leuad5xckxe
		removed bafkreiblcvcr7letdbp2k2thkbjyunznrwq3y6pyoylzaq4epawqcca2my
		[...]
	目标文件现已从您的 IPFS 节点和我们未固定的任何其他文件中完全删除。如果刚刚被垃圾收集的内容保存到您计算机的本地存储中，它仍然存在。如果您希望从计算机的本地存储中删除内容，您需要找到它的保存位置并使用正常的删除方法将其删除。