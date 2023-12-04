# docker-log
为了解决用户提出的容器日志收集的需求，首先要了解 docker 日志的真正使用方法，所以先看下docker 官方如何定义容器日志使用。
## 查看 docker 日志
通过 `docker logs` 命令查看一个运行中的实例，输出的日志的信息和格式完全依赖于执行的查看命令。 
如果在终端交互命令，容器日志会显示命令输出。unix 系统通常运行 3个i/o，默认 docker 命令显示 stdout 和 stderr。

- stdin

		通过一种方式来输入计算机接收的命令，比如键盘
- stdout

		通常是一个执行程序的正常输出
- stderr

		和 stdout 相反，一个执行程序的错误输出。

下面使用 `docker logs` 将无法接收到日志

- 程序的日志文件
	
		程序任何将日志直接输出到日志文件，而不输出到 stdout\stderr 的日志数据，docker 均将无法获取
	- [nginx 官方镜像](https://github.com/nginxinc/docker-nginx/blob/8921999083def7ba43a06fabd5f80e4406651353/mainline/jessie/Dockerfile#L21-L23)
		
			使用将 /var/log/nginx/access.log 通过一个符号链接到 /dev/stdout，/var/log/nginx/error.log 链接到另外的一个 /dev/stderror
	- [docker 官方](https://github.com/docker-library/httpd/blob/b13054c7de5c74bbaa6d595dbe38969e6d4f860c/2.2/Dockerfile#L72-L75)
		
			将修改nginx 配置将正常输出直接写到  /proc/self/fd/1 即 stdout，错误输出直接写到 /proc/self/fd/2 即 stderr		
- 通过 `logging driver` 转发的日志数据

		通过 logging driver 将日志文件从网络输出到外部存储的，也无法获取。这种情况日志将用其他方式处理，可以不使用 docker logs

## 设置 docker 日志驱动
docker 包含多个记录机制，可以帮助用户从运行中的实例获取信息。这些机制称之为 log 驱动程序。每个 docker deamon都有一个默认的记录机制，但也可以在运行实例的时候修改。
### 配置 docker daemon 的默认 log 驱动
通过使用参数 `--log-driver=<VALUE>` 配置，如果需要更详细的设置可以使用 `--log-opt <NAME>=<VALUE>`。如果没有设定特殊的驱动设置，那么默认使用的是 `json-file`，可以通过 `docker inspect <CONTAINER>` 得到结果。

		$ docker info |grep 'Logging Driver'
		Logging Driver: json-file
### 设置日志驱动给一个实例
当你启动一个实例后，你可以配置使用它不同于默认的日志驱动。日志驱动可以通过使用 `--log-opt <NAME>=<VALUE>` 参数，即使使用相同的驱动也可以使用这个命令进行驱动能力扩展。$ docker 

		inspect -f '{{.HostConfig.LogConfig.Type}}' <CONTAINER>
		json-file
## 支持的驱动
- none

		日志功能不激活，将没有日志可以从 docker logs 获取
- json-file

		docker 日志默认的驱动方式，使用 json 格式文件保存。
- syslog

		将日志信息写入 syslog 设备，syslog daemon 必须和 docker 安装在同一本机
- journald

		将日志信息写入 journald， journald daemon 必须和 docker 安装在同一本机
- gelf

		将日志的信息发送到一个 Graylog 扩展日志格式 (GELF)节点上，例如 graylog 或者 logstash 
- fluentd

		日志信息发送到 fluentd，fluentd 必须和 docker 安装在同一本机
- awslogs

		日志信息发送到 amazon cloudwatch 日志系统
- splunk

		日志信息发送到 splunk 的 HTTP 事件控制器.
- etwlogs

		日志信息发送到 Event Tracing for Windows (ETW) events。只有 win 平台使用
- gcplogs

		日志输出到  Google Cloud Platform (GCP) 日志平台
## 展现的局限性
`docker logs` 命令展现日志只可以适用于 `json-file` 和 `jourald` 模式，其他模式将无法得到输出。
## 例子
### 设置日志使用的标签(lables)和环境变量(env)
可以对不同的日志实例设置特殊的环境变量和标签，有些驱动需要使用到这些参数。 如果环境变量和标签发生冲突，那么环境变量的优先级最高。参数可以通过几种方法增加：
- docker daemon 设置

		dockerd \
         --log-driver=json-file \
         --log-opt labels=production_status \
         --log-opt env=os,customer
- 或者启动实例的时候增加

		docker run -dit --label production_status=testing -e os=ubuntu alpine sh

如果日志启动支持，这个标签将作为日志输出同时输出，如 `json-file`

		"attrs":{"production_status":"testing","os":"ubuntu"}
## 参数说明
除了设置 `none` 这个驱动将关闭 docker 日志启动能力,不需要增加参数外，其他驱动都可以增加更多参数达到更细粒度的日志控制。
### 通用参数
注意:这些参数只适用于 docker ce edge 版本

- mode

	插入日志的模式，默认参数为 `blocking`和 `non-blocking`,当设置为 `non-blocking`时，如果日志缓存区填满，日志消息将丢失。信息丢失为定义。
	
	- 例
		
			--log-opt mode=non-blocking 
- max-buffer-size								
	 只有 mod 设置为 `non-blocking`时，才可以设置这个参数，参数定义一旦到达了此设置，日志信息将被抛弃。
	 
	- 例

			--log-opt max-buffer-size 5m

### json-file 参数
- max-size

	日志当到达设置大小后进行滚动，单位设置为(k、m、g)
	
	- 例
	
			--log-opt max-size=10m		
- max-file

	当 max-size 参数生效时才生效，当滚转的文件数量大于这个设定，将被删除，设置值必须是整数。
	
	- 例

			--log-opt max-file=3
- lables[高级设置](https://docs.docker.com/engine/admin/logging/log_tags/)
		
	适用于启动 docker daemon 时，它将接收日志相关的标签(用逗号分隔的列表)
	
	- 例
	
			--log-opt labels=production_status,geo
- env[高级设置](https://docs.docker.com/engine/admin/logging/log_tags/)

	适用于启动 docker daemon 时，它将接收日志相关的环境变量(用逗号分隔的列表)
	
	- 例
	
			--log-opt env=os,customer

注意: 当 `max-size` 和 `max-file` 被设定的时候， docker logs 只返回最后一个滚转的日志文件。

### syslog 参数
- syslog-addrees

	设置一个外部的 syslog 服务器，格式为:[tcp|udp|tcp+tls]://host:port, unix://path 或 unixgram://path。如果传输定义为 tcp、udp 或者 tcp+tls ，那么默认端口为 514
	
	- 例
	
			--log-opt syslog-address=tcp+tls://192.168.1.3:514
			--log-opt syslog-address=unix:///tmp/syslog.sock
- syslog-facility

	syslog 设备定义，可以使用任何合法的数字或者名字，可以查看[官方定义](https://tools.ietf.org/html/rfc5424#section-6.2.1)
	
	- 例

			--log-opt syslog-facility=daemon
- syslog-tls-ca-cert

	签发的ca证书完整路径，当设置为 tcp+tls 时
	
	- 例

			--log-opt syslog-tls-ca-cert=/etc/ca-certificates/custom/ca.pem
- syslog-tls-cert

	tls 证书完整路径，当设置为 tcp+tls 时
	
	- 例

			--log-opt syslog-tls-cert=/etc/ca-certificates/custom/cert.pem
- syslog-tls-key

	tls 密钥文件完整路径，当设置为 tcp+tls 时
	
	- 例

			--log-opt syslog-tls-key=/etc/ca-certificates/custom/key.pem
- syslog-tls-skip-verify

	如果设置为 true 时，遇到 tls 链接验证的时会跳过,当设置为 tcp+tls 时，默认为 flase。
	
	- 例
		
			--log-opt syslog-tls-skip-verify=true		
- tag

	这个字符串将追加 APP-NAME 到 syslog 的信息中，默认 docker 将从容器id获取前12个字符进行发送。[高级](https://docs.docker.com/engine/admin/logging/log_tags/)
	
	- 例

			--log-opt tag=mailer
- syslog-format

	定义 syslog 的信息格式。如果没有明确标注，将使用本地的 unix 格式，没有标注名。可用格式
	
	- rfc3164
	- rfc5424
	- rfc5424micro(微秒时间戳)
	- 例

			--log-opt syslog-format=rfc5424micro
- lables[高级设置](https://docs.docker.com/engine/admin/logging/log_tags/)
		
	适用于启动 docker daemon 时，它将接收日志相关的标签(用逗号分隔的列表)
	
	- 例
	
			--log-opt labels=production_status,geo
- env[高级设置](https://docs.docker.com/engine/admin/logging/log_tags/)

	适用于启动 docker daemon 时，它将接收日志相关的环境变量(用逗号分隔的列表)
	
	- 例
	
			--log-opt env=os,customer
- 例1

		docker run \
         --log-driver=syslog \
         --log-opt syslog-address=tcp://192.168.0.42:123 \
         --log-opt syslog-facility=daemon \
         alpine ash
- 例2

		docker run \
         --log-driver=syslog \
         --log-opt syslog-address=tcp+tls://192.168.0.42:123 \
         --log-opt syslog-tls-ca-cert=syslog-tls-ca-cert=/etc/ca-certificates/custom/ca.pem \
         --log-opt syslog-tls-cert=syslog-tls-ca-cert=/etc/ca-certificates/custom/cert.pem \
         --log-opt syslog-tls-key=syslog-tls-ca-cert=/etc/ca-certificates/custom/key.pem \
         alpine ash
                  				
### journald 参数
 journald 日志驱动将存储容器 id 到 journald 的 CONTAINER_ID 字段。[详细信息](https://docs.docker.com/engine/admin/logging/journald/)

- tag

	设定 journald 中的 CONTAINER_TAG 值
	
	- 例

			--log-opt tag=mailer
- lables[高级设置](https://docs.docker.com/engine/admin/logging/log_tags/)
		
	适用于启动 docker daemon 时，它将接收日志相关的标签(用逗号分隔的列表)
	
	- 例
	
			--log-opt labels=production_status,geo
- env[高级设置](https://docs.docker.com/engine/admin/logging/log_tags/)

	适用于启动 docker daemon 时，它将接收日志相关的环境变量(用逗号分隔的列表)
	
	- 例
	
			--log-opt env=os,customer
- 例

		docker run \
         --log-driver=journald \
         alpine ash

### gelf 参数
- gelf-address
	- 例

			--log-opt gelf-address=udp://192.168.0.42:12201          
- gelf-compression-type
	- 例
	
			--log-opt gelf-compression-type=gzip
- gelf-compression-level
	- 例

			--log-opt gelf-compression-level=2
- tag
	- 例

			--log-opt tag=mailer
- labels
	- 例

			--log-opt labels=production_status,geo
- env
	- 例

			--log-opt env=os,customer 		
- 例

		docker run -dit \
             --log-driver=gelf \
             --log-opt gelf-address=udp://192.168.0.42:12201 \
             alpine sh				

### fluentd 参数
- fluentd-address
	- 例

			--log-opt fluentd-address=192.168.0.42:24224 
- fluentd-buffer-limit
	- 例

			--log-opt fluentd-buffer-limit=8MB
- fluentd-retry-wait
	- 例
	
			--log-opt fluentd-retry-wait=1000ms
- fluentd-max-retries
	- 例
	
			--log-opt fluentd-max-retries=200
- fluentd-async-connect
	- 例
			
			--log-opt fluentd-async-connect=false
- tag
	- 例

			--log-opt tag=mailer
- 例

		docker run -dit \
             --log-driver=fluentd \
             --log-opt fluentd-address=localhost:24224 \
             --log-opt tag="docker.{{.Name}}" \
             alpine sh

### awslogs
- awslogs-region
	- 例

			--log-opt awslogs-region=us-east-1 
- awslogs-group
	- 例

			--log-opt awslogs-group=myLogGroup 
- awslogs-stream
	- 例
			
			--log-opt awslogs-stream=myLogStream
- awslogs-create-group
	- 例

			--log-opt awslogs-create-group=true 
- 例

		docker run \
         --log-driver=awslogs \
         --log-opt awslogs-region=us-east-1 \
         --log-opt awslogs-group=myLogGroup \
         --log-opt awslogs-create-group=true \
         alpine sh						 
         
### splunk
暂无
### etwlogs
暂无
### gcplogs
暂无
## 日志启动中的 tag
标记日志选项如何格式化日志信息，默认使用容器id前12个字符，覆盖方法

	docker run --log-driver=xxx --log-opt xxx=xxx --log-opt tag="mailer"
docker 支持从实例中获取标记信息

- {{.ID}}

	容器id前12个字符
- {{.FullID}}

	容器id全字符
- {{.Name}}

	容器名
- {{.ImageID}}

	镜像id前12个字符
- {{.ImageFullID}}

	镜像全id
- {{.ImageName}}

	镜像名
- {{.DaemonName}}

	docker 守护进程名(docker)
例如使用 syslog 定义标签如: `--log-opt tag="{{.ImageName}}/{{.Name}}/{{.ID}}"` 打印的日志如 `Aug  7 18:33:19 HOSTNAME docker/hello-world/foobar/5790672ab6a0[9103]: Hello from Docker.`

当使用 `container_name` 字段获取 `{{.Name}}` tag,如果使用命令 `docker rename` 修改了一个实例的名字，那么新名字无法进入这个实例的日志信息，还会使用老名字。对于高级使用 [go 模版](http://golang.org/pkg/text/template/)和[日志关联](https://github.com/moby/moby/blob/17.05.x/daemon/logger/loginfo.go) 

- 例

		docker run -it --rm \
    		--log-driver syslog \
    		--log-opt tag="{{ (.ExtraAttributes nil).SOME_ENV_VAR }}" \
    		--log-opt env=SOME_ENV_VAR \
    		-e SOME_ENV_VAR=logtester.1234 \
    		flyinprogrammer/logtester																输出日志
    	Apr  1 15:22:17 ip-10-27-39-73 docker/logtester.1234[45499]: + exec app
		Apr  1 15:22:17 ip-10-27-39-73 docker/logtester.1234[45499]: 2016-04-01 15:22:17.075416751 +0000 UTC stderr msg: 1	
## Journald 日志驱动
journald 日志驱动将实例日志发送到 systemd 的 journal 中。日志可以通过 `journalctl` 命令行或者 journal api 或者 docker logs 命令行查询。除了日志消息本身，驱动还将以下信息默认插入到每条原数据中。

- CONTAINER_ID

	容器 id 前 12个字符
- CONTAINER_ID_FULL

	容器 id 全部64字符
- CONTAINER_NAME

	容器名字，通过 `docker rename` 修改的新名字无法获取
- CONTAINER_TAG

	容器 tag [参数](https://docs.docker.com/engine/admin/logging/log_tags/) 
- CONTAINER_PARTIAL_MESSAGE   	
    一个标示日志完整性的字段，改善多行日志问题。	   
### 使用
- 设置驱动到 docker daemon

		dockerd --log-driver=journald
- 设置驱动到实例中

		docker run  --log-driver=journald ...

### 参数
使用 `--log-opt NAME=VALUE` 参数添加额外的启动选项

- tag

	通过加入 `CONTAINER_TAG` 值进入 journald 日志。
- labels 和 env
	  适用于启动 docker daemon 时，它将接收日志相关的标签或者环境变量(用逗号分隔的列表)  ，如果冲突环境变量的优先级更高。这个会加入到每条信息的元数据中。  

### 通过 journalctl 查询日志
使用 `journalctl` 命令行查询日志，可以通过添加过滤信息来过滤日志，方法:

		sudo journalctl CONTAINER_NAME=webserver
通过添加 `journalctl` 参数来过滤信息，`-b` 参数只获取上次启动后的信息

		sudo journalctl -b CONTAINER_NAME=webserver
通过 `-o` 指定查询的日志格式，比如输出 json 格式

		sudo journalctl -o json CONTAINER_NAME=webserver
### 从 journal api 获取日志
例子使用 systemd python 模块获取日志

		import systemd.journal

		reader = systemd.journal.Reader()
		reader.add_match('CONTAINER_NAME=web')

    		for msg in reader:
      			print '{CONTAINER_ID_FULL}: {MESSAGE}'.format(**msg)							       
## 参考资料
[docker logs](https://docs.docker.com/engine/admin/logging/)