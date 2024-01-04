---
title: 签名方式
description: 如何进行签名？
keywords: [石墨文档, 文档中台, 协同办公, 在线文档, 文件共享, 签名, Token]
sidebar_position: 4
---

使用 [App ID 和 App Secret](./../02quick-start/process-specification.md#appid) 签名，用于石墨服务器验证接入方的请求合法性。

签名后的结果（即 signature 的值）为 [JSON Web Token](https://jwt.io/introduction?accessToken=eyJhbGciOiJIUzI1NiIsImtpZCI6ImRlZmF1bHQiLCJ0eXAiOiJKV1QifQ.eyJleHAiOjE2MTcyODEyOTAsImciOiJXbEFyemI4WGFuZnBQYUEyIiwiaWF0IjoxNjE3Mjc5NDkwLCJ1Ijo1MDczNjIyfQ.2iCF5Xl2YB3hjkyOd29IrEJiDefUrQKpbHxM8hwhCBE&fileGuid=Wr3DVn8lO4HE2kJQ)（简称为 JWT）格式字符串，其结构包括 Header、Payload、Signature 三个部分。

JWT 通常形式为 xxxxx.yyyyy.zzzzz

- Header 部分

  - 默认包含的字段：alg 、typ

    - **alg** 表示签名算法，通常默认为 **HS256**
    - **typ** 标识 Token 类型，JSON Web Token 的类型默认为 **JWT**

  - 添加 **kid** 字段，值为 **App ID**

- Payload 部分

  - 仅添加 **exp** 字段即可，值为 JWT 的过期时间的秒数 ，例如 1612411882 （如当前时间的秒数值比 exp 字段的值要小，说明还没有到达过期时间，此签名不会被判断为失效）

  - 当请求 `/admin` 相关 API 时，会额外增加 `fileId` 信息

**签名前的数据结构：**

Header

```json
{
  "alg": "HS256", // 默认字段
  "typ": "JWT", // 默认字段
  "kid": "your AppId"
}
```

Payload

```json
{
  "exp": 1612411882
}
```

Payload 参数

| 字段名 | 字段名 | 值示例     | 必选 | 说明                                                         |
| ------ | ------ | ---------- | ---- | ------------------------------------------------------------ |
| exp    | number | 1612411882 | 是   | 签名的过期时间称                                             |
| scope  | string | license    | 否   | 用于访问某些限定 scope 的 API，详细信息请参考对应的 API 说明 |

**Java 示例：**

以 https://github.com/auth0/java-jwt 为例

```java
<dependency>
  <groupId>com.auth0</groupId>
  <artifactId>java-jwt</artifactId>
  <version>4.4.0</version>
</dependency>
```

```java
// JDK8或以上
// “获取App详情”和“更新App回调地址”接口需要带上 scope: license
String signature = JWT.create()
                // .withClaim("scope", "license") //  如果调用需要scope参数的接口，需要添加此行，且将过期时间改为4分钟
                .withKeyId("your shimo sdk AppId")
                .withExpiresAt(new Date().toInstant().plus(7, ChronoUnit.DAYS))
                .sign(Algorithm.HMAC256("your shimo sdk AppSecret"));
```
