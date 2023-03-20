# Docker 卷备份
	使用 Docker Volume Backup 将 Docker 容器卷备份到 Filebase。
## 什么是 Docker 卷备份？
[Docker volumeBackup](https://github.com/offen/docker-volume-backup) 是一个 Docker 镜像，可以用作任何现有 Docker 设置的 sidecar 容器。这个 sidecar 容器可以配置为将 Docker 容器卷定期或一次性备份到任何 S3 兼容存储。它还可以配置为定期删除旧备份并发送备份失败的电子邮件通知。

- 先决条件：
	- 下载并安装 [docker](https://docs.docker.com/get-docker/) 和。[docker compose](https://docs.docker.com/compose/install/)
	- 注册一个免费的 Filebase 帐户。
	- 拥有您的文件库访问权限和密钥。了解如何查看您的访问密钥。
	- 创建文件库存储桶。了解如何创建存储桶。

## 在 docker-compose 文件中配置定期备份
以下是 docker-compose.yml 文件的示例配置，用于将备份服务添加到您的组合设置并安装您希望备份的卷：

	version: '3'
	
	services:
	  volume-consumer:
	    build:
	      context: ./my-container
	    volumes:
	      - data:/var/my-container
	    labels:
	      # 这意味着容器将在备份期间停止，以确保备份完整性。
	      - docker-volume-backup.stop-during-backup=true
	
	  backup:
	    image: offen/docker-volume-backup:latest
	    restart: always
			environment:
					AWS_S3_BUCKET_NAME="filebase-bucket-name"
					AWS_ACCESS_KEY_ID="Filebase-Access-Key"
					AWS_SECRET_ACCESS_KEY="Filebase-Secret-Key"
					AWS_ENDPOINT="s3.filebase.com"
					AWS_ENDPOINT_PROTO="https"
	   
	    volumes:
	      - data:/backup/my-app-backup:ro
	     
	      # 挂载 Docker sock 允许脚本在备份期间停止并重新启动容器。如果你不想停止容器，你可以省略这个
	      - /var/run/docker.sock:/var/run/docker.sock:ro
	     
	      # 如果您将本地目录或卷挂载到 '/archive'，则备份的本地副本将存储在那里。你可以通过设置 'BACKUP_ARCHIVE' 来覆盖容器内部的位置。如果不想保留本地备份，可以省略此选项。
	      - /path/to/local_backups:/archive
	volumes:
	  data:


### `env_file` 模板：
填写以下环境变量模板并将其保存 `backup.env` 在 docker-compose 文件的目录中。如果使用此方法，则可以将 docker-compose 文件中的环境变量部分替换为 `env-file: ./backup.env`.

	########### 备份计划
	
	# 备份以 “busybox” 风格在给定的 cron 调度上运行。如果没有设置将使用 “@daily”。如果您不希望运行 cron，请使用 `0 0 5 31 2 ?`.
	# BACKUP_CRON_EXPRESSION="0 2 * * *"
	
	
	# 备份文件的名称，包括' .tar.gz '扩展名。默认的结果是这样的文件名 `backup-2021-08-29T04-00-00.tar.gz`.
	# BACKUP_FILENAME="backup-%Y-%m-%dT%H-%M-%S.tar.gz"
	
	# 创建 tar 归档前是否拷贝备份文件夹内容。 在极少数情况下，源备份卷的内容在不断更新，但我们不希望在执行备份时停止容器，可以使用此设置来确保 tar.gz 文件的完整性。
	# BACKUP_FROM_SNAPSHOT="false"
	
	########### 备份存储

	# Filebase 桶名称
	# AWS_S3_BUCKET_NAME="filebase-bucket-name"
	
	# Filebase 认证
	# AWS_ACCESS_KEY_ID="Filebase-Access-Key"
	# AWS_SECRET_ACCESS_KEY="Filebase-Secret-Key"
	
	# Filebase API 端点 
	# AWS_ENDPOINT="s3.filebase.com"
	
	# Filebase 只支持 https.
	# AWS_ENDPOINT_PROTO="https"
	
	# 将此变量设置为 “true” 将禁用 SSL 证书验证。除非对远程存储后端使用自签名证书，否则不应该使用这种方法。这只能在AWS_ENDPOINT_PROTO 设置为 “https” 时使用。
	# AWS_ENDPOINT_INSECURE="true"
	
	# 除了远程存储备份外，还可以保留本地副本。如果需要，传递一个容器本地路径来存储备份。运行容器时，还需要将本地文件夹或 Docker卷挂载到该位置(默认为 '/archive')。如果在备份运行时容器中不存在指定的目录(没有挂载任何东西)，则将跳过本地备份。本地路径也会按照下面的定义修剪旧备份。
	# BACKUP_ARCHIVE="/archive"
	
	########### 备份修剪
	
	# **重要，请在使用此功能**: 用于删除旧备份的机制不是很复杂，默认情况下将其规则应用于**目标目录中的所有文件**，这意味着如果您将备份存储在其他文件旁边，这些文件也可能会被删除。使用此选项时，请确保备份文件存储在专门用于此类文件的目录中或者配置BACKUP_PRUNING_PREFIX 以限制删除某些文件。
	
	# 定义此值以启用旧备份的自动旋转。该值声明保留备份的天数。
	# BACKUP_RETENTION_DAYS="7"
	
	# 如果备份的持续时间在您的设置中明显波动，您可以调整此设置，以确保在备份完成和旋转之间没有竞争条件，不删除位于时间窗口边缘的备份。将此值设置为一个持续时间，期望该持续时间大于备份的最大差异。有效值后缀为(s) seconds， (m)inutes或(h)ours。缺省值为1分钟。
	# BACKUP_PRUNING_LEEWAY="1m"
	
	# 如果您的目标桶或目录包含除此容器管理的文件之外的其他文件，您可以通过设置前缀值来限制旋转范围。这通常是 BACKUP_FILENAME 的非参数化部分。例如，如果 BACKUP_FILENAME 是' db-backup-%Y-%m-%dT%H-% m-% S.tar.gz '，您可以将BACKUP_PRUNING_PREFIX 设置为 'db-backup-'，并确保不相关的文件不受旋转机制的影响。
	# BACKUP_PRUNING_PREFIX="backup-"
	
	########### 备份加密
	
	# 如果给出了密码，备份可以使用gpg加密。
	# GPG_PASSPHRASE="Passphrase"
	
	########### 在备份期间停止容器
	
	# 可以通过应用 “docker-volume-backup” 来停止容器。stop-during-backup” 标签。默认情况下，所有标记为' true '的容器将被停止。如果你需要更细粒度的控制(例如，当基于这个映像运行多个容器时)，你可以在这里指定一个不同的值来覆盖这个默认值。
	# BACKUP_STOP_CONTAINER_LABEL="service1"
	
	########### 备份运行失败时的电子邮件通知
	
	# 如果提供了SMTP凭据，则可以在备份运行失败时发送通知电子邮件。这些电子邮件将包含开始时间、错误消息和失败前的所有日志输出。
	
	# 通知的接收者。如果要通知多个收件人，请提供以逗号分隔的地址列表。如果没有设置，将不会发送电子邮件。
	# EMAIL_NOTIFICATION_RECIPIENT="you@example.com"
	
	# 发送邮件的“发件人”标头。默认为' noreply@nohost '。
	# EMAIL_NOTIFICATION_SENDER="no-reply@example.com"
	
	# 要使用的SMTP服务器的配置和凭据。
	# EMAIL_SMTP_PORT 默认 587.
	
	# EMAIL_SMTP_HOST="posteo.de"
	# EMAIL_SMTP_PASSWORD="SMTP-password"
	# EMAIL_SMTP_USERNAME="no-reply@example.com"
	# EMAIL_SMTP_PORT="SMTP-port"

## 使用 Docker CLI 进行一次性备份
要运行一次性备份，请将要备份的卷装入容器并运行备份命令：

	docker run --rm \\
	-v data:/backup/data \\
	--env AWS_ACCESS_KEY_ID="Filebase-Access-Key" \\
	--env AWS_SECRET_ACCESS_KEY="Filebase-Secret-Key" \\
	--env AWS_S3_BUCKET_NAME="Filebase-Bucket" \\
	--env AWS_ENDPOINT="s3.filebase.com" \\
	--env AWS_ENDPOINT_PROTO="https" \\
	--entrypoint backup \\
	offen/docker-volume-backup:latest
或者，您可以backup.env按上述方式传递文件：

	docker run --rm \\
	-v data:/backup/data \\
	--env-file ./backup.env

## 从备份恢复卷
要从备份还原卷，请使用以下工作流程：

1. 停止正在使用该卷的容器。
2. 解压要恢复的备份：

		tar -C /tmp -xvf  backup.tar.gz
3. 使用临时容器装载卷（在本例中称为“数据”）并复制备份。

		docker run -d --name backup_restore -v data:/backup_restore alpine
		docker cp /tmp/backup/data-backup backup_restore:/backup_restore
		docker stop backup_restore
		docker rm backup_restore
4. 重新启动正在使用该卷的容器。

根据您的设置，可能需要额外的步骤。