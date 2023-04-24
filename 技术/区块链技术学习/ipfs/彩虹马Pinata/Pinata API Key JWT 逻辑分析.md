# Pinata API Key JWT 分析
## 问题
- Pinata 中用户的 API KEY 是如何生成、储存、验证的
- 不同权限的 JWT token 有什么区别

## 管理员权限分析
### 获取 JWT 数据
使用界面创建一个 API KEY 记录下面3个信息

- APIKEY
	
		14ba431fef9b543dff86 
- API Secret

		bdf11003076c932e6d14429bb24a510a60471c161ff33e0db67225a819a673a2
- JWT token

			eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VySW5mb3JtYXRpb24iOnsiaWQiOiI4MGZkY2Q1MS02MDI4LTQwZTYtYjZmMC1lMTMwMTQ5Y2MyMTUiLCJlbWFpbCI6InBhbmd6aGVuZ0BnbWFpbC5jb20iLCJlbWFpbF92ZXJpZmllZCI6dHJ1ZSwicGluX3BvbGljeSI6eyJyZWdpb25zIjpbeyJpZCI6IkZSQTEiLCJkZXNpcmVkUmVwbGljYXRpb25Db3VudCI6MX1dLCJ2ZXJzaW9uIjoxfSwibWZhX2VuYWJsZWQiOmZhbHNlLCJzdGF0dXMiOiJBQ1RJVkUifSwiYXV0aGVudGljYXRpb25UeXBlIjoic2NvcGVkS2V5Iiwic2NvcGVkS2V5S2V5IjoiMTRiYTQzMWZlZjliNTQzZGZmODYiLCJzY29wZWRLZXlTZWNyZXQiOiJiZGYxMTAwMzA3NmM5MzJlNmQxNDQyOWJiMjRhNTEwYTYwNDcxYzE2MWZmMzNlMGRiNjcyMjVhODE5YTY3M2EyIiwiaWF0IjoxNjgxNzE5Mzg0fQ.6D4vzp1juTqJo4flmM3LoLWhQ0RM0qW8CaiJrstyWPM

### 使用 JWT 的解析工具分析
- 将数据填入

	https://jwt.io/ 的 Encoded 
- Decoded 输出
	- HEADER

			{
			  "alg": "HS256",
			  "typ": "JWT"
			} 
	- Paylaod

			{
			  "userInformation": {
			    "id": "80fdcd51-6028-40e6-b6f0-e130149cc215", 	<- 类似数据库 ID
			    "email": "pangzheng@gmail.com", 				<-注册邮箱
			    "email_verified": true,							<- 是否验证
			    "pin_policy": {									<- 存储策略
			      "regions": [									<- 注册区
			        {
			          "id": "FRA1",
			          "desiredReplicationCount": 1
			        }
			      ],
			      "version": 1									<- 版本
			    },
			    "mfa_enabled": false,							<- 未知?
			    "status": "ACTIVE"								<- Token状态
			  },
			  "authenticationType": "scopedKey",				<- 验证类型 scopedKey
			  "scopedKeyKey": "14ba431fef9b543dff86",			<- APIKey
			  "scopedKeySecret": "bdf11003076c932e6d14429bb24a510a60471c161ff33e0db67225a819a673a2", <- API Secret
			  
			  "iat": 1681719384									<- 签发时间
			}
	- 验证

			HMACSHA256(
			  base64UrlEncode(header) + "." +
			  base64UrlEncode(payload),
				your-256-bit-secret
			) secret base64 encoded

### 数据分析			
我们发现里面的


	package main
	
	import (
	    "crypto/rand"
	    "crypto/rsa"
	    "encoding/base64"
	    "fmt"
	    "time"
	
	    "github.com/dgrijalva/jwt-go"
	)
	
	func generateScopedKey(clientID string, privateKey *rsa.PrivateKey) (string, error) {
	    // 设置JWT的有效期为1小时
	    token := jwt.NewWithClaims(jwt.SigningMethodRS256, jwt.StandardClaims{
	        Issuer:    clientID,
	        ExpiresAt: time.Now().Add(time.Hour).Unix(),
	    })
	    // 使用私钥对JWT进行签名
	    scopedKey, err := token.SignedString(privateKey)
	    if err != nil {
	        return "", err
	    }
	    return scopedKey, nil
	}
	
	func verifyScopedKey(scopedKey string, clientID string, publicKey *rsa.PublicKey) (bool, error) {
	    // 解析JWT
	    token, err := jwt.Parse(scopedKey, func(token *jwt.Token) (interface{}, error) {
	        // 验证签名算法是否为RS256
	        if _, ok := token.Method.(*jwt.SigningMethodRSA); !ok {
	            return nil, fmt.Errorf("invalid signing method: %v", token.Header["alg"])
	        }
	        return publicKey, nil
	    })
	    if err != nil {
	        return false, err
	    }
	    // 验证JWT的有效期和发行者是否匹配
	    claims, ok := token.Claims.(jwt.MapClaims)
	    if !ok || !token.Valid || claims["iss"] != clientID {
	        return false, nil
	    }
	    return true, nil
	}
	
	func generateScopedKeySecret() string {
	    // 生成32位随机字符串作为scopedKeySecret
	    key := make([]byte, 32)
	    _, err := rand.Read(key)
	    if err != nil {
	        panic(err)
	    }
	    // 将密钥转换为Base64编码字符串
	    return base64.StdEncoding.EncodeToString(key)
	}
	
	func verifyScopedKeySecret(scopedKeySecret string, expectedSecret string) bool {
	    // 使用常量时间比较函数比较 expectedSecret 和 scopedKeySecret
	    return hmac.Equal([]byte(scopedKeySecret), []byte(expectedSecret))
	}
	
	func main() {
	    // 调用 RSA 库，生成 RSA 公私钥密钥对，其中私钥用于生成 scopedKey，而公钥用于验证 scopedKey 的签名。
	    privateKey, err := rsa.GenerateKey(rand.Reader, 2048)
	    if err != nil {
	        panic(err)
	    }
	    publicKey := &privateKey.PublicKey
	
	    // 生成一个名为“scopedKey”的临时凭证，以便可以使用某些服务或 API。为了生成这个凭证，我们使用了 Golang 中的一个 JWT 库（JWT 是一种用于生成和验证令牌的标准方法）。
	    // 使用私钥生成了scopedKey，并将其设置为有效期为1小时。scopedKey 是一个由三个部分组成的字符串，分别是Header、Payload和Signature。其中Header包含了加密算法和类型等信息，Payload包含了一些关于scopedKey的元数据，例如签发者、过期时间等信息，而Signature 则是使用私钥对 Header 和 Payload 进行签名后的结果。将这三个部分组合在一起后，我们就得到了一个完整的 scopedKey。
	    
	    // 我们将这个 scopedKey 传递给某些服务或 API，以便它们可以验证我们的身份并授权我们进行一些操作。服务或 API 使用我们提供的公钥来验证 scopedKey 的签名，以确保 scopedKey 是由我们发出的，并且没有被篡改或伪造。如果验证通过，则服务或API将继续执行我们请求的操作。	   
	    clientID := "myClientID"
	    scopedKey, err := generateScopedKey(clientID, privateKey)
	    if err != nil {
	        panic(err)
	    }
	    fmt.Println("Scoped Key:", scopedKey)
	
	    // 验证 scopedKey
	    isValid, err := verifyScopedKey(scopedKey, clientID, publicKey)
	    if err != nil {
	        panic(err)
	    }
	    fmt.Println("Scoped Key is valid:", isValid)
	
	    // 生成 scopedKeySecret，生成一个名为“scopedKeySecret”的秘密密钥，以便我们可以使用某些服务或API。scopedKeySecret是一个由随机生成的字符串组成的字符串，用于验证我们的身份并授权我们进行一些操作。请注意，scopedKeySecret 仅在服务器端使用，并且不应该在客户端中存储或传输，以确保其安全
	    
	    // 将scopedKeySecret传递给某些服务或API，以便它们可以验证我们的身份并授权我们进行一些操作。服务或API使用我们提供的密钥来验证scopedKeySecret，以确保我们是经过授权的用户，并且具有执行特定操作的权限。如果验证通过，则服务或API将继续执行我们请求的操作。
	    scopedKeySecret := generateScopedKeySecret()
	    fmt.Println("Scoped Key Secret:", scopedKeySecret)
	
	    // 验证 scopedKeySecret
	    expectedSecret := "myExpectedSecret"
	    isValidSecret := verifyScopedKeySecret(scopedKeySecret, expectedSecret)
	    fmt.Println("Scoped Key Secret is valid:", isValidSecret)
	}									