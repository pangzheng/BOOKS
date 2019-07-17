# dnsmasq编译
## 说明
[参考](https://github.com/Dataman-Cloud/operation/blob/master/%E8%BF%90%E7%BB%B4%E6%96%87%E6%A1%A3/1-%E6%8A%80%E6%9C%AF%E6%96%87%E6%A1%A3/dnsmasq/dnsmasq-dns.md)
## 为什么要自定义版本
默认使用系统版本是 centos7.2 的，虽然自带了 dnsmasq，但是版本比较老，没有需求的参数，所以需要编译新版本来支持，新版本主要使用参数为 `rev-server`
## 编译
- 下载新包
	- 进入[官方网站](http://www.thekelleys.org.uk/dnsmasq/)
	- 下载版本的是 `Dnsmasq version 2.76`
	- 找到对应的 tag 包
	- wget xxx.tar.gz
- 解压
	- tar xzvf xxx.tar.gz
	- cd xxx
- 编译
	- make V=s

## 安装方法(仅限于 centos 系统)
因为想使用系统服务，所以可以将现有系统默认自带的 dnsmasq 替换。

- 备份现有二进制模块
	
		cd /usr/sbin/dnsmasq && mv /usr/sbin/dnsmasq /usr/sbin/dnsmasq.bak
- 拷贝新版本程序到这个位置

		cp dnsmasq /usr/sbin/dnsmasq
- 重启服务

		service dnsmasq restart
						  