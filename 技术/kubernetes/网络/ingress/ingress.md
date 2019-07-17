## 什么是Ingress？
通常情况下，`service` 和 `pod` 仅可在集群内部网络中通过 IP 地址访问。所有到达边界路由器的流量或被丢弃或被转发到其他地方。从概念上讲，可能像下面这样：

	    internet
	        |
	  ------------
	  [ Services ]
`Ingress` 是授权入站连接到达集群服务的规则集合,可以理解成边缘网络设备或者路由

	    internet
	        |
	   [ Ingress ]
	   --|-----|--
	   [ Services ]
可以给 `Ingress` 配置提供外部可访问的 URL、负载均衡、SSL、基于名称的虚拟主机等。用户通过设置 `ingress` 资源到 `API server` 的方式来请求 `ingress`。 `Ingress controller` 负责实现，通常使用负载平衡器，它还可以配置边界路由和其他前端，这有助于以 HA 方式处理流量。
## 先决条件
在使用 Ingress resource 之前，有必要先了解下面几件事情。Ingress 是 beta 版本的 resource。需要一个 [Ingress Controller](https://kubernetes.io/docs/concepts/services-networking/ingress/#ingress-controllers) 来实现 Ingress，单纯的创建一个 Ingress 没有任何意义。

GCE/GKE 会在 master 节点上部署一个 `ingress controller`。烨可以在一个 `pod` 中部署任意个自定义的 `ingress controller`。但必须正确地 `annotate` 每个 `ingress`，比如运行多个 [ingress controller](https://github.com/kubernetes/ingress/tree/master/controllers/nginx#running-multiple-ingress-controllers) 和关闭 [glbc](https://github.com/kubernetes/ingress/blob/master/controllers/gce/BETA_LIMITATIONS.md#disabling-glbc).

使用前需要阅读 Ingress controller 的 [beta版本限制](https://github.com/kubernetes/ingress/blob/master/controllers/gce/BETA_LIMITATIONS.md)。在非GCE/GKE的环境中需要在 pod中部署一个 [controller](https://github.com/kubernetes/ingress/tree/master/controllers)。

## Ingress Resource
最简化的 Ingress 配置

	1: apiVersion: extensions/v1beta1
	2: kind: Ingress
	3: metadata:
	4:   name: test-ingress
	5: spec:
	6:   rules:
	7:   - http:
	8:       paths:
	9:       - path: /testpath
	10:        backend:
	11:           serviceName: test
	12:           servicePort: 80
如果没有配置 `Ingress controller` 就将其 POST 到 API server 将不会有任何用

- 配置说明
	- 1-4行

		跟 Kubernetes 的其他配置一样，ingress 的配置也需要 `apiVersion`，`kind` 和 `metadata` 字段。配置文件的详细说明请查看[部署应用](https://kubernetes.io/docs/user-guide/deploying-applications), [配置容器](https://kubernetes.io/docs/user-guide/configuring-containers)和[使用resources](https://kubernetes.io/docs/user-guide/working-with-resources).
	- 5-7行

		Ingress [spec](https://github.com/kubernetes/community/blob/master/contributors/devel/api-conventions.md#spec-and-status) 中包含配置一个 loadbalancer 或 proxy server 的所有信息。最重要的是，它包含了一个匹配所有入站请求的规则列表。目前 `ingress`只支持 http 规则。
	- 8-9行

		每条 http 规则包含以下信息：一个 `host` 配置项（比如for.bar.com，在这个例子中默认是*），`path` 列表（比如：/testpath），每个 path 都关联一个 `backend` (比如test:80)。在 loadbalancer 将流量转发到 `backend` 之前，所有的入站请求都要先匹配 `host` 和 `path` 。
	- 10-12行

		正如 services doc 中描述的那样，`backend` 是一个 `service:port` 的组合。`Ingress` 的流量被转发到它所匹配的 `backend`。
	- 全局参数：为了简单起见，Ingress 示例中没有全局参数，请参阅资源完整定义的 [api参考](https://releases.k8s.io/master/pkg/apis/extensions/v1beta1/types.go)。 在所有请求都不能跟 `spec` 中的 `path` 匹配的情况下，请求被发送到 `Ingress controller` 的默认后端，可以指定全局缺省 `backend`。

## Ingress Controllers
为了使 Ingress 正常工作，集群中必须运行 Ingress controller。 这与其他类型的控制器不同，其他类型的控制器通常作为 `kube-controller-manager` 二进制文件的一部分运行，在集群启动时自动启动。 你需要选择最适合自己集群的 Ingress controller 或者自己实现一个。 示例和说明可以在[这里](https://github.com/kubernetes/ingress/tree/master/controllers)找到。

## 在你开始前
以下文档描述了 Ingress 资源中公开的一组跨平台功能。 理想情况下，所有的 Ingress controller 都应该符合这个规范，但是我们还没有实现。 可查看 [GCE](https://github.com/kubernetes/ingress/blob/master/controllers/gce/README.md) 和 [nginx控制器](https://github.com/kubernetes/ingress/blob/master/controllers/nginx/README.md) 的文档。确保您查看控制器特定的文档，以便您了解每个文档的注意事项。
 
## Ingress类型
### 单 Service Ingress
Kubernetes中已经存在一些概念可以暴露单个service（查看[替代方案](https://kubernetes.io/docs/concepts/services-networking/ingress/#alternatives)），但是仍然可以通过 Ingress 来实现，通过指定一个没有 rule 的默认 backend 的方式。

ingress.yaml 定义文件

	apiVersion: extensions/v1beta1
	kind: Ingress
	metadata:
	  name: test-ingress
	spec:
	  backend:
	    serviceName: testsvc
	    servicePort: 80
使用 `kubectl create -f` 命令创建，然后查看 ingress：

	$ kubectl get ing
	NAME                RULE          BACKEND        ADDRESS
	test-ingress        -             testsvc:80     107.178.254.228
107.178.254.228 就是 Ingress controller 为了实现 Ingress 而分配的IP地址。`RULE` 列表示所有发送给该IP的流量都被转发到了 `BACKEND` 所列的 Kubernetes service 上。

### 简单展开
如前面描述的那样，kubernete pod 中的 IP 只在集群网络内部可见，需要在边界设置一个东西，让它能够接收 ingress 的流量并将它们转发到正确的端点上。这个东西一般是高可用的 loadbalancer 。使用 Ingress 能够允许你将 loadbalancer 的个数降低到最少，例如，想要创建这样的一个设置：

	foo.bar.com -> 178.91.123.132 -> / foo    s1:80
	                                 / bar    s2:80
需要一个这样的 ingress：

	apiVersion: extensions/v1beta1
	kind: Ingress
	metadata:
	  name: test
	spec:
	  rules:
	  - host: foo.bar.com
	    http:
	      paths:
	      - path: /foo
	        backend:
	          serviceName: s1
	          servicePort: 80
	      - path: /bar
	        backend:
	          serviceName: s2
	          servicePort: 80
使用 `kubectl create -f` 创建完 ingress 后：

	$ kubectl get ing
	NAME      RULE          BACKEND   ADDRESS
	test      -
	          foo.bar.com
	          /foo          s1:80
	          /bar          s2:80
只要服务（s1，s2）存在，Ingress controller 就会将提供一个满足该 Ingress 的特定 loadbalancer 实现。 这一步完成后，将在 Ingress 的最后一列看到 loadbalancer 的地址。

### 基于名称的虚拟主机
Name-based 的虚拟主机在同一个 IP 地址下拥有多个主机名。

	foo.bar.com --|                 |-> foo.bar.com s1:80
	              | 178.91.123.132  |
	bar.foo.com --|                 |-> bar.foo.com s2:80
下面这个 ingress 说明基于 [Host header](https://tools.ietf.org/html/rfc7230#section-5.4) 的后端 loadbalancer 的路由请求

	apiVersion: extensions/v1beta1
	kind: Ingress
	metadata:
	  name: test
	spec:
	  rules:
	  - host: foo.bar.com
	    http:
	      paths:
	      - backend:
	          serviceName: s1
	          servicePort: 80
	  - host: bar.foo.com
	    http:
	      paths:
	      - backend:
	          serviceName: s2
	          servicePort: 80
默认 backend,一个没有 rule 的 ingress，如前面章节中所示，所有流量都将发送到一个默认  backend。可以用该技巧通知 loadbalancer 如何找到网站的404页面，通过制定一些列 rule 和一个默认 backend 的方式。如果请求 header 中的 host 不能跟 ingress 中的 host 匹配，并且 / 或请求的URL 不能与任何一个 path 匹配，则流量将路由到的默认 backend。

### TLS
可以通过指定包含 TLS 私钥和证书的 [secret](https://kubernetes.io/docs/user-guide/secrets) 来加密 Ingress。 目前，Ingress 仅支持单个 TLS 端口443，并假定 TLS termination。 如果 Ingress 中的 TLS 配置部分指定了不同的主机，则它们将根据通过 SNI TLS 扩展指定的主机名（假如 Ingress controller 支持 SNI ）在多个相同端口上进行复用。 TLS secret 中必须包含名为 tls.crt 和 tls.key 的密钥，这里面包含了用于 TLS 的证书和私钥，例如：

	apiVersion: v1
	data:
	  tls.crt: base64 encoded cert
	  tls.key: base64 encoded key
	kind: Secret
	metadata:
	  name: testsecret
	  namespace: default
	type: Opaque

在 Ingress 中引用这个 secret 将通知 Ingress controller 使用 TLS 加密从将客户端到 loadbalancer 的 channel：
	
	apiVersion: extensions/v1beta1
	kind: Ingress
	metadata:
	  name: no-rules-map
	spec:
	  tls:
	    - secretName: testsecret
	  backend:
	    serviceName: s1
	    servicePort: 80
	    
请注意，各种 Ingress controller 支持的 TLS 功能之间存在差距。 请参阅有关[nginx](https://github.com/kubernetes/ingress/blob/master/controllers/nginx/README.md#https)，[GCE](https://github.com/kubernetes/ingress/blob/master/controllers/gce/README.md#tls) 或任何其他平台特定 Ingress controller 的文档，以了解 TLS 在你的环境中的工作原理。

Ingress controller 启动时附带一些适用于所有 Ingress 的负载平衡策略设置，例如负载均衡算法，后端权重方案等。更高级的负载平衡概念（例如持久会话，动态权重）尚未在 Ingress 中公开。 你仍然可以通过 [service loadbalancer](https://github.com/kubernetes/contrib/tree/master/service-loadbalancer) 获取这些功能。 随着时间的推移，我们计划将适用于跨平台的负载平衡模式加入到Ingress资源中。

还值得注意的是，尽管健康检查不直接通过 Ingress 公开，但 Kubernetes 中存在并行概念，例如[准备探查](https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-probes/)，可以使你达成相同的最终结果。 请查看特定控制器的文档，以了解他们如何处理健康检查（[nginx](https://github.com/kubernetes/ingress/blob/master/controllers/nginx/README.md)，[GCE](https://github.com/kubernetes/ingress/blob/master/controllers/gce/README.md#health-checks)）。

### 更新Ingress
假如想要向已有的 ingress 中增加一个新的 Host，可以编辑和更新该 ingress：

	$ kubectl get ing
	NAME      RULE          BACKEND   ADDRESS
	test      -                       178.91.123.132
	          foo.bar.com
	          /foo          s1:80
	$ kubectl edit ing test

这会弹出一个包含已有的 yaml 文件的编辑器，修改它，增加新的 Host 配置。

	spec:
	  rules:
	  - host: foo.bar.com
	    http:
	      paths:
	      - backend:
	          serviceName: s1
	          servicePort: 80
	        path: /foo
	  - host: bar.baz.com
	    http:
	      paths:
	      - backend:
	          serviceName: s2
	          servicePort: 80
	        path: /foo

保存它会更新 API server 中的资源，这会触发 ingress controller 重新配置 loadbalancer。

	$ kubectl get ing
	NAME      RULE          BACKEND   ADDRESS
	test      -                       178.91.123.132
	          foo.bar.com
	          /foo          s1:80
	          bar.baz.com
	          /foo          s2:80
在一个修改过的 ingress yaml 文件上调用 `kubectl replace -f` 命令一样可以达到同样的效果。

## 跨可用域故障
联邦
### 未来计划
- 多样化的 HTTPS/TLS 模型支持（如SNI，re-encryption）
- 通过声明来请求IP或者主机名
- 结合 L4 和 L7 Ingress
- 更多的 Ingress controller

请跟踪 [L7和 Ingress 的 proposal](https://github.com/kubernetes/kubernetes/pull/12827)，了解有关资源演进的更多细节，以及 [Ingress repository](https://github.com/kubernetes/ingress/tree/master)，了解有关各种 Ingress controller 演进的更多详细信息。
### 替代方案
可以通过很多种方式暴露 service 而不必直接使用 ingress：

- 使用 [Service.Type=LoadBalancer](https://kubernetes.io/docs/user-guide/services/#type-loadbalancer)
- 使用 [Service.Type=NodePort](https://kubernetes.io/docs/user-guide/services/#type-nodeport)
- 使用 [Port Proxy](https://github.com/kubernetes/contrib/tree/master/for-demos/proxy-to-service)
- 部署一个 [Service loadbalancer](https://github.com/kubernetes/contrib/tree/master/service-loadbalancer) 这允许你在多个 service 之间共享单个 IP，并通过 Service Annotations 实现更高级的负载平衡。


## 参考
[Kubernetes Ingress解析](https://www.kubernetes.org.cn/1885.html)
[官方文档](https://kubernetes.io/docs/concepts/services-networking/ingress/)