# 在网络上解决 IPFS
如何链接到 IPFS 上的内容。

	https://ipfs.io/ipfs/<CID>
	# e.g
	https://ipfs.io/ipfs/Qme7ss3ARVgxv6rXqVPiikMJ8u2NLgmgszg13pYrDKEoiu
支持 IPFS 的浏览器可以将这些请求重定向到您的本地 IPFS 节点，而那些不支持的浏览器可以从 ipfs.io 网关获取资源。

您可以换出您ipfs.io自己的 http-to-ipfs 网关，但您必须让该网关永远运行。如果您的网关出现故障，具有 IPFS 感知工具的用户仍然能够从 IPFS 网络获取内容，只要任何节点仍然托管它，但对于那些没有节点的用户，链接将会断开。不要那样做。

Dweb寻址简介
在 IPFS 中，地址（用于内容）是类似路径的；它们是由斜杠分隔的组件。
第一个组件是协议，它告诉您如何解释它之后的所有内容。
哈希引用的内容可能具有命名链接。（例如，一个 Git 提交有一个名为 的链接parent，它实际上只是指向另一个 Git 提交的哈希的指针。）IPFS 地址中 CID 之后的所有内容都是那些命名链接。
由于这些地址不是 URL，因此在 Web 浏览器中使用它们需要稍微重新格式化它们：
通过 HTTP 网关，如http://<gateway host>/<IPFS address>
通过网关子域（更安全，更难设置）：http://<cid>.ipfs.<gateway host>/<path>，因此协议和 CID 是子域。
通过自定义 URL 协议，如ipfs://<CID>/<path>、ipns://<peer ID>/<path>和dweb://<IPFS address>
HTTP 网关
提供网关只是为了方便：换句话说，它们帮助使用 HTTP 但不使用分布式协议（例如 IPFS）的工具进行通信。它们是网络升级路径的第一阶段。有关 IPFS 网关的更多信息。

集权
HTTP 网关自 2015 年以来一直运行良好，但它们具有一系列与 HTTP 的集中性和某些 HTTP 语义相关的重要限制。网关的基于位置的寻址取决于 DNS 和 HTTPS/TLS，这依赖于对证书颁发机构的信任 （打开新窗口）(CA) 和公钥基础设施 （打开新窗口）（公钥基础设施）。从长远来看，这些问题应该通过使用机会协议升级方案来缓解。

协议升级
工具和浏览器扩展应该检测 IPFS 内容路径并直接通过 IPFS 协议解析它们。当没有可用的本机实现时，他们应该仅将 HTTP 网关用作后备，以确保平稳、向后兼容的过渡。

提示

使用包含内容寻址路径的相对或绝对 URL。这将利用当今的内容寻址，同时确保与遗留网络的向后兼容性。

例如，网站可以从内容寻址路径加载静态资产：

<link rel="stylesheet" href="https://example.com/ipfs/QmNrgEMcUygbKzZeZgYFosdd27VE9KnWbyUD73bKZJ3bGi?filename=style.css">
<link rel="stylesheet" href="/ipfs/QmNrgEMcUygbKzZeZgYFosdd27VE9KnWbyUD73bKZJ3bGi?filename=style.css">
支持 IPFS 的用户代理，例如带有ipfs-companion的浏览器 （打开新窗口），可以识别/ipfs/<CID>内容路径并通过 IPFS 而不是 HTTP 加载相关资产。不支持 IPFS 的用户代理仍然从原始 HTTP 服务器获取正确的数据。

路径网关
在最基本的方案中，用于内容寻址的 URL 路径实际上是没有规范位置的资源名称。HTTP 服务器提供位置部分，这使得浏览器可以将 IPFS 内容路径解释为相对于当前服务器的路径，并且无需任何转换即可正常工作。给定网关主机地址（例如ipfs.io）和资源路径（/path/to/resource）、CID（<cid>）、IPNS ID（<ipnsid>）或DNSLink（<dnslink>）都可以使用。

使用 CID
使用 IPNS
使用 DNSLink
使用 CID
给定一个 CID <cid>，一个 URL 路径可以构造如下：

https://<gateway-host>.tld/ipfs/<cid>/path/to/resource
例子：

https://ipfs.io/ipfs/bafybeiemxf5abjwjbikoz4mc3a3dla6ual3jsgpdr4cjr3oz3evfyavhwq/wiki/Vincent_van_Gogh.html
https://ipfs.io/ipfs/QmT5NvUtoM5nWFfrQdVrFtvGfKFmG7AHE8P34isapyhCxX/wiki/Mars.html
使用 IPNS
给定一个 IPNS <ipnsid>，一个 URL 路径可以构造如下：

https://<gateway-host>.tld/ipns/<ipnsid>/path/to/resource
例子：

https://ipfs.io/ipns/k51qzi5uqu5dlvj2baxnqndepeb86cbk3ng7n3i46uzyxzyqj2xjonzllnv0v8
使用 DNSLink
给定 DNSLink <dnslink>，可以按如下方式构造 URL 路径：

https://<gateway-host>.tld/ipns/<dnslink>/path/to/resource
例子：

https://ipfs.io/ipns/tr.wikipedia-on-ipfs.org/wiki/Anasayfa.html
危险

在此方案中，所有页面共享一个来源 （打开新窗口），这意味着只有在站点隔离无关紧要时（没有 cookie 的静态内容、本地存储或需要用户许可的 Web API）才应使用这种类型的网关。

如有疑问，请使用subdomain gateway。

子域网关
当基于源的安全性 （打开新窗口）需要时，子域中应使用Base32或Base36等不区分大小写编码的CIDv1 ：

https://<cidv1b32>.ipfs.<gateway-host>.tld/path/to/resource
例子：

https://bafybeiemxf5abjwjbikoz4mc3a3dla6ual3jsgpdr4cjr3oz3evfyavhwq.ipfs.dweb.link/wiki/
https://bafybeiemxf5abjwjbikoz4mc3a3dla6ual3jsgpdr4cjr3oz3evfyavhwq.ipfs.cf-ipfs.com/wiki/Vincent_van_Gogh.html
https://bafybeiemxf5abjwjbikoz4mc3a3dla6ual3jsgpdr4cjr3oz3evfyavhwq.ipfs.localhost:8080/wiki/
Kubo 0.5+ 的原生支持
久保 （打开新窗口）为在中定义的主机名上的子域网关提供本机支持Gateway.PublicGateways （打开新窗口）配置图。

了解有关托管公共网关的守护程序配置的更多信息：

Gateway.PublicGateways文档 （打开新窗口）用于定义指定主机名上的网关行为
Gateway食谱 （打开新窗口）准备好为最常见的用例使用单行
已知的问题

一些浏览器和其他用户代理强制 URL 的授权部分小写，在 HTTP 网关有机会读取它们之前破坏区分大小写的 CID
DNS 标签长度限制为 63 个字符 ( RFC 1034 （打开新窗口）)
由于这些限制，建议在子域上下文中使用简短的、不区分大小写的 CIDv1。Base32 是安全的默认值；不太流行的 Base36 可用于更长的 ED25519 libp2p 密钥。

请参阅下一节以了解如何将现有 CIDv0 转换为 DNS 安全表示。

子域的 CID 转换
如果您有由旧 CIDv0 标识的内容，可以通过简单的方法将其安全地表示为 CIDv1，以便在子域和其他不区分大小写的上下文中使用。

自动——利用 Kubo 中的网关
TL;DR：使用子域网关作为路径的直接替代，消除了手动 CID 转换的需要。

发送到网关域的内容路径请求将返回 HTTP 301 重定向到正确的子域版本，并在需要时处理任何必要的编码转换：

https://<gateway-host>.tld/ipfs/<cid> -> https://<cidv1>.ipfs.<gateway-host>.tld/
为了说明，打开 CIDv0 资源https://dweb.link/ipfs/QmT5NvUtoM5nWFfrQdVrFtvGfKFmG7AHE8P34isapyhCxX/wiki/Mars.html （打开新窗口）返回到 CIDv1 表示的重定向https://bafybeicgmdpvw4duutrmdxl4a7gc52sxyuk7nz5gby77afwdteh3jc5bqa.ipfs.dweb.link/wiki/Mars.html （打开新窗口）.

网关负责将 CID 转换为不区分大小写的编码。CIDv1 中的 multihash 与原始 CIDv0 中的相同。

手动 — 使用 cid.ipfs.io 或命令行
也可以手动进行转换。

将 CID 转换为 Base32 ( RFC4648 （打开新窗口）, 没有填充）使用cid.ipfs.io （打开新窗口）或命令行：

$ ipfs cid base32 QmbWqxBEKC3P8tqsKc98xmWNzrzDtRLMiMPL8wBuTGsMnR
bafybeigdyrzt5sfp7udm7hu76uh7y26nf3efuylqabf3oclgtqy55fbzdi
PeerID 可以表示为具有多编解码器的 CIDlibp2p-key （打开新窗口）. 建议将 Base36 作为较长密钥的更安全默认值：

$ ipfs key list -l --ipns-base base36
k51qzi5uqu5dh9ihj4p2v5sl3hxvv27ryx2w0xrsv6jmmqi91t9xp8p9kaipc2 self

$ ipfs cid format -v 1 -b base36 --codec libp2p-key QmNnooDu7bfjPFoTZYxMNLWUQJyrVwtbZg5gBMjTezGAJN
k2k4r8jl0yz8qjgqbmc2cdu5hkqek5rj6flgnlkyywynci20j0iuyfuj
DNSLink网关
IPFS 守护程序提供的网关了解HostHTTP 请求中存在的标头，并将检查指定域名的DNSLink是否存在 （打开新窗口）. 如果存在 DNSLink，网关将从通过 DNS TXT 记录解析的路径返回内容。这种类型的网关提供完全的源隔离 （打开新窗口）.

示例： https: //docs.ipfs.tech （打开新窗口）（本网站）

提示

如需完整的 DNSLink 指南，包括教程、使用示例和常见问题解答，请查看dnslink.io （打开新窗口）.

本机网址
ipfs://{cid}/path/to/subresource/cat.jpg
本机地址格式与子域网关相同 （打开新窗口）HTTP URL，但带有：

ipfs由或ipns命名空间替换的协议方案
基于位置的权限组件（网关主机+端口）替换为唯一内容标识符 (CID) 形式的内容寻址组件
例如：

ipfs://{cidv1}
ipfs://{cidv1}/path/to/resource
ipfs://{cidv1}/path/to/resource?query=foo#fragment

ipns://{cidv1-libp2p-key}
ipns://{cidv1-libp2p-key}/path/to/resource
ipns://{dnslink-name}/path/to/resource?query=foo#fragment
提示

我们的主要目标是重用现有标准，最大限度地提高与现有用户代理（如浏览器和 CLI 工具）的互操作性。如果不清楚，HTTP URL 规则适用。

双斜杠后的第一个元素是表示内容根的不透明标识符。它被解释为用于源计算的权限组件，它在不同内容树的安全上下文之间提供必要的隔离。

例子：

ipfs://bafybeiemxf5abjwjbikoz4mc3a3dla6ual3jsgpdr4cjr3oz3evfyavhwq/wiki/Vincent_van_Gogh.html
避免在 ipfs:// 中区分大小写的 CID

一些用户代理会强制小写 URL 类地址的 CID 组件。为确保与现有库和软件的互操作性，请使用不区分大小写的 CID 编码。建议在 Base32 或 Base36 中使用 CIDv1。

将本机地址转换为规范的内容路径
每个“URL”地址都可以轻松转换回内容路径：

ipfs://{immutable-root}/path/to/resourceA→ /ipfs/{immutable-root}/path/to/resourceA
ipns://{mutable-root}/path/to/resourceB→/ipns/{mutable-root}/path/to/resourceB

更多资源
实施者技术规范
关于 IPFS 寻址的最佳和最新的真实来源可以在IPFS in-web-browsers repo中找到 （打开新窗口）.

地址方案讨论的背景
自@jbenet以来，围绕 IPFS 寻址的讨论一直在进行 （打开新窗口）发布IPFS 白皮书 （打开新窗口），并提出了许多其他方法。这个长期存在的设计讨论包括许多冗长的 GitHub 问题线程，但是可以在这个 PR中找到一个很好的总结 （打开新窗口）.

IPFS伴侣
IPFS伴侣 （打开新窗口）是一个浏览器扩展，可以简化对 IPFS 资源的访问。

它提供对本机 URL 的支持，并将自动将 IPFS 网关请求重定向到您的本地守护程序，这样您就不会依赖或信任远程网关。

共享 d-web 命名空间
这个概念尚未建立，但可能会在未来进行探索和试验。分布式网络社区正在探索共享命名空间的想法，dweb以消除寻址 IPFS 和其他内容寻址协议的复杂性。目前研究的方法包括：

dweb://协议处理程序 ( arewedistributedyet/issues/28 （打开新窗口）)
.dweb特殊用途顶级域名（arewedistributedyet/issues/34 （打开新窗口）)