# 体系-网络-OpenShift SDN
## 概观
OpenShift Origin 使用软件定义网络（SDN）方法提供统一的群集网络，支持跨 OpenShift Origin 群集的 Pod 之间的通信。该 Pod网络由 OpenShift SDN 建立和维护，OpenShift SDN 使用 Open vSwitch（OVS）配置覆盖网络。

OpenShift SDN 提供三个 SDN 插件，用于配置 pod 网络：

- ovs-subnet

	最初的插件，提供一个扁平的 pod 网络，每个 pod 都可以与其他的 pod 和服务进行通讯。
- ovs-multitenant

	作为 pod 和服务提供项目隔离。每个项目都会收到唯一的虚拟网络ID（VNID）,用于识别分配给项目的 pod 流量。来自不同的项目的 pod 将不能将数据包发送或接收。但是，接收 vnid 0 的项目有特权，因为它们可以与所有其他项目的 pod 一起通讯，而所有其他项目的 pod 也可以与它们进行通讯。在OpenShift Origin群集中，默认项目的VNID为0。这有利于某些项目，如负载均衡器，与集群中的所有其他 pod 进行通讯。
	
- ovs-networkpolicy 	

	允许项目管理员使用 NetworkPolicy 对象来配置自己的隔离政策。

有关在 master 和节点上配置SDN的信息，请参阅[配置SDN](https://docs.openshift.org/3.9/install_config/configuring_sdn.html#install-config-configuring-sdn)。

## master 节点设计
在 OpenShift Origin master上，OpenShift SDN 维护一个存储在 etcd 中的节点注册表。当系统管理员注册节点时，OpenShift SDN 会从群集网络中分配未使用的子网，并将此子网存储在注册表中。删除节点时，OpenShift SDN会从注册表中删除子网，并认为可以再次分配子网。

在默认配置中，集群网络是 10.128.0.0/14 网络（即10.128.0.0 - 10.131.255.255），和节点被分配/23的子网（即，10.128.0.0/23,10.128.2.0/23,10.128.4.0/23，等等）。这意味着群集网络有 512 个子网可用于分配给节点，并且给定节点分配510 个可分配给在其上运行的容器的地址。集群网络的大小和地址范围是可配的，主机子网大小也是可配的。

如果子网扩展到下一个更高的8位字节，则将其旋转，以便首先分配共享8位字节中具有0的子网位。

例如，如果网络是10.1.0.0/16,`hostsubnetlength=6` 则在`10.1.0.64/26,10.1.1.64`之前分配`10.1.0.0/26`和`10.1.1.0/26`到`10.1.255.0/26`的子网。 这可确保子网更易于遵循。

请注意，master 上的 OpenShift SDN 不会将本地（master）主机配置为可以访问任何群集网络。因此，master 主机无法通过群集网络访问 pod，除非它也作为节点运行。

使用 `ovs-multitenant` 插件时，OpenShift SDN 主机还会监视项目的创建和删除，并为它们分配 `VXLAN VNID`，这些 `VNLAN` 稍后将被节点用于正确隔离流量。
## 节点设计
在节点上，OpenShift SDN 首先在上述注册表中将本地主机与SDN主机注册，以便主机为该节点分配子网。

接下来，OpenShift SDN 创建并配置三个网络设备：

- br0

	pod 容器将连接到的 OVS 桥接设备。OpenShift SDN 还在此网桥上配置一组非子网特定的流规则。
- tun0

	OVS 内部端口（br0 上的端口2）。这将分配群集子网网关地址，并用于外部网络访问。OpenShift SDN 配置 netfilter 和路由规则，以允许通过NAT从群集子网访问外部网络。
- vxlan_sys_4789

	OVS VXLAN设备（br0 上端口1），提供对远程节点上容器的访问。在OVS规则中被称为 vxlan0 。

每次在主机上启动 pod 时，OpenShift SDN：

- 从节点的群集子网中为 pod 分配一个空闲 IP 地址。
- 将 pod 的 veth 接口对的主机端连接到 OVS 桥 br0。
- 将 `OpenFlow` 规则添加到 OVS 数据库，以将发往新 pod 的流量路由到正确的OVS端口。
- 在 `ovs-multitenant` 插件的情况下，添加 `OpenFlow` 规则来标记来自具有 pod 的 `VNID` 的 pod 的流量，并且如果流量的 `VNID` 与 pod 的 `VNID` 匹配（或者是特权`VNID` 0），则允许流量进入pod。 或通用规则会过滤掉不匹配的流量。

OpenShift SDN 节点还会监视来自 SDN 主站的子网更新。添加新子网时，节点会添加 `OpenFlow` 规则，br0 以便远程子网的目标 IP 地址的数据包进入 vxlan0（br0的端口1），从而进入网络。`ovs-subnet` 插件通过 VNLAN 发送 `VNID` 为0的所有数据包，但`ovs-multitenant` 插件使用适当的 `VNID` 作为源容器。
## 数据包流
假设两个容器 ab，容器 a 的 eth0 对等虚拟网卡设备为 vetha，容器 b 的 eth0 对等虚拟网卡设备为 vethb。如果 docker 服务对对象虚拟以太网设备的使用不熟悉，请参考 [docker 高级网络文档](https://docs.docker.com/engine/userguide/networking/dockernetworks/)。

网络连接的三种情况:

- 容器在相同主机

	假设容器 a 在本地主机上，而容器b也在本地主机上。然后从容器a到容器b 的数据包流:

		eth0 (在a容器中) → vethA → br0 → vethB → eth0 (在b容器中)
- 容器在不同主机

	假设容器a在本地主机上，而容器b在集群网络的远程主机上。然后 a 到 b 的数据包流:

		eth0 (在a容器中) → vethA → br0 → vxlan0 → network [1] → vxlan0 → br0 → vethB → eth0 (在B容器中)
- 容器和主机外部通讯

	假设 a主机连接外部主机，则流量如

		eth0 (在a容器中) → vethA → br0 → tun0 → (NAT) → eth0 (物理设备) → Internet
几乎所有的数据包传输决策都是在 ovs 桥接器 br0 中使用 `OpenFlow` 规则来执行，这简化了插入式网络架构并提供了灵活的路由。在 `ovs-multitenant` 插件情况下，这也提供了可执行的网络隔离。
### 网络隔离
可以使用 `ovs-multitenant` 插件来实现网络隔离。当一个数据包退出分配给非默认项目的数据包时， ovs 网桥 `br0` 会将该数据包标记为项目分配的 `vnid`。
- 相同子网

	如果数据包被引导到节点的群集子网中的另外一个地址，则 ovs 网桥只允许数据包在 `vnid` 匹配的情况下被传递到目标数据包。
- 通过隧道

	如果通过 vxlan 隧道从另一个节点接收到数据包，隧道 id 被用作 `vnid`，如果隧道 id 匹配目标 pod 的 `vnid` 则 ovs 网桥只允许数据包被传送到本地的 pod，。
- 不同子网

	发往其他集群子网的数据包将使用其 `vnid` 进行标记，并将该数据包发送到拥有该集群子网的节点的隧道目标地址的 vxlan 隧道。

vnid0 具有特权，允许具有任何 `vnid` 的流量进入标记 vnid 0的项目，并允许具有 vnid 0 的流量进入任何其他 vnid。只有 `default ` 的项目分配了 vnid 0.所有其他的项目都会被分配一个唯一的隔离启用 `vnid`。集群管理员可以使用管理员 cli 选择性的控制项目的 pod 网络。


