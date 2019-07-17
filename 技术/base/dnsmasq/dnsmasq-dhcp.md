- -F
		
		--dhcp-range=[tag:<tag>[,tag:<tag>],][set:<tag>,]<start-addr>[,<end-addr>|<mode>][,<netmask>[,<broadcast>]][,<lease time>]
- -F
		
		--dhcp-range=[tag:<tag>[,tag:<tag>],][set:<tag>,]<start-IPv6addr>[,<end-IPv6addr>|constructor:<interface>][,<mode>][,<prefix-len>][,<lease time>]

		Enable the DHCP server. Addresses will be given out from the range <start-addr> to <end-addr> and from statically defined addresses given in dhcp-host options. If the lease time is given, then leases will be given for that length of time. The lease time is in seconds, or minutes (eg 45m) or hours (eg 1h) or "infinite". If not given, the default lease time is one hour. The minimum lease time is two minutes. For IPv6 ranges, the lease time maybe "deprecated"; this sets the preferred lifetime sent in a DHCP lease or router advertisement to zero, which causes clients to use other addresses, if available, for new connections as a prelude to renumbering.

		This option may be repeated, with different addresses, to enable DHCP service to more than one network. For directly connected networks (ie, networks on which the machine running dnsmasq has an interface) the netmask is optional: dnsmasq will determine it from the interface configuration. For networks which receive DHCP service via a relay agent, dnsmasq cannot determine the netmask itself, so it should be specified, otherwise dnsmasq will have to guess, based on the class (A, B or C) of the network address. The broadcast address is always optional. It is always allowed to have more than one dhcp-range in a single subnet.

		For IPv6, the parameters are slightly different: instead of netmask and broadcast address, there is an optional prefix length which must be equal to or larger then the prefix length on the local interface. If not given, this defaults to 64. Unlike the IPv4 case, the prefix length is not automatically derived from the interface configuration. The mimimum size of the prefix length is 64.

		IPv6 (only) supports another type of range. In this, the start address and optional end address contain only the network part (ie ::1) and they are followed by constructor:<interface>. This forms a template which describes how to create ranges, based on the addresses assigned to the interface. For instance

- --dhcp-range=::1,::400,constructor:eth0

		will look for addresses on eth0 and then create a range from <network>::1 to <network>::400. If the interface is assigned more than one network, then the corresponding ranges will be automatically created, and then deprecated and finally removed again as the address is deprecated and then deleted. The interface name may have a final "*" wildcard. Note that just any address on eth0 will not do: it must not be an autoconfigured or privacy address, or be deprecated.

		If a dhcp-range is only being used for stateless DHCP and/or SLAAC, then the address can be simply ::

- --dhcp-range=::,constructor:eth0

		The optional set:<tag> sets an alphanumeric label which marks this network so that dhcp options may be specified on a per-network basis. When it is prefixed with 'tag:' instead, then its meaning changes from setting a tag to matching it. Only one tag may be set, but more than one tag may be matched.

		The optional <mode> keyword may be static which tells dnsmasq to enable DHCP for the network specified, but not to dynamically allocate IP addresses: only hosts which have static addresses given via dhcp-host or from /etc/ethers will be served. A static-only subnet with address all zeros may be used as a "catch-all" address to enable replies to all Information-request packets on a subnet which is provided with stateless DHCPv6, ie --dhcp-range=::,static

		For IPv4, the <mode> may be proxy in which case dnsmasq will provide proxy-DHCP on the specified subnet. (See pxe-prompt and pxe-service for details.)

		For IPv6, the mode may be some combination of ra-only, slaac, ra-names, ra-stateless, ra-advrouter, off-link.

		ra-only tells dnsmasq to offer Router Advertisement only on this subnet, and not DHCP.

		slaac tells dnsmasq to offer Router Advertisement on this subnet and to set the A bit in the router advertisement, so that the client will use SLAAC addresses. When used with a DHCP range or static DHCP address this results in the client having both a DHCP-assigned and a SLAAC address.

		ra-stateless sends router advertisements with the O and A bits set, and provides a stateless DHCP service. The client will use a SLAAC address, and use DHCP for other configuration information.

		ra-names enables a mode which gives DNS names to dual-stack hosts which do SLAAC for IPv6. Dnsmasq uses the host's IPv4 lease to derive the name, network segment and MAC address and assumes that the host will also have an IPv6 address calculated using the SLAAC algorithm, on the same network segment. The address is pinged, and if a reply is received, an AAAA record is added to the DNS for this IPv6 address. Note that this is only happens for directly-connected networks, (not one doing DHCP via a relay) and it will not work if a host is using privacy extensions. ra-names can be combined with ra-stateless and slaac.

		ra-advrouter enables a mode where router address(es) rather than prefix(es) are included in the advertisements. This is described in RFC-3775 section 7.2 and is used in mobile IPv6. In this mode the interval option is also included, as described in RFC-3775 section 7.3.

		off-link tells dnsmasq to advertise the prefix without the on-link (aka L) bit set.

- -G
		
		--dhcp-host=[<hwaddr>][,id:<client_id>|*][,set:<tag>][,<ipaddr>][,<hostname>][,<lease_time>][,ignore]
		
		Specify per host parameters for the DHCP server. This allows a machine with a particular hardware address to be always allocated the same hostname, IP address and lease time. A hostname specified like this overrides any supplied by the DHCP client on the machine. It is also allowable to omit the hardware address and include the hostname, in which case the IP address and lease times will apply to any machine claiming that name. For example --dhcp-host=00:20:e0:3b:13:af,wap,infinite tells dnsmasq to give the machine with hardware address 00:20:e0:3b:13:af the name wap, and an infinite DHCP lease. --dhcp-host=lap,192.168.0.199 tells dnsmasq to always allocate the machine lap the IP address 192.168.0.199.
		Addresses allocated like this are not constrained to be in the range given by the --dhcp-range option, but they must be in the same subnet as some valid dhcp-range. For subnets which don't need a pool of dynamically allocated addresses, use the "static" keyword in the dhcp-range declaration.

		It is allowed to use client identifiers (called client DUID in IPv6-land rather than hardware addresses to identify hosts by prefixing with 'id:'. Thus: --dhcp-host=id:01:02:03:04,..... refers to the host with client identifier 01:02:03:04. It is also allowed to specify the client ID as text, like this: --dhcp-host=id:clientidastext,.....

		A single dhcp-host may contain an IPv4 address or an IPv6 address, or both. IPv6 addresses must be bracketed by square brackets thus: --dhcp-host=laptop,[1234::56] IPv6 addresses may contain only the host-identifier part: --dhcp-host=laptop,[::56] in which case they act as wildcards in constructed dhcp ranges, with the appropriate network part inserted. Note that in IPv6 DHCP, the hardware address may not be available, though it normally is for direct-connected clients, or clients using DHCP relays which support RFC 6939.

		For DHCPv4, the special option id:* means "ignore any client-id and use MAC addresses only." This is useful when a client presents a client-id sometimes but not others.

		If a name appears in /etc/hosts, the associated address can be allocated to a DHCP lease, but only if a --dhcp-host option specifying the name also exists. Only one hostname can be given in a dhcp-host option, but aliases are possible by using CNAMEs. (See --cname ).

		The special keyword "ignore" tells dnsmasq to never offer a DHCP lease to a machine. The machine can be specified by hardware address, client ID or hostname, for instance --dhcp-host=00:20:e0:3b:13:af,ignore This is useful when there is another DHCP server on the network which should be used by some machines.

		The set:<tag> construct sets the tag whenever this dhcp-host directive is in use. This can be used to selectively send DHCP options just for this host. More than one tag can be set in a dhcp-host directive (but not in other places where "set:<tag>" is allowed). When a host matches any dhcp-host directive (or one implied by /etc/ethers) then the special tag "known" is set. This allows dnsmasq to be configured to ignore requests from unknown machines using --dhcp-ignore=tag:!known Ethernet addresses (but not client-ids) may have wildcard bytes, so for example --dhcp-host=00:20:e0:3b:13:*,ignore will cause dnsmasq to ignore a range of hardware addresses. Note that the "*" will need to be escaped or quoted on a command line, but not in the configuration file.

		Hardware addresses normally match any network (ARP) type, but it is possible to restrict them to a single ARP type by preceding them with the ARP-type (in HEX) and "-". so --dhcp-host=06-00:20:e0:3b:13:af,1.2.3.4 will only match a Token-Ring hardware address, since the ARP-address type for token ring is 6.

		As a special case, in DHCPv4, it is possible to include more than one hardware address. eg: --dhcp-host=11:22:33:44:55:66,12:34:56:78:90:12,192.168.0.2 This allows an IP address to be associated with multiple hardware addresses, and gives dnsmasq permission to abandon a DHCP lease to one of the hardware addresses when another one asks for a lease. Beware that this is a dangerous thing to do, it will only work reliably if only one of the hardware addresses is active at any time and there is no way for dnsmasq to enforce this. It is, for instance, useful to allocate a stable IP address to a laptop which has both wired and wireless interfaces.
- --dhcp-hostsfile=<path>

		Read DHCP host information from the specified file. If a directory is given, then read all the files contained in that directory. The file contains information about one host per line. The format of a line is the same as text to the right of '=' in --dhcp-host. The advantage of storing DHCP host information in this file is that it can be changed without re-starting dnsmasq: the file will be re-read when dnsmasq receives SIGHUP.
- --dhcp-optsfile=<path>

		Read DHCP option information from the specified file. If a directory is given, then read all the files contained in that directory. The advantage of using this option is the same as for --dhcp-hostsfile: the dhcp-optsfile will be re-read when dnsmasq receives SIGHUP. Note that it is possible to encode the information in a
- --dhcp-hostsdir=<path>

		This is equivalent to dhcp-hostsfile, except for the following. The path MUST be a directory, and not an individual file. Changed or new files within the directory are read automatically, without the need to send SIGHUP. If a file is deleted for changed after it has been read by dnsmasq, then the host record it contained will remain until dnsmasq recieves a SIGHUP, or is restarted; ie host records are only added dynamically.
- --dhcp-optsdir=<path>

		This is equivalent to dhcp-optsfile, with the differences noted for --dhcp-hostsdir.
- --dhcp-boot

		flag as DHCP options, using the options names bootfile-name, server-ip-address and tftp-server. This allows these to be included in a dhcp-optsfile.
- -Z, --read-ethers

		Read /etc/ethers for information about hosts for the DHCP server. The format of /etc/ethers is a hardware address, followed by either a hostname or dotted-quad IP address. When read by dnsmasq these lines have exactly the same effect as --dhcp-host options containing the same information. /etc/ethers is re-read when dnsmasq receives SIGHUP. IPv6 addresses are NOT read from /etc/ethers.
- -O

		--dhcp-option=[tag:<tag>,[tag:<tag>,]][encap:<opt>,][vi-encap:<enterprise>,][vendor:[<vendor-class>],][<opt>|option:<opt-name>|option6:<opt>|option6:<opt-name>],[<value>[,<value>]]

		Specify different or extra options to DHCP clients. By default, dnsmasq sends some standard options to DHCP clients, the netmask and broadcast address are set to the same as the host running dnsmasq, and the DNS server and default route are set to the address of the machine running dnsmasq. (Equivalent rules apply for IPv6.) If the domain name option has been set, that is sent. This configuration allows these defaults to be overridden, or other options specified. The option, to be sent may be given as a decimal number or as "option:<option-name>" The option numbers are specified in RFC2132 and subsequent RFCs. The set of option-names known by dnsmasq can be discovered by running "dnsmasq --help dhcp". For example, to set the default route option to 192.168.4.4, do --dhcp-option=3,192.168.4.4 or --dhcp-option = option:router, 192.168.4.4 and to set the time-server address to 192.168.0.4, do --dhcp-option = 42,192.168.0.4 or --dhcp-option = option:ntp-server, 192.168.0.4 The special address 0.0.0.0 is taken to mean "the address of the machine running dnsmasq".
		Data types allowed are comma separated dotted-quad IPv4 addresses, []-wrapped IPv6 addresses, a decimal number, colon-separated hex digits and a text string. If the optional tags are given then this option is only sent when all the tags are matched.

		Special processing is done on a text argument for option 119, to conform with RFC 3397. Text or dotted-quad IP addresses as arguments to option 120 are handled as per RFC 3361. Dotted-quad IP addresses which are followed by a slash and then a netmask size are encoded as described in RFC 3442.

		IPv6 options are specified using the option6: keyword, followed by the option number or option name. The IPv6 option name space is disjoint from the IPv4 option name space. IPv6 addresses in options must be bracketed with square brackets, eg. --dhcp-option=option6:ntp-server,[1234::56] For IPv6, [::] means "the global address of the machine running dnsmasq", whilst [fd00::] is replaced with the ULA, if it exists, and [fe80::] with the link-local address.

		Be careful: no checking is done that the correct type of data for the option number is sent, it is quite possible to persuade dnsmasq to generate illegal DHCP packets with injudicious use of this flag. When the value is a decimal number, dnsmasq must determine how large the data item is. It does this by examining the option number and/or the value, but can be overridden by appending a single letter flag as follows: b = one byte, s = two bytes, i = four bytes. This is mainly useful with encapsulated vendor class options (see below) where dnsmasq cannot determine data size from the option number. Option data which consists solely of periods and digits will be interpreted by dnsmasq as an IP address, and inserted into an option as such. To force a literal string, use quotes. For instance when using option 66 to send a literal IP address as TFTP server name, it is necessary to do --dhcp-option=66,"1.2.3.4"

		Encapsulated Vendor-class options may also be specified (IPv4 only) using --dhcp-option: for instance --dhcp-option=vendor:PXEClient,1,0.0.0.0 sends the encapsulated vendor class-specific option "mftp-address=0.0.0.0" to any client whose vendor-class matches "PXEClient". The vendor-class matching is substring based (see --dhcp-vendorclass for details). If a vendor-class option (number 60) is sent by dnsmasq, then that is used for selecting encapsulated options in preference to any sent by the client. It is possible to omit the vendorclass completely; --dhcp-option=vendor:,1,0.0.0.0 in which case the encapsulated option is always sent.

		Options may be encapsulated (IPv4 only) within other options: for instance --dhcp-option=encap:175, 190, iscsi-client0 will send option 175, within which is the option 190. If multiple options are given which are encapsulated with the same option number then they will be correctly combined into one encapsulated option. encap: and vendor: are may not both be set in the same dhcp-option.

		The final variant on encapsulated options is "Vendor-Identifying Vendor Options" as specified by RFC3925. These are denoted like this: --dhcp-option=vi-encap:2, 10, text The number in the vi-encap: section is the IANA enterprise number used to identify this option. This form of encapsulation is supported in IPv6. The address 0.0.0.0 is not treated specially in encapsulated options.

- ? 

		--dhcp-option-force=[tag:<tag>,[tag:<tag>,]][encap:<opt>,][vi-encap:<enterprise>,][vendor:[<vendor-class>],]<opt>,[<value>[,<value>]]

		This works in exactly the same way as --dhcp-option except that the option will always be sent, even if the client does not ask for it in the parameter request list. This is sometimes needed, for example when sending options to PXELinux.
- --dhcp-no-override

		(IPv4 only) Disable re-use of the DHCP servername and filename fields as extra option space. If it can, dnsmasq moves the boot server and filename information (from dhcp-boot) out of their dedicated fields into DHCP options. This make extra space available in the DHCP packet for options but can, rarely, confuse old or broken clients. This flag forces "simple and safe" behaviour to avoid problems in such a case.
- --dhcp-relay=<local address>,<server address>[,<interface]

		Configure dnsmasq to do DHCP relay. The local address is an address allocated to an interface on the host running dnsmasq. All DHCP requests arriving on that interface will we relayed to a remote DHCP server at the server address. It is possible to relay from a single local address to multiple remote servers by using multiple dhcp-relay configs with the same local address and different server addresses. A server address must be an IP literal address, not a domain name. In the case of DHCPv6, the server address may be the ALL_SERVERS multicast address, ff05::1:3. In this case the interface must be given, not be wildcard, and is used to direct the multicast to the correct interface to reach the DHCP server.
		Access control for DHCP clients has the same rules as for the DHCP server, see --interface, --except-interface, etc. The optional interface name in the dhcp-relay config has a different function: it controls on which interface DHCP replies from the server will be accepted. This is intended for configurations which have three interfaces: one being relayed from, a second connecting the DHCP server, and a third untrusted network, typically the wider internet. It avoids the possibility of spoof replies arriving via this third interface.

		It is allowed to have dnsmasq act as a DHCP server on one set of interfaces and relay from a disjoint set of interfaces. Note that whilst it is quite possible to write configurations which appear to act as a server and a relay on the same interface, this is not supported: the relay function will take precedence.

		Both DHCPv4 and DHCPv6 relay is supported. It's not possible to relay DHCPv4 to a DHCPv6 server or vice-versa.

- -U, --dhcp-vendorclass=set:<tag>,[enterprise:<IANA-enterprise number>,]<vendor-class>
				
		Map from a vendor-class string to a tag. Most DHCP clients provide a "vendor class" which represents, in some sense, the type of host. This option maps vendor classes to tags, so that DHCP options may be selectively delivered to different classes of hosts. For example dhcp-vendorclass=set:printers,Hewlett-Packard JetDirect will allow options to be set only for HP printers like so: --dhcp-option=tag:printers,3,192.168.4.4 The vendor-class string is substring matched against the vendor-class supplied by the client, to allow fuzzy matching. The set: prefix is optional but allowed for consistency.

		Note that in IPv6 only, vendorclasses are namespaced with an IANA-allocated enterprise number. This is given with enterprise: keyword and specifies that only vendorclasses matching the specified number should be searched.

- -j, --dhcp-userclass=set:<tag>,<user-class>

		Map from a user-class string to a tag (with substring matching, like vendor classes). Most DHCP clients provide a "user class" which is configurable. This option maps user classes to tags, so that DHCP options may be selectively delivered to different classes of hosts. It is possible, for instance to use this to set a different printer server for hosts in the class "accounts" than for hosts in the class "engineering".
- -4, --dhcp-mac=set:<tag>,<MAC address>

		Map from a MAC address to a tag. The MAC address may include wildcards. For example --dhcp-mac=set:3com,01:34:23:*:*:* will set the tag "3com" for any host whose MAC address matches the pattern.
- --dhcp-circuitid=set:<tag>,<circuit-id>, --dhcp-remoteid=set:<tag>,<remote-id>

		Map from RFC3046 relay agent options to tags. This data may be provided by DHCP relay agents. The circuit-id or remote-id is normally given as colon-separated hex, but is also allowed to be a simple string. If an exact match is achieved between the circuit or agent ID and one provided by a relay agent, the tag is set.
		dhcp-remoteid (but not dhcp-circuitid) is supported in IPv6.

- --dhcp-subscrid=set:<tag>,<subscriber-id>

		(IPv4 and IPv6) Map from RFC3993 subscriber-id relay agent options to tags.
- --dhcp-proxy[=<ip addr>]......

		(IPv4 only) A normal DHCP relay agent is only used to forward the initial parts of a DHCP interaction to the DHCP server. Once a client is configured, it communicates directly with the server. This is undesirable if the relay agent is adding extra information to the DHCP packets, such as that used by dhcp-circuitid and dhcp-remoteid. A full relay implementation can use the RFC 5107 serverid-override option to force the DHCP server to use the relay as a full proxy, with all packets passing through it. This flag provides an alternative method of doing the same thing, for relays which don't support RFC 5107. Given alone, it manipulates the server-id for all interactions via relays. If a list of IP addresses is given, only interactions via relays at those addresses are affected.
- --dhcp-match=set:<tag>,<option number>|option:<option name>|vi-encap:<enterprise>[,<value>]

		Without a value, set the tag if the client sends a DHCP option of the given number or name. When a value is given, set the tag only if the option is sent and matches the value. The value may be of the form "01:ff:*:02" in which case the value must match (apart from wildcards) but the option sent may have unmatched data past the end of the value. The value may also be of the same form as in dhcp-option in which case the option sent is treated as an array, and one element must match, so
- --dhcp-match=set:efi-ia32,option:client-arch,6

		will set the tag "efi-ia32" if the the number 6 appears in the list of architectures sent by the client in option 93. (See RFC 4578 for details.) If the value is a string, substring matching is used.

		The special form with vi-encap:<enterprise number> matches against vendor-identifying vendor classes for the specified enterprise. Please see RFC 3925 for more details of these rare and interesting beasts.

- --tag-if=set:<tag>[,set:<tag>[,tag:<tag>[,tag:<tag>]]]

		Perform boolean operations on tags. Any tag appearing as set:<tag> is set if all the tags which appear as tag:<tag> are set, (or unset when tag:!<tag> is used) If no tag:<tag> appears set:<tag> tags are set unconditionally. Any number of set: and tag: forms may appear, in any order. Tag-if lines ares executed in order, so if the tag in tag:<tag> is a tag set by another tag-if, the line which sets the tag must precede the one which tests it.
- -J, --dhcp-ignore=tag:<tag>[,tag:<tag>]

		When all the given tags appear in the tag set ignore the host and do not allocate it a DHCP lease.
- --dhcp-ignore-names[=tag:<tag>[,tag:<tag>]]

		When all the given tags appear in the tag set, ignore any hostname provided by the host. Note that, unlike dhcp-ignore, it is permissible to supply no tags, in which case DHCP-client supplied hostnames are always ignored, and DHCP hosts are added to the DNS using only dhcp-host configuration in dnsmasq and the contents of /etc/hosts and /etc/ethers.
- --dhcp-generate-names=tag:<tag>[,tag:<tag>]

		(IPv4 only) Generate a name for DHCP clients which do not otherwise have one, using the MAC address expressed in hex, separated by dashes. Note that if a host provides a name, it will be used by preference to this, unless --dhcp-ignore-names is set.
- --dhcp-broadcast[=tag:<tag>[,tag:<tag>]]

		(IPv4 only) When all the given tags appear in the tag set, always use broadcast to communicate with the host when it is unconfigured. It is permissible to supply no tags, in which case this is unconditional. Most DHCP clients which need broadcast replies set a flag in their requests so that this happens automatically, some old BOOTP clients do not.
- -M, --dhcp-boot=[tag:<tag>,]<filename>,[<servername>[,<server address>|<tftp_servername>]]

		(IPv4 only) Set BOOTP options to be returned by the DHCP server. Server name and address are optional: if not provided, the name is left empty, and the address set to the address of the machine running dnsmasq. If dnsmasq is providing a TFTP service (see --enable-tftp ) then only the filename is required here to enable network booting. If the optional tag(s) are given, they must match for this configuration to be sent. Instead of an IP address, the TFTP server address can be given as a domain name which is looked up in /etc/hosts. This name can be associated in /etc/hosts with multiple IP addresses, which are used round-robin. This facility can be used to load balance the tftp load among a set of servers.
- --dhcp-sequential-ip
		
		Dnsmasq is designed to choose IP addresses for DHCP clients using a hash of the client's MAC address. This normally allows a client's address to remain stable long-term, even if the client sometimes allows its DHCP lease to expire. In this default mode IP addresses are distributed pseudo-randomly over the entire available address range. There are sometimes circumstances (typically server deployment) where it is more convenient to have IP addresses allocated sequentially, starting from the lowest available address, and setting this flag enables this mode. Note that in the sequential mode, clients which allow a lease to expire are much more likely to move IP address; for this reason it should not be generally used.
- --pxe-service=[tag:<tag>,]<CSA>,<menu text>[,<basename>|<bootservicetype>][,<server address>|<server_name>]

		Most uses of PXE boot-ROMS simply allow the PXE system to obtain an IP address and then download the file specified by dhcp-boot and execute it. However the PXE system is capable of more complex functions when supported by a suitable DHCP server.
		This specifies a boot option which may appear in a PXE boot menu. <CSA> is client system type, only services of the correct type will appear in a menu. The known types are x86PC, PC98, IA64_EFI, Alpha, Arc_x86, Intel_Lean_Client, IA32_EFI, X86-64_EFI, Xscale_EFI, BC_EFI, ARM32_EFI and ARM64_EFI; an integer may be used for other types. The parameter after the menu text may be a file name, in which case dnsmasq acts as a boot server and directs the PXE client to download the file by TFTP, either from itself ( enable-tftp must be set for this to work) or another TFTP server if the final server address/name is given. Note that the "layer" suffix (normally ".0") is supplied by PXE, and need not be added to the basename. Alternatively, the basename may be a filename, complete with suffix, in which case no layer suffix is added. If an integer boot service type, rather than a basename is given, then the PXE client will search for a suitable boot service for that type on the network. This search may be done by broadcast, or direct to a server if its IP address/name is provided. If no boot service type or filename is provided (or a boot service type of 0 is specified) then the menu entry will abort the net boot procedure and continue booting from local media. The server address can be given as a domain name which is looked up in /etc/hosts. This name can be associated in /etc/hosts with multiple IP addresses, which are used round-robin.

- --pxe-prompt=[tag:<tag>,]<prompt>[,<timeout>]

		Setting this provides a prompt to be displayed after PXE boot. If the timeout is given then after the timeout has elapsed with no keyboard input, the first available menu option will be automatically executed. If the timeout is zero then the first available menu item will be executed immediately. If pxe-prompt is omitted the system will wait for user input if there are multiple items in the menu, but boot immediately if there is only one. See pxe-service for details of menu items.
		Dnsmasq supports PXE "proxy-DHCP", in this case another DHCP server on the network is responsible for allocating IP addresses, and dnsmasq simply provides the information given in pxe-prompt and pxe-service to allow netbooting. This mode is enabled using the proxy keyword in dhcp-range.

- -X, --dhcp-lease-max=<number>

		Limits dnsmasq to the specified maximum number of DHCP leases. The default is 1000. This limit is to prevent DoS attacks from hosts which create thousands of leases and use lots of memory in the dnsmasq process.
- -K, --dhcp-authoritative

		Should be set when dnsmasq is definitely the only DHCP server on a network. For DHCPv4, it changes the behaviour from strict RFC compliance so that DHCP requests on unknown leases from unknown hosts are not ignored. This allows new hosts to get a lease without a tedious timeout under all circumstances. It also allows dnsmasq to rebuild its lease database without each client needing to reacquire a lease, if the database is lost. For DHCPv6 it sets the priority in replies to 255 (the maximum) instead of 0 (the minimum).
- --dhcp-alternate-port[=<server port>[,<client port>]]

		(IPv4 only) Change the ports used for DHCP from the default. If this option is given alone, without arguments, it changes the ports used for DHCP from 67 and 68 to 1067 and 1068. If a single argument is given, that port number is used for the server and the port number plus one used for the client. Finally, two port numbers allows arbitrary specification of both server and client ports for DHCP.
- -3, --bootp-dynamic[=<network-id>[,<network-id>]]

		(IPv4 only) Enable dynamic allocation of IP addresses to BOOTP clients. Use this with care, since each address allocated to a BOOTP client is leased forever, and therefore becomes permanently unavailable for re-use by other hosts. if this is given without tags, then it unconditionally enables dynamic allocation. With tags, only when the tags are all set. It may be repeated with different tag sets.
- -5, --no-ping
	
		(IPv4 only) By default, the DHCP server will attempt to ensure that an address is not in use before allocating it to a host. It does this by sending an ICMP echo request (aka "ping") to the address in question. If it gets a reply, then the address must already be in use, and another is tried. This flag disables this check. Use with caution.
- --log-dhcp

		Extra logging for DHCP: log all the options sent to DHCP clients and the tags used to determine them.
- --quiet-dhcp, --quiet-dhcp6, --quiet-ra

		Suppress logging of the routine operation of these protocols. Errors and problems will still be logged. --quiet-dhcp and quiet-dhcp6 are over-ridden by --log-dhcp.
- -l, --dhcp-leasefile=<path>

		Use the specified file to store DHCP lease information.
- --dhcp-duid=<enterprise-id>,<uid>

		(IPv6 only) Specify the server persistent UID which the DHCPv6 server will use. This option is not normally required as dnsmasq creates a DUID automatically when it is first needed. When given, this option provides dnsmasq the data required to create a DUID-EN type DUID. Note that once set, the DUID is stored in the lease database, so to change between DUID-EN and automatically created DUIDs or vice-versa, the lease database must be re-intialised. The enterprise-id is assigned by IANA, and the uid is a string of hex octets unique to a particular device.
- -6 --dhcp-script=<path>

		Whenever a new DHCP lease is created, or an old one destroyed, or a TFTP file transfer completes, the executable specified by this option is run. <path> must be an absolute pathname, no PATH search occurs. The arguments to the process are "add", "old" or "del", the MAC address of the host (or DUID for IPv6) , the IP address, and the hostname, if known. "add" means a lease has been created, "del" means it has been destroyed, "old" is a notification of an existing lease when dnsmasq starts or a change to MAC address or hostname of an existing lease (also, lease length or expiry and client-id, if leasefile-ro is set). If the MAC address is from a network type other than ethernet, it will have the network type prepended, eg "06-01:23:45:67:89:ab" for token ring. The process is run as root (assuming that dnsmasq was originally run as root) even if dnsmasq is configured to change UID to an unprivileged user.
		The environment is inherited from the invoker of dnsmasq, with some or all of the following variables added
		For both IPv4 and IPv6:
		DNSMASQ_DOMAIN if the fully-qualified domain name of the host is known, this is set to the domain part. (Note that the hostname passed to the script as an argument is never fully-qualified.)
		If the client provides a hostname, DNSMASQ_SUPPLIED_HOSTNAME
		If the client provides user-classes, DNSMASQ_USER_CLASS0..DNSMASQ_USER_CLASSn
		If dnsmasq was compiled with HAVE_BROKEN_RTC, then the length of the lease (in seconds) is stored in DNSMASQ_LEASE_LENGTH, otherwise the time of lease expiry is stored in DNSMASQ_LEASE_EXPIRES. The number of seconds until lease expiry is always stored in DNSMASQ_TIME_REMAINING.
		If a lease used to have a hostname, which is removed, an "old" event is generated with the new state of the lease, ie no name, and the former name is provided in the environment variable DNSMASQ_OLD_HOSTNAME.
		DNSMASQ_INTERFACE stores the name of the interface on which the request arrived; this is not set for "old" actions when dnsmasq restarts.
		DNSMASQ_RELAY_ADDRESS is set if the client used a DHCP relay to contact dnsmasq and the IP address of the relay is known.
		DNSMASQ_TAGS contains all the tags set during the DHCP transaction, separated by spaces.
		DNSMASQ_LOG_DHCP is set if --log-dhcp is in effect.

		For IPv4 only:

		DNSMASQ_CLIENT_ID if the host provided a client-id.

		DNSMASQ_CIRCUIT_ID, DNSMASQ_SUBSCRIBER_ID, DNSMASQ_REMOTE_ID if a DHCP relay-agent added any of these options. 
  		If the client provides vendor-class, DNSMASQ_VENDOR_CLASS.

		For IPv6 only:

		If the client provides vendor-class, DNSMASQ_VENDOR_CLASS_ID, containing the IANA enterprise id for the class, and DNSMASQ_VENDOR_CLASS0..DNSMASQ_VENDOR_CLASSn for the data.

		DNSMASQ_SERVER_DUID containing the DUID of the server: this is the same for every call to the script.

		DNSMASQ_IAID containing the IAID for the lease. If the lease is a temporary allocation, this is prefixed to 'T'.

		DNSMASQ_MAC containing the MAC address of the client, if known.

		Note that the supplied hostname, vendorclass and userclass data is only supplied for "add" actions or "old" actions when a host resumes an existing lease, since these data are not held in dnsmasq's lease database.

		All file descriptors are closed except stdin, stdout and stderr which are open to /dev/null (except in debug mode).

		The script is not invoked concurrently: at most one instance of the script is ever running (dnsmasq waits for an instance of script to exit before running the next). Changes to the lease database are which require the script to be invoked are queued awaiting exit of a running instance. If this queueing allows multiple state changes occur to a single lease before the script can be run then earlier states are discarded and the current state of that lease is reflected when the script finally runs.

		At dnsmasq startup, the script will be invoked for all existing leases as they are read from the lease file. Expired leases will be called with "del" and others with "old". When dnsmasq receives a HUP signal, the script will be invoked for existing leases with an "old" event.

		There are four further actions which may appear as the first argument to the script, "init", "arp-add", "arp-del" and "tftp". More may be added in the future, so scripts should be written to ignore unknown actions. "init" is described below in --leasefile-ro The "tftp" action is invoked when a TFTP file transfer completes: the arguments are the file size in bytes, the address to which the file was sent, and the complete pathname of the file. 
		The "arp-add" and "arp-del" actions are only called if enabled with --script-arp They are are supplied with a MAC address and IP address as arguments. "arp-add" indicates the arrival of a new entry in the ARP or neighbour table, and "arp-del" indicates the deletion of same.

- --dhcp-luascript=<path>

		Specify a script written in Lua, to be run when leases are created, destroyed or changed. To use this option, dnsmasq must be compiled with the correct support. The Lua interpreter is intialised once, when dnsmasq starts, so that global variables persist between lease events. The Lua code must define a lease function, and may provide init and shutdown functions, which are called, without arguments when dnsmasq starts up and terminates. It may also provide a tftp function.
		The lease function receives the information detailed in --dhcp-script. It gets two arguments, firstly the action, which is a string containing, "add", "old" or "del", and secondly a table of tag value pairs. The tags mostly correspond to the environment variables detailed above, for instance the tag "domain" holds the same data as the environment variable DNSMASQ_DOMAIN. There are a few extra tags which hold the data supplied as arguments to --dhcp-script. These are mac_address, ip_address and hostname for IPv4, and client_duid, ip_address and hostname for IPv6.

		The tftp function is called in the same way as the lease function, and the table holds the tags destination_address, file_name and file_size.

		The arp and arp-old functions are called only when enabled with --script-arp and have a table which holds the tags mac_addres and client_address.
- --dhcp-scriptuser

		Specify the user as which to run the lease-change script or Lua script. This defaults to root, but can be changed to another user using this flag.
- --script-arp

		Enable the "arp" and "arp-old" functions in the dhcp-script and dhcp-luascript.
- -9, --leasefile-ro

		Completely suppress use of the lease database file. The file will not be created, read, or written. Change the way the lease-change script (if one is provided) is called, so that the lease database may be maintained in external storage by the script. In addition to the invocations given in --dhcp-script the lease-change script is called once, at dnsmasq startup, with the single argument "init". When called like this the script should write the saved state of the lease database, in dnsmasq leasefile format, to stdout and exit with zero exit code. Setting this option also forces the leasechange script to be called on changes to the client-id and lease length and expiry time.
- --bridge-interface=<interface>,<alias>[,<alias>]

		Treat DHCP (v4 and v6) request and IPv6 Router Solicit packets arriving at any of the <alias> interfaces as if they had arrived at <interface>. This option allows dnsmasq to provide DHCP and RA service over unaddressed and unbridged Ethernet interfaces, e.g. on an OpenStack compute host where each such interface is a TAP interface to a VM, or as in "old style bridging" on BSD platforms. A trailing '*' wildcard can be used in each <alias>.
- -s, --domain=<domain>[,<address range>[,local]]

		Specifies DNS domains for the DHCP server. Domains may be be given unconditionally (without the IP range) or for limited IP ranges. This has two effects; firstly it causes the DHCP server to return the domain to any hosts which request it, and secondly it sets the domain which it is legal for DHCP-configured hosts to claim. The intention is to constrain hostnames so that an untrusted host on the LAN cannot advertise its name via dhcp as e.g. "microsoft.com" and capture traffic not meant for it. If no domain suffix is specified, then any DHCP hostname with a domain part (ie with a period) will be disallowed and logged. If suffix is specified, then hostnames with a domain part are allowed, provided the domain part matches the suffix. In addition, when a suffix is set then hostnames without a domain part have the suffix added as an optional domain part. Eg on my network I can set --domain=thekelleys.org.uk and have a machine whose DHCP hostname is "laptop". The IP address for that machine is available from dnsmasq both as "laptop" and "laptop.thekelleys.org.uk". If the domain is given as "#" then the domain is read from the first "search" directive in /etc/resolv.conf (or equivalent).
		The address range can be of the form <ip address>,<ip address> or <ip address>/<netmask> or just a single <ip address>. See --dhcp-fqdn which can change the behaviour of dnsmasq with domains.

		If the address range is given as ip-address/network-size, then a additional flag "local" may be supplied which has the effect of adding --local declarations for forward and reverse DNS queries. Eg. --domain=thekelleys.org.uk,192.168.0.0/24,local is identical to --domain=thekelleys.org.uk,192.168.0.0/24 --local=/thekelleys.org.uk/ --local=/0.168.192.in-addr.arpa/ The network size must be 8, 16 or 24 for this to be legal.
- --dhcp-fqdn

		In the default mode, dnsmasq inserts the unqualified names of DHCP clients into the DNS. For this reason, the names must be unique, even if two clients which have the same name are in different domains. If a second DHCP client appears which has the same name as an existing client, the name is transferred to the new client. If --dhcp-fqdn is set, this behaviour changes: the unqualified name is no longer put in the DNS, only the qualified name. Two DHCP clients with the same name may both keep the name, provided that the domain part is different (ie the fully qualified names differ.) To ensure that all names have a domain part, there must be at least --domain without an address specified when --dhcp-fqdn is set.
- --dhcp-client-update
	
		Normally, when giving a DHCP lease, dnsmasq sets flags in the FQDN option to tell the client not to attempt a DDNS update with its name and IP address. This is because the name-IP pair is automatically added into dnsmasq's DNS view. This flag suppresses that behaviour, this is useful, for instance, to allow Windows clients to update Active Directory servers. See RFC 4702 for details.