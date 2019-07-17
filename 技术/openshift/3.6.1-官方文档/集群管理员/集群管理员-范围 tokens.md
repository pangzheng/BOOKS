# 集群管理员-范围 tokens
用户可能希望赋予另一个实体以现有的方式行事的权利，但只能是有限的方式行事。例如，项目管理员可能想要委托创建 pod 的权利。一种方法是创建一个范围 token。

作用域标记是为了给定用户的标记，但仅限于其范围内的某些操作。默认，只有一个 `cluster-admin` 可以创建范围 token.
## 评估
通过将标记的范围集合转换成为一组 `PolicyRules `来评估范围。然后，请求与这些规则相匹配。请求属性必须至少匹配其中一个范围规则，以传递给 "正常" 授权人进行进一步授权检查。
## 用户范围
用户范围着重于获取关于给定用户的信息。基于意图的，所有规则会自动创建:

- `user:full`

	允许完整的读写访问 API 和所有用户权限
- `user:info`

	允许以只读的方式的用户的信息，例如: `name`,`gruop`等
- `user:check-access`

	允许访问自 `self-localsubjectaccessreviews` 和 `self-localsubjectaccessreviews` ,这些是请求对象中传递的一个空用户和组的变量。
- `user:list-projects`	 

	允许只读访问列出用户有权访问的项目

## 角色范围
角色范围允许用户通过命名空间像过滤给定角色一样具有相同的访问级别。

- `role:<cluster-role name>:<namespace or * for all>`

	将范围限制为在 `cluster-role` 指定的规则，但仅限于指定的命名空间

	警告：
	
	这可以防止升级访问。即时角色允许访问 `secrets\rolebindings\roles` 等资源，该范围也会拒绝这些资源的访问。这有助于防止意外升级。许多人认为 `edit` 角色不是升级角色，但是可以获得 `secret`。		
- `role:<cluster-role name>:<namespace or * for all>:!`

	与上面类似，不同的是，除了指定项目项目，其他项目都要指定规则。

	

	