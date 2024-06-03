[toc]

# 基本概念

JWT 由三个部分组成，它们之间用 . 分隔，格式如下：Header.Payload.Signature → eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJhdWQiOiJQcm9ncmFtbWVyIiwiaXNzIjoi56iL5bqP5ZGY6ZmI5piO5YuHIiwic3ViIjoiY2hlbm1pbmd5b25nLmNuIn0.uRnH-rUb7lsZtQ11o8wXjIOJnIMBxszkvU1gY6hCGjo

1. **Hedaer** 部分用于描述该 `JWT` 的基本信息，比如其类型（通常是 `JWT`）以及所使用的签名算法（如 `HMAC SHA256` 或 `RSA`）。

2. **Payload（负载）**： `Payload` 部分包含所传递的声明。声明是关于实体（通常是用户）和其他数据的语句。声明可以分为三种类型：**注册声明**、**公共声明** 和 **私有声明**。
   
   - **注册声明**：这些声明是预定义的，非必须使用的但被推荐使用。官方标准定义的注册声明有 7 个：
   
    | Claim（声明）        | **含义**                                            |
    | -------------------- | --------------------------------------------------- |
    | iss(Issuer)          | 发行者，标识 JWT 的发行者。                         |
    | sub(Subject)         | 主题，标识 JWT 的主题，通常指用户的唯一标识         |
    | aud(Audience)        | 观众，标识 JWT 的接收者                             |
    | exp(Expiration Time) | 过期时间。标识 JWT 的过期时间，这个时间必须是将来的 |
    | nbf(Not Before)      | 不可用时间。在此时间之前，JWT 不应被接受处理        |
    | iat(Issued At)       | 发行时间，标识 JWT 的发行时间                       |
    | jti(JWT ID)          | JWT 的唯一标识符，用于防止 JWT 被重放（即重复使用） |
   
   - **公共声明**：可以由使用 JWT 的人自定义，但为了避免冲突，任何新定义的声明都应已在 IANA JSON Web Token Registry 中注册或者是一个 公共名称，其中包含了碰撞防抗性名称（Collision-Resistant Name）。
   - **私有声明**：发行和使用 JWT 的双方共同商定的声明，区别于 注册声明 和 公共声明。
   
3. **Signature**（签名）：为了防止数据篡改，将头部和负载的信息进行一定算法处理，加上一个密钥，最后生成签名。如果使用的是 HMAC SHA256 算法，那么签名就是将编码后的头部、编码后的负载拼接起来，通过密钥进行 HMAC SHA256 运算后的结果。

# 鉴权流程

## 总体流程

![image-20240603211808788](JWT鉴权.assets/image-20240603211808788.png)

## 拦截器工作流程

![image-20240603213128231](JWT鉴权.assets/image-20240603213128231.png)

# 代码实现

> JWT官方工具包 [golang-jwt/jwt: Community maintained clone of https://github.com/dgrijalva/jwt-go](https://github.com/golang-jwt/jwt)
## JWT生成

```go
func JwtEncode(ctx context.Context, userId int) (tokenString string, err error) {
	data := jwt.MapClaims{
		"userid": userId,
		"exp":    jwt.NewNumericDate(time.Now().Add(time.Duration(config.Jwt().Duration) * time.Hour)),
		"jti":    guid.S(),
	}
    ## 加载密钥
	key := config.Jwt().Key
	Secret := []byte(key)
    ## 生成令牌
	token := jwt.NewWithClaims(jwt.SigningMethodHS256, data)
    ## 通过密钥加密
	tokenString, err = token.SignedString(Secret)
	return
}
```

## JWT拦截器---中间件

```go
// JWT校验拦截器 
func (s *sMiddleware) JwtInterceptor(r *ghttp.Request) {
	var (
		jwtKey      = []byte(config.Jwt().Key)
		tokenString = r.Header.Get("Authorization")
	)
	// 解析JWT令牌
	token, err := jwt.Parse(tokenString, func(token *jwt.Token) (interface{}, error) {
		return jwtKey, nil
	})
	// token是否有效
	if err != nil || !token.Valid {
		r.Response.WriteStatus(http.StatusForbidden)
		r.Exit()
	}
	// 获取用户id
	claims := token.Claims.(jwt.MapClaims)
	userid := gconv.Int(claims["userid"])
	// Redis中是否存在JWT
	ctx := r.GetCtx()
	search, _ := g.Redis().Do(ctx, "GET", fmt.Sprintf("jwt.token.%d", userid))
	if search.String() != tokenString {
		r.Response.WriteStatus(http.StatusForbidden)
		r.Exit()
	}
    // // 上下文用户信息设置
    // service.Context().SetUser(r.GetCtx(),&ucontext.User{
	// 	UserId: userid,
	// })
	r.Middleware.Next()
}
```

## 路由绑定

> goframe 框架

```go
// 用户组
group.Group("/user",func(group *ghttp.RouterGroup) {
    // 绑定JWT校验拦截器
    group.Middleware(service.Middleware().JwtInterceptor)
    group.Bind(
        user.NewV1(),
    )
})
```

