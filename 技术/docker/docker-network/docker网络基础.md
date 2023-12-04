## 基础信息
- docker0 的网段地址

	docker 启动的时候会在主机上创建一个 docker0 的虚拟网卡，随机挑选 `RFC1918` 私有网络中的一段地址给docker0，比如 `172.17.42.1/16` ,16位掩码的网段可以拥有 65534 个地址可以使用
- docker0 不是普通网卡，而是桥接到其他网卡的虚拟网卡，容器使用它和主机相互通讯。当创建一个 docker 容器的时候，它就创建了一个对接口，当数据包发送到一个接口时，另外一个接口也接收到相通的数据包是孪生接口。容器会在主机接口上制定一个唯一的名字，比如vethxxx这样的名字，这种接口不在主机的命名空间中。所有虚拟网卡都桥接到 docker0，形成一个虚拟网络。
- 启动 docker daemon 的网络参数，且不能马上生效

		-b BRIDGE or --bridge=BRIDGE #桥接
		--bip=CIDR #定制docker0
		-H SOCKET... or --host=SOCKET... #实际上是告诉 docker 从哪个通道来接收 run container stop container这样的命令。
		--icc=true|false #容器之间的通信
		--ip=IP_ADDRESS #绑定容器端口
		--ip-forward=true|false #容器之间的通信
		--iptables=true|false #容器之间的通信
		--mtu=BYTES #定制docker0
- 启动 daemon 和实例启动均可使用

		--dns=IP_ADDRESS... #添加dns服务器到容器的/etc/resolv,conf中，让容器用这ip地址来解析所有不在/etc/hosts中的主机名。
		--dns-search=DOMAIN... #设定容器的搜索域，当设定搜索域为.example.com时，会在搜索一个host主机名时，dns不仅搜索host，还会搜索host.example.com
		
		如果没有上述最后2个选项，docker会用主机上的/etc/resolv.conf来配置容器，它是默认配置
- 实例启动可用

		-h HOSTNAME or --hostname=HOSTNAME #主机名
		设定容器的主机名，它会被写到/etc/hostname，/etc/hosts中的ip地址自动写成分配的ip地址，在/bin/bash中显示该主机名。但它不会在docker ps中显示，也不会在其他的容器的/etc/hosts中显示。
		--link=CONTAINER_NAME:ALIAS #链接
		这选项会在创建容器的时候添加一个其他容器CONTAINE_NAME的主机名到/etc/hosts文件中，让新容器的进程可以使用主机名ALIAS就可以连接它。--link=会在容器之间的通信中更详细的介绍
		--net=bridge|none|container:NAME_or_ID|host #桥接
		-p SPEC or --publish=SPEC #端口绑定
		-P or --publish-all=true|false #端口绑定

## 容器间通讯
判断2个容器之间是否能够通信，在操作系统层面，取决于3个因素：

1. 网络拓扑是否连接到容器的网络接口？默认 docker 会将所有的容器连接到 docker0 这网桥来提供数据包通信。其他拓扑结构将在稍后的文档中详细介绍。
2. 主机是否开启 ip 转发，`ip_forward` 参数为1的时候可以提供数据包转发。通常只需要为 docker 设定 `--ip-forward=true`,docker 就会在服务启动的时候设定 ip_forward 参数为1。下面是手工检查并手工设定该参数的方法

		 cat /proc/sys/net/ipv4/ip_forward
3. iptables 是否允许这条特殊的连接被建立？当docker的设定--iptables=false时，docker不会改变系统的 iptables 设定，否则它会在 --icc=true 的时候添加一条默认的ACCEPT策略到 FORWARD链，当--icc=false时，策略为DROP。	

	几乎所有的人会开启 ip_forward 来启用容器间的通信。当是否要改变 `icc-true` 配置是一个战略问题（在ubuntu中，可以在`/etc/default/docker`编辑 DOCKER_OPTS变量，然后重启docker服务来设定），这样iptable就可以防止其他被感染容器特别是主机的恶意端口扫描和访问。

	当选择更安全的设定 `--icc=false` 后，如何保持容器之间通信呢？`--link=CONTAINER_NAME:ALIAS` 如果docker 使用 `icc=false` 和 `--iptables=true` 2个参数，当 docker run 使用`--link`时，docker 会为2个容器在 iptable 中参数一对ACCEPT规则，开放的端口取决与 dockerfile 中的 EXPOSE。注意：`--link` 中的CONTAINER_NAME 必须是自动生成的 docker 名字比如 `stupefied_pare`，或则用 `--name` 参数指定的名字，主机名在 `--link` 中不会被识别。

	可以使用iptable 命令查看 forward 的打开和关闭

	- 当 `-icc=false` 时，规则应该是这样

			$ sudo iptables -L -n
			Chain FORWARD (policy ACCEPT)target     prot opt source               			destination
			DROP       all  --  0.0.0.0/0            0.0.0.0/0
	- 当添加了 `--link` 后，ACCEPT 规则被改写了，添加了新的端口和IP规则

			$ sudo iptables -L -n
			Chain FORWARD (policy ACCEPT)target     prot opt source               destination
			ACCEPT     tcp  --  172.17.0.2           172.17.0.3           tcp spt:80
			ACCEPT     tcp  --  172.17.0.3           172.17.0.2           tcp dpt:80
			DROP       all  --  0.0.0.0/0            0.0.0.0/0
- 注意：docker对主机级别的的iptable处理非常小心，为了避免完全暴露容器之间的原始ip地址，连接仅现实连接源容器自己的本身的ip。

## 绑定一个容器端口到主机	
默认容器可以建立到外部网络的连接，但是外部网络无法连接到容器。所有到外部的连接源都会被伪装成主机的ip地址，iptables 的 masquerading 来做到这一点

	# 查看主机的 masquerade 规则 
	
	sudo iptables -t nat -L -n
	Chain POSTROUTING (policy ACCEPT)target     prot opt source               destination
	MASQUERADE  all  --  172.17.0.0/16       !172.17.0.0/16...
当希望容器接收外部连接时，需要在 docker run 执行的时候就指定对应选项，在 docker 用户指南页中有详细介绍，2种方法

- 指定 -P --publish-all=true|false 选项会映射 dockerfile 中 expose 的所有端口，主机端口在49000-49900中随机挑选。当的另外一个容器需要学习这个端口时候，很不方便。
- 更方便的方法是指定-p SPEC或则 --publish=SPEC,可以指定任意端口从主机映射容器内部

不管用那种办法，你可以通过查看iptable的 nat表来观察docker 在网络层

- 使用 -P 时

		$ iptables -t nat -L -n...Chain DOCKER (2 references)target     prot opt source               destination
		DNAT       tcp  --  0.0.0.0/0            0.0.0.0/0            tcp dpt:49153 to:172.17.0.2:80
- 使用-p 80:80时

		Chain DOCKER (2 references)target     prot opt source               destination
		DNAT       tcp  --  0.0.0.0/0            0.0.0.0/0            tcp dpt:80 to:172.17.0.2:80
		
这里可以看到docker映射了0.0.0.0.意味着接受主机上的所有接口地址，更严格的规则可以通过-p IP:host_port:container_port 或则 -p IP::port 来指定主机上的ip、接口。或则希望永久指定需要绑定的主机ip地址，可以 在dcoker 配置中指定--ip=IP_ADDRESS. 记得重启服务。

## 自定义 docker0
docker服务默认会创建一个docker0接口，它在linux内核层面桥接所有物理或则虚拟网卡，这实际上将他们都放到一个物理网络。docker 指定 docker0 的 ip地址和子网掩码，让主机和容器之间可以通过网桥相互通信，它还给出了 MTU -接口允许接收的最大传输单元，通常是 1500bytes 或则主机网络路由上支持的默认值。这2个都是在daemon启动的时候配置。

	--bip=CIDR #192.168.1.5/24.ip地址加掩码 使用这种格式
	--mtu=BYTES #覆盖默认的coker mut配置
当容器启动后，可以使用brctl来确认他们是否已经连接到 docker0 网桥。如：

	$ sudo brctl show
	bridge name     bridge id               STP enabled     interfaces
	docker0         8000.3a1d7362b4ee       no              veth65f9                                                        vethdda6
最后，docker0 网桥设置会在每次创建新容器的时候被使用。docker从可用的地址段中选择一个空闲的ip地址给容器的eth0端口，子网掩码使用网桥docker0的配置，docker主机本身的ip作为容器的网关使用。

	# ip addr show
	1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
	2: ens192: <BROADCAST,MULTICAST,PROMISC,UP,LOWER_UP> mtu 1500 qdisc mq state UP qlen 1000
    link/ether 00:50:56:85:42:0b brd ff:ff:ff:ff:ff:ff
    inet 192.168.1.213/24 brd 192.168.1.255 scope global ens192
       valid_lft forever preferred_lft forever
	3: docker0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue state DOWN
    link/ether 02:42:b9:dd:80:2b brd ff:ff:ff:ff:ff:ff
    inet 172.17.0.1/16 scope global docker0
       valid_lft forever preferred_lft forever

	# ip route
	default via 192.168.1.1 dev ens192
	169.254.0.0/16 dev ens192  scope link  metric 1002
	172.17.0.0/16 dev docker0  proto kernel  scope link  src 172.17.0.1
	192.168.1.0/24 dev ens192  proto kernel  scope link  src 192.168.1.213
## 使用自己的桥接
如果希望完全使用新的桥接设置，可以在启动 docker 服务的时候，使用 `-b BRIDGE or --bridge=BRIDGE` 来告诉 docker 使用新网桥。如果服务已经启动，旧的网桥还在使用中，那需要先停止服务，再删除旧的网桥

- 停止网桥和删除 docker0

		$ sudo service docker stop
		$ sudo ip link set dev docker0 down
		$ sudo brctl delbr docker0
- 创建新网桥

		$ sudo brctl addbr bridge0
		$ sudo ip addr add 192.168.5.1/24 dev bridge0
		$ sudo ip link set dev bridge0 up  #确认网桥启动
		$ ip addr show 
		
			bridge04: bridge0:  mtu 1500 qdisc noop state UP group default
    		link/ether 66:38:d0:0d:76:18 brd ff:ff:ff:ff:ff:ff
    		inet 192.168.5.1/24 scope global bridge0
    		valid_lft forever preferred_lft forever #告诉docker桥接设置，并启动docker服务（ubuntu上）
		$ echo 'DOCKER_OPTS="-b=bridge0"' >> /etc/default/docker
		$ sudo service docker start
结果会是dicker服务启动成功并绑定容器到新的网桥。之暂停并确认桥接配置之前，新建一个容器，你会看到它的ip是的新ip段，docker会自动检测到它。

## 一个实例的容器网络
机器需要一个网络接口来使用ip发送和接受数据包，路由表来定义如何到达哪些地址段。这里的网络接口可以不是物理接口。事实上，每个linxu机器上的lo环回接口（docker 容器中也有）就是一个完全的linux内核虚拟接口，它直接复制发送缓存中的数据包到接收缓存中。docker让主机和容器使用特殊的虚拟接口来通信--通信的2端叫 “peers“，他们在主机内核中连接在一起，所以能够相互通信。创建他们很简单，前面介绍过了。	docker创建容器的步骤如下：

- 创建一对虚拟接口
- 一端使用一个唯一的名字比如veth65f9,另外一端可以桥接到默认的docker0,或则其它你自己指定的桥接网卡。
- 映射其它接口到新的容器中，并给它重新命名为eth0,在容器这个隔离的网络接口命名空间中，它是唯一的，不会有物理接口和它冲突。
- 从主机桥接地址段中获取一个空闲地址给eth0使用，并设定默认路由到桥接网卡。

完成这些之后，容器就可以使用这eth0虚拟网卡来连接其他容器和其他网络。完成这些之后，容器就可以使用这eth0虚拟网卡来连接其他容器和其他网络。

- --net=bridge 

	默认选项，连接到指定网桥。
- --net=host 
	
	告诉docker不要将容器放到隔离的网络堆栈中。从本质上讲，这个选项告诉docker不要容器化容器的网络！容器进程还是有自己的文件系统、进程列表和资源限制。使用ip addr命令这样的容器处于docker 主机的外部，它有完全的主机接口访问权限。注意它不会让容器重新配置主机的网络堆栈，除非 --privileged=true 但是容器进程可以跟其他root进程一样打开低数字的端口，可以访问本地网络服务比如 D-bus。还可以让容器做一些意想不到的事情，比如重启主机。这个选项要非常小心的使用。
- --net=container:NAME_or_ID 

	告诉docker将新容器的进程放到一个已经存在的容器的网络堆栈中，新容器进程有它自己的文件系统、进程列表和资源限制，但它会和那个已经存在的容器共享ip地址和端口，他们之间来可以通过环回接口通信。
- --net=none

	告诉docker将新容器放到自己的网络堆栈中，但是不要配置它的网络。这可以让你创建任何自定义的配置。

当使用--net=none时候，如何配置

- 启动一个/bin/bash 指定--net=none
	
		$ sudo docker run -i -t --rm --net=none base /bin/bash
		root@63f36fc01b5f:/#
- 创建一个容器，查找这容器的进程id，创建 它的命名空间，后面的ip netns会用到

		$ sudo docker inspect -f '{{.State.Pid}}' 63f36fc01b5f2778
		$ pid=2778
		$ sudo mkdir -p /var/run/netns
		$ sudo ln -s /proc/$pid/ns/net /var/run/netns/$pid     
- 检查桥接网卡的ip和子网掩码

		$ ip addr show 
		docker021: docker0: 
		...inet 172.17.42.1/16 scope global docker0...
- 创建一对”peer“接口A和B，绑定A到网桥，并启用它

		$ sudo ip link add A type veth peer name B
		$ sudo brctl addif docker0 A
		$ sudo ip link set A up
- 将B放到容器的网络命名空间，命名为eth0,配置一个空闲的ip

		$ sudo ip link set B netns $pid
		$ sudo ip netns exec $pid ip link set dev B name eth0
		$ sudo ip netns exec $pid ip link set eth0 up
		$ sudo ip netns exec $pid ip addr add 172.17.42.99/16 dev eth0
		$ sudo ip netns exec $pid ip route add default via 172.17.42.1
到这里，你又可以想平常一样使用网络了。当退出shell后，docker清空容器，容器的eth0随网络命名空间一起被摧毁，A 接口也被自动从docker0取消注册。不用其他命令，所有东西都被清理掉了！ip 命令替代了旧的ifconfig route  等命令，ip addr 可以用ip a简写。注意ip netns exec命令，它可以让我们像root一样配置网络命名空间。但在容器内部无法使用，因为统一的安全策略，docker限制容器进程配置自己的网络。使用ip netns exec 可以让我们不用设置--privileged=true就可以完成一些可能带来危险的操作。

## 工具
- Jérme Petazzoni 创建了一个叫[pipework](https://github.com/jpetazzo/pipework)的shell脚本来帮助我们在复杂的场景中完成网络连接
- [Brandon Rhodes](https://github.com/brandon-rhodes/fopnp/tree/m/playground) 创建了一个完整的docker容器网络拓扑，包含 nat 防火墙，服务包括HTTP, SMTP, POP, IMAP, Telnet, SSH, and FTP

## 创建一个点对点连接
有时候你想要2个特殊的容器可以直连通信，而不用去配置复杂的主机网卡桥接。解决办法很简单：当创建一对接口节点，把他们都放进容器中，配置成点到点链路类型。这2个容器就可以直接通信了。配置如下：

在2个终端中启动2个容器

	$ sudo docker run -i -t --rm --net=none base /bin/bash
	root@1f1f4c1f931a:/#

	$ sudo docker run -i -t --rm --net=none base /bin/bash
	root@12e343489d2f:/#

	$ sudo docker inspect -f '{{.State.Pid}}' 1f1f4c1f931a
	2989
	$ sudo docker inspect -f '{{.State.Pid}}' 12e343489d2f
	3004
	$ sudo mkdir -p /var/run/netns
	$ sudo ln -s /proc/2989/ns/net /var/run/netns/2989$ sudo ln -s /proc/3004/ns/net /var/run/netns/3004  #创建”peer“接口，然后配置路由
	
	$ sudo ip link add A type veth peer name B
	$ sudo ip link set A netns 2989
	$ sudo ip netns exec 2989 ip addr add 10.1.1.1/32 dev A
	$ sudo ip netns exec 2989 ip link set A up
	$ sudo ip netns exec 2989 ip route add 10.1.1.2/32 dev A

	$ sudo ip link set B netns 3004
	$ sudo ip netns exec 3004 ip addr add 10.1.1.2/32 dev B
	$ sudo ip netns exec 3004 ip link set B up
	$ sudo ip netns exec 3004 ip route add 10.1.1.1/32 dev B
现在这2个容器就可以相互ping通，并成功建立连接。点到点链路不需要子网和子网掩码，使用ip route 来连接单个ip地址到指定的网络接口。	