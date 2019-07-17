# 使用 minishift
# 基本用法
## 概观
使用 Minishift 时，将与以下组件进行交互：

- Minishift虚拟机（VM）
- VM 上运行的 Docker 守护程序
- 在 Docker 守护程序上运行的 OpenShift 集群

所述 Minishift 架构图概述这些组件。minishift 二进制的，放置在 `PATH` 便于执行，是用来 `start`，`stop` 和 `delete` 所述 Minishift VM。VM 本身是从[可插拔的实时 ISO 映像](https://docs.okd.io/latest/minishift/using/choosing-iso-image.html) 引导的。

例如 [docker-env](https://docs.okd.io/latest/minishift/using/docker-daemon.html)，一些 Minishift 命令与 Docker 守护程序交互，而其他 [openshift](https://docs.okd.io/latest/minishift/command-ref/minishift_openshift.html) 命令与 OpenShift 集群通信，例如命令。

一旦 OpenShift 集群启动并运行，就可以使用 `oc` 二进制文件与它进行交互。Minishift 缓存此二进制文件 `$MINISHIFT_HOME`（默认为 〜/.minishift）. [minishift oc-env](https://docs.okd.io/latest/minishift/openshift/openshift-client-binary.html#openshift-client-binary-overview) 是一种简单的方法来将 `oc` 二进制文件添加到操作系统的 `PATH` 中。

有关使用 Minishift 管理本地 OpenShift 集群的更多详细信息，请参阅与 [OpenShift交互](https://docs.okd.io/latest/minishift/openshift/index.html) 部分。

## 架构图
![](./pic/minishift1.png)
### Minishift生命周期管理
- Minishift 启动命令

	该 [minishift start](https://docs.okd.io/latest/minishift/command-ref/minishift_start.html) 命令创建并配置 Minishift VM，并在 Minishift VM 中配置本地单节点 OpenShift 集群。

	该命令还将 `oc` 二进制文件复制到主机，以便可以通过 `oc` 命令行工具或 Web 控制台与 OpenShift 进行交互，可以通过命令输出中提供的URL进行访问 `minishift start`。
- Minishift 停止命令

	该 [minishift stop](https://docs.okd.io/latest/minishift/command-ref/minishift_stop.html) 命令将停止 OpenShift 集群并关闭 Minishift VM，但保留 OpenShift集群状态。

	再次启动 Minishift 将恢复 OpenShift 群集，允许您从上一个会话继续工作。但是，您必须输入与原始启动命令中使用的参数相同的参数。
- Minishift 删除命令(这条指令慎用，因为国内不好抓数据)

	该 [minishift delete](https://docs.okd.io/latest/minishift/command-ref/minishift_delete.html) 命令删除 OpenShift 集群，并关闭并删除 Minishift VM。不保留任何数据或状态。

### 运行时选项
可以通过标志，环境变量和持久配置选项来控制 Minishift 的运行时行为。应用以下优先顺序来控制 Minishift 的行为。列表中的每个操作都优先于其下面的操作：

- 使用 [Flags](https://docs.okd.io/latest/minishift/using/basic-usage.html#flags) 部分中指定的命令行标志。
- 按照 [环境变量](https://docs.okd.io/latest/minishift/using/basic-usage.html#environment-variables) 部分中的说明设置环境变量。
- 使用[持久配置](https://docs.okd.io/latest/minishift/using/basic-usage.html#persistent-configuration)部分中描述的持久配置选项。
- 接受 Minishift 定义的默认值。

### Flags
可以使用命令行标志与 Minishift 指定选项并指导其行为。这具有最高优先级。尽管不同的命令可能有不同的标志，但几乎所有命令都有标志。命令的一些常用命令行标志 `minishift start` 是 `cpus`，`memory`或`vm-driver`。
### 环境变量
Minishift 允许您指定通常通过环境变量使用的命令行标志。为此，请将以下规则应用于要设置为环境变量的标志。

- 应用 `MINISHIFT_` 作为前缀，要设置为一个环境变量的标志。例如，命令的 `vm-driver` 标志 [minishift start](https://docs.okd.io/latest/minishift/command-ref/minishift_start.html) 变为 `MINISHIFT_vm-driver`。
- 使用大写字符作为标志，因此 `MINISHIFT_vm-driver` 在上面的示例中变为 `MINISHIFT_VM-DRIVER`。
- 替换`-`为`_`，所以 `MINISHIFT_VM-DRIVER` 成为 `MINISHIFT_VM_DRIVER`。

环境变量可用于替换任何 Minishift 命令的任何选项。一个常见的例子是要使用的 ISO 的URL。通常，使用命令的 `iso-url` 标志指定它 `minishift start`。应用上述规则，还可以通过将环境变量设置为指定此 `URL MINISHIFT_ISO_URL`。

您还可以使用 `MINISHIFT_HOME` 环境变量为 Minishift 选择不同的主目录。默认情况下，Minishift 将所有运行时状态放入 `~/.minishift`。此环境变量目前是实验性的，语义可能在将来的版本中发生变化

### 持久配置
使用持久性配置允许在不指定实际命令行标志的情况下控制 Minishift 行为，类似于使用环境变量的方式。还可以使用 `--global` flag 定义全局配置。全局配置适用于所有配置文件。

Minishift在 `$MINISHIFT_HOME/config/config.json` 中维护一个配置文件。此文件可用于为各个配置文件持久设置常用命令行标志。{全局配置文件维护在 `$MINISHIFT_HOME/config/config.json`。

- 持久配置只能应用于 [minishift config](https://docs.okd.io/latest/minishift/command-ref/minishift_config.html) 子命令概要中列出的受支持配置选项集。这与环境变量不同，环境变量可用于替换任何命令的任何选项。
- 默认情况下，minishift 持久启动标志到持久配置，禁用它，使用 `minishift config set save-start-flags false`。

#### 设置持久配置值
更改持久配置选项的最简单方法是使用 [minishift config set](https://docs.okd.io/latest/minishift/command-ref/minishift_config_set.html) 子命令。例如：

	# 设置默认内存 4096 MB
	$ minishift config set memory 4096
在所有配置文件中设置持久配置选项的最简单方法是使用 `minishift config set --global` 子命令。例如，可以为每个配置文件将默认内存设置为8192 MB，如下所示：

	$ minishift config set --global memory 8192
每个命令调用可以多次使用的标志，如 `docker-env` 或者 `insecure-registry` ，在与 `config set` 命令一起使用时需要以逗号分隔。例如，从CLI中，您可以 `insecure-registry` 像这样使用

	$ minishift start --insecure-registry hub.foo.com --insecure-registry hub.bar.com
如果要在持久性配置中配置相同的注册表，则应运行
	
	$ minishift config set insecure-registry hub.foo.com,hub.bar.com
要查看所有持久配置值，可以使用 [minishift config view](https://docs.okd.io/latest/minishift/command-ref/minishift_config_view.html) 子命令

	$ minishift config view
	- memory: 4096			
要查看全局配置文件中的所有持久性配置值，可以使用 [minishift config view --global](https://docs.okd.io/latest/minishift/command-ref/minishift_config_view.html) 子命令：

	$ minishift config view --global
	- memory: 8192     
可以使用 [minishift config get](https://docs.okd.io/latest/minishift/command-ref/minishift_config_get.html) 子命令显示单个值

	$ minishift config get memory
	4096
#### 取消设置持久配置值
要删除持久配置选项，可以使用 [minishift config unset](https://docs.okd.io/latest/minishift/command-ref/minishift_config_unset.html) 子命令。例如：

	$ minishift config unset memory
	
	
用户定义值的优先级如下：

- 命令行标志
- 环境变量
- 特定于实例的配置
- 全局配置

### 持续卷
作为 OpenShift 群集配置的一部分，将为 OpenShift 群集创建 100 个[持久卷](https://docs.okd.io/latest/dev_guide/persistent_volumes.html)。这允许应用程序进行[持久卷声明](https://docs.okd.io/latest/dev_guide/persistent_volumes.html#persistent-volumes-claims-as-volumes-in-pods)。持久数据的位置 `host-pv-dir` 在 [minishift start](https://docs.okd.io/latest/minishift/command-ref/minishift_start.html) 命令的标志中确定，默认为 Minishift VM 上的 `/var/lib/minishift/openshift.local.pv`。
### HTTP/HTTPS代理
如果您位于 HTTP/HTTPS 代理之后，则需要提供代理选项以允许 Docker 和 OpenShift 正常工作。为此，请在期间传递所需的标志 `minishift start`。

例如：
	
	$ minishift start --http-proxy http://YOURPROXY:PORT --https-proxy https://YOURPROXY:PORT
在经过身份验证的代理环境中，`proxy_user` 并且 `proxy_password` 必须是代理URI的一部分

	 $ minishift start --http-proxy http：// <proxy_username>：<proxy_password> @YOURPROXY：PORT \
	                   --https-proxy https：// <proxy_username>：<proxy_password> @YOURPROXY：PORT

还可以使用该 `--no-proxy` 标志指定不应代理的以逗号分隔的主机列表。

使用代理选项将透明地配置 Docker 守护程序以及 OpenShift 以使用指定的代理。

- `minishift start` 尊重环境变量 `HTTP_PROXY`，`HTTPS_PROXY`和`NO_PROXY`。环境变量可以小写或大写形式指定。小写变量优先于大写变量。如果设置了这些变量，则 `minishift start` 除非被相应的命令行标志明确覆盖，否则它们将被隐式使用。
- Minishift 不会转义代理的环境变量中的特殊字符，因此用户需要手动转义特殊字符才能使其正常工作。

### 联网
Minishift VM 通过主机系统向主机系统公开，该 IP 地址可通过该 [minishift ip](https://docs.okd.io/latest/minishift/command-ref/minishift_ip.html) 命令获得。
### 使用SSH连接到Minishift VM
- 可以使用该 [minishift ssh](https://docs.okd.io/latest/minishift/command-ref/minishift_ssh.html) 命令与 Minishift VM 进行交互。
- 可以在 `minishift ssh` 没有子命令的情况下运行以打开交互式 shell 并在 Minishift VM 上运行命令，就像使用SSH在任何远程计算机上以交互方式运行命令一样。
- 还可以 `minishift ssh` 使用子命令将子命令直接发送到 Minishift VM 并将结果返回到本地 shell。例如：

		$ minishift ssh -- docker ps
		CONTAINER    IMAGE                   COMMAND                CREATED        STATUS        NAMES
		71fe8ff16548 openshift/origin:v1.5.1 "/usr/bin/openshift s" 4 minutes ago  Up 4 minutes  origin

# Minishift 配置文件
配置文件是 Minishift VM 的一个实例及其所有配置和状态。配置文件功能允许创建和管理这些孤立的 Minishift 实例。

每个 Minishift 配置文件都是使用自己的配置（内存，CPU，磁盘大小，附加组件等）创建的，并且独立于其他配置文件。如果要确保某些配置设置，请参阅[环境变量](https://docs.okd.io/latest/minishift/using/basic-usage.html#environment-variables)的使用。 例如:`cpus` 或 `memory` 

除非使用全局 `--profile` 标志，否则活动配置文件是执行全局命令的配置文件。 可以使用 [minishift profile list](https://docs.okd.io/latest/minishift/command-ref/minishift_profile_list.html) 命令确定活动配置文件。 可以使用 `--profile` 标志对非活动配置文件执行单个命令，例如 `minishift --profile profile-demo console`，以打开指定配置文件 `profile-demo` 的 OpenShift 控制台。

在顶部 `--profile` 的标志，有[命令](https://docs.okd.io/latest/minishift/command-ref/minishift_profile.html)对[列表](https://docs.okd.io/latest/minishift/command-ref/minishift_profile_list.html)，[删除](https://docs.okd.io/latest/minishift/command-ref/minishift_profile_delete.html)和[设置](https://docs.okd.io/latest/minishift/command-ref/minishift_profile_set.html)活动的配置文件。

尽管配置文件彼此独立，但它们共享 ISO，`oc` 二进制文件和容器镜像的相同缓存。 `minishift delete --clear-cache` 因此会影响所有配置文件。我们建议谨慎使用 `--clear-cache` 。
## 创建配置文件
有两种方法可以创建新的配置文件。(配置文件名称只能包含字母数字字符。允许使用连字符（-）作为分隔符。)
### 使用 `--profile` FLAG
当运行 Minishift 的 `start` 使用 `--profile` 标志命令时，如果配置文件不存在，则创建配置文件，例如：

	$ minishift --profile profile-demo start
	-- Checking if requested hypervisor 'xhyve' is supported on this platform ... OK
	-- Checking if xhyve driver is installed ...
	   Driver is available at /usr/local/bin/docker-machine-driver-xhyve
	   Checking for setuid bit ... OK
	-- Checking the ISO URL ... OK
	-- Starting local OpenShift cluster using 'xhyve' hypervisor ...
	-- Minishift VM will be configured with ...
	   Memory:    2 GB
	   vCPUs :    2
	   Disk size: 20 GB
另见[配置文件配置的工作流程](https://docs.okd.io/latest/minishift/using/profiles.html#example-workflow-profile-config)。

通过成功启动 Minishift 实例时，配置文件自动成为 `minishift start` 活动的配置文件 。
### 使用 `profile set` 命令
创建配置文件的另一个选项是使用该 `profile set` 命令。如果指定的配置文件不存在，则隐式创建：

	$ minishift profile set demo
	Profile 'demo' set as active profile	   
	
默认配置文件是 `minishift`。它将默认存在，不需要创建。
### 列出配置文件
可以使用 `minishift profile list` 命令列出所有现有配置文件。您还可以在输出中看到突出显示的活动配置文件。

	$ minishift profile list
	- minishift     Running     	(Active)
	- profile-demo  Does Not Exist
### 切换配置文件
要切换之间配置文件使用的 `minishift profile set` 命令：

	$ minishift profile set profile-demo
	Profile 'profile-demo' set as active profile
任何时候只能有一个配置文件处于活动状态。
### 删除配置文件
要删除配置文件，请运行

	$ minishift profile delete profile-demo
	You are deleting the active profile. It will remove the VM and all related artifacts. Do you want to continue [y/N]?: y
	Deleted:  /Users/john/.minishift/profiles/profile-demo
	Profile 'profile-demo' deleted successfully
	Switching to default profile 'minishift' as the active profile.
无法删除默认配置文件minishift
### 配置文件配置的示例工作流程
有两种方法可以创建新配置文件并配置其[持久配置](https://docs.okd.io/latest/minishift/using/basic-usage.html#persistent-configuration)。第一个选项是通过使用 `profile set` 命令使其成为活动配置文件来隐式创建新配置文件。配置文件激活后，可以运行任何 `minishift config` 命令。最后，启动实例：

	$ minishift profile set profile-demo
	$ minishift config set memory 8GB
	$ minishift config set cpus 4
	$ minishift addon enable anyuid
	$ minishift start
另一种方法是执行一系列命令，每个命令使用 `--profile` 标志显式指定目标配置文件

	$ minishift --profile profile-demo config set memory 8GB
	$ minishift --profile profile-demo config set cpus 4
	$ minishift --profile profile-demo addon enable anyuid
	$ minishift --profile profile-demo minishift start
# 镜像缓存
为了加快 OpenShift 群集的配置并最大限度地减少网络流量，可以在主机上缓存容器映像。然后可以将这些镜像导入到正在运行的 Docker 守护程序中，可以根据请求显式导入，也可以隐式导入 `minishift start` 。以下部分更详细地描述了镜像缓存及其配置。

- 使用 Minishift 版本 1.10.0 更改了缓存镜像的格式。在 1.10.0 之前，镜像存储为 tar 文件。从 1.10.0 开始，镜像以 [OCI 镜像](https://github.com/opencontainers/image-spec/blob/master/spec.md)格式存储。
- 如果在 Minishift 1.10.0 之前使用了镜像缓存，则需要重新创建缓存。如果要删除过时的 1.10.0 之前的镜像，可以通过以下方式清除缓存：

		$ minishift delete --clear-cache	

## 显式镜像缓存
Minishift 提供 `image` 命令及其子命令来控制镜像缓存的行为。要从 Minishift VM 的 Docker 守护程序导出和导入镜像，请使用 `minishift image export` 和 `minishift image import`。
### 导入和导出单个图像
Minishift VM 运行后，可以从 Docker 守护程序显式导出镜像，Docker 守护程序中没有的镜像将在导出到主机之前被提取。

	$ minishift image export <image-name-0> <image-name-1> ...
	Pulling image <image-name-0> .. OK
	Exporting <image-name-0>. OK
	Pulling image <image-name-1> .. OK
	Exporting <image-name-2>. OK
要导入以前缓存的镜像，请使用以下 `minishift image import` 命令：

	$ minishift image import <image-name-0> <image-name-1> ...
	Importing <image-name-0> . OK
### 列出缓存的图像
[minishift image list](https://docs.okd.io/latest/minishift/command-ref/minishift_image_list.html) 命令列出当前缓存的镜像或 Minishift Docker 守护程序中可用的镜像。

要在主机上查看当前缓存的图像：


	$ minishift image list
	openshift/origin-docker-registry:v3.6.0
	openshift/origin-haproxy-router:v3.6.0
	openshift/origin:v3.6.0
要查看 Docker 守护程序中可用的镜像

	$ minishift image list --vm
	openshift/origin-deployer:v3.6.0
	openshift/origin-docker-registry:v3.6.0
	openshift/origin-haproxy-router:v3.6.0
	openshift/origin-pod:v3.6.0
	openshift/origin:v3.6.0

### 持久缓存的图像名称
为了避免必须在 `image export` 或者 `image import` 命令中显式键入镜像名称，可以在持久性配置中存储导入和导出的镜像名称列表。

使用 [minishift image cache-config view](https://docs.okd.io/latest/minishift/command-ref/minishift_image_cache-config_view.html) 查看当前配置的图像列表，[minishift image cache-config add](https://docs.okd.io/latest/minishift/command-ref/minishift_image_cache-config_add.html) 将图片添加到列表：

	$ minishift image cache-config view
	$ minishift image cache-config add alpine:latest busybox:latest
	$ minishift image cache-config view
	alpine:latest
	busybox:latest
要从列表中删除镜像，请使用 `minishift image cache-config remove`

	$ minishift image cache-config remove alpine:latest
	$ minishift image cache-config view
	busybox:latest
一旦镜像名称存储在持久配置中，就可以运行 [minishift image export](https://docs.okd.io/latest/minishift/command-ref/minishift_image_export.html) 并且 [minishift image import](https://docs.okd.io/latest/minishift/command-ref/minishift_image_import.html) 不带任何参数
### 导出和导入所有图像
可以使用 `export` 命令的 `--all` 标志导出和导入所有镜像。这意味着 Docker 守护程序上当前可用的所有镜像都将导出到主机。对于 `import` 命令，这意味着本地 Minishift 缓存中的所有可用镜像都将导入到 Minishift VM 的 Docker 守护程序中。

导出和导入所有镜像可能需要很长时间，本地缓存的镜像可能占用大量磁盘空间。建议谨慎使用此功能。
### 隐式镜像缓存
Minishift 默认启用镜像缓存。它在 [minishift start](https://docs.okd.io/latest/minishift/command-ref/minishift_start.html) 第一次完成命令后在后台进程中发生。一旦镜像缓存在 `$MINISHIFT_HOME/cache/images`下，后续的 Minishift VM 创建将使用这些缓存的图像。

要禁用此功能，需要 `image-caching` 使用以下 [minishift config set](https://docs.okd.io/latest/minishift/command-ref/minishift_config_set.html) 命令禁用持久配置中的属性：

	$ minishift config set image-caching false

- 隐式镜像缓存将透明地将所需的 OpenShift 图像添加到按 `cache-images` 配置选项指定的缓存镜像列表中。请参阅[保留缓存的镜像名称](https://docs.okd.io/latest/minishift/using/image-caching.html#persisting-image-names)。
- 每次运行镜像导出后台进程时，都会在 `$MINISHIFT_HOME/logs` 下生成一个日志文件，该文件可用于验证导出的进度。

可以通过使用以下设置完全设置 `image-caching` 变成 `true`,或者来重新启用 OpenShift 镜像的缓存：[minishift config unset](https://docs.okd.io/latest/minishift/command-ref/minishift_config_unset.html)	

	$ minishift config unset image-caching

### 从本地缓存中删除镜像
镜像缓存在 `$MINISHIFT_HOME/cache/images` 下，并存储为 sha256 blob 以及索引，其中包含有关匹配 sha 与镜像名称的详细信息。

要从本地缓存中删除镜像，请使用以下 [minishift image delete](https://docs.okd.io/latest/minishift/command-ref/minishift_image_delete.html) 命令

	$ minishift image delete <image-name-0> <image-name-1> ...
	Deleting <image-name-0> from the local cache OK
	Deleting <image-name-1> from the local cache OK
# 附加组件
Minishift 允许通过 `cluster up `添加附加组件机制来扩展 OpenShift 设置。

加载项是包含扩展名为 `.addon` 的文本文件的目录。该目录还可以包含其他资源文件，例如 JSON 模板文件。但是，每个加载项只允许一个 `.addon` 文件。

以下示例显示了加载项文件的内容，包括加载项的名称和描述，其他元数据以及要应用的实际加载项命令

	# Name: anyuid  <-（必需）加载项的名称                                                                        
	# Description: Allows authenticated users to run images under a non pre-allocated UID   <-（必需）加载项的描述
	# Required-Vars: ACME_TOKEN   <-（可选）逗号分隔的所需插值变量列表。                                                       
	# OpenShift-Version: >3.6.0   <- （可选）运行特定加载项所需的OpenShift版本                                                          
	# Minishift-Version: >1.22.0  <-  （可选）运行特定加载项所需的Minishift版本                                                         
	
	mytoken := oc sa get-token oauth-client   <- 运行实际的附加命令并将其输出存储在变量中                                              
	
	oc new-app -p OPENSHIFT_OAUTH_CLIENT_SECRET=#{mytoken} <- 实际的附加命令。在这种情况下，该命令使用变量值执行 oc 二进制 mytoken                                 
	oc adm policy add-scc-to-group anyuid system:authenticated  <- 实际的附加命令。在这种情况下，该命令执行 oc 二进制文件  
- 可变插件值[参考](https://docs.okd.io/latest/minishift/using/addons.html#addon-variable-interpolation)
- openshift [版本定义](https://docs.okd.io/latest/minishift/using/addons.html#addon-openshift-version-semantics)
- minishift [版本定义](https://docs.okd.io/latest/minishift/using/addons.html#addon-minishift-version-semantics)
- 内部变量[参考](https://docs.okd.io/latest/minishift/using/addons.html#internal-addon-variable)

- 以“＃”字符开头的注释行可以插入文件中的任何位置。
- 以!字符开头的命令会忽略执行失败。借助于此，附加组件将是幂等的，即命令可以多次执行而不会更改附加组件的最终行为。

在 [minishift start](https://docs.okd.io/latest/minishift/command-ref/minishift_start.html) 成功完成初始集群设置之后立即应用已启用的加载项。

### OpenShift-Version语义
作为附加元数据的一部分，您可以指定需要运行的OpenShift版本才能应用附加组件。为此，您可以指定可选的OpenShift-Version元数据字段。语义如下

- `> 3.6.0`

	需要运行高于3.6.0的版本才能应用附加组件。
- `> = 3.6.0`

	需要运行3.6.0或更高版本才能应用附加组件。
- `3.6.0`

	需要运行版本3.6.0才能应用该加载项。
- `> = 3.5.0，<3.8.0`

	版本应高于或等于3.5.0但低于3.8.0。
- `> = 3.5.0，<= 3.8.0`

	版本应高于或等于3.5.0但低于或等于3.8.0。

	
如果未在加载项标头中指定元数据字段 `OpenShift-Version`，则可以对任何版本的 OpenShift 应用加载项。

`OpenShift-Version` 仅支持<major>。<minor>。<patch>形式的版本。	
### Minishift-Version Semantics
作为附加元数据的一部分，您可以指定需要运行的 Minishift 版本才能应用附加组件。为此，您可以指定可选的 `Minishift-Version` 元数据字段。语义如下：

- `> 1.22.0`

	需要运行高于1.22.0的版本才能应用附加组件。
- `> = 1.22.0`

	需要运行版本1.22.0或更高版本才能应用附加组件。
- `1.22.0`

	需要运行版本1.22.0才能应用附加组件。
- `> = 1.21.0，<1.25.0`

	版本应高于或等于1.21.0但低于1.25.0。
- `> = 1.22.0，<= 1.25.0`

	版本应高于或等于1.22.0但低于或等于1.25.0

### 定义附加依赖项
可以使用 Depends-On 元数据字段定义附加依赖项。可以使用以逗号分隔的附加组件名称列表来定义多个依赖项。

	# Name: example
	# Description: Shows the use of Depends-On tag
	# Depends-On: anyuid, admin-user
	
	# This add-on depends on anyuid and admin-user add-on
	
	echo Depends on anyuid, admin-user add-on, and requires them to be installed
### 附加命令
本节介绍加载项文件可以包含的命令。它们为 Minishift 附加组件构成了一个特定于域的特定语言：

- SSH

	如果以加载项命令开头 `ssh`，则可以在 Minishift 管理的VM中运行任何命令。这类似于 `minishift ssh` 在VM上运行然后执行任何命令。

- OC

	如果加载项命令以开头 `oc` ，它将使用 `oc` 主机上缓存的二进制文件来执行指定的 `oc` 命令。这类似于 `oc --as system:admin …​` 从命令行运行。

	该 `oc` 命令作为 `system：admin` 执行。
- openshift

	如果加载项命令以开头 `openshift`，它使用 `oc` OpenShift容器中的二进制文件来执行命令。这意味着任何文件参数或其他特定于系统的参数都必须与容器的环境匹配，而不是与主机相匹配。

- docker

	如果加载项命令以开头 `docker` ，它将 `docker` 对 Minishift VM 中的 Docker 守护程序执行命令。这与运行单节点 OpenShift 集群的守护程序相同。这类似于 `eval $(minishift docker-env)` 在主机上运行然后执行任何 `docker` 命令。另见 [minishift docker-env](https://docs.okd.io/latest/minishift/command-ref/minishift_docker-env.html)。

- echo

	如果以加载项命令开头 `echo`，则 `echo` 命令后面的参数将打印到控制台。这可用于在附加执行期间提供额外的反馈。
- sleep

	如果加载项命令以开头 `sleep` ，则等待指定的秒数。如果知道 `oc` 在查询某个资源之前可能需要几秒钟的命令，这可能很有用。

- cat 

	如果加载项命令以开头 `cat`，则 `cat` 命令后面的参数应该是有效的文件路径。这会将文件内容打印到控制台以获取有效文件，否则会显示有关文件不存在的错误消息。如果要将文件内容与其他命令一起使用，这可能很有用。

尝试使用未定义的命令将在解析加载项时导致错误。

### 可变插值
Minishift 允许在附加命令中使用变量。变量具有格式 `{<variable-name>}`。

示例显示如何将 OpenShift 路由后缀插入到 openshift 命令中以创建新证书，作为保护 OpenShift registry 的一部分。used 变量 `{routing-suffix}` 是内置附加变量的一部分。

	$ openshift admin ca create-server-cert \
	  --signer-cert=/var/lib/origin/openshift.local.config/master/ca.crt \
	  --signer-key=/var/lib/origin/openshift.local.config/master/ca.key \
	  --signer-serial=/var/lib/origin/openshift.local.config/master/ca.serial.txt \
	  --hostnames='docker-registry-default.#{routing-suffix},docker-registry.default.svc.cluster.local,172.30.1.1' \
	  --cert=/etc/secrets/registry.crt \
	  --key=/etc/secrets/registry.key
### 内置变量
存在几个可随时插值的内置变量。下表显示了这些变量。

- IP

	Minishift VM 的 IP。
- routing-suffix

	应用程序的 OpenShift 路由后缀。
- addon-name

	当前加载项的名称。
- user

	Minishift用于通过ssh执行命令的用户。
	
### 动态变量
命令 `minishift start` 以及 [minishift addons apply](https://docs.okd.io/latest/minishift/command-ref/minishift_addons_apply.html) 还提供了一种 `--addon-env` 标志，其允许动态地传递变量用于内插，例如：

	$ minishift addons apply --addon-env PROJECT_USER=john acme
`--addon-env` 可以多次指定该标志以定义多个用于插值的变量。

指定动态变量也可以与[设置持久配置值](https://docs.okd.io/latest/minishift/using/basic-usage.html#setting-persistent-configuration-values)一起使用

	$ minishift config set addon-env PROJECT_USER=john
	$ minishift addons apply acme
	
[minishift config set](https://docs.okd.io/latest/minishift/command-ref/minishift_config_set.html) 使用命令时，需要以逗号分隔多个变量。

还可以在加载项应用时使用环境变量的值动态插值变量。为此，变量值需要以 env 为前缀

	$ minishift config set addon-env PROJECT_USER=env.USER        
	$ minishift addons apply acme    
- 使用 env 前缀可确保使用 “env.USER” 替换 “＃{PROJECT_USER}”，而不是 `USER` 使用环境变量的值。如果未设置环境变量，则不会进行插值。
- 应用加载项时，`#{PROJECT_USER}` 加载项命令中的每次出现都将替换为环境变量的值 `USER`。

作为附加组件开发人员，可以通过将变量名称添加到 Required-Vars 元数据头来强制应用加载项时提供变量值。多个变量需要以逗号分隔。

	# Name: acme
	# Description: ACME add-on
	# Required-Vars: PROJECT_USER
还可以使用 Var-Defaults 元数据标头为变量提供默认值。 Var-Defaults 需要以格式指定 `<key>=<value>`。多个默认键/值对需要以逗号分隔。

	# Name: acme
	# Description: ACME add-on
	# Required-Vars: PROJECT_USER
	# Var-Defaults: PROJECT_USER=john	
- `=` 和 `,` 是元字符，不能用作键或值的一部分。
- 如果将 NULL，Null 或 null 指定为 _Var-Defaults 键的值，则将设置空值。例如 

		#Var-Defaults：PROJECT_USER = null
- 使用 `--addon-env` 或设置 via 通过命令行指定的变量值 `minishift config set addon-env` 优先于 Var-Defaults。	
### 内部变量
附加组件允许在附加组件中定义变量。这些变量可用于存储附加命令的输出。

例如，可以将 `oc` 命令输出保存在变量中 `mytoken`，并在后续的附加命令中使用它。

例如：

	mytoken := oc sa get-token oauth-client
	oc new-app -p OPENSHIFT_OAUTH_CLIENT_SECRET=#{mytoken}
### 默认加载项
Minishift 提供了一组内置附加组件，提供一些常见的 OpenShift 定制以协助开发。要安装默认加载项，请运行：

	$ minishift addons install --defaults
此命令提取加载项安装目录 `$MINISHIFT_HOME/addons` 的默认加载项。要查看已安装的加载项列表，可以运行：

	$ minishift addons list --verbose=true
此命令打印已安装的加载项列表。应该至少看到列出的 `anyuid` 插件。这是一个重要的附加组件，允许运行不使用预先分配的 UID 的图像。默认情况下，OpenShift 中不允许这样做。
	
- anyuid

	更改默认安全上下文约束以允许 pod 使用任意 UID 运行。
- admin-user

	创建名为“admin”的用户，并为其分配 cluster-admin 角色。
- registry-route

	为 OpenShift registry 创建边缘路由。
- htpasswd-identity-provider

	用户可以更改并添加OpenShift实例的默认登录用户名和密码。
- admissions-webhook

	启用 validating and mutating 的 webhook。
- xpaas

	导入 xPaaS 模板。
- redhat-registry-login

	创建在 registry.redhat.io 上访问镜像的 secret。
- che

	在 Minishift 上部署 che。
	
### 社区的附加组件
除了几个默认的附加组件外，还有许多社区开发的 Minishift 附加组件。社区附加组件可以在 [minishift-addons](https://github.com/minishift/minishift-addons/tree/master/add-ons) 存储库中找到。可以在[存储库](https://github.com/minishift/minishift-addons)中获取有关这些加载项的所有信息。有关安装它们的说明，请参见[自述文件](https://github.com/minishift/minishift-addons/blob/master/README.adoc)。
### 安装附加组件
随 [minishift addons install](https://docs.okd.io/latest/minishift/command-ref/minishift_addons_install.html) 命令一起安装加载项。

以下示例显示如何安装加载项。	
	
	$ minishift addons install <path_to_addon_directory>
### 启用和禁用加载项
使用该 [minishift addons enable](https://docs.okd.io/latest/minishift/command-ref/minishift_addons_enable.html) 命令启用加载项，并使用该命令禁用加载项 [minishift addons disable](https://docs.okd.io/latest/minishift/command-ref/minishift_addons_disable.html)。启用的加载项会在执行期间自动执行 `minishift start`。

以下示例显示如何启用和禁用 `anyuid` 加载项。	

- 启动

		$ minishift addons enable anyuid
- 禁用
		
		$ minishift addons disable anyuid

### 附加优先事项
启用加载项时，还可以指定优先级，该优先级确定加载项的应用顺序。

以下示例说明如何启用具有更高优先级值的 `registry` 加载项。

	$ minishift addons enable registry --priority=5
附加优先级属性确定应用附加组件的顺序。默认情况下，加载项的优先级为0.首先应用优先级值较低的加载项。

在下面的示例中，`anyuid`，`registry` 和 `eap` 加载项的相应优先级为0,5和10.这意味着首先应用 `anyuid`，然后是 `registry`，最后是 `eap` 加载项。

	$ minishift addons list
	- anyuid         : enabled    P(0)
	- registry       : enabled    P(5)
	- eap            : enabled    P(10)	
如果两个加载项具有相同的优先级，则不确定它们应用的顺序。
### 应用附加组件
可以使用该 [minishift addons apply](https://docs.okd.io/latest/minishift/command-ref/minishift_addons_apply.html) 命令显式执行加载项。可以 `apply` 对启用和禁用的加载项使用该命令。要使用单个命令应用多个加载项，请指定以空格分隔的加载项名称。

以下示例显示如何显式应用 `anyuid` 和 `admin-user` 加载项。

	$ minishift addons apply anyuid admin-user
### 删除加载项
可以使用该 [minishift addons remove](https://docs.okd.io/latest/minishift/command-ref/minishift_addons_remove.html) 命令删除加载项。它是镜像命令，[minishift addons apply](https://docs.okd.io/latest/minishift/using/addons.html#apply-addons) 无论是否启用加载项，都可以使用它。如果安装了指定的加载项并且 `<addon_name>.addon.remove` 文件存在，`minishift addons remove` 则将执行此文件中指定的命令。

要使用单个命令删除多个加载项，请指定以空格分隔的加载项名称。以下示例显示如何显式删除 `admin-user` 加载项。

	$ minishift addons remove admin-user
	-- Removing addon 'admin-user':.
	admin user deleted
### 卸载加载项
可以使用该 [minishift addons uninstall](https://docs.okd.io/latest/minishift/command-ref/minishift_addons_uninstall.html) 命令卸载加载项。它是镜像命令，[minishift addons install](https://docs.okd.io/latest/minishift/command-ref/minishift_addons_install.html) 无论是否启用加载项，都可以使用它。如果安装了指定的加载项，`minishift addons uninstall` 将从 `$MINISHIFT_HOME/addons` 中删除相应的加载项目录。

以下示例显示如何显式卸载 `admin-user` 附加组件：

	$ minishift addons uninstall admin-user
	Add-on 'admin-user' uninstalled
### 编写自定义加载项
要编写自定义加载项，应该创建一个目录，并在其中创建至少一个扩展名为 `.addon` 的文本文件，例如 `admin-role.addon`。

此文件需要包含 `名称` 和 `描述` 元数据字段，以及要作为加载项的一部分执行的命令。

以下示例显示了为开发人员用户提供集群管理员权限的加载项的定义。

	# Name: admin-role
	# Description: Gives the developer user cluster-admin privileges
	
	oc adm policy add-role-to-user cluster-admin developer
定义加载项后，可以通过运行以下命令来安装它：

	$ minishift addons install <ADDON_DIR_PATH>
还可以使用多行编写元数据。

	# Name: prometheus
	# Description: This template creates a Prometheus instance preconfigured to gather OpenShift and
	# Kubernetes platform and node metrics and report them to admins. It is protected by an
	# OAuth proxy that only allows access for users who have view access to the prometheus
	# namespace. You may customize where the images (built from openshift/prometheus
	# and openshift/oauth-proxy) are pulled from via template parameters.
	# Url: https://prometheus.io/
	
也可以直接在 Minishift 附加安装目录 `$MINISHIFT_HOME/addons` 中编辑您的加载项。请注意，如果加载项中存在错误，则在运行任何 `addons` 命令时都不会显示该错误，并且在此 `minishift start` 过程中不会应用该错误。

要提供加载项删除说明，可以使用扩展名 `.addon.remove` 创建文本文件，例如 `admin-user.addon.remove`。与 `.addon` 文件类似，它需要 `Name` 和 `Description` 元数据字段。如果存在 `.addon.remove` 文件，则可以通过该[remove](https://docs.okd.io/latest/minishift/using/addons.html#remove-addons)命令应用它。

# 主机文件夹
主机文件夹是主机上的目录，它们在主机和 Minishift VM 之间共享。这允许主机和 VM 之间的双向文件同步。以下部分讨论该 `minishift hostfolder` 命令的用法。
## minishift hostfolder 命令
Minishift 提供 [minishift hostfolder](https://docs.okd.io/latest/minishift/command-ref/minishift_hostfolder.html) 列出，添加，装载，卸载和删除主机文件夹的命令。您可以使用该 `hostfolder` 命令将多个共享文件夹装载到自定义指定的装入点上。

目前支持基于 [CIFS](https://en.wikipedia.org/wiki/Server_Message_Block) 和 [SSHFS](https://en.wikipedia.org/wiki/SSHFS) 的主机文件夹。
## 先决条件
- SSHFS

	SSHFS 是共享主机文件夹的默认技术。它没有任何先决条件。
- CIFS

	可以使用 CIFS 共享主机文件夹。可以在 Windows，macOS 和 Linux 上使用 CIFS。

在 macOS 上，您可以在 系统首选项 > 共享 下启用基于`CIFS` 的共享。有关详细的设置说明，请参阅如何在[Mac上连接文件共享](https://support.apple.com/en-us/HT204445)。

在Linux上，按照特定于发行版的说明安装Samba。
## 显示主机文件夹
该 [minishift hostfolder list](https://docs.okd.io/latest/minishift/command-ref/minishift_hostfolder_list.html)命令概述了已定义的主机文件夹，其名称，安装点，远程路径以及它们当前是否已装入。

示例输出可能如下所示：

	$ minishift hostfolder list
	Name    Type    Source               Mountpoint       Mounted
	test    sshfs   /Users/john/test     /mnt/sda1/test   N
	
在此示例中，有一个基于 sshfs 的主机文件夹，其名称为 test，它将 `/Users/john/test` 安装到 Minishift VM 中的 `/mnt/sda1/test` 上。该份额目前尚未安装。
## 添加主机文件夹
该 [minishift hostfolder add](https://docs.okd.io/latest/minishift/command-ref/minishift_hostfolder_add.html) 命令允许定义新的主机文件夹。

要使用的确切语法取决于主机文件夹类型。可以在非交互式和交互式配置之间选择类型。默认情况下它是非交互式的。通过指定，`--interactive` 可以选择交互式配置模式。

以下部分提供了配置CIFS和SSHFS主机文件夹的示例。
### CIFS
	$ minishift hostfolder add -t cifs --source //192.168.99.1/MYSHARE --target /mnt/sda1/myshare --options username=john,password=mysecret,domain=MYDOMAIN myshare
上面的命令将创建一个名为 `myshare` 的新主机文件夹。主机文件夹的源是 UNC 路径 `//192.168.99.1/MYSHARE`，这是必需的。目标或挂载点是 Minishift VM 中的 `/mnt/sda1/myshare` 。使用 `--options` 标志主机文件夹类型特定选项可以使用逗号分隔的键/值对列表指定。对于 CIFS 主机文件夹，所需选项是 CIFS 共享的用户名以及密码。通过 `--options` 标记共享的域也可以指定。通常这可以省略，但是例如在 Windows 上，当您的帐户链接到 Microsoft 帐户时，您必须使用 Microsoft 帐户电子邮件地址作为用户名以及作为 `$env:COMPUTERNAME` 域显示的计算机名称

在 Windows 主机上，该 `minishift hostfolder add` 命令还提供了一个 `users-share` 选项。如果指定此选项，则不需要指定UNC路径，并假定为 `C:\Users`。
### SSHFS
	$ minishift hostfolder add -t sshfs --source /Users/john/myshare --target /mnt/sda1/myshare myshare
对于基于 SSHFS 的主机文件夹，只需指定主机文件夹的源和目标。

添加主机文件夹时，可以使用 `--options` 标志在安装时使用特定类型的选项，例如：

	$ minishift hostfolder add -t sshfs --source ~/myshare/ --target /mnt/sda1/myshare --options "-o compression=no" myshare
### 特定于实例的主机文件夹
默认情况下，主机文件夹定义是持久性的，类似于其他[持久性配置](https://docs.okd.io/latest/minishift/using/basic-usage.html#persistent-configuration)选项。这意味着这些主机文件夹定义将在删除和随后重新创建 Minishift VM 后继续存在。

在某些情况下，可能希望仅为特定的 Minishift 实例定义主机文件夹。为此，可以使用命令的 `--instance-only` 标志 [minishift hostfolder add](https://docs.okd.io/latest/minishift/command-ref/minishift_hostfolder_add.html)。使用该 `--instance-only` 标志创建的主机文件夹定义将与其他任何特定于实例的状态一起删除 [minishift delete](https://docs.okd.io/latest/minishift/command-ref/minishift_delete.html)。
### 挂载主机文件夹
添加主机文件夹后，使用该 [minishift hostfolder mount](https://docs.okd.io/latest/minishift/command-ref/minishift_hostfolder_mount.html) 命令按名称安装主机文件夹：

	$ minishift hostfolder mount myshare	
可以通过运行来验证是否已装入主机文件夹：

	$ minishift hostfolder list
	Name       Mountpoint          Remote path              Mounted
	myshare    /mnt/sda1/myshare   //192.168.99.1/MYSHARE   Y
或者，您可以列出已装载的主机文件夹的实际内容：

	$ minishift ssh "ls -al /mnt/sda1/myshare"
挂载基于 SSHFS 的主机文件夹时，将在主机的端口 2022 上启动 SFTP 服务器进程。确保网络和防火墙设置允许打开此端口。如果需要配置此端口，可以使用密钥来使用 Minishift 的持久配置 `hostfolders-sftp-port`，例如：	

	$ minishift config set hostfolders-sftp-port 2222
### 自动挂载主机文件夹
每次运行时也可以自动挂载主机文件夹 `minishift start`。要设置自动安装，需要 `hostfolders-automount` 在Minishift配置文件中设置该选项。

	$ minishift config set hostfolders-automount true
`hostfolders-automount` 设置该选项后，Minishift 将尝试在此期间挂载所有已定义的主机文件夹 `minishift start`。
### 卸载主机文件夹
可以使用该 [minishift hostfolder umount](https://docs.okd.io/latest/minishift/command-ref/minishift_hostfolder_umount.html) 命令卸载主机文件夹。

	$ minishift hostfolder umount myshare
	
	$ minishift hostfolder list
	Name       Mountpoint          Remote path              Mounted
	myshare    /mnt/sda1/myshare   //192.168.99.1/MYSHARE   N
### 删除主机文件夹
可以使用该 [minishift hostfolder remove](https://docs.okd.io/latest/minishift/command-ref/minishift_hostfolder_umount.html) 命令删除主机文件夹定义。

	$ minishift hostfolder list
	Name        Mountpoint            Remote path              Mounted
	myshare     /mnt/sda1/myshare     //192.168.1.82/MYSHARE   N
	
	$ minishift hostfolder remove myshare
	
	$ minishift hostfolder list
	No host folders defined
# 分配静态IP地址
当使用 DHCP 分配 IP 时，大多数虚拟机管理程序不支持延长租用时间。这可能导致在重新启动后为 VM 分配新的 IP，因为它将与为旧 IP 生成的安全证书冲突。这将使 Minishift 完全无法使用，直到通过运行`minishift delete` 后设置新实例 `minishift start`。

为了防止这种情况，Minishift 包含为 VM 设置静态 IP 地址的功能。这将防止 IP 地址在重新启动之间切换。但是，由于 IP 地址的解析方式，它目前无法在所有驱动程序插件上运行。

- 仅为 Hyper-V 正式支持为 Minishift VM 分配静态 IP 地址。
- 使用KVM驱动程序插件时，无法为 Minishift VM 分配静态 IP 地址。	
# Minishift Docker守护进程
在单个 VM 中运行 OpenShift 时，可以重用 Minishift 管理的 Docker 守护程序以用于其他 Docker 用例。通过使用与 Minishift 相同的 Docker 守护程序，可以加快本地开发速度。
## 控制台配置
要配置的控制台以重用 Minishift Docker 守护程序，请按照下列步骤操作：

- 确保在计算机上安装了 Docker 客户端二进制文件。有关操作系统的特定二进制安装的信息，请参阅[Docker安装站点](https://docs.docker.com/engine/installation/)。
- 使用 `minishift start` 命令启动 Minishift 。
- 运行该 [minishift docker-env](https://docs.okd.io/latest/minishift/command-ref/minishift_docker-env.html) 命令以显示需要在 shell 中键入的命令，以便配置 Docker 客户端。命令输出将根据 OS 和 shell 类型而有所不同。
	- 打印 env  
	
			$ minishift docker-env
			export DOCKER_TLS_VERIFY="1"
			export DOCKER_HOST="tcp://192.168.99.100:2376"
			export DOCKER_CERT_PATH="/Users/Prometheus/.minishift/certs"
			# Run this command to configure your shell:
			# eval $(minishift docker-env)
	- 配置环境变量文件

			vi ~/.bash_profile
	- 加载

			source .bash_profile
- 运行命令测试

		docker info

# 选择ISO映像
当启动 Minishift 时，它会下载虚拟机​​管理程序用于配置 Minishift VM 的实时 ISO 映像。

默认情况下，Minishift 使用 [Minishift CentOS ISO](https://github.com/minishift/minishift-centos-iso/releases) 映像。此 ISO 映像基于 [CentOS](https://www.centos.org/)，这是一种类似于生产环境的企业级 Linux 发行版。

	不能同时运行具有不同 ISO 映像的相同 Minishift 实例。要在 ISO 映像之间切换，请删除 Minishift 实例并使用要使用的ISO映像启动新实例。

## 使用远程 ISO 映像
要使用远程可用的 ISO 映像，请在 `minishift start` 使用 `--iso-url` 标志期间指定 `Minishift ISO` 映像的URL ：

	$ minishift start --iso-url https://github.com/minishift/minishift-centos-iso/releases/download/v1.14.0/minishift-centos7.iso
## 使用本地ISO映像
要使用本地可用的 ISO 映像，请按照下列步骤操作：

- 从 [Minishift CentOS ISO 版本页面](https://github.com/minishift/minishift-centos-iso/releases)手动下载 Minishift CentOS ISO 映像。
- 在 `minishift start` 使用 `--iso-url` 标志期间指定镜像的文件URI 。

为文件URI指定的路径必须是绝对路径。

- Linux

		$ minishift start --iso-url file:///path/to/image.iso
- Windows

		C:\> minishift.exe start --iso-url file://d:/path/to/image.iso
		
# 实验特点
如果想访问一些即将推出的功能和实验，可以设置环境变量 `MINISHIFT_ENABLE_EXPERIMENTAL`，这样可以使用其他功能标记：

	$ export MINISHIFT_ENABLE_EXPERIMENTAL=y
实验性功能未得到官方支持，可能会破坏或导致意外行为。要分享您对这些功能的反馈，欢迎您联系Minishift社区
## 启用实验 oc cluster up 标志
默认情况下，Minishift 不会公开 [oc cluster up](https://github.com/openshift/origin/blob/release-3.11/docs/cluster_up_down.md) Minishift CLI 中的所有标志。

可以设置 `MINISHIFT_ENABLE_EXPERIMENTAL` 环境变量以为 `minishift start` 命令启用以下选项：

	extra-clusterup-flags
允许直接将标志传递给 `oc cluster up`。

使用实验标志，在 OKD 3.10 或更高版本中启用服务目录，命令为

	$ MINISHIFT_ENABLE_EXPERIMENTAL=y minishift start --extra-clusterup-flags "--enable=*,service-catalog"	
启用服务目录的推荐方法是 [minishift openshift component add](https://docs.okd.io/latest/minishift/command-ref/minishift_openshift_component_add.html) 命令
## 本地代理服务器
为了帮助在组织中使用安全证书但无法轻松与实例共享的情况，Minishift 可以在主机上运行本地代理服务器，Minishift 实例可以使用该服务器访问外部资源。

使用以下命令启用代理服务器
				
	$ minishift config set local-proxy true
这将启动主机上的代理服务器，该服务器将自动分配给 Minishift 实例。

知道公司或上游代理时，可以使用以下配置选项指定：

	$ minishift config set local-proxy-upstream http(s)://[username:password@]host:port
使用此选项时，将使用 Minishift 特定证书重新加密所有流量。因此，此代理仅应用于开发和与 Minishift 一起使用。

要允许外部流量到本地主机，可能必须 3128/tcp 在主机防火墙中启用端口。
## 本地DNS服务器
Minishift 提供 DNS 服务器以供离线使用或在测试时覆盖 DNS 记录的可能性。这将允许在没有 Internet 的情况下访问 OpenShift 路由。

DNS 服务器特定于配置文件。

启动 DNS 服务器可以按如下方式完成：

	$ minishift dns start
启动 DNS 服务器后，需要配置设备设置以使用此名称服务器。start 命令将显示一个临时选项，可在进入离线使用时使用。

在当前实现中，需要启动服务器并手动对主机设置进行必要的更改。DNS 配置不是永久性的，可能会在设备的网络状态发生变化时重置。

停止 DNS 服务器可以按如下方式完成：

	$ minishift dns stop
要获取DNS服务器的状态：

	$ minishift dns status
## macOS的本地DNS设置
最新版本的 macOS 不会在脱机模式下发送 DNS 查询，而使用 Minishift 的本地 DNS 服务器的过程比其他操作系统更复杂。
### 启用点击设备
检查 `/dev` 中是否存在 `tap` 设备

	$ ls /dev | grep tap
如果没有 tap 设备，请安装 tuntap 软件包

	$ brew install tuntap
### 使用点击设备创建网络服务
以 root 用户身份打开 `/Library/Preferences/SystemConfiguration/preferences.plist` 文件，并在该 `<key>NetworkServices</key>` 元素下添加以下XML ：

			<key>D16F22CE-6DDE-4E63-837C-E16538EA5CCB</key>   # 这是网络服务的 UUID。将此值替换为输出 uuidgen      
			<dict>
			    <key>DNS</key>
		    <dict />
		    <key>IPv4</key>
		    <dict>
		        <key>Addresses</key>
		        <array>
		            <string>10.10.90.1</string>     #网络服务的IP地址           
		        </array>
		        <key>ConfigMethod</key>
		        <string>Manual</string>
		        <key>SubnetMasks</key>
		        <array>
		            <string>255.255.0.0</string>
		        </array>
		    </dict>
		    <key>IPv6</key>
		    <dict>
		        <key>ConfigMethod</key>
		        <string>Automatic</string>
		    </dict>
		    <key>Interface</key>
		    <dict>
		        <key>DeviceName</key>
		        <string>tap0</string>       #/dev/tap要使用的设备                 
		        <key>Hardware</key>
		        <string>Ethernet</string>
		        <key>Type</key>
		        <string>Ethernet</string>
		        <key>UserDefinedName</key>
		        <string>MiniTap</string>      #网络服务的名称（这将显示在网络首选项GUI中          
		    </dict>
		    <key>Proxies</key>
		    <dict>
		        <key>ExceptionsList</key>
		        <array>
		            <string>*.local</string>
		            <string>169.254/16</string>
		        </array>
		        <key>FTPPassive</key>
		        <integer>1</integer>
		    </dict>
		    <key>SMB</key>
		    <dict />
		    <key>UserDefinedName</key>
		    <string>MiniTap</string>      #网络服务的名称（这将显示在网络首选项GUI中                  
		</dict>
### 将网络服务添加到 SERVICEORDER 数组
在 `/Library/Preferences/SystemConfiguration/preferences.plist` 文件中，查找 `<key>ServiceOrder</key>` 元素。以root 身份，将我们的 `MiniTap` 网络服务的 UUID 附加到此阵列。

	<key>ServiceOrder</key>
	    <array>
	        <string>06BFF3C7-13DA-420F-AE9C-B036401184D7</string>
	        <string>58231F56-CA25-4D41-930F-46D83CA07BFE</string>
	        <string>304203B0-AC87-459F-9761-C2799EEBB2E3</string>
	        <string>8655D244-C6E7-4CC0-BF06-BB18F9C3BB85</string>
	        <string>3C26FB9D-D918-4B79-9C7B-ADECD8EFE00F</string>
	        <string>D16F22CE-6DDE-4E63-837C-E16538EA5CCB</string>    #MiniTap网络服务的UUID    
	    </array>	
### 将网络服务添加到服务字典
在 `/Library/Preferences/SystemConfiguration/preferences.plist` 文件中，查找 `<key>Service</key>` 元素。以root身份，将以下XML附加到其字典中：

	<key>Service</key>
	    <dict>
	        <key>06BFF3C7-13DA-420F-AE9C-B036401184D7</key>
	        <dict>
	            <key>__LINK__</key>
	            <string>/NetworkServices/06BFF3C7-13DA-420F-AE9C-B036401184D7</string>
	        </dict>
	        <key>304203B0-AC87-459F-9761-C2799EEBB2E3</key>
	        <dict>
	            <key>__LINK__</key>
	            <string>/NetworkServices/304203B0-AC87-459F-9761-C2799EEBB2E3</string>
	        </dict>
	        <key>3C26FB9D-D918-4B79-9C7B-ADECD8EFE00F</key>
	        <dict>
	            <key>__LINK__</key>
	            <string>/NetworkServices/3C26FB9D-D918-4B79-9C7B-ADECD8EFE00F</string>
	        </dict>
	        <key>58231F56-CA25-4D41-930F-46D83CA07BFE</key>
	        <dict>
	            <key>__LINK__</key>
	            <string>/NetworkServices/58231F56-CA25-4D41-930F-46D83CA07BFE</string>
	        </dict>
	        <key>8655D244-C6E7-4CC0-BF06-BB18F9C3BB85</key>
	        <dict>
	            <key>__LINK__</key>
	            <string>/NetworkServices/8655D244-C6E7-4CC0-BF06-BB18F9C3BB85</string>
	        </dict>
	        <key>D16F22CE-6DDE-4E63-837C-E16538EA5CCB</key>       #MiniTap服务的UUID                            
	        <dict>
	            <key>__LINK__</key>
	            <string>/NetworkServices/D16F22CE-6DDE-4E63-837C-E16538EA5CCB</string> #将此UUID替换为MiniTap服务的UUID
	        </dict>
	    </dict>	    
重新启动 macOS，应该在网络首选项 GUI 中看到一个 MiniTap 服务。此服务将被断开。要打开它，请发出以下命令

	$ exec 4<>/dev/tap0		#	将其替换/dev/tap为MiniTap Service 使用的设备
	$ ifconfig tap0 10.10.90.1 255.255.0.0   #	IP地址应与MiniTap服务定义中的IP地址相同。
	$ ifconfig tap0 up		#	将其替换/dev/tap为MiniTap Service 使用的设备
### 添加解析器配置
`/etc/resolver/nip.io` 使用以下内容创建文件：

	nameserver <ip_address_of_the_minishfit_vm>
	search_order 1
### Minishift 系统托盘
为了帮助 macish 和 Windows 上的 Minishift 用户执行简单的任务，例如从 GUI 启动和停止配置文件，可以编译这些平台的二进制文件以包含实验系统托盘。按照 [Developing Minishift](https://docs.okd.io/latest/minishift/contributing/developing.html) 指南设置开发环境。

要使用系统托盘构建 Minishift，请使用以下命令：

	$ make systemtray
默认情况下，系统托盘在运行时自动启动 minishift start。要禁用自动启动行为，请使用以下命令

	$ minishift config set auto-start-tray false
要启动系统托盘

	$ minishift service start systemtray
### 时区设置
如果要设置与默认时区不同的时区，请使用以下命令：

	$ minishift timezone --set <Valid_Timezome>
要检查可用时区，请使用以下命令：

	$ minishift timezone --list
要检查Minishift实例的当前时区，请使用以下命令：

	$ minishift timezone		

# 针对现有机器运行
可以使用 `vm-driveras` 在现有远程计算机上运行 Minishift `generic`。

CentOS 7，Red Hat Enterprise Linux 7 和 Fedora> 26 是此功能的建议 Linux 发行版。
## 配置现有远程计算机
要使用具有 Minishift 的现有计算机，需要按如下方式配置：

建立从主机到现有远程机器的无密码SSH：

	Host$ ssh-copy-id <user>@<remote_IP_address> # ＃确保用户没有密码或使用root权限进行sudo访问
	Host$ ssh <user>@<remote_IP_address>         # 确保此登录无需密码即可运
如果您使用的是CentOS 7，Red Hat Enterprise Linux 7或Fedora（版本> 26），请跳过以下步骤。

1. 配置现有远程计算机

		Remote_Machine$ yum install -y docker net-tools firewalld
		Remote_Machine$ systemctl start docker
		Remote_Machine$ systemctl enable docker
		Remote_Machine$ systemctl start firewalld
		Remote_Machine$ systemctl enable firewalld
- 允许2376，8443和80TCP通过远程机器上的防火墙端口具有来自主机的通信

		Remote_Machine$ firewall-cmd --permanent --add-port 2376/tcp --add-port 8443/tcp --add-port 80/tcp 
- 确定Docker桥接网络容器子网

		Remote_Machine $ docker network inspect -f“{{range .IPAM.Config}} {{.Subnet}} {{end}}”bridge
	此命令显示子网，例如172.17.0.0/16
- 使用Docker桥接网络容器子网，为子网的源代码创建防火墙的minishift区域

		Remote_Machine $ firewall-cmd --permanent --new-zone minishift
		Remote_Machine $ firewall-cmd --permanent --zone minishift --add-source <subnet>
- 允许53和8053UDP端口通过远程计算机上的防火墙，以允许容器访问OpenShift主API和DNS端点

		Remote_Machine$ firewall-cmd --permanent --zone minishift --add-port 53/udp --add-port 8053/udp
- 重新加载远程计算机上的防火墙：

		Remote_Machine $ firewall-cmd --reload

## 针对现有远程计算机运行
在主机上使用以下命令对远程计算机运行 Minishift

	$ minishift start --vm-driver generic --remote-ipaddress <remote_IP_address> --remote-ssh-user <username> --remote-ssh-key <private_ssh_key>
	$ minishift addon apply htpasswd-identity-provider --addon-env USER_PASSWORD=<NEW_PASSWORD>
- `--remote-ssh-key` 标志的值必须是主机上私有SSH密钥的位置
- 要更改默认用户名和密码，必须 `htpasswd-identity-provider` 在运行 `minishift start` 命令后使用所需的用户名和密码应用加载项。默认情况下，Minishift 使用 `Allow All` 身份提供程序。该`Allow All` 身份提供者允许任何非空的用户名和密码进行登录。									