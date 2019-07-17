# webconsole 响应 502
## 版本
	oc v3.11.0+62803d0-1
	kubernetes v1.11.0+d4cacc0
	features: Basic-Auth GSSAPI Kerberos SPNEGO
	
	Server https://master195.dmos.dataman:8443
	openshift v3.11.0+06cfa24-67
	kubernetes v1.11.0+d4cacc0
## 故障表象
访问应用端 webconsole 返回 502
## 检查
- curl 检查

		# curl -vk https://localhost/console/
		* About to connect() to localhost port 443 (#0)
		*   Trying ::1...
		* Connection refused
		*   Trying 127.0.0.1...
		* Connected to localhost (127.0.0.1) port 443 (#0)
		* Initializing NSS with certpath: sql:/etc/pki/nssdb
		* skipping SSL peer certificate verification
		* NSS: client certificate not found (nickname not specified)
		* SSL connection using TLS_ECDHE_RSA_WITH_CHACHA20_POLY1305_SHA256
		* Server certificate:
		* 	subject: CN=10.252.6.72
		* 	start date: Jun 14 01:24:24 2018 GMT
		* 	expire date: Jun 13 01:24:25 2020 GMT
		* 	common name: 10.252.6.72
		* 	issuer: CN=openshift-signer@1526615121
		> GET /console/ HTTP/1.1
		> User-Agent: curl/7.29.0
		> Host: localhost
		> Accept: */*
		>
		< HTTP/1.1 502 Bad Gateway
		< Date: Thu, 14 Jun 2018 14:31:38 GMT
		< Content-Length: 0
		< Content-Type: text/plain; charset=utf-8
		<
		* Connection #0 to host localhost left intact
- 服务检查(需要下次来做)

		# oc get service webconsole -n openshift-web-console
- 管理端 webconsole 正常
- oc 命令行正常

## 修复
- 紧急修复

	重启 master 节点正常
- 优雅修复

	尝试使用 github 修复方法 [502 error on webconsole](https://github.com/openshift/origin/issues/20005),但无效
	
		oc project openshift-console
		oc delete secret console-serving-cert
		oc delete pods console*			