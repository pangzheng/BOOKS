# 使用 ceph Rados 块设备(RBD)
## 概述
使用 ceph rbd 可以为 openshift 集群配置[持久化存储](https://docs.openshift.org/3.9/architecture/additional_concepts/storage.html#architecture-additional-concepts-storage).

持久化卷(pv) 和持久化卷申请(pvc) 可以跨单个项目共享卷。虽然 pv 定义包含 ceph rbd 也可以直接在 pod 中定义。但这样做不会将卷创建为独特的集群资源，从而容易产生冲突。

	注意:基础设施中的高可用存储由低成的存储提供商提供。

## 供应
要配置 ceph 卷，需要以下内容

- 基础架构中的现有存储设备
- 将 oepnshift secret 对象中使用 ceph 秘钥
- ceph 镜像名
- 块存储顶部的文件系统类型(如，ext4)
- `ceph-common` 安装在集群中每个可调度计算节点安装 `ceph-commo`

- 准备工作
	- 安装

			＃yum install ceph-common 
	- 获取授权密钥，并使用 base64 转换供 openshit 使用
		
			在 ceph mon 节点上运行 ceph auth get-key 显示 client.admin 键值
			
			注意:必须在 pvc 和 pv 相同的项目中创建 secret，不简单的在 default 项目下面。
	- 创建 secret 文件

			vi ceph-secret.yaml
			
			apiVersion: v1
			kind: Secret
			metadata:
			  name: ceph-secret
			data:
			key: QVFBOFF2SlZheUJQRVJBQWgvS2cwT1laQUhPQno3akZwekxxdGc9PQ==
	- 创建

			$ oc create -f ceph-secret.yaml
			# oc get secret ceph-secret
			NAME          TYPE      DATA      AGE
			ceph-secret   Opaque    1         23d

- 创建持久化卷
	- 说明

		ceph rbd 不支持 "Recycle" 回收策略

		开发人员通过直接在 pod 引入 pvc 或者集群卷来请求 ceph rbd。 pvc 仅存在于用户的项目中，并且只能由一个项目内的容器引用。任何想从其他项目访问 pv 都会失败。
	- 在 openshift 定义 pv
			
			vi ceph-pv.yaml
			
			apiVersion: v1
			kind: PersistentVolume
			metadata:
			  name: ceph-pv #引用的 pv 的名称
			spec:
			  capacity:
			    storage: 2Gi #分配的大小
			  accessModes:
			    - ReadWriteOnce #作为pvc 和pv 批评的标签，所有块存储都定义成单用户，非共享存储使用
			  rbd: # 使用的插件为 rbd
			    monitors: #ceph 监听 ip 和端口
			      - 192.168.122.133:6789
			    pool: rbd
			    image: ceph-image
			    user: admin
			    secretRef:
			      name: ceph-secret #用于 openshift 同 ceph 的安全链接
			    fsType: ext4 # 安装在 rbd 设备上的文件系统类型，如果对卷进行格式化或配置参数，可能会丢失数据
			    readOnly: false
			  persistentVolumeReclaimPolicy: Retain
	-  创建卷

			＃oc create -f ceph-pv.yaml
	- 验证

			# oc get pv
			NAME                     LABELS    CAPACITY     ACCESSMODES   STATUS      CLAIM     REASON    AGE
			ceph-pv                  <none>    2147483648   RWO           Available                       2s			
	- 创建一个 pvc 绑定对象
		
			vi ceph-claim.yaml
			
			kind: PersistentVolumeClaim
			apiVersion: v1
			metadata:
			  name: ceph-claim
			spec:
			  accessModes: #访问模式作为标签匹配
			    - ReadWriteOnce 
			  resources:
			    requests:
			      storage: 2Gi #存储申请需求 pv 提供 2gi 或更大空间
	- 创建pvc 定义

			＃oc create -f ceph-claim.yaml	

## ceph 卷安全
- 注意

	实施 ceph rbd 卷之前，需要完整阅读[卷安全](https://docs.openshift.org/3.9/install_config/persistent_storage/pod_security_context.html#install-config-persistent-storage-pod-security-context)

	即使在 pod 规范中为定义用户和组 id,所生成的 pod 可能会根据其匹配的 scc 或其项目为这些 id 定义默认值。请参考完整的[卷安全](https://docs.openshift.org/3.9/install_config/persistent_storage/pod_security_context.html#install-config-persistent-storage-pod-security-context)，其中详细介绍了 scc 的存储方面和默认值。
	
将 pod 定义或容器镜像中定义的用户和组 id 应用于目标物理存储，共享卷(NFS/GlusterFS) 与块卷(Ceph RBD,iscsi等)之间有明显的差异。这被称为管理块设备的所有权。例如，如果 ceph rbd 所有者设置为 `123`，并将其组织标识设置为 `567`，并该 pod 将其 `runAsUser` 设置为 222 并将其 `fsGroup` 设置为 `7777`,则 ceph rbd 物理安装所有权需要更改为 222:7777

pod 使用 `fsGroup` pod `securityContext` 定义下的节来定义 Ceph rbd 卷的所有权。

	spec:
	  containers:
	    - name:
	    ...
	  securityContext: # 必须在pod 设置，而不是容器
	    fsGroup: 7777 #该容器具有相同的 fsGroup ID

			      