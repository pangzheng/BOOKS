# 什么是 yarn
## 问题来源
部署以太坊 opensea 的测试项目

- 原来以太坊操作

		git clone xxx
		npm install
		npm run dev
- opensea 操作

		git clone xxx
		yarn
		yarn start

## Yarn 是什么
“Yarn 是由 Facebook、Google、Exponent 和 Tilde 联合推出了一个新的 JS 包管理工具 ，正如官方文档中写的，Yarn 是为了弥补 npm 的一些缺陷而出现的。”
### npm install 问题
- `npm install` 的时候巨慢。特别是新的项目拉下来要等半天，删除 `node_modules`，重新 install 的时候依旧如此。
- 同一个项目，安装的时候`无法保持一致性`。由于 `package.json` 文件中版本号的特点，下面三个版本号在安装的时候代表不同的含义

		"5.0.3", #“5.0.3”表示安装指定的5.0.3版本
		"~5.0.3", #“～5.0.3”表示安装5.0.X中最新的版本
		"^5.0.3"  #“^5.0.3”表示安装5.X.X中最新的版本

	这就麻烦了，常常会出现同一个项目，有的同事是 OK 的，有的同事会由于安装的版本不一致出现 bug。
- 安装的时候，包会在同一时间下载和安装，中途某个时候，一个包抛出了一个错误，但是 npm 会继续下载和安装包。因为 npm 会把所有的日志输出到终端，有关错误包的错误信息就会在一大堆 npm 打印的警告中丢失掉，并且你甚至永远不会注意到实际发生的错误。

### Yarn的优点
- 速度快 

	速度快主要来自以下两个方面：
	
	- 并行安装：无论 npm 还是 Yarn 在执行包的安装时，都会执行一系列任务。
		- npm 是按照队列执行每个 package，也就是说必须要等到当前 package 安装完成之后，才能继续后面的安装。
		- 而 Yarn 是同步执行所有任务，提高了性能。
	- 离线模式：如果之前已经安装过一个软件包，用 Yarn 再次安装时之间从缓存中获取，就不用像 npm 那样再从网络下载了。
- 安装版本统一

	为了防止拉取到不同的版本，Yarn 有一个锁定文件 (lock file) 记录了被确切安装上的模块的版本号。每次只要新增了一个模块，Yarn 就会创建（或更新）yarn.lock 这个文件。这么做就保证了，每一次拉取同一个项目依赖时，使用的都是一样的模块版本。
	
	npm 其实也有办法实现处处使用相同版本的 `packages`，但需要开发者执行 `npm shrinkwrap` 命令。这个命令将会生成一个锁定文件，在执行 `npm install` 的时候，该锁定文件会先被读取，和 Yarn 读取 `yarn.lock` 文件一个道理。
	
	npm 和 Yarn 两者的不同之处在于，Yarn 默认会生成这样的锁定文件，而 npm 要通过 `shrinkwrap` 命令生成 `npm-shrinkwrap.json` 文件，只有当这个文件存在的时候，packages 版本信息才会被记录和更新。

- 更简洁的输出

	npm 的输出信息比较冗长。在执行 `npm install` 的时候，命令行里会不断地打印出所有被安装上的依赖。
	
	相比之下，Yarn 简洁太多：默认情况下，结合了 emoji 直观且直接地打印出必要的信息，也提供了一些命令供开发者查询额外的安装信息。
- 多注册来源处理

	所有的依赖包，不管他被不同的库间接关联引用多少次，安装这个包时，只会从一个注册来源去装，要么是 npm 要么是 bower, 防止出现混乱不一致。
- 更好的语义化

	 yarn 改变了一些 npm 命令的名称，比如 `yarn add/remove`，感觉上比 npm 原本的 `install/uninstall` 要更清晰。

## Yarn和npm命令对比
	npm install === yarn 
	npm install taco --save === yarn add taco
	npm uninstall taco --save === yarn remove taco
	npm install taco --save-dev === yarn add taco --dev
	npm update --save === yarn upgrade	

## 安装 yarn
### Homebrew(macOS)
通过 Homebrew 安装，如果还未安装 Node.js，Homebrew 会自动为你安装。

	brew install yarn
如果你用的是 `nvm` 或类似工具，你需要确保 `PATH` 变量中列出的 `nvm` 版本的路径在 `Homebrew` 安装的 `Node.js` 路径之前。

### MacPorts
你可以通过 [MacPorts](https://www.macports.org/) 安装 Yarn 。 如果你还未安装 Node.MacPorts 会自动为你安装。

	sudo port install yarn	
### 脚本安装
在 macOS 和通用 Unix 环境上安装 Yarn 的最简单方法之一是通过 Shell 脚本。您可以通过在终端中运行以下代码来安装Yarn：

	curl -o- -L https://yarnpkg.com/install.sh | bash
安装过程包括验证GPG签名,代码在[这里](https://github.com/yarnpkg/website/blob/master/install.sh),还可以通过在终端中运行以下代码来指定版本

	curl -o- -L https://yarnpkg.com/install.sh | bash -s -- --version [version]
[查看发行版本列表](https://github.com/yarnpkg/yarn/releases)
### 源码 tar 包安装
[下载tar包](https://yarn.bootcss.com/latest.tar.gz)	,解压安装

	cd /opt
	wget https://yarnpkg.com/latest.tar.gz
	tar zvxf latest.tar.gz
	# Yarn is now in /opt/yarn-[version]/
建议使用 GPG 验证 tarball

	wget -qO- https://dl.yarnpkg.com/debian/pubkey.gpg | gpg --import
	wget https://yarnpkg.com/latest.tar.gz.asc
	gpg --verify latest.tar.gz.asc
	# Look for "Good signature from 'Yarn Packaging'" in the output
### 路径设置
如果未在 `PATH` 环境变量中找到 yarn，请按照以下步骤添加 yarn 到 `PATH` 环境变量中，使其可以随处运行。

注意：您的配置文件可能是 `.profile、.bash_profile、.bashrc、.zshrc` 等。

- 将此项加入您的配置文件： 

		export PATH="$PATH:/opt/yarn-[version]/bin" （路径可能根据您安装 Yarn 的位置而有差异）
- 在终端中，执行登录并登出以使更改生效

		source xxx

为了可以全局访问 Yarn 的可执行文件，你需要在控制台（或命令行）中设置 PATH 环境变量。若要执行此操作，请添加 

	export PATH="$PATH:`yarn global bin`
到你的配置文件中，或者，如果你使用的是 Fish shell，直接执行此命令 

	set -U fish_user_paths (yarn global bin) $fish_user_paths 
即可	

## 升级 Yarn
有新版时，Yarn 会给你提示。 如需升级 Yarn ，仍然可以通过 Homebrew 来升级。

	brew upgrade yarn
通过如下命令测试 Yarn 是否安装成功

	yarn --version
	
## 参考
- https://yarn.bootcss.com/docs/install/#mac-stable
- https://zhuanlan.zhihu.com/p/27449990		