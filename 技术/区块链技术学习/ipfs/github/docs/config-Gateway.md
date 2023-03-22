# Gateway
HTTP 网关的选项。
## Gateway.NoFetch
当设置为 true 时，网关将只提供本地存储库中已有的内容，不会从网络中获取文件。

默认：false

类型：bool
## Gateway.NoDNSLink
一个布尔值，用于配置是否应执行 DNSLink 查找 HostHTTP 标头中的值。如果存在 DNSLink，则存储在 DNS TXT 记录中的内容路径将变为 ，/ 并将相应的有效负载返回给客户端。

默认：false

类型：bool
## Gateway.HTTPHeaders
要在网关响应上设置的标头。

类型：object[string -> array[string]]

默认：

	{
		"Access-Control-Allow-Headers": [
			"X-Requested-With"
		],
		"Access-Control-Allow-Methods": [
			"GET"
		],
		"Access-Control-Allow-Origin": [
			"*"
		]
	}
## Gateway.RootRedirect
将请求重定向到的 url `/`。

默认：""

类型：（`string` 网址）
## Gateway.FastDirIndexThreshold(废弃)
删除：不再需要此选项。自Kubo 0.18以来被忽略 。
## Gateway.Writable(废弃)
已弃用：启用遗留 PUT/POST 请求处理。

此 API 未标准化，不应用于新项目。我们正在研究现代替代品。可以在 [ipfs/specs#375](https://github.com/ipfs/specs/issues/375) 中跟踪 IPIP 。

默认：false

类型：bool
## Gateway.PathPrefixes(废弃)
删除：参见 [go-ipfs#7702](https://github.com/ipfs/go-ipfs/issues/7702)
## Gateway.PublicGateways
`PublicGateways` 是一个字典，用于定义指定主机名上的网关行为。

可以选择使用一个或多个通配符定义主机名。

例子：

- `*.example.com` 将匹配请求到 `http://foo.example.com/ipfs/*` 或 `http://{cid}.ipfs.bar.example.com/*`。
- `foo-*.example.com` 将匹配请求到 `http://foo-bar.example.com/ipfs/*` 或 `http://{cid}.ipfs.foo-xyz.example.com/*`。

### Gateway.PublicGateways: Paths
应在主机名上公开的一组路径。

例子：

	{
	  "Gateway": {
	    "PublicGateways": {
	      "example.com": {
	        "Paths": ["/ipfs", "/ipns"],
	      }
	    }
	  }
	}
以上启用 `http://example.com/ipfs/*` 但 `http://example.com/ipns/*` 不启用 `http://example.com/api/*`

默认：`[]`

类型：`array[string]`

### Gateway.PublicGateways: UseSubdomains
一个布尔值，用于配置主机名处的网关是否提供内容根之间的[源隔离](https://developer.mozilla.org/en-US/docs/Web/Security/Same-origin_policy) 。

- `true` - 启用 [子域网关](https://docs.ipfs.tech/how-to/address-ipfs-on-web/#subdomain-gateway) `http://*.{hostname}/`
	- 需要白名单：确保 `Paths` 设置了相应的白名单。例如，`Paths: ["/ipfs", "/ipns"]` 需要 `http://{cid}.ipfs.{hostname}` 和 `http://{foo}.ipns.{hostname}` 工作：

			"Gateway": {
			    "PublicGateways": {
			        "dweb.link": {
			            "UseSubdomains": true,
			            "Paths": ["/ipfs", "/ipns"],
			        }
			    }
			}
	- 向后兼容：对内容路径的请求，例如 `http://{hostname}/ipfs/{cid}` 重定向到 `http://{cid}.ipfs.{hostname}`
	- API：如果 `/api` 在 Paths 白名单上，`http://{hostname}/api/{cmd}` 则生成重定向到 `http://api.{hostname}/api/{cmd}`
- `false` - 启用路径网关 `http://{hostname}/*`
	- 例子：

			"Gateway": {
			    "PublicGateways": {
			        "ipfs.io": {
			            "UseSubdomains": false,
			            "Paths": ["/ipfs", "/ipns", "/api"],
			        }
			    }
			}

默认：false

类型：bool

### Gateway.PublicGateways: NoDNSLink
一个布尔值，用于配置是否 `Host` 应解析 HTTP 标头中存在的主机名的 DNSLink。覆盖全局设置。如果 `Paths` 已定义，则它们优先于 DNSLink。

默认值：（`false` 默认情况下为每个定义的主机名启用 DNSLink 查找）

类型：bool
### Gateway.PublicGateways: InlineDNSLink
一个可选标志，用于显式配置子域网关的重定向（由 `UseSubdomains: true` 启用）是否应始终将 DNSLink 名称 (FQDN) 内联到单个 DNS 标签中：

	//example.com/ipns/example.net → HTTP 301 → //example-net.ipns.example.com
DNSLink 名称内联允许在具有单标签通配符 TLS 证书的公共子域网关上使用 HTTPS（在传递时也启用），并在 [https://publicsuffix.org](https://publicsuffix.org/) 等特殊规则或浏览器中的自定义本地主机逻辑 `X-Forwarded-Proto: https` 时为每个根 CID 提供不相交的 Origin 必须应用。

默认：false

类型：flag

#### 隐式默认值Gateway.PublicGateways
主机名和环回 IP的默认条目 `localhost` 始终存在。如果为这些主机名提供了额外的配置，它将在隐式值之上合并：

	{
	  "Gateway": {
	    "PublicGateways": {
	      "localhost": {
	        "Paths": ["/ipfs", "/ipns"],
	        "UseSubdomains": true
	      }
	    }
	  }
	}
也可以通过将默认值设置为 来删除默认值 `null`。

例如，要禁用子域网关 `localhost` 并使该主机名与以下行为相同 `127.0.0.1`：

	$ ipfs config --json Gateway.PublicGateways '{"localhost": null }'

## Gateway 列表
以下是最常见的公共网关设置列表。

- 公共[子域网关](https://docs.ipfs.tech/how-to/address-ipfs-on-web/#subdomain-gateway)位于 `http://{cid}.ipfs.dweb.link`（每个内容根都有自己的来源）

		$ ipfs config --json Gateway.PublicGateways '{
		    "dweb.link": {
		      "UseSubdomains": true,
		      "Paths": ["/ipfs", "/ipns"]
		    }
		  }'
	- 向后兼容：此功能支持从内容路径自动重定向到子域：

			http://dweb.link/ipfs/{cid}→http://{cid}.ipfs.dweb.link
	- `X-Forwarded-Proto`

		如果您在提供 TLS 的反向代理后面运行 Kubo，请让它添加 `X-Forwarded-Proto: https` HTTP 标头以确保将用户重定向到 `https://`，而不是 `http://`。它还将确保内联 DNSLink 名称以适应单个 DNS 标签，因此它们可以与 wildcart TLS 证书一起正常工作（[详细信息](https://github.com/ipfs/in-web-browsers/issues/169)）。NGINX 指令是 `proxy_set_header X-Forwarded-Proto "https";`.:

			http://dweb.link/ipfs/{cid}→https://{cid}.ipfs.dweb.link

			http://dweb.link/ipns/your-dnslink.site.example.com→https://your--dnslink-site-example-com.ipfs.dweb.link
	- `X-Forwarded-Host`

		`X-Forwarded-Host: example.com` 如果您想从原始请求中覆盖子域网关主机，我们也支持：

			http://dweb.link/ipfs/{cid}→http://{cid}.ipfs.example.com
- 公共[路径网关](https://docs.ipfs.tech/how-to/address-ipfs-on-web/#path-gateway)处 `http://ipfs.io/ipfs/{cid}`（无 Origin 分离）

		$ ipfs config --json Gateway.PublicGateways '{
		    "ipfs.io": {
		      "UseSubdomains": false,
		      "Paths": ["/ipfs", "/ipns", "/api"]
		    }
		  }'
- 公共 [DNSLink](https://dnslink.io/) 网关解析 `Host` 标头中传递的每个主机名。

		$ ipfs config --json Gateway.NoDNSLink false
	- 请注意，这 `NoDNSLink: false` 是默认设置（除非手动设置，否则开箱即用 `true`）
- 强化的、站点特定的 [DNSLink 网关](https://docs.ipfs.tech/how-to/address-ipfs-on-web/#dnslink-gateway)。

	禁用获取远程数据 ( `NoFetch: true` ) 和解析未知主机名 ( `NoDNSLink: true` ) 的 DNSLink。然后，只为特定的主机名启用 DNSLink 网关（节点上已经存在数据），而不暴露任何内容寻址 `Paths`：

		$ ipfs config --json Gateway.NoFetch true
		$ ipfs config --json Gateway.NoDNSLink true
		$ ipfs config --json Gateway.PublicGateways '{
		    "en.wikipedia-on-ipfs.org": {
		      "NoDNSLink": false,
		      "Paths": []
		    }
		  }'	