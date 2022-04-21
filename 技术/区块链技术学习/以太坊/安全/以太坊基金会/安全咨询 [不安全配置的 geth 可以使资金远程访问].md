# 安全咨询 [不安全配置的 geth 可以使资金远程访问]
	没有防火墙和未锁定帐户的不安全配置的以太坊客户端可能导致攻击者远程访问资金。
- 受影响的配置

	Geth 报告的问题，尽管所有实现包括。如果使用不安全，C++ 和 Python 原则上会显示此行为；仅适用于将 JSON-RPC 端口对攻击者开放的节点（这排除了 NAT 后面内部网络上的大多数节点），将接口绑定到公共 IP，同时在启动时让帐户解锁。
- 可能性:低
- 严重性:高
- 影响

	与客户导入或生成的钱包相关的资金损失
- 细节

	我们注意到有些人已经绕过了放在 JSON-RPC 接口上的内置安全性。RPC 接口允许您从在发送交易之前已解锁的任何帐户发送交易，并将在整个会话期间保持解锁状态。

	默认情况下，RPC 被禁用，启用它只能从运行以太坊客户端的同一主机访问。通过打开 RPC 以供 Internet 上的任何人访问并且不包括防火墙规则，您打开了您的钱包以防任何知道您的地址和您的 IP 的人盗窃。
- 对预期链重组深度的影响：无
- 以太坊采取的补救措施

	通过要求任何潜在的远程交易的明确用户授权，eth RC1 将是完全安全的。Geth 的更高版本可能支持此功能。
- 建议的临时解决方法

	仅为每个客户端运行默认设置，并且在您进行更改时评估这些更改如何影响您的安全性。
- 注意：这不是错误，而是对 JSON-RPC 的误用。
- 建议

	如果没有防火墙策略阻止 JSON-RPC 端口（默认值：8545），切勿在可访问 Internet 的机器上启用 JSON-RPC 接口。

	- eth：使用 RC1 或更高版本。
	- geth：使用安全的默认值，并了解选项的安全含义。
		- `- rpcaddr“127.0.0.1”`

			这是仅允许源自本地计算机的连接的默认值；远程 RPC 连接被禁用, [localhost 设置方法](https://ethereum.stackexchange.com/questions/3305/what-is-http-localhost8545/3306#3306)
		- `–unlock.`

			此参数用于在启动时解锁帐户以帮助自动化。默认情况下，所有帐户都被锁定
		
## 参考
- [Security Advisory [Insecurely configured geth can make funds remotely accessible]](https://blog.ethereum.org/2015/08/29/security-alert-insecurely-configured-geth-can-make-funds-remotely-accessible/)
- [What is http://localhost:8545?](https://ethereum.stackexchange.com/questions/3305/what-is-http-localhost8545/3306#3306)