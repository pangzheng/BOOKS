# 安全建议 [eth (cpp-ethereum) 如果在启用 UPnP 的情况下运行可能容易受到攻击]
- 受影响的配置： eth (cpp-ethereum) 报告的问题
- 可能性：中等
- 严重性：高
- 影响：可能在运行 eth (cpp-ethereum) 的机器上实现远程代码执行
- 详细信息：

	在 MiniUPnP 库中发现的漏洞可能会影响启用 UPnP 运行的 eth 客户端。
- 对预期链重组深度的影响：无
- 以太坊采取的补救措施：我们正在验证这是否确实会影响 cpp-ethereum 并将发布很快就会更新。
- 建议的临时解决方法：仅在禁用 UPNP 的情况下运行 eth (cpp-ethereum) 通过传递–upnp off到 eth.
- 建议：如果运行 eth 客户端 (cpp-ethereum)，则禁用 UPnP	