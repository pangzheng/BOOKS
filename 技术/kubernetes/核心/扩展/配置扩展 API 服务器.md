# 配置扩展 API 服务器
建立一个扩展 API 服务器作用于汇聚层（aggregation layer），使得 Kubernetes apiserver 能扩展额外的非 Kubernetes 核心 API。

## 在此之前
- 需要有一个正在运行的 Kubernetes 集群
- 必须[配置汇聚层](https://k8smeetup.github.io/docs/tasks/access-kubernetes-api/configure-aggregation-layer/)，并启用 apiserver 标记。

## 配置一个作用于汇聚层的扩展 api-server
以下步骤高度概括地描述了如何建立扩展 apiserver。对于可实施的具体例子，您可以查阅 Kubernetes repo 中的 [apiserver 例子](https://github.com/kubernetes/sample-apiserver/blob/master/README.md)。

或者，可以使用现有的第三方方案，比如 [apiserver-builder](https://github.com/kubernetes/sample-apiserver/blob/master/README.md)，能够实现下面步骤的自动化。

1. 确保 APIServer API 是启用的（检查 --runtime-config）。除非特意关闭，该配置默认启用。
2. 需要创建 RBAC 规则，以让您能有权限添加 APIserver 对象，或者让集群管理员创建一个。(由于 API 扩展对于整个集群生效，所以不推荐在生产集群中对 API 扩展进行 测试/开发/debug。)
3. 创建一个 Kubernetes namespace 用于运行扩展 api-service。
4. 创建／获取一张 CA 证书用于对 api-server 的 HTTPS 的服务器证书进行签名。
5. 创建服务器证书／秘钥用于 api-server 的 HTTPS 服务。这个证书应该由上面提及的 CA 签名。它也应该包含含有 Kube DNS 名的 CN。这来自于 Kubernetes 服务并且是 ..svc 的形式。
6. 在这个 namespace 中使用服务器证书／秘钥创建 Kubernetes secret。
7. 为扩展 api-server 创建 Kubernetes deployment，并确认使用卷的方式加载 secret。它应改包含一个关于您扩展 api-server 镜像的参考(reference)。deployment 也应该存在之前创建的 namespace 中。
8. 确保 api-server 通过卷加载这些证书，并用于 HTTPS 握手。
9. 在这个 namespace 中创建一个 Kubernetes service account。
10. 创建一个 Kubernetes cluster role 用于允许对您对资源进行操作。
11. 使用在这个 namespace 中默认的 service account 为您刚刚创建的 cluster role
12. 创建一个 Kubernetes cluster rolebinding。
13. 创建一个 Kubernetes apiservice。上面提到的 CA 证书应该用 base64 加密，去除换行符并作为 spec.caBundle 在 apiservice 中使用。这不应该在 namespace 中。
使用 kubectl 获得您的资源。它应该返回 “No resources found.”，意味着一切工作正常，但是您目前还没有创建那个类型的资源对象。
