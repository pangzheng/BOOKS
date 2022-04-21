# 安全警报 – Geth 遭受了极低可能性的 DoS 攻击向量 – 立即更新
- 受影响的配置： 所有 Go 客户端版本 
- 可能性： 非常低
- 严重性： 高
- 详细信息

	Geth（以及可能的其他客户端）中的错误可能会遭受 DoS 攻击，并允许远程攻击者通过提供有效的、更轻的链来几乎无限期地停止同步过程。稍后将提供更多信息，包括通过错误赏金计划提交的报告。
- 对预期链重组深度的影响： 无
- 建议的临时解决方法： 无
- 以太坊采取的补救措施：提供以下修补程序：
	- 如果您使用 Mist：从发布页面下载[更新的二进制文件](https://github.com/ethereum/mist/releases/tag/0.7.4)
	- 如果使用 PPA： `sudo apt-get update` 那么 `sudo apt-get upgrade`
	- 如果使用 `brew: brew update` 那么 `brew reinstall ethereum`
	- 如果使用 Windows 二进制文件：从 发布页面下载更新的二进制文件
	- 如果您从源代码构建： ` git pull` 后跟 `make geth`  （请使用 Master 分支94ad694a26ca3f7776ec8240802596755e5d5c0a）