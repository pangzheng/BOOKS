# 在 dmos 集群中部署 NGINX Ingress Controller    

Ingress 是一种 Kubernetes 第三方资源，借助 Nginx、Haproxy 或云厂商的负载均衡器将 Kubernetes 集群内的 Service 暴露到集群外。    

## 1. 导入镜像

+ 将 nginx-ingress-controller.tar 上传至其中一个master节点
+ 解压 nginx-ingress-controller.tar
+ 进入解压目录
+ 加载镜像并推送至 offline-registry 镜像仓库    
    
    ```
    docker load -i defaultbackend-1.4.tar
    docker load -i nginx-ingress-controller-0.21.0.tar
    docker push offlineregistry.dataman-inc.com:5000/openshift/defaultbackend:1.4
    docker push offlineregistry.dataman-inc.com:5000/openshift/nginx-ingress-controller:0.21.0
    
    ```

## 2. 创建namespace    

首先创建名称为 ingress-nginx 的 namespace，ingress-nginx 的相关组件的都将在这个 namespace 下，注意这里要是使用网络策略，需要将这个项目设置为可以连接所有项目，如 default 项目的标签 “dmos.io/netwok-policy.project: system”，或者直接创建到 default 下面    

+ 采用admin账号登录
+ 点击右上角"用户头像"按钮-->点击"集群管理"按钮-->点击"项目"按钮-->点击"创建新项目"按钮-->"项目名称"输入框输入: ingress-nginx。"显示名称"输入框输入： system-ingress-nginx -->点击"创建"按钮。    

## 3.部署default backend    

下面来创建default-http-backend的Deployment和Service.    

default-backend.yaml的内容如下：    

```
---

apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: default-http-backend
  labels:
    app: default-http-backend
  namespace: system-ingress-nginx
spec:
  replicas: 1
  selector:
    matchLabels:
      app: default-http-backend
  template:
    metadata:
      labels:
        app: default-http-backend
    spec:
      terminationGracePeriodSeconds: 60
      containers:
      - name: default-http-backend
        # Any image is permissible as long as:
        # 1. It serves a 404 page at /
        # 2. It serves 200 on a /healthz endpoint
        image: offlineregistry.dataman-inc.com:5000/openshift/defaultbackend:1.4
        livenessProbe:
          httpGet:
            path: /healthz
            port: 8080
            scheme: HTTP
          initialDelaySeconds: 30
          timeoutSeconds: 5
        ports:
        - containerPort: 8080
---

apiVersion: v1
kind: Service
metadata:
  name: default-http-backend
  namespace: system-ingress-nginx
  labels:
    app: default-http-backend
spec:
  ports:
  - port: 80
    targetPort: 8080
  selector:
    app: default-http-backend

```    

oc 命令部署default backend    

```
oc project system-ingress-nginx
oc apply -f default-backend.yaml
deployment "default-http-backend" created
service "default-http-backend" created

```    

确认default-http-backend的Service和Pod被创建，并且Pod处于Running状态。    

```
oc get svc,deploy,pod -n system-ingress-nginx
NAME                       TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)   AGE
svc/default-http-backend   ClusterIP   172.30.133.160   <none>        80/TCP    1m

NAME                          DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
deploy/default-http-backend   1         1         1            1           1m

NAME                                       READY     STATUS    RESTARTS   AGE
po/default-http-backend-59474db447-mz846   1/1       Running   0          1m

```    

default-http-backend从名称上来看即默认的后端，当集群外部的请求通过ingress进入到集群内部时，如果无法负载到相关后端的Service上，这种未知的请求将会被负载到这个默认的后端上。    

## 4.创建ConfigMap    

configmap用来定义nginx-ingress-controller全局配置以及是否开启tcp或ucp service功能。具体参照[官方configmap](https://kubernetes.github.io/ingress-nginx/user-guide/nginx-configuration/configmap/)。    

configmap内容如下：    

```
kind: ConfigMap
apiVersion: v1
metadata:
  name: nginx-configuration
  namespace: system-ingress-nginx
  labels:
    app.kubernetes.io/name: ingress-nginx
    app.kubernetes.io/part-of: ingress-nginx
data:
  load-balance: "ip_hash"

---
kind: ConfigMap
apiVersion: v1
metadata:
  name: tcp-services
  namespace: system-ingress-nginx
  labels:
    app.kubernetes.io/name: ingress-nginx
    app.kubernetes.io/part-of: ingress-nginx

---
kind: ConfigMap
apiVersion: v1
metadata:
  name: udp-services
  namespace: system-ingress-nginx
  labels:
    app.kubernetes.io/name: ingress-nginx
    app.kubernetes.io/part-of: ingress-nginx

```    

创建configmap：    

```
oc apply -f configmap.yaml
configmap "nginx-configuration" created
configmap "tcp-services" created
configmap "udp-services" created

oc get cm -n system-ingress-nginx
NAME                  DATA      AGE
nginx-configuration   1         45s
tcp-services          0         45s
udp-services          0         45s

```    

## 5.创建ServiceAccount并进行授权    

接下来创建ServiceAccount nginx-ingress-serviceaccount，并创建相关的ClusterRole, Role, ClusterRoleBinding, RoleBinding以对其进行授权。serviceaccount和角色进行绑定以使nginx-ingress-controller具备跨项目检查和更新ingress能力.：    

```
apiVersion: v1
kind: ServiceAccount
metadata:
  name: nginx-ingress-serviceaccount
  namespace: system-ingress-nginx
  labels:
    app.kubernetes.io/name: ingress-nginx
    app.kubernetes.io/part-of: ingress-nginx

---
apiVersion: rbac.authorization.k8s.io/v1beta1
kind: ClusterRole
metadata:
  name: nginx-ingress-clusterrole
  labels:
    app.kubernetes.io/name: ingress-nginx
    app.kubernetes.io/part-of: ingress-nginx
rules:
  - apiGroups:
      - ""
    resources:
      - configmaps
      - endpoints
      - nodes
      - pods
      - secrets
    verbs:
      - list
      - watch
  - apiGroups:
      - ""
    resources:
      - nodes
    verbs:
      - get
  - apiGroups:
      - ""
    resources:
      - services
    verbs:
      - get
      - list
      - watch
  - apiGroups:
      - "extensions"
    resources:
      - ingresses
    verbs:
      - get
      - list
      - watch
  - apiGroups:
      - ""
    resources:
      - events
    verbs:
      - create
      - patch
  - apiGroups:
      - "extensions"
    resources:
      - ingresses/status
    verbs:
      - update

---
apiVersion: rbac.authorization.k8s.io/v1beta1
kind: Role
metadata:
  name: nginx-ingress-role
  namespace: system-ingress-nginx
  labels:
    app.kubernetes.io/name: ingress-nginx
    app.kubernetes.io/part-of: ingress-nginx
rules:
  - apiGroups:
      - ""
    resources:
      - configmaps
      - pods
      - secrets
      - namespaces
    verbs:
      - get
  - apiGroups:
      - ""
    resources:
      - configmaps
    resourceNames:
      # Defaults to "<election-id>-<ingress-class>"
      # Here: "<ingress-controller-leader>-<nginx>"
      # This has to be adapted if you change either parameter
      # when launching the nginx-ingress-controller.
      - "ingress-controller-leader-nginx"
    verbs:
      - get
      - update
  - apiGroups:
      - ""
    resources:
      - configmaps
    verbs:
      - create
  - apiGroups:
      - ""
    resources:
      - endpoints
    verbs:
      - get

---
apiVersion: rbac.authorization.k8s.io/v1beta1
kind: RoleBinding
metadata:
  name: nginx-ingress-role-nisa-binding
  namespace: system-ingress-nginx
  labels:
    app.kubernetes.io/name: ingress-nginx
    app.kubernetes.io/part-of: ingress-nginx
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: nginx-ingress-role
subjects:
  - kind: ServiceAccount
    name: nginx-ingress-serviceaccount
    namespace: system-ingress-nginx

---
apiVersion: rbac.authorization.k8s.io/v1beta1
kind: ClusterRoleBinding
metadata:
  name: nginx-ingress-clusterrole-nisa-binding
  labels:
    app.kubernetes.io/name: ingress-nginx
    app.kubernetes.io/part-of: ingress-nginx
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: nginx-ingress-clusterrole
subjects:
  - kind: ServiceAccount
    name: nginx-ingress-serviceaccount
    namespace: system-ingress-nginx

---

```    

创建rbac：    

```
oc apply -f rbac.yaml

```    

## 6.部署nginx-ingress-controller    

下面部署关键组件nginx-ingress-controller这个ingress controller，前面提到Ingress是一种Kubernetes资源，借助Nginx、Haproxy或云厂商的负载均衡器将Kubernetes集群内的Service暴露到集群外，nginx-ingress-controller就是这个负载均衡器。    

```
---

apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: nginx-ingress-controller
  namespace: system-ingress-nginx
spec:
  replicas: 2
  selector:
    matchLabels:
      app: ingress-nginx
  template:
    metadata:
      labels:
        app: ingress-nginx
      annotations:
        prometheus.io/port: '10254'
        prometheus.io/scrape: 'true'
    spec:
      serviceAccountName: nginx-ingress-serviceaccount
      affinity:
        nodeAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            nodeSelectorTerms:
            - matchExpressions:
              - key: kubernetes.io/hostname
                operator: In
                values:
                - node159.dmos.dataman
                - infra202.dmos.dataman
      containers:
        - name: nginx-ingress-controller
          image: offlineregistry.dataman-inc.com:5000/openshift/nginx-ingress-controller:0.15.0
          args:
            - /nginx-ingress-controller
            - --default-backend-service=$(POD_NAMESPACE)/default-http-backend
            - --configmap=$(POD_NAMESPACE)/nginx-configuration
            - --tcp-services-configmap=$(POD_NAMESPACE)/tcp-services
            - --udp-services-configmap=$(POD_NAMESPACE)/udp-services
            - --publish-service=$(POD_NAMESPACE)/ingress-nginx
            - --annotations-prefix=nginx.ingress.kubernetes.io
          env:
            - name: POD_NAME
              valueFrom:
                fieldRef:
                  fieldPath: metadata.name
            - name: POD_NAMESPACE
              valueFrom:
                fieldRef:
                  fieldPath: metadata.namespace
          ports:
          - name: http
            containerPort: 80
          - name: https
            containerPort: 443
          livenessProbe:
            failureThreshold: 3
            httpGet:
              path: /healthz
              port: 10254
              scheme: HTTP
            initialDelaySeconds: 10
            periodSeconds: 10
            successThreshold: 1
            timeoutSeconds: 1
          readinessProbe:
            failureThreshold: 3
            httpGet:
              path: /healthz
              port: 10254
              scheme: HTTP
            periodSeconds: 10
            successThreshold: 1
            timeoutSeconds: 1
          securityContext:
            runAsNonRoot: false

```    

注： 注意这里使用Kubernetes Pod调度的Node亲和性特性，将nginx-ingress-controller的两个Pod实例固定调度到node159.dmos.dataman和infra202.dmos.dataman两个节点上。在部署前，需要先将nginx-ingress-controller.yaml文件的node亲和性节点域名修改为实际的部署节点(节点是没有监听80和443端口；否则将会出现端口冲突);另外需要注意副本数和节点数一致。    

部署nginx-ingress-controller    

```
oc apply -f nginx-ingress-controller.yaml    
deployment "nginx-ingress-controller" created

```    

## 7. 将nginx-ingress-controller暴露到Kubernetes集群外    

再看前面提到的Ingress是一种Kubernetes资源，借助Nginx、Haproxy或云厂商的负载均衡器将Kubernetes集群内的Service暴露到集群外，nginx-ingress-controller就是这个负载均衡器。所以还需要为nginx-ingress-controller创建一个Service并暴露到集群外边(ps: 这个有点先有鸡还是先有蛋的意思)。 因为Kubernetes集群是采以Bare-metal部署的，这里使用ExternalIP的形式创建一个ingress-nginx的Service，ingress-nginx.svc.yaml如下：    

```
apiVersion: v1
kind: Service
metadata:
  name: ingress-nginx
  namespace: system-ingress-nginx
spec:
  externalIPs:
  - 192.168.x.159
  - 192.168.x.201
  ports:
  - name: http
    port: 80
    targetPort: 80
    protocol: TCP
  - name: https
    port: 443
    targetPort: 443
    protocol: TCP
  selector:
    app: ingress-nginx

```    

其中192.168.x.159, 192.168.x.201分别对应node159.dmos.dataman和infra202.dmos.dataman两个openshift node的ip;需要先修改ingress-nginx.svc.yaml文件的externalIPs再创建。    

```
oc apply -f ingress-nginx.svc.yaml
service "ingress-nginx" created

```    

```
oc get svc -n system-ingress-nginx
NAME                   TYPE        CLUSTER-IP       EXTERNAL-IP     PORT(S)          AGE
default-http-backend   ClusterIP   172.30.133.160   <none>          80/TCP           1h
ingress-nginx          ClusterIP   172.30.190.9     192.168.x.159,192.168.x.201   80/TCP,443/TCP   1m

```

可以看出通过192.168.x.159,192.168.x.201:80/443将nginx-ingress-controller的nginx的80和443端口暴露到集群外边。

分别请求这两个ExternalIP:    

```
curl 192.168.x.159:80
default backend - 404

curl 192.168.x.201:80
default backend - 404

```    

因为我们还没有在Kubernetes集群中创建ingress资源，所以直接对这两个ExternalIP的请求被负载到了default backend上。

分别在node159.dmos.dataman和infra202.dmos.dataman上查看：    

```
netstat -nltp | grep openshift | grep 80
tcp        0      0 192.168.x.159:80        0.0.0.0:*               LISTEN      22718/openshift

netstat -nltp | grep kube-proxy | grep 80
tcp        0      0 192.168.x.201:80        0.0.0.0:*               LISTEN      22719/openshift

```    

可以看到ExternalIP的Service也是通过kube-proxy对外暴露的。这里的192.168.x.159和192.168.x.201是两个内网ip。 实际中需要将边缘路由器或全局统一接入层的负载均衡器将到达公网ip的外网流量转发到内网ip上，外部通过域名访问集群中将会以ingress暴露的所有服务。    
 
## 创建一个 ingress 资源
- 发布一个服务
		
	进入一个项目，比如 default && 创建一个任务，比如 2048
- 创建一个目录

		mkdir -p /data/test && cd /data/test
- 创建一个策略

		vi 2048-test-ingress.yaml
		
		apiVersion: extensions/v1beta1
		kind: Ingress
		metadata:
		  annotations:
		    ingress.kubernetes.io/class: nginx #ingress 类型
		    nginx.ingress.kubernetes.io/rewrite-target: / # 流量会导入到重写项目路径到实际项目的根目录下
		    nginx.ingress.kubernetes.io/affinity: cookie # 会话保持
		    nginx.ingress.kubernetes.io/ssl-redirect: "false" # 关闭 https 直通
		  name: 2048.test.com # ingress 名称
		  namespace: default # 所在项目
		spec:
		  rules:
		  - host: 2048.test.com # 项目对外域名
		    http:
		      paths:
		      - backend:
		          serviceName: blackicebird-2048 #对service 用来给ingress 做服务发现
		          servicePort: 80 #端口 80
		        path: /aa # 映射目录
- 执行

		oc apply -f 2048-test-ingress.yaml && oc get ingress 
- 添加项目

		vi 	2048-test-ingress.yaml.bak
		
		apiVersion: extensions/v1beta1
		kind: Ingress
		metadata:
		  annotations:
		    ingress.kubernetes.io/class: nginx
		    nginx.ingress.kubernetes.io/rewrite-target: /
		  name: 2048.test.com
		  namespace: default
		spec:
		  rules:
		  - host: 2048.test.com
		    http:
		      paths:
		      - backend:
		          serviceName: blackicebird-2048
		          servicePort: 80
		        path: /aa
		  - host: 2048.test.com
		    http:
		      paths:
		      - backend:
		          serviceName: blackicebird-2048
		          servicePort: 80
		        path: /bb
		  - host: aa.test.com
		    http:
		      paths:
		      - backend:
		          serviceName: blackicebird-2048
		          servicePort: 80
		        path: /cc

注意修改 ingress 的权限，请使用集群管理员或者项目管理员		        	        
## 参考    

[官方nginx-ingress](https://kubernetes.github.io/ingress-nginx/)    
[Kubernetes Ingress实战(一)：在Kubernetes集群中部署NGINX Ingress Controller](https://blog.frognew.com/2018/06/kubernetes-ingress-1.html)    
[kubernetes-ingress](https://kubernetes.io/docs/concepts/services-networking/ingress/)
