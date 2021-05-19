# Git 用 shadowsocks 加速
利用shadowsocks的socks5代理，配置好后明显加速。用下面两条命令配置好后，保持shadowsocks客户端开启就行了。

	git config --global http.proxy 'socks5://127.0.0.1:1080' 
	git config --global https.proxy 'socks5://127.0.0.1:1080'

shadowsocks 的本地端口默认是 1080

	git config --global --unset http.proxy
	git config --global --unset https.proxy	
## git 开启 http/https 代理详解
	# 查看git 可以配置的内容
	git config
	# 部分配置内容详解
	global  即是读/写当前用户全局的配置文件(~/.gitconfig 文件，属于某个计算机用户)
	system  即是读写系统全局的配置文件(/etc/gitconfig 文件，属于计算机)
	local   即是当前 clone 仓库 的配置文件(位于 clone仓库下 .git/config)。
	blob    配置是另外一种形式，提供一个 blob 大对象格式，没有验证过，估计与 local 是一样的，只是形式不同。
	
	# 查看当前代理设置
	git config --global http.proxy
	git config --global https.proxy
	
	# 设置当前代理为 http://127.0.0.1:1080 或 socket5://127.0.0.1:1080
	git config --global http.proxy 'http://127.0.0.1:1080'
	git config --global https.proxy 'https://127.0.0.1:1080'
	
	git config --global http.proxy 'socks5://127.0.0.1:1080'
	git config --global https.proxy 'socks5://127.0.0.1:1080'
	
	# 删除 proxy
	git config --global --unset http.proxy
	git config --global --unset https.proxy
## 设置 Git SSH 代理(暂不可用)
还有一种情况，我们通过 SSH 方法 clone 代码，提交代码，因为这样不用输入密码，通常我们会在自己的常用电脑上这么做。上面设置的 HTTP 代理对这种方式 clone 代码是没有影响的，也就是并不会加速，SSH 的代理需要单独设置，其实这个跟 Git 的关系已经不是很大，我们需要改的，是SSH 的配置。在用户目录下建立如下文件 ~/.ssh/config，对 GitHub 的域名做单独的处理
	
	# 这里必须是 github.com，因为这个跟我们 clone 代码时的链接有关
	Host github.com
	   # 如果用默认端口，这里是 github.com，如果想用443端口，这里就是 ssh.github.com 详见 https://help.github.com/articles/using-ssh-over-the-https-port/
	   HostName github.com
	   User git
	   # 如果是 HTTP 代理，把下面这行取消注释，并把 proxyport 改成自己的 http 代理的端口
	   # ProxyCommand socat - PROXY:127.0.0.1:%h:%p,proxyport=6667
	   # 如果是 socks5 代理，则把下面这行取消注释，并把 6666 改成自己 socks5 代理的端口
	   # ProxyCommand nc -v -x 127.0.0.1:6666 %h %p
## git 开启 ssh 代理详解
git 使用 ssh 代理要借助第三方工具,比如:nc,通过修改ssh配置文件同时将整个系统的ssh连接走代理

nc 这个工具一般的Ubuntu系统都带的有.没有的话 `apt insttall nc` Mac使用 `brew install nc`

- 添加代理

		# ServerAliveInterval定时向服务器发送消息以保持连接,防止不操作而断开.需要的时候才添加
		echo "ProxyCommand nc -X 5 -x 127.0.0.1:1080 %h %p" >> ~/.ssh/config
		echo "ServerAliveInterval 30" >> ~/.ssh/config
- 取消代理

		# Linux直接修改源文件
		sed -i '/ProxyCommand/d' ~/.ssh/config
		
		# Mac下只能这么弄,否则提示参数错误
		sed '/ProxyCommand/d' ~/.ssh/config  > ~/.ssh/config.bk && mv ~/.ssh/config.bk   ~/.ssh/config
		# 或者
		sed '/ProxyCommand/d;/ServerAliveInterval/d' ~/.ssh/config  > ~/.ssh/config.bk && mv ~/.ssh/config.bk   ~/.ssh/config

## 参考
- [Git用shadowsocks加速](https://my.oschina.net/chinaliuhan/blog/3065281)
- [Failed to connect to github.com port 443: Operation timed out](https://www.jianshu.com/p/471aeba64724)				