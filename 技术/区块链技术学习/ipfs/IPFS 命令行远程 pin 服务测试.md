# IPFS 命令行远程 pin 服务测试
## 帮助
	 % ipfs pin remote -h
	USAGE
	  ipfs pin remote - Pin (and unpin) objects to remote pinning service.
	
	  ipfs pin remote
	
	SUBCOMMANDS
	  ipfs pin remote add <ipfs-path> - Pin object to remote pinning service.
	  ipfs pin remote ls              - List objects pinned to remote pinning
	                                    service.
	  ipfs pin remote rm              - Remove pins from remote pinning service.
	  ipfs pin remote service         - Configure remote pinning services.
	
	  For more information about each command, use:
	  'ipfs pin remote <subcmd> --help'	 
## Pin 服务配置
### 查看 Pin 服务
- 简单查询命令与结果
	
		ipfs pin remote service ls
			Filebase https://api.filebase.io/v1/ipfs
			Pinata   https://api.pinata.cloud/psa
- 查询 Pin 服务状态

		ipfs pin remote service ls --stat
			Filebase https://api.filebase.io/v1/ipfs invalid
			Pinata   https://api.pinata.cloud/psa    0/0/2/0			
### 添加 pin 服务
添加所有符合规范的提供商，比如 pinata、filebase 等等，注意这里不检查 API 是否可用，只添加数据 

- 模版	
	
		ipfs pin remote service add <service> <endpoint> <key>
		
	- `service `

		服务名称，任意写
	- `endpoint `

		服务的 API 地址，注意大部分服务分为存储服务 API 和 Pin 服务 API ，你需要填写 Pin 服务的 API 
	- `key `

		服务 key ，需要去服务页面或者 API 申请	
- 例子	
	- 命令
		
			ipfs pin remote service add chm https://api.pinata.cloud/psa eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VySW5mb3JtYXRpb24iOnsiaWQiOiI4MGZkY2Q1MS02MDI4LTQwZTYtYjZmMC1lMTMwMTQ5Y2MyMTUiLCJlbWFpbCI6InBhbmd6aGVuZ0BnbWFpbC5jb20iLCJlbWFpbF92ZXJpZmllZCI6dHJ1ZSwicGluX3BvbGljeSI6eyJyZWdpb25zIjpbeyJpZCI6IkZSQTEiLCJkZXNpcmVkUmVwbGljYXRpb25Db3VudCI6MX1dLCJ2ZXJzaW9uIjoxfSwibWZhX2VuYWJsZWQiOmZhbHNlLCJzdGF0dXMiOiJBQ1RJVkUifSwiYXV0aGVudGljYXRpb25UeXBlIjoic2NvcGVkS2V5Iiwic2NvcGVkS2V5S2V5IjoiODc1NDI3MWUwNGJmMDA4YTc3OTYiLCJzY29wZWRLZXlTZWNyZXQiOiI4MTdkOTZkYmYzY2FhN2IwNzJiMTk4ZDEwM2Y0YmZkOGY3ZGFlNGYyMTAwZmEwNDI0OGU5YTY3Y2Y1MzQ4ODYyIiwiaWF0IjoxNjc5MzgzOTIwfQ.DXuzo-brH2oXpAwWmXM29xHWUyQQosD9nmTbkKhfhCE
	- 响应

		200
	- 查询命令与结果

			% ipfs pin remote service ls
				Filebase https://api.filebase.io/v1/ipfs
				Pinata   https://api.pinata.cloud/psa
				chm      https://api.pinata.cloud/psa

### 删除 Pin 服务配置
添加所有符合规范的提供商，比如 pinata、filebase 等等，注意这里不检查 API 是否可用，只添加数据 

- 模版	
	
		ipfs pin remote service rm <service>
		
	- `service `

		之前写的服务名
- 例子	
	- 命令
		- 查看已有服务

				% ipfs pin remote service ls
					Filebase https://api.filebase.io/v1/ipfs
					Pinata   https://api.pinata.cloud/psa
					chm      https://api.pinata.cloud/psa
		- 删除 chm

				% ipfs pin remote service rm chm
		- 查看删除结果

				% ipfs pin remote service ls
					Filebase https://api.filebase.io/v1/ipfs	
					Pinata   https://api.pinata.cloud/psa

## 远程 Pin 服务端对象操作
### 查看对象
通过命令查看远程服务端存储的文件

- 模版

		ipfs pin remote ls [--service=<service>] [--name=<name>] [--cid=<cid>]... [--status=<status>]
	- `service`

		之前写的服务名
	- `name`

		远程对象名称
	- `cid`

		远程对象的 cid
	- `status`

		对象的 pin 状态	
- 查询所有对象

		% ipfs pin remote ls --service chm
			QmWgnG7pPjG31w328hZyALQ2BgW5aQrZyKpT47jVpn8CNo	pinned	Tarkov.mp4
			QmUtDugQiYihkgeJz1QBTbwEiZm7R2UVYcrpPDPDL5TczW	pinned	xxa.pdf
- 查询对应 cid 对象

		ipfs pin remote ls --service Pinata --cid QmWgnG7pPjG31w328hZyALQ2BgW5aQrZyKpT47jVpn8CNo
		QmWgnG7pPjG31w328hZyALQ2BgW5aQrZyKpT47jVpn8CNo	pinned	Tarkov.mp4
- 查询对应对象名

		% ipfs pin remote ls --service Pinata --name xxa.pdf
			QmUtDugQiYihkgeJz1QBTbwEiZm7R2UVYcrpPDPDL5TczW			pinned	xxa.pdf		
- 查询对应 pin 状态，注意这里的状态是队列状态,包括 `"queued" "pinning" "pinned" "failed"`

		ipfs pin remote ls --service Pinata --status=pinned
			QmZAaMiZVddnSEps7jWAba3gmzobLxD8pXUKbJwNpcjeST	pinned	javelin-720p.mp4
			QmWgnG7pPjG31w328hZyALQ2BgW5aQrZyKpT47jVpn8CNo	pinned	Tarkov.mp4
			QmUtDugQiYihkgeJz1QBTbwEiZm7R2UVYcrpPDPDL5TczW	pinned	xxb.pdf

## 向 Pin 服务端添加文件
通过 CID 向 Pin 服务添加一个本地 IPFS 客户端已有账户

- 模版

		 ipfs pin remote add <cid> --service <service>
		 
	- `cid`

		添加对应的 CID
	- `service `

		之前写的服务名
- 例子
	- 命令

			 ipfs pin remote add QmWgnG7pPjG31w328hZyALQ2BgW5aQrZyKpT47jVpn8CNo --service chm
	- 响应

			CID:	QmWgnG7pPjG31w328hZyALQ2BgW5aQrZyKpT47jVpn8CNo
			Name:
			Status: pinned
	- 查询 

			% ipfs pin remote ls --service chm
- 问题

	注意：你当前的 IPFS 客户端需要有公网 IP，否则提示供给者错误
	
		Error: reason: "INVALID_ORIGINS", details: "provider array entry: /ip6/::1/udp/4001/quic-v1/webtransport/certhash/uEiDDjAOAbRxx05p6gZaLJ_Ro5YtXWVsYySLgpksAhPEgIg/certhash/uEiB-jVdJ65hyYZ2asid1NC84Ip9rBk5GY4x1AkblQYe5Mw/p2p/12D3KooWMyJc5cHbQv5hoLzuRy3uCsVs8XXeo2ZK2dq8Dq53WTb5 is not a valid peer multiaddr": 400 Bad Request			
## 删除远程 Pin 服务端上的对象
通过命令删除远程 Pin 服务上的对象

- 模版

		ipfs pin remote rm [--service=<service>] [--name=<name>] [--cid=<cid>]... [--status=<status>]... [--force]
		
	- `cid`

		添加对应的 CID
	- `service `

		之前写的服务名
	- `name`
	
		文件名
	- `status`

		pinned 状态
	- `force`

		命中多的对象时需要添加			
- 例子
	- 查询远程服务器对象

			%  ipfs pin remote ls --service Pinata
				Qmac3QvJs353vrr3G4sV2nQKWNv3w6ufU3DbMWUaiohzfj	pinned	delect-test.png
				QmRajrhBHhkenmQYjxuMD1ysWLxCShXRrNJzUqhBH3jtxV	pinned	delect-test.png
				QmPYYqzVTviAggTQK5SSj9JoR3z5TaDWiAqfidRibQYcG6	pinned	vr.m4v
				QmZAaMiZVddnSEps7jWAba3gmzobLxD8pXUKbJwNpcjeST	pinned	javelin-720p.mp4
				QmWgnG7pPjG31w328hZyALQ2BgW5aQrZyKpT47jVpn8CNo	pinned	Tarkov.mp4
	- 按照 CID 删除

			ipfs pin remote rm --service Pinata --cid QmPYYqzVTviAggTQK5SSj9JoR3z5TaDWiAqfidRibQYcG6
				200
		- 查询
	
				%  ipfs pin remote ls --service Pinata
					Qmac3QvJs353vrr3G4sV2nQKWNv3w6ufU3DbMWUaiohzfj	pinned	delect-test.png
					QmRajrhBHhkenmQYjxuMD1ysWLxCShXRrNJzUqhBH3jtxV	pinned	delect-test.png
					QmZAaMiZVddnSEps7jWAba3gmzobLxD8pXUKbJwNpcjeST	pinned	javelin-720p.mp4
					QmWgnG7pPjG31w328hZyALQ2BgW5aQrZyKpT47jVpn8CNo	pinned	Tarkov.mp4
	- 按照名字删除

			% ipfs pin remote rm --service Pinata --name delect-test.png
				200
		- 提示多对象命中			
		
				Error: multiple remote pins are matching this query, add --force to confirm the bulk removal
			- 增加参数
				
					ipfs pin remote rm --service Pinata --name delect-test.png --force
						200
			- 查询

					%  ipfs pin remote ls --service Pinata
						QmZAaMiZVddnSEps7jWAba3gmzobLxD8pXUKbJwNpcjeST	pinned	javelin-720p.mp4
						QmWgnG7pPjG31w328hZyALQ2BgW5aQrZyKpT47jVpn8CNo	pinned	Tarkov.mp4
	- 按照状态删除

			% ipfs pin remote rm --service Pinata --status pinned
				200
		- 提示多对象命中			
		
				Error: multiple remote pins are matching this query, add --force to confirm the bulk removal
	
							
		 		