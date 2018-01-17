## [JWT](http://www.jianshu.com/p/576dbf44b2ae)
JWT是什么 ？ JSON WEB TOKEN ， 是为了在网络应用环境间传递声明而执行的一种基于JSON的开放标准。此TOKEN被设计为紧凑且安全。特别适合分布式站点的单点登录(SSO)场景。
### JWT 构成
- 头部(header)  
头部包含信息
    - 声明类型
    - 声明加密算法(通常使用HMAC 或 SHA256)
    ```
    //完整头部
    {
        'type': 'JWT',//类型
        'alg': 'HS256'//加密算法
    }
    // 然后将头部进行base64编码(该编码可以对称解码)，这样就构成了第一部分
    // 编码后
    eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9
    ```
- 载荷(payload)  
载荷就是存放有效信息的地方，包括三部分
    - 标准中注册的声明(建议但不强制使用)
        - iis(Issuer)：jwt签发者
        - sub(Subject)：主题
        - aud(Audience)：接收jwt的一方
        - exp(Expiration time)：jwt的过期时间(这个时间必须大于签发时间)
        - nbf(Not before)：定义在什么时间之前，该jwt都是不可用的
        - iat(Issued at)：jwt的签发时间
        - jti(JWT ID)：jwt的唯一身份标识(主要用来作为一次性TOKEN，从而回避重放攻击)
        - name:
        - admin:
    - 公共的声明  
    公共声明可以添加任何信息，一般添加用户的相关信息或者其他业务需要的必要信息，但不建议添加敏感信息，因为该部分在客户端可以解码
    - 私有的声明  
    私有声明是提供者和消费者所共同定义的声明(不建议添加敏感信息)
        ```
        {
            "iss": "fadada.com",
            "exp": "1438955445",
            "name": "FG",
            "admin": "true"
        }
        // 使用 base64 编码后
        eyJpc3MiOiJuaW5naGFvLm5ldCIsImV4cCI6IjE0Mzg5NTU0NDUiLCJuYW1lIjoid2FuZ2hhbyIsImFkbWluIjp0cnVlfQ
        ```
- 签证(signature)  
    组成
    - header(base64后的header)
    - paylode(base64后的paylode)
    - secret(相当于服务端私钥，存储在服务端)
    ```
    var encodedString = base64UrlEncode(header) + "." + base64UrlEncode(payload); 
    HMACSHA256(encodedString, 'secret');//加密
    //处理后：SwyHTEx_RQppr97g4J5lKXtabJecpejuef8AqKYMAJc
    ```
    
**最终在服务端生成的并且发送给客户端的TOKEN**
```
// header.payload.signature
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJuaW5naGFvLm5ldCIsImV4cCI6IjE0Mzg5NTU0NDUiLCJuYW1lIjoid2FuZ2hhbyIsImFkbWluIjp0cnVlfQ.SwyHTEx_RQppr97g4J5lKXtabJecpejuef8AqKYMAJc
```
    
### 注意
1. secret保存在服务端
2. secret用于jwt的生成与验证
2. jwt的生成发生在服务端
### 应用
一般是在请求头中加入Authorization，并加上Bearer标注
```
fetch('api/user/1', {
  headers: {
    'Authorization': 'Bearer ' + token
  }
})
```
### 大致使用流程

![](http://ww1.sinaimg.cn/large/9da83df8gy1floesax3a5j21120ksta9.jpg)

### 优点
- 因为json的通用性，所以JWT是可以进行跨语言支持的，想java，javascript，NodeJS，PHP等语言都可以使用
- 因为有了payload(载荷)部分，所以JWT可以存储一些其他业务逻辑所必要的非敏感信息
- 便于传输，JWT结构非常简单，字节占用小
- 易扩展，因为不需要在服务端保存会话信息

### 安全相关
- 不要在JWT中存放敏感信息
- 保护好secret私钥
- 尽量使用https协议