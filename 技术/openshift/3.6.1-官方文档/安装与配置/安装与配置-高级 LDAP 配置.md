# 安装与配置-高级 LDAP 配置
## 为 LDAP 故障转移设置 SSSD
openshift 提供了一个 LDAP 设置的身份验证提供程序，但它只能连接单个的 LDAP 服务器。如果 LDAP 服务器不可用，会出现很多问题。系统安全服务守护进程 SSSD 可用于解决这个问题。

SSSD 最初设计用来管理对主机操作系统的本地和远程认证，现在可以配置为向 openshift 等 web 服务提供身份认证、授权、授权服务。SSSD 提供优于内置 LDAP 提供程序的优势，包括能够连接任意数量的故障转移 LDAP 服务器，并且能够缓存身份验证，以防止无法再访问任何这些服务器。

此配置的设置是高级的，需要单独认证服务器才能与 openshift 进行通讯。本主题介绍如何在专用服务器上执行设置 ，但这些概念也适用于容器设置。
### 验证代理安装的先决条件
- 在开始安装之前，需要知道有关 LDAP 服务器以下信息.
	- 目录服务器是否由  FreeIPA, 活动目录 或者其他 LDAP 解决方案支持。
	- LDAP 服务器的统一资源标识符(URI),如 ldap.example.com
	- LDAP 服务器的 ca 证书位置
	- LDAP 服务器是否对应于用户组的 RFC 2307 或 RFC2307bis
- 虚拟机准备
	- proxy.example.com

		用作身份验证代理的虚拟机。该机器必须至少有 SSSD 1.12.0 可用，这意味着一个相当新的操作系统。用 centos 7.2
	- openshift.example.com 

		用于运行 openshift 的虚拟机。这些虚拟机可以配置为在同一个系统上运行，但对于本主题使用的例子，他们保持独立。

### 第一阶段:生成证书
在 ansible 清单文件中列出第一个主控主机上执行此过程，默认情况下为 `/etc/ansible/hosts`

- 为确保身份验证代理和openshift 之前的通讯值得信任，请创建一组传输层安全性 TLS 证书，以再次设置的其他阶段使用。在 openshift 系统中，首先使用作为运行的一部分创建的自生成证书:

```
# openshift start \
    --public-master=https://openshift.example.com:8443 \
    --write-config=/etc/origin/
```

除此之外，这会生成一个 `/etc/origin/master/ca.{cert|key}`。使用此签名证书来生成在验证代理上使用的密钥。

```
# mkdir -p /etc/origin/proxy/
# oc adm ca create-server-cert \
    --cert='/etc/origin/proxy/proxy.example.com.crt' \
    --key='/etc/origin/proxy/proxy.example.com.key' \
    --hostnames=proxy.example.com \
    --signer-cert=/etc/origin/master/ca.crt \
    --signer-key='/etc/origin/master/ca.key' \
    --signer-serial='/etc/origin/master/ca.serial.txt'
```

这里需要确保主机ip和端口，否则 https 连接将失败。

- 创建一个新的 CA 来签署此客户端证书

```
# oc adm ca create-signer-cert \
  --cert='/etc/origin/proxy/proxyca.crt' \
  --key='/etc/origin/proxy/proxyca.key' \
  --name='openshift-proxy-signer@UNIQUESTRING' \  # 使 UNIQUESTRING
  --serial='/etc/origin/proxy/proxyca.serial.txt'
```

- 生成认证代理将用于 openshift 证明其身份的 API 客户端证书

```
# oc adm create-api-client-config \
    --certificate-authority='/etc/origin/proxy/proxyca.crt' \
    --client-dir='/etc/origin/proxy' \
    --signer-cert='/etc/origin/proxy/proxyca.crt' \
    --signer-key='/etc/origin/proxy/proxyca.key' \
    --signer-serial='/etc/origin/proxy/proxyca.serial.txt' \
    --user='system:proxy'
``` 

可以防止恶意用户冒充代理并发送伪造身份

- 将证书和密钥信息复制到相应的文件以供将来使用

```
# cat /etc/origin/proxy/system\:proxy.crt \
      /etc/origin/proxy/system\:proxy.key \
      > /etc/origin/proxy/authproxy.pem
```

### 第二阶段:验证代理安装
#### 第一步 复制证书
从 openshift.example.com 安全的将必要的证书复制到代理机器

```
# scp /etc/origin/master/ca.crt \
      root@proxy.example.com:/etc/pki/CA/certs/

# scp /etc/origin/proxy/proxy.example.com.crt \
      /etc/origin/proxy/authproxy.pem \
      root@proxy.example.com:/etc/pki/tls/certs/

# scp /etc/origin/proxy/proxy.example.com.key \
      root@proxy.example.com:/etc/pki/tls/private/
```
#### 第二部 构建 SSSD
- 使用包含 1.12.0 或更高版本的操作系统安装 vm，以便使用 `mod_identity_lookup` 模块。
- 安装所有必要的依赖关系
	
	```
# yum install -y sssd \
                 sssd-dbus \
                 realmd \
                 httpd \
                 mod_session \
                 mod_ssl \
                 mod_lookup_identity \
                 mod_authnz_pam
```
	这为提供了所需的 SSSD 和 web 服务器组件

- 编辑 ` /etc/httpd/conf.modules.d/55-authnz_pam.conf` 文件并删除以下注解

		LoadModule authnz_pam_module modules/mod_authnz_pam.so
- 设置 SSSD 以对照 LDAP 服务器验证此 vm。如果 LDAP 服务器是 FreeIPA 或者服务目录环境，则可以使用 `realmd` 将本季连接到域。

		# realm join ldap.example.com
		
	更高需求可以参考[系统级身份验证指南](https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/7/html/System-Level_Authentication_Guide/authconfig-ldap.html)。如果要使用 SSSD 管理 LDAP 故障转移情况，可以通过 `ldap_uri` 的 ` /etc/sssd/sssd.conf` 中添加其他条目来配置此功能。使用 FreeIPA 注册系统可以使用 DNS SRV 记录自动处理故障转移。

- 重新启动 SSSD 以保证正确应用所有改变

		$ systemctl restart sssd.service
- 测试可以正确检索用户信息

		$ getent passwd <username>
		username:*:12345:12345:Example User:/home/username:/usr/bin/bash
- 尝试以 LDAP 用户身份登录虚拟机，并确认身份验证已正确设置。这可以通过本地控制台或者远程服务，ssh 来完成。

	如果不希望 ldap 用户能够登录计算机，建议修改 ` /etc/pam.d/system-auth` 和 `/etc/pam.d/password-auth`	以删除包含 `pam_sss.so` 的行即可

#### 第三步 apache 配置
需要设置 apache 以及 SSSD 进行通讯。创建一个供 apache 使用的 pam 堆栈文件。要做到这一点：

- 创建 `/etc/pam.d/openshift` 文件并添加以下内容，此配置使 PAM(可插拔认证模块)能够在为 openshift 堆栈发出认证请求时使用 pam_sss.so 来确定认证和访问控制。

```
auth required pam_sss.so
account required pam_sss.so
```

- 配置 apache httpd.conf 。步骤重点介绍如何设置质询验证，这对于使用 oc 登录和类似的自动化工具进行登录非常有用。

	配置基于表单的身份验证解释了如何使用 SSSD 设置图形登录，但它需要将此设置的其余部分作为先决条件。
- 在 `/etc/httpd/conf.d` 中创建新文件 `openshift-proxy.conf` ，在指示的地方替换正确的主机名。

```
LoadModule request_module modules/mod_request.so
LoadModule lookup_identity_module modules/mod_lookup_identity.so
# Nothing needs to be served over HTTP.  This virtual host simply redirects to
# HTTPS.
<VirtualHost *:80>
  DocumentRoot /var/www/html
  RewriteEngine              On
  RewriteRule     ^(.*)$     https://%{HTTP_HOST}$1 [R,L]
</VirtualHost>

<VirtualHost *:443>
  # This needs to match the certificates you generated.  See the CN and X509v3
  # Subject Alternative Name in the output of:
  # openssl x509 -text -in /etc/pki/tls/certs/proxy.example.com.crt
  ServerName proxy.example.com

  DocumentRoot /var/www/html
  SSLEngine on
  SSLCertificateFile /etc/pki/tls/certs/proxy.example.com.crt
  SSLCertificateKeyFile /etc/pki/tls/private/proxy.example.com.key
  SSLCACertificateFile /etc/pki/CA/certs/ca.crt

  # Send logs to a specific location to make them easier to find
  ErrorLog logs/proxy_error_log
  TransferLog logs/proxy_access_log
  LogLevel warn
  SSLProxyEngine on
  SSLProxyCACertificateFile /etc/pki/CA/certs/ca.crt
  # It's critical to enforce client certificates on the Master.  Otherwise
  # requests could spoof the X-Remote-User header by accessing the Master's
  # /oauth/authorize endpoint directly.
  SSLProxyMachineCertificateFile /etc/pki/tls/certs/authproxy.pem

  # Send all requests to the console
  RewriteEngine              On
  RewriteRule     ^/console(.*)$     https://%{HTTP_HOST}:8443/console$1 [R,L]

  # In order to using the challenging-proxy an X-Csrf-Token must be present.
  RewriteCond %{REQUEST_URI} ^/challenging-proxy
  RewriteCond %{HTTP:X-Csrf-Token} ^$ [NC]
  RewriteRule ^.* - [F,L]

  <Location /challenging-proxy/oauth/authorize>
    # Insert your backend server name/ip here.
    ProxyPass https://openshift.example.com:8443/oauth/authorize
    AuthType Basic
    AuthBasicProvider PAM
    AuthPAMService openshift
    Require valid-user
  </Location>

  <ProxyMatch /oauth/authorize>
    AuthName openshift
    RequestHeader set X-Remote-User %{REMOTE_USER}s env=REMOTE_USER
  </ProxyMatch>
</VirtualHost>

RequestHeader unset X-Remote-User
```
配置基于表单的身份验证解释了如何添加登录代理块以支持表单身份验证。

- 设置一个布尔值来告诉 SELinux apache 可以接受 PAM 子系统  

		# setsebool -P allow_httpd_mod_auth_pam on
- 启动 

		# systemctl start httpd.service

### 阶段3：openshift 配置
介绍了如何在 all in one 配置下从头开始设置 openshift 服务器。 master和node 节点配置提供了有关备用配置的更多信息。修改默认配置以使用刚创建的新身份提供程序。要做到这一点：

- 修改 `/etc/origin/master/master-config.yaml`
- 扫描它并找到 `identityProviders` 部分并将其替换

```
 identityProviders:
  - name: any_provider_name
    challenge: true
    login: false
    mappingMethod: claim
    provider:
      apiVersion: v1
      kind: RequestHeaderIdentityProvider
      challengeURL: "https://proxy.example.com/challenging-proxy/oauth/authorize?${query}"
      clientCA: /etc/origin/proxy/proxyca.crt
      headers:
      - X-Remote-User
```

[配置基于表单的身份验证](https://docs.openshift.org/3.6/install_config/advanced_ldap_configuration/configuring_form_based_authentication.html#configuring-form-based-authentication)解释了如何添加登录 URL 以及支持 web 登录

[配置增强 LDAP 属性](https://docs.openshift.org/3.6/install_config/advanced_ldap_configuration/configuring_extended_ldap_attributes.html#configuring-extended-ldap-attributes)说明如何添加电子邮件和全名属性。请注意，全名属性仅在第一次登录时存储到数据库中

- 更新配置

```
# openshift start \
    --public-master=https://openshift.example.com:8443 \
    --master-config=/etc/origin/master/master-config.yaml \
    --node-config=/etc/origin/node-node1.example.com/node-config.yaml
```			
			
- 测试登录

		oc login https://openshift.example.com:8443

## 配置基于表单的身份验证
主题介绍基于 SSSD 故障转移基础上，介绍了如何设置基于表单的身份验证以及登录到 openshift web 控制台。
### 准备登录页面
openshift 存储哭都具有表单模版。将其复制到 ` proxy.example.com` 上的身份验证代理，修改 .html 文件以更改需求的标志和内容。

	# curl -o /var/www/html/login.html \
    https://raw.githubusercontent.com/openshift/openshift-extras/master/misc/form_auth/login.html
### 安装另一个 apache 模块
要拦截基于表单的身份验证，需要安装 apache 模块

	 # yum -y install mod_intercept_form_submit
### 配置 apache
- 修改 `/etc/httpd/conf.modules.d/55-intercept_form_submit.conf` 并取消注释 `LoadModule` 行。
- 在 `<VirtualHost *:443>` 块内的 `openshift-proxy.conf` 文件中新添加一个新节点
	
```
 <Location /login-proxy/oauth/authorize>
  # Insert your backend server name/ip here.
  ProxyPass https://openshift.example.com:8443/oauth/authorize

  InterceptFormPAMService openshift
  InterceptFormLogin httpd_username
  InterceptFormPassword httpd_password

  RewriteCond %{REQUEST_METHOD} GET
  RewriteRule ^.*$ /login.html [L]
</Location>
```	
这告诉 apache 监听 ` /login-proxy/oauth/authorize` 上的 POST 请求，并将用户名和密码传递给 openshift PAM 服务。

- 重新启动服务并返回到 openshift 配置

### 配置 openshift 
- 在 `master-config.yaml` 更新 `identityProviders`

```
identityProviders:
- name: any_provider_name
  challenge: true
  login: true  #这里设置为 true
  mappingMethod: claim
  provider:
    apiVersion: v1
    kind: RequestHeaderIdentityProvider
    challengeURL: "https://proxy.example.com/challenging-proxy/oauth/authorize?${query}"
    loginURL: "https://proxy.example.com/login-proxy/oauth/authorize?${query}" #新增行
    clientCA: /etc/origin/master/proxy/proxyca.crt
    headers:
    - X-Remote-User
```

- 重启 openshift 更新配置

## 配置增强的 LDAP 属性
本主题基础是设置 SSSD 后，并着重于配置扩展 LDAP 属性
### 先决条件
- SSSD 1.12.0 或更高版本
- `mod_lookup_identity 0.9.4` 或更高版本

### 配置 SSSD
需要请求系统安全服务守护程序 SSSD 在 LDAP 中查找通常不关心登录用例属性。在 openshift 的情况下，只有一个这样的属性:电子邮件。所以需要

- 修改代理上  `/etc/sssd/sssd.conf` 的 ` [domain/DOMAINNAME]` 部分并添加以下属性

```
[domain/example.com]
...
ldap_user_extra_attrs = mail
```

- 告诉 SSSD ，这个属性可以被 apache 检索出来是可以接受的。将以下两行添加到 `/etc/sssd/sssd.conf` 的 ifp 部分

```
[ifp]
user_attributes = +mail
allowed_uids = apache, root
```

- 重启

		# systemctl restart sssd.service
- 测试

### 配置 apache
现在 SSSD 已经设置并成功提供扩展属性，请将 web 服务器配置为请求并将他们插入到正确的位置。

- 使用模块能够被 apache 加载。为此，请修改 `/etc/httpd/conf.modules.d/55-lookup_identity.conf` 并取消注释

		LoadModule lookup_identity_module modules/mod_lookup_identity.so
- 设置一个 SELinunx 不二之，以便 SElinux 允许 apache 通过 D-BUS 连接到 SSSD

		# setsebool -P httpd_dbus_sssd on
- 编辑 ` /etc/httpd/conf.d/openshift-proxy.conf` 并在 `<ProxyMatch /oauth/authorize>` 部分增加以下内容

```
<ProxyMatch /oauth/authorize>
  AuthName openshift

  LookupOutput Headers # 新增
  LookupUserAttr mail X-Remote-User-Email # 新增
  LookupUserGECOS X-Remote-User-Display-Name # 新增

  RequestHeader set X-Remote-User %{REMOTE_USER}s env=REMOTE_USER
</ProxyMatch>
```				

- 重启

		# systemctl restart httpd.service

### 配置 openshift 
告诉 openshift 在登录过程中，在哪里可以找到这些新属性。要做到这一点

- 编辑 `/etc/origin/master/master-config.yaml` 文件并增加内容到 `identityProviders`

```
identityProviders:
 - name: sssd
 challenge: true
 login: true
 mappingMethod: claim
 provider:
   apiVersion: v1
   kind: RequestHeaderIdentityProvider
   challengeURL: "https://proxy.example.com/challenging-proxy/oauth/authorize?${query}"
   loginURL: "https://proxy.example.com/login-proxy/oauth/authorize?${query}"
   clientCA: /home/example/workspace/openshift/configs/openshift.example.com/proxy/proxyca.crt
   headers:
   - X-Remote-User
   emailHeaders: # 添加的部分
   - X-Remote-User-Email # 添加的部分
   nameHeaders: # 添加的部分
   - X-Remote-User-Display-Name # 添加的部分
```	

- 重新启动 openshift ,并以新用户身份登录

	可以看到他们全名出现在屏幕右上角。还可以使用 oc 获取身份验证，可以得到 邮件地址和全名。
- 调试

	目前， openshift 仅在第一次登录时将这些属性保存到用户，并且在此之后不在更新它们。因此，当正在测试时，运行 ` oc delete users,identities --all` 清除用户标识，以便可以再次登录。 			