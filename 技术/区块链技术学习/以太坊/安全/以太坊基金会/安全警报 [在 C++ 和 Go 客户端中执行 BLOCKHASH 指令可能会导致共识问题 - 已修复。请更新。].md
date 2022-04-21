# 安全警报 [在 C++ 和 Go 客户端中执行 BLOCKHASH 指令可能会导致共识问题 - 已修复。请更新。]
- 总结： BLOCKHASH 的错误实现会触发链重组，导致共识问题
- 受影响的配置：直到 1.1.3 和 1.2.2 的所有 geth 版本。1.0.0 之前的所有 eth 版本。
- 可能性：低
- 严重性：中
- 影响：中等
- 详细信息

	C++ (eth) 和 Go (geth) 客户端在以太坊虚拟机中都有错误实现边缘情况，特别是 BLOCKHASH 指令用于检索块哈希的链。这种边缘情况不太可能在实时网络上发生，因为它只会在某些类型的链重组中触发（在一个合约执行 BLOCKHASH(N-1) ，其中 N 是尚未完成的非规范子链的头部重组为规范（最佳/最长）链，但将在处理块之后）。

	pyethereum 不受影响。

- 对预期链重组深度的影响：无
- 以太坊采取的补救措施：提供如下修补程序。
- Geth：
	- PPA：`sudo apt-get update` 然后 `sudo apt-get upgrade`
	- Brew : `brew update` 然后 `brew reinstall ethereum`
	- Windows ：从 `https://github.com/ethereum/go-ethereum/releases/tag/v1.2.3` 下载更新的二进制文件
	- 从源代码构建：

			git fetch origin && git checkout origin/master
			make geth

		主分支提交：e55594ca0e131d518944e98701fc067735b11152 </div>
- eth：
	- PPA：https ://gavofyork.gitbooks.io/turboethereum/content/chapter1.html
	- win和 Mac OS X：  https ://build.ethdev.com/cpp-binaries-data/
	- 来源： https ://github.com/ethereum/webthree-umbrella/tree/release
	- 从源代码构建： https ://github.com/ethereum/webthree-umbrella/wiki