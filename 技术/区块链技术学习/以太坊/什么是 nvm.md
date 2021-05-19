# 什么是 nvm
[nvm](https://github.com/creationix/nvm/blob/master/README.md) 全称 Node Version Manager 通过 shell 脚本实现, nodejs 的版本管理工具。：一个 nvm 可以管理很多 node 版本和 npm 版本
## 安装
	curl -o- https://raw.githubusercontent.com/creationix/nvm/v0.33.8/install.sh | bash
or
	
	wget -qO- https://raw.githubusercontent.com/creationix/nvm/v0.33.8/install.sh | bash

## 加载环境变量
- 加载到 `.bash_profile` or `.zshrc`

		export NVM_DIR="$HOME/.nvm"
		[ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh"  # This loads nvm
		[ -s "$NVM_DIR/bash_completion" ] && \. "$NVM_DIR/bash_completion"  # This loads nvm bash_completion
- 加载

		source ~/.bashrc		
				
## nvm 常用命令
- `nvm install stable` ## 安装最新稳定版 node，当前是node v9.5.0 (npm v5.6.0)
- `install <version>` ## 安装指定版本，可模糊安装，如：安装v4.4.0，既可nvm install v4.4.0，又可nvm install 4.4
- `nvm uninstall <version>` ## 删除已安装的指定版本，语法与install类似
- `nvm use <version>` ## 切换使用指定的版本node
- `nvm ls` ## 列出所有安装的版本
- `nvm ls-remote` ## 列出所有远程服务器的版本（官方node version list）
- `nvm current` ## 显示当前的版本
- `nvm alias <name> <version>` ## 给不同的版本号添加别名
- `nvm unalias <name>` ## 删除已定义的别名
- `nvm reinstall-packages <version>` ## 在当前版本 node 环境下，重新全局安装指定版本号的 npm 包

### 参考
[Mac OS 下 NVM 的安装与使用](https://www.jianshu.com/p/622ad36ee020)