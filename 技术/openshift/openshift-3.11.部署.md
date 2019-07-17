# openshift-部署
## 一 部署主机
部署角色|数量|配置|ip|部署服务
---|---|---|---|---
安装及|1|4c/4g|192.168.1.216|dnsmasq(dns-server), ansible, offline-yumrepo, offline-registry
主控节点|1|4c/8g|192.168.1.214|origin-master, openshift-web-console
计算节点+基础设施|1|16c/24g|192.168.1.215|origin-node, logging, metrics, router, registry

## 二 获取安装包
- 登陆安装机
	
		mkdir -p /data && cd /data
		scp root@打包机:/data/offline.3.11.tar .
		tar xzvf data/offline.3.11.tar
		cd /data/offline-openshift-origin/offline-dmos
		cp ansible.hosts.tpl ansible.hosts.tmp 

## 三 编辑 ansible hosts 文件		
		vi ansible.hosts.tmp

		# Create an OSEv3 group that contains the masters and   nodes groups
		[OSEv3:children]
		masters
		nodes
		etcd
		nfs
		install
		
		# Set variables common for all OSEv3 hosts
		[OSEv3:vars]
		# SSH user, this user should allow ssh based auth without requiring a password
		ansible_ssh_user=root
		ansible_ssh_pass=123
		ansible_port=22
		
		#默认 router 后缀
		openshift_master_default_subdomain=apps181.dmos.dataman
		
		# If ansible_ssh_user is not root, ansible_become must be set to true
		#ansible_become=true
		
		# The default type is openshift-enterprise, and notice that you need to change the type to origin (must check) # 默认版本
		openshift_deployment_type=origin
		openshift_image_tag=v3.11.0
		
		
		# certs exprie days
		# 证书的时间
		openshift_hosted_registry_cert_expire_days=3650
		openshift_ca_cert_expire_days=3650
		openshift_node_cert_expire_days=3650
		openshift_master_cert_expire_days=3650
		etcd_ca_default_days=3650
		
		
		# router selector
		# 路由节点选择器
		openshift_router_selector='node-role.kubernetes.io/infra=true'
		#路由配置，如健康检查，资源限制等
		openshift_hosted_router_edits=[{"key":"spec.strategy.rollingParams.intervalSeconds","value":1,"action":"put"},{"key":"spec.strategy.rollingParams.updatePeriodSeconds","value":1,"action":"put"},{"key":"spec.strategy.activeDeadlineSeconds","value":21600,"action":"put"},{"key":"spec.template.spec.containers[0].resources.limits.cpu","value":1,"action":"put"},{"key":"spec.template.spec.containers[0].resources.limits.memory","value":"512Mi","action":"put"}]
		
		# disable origin repo
		# 关闭线上 repo
		openshift_enable_origin_repo=false
		# set local registry image prefix
		# 设置本地安装源镜像仓库
		system_images_registry="offlineregistry.dataman-inc.com:5000"
		# 组件镜像地址格式
		oreg_url=offlineregistry.dataman-inc.com:5000/openshift/origin-${component}:${version}
		# 添加镜像仓库
		openshift_docker_additional_registries=offlineregistry.dataman-inc.com:5000
		# 添加镜像仓库安全跳过
		openshift_docker_insecure_registries=offlineregistry.dataman-inc.com:5000
		# ？ 
		openshift_examples_modify_imagestreams=true
		
		# network cidr,注意osm_cluster_network_cidr和openshift_portal_net的网段不要和
		#当前集群的网段子网冲突或者包含了当前集群网络的网段地址。
		osm_cluster_network_cidr=10.128.0.0/14
		# svc 地址
		openshift_portal_net=172.30.0.0/16
		
		# sdn network pluging name
		# ovs
		os_sdn_network_plugin_name='redhat/openshift-ovs-networkpolicy'
		# 打开 cni 网络
		# os_sdn_network_plugin_name=cni
		# calico 设置
		# openshift_use_calico=true
		# openshift_use_openshift_sdn=false
		# calico images
		calico_url_policy_controller="offlineregistry.dataman-inc.com:5000/calico/kube-controllers:v3.1.3"
		calico_node_image="offlineregistry.dataman-inc.com:5000/calico/node:v3.1.3"
		calico_cni_image="offlineregistry.dataman-inc.com:5000/calico/cni:v3.1.3"
		calico_upgrade_image="offlineregistry.dataman-inc.com:5000/calico/upgrade:v1.0.5"
		
		# OpenShift Registry Console Options
		# openshift 镜像仓库控制台设置
		openshift_cockpit_deployer_prefix='offlineregistry.dataman-inc.com:5000/cockpit/'
		openshift_cockpit_deployer_version='latest'
		
		#openshift registry selector
		# 镜像仓库选择器
		openshift_registry_selector='node-role.kubernetes.io/infra=true'
		# openshift_hosted_registry_edits
		# 镜像仓库配置
		openshift_hosted_registry_edits=[{"key":"spec.strategy.rollingParams","value":{"intervalSeconds":1,"maxSurge":"25%","maxUnavailable":"25%","timeoutSeconds":600,"updatePeriodSeconds":1},"action":"put"},{"key":"spec.template.spec.containers[0].resources.limits.cpu","value":1,"action":"put"},{"key":"spec.template.spec.containers[0].resources.limits.memory","value":"512Mi","action":"put"}]
		
		# Registry Storage Options
		# 镜像仓库存储配置，默认暂时写成 nfs
		# NFS Host Group
		# An NFS volume will be created with path "nfs_directory/volume_name"
		# on the host within the [nfs] host group.  For example, the volume
		# path using these options would be "/exports/registry"
		openshift_enable_unsupported_configurations=True
		openshift_hosted_registry_storage_kind=nfs
		openshift_hosted_registry_storage_access_modes=['ReadWriteMany']
		openshift_hosted_registry_storage_nfs_directory=/data/exports
		openshift_hosted_registry_storage_nfs_options='*(rw,root_squash)'
		openshift_hosted_registry_storage_volume_name=registry
		openshift_hosted_registry_storage_volume_size=10Gi
		
		# Project Configuration, 项目默认模版，集成dmos填写下面内容，不集成dmos则注释此配置
		# osm_project_request_template='default/project-request'
		
		# Admission plugin config, limit projects, 控制插件配置，集成dmos填写下面内容，不集成dmos则注释此配置
		openshift_master_admission_plugin_config={"ProjectRequestLimit":{"configuration":{"apiVersion":"v1","kind":"ProjectRequestLimitConfig","limits":[{"maxProjects":1,"selector":{"level":"project1"}},{"maxProjects":5,"selector":{"level":"project5"}},{"maxProjects":10,"selector":{"level":"project10"}},{"maxProjects":20,"selector":{"level":"project20"}},{"maxProjects":0}]}},"PodNodeConstraints":{"configuration":{"apiVersion":"v1","kind":"PodNodeConstraintsConfig"}}}
		
		
		# Configure node kubelet arguments. pods-per-core is valid in OpenShift Origin 1.3 or OpenShift Container Platform 3.3 and later.
		# 设置 node 的参数
		openshift_node_kubelet_args={'pods-per-core': ['20'], 'max-pods': ['250'], 'image-gc-high-threshold': ['85'], 'image-gc-low-threshold': ['80']}
		
		# Enable API service auditing
		# 打开审计功能
		openshift_master_audit_config={"enabled": true, "auditFilePath": "/var/lib/origin/openpaas-oscp-audit/openpaas-oscp-audit.log", "maximumFileRetentionDays": 7, "maximumFileSizeMegabytes": 100, "maximumRetainedFiles": 10}
		openshift.master.audit_config=${openshift_master_audit_config}
		
		# Configure imagePolicyConfig in the master config
		# See: https://godoc.org/github.com/openshift/origin/pkg/cmd/server/api#ImagePolicyConfig
		# 镜像流导入策略，默认只导入前5个，现在默认导入无限
		openshift_master_image_policy_config={"maxImagesBulkImportedPerRepository": -1}
		
		# openshift web console
		# 是否安装 web 控制器
		openshift_web_console_install=true
		#openshift_web_console_prefix="offlineregistry.dataman-inc.com:5000/openshift/origin-"
		
		# enable service catalog
		# 是否安装应用目录
		openshift_enable_service_catalog=false
		
		# init docker options (must check)
		# 安装 docker 参数
		openshift_docker_options="--selinux-enabled --signature-verification=false -s overlay2 --log-driver=journald --registry-mirror=https://offlineregistry.dataman-inc.com:5000"
		
		# uncomment the following to enable htpasswd authentication; defaults to DenyAllPasswordIdentityProvider
		# 认证方式
		openshift_master_identity_providers=[{'name': 'htpasswd_auth', 'login': 'true', 'challenge': 'true', 'kind': 'HTPasswdPasswordIdentityProvider'}]
		
		# deployment disable disk,memory,docker image check (must check)
		# 部署检测
		openshift_disable_check=disk_availability,memory_availability
		#docker_image_availability
		
		# journald vars
		# journald 服务配置文件
		journald_vars_to_replace=[{"var":"Storage","val":"persistent"},{"var":"Compress","val":"yes"},{"var":"SyncIntervalSec","val":"1s"},{"var":"RateLimitInterval","val":"1s"},{"var":"RateLimitBurst","val":10000},{"var":"SystemMaxUse","val":"8G"},{"var":"SystemKeepFree","val":"20%"},{"var":"SystemMaxFileSize","val":"1G"},{"var":"MaxRetentionSec","val":"1month"},{"var":"MaxFileSec","val":"1day"},{"var":"ForwardToSyslog","val":"no"},{"var":"ForwardToWall","val":"no"}]
		
		# enable metrics
		# 打开 hawkular 监控
		openshift_metrics_install_metrics=true
		openshift_metrics_hawkular_hostname=hawkular-metrics181.dmos.dataman
		openshift_metrics_cassandra_image="offlineregistry.dataman-inc.com:5000/openshift/origin-metrics-cassandra:v3.11.0"
		openshift_metrics_hawkular_agent_image="offlineregistry.dataman-inc.com:5000/openshift/origin-metrics-hawkular-openshift-agent:v3.11.0"
		openshift_metrics_hawkular_metrics_image="offlineregistry.dataman-inc.com:5000/openshift/origin-metrics-hawkular-metrics:v3.11.0"
		openshift_metrics_schema_installer_image="offlineregistry.dataman-inc.com:5000/openshift/origin-metrics-schema-installer:v3.11.0"
		openshift_metrics_heapster_image="offlineregistry.dataman-inc.com:5000/openshift/origin-metrics-heapster:v3.11.0"
		openshift_metrics_cassandra_nodeselector={"node-role.kubernetes.io/infra":"true"}
		openshift_metrics_hawkular_nodeselector={"node-role.kubernetes.io/infra":"true"}
		openshift_metrics_heapster_nodeselector={"node-role.kubernetes.io/infra":"true"}
		openshift_metrics_duration=7
		# metrics storage
		# 监控存储
		#openshift_metrics_cassandra_storage_type="hostmount"
		#openshift_metrics_cassandra_hostmount_path="/data/metrics-cassandra-data"
		
		# enable logging
		# 打开日志
		openshift_logging_install_logging=true
		openshift_logging_kibana_hostname=kibana181.dmos.dataman
		openshift_logging_image_prefix="offlineregistry.dataman-inc.com:5000/openshift/origin-"
		openshift_logging_es_memory_limit="4Gi"
		openshift_logging_elasticsearch_cpu_request="2"
		openshift_logging_elasticsearch_cpu_limit="2"
		openshift_logging_elasticsearch_proxy_cpu_limit="200m"
		openshift_logging_curator_cpu_limit="200m"
		openshift_logging_kibana_cpu_limit="500m"
		openshift_logging_kibana_proxy_cpu_limit="200m"
		openshift_logging_image_version=v3.10
		openshift_logging_elasticsearch_image="offlineregistry.dataman-inc.com:5000/openshift/origin-logging-elasticsearch5:v3.11"
		openshift_logging_kibana_image="offlineregistry.dataman-inc.com:5000/openshift/origin-logging-kibana5:v3.11"
		openshift_logging_kibana_proxy_image="offlineregistry.dataman-inc.com:5000/openshift/oauth-proxy:v1.1.0"
		#openshift_logging_kibana_proxy_image="offlineregistry.dataman-inc.com:5000/openshift/origin-logging-auth-proxy:v3.11"
		openshift_logging_curator_image="offlineregistry.dataman-inc.com:5000/openshift/origin-logging-curator5:v3.11"
		openshift_logging_fluentd_image="offlineregistry.dataman-inc.com:5000/openshift/origin-logging-fluentd:v3.11"
		openshift_logging_fluentd_audit_container_engine=true
		openshift_logging_kibana_nodeselector={"node-role.kubernetes.io/infra":"true"}
		openshift_logging_es_nodeselector={"node-role.kubernetes.io/infra":"true"}
		openshift_logging_curator_nodeselector={"node-role.kubernetes.io/infra":"true"}
		openshift_logging_curator_default_days=7
		
		# logging storage options
		# 日志存储
		#openshift_logging_elasticsearch_storage_type="hostmount"
		#openshift_logging_elasticsearch_hostmount_path="/data/es-storage"
		#openshift_logging_elasticsearch_container_security_context={"privileged": true}
		
		# enable native prometheus
		# 打开 prometheus
		#openshift_hosted_prometheus_deploy=false
		#openshift_prometheus_node_selector={"node-role.kubernetes.io/infra":"true"}
		
		# enable native grafana
		# 打开 grafana
		#openshift_grafana_image="offlineregistry.dataman-inc.com:5000/mrsiano/grafana-ocp:latest"
		#openshift_grafana_node_selector={"node-role.kubernetes.io/infra":"true"}
		#openshift_grafana_node_exporter=true
		#openshift_grafana_pod_timeout=360
		
		#cluster prometheus
		openshift_cluster_monitoring_operator_image="offlineregistry.dataman-inc.com:5000/coreos/cluster-monitoring-operator:v0.1.1"
		openshift_cluster_monitoring_operator_configmap_reloader_repo="offlineregistry.dataman-inc.com:5000/coreos/configmap-reload"
		openshift_cluster_monitoring_operator_kube_rbac_proxy_image="offlineregistry.dataman-inc.com:5000/coreos/kube-rbac-proxy"
		openshift_cluster_monitoring_operator_kube_state_metrics_image="offlineregistry.dataman-inc.com:5000/coreos/kube-state-metrics"
		openshift_cluster_monitoring_operator_prometheus_reloader_repo="offlineregistry.dataman-inc.com:5000/coreos/prometheus-config-reloader"
		openshift_cluster_monitoring_operator_prometheus_operator_repo="offlineregistry.dataman-inc.com:5000/coreos/prometheus-operator"
		
		# elm
		# operator 镜像
		olm_operator_image="offlineregistry.dataman-inc.com:5000/coreos/olm:0.8.0"
		# catalog 镜像
		olm_catalog_operator_image="offlineregistry.dataman-inc.com:5000/coreos/catalog:0.7.1"
		
		# 节点设置
		openshift_node_groups=[{'name': 'node-config-master', 'labels': ['node-role.kubernetes.io/master=true']}, {'name': 'node-config-infra', 'labels': ['node-role.kubernetes.io/infra=true']}, {'name': 'node-config-compute', 'labels': ['node-role.kubernetes.io/compute=true']}, {'name': 'node-config-infra-compute', 'labels': ['node-role.kubernetes.io/infra=true,node-role.kubernetes.io/compute=true']}]
		
		[install]
		# 安装机
		install180.dmos.dataman
		
		# host group for masters
		[masters]
		# 主控
		master181.dmos.dataman ADVERTISE_IP=192.168.1.181
		
		# host group for nfs
		[nfs]
		# nfs 主控
		master181.dmos.dataman
		
		# host group for etcd
		[etcd]
		# 主控
		master181.dmos.dataman
		
		[infra]
		# 基础设施节点
		infra182.dmos.dataman
		
		# host group for nodes, includes region info
		[nodes]
		master181.dmos.dataman openshift_node_group_name='node-config-master'
		infra182.dmos.dataman openshift_node_group_name='node-config-infra-compute' 
		#node217.dmos.dataman openshift_node_group_name='node-config-compute' 		
## 四 编辑安装机解析
修改配置本机

- 单机版

		cat > /etc/hosts << EOF
		127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
		::1         localhost localhost.localdomain localhost6 localhost6.localdomain6
		
		192.168.1.216 offlineregistry.dataman-inc.com
		192.168.1.216 install216.dmos.dataman
		192.168.1.214 master214.dmos.dataman
		192.168.1.215 infra215.dmos.dataman
		EOF
		cat /etc/hosts
		
- 高可用

		cat > /etc/hosts << EOF
		127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
		::1         localhost localhost.localdomain localhost6 localhost6.localdomain6
		
		192.168.1.200 offlineregistry.dataman-inc.com
		192.168.1.200 install200.shurenyun.com
		192.168.1.151 master151.shurenyun.com
		192.168.1.153 master153.shurenyun.com
		192.168.1.155 master155.shurenyun.com
		192.168.1.156 node156.shurenyun.com
		192.168.1.159 node159.shurenyun.com
		192.168.1.200 lbhatest.shurenyun.com
		EOF
		cat /etc/hosts
		
注: lb节点看机器资源情况酌情规划;若是短域名情况域名规划原则如下：

- 主机域名为:hostname.xxx.yyy格式,其中hostname即为当前主机的主机名，
- 需要确保hostname不冲突.
		
## 五 编辑安装配置文件
	cd /data/offline-openshift-origin/offline-dmos/
	cp config.cfg.example config.cfg
	vi config.cfg

注：配置文件根据注释酌情修改，高可用版需要将HA="false"置为true；默认只需要修改需要改动的区块.

	#!/bin/bash
	set -e
	
	##### 需要改动部分 #####	
	# 当前offline-dmos目录所在路径,默认一般情况下不需要改动，如有变动请根据实际情况修改
	BASE_DIR="/data/offline-openshift-origin/offline-dmos"
	
	# 设置网卡名称
	LOCAL_ENNAME=ens192 ## Need to check
	
	# 定义网关,根据实际情况配置网关ip，网关地址如果没有虚拟的也可以
	GATEWAY_IP="192.168.1.1"
	
	# openshift 路由地址
	OPENSHIFT_ROUTE_IP="192.168.1.182"  # openshift route所在的node 节点ip,默认为/etc/ansible/hosts定义的infra节点,根据实际情况修改, 如果高可用环境配置为同下面LB_OPENSHIFT_HA_VIRTUAL_IPS
	
	# 定义集群是不是使用安装机是不是 dns 或非 dns
	IS_INSTALL_DNS="true" # 值为true或false
	
	# 主机环境实际使用的 dns server，酌情修改,公司内部实际使用的dns服务ip地址,poc建议为空（公司测试环境不建议配置，因为配置后所有节点就都可以访问外网，有可能影响线下包测试的准确性）
	REAL_DNS_IP=""
	
	# 定义当前机器是否来自 ucloud,或者类似 ucloud机器 networkmanager 被禁用的机器,请将此值置为true,注意将
	# ./init_scripts/ucloudConfig.sh 脚本中的网卡名称修改为实际名称.
	# IS_UCLOUD="false" # 默认是false，若是ucloud或类似ucloud机器(networkmanager被禁用)，则值为true
	
	# 定义部署 origin-dm-web,dmos-oc,dmos-oc-ui 部署 tag,开发环境为 dev，测试环境为 test,生产 poc 为 stable,实际情况修改
	DEPLOY_TAG=stable
	
	# 定义读取 imagelist
	if [ "$DEPLOY_TAG" == "dev" ]; then
	        export IMAGELIST_FILE_PATH="./dmos.dev.imagelist.txt"
	else
	        export IMAGELIST_FILE_PATH="./dmos.imagelist.txt"
	fi
	
	# 定义 openshift master1 主机域名 for ansible
	MASTER1_NODE=`${BASE_DIR}/read-ansible-hosts.py 2>/dev/null | awk 'NR==1{print $1}'`
	
	# 获取主机名后续域名, -k insall 的install为ansible hosts 文件的安装节点的section名称
	INSTALL_DOMAIN=`${BASE_DIR}/read-ansible-hosts.py -k install 2>/dev/null | awk '{print $1}'`
	SUFFIX_DOMAIN=${INSTALL_DOMAIN#*.}
	
	# 定义页面隐藏项目,比如："kube-public,kube-system,openshift,openshift-infra" 多个项目以逗号分隔，默认配置空
	DMOS_OPENSHIFT_PROJECTNAMES="kube-public,kube-system,management-infra"
	
	# 定义lb和router的keepalived的虚拟ip,虚拟ip为当前宿主机同一网段，并且未被分配的ip地址;只有高可用部署时用到;两者最后一位差值10+，单机不需要改
	LB_OPENSHIFT_HA_VIRTUAL_IPS="192.168.1.57"
	KA_OPENSHIFT_HA_VIRTUAL_IPS="192.168.1.67"
	
	# 开关项,设置
	CHRONYD_INSTALL="yes" # 是否安装时间同步服务(chronyd),yes表示安装，no表示不安装，默认yes，只有特殊情况不进行安装;
	IS_SHORT_DOMAIN="no" # 主机名是否为短域名,yes 表示是,no表示长域名,默认长域名;当主机名为短域名时也就是主机域名的
	                  # 第一段作为主机名,此主机名和客户定义主机名(必须小写)的一致；此时主机的域名规划为"主机名.xxx.yyy"方式
	
	# 是否为openshift社区版,默认true表示为社区版，false表示企业版.
	IS_ORIGIN="true"
	
	# 当对接企业版时 SOLAR_SWITCH 默认为 false，也就是不安装 solar
	if [ "x$IS_ORIGIN" == "xtrue" ]; then
	        SOLAR_SWITCH="true" # 是否部署solar，默认true表示部署；false表示不部署.
	elif [ "x$IS_ORIGIN" == "xfalse" ]; then
	        SOLAR_SWITCH="false" # 是否部署solar，默认true表示部署；false表示不部署.
	fi
	
	# 是否开启selinux，默认true表示开启；false表示关闭
	SELINUX_SWITCH="true" 
	# 是否禁用swap，默认false禁用；true表示开启
	SWAP_SWITCH="false" 
	# 是否对接外部dns,默认false不对接，true表示对接
	OUTLINE_DNS="false" 
	# 是否更新操作系统及内核, 默认不更新;
	IS_SYSTEM_UPGRATE="false" 
	
	# 定义外部镜像仓库
	# 默认无需改动,当对接客户的企业版 ocp 环境时，需要修改为客户的外部镜像仓库地址;
	REGISTRY_EXP="offlineregistry.dataman-inc.com:5000"    	# 默认无需改动,当对接客户企业版ocp环境时,部署dmos所需的镜像源镜像仓库;
	REGISTRY_EXP_SRC="offlineregistry.dataman-inc.com:5002" 	
	# 定义dmos审计日志参数
	DMOS_AUDIT_FILEPATH="/var/lib/origin/dmos-audit"  # dmos审计日志生成的目录;
	DMOS_AUDIT_FILENAME="dmosaudit"  # dmos审计日志生成的文件名称;
	DMOS_AUDIT_MAXFILESIZE="100MB"  # dmos审计日志大小，单个文件最大为100MB;
	DMOS_AUDIT_MAXHISTORY="7"  # dmos审计日志保存7天；
	DMOS_AUDIT_TOTALSIZECAP="10GB"  # dmos审计日志的目录文件总大小为10GB;
	
	##### 无需改动部分 #####
	# 高可用部署开关, true 为部署高可用版，false 为单机版,自动判断获取，一般无需改动
	grep ^openshift_master_cluster_public_hostname /etc/ansible/hosts &>/dev/null && HA="true" || HA="false"
	
	if [ "x$HA" == "xtrue" ];then
	        LB_DOMAIN=`grep openshift_master_cluster_hostname /etc/ansible/hosts 2>/dev/null | awk -F= 'NR>1{print $2F}'`
	        # 定义访问origin-dm-web,dmos-oc,dmos-oc-ui,solar入口地址
	        ACCESS_DOMAIN="$LB_DOMAIN"
	        # 获取所有masters域名
	        MASTER_DOMAINS=`${BASE_DIR}/read-ansible-hosts.py 2>/dev/null | awk '{print $1}'`
	else
	        # 定义访问origin-dm-web,dmos-oc,dmos-oc-ui,solar入口地址
	        ACCESS_DOMAIN=$MASTER1_NODE
	        MASTER_DOMAINS=$MASTER1_NODE
	fi
	
	export LOCAL_IP=`ip a show $LOCAL_ENNAME|awk '/inet.*brd.*'$LOCAL_ENNAME'/{print $2}'|awk -F "/" '{print $1}'`  # 无需改动
	if [ -z "$LOCAL_IP" ];then
	        echo "get $LOCAL_ENNAME ip error!" && exit 1
	fi
	
	# 当前集群网段反向解析ip段,这是假设整个集群在同一个网段内
	REVERSE_SUB_NET=`echo $LOCAL_IP | awk -F. '{print $2"."$1}'`
	
	# yum repo && config server
	export CONFIGSERVER_IP="$LOCAL_IP"
	export CONFIGSERVER_PORT="8081"
	
	NTP_IP="$LOCAL_IP"
	
	# openshift image tag
	OKD_IMAGE_VERSION=`cat /etc/ansible/hosts 2>/dev/null | grep -v "#" | grep "openshift_image_tag" | awk -F= '{print $2}'`	
	
	# solar config
	SOLAR_MASTER_ADDR="$ACCESS_DOMAIN":88  # 定义solar 服务域名,需要根据实际改动,一般无需改动
	SOLAR_URL="http://$SOLAR_MASTER_ADDR"  # 定义solar 服务url，需要根据实际改动，一般无需改动
	SOLAR_USERNAME="admin"    # 定义solar管理员用户名，根据实际改动，一般无需改动
	SOLAR_PASSWORD="admin"    # 定义solar管理员密码，目前会在第一次安装install_dmos阶段随机生成一个随机的复杂密码;后续重装若是还需要改动则需要修改此变量值。
	SOLAR_CLUSTER_NAME="dmos" # solar cluster name, 根据实际改动，一般无需改动
	
	# solar 全局变量定义,OPENSHIFT_MASTER_API_URL高可用版是取/etc/ansible/hosts的openshift_master_cluster_hostname的值，
	#要不然就是取第一个master的域名，例如：https://lbhatest200.dmos.dataman:8443或https://master214.dmos.dataman:8443;
	#CERTIFICATE_EXPIRE_DAYS证书有效期，默认十年看具体情况修改，默认可以不修改,IMAGE_VERSION镜像版本，随openshift版本变化而变化;
	SOLAR_GLOBAL_ATTRS="OPENSHIFT_MASTER_API_URL=https://$ACCESS_DOMAIN:8443  CERTIFICATE_EXPIRE_DAYS=3650  IMAGE_VERSION=$OKD_IMAGE_VERSION CONFIGSERVER_IP=$CONFIGSERVER_IP CONFIGSERVER_PORT=$CONFIGSERVER_PORT  os_sdn_network_plugin_name={{os_sdn_network_plugin_name}} SUFFIX_DOMAIN=$SUFFIX_DOMAIN DNS_SERVER_IPS=$CONFIGSERVER_IP CHRONYD_INSTALL=$CHRONYD_INSTALL IS_SHORT_DOMAIN=false OUTLINE_DNS=$OUTLINE_DNS"
	
	# 定义openshift master主机组for ansible
	MASTER_NODES="masters" # 无需改动
	
	# 获取app router子域名
	SUB_DMOAIN=`grep openshift_master_default_subdomain /etc/ansible/hosts 2>/dev/null | awk -F '=' '{print $2}'`
	
	# 定义openshift内部镜像仓库
	REGISTRY_OPENSHIFT="docker-registry-default.$SUB_DMOAIN" # 默认无需改动
	
	# 定义login-template.html源文件所在路径, 默认无需改动
	LOGIN_TPL_PATH="./tpl_file/login-template.html"
	
	# 定义ansible拷贝目标路径, 默认无需改动
	ANSIBLE_TARGET_PATH="/data/dmos"
	
	# 定义dmos-oc部署先关环境变量
	DMOS_OC_TPL_NAME="dmos-oc-daemonset"   # 默认无需改动
	
	# 定义汉化服务发布名称及先关环境变量
	TPL_NAME="polypite-daemonset"   # 此汉化前端的模板名称，需要与汉化前端的模板定义的名称一致，当前为polypite,默认无需改动
	LISTEN_PORT="9443"  # 汉化前端console端口，固定9443
	LOGGING_URL="https://`grep openshift_logging_kibana_hostname /etc/ansible/hosts 2>/dev/null | awk -F= '{print $2}'`"  # 一般将LOGGING_URL改成和/etc/origin/master/master-config.yaml里assetConfig的loggingPublicURL保持一致,默认无需改动
	METRICS_URL="https://`grep openshift_metrics_hawkular_hostname /etc/ansible/hosts 2>/dev/null | awk -F= '{print $2}'`/hawkular/metrics"  # 一般将METRICS_URL改成和/etc/origin/master/master-config.yaml里assetConfig的metricsPublicURL保持一致
	;默认无需改动
	
	# 是否安装默认prometheus grafana
	NATIVE_PROMETHEUS_DEPLOY=`grep 'openshift_hosted_prometheus_deploy' /etc/ansible/hosts 2>/dev/null | awk -F '=' '{print $2}'`
	
## 六 部署
### 第一步
- 进入目录

		cd /data/offline-openshift-origin/offline-dmos/

- 初始化集群主机

		./base_init.sh
	注： 初始化完毕后会重启所有主机，为此下一步执行安装操作时需要重新ssh至安装机。

### 第二步
- 安装(此安装包含 openshift+origin-dm-web+dmos-oc )

		./install_openshift.sh
- 授权

	如果只装 openshift ，用以下命令到 openshift master 进行授权
	
	- 创建用户密码(htpaaswd)
		- 登陆 master 主机
		
				htpasswd /etc/origin/master/htpasswd cluster_admin
				oc adm policy add-cluster-role-to-user admin cluster_admin
	- 修改 scc
		- 授权登陆用户组可以使用任何uid启动服务，默认只能使用非 root uid 

				oc adm policy add-scc-to-group anyuid system:authenticated
		- 授权登陆用户组可以使用超权

				oc adm policy add-scc-to-group privileged system:authenticated		 
## 七 平台访问
1. 单机版 https://${master1_domain}:9443
2. 高可用 https://${lb_domain}:1443

	注：当前高可用版lb以及router已经支持高可用，为此lb的域名以及路由的域名均配置为虚拟ip，虚拟ip
通过config.cfg查看获取。

	注：安装完 openshift 后，会设置 /etc/resolv.conf 为只读文件，防止重启网络后会重置 /etc/resolv.conf,如果需要修改 /etc/resolv.conf 内容，需要执行 chattr -i /etc/resolv.conf,	修改完内容之后再设置为只读执行 chattr +i /etc/resolv.conf

- 镜像仓库的域名解析设置到所有节点的/etc/hosts上
- 执行./init_scripts/set-audit-qouta-networkpolicy.sh audit 开启审计功能；
- 执行./init_scripts/set-audit-qouta-networkpolicy.sh qouta 设置qouta；
- 若是ansible hosts文件os_sdn_network_plugin_name的配置是redhat/openshift-ovs-networkpolicy，但实际环境中的网络策略并非redhat/openshift-ovs-networkpolicy；并且又需要修改为redhat/openshift-ovs-networkpolicy时，请按如下操作(注： 客户环境需要和客户沟通是否进行容器网络方案调整，最好让客户自行调整，本文只是针对自己环境进行说明):

	- 若是3.10之前的版本可以在安装机执行：
	
		    ./init_scripts/set-audit-qouta-networkpolicy.sh network
	- 若是3.10之后版本需要登录到其中一台master节点；
	- oc project openshift-node;
	- oc get configmap
	- 采用oc edit configmap 命令将oc get configmap命令获取到的configmap的网络策略均设置为redhat/openshift-ovs-networkpolicy，保存退出。
- 执行./init_scripts/set-audit-qouta-networkpolicy.sh project 设置创建项目的默认模板；
- 执行./install_dmos.sh	

## 八 卸载
	cd /data/offline-openshift-origin/offline-dmos/
	./uninstall_openshift.sh
	注： 针对企业版不能执行此操作.
	
## 九 重装
	cd /data/offline-openshift-origin/offline-dmos/
	./reinstall_openshift.sh
	注： 此卸载以及重装是在不部署glusterfs情况下适用。


## 十 nfs 持久化存储卷创建
登录安装机,切换到 offline-dmos 的 nfs 目录，根据 config.cfg 配置文件修改相应的配置：

	cd offline-dmos/nfs

修改config.cfg

	#!/bin/bash
	set -e
	
	# 定义nfs服务ip所对应的域名地址以及ip,注意这个域名必须在ansible hosts文件中定义了
	NFS_SERVICE_DOMAIN="glusterfs3.dev.dataman.com"
	NFS_SERVICE_IP="192.168.1.223"
	
	# 定义openshift master1主机域名 for ansible
	MASTER1_NODE="master1.dev.dataman.com"
	
	# 定义服务pv集,服务pv为模块(属于哪个产品线避免冲突,-的第一个字段即为模块名)名+应用名;多个服务pv以逗号分隔;此变量用来创建服务的nfs-pv
	APP_SET="octopus-kafka-service,octopus-octopus-db,octopus-octopus-zookeeper,squid-squid-db,squid-kafka,squid-zk,hawk-bootstrap-etcd,hawk-portal-db"
	
	# 定义创建pv的模板文件包含路径,此变量路径请不要发生变化，
	# 单独执行create_pv.py脚本传参路径无所谓
	PV_TPL_FILE="/data/dmos/pvTpl/nfs-pv.yaml"

执行一键创建pv脚本:

	当前目录在offline-dmos目录下
	cd ./nfs
	sh pv_create.sh