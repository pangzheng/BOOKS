# Mac安装brew并更改源
brew 是 Mac 下的一个包管理工具，类似于 centos 下的 yum，可以很方便地进行安装/卸载/更新各种软件包
## 安装 brew

	/usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
## 使用
### brew
- 安装软件

		brew install nodejs
- 更新

		brew upgrade nodejs
- 卸载

		brew remove nodejs
- 列表

		brew list                   # 列出当前安装的软件
- 查询		

		brew search nodejs          # 查询与 nodejs 相关的可用软件
- 安装信息

		brew info nodejs            # 查询 nodejs 的安装信息
		
### brew service
安装并启动一个 es 服务器
	
	brew install elasticsearch          # 安装 elasticsearch
	brew services start elasticsearch   # 启动 elasticsearch
	brew services stop elasticsearch    # 停止 elasticsearch
	brew services restart elasticsearch # 重启 elasticsearch
	brew services list                  # 列出当前的状态

## 更换源
- 修改中科大源

		# cd "$(brew --repo)"
		# git remote set-url origin https://mirrors.ustc.edu.cn/brew.git
		# cd "$(brew --repo)/Library/Taps/homebrew/homebrew-core"
		# git remote set-url origin https://mirrors.ustc.edu.cn/homebrew-core.git
		# brew update
		# brew doctor
		# brew config


			
## 参考
- [官网](https://brew.sh/)
- [Mac HomeBrew国内镜像安装方法](https://juejin.im/post/5c738bacf265da2deb6aaf97)