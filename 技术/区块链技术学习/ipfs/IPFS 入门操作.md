
# 如何在IPFS新增一个文件
## 测试安装 ipfs 是否正常
	$ ipfs help && ipfs version
## 启动 ipfs 后台
	$ ipfs daemon
	Initializing daemon...
	go-ipfs version: 0.4.20-
	Repo version: 7
	System version: amd64/darwin
	Golang version: go1.12.4
	Swarm listening on /ip4/10.213.2.56/tcp/4001
	Swarm listening on /ip4/127.0.0.1/tcp/4001
	Swarm listening on /ip6/::1/tcp/4001
	Swarm listening on /p2p-circuit
	Swarm announcing /ip4/10.213.2.56/tcp/4001
	Swarm announcing /ip4/127.0.0.1/tcp/4001
	Swarm announcing /ip6/::1/tcp/4001
	API server listening on /ip4/127.0.0.1/tcp/5001
	WebUI: http://127.0.0.1:5001/webui
	Gateway (readonly) server listening on /ip4/127.0.0.1/tcp/8080
	Daemon is ready
## 创建文件	
### 新建 file.txt 文件
- 打开终端
- 新建一个文件夹 test
- 切换到 test 中
- 新建一个文件 file.txt
- 文件里面随便输入 ipfs 保存并且退出。

### 添加 file.txt 进入 ipfs 本地节点
- 进入 test 文件夹

		$ cd test/
- 查看 file.txt

		$ cat file.txt
		ipfs is good
- 使用 `ipfs add` 添加

		$ ipfs add file.txt
		added QmdG2ZZQV5Ly1jneToDQEW4mPCSbgdxNmvMZT9cB4D5LVB file.txt
		 13 B / 13 B [========================================================================================] 100.00%
- 查看

		$ ipfs cat QmdG2ZZQV5Ly1jneToDQEW4mPCSbgdxNmvMZT9cB4D5LVB
		ipfs is good	 


注意这里 ipfs 返回的是文件的 hash 值 `QmdG2ZZQV5Ly1jneToDQEW4mPCSbgdxNmvMZT9cB4D5LVB`

- 查看 ipfs 目录,是空，因为还没有映射

		ipfs files ls /
		

## 创建目录
- 创建目录

		$ ipfs files mkdir /pro
- 将文件 cp 到 ipfs 目录中

		$ ipfs files cp /ipfs/QmdG2ZZQV5Ly1jneToDQEW4mPCSbgdxNmvMZT9cB4D5LVB /pro/file.txt
- 查看目录

		$ ipfs files ls /
		pro	
		$ ipfs files ls /pro
		file.txt
- 读取文件

		$ ipfs files read  /pro/file.txt
		ipfs is good

## 上传整个目录
- 创建一个目录，包含文件如下

		$ ll website/
		total 16
		-rw-r--r--  1 Prometheus  staff   386B  6 26 16:52 index.html
		-rw-r--r--  1 Prometheus  staff    21B  6 26 14:30 style.css

		$ cat website/index.html
		<!DOCTYPE html>
		<html lang="en">
		<head>
		  <meta charset="UTF-8">
		  <title>Hello Tarkov</title>
		  <link rel="stylesheet" href="./style.css" />
		</head>
		<body>
		  <h1>Welcome to IPFS Escape from Tarkov</h1>
		  <video width="640" height="480" controls="controls">
		  <source src="http://ipfs.io/ipfs/QmUT97Bnskv9bZX27xtLLheD5ryjEYk54PvhL3wcnY4hfQ" type="video/mp4" />
		</video>
		</body>
		</html>
		
		$ cat website/style.css
		h1 {
		  color: red;
		}
- 使用 `ipfs add -r` 上传整个目录

		$ ipfs add -r website
		added QmYWs51HwfZSW1ncCEKg2yasHWqVgdU76D9d3o2ybbC33x website/index.html
		added QmfUcVBJseiTvr2BHeCRE4aTmwYt1tBBRxsvrs22uipUXh website/style.css
		added QmYUhnp3XPWAwZ123HqtqsAgHxkxFSbyQ3PSrZJmn73KjA website
		 407 B / 407 B [======================================================================================] 100.00
- 通过路径访问 `index.html`
	- 直接访问文件 id

			$ ipfs cat QmYWs51HwfZSW1ncCEKg2yasHWqVgdU76D9d3o2ybbC33x
			<!DOCTYPE html>
			<html lang="en">
			<head>
			  <meta charset="UTF-8">
			  <title>Hello Tarkov</title>
			  <link rel="stylesheet" href=".
			  ...	
	- 访问 ipfs 路径+ id

			$ ipfs cat /ipfs/QmYWs51HwfZSW1ncCEKg2yasHWqVgdU76D9d3o2ybbC33x
			<!DOCTYPE html>
			<html lang="en">
			<head>
			  <meta charset="UTF-8">
			  <title>Hello Tarkov</title>
			  <link rel="stylesheet" href=".
			  ...	
	- 访问 ipfs 目录加 id

			$ ipfs cat /ipfs/QmYUhnp3XPWAwZ123HqtqsAgHxkxFSbyQ3PSrZJmn73KjA/index.html
			<!DOCTYPE html>
			<html lang="en">
			<head>
			  <meta charset="UTF-8">
			  <title>Hello Tarkov</title>
			  <link rel="stylesheet" href=".
			  ...				
- 通过游览器查看 ipfs 数据
	- 打开游览器
		- 输入内网地址
		
				http://127.0.0.1:8080/
		- 输入外网地址(注意新上传的外网地址能需要一段时间)
		
				http://ipfs.io/
	- 访问目录地址(注意这里是红色的字，载入了 css)
			
			http://127.0.0.1:8080/ipfs/QmYWs51HwfZSW1ncCEKg2yasHWqVgdU76D9d3o2ybbC33x
	- 访问目录文件(注意这里是红色的字，载入了 css)
	
			http://127.0.0.1:8080/ipfs/QmYUhnp3XPWAwZ123HqtqsAgHxkxFSbyQ3PSrZJmn73KjA/index.html
	- 直接访问文件(注意这里是黑色的字，没有载入 css)

			http://127.0.0.1:8080/ipfs/QmYWs51HwfZSW1ncCEKg2yasHWqVgdU76D9d3o2ybbC33x		

## 网站绑定 IPNS
- 绑定域名

	格式是 `ipfs name publish your_hash`，这里的 your_hash 是刚才生成的网站文件目录 hash 地址

		$ ipfs name publish /ipfs/QmU6bZBQpm5VWYBChjD9RA2wsc8DKmcsrevMC3B7xD6bvh
		Published to QmUqaJdyzAMyJASdiyc5diujCGAusLWN9iKnPsQkXYD4Pd: /ipfs/QmU6bZBQpm5VWYBChjD9RA2wsc8DKmcsrevMC3B7xD6bvh
		
- 查看绑定结果(注意这里的字是红色的)

		http://127.0.0.1:8080/ipns/QmUqaJdyzAMyJASdiyc5diujCGAusLWN9iKnPsQkXYD4Pd/
- 修改网站内容并上传
	- 修改内容 

			$ cat website/style.css
			h1 {
			  color: gold;
			}
	- 上传
	
			$ ipfs add -r website
			added QmYWs51HwfZSW1ncCEKg2yasHWqVgdU76D9d3o2ybbC33x website/index.html
			added Qmbe1NEALn27vD3U9NhzNJxmh6J4h8GLf8DrZZ5XKv5Pd9 website/style.css
			added QmeXz8wM42NMc27REiG7uxE1gmBkEVeWhRtD3Bkr5XPLWa website
			 408 B / 408 B [======================================================================================] 100.00%
	- 重新更新

			$ipfs name publish /ipfs/QmeXz8wM42NMc27REiG7uxE1gmBkEVeWhRtD3Bkr5XPLWa
			Published to 		QmUqaJdyzAMyJASdiyc5diujCGAusLWN9iKnPsQkXYD4Pd: /ipfs/QmeXz8wM42NMc27REiG7uxE1gmBkEVeWhRtD3Bkr5XPLWa
	- 查看更新效果

			$ ipfs name resolve QmUqaJdyzAMyJASdiyc5diujCGAusLWN9iKnPsQkXYD4Pd
			/ipfs/QmeXz8wM42NMc27REiG7uxE1gmBkEVeWhRtD3Bkr5XPLWa		
	- 查看 ipns(变成金色,有时候不好使)

			http://127.0.0.1:8080/ipns/QmUqaJdyzAMyJASdiyc5diujCGAusLWN9iKnPsQkXYD4Pd/

## IPNS
哈希跟 ip 地址一样难以记忆和传播，ipfs 提供 ipns 来解决这个问题，ipns 允许为哈希地址绑定域名，只需要在域名解析里面添加一条TXT记录即可

	dnslink=/ipfs/<your_hash>

例如小编的哈希地址是 
	
	QmYaGz9ChV3PcRuz3Zmr8XP34gxAe2gunZdtM7sKhDMqUS

TXT 解析的值设置在需要解析的域名上
	
	dnslink=/ipfs/QmYaGz9ChV3PcRuz3Zmr8XP34gxAe2gunZdtM7sKhDMqUS

一旦域名解析生效，那么可以通过以下地址来访问网站了

	http://ipfs.io/ipns/your.domain 
## DNS 地址链接
- 有自己服务器

	可以将域名A记录指向到 ipfs 服务器
- 没有自己服务器
	
	可以借助官方 IPFS 提供的网关地址 ` http://gateway.ipfs.io`，在域名解析里面建立一条 CNAME 记录，将解析指向 `http://gateway.ipfs.io`，同时建立一条 TXT 记录指向 `_dnslink.your.domain` 指向 `dnslink=/ipns/<你的节点ID>`   

如 `ipfsearch.xyz` 查询地址为

	$ dig +noall +answer TXT ipfsearch.xyz
	ipfsearch.xyz.		48	IN	TXT	"v=spf1 include:spf.efwd.registrar-servers.com ~all"
	ipfsearch.xyz.		48	IN	TXT	"dnslink=/ipns/QmSE8g9k5JS1vJ7y5znhSZikmybvdsm3yDj7sbKjPRqsJW"

还可以使用名为 _dnslink 的特殊子域发布 DNS 链接记录。当希望提高自动设置的安全性或将对 DNS 链接记录的控制权委派给第三方而不想放弃对原始 DNS 区域的完全控制时，这非常有用。

例如，docs.ipfs.io 没有 TXT 记录，但页面仍然加载，因为存在 TXT 记录 _dnslink.docs.ipfs.io：

	$ dig +noall +answer TXT _dnslink.docs.ipfs.io
	_dnslink.docs.ipfs.io.  34  IN  TXT "dnslink=/ipfs/QmeveuwF5wWBSgUXLG6p1oxF3GKkgjEnhA6AAwHUoVsx6E"
## 删除文件和删除块
### 删除文件
- 查看存储空间大小

		$ ipfs stats repo
		NumObjects: 9405
		RepoSize:   2142639862
		StorageMax: 10000000000
		RepoPath:   /Users/Prometheus/.ipfs
		Version:    fs-repo@7
- 删除文件

		ipfs files rm /太空旅客
- 再查看				

		$ ipfs stats repo
		NumObjects: 9405
		RepoSize:   2142639862
		StorageMax: 10000000000
		RepoPath:   /Users/Prometheus/.ipfs
		Version:    fs-repo@7			
- 文件存储依然存在

		ipfs ls QmdxpUVnvFnert9nmEkzwwz2tWdavU3fUQzrgBsTZP5yyG

### 删除存储块
- 删除存储块

		$ ipfs block rm QmdxpUVnvFnert9nmEkzwwz2tWdavU3fUQzrgBsTZP5yyG

## 测试资源
- 神秘巨星：QmWBbKvLhVnkryKG6F5YdkcnoVahwD7Qi3CeJeZgM6Tq68
- 芳华：QmYVri7jyBdPyfR8AgBLTgyTjiJifCgpeHFiFrKxowQeq8
- 大佛普拉斯：QmdpR9iP9EhUg1rmduHqwA4ddyHNMcsR8t9saXA9BmMU4t
- 看不见的客人：QmYWwXkgjdhMps9mB6DyEp4zSFmDQ9U6SuqGRGovEycr49
- 勇往直前：QmZRJevYhADpXmCGGF6eCcP1afNEYFahDW5jxje3iyyCJS
- 至暗时刻：QmUPvs7iyM5ZWPQwDovRqvNzxMJHSUWNRWAWRkAsseVcvs
- 银翼杀手2049：QmcUHdzKgRrcJrD5Ah46HgBHF7urWDhmAnLKYwcHaLgeGP
- 盗梦空间：QmQATmpxXvSiQgt9c9idz9k3S3gQnh7wYj4DbdMQ9VGyLh
- 狮子王：QmfHGQZNQNymHDC6b7TZjgGbh962VWQQN5oV92w9jHE4qt
- 祖宗十九代：QmbrwEH4AEQhUN929yPy4j5B2PfQYk3JJyG8iq7HVoXbia
- 疯狂动物城：QmUKaQwN2ppapUEFhbHsKoVXn2yBRM7mLpu5HQv9am7dB7
- 彩绘心天地：QmXg1c6qPtoQAyfrXrWnuDrUgFehnt4kLvv1hxheMUeFBC
- 肖申克的救赎：QmRUYeMkvirV4frGX8wcntCq6x5GqDixAjZnFj5Jg1E3qj
- 太空旅客：QmdxpUVnvFnert9nmEkzwwz2tWdavU3fUQzrgBsTZP5yyG

## 参考
- [How to add site to IPFS and IPNS](https://medium.com/coinmonks/how-to-add-site-to-ipfs-and-ipns-f121b4cfc8ee)
- [IPFS+IPNS搭建简单网页及更新内容](https://www.jianshu.com/p/9e7fb59d2bb5)
- [如何基于IPFS建一个静态网站](https://zhuanlan.zhihu.com/p/32869413)
-[资源类](https://thornbird.org/ipfs-look-movie.html)