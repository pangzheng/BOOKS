# 配置SDN
## 概观
所述 [OpenShift SDN](https://docs.openshift.org/3.9/architecture/networking/sdn.html#architecture-additional-concepts-sdn) 使得横跨 OpenShift pod，建立一个 pod 之间的通信网络。 目前有三种 [SDN 插件](https://docs.openshift.org/3.9/architecture/networking/sdn.html#architecture-additional-concepts-sdn)（ovs-subnet，ovs- multitenant和ovs-networkpolicy），它们提供了配置pod网络的不同方法。
## 可用的SDN提供商
上游 Kubernetes 项目没有提供默认的网络解决方案。相反，Kubernetes 开发了一个 docker 网络接口（CNI），允许网络供应商与他们自己的SDN解决方案集成。

红帽提供了几个 OpenShift SDN 插件，以及第三方插件。

红帽与许多SDN提供商合作，通过 Kubernetes CNI 接口在 OpenShift 上认证其 SDN 网络解决方案，包括通过其产品授权流程为其 SDN 插件提供支持流程。如果使用 OpenShift 开放支持案例，红帽可以促进交流过程，以便任何公司都能满足需求。

以下SDN解决方案直接由第三方供应商在 OpenShift 上进行验证和支持：

- Cisco Contiv (™)
- Juniper Contrail (™)
- Nokia Nuage (™)
- Tigera Calico (™)
- VMware NSX-T (™)

## 在 OPENSHIFT 上安装 VMWARE NSX-T（™）
VMware NSX-T（TM）提供SDN和安全基础架构，以构建云本机应用程序环境。除 vSphere 管理程序（ESX）外，这些环境还包括KVM和本机公共云。

目前的集成需要 NSX-T 和 OpenShift 的全新安装。目前，支持 NSX-T 2.1版，目前仅支持使用 ESX 和 KVM 管理程序。

有关更多信息，请参阅适用于 [OpenShift的 NSX-T容器插件安装和管理指南](https://docs.vmware.com/en/VMware-NSX-T/2.1/nsxt_21_ncp_openshift.pdf)。

## 用 Ansible 配置 Pod 网络
对于初始[高级安装](https://docs.openshift.org/3.9/install_config/install/advanced_install.html#install-config-install-advanced-install)，默认情况下安装并配置 `ovs-subnet` 插件，尽管可以在安装期间使用 `os_sdn_network_plugin_name` 可在Ansible 清单文件中配置的参数进行覆盖。

例如，要覆盖标准的 `ovs-subnet` 插件成 `ovs-multitenant` 插件：

		＃配置多租户SDN插件（默认为'redhat/openshift-ovs-subnet'）
		os_sdn_network_plugin_name='redhat/openshift-ovs-multitenant'
有关可以在清单文件中设置的与网络有关的Ansible变量的说明，请参阅[配置群集变量](https://docs.openshift.org/3.9/install_config/install/advanced_install.html#advanced-install-networking-variables-table)。
## 在 master 上配置Pod网络
群集管理员可以通过修改[主配置文件](https://docs.openshift.org/3.9/install_config/master_node_configuration.html#install-config-master-node-configuration)（默认情况下位于 /etc/origin/master/master-config.yaml ）`networkConfig` 部分中的参数来控制 pod 网络设置：

为单个CIDR配置pod网络

		networkConfig:
		  clusterNetworks:
		  - cidr: 10.128.0.0/14 #用于节点IP分配的集群网络
		    hostSubnetLength: 9 #	一个节点内的 pod IP分配的位数
		  networkPluginName: "redhat/openshift-ovs-subnet" # 设置 redhat/openshift-ovs-subnet 为OVS大二网插件， redhat/openshift-ovs-multitenant 为 OVS 多租户的插件，或 redhat/openshift-ovs-networkpolicy 为 OVS-networkpolicy 插件
		  serviceNetworkCIDR: 172.30.0.0/16 #服务群集的IP分配
或者，可以创建一个具有多个 CIDR 范围的 Pod 网络，方法是将 `clusterNetworks` 范围和范围分别添加到该范围中 `hostSubnetLength`。

一次可以使用多个范围，范围可以扩大或缩小。通过撤出节点，然后删除并重新创建节点，节点可以从一个范围移动到另一个范围。请参阅 [管理节点](https://docs.openshift.org/3.9/admin_guide/manage_nodes.html#admin-guide-manage-nodes) 。节点分配以列出的顺序进行，然后当范围满时，移动到列表中的下一个。		  

为多个CIDR配置一个pod网络

	networkConfig:
	  clusterNetworks:
	  - cidr: 10.128.0.0/14 #用于节点IP分配的集群网络
	    hostSubnetLength: 9 #一个节点内的pod IP分配的位数
	  - cidr: 10.132.0.0/14
	    hostSubnetLength: 9
	  externalIPNetworkCIDRs: null
	  hostSubnetLength: 9
	  ingressIPNetworkCIDR: 172.29.0.0/16
	  networkPluginName: redhat/openshift-ovs-multitenant #设置多租户插件
	  serviceNetworkCIDR: 172.30.0.0/16
可以添加元素的 `clusterNetworks` 值，要重新启动  `atomic-openshift-master-api` 和 `atomic-openshift-master-controllers` 服务用于任何更改生效。

在 `hostSubnetLength` 第一次创建集群后，该值无法更改。

cidr 只有当节点在其范围内分配且 `serviceNetworkCIDR` 扩展时，会将该字段更改为仍包含原始网络的较大网络 。

例如，给定默认值 `10.128.0.0/14` ，可以更改 cidr 为`10.128.0.0/9`（即网络 10 的整个上半部分），但不会更改为 `10.64.0.0/16`，因为它不会与原始值。

可以 `serviceNetworkCIDR` 从 `172.30.0.0/16` 更改为 `172.30.0.0/15`，但不更改为`172.28.0.0/14`，因为即使原始范围完全位于新范围内，原始范围也必须位于CIDR的起始位置。

## 在节点上配置Pod网络
群集管理员可以通过修改[节点配置文件](https://docs.openshift.org/3.9/install_config/master_node_configuration.html#install-config-master-node-configuration) `networkConfig` 部分中的参数 （缺省情况下位于 `/etc/origin/node/node-config.yaml`）来控制节点上的 pod 网络设置：

	networkConfig:
	  mtu: 1450 #传输单元覆盖网络的最大传输单位（MTU）
	  networkPluginName: "redhat/openshift-ovs-subnet" # ovs 网络插件模式
### 在SDN插件之间迁移
如果您已经在使用一个 SDN 插件并想切换到另一个插件：

- 更改所有[主节点](https://docs.openshift.org/3.9/install_config/configuring_sdn.html#configuring-the-pod-network-on-masters)和[节点](https://docs.openshift.org/3.9/install_config/configuring_sdn.html#configuring-the-pod-network-on-nodes)上配置文件中 `networkPluginName` 参数  。
- 在所有主节点上重新启动 `origin-master-api` 和 `origin-master-controller` 服务。
- 停止所有主节点和节点上的 `origin-node` 服务。
- 如果在OpenShift SDN插件之间切换，请在所有主站和节点上重新启动 `openvswitch` 服务。
- 重新启动所有主节点上的 `origin-node` 服务。
- 如果要从 OpenShift SDN 插件切换到第三方插件，请清除 OpenShift SDN 特定组件

		$ oc delete clusternetwork --all
		$ oc delete hostsubnets --all
		$ oc delete netnamespaces --all

#### 当从 ovs-subnet 切换到 ovs-multitenant OpenShift SDN
所有现有的集群中的项目将完全隔离（分配唯一VNIDs）。群集管理员可以选择使用管理员 CLI [修改项目网络](https://docs.openshift.org/3.9/admin_guide/managing_networking.html#admin-guide-pod-network)。

通过运行检查VNID：

	$ oc get netnamespace
#### 从 ovs-multitenant 迁移到ovs-networkpolicy
在从 `ovs-multitenant` 插件迁移到 `ovs-networkpolicy` 插件之前，请确保每个名称空间都有唯一的名称空间 NetID。这意味着如果之前已将[项目合并在一起](https://docs.openshift.org/3.9/admin_guide/managing_networking.html#joining-project-networks)或将[项目设置为全局项目](https://docs.openshift.org/3.9/admin_guide/managing_networking.html#making-project-networks-global)，则在切换到`ovs-networkpolicy` 插件之前，需要撤消该项目，否则 `NetworkPolicy` 对象可能无法正常运行。

可以修复的帮助程序脚本 `NetID’s`，创建 `NetworkPolicy` 对象以隔离以前隔离的名称空间，并启用以前连接的名称空间之间的连接。

使用以下步骤通过使用此帮助程序脚本迁移到 `ovs-networkpolicy` 插件，同时仍运行 `ovs-multitenant` 插件：

1. 从 `https://raw.githubusercontent.com/openshift/origin/master/contrib/migration/migrate-network-policy.sh`下载脚本并添加exexcution文件权限：

		$ curl -O https://raw.githubusercontent.com/openshift/origin/master/contrib/migration/migrate-network-policy.sh
		$ chmod a+x migrate-network-policy.sh
2. 运行脚本（需要集群管理员角色)

		$ ./migrate-network-policy.sh

运行此脚本后，每个名称空间都与每个其他名称空间完全隔离，因此，在完成向 `ovs-networkpolicy`插件的迁移之前，不同名称空间中的 pod 之间的连接尝试将会失败。

如果希望默认情况下新创建的名称空间也具有相同的策略，则可以将[默认的NetworkPolicy对象](https://docs.openshift.org/3.9/admin_guide/managing_networking.html#admin-guide-networking-networkpolicy-setting-default)设置为与迁移脚本创建的策略 `default-deny` 和 `allow-from-global-namespaces` 创建的策略相匹配。

如果出现脚本故障或其他错误，或者决定要恢复到 `ovs-multitenant` 插件，则可以使用 [un-migration脚本](https://raw.githubusercontent.com/openshift/origin/master/contrib/migration/unmigrate-network-policy.sh)。该脚本撤消了迁移脚本所做的更改并重新加入了先前加入的名称空间。
## 外部访问群集网络
如果 OpenShift 外部的主机需要访问群集网络，则有两种选择：

- 配置主机作为 OpenShift 将其标记[不可调度](https://docs.openshift.org/3.9/admin_guide/manage_nodes.html#marking-nodes-as-unschedulable-or-schedulable) ，这样，主不上安排容器。
- 在主机和群集网络上的主机之间创建一个通道。

这两个选项均作为配置文件中实际使用案例的一部分，用于配置[从边缘负载均衡器到OpenShift SDN中的容器的路由](https://docs.openshift.org/3.9/install_config/routing_from_edge_lb.html#install-config-routing-from-edge-lb)。
## 使用 Flannel
作为默认 SDN 的替代品，OpenShift 还提供 Ansible 系列剧本以安装基于 Flannel 网络。如果在也依赖于 SDN 的云提供商平台（如OpenStack Platform）中运行OpenShift ，并且希望避免通过两个平台两次封装数据包，这将非常有用。

Flannel 为所有容器分配一个连续的空间子集给每个实例使用一个 IP 网络空间。因此，没有任何东西可以阻止容器尝试联系同一网络空间中的任何IP地址。这妨碍了多租户，因为网络不能用于隔离一个应用程序中的容器。

根据更喜欢多租户隔离还是性能，在决定 OpenShift SDN（多租户）和内部网络的 Flannel（性能）时，应该确定适当的选择。

OpenStack 的 Neutron 的当前版本默认强制端口安全。这可以防止端口发送或接收具有与端口本身不同的 MAC 地址的数据包。Flannel 创建虚拟 MAC 和 IP 地址，并且必须在该端口上发送和接收数据包，因此必须在执行 Flannel 流量的端口上禁用端口安全性。

要在 OpenShift 群集中启用 Flannel：

1. Neutron 端口安全控制必须配置为与 Flannel 兼容。OpenStack Platform 的默认配置禁止用户控制 `port_security`。配置 `Neutron` 以允许用户控制 `port_security`各个端口上的设置。
	- 在 Neutron 服务器上，将以下内容添加到 `/etc/neutron/plugins/ml2/ml2_conf.ini` 文件中：

			[ML2]
			...
			extension_drivers = port_security
	- 然后，重新启动Neutron服务

			service neutron-dhcp-agent restart
			service neutron-ovs-cleanup restart
			service neutron-metadata-agentrestart
			service neutron-l3-agent restart
			service neutron-plugin-openvswitch-agent restart
			service neutron-vpn-agent restart
			service neutron-server  restart
2. 在 Red Hat OpenStack 平台上创建 OpenShift实例时，请禁用容器网络 Flannel 接口所在主机端口的端口安全和安全组

		neutron port-update $port --no-security-groups --port-security-enabled=False		
	Flannel 从 etcd 收集信息以配置和分配节点中的子网。因此，连接到 etcd 主机的安全组应允许从节点访问端口 2379/tcp ，并且节点安全组应允许到 etcd 主机上的该端口的出站通信。
	
	- 在运行安装之前，在 Ansible 清单文件中设置以下变量

			openshift_use_openshift_sdn=false #禁用默认的SDN
			openshift_use_flannel=true #启用 flannel
			flannel_interface=eth0
	- 或者，可以使用 flannel_interface 变量指定要用于主机间通信的接口。如果没有这个变量，OpenShift 安装将使用默认接口。

		未来版本将支持定制网络CIDR，用于使用 flannel 的Pod和服务
3. 在 OpenShift 安装之后，在每个 OpenShift 节点上添加一组iptables规则：

		iptables -A DOCKER -p all -j ACCEPT
		iptables -t nat -A POSTROUTING -o eth1 -j MASQUERADE
	要在 `/etc/sysconfig/iptables` 中保留这些更改，请在每个节点上使用以下命令：
	
		cp /etc/sysconfig/iptables{,.orig}
		sh -c "tac /etc/sysconfig/iptables.orig | sed -e '0,/:DOCKER -/ s/:DOCKER -/:DOCKER ACCEPT/' | awk '"\!"p && /POSTROUTING/{print \"-A POSTROUTING -o eth1 -j MASQUERADE\"; p=1} 1' | tac > /etc/sysconfig/iptables"
	该 	`iptables-save` 命令将所有当前内存保存在 iptables 规则中。但是，由于 Docker，Kubernetes 和 OpenShift 会创建大量不适合持久保留的 iptables 规则（服务等），因此保存这些规则可能会产生问题。

为了将容器流量与 OpenShift 流量的其余部分隔离，建议创建一个隔离的租户网络并将所有节点连接到该网络。如果使用的是不同的网络接口（eth1），请记住将接口配置为在引导时通过 `/etc/sysconfig/network-scripts/ifcfg-eth1` 文件启动：

	DEVICE=eth1
	TYPE=Ethernet
	BOOTPROTO=dhcp
	ONBOOT=yes
	DEFTROUTE=no
	PEERDNS=no

