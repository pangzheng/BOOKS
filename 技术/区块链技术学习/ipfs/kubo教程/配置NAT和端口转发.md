# 配置NAT和端口转发
运行 IPFS Kubo 的新用户的一个常见问题，尤其是在家庭网络上，是他们的 IPFS 节点位于 NAT （[网络地址转换](https://docs.ipfs.tech/concepts/glossary/#nat)）层（例如住宅网络路由器）之后。这可能会导致用户等待很长时间或请求失败率很高。由 NAT 引起的连接问题可以通过各种路由器配置解决，例如端口转发，本指南中对每一项都进行了讨论。
## 背景
NAT 后面的 IPFS 节点通常难以连接到 IPFS 网络上的其余节点。但是 NAT 有很多好处，因为它：

- 允许多台机器共享一个公共地址
- 对于 IPv4 协议的持续运行至关重要，否则该协议将无法通过其 32 位地址空间满足现代网络人口的需求。

当您连接到家庭 WiFi 时，您的计算机会获得类似 `10.0.1.15`. 这是为专用网络内部使用保留的 IP 地址范围的一部分。当您与公共 IP 地址建立传出连接时，路由器会将您的内部 IP 替换为它自己的公共 IP 地址。当数据从另一端返回时，路由器将转换回内部地址。

虽然 NAT 对于传出连接通常是透明的，但侦听传入连接需要一些配置。路由器侦听单个公共 IP 地址，但内部网络上的任意数量的机器都可以处理该请求。要处理请求，您的路由器必须配置为将特定流量发送到特定机器。
## 配置选项
路由器的适当配置选项取决于您的特定设置：

- 如果您的路由器支持它们，请启用 [IPv6](https://docs.ipfs.tech/how-to/nat-configuration/#enable-ipv6) 或启用 [UPnP](https://docs.ipfs.tech/how-to/nat-configuration/#enable-upnp) 以解决大多数连接问题
- 使用 [DCUtR holepunching](https://docs.ipfs.tech/how-to/nat-configuration/#use-dcutr-holepunching)，从 Kubo v0.13 开始默认启用
- 如果 IPv6 或 UPnP 不可用或者 DCUtR 打洞不满足您的性能和可靠性要求，请启用手动端口转发

### 启用 IPv6
	虽然启用 IPv6 可以解决许多连接问题，但它可能无法解决所有连接问题。例如，您的路由器可能支持 IPv6，但可能仍在运行阻止入站连接的防火墙。

如果您的路由器和互联网服务提供商 (ISP) 支持 IPv6，则启用它可以缓解一些连接问题。大多数现代路由器都有一个启用 IPv6 的选项。我们无法在此处提供有关每个路由器的设置和偏好的详细信息。在路由器制造商的网站上搜索 IPv6，了解有关如何启用它的信息。
### 启用即插即用
如果您的路由器支持 UPnP，IPFS 将尝试自动允许入站流量访问您的本地内容。某些家庭路由器可能需要配置为明确启用 UPnP。我们无法在此处提供有关每个路由器的设置和偏好的详细信息。在路由器制造商的网站上搜索 UPnP。
### 使用 DCUtR 打孔
从 Kubo v0.13 开始，默认启用 [DCUtR 打孔](https://github.com/ipfs/kubo/blob/master/docs/changelogs/v0.13.md#-relay-v2-client-with-auto-discovery-swarmrelayclient) （打开新窗口）.

DCUtR 打孔有各种缺点和权衡。目前，连接信令通过中继，打开连接时平均延迟 5 秒。一旦建立直接连接，延迟是正常的。此外，DCUtR 打孔没有 100% 的成功率并且可能会失败。要更深入地了解 Kubo 打孔的工作原理、它的缺点以及有关使用 DCUtR 的更多信息，请参阅 [libp2p 中的打孔 - 克服防火墙博客文章](https://blog.ipfs.tech/2022-01-20-libp2p-hole-punching/) （打开新窗口）.

由于这些缺点，您可能需要使用其他解决方案，例如手动端口转发。要启用手动端口转发，请参阅以下说明。
### 启用手动端口转发
如果您的路由器不支持 UPnP 和/或 IPv6 或者您想要比 DCUtR 提供的更好的可靠性和性能，请设置手动端口转发。完成以下步骤以启用手动端口转发：

1. [打开一个从互联网到您的内部 Kubo 节点的端口](https://docs.ipfs.tech/how-to/nat-configuration/#open-a-port)

	每个路由器都有不同的端口转发选项和解决方案。大多数路由器制造商都有在其设备上设置端口转发的指南。一般来说，步骤是：
	
	- 登录到您的路由器
	- 找到您的路由器端口转发部分
	- 输入 IPFS 节点的 IP 地址
	- 将流量设置为通过外部端口
	
			提示
			如果您不确定要使用哪个端口，4001建议使用。
	- 重新启动您的 IPFS 节点以使更改生效。确保重启整个机器，而不仅仅是 IPFS 守护进程。
2.  [更新 kubo 配置](https://docs.ipfs.tech/how-to/nat-configuration/#update-the-kubo-configuration)

	在此步骤中，您将更新您的 Kubo 配置以设置 `Swarm.AppendAnnounce` 为其他 IPFS 节点将尝试与您联系的地址列表。此列表让网络上的其他节点知道您在上一步中创建的端口转发存在并且这些是可以联系到您的地址。要更新您的配置：
	
	- 打开您的 Kubo 配置文件。

			提示
			配置文件的默认位置是 ~/.ipfs/config. 如果您已设置 $IPFS_PATH，您可以在 $IPFS_PATH/config 找到您的配置文件。
	- `Addresses ` 查找 `AppendAnnounce` 条目或创建 `Addresses` 的条目：

			 "Addresses": {
			     "Swarm": [
			     "/ip4/0.0.0.0/tcp/4001",
			     "/ip6/::/tcp/4001",
			     "/ip4/0.0.0.0/udp/4001/quic",
			     "/ip6/::/udp/4001/quic"
			     ],
			     "Announce": [],
			     "AppendAnnounce": [],
			     "NoAnnounce": [],
			     "API": "/ip4/127.0.0.1/tcp/5001",
			     "Gateway": "/ip4/127.0.0.1/tcp/8080"
			 },
	- 更新 `AppendAnnounce`，其中 `<public-ip>` 是你的公网IP地址，`<port>` 是上一步设置的端口号：

			"AppendAnnounce": [
			  "/ip4/<public-ip>/tcp/<port>",
			  "/ip4/<public-ip>/udp/<port>/quic",
			  "/ip4/<public-ip>/udp/<port>/quic-v1",
			  "/ip4/<public-ip>/udp/<port>/quic-v1/webtransport",
			 ],
	- 例如，如果您的公共 IP 地址是 `1.2.3.4` 并且 `12345` 是您选择的端口，`AppendAnnounce`则如下所示：

			"AppendAnnounce": [
			  "/ip4/1.2.3.4/tcp/12345",
			  "/ip4/1.2.3.4/udp/12345/quic",
			  "/ip4/1.2.3.4/udp/12345/quic-v1",
			  "/ip4/1.2.3.4/udp/12345/quic-v1/webtransport",
			 ],
3. [重新启动您的 Kubo 节点](https://docs.ipfs.tech/how-to/nat-configuration/#restart-your-kubo-node)

	现在你已经更新了你的 Kubo 配置，重启 Kubo 以便你的节点可以访问，并检查外部端口是否打开。
	
	- 重新启动您的 IPFS 节点以使更改生效。确保重启整个机器，而不仅仅是 IPFS 守护进程。一旦 Kubo 重新启动，您的节点应该可以访问。
	- 检查外部端口是否打开。