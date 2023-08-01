# 简介

JSON Web Token

官网https://jwt.io/



# 组成

jwt 由头部、载荷、签名三部分通过`.`分割共同组成，头部是表明使用的编码类型。

长度不超过128

组成形式为

```json
xxx.yyy.zzz
```



头部和载荷使用Base64编码，签名必须要有已编码的头部和已编码的有效载荷，使用头部声明的算法把**密钥**签在上面，就形成第三部分签名。

例如使用`HMAC SHA256`算法：

```
HMACSHA256(base64UrlEncode(header) + "." + base64UrlEncode(payload), secret)
```

![legacy-app-auth-5](assets/JWT.assets/legacy-app-auth-5.png)

# 问题

jwt 是明文密钥，但是机制使得不能被篡改。一次签发永久有效，不过可以设置过期时间，但是过期时间前都是有效的

所以要防止被截取。

处理方法：

1. 使用 HTTPS 协议
2. 代码层面验证，比如ip地址，mac地址