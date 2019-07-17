# DNSMASQ
## 介绍
dnsmasq 是一个轻量级的 DHCP 和 DNS 缓存服务器,可以作为 lan 环境下的 dns、tftp、pxe、广告路由和 DHCP 服务器。它可以编译二进制中去掉不需要的功能，从而节省内存消耗。 
### DNS 功能
- 接受 dns 查询，并从小型本地缓存中进行应答或将其转发到一个真正递归的 dns 服务器。
- 可以加载 /etc/hosts，以便解析不显示在全局的本地主机名，还可以回答 dhcp 配置的主机 dns 查询。
- 可以作为一个或者多个域的权威 dns 服务器，允许本地名显示在全局 dns 中
- 可以配置 DNSSEC 验证

### DHCP 功能
- 支持分配静态地址分配和多个网络
- 会自动发送一组合理的默认 DHCP 选项，并且可以配置以及可以封装  vendor-encapsulated。
- 还包括了一个安全的只读 TFTP 服务器，也许 DHCP 主机网络的 PXE 启动，还支持 BOOTP。
- PXE 支持是全功能的，并且包括一个向客户端提供 PXE 信息的代理模式，而 DHCP 地址又另外一台服务器完成。
- DHCPv6 提供与 DHCPv4 服务相同的功能集，此外，它还包括路由器通告和一个清洁功能，也许仅对 IPv6 配置使用 DHCPv4 和无状态自动配置的客户端进行命名。支持从通过 DHCPv6 前缀动态委派子网地址分配。 

### 参数
注意，一般来说，也许缺少参数代表关闭功能，例如	`--pid-file` 禁用写入 pid 文件。

- --test

		配置文件的读取和语法检测，如果所有都可以，退出代码为0，否则为非零，不会启动 dnsmasq
- -w, --help

		显示所有命令行选项。 --hlep dhcp 将现实已知的 DHCPV4 选项，而 --hlep hdcp6 将显示 DHCPv6 选项。
- -h, --no-hosts

		不读取 /etc/hosts 文件
- -H, --addn-hosts=<file>

		多个主机文件，读取指定的文件以及 /etc/hosts ，如果给出了 -h ，只读指定的文件。多个主机可能会重复此选项。如果给出了一个目录，则读取目录中所有文件。
- --hostsdir=<path>

		读取目录中包含所有的主机文件，将自动读取新的或更改的文件。详细参考  --dhcp-hostsdir 参数描述
- -E, --expand-hosts

		添加主机名扩展给 /etc/hosts 中的简单名称。清注意，这个不适用于 cnames, PTR records, TXT等，例如，/etc/hosts中的db1将扩展成db1.freeoa.net
		expand-hosts
		local=/freeoa.net/
- -T, --local-ttl=<time>

		当从 /etc/hosts 或者配置或者DHCP租约文件 dnsmasq 恢复信息时，默认情况下将生存时间字段设置为零，这意味着请求本身不应缓存信息。这在几乎所有情况下都是正确的。此选项允许给出这些回复的生存时间(以秒为单位)。这将在某些情况下以牺牲客户端使用陈旧数据见效服务器负载为代价。
- --dhcp-ttl=<time>

		至于 --local-ttl，但仅影响来自 DHCP 租约的信息的回复。如果两者都给出，--dhcp-ttl 适用于 DHCP 信息，而 --local-ttl 适用于其他信息。如果设置为零将消除 --local-ttl 对 DHCP 的影响。
- --neg-ttl=<time>

		通常来自上游服务器的回复包含 dnsmasq 用于缓存的 soa 记录中的生存时间信息。如果没有回复信息，则 dnsmasq 不缓存回复信息，此选项给出了即使在没有 soa 的纪录情况下，dnsmasq 用于缓存回复的生存时间的默认值。
- --max-ttl=<time>

		设置将发送给客户端的最大 ttl 值。如果 ttl 值比较低，指定的最大 ttl 将提供给客户端。然而，真正的 ttl 值保存在缓存中，以避免洪水上有 dns 服务器。
- --max-cache-ttl=<time>
		
		缓存条目的时间设置最大的 ttl
- --min-cache-ttl=<time>

		和上条相反，设定最小 ttl ，认为设定这个值通常是个错误的，不建议这样做。 dnsmasq 将默认设置为1小时，除非重新编译。
- --auth-ttl=<time>

		设置权威服务器回答的 ttl 
- -k, --keep-in-foreground

		保持前台运行，可以在 docker 时使用
- -d, --no-daemon

		调试模式，不回在后台运行，不写 pid 文件，不改变用户 id，在 SIGUSR1 上收到完整的缓存存储。日志打印到 stderr 以及 syslog，部分配信的进程处理 tcp 查询。注意这个仅用于调试，生产中禁止使用。
- -q, --log-queries

		纪录由 dnsmasq 处理的 dns 查询结果。在收到 SIGUSR1 时，启用完整的缓存转存。如果提供参数是 extra，即  --log-queries=extra ，那么日志在每行的开头都有额外的信息。信息将包括与单个查询相关的日志在一起的序列号和请求者的ip。
- -8, --log-facility=<facility>

		设置 dnsmasq 将发送 syslog 条目的工具，默认是 DAEMON, 当调试的时候是 LOCAL0。如果给定的设施包含至少一个 ‘/’ 字符，则将其视为文件名，并将 dnsmasq 信息发送到文件而非 syslog。如果设备是 '_',那么 dnsmasq 将会发送到stderr。(读取配置时的错误仍然会进入 syslog，但所有的输出都是从成功启动中分离出独立的文件。)当日志打印到文件时，当收到 SIGUSR2 时， dnsmasq 将关闭并重新打开该文件。这允许在不停止 dnsmasq 的情况下转换日志文件。
- --log-async[=<lines>]

		启动异步日志记录，将日志写入缓存的数量进行限制，dnsmasq 在对排队进行处理，缓解阻塞。 异步记录允许它继续运行而不被 syslog 阻止，允许 syslog 使用 dnsmasq 进行 dns 查询，而不回有死锁的风险。如果日志队列变满， dnsmasq 的日志将会丢弃。默认队列长度为5，推荐 5-25，最大 100.
- -x, --pid-file=<path>

		指定 dnsmasq 进程号文件，默认 /var/run/dnsmasq.pid
- -u, --user=<username>

		改变运行 dnsmasq 的用户，默认 root
-g, --group=<groupname>

		改变运行 dnsmasq 的组，默认为 ‘dip’，以便访问  /etc/ppp/resolv.conf。
- -v, --version

		版本号
- -p, --port=<port>

		设置 dns 端口号，如果设置成 0，将完全禁用 dns 功能，只留下 dhcp 功能或者 tftp 服务。
- -P, --edns-packet-max=<size>

		指定 dns 转发器最大支持 EDNS.0 udp 的数据包，默认为 4096，这个是 RFC5625 推荐大小。
- -Q, --query-port=<query_port>

		使用特定的 udp 端口，而不是随机端口发出 dns 查询，并监听回复。 注意，使用此选项使 dnsmasq 降低了抵抗 dns 欺骗攻击的安全性，但可能会更快和更少的使用资源。如果设置为零，系统将分配一个单端口。
- --min-port=<port>

		这个适用于防火墙后面的系统，当使用这个选项时，使用端口将始终大于使用的端口。不要使用小于作为出站 dns 查询来源的端口。
--max-port=<port>

		这个适用于防火墙后面的系统，当使用这个选项时，使用端口将始终小于使用的端口。使用小于作为出站 dns 查询的来源端口。
- -i, --interface=<interface name> 

		监听网卡设置，当使用这个参数时， dnsmasq 将自动的将 loopback 添加到使用的接口列表中。如果没有给出 --interface 或者 --listen-address，将在所有的网卡上监听，除了 --except-interface 选项中的。有网卡别名(eg "eth1:0") 将不可以使用。需要使用  --listen-address 。一个简单的通配符，包括尾随'*'，可用于选项。
- -I, --except-interface=<interface name>

		不监听的网卡设置，这个选项会覆盖 --interface 设置，优先级高于它。
- --auth-server=<domain>,<interface>|<ip-address>

		访问接口或者查询启用 dns 权威模式。它应该在全局 dns 中解析为 A 和 AAAA 记录，并指向 dnsmasq 监听地址。设置会覆盖 -i 和 -I 设置
- --local-service

		仅接收来自本地网络的 DNS 查询，即查询服务器接口存在于子网。它将被设置为默认安装。
- -2, --no-dhcp-interface=<interface name>
		
		不要在指定的端口上提供 dhcp 和 tftp 服务，而只提供 dns 服务
- -a, --listen-address=<ipaddr>

		设置监听地址，如果要本机使用则写上127.0.0.1，例：listen-address=192.168.8.132,127.0.0.1**** 。
- -z, --bind-interfaces

		和上面的参数联合使用，上面的参数才可以生效。丢弃不应答请求，即使接口地址改变。强制绑定监听网卡。可以提供多个dnsmasq实例运行在同一台机器上提供便利。
- --bind-dynamic
	
		启动动态网络模式，它是 --bind-interfaces 和默认之间的混合。如果出现新的网卡或者地址，他会自动启动监听，根据访问控制配置。可以做到动态添加端口和监听同步。仅仅在 linux 下使用。
- -y, --localise-queries

		从 /etc/hosts 返回 dns 查询答案，这取决于接收到该查询的接口。如果 hosts 中的一个名称于多个地址相关联，并且至少一个地址于发送查询的接口位于同一个子网，则只返回该子网上地址。这允许服务器在每个接口对应的 hosts 文件中拥有多个地址，主机根据所连接的网络获取正确的地址。仅限于 ipv4
- -b, --bogus-priv

		提供私有反向查询。在 host 文件和 dhcp 租约文件中找不到的私有ip范围的所有反向查询都以 `no such domain` 返回而不是转向上游服务器， 从不转发不在路由地址中的域名。
- -V, --alias=[<old-ip>]|[<start-ip>-<end-ip>],<new-ip>[,<mask>]
		
		修改从上游服务器查询返回的 ipv4 地址，旧的被新的地址所替代。如果提供了子网掩码则将重写与旧版本匹配的任何地址。功能类似 Cisco PIX 的 dns 游览。如：--alias=1.2.3.0,6.7.8.0,255.255.255.0、--alias=192.168.0.10-192.168.0.40,10.0.0.0,255.255.255.0 maps 192.168.0.10->192.168.0.40 to 10.0.0.10->10.0.0.40 
- -B, --bogus-nxdomain=<ipaddr>

		Transform replies which contain the IP address given into "No such domain" replies. This is intended to counteract a devious move made by Verisign in September 2003 when they started returning the address of an advertising web page in response to queries for unregistered names, instead of the correct NXDOMAIN response. This option tells dnsmasq to fake the correct response when it sees this behaviour. As at Sept 2003 the IP address being returned by Verisign is 64.94.110.11
- --ignore-address=<ipaddr>

		忽略包含指定地址的 A 记录查询。防止伪造应答攻击。
- -f, --filterwin2k

		Later versions of windows make periodic DNS requests which don't get sensible answers from the public DNS and can cause problems by triggering dial-on-demand links. This flag turns on an option to filter such requests. The requests blocked are for records of types SOA and SRV, and type ANY where the requested name has underscores, to catch LDAP requests.
- -r, --resolv-file=<file>

		获取上游服务器的ip 地址文件，默认是 /etc/resolv.conf。 dnsmasq 可以指定多个这样的文件，指定的第一个文件将被覆盖，后续的添加列表中。当轮询时才允许这样做，当前最新的修改时间。
		resolv-file=/etc/dnsmasq.d/upstream_dns.conf
- -R, --no-resolv

		不读 /etc/resolv.conf 获取上游服务，而从命令行或者dnsmasq配置文件中获取，即不使用上级DNS主机配置文件(/etc/resolv.conf和resolv-file）
- -1, --enable-dbus[=<service-name>]

		Allow dnsmasq configuration to be updated via DBus method calls. The configuration which can be changed is upstream DNS servers (and corresponding domains) and cache clear. Requires that dnsmasq has been built with DBus support. If the service name is given, dnsmasq provides service at that name, rather than the default which is uk.org.thekelleys.dnsmasq
- -o, --strict-order

		默认情况下， dnsmasq 向已知的任何上游服务器查询发送，并尝试倾向向已知服务器查询，而设置这个标志强制按照 /etc/resolv.conf 文件顺序来查询。
- --all-servers

		默认情况下，当 dnsmasq 有多个上游服务器可用时，只会向一个服务器发送查询请求。设置标志是向所有服务器查询，首先应达的将作为原始请求者。
- --dns-loop-detect

		启用检查 dns 转发循环，即发送到上游服务器之一的查询最终返回到 dnsmasq 实例的新查询的情况。如果查询返回给发送它的服务器，则禁止发送它的上游服务器，并记录该时间。每次上游服务器的更改都会重新运行，包括以前禁用的测试。
- --stop-dns-rebind

		拒绝和记录位于私有ip范围的上游服务器名称地址，这将阻止攻击。
- --rebind-localhost-ok

		绑定检查中免除 127.0.0.0/8，该地址是范围是黑洞服务器返回。
- --rebind-domain-ok=[<domain>]|[[/<domain>/[<domain>/]

		不检查这些域的查询，参数可以是单个域，也可以是多个域，通过 '/' 分割 ，例 --rebind-domain-ok=/domain1/domain2/domain3/
- -n, --no-poll

		如果不允许 Dnsmasq 通过轮询 /etc/resolv.conf 或者其他文件来获取配置的改变，则取消注释。
- --clear-on-reload

		无论何时都重新读取 /etc/resolv.conf 或者上游服务器，清除 dns 缓存。当新的 dns 服务器和原有缓存不同步时用。
- -D, --domain-needed

		如果不能从 /etc/resolv.conf 获取到名称，将返回 “"not found” ，不向上游服务器转发没有点或者没有域的 A 或者 AAAA 查询，强制使用完整的解析名，从不转发格式错误的域名。
- -S, --local, --server=[/[<domain>]/[domain/]][<ipaddr>[#<port>][@<source-ip>|<interface>[#<port>]]

		指定上游服务器ip 地址。设置此标志不会抑制 /etc/resolv.conf 的读取，请使用 -R 来执行操作。如果给出一个或者多个可选域，则该服务器仅用于哪些域，并仅使用指定的服务器查询。这适用于私有域名服务器。 DNSSEC验证会关闭此类私有服务器和涉及的域名。例如：-S /internal.thekelleys.org.uk/192.168.1.1
		--server=/google.com/1.2.3.4 
		--server=/www.google.com/2.3.4.5
		以上设置是 *.google.com 的查询请求均会发送到1.2.3.4，除了www.google.com 这个域名。
		 --server=/google.com/1.2.3.4 
		 --server=/www.google.com/#
		以上设置是 *.google.com 的查询请求均会发送到1.2.3.4，除了www.google.com 这个域名将使用标准服务器。
		可以给出一个域，但是不给出ip，告诉 dnsmasq 一个域时本地的。可以通过 /etc/host 或者 dhcp 查询，但不应该转发到任何上游服务器，这种情况下和local同意义／
		IPv6 addresses may include a %interface scope-id, eg fe80::202:a412:4512:7bbf%eth0.
		@字符串是可选的，告诉 dnsmasq 如何查询来源设置为此的服务器。它应该是一个ip地址，属于运行dnsmasq的机器，否则这个服务器将被记录，然后被忽略。如果给出一个网卡名，则通过该网卡强制对服务器查询，如果给出了一个ip地址，则查询源地址被设置为该地址。
		# 控制Dnsmasq和Server之间的查询从哪个网卡出去
		# server=10.1.2.3@eth1
		
		# 可以为特定的域名指定解析它专用的nameserver。一般是内部dns name server
		# server=/myserver.com/192.168.8.1

		# 增加一个name server，一般用于内网域名
		# server=/localnet/192.168.8.1

		# 设置一个反向解析，所有192.168.3.0/24的地址都到10.1.2.3去解析
		# server=/3.168.192.in-addr.arpa/10.1.2.3
		
		# 增加一个本地域名，会在/etc/hosts中进行查询
		# local=/localnet/
		
		# 指定dnsmasq默认查询的上游服务器，此处以Google Public DNS为例。等同于读取文件
		server=8.8.8.8
		server=8.8.4.4
		
		# 把所有.cn的域名全部通过114.114.114.114这台国内DNS服务器来解析
		server=/cn/114.114.114.114			
		
		# 给*.apple.com和taobao.com使用专用的DNS
		server=/taobao.com/223.5.5.5
		server=/.apple.com/223.6.6.6
		
		# 指定源地址携带10.1.2.3地址和192.168.8.1的55端口进行通讯
		# server=10.1.2.3@192.168.8.1#55

- --rev-server=<ip-address>/<prefix-len>,<ipaddr>[#<port>][@<source-ip>|<interface>[#<port>]]

		功能与server相同，但是提供 ip 主机名反查的。
- -A, --address=/<domain>/[domain/][<ipaddr>]

		工作原理类似 server ，域中的查询将永远不会转发到上游服务器中，而始终返回指定的ip地址。通常的做法将一个域重定向到本地的服务器中避免劫持。可以通过 ' /#/ '匹配任何域。使用 ' /#/ ' 后会导致任何没有被 /etc/hosts 查询应答的，全部配匹配返回这里设置的ip地址，如 --address=/#/1.2.3.4 ，而不是通过更具体的 --server 参数发送到上游dns 服务器。但 --address=/example.com/ 的设置等效于 --server=/example.com/，返回 NXDOMAIN
		
		# 增加一个域名，强制解析到所指定的地址上，强行指定domain的IP地址，dns欺骗
		address=/doubleclick.net/127.0.0.1
		address=/.phobos.apple.com/202.175.5.114
- --ipset=/<domain>/[domain/]<ipset>[,<ipset>]

		将指定的域查询解析ip地址放在指定的 netfilter ip sets 中，域和子域的匹配方式与 --address 相同，这些 ip 集必须存在于。例子: #ipset=/yahoo.com/google.com/ff,search 
- -m, --mx-host=<mx name>[[,<hostname>],<preference>]

		Return an MX record named <mx name> pointing to the given hostname (if given), or the host specified in the --mx-target switch or, if that switch is not given, the host on which dnsmasq is running. The default is useful for directing mail from systems on a LAN to a central server. The preference value is optional, and defaults to 1 if not given. More than one MX record may be given for a host.
- -t, --mx-target=<hostname>

		Specify the default target for the MX record returned by dnsmasq. See --mx-host. If --mx-target is given, but not --mx-host, then dnsmasq returns a MX record containing the MX target for MX queries on the hostname of the machine on which dnsmasq is running.
- -e, --selfmx

		Return an MX record pointing to itself for each local machine. Local machines are those in /etc/hosts or with DHCP leases.
- -L, --localmx

		Return an MX record pointing to the host given by mx-target (or the machine on which dnsmasq is running) for each local machine. Local machines are those in /etc/hosts or with DHCP leases.
- -W, --srv-host=<_service>.<_prot>.[<domain>],[<target>[,<port>[,<priority>[,<weight>]]]]

		Return a SRV DNS record. See RFC2782 for details. If not supplied, the domain defaults to that given by --domain. The default for the target domain is empty, and the default for port is one and the defaults for weight and priority are zero. Be careful if transposing data from BIND zone files: the port, weight and priority numbers are in a different order. More than one SRV record for a given service/domain is allowed, all that match are returned.
- --host-record=<name>[,<name>....],[<IPv4-address>],[<IPv6-address>][,<TTL>]

		Add A, AAAA and PTR records to the DNS. This adds one or more names to the DNS with associated IPv4 (A) and IPv6 (AAAA) records. A name may appear in more than one host-record and therefore be assigned more than one address. Only the first address creates a PTR record linking the address to the name. This is the same rule as is used reading hosts-files. host-record options are considered to be read before host-files, so a name appearing there inhibits PTR-record creation if it appears in hosts-file also. Unlike hosts-files, names are not expanded, even when expand-hosts is in effect. Short and long names may appear in the same host-record, eg. --host-record=laptop,laptop.thekelleys.org,192.168.0.1,1234::100
		If the time-to-live is given, it overrides the default, which is zero or the value of --local-ttl. The value is a positive integer and gives the time-to-live in seconds.
- -Y, --txt-record=<name>[[,<text>],<text>]

		Return a TXT DNS record. The value of TXT record is a set of strings, so any number may be included, delimited by commas; use quotes to put commas into a string. Note that the maximum length of a single string is 255 characters, longer strings are split into 255 character chunks.
- --ptr-record=<name>[,<target>]

		Return a PTR DNS record.
- --naptr-record=<name>,<order>,<preference>,<flags>,<service>,<regexp>[,<replacement>]

		Return an NAPTR DNS record, as specified in RFC3403.
- --cname=<cname>,<target>[,<TTL>]

		Return a CNAME record which indicates that <cname> is really <target>. There are significant limitations on the target; it must be a DNS name which is known to dnsmasq from /etc/hosts (or additional hosts files), from DHCP, from --interface-name or from another --cname. If the target does not satisfy this criteria, the whole cname is ignored. The cname must be unique, but it is permissable to have more than one cname pointing to the same target.
		If the time-to-live is given, it overrides the default, which is zero or the value of -local-ttl. The value is a positive integer and gives the time-to-live in seconds.
- --dns-rr=<name>,<RR-number>,[<hex data>]

		Return an arbitrary DNS Resource Record. The number is the type of the record (which is always in the C_IN class). The value of the record is given by the hex data, which may be of the form 01:23:45 or 01 23 45 or 012345 or any mixture of these.
- --interface-name=<name>,<interface>[/4|/6]

		Return a DNS record associating the name with the primary address on the given interface. This flag specifies an A or AAAA record for the given name in the same way as an /etc/hosts line, except that the address is not constant, but taken from the given interface. The interface may be followed by "/4" or "/6" to specify that only IPv4 or IPv6 addresses of the interface should be used. If the interface is down, not configured or non-existent, an empty record is returned. The matching PTR record is also created, mapping the interface address to the name. More than one name may be associated with an interface address by repeating the flag; in that case the first instance is used for the reverse address-to-name mapping.
- --synth-domain=<domain>,<address range>[,<prefix>]
		
		Create artificial A/AAAA and PTR records for an address range. The records use the address, with periods (or colons for IPv6) replaced with dashes.
		An example should make this clearer. --synth-domain=thekelleys.org.uk,192.168.0.0/24,internal- will result in a query for internal-192-168-0-56.thekelleys.org.uk returning 192.168.0.56 and a reverse query vice versa. The same applies to IPv6, but IPv6 addresses may start with '::' but DNS labels may not start with '-' so in this case if no prefix is configured a zero is added in front of the label. ::1 becomes 0--1.
		The address range can be of the form <ip address>,<ip address> or <ip address>/<netmask>
- --add-mac[=base64|text]

		Add the MAC address of the requestor to DNS queries which are forwarded upstream. This may be used to DNS filtering by the upstream server. The MAC address can only be added if the requestor is on the same subnet as the dnsmasq server. Note that the mechanism used to achieve this (an EDNS0 option) is not yet standardised, so this should be considered experimental. Also note that exposing MAC addresses in this way may have security and privacy implications. The warning about caching given for --add-subnet applies to --add-mac too. An alternative encoding of the MAC, as base64, is enabled by adding the "base64" parameter and a human-readable encoding of hex-and-colons is enabled by added the "text" parameter.
- --add-cpe-id=<string>

		Add a arbitrary identifying string to o DNS queries which are forwarded upstream.
- --add-subnet[[=[<IPv4 address>/]<IPv4 prefix length>][,[<IPv6 address>/]<IPv6 prefix length>]]

		Add a subnet address to the DNS queries which are forwarded upstream. If an address is specified in the flag, it will be used, otherwise, the address of the requestor will be used. The amount of the address forwarded depends on the prefix length parameter: 32 (128 for IPv6) forwards the whole address, zero forwards none of it but still marks the request so that no upstream nameserver will add client address information either. The default is zero for both IPv4 and IPv6. Note that upstream nameservers may be configured to return different results based on this information, but the dnsmasq cache does not take account. If a dnsmasq instance is configured such that different results may be encountered, caching should be disabled.
		For example, --add-subnet=24,96 will add the /24 and /96 subnets of the requestor for IPv4 and IPv6 requestors, respectively. --add-subnet=1.2.3.4/24 will add 1.2.3.0/24 for IPv4 requestors and ::/0 for IPv6 requestors. --add-subnet=1.2.3.4/24,1.2.3.4/24 will add 1.2.3.0/24 for both IPv4 and IPv6 requestors.
- -c, --cache-size=<cachesize>

		设置 dnsmasq 缓存大小，默认150 条。参数设置成零将禁用高速缓存。
- -N, --no-negcache

		Disable negative caching. Negative caching allows dnsmasq to remember "no such domain" answers from upstream nameservers and answer identical queries without forwarding them again.
- -0, --dns-forward-max=<queries>

		Set the maximum number of concurrent DNS queries. The default value is 150, which should be fine for most setups. The only known situation where this needs to be increased is when using web-server log file resolvers, which can generate large numbers of concurrent queries.
- --dnssec

		Validate DNS replies and cache DNSSEC data. When forwarding DNS queries, dnsmasq requests the DNSSEC records needed to validate the replies. The replies are validated and the result returned as the Authenticated Data bit in the DNS packet. In addition the DNSSEC records are stored in the cache, making validation by clients more efficient. Note that validation by clients is the most secure DNSSEC mode, but for clients unable to do validation, use of the AD bit set by dnsmasq is useful, provided that the network between the dnsmasq server and the client is trusted. Dnsmasq must be compiled with HAVE_DNSSEC enabled, and DNSSEC trust anchors provided, see --trust-anchor. Because the DNSSEC validation process uses the cache, it is not permitted to reduce the cache size below the default when DNSSEC is enabled. The nameservers upstream of dnsmasq must be DNSSEC-capable, ie capable of returning DNSSEC records with data. If they are not, then dnsmasq will not be able to determine the trusted status of answers. In the default mode, this menas that all replies will be marked as untrusted. If --dnssec-check-unsigned is set and the upstream servers don't support DNSSEC, then DNS service will be entirely broken.
- --trust-anchor=[<class>],<domain>,<key-tag>,<algorithm>,<digest-type>,<digest>

		Provide DS records to act a trust anchors for DNSSEC validation. Typically these will be the DS record(s) for Zone Signing key(s) of the root zone, but trust anchors for limited domains are also possible. The current root-zone trust anchors may be downloaded from https://data.iana.org/root-anchors/root-anchors.xml
- --dnssec-check-unsigned

		As a default, dnsmasq does not check that unsigned DNS replies are legitimate: they are assumed to be valid and passed on (without the "authentic data" bit set, of course). This does not protect against an attacker forging unsigned replies for signed DNS zones, but it is fast. If this flag is set, dnsmasq will check the zones of unsigned replies, to ensure that unsigned replies are allowed in those zones. The cost of this is more upstream queries and slower performance. See also the warning about upstream servers in the section on --dnssec
- --dnssec-no-timecheck

		DNSSEC signatures are only valid for specified time windows, and should be rejected outside those windows. This generates an interesting chicken-and-egg problem for machines which don't have a hardware real time clock. For these machines to determine the correct time typically requires use of NTP and therefore DNS, but validating DNS requires that the correct time is already known. Setting this flag removes the time-window checks (but not other DNSSEC validation.) only until the dnsmasq process receives SIGHUP. The intention is that dnsmasq should be started with this flag when the platform determines that reliable time is not currently available. As soon as reliable time is established, a SIGHUP should be sent to dnsmasq, which enables time checking, and purges the cache of DNS records which have not been throughly checked.
- --dnssec-timestamp=<path>

		Enables an alternative way of checking the validity of the system time for DNSSEC (see --dnssec-no-timecheck). In this case, the system time is considered to be valid once it becomes later than the timestamp on the specified file. The file is created and its timestamp set automatically by dnsmasq. The file must be stored on a persistent filesystem, so that it and its mtime are carried over system restarts. The timestamp file is created after dnsmasq has dropped root, so it must be in a location writable by the unprivileged user that dnsmasq runs as.
- --proxy-dnssec

		Copy the DNSSEC Authenticated Data bit from upstream servers to downstream clients and cache it. This is an alternative to having dnsmasq validate DNSSEC, but it depends on the security of the network between dnsmasq and the upstream servers, and the trustworthiness of the upstream servers.
- --dnssec-debug

		Set debugging mode for the DNSSEC validation, set the Checking Disabled bit on upstream queries, and don't convert replies which do not validate to responses with a return code of SERVFAIL. Note that setting this may affect DNS behaviour in bad ways, it is not an extra-logging flag and should not be set in production.
- --auth-zone=<domain>[,<subnet>[/<prefix length>][,<subnet>[/<prefix length>].....]]

		Define a DNS zone for which dnsmasq acts as authoritative server. Locally defined DNS records which are in the domain will be served. If subnet(s) are given, A and AAAA records must be in one of the specified subnets.
		As alternative to directly specifying the subnets, it's possible to give the name of an interface, in which case the subnets implied by that interface's configured addresses and netmask/prefix-length are used; this is useful when using constructed DHCP ranges as the actual address is dynamic and not known when configuring dnsmasq. The interface addresses may be confined to only IPv6 addresses using <interface>/6 or to only IPv4 using <interface>/4. This is useful when an interface has dynamically determined global IPv6 addresses which should appear in the zone, but RFC1918 IPv4 addresses which should not. Interface-name and address-literal subnet specifications may be used freely in the same --auth-zone declaration.

		The subnet(s) are also used to define in-addr.arpa and ip6.arpa domains which are served for reverse-DNS queries. If not specified, the prefix length defaults to 24 for IPv4 and 64 for IPv6. For IPv4 subnets, the prefix length should be have the value 8, 16 or 24 unless you are familiar with RFC 2317 and have arranged the in-addr.arpa delegation accordingly. Note that if no subnets are specified, then no reverse queries are answered.

- --auth-soa=<serial>[,<hostmaster>[,<refresh>[,<retry>[,<expiry>]]]]

		Specify fields in the SOA record associated with authoritative zones. Note that this is optional, all the values are set to sane defaults.
- --auth-sec-servers=<domain>[,<domain>[,<domain>...]]
		
		Specify any secondary servers for a zone for which dnsmasq is authoritative. These servers must be configured to get zone data from dnsmasq by zone transfer, and answer queries for the same authoritative zones as dnsmasq.
- --auth-peer=<ip-address>[,<ip-address>[,<ip-address>...]]

		Specify the addresses of secondary servers which are allowed to initiate zone transfer (AXFR) requests for zones for which dnsmasq is authoritative. If this option is not given, then AXFR requests will be accepted from any secondary.
- --conntrack

		Read the Linux connection track mark associated with incoming DNS queries and set the same mark value on upstream traffic used to answer those queries. This allows traffic generated by dnsmasq to be associated with the queries which cause it, useful for bandwidth accounting and firewalling. Dnsmasq must have conntrack support compiled in and the kernel must have conntrack support included and configured. This option cannot be combined with --query-port.

- -C, --conf-file=<file>

		Specify a different configuration file. The conf-file option is also allowed in configuration files, to include multiple configuration files. A filename of "-" causes dnsmasq to read configuration from stdin.
- -7, --conf-dir=<directory>[,<file-extension>......],

		Read all the files in the given directory as configuration files. If extension(s) are given, any files which end in those extensions are skipped. Any files whose names end in ~ or start with . or start and end with # are always skipped. If the extension starts with * then only files which have that extension are loaded. So --conf-dir=/path/to/dir,*.conf loads all files with the suffix .conf in /path/to/dir. This flag may be given on the command line or in a configuration file. If giving it on the command line, be sure to escape * characters.
- --servers-file=<file>

		A special case of --conf-file which differs in two respects. Firstly, only --server and --rev-server are allowed in the configuration file included. Secondly, the file is re-read and the configuration therein is updated when dnsmasq recieves SIGHUP