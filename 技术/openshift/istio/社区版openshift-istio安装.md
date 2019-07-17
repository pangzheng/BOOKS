# 社区版 openshift-istio 安装
本文档介绍了将 Istio Tech Preview 版本安装到使用 OpenShift ansible 安装程序安装的现有 OCP 3.11 安装中的步骤。如果想使用，`oc cluster up` 请查看 [https://github.com/Maistra/origin/releases](https://github.com/Maistra/origin/releases)，oc 需要包含一个 Istio operator 的版本。
## 准备工作
在将 Istio 安装到 OCP 3.11 安装之前，必须对 master 配置和每个可调度节点进行一些更改。这些更改将启用 Istio 中所需的功能，并确保 Elasticsearch 正常运行。

### 更新 master
为了能够自动注入 Istio sidecar，首先需要修改每个 master 上的主配置，以包括对 webhook 的支持和证书签名请求（CSR）的签名。

在 OCP 3.11 安装中的每个 master 服务器上进行以下更改。

- 切换到包含 master 配置文件的目录（master-config.yaml）
- 使用以下内容创建名为 `master-config.patch` 的文件

		admissionConfig:
		  pluginConfig:
		    MutatingAdmissionWebhook:
		      configuration:
		        apiVersion: apiserver.config.k8s.io/v1alpha1
		        kubeConfigFile: /dev/null
		        kind: WebhookAdmission
		    ValidatingAdmissionWebhook:
		      configuration:
		        apiVersion: apiserver.config.k8s.io/v1alpha1
		        kubeConfigFile: /dev/null
		        kind: WebhookAdmission
- 在同一目录中发出以下命令

		cp -p master-config.yaml master-config.yaml.prepatch
		oc ex config patch master-config.yaml.prepatch -p "$(cat master-config.patch)" > master-config.yaml
		/usr/local/bin/master-restart api && /usr/local/bin/master-restart controllers

### 更新节点
为了运行 Elasticsearch 应用程序，必须对每个节点上的内核配置进行更改，此更改将通过该 sysctl 服务进行处理。在每个节点上进行以下更改

- 创建一个 `/etc/sysctl.d/99-elasticsearch.conf` 使用以下内容命名的文件：

		vm.max_map_count = 262144
- 再执行以下命令

		sysctl vm.max_map_count=262144	

### 获取发布麽版
[发布模版在这里获取](https://github.com/pangzheng/openshift-ansible/tree/maistra-0.9.0/istio),主要分4个类型的文件

- 安装手册

	Installation.md 安装手册
	
	[Installation.md](https://github.com/pangzheng/openshift-ansible/blob/maistra-0.9.0/istio/Installation.md)
- istio operator 的发布麽版

	部署 istio operator 需要授权和发布等操作，所以这里将所有这些操作做成了模版，并且把关键参数进行 ENV 抽象。
	
	- 好处

		发布方便
	- 坏处

		重新发布比较复杂，不如使用 `oc apply -f`  声明式方便
	- 版本区分，主要取决于镜像获取的地址
		- [企业版](https://github.com/pangzheng/openshift-ansible/blob/maistra-0.9.0/istio/istio_product_operator_template.yaml)
		- [社区版](https://github.com/pangzheng/openshift-ansible/blob/maistra-0.9.0/istio/istio_community_operator_template.yaml) 
- master 配置

	用来开放 master 的准入控制器功能的。
	
	[master-config.patch](https://github.com/pangzheng/openshift-ansible/blob/maistra-0.9.0/istio/master-config.patch) 
- operator 发布 istio 配置，主要负责配置整体系统的发布大小，分为3个类型
	- [完整安装](https://github.com/pangzheng/openshift-ansible/blob/maistra-0.9.0/istio/istio-installation-full.yaml)
	- [额外 kiali 安装](https://github.com/pangzheng/openshift-ansible/blob/maistra-0.9.0/istio/istio-installation-kiali.yaml)
	- [最小化安装](https://github.com/pangzheng/openshift-ansible/blob/maistra-0.9.0/istio/istio-installation-minimal.yaml) 

## 使用镜像列表
这里列出了所有需要的镜像和对应的启动对象。注意这里没有给出镜像仓库的都是在 docker-hub,如果要公网安装，因为众所周知的原因，可以使用 mirror 源。线下部署，可以提前下载放到线下对应的私有仓库中。(现在还没测试成功)

- 支撑服务
	- 日志系统 
		- elastiseatch(stateful sets)

				registry.centos.org/rhsyseng/elasticsearch:5.6.10
	- 监控系统
		- grafana(deployment)

				grafana/grafana:5.4.0
		- prometheus(deployment)

				prom/prometheus:v2.3.1
- 部署系统
	- origin-ansible(job)	
		
			maistra/origin-ansible:0.9.0
	- istio-operator(deployment)

			maistra/istio-operator-centos7:0.9.0
- istio
	- istio-citadel(deployment)

			istio/citadel:1.1.0-rc.2
	- istio-egressgateway(deployment)

			istio/proxyv2:1.1.0-rc.2
	- istio-galley(deployment)

			istio/galley:1.1.0-rc.2
	- istio-ingressgateway(deployment)

			istio/proxyv2:1.1.0-rc.2
	- istio-pilot（deployment）

			istio/pilot:1.1.0-rc.2
			istio/proxyv2:1.1.0-rc.2
	- istio-mixer
		- istio-policy(deployment)

				istio/mixer:1.1.0-rc.2
				istio/proxyv2:1.1.0-rc.2
		- istio-telemetry(deployment)

				istio/mixer:1.1.0-rc.2
				istio/proxyv2:1.1.0-rc.2 
	- istio-sidecar-injector(deployment)
	
			istio/sidecar_injector:1.1.0-rc.2
- jaeger
	- jaeger-collector(deployment)

			jaegertracing/jaeger-collector:1.11
	- jaeger-query(deployment)

			jaegertracing/jaeger-query:1.11
	- jaeger-agent(daemonset)

			jaegertracing/jaeger-agent:1.11
- fabric8-launcher
	- configmapcontroller(deployment)

			fabric8/configmapcontroller:2.3.7
	- launcher-backend(deployment)

			fabric8/launcher-backend:ab535bc
	- launcher-frontend(deployment)

			fabric8/launcher-frontend:4571dfc
			
## 部署 Istio Operator	
### 安装 istio operator
Maistra 安装过程引入了 Operator 来管理 `istio-syste` 命名空间内 Istio 控制平面的安装。此 Operator 定义和监视与 Istio 控制平面的部署，更新和删除相关的自定义资源。

以下步骤将 Maistra Operator 安装到现有的集群中，这些操作可以从具有集群访问权限的任何主机执行。请确保在执行以下操作之前以群集管理员身份登录	


- 社区版

		oc new-project istio-operator
		oc new-app -f istio_community_operator_template.yaml --param=OPENSHIFT_ISTIO_MASTER_PUBLIC_URL=<master public url>
		
	例
	
		oc new-app -f istio_community_operator_template.yaml --param=OPENSHIFT_ISTIO_MASTER_PUBLIC_URL=https://master195.ds.com:8443 --param=OPENSHIFT_ISTIO_PREFIX=registry.ds.com/maistra/
- 企业版

		oc new-project istio-operator
		oc new-app -f istio_product_operator_template.yaml --param=OPENSHIFT_ISTIO_MASTER_PUBLIC_URL=<master public url>


### 验证安装
上述指令将在 istio-operator 项目中创建一个新部署，执行负责通过自定义资源管理 Istio 控制平面状态的 operator。

要验证 operator 是否已正确安装，请使用以下命令从 operator pod 访问日志

	# oc logs -n istio-operator $(oc -n istio-operator get pods -l name=istio-operator --output=jsonpath={.items..metadata.name})
	
	time="2019-03-25T04:11:31Z" level=info msg="Go Version: go1.11.5"
	time="2019-03-25T04:11:31Z" level=info msg="Go OS/Arch: linux/amd64"
	time="2019-03-25T04:11:31Z" level=info msg="operator-sdk Version: 0.0.5+git"
	time="2019-03-25T04:11:31Z" level=info msg="Metrics service istio-operator created"
	time="2019-03-25T04:11:31Z" level=info msg="Watching resource istio.openshift.com/v1alpha1, kind Installation, namespace istio-operator, resyncPeriod 0"  #<- 注意这里是还没有导入 istio 发布配置的时候显示
	time="2019-03-25T04:12:01Z" level=info msg="Installing istio for Installation istio-installation"	 #<- 注意这里是导入 istio 发布配置的后显
## 部署 Istio 控制平面
为了部署 Istio 控制平面，需要部署自定义资源，例如下例，该例演示了 operate 支持的配置选项。必须将自定义资源部署到 istio-operator 命名空间中，并且必须调用它 istio-installation。
	
	apiVersion: "istio.openshift.com/v1alpha1"
	kind: "Installation"
	metadata:
	  name: "istio-installation"
	  namespace: istio-operator
	spec:
	  deployment_type: origin  #<- 这里默认是 openshift 企业版，社区版一定要修改，否则获取镜像名不正确。
	  istio:
	    authentication: true
	    community: true
	    prefix: registry.private.com/maistra/  #<- 变更镜像前缀
	    version: 0.9.0
	  jaeger:
	    prefix: registry.paradeum.com/jaegertracing/  #<- 变更镜像前缀
	    version: 1.11.0
	    elasticsearch_memory: 1Gi
	  kiali:
	    username: cluster_admin #<- kiali 自己系统的账户，没有用 sso
	    password: 123456 #<- kiali 自己系统的密码，没有用 sso
	    prefix: registry.private.com/kiali/ #<- 变更镜像前缀
	    version: v0.15.0
	  launcher:
	    openshift:
	      user: cluster_admin  #<- openshift 用户名
	      password: @@@@@@@ #<- openshift 密码
	    github:
	      username: dxc520 #<- github 用户名
	      token: a65c4b21fb68b94f6c7dd259a3846dd68ea6f419 #<- github token
	    catalog:
	      filter: booster.mission.metadata.istio
	      branch: v85
	      repo: https://github.com/fabric8-launcher/launcher-booster-catalog.git
	  threeScale:
	    enabled: false
	    prefix: quay.io/3scale/ #<- 变更镜像前缀
	    version: 0.4.1
	    adapter:
	      listenAddr: 3333
	      logLevel: info
	      logJSON: true
	      reportMetrics: true
	      metricsPort: 8080
	      cacheTTLSeconds: 300
	      cacheRefreshSeconds: 180
	      cacheEntriesMax: 1000
	      cacheRefreshRetries: 1
	      allowInsecureConn: false
	      clientTimeoutSeconds: 10

导入配置，并需要等待

	oc apply -f istio-installation.yaml

## 部署验证
operator 将在 `istio-system` 命名空间创建和运行安装程序作业，此作业将使用 `Ansible playbooks` 设置 Istio 控制平面。安装进度可以通过观察 pod 或从 `openshift-ansible-istio-installer-job` pod 输出日志输出来进行。 		

	oc get pods -n istio-system -w
	
如果顺利安装完成，pod 中的	`openshift-ansible-istio-installer-job` 就会显示 `Completed` 状态。

	# oc get pods -n istio-system
	NAME                                          READY     STATUS      RESTARTS   AGE
	elasticsearch-0                               1/1       Running     0          4h
	grafana-74b5796d94-bdppr                      1/1       Running     0          4h
	istio-citadel-6f8968dfdc-rb2cl                1/1       Running     0          4h
	istio-egressgateway-7d999cdc45-dbfl2          1/1       Running     0          4h
	istio-galley-7749cccdc6-5dlc8                 1/1       Running     0          4h
	istio-ingressgateway-7d5bc4dc64-8qznj         1/1       Running     0          4h
	istio-pilot-657f75d985-2t47x                  2/2       Running     0          4h
	istio-policy-656545b844-2tj4r                 2/2       Running     4          4h
	istio-sidecar-injector-cd9d649fd-9l8jp        1/1       Running     0          4h
	istio-telemetry-7967f989bc-6s5mh              2/2       Running     3          4h
	jaeger-agent-ctj2v                            1/1       Running     0          4h
	jaeger-collector-64bc5678dd-rgd48             1/1       Running     0          4h
	jaeger-query-776d4d754b-928dn                 1/1       Running     0          4h
	kiali-56f668749d-vv9sb                        1/1       Running     0          4h
	openshift-ansible-istio-installer-job-9fhfh   0/1       Completed   0         4h
	prometheus-75b849445c-rcgql                   1/1       Running     0          4h			 
	
注意，如果选择了 Farbic8 启动器，可以查看 `devex` 项目

	# oc get pods -n devex
	NAME                          READY     STATUS    RESTARTS   AGE
	configmapcontroller-1-d64j2   1/1       Running   0          4h
	launcher-backend-2-htlq7      1/1       Running   0          4h
	launcher-frontend-2-k8khf     1/1       Running   0          4h

## 删除 Istio
以下步骤将从现有安装中删除 Istio，它可以从具有集群访问权限的任何主机执行。

	oc delete -n istio-operator Installation istio-installation
### 删除 Istio operator
以下步骤将从现有的 OCP 3.11 安装中删除 Maistra operator，这些操作可以从具有群集访问权限的任何主机执行。请确保在执行以下操作之前以群集管理员身份登录

- 社区版

		oc process -f istio_community_operator_template.yaml | oc delete -f -
- 企业版

		oc process -f istio_product_operator_template.yaml | oc delete -f -

## 简单测试 istio sidecar 注入
### 注入镜像
- istio/proxyv2:1.1.0-rc.2
- istio/proxy_init:1.1.0-rc.2

### sleep 测试程序
- 创建文件
	
		vi sleep.yaml 
		
		apiVersion: extensions/v1beta1
		kind: Deployment
		metadata:
		  name: sleep
		spec:
		  replicas: 1
		  template:
		    metadata:
		      annotations:
		        sidecar.istio.io/inject: "true"
		      labels:
		        app: sleep
		    spec:
		      containers:
		      - name: sleep
		        image: tutum/curl
		        command: ["/bin/sleep","infinity"]
		        imagePullPolicy: IfNotPresent
- 启动

		oc apply -f  sleep.yaml
- 查看
			        
		
## 问题
- 问题1
	- 现象
	
		istio-operator 无法发布,日志提示 `Error creating deployer pod: Internal error occurred: failed calling admission webhook "sidecar-injector.istio.io": Post https://istio-sidecar-injector.istio-system.svc:443/inject?timeout=30s: dial tcp 172.30.190.192:443: connect: connection refused`
	- 分析原因

		因为之前部署的 istio operator 有问题，所以删除了 operator 相关的所有数据重新部署，但是因为 istio sidecar 控制器已经注册到了 `MutatingWebhookConfiguration` 列表中，所以再准入审计的时候还会生效，所以导致这个问题。可以使用以下命令查看
		
			oc get MutatingWebhookConfiguration
		
		从这里推导出，如果注册控制器失效会给新建任务带来无法发布结果，但是原先发布的服务不再其中，即使重启。
	- 解决方案

		为 `istio-operator` 增加准入控制忽略标签
		
			oc label ns istio-operator istio.openshift.com/ignore-namespace=yes
- 问题2
	- 现象

		发布 istio sidecar 控制器报错，无法通过健康检查
	- 分析原因

		因为属于全自动部署，怀疑是 0.8.0 版本(参考红帽官方最新文档)有bug
	- 解决方案

		更新最新稳定版本(0.9.0)
- 问题3
	- 现象

		各种服务挂
	- 分析原因
		- 通过看日志发现 istio 的一个服务不正常
		
				# telnet istio-galley.istio-system.svc 9901
				Trying 172.30.236.77...
				telnet: connect to address 172.30.236.77: No route to host
		- 直接访问容器 ip 

			发现访问不了
		- 发现该节点的 sdn 服务异常

			各种超时	
	- 解决方案

		因为服务太多了，导致响应满，相应满导致 istio 异常， istio 异常导致各种重启，各种重启带来响应更满，拖垮了 sdn。
		
		关闭不需要的基础服务，正式部署的时候要分开部署
- 问题4
	- 现象

		写入 istio 相关的对象无法写入，反馈超时
		
			...
			for: "virtual-service-ratings-test-delay.yaml": Timeout: request did not complete within allowed duration
			...
	- 分析原因

		因为测试环境，所有 openshift 的服务都跑在1台 infra 的 node 上，导致因为响应慢而程序自带的健康检查失效，导致连续崩溃，从而 isito 服务各种异常重启，所以怀疑导致 istio 对象被锁。
	- 解决方案

		重启 k8s api 和控制器
		
			/usr/local/bin/master-restart api && /usr/local/bin/master-restart controllers

- 问题5	
	- 现象

		istio-sidecar-injector 服务异常启动不了，查看日志如下
		
			Error: failed to start patch cert loop mutatingwebhookconfigurations.admissionregistration.k8s.io "istio-sidecar-injector" not found	
		使用命令查看 `istio-sidecar-injector` 对象，发现不存在了
		
			# oc get mutatingwebhookconfiguration istio-sidecar-injector
	- 分析

		查询审计日志，发现在问题4重启后，就存在这个问题了，怀疑是因为某种操作丢失了，这块未来要重点关注。
	- 解决办法
		- 查询源码找到对应的对象文件，[源码文件位置](https://github.com/Maistra/openshift-ansible/blob/maistra-0.9.0/roles/openshift_istio/files/istio-auth.yaml)，注意版本问题。
		- 恢复对象
			- 编辑恢复文件 

					vim istio-sidecar-injector-mutatingwebhook.yaml
					
					apiVersion: admissionregistration.k8s.io/v1beta1
					kind: MutatingWebhookConfiguration
					metadata:
					  name: istio-sidecar-injector
					  namespace: istio-system
					  labels:
					    app: sidecarInjectorWebhook
					    chart: sidecarInjectorWebhook
					    heritage: Tiller
					    maistra-version: MAISTRA_VERSION
					    release: istio-1-1-0-rc-2
					webhooks:
					  - name: sidecar-injector.istio.io
					    clientConfig:
					      service:
					        name: istio-sidecar-injector
					        namespace: istio-system
					        path: "/inject"
					      caBundle: ""
					    rules:
					      - operations: [ "CREATE" ]
					        apiGroups: [""]
					        apiVersions: ["v1"]
					        resources: ["pods"]
					    failurePolicy: Fail
					    namespaceSelector:
					      matchExpressions:
					        - {key: istio.openshift.com/ignore-namespace, operator: DoesNotExist}
			- 执行

					# oc apply -f istio-sidecar-injector-mutatingwebhook.yaml
		- 重启 istio-sidecar-injector 服务，恢复
	- 其他
		
		查看 0.10 版本的时候，发现该对象已经被 configmap 管理起来了。应该是被 operator 接管了。后续再测试
			 							
				
## 参考
[Installing Istio into an existing OCP 3.11 Installation](https://github.com/Maistra/openshift-ansible/blob/maistra-0.10/istio/Installation.md)