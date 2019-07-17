# 使用 ipfs 传输文件
本文档是帮助解决使用 ipfs 在两台计算机之间传输文件的故障的指南。
## 文件传输
首先，确保 ipfs 在两台计算机上运行。要验证，请 `ipfs id` 在每台计算机上运行并检查该 `Addresses` 字段中是否包含任何内容。如果它说 `null`，那么您的节点不在线，您将需要运行 `ipfs daemon`。

现在，让我们调用节点，将要传输节点“A”的文件和要获取文件的节点传送到节点“B”。在节点A上，使用该 `ipfs add` 命令将文件添加到 ipfs 。这将打印出您添加的内容 hash。现在，在节点B上，您可以使用 `ipfs get <hash>` 获取内容 。

	# On A
	> ipfs add myfile.txt
	added QmZJ1xT1T9KYkHhgRhbv8D7mYrbemaXwYUkg7CeHdrk1Ye myfile.txt
	
	# On B
	> ipfs get QmZJ1xT1T9KYkHhgRhbv8D7mYrbemaXwYUkg7CeHdrk1Ye
	Saving file(s) to QmZJ1xT1T9KYkHhgRhbv8D7mYrbemaXwYUkg7CeHdrk1Ye
	 13 B / 13 B [=====================================================] 100.00% 1s
如果有效，并下载了文件，那么恭喜！刚刚使用 ipfs 在互联网上移动文件！但是，如果该 `ipfs get` 命令挂起，没有输出，请继续阅读。
## 故障排除
所以你的 ipfs 文件传输似乎无法正常工作。发生这种情况的主要原因是节点B无法弄清楚如何连接到节点A，或者节点B甚至不知道它必须连接到节点A.
### 检查现有连接
首先要做的是仔细检查两个节点实际上是在运行还是在线运行。为此，请使用 `ipfs id` 在每台计算机上运行。如果两个节点都显示某些地址（如下例所示），则表示您的节点处于联机状态。

	{
	        "ID": "QmTNwsFkLAed15kQEC1ZJWPfoNbBQnMFojfJKQ9sZj1dk8",
	        "PublicKey": "CAASpgIwggEiMA0GCSqGSIb3DQEBAQUAA4IBDwAwggEKAoIBAQDZb6znj3LQZKP1+X81exf+vbnqNCMtHjZ5RKTCm7Fytnfe+AI1fhs9YbZdkgFkM1HLxmIOLQj2bMXPIGxUM+EnewN8tWurx4B3+lR/LWNwNYcCFL+jF2ltc6SE6BC8kMLEZd4zidOLPZ8lIRpd0x3qmsjhGefuRwrKeKlR4tQ3C76ziOms47uLdiVVkl5LyJ5+mn4rXOjNKt/oy2O4m1St7X7/yNt8qQgYsPfe/hCOywxCEIHEkqmil+vn7bu4RpAtsUzCcBDoLUIWuU3i6qfytD05hP8Clo+at+l//ctjMxylf3IQ5qyP+yfvazk+WHcsB0tWueEmiU5P2nfUUIR3AgMBAAE=",
	        "Addresses": [
	                "/ip4/127.0.0.1/tcp/4001/ipfs/QmTNwsFkLAed15kQEC1ZJWPfoNbBQnMFojfJKQ9sZj1dk8",
	                "/ip4/192.168.2.131/tcp/4001/ipfs/QmTNwsFkLAed15kQEC1ZJWPfoNbBQnMFojfJKQ9sZj1dk8",
	        ],
	        "AgentVersion": "go-ipfs/0.4.11-dev/",
	        "ProtocolVersion": "ipfs/0.1.0"
	}
接下来，检查节点是否彼此连接。您可以通过 `ipfs swarm peers` 在一个节点上运行并检查输出中的其他节点对等 ID 来执行此操作。如果两个节点已连接，并且 `ipfs get` 命令仍然挂起，则会发生意外情况，我建议提交有关它的问题。如果它们没有连接，那么让我们尝试调试原因。（注意：如果您只是想要工作，可以跳到'手动将节点A连接到节点B'。完成调试过程并报告 IRC 上 ipfs 团队发生的事情有助于我们理解人们运行的常见问题）

	$ ipfs swarm peers
	/ip4/104.131.131.82/tcp/4001/ipfs/QmaCpDMGvV2BGHeYERUEnRQAwe3N8SzbUtfsmvsqQLuvuJ
	/ip4/104.236.179.241/tcp/4001/ipfs/QmSoLPppuBtQSGwKDZT2M73ULpjvfd3aZ6ha4oFGL1KrGM
	/ip4/114.230.67.186/tcp/25918/ipfs/QmdsGbKtgPsSb2AfqMm9jZfa3k2BD8nhUQSupdGctKiAay
	/ip4/115.238.154.85/tcp/31622/ipfs/QmXmgwAocyUsPKgbmPjo1vyiD34FoWHVQRcK6vKMaHfVuM
	/ip4/117.40.214.150/tcp/36314/ipfs/QmPDgJXvzvGYT5ZKs8UC4vdLLkxQafCvVK2kEhST8kyrEm
	/ip4/117.91.115.116/tcp/64799/ipfs/QmXn7wD7oC3DDySykxiSHrumRqhxYVS8focb9zYN559Z3N
	/ip4/119.147.149.203/tcp/14001/ipfs/Qmak1PLzqPpwmfcYaUdyAH4z79khYyokMfBknaBrYLyR2v
	....
### 检查供应商
在 ipfs 上请求内容时，节点会在 DHT 中搜索“提供者记录”，以查看谁拥有哪些内容。让我们在节点B上手动执行此操作，以确保节点B能够确定节点A具有该数据。跑 `ipfs dht findprovs <hash>`。我们希望看到节点A的对等ID打印出来。如果此命令不返回任何内容（或返回不是节点A的ID），则网络上不存在具有该数据的A的记录。如果在节点A没有运行守护程序的情况下添加数据，则会发生这种情况。如果发生这种情况，您可以 `ipfs dht provide <hash>` 在节点A上运行以向网络通知您具有该哈希值。然后，如果重新启动 `ipfs get` 命令，节点B现在应该能够告诉节点A具有它想要的内容。如果节点A的对等ID出现在初始状态 `findprovs` 调用或手动提供哈希没有解决问题，那么节点B很可能无法与节点A建立连接。
### 检查地址
在节点B不能形成与节点A的连接的情况下，尽管知道它需要，但可能的罪魁祸首是坏的NAT。当节点B获知它需要连接到节点A时，它检查 DHT 以查找节点A的地址，然后开始尝试连接它们。我们可以通过 `ipfs dht findpeer <node A peerID>` 在节点B上运行来检查这些地址。该命令应返回节点A的地址列表。如果它没有返回任何地址，那么您应该尝试运行前面步骤中的手动提供命令。地址输出示例可能如下所示：

	/ip4/127.0.0.1/tcp/4001
	/ip4/192.168.2.133/tcp/4001
	/ip4/88.157.217.196/tcp/63674
在这种情况下，可以看到 `localhost（127.0.0.1）` 地址，LAN地址`（192.168.2 ）`和另一个地址。如果此第三个地址与您的外部IP匹配，则网络知道您的节点的有效外部地址。此时，可以安全地假设您的节点难以遍历 NAT 情况。如果是这种情况，可以尝试在节点A的路由器上启用 `UPnP` 或 `NAT-PMP`，然后重试该过程。否则，可以尝试手动将节点A连接到节点B.
### 手动将节点A连接到B.
在节点B上运行 `ipfs id` 并获取包含其公共IP地址的多个加载项之一，然后在节点A上运行 `ipfs swarm connect <multiaddr>` 。您也可以尝试使用中继连接，有关详细信息，请[阅读此文档](https://github.com/ipfs/go-ipfs/blob/master/docs/experimental-features.md#circuit-relay)。如果仍然无效，那么您应该加入 IRC 并在那里寻求帮助，或者在 github 上提出问题。

如果此手动步骤确实有效，那么您可能遇到 NAT 遍历问题，而 ipfs 无法弄清楚如何通过。请向我们报告此类情况，以便我们可以修复它们。