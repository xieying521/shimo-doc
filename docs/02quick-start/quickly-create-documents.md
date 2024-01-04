---
title: 10分钟创建协同文档
description: 10分钟创建协同文档
keywords: [石墨文档, 文档中台, 协同办公, 在线文档, 文件共享, 接口,  文档元信息]
sidebar_position: 3
---

如何快速创建一个word文档，并在石墨编辑器中使用，请参考下列步骤。

**步骤1：**接入方生成文件id，例如：my-new-file-u98213。

**步骤2**：接入方提供两个回调接口。

- 文档元信息接口
  提供一个公网可访问的HTTP接口，使公网GET请求访问
  http://your-callback.com/files/my-new-file-u98213 时，结果为：

```json
{
  "id": "my-new-file-u98213", // 此id和请求id要对应上
  "name": "第一个协同文档",
  "type": "documentPro",
  "permissions": {
    "commentable": true,
    "editable": true,
    "readable": true,
    "copyable": true,
    "exportable": true
  },
  "views": 100,
  "creatorId": "1",
  "createdAt": "2021-08-01T00:00:00Z",
  "updatedAt": "2021-08-02T00:00:00Z",
  "teamGuid": "123"
}
```

- 当前用户信息接口
  提供一个公网可访问的HTTP接口，使公网GET请求访问
   http://your-callback.com/users/current/info 时，结果为：

```json
{
  "id": "userid123",
  "name": "张三",
  "avatar": "http://fake.site/user-123.png", 
  "email": "user123@fake.site", 
  "teamGuid": "123"
}
```

**步骤3**：创建协同文档。

POST 调用接口 https://office.shimoapi.com/sdk/v2/api/files ，
请求参数分为query和body两个部分。

- query参数包括：appId，signature，token。
  - appId：license的appId。
  - signature：签名算法算出的结果。
  - token：接入方提供的 token ，在石墨请求接入方的接口时，会放到 HTTP Headers X-Shimo-Token 中作为值进行传递。

- body使用json格式，参数包括：type，fileId。
  - type：documentPro
  - fileId：my-new-file-u98213

请求示例：

```
curl --location --request POST 'https://office.shimoapi.com/sdk/v2/api/files?appId=yourShimoAppId&signature=yourSignatureString&token=yourTokenString' \
--header 'Content-Type: application/json' \
--data-raw '{"type": "document","fileId": "my-new-file-u98213"}'
```

