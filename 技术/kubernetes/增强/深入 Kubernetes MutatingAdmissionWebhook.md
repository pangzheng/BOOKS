# 深入 Kubernetes MutatingAdmissionWebhook
[准入控制器(Admission controllers)](https://kubernetes.io/docs/admin/admission-controllers/)是在对象持久化之前,拦截 apiserver 请求的强大工具。但由于要求将它们编译为二进制的kube-apiserver，并由集群管理员配置，因此非常不灵活。从 Kubernetes 1.7 开始，引入 [Initializers](https://v1-8.docs.kubernetes.io/docs/admin/extensible-admission-controllers/#initializers) 和 [External Admission Webhooks](https://v1-8.docs.kubernetes.io/docs/admin/extensible-admission-controllers/#external-admission-webhooks) 来解决此限制。在 Kubernetes 1.9 中，`Initializers` 保持 alpha 阶段，同期的 `External Admission Webhooks` 提升为 beta，并分为 `MutatingAdmissionWebhook`和`ValidatingAdmissionWebhook`。

`MutatingAdmissionWebhook` 与 `ValidatingAdmissionWebhook` 和特殊类型的 `admission controllers` 进程一起变更和验证 [MutatingWebhookConfiguration](https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.9/#mutatingwebhookconfiguration-v1beta1-admissionregistration) 匹配定义规则的请求。

在本文中，将逐步深入了解 `MutatingAdmissionWebhook` 并有效编写 `Admission controllers`。
## Webhooks 的好处
Webhooks 允许 Kubernetes 集群管理员在 apiserver 不重新编译的情况下为准入链创建额外的变更和验证准入插件。这最终提供了一种自由和灵活的方法，可以在任何资源上对多个操作`（“CREATA”，“UPDATE”，“DELETE”...）`自定义准入逻辑。

一些常见用例包括：

- 在创建资源之前调整资源本身

	[Istio](https://github.com/istio) 是一个有代表性的例子，它将 [Envoy](https://github.com/envoyproxy/envoy) 以边车容器的方式注入到目标 pod 以实施流量管理。
- 自动配置 `StorageClass`

	观察 `PersistentVolumeClaim` 对象的创建，并根据预定义的策略自动为其添加存储类。不需要用户关心 `StorageClass` 的创建。
- 验证复杂的自定义资源

	确保只能在所有依赖和定义资源创建可用后创建自定义资源。
- 限制命名空间。

	在多租户系统上，避免在保留的命名空间中创建'非法'的资源。

除了上面列出的用户案例之外，还可以根据其功能创建更多类型的 `webhooks` 应用程序 。
## Webhooks VS Initializers
基于对 `External Admission Webhooks` 和`Initializers` 从社区和使用情况的反馈中,Kubernetes 社区决定推动 `webhook` 到 beta 版本，并将它拆分成 `MutatingAdmissionWebhook` 和 `ValidatingAdmissionWebhook`。这使 `webhooks` 与其他准入控制器保持一致并强制执行 `mutate-before-validate`。`Initializers` 还可以通过在实际创建资源之前修改它们来实现动态准入控制。如果不熟悉 `Initializers`，请参阅[Kubernetes Initializers Deep Dive and Tutorial](https://medium.com/ibm-cloud/kubernetes-initializers-deep-dive-and-tutorial-3bc416e4e13e)。

那么 `Webhooks` 和 `Initializers` 之间的区别是什么？

- `Webhooks` 可以应用于更多操作，包括对`'CREATE''UPDATE''DELETE'`的资源请求进行 `mutate`(变更) 或 `admit`(接纳) ，而 `Initializers` 不能进行 `admit` 资源的 `DELETE` 请求。
- `Webhooks` 不允许在创建之前查询资源，而 `Initializers` 能够通过查询参数观察未初始化的资源 `includeUninitialized=true`。
- `Initializers` 有“预创建”状态的持续存在，因此导致了 etcd 更高的延迟和更多的负担，尤其是在 apiserver 升级或失败时。而 Webhooks 则消耗更少的内存和计算资源。
- 从`Webhooks` 比 `Initializers` 的失败健壮性来看， `Webhooks` 可以配置失败策略，以避免挂起创建的资源。而 `Initializers` 的问题可能会阻止创建所有匹配的资源。

除了上面列出的差异之外，`Initializer` 还存在一些其他问题，包括配额补充错误。升级 `Webhooks` 到 beta 可能是一个信号，表明将来会有更多的支持。如果以稳定行为首选，建议选择 `Webhooks`。

## MutatingAdmissionWebhook 的工作原理
`MutatingAdmissionWebhook` 截取 `MutatingWebhookConfiguration` 预先存入 etcd 定义的规则来匹配的请求。`MutatingAdmissionWebhook` 通过向 `webhook` 服务器发送准入请求来执行变更。而 `Webhook` 服务器只是简单的 http 服务器，遵循 [API](https://github.com/kubernetes/kubernetes/blob/v1.9.0/pkg/apis/admission/types.go)。

下图描述了 `MutatingAdmissionWebhook` 详细的工作原理

![](./pic/mutating-admission-webhook.jpg)
`MutatingAdmissionWebhook` 需要三个对象的功能

1. MutatingWebhookConfiguration

	`MutatingAdmissionWebhook` 通过 apiserver 提供的 `MutatingWebhookConfiguration` 注册。在注册过程中，`MutatingAdmissionWebhook` 需要完成：

	- 如何连接到 webhook 准入控制器
	- 如何验证 webhook 准入控制器
	- webhook 准入控制器的 URL 路径
	- 定义哪个资源及其处理的操作的规则
	- 如何处理来自 webhook 准入控制器的无法识别的错误
- MutatingAdmissionWebhook

	MutatingAdmissionWebhook 是一个插件式的准入控制器，可在 apiserver 功能门进行配置，默认关闭。MutatingAdmissionWebhook 插件从 MutatingWebhookConfiguration 中获取准入 webhooks的列表。然后 MutatingAdmissionWebhook 观察对 apiserver 的请求并拦截与准入 webhook 中的规则，匹配的请求将并行调用它们处理。
- Webhook Admission Server

	Webhook Admission Server 只是简单的 http 服务器，遵循 [Kubernetes API](https://github.com/kubernetes/kubernetes/blob/v1.9.0/pkg/apis/admission/types.go)。对于 apiserver 的每个请求，MutatingAdmissionWebhook 会发送 admissionReview（参考API）到相关的 webhook 准入控制器。webhook 准入控制器收集诸如 `object`，`oldobject` 和 `userInfo` 来自的信息，并发回 `admissionReview` 包括 `AdmissionResponse` 的 `Allowed` 和 `Result` 等字段填充准入决定的响应，并且可选择 Patch 资源变更。
	
## MutatingAdmissionWebhook 教程
这里是一个简单的 Webhook Admission Server 来实现注入 nginx sidecar 容器和卷的例子。完整的代码可以在 [kube-mutating-webhook-tutorial](https://github.com/morvencao/kube-mutating-webhook-tutorial) 中找到。该项目涉及 [Kunernetes webhook](https://github.com/kubernetes/kubernetes/tree/release-1.9/test/images/webhook) 示例和 [Istio 边车](https://github.com/istio/istio/tree/master/pilot/pkg/kube/inject)。
### 先决条件
在 kubernetes api 上启动 MutatingAdmissionWebhook。 在 `admissionregistration.k8s.io/v1beta1` 启用 API 的情况下需要 Kubernetes 1.9.0 或更高版本。通过以下命令验证：

	kubectl api-versions | grep admissionregistration.k8s.io/v1beta1
结果应该是：

	admissionregistration.k8s.io/v1beta1
此外，应添加 `MutatingAdmissionWebhook` 和 `ValidatingAdmissionWebhook` 接纳控制器，并在 `admission-control` 标志中以正确的顺序列出 `kube-apiserver`。(注意这里新版不在需要)
### 编写 Webhook 服务器
Webhook Admission Server 只是简单的 http 服务器，遵循Kubernetes API。这里粘贴一些伪代码来描述主要逻辑

	sidecarConfig, err := loadConfig(parameters.sidecarCfgFile)
	pair, err := tls.LoadX509KeyPair(parameters.certFile, parameters.keyFile)
	
	whsvr := &WebhookServer {
	    sidecarConfig:    sidecarConfig,
	    server:           &http.Server {
	        Addr:        fmt.Sprintf(":%v", 443),
	        TLSConfig:   &tls.Config{Certificates: []tls.Certificate{pair}},
	    },
	}
		
	// define http server and server handler
	mux := http.NewServeMux()
	mux.HandleFunc("/mutate", whsvr.serve)
	whsvr.server.Handler = mux
	
	// start webhook server in new rountine
	go func() {
	    if err := whsvr.server.ListenAndServeTLS("", ""); err != nil {
	        glog.Errorf("Filed to listen and serve webhook server: %v", err)
	    }
	}()
	
上述代码的说明：

- `sidecarCfgFile` 包含 `ConfigMap` 下面定义的边车注入器模板。
- `certFile` 和 `keyFile` 被用作 apiserver 和 webhook server 之间建立 TLS 通信的密钥。
- 第19行启动 https 服务器侦听路径 '/mutate' 上的443。

接下来关注处理函数的主要逻辑 serve

	// Serve method for webhook server
	func (whsvr *WebhookServer) serve(w http.ResponseWriter, r *http.Request) {
		var body []byte
		if r.Body != nil {
			if data, err := ioutil.ReadAll(r.Body); err == nil {
				body = data
			}
		}
	
		var reviewResponse *v1beta1.AdmissionResponse
		ar := v1beta1.AdmissionReview{}
		deserializer := codecs.UniversalDeserializer()
		if _, _, err := deserializer.Decode(body, nil, &ar); err != nil {
			glog.Error(err)
			reviewResponse = toAdmissionResponse(err)
		} else {
			reviewResponse = mutate(ar)
		}
	
		response := v1beta1.AdmissionReview{}
		if reviewResponse != nil {
			response.Response = reviewResponse
			response.Response.UID = ar.Request.UID
		}
		// reset the Object and OldObject, they are not needed in a response.
		ar.Request.Object = runtime.RawExtension{}
		ar.Request.OldObject = runtime.RawExtension{}
	
		resp, err := json.Marshal(response)
		if err != nil {
			glog.Error(err)
		}
		if _, err := w.Write(resp); err != nil {
			glog.Error(err)
		}
	}	
该 `serve` 函数是带有 `http request` 和 `response writer` 参数的普通 http 处理程序。

- 首先解组的请求 `AdmissionReview`，其中包含像信息 `object，oldobject、userInfo...`
- 然后调用 Webhook 核心函数 `mutate` 来创建 `patch` 注入 sidecar 容器和卷。
- 最后，通过准入决定和可选补丁解组响应，将结果发回 apiserver。

对于 mutate 功能的一部分，可以自由地以喜欢的方式完成它。样例实现如下：

	// main mutation process
	func (whsvr *WebhookServer) mutate(ar *v1beta1.AdmissionReview) *v1beta1.AdmissionResponse {
		req := ar.Request
		var pod corev1.Pod
		if err := json.Unmarshal(req.Object.Raw, &pod); err != nil {
			glog.Errorf("Could not unmarshal raw object: %v", err)
			return &v1beta1.AdmissionResponse {
				Result: &metav1.Status {
					Message: err.Error(),
				},
			}
		}
		
		// determine whether to perform mutation
		if !mutationRequired(ignoredNamespaces, &pod.ObjectMeta) {
			glog.Infof("Skipping mutation for %s/%s due to policy check", pod.Namespace, pod.Name)
			return &v1beta1.AdmissionResponse {
				Allowed: true, 
			}
		}
	
		annotations := map[string]string{admissionWebhookAnnotationStatusKey: "injected"}
		patchBytes, err := createPatch(&pod, whsvr.sidecarConfig, annotations)
		
		return &v1beta1.AdmissionResponse {
			Allowed: true,
			Patch:   patchBytes,
			PatchType: func() *v1beta1.PatchType {
				pt := v1beta1.PatchTypeJSONPatch
				return &pt
			}(),
		}
	}
从上面的代码中，mutate 函数调用 [mutationRequired](https://github.com/morvencao/kube-mutating-webhook-tutorial/blob/master/webhook.go#L98-L130) 来确定是否需要进行变更。对于那些需要变更的请求来说，该 mutate 函数从另一个函数 [createPatch](https://github.com/morvencao/kube-mutating-webhook-tutorial/blob/master/webhook.go#L196-L205) 获得变更 'patch' 。注意功能上的小技巧 `mutationRequired`，跳过 pods 没有注释 `sidecar-injector-webhook.morven.me/inject:true`。在部署应用程序时会提到这一点。有关完整代码，请参阅[这里](https://github.com/morvencao/kube-mutating-webhook-tutorial/blob/master/webhook.go)

### 创建Dockerfile并构建容器
创建 build 脚本：

	dep ensure
	CGO_ENABLED=0 GOOS=linux go build -a -installsuffix cgo -o kube-mutating-webhook-tutorial .
	docker build --no-cache -t morvencao/sidecar-injector:v1 .
	rm -rf kube-mutating-webhook-tutorial
	
	docker push morvencao/sidecar-injector:v1
创建 Dockerfile 为构建脚本的依赖：

	FROM alpine:latest
	
	ADD kube-mutating-webhook-tutorial /kube-mutating-webhook-tutorial
	ENTRYPOINT ["./kube-mutating-webhook-tutorial"]
注意这里需要有镜像仓库对应权限：

	[root@mstnode kube-mutating-webhook-tutorial]# ./build
	Sending build context to Docker daemon  44.89MB
	Step 1/3 : FROM alpine:latest
	 ---> 3fd9065eaf02
	Step 2/3 : ADD kube-mutating-webhook-tutorial /kube-mutating-webhook-tutorial
	 ---> 432de60c2b3f
	Step 3/3 : ENTRYPOINT ["./kube-mutating-webhook-tutorial"]
	 ---> Running in da6e956d1755
	Removing intermediate container da6e956d1755
	 ---> 619faa936145
	Successfully built 619faa936145
	Successfully tagged morvencao/sidecar-injector:v1
	The push refers to repository [docker.io/morvencao/sidecar-injector]
	efd05fe119bb: Pushed
	cd7100a72410: Layer already exists
	v1: digest: sha256:7a4889928ec5a8bcfb91b610dab812e5228d8dfbd2b540cd7a341c11f24729bf size: 739
### 创建 Sidecar 注入配置
创建一个 `ConfigMap` ，其中包括 `container` 和 `volume` 将被注入到目标 pod 。

	apiVersion: v1
	kind: ConfigMap
	metadata:
	  name: sidecar-injector-webhook-configmap
	data:
	  sidecarconfig.yaml: |
	    containers:
	      - name: sidecar-nginx
	        image: nginx:1.12.2
	        imagePullPolicy: IfNotPresent
	        ports:
	          - containerPort: 80
	        volumeMounts:
	          - name: nginx-conf
	            mountPath: /etc/nginx
	    volumes:
	      - name: nginx-conf
	        configMap:
	          name: nginx-configmap
从上面的清单中，nginx-conf 需要包含另一个 `ConfigMap` 。这里把它放在 [nginxconfigmap.yaml](https://github.com/morvencao/kube-mutating-webhook-tutorial/blob/master/deployment/nginxconfigmap.yaml)里,然后将两个 `ConfigMaps` 部署到集群

	[root@mstnode kube-mutating-webhook-tutorial]# kubectl create -f ./deployment/nginxconfigmap.yaml
	configmap "nginx-configmap" created
	[root@mstnode kube-mutating-webhook-tutorial]# kubectl create -f ./deployment/configmap.yaml
	configmap "sidecar-injector-webhook-configmap" created
	
### 创建签名密钥/证书对
因为准入是一种高安全性操作，所以需要 `Kubernetes CA` 签名的 TLS 证书来保护 webhook 服务器与 apiserver 之间的通信。有关完整的创建和批准CSR过程，请参阅[信息](https://kubernetes.io/docs/tasks/tls/managing-tls-in-a-cluster/)。

为了简单起见，这里从 Istio 拿来一个类似的脚本 [webhook-create-signed-cert.sh](https://github.com/istio/istio/blob/master/install/kubernetes/webhook-create-signed-cert.sh) 来自动创建证书/密钥对，并将其包含在 Kubernetes `secret`

	#!/bin/bash
	while [[ $# -gt 0 ]]; do
	    case ${1} in
	        --service)
	            service="$2"
	            shift
	            ;;
	        --secret)
	            secret="$2"
	            shift
	            ;;
	        --namespace)
	            namespace="$2"
	            shift
	            ;;
	    esac
	    shift
	done
	
	[ -z ${service} ] && service=sidecar-injector-webhook-svc
	[ -z ${secret} ] && secret=sidecar-injector-webhook-certs
	[ -z ${namespace} ] && namespace=default
	
	csrName=${service}.${namespace}
	tmpdir=$(mktemp -d)
	echo "creating certs in tmpdir ${tmpdir} "
	
	cat <<EOF >> ${tmpdir}/csr.conf
	[req]
	req_extensions = v3_req
	distinguished_name = req_distinguished_name
	[req_distinguished_name]
	[ v3_req ]
	basicConstraints = CA:FALSE
	keyUsage = nonRepudiation, digitalSignature, keyEncipherment
	extendedKeyUsage = serverAuth
	subjectAltName = @alt_names
	[alt_names]
	DNS.1 = ${service}
	DNS.2 = ${service}.${namespace}
	DNS.3 = ${service}.${namespace}.svc
	EOF
	
	openssl genrsa -out ${tmpdir}/server-key.pem 2048
	openssl req -new -key ${tmpdir}/server-key.pem -subj "/CN=${service}.${namespace}.svc" -out ${tmpdir}/server.csr -config ${tmpdir}/csr.conf
	
	# clean-up any previously created CSR for our service. Ignore errors if not present.
	kubectl delete csr ${csrName} 2>/dev/null || true
	
	# create  server cert/key CSR and  send to k8s API
	cat <<EOF | kubectl create -f -
	apiVersion: certificates.k8s.io/v1beta1
	kind: CertificateSigningRequest
	metadata:
	  name: ${csrName}
	spec:
	  groups:
	  - system:authenticated
	  request: $(cat ${tmpdir}/server.csr | base64 | tr -d '\n')
	  usages:
	  - digital signature
	  - key encipherment
	  - server auth
	EOF
	
	# verify CSR has been created
	while true; do
	    kubectl get csr ${csrName}
	    if [ "$?" -eq 0 ]; then
	        break
	    fi
	done
	
	# approve and fetch the signed certificate
	kubectl certificate approve ${csrName}
	# verify certificate has been signed
	for x in $(seq 10); do
	    serverCert=$(kubectl get csr ${csrName} -o jsonpath='{.status.certificate}')
	    if [[ ${serverCert} != '' ]]; then
	        break
	    fi
	    sleep 1
	done
	if [[ ${serverCert} == '' ]]; then
	    echo "ERROR: After approving csr ${csrName}, the signed certificate did not appear on the resource. Giving up after 10 attempts." >&2
	    exit 1
	fi
	echo ${serverCert} | openssl base64 -d -A -out ${tmpdir}/server-cert.pem
	
	
	# create the secret with CA cert and server cert/key
	kubectl create secret generic ${secret} \
	        --from-file=key.pem=${tmpdir}/server-key.pem \
	        --from-file=cert.pem=${tmpdir}/server-cert.pem \
	        --dry-run -o yaml |
	    kubectl -n ${namespace} apply -f -

然后执行它并创建 `secret` 包含证书/密钥

	[root@mstnode kube-mutating-webhook-tutorial]# ./deployment/webhook-create-signed-cert.sh
	creating certs in tmpdir /tmp/tmp.wXZywp0wAF
	Generating RSA private key, 2048 bit long modulus
	...........................................+++
	..........+++
	e is 65537 (0x10001)
	certificatesigningrequest "sidecar-injector-webhook-svc.default" created
	NAME                                   AGE       REQUESTOR                                           CONDITION
	sidecar-injector-webhook-svc.default   0s        https://mycluster.icp:9443/oidc/endpoint/OP#admin   Pending
	certificatesigningrequest "sidecar-injector-webhook-svc.default" approved
	secret "sidecar-injector-webhook-certs" created
### 创建边车注入部署和服务
在 `deployment` 1个带有 `sidecar-injector` 容器 pod 时。容器以特殊参数开头：

- `sidecarCfgFile` 指向从 `sidecar-injector-webhook-configmap` 上面创建的 `ConfigMap ` 安装的 sidecar 注入器配置文件
- `tlsCertFile` 并且 `tlsKeyFile` 是从 `sidecar-injector-webhook-certs` 上面的脚本创建的秘密安装的证书/密钥对
- `alsologtostderr` `v=4` 和 `2>&1` 记录参数

		apiVersion: extensions/v1beta1
		kind: Deployment
		metadata:
		  name: sidecar-injector-webhook-deployment
		  labels:
		    app: sidecar-injector
		spec:
		  replicas: 1
		  template:
		    metadata:
		      labels:
		        app: sidecar-injector
		    spec:
		      containers:
		        - name: sidecar-injector
		          image: morvencao/sidecar-injector:v1
		          imagePullPolicy: IfNotPresent
		          args:
		            - -sidecarCfgFile=/etc/webhook/config/sidecarconfig.yaml
		            - -tlsCertFile=/etc/webhook/certs/cert.pem
		            - -tlsKeyFile=/etc/webhook/certs/key.pem
		            - -alsologtostderr
		            - -v=4
		            - 2>&1
		          volumeMounts:
		            - name: webhook-certs
		              mountPath: /etc/webhook/certs
		              readOnly: true
		            - name: webhook-config
		              mountPath: /etc/webhook/config
		      volumes:
		        - name: webhook-certs
		          secret:
		            secretName: sidecar-injector-webhook-certs
		        - name: webhook-config
		          configMap:
		            name: sidecar-injector-webhook-configmap
		            
在 `service` 需要定义和 pod定义一致的 `app=sidecar-injector` 标记，好使其在集群内访问。`MutatingWebhookConfigurationin` 中的 `clientConfig` 中部分引将引用此服务，默认情况下`spec.ports.port` 为 443。

	apiVersion: v1
	kind: Service
	metadata:
	  name: sidecar-injector-webhook-svc
	  labels:
	    app: sidecar-injector
	spec:
	  ports:
	  - port: 443
	    targetPort: 443
	  selector:
	    app: sidecar-injector
接下来，部署以上内容 `Deployment` 和 `Service` 。部署后验证 `sidecar injectorwebhook` 服务器是否正在运行

	[root@mstnode kube-mutating-webhook-tutorial]# kubectl create -f ./deployment/deployment.yaml
	deployment "sidecar-injector-webhook-deployment" created
	[root@mstnode kube-mutating-webhook-tutorial]# kubectl create -f ./deployment/service.yaml
	service "sidecar-injector-webhook-svc" created
	[root@mstnode kube-mutating-webhook-tutorial]# kubectl get deployment
	NAME                                  DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
	sidecar-injector-webhook-deployment   1         1         1            1           2m
	[root@mstnode kube-mutating-webhook-tutorial]# kubectl get pod
	NAME                                                  READY     STATUS    RESTARTS   AGE
	sidecar-injector-webhook-deployment-bbb689d69-fdbgj   1/1       Running   0          3m
	
### 配置 webhook 准入控制器
`MutatingWebhookConfiguration` 指定启用哪些 webhook 准入服务器，以及哪些资源需要准入服务器。建议首先部署 webhook 准入服务器，并确保它在创建 `MutatingWebhookConfiguration` 之前正常工作。否则，可以设置根据 `failurePolicy` 来进行无条件接受或者拒绝。

`MutatingWebhookConfiguration` 使用以下内容创建清单：

	apiVersion: admissionregistration.k8s.io/v1beta1
	kind: MutatingWebhookConfiguration
	metadata:
	  name: sidecar-injector-webhook-cfg
	  labels:
	    app: sidecar-injector
	webhooks:
	  - name: sidecar-injector.morven.me # webhook 的名称。应该是完全限定的。通过提供链对多个变更 webhook 进行排序。
	    clientConfig: # 介绍如何连接到 webhook 准入服务器和TLS证书。在例子中，指定 `sidecar-injector-webhook-svc` 服务。
	      service:
	        name: sidecar-injector-webhook-svc #
	        namespace: default
	        path: "/mutate"
	      caBundle: ${CA_BUNDLE}
	    rules: # 指定 webhook 服务器处理的资源和操作。在例子中，只拦截创建 pod 的请求。
	      - operations: [ "CREATE" ]
	        apiGroups: [""]
	        apiVersions: ["v1"]
	        resources: ["pods"]
	    namespaceSelector: #根据该对象的名称空间是否与选择器匹配，决定是否在对象上发送对 webhook 服务器的准入请求。
	      matchLabels:
	        sidecar-injector: enabled

在部署之前 `MutatingWebhookConfiguration`，需要替换 `${CA_BUNDLE}` 成 apiserver 的默认值`caBundle`。使用脚本 `webhook-patch-ca-bundle.sh` 来自动化这个过程：

	#!/bin/bash
	set -o errexit
	set -o nounset
	set -o pipefail
	
	ROOT=$(cd $(dirname $0)/../../; pwd)
	
	export CA_BUNDLE=$(kubectl get configmap -n kube-system extension-apiserver-authentication -o=jsonpath='{.data.client-ca-file}' | base64 | tr -d '\n')
	
	if command -v envsubst >/dev/null 2>&1; then
	    envsubst
	else
	    sed -e "s|\${CA_BUNDLE}|${CA_BUNDLE}|g"
	fi
然后执行

	[root@mstnode kube-mutating-webhook-tutorial]# cat ./deployment/mutatingwebhook.yaml |\
	>   ./deployment/webhook-patch-ca-bundle.sh >\
	>   ./deployment/mutatingwebhook-ca-bundle.yaml
最后部署 `MutatingWebhookConfiguration`

	[root@mstnode kube-mutating-webhook-tutorial]# kubectl create -f ./deployment/mutatingwebhook-ca-bundle.yaml
	mutatingwebhookconfiguration "sidecar-injector-webhook-cfg" created
	[root@mstnode kube-mutating-webhook-tutorial]# kubectl get mutatingwebhookconfiguration
	NAME                           AGE
	sidecar-injector-webhook-cfg   11s
### 验证和故障排除
是时候验证边车注入器是否按预期工作。通常在 `default` 命名空间中创建和部署睡眠应用程序，以查看是否可以注入边车。

	[root@mstnode kube-mutating-webhook-tutorial]# cat <<EOF | kubectl create -f -
	> apiVersion: extensions/v1beta1
	> kind: Deployment
	> metadata:
	>   name: sleep
	> spec:
	>   replicas: 1
	>   template:
	>     metadata:
	>       annotations:
	>         sidecar-injector-webhook.morven.me/inject: "true"
	>       labels:
	>         app: sleep
	>     spec:
	>       containers:
	>       - name: sleep
	>         image: tutum/curl
	>         command: ["/bin/sleep","infinity"]
	>         imagePullPolicy: IfNotPresent
	> EOF
	deployment "sleep" created
密切关注，`spec.template.metadata.annotations` 因为添加了新注释

	sidecar-injector-webhook.morven.me/inject: "true"
边车注入具有一些逻辑，用于在注入边车容器和容积之前检查上述注释的存在。在构建 sidecar 注入器容器之前，可以自由删除逻辑或自定义逻辑。

检查 `deployment` 和 `pod`：

	[root@mstnode kube-mutating-webhook-tutorial]# kubectl get deployment
	NAME                                  DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
	sidecar-injector-webhook-deployment   1         1         1            1           18m
	sleep                                 1         1         1            1           58s
	[root@mstnode kube-mutating-webhook-tutorial]# kubectl get pod
	NAME                                                  READY     STATUS    RESTARTS   AGE
	sidecar-injector-webhook-deployment-bbb689d69-fdbgj   1/1       Running   0          18m
	sleep-6d79d8dc54-r66vz                                1/1       Running   0          1m	
那里没有？检查一下边车注射器日志。

	[root@mstnode kube-mutating-webhook-tutorial]# kubectl logs -f sidecar-injector-webhook-deployment-bbb689d69-fdbgj
	I0314 08:48:15.140858       1 webhook.go:88] New configuration: sha256sum 21669464280f76170b88241fd79ecbca3dcebaec5c152a4a9a3e921ff742157f
找不到任何表明 webhook 服务器收到录取请求的日志，似乎请求还没有发送到 `sidecar injector` webhook 服务器。因此，问题可能是由配置引起的 `MutatingWebhookConfiguration`。进行仔细检查，`MutatingWebhookConfiguration` 我们发现以下内容：

	  namespaceSelector:
	      matchLabels:
	        sidecar-injector: enabled
### 控制边车注射器 namespaceSelector
`MutatingWebhookConfiguration` 的配置已经设置了 `namespaceSelector` ，这意味着只有匹配选择器的命名空间中的资源才会被发送到 webhook 服务器。因此，需要标记 default 命名空间 `sidecar-injector=enabled`：

	[root@mstnode kube-mutating-webhook-tutorial]# kubectl label namespace default sidecar-injector=enabled
	namespace "default" labeled
	[root@mstnode kube-mutating-webhook-tutorial]# kubectl get namespace -L sidecar-injector
	NAME          STATUS    AGE       sidecar-injector
	default       Active    1d        enabled
	kube-public   Active    1d
	kube-system   Active    1d
我们已经配置了 `MutatingWebhookConfiguration` 在 pod 创建时发生的 边车注入。杀死正在运行的pod 并验证是否使用注入的边车创建了新 pod。	

	[root@mstnode kube-mutating-webhook-tutorial]# kubectl delete pod sleep-6d79d8dc54-r66vz
	pod "sleep-6d79d8dc54-r66vz" deleted
	[root@mstnode kube-mutating-webhook-tutorial]# kubectl get pods
	NAME                                                  READY     STATUS              RESTARTS   AGE
	sidecar-injector-webhook-deployment-bbb689d69-fdbgj   1/1       Running             0          29m
	sleep-6d79d8dc54-b8ztx                                0/2       ContainerCreating   0          3s
	sleep-6d79d8dc54-r66vz                                1/1       Terminating         0          11m
	[root@mstnode kube-mutating-webhook-tutorial]# kubectl get pod sleep-6d79d8dc54-b8ztx -o yaml
	apiVersion: v1
	kind: Pod
	metadata:
	  annotations:
	    kubernetes.io/psp: default
	    sidecar-injector-webhook.morven.me/inject: "true"
	    sidecar-injector-webhook.morven.me/status: injected
	  labels:
	    app: sleep
	    pod-template-hash: "2835848710"
	  name: sleep-6d79d8dc54-b8ztx
	  namespace: default
	spec:
	  containers:
	  - command:
	    - /bin/sleep
	    - infinity
	    image: tutum/curl
	    imagePullPolicy: IfNotPresent
	    name: sleep
	    resources: {}
	    volumeMounts:
	    - mountPath: /var/run/secrets/kubernetes.io/serviceaccount
	      name: default-token-d7t2r
	      readOnly: true
	  - image: nginx:1.12.2
	    imagePullPolicy: IfNotPresent
	    name: sidecar-nginx
	    ports:
	    - containerPort: 80
	      protocol: TCP
	    resources: {}
	    terminationMessagePath: /dev/termination-log
	    terminationMessagePolicy: File
	    volumeMounts:
	    - mountPath: /etc/nginx
	      name: nginx-conf
	  volumes:
	  - name: default-token-d7t2r
	    secret:
	      defaultMode: 420
	      secretName: default-token-d7t2r
	  - configMap:
	      defaultMode: 420
	      name: nginx-configmap
	    name: nginx-conf
	...
可以看到 sidecar 容器和卷已成功注入睡眠应用程序。到现在为止，`MutatingAdmissionWebhook` 已经开始使用边车注射器了。随着 `namespaceSelector` 可以很容易地控制指定命名空间的 pod 是否会被注入与否。

但是存在一个问题，通过上述配置，`default` 命名空间中的所有 `pod` 都会注入一个 sidecar，这在某些情况下可能不会出现。

### 控制边车注射器 annotation
`MutatingAdmissionWebhook` 由于灵活性 ，应该可以轻松地定制变更逻辑，以使用指定的注释过滤资源。还记得 `sidecar-injector-webhook.morven.me/inject:"true"` 上面提到的注释吗？它可以用作边车注射器的额外控制。我已经在 webhook 服务器中编写了一些代码，以便在没有注释的情况下跳过注入 pod。

试一试吧。在这种情况下，创建了另一个没有 `sidecar-injector-webhook.morven.me/inject: "true"` 注释的睡眠应用程序。

	[root@mstnode kube-mutating-webhook-tutorial]# kubectl delete deployment sleep
	deployment "sleep" deleted
	[root@mstnode kube-mutating-webhook-tutorial]# cat <<EOF | kubectl create -f -
	apiVersion: extensions/v1beta1
	> kind: Deployment
	> metadata:
	>   name: sleep
	> spec:
	>   replicas: 1
	>   template:
	>     metadata:
	>       labels:
	>         app: sleep
	>     spec:
	>       containers:
	>       - name: sleep
	>         image: tutum/curl
	>         command: ["/bin/sleep","infinity"]
	>         imagePullPolicy: IfNotPresent
	> EOF
	deployment "sleep" created
然后验证边车注射器是否跳过了 pod

	[root@mstnode kube-mutating-webhook-tutorial]# kubectl get deployment
	NAME                                  DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
	sidecar-injector-webhook-deployment   1         1         1            1           45m
	sleep                                 1         1         1            1           17s
	[root@mstnode kube-mutating-webhook-tutorial]# kubectl get pod
	NAME                                                  READY     STATUS        RESTARTS   AGE
	sidecar-injector-webhook-deployment-bbb689d69-fdbgj   1/1       Running       0          45m
	sleep-776b7bcdcd-4bz58                                1/1       Running       0          21s
输出显示睡眠应用程序仅包含一个容器，没有额外的容器和注入的卷。然后修改睡眠部署以添加额外的注释并验证它将在重新创建后注入

	[root@mstnode kube-mutating-webhook-tutorial]# kubectl patch deployment sleep -p '{"spec":{"template":{"metadata":{"annotations":{"sidecar-injector-webhook.morven.me/inject": "true"}}}}}'
	deployment "sleep" patched
	[root@mstnode kube-mutating-webhook-tutorial]# kubectl delete pod sleep-776b7bcdcd-4bz58
	pod "sleep-776b7bcdcd-4bz58" deleted
	[root@mstnode kube-mutating-webhook-tutorial]# kubectl get pods
	NAME                                                  READY     STATUS              RESTARTS   AGE
	sidecar-injector-webhook-deployment-bbb689d69-fdbgj   1/1       Running             0          49m
	sleep-3e42ff9e6c-6f87b                                0/2       ContainerCreating   0          18s
	sleep-776b7bcdcd-4bz58                                1/1       Terminating         0          3m
正如所料，pod 已注入额外的边车容器。现在，开始使用边车注射器，`mutatingAdmissionWebhook` 通过额外的 `namespaceSelector` 和 `annotation` 来控制规则的粗细粒度。

### 结论
`MutatingAdmissionWebhook` 通过新的政策控制，资源变更，扩展 Kubernetes 是最简单的方法之一......

此功能将支持更多工作负载并支持更多生态系统组件，包括[Istio](https://github.com/istio/istio) 服务网状平台。从 Istio 0.5.0 开始，Istio 已重构以支持其自动注入代码并进行`MutatingAdmissionWebhook` 替换 `initializers`。
## 翻译
[Diving into Kubernetes MutatingAdmissionWebhook](https://github.com/morvencao/kube-mutating-webhook-tutorial/blob/master/medium-article.md#how-mutatingadmissionwebhook-works)
	