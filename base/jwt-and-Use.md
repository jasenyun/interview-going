# 在 Go 中学以致用 JWT

## 什么是 JWT

JWT (JSON Web Token) 是一个开放标准。各服务之间认证方式主要通过生成 JSON 对象。所以该 JSON 对象是一种数字签名，是可以被验证和信任的。

### JWT 组成

JWT 主要由三个部分组成：

- Header（头部）

- Payload （负载）

- Signature（签名）

```bash
Header.Payload.Signature
```

#### Header

主要描述 JWT 的元数据

```json
{
  "alg": "HS256",
  "typ": "JWT"
}
```

- alg 表示签名的算法，默认是 HMAC SHA256（写成 HS256）

- typ 表示这个 token 的类型，类型为 “JWT”

最后再使用 Base64算法进行编码即得到 Header 的值

#### Payload

主要存放实际要传递的数据

- iss (issuer)：签发人
- exp (expiration time)：过期时间
- sub (subject)：主题
- aud (audience)：受众
- nbf (Not Before)：生效时间
- iat (Issued At)：签发时间
- jti (JWT ID)：编号

还支持添加自定义字段

```json
{
  "sub": "1234567890",
  "name": "John Doe",
  "userId": 121212121212
}
```

最后再使用 Base64算法进行编码即得到 payload 的值

因 JWT 默认是不加密的，所有人都可以读取到，所以务必不要将重要信息放在 Header 或 payload 里面。

#### Signature

主要是对 Header 和 payload 的签名，防止数据被窜改。签名还需要一个密钥，该密钥仅保存在服务器，不可公开，否则容易被人伪造 token。签名的算法就是 Header 中指定的签名算法。根据签名公式，就可以得到签名：

```shell
HMACSHA256(base64UrlEncode(header) + "." + base64UrlEncode(payload),secret)
```

这样以后就计算出三个值出来，再将其用点 . 分隔开就得到一个 Token。

## 使用场景

- Authorization 授权。用户第一次登录后用，服务器上为该用户生成一个 Token ，该 Token 存储在用户浏览器中，随后用户所有的请求都需携带该 Token，服务器才能根据 Token 认证授权该请求。用户退出或登出则将该 Token 销毁。

- xInformation Exchange 信息交换，是各服务之间安全传输的最佳方式。

## Golang 中实践 JWT

基于 token 认证流程：

- 通过登录请求，验证成功后生成 Token 并返回客户端

- 再请求其他接口如获取用户列表时，需在 header 中携带 token 

- 服务端主要是用 gin 框架，在路由请求之前拦截，获取用户请求的 header ，获取用户信息并进行签名校验，在有效期内 token 有效则可继续请求否则中断

使用第三方 jwt 包：

```go
github.com/golang-jwt/jwt/v4
```

安装完就可以使用 jwt 包中提供的方法，封装一个生成token 和 解析 Token 的方法

```go
// 自定义 claim 其中包含 jwt 提供的标准 claims 即7个参数
type MyClaims struct {
	uid  string
	name string
	jwt.RegisteredClaims
}func GenerateToken(userId string, expired time.Duration) (string, error) {
	expire := time.Now().Add(expired)

	claims := MyClaims{
		userId,
		"tokentest",
		jwt.RegisteredClaims{
			ExpiresAt: jwt.NewNumericDate(expire),
			Issuer:    "test",
		},
	}
	token := jwt.NewWithClaims(jwt.SigningMethodHS256, claims)

	return token.SignedString([]byte(SecretKey))
}

func ParseToken(tokenString string) interface{} {
	var token, err = jwt.ParseWithClaims(tokenString, &MyClaims{}, func(token *jwt.Token) (interface{}, error) {
		return []byte(SecretKey), nil
	})
	if claims, ok := token.Claims.(*MyClaims); ok && token.Valid {
		return claims
	} else {
		fmt.Println(err)
		return nil
	}
}
```

提供2个接口，登录接口，实现生成 token 并返回，获取用户列表接口，但该接口有拦截器登录校验拦截，主要是验证用户是否携带 token

```go
APIV1User := r.Group("/api/v1/user")
	{
		APIV1User.Use(LoginRequest) // 路由中间件，拦截/api/v1/user 下所有请求
		APIV1User.GET("/", GetUserList)
	}


// 登录校验拦截
func LoginRequest(ctx *gin.Context) {
	header := ctx.Request.Header
	if header == nil {

	}
	auth := header["Authorization"]
	if auth == nil {
		fmt.Errorf("结果错误，返回")
		ctx.JSON(http.StatusBadRequest, gin.H{"code": 400, "message": "缺少authorization"})
		ctx.Abort()
		return
	} else {
		utils.ParseToken(auth[0])

		ctx.Next()
	}
}
```



参考资料：

- [JSON Web Token Introduction - jwt.io](https://jwt.io/introduction)

- https://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html
