---
sidebar_position: 7
---

# 开发资源

## 签名 {#signature}

使用和 appId 和 secret 签名，用于石墨服务器验证请求合法性。

签名后的结果（即 signature 的值）为 [JSON Web Token](https://jwt.io/introduction?accessToken=eyJhbGciOiJIUzI1NiIsImtpZCI6ImRlZmF1bHQiLCJ0eXAiOiJKV1QifQ.eyJleHAiOjE2MTcyODEyOTAsImciOiJXbEFyemI4WGFuZnBQYUEyIiwiaWF0IjoxNjE3Mjc5NDkwLCJ1Ijo1MDczNjIyfQ.2iCF5Xl2YB3hjkyOd29IrEJiDefUrQKpbHxM8hwhCBE&fileGuid=Wr3DVn8lO4HE2kJQ)（简称为 JWT）格式字符串，其结构包括 Header、Payload、Signature 三个部分。

JWT 通常形式为 xxxxx.yyyyy.zzzzz

- Header 部分
  - 默认包含的字段：alg 、typ
    - **alg** 表示签名算法，通常默认为 **HS256**
    - **typ** 标识 Token 类型，JSON Web Token 的类型默认为 **JWT**
  - 添加 **kid** 字段，值为 **AppId** 的值
- Payload
  - 仅添加 **exp** 字段即可，值为 **JWT 的过期时间** 的 **秒数** ，例如 1612411882 （如当前时间的秒数值比 exp 字段的值要小，说明还没有到达过期时间，此签名不会被判断为失效）
  - 当请求 `/admin` 相关 API 时，会额外增加 `fileId` 信息

签名前的数据结构：

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

| 字段名 | 类型   | 值示例     | 必选 | 说明                                                 |
| :----- | :----- | :--------- | :--- | :--------------------------------------------------- |
| exp    | number | 1612411882 | 是   | 签名的过期时间称                                     |
| scope  | string | license    | 否   | 用于访问某些限定 scope 的 API，具体见对应的 API 说明 |

### Java 示例

使用 [https://github.com/auth0/java-jwt](https://github.com/auth0/java-jwt?accessToken=eyJhbGciOiJIUzI1NiIsImtpZCI6ImRlZmF1bHQiLCJ0eXAiOiJKV1QifQ.eyJleHAiOjE2MTcyODEyOTAsImciOiJXbEFyemI4WGFuZnBQYUEyIiwiaWF0IjoxNjE3Mjc5NDkwLCJ1Ijo1MDczNjIyfQ.2iCF5Xl2YB3hjkyOd29IrEJiDefUrQKpbHxM8hwhCBE&fileGuid=Wr3DVn8lO4HE2kJQ)
为例

```xml
<dependency>
  <groupId>com.auth0</groupId>
  <artifactId>java-jwt</artifactId>
  <version>4.4.0</version>
</dependency>
```

```java
// JDK8或以上
String signature = JWT.create()
                // .withClaim("scope", "license") //  如果调用需要scope参数的接口，需要添加此行
                .withKeyId("your shimo sdk AppId")
                .withExpiresAt(new Date().toInstant().plus(4, ChronoUnit.MINUTES))
                .sign(Algorithm.HMAC256("your shimo sdk AppSecret"));
```

## 回调凭证类型 {#credential-types}

由于请求回调存在不同的触发场景，因此存在不同的凭证类型，通过回调请求的 HTTP Header 指定 `X-Shimo-Credential-Type`。已知的凭证类型：

- `X-Shimo-Credential-Type` == `0`，此类型通常是由用户的操作行为触发，比如打开页面、编辑文件等，请求会通过 HTTP Header `X-Shimo-Token` 携带用户在浏览器操作时接入方颁发的 `token`
- `X-Shimo-Credential-Type` == `3`, 此类型通常由石墨 SDK 主动触发，如跨表格引用的更新等，请求会通过 HTTP Header `X-Shimo-Signature` 根据接入方当前的 `appId` `secret` 主动签名后供接入方回调进行校验，确认此请求是否来自石墨 SDK

## JS SDK

JS SDK 的作用是生成一个指向石墨服务器的编辑器地址，然后以 iframe 的形式加载到当前页面中。同时封装了 iframe 页面通信方法。使用 JSSDK 可以实现以下动作/事件：

- 打开侧边栏（历史、版本、全局讨论等功能的展示和隐藏）
- 获取当前协作者列表
- 订阅文档保存成功/失败事件
- 更多动作和事件还在持续更新中

JS SDK 以 npm 包的形式发布：
- [npm.js](https://www.npmjs.com/package/shimo-js-sdk)
- [npmmirror.com](https://npmmirror.com/package/shimo-js-sdk)

JS SDK 版本 | 兼容的版本 | 文档链接 |
---------|----------| ---- |
\>= 1.0 | 2022.12.1 (SDK 3.10) 或更早的版本 | [文档链接](https://github.com/shimohq/shimo-js-sdk/tree/master/docs) |
< 1.0 | 2022.10.1 (SDK 3.9) 或更早的版本 | [文档链接](https://github.com/shimohq/shimo-js-sdk/tree/v0.x/docs) |
