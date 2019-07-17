# LXCFS(docker top free 系统资源恢复）
## docker or mesos
### rpm 安装(2.5版本)
	wget https://copr-be.cloud.fedoraproject.org/results/ganto/lxd/epel-7-x86_64/00486278-lxcfs/lxcfs-2.0.5-3.el7.centos.x86_64.rpm
	yum install -y lxcfs-2.0.5-3.el7.centos.x86_64.rpm
- lxcfs 启动

		# lxcfs /var/lib/lxcfs
		mount namespace: 5
		hierarchies:
		  0: fd:   6: pids
		  1: fd:   7: blkio
		  2: fd:   8: cpuset
		  3: fd:   9: cpuacct,cpu
		  4: fd:  10: hugetlb
		  5: fd:  11: net_prio,net_cls
		  6: fd:  12: devices
		  7: fd:  13: memory
		  8: fd:  14: perf_event
		  9: fd:  15: freezer
		 10: fd:  16: name=systemd 

- 测试
	- 测试脚本
		
			mkdir -p /data/pztest && cd /data/pztest
			vi lxcfs_test.sh
						
			docker run --rm -it -m 100m --cpuset-cpus 0 \
			      -v /var/lib/lxcfs/proc/cpuinfo:/proc/cpuinfo:rw \
			      -v /var/lib/lxcfs/proc/diskstats:/proc/diskstats:rw \
			      -v /var/lib/lxcfs/proc/meminfo:/proc/meminfo:rw \
			      -v /var/lib/lxcfs/proc/stat:/proc/stat:rw \
			      -v /var/lib/lxcfs/proc/swaps:/proc/swaps:rw \
			      -v /var/lib/lxcfs/proc/uptime:/proc/uptime:rw \
			      ubuntu
			      #centos
			      #alpine
	- 测试结果
		- ubuntu
			
			top、free 均正常 	
		- centos
			
			top、free 均正常
		- alpine

			free 异常(有待查询)	
							      
### 新版本编译
yum 更新的比较慢，所以可能有 bug 需要编译来做，所以这里准备一个编译版本 
#### 准备工作
- 恢复 yum 远程源(纯线下系统，默认系统可以无视)
	
		cd  /etc/yum.repos.d/
		cp repobak/CentOS-Base.repo .
		yum check-update -y

#### 编译
- 编译安装 lxcfs

		yum install automake libtool fuse* -y 
		mkdir -p /data/pztest && cd /data/pztest
		git clone git://github.com/lxc/lxcfs && cd lxcfs
		./bootstrap.sh
		./configure
		make
		make install

	编译的库文件会被放到 `/usr/local/lib/lxcfs/*.so`

#### 启动测试
- lxcfs 启动

		mkdir -p /var/lib/lxcfs
		./lxcfs --version (最新版本3.0.x)
		
			3.0.0
		./lxcfs /var/lib/lxcfs
- 启动打印

		# ./lxcfs /var/lib/lxcfs
		mount namespace: 5
		hierarchies:
		  0: fd:   6: pids
		  1: fd:   7: blkio
		  2: fd:   8: cpuset
		  3: fd:   9: cpuacct,cpu
		  4: fd:  10: hugetlb
		  5: fd:  11: net_prio,net_cls
		  6: fd:  12: devices
		  7: fd:  13: memory
		  8: fd:  14: perf_event
		  9: fd:  15: freezer
		 10: fd:  16: name=systemd
- 测试
	- 运行测试脚本
	
			sh lxcfs_test.sh
	- 进入容器执行 top 和 free，查看结果
	- 测试结果
		- ubuntu
			
			top、free 均正常 	
		- centos
			
			top、free 均正常
		- alpine

			free 异常(有待查询)	

## 服务设置
- 设置

		# cat /lib/systemd/system/lxcfs.service
		[Unit]
		Description=FUSE filesystem for LXC
		ConditionVirtualization=!container
		Before=lxc.service
		Documentation=man:lxcfs(1)
		
		[Service]
		ExecStart=/usr/bin/lxcfs /var/lib/lxcfs/
		KillMode=process
		Restart=on-failure
		ExecStopPost=-/bin/fusermount -u /var/lib/lxcfs
		Delegate=yes
		
		[Install]
		WantedBy=multi-user.target

- 启动

		# service lxcfs status
		Redirecting to /bin/systemctl status lxcfs.service
		● lxcfs.service - FUSE filesystem for LXC
		   Loaded: loaded (/usr/lib/systemd/system/lxcfs.service; disabled; vendor preset: disabled)
		   Active: active (running) since 四 2019-01-31 10:52:15 CST; 1s ago
		     Docs: man:lxcfs(1)
		 Main PID: 25025 (lxcfs)
		   Memory: 312.0K
		   CGroup: /system.slice/lxcfs.service
		           └─25025 /usr/bin/lxcfs /var/lib/lxcfs/
		
		1月 31 10:52:15 install182.hisun.com lxcfs[25025]: 1: fd:   6: blkio
		1月 31 10:52:15 install182.hisun.com lxcfs[25025]: 2: fd:   7: cpuset
		1月 31 10:52:15 install182.hisun.com lxcfs[25025]: 3: fd:   8: cpuacct,cpu
		1月 31 10:52:15 install182.hisun.com lxcfs[25025]: 4: fd:   9: hugetlb
		1月 31 10:52:15 install182.hisun.com lxcfs[25025]: 5: fd:  10: net_prio,net_cls
		1月 31 10:52:15 install182.hisun.com lxcfs[25025]: 6: fd:  11: devices
		1月 31 10:52:15 install182.hisun.com lxcfs[25025]: 7: fd:  12: memory
		1月 31 10:52:15 install182.hisun.com lxcfs[25025]: 8: fd:  13: perf_event
		1月 31 10:52:15 install182.hisun.com lxcfs[25025]: 9: fd:  14: freezer
		1月 31 10:52:15 install182.hisun.com lxcfs[25025]: 10: fd:  15: name=systemd

## lxcfs on openshift(kubernetes) 
lxcfs 守护进程用于在节点上生成 lxcfs 的系统文件，k8s 部署需要按照上面的步骤安装 lxcfs ，因为 daemonset 只是将宿主机的 lxcfs 挂载到了容器中执行。[ali lxcfs daemonset方式运行](https://blog.51cto.com/bingdian/2356632?source=dra)
### 准备工作
- 有 root 权限
- 按照上面的规约在每个 node 上安装 lxcfs
- 安装 openshift 企业版或社区版
- 设置管理员用户和设置 scc
	- 授权

		如果只装 openshift ，用以下命令到 openshift master 进行授权
		
		- 创建用户密码(htpaaswd)

			登陆 master 主机
			
				htpasswd /etc/origin/master/htpasswd cluster_admin
				oc adm policy add-cluster-role-to-user admin cluster_admin
		- 修改 scc

			授权登陆用户组可以使用任何uid启动服务，默认只能使用非 root uid

				oc adm policy add-scc-to-group anyuid system:authenticated
			授权登陆用户组可以使用超权
			
				oc adm policy add-scc-to-group privileged system:authenticated		 

### 安装
- 获取

	登陆 master 节点
	
		git clone https://github.com/denverdino/lxcfs-initializer
		cd lxcfs-initializer

- lxcfs(daemonset)

		vi lxcfs-daemonset.yaml
		
		apiVersion: apps/v1beta2
		kind: DaemonSet
		metadata:
		  name: lxcfs
		  labels:
		    app: lxcfs
		  namespace: default
		spec:
		  selector:
		    matchLabels:
		      app: lxcfs
		  template:
		    metadata:
		      labels:
		        app: lxcfs
		    spec:
		      hostPID: true
		      containers:
		      - name: lxcfs
		        image: registry.cn-hangzhou.aliyuncs.com/denverdino/lxcfs:2.0.8-1
		        imagePullPolicy: Always
		        securityContext:
		          privileged: true  //<-- 注意这里是特权
		        volumeMounts:
		        - name: cgroup
		          mountPath: /sys/fs/cgroup
		        - name: lxcfs
		          mountPath: /var/lib/lxcfs
		          mountPropagation: Bidirectional //<-- 卷共享新特性 1.11 测试版本、1.12 GA
		        - name: usr-local
		          mountPath: /usr/local
		      volumes:
		      - name: cgroup
		        hostPath:
		          path: /sys/fs/cgroup
		      - name: usr-local
		        hostPath:
		          path: /usr/local
		      - name: lxcfs
		        hostPath:
		          path: /var/lib/lxcfs
		          type: DirectoryOrCreate	
- 执行

		oc project default
		oc apply -f lxcfs-daemonset.yaml
- 查看结果

		# oc get pod | grep lxcfs
		lxcfs-5zbz2                                            1/1       Running   0          35m
		lxcfs-7wxw5                                            1/1       Running   0          35m		

		# oc logs lxcfs-5zbz2
		hierarchies:
		  0: fd:   5: pids
		  1: fd:   6: memory
		  2: fd:   7: hugetlb
		  3: fd:   8: perf_event
		  4: fd:   9: freezer
		  5: fd:  10: cpuacct,cpu
		  6: fd:  11: net_prio,net_cls
		  7: fd:  12: devices
		  8: fd:  13: blkio
		  9: fd:  14: cpuset
		 10: fd:  15: name=systemd
		 
## 注入 LXCFS 方法
lxcfs 服务启动完毕后，剩下需要的就是启动 pod 的时候将 lxcfs 生成目录注入到匹配的 pod 中。有两种方法: `Initializer` 和 `MutatingAdmissionWebhook`
### Initializer(openshift暂时不可用)
网上可以查到的基本上是用 `Initializer`。这里尝试引用了 `Initializer` 扩展机制，可以用于对资源创建进行拦截和注入处理，可以借助它优雅地完成对 lxcfs 文件的自动注入。
#### openshift initializer 启动方法
因为 openshift 关于 3.11 的 initializer 配置方法没有官方说明，所以参考了 kubernetes 相关的做法

initializer 默认没有开启，打开步骤如下.参考 [Enable initializers alpha feature](https://kubernetes.io/docs/reference/access-authn-authz/extensible-admission-controllers/) ,注意，因为kubernetes 版本升级，所以对于 1.9 生效的打开方法对 1.10 以后版本无效。(openshift 可查询方法是1.9)

- 打开 `--enable-admission-plugins` 中的 `Initializers` 标示,注意如果是多 master 需要全部设置。 
- 通过配置 `apiServerArguments` 的 `--runtime-config=admissionregistration.k8s.io/v1alpha1` 启动动态准入控制器注册 api.  

#### 准备工作
为了能够让 lxcfs 控制器自动注入所需数据，首先需要修改每个 master 上的配置，以包括对 initializer 的支持和证书签名请求（CSR）的签名，用来打开 `initializer` 功能

1. 创建 master-config patch 文件

		cd /etc/origin/master && vim master-config.patch 
		
		admissionConfig:
		  pluginConfig: # 设置 pluginconfig
		    Initializers:
		      configuration:
		        kind: DefaultAdmissionConfig
		        apiVersion: v1
		        disable: false
		kubernetesMasterConfig:
		  apiServerArguments:
		    runtime-config: # 设置api runtime
		    - admissionregistration.k8s.io/v1alpha1=true
- 创建原配置文件

		cp master-config.yaml master-config.yaml.prepatch
- 生成新配置

		oc ex config patch master-config.yaml.prepatch -p "$(cat master-config.patch)" > master-config.yaml		    
- 重启 api & controller 加载配置

		/usr/local/bin/master-restart api && /usr/local/bin/master-restart controllers
- 使用 oc 命令 api 服务是否恢复

		oc get no
- 检查 master api 日志，确认服务打开

		master-log api api | less
		
		...
		I0319 04:02:31.547848       1 feature_gate.go:194] feature gates: map[AdvancedAuditing:true Initializers:true]
		I0319 04:02:31.547873       1 initialization.go:90] enabled Initializers feature as part of admission plugin setup
		...
- 检查 api 接口

		https://openshift-master-name:8443/apis/admissionregistration.k8s.io/v1alpha1

#### 遇到问题
但是因为 [openshift bug](https://github.com/openshift/origin/issues/21789) 无法正常调用 kubernetes initailizer 服务，所以后面选择了 `MutatingAdmissionWebhook` 自己实现。
### MutatingAdmissionWebhook	
#### 准备工作
为了能够让 lxcfs 控制器自动注入所需数据，首先需要修改每个 master 上的 master 配置，以包括对 webhook 的支持和证书签名请求（CSR）的签名，用来打开 `MutatingAdmissionWebhook` 功能

1. 创建 master-config patch 文件

		cd /etc/origin/master && vim master-config.patch 
		
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
- 创建原配置文件

		cp master-config.yaml master-config.yaml.prepatch
- 生成新配置

		oc ex config patch master-config.yaml.prepatch -p "$(cat master-config.patch)" > master-config.yaml		    
- 重启 api & controller 加载配置

		/usr/local/bin/master-restart api && /usr/local/bin/master-restart controllers
- 使用 oc 命令 api 服务是否恢复

		oc get no
- 检查 master api 日志，确认服务打开

		master-log api api | less
		
		...
		I0322 06:16:20.151321       1 plugins.go:158] Loaded 1 mutating admission controller(s) successfully in the following order: MutatingAdmissionWebhook.
		I0322 06:16:26.446992       1 store.go:1397] Monitoring mutatingwebhookconfigurations.admissionregistration.k8s.io count at <storage-prefix>//mutatingwebhookconfigurations
		...
- 检查 api 接口

		https://openshift-master-name:8443/apis/admissionregistration.k8s.io/v1beta1

### 安装 webhook 控制器








## openshift 3.11 打开 initializers 功能 bug
[openshift initializers bug](https://github.com/openshift/origin/issues/22394)

[openshift other bug](https://github.com/openshift/origin/issues/21789)

## 错误处理
- 错误1
	- 错误原因 

		运行 `lxcfs-daemonset.yaml` 报错 `lxcfs nsenter: failed to execute lxcfs: No such file or directory`
	- 错误分析

		没有在宿主机安装 lxcfs
	- 解决方法

		安装 lxcfs
- 错误2
	- 错误原因

		运行 `lxcfs-daemonset.yaml` 报错
		
			fuse: mountpoint is not empty
			fuse: if you are sure this is safe, use the 'nonempty' mount option 
	- 错误分析

		在没有启动 lxcfs 请款下，启动了 docker 挂载的目录，然后自动会在 `var/lib/lxcfs/` 目录下创建挂载目录。
	- 解决办法

		删除 `var/lib/lxcfs/` 下的所有数据	

## 参考
[docker容器中使用top、free命令查看容器真实cpu和内存使用情况的](http://www.qingpingshan.com/rjbc/yunjisuan/343272.html)

[lxcfs-github](https://github.com/lxc/lxcfs)

[利用LXCFS增强docker容器隔离性和资源可见性](https://yq.aliyun.com/articles/566208)

[lxcfs-initializer-github](https://github.com/denverdino/lxcfs-initializer)

[基于LXCFS增强docker容器隔离性的分析](https://tech.meili-inc.com/docker-with-lxcfs-130)

[LXDDownloads](https://linuxcontainers.org/lxd/downloads/)

[ali lxcfs daemonset方式运行](https://blog.51cto.com/bingdian/2356632?source=dra)

[openshift一般准入规则](https://github.com/pangzheng/BOOK/blob/master/%E6%8A%80%E6%9C%AF/openshift/3.11%E5%AE%98%E6%96%B9%E6%96%87%E6%A1%A3/%E7%BB%93%E6%9E%84/%E9%99%84%E5%8A%A0%E6%A6%82%E5%BF%B5/%E5%87%86%E5%85%A5%E6%8E%A7%E5%88%B6%E5%99%A8.md)


