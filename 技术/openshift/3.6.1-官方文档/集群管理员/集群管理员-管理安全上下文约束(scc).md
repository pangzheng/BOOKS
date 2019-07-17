# 集群管理员-管理安全上下文约束(scc)
安全上下文约束条件允许管理员限制 pod 权限。要了解更多信息，请查看 [security context constraints (SCCs)](https://docs.openshift.org/3.6/architecture/additional_concepts/authorization.html#security-context-constraints),可以在 CLI 设置实例的 SCC 作为正常的 API 对象进行管理。但必须拥有 `cluster-admin` 权限才可以管理 SCC。
## 列出安全上下文约束
获取最新的 scc 列表

	oc get scc
## 检查安全上下文约束对象
为了检查特定的 scc ，使用 `oc get；oc describe；oc export；oc edit`。例如检查`restricted` 的 scc：

	oc describe scc restricted	
为了在升级过程中保存定制的 scc，不要编辑除 `priority
, users, groups, labels, annotations` 之外的默认 SCC。
## 创建新的安全上下文约束
创建一个新的安全上下文

- 定义 scc 文件，可以是 json 或 yaml

```
kind: SecurityContextConstraints
apiVersion: v1
metadata:
  name: scc-admin
allowPrivilegedContainer: true
runAsUser:
  type: RunAsAny
seLinuxContext:
  type: RunAsAny
fsGroup:
  type: RunAsAny
supplementalGroups:
  type: RunAsAny
users:
- my-admin-user
groups:
- my-admin-group
```	
可选功能包含，可以通过使用所需的值设置 `requiredDropCapabilities` 字段来为 scc 添加丢弃功能。任何指定的功能将从容器中删除。例如要使用 `KILL, MKNOD, SYS_CHROOT` 创建一个 SCC ，需要使用 `drop` 功能，将以下内容添加到 SCC 对象。

```
requiredDropCapabilities:
- KILL
- MKNOD
- SYS_CHROOT
```

这个可以在 [Docker 文档](https://docs.docker.com/engine/reference/run/#runtime-privilege-and-linux-capabilities)。由于功能被传递给 docker ,可以使用特殊的 `ALL` 值来删除所有可能的功能。

- 创建 scc 策略

		oc create -f scc_admin.yaml
- 验证策略

		oc get scc scc-admin
		
## 删除 SCC
删除 scc 策略

	oc delete scc <scc_name>
如果删除了默认的 scc ，重新启动则会重新生成。
## 更新 scc
更新现有的 scc

	$ oc edit scc <scc_name>
为了在升级过程中保留自定义 scc，请勿在优先级、用户、组以外的默认 SCC 上编辑设置
## 更新默认 SCC
缺省情况下， master 服务会在启动时创建默认的 scc。要将 scc 重置为默认值或升级后将现有的 scc 更新为新的默认定义，做法:

- 删除任何想要重置的 scc，并通过重新启动 master 来重新创建。
- 使用 `oc adm policy reconcile-sccs` 命令

	`oc adm policy reconcile-sccs` 命令将所有 scc 策略设置为默认值，但保留任何其他用户、组、标签、注释以及可能已经设置的优先级。要查看哪些 scc 将被更改，可以运行没有选项的命令，或者使用 `-o  <format>` 来指定输出格式。在审查之后，建议备份现有的 scc，然后使用 `--confirm` 选项来保存数据。

- 如果想重置优先级和授权，请使用 `--additive-only=false`
- 如果在 scc 中拥有除优先级，用户，组，标签或者注释以外的自定义设置，则在协调时将会丢失这些设置。 	

## 样例场景
### 授予访问特权 scc 权限
在某些情况下，管理员可能希望允许管理员组访问权限之外的用户或组创建更多的特权的 pod ，为此可以:

- 确定想要访问 scc 的用户或组

	授权用户访问权限仅在用户直接创建 pod 时才有效。在大多数情况下，对于代表用户创建的 pod,由系统本身创建，应该提供一个 `serviceaccount`在这个账户下有相关的管理员进行操作。代表用户创建 pod 的资源，包括 `Deployments, StatefulSets, DaemonSets` 等
- 运行

		oc adm policy add-scc-to-user <scc_name> <user_name>
		oc adm policy add-scc-to-group <scc_name> <group_name>
		
		例如要允许 e2e 用户访问特权 scc
		
		oc adm policy add-scc-to-user privileged e2e-user
- 修改容器的 `SecurityContext` 以请求特权模式。			
### 授权服务账户访问特权 scc 权限
首先创建一个 `serviceaccount`。例如在项目 `myproject` 中创建账户 `mysvcacct`

		oc create serviceaccount mysvcacct -n myproject
然后将该服务账户添加到特权的 scc

		oc adm policy add-scc-to-user privileged system:serviceaccount:myproject:mysvcacct
然后，确保代表服务账户正在创建资源。为此请将 `spec.serviceAccountName` 字段设置为服务账户名称。将服务账户名称留空将导致正在使用 `default` 服务账户。确保至少有一个容器的容器在安全上下文中请求特权模式。
### 使镜像在 Dockerfile 中与 USER 在一起运行
为了放松集群中的安全性，以便镜像不会被强制用预先分配的 UID 运行，授权每个人访问特权 SCC 权限。

		oc adm policy add-scc-to-group anyuid system:authenticated
如果没有在 Dockerfile 中指明 USER，则允许镜像作为 Root 运行。
### 启动需要 root 的容器镜像
一些容器镜像(如 postgres 和 redis) 需要 root 访问权限，并对卷拥有需求。对于这些镜像，将服务账户添加到 `anyuid` SCC。

		oc adm policy add-scc-to-user anyuid system:serviceaccount:myproject:mysvcacct
### 在 registry 使用 `--mount-host`
建议使用 `PersistentVolume 和 PersistentVolumeClaim`对象的持久化存储用于 registry 部署。如果正在测试，并且希望将 `oadm registry` 命令与 `--mount-host` 选项一起使用，则必须首先为注册表创建一个新的 serviceaccount ，并将其添加到特权 scc。详细可以查看[管理员指南](https://docs.openshift.org/3.6/install_config/registry/deploy_registry_existing_clusters.html#storage-for-the-registry)
### 提供额外的功能
在某些情况下，镜像可能需要 docker 没有提供的功能。可能提供在 pod 规范中请求附加功能，这些将根据 scc 进行验证。这也许镜像以提升的能力运行，并且只有在必要时才使用。不应该编辑默认的受限 scc 以启动其他功能。 

当与非 root 用户一起使用时，还必须确保需要附加功能的文件被授予使用 `setcap` 命令的功能。例如，在镜像的 dockerfile 中：				

	setcap cap_net_raw,cap_net_admin+p /usr/bin/ping
此外，如果在 docker 中默认提供一个功能，则不需要修改该功能模块规范来请求它。例如，默认情况提供了 `NET_RAW`，并且应该已经在 `ping` 上设置了功能，因此不需要执行特殊步骤来运行 `ping`.

提供附加功能

- 创建一个新的 scc
- 使用 `allowedCapabilities` 字段添加允许的功能
- 在创建 pod 时，在 `securityContext.capabilities.add` 字段中添加请求该功能。

### 修改集群默认行为
修改集群，使其不预先分配 UID，允许容器以任何身份用户运行，并组织特权容器:

- 编辑 `restricted `  SCC

		oc edit scc restricted
- 修改 `runAsUser.Type` 到  `RunAsAny`
- 修改 `allowPrivilegedContainer` 到 `false`
- 记录修改

要修改集群，以便它不会预先分配 UID 且不允许容器以 root 身份运行:

- 编辑 `restricted ` SCC
- 修改 `runAsUser.Type` 到  `MustRunAsNonRoot`
- 记录修改

### 使用 `hostPath` 卷插件
允许 pods 使用 `hostPath` 卷插件。

- 编辑 `restricted ` SCC
	
		oc edit scc restricted
- 添加 `allowHostDirVolumePlugin: true`
- 记录修改

### 确保 `Admission` 受限使用特定的 scc
可以通过设置 scc 的`Priority `优先级字段来控制 `Admission` 中的 SCC 的排序优先级。有关更多信息可以查看 [scc 优先级部分](https://docs.openshift.org/3.6/architecture/additional_concepts/authorization.html#scc-prioritization)

### 添加 SCC 添加到用户、组或项目
在将 scc 添加到用户或组前，可以使用 `scc-review` 选项来检查用户或组是否可以创建 pod。[更多信息](https://docs.openshift.org/3.6/dev_guide/authorization.html#dev-guide-authorization)

SCC 不能直接授予项目。替换方法时，将 SCC 增加到服务账户中，并将 pod 上指定服务账户名称，当未知定的时候，将使用默认的服务账户运行。

- 添加一个 SCC 用户 

		oc adm policy add-scc-to-user <scc_name> <user_name>
- 添加一个 SCC 的服务账户

		oc adm policy add-scc-to-user <scc_name> \
    		system:serviceaccount:<serviceaccount_namespace>:<serviceaccount_name>
- 如果当前正在服务账户所属得项目中，可以使用 `-z` 标示并指定 `<serviceaccount_name>`

		oc adm policy add-scc-to-user <scc_name> -z <serviceaccount_name>
- 添加一个 scc 到一个组

		oc adm policy add-scc-to-group <scc_name> <group_name>
- 添加一个 SCC 到一个项目中的所有的服务账户

		oc adm policy add-scc-to-group <scc_name> \
			system:serviceaccounts:<serviceaccount_namespace>		