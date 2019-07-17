# Kubernetes 新概念 “Initializers”解析
Kubernetes如今能大展拳脚的原因有二：

- google 技术社区的无限优势;
- 源于 Kubernetes API 的灵活性，以及​​能轻而易举地在其上编写自定义扩展或者插件。其中一个新的概念：Initalizers 能在实际创建之前动态修改 Kubernetes 资源，并且是可插拔方式。Initalizers 在 Kubernetes 1.7 中作为 Alpha 功能。

举个例子来说，在 GCE 中使用 Intializers 来扩展 Kubernetes 的基础功能，也可以根据需求实现新的 initializers 设置。

到目前为止，Kubernetes 只有在插件创建之前用[准入控制插件](https://kubernetes.io/docs/admin/admission-controllers/)来拦截资源。

例如，可以使用一个准入插件来强制让所有容器镜像来自特定的注册表，并组织其他镜像被部署在 pod 中，有相当多的准入控制器提供的功能，如执行限制，应用创建检查，并为缺少的字段设置默认值。

但准入控制器也存在如下问题：

- 他们被编译在 Kubernetes：如果你寻找的目标缺失，你需要fork Kubernetes，并且写一个准入插件，并自己保持fork。
- 您需要通过将其名称传递给 kube-apiserver 的 "–" 入场控制标志来启用每个准入插件。在很多情况下，这意味着重新部署集群。
- 某些托管集群的供应商可能不允许自定义API服务器标志，因此可能无法启用源代码中可用的所有准入控制器。

目前有两种类型的插件：Initializers 和 webhook。Initializers 类似于准入控制器插件，可以在创建资源之前截取它。它们也与准入控制器插件不同，因为它们不是 Kubernetes源代码的一部分，也不是编译而成的。需要自己写一个控制器。

## 能用初始化器做什么？
当在创建 Kubernetes 对象之前截取它们时，存在着无穷的变数：可以以任何方式来变换对象，或防止对象被创建。

以下是对 Initializers 的一些想法，每个都在你的集群中实施一个特定的策略：

- 如果该容器开放 80 端口或具有特定的注释，则向该 pod 注入一个代理 sidecar 容器。
- 将带有测试证书的卷自动插入测试命名空间中的所有 pod。
- 如果一个隐藏密码少于20个字符（有可能是密码），则阻止其创建。

如果不打算修改对象并且拦截只读对象，[webhook](https://kubernetes.io/docs/admin/extensible-admission-controllers/#external-admission-webhooks) 可能会是一个更快和更精简的选择来获取有关对象的通知。可以查看[例子](https://github.com/caesarxuchao/example-webhook-admission-controller)来进行了解

上面列出的一些功能，例如注入 sidecar 容器或卷，也可以使用 [Pod Presets](https://kubernetes.io/docs/tasks/inject-data-application/podpreset/) 以牺牲灵活性来进行实现。如果可以使用，不必费心开发调度器

## initialization 剖析
1. 配置需要 initialization 的资源类型

	[InitializerConfiguration](https://kubernetes.io/docs/admin/extensible-admission-controllers/#configure-initializers-on-the-fly) 对象允许配置被 initializers 分配到的资源类型。

	举例子来说，可以进行创建，将 `myproxy` initializer 添加到类型 `apps/v1beta1.Deployment` 和 `v1.DeamonSet` 对象中。可以尽可能多地创建 `InitializerConfigurations`，它们将适用于所有命名空间。
- API 服务器将为新资源分配 initializers：

	当向 apiserver 提交 `Deployment` 对象时，将更新 Deployment 的 `metadata.initalizers.pending` 并在其中添加 `myproxy` 值。此字段显示当前分配给资源的initializers 。

	准确的说，不是由 apiserver 添加 initializers。有一个名为 `Initializer` 的准入控制插件，这使整个 initialization 流程成为可能。它通过将 `–admission-controller=Initializer` 标志添加到 kube-apiserver 来进行启用 。
- 编写一个控制器来检测资源情况

	开发并部署到集群的自定义控制器使用 Watch API 来对新的资源进行监听，捕获并进行所需的修改。
- 等待修改资源的轮次：

	一旦控制器通过 Watch API 拦截一个对象，它只能修改对象，如果它在初始化器列表`(metadata.initializers.pending[0])`的第一个元素上检查到它的名字的话。否则，这意味着它是一些其他 initializers 修改资源的轮次，而现在应该跳过修改。
- 完成修改, 生成下一个 initializer。

	完成对资源的修改后, 控制器应从对象的 `metadata.initializers.pending` 列表中删除其名称, 并将该对象保存回 API 服务器。
- 不存在更多 initializers 时, 资源准备好进行实现：

	当 Kubernetes API 服务器检测到该对象没有其他挂起的 initializers 时, 它会判定对象 “已initialized”。Kubernetes 调度程序和其他控制器可以检测到完全 initialized 的对象并加以利用。

	可以同时在群集上运行多个 initializers。这些自定义控制器中的每一个都将收到有关对资源 (如 Pods) 的修改的通知,  但他们会等待轮次来对对象进行修改，直到他们在列表中检测他们的名字。
	
## Initializers：在 Kubernetes API 中进行实现
我们来详细讨论一下它是如何在 Kubernetes API 中进行实现的：

社区已经开始考虑尽[早提](https://github.com/kubernetes/kubernetes/issues/3585)供钩子的形式来进行扩展。首先在[这里提出](https://github.com/kubernetes/kubernetes/pull/17305)，该功能看起来有点类似的实现方式。[后面的提案](https://github.com/kubernetes/community/blob/master/contributors/design-proposals/admission_control_extension.md#initializers)解释了该功能如何工作。如果想了解原理，请阅读[这里](https://github.com/kubernetes/kubernetes/pull/36721)

当一个 Pod 资源提交到 API 并被分配一个待定的 initializers 列表时, 直到 initialization 完成，它实际上都不会被安排。Kubernetes 调度程序, 这个所谓的调度程序其实是另一个控制器，检测在 API 服务器中显示的 pod，并分配给每个节点。[更多信息](https://jvns.ca/blog/2017/07/27/how-does-the-kubernetes-scheduler-work/)

那么, 为什么调度程序和其他控制器在初始化之前无法看到该对象, 即使该对象保存在 API 服务器 （和预计数据库中）, 并且对某些其他控制器 （即 initializers）可见？

答案是存在一个叫 `includeUninitialized` 的请求参数。此参数默认为 false, 因此 API 将会在 `WATCH or LIST` 中隐藏默认客户端（如 kubectl）和控制器 (如调度程序)中的初始化对象。所以开发的 initializers 必须设置 `?includeUninitialized=true` 查询参数才能监测这些对象。

Initializers 阻止创建请求。当创建对象的请求提交到 API 服务器时, 它不会马上返回, 并且请求会一直被阻止, 直到 initialization 完成。如果使用的是 kubectl, 并且对象在 uninitialized 状态下被阻止, 会在30秒后发现 kubectl 超时。

## 优点和缺点
[Admission Control 插件](https://kubernetes.io/docs/admin/admission-controllers/)被编译到 Kubernetes API 服务器中。要添加新插件，需要分叉 Kubernetes 源代码树并在顶部开发插件并继续维护分支。另一方面，[Initializers](https://kubernetes.io/docs/admin/extensible-admission-controllers/#initializers)是在Kubernetes源树之外开发的。可以使用 Kubernetes API 客户端轻松开发一个。它们就像任何其他工作负载一样在集群上运行，减少其他担心的事情。

- 初始化器非常灵活

	一旦在实际创建对象之前抓住了对象，天空就是极限。请注意，这种灵活性还可以轻松地产生问题。应该限制每个初始化程序执行一项任务，并且不要影响其他服务。

- 编写初始化程序很简单

	通过查看此[示例](https://github.com/kelseyhightower/kubernetes-initializer-tutorial)，该示例基于注释向 Pod 添加边车容器。它大约有200行Go代码，但它可以很好地自动执行任务。你现在可以继续自己开发一个。但是如何开发生产级初始化程序并在生产中运行它？这可能会更具挑战性。

- 初始化器的正常运行时间是一个大问题

	当初始化器脱机时，它仍将被分配以初始化新资源。除非初始化程序返回，否则这些资源将无限期地陷入“未初始化”状态。这可能会产生真实的现场影响：如果有 pod 的初始化程序，并且如果初始化程序在扩展事件期间脱机，则不会创建新的pod，这可能会导致自动扩展操作失败并导致停电。

- 现在还为时尚早

	在撰写本文时，初始化程序目前处于 Kubernetes v1.7中的alpha版本，并且它的目标是v1.8中的beta。您需要在群集中启用Alpha标志才能立即开始使用此功能。另请注意，我解释本文的许多内容可能不适用于该功能的稳定版本

## 开发自己的初始化程序
最简单的入门方法是使用Kelsey Hightower分叉这个[初始化程序示例](https://github.com/kelseyhightower/kubernetes-initializer-tutorial)，该程序是用Go编写的，并根据注释的存在为Side对象添加一个sidecar容器。

在[源代码](https://github.com/kelseyhightower/kubernetes-initializer-tutorial/blob/2645c93e05f281c251ae7d835c7bb93dfdded0b8/envoy-initializer/main.go)中，应该注意以下几点：List / Watch 函数指定 `IncludeUninitialized= true` 并定位所有名称空间，通知方及其重新同步周期，如何cloned/mutated API对象，以及如何使用 PATCH 执行更新

开发初始化程序时需要注意的事项：

- 确保初始化程序不会出现故障。

	之前提到过，能做的最好的事情就是确保有一个活跃的探测器，并对它进行监视/警告。如果所有 pod 的初始化程序，则尤其需要这样做。(否则初始化程序异常将会影响服务)

- 鸡与蛋的问题

	如果 有 pods/deployments 的初始化程序，部署初始化程序可能会因为无法初始化而阻止。需要手动指定待处理的初始值设定项列表以清空容器清单中的数组值（`[]`）。

- 初始化程序是正常的工作负载

	应该部署到与正常工作负载不同的命名空间。内置的`kube-system` 命名空间可能是托管初始化程序的好地方。

- 初始化程序可能会收到不完整的对象

	例如，尚未安排的 Pod 可能会丢失一些字段，例如 “nodeName”或“status”。应该使用此假设进行测试初始化​​程序。

- 初始化程序可以以不同的顺序应用

	每次对初始化程序列表进行不同的排序，实现应该可以正常工作。

- 确保初始化程序快速处理对象。初始化阻止创建资源的请求。

## 进一步阅读
- [Initializers documentation](https://kubernetes.io/docs/admin/extensible-admission-controllers/)
- [External Admission Controllers design proposal](https://github.com/kubernetes/community/blob/master/contributors/design-proposals/admission_control_extension.md)
- [Patch that added Initializers to Kubernetes](https://github.com/kubernetes/kubernetes/pull/36721)
- [Example Initializer implementation](https://github.com/kelseyhightower/kubernetes-initializer-tutorial/)

## 参考
[How Kubernetes Initializers work](https://ahmet.im/blog/initializers/)