# cloud-provider-alibaba-cloud
## 先觉条件
- kubernetes 版本大于 1.7.2
- CloudNetwork 仅支持阿里 vpc 网络

## 在阿里云里部署 out-of-tree cloud-provider
- 安装 kubernetes,因为是 openshift 忽略
- 安装容器运行时，默认使用 docker 
- 设置 provider id ，这块参考[这里](https://github.com/kubernetes/cloud-provider-alibaba-cloud/blob/master/docs/examples/kubelet.service)，这里需要设置好后重启 node 节点服务
	- 参考 `kubelet.service` 
			
			...
						Environment="KUBELET_CLOUD_PROVIDER_ARGS=--cloud-provider=external --hostname-override=${REGION_ID}.${INSTANCE_ID} --provider-id=${REGION_ID}.${INSTANCE_ID}" #<- 注意这里
			...
						ExecStart=
						ExecStart=/usr/bin/kubelet $KUBELET_CLOUD_PROVIDER_ARGS $KUBELET_KUBECONFIG_ARGS $KUBELET_SYSTEM_PODS_ARGS $KUBELET_NETWORK_ARGS $KUBELET_DNS_ARGS $KUBELET_AUTHZ_ARGS $KUBELET_CADVISOR_ARGS $KUBELET_CGROUP_ARGS $KUBELET_CERTIFICATE_ARGS $KUBELET_EXTRA_ARGS #<- 注意这里
			...

- 注意
	- 不确定如何查找ECS实例的ID和区域ID，请尝试在ECS实例中运行以下命令

			$ META_EP=http://100.100.100.200/latest/meta-data
			$ echo `curl -s $META_EP/region-id`.`curl -s $META_EP/instance-id`  


			echo `curl -s http://100.100.100.200/latest/meta-data/region-id`.`curl -s http://100.100.100.200/latest/meta-data/instance-id`
	- 设置完毕后需要重启 node
	- 通过 `oc get no` 检查
	
## 安装 cloud-provider
cloud-provider 执行需要阿里云的访问权限，需要生成 `AccessKeyID＆Secret`.
### RAM 角色策略
参考[什么是实例的 RAM 角色](https://www.alibabacloud.com/help/doc-detail/54235.htm)	

样本 [master policy](https://github.com/kubernetes/cloud-provider-alibaba-cloud/blob/master/docs/examples/master.policy) 开放权限较大，可以按照实际情况进行缩减。
### AccessKeyID & AccessKeySecret
也可以使用 AccessKeyID＆Secret 来授权CloudProvider。请确保 AccessKeyID 在[master.policy](https://github.com/kubernetes/cloud-provider-alibaba-cloud/blob/master/docs/examples/master.policy)中具有列出的权限

[如何获得AccessKey](https://usercenter.console.aliyun.com/#/manage/ak)？

然后，`cloud-config` 在群集中创建 configmap。

	$ kubectl -n kube-system create configmap cloud-config \
        --from-literal=special.keyid="$ACCESS_KEY_ID" \
        --from-literal=special.keysecret="$ACCESS_KEY_SECRET"

### system:cloud-controller-manager 的 ServiceAccount 

CloudProvider 使用 `system:cloud-controller-manager` 服务帐户来授权启用 RBAC 的 Kubernetes集群。

1. 必须创建某些 RBAC 角色和绑定。有关详细信息，请参阅 [cloud-controller-manager.yml](https://github.com/kubernetes/cloud-provider-alibaba-cloud/blob/master/docs/examples/cloud-controller-manager.yml)。
- kubeconfig 文件必须存在的。将文件保存到 `/etc/kubernetes/cloud-controller-manager.conf`。 并替换 `$CA_DATA` 为命令的输出`cat /etc/kubernetes/pki/ca.crt|base64 -w 0`。并使用自己的 apiserver 替换服务器。

		kind: Config
		contexts:
		- context:
		    cluster: kubernetes
		    user: system:cloud-controller-manager
		  name: system:cloud-controller-manager@kubernetes
		current-context: system:cloud-controller-manager@kubernetes
		users:
		- name: system:cloud-controller-manager
		  user:
		    tokenFile: /var/run/secrets/kubernetes.io/serviceaccount/token
		apiVersion: v1
		clusters:
		- cluster:
		    certificate-authority-data: $CA_DATA
		    server: https://192.168.1.76:6443
		  name: kubernetes

### 发布 cloud-provider 的ds
正在 [cloud-controller-manager.yml](https://github.com/kubernetes/cloud-provider-alibaba-cloud/blob/master/docs/examples/cloud-controller-manager.yml) 中准备一个可用的cloudprovider 的发布 yaml 文件。唯一需要做的就是用自己的真实集群 cidr 替换 `${CLUSTER_CIDR}`。然后  `kubectl apply -f examples/cloud-controller-manager.yml` 完成安装。

例子

	kind: ClusterRole
	apiVersion: rbac.authorization.k8s.io/v1beta1
	metadata:
	  name: system:cloud-controller-manager
	rules:
	  - apiGroups:
	      - ""
	    resources:
	      - persistentvolumes
	      - services
	      - secrets
	      - endpoints
	      - serviceaccounts
	    verbs:
	      - get
	      - list
	      - watch
	      - create
	      - update
	      - patch
	  - apiGroups:
	      - ""
	    resources:
	      - nodes
	    verbs:
	      - get
	      - list
	      - watch
	      - delete
	      - patch
	      - update
	  - apiGroups:
	      - ""
	    resources:
	      - services/status
	    verbs:
	      - update
	  - apiGroups:
	      - ""
	    resources:
	      - nodes/status
	    verbs:
	      - patch
	      - update
	  - apiGroups:
	      - ""
	    resources:
	      - events
	      - endpoints
	    verbs:
	      - create
	      - patch
	      - update
	---
	kind: ClusterRoleBinding
	apiVersion: rbac.authorization.k8s.io/v1beta1
	metadata:
	  name: system:cloud-controller-manager
	roleRef:
	  apiGroup: rbac.authorization.k8s.io
	  kind: ClusterRole
	  name: system:cloud-controller-manager
	subjects:
	- kind: ServiceAccount
	  name: cloud-controller-manager
	  namespace: kube-system
	---
	kind: ClusterRoleBinding
	apiVersion: rbac.authorization.k8s.io/v1beta1
	metadata:
	  name: system:shared-informers
	roleRef:
	  apiGroup: rbac.authorization.k8s.io
	  kind: ClusterRole
	  name: system:cloud-controller-manager
	subjects:
	- kind: ServiceAccount
	  name: shared-informers
	  namespace: kube-system
	---
	kind: ClusterRoleBinding
	apiVersion: rbac.authorization.k8s.io/v1beta1
	metadata:
	  name: system:cloud-node-controller
	roleRef:
	  apiGroup: rbac.authorization.k8s.io
	  kind: ClusterRole
	  name: system:cloud-controller-manager
	subjects:
	- kind: ServiceAccount
	  name: cloud-node-controller
	  namespace: kube-system
	---
	kind: ClusterRoleBinding
	apiVersion: rbac.authorization.k8s.io/v1beta1
	metadata:
	  name: system:pvl-controller
	roleRef:
	  apiGroup: rbac.authorization.k8s.io
	  kind: ClusterRole
	  name: system:cloud-controller-manager
	subjects:
	- kind: ServiceAccount
	  name: pvl-controller
	  namespace: kube-system
	---
	kind: ClusterRoleBinding
	apiVersion: rbac.authorization.k8s.io/v1beta1
	metadata:
	  name: system:route-controller
	roleRef:
	  apiGroup: rbac.authorization.k8s.io
	  kind: ClusterRole
	  name: system:cloud-controller-manager
	subjects:
	- kind: ServiceAccount
	  name: route-controller
	  namespace: kube-system
	---
	apiVersion: v1
	kind: ServiceAccount
	metadata:
	  name: cloud-controller-manager
	  namespace: kube-system
	---
	apiVersion: extensions/v1beta1
	kind: DaemonSet
	metadata:
	  labels:
	    app: cloud-controller-manager
	    tier: control-plane
	  name: cloud-controller-manager
	  namespace: kube-system
	spec:
	  selector:
	    matchLabels:
	      app: cloud-controller-manager
	      tier: control-plane
	  template:
	    metadata:
	      labels:
	        app: cloud-controller-manager
	        tier: control-plane
	      annotations:
	        scheduler.alpha.kubernetes.io/critical-pod: ''
	    spec:
	      serviceAccountName: cloud-controller-manager
	      tolerations:
	      - effect: NoSchedule
	        operator: Exists
	        key: node-role.kubernetes.io/master
	      - effect: NoSchedule
	        operator: Exists
	        key: node.cloudprovider.kubernetes.io/uninitialized
	      nodeSelector:
	         node-role.kubernetes.io/master: ""
	      containers:
	      - command:
	        -  /cloud-controller-manager
	        - --kubeconfig=/etc/kubernetes/cloud-controller-manager.conf
	        - --address=127.0.0.1
	        - --allow-untagged-cloud=true
	        - --leader-elect=true
	        - --cloud-provider=alicloud
	        - --allocate-node-cidrs=true
	        - --cluster-cidr=${CLUSTER_CIDR}
	        - --use-service-account-credentials=true
	        - --route-reconciliation-period=30s
	        - --v=5
	        image: registry.cn-hangzhou.aliyuncs.com/acs/cloud-controller-manager-amd64:v1.9.3.10-gfb99107-aliyun
	        env:
	        - name: ACCESS_KEY_ID
	          valueFrom:
	            configMapKeyRef:
	              name: cloud-config
	              key: special.keyid
	        - name: ACCESS_KEY_SECRET
	          valueFrom:
	            configMapKeyRef:
	              name: cloud-config
	              key: special.keysecret
	        livenessProbe:
	          failureThreshold: 8
	          httpGet:
	            host: 127.0.0.1
	            path: /healthz
	            port: 10252
	            scheme: HTTP
	          initialDelaySeconds: 15
	          timeoutSeconds: 15
	        name: cloud-controller-manager
	        resources:
	          requests:
	            cpu: 200m
	        volumeMounts:
	        - mountPath: /etc/kubernetes/
	          name: k8s
	          readOnly: true
	        - mountPath: /etc/ssl/certs
	          name: certs
	        - mountPath: /etc/pki
	          name: pki
	      hostNetwork: true
	      volumes:
	      - hostPath:
	          path: /etc/kubernetes
	        name: k8s
	      - hostPath:
	          path: /etc/ssl/certs
	        name: certs
	      - hostPath:
	          path: /etc/pki
	        name: pki

注意：

如果使用 RAM 角色策略，请从 `cloud-controller-manager.yml` 中删除 env ACCESS_KEY_ID 和 ACCESS_KEY_SECRET 。
## 简单示例
一旦 `cloud-controller-manager` 启动并运行，可以尝试使用 nginx 部署测试

	$ cat <<EOF >nginx.yaml
	apiVersion: extensions/v1beta1
	kind: Deployment
	metadata:
	  name: nginx-example
	spec:
	  replicas: 1
	  revisionHistoryLimit: 2
	  template:
	    metadata:
	      labels:
	        app: nginx-example
	    spec:
	      containers:
	      - image: nginx:latest
	        name: nginx
	        ports:
	          - containerPort: 80
	EOF
	
	$ kubectl create -f nginx.yaml
	
然后使用类型：LoadBalancer创建服务

	$ kubectl expose deployment nginx-example --name=nginx-example --type=LoadBalancer --port=80
	$ kubectl get svc
	NAME            CLUSTER-IP        EXTERNAL-IP     PORT(S)        AGE
	nginx-example   192.168.250.19    106.xx.xx.xxx   80:31205/TCP   5s	
## 参考地址
- [官方](https://github.com/kubernetes/cloud-provider-alibaba-cloud)
- [cloud-provider-alibaba-cloud/docs/examples/](https://github.com/kubernetes/cloud-provider-alibaba-cloud/tree/master/docs/examples)
- [Kubernetes Cloud Controller Manager](https://kubernetes.io/docs/tasks/administer-cluster/running-cloud-controller/)
## 启动参数
- --address="127.0.0.1"

		要服务的 ip 地址，对所有接口设置为 0.0.0.0
- --allocate-node-cidrs="true"

		是否应在 cloud provider 上分配和设置 Pod 的 CIDR。
- --allow-untagged-cloud="true"

		允许群集在云实例上没有 cluster-id 的情况下运行。 此标志已弃用，将在以后的版本中删除。云实例必须需要 cluster-id。
- --alsologtostderr="false"
- --cloud-config=""

		cloud provider 配置文件的路径。配置文件不存在将为空字符串。
- --cloud-provider="alicloud"

		cloud provider 的名称，不能为空
- --cluster-cidr="172.30.0.0/16"

		集群中Pod的CIDR范围。
- --cluster-name="kubernetes"

		集群的实例前缀
- --concurrent-service-syncs="1"

		允许同时同步的服务数。更大的数字等于更快的服务管理响应，但意味着更多的CPU（和网络）负载
- --configure-cloud-routes="true"

		是否应在 cloud provider 上配置 allocate-node-cidrs 分配的 CIDR。
- --contention-profiling="false"

		如果启用了性能分析，则启用锁争用性能分析。
- --controller-start-interval="0s"

		启动控制器管理器之间的间隔
- --feature-gates=""
- --kube-api-burst="30"

		与 kubernetes api 交互时使用
- --kube-api-content-type="application/vnd.kubernetes.protobuf"

		与 api 服务交互请求的类型
- --kube-api-qps="20"

		与 api 服务交互时的 qps
- --kubeconfig="/etc/kubernetes/cloud-controller-manager.conf"

		具有授权和 master 位置信息的 kubeconfig 文件的路径。
- --leader-elect="true"
- --leader-elect-lease-duration="15s"
- --leader-elect-renew-deadline="10s"
- --leader-elect-resource-lock="endpoints"
- --leader-elect-retry-period="2s"

- --log-backtrace-at=":0"
- --log-dir=""
- --log-flush-frequency="5s"
- --logtostderr="true"
- --master=""

		Kubernetes API 服务器的地址（可以覆盖 kubeconfig 中的任何值）。
- --min-resync-period="12h0m0s"

		反射器中的重新同步周期在 MinResyncPeriod 和 2 * MinResyncPeriod 之间是随机的
- --node-monitor-period="5s"

		在 NodeController 中同步 NodeStatus 的时间段。
- --node-status-update-frequency="5m0s"

		指定控制器更新节点状态的频率。
- --port="10253"

		cloud-controller-manager http 监听端口
- --profiling="true"

		启用分析时，可以通过Web界面查看，访问 host:port/debug/pprof/.
- --route-reconciliation-period="30s"
- --service-account-private-key-file=""

		在1.8-1.10 版本之间废除，文件名包含用于签署服务帐户令牌的PEM编码的私有RSA或ECDSA密钥。此标志当前为 no-op，将被删除。
- --stderrthreshold="2"
- --use-service-account-credentials="true"

		如果为 true，请为每个控制器使用单独的服务帐户凭据。由 cloud provider 为协调节点创建路由的时间。
--v="5"
--version="false"
--vmodule=""


[providerID bug](https://github.com/openshift/origin/issues/22337)

[api&controller setting cloud-provider](https://github.com/kubernetes/cloud-provider-alibaba-cloud/issues/32)