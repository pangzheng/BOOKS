# 安全警报 - [以前的安全补丁可能导致具有特定事务序列的 Go 客户端上的无效状态根 - 已修复。请更新。]
- 总结：go 客户端的实现 bug 可能导致状态无效
- 受影响的客户端版本： Go 客户端的最新（未打补丁）版本；v1.1.2、v1.0.4 标记和开发，9 月 9 日之前的 master 分支。
- 可能性：低
- 严重性：高
- 影响：高
- 详细信息

	如果-在同一个区块内-合约被取消，则当交易耗尽气体时，Go ethereum 客户端无法正确恢复执行环境的状态。这会导致状态对象的复制操作无效；将合约标记为未删除。此操作将导致其他实现之间的共识问题。
- 对预期链重组深度的影响：无
- 以太坊采取的补救措施：提供如下修补程序。
	- 建议的临时解决方法：使用 Python 或 C++ 客户端
	- 如果使用 PPA：`sudo apt-get update` 那么 `sudo apt-get upgrade`
	- 如果使用 `brew:brew update` 那么 `brew reinstall ethereum`
	- 如果使用 Windows 二进制文件：从下载更新的二进制文件 `https://github.com/ethereum/go-ethereum/releases/tag/v1.1.3`
	- 主分支提交：`https://github.com/ethereum/go-ethereum/commit/9ebe787d3afe35902a639bf7c1fd68d1e591622a` 如果您从源代码构建：` git fetch origin && git checkout origin/master `后跟 `make geth`