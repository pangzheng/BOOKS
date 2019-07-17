# istio 重要术语说明
- VirtualService 

	在 Istio 服务网格中定义路由规则，控制 envoy 如何路由到服务上。例,发送到 reviews 服务的100%流量要路由到 reviews 服务实例的 v1 子集中，路由中的 subset 制定了一个预定义的子集名称，子集的定义来自于目标规则配置：子集指定了一个或多个特定版本的实例标签。例如，在 Kubernetes 中部署 Istio 时，“version: v1” 表示只有包含 “version: v1” 标签版本的 pod 才会接收流量。
	
		apiVersion: networking.istio.io/v1alpha3
		kind: VirtualService
		metadata:
		  name: reviews #路由规则对应着一或多个用 VirtualService 配置指定的请求目的主机。
		spec:
		  hosts: #本字段用显示或者隐式的方式定义了一或多个完全限定名（FQDN）。值 reviews，会隐式的扩展成为特定的 FQDN，在 Kubernetes 环境中，全名会从 VirtualService 所在的集群和命名空间中继承而来（比如 reviews.default.svc.cluster.local）
		  - reviews #主机可以是或不是实际的目标负载，甚至可以不是同一网格内可路由的服务。如要给到 reviews 服务的请求定义路由规则，可以使用内部的名称 reviews，也可以用域名 bookinfo.com
		  http:
		  - route:
		    - destination:
		        host: reviews
		        subset: v1
	- 流量拆分
	
			apiVersion: networking.istio.io/v1alpha3
			kind: VirtualService
			metadata:
			  name: reviews
			spec:
			  hosts:
			    - reviews
			  http:
			  - route:
			    - destination:
			        host: reviews
			        subset: v1
			      weight: 75 # 75% 流量到 v1 版本
			    - destination:
			        host: reviews
			        subset: v2
			      weight: 25	 # 25% 流量到 v1 版本
	- 超时，超时还可以针对[每个请求分别设置](https://istio.io/zh/docs/concepts/traffic-management/#%E5%BE%AE%E8%B0%83)

			apiVersion: networking.istio.io/v1alpha3
			kind: VirtualService
			metadata:
			  name: ratings
			spec:
			  hosts:
			    - ratings
			  http:
			  - route:
			    - destination:
			        host: ratings
			        subset: v1
			    timeout: 10s	#默认设置是 15秒，用这个参数覆盖
	- 重试，重试和超时还可以针对每个请求分别设置

			apiVersion: networking.istio.io/v1alpha3
			kind: VirtualService
			metadata:
			  name: ratings
			spec:
			  hosts:
			    - ratings
			  http:
			  - route:
			    - destination:
			        host: ratings
			        subset: v1
			    retries:
			      attempts: 3 #最大重测次数
			      perTryTimeout: 2s	 #规定时间内一直重试
	- 错误注入
		- 延迟错误
		
				apiVersion: networking.istio.io/v1alpha3
				kind: VirtualService
				metadata:
				  name: ratings
				spec:
				  hosts:
				  - ratings
				  http:
				  - fault:
				      delay:
				        percent: 10 #对百分之10的请求
				        fixedDelay: 5s #注入5秒延迟
				    route:
				    - destination:
				        host: ratings
				        subset: v1
		- 400错误

				apiVersion: networking.istio.io/v1alpha3
				kind: VirtualService
				metadata:
				  name: ratings
				spec:
				  hosts:
				  - ratings
				  http:
				  - fault:
				      abort:
				        percent: 10 #对百分之10的请求
				        httpStatus: 400 #注入400错误
				    route:
				    - destination:
				        host: ratings
				        subset: v1
		- 同时使用

				apiVersion: networking.istio.io/v1alpha3
				kind: VirtualService
				metadata:
				  name: ratings
				spec:
				  hosts:
				  - ratings
				  http:
				  - match:
				    - sourceLabels:
				        app: reviews
				        version: v2
				    fault:
				      delay:
				        fixedDelay: 5s #所有请求5秒延迟
				      abort:
				        percent: 10
				        httpStatus: 400 # 400错误
				    route:
				    - destination:
				        host: ratings
				        subset: v1
		- 条件规则
			- 根据 pod 标签
				- 单一标签 
	
						apiVersion: networking.istio.io/v1alpha3
						kind: VirtualService
						metadata:
						  name: ratings
						spec:
						  hosts:
						  - ratings
						  http:
						  - match:
						      sourceLabels: #sourceLabels 的值取决于服务的实现。如，在 Kubernetes 中，它与相应 Kubernetes 服务的 pod 选择器中使用的 label 相同
						        app: reviews #规则指示仅适用于 reviews 服务 pod 的调用
						    ...
				- 多标签
	
						apiVersion: networking.istio.io/v1alpha3
						kind: VirtualService
						metadata:
						  name: ratings
						spec:
						  hosts:
						  - ratings
						  http:
						  - match:
						    - sourceLabels:
						        app: reviews
						        version: v2 #规则指示仅适用于 reviews v2 版本的服务 pod 的调用
						    ...
			- 根据 HTTP Header ，如果规则中指定了多个标头，则所有相应的标头必须匹配才能应用规则。

					apiVersion: networking.istio.io/v1alpha3
					kind: VirtualService
					metadata:
					  name: reviews
					spec:
					  hosts:
					    - reviews
					  http:
					  - match: # end-user 标头的来源请求，且值为 jason 的请求生效
					    - headers:
					        end-user: 
					          exact: jason
					    ...
			- 根据请求URI

					apiVersion: networking.istio.io/v1alpha3
					kind: VirtualService
					metadata:
					  name: productpage
					spec:
					  hosts:
					    - productpage
					  http:
					  - match: # URI 路径以 /api/v1 开头
					    - uri:
					        prefix: /api/v1
					    ...
			- 多重匹配条件
				- and

						apiVersion: networking.istio.io/v1alpha3
						kind: VirtualService
						metadata:
						  name: ratings
						spec:
						  hosts:
						  - ratings
						  http:
						  - match:
						    - sourceLabels:
						        app: reviews
						        version: v2
						      headers:
						        end-user:
						          exact: jason
						    ...
				- or 
				
						apiVersion: networking.istio.io/v1alpha3
						kind: VirtualService
						metadata:
						  name: ratings
						spec:
						  hosts:
						  - ratings
						  http:
						  - match:
						    - sourceLabels:
						        app: reviews
						        version: v2
						    - headers:
						        end-user:
						          exact: jason
						    ...		    	
			- 优先级

				当对同一目标有多个规则时，会按照在 VirtualService 中的顺序进行应用，换句话说，列表中的第一条规则具有最高优先级
				
					apiVersion: networking.istio.io/v1alpha3
					kind: VirtualService
					metadata:
					  name: reviews
					spec:
					  hosts:
					  - reviews
					  http:
					  - match: #如果 Header 包含 Foo=bar，就会被路由到 v2 版本
					    - headers:
					        Foo:
					          exact: bar
					    route:
					    - destination:
					        host: reviews
					        subset: v2
					  - route: # 其他流量匹配到 v1 版本
					    - destination:
					        host: reviews
					        subset: v1	     		    	      	        	      
- DestinationRule

	在请求被 VirtualService 路由之后，DestinationRule 配置的一系列策略就生效了。这些策略由运维人员或服务开发人员编写，包含断路器、负载均衡以及 TLS 等的配置内容，在单个 DestinationRule 配置中可以包含多条策略（比如 default 和 v2）。虽然 Istio 在没有定义任何规则的情况下，能将所有来源的流量发送给所有版本的目标服务。然而一旦需要对版本有所区别，就需要制定规则了。从一开始就给每个服务设置缺省规则，是 Istio 世界里推荐的最佳实践。

		apiVersion: networking.istio.io/v1alpha3
		kind: DestinationRule
		metadata:
		  name: reviews
		spec:
		  host: reviews #对 VirtualService 路由规则定义了规则
		  trafficPolicy:
		    loadBalancer:
		      simple: RANDOM #定义指定使用随机负载均衡模式
		  subsets: #定义子集标签
		  - name: v1
		    labels:
		      version: v1
		  - name: v2
		    labels:
		      version: v2
		    trafficPolicy:
		      loadBalancer:
		        simple: ROUND_ROBIN #定义指定使用随机负载均衡模式
		  - name: v3
		    labels:
		      version: v3	
	- 断路器

			apiVersion: networking.istio.io/v1alpha3
			kind: DestinationRule
			metadata:
			  name: reviews # 没有设置路由规则，缺省的 round-robin 策略，如果只有一个 v1 实例，那么所有请求都会发送给它。然而默认的 rr 的策略是永远不会生效的
			spec:
			  host: reviews #只为 review 服务定义了规则（没有对应的 VirtualService 路由规则）
			  subsets:
			  - name: v1
			    labels:
			      version: v1
			    trafficPolicy:
			      connectionPool:
			        tcp:
			          maxConnections: 100 #给 reviews 服务的 v1 版本设置了 100 连接的限制
		- 解决 VirtualService 永远不生效的办法
			- 提高策略层级，对所有请求生效

					apiVersion: networking.istio.io/v1alpha3
					kind: DestinationRule
					metadata:
					  name: reviews
					spec:
					  host: reviews
					  trafficPolicy:
					    connectionPool:
					      tcp:
					        maxConnections: 100
					  subsets:
					  - name: v1
					    labels:
					      version: v1 
			- 服务定义路由规则

					apiVersion: networking.istio.io/v1alpha3
					kind: VirtualService
					metadata:
					  name: reviews
					spec:
					  hosts:
					  - reviews
					  http:
					  - route:
					    - destination:
					        host: reviews
					        subset: v1		      		          
				      
- ServiceEntry

	Istio 内部会维护一个服务注册表，可以用 ServiceEntry 向其中加入额外的条目。通常这个对象用来启用对 Istio 服务网格之外的服务发出请求。ServiceEntry 的配置不仅限于外部服务，它有两种类型：1、网格内部的条目和其他的内部服务类似，用于显式的将服务加入网格。可以用来把服务作为服务网格扩展的一部分加入不受管理的基础设置（例如加入到基于 Kubernetes 的服务网格中的虚拟机）中。2、网格外的条目用于表达网格外的服务。对这种条目来说，双向 TLS 认证被禁用，策略实现需要在客户端执行，而不像内部服务请求那样在服务端执行。流量的重定向和转发、定义重试和超时以及错误注入策略都支持外部目标。然而由于外部服务没有多版本的概念，因此权重（基于版本）路由就无法实现了。
	
		apiVersion: networking.istio.io/v1alpha3
		kind: ServiceEntry
		metadata:
		  name: foo-ext-svc
		spec:
		  hosts: # 目标地址，字段值可以是一个完全限定名，也可以是个通配符域名
		  - *.foo.com
		  ports:
		  - number: 80
		    name: http
		    protocol: HTTP
		  - number: 443
		    name: https
		    protocol: HTTPS
	设置 VirtualService 配合

		apiVersion: networking.istio.io/v1alpha3
		kind: VirtualService
		metadata:
		  name: bar-foo-ext-svc 
		spec:
		  hosts:
		    - bar.foo.com 
		  http:
		  - route:
		    - destination:
		        host: bar.foo.com #访问 bar.foo.com
		    timeout: 10s	#设置一个10秒超时	    
    	
- Gateway 

	为 HTTP/TCP 流量配置负载均衡器，最常见的是在网格的边缘的操作，用于应用程序的入口流量，和 Kubernetes Ingress 不同，Istio Gateway 只配置四层到六层的功能（例如开放端口或者 TLS 配置）。绑定一个 VirtualService 到 Gateway 上，用户就可以使用标准的 Istio 规则来控制进入的 HTTP 和 TCP 流量。
	
	- 配合一个负载均衡，允许外部针对主机 bookinfo.com 的 https 流量
	
			apiVersion: networking.istio.io/v1alpha3
			kind: Gateway
			metadata:
			  name: bookinfo-gateway
			spec:
			  servers:
			  - port:
			      number: 443
			      name: https
			      protocol: HTTPS
			    hosts:
			    - bookinfo.com
			    tls:
			      mode: SIMPLE
			      serverCertificate: /tmp/tls.crt
			      privateKey: /tmp/tls.key
	- 要为 Gateway 配置对应的路由，必须为定义一个同样 hosts 定义的 VirtualService，其中用 gateways 字段来绑定到定义好的 Gateway 上：

			apiVersion: networking.istio.io/v1alpha3
			kind: VirtualService
			metadata:
			  name: bookinfo
			spec:
			  hosts:
			    - bookinfo.com
			  gateways:
			  - bookinfo-gateway # <---- 绑定到 Gateway
			  http:
			  - match:
			    - uri:
			        prefix: /reviews
			    route:
			    ...

## 参考
[istio 流量管理](https://istio.io/zh/docs/concepts/traffic-management/)			    