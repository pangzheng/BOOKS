# 安装与配置-使用 LDAP 同步组
作为集群管理员，可以使用组来管理用户，更改其权限并增强协作。用户组织可能已经创建了用户组并将其存储在 LDAP 服务器中。 openshift 可以将这些 LDAP 记录与内部的记录同步，使用户能够在一个地方管理组。 openshift 目前支持使用三种常规模式定义组成员的 LDAP 服务器的同步。`RFC 2307, AD(活动目录),增强活动目录`

注意，必须具有管理员(cluster_admin)权限才能同步群组。
## 设置 LDAP 同步
在[运行 LDAP 同步](https://docs.openshift.org/3.6/install_config/syncing_groups_with_ldap.html#running-ldap-sync)之前，需要一个同步配置文件。该文件包含 LDAP 客户端配置详细信息:

- 用于连接到 LDAP 服务器的配置
- 同步取决于 LDAP 服务器使用的架构的配置选项
- 同步配置文件还包含管理员定义的名称映射列表，这些列表将 openshift group 名称映射到 LDAP 的组。

### LDAP 客户端配置
```
url: ldap://10.0.0.0:389 #连接协议，托管数据库的 LDAP 服务器的IP地址以及端口，格式为  scheme://host:port
bindDN: cn=admin,dc=example,dc=com #绑定 DN 的可选专有名称(DN).如果需要提升特权才能检索同步操作的条目，则 openshift 会使用此选项。
bindPassword: password #用于绑定的可选密码。如果需要提升特权才能检索同步操作的条目，则 open shift 会使用此选项。该值也可以在环境变量，外部文件或加密文件中提供。
insecure: false #如果该值为true，则不会向服务器进行 TLS 连接。如果是 false，则使用 TLS 连接安全的 LDAP (ldaps://) URLs，并将不安全的 (ldap://) URL 升级为 TLS
ca: my-ldap-ca-bundle.crt #证书包用于验证配置的 URL 的服务器证书。如果为空，则 openshift 使用系统信任的根。这仅适用于不安全设置为 false的情况。
```
### LDAP 查询定义
同步配置由用于同步所需条目的 LDAP 查询定义组成。 LDAP 查询的具体定义取决于用于在 LDAP 服务器中存储成员资格信息的模式。

```
baseDN: ou=users,dc=example,dc=com # 所有搜索将从其开始的目录分支的可变名称(DN).需要指定目录树的顶部，但也可以在目录中指定一个子树。
scope: sub # 搜索范围，有效值为  base, one, sub.如果这是未定义的，那么假定一个子范围。范围选项的说明可以在下表中找到。
derefAliases: never #搜索相对于 LDAP 树中别名的行为，never, search, base, always为有效值。如果这是未定义的，那么默认总是取消引用别名。解释引用行为的描述可在下表中找到。
timeout: 0 #客户搜索允许的时间限制，秒为单位。值为0时不会产生客户端限制。
filter: (objectClass=inetOrgPerson) #有效的 LDAP 搜索过滤器。如果这是未定义的，那么默认值是  (objectClass=*)
pageSize: 0 #在 LDAP 条目中测量的服务器响应页最大可选大小。如果设置为0，则不会在响应页面上进行大小限制。当查询返回的条目超过客户端或服务器默认允许的情况是，则分页大小是必须的。
```

- LDAP 搜索范围选项

LDAP 搜索范围|描述
---|---
base|只考虑为查询指定的基准 DN 指定的对象
one|考虑树中的同一级别上所有对象作为查询的基本 DN。
sub|考虑以给定查询的基础 DN 为根的整个子树。

- 引用行为

引用行为|描述
---|---
never|切勿取消引用 LDAP 树中的任何别名
search|搜索时只能找到取消引用别名
base|只有在找到基础对象时才引用别名
always|始终取消引用 LDAP 树中的所有别名

### 用户定义的名称映射
用户定义的名称映射明确的将 openshift 的名称映射到在 LDAP 服务器上查找组的唯一标识符。该映射使用正常的 yaml 语法。用户定义的映射可以包含 LDAP 服务器中每个组的条目或包含这些组的子集。如果 LDAP 服务器上的组没有用户定义的名称映射，则同步期间的默认行为时使用指定的组名称的属性。

用户定义名称映射

```
groupUIDNameMapping:
  "cn=group1,ou=groups,dc=example,dc=com": firstgroup
  "cn=group2,ou=groups,dc=example,dc=com": secondgroup
  "cn=group3,ou=groups,dc=example,dc=com": thirdgroup
```
## 运行 LDAP 同步
一旦创建了同步配置文件，就可以开始同步。 openshift 允许管理员使用同一台服务器执行多种不同的同步类型。

默认情况下，所有组同步或修剪操作都是空运行的，因此必须在 `sync-groups` 命令上设置 `--confirm` 标识以更改 openshift 组记录。

使用 openshift 同步来自 LDAP 服务器的所有组:

	$ oc adm groups sync --sync-config=config.yaml --confirm
要同步已在 openshift 中的所有组，这些组对应配置文件中指定的 LDAP 服务器中的组

	$ oc adm groups sync --type=openshift --sync-config=config.yaml --confirm
要使用 openshift 同步 LDAP 组的子集，可以使用白名单文件，黑名单文件或两者:黑白名单文件或文字的任何组合都可以使用；白名单文字可以直接包含在命令中。这适用于在 LDAP 服务器上找到的组，以及已存在于 openshift 中的组。文件每行必须包含一个唯一的组标识符。

```
$ oc adm groups sync --whitelist=<whitelist_file> \
                   --sync-config=config.yaml    \
                   --confirm
$ oc adm groups sync --blacklist=<blacklist_file> \
                   --sync-config=config.yaml    \
                   --confirm
$ oc adm groups sync <group_unique_identifier>    \
                   --sync-config=config.yaml    \
                   --confirm
$ oc adm groups sync <group_unique_identifier>    \
                   --whitelist=<whitelist_file> \
                   --blacklist=<blacklist_file> \
                   --sync-config=config.yaml    \
                   --confirm
$ oc adm groups sync --type=openshift             \
                   --whitelist=<whitelist_file> \
                   --sync-config=config.yaml    \
                   --confirm
```		
## 运行组修剪作业
如果创建他们的 LDAP 服务器上的记录不再存在，管理员也可以选择从 openshift 记录中删除组。修剪作业将接受用于同步作业的同一配置文件和白名单或黑名单。

例如，如果组以前使用某些 `config.yaml` 文件从 LDAP 进行了同步，并且其中一些组不再存在于 LDAP 服务器上，则以下命令将确定 openshift 中的哪些组对应于 LDAP 中的已删除组，然后从 openshift 中删除它们来源。

	$ oc adm groups prune --sync-config=config.yaml --confirm
## 同步例子
本节包含 `RFC 2307, Active Directory, 增强 Active Directory`架构的例子。以下所有例子都同步一个名为 admins 的组，其中两个成员:jane和jim。每个例子说明:

- 如何将组和用户添加到 LDAP 服务器中。
- LDAP 同步配置文件的外观
- 同步之后， openshift 中生成的组记录是什么？

这些示范假设所有用户都是其各自组的直接成员。具体来说，没有组别有其他组成员。有关如何同步且套组的信息，请参考[同步嵌套成员](https://docs.openshift.org/3.6/install_config/syncing_groups_with_ldap.html#sync-ldap-nested-example)
### RFC 2307
在 RFC 2307 模式中，用户(Jane 和 Jim)和组在 LDAP 服务器上作为一级条目存在，并且组成员身份存储在组的属性中。下面的 idif 片段为这个模式定义了用户和组:

```
  dn: ou=users,dc=example,dc=com
  objectClass: organizationalUnit
  ou: users

  dn: cn=Jane,ou=users,dc=example,dc=com
  objectClass: person
  objectClass: organizationalPerson
  objectClass: inetOrgPerson
  cn: Jane
  sn: Smith
  displayName: Jane Smith
  mail: jane.smith@example.com

  dn: cn=Jim,ou=users,dc=example,dc=com
  objectClass: person
  objectClass: organizationalPerson
  objectClass: inetOrgPerson
  cn: Jim
  sn: Adams
  displayName: Jim Adams
  mail: jim.adams@example.com

  dn: ou=groups,dc=example,dc=com
  objectClass: organizationalUnit
  ou: groups

  dn: cn=admins,ou=groups,dc=example,dc=com  #该组是 LDAP 服务器中的一类条目
  objectClass: groupOfNames
  cn: admins
  owner: cn=admin,dc=example,dc=com
  description: System Administrators
  member: cn=Jane,ou=users,dc=example,dc=com #一个组的成员累出了一个识别参考，作为该组的属性。
  member: cn=Jim,ou=users,dc=example,dc=com
```	
要同步此组，必须先创建配置文件。 RFC 2307 模式要求为用户和组提供 LDAP 查询定义，以及用于在内部 openshift 记录中表示它们的属性。

为了清楚起见，在 openshift 中创建的组应尽可能使用专有名称以外的属性，以用于面向用户或管理员的字段。例如，通过电子邮件识别组的用户，并将该组的名称作通用名称。以下配置文件创建这些关系:

	如果使用用户定义的名称映射，则配置文件将有所不同

同步配置文件 rfc2307_config.yaml
	
```
kind: LDAPSyncConfig
apiVersion: v1
url: ldap://LDAP_SERVICE_IP:389 #存储该组记录的LDAP 服务器的ip 地址和主机。
insecure: false #如果为 true，则不会向服务器进行 TLS 连接。如果 false，则使用 TLS 安全连接，如果是不安全的连接，也会升级成 TLS。
rfc2307:
    groupsQuery:
        baseDN: "ou=groups,dc=example,dc=com"
        scope: sub
        derefAliases: never
        pageSize: 0
    groupUIDAttribute: dn # 唯一标识 LDAP 服务器上的组的属性。使用 DN 作为 groupUIDAttribute 时，无法指定 groupsQuery 过滤器。对于细颗粒度过滤，请使用黑白名单。
    groupNameAttributes: [ cn ] # 用作组的名称属性
    groupMembershipAttributes: [ member ] #存储成员信息的组上的属性。
    usersQuery:
        baseDN: "ou=users,dc=example,dc=com"
        scope: sub
        derefAliases: never
        pageSize: 0
    userUIDAttribute: dn # 唯一标识 LDAP 服务器上的用户的属性。在 userUIDAttribute 使用 DN 时，不能指定 usersQuery 过滤器。对于细颗粒度过滤，请使用黑白名单。
    userNameAttributes: [ mail ] #用作 openshift 原始组记录中用户名称的属性。
    tolerateMemberNotFoundErrors: false
    tolerateMemberOutOfScopeErrors: false
```	
运行同步文件

	$ oc adm groups sync --sync-config=rfc2307_config.yaml --confirm
同步结果

```
apiVersion: v1
kind: Group
metadata:
  annotations:
    openshift.io/ldap.sync-time: 2015-10-13T10:08:38-0400 #上一次该组同步时间，采用 ISO 6801 格式
    openshift.io/ldap.uid: cn=admins,ou=groups,dc=example,dc=com # LDAP 服务器傻姑娘组的唯一标识符
    openshift.io/ldap.url: LDAP_SERVER_IP:389 #存储该组记录的 LDAP 服务器地址和端口
  creationTimestamp:
  name: admins #由同步文件制定的组的名称
users: # 作为该组成员的用户，按照同步文件制定的名称命名
- jane.smith@example.com
- jim.adams@example.com
```
#### RFC2307 使用用户定义的名称映射
将组与用户定义的名称映射进行同步，配置文件将更改为包含这些映射，如例:使用带用户定义名称映射的 RFC2307 架构的 LDAP 同步配置: `rfc2307_config_user_defined.yaml`

```
kind: LDAPSyncConfig
apiVersion: v1
groupUIDNameMapping:
  "cn=admins,ou=groups,dc=example,dc=com": Administrators #用户定义的名称映射
rfc2307:
    groupsQuery:
        baseDN: "ou=groups,dc=example,dc=com"
        scope: sub
        derefAliases: never
        pageSize: 0
    groupUIDAttribute: dn # 用于用户定义的名称映射中的键的唯一标识属性。使用 DN 作为 groupUIDAttribute 时，无法指定 groupsQuery 过滤器。对于细颗粒度过滤需要使用黑白名单。
    groupNameAttributes: [ cn ] # 如果 openshift 组的唯一标识不再用户定义的名称映射中，则该属性命名为 openshift 组。
    groupMembershipAttributes: [ member ]
    usersQuery:
        baseDN: "ou=users,dc=example,dc=com"
        scope: sub
        derefAliases: never
        pageSize: 0
    userUIDAttribute: dn #唯一标识 LDAP 服务器的用户的属性。在为 userUIDAttribute 使用 DN 时，不能指定 usersQuery 过滤器。对于细颗粒度过滤需要使用黑白名单。
    userNameAttributes: [ mail ]
    tolerateMemberNotFoundErrors: false
    tolerateMemberOutOfScopeErrors: false
```

同步方法

	$ oc adm groups sync --sync-config=rfc2307_config_user_defined.yaml --confirm
同步结果

```
apiVersion: v1
kind: Group
metadata:
  annotations:
    openshift.io/ldap.sync-time: 2015-10-13T10:08:38-0400
    openshift.io/ldap.uid: cn=admins,ou=groups,dc=example,dc=com
    openshift.io/ldap.url: LDAP_SERVER_IP:389
  creationTimestamp:
  name: Administrators #由用户定义的名称映射指定的组的名称
users:
- jane.smith@example.com
- jim.adams@example.com
```	
### 具有用户定义的错误容错的 RFC 2307
默认情况下，如果正在同步的组包含其他条目在成员查询中定义的范围之外的成员，则组同步会失败并显示错误。

	Error determining LDAP group membership for "<group>": membership lookup for user "<user>" in group "<group>" failed because of "search for entry with dn="<user-dn>" would search outside of the base dn specified (dn="<base-dn>")".
这通常表明 `usersQuery` 字段中的配置错误 `baseDN`。但是，如果 `baseDN` 有意不包含组中的某些成员，则设置 `tolerateMemberOutOfScopeErrors：true` 将允许组同步继续。超出范围的成员将被忽略。

同样，如果组同步过程中无法找到某个组的成员，也会出错

	Error determining LDAP group membership for "<group>": membership lookup for user "<user>" in group "<group>" failed because of "search for entry with base dn="<user-dn>" refers to a non-existent entry".

	Error determining LDAP group membership for "<group>": membership lookup for user "<user>" in group "<group>" failed because of "search for entry with base dn="<user-dn>" and filter "<filter>" did not return any results".
这通畅表示错误配置的 `usersQuery` 字段。但是，如果组中包含已知却是的成员条目，则设置 `tolerateMemberNotFoundErrors: true` 将允许组同步继续。有问题的成员将被忽略。

为了 LDAP 组同步启用容错会导致过程中忽略有问题的成员条目。如果 LDAP 组同步配置不正确，则可能会导致 openshift 组成员缺少。

使用带有问题组成员资格的 RFC 2307 架构的 LDAP 条目: `rfc2307_problematic_users.ldif`

```
  dn: ou=users,dc=example,dc=com
  objectClass: organizationalUnit
  ou: users

  dn: cn=Jane,ou=users,dc=example,dc=com
  objectClass: person
  objectClass: organizationalPerson
  objectClass: inetOrgPerson
  cn: Jane
  sn: Smith
  displayName: Jane Smith
  mail: jane.smith@example.com

  dn: cn=Jim,ou=users,dc=example,dc=com
  objectClass: person
  objectClass: organizationalPerson
  objectClass: inetOrgPerson
  cn: Jim
  sn: Adams
  displayName: Jim Adams
  mail: jim.adams@example.com

  dn: ou=groups,dc=example,dc=com
  objectClass: organizationalUnit
  ou: groups

  dn: cn=admins,ou=groups,dc=example,dc=com
  objectClass: groupOfNames
  cn: admins
  owner: cn=admin,dc=example,dc=com
  description: System Administrators
  member: cn=Jane,ou=users,dc=example,dc=com
  member: cn=Jim,ou=users,dc=example,dc=com
  member: cn=INVALID,ou=users,dc=example,dc=com # LDAP 服务器上不存在的成员
  member: cn=Jim,ou=OUTOFSCOPE,dc=example,dc=com #可能存在的成员，但不在用户查询同步作业下的 baseDN
```
为了容忍上述例子中的错误，必须对同步配置文件进行以下添加:例子使用 RFC 2307 架构容忍错误的 LDAP 同步配置: `rfc2307_config_tolerating.yaml`

```
kind: LDAPSyncConfig
apiVersion: v1
url: ldap://LDAP_SERVICE_IP:389
rfc2307:
    groupsQuery:
        baseDN: "ou=groups,dc=example,dc=com"
        scope: sub
        derefAliases: never
    groupUIDAttribute: dn
    groupNameAttributes: [ cn ]
    groupMembershipAttributes: [ member ]
    usersQuery:
        baseDN: "ou=users,dc=example,dc=com"
        scope: sub
        derefAliases: never
    userUIDAttribute: dn #唯一标识 LDAP 服务器傻姑娘的用户属性。在为 userUIDAttribute 使用 DN 时，不能指定 usersQuery 过滤器。对于细颗粒度过滤需要使用黑白名单。
    userNameAttributes: [ mail ]
    tolerateMemberNotFoundErrors: true #如果设置为 true，则同步作业将容忍未找到某些成员的组。
    tolerateMemberOutOfScopeErrors: true #如果为 true,则同步作业将容忍某些成员位于 usersQuery 基本 DN 中给定的用户范围之外的组，并且会忽略它们。
```
同步

	$ oc adm groups sync --sync-config=rfc2307_config_tolerating.yaml --confirm
同步结果

```
apiVersion: v1
kind: Group
metadata:
  annotations:
    openshift.io/ldap.sync-time: 2015-10-13T10:08:38-0400
    openshift.io/ldap.uid: cn=admins,ou=groups,dc=example,dc=com
    openshift.io/ldap.url: LDAP_SERVER_IP:389
  creationTimestamp:
  name: admins
users: #这样就不会报错了
- jane.smith@example.com
- jim.adams@example.com
```
### 活动目录
在活动目录架构中，两个用户作为一级条目存在于 LDAP 服务器中，并且组成员资格存储在用户的属性中。下面的 idif 片段为这个模式定义了用户和组。使用活动目录的 LDAP 条目: `active_directory.ldif`

```
dn: ou=users,dc=example,dc=com
objectClass: organizationalUnit
ou: users

dn: cn=Jane,ou=users,dc=example,dc=com
objectClass: person
objectClass: organizationalPerson
objectClass: inetOrgPerson
objectClass: testPerson
cn: Jane
sn: Smith
displayName: Jane Smith
mail: jane.smith@example.com
memberOf: admins #用户的组成员身份被列为用户属性，并且该组不存在作为服务器上的条目。memberOf 属性不一定是用户的文字属性，某些 LDAP 服务器中，它在搜索过程中创建并返回给客户端，但为提交数据库。

dn: cn=Jim,ou=users,dc=example,dc=com
objectClass: person
objectClass: organizationalPerson
objectClass: inetOrgPerson
objectClass: testPerson
cn: Jim
sn: Adams
displayName: Jim Adams
mail: jim.adams@example.com
memberOf: admins
```
要同步此组，必须先创建配置文件。活动目录架构要求您为用户条目提供 LDAP 查询定义，以及用于在内部 openshift group 记录中表示它们的属性。

为了清楚起见，在 openshift 中创建的组应该尽可能使用专有名称以外的属性，以用于面向用户或者管理员的字段。例如通过电子邮件标识组的用户，但通过 LDAP 服务器傻姑娘组的名称定义组的名称。以下配置文件创建这些关系: 例 使用活动目录架构的 LDAP 同步配置: `active_directory_config.yaml`

```
kind: LDAPSyncConfig
apiVersion: v1
url: ldap://LDAP_SERVICE_IP:389
activeDirectory:
    usersQuery:
        baseDN: "ou=users,dc=example,dc=com"
        scope: sub
        derefAliases: never
        filter: (objectclass=inetOrgPerson)
        pageSize: 0
    userNameAttributes: [ mail ] #作为openshift 原始组记录中用户的名称的属性
    groupMembershipAttributes: [ memberOf ] #存储会员信息的属性
```
同步

	$ oc adm groups sync --sync-config=active_directory_config.yaml --confirm
同步结果

```
apiVersion: v1
kind: Group
metadata:
  annotations:
    openshift.io/ldap.sync-time: 2015-10-13T10:08:38-0400 #同步时间
    openshift.io/ldap.uid: admins # LDAP 服务器组的唯一标识
    openshift.io/ldap.url: LDAP_SERVER_IP:389 # 存储该组记录 LDAP 服务器信息
  creationTimestamp:
  name: admins # openshift 组名称
users: #成员名称
- jane.smith@example.com
- jim.adams@example.com
```	
### 增强的活动目录
在增强的活动目录架构中，用户和组在 LDAP 服务器中作为一级条目存在，并且组成员资格存储在用户的属性中；下面的 idif 片段为这个模式定义了用户和组，使用增强活动目录架构的 LDAP 条目: `augmented_active_directory.ldif`

```
dn: ou=users,dc=example,dc=com
objectClass: organizationalUnit
ou: users

dn: cn=Jane,ou=users,dc=example,dc=com
objectClass: person
objectClass: organizationalPerson
objectClass: inetOrgPerson
objectClass: testPerson
cn: Jane
sn: Smith
displayName: Jane Smith
mail: jane.smith@example.com
memberOf: cn=admins,ou=groups,dc=example,dc=com #用户的组成员身份被列为用户属性

dn: cn=Jim,ou=users,dc=example,dc=com
objectClass: person
objectClass: organizationalPerson
objectClass: inetOrgPerson
objectClass: testPerson
cn: Jim
sn: Adams
displayName: Jim Adams
mail: jim.adams@example.com
memberOf: cn=admins,ou=groups,dc=example,dc=com

dn: ou=groups,dc=example,dc=com
objectClass: organizationalUnit
ou: groups

dn: cn=admins,ou=groups,dc=example,dc=com #该组是 LDAP 服务器上的一级条目
objectClass: groupOfNames
cn: admins
owner: cn=admin,dc=example,dc=com
description: System Administrators
member: cn=Jane,ou=users,dc=example,dc=com
member: cn=Jim,ou=users,dc=example,dc=com
```
要同步此组，必须先创建配置文件。增强的活动目录架构要求为用户条目和组条目提供 LDAP 查询定义，以及用于在内部 openshift 组记录中表示它们的属性。

为了清楚起见，在 openshift 中创建的组应该尽可能使用专用名称以外的属性，以用于面向用户或管理员字段。例如电子邮件，并将组的名称用作通用名称。以下配置创建了这些。例子使用增强活动目录架构进行 LDAP 同步配置: `augmented_active_directory_config.yaml
`

```
kind: LDAPSyncConfig
apiVersion: v1
url: ldap://LDAP_SERVICE_IP:389
augmentedActiveDirectory:
    groupsQuery:
        baseDN: "ou=groups,dc=example,dc=com"
        scope: sub
        derefAliases: never
        pageSize: 0
    groupUIDAttribute: dn 唯一标识 LDAP 服务器上组的属性。使用 DN 作为 groupUIDAttribute时，不能指定 usersQuery 过滤器。对于细颗粒度过滤需要使用黑白名单。
    groupNameAttributes: [ cn ] #作用组的名称属性
    usersQuery:
        baseDN: "ou=users,dc=example,dc=com"
        scope: sub
        derefAliases: never
        pageSize: 0
    userNameAttributes: [ mail ] #作用 openshift 原始组记录中的用户名称的属性。
    groupMembershipAttributes: [ memberOf ] #存储会员信息的用户属性。
```
同步

	$ oc adm groups sync --sync-config=augmented_active_directory_config.yaml --confirm
结果

```
apiVersion: v1
kind: Group
metadata:
  annotations:
    openshift.io/ldap.sync-time: 2015-10-13T10:08:38-0400 
    openshift.io/ldap.uid: cn=admins,ou=groups,dc=example,dc=com 
    openshift.io/ldap.url: LDAP_SERVER_IP:389 
  creationTimestamp:
  name: admins 
users: 
- jane.smith@example.com
- jim.adams@example.com
```
## 嵌套成员同步示例
openshift 中的组不会嵌套。在可以使用数据之前，LDAP 必须将组成员身份扁平化。微软的活动目录通过 `LDAP_MATCHING_RULE_IN_CHAIN` 规则支持此功能，该规则有 `OID 1.2.840.113556.1.4.1941` 。此外，使用此匹配规则时，只有明确列入白名单的组才能同步。

本节举例了增强活动目录架构，该架构同步一个名为 admins 的组，该组具有一个用户 jane 和一个组用户作为成员。 otheradmins 小组有一名用户成员: JIM. 这个例子解释了:

- 如何将组和用户添加到 LDAP 服务器
- LDAP 同步配置文件外观
- 同步后， openshift 生成的组记录会时什么？

在增强的活动目录架构中，用户和组都作为一级条目存在于 LDAP ，并且组成员资格存储在用户或组的属性中。下面 idif 片段为了这个模式定义了用户和组:`augmented_active_directory_nested.ldif`

```
dn: ou=users,dc=example,dc=com
objectClass: organizationalUnit
ou: users

dn: cn=Jane,ou=users,dc=example,dc=com
objectClass: person
objectClass: organizationalPerson
objectClass: inetOrgPerson
objectClass: testPerson
cn: Jane
sn: Smith
displayName: Jane Smith
mail: jane.smith@example.com
memberOf: cn=admins,ou=groups,dc=example,dc=com #用户和组成员资格对象中列为属性。

dn: cn=Jim,ou=users,dc=example,dc=com
objectClass: person
objectClass: organizationalPerson
objectClass: inetOrgPerson
objectClass: testPerson
cn: Jim
sn: Adams
displayName: Jim Adams
mail: jim.adams@example.com
memberOf: cn=otheradmins,ou=groups,dc=example,dc=com #用户和组成员资格对象中列为属性。

dn: ou=groups,dc=example,dc=com
objectClass: organizationalUnit
ou: groups

dn: cn=admins,ou=groups,dc=example,dc=com #这些组是 LDAP 服务器上的一级条目
objectClass: group
cn: admins
owner: cn=admin,dc=example,dc=com
description: System Administrators
member: cn=Jane,ou=users,dc=example,dc=com
member: cn=otheradmins,ou=groups,dc=example,dc=com # otheradmins 组是 admin 的组成员

dn: cn=otheradmins,ou=groups,dc=example,dc=com # 这些组是 LDAP 服务器上的一级条目
objectClass: group
cn: otheradmins
owner: cn=admin,dc=example,dc=com
description: Other System Administrators
memberOf: cn=admins,ou=groups,dc=example,dc=com   #用户和组成员资格对象中列为属性。
member: cn=Jim,ou=users,dc=example,dc=com
```
要将嵌套组与活动目录同步，必须为用户条目和组条目提供 LDAP 查询定义，以及用于在内部 openshift 组记录中表示它们的属性。此外，此配置与奥进行某些更改:

- `oc adm groups sync` 命令必须明确白名单组
- 用户的 `groupMembershipAttributes` 必须包含 `memberOf:1.2.840.113556.1.4.1941:` 以符合 `LDAP_MATCHING_RULE_IN_CHAIN` 规则。
- `groupUIDAttribute` 必须设置为 dn
- `groupsQuery`
	- 不得设置过滤器
	- 必须设置有效的 `derefAliases`
	- 不应该将 `baseDN` 设置为该值被忽略
	- 不应该将范围设置为该值被忽略

为了清楚起见，在 openshift 中创建的组应尽可能使用专有名称以外的属性，以用于面向用户或管理员的字段。例如,通过电子邮件。配置关系如下: `augmented_active_directory_config_nested.yaml`

```
kind: LDAPSyncConfig
apiVersion: v1
url: ldap://LDAP_SERVICE_IP:389
augmentedActiveDirectory:
    groupsQuery: # 不能指定 groupsQuery 过滤器。groupsQuery 基本 dn和范围值将被忽略。groupsQuery必须设置有效的 derefAliases。
        derefAliases: never
        pageSize: 0
    groupUIDAttribute: dn # 唯一标识 LDAP 服务器上的组的属性。它必须设置为 dn
    groupNameAttributes: [ cn ] # 用作组的名称的属性
    usersQuery:
        baseDN: "ou=users,dc=example,dc=com"
        scope: sub
        derefAliases: never
        filter: (objectclass=inetOrgPerson)
        pageSize: 0
    userNameAttributes: [ mail ] #openshift 原始组记录中用户名称的属性。mail 或 sAMAccountName 是大多数首选。
    groupMembershipAttributes: [ "memberOf:1.2.840.113556.1.4.1941:" ] #存储会员信息的用户属性。倾注于使用 LDAP_MATCHING_RULE_IN_CHAIN 
```
同步

```
$ oc adm groups sync \
    'cn=admins,ou=groups,dc=example,dc=com' \ # 必须将这些信息列入白名单
    --sync-config=augmented_active_directory_config_nested.yaml \
    --confirm
``` 
openshift 的组信息

```
apiVersion: v1
kind: Group
metadata:
  annotations:
    openshift.io/ldap.sync-time: 2015-10-13T10:08:38-0400 #上次同步时间
    openshift.io/ldap.uid: cn=admins,ou=groups,dc=example,dc=com # ldap 服务器上组的唯一标识符。 
    openshift.io/ldap.url: LDAP_SERVER_IP:389 #ldap 服务器信息
  creationTimestamp:
  name: admins #同步的组名
users: #因为是扁平化关系，所以这里也包含嵌套组成员信息。
- jane.smith@example.com
- jim.adams@example.com
```
## LDAP 同步配置范围
下面是配置文件的对象范围。请注意，不同的模式对象具有不同的字段。例如，[v1.ActiveDirectoryConfig](https://docs.openshift.org/3.6/install_config/syncing_groups_with_ldap.html#sync-ldap-v1-activedirectoryconfig)没有 `groupsQuery` 字段，而 `v1.RFC2307Config` 和 `v1.AugmentedActiveDirectoryConfig` 都有。

不支持二进制属性。来自 LDAP 服务器的所属属性数据必须采用 utf8 编码字符串格式。例如，绝对不要使用二进制属性，如(objectGUID) 作为 ID 属性。必须改用字符串属性，例如 ` sAMAccountName` 或 `userPrincipalName`
### v1.LDAPSyncConfig
`LDAPSyncConfig` 包含必要的配置选项以定义 LDAP 组同步

名称|描述|Schema
---|---|---
kind|表示此对象所代表的REST资源的字符串值。服务器可以从客户端向其提交请求的断点推断出这一点。无法更新。在 [CamelCase](.https://github.com/kubernetes/community/blob/master/contributors/devel/api-conventions.md#types-kinds)|字符串
apiVersion|定义对象的这种表示的版本化模式。服务器应将已识别的模式转换成最新的内部值，并可能拒绝未识别的值。[更多信息](https://github.com/kubernetes/community/blob/master/contributors/devel/api-conventions.md#resources)|字符串
url|主机是要连接的 ldap 服务器方案，主机和端口 `scheme://host:port`|字符串
bindDN|可选的 dn 使用绑定到 LDAP 服务器|字符串
bindPassword|在搜索阶段绑定的可选密码|[v1.StringSource](https://docs.openshift.org/3.6/install_config/syncing_groups_with_ldap.html#sync-ldap-v1-stringsource)
insecure|如果为 true，则表示连接不应使用 TLS。要使用 ldap url方案 ldaps:// 就不能设置为 true。ldaps:// URLs 使用 TLS 连接，而  ldap:// URL 使用 `https://tools.ietf.org/html/rfc2830` 中指定的 StartTLS 升级到 TLS 连接。| boolean
groupUIDNameMapping|可选的将 LDAP 组UIDs 直接映射到 openshift group 名称| object
rfc2307|保存从LDAP 服务器提取数据的配置，其设置方式与 RFC 2307 类似:一级组和用户条目，组成员资格由列组成员的多属性确定。|[v1.RFC2307Config](https://docs.openshift.org/3.6/install_config/syncing_groups_with_ldap.html#sync-ldap-v1-rfc2307config)
activeDirectory|保存配置，以便类似于活动目录中使用的方式设置 LDAP 服务器提取数据:一级的用户条目，其组成员资格由成员列表组中的多值属性确定，他们是成员列表组。| [v1.ActiveDirectoryConfig](https://docs.openshift.org/3.6/install_config/syncing_groups_with_ldap.html#sync-ldap-v1-activedirectoryconfig)
augmentedActiveDirectory|按照上述方式保存配置，以便以类似于活动目录中使用的方式设置 LDAP 服务器提取数据，并增加一项，一级组条目存在并用于保存元数据但不包含组成员资格。|[v1.AugmentedActiveDirectoryConfig](https://docs.openshift.org/3.6/install_config/syncing_groups_with_ldap.html#sync-ldap-v1-augmentedactivedirectoryconfig)
### v1.StringSource
`StringSource` 允许指定一个内联字符串，或通过环境变量或文件外部指定。当它仅包含一个字符串值时，它将封装到一个简单的 json 字符串。

名称|描述|Schema
---|---|---
value|指定明文值，或指定 keyFile 时的加密值|字符串
env |指定包含明文值的环境变量，或指定keyFile 时的加密值|字符串
file|引用包含明文值的文件，或指定keyFile 时的加密值|字符串
keyFile|引用包含用于解密值的密钥文件|字符串
### v1.LDAPQuery
LDAPQuery包含构建LDAP查询所需的选项。

名称|描述|Schema
---|---|---
baseDN|所有搜索应从其开始的目录分支DN |字符串
scope|搜索的可选范围。可以是`base`:只有基地对象、`one`:基础级别上的所有对象，`sub`:整个子树。如果未设置，则默认为 sub |字符串
derefAliases|与 alisases 有关的搜索行为(可选)。可以是 `never`:从来没有取消别名、`search`:只有搜索引用、`base`:只有在查找基础对象的引用、`always`:总是取消引用。如果未设置，则默认为 `always` |字符串
timeout|超时时间，以秒为单位保持对服务器的任何请求可以保持未完成状态的时间限制。如果是0则不会有超时| 整数
filter|一个有效的 LDAP 搜索过滤器，它使用基本 dn 从 LDAP 服务器中检索所有相关条目|字符串
pageSize|在 LDAP 条目中测量的最大首选页面大小。页面大小为0表示不会进行分页|整数
### v1.RFC2307Config
RFC2307Config 包含必要的配置选项以定义LDAP组同步如何使用RFC2307架构与LDAP服务器进行交互。
### v1.ActiveDirectoryConfig
ActiveDirectoryConfig包含必要的配置选项，以定义LDAP组同步如何使用Active Directory架构与LDAP服务器进行交互。

名称|描述|Schema
---|---|---
usersQuery|包含用于返回用户条目的 LDAP 查询的模版|[v1.LDAPQuery](https://docs.openshift.org/3.6/install_config/syncing_groups_with_ldap.html#sync-ldap-v1-ldapquery)
userNameAttributes|定义LDAP用户条目上的那些属性将被解释为其 openshift 用户名。用作 openshift 原始组记录中用户名的属性。 `mail` 或 `sAMAccountName` 是大多数安装中的首选。|字符串数组
groupMembershipAttributes|定义 LDAP 用户条目上的那些属性将被解释为它所属的组|字符串数组
### v1.AugmentedActiveDirectoryConfig
AugmentedActiveDirectoryConfig包含必要的配置选项，以定义LDAP组同步如何使用扩展的Active Directory架构与LDAP服务器进行交互。

名称|描述|Schema
---|---|---
usersQuery|包含用于返回用户条目的 LDAP 查询的模版|[v1.LDAPQuery](https://docs.openshift.org/3.6/install_config/syncing_groups_with_ldap.html#sync-ldap-v1-ldapquery)
userNameAttributes|定义LDAP用户条目上的那些属性将被解释为其 openshift 用户名。用作 openshift 原始组记录中用户名的属性。 `mail` 或 `sAMAccountName` 是大多数安装中的首选。|字符串数组
groupMembershipAttributes|定义 LDAP 用户条目上的那些属性将被解释为它所属的组|字符串数组
groupsQuery|保存用于返回组条目的LDAP查询的模板|[v1.LDAPQuery](https://docs.openshift.org/3.6/install_config/syncing_groups_with_ldap.html#sync-ldap-v1-ldapquery)
groupUIDAttribute|定义 LDAP 组条目上的哪个属性将被解释为其标识唯一标识符(ldapGroupUID)|字符串
groupNameAttributes|定义LDAP组条目上的哪些属性将被解释为其用于OpenShift组的名称。|字符串数组
