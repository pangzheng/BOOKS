## 独立部署一个 openshift 镜像仓库
ocp 是一个工嗯那个齐全的 paas 企业解决方案，其中包含一个 OpenShift Container Registry（OCR） 的集成 registry。当然可以独立部署一个 OCR。

当 OCR 独立部署时，仍然安装了一组 master 和 node 节点，类似 ocp 经典安装。然后包含 registry 被部署在集群上运行。这种独立部署对于需要 registry 不需要包含面向开发人员的 web 控制台和应用构建部署工具。 OCR 不应该与上游项目 atomic registry 混淆，后者是给非 k8s 部署方法的。利用的是 systemd 和本地配置文件来管理。

OCR 提供以下功能

- 需要的 web 控制台
- 默认安全流量，通过 TLS 提供
- 全局身份验证
- 项目命名空间模型，可以使团队能够基于角色的访问控制 RBAC 授权进行协作。
- 基于 kubernetes 的集群管理
- 抽象的镜像管理

管理员可以独立部署 ocr 已分别管理多个 ocp 的 registry。独立的 OCR 还使管理员能够分离其 registry 以满足安全。

### 最低硬件配置
- OS
	- RH 7.3/7.4 版本最小安装
	- NetworkManager 1.0 或者更新
- 硬件
	- 1C/8GB
	- disk
		- /+/var 最小 16 GB
		- 未分配空间给 docker 使用最小 15 GB

### 部署拓扑
- 一体

	包含 master 、node、etcd和 registry 的单主机
- 多 master(高可用) 

	每个主机包含 master 、node、etcd和 registry。

### 主机准备
同安装 ocp
### 安装方法
安装独立 registry，使用 ocp 的标准安装方法。

- 快速安装

	当使用快速安装时，需要启动交互式安装
	
		$ atomic-openshift-installer install
	然后按照要求交互，同 ocp 安装
	
		Which variant would you like to install?


		(1) OpenShift Container Platform
		(2) Registry	
- 高级安装

	高级安装 OCR，请使用 ocp 相同方法，使用完整的高级表述文件。主要区别在于 `inventory` 中设置的 `deployment_subtype=registry`
	
	- 单机
	
			[OSEv3:children]
			masters
			nodes
		
			[OSEv3:vars]
			ansible_ssh_user=root
			openshift_master_default_subdomain=apps.test.example.com
			openshift_deployment_type=openshift-enterprise
			deployment_subtype=registry #这里不同

			[masters]
			registry.example.com

			[nodes]
			registry.example.com openshift_node_labels="{'region': 'infra', 'zone': 'default'}" openshift_schedulable=true  #这里调度需要设置为开
	- 多机

			[OSEv3:children]
			masters
			nodes
			etcd
			lb

			[OSEv3:vars]
			ansible_ssh_user=root
			openshift_deployment_type=openshift-enterprise
			deployment_subtype=registry 

			openshift_master_default_subdomain=apps.test.example.com


			openshift_master_cluster_method=native
			openshift_master_cluster_hostname=openshift-cluster.example.com
			openshift_master_cluster_public_hostname=openshift-cluster.example.com


			openshift_node_kubelet_args={'pods-per-core': ['10'], 'max-pods': ['250'], 'image-gc-high-threshold': ['90'], 'image-gc-low-threshold': ['80']}

			openshift_clock_enabled=true

			[masters]
			master1.example.com
			master2.example.com
			master3.example.com

			[etcd]
			etcd1.example.com
			etcd2.example.com
			etcd3.example.com

			[lb]
			lb.example.com

			[nodes]
			master[1:3].example.com openshift_node_labels="{'region': 'infra', 'zone': 'default'}" openshift_schedulable=true
			node1.example.com openshift_node_labels="{'region': 'primary', 'zone': 'east'}"
			node2.example.com openshift_node_labels="{'region': 'primary', 'zone': 'west'}"
通过 `/etc/ansible/hosts` 定义的配置文件后，可以执行以下命令进行安装
	
			# ansible-playbook /usr/share/ansible/openshift-ansible/playbooks/byo/config.yml			