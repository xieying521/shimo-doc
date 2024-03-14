---
sidebar_position: 7
description: >-
  Signature, a tool used by graphite servers to verify the legitimacy of requests, is signed with appId and secret, and then generates a JSON Web Token (JWT) format string.
  The signed result consists of three parts: Header, Payload, and Signature.
  By default, the header contains the following fields: alg and typ, which represent the signature algorithm, typ identifies the token type, and the kid field is the value of the AppId.
  The payload section has added an exp field with a value of the number of seconds of the JWT's expiration time. When requesting the /admin API, additional fileId information will be added.
  Callback credential types are divided into X-Shimo-Credential-Type and JSSDK. The JSSDK generates an editor address that points to the graphite server, which is then represented by an iframe
  into the current page.
---

# Development Resources

## signature

Use and appId and secret signatures for the graphite server to verify the legitimacy of the request.

The result of the signature (i.e., the value of the signature) is a [JSON Web Token ](https://jwt.io/introduction?accessToken=eyJhbGciOiJIUzI1NiIsImtpZCI6ImRlZmF1bHQiLCJ0eXAiOiJKV1QifQ.eyJleHAiOjE2MTcyODEyOTAsImciOiJXbEFyemI4WGFuZnBQYUEyIiwiaWF0IjoxNjE3Mjc5NDkwLCJ1Ijo1MDczNjIyfQ.2iCF5Xl2YB3hjkyOd29IrEJiDefUrQKpbHxM8hwhCBE&fileGuid=Wr3DVn8lO4HE2kJQ)(JWT) format string, and its structure includes three parts: Header, Payload, and Signature. 

JWT 通常形式为 xxxxx.yyyyy.zzzzz

- Header 部分
  - The following fields are included by default: alg and typ
    - **alg** stands for Signature Algorithm and usually defaults to **HS256**
    - **typ** identifies the token type, and the type of JSON Web Token defaults to **JWT**
  - Add ** the kid** field with the value ** of AppId**
- Payload
  - Only the **exp** field can be added, and the value is ** the number of seconds** of the expiration time** of the  JWT ** , for example, 1612411882 (if the value of seconds in the current time is smaller than the value of the exp field, it means that the expiration time has not been reached, and the signature will not be judged to be invalid).
  - When the `/admin` relevant API is requested, additional information is added `fileId` 

Data structure before signing:

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

Payload parameter

| The name of the field | type   | Examples of values     | Required | illustrate                                                 |
| :----- | :----- | :--------- | :--- | :--------------------------------------------------- |
| Exp    | number | 1612411882 | be   | The expiration time of the signature is called                                     |
| scope  | string | license    | not   | APIs used to access certain scopes, as described in the corresponding API descriptions |

### Java examples

Use [https://github.com/auth0/java-jwt](https://github.com/auth0/java-jwt?accessToken=eyJhbGciOiJIUzI1NiIsImtpZCI6ImRlZmF1bHQiLCJ0eXAiOiJKV1QifQ.eyJleHAiOjE2MTcyODEyOTAsImciOiJXbEFyemI4WGFuZnBQYUEyIiwiaWF0IjoxNjE3Mjc5NDkwLCJ1Ijo1MDczNjIyfQ.2iCF5Xl2YB3hjkyOd29IrEJiDefUrQKpbHxM8hwhCBE&fileGuid=Wr3DVn8lO4HE2kJQ)
For example

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

## Callback certificate type {#credential-types}

Because there are different triggering scenarios for request callbacks, there are different credential types, which are specified by the HTTP header of the callback request `X-Shimo-Credential-Type`. Known Credential Types:

- `X-Shimo-Credential-Type` == `0`This type is usually triggered by the user's operation behavior, such as opening a page, editing a file, etc., and the request will `X-Shimo-Token` be issued by the accessor through the HTTP header carrying the user during the browser operation `token`
- `X-Shimo-Credential-Type` == `3`, This type is usually triggered by the Graphite SDK, such as the update of cross-table references, etc., and the request will be verified by the HTTP header `X-Shimo-Signature` according to the current `appId` `secret` active signature of the accessing party for the accessing callback to confirm whether the request comes from the Graphite SDK

## JS SDK

The purpose of the JSSDK is to generate an editor address that points to the graphite server, which is then loaded into the current page as an iframe. It also encapsulates the iframe page communication method. The following actions/events can be achieved using the JSSDK:

- Open the sidebar (show and hide features such as history, version, global discussion, etc.)
- Get a list of current collaborators
- Subscribing to a document saves success/failure events
- More actions and events are being added on the way

JSSDK is published as an npm package,[ click-through access](https://www.npmjs.com/package/shimo-js-sdk), and webpack. 
