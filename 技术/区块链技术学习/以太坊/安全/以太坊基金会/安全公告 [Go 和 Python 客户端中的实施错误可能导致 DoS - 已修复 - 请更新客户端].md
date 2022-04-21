# 安全公告 [Go 和 Python 客户端中的实施错误可能导致 DoS - 已修复 - 请更新客户端]
当处理具有特定交易组合的（有效）块时，geth 客户端中的状态转换和共识问题会导致恐慌（崩溃），如果块被未受影响的客户端接受并中继从而导致 DoS，则可能导致整体网络不稳定。这可能发生在包含自杀到块奖励地址的交易的块中。

- 受影响的配置

	Geth 报告的问题。在调查该问题时，在 pyethereum 中发现并纠正了相关问题，因此 pyethapp 也受到影响。C++ 客户端不受影响。
- 可能性：低
- 严重性：高
- 复杂性：高
- 影响：网络不稳定和 DoS
- 详细信息

	包含一个或多个 SUICIDE 调用的特定交易组合的块虽然有效，但会导致 go-ethereum 客户端的恐慌崩溃和 pyethereum 的崩溃。其他详细信息可能会在可用时发布。
- 对预期链重组深度的影响： 无。
- 以太坊采取的补救措施：提供如下修复。
- 建议的临时解决方法：切换到不受影响的客户端，例如 eth (C++)。
- 修复：升级 geth 和 pyethereum 客户端软件。
- goeth（geth）：

	请注意，geth 的当前稳定版本现在是 1.1.1；如果您正在运行 1.0 并使用包管理器（例如 apt-get 或 homebrew），则客户端将被升级。

	如果使用 PPA：`sudo apt-get update` 然后 `sudo apt-get upgrade`

	如果使用 `brew: brew update` 然后 `brew reinstall ethereum`

	如果使用 Windows 二进制文件：下载更新的二进制文件。

	如果您从源代码构建：git pull后跟make geth （请使用主分支提交8f09242d7f527972acb1a8b2a61c9f55000e955d)

	此更新在 Ubuntu 和 OSX 上的正确版本是 Geth/ v1.1.1-8f09242d

	- pyethereum：
	
			> pip install pyethapp --force-reinstall