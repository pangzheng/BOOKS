# minishift 安装(mac).md
## 准备工作
## 安装 VirtualBox
虽然有其他的虚拟机放哪，但这里选择 VirtualBox，现在比较通用，必须手动安装 VirtualBox 才能使用嵌入式 VirtualBox 驱动程序。需要 VirtualBox 版本5.1.12或更高版本。确保在使用嵌入式驱动程序之前[下载并安装 VirtualBox](https://www.virtualbox.org/wiki/Downloads)。
## 本机安装 docker 客户端
[安装参考](https://docs.docker.com/docker-for-mac/install/)
## 安装最新版本 brew
brew 是类似于 centos yum 工具一样的mac包管理器。详细参考[Homebrew](https://brew.sh/index_zh-cn)

- 安装

		$ /usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
- 升级

	brew update

## 安装 minishift
- 安装 minishift 二进制

		$ brew cask install minishift
- 启动 minishift

		$ minishift start --profile minishift --registry-mirror https://reg-mirror.qiniu.com --vm-driver virtualbox
	- 注意
		- 下载三个部分的数据包
			- oc 二进制数据包
			- 虚拟机 iso
			- minishift 启动所需的镜像，这里启动带的参数就是加速容器镜像下载
		- 虚拟机参数

			`--vm-driver virtualbox	` 是 minishift 选用 virtuabox 必用参数。
- 安装完毕后可以通过命令查询参数

		$ minishift config view
		- cpus                               : 4
		- memory                             : 8192
		- registry-mirror                    : [https://reg-mirror.qiniu.com]
		- vm-driver                          : virtualbox
- 使用 docker 客户端使用 minishift 内部 docker
	- 查看需要添加的环境变量
		
			$ minishift docker-env
			export DOCKER_TLS_VERIFY="1"
			export DOCKER_HOST="tcp://192.168.99.105:2376"
			export DOCKER_CERT_PATH="/Users/Prometheus/.minishift/certs"
			# Run this command to configure your shell:
			# eval $(minishift docker-env)
	- 添加环境变量到文件

			vi ~/.bash_profile
			
			...
				export DOCKER_TLS_VERIFY="1"
				export DOCKER_HOST="tcp://192.168.99.105:2376"
				export DOCKER_CERT_PATH="/Users/Prometheus/.minishift/certs"
			....		
	- 执行环境变量

			source .bash_profile
	- 测试

			docker info
- 使用 oc 
	- 查看需要添加的环境变量

			$ minishift oc-env
			export PATH="/Users/Prometheus/.minishift/cache/oc/v3.11.0/darwin:$PATH"
			# Run this command to configure your shell:
			# eval $(minishift oc-env)
	- 添加环境变量到文件

			vi ~/.bash_profile
			
			...
				export PATH="/Users/Prometheus/.minishift/cache/oc/v3.11.0/darwin:$PATH"
			....		
	- 执行环境变量

			source .bash_profile
	- 测试

			oc version 		
							
- 安装检测

		$ minishift status
		Minishift:  Running
		Profile:    minishift
		OpenShift:  Running (openshift v3.11.0+d0c29df-98)
		DiskUsage:  14% of 19G (Mounted On: /mnt/sda1)
		CacheUsage: 3.588 GB (used by oc binary, ISO or cached images)

## 高级 
### 缓存介绍 
上面介绍了一共有3部分需要下载的数据包比较大，如果网路不好，可能会导致下载很慢，所以可以用下载器下载后，放到 minishift 的缓存目录里，启动的时候用来加速使用

- 找到缓存目录

	安装 minishift 后，在执行用户的目录下，可以找到 `.minishift` 目录,这个就是 minishift 安装路径，缓存目录就在类似，需要将下载后的数据包存放在对应的目录中
	
		ll /Users/$user/.minishift/cache
	下面有3个目录
		
	- image

		存放缓存镜像使用,镜像因为使用了国内的 --registry-mirror ，并且它存储的格式不是按文件而是按照序列化的方式存储的，所以不能直接cp文件，而必须是下载完了用命令行备份到 cache 中
		
		- 格式

				～/xxx/.minishift/cache/images/
	- oc

		存放客户端包,有包后可以自动加载
		
		- 格式 
		
				～/xxx/.minishift/cache/oc/v3.11.0/darwin/oc
	- iso 

		存放虚拟启动 iso,文件放入后，有包后可以自动加载
		
		- 格式

				～/xxx/.minishift/cache/iso/centos/v1.15.0/minishift-centos7.iso
### 镜像缓存加载方法
因为镜像缓存比较特殊，在特殊情况下需要更新，所以镜像缓存默认是不开放的，这里在使用 `minishift start` 的时候会看到

				...
				-- Checking available disk space ... 1% used OK
				-- Writing current configuration for static assignment of IP address ... OK
				   Importing 'openshift/origin-control-plane:v3.11.0' . CACHE MISS
				   Importing 'openshift/origin-docker-registry:v3.11.0' . CACHE MISS
				   Importing 'openshift/origin-haproxy-router:v3.11.0' . CACHE MISS
				-- OpenShift cluster will be configured with ...
				   Version: v3.11.0
				-- Pulling the OpenShift Container Image
				... 			
当然通过上面设置启动参数 `--registry-mirror https://reg-mirror.qiniu.com` 可以进行镜像下载加速，但是如果在 pull 过程中，或者 minishift 启动中出现问题，导致需要删除已经下载好镜像的 vm 进行重制，那么下面的本方法就比较有用。
#### 镜像缓存加载步骤
- 创建缓存

	大部分因网络原因，minishift 将会启动失败，可以使用 `minishift stop && minishift start` 尝试重新启动，当然结果基本上都会是失败，这里要保证的是失败安装确保缓存都已经创建成功，其中包含 vm 的 iso 、oc 二进制，以及镜像。
- 获取镜像缓存

	在 minishift 虚拟主机启动且镜像都获取到后，执行以下命令
	
	- 查看本地镜像目录，该命令应得到结果应该是空

			minishift image list
	- 查看 vm 	docker 镜像目录，应该可以看到 minishift 已经拉好的镜像

			minishift image list --vm
	- 到出镜像到本地，并查看

			minishift image export all	 && minishift image list
				
				docker.io/openshift/origin-cli:v3.11.0
				docker.io/openshift/origin-control-plane:latest
				docker.io/openshift/origin-control-plane:v3.11.0
				docker.io/openshift/origin-deployer:v3.11.0
				docker.io/openshift/origin-docker-registry:v3.11.0
				docker.io/openshift/origin-haproxy-router:v3.11.0
				docker.io/openshift/origin-hyperkube:v3.11.0
				docker.io/openshift/origin-hypershift:v3.11.0
				docker.io/openshift/origin-node:v3.11.0
				docker.io/openshift/origin-pod:v3.11.0
				docker.io/openshift/origin-service-serving-cert-signer:v3.11
				docker.io/openshift/origin-web-console:v3.11.0
				openshift/origin-control-plane:v3.11.0
				openshift/origin-docker-registry:v3.11.0
				openshift/origin-haproxy-router:v3.11.0
	- 倒入这些镜像名到缓存 list

			minishift image cache-config add \
				docker.io/openshift/origin-cli:v3.11.0 \
				docker.io/openshift/origin-control-plane:latest \
				docker.io/openshift/origin-control-plane:v3.11.0 \
				docker.io/openshift/origin-deployer:v3.11.0 \
				docker.io/openshift/origin-docker-registry:v3.11.0 \
				docker.io/openshift/origin-haproxy-router:v3.11.0 docker.io/openshift/origin-hyperkube:v3.11.0 \
				docker.io/openshift/origin-hypershift:v3.11.0 \
				docker.io/openshift/origin-node:v3.11.0 \
				docker.io/openshift/origin-pod:v3.11.0 \
				docker.io/openshift/origin-service-serving-cert-signer:v3.11 docker.io/openshift/origin-web-console:v3.11.0
	- 查看 cache list

			minishift image cache-config view
	- 插入启动缓存配置

			minishift config set image-caching true
	- 删除现有 vm 主机和相关目录

			minishift stop && minishift delete -f && rm -Rf ~/.minishift/machines/* && rm -Rf ~/.minishift/logs/* && rm -Rf rm -Rf ~/.minishift/tmp/*
	- 启动服务

			minishift start --profile minishift --registry-mirror https://reg-mirror.qiniu.com --vm-driver virtualbox
	- 验证配置

		再打开一个终端，`minishift config view`
		
			$ minishift config view
			- cpus                               : 4
			- image-caching                      : true #< 这里
			- memory                             : 8192
			- registry-mirror                    : [https://reg-mirror.qiniu.com]
			- vm-driver                          : virtualbox
	- 看到的启动应该是跳过镜像加载直接进入

			....
					-- Checking HTTP connectivity from the VM ...
					   Retrieving http://minishift.io/index.html ... OK
					-- Checking if persistent storage volume is mounted ... OK
					-- Checking available disk space ... 1% used OK
					-- Writing current configuration for static assignment of IP address ... OK
					   Importing 'docker.io/openshift/origin-cli:v3.11.0' ..... OK
					   Importing 'docker.io/openshift/origin-control-plane:latest' . CACHE MISS
					   Importing 'docker.io/openshift/origin-control-plane:v3.11.0' ......... OK
					   Importing 'docker.io/openshift/origin-deployer:v3.11.0' ... OK
					   Importing 'docker.io/openshift/origin-docker-registry:v3.11.0' ... OK
					   Importing 'docker.io/openshift/origin-haproxy-router:v3.11.0' ... OK
					   Importing 'docker.io/openshift/origin-hyperkube:v3.11.0' ..... OK
					   Importing 'docker.io/openshift/origin-hypershift:v3.11.0' ..... OK
					   Importing 'docker.io/openshift/origin-node:v3.11.0' ......... OK
					   Importing 'docker.io/openshift/origin-pod:v3.11.0' ... OK
					   Importing 'docker.io/openshift/origin-service-serving-cert-signer:v3.11' .... OK
					   Importing 'docker.io/openshift/origin-web-console:v3.11.0' .... OK
					   Importing 'openshift/origin-control-plane:v3.11.0' . CACHE MISS
					   Importing 'openshift/origin-docker-registry:v3.11.0' . CACHE MISS
					   Importing 'openshift/origin-haproxy-router:v3.11.0' . CACHE MISS
					-- OpenShift cluster will be configured with ...
					   Version: v3.11.0
					-- Copying oc binary from the OpenShift container image to VM ... OK
					-- Starting OpenShift cluster ...........................
			....

## 问题处理
- 问题1
	- 问题 

			-- OpenShift cluster will be configured with ...
			   Version: v3.11.0
			-- Copying oc binary from the OpenShift container image to VM ... OK
			-- Starting OpenShift cluster .................................................................Error during 'cluster up' execution: Error starting the cluster. ssh command error:
			command : /var/lib/minishift/bin/oc cluster up --public-hostname 192.168.99.106 --routing-suffix 192.168.99.106.nip.io --base-dir /var/lib/minishift/base --image 'openshift/origin-${component}:v3.11.0'
			err     : exit status 1
			output  : Getting a Docker client ...
			Checking if image openshift/origin-control-plane:v3.11.0 is available ...
			Checking type of volume mount ...
			Determining server IP ...
			Using public hostname IP 192.168.99.106 as the host IP
			Checking if OpenShift is already running ...
			Checking for supported Docker version (=>1.22) ...
			Checking if insecured registry is configured properly in Docker ...
			Checking if required ports are available ...
			Checking if OpenShift client is configured properly ...
			Checking if image openshift/origin-control-plane:v3.11.0 is available ...
			Starting OpenShift using openshift/origin-control-plane:v3.11.0 ...
			I0305 06:55:47.895394    5370 config.go:40] Running "create-master-config"
			I0305 06:55:50.617588    5370 config.go:46] Running "create-node-config"
			I0305 06:55:51.369654    5370 flags.go:30] Running "create-kubelet-flags"
			I0305 06:55:51.789125    5370 run_kubelet.go:49] Running "start-kubelet"
			I0305 06:55:52.003726    5370 run_self_hosted.go:181] Waiting for the kube-apiserver to be ready ...
			E0305 07:00:52.006632    5370 run_self_hosted.go:571] API server error: Get https://192.168.99.106:8443/healthz?timeout=32s: dial tcp 192.168.99.106:8443: connect: connection refused ()
			Error: timed out waiting for the condition
	- 分析

		这里有的时候正常有的时候不正常，这里可能和主机的性能或者网络有关，所以重复 `cluster up` 应该可以解决
	- 解决步骤
		- 进入 minishift vm

				minishift ssh
		- 关闭 cluster

				/var/lib/minishift/bin/oc cluster down
		- 启动 cluster

			复制上面 command 后面的启动命令行,如本次用例
			
				/var/lib/minishift/bin/oc cluster up --public-hostname 192.168.99.106 --routing-suffix 192.168.99.106.nip.io --base-dir /var/lib/minishift/base --image 'openshift/origin-${component}:v3.11.0'
			
								
[安装 bug](https://github.com/minishift/minishift/issues/3104)