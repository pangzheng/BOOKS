# docker tls 认证

创建人： 刘金烨

创建时间：2017-06-08

参考资料：

```
https://docs.docker.com/engine/security/https
http://dockone.io/article/1346
```

## 创建证书

修改下面脚本中的 LOCAL_ENNAME 变量设置网卡名称，在需要配置docker tls的服务器执行下面脚本生成证书相关文件

cat mkpems.sh

```
#!/bin/bash
# Author: jyliu
# Date: 2017-06-08

DOMAIN=offlinedocker
LOCAL_ENNAME=ens192 ## Need to check
LOCAL_IP=`ip a show $LOCAL_ENNAME|awk '/inet.*brd.*'$LOCAL_ENNAME'/{print $2}'|awk -F "/" '{print $1}'`

create_ca(){
	# 生成ca key
	openssl genrsa -out ca-key.pem 4096

	# 创建ca证书
	openssl req -new -x509 -days 365 -key ca-key.pem -sha256 -out ca.pem -subj "/C=CN/ST=bejjing/L=dataman/O=dataman/OU=dataman/CN=$DOMAIN"

}

create_server(){
	item=server
	# 创建服务key
	openssl genrsa -out ${item}-key.pem 4096

	# 创建根证书
	openssl req -subj "/CN=$DOMAIN" -sha256 -new -key ${item}-key.pem -out ${item}.csr

	echo subjectAltName = IP:$LOCAL_IP,IP:127.0.0.1 > extfile.cnf

	# 创建服务证书
	openssl x509 -req -days 365 -sha256 -in ${item}.csr -CA ca.pem -CAkey ca-key.pem \
	  -CAcreateserial -out ${item}-cert.pem -extfile extfile.cnf

}

create_clent(){
	# 创建client key
	openssl genrsa -out key.pem 4096
	# 创建client 证书
	openssl req -subj '/CN=client' -new -key key.pem -out client.csr

	echo extendedKeyUsage = clientAuth > extfile.cnf

	openssl x509 -req -days 365 -sha256 -in client.csr -CA ca.pem -CAkey ca-key.pem \
 	 -CAcreateserial -out cert.pem -extfile extfile.cnf
}

clean(){
	rm -f client.csr server.csr extfile.cnf ca.srl
}

update_file_permissions(){
        chmod -v 0400 ca-key.pem key.pem server-key.pem
        chmod -v 0444 ca.pem server-cert.pem cert.pem
}

# 创建ca证书和key
create_ca
# 创建server证书和key
create_server
# 创建client证书和key
create_clent
# 清理无用文件
clean
# 修改文件权限
update_file_permissions
```

执行完成后生成下面几个文件

```
ca-key.pem
ca.pem
cert.pem
key.pem
mkcsr.sh
server-cert.pem
server-key.pem
```

注：ca 和 client 只生成一次即可，其它主机使用相同ca证书生成server证书

## 修改docker daemon 启动参数 

复制server 相关pem 到 /etc/docker/certs.d/offlinedocker

```
mkdir -p /etc/docker/certs.d/offlinedocker

cp -v {ca,server-cert,server-key}.pem /etc/docker/certs.d/offlinedocker
```

修改docker daemon参数 

vi /usr/lib/systemd/system/docker.service

添加或修改下面内容（docker tls 默认端口2376）

```
--tlsverify --tlscacert=/etc/docker/certs.d/offlinedocker/ca.pem --tlscert=/etc/docker/certs.d/offlinedocker/server-cert.pem --tlskey=/etc/docker/certs.d/offlinedocker/server-key.pem -H=192.168.1.214:2376
```

## 

重启docker

```
systemctl daemon-reload
service docker restart
```
## 测试docker api tls

```
curl https://192.168.1.214:2376/images/json \
  --cert ./cert.pem \
  --key ./key.pem \
  --cacert ./ca.pem
```

## docker https 健康检查注册到consul

```
curl -L -X PUT -d '{"id":"docker-192.168.1.214", "name": "docker", "tags": [""], "port": 2376,"check": {"script": "curl -s https://192.168.1.214:2376/_ping --cert /root/docker_tls/cert.pem --key /root/docker_tls/key.pem --cacert /root/docker_tls/ca.pem", "interval": "10s","timeout": "1s"}}' 192.168.1.214:8500/v1/agent/service/register
```
