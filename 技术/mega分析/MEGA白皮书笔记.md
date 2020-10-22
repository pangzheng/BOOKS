## 设计目标
围绕用户控制的端到端加密，通过游览器访问，而不是先安装客户端，可以大大的方便用户
## 设计目的
- 独立部署硬件而不使用云，防止使用虚拟机导致的 cpu Spectre 攻击
- 基础设施架构保证数据冗余性和高可用性，每种基础架构都有自己的冗余和数据完整性保护机制
- 数据保存在欧洲机房(卢森堡最先进的 Tier IV 数据中心)，受到 GDPR 第45条获得充分的保护

## 架构包含
- MEGA 后台服务
	- API
		- 所有操作都通过 API，每次操作都存储到数据库中，所有数据库集群在各个地理区域都具有实时和延迟的从属复制位置并具有各种异地备份机制
		- 对 API 的更新也是要安全可控的，需要建立完整的更新流程，通过密钥对各级代码进行审查，以保证代码对数据库操作的完整性
	- 数据库
		- 加密后的密钥
		- 加密后文件的元数据
		- 用户信息 
	- 等 
- 数据存储
	- 有 200 PB
	- 有 680 亿个文件
	- 服务器使用 raid6,使用硬件基数来保证基础的可用性 
	- CloudRAID，保证单数据中心可用
- 文件元数据基础结构
	- 用户生成和加密的属性存储在 MEGA 上，例如图像，视频和文档的缩略图
	- 跨机房实时备份  

## 合规
- 法律合规保证合法用户的利益
- 版权问题
	- 需要拿到数据密钥的链接，可以申请数据问题
	- 数据下架遵循的是运营公司所在地法律
	- 找到法规保护运营公司在客户非法使用产品后，可以使用合规流程规避法律风险
	- 接收版权方、法律方声明，可以根据条款禁止该数据被外界访问，关闭访问接口法律是24小时，他们可以做到4个小时 
	- [隐私权](https://mega.nz/privacy)
	- [合规信息统计](https://mega.nz/Mega_Transparency_Report_201909.pdf)

## web UI 前端
核心网络客户端文件（`index.html` 和 `secureboot.js`）是通过 SSL 从根网络群集加载，https://mega.nz/secureboot.js 通过 sha256 验证，文件名用16进制表示

- `secureboot.js` 

	文件包含所有有效列表静态文件及其哈希摘要在相应的文件名中，负责验证静态文件是否一致。这些操作要在访问 api 之前完成，验证失败的话，将提示失败。因为静态资源是被攻击的一个核心点，这里包含客户端攻击和服务端攻击两部分。 也就是说，所有的静态文件引用的库都不会依赖第三方，而是完全自己提供和做 hash。
- 移动端不能使用游览器扩展，但可以提供全功能(待测试)，所以如果想增加安全性推荐使用客户端。

## GUI 移动应用
- 安卓
- IOS

## MEGAsync
- MAC
- WIN
- Linux

## CMD 命令行 
- MAC
- WIN
- Linux

## 注册流程
### 客户端处理
- 用户输入邮箱和密码
	- 邮箱不能超过190个字符，客户端和 api
	- 判断密码最小8位
	- 并根据 ZXCVBN 给密码强度打分
- 密码处理功能(PPF)
	- 选型
		- 使用 PBKDF2 对密码进行处理
		- API 端 WebCrypto API
		- web 客户端没有一个性能好的实现(是否使用 webassembly)

### 流程
- 使用 CSPRNG 生成128位随机数，类似 Crypto.getRandomValues()，包含
	- `Master Key` 	
	- `Client Random Value` 
- 派生密钥 `Derived Key` 为 256 位 32 字节 
	
		Derived Key = PBKDF2-HMAC-SHA-512( Password , Salt , Iterations , Length )
	- 迭代数(`Iterations`)设置为 100,000 ，主要取决于移动端处理性能和可接受时间范围内
	- salt 生成方法
	
			Salt = SHA-256( “mega.nz” || “Padding” || Client Random Value )
		- `mega.nz` 和填充字符的总长度将恰好为 200 个字符，填充字符（`Padding`) 为大写 P
		- 安全论文方案推荐用户标示符(用户邮箱)增加到 Salt 计算中，但因为 MEGA 给用户修改邮箱的权限，所以没有加入
- 派生密钥前 128 位，为派生加密密钥(`Derived Encryption Key`)，用来加密主密钥 `Master Key`

		Encrypted Master Key = AES-ECB( Derived Encryption Key , Master Key ) 
- 派生密钥后 128 位，用于生身份验证密钥(`Derived Authentication Key`),用于生成 `Hashed Authentication Key`
		
		Hashed Authentication Key = SHA-256( Derived Authentication Key )
	- 哈希认证密钥(`Hashed Authentication Key`)仅包含 SHA-256 输出的前128位来节省存储空间
- 客户端提交到 API 的数据
		
		First Name
		Last Name
		Email
		Client Random Value (128 bits / 16 Bytes)
		Encrypted Master Key (128 bits / 16 Bytes)
		Hashed Authentication Key (128 bits / 16 Bytes)
	不会存储 Salt ，它始终会被客户端生成
	
### 后端 API 	处理
- 后端使用 CSPRNG 生成 128 位的 `Email Confirmation Token`,并使用它、 `Base64UrlEncode`、`ConfirmCodeV2` 和邮箱，用 `Base64UrlEncode` 生成确认码

		Confirm Link = “https://mega.nz/#confirm” || Base64UrlEncode( “ConfirmCodeV2” || Email ConfirmationToken || Email )
	- `Base64UrlEncode` 替换字符串中的非字母字符，以便它可以在 url 中运行
- 将 `Confirm Link` 发到客户邮箱进行邮件确认

### 客户端处理
- 用户进入注册邮箱，单击确认后，客户端将发送确认码到其解码的 API
	- 如果注册成功， 客户端将跳转到前端，并自动输入登录邮箱，但是密码需要手动输入
	- 用户再次登录为了防止再另一个游览器打开相同链接(注册步骤)
	- 用户首次登录将生成真正给数据加密的密钥
		- 2048 位 RSA 密钥对（用于共享文件夹和文件？）
		- 256 位 `Ed25519` 密钥对（用户指纹验证和其他密钥签名的信任根），该密钥称为签名密钥。

			`Ed25519` 是一个数字签名算法，签名和验证的性能都极高， 一个4核2.4GHz 的 Westmere cpu，每秒可以验证 71000 个签名，安全性极高，等价于RSA约3000-bit。签名过程不依赖随机数生成器，不依赖hash函数的防碰撞性，没有时间通道攻击的问题，并且签名很小，只有64字节，公钥也很小，只有32字节。
		- 256 位 `Curve25519` 密钥对（用于 MEGAchat）
		
			`Curve25519` 是一个椭圆曲线提供 128 位安全性，设计用于椭圆曲线 Diffie-Hellman（ECDH）密钥协商方案。它是最快的 ECC 曲线之一，并未被zh任何已知专利所涵盖。
			
## 登陆流程
### 客户端处理
- 输入邮箱密码
- 邮箱发送到 API 验证

### 服务端处理
- 查询到邮箱后，发送管理数据
	- 有效返回 `Client Random Value` 
	- 无效返回 `Server Random Value`
		- 电子邮件的长度为 190，超过被拒绝
	
			因为有效和无效返回的计算量,总长度为200的 sha256 运算是一致的
		- `Server Random Value` 

			该值由 CSPRNG 生成并由 API 存储的静态字符串。当登陆邮件不存在时，使用它，可以防止垃圾电子邮件枚举攻击。
		- API  设计了一个0-50毫秒的随机延迟
			- 50 毫秒是数据库搜索和找不到的最大延迟
			- 通过该设计，防止攻击者通过延迟判断邮件在数据库中是否有效
			- 电子邮件长度检测和返回也受到延迟设计保护，防止 dos

### 客户端处理
- 计算 Slat ，方法同上
	- 有效

			Salt (computed by SHA-256( “mega.nz” || “Padding” || Client Random Value )
	- 无效

			Salt (computed by SHA-256( Email || “mega.nz” || “Padding” || Server Random Value )
- 计算派生密钥 `Derived Key`

		Derived Key = PBKDF2-HMAC-SHA-512( Password , Salt , Iterations , Length ) 
	- 长度？？
- 从派生密钥 `Derived Key` 前128位获取加密派生密钥 `Derived Encryption Key `
- 从派生密钥 `Derived Key` 后128位获取身份认证密钥 `Authentication Key`
- 发送邮箱和认证密钥 `Authentication Key` 给 API

### 服务端处理
- 后台计算身份认证密钥 `Authentication Key` 的 sha256 哈希

		Hashed Authentication Key = SHA-256( Authentication Key )
	这个操作怕数据库泄露后，攻击者通过 `Hashed Authentication Key` 验证用户
- 用身份验证密钥哈希 `Hashed Authentication Key` 前128位同数据库中的对比
	- 成功返回
		- 加密 `Master Key`
		- 加密的 `RSA` 私钥
		- 加密会话标示 `Encrypted Session Identifier`(en token)
			- 会话标识`Session Identifier` 符是API 在每个登录会话中生成的随机字符串（令牌），并用用户的 RSA 公钥 `RSA Public Key` 加密的 
			- 生成加密会话标示
				- 获取随机值，生成会话标示,可能使用  CSPRNG
				- 通过 `RSA` 公钥加密该标示
		- 等
	- 失败
		- 返回失败
	- API 通过 `Double HMAC` 来避免定时验证攻击
		- [HMAC](https://www.nccgroup.com/us/about-us/newsroom-and-events/blog/2011/february/double-hmac-verification/)  

			`HMAC` 是一种使用哈希函数生成键控消息身份验证代码的算法。（`HMAC`算法的输出也称为`HMAC`。）`HMAC`将消息和密钥作为输入，并生成固定长度的摘要输出，该摘要输出可与消息一起发送。知道密钥的任何人都可以重复该算法，并将计算出的 `HMAC` 与他们收到的`HMAC` 进行比较，以验证一条消息，该消息源自知道密钥的某人并且未被篡改。
			
			该算法的问题在于两个数组中匹配的字节越多，执行的比较越多，返回所需的时间也越长,攻击者通过返回时检查进行判断和攻击
		- 发 `Double HMAC`

			解决上面的问题

### 客户端处理
- 客户端使用派生加密密钥 `Derived Encryption Key` 解密加密主密钥 `Encrypted Master Key` 得到 `Master Key`
- 主密钥 `Master Key` 将用于解密 RSA 私钥 `RSA Private Key`。 
- RSA 私钥 `RSA Private Key` 将用于解密加密会话标识符` Encrypted Session Identifier`。 
- 后续请求 API 必须使用 TLS ，否则 API 将拒绝下载任何账户数据，如联系信息、加密文件、文件属性等

## 修改密码
- 修改密码将重新生成派生密钥
- 重新生成派生加密密钥
- 重新生成派生认证密钥
- 重新生成加密主密钥
- 重新上传加密主密钥
- 重新发送注册邮箱确认

修改密码需要注册邮箱参与

## 不变
- 主 key 不变
- RSA 密钥对不变
- ECC 密钥对不变
- 加密数据的 AES 不变
	
## 文件上传加密
### 客户端处理
- 生成文件密钥 `File Key`

	生成一个 128位和64位随机数的文件密钥对象 `File Key`，(随机数生成通过生成6组32位数字组成)
	
	- 128位是 Key
	- 64 位是 IV
- 使用 AES-CCM 对每个块进行加密
- 计算压缩 MAC
	- 通过将一个 128 位数组初始化为0
	- 然后按顺序使用第一个 MAC 进行异或，并使用 AES-ECB 加密
	- 然后使用加密结果与第二个块进行，再加密，再循环
	- 直到生成最终出来的 MAC
- 混淆 `File Key`

		Obfuscated File Key = [
		 File Key[0] ⊕ IV[0],
		 File Key[1] ⊕ IV[1],
		 File Key[2] ⊕ Condensed MAC[0] ⊕ Condensed MAC[1],
		 File Key[3] ⊕ Condensed MAC[2] ⊕ Condensed MAC[3],
		 IV[0],
		 IV[1],
		 Condensed MAC[0] ⊕ Condensed MAC[1],
		 Condensed MAC[2] ⊕ Condensed MAC[3]
		];					
- 通过主密钥加密混淆文件密钥 `File Key`

		Encrypted File Key = AES-ECB(Master Key, Obfuscated File Key)
- 加密文件属性数据(文件名、缩图、预览)

		AES-CBC(File Key, File Attribute Data)
- 上传到 API
 
## 文件数据下载
链接

	https://mega.nz/#! || Base64(File Handle)||!||Base64(Obfuscated File Key)

- 使用上面这些信息可以在服务器上找到由其文件句柄标识 `File Handle` 的文件
- 下载加密数据
- 验证整个文件的 `Condensed MAC`
- 解析混淆得到文件密钥 `File Key`
- 使用文件密钥对文件解密 `File Key` 和 `IV`。

请注意，URL中的锚点散列（＃）之后的所有内容都不会发送到 MEGA 服务器-这块是如何找到文件的？而是保留在客户浏览器的本地

## 目录文件夹
### 上传
- 创建共享文件夹？？做了什么事
- 生成随机共享密钥
- 用共享密钥加密所有文件数据混淆文件密钥(` Obfuscated File Key`)，注意这个基本上是明文

		AES-ECB(Share Key, Obfuscated File Key)
- 推送服务器这些数据？？？
- 这个目录是一个动态的，随着宿主的修改而变化

### 下载
公用文件夹链接被共享时

	https://mega.nz/#F! || Base64( Folder Share Handle ) || ! || Base64( Share Key )

- 通过文件夹共享句柄 ` Folder Share Handle`找到文件
- 下载所有数据和文件夹
- 验证压缩 ` Condensed MAC` 
- 取消混淆文件密钥 ` File Keys`
- 使用文件密钥和解密文件 ` File Key` 和 `IV`		

## 受密码保护的链接
### 生成
使用密码对文件夹或文件链接的密钥进行加密，方便在不安全的共享渠道进行数据共享

- 生成一个 256 位的 `salt`
- 生成一个 512 位的派生密钥(`Derived Key`)
- 获取派生密钥前128/256位当成加密密钥
- 使用异或加密文件或文件夹密钥，得到文件或目录的加密密钥(`Encrypted Key`)
- 派生密钥最后256位作为 MAC 密钥(` MAC Key`)
- 然后再进行 base 64 编码

		Base64(HMAC-SHA-256( MAC Key, ( Algorithm || Type || Public Handle || Salt || Encrypted Key )))

参数描述

- Algorithm = 1个字节-用来标识使用哪个算法（以备将来升级）的字节，最初设置为0
- Type = 1字节-标识链接是文件夹还是文件链接的字节（0 =文件夹，1 =文件）
- Public Handle = 6个字节-公用文件夹/文件句柄
- Salt = 32个字节-随机生成的 salt
- Encrypted Key = 16或32字节-加密的实际文件夹或文件文件密钥
- MAC Tag = 32字节-所有先前数据的 MAC，以确保链接的完整性，即计算为

如下链接，#P！之后解码URL的 Base64 部分标识符
		
		https://mega.nz/#P!AAA5TWTcs7YZg_hVxF0JTKxOZQ_s2d…
 
### 接受
- 转码 base 64 部分
- 获取第一个字节以检查使用哪种算法对数据进行加密
- 它将使用密码用相同的算法（提供了salt 和密码）导出相同的文件或者目录加密密钥
- 计算数据的 MAC。 如果匹配，则说明该链接未被篡改或损坏。 如果不匹配，显示一个错误，表明可能已篡改或更可能是用户输入了错误的密码。
 
## 公共链接过期
通过有效时间控制链接过期
## 导入公共链接文件
- 和上传一样，通过主密钥 ` Master Key` 加密需要添加文件的混淆密钥 `Obfuscated File Key`，但不上传文件
	- 文件虽然相同，但是加密文件密钥是重新加密重新上传的
	- 采用原始文件链接将违反版权法，要求导入后的数据将不能再访问原始数据
- 将密钥存储到用户的 API 上？存储到用户文件描述的表里？

## 协作共享(企业)
### 建立关系
- 通过电子邮件地址发送给另外一个用户，并让另外一个用户批准
- 也可以扫描 QR 共享联系方式，QR 默认将自动批准添加联系人，可以关闭自动添加。

### 交换密钥和验证
后台将为用户添加每个被添加的用户的数据，包含

- Ed25519 Public Key
- Ed25519 Public Key Fingerprint
- RSA Public Key
- RSA Public Key Signature
- Curve25519 Public Key
- Curve25519 Public Key Signature

将用户使用任意设备中的一个添加联系人时，该数据内容将会通过后台 API 更新。用户重新打开设备，都会成重新更新用户状态，包含文件、联系人、权限等。

首次登陆后，都会生成以上密钥对

- 每个用户都使用 `Ed25519 Private Key` 对 `RSA Public Key`和 `Curve25519 Public Key`签名
- 然后用  `Master Key` 加密所有私钥，上传 API
- 所有公钥和签名按照原数据上传到 API

#### 添加联系人流程
- 首次添加联系人时，将计算其第一个看到的公钥的 hash，并使用其前 160 位保留指纹，后续更改将提示警告提示公钥已经更改并需要验证，否则将无法通讯。

		Fingerprint = SHA-256( Ed25519 Public Key )
- 添加联系人时，将用以下方法生成加密联系人指纹，并存储到用户的API，并且只有用户才可以检索。

		Encrypted Fingerprint = AES-GCM( Master Key, Fingerprint || Verification Type )
- 当用户获取联系人信息时，将现验证对方的签名文件来验证对方发过来的数据是否可靠
	- 不匹配报错，防止中间人攻击 （MITM）

### 文件夹共享
- 共享文件夹先将创建一个随机的 128 位共享密钥 ` Share Key`。 文件夹中的所有文件密钥均其加密：

		Encrypted File Key = AES-CBC( Share Key , Obfuscated File Key )
- 然后使用联系人的公钥进行加密共享密钥 ` Share Key`

		Encrypted Share Key = RSA( Public RSA Key, Share Key )
- 将加密文件密钥 ` Encrypted File Keys` 和加密共享密钥 ` Encrypted Share Key` 发送到 API。 
- 联系人将下载加密共享密钥，并使用其 RSA 密钥  `Private RSA Key` 解密。 
- 然后他们将能够下载并解密文件夹共享的文件密钥，可用于解密文件。
- 与非联系人共享时，必须双方用户全部在线才能加密联系人的密钥，不在线无法处理，直到用户在线才可以。
- 未注册的用户，通过电子邮箱发送邀请后，除非该用户注册，否则无法访问对应的链接

### 企业用户密钥交换
三种账户类型

- 主账户
	
	账户管理者，有全部权限
- 子账户

	用户，但自己的 `MasterKey` 管理账户也加密保存一份，所以数据可以被管理账户查看
- 未登陆账户(业务账户)

	保留整个企业账户的key 和属性？
	
企业流程
	
- 注册单个账户，并将该用户帐户（新的临时帐户或注册的临时账户）转换为主帐户来创建业务账户
- 主用户账户为企业创建一组业务帐户新的密钥（`Master Key`和 `RSA对`）
- 业务账户 `Master Key` 已使用主用户账户的 `Master Key` 加密，并由 API 存储-主用户数据中？

		Encrypted Business Master Key = AES-ECB( Master User Master Key, Business Master Key )
- 主用户账户通过发送带有唯一 URL 的邀请电子邮件来创建子用户。加载此 URL 表示 API 将新创建的用户链接到公司？
- 子用户创建过程与单个用户相同，额外的步骤是使用企业账户的 `RSA 公钥` 加密自己的 `Master Key`，并将其与其他密钥一起发送给 API。而不是直接加密 `Master Key`，以支持未来的可能性每个企业有多个主用户帐户

		Encrypted Sub-user Master Key = RSA( Business Public RSA Key, Sub-user Master Key )
- 由于 API 允许主用户账户请求子用户的文件树，因此主用户将始终有子用户上传的任何文件的访问权，即使这些文件离开了业务。

## MEGAchat 短信
MEGAchat 旨在保护交换内容的隐私和机密性。 MEGAchat 支持交换短信、文件和联系人。 

- 保密性

	只有作者和预定的收件人才能解密和阅读内容；
- 真实性

	系统确保接收到的数据是从发送方发出，并且内容没有在传输过程中被篡改

### 密码原语
使用 128 位 AES 加密做密码，等效 256 位 ECC 和 3072 位 RSA 加密，但后两者加密计算成本过高
### 消息加密
#### 加密密钥
- 随机生成唯一 128 位长的加密密钥，可在 `AES-CTR` 模式下使用,注意这里是用来加密真正的消息。
- 生成共享的秘密(万能密钥)

	使用一个人自己的私钥（𝑆𝑜𝑤𝑛）和另一个参与者的公钥（𝑃other）通过 `Curve25519 Diffie-Hellman` 标量乘法（ECDH并随后应用密钥导出函数（KDF, specifically [HKDF]-SHA-256）可以导出密钥。它被修剪到所需的密钥大小（128个最高有效位）

		𝐾𝐷𝐻, 𝑑𝑒𝑠𝑡 = 𝐾𝐷𝐹(𝐸𝐶𝐷𝐻(𝑆𝑜𝑤𝑛 , 𝑃𝑜𝑡h𝑒𝑟))
- 加密密钥 ID 的长度为32位，因为每个聊天的每个发送者都必须是唯一的
- 生成收件人的 IV
	- 收件人的用户句柄被 base64 URL 解码，产生一个8字节（64位）的序列（𝑢）
	- 从这些字节中使用消息的主随机数`master nonce (𝑛)`作为密钥，来计算密钥键哈希消息验证码（HMAC-哈希消息认证码，HMAC-SHA-256=sha256、HMAC-SHA-512=sha512）
	- 然后将其修整为所需的 IV 大小（128个最高有效位）
	
			𝐼𝑉𝑑𝑒𝑠𝑡 = 𝐻𝑀𝐴𝐶(𝑛𝑚𝑎𝑠𝑡𝑒𝑟 , 𝑢𝑜𝑡h𝑒𝑟)
- 加密密钥要使用 `AES-CBC` 模式进行加密，需要使用初始化向量（IV）和万能密钥(ECDH 与每个参与者共享秘密)，加密 `AES-CTR` 消息密钥

### 消息加密
- 用户生成自己的 AES 加密密钥对聊天内容加密
- 然后与其他参与者成对交换这些密钥
- 然后对加密密钥和消息进行加密并传输到 API


## 加密方案使用统计
加密方案|使用方法|加密对象|得到
---|---|---|---
CSPRNG|系统随机算法，算出各种随机 key|无|随机值，比如 `Master Key`
PBKDF2-HMAC-SHA-512|配合密码、salt、迭代数等得到派生key|密码| `Derived Key`
AES-ECB/对称加密|使用密码加密数据 | `Master Key`、`RSA私钥`、`Obfuscated File Key` | 加密的 `Master Key`、`RSA私钥`、`Obfuscated File Key`
RSA-2048/非对称加密|使用公钥加密用户登陆 `token`|登陆 `token`|加密的 `token`
Curve25519/非对称加密|和其他用户的公钥一起生成万成密钥|提供 ECDH 方案使用|无|万能密钥
AES-CBC(块?)/对称加密| 配合 ECDH 万能密钥+IV 的方式使用| AES-CTR  的数据密钥和真实文件属性| 加密的 AES-CRT 密钥和真实文件属性
AES-CTR(流?)/对称加密|使用随机密钥加密消息数据|真实的数据|加密后的数据
AES-CCM(块?)/对称加密|加密使用随机密钥加密用户上传文件|分块文件|加密后的分块文件
AES-GCM(块?)/对称加密|加密 `Master Key` 加密联系人指纹 |联系人指纹|加密的联系人指纹(`Fingerprint = SHA-256( Ed25519 Public Key )`)