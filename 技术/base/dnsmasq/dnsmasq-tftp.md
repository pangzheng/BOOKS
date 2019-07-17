- --enable-ra

		Enable dnsmasq's IPv6 Router Advertisement feature. DHCPv6 doesn't handle complete network configuration in the same way as DHCPv4. Router discovery and (possibly) prefix discovery for autonomous address creation are handled by a different protocol. When DHCP is in use, only a subset of this is needed, and dnsmasq can handle it, using existing DHCP configuration to provide most data. When RA is enabled, dnsmasq will advertise a prefix for each dhcp-range, with default router as the relevant link-local address on the machine running dnsmasq. By default, the "managed address" bits are set, and the "use SLAAC" bit is reset. This can be changed for individual subnets with the mode keywords described in --dhcp-range. RFC6106 DNS parameters are included in the advertisements. By default, the relevant link-local address of the machine running dnsmasq is sent as recursive DNS server. If provided, the DHCPv6 options dns-server and domain-search are used for the DNS server (RDNSS) and the domain serach list (DNSSL).
- --ra-param=<interface>,[high|low],[[<ra-interval>],<router lifetime>]

		Set non-default values for router advertisements sent via an interface. The priority field for the router may be altered from the default of medium with eg --ra-param=eth0,high. The interval between router advertisements may be set (in seconds) with --ra-param=eth0,60. The lifetime of the route may be changed or set to zero, which allows a router to advertise prefixes but not a route via itself. --ra-parm=eth0,0,0 (A value of zero for the interval means the default value.) All three parameters may be set at once. --ra-param=low,60,1200 The interface field may include a wildcard.
- --enable-tftp[=<interface>[,<interface>]]

		Enable the TFTP server function. This is deliberately limited to that needed to net-boot a client. Only reading is allowed; the tsize and blksize extensions are supported (tsize is only supported in octet mode). Without an argument, the TFTP service is provided to the same set of interfaces as DHCP service. If the list of interfaces is provided, that defines which interfaces recieve TFTP service.
- --tftp-root=<directory>[,<interface>]
		
		Look for files to transfer using TFTP relative to the given directory. When this is set, TFTP paths which include ".." are rejected, to stop clients getting outside the specified root. Absolute paths (starting with /) are allowed, but they must be within the tftp-root. If the optional interface argument is given, the directory is only used for TFTP requests via that interface.
- --tftp-no-fail

		Do not abort startup if specified tftp root directories are inaccessible.
- --tftp-unique-root

		Add the IP address of the TFTP client as a path component on the end of the TFTP-root (in standard dotted-quad format). Only valid if a tftp-root is set and the directory exists. For instance, if tftp-root is "/tftp" and client 1.2.3.4 requests file "myfile" then the effective path will be "/tftp/1.2.3.4/myfile" if /tftp/1.2.3.4 exists or /tftp/myfile otherwise.
- --tftp-secure

		Enable TFTP secure mode: without this, any file which is readable by the dnsmasq process under normal unix access-control rules is available via TFTP. When the --tftp-secure flag is given, only files owned by the user running the dnsmasq process are accessible. If dnsmasq is being run as root, different rules apply: --tftp-secure has no effect, but only files which have the world-readable bit set are accessible. It is not recommended to run dnsmasq as root with TFTP enabled, and certainly not without specifying --tftp-root. Doing so can expose any world-readable file on the server to any host on the net.
- --tftp-lowercase

		Convert filenames in TFTP requests to all lowercase. This is useful for requests from Windows machines, which have case-insensitive filesystems and tend to play fast-and-loose with case in filenames. Note that dnsmasq's tftp server always converts "\" to "/" in filenames.
- --tftp-max=<connections>

		Set the maximum number of concurrent TFTP connections allowed. This defaults to 50. When serving a large number of TFTP connections, per-process file descriptor limits may be encountered. Dnsmasq needs one file descriptor for each concurrent TFTP connection and one file descriptor per unique file (plus a few others). So serving the same file simultaneously to n clients will use require about n + 10 file descriptors, serving different files simultaneously to n clients will require about (2*n) + 10 descriptors. If --tftp-port-range is given, that can affect the number of concurrent connections.
- --tftp-mtu=<mtu size>

		Use size as the ceiling of the MTU supported by the intervening network when negotiating TFTP blocksize, overriding the MTU setting of the local interface if it is larger.
- --tftp-no-blocksize

		Stop the TFTP server from negotiating the "blocksize" option with a client. Some buggy clients request this option but then behave badly when it is granted.
- --tftp-port-range=<start>,<end>

		A TFTP server listens on a well-known port (69) for connection initiation, but it also uses a dynamically-allocated port for each connection. Normally these are allocated by the OS, but this option specifies a range of ports for use by TFTP transfers. This can be useful when TFTP has to traverse a firewall. The start of the range cannot be lower than 1025 unless dnsmasq is running as root. The number of concurrent TFTP connections is limited by the size of the port range.