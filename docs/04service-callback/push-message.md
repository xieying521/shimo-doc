---
title: 消息推送
description: 消息推送的触发场景
keywords: [石墨文档, 文档中台, 协同办公, 在线文档, 文件共享, 触发, 推送, 通知]
sidebar_position: 8
---

## 概述

此接口用于用户在石墨文件中触发特定行为时，向接入方发送通知。

目前会推送的事件：

- 评论 (Comment)
- 讨论 (Discussion)
- 提及 (MentionAt @人)
- 日期提醒 (DateMention)
- 文件内容更新 (FileContent)
- 文档协作者协同状态变化
- 版本 (Revision)
- 系统事件 (System)
  - 回调请求错误

**请求地址**

*POST* /events

**HTTP Request Headers**

| **Header 名**     | **值**             | **说明**                 |
| :---------------- | :----------------- | :----------------------- |
| X-Shimo-Token     | 接入方提供的 Token | 用于接入方对本次请求鉴权 |
| X-Shimo-Sdk-Event | Comment            | 用于识别具体的事件类型   |

**HTTP Response Code**
200

**HTTP Response Body**
空 json，无需返回内容。

请求参数和请求体根据不同事件有所不同。

## 评论 （Comment）

在创建评论、更新评论、删除评论、关闭评论时，会触发相应事件推送。

**HTTP Request Headers**

| Header 名         | 值      | 说明                   |
| :---------------- | :------ | :--------------------- |
| X-Shimo-Sdk-Event | Comment | 用于识别具体的事件类型 |

### 轻文档

添加评论

```json
{
  "kind": "comment",
  "type": "comment", // 预留评论下的小分类，目前就 comment
  "action": "create", // 业务操作
  "fileId": "file1", // 接入方文件 ID
  "userId": "user1", // 接入方用户 ID
  "comment": {
    "guid": "wS2uTQBMuujrwaGZ", // 评论 ID
    "content": "这是评论内容", // 评论内容
    "userIds": ["user1", "user2"], // 评论中若有 at 用户，此参数会传递一份，也可通过 mention_at 事件中获取
    "selectionGuid": "comment-Ynimup3kBXoxwF4j", // 评论划词高亮区域的 ID
    "selectionContent": "123123123123"
  }
}
```

删除评论

```json
{
  "kind": "comment",
  "type": "comment",
  "action": "delete",
  "fileId": "5b4f80b2aeb9d47d",
  "userId": "1",
  "deleteComment": {
    "guid": "EGhnSobUZWujKG2g", // 评论 ID
    "selectionGuid": "comment-Ynimup3kBXoxwF4j" // 划词高亮区域 ID
  }
}
```

结束评论

```json
{
  "kind": "comment",
  "type": "comment",
  "action": "closeComments",
  "fileId": "5b4f80b2aeb9d47d",
  "userId": "1",
  "closeComment": {
    "selectionGuid": "comment-Ynimup3kBXoxwF4j" // 划词高亮区域 ID
  }
}
```

### 表格

添加评论

```json
{
  "kind": "comment",
  "type": "comment",
  "action": "create",
  "fileId": "6ee9f8da39498a33",
  "userId": "1",
  "comment": {
    "guid": "8dkDnFH6HPaGPqD1", // 评论 ID
    "selectionTitle": "MODOC", // 评论所在表格 sheet ID
    "selectionGuid": "comment-OWTDgRUtF1LDoLwh", // 评论所在位置 ID
    "content": "测试表格评论", // 评论内容
    "userIds": ["user1", "user2"] // at 的用户列表，没有则空
  }
}
```

删除评论

```json
{
  "kind": "comment",
  "type": "comment",
  "action": "delete",
  "fileId": "6ee9f8da39498a33",
  "userId": "1",
  "deleteComment": {
    "guid": "bfdbJyDNlctJXcnw", // 评论 ID
    "selectionGuid": "comment-OWTDgRUtF1LDoLwh" // 评论位置区域 ID
  }
}
```

结束评论

```json
{
  "kind": "comment",
  "type": "comment",
  "action": "closeComments",
  "fileId": "6ee9f8da39498a33",
  "userId": "1",
  "closeComment": {
    "selectionGuid": "comment-OWTDgRUtF1LDoLwh" // 评论位置区域 ID
  }
}
```

### 传统文档

添加评论

```json
{
  "kind": "comment",
  "type": "comment",
  "action": "create",
  "fileId": "691f6abc72f769a4",
  "userId": "1",
  "comment": {
    "guid": "ZwOJjlTDHcPda0V4", // 评论 ID
    "selectionGuid": "comment-Y29tbWVudD05Ng==", // 划词高亮区域 ID
    "content": "3123123", // 评论内容
    "userIds": ["user1"] // 评论中 at 的用户，若无则为空
  }
}
```

更新评论

```json
{
  "kind": "comment",
  "type": "comment",
  "action": "update",
  "fileId": "691f6abc72f769a4",
  "userId": "1",
  "comment": {
    "guid": "ZwOJjlTDHcPda0V4", // 评论 ID
    "selectionGuid": "comment-Y29tbWVudD05Ng==", // 划词高亮区域 ID
    "content": "312312345666", // 更新的评论内容
    "userIds": ["user1"] // 评论中 at 的用户，若无则为空
  }
}
```

删除评论

```json
{
  "kind": "comment",
  "type": "comment",
  "action": "delete",
  "fileId": "691f6abc72f769a4",
  "userId": "1",
  "deleteComment": {
    "guid": "JrJWv9VuuOWlEo64", // 评论 ID
    "selectionGuid": "comment-Y29tbWVudD05Ng==" // 划词高亮区域 ID
  }
}
```

### 幻灯片

添加评论

```json
{
  "kind": "comment",
  "type": "comment",
  "action": "create",
  "fileId": "6ee9f8da39498a33",
  "userId": "1",
  "comment": {
    "guid": "Ddg3iIcz1nfqJSo9", // 评论 ID
    "selectionTitle": "s2f6cb94", // 评论所在幻灯片页面 ID
    "selectionGuid": "comment-rrsIAvomFD3girqU", // 评论所在位置 ID
    "content": "测试评论", // 评论内容
    "userIds": [] // 幻灯片暂不支持 at 用户，为空
  }
}
```

删除评论

```json
{
  "kind": "comment",
  "type": "comment",
  "action": "delete",
  "fileId": "6ee9f8da39498a33",
  "userId": "1",
  "deleteComment": {
    "guid": "Ddg3iIcz1nfqJSo9", // 评论 ID
    "selectionGuid": "comment-rrsIAvomFD3girqU" // 评论位置区域 ID
  }
}
```

结束评论

```json
{
  "kind": "comment",
  "type": "comment",
  "action": "closeComments",
  "fileId": "6ee9f8da39498a33",
  "userId": "1",
  "closeComment": {
    "selectionGuid": "comment-rrsIAvomFD3girqU" // 评论位置区域 ID
  }
}
```

### 应用表格

添加评论

```json
{
  "kind": "comment",
  "type": "comment",
  "action": "create",
  "fileId": "794821fa1c537c88",
  "userId": "5",
  "comment": {
    "guid": "vi3sZCU3SPmYkaPL",
    "selectionTitle": "tiAtgfY7iRH", // 工作表 ID
    "selectionGuid": "comment-5CZl3gADOnG", // 评论位置
    "selectionContent": "JAuXsJM8obN:标题", //
    "content": "123123123 @yanjixiong ",
    "userIds": ["5"]
  }
}
```

对于评论的回复评论

```json
{
  "kind": "comment",
  "type": "comment",
  "action": "create",
  "fileId": "794821fa1c537c88",
  "userId": "5",
  "comment": {
    "guid": "wY2THFlt3HoxbpMI", // 评论 ID
    "selectionTitle": "tiAtgfY7iRH", // 工作表 ID
    "selectionGuid": "comment-r3Zmnqa1lm2", // 评论位置
    "content": "123132", // 评论内容
    "userIds": [],
    "replyTo": "JIyRqgGY3GzyNj0d", // 回复的评论 ID
    "replyComment": {
      "userId": "5", // 被回复评论的创建者
      "commentGuid": "JIyRqgGY3GzyNj0d", // 被回复评论的 ID
      "selectionTitle": "tiAtgfY7iRH", // 工作表 ID
      "selectionGuid": "comment-r3Zmnqa1lm2" // 评论位置
    }
  }
}
```

删除评论

```json
{
  "kind": "comment",
  "type": "comment",
  "action": "delete",
  "fileId": "794821fa1c537c88",
  "userId": "5",
  "deleteComment": {
    "guid": "vi3sZCU3SPmYkaPL", // 评论 ID
    "selectionGuid": "comment-5CZl3gADOnG" // 评论位置
  }
}
```

## 讨论 （Discussion）

在创建讨论消息时，会触发相应事件推送。其中表格、传统文档、应用表格不支持讨论功能。

**HTTP Request Headers**

| Header 名         | 值         | 说明                   |
| :---------------- | :--------- | :--------------------- |
| X-Shimo-Sdk-Event | Discussion | 用于识别具体的事件类型 |

### 轻文档

发送讨论消息

```json
{
  "kind": "discussion",
  "type": "discussion", // 预留讨论下的小分类，目前就 discussion
  "action": "create",
  "fileId": "5b4f80b2aeb9d47d", // 接入方文件 ID
  "userId": "user1", // 接入方用户 ID
  "discussion": {
    "id": "61f0bc916014eb0006c85c6f", // 讨论消息 ID
    "unixus": 1643166865906215, // 时间，微秒
    "content": "123",
    "positionId": "" // 预留字段，目前无用
  }
}
```

## 提及（@ MentionAt）

在 评论、讨论 消息中 @ 一个或多个用户时，将会触发此事件。

被 @ 的用户 ID 列表，由 SDK 在事件数据中传递给接入方。

**HTTP Request Headers**

| Header 名         | 值        | 说明                   |
| :---------------- | :-------- | :--------------------- |
| X-Shimo-Sdk-Event | MentionAt | 用于识别具体的事件类型 |

### 轻文档

在评论中 at

```json
{
  "kind": "mention_at",
  "type": "comment", // 子分类，comment、discussion、mention_at
  "action": "create", // create 等
  "fileId": "file1", // 接入方文件 ID
  "userId": "user1", // 接入方用户 ID
  "comment": {
    "guid": "yAvBcaQON88pGq5C", // 评论 ID
    "selectionGuid": "comment-MSS0l7ZjOzoDzs8M", // 评论划词 ID
    "selectionContent": "123123", // 选中文字
    "content": "@zhangsan @lisi 123", // 评论内容
    "userIds": ["user1", "user2"] // 评论中 @ 的用户
  }
}
```

在讨论中 at

```json
{
  "kind": "mention_at",
  "type": "discussion", // 子分类，comment、discussion、mention_at
  "action": "create", // create 等
  "fileId": "file1", // 接入方文件 ID
  "userId": "user1", // 接入方用户 ID
  "discussion": {
    "id": "61e69df547051f0006eb972e", // 讨论 ID
    "unixus": 1642503669147636, // 发送时间
    "content": "@zhangsan 12312313", // 讨论发送内容
    "userIds": ["user1", "user2"] // 讨论中 @ 的用户
  }
}
```

在正文中 at

```json
{
  "kind": "mention_at",
  "type": "mention_at", // 子分类，comment、discussion、mention_at
  "action": "create", // create 等
  "fileId": "file1", // 接入方文件 ID
  "userId": "user1", // 接入方用户 ID
  "mentionAt": {
    "guid": "lJ48R1wX26xARMni", // at 人在内容中的标识
    "content": "在 at 人前面的文字 @yinfeng", // at 文字内容，轻文档会包含 at 前面的文字
    "userId": "user1" // 提及的 userId
  }
}
```

### 表格

在评论中 at

```json
{
  "kind": "mention_at",
  "type": "comment", // 子分类，comment、discussion、mention_at
  "action": "create", // create 等
  "fileId": "file1", // 接入方文件 ID
  "userId": "user1", // 接入方用户 ID
  "comment": {
    "guid": "AChYTfButGeGArBi", // 评论 ID
    "selectionGuid": "comment-vTBHz6FpLckq2duh", // 评论划词 ID
    "selectionTitle": "MODOC", // 表格场景为 sheetId
    "content": "@zhangsan 123", // 评论内容
    "userIds": ["user1"] // 评论中 @ 的用户
  }
}
```

在正文中 at

```json
{
  "kind": "mention_at",
  "type": "mention_at", // 子分类，comment、discussion、mention_at
  "action": "create", // create 等
  "fileId": "file1", // 接入方文件 ID
  "userId": "user1", // 接入方用户 ID
  "mentionAt": {
    "guid": "MODOChZuU2S8M35Y", // at 人在内容中的标识
    "content": "at 前面的文字", // at 文字内容，不包含 at 文字本身
    "userId": "user1" // 提及的 userId
  }
}
```

### 传统文档

在评论中 at

```json
{
  "kind": "mention_at",
  "type": "comment", // 子分类，comment、discussion、mention_at
  "action": "create", // create 等
  "fileId": "file1", // 接入方文件 ID
  "userId": "user1", // 接入方用户 ID
  "comment": {
    "guid": "A3pRPpD08scPV1Pc", // 评论 ID
    "selectionGuid": "comment-A3pRPpD08scPV1Pc", // 评论划词 ID
    "content": "@zhanguang 123123", // 评论内容
    "userIds": ["user1"] // 评论中 @ 的用户
  }
}
```

在正文中 at

```json
{
  "kind": "mention_at",
  "type": "mention_at", // 子分类，comment、discussion、mention_at
  "action": "create", // create 等
  "fileId": "file1", // 接入方文件 ID
  "userId": "user1", // 接入方用户 ID
  "mentionAt": {
    "guid": "Ym9va21hcms9X0F0MQ==", // at 人在内容中的标识
    "userId": "user1" // 提及的 userId
  }
}
```

### 幻灯片

暂不支持 at

### 应用表格

在评论中 at

```json
{
  "kind": "mention_at",
  "type": "comment",
  "action": "create",
  "fileId": "794821fa1c537c88",
  "userId": "5",
  "comment": {
    "guid": "61p3ftQ8XuE31oba", // 评论 ID
    "selectionTitle": "tiAtgfY7iRH", // 工作表 ID
    "selectionGuid": "comment-5ehKQW140LZ", // 评论位置 ID
    "selectionContent": "JAuXsJM8obN:标题",
    "content": "12312312313 @zhangsan", // 评论 at 内容
    "userIds": ["2"] // 提及用户
  }
}
```

在正文中 at

```json
{
  "kind": "mention_at",
  "type": "mention_at",
  "action": "create",
  "fileId": "794821fa1c537c88",
  "userId": "5",
  "mentionAt": {
    "guid": "tiAtgfY7iRHthWE9C0jIFcskBkByFcwVAhFQMQVEgj0B",
    "userId": "1" // 提及用户
  }
}
```

## 日期提醒 (DateMention)

在文件中插入日期提醒规则时，会将规则信息推送至接入方，接入方可按需根据创建、更新、删除事件实现业务系统中的定时提醒功能。

:::caution 说明

石墨文档中台会在页面上操作后  `立即`  将  `创建`、`修改`、`删除`  事件推送至接入方，定时提醒功能需要业务方系统实现，石墨 SDK 不会在目标时间达到时发送任何事件通知。

:::

**HTTP Request Headers**

| Header 名         | 值          | 说明                   |
| :---------------- | :---------- | :--------------------- |
| X-Shimo-Sdk-Event | DateMention | 用于识别具体的事件类型 |

### 轻文档

HTTP Request Body

创建

```json
{
  "kind": "mention",
  "type": "date_mention",
  "action": "create",
  "createData": {
    "id": "MlRFslp55Mrt19Iq", // 提醒 ID
    "fileId": "ac4ce108419f103c", // 文件 ID
    "authorId": "12", // 创建者用户 ID
    "content": "<a>2022年1月19日 </a>", // 提醒文字
    "remindUserIds": ["12"], // 提醒用户
    "remindAt": "2021-12-07T15:00:00Z", // 提醒时间
    "positionId": "12" // 预留字段，暂无用途
  }
}
```

修改

```json
{
  "kind": "mention",
  "type": "date_mention",
  "action": "update",
  "updateData": {
    "id": "MlRFslp55Mrt19Iq", // 提醒 ID
    "content": "<a>2022年1月20日 </a>", // 提醒文字
    "remindUserIds": ["user1"], // 提醒用户
    "remindAt": "2021-12-07T15:00:00Z", // 提醒时间
    "positionId": "12" // 预留字段，暂无用途
  }
}
```

删除

```json
{
  "kind": "mention",
  "type": "date_mention",
  "action": "remove",
  "removeData": {
    "id": "MlRFslp55Mrt19Iq" // 提醒 ID
  }
}
```

### 表格

HTTP Request Body

创建

```json
{
  "kind": "mention",
  "type": "date_mention",
  "action": "create",
  "createData": {
    "id": "MlRFslp55Mrt19Iq", // 提醒 ID
    "fileId": "ac4ce108419f103c", // 文件 ID
    "authorId": "12", // 创建者用户 ID
    "content": "123", // 提醒文字，若未设置文字，则为空
    "remindUserIds": ["12"], // 提醒用户
    "remindAt": "2021-12-07T15:00:00Z", // 提醒时间
    "positionId": "MODOC-QjUpDSt3swY6Tlir" // 预留字段，暂无用途
  }
}
```

修改

```json
{
  "kind": "mention",
  "type": "date_mention",
  "action": "update",
  "updateData": {
    "id": "MlRFslp55Mrt19Iq", // 提醒 ID
    "content": "12313", // 提醒文字
    "remindUserIds": ["12"], // 提醒用户
    "remindAt": "2021-12-07T15:00:00Z", // 提醒时间
    "positionId": "MODOC-QjUpDSt3swY6Tlir" // 预留字段，暂无用途
  }
}
```

删除

```json
{
  "kind": "mention",
  "type": "date_mention",
  "action": "remove",
  "removeData": {
    "id": "MlRFslp55Mrt19Iq" // 提醒 ID
  }
}
```

### 传统文档

HTTP Request Body

创建

```json
{
  "kind": "mention",
  "type": "date_mention",
  "action": "create",
  "createData": {
    "id": "MlRFslp55Mrt19Iq", // 提醒 ID
    "fileId": "ac4ce108419f103c", // 文件 ID
    "authorId": "12", // 创建者用户 ID
    "content": "2022年01月19日, 周三", // 提醒文字
    "remindAt": "2021-12-07T15:00:00Z", // 提醒时间
    "positionId": "36" // 预留字段，暂无用途
  }
}
```

修改

```json
{
  "kind": "mention",
  "type": "date_mention",
  "action": "update",
  "updateData": {
    "id": "MlRFslp55Mrt19Iq", // 提醒 ID
    "content": "2022年01月21日, 周五 19:56", // 提醒文字
    "remindAt": "2021-12-07T15:00:00Z", // 提醒时间
    "positionId": "36" // 预留字段，暂无用途
  }
}
```

删除

```json
{
  "kind": "mention",
  "type": "date_mention",
  "action": "remove",
  "removeData": {
    "id": "MlRFslp55Mrt19Iq" // 提醒 ID
  }
}
```

### 幻灯片

暂不支持

## 文件内容更新 (FileContent){#update-file}

在文件有用户进行修改并被服务器处理成功后，将会触发此事件。

**HTTP Request Headers**

| Header 名               | 值                                   | 说明                                                                                            |
| :---------------------- | :----------------------------------- | :---------------------------------------------------------------------------------------------- |
| X-Shimo-Credential-Type | 3                                    | 无法提供 token 的 [回调凭证类型](./../04service-callback/overview.md#credential-types)          |
| X-Shimo-Sdk-Event       | FileContent                          | 用于识别具体的事件类型                                                                          |
| X-Shimo-Signature       | ebc1cde3-9b57-4962-883d-54302d428600 | 当石墨无法提供接入方 token 时，使用接入方对应 appId, secret 主动生成 signature 用于回调接口校验 |

**HTTP Request Body**

```json
{
  "kind": "fileContent",
  "type": "fileContent",
  "action": "update",
  "fileId": "file1", // 接入方文件 ID
  "userId": "user1", // 接入方用户 ID
  "fileContent": {
    "version": 2 // 此事件发生时产生的新版本
  },
  "timestamp": 1635732089224 // 此事件产生的时间
}
```

## 文档协作者协同状态变化 (Collaborator)

当文档当前参与协作的用户发生改变时，推送此事件。

事件可通过  `action`  的值区分用户行为：

- `enter`  网络无异常情况下，进入文档，触发此事件。
- `leave`  用户关闭文档或网络断开时，触发此事件。
  **HTTP Request Headers**

| Header 名               | 值                                   | 说明                                                                                              |
| :---------------------- | :----------------------------------- | :------------------------------------------------------------------------------------------------ |
| X-Shimo-Credential-Type | 3                                    | 无法提供 token 的  [回调凭证类型](./../04service-callback/overview.md#credential-types)           |
| X-Shimo-Sdk-Event       | Collaborator                         | 用于识别具体的事件类型                                                                            |
| X-Shimo-Signature       | ebc1cde3-9b57-4962-883d-54302d428600 | 当石墨无法提供接入方 Token 时，使用接入方对应 appId, secret 主动生成 signature 用于回调接口校验。 |

**HTTP Request Body**

```json
{
  "kind": "collaborator",
  "action": "enter", // enter 或 leave 代表用户加入或退出协作
  "fileId": "file1", // 接入方文件 ID
  "userId": "user1", // 接入方用户 ID
  "collaboratorChanged": {
    // 一个用户可能有多个clientId参与同一个文档的协作
    "clientId": "47c8c791-9938-4f09-b76a-d6a2dc3cda5e"
  },
  "timestamp": 1635732089224 // 此事件产生的时间
}
```

## 版本 (Revision)

当有用户创建、修改、删除版本并被服务器处理成功后，推送此事件。

事件可通过  `action`  的值区分用户行为：

- `create`  用户创建版本时，触发此事件
- `update`  用户修改版本时，触发此事件
- `delete`  用户删除版本时，触发此事件

**HTTP Request Headers**

| Header 名         | 值       | 说明                   |
| :---------------- | :------- | :--------------------- |
| X-Shimo-Sdk-Event | Revision | 用于识别具体的事件类型 |

**HTTP Request Body**

```json
{
  "kind": "revision",
  "action": "create",
  "fileId": "file1", // 接入方文件 ID
  "userId": "user1", // 接入方用户 ID
  "revision": {
    // 事件对应版本的 id
    "revisionId": 1348,
    // 版本的标题
    "title": "无标题",
    // 版本的标签
    "label": "2022/2/28 星期一 18:12",
    // 文档版本的 id
    "docHistoryId": "621c9ff51125670006875854"
  },
  "timestamp": 1635732089224 // 此事件产生的时间
}
```

## 系统事件 (System)

推送 SDK 系统级别的事件，目前为实验性功能，未来可能会有调整或变化。

**HTTP Request Headers**

| Header 名         | 值     | 说明                   |
| :---------------- | :----- | :--------------------- |
| X-Shimo-Sdk-Event | System | 用于识别具体的事件类型 |

### (实验性) 回调请求错误

当石墨文档中台请求接入方接口出现异常时，将  `请求数据`  和  `响应数据`  以及文档中台错误通过此消息推送至接入方接口，用于及时发现接入方自身回调接口问题。

:::caution 说明

此事件目前仅包含部分请求的错误信息，暂未包含所有场景，不建议依赖此事件作为回调是否正常的依据。

:::

**HTTP Request Body**

Request Headers 中的相关信息参考  [回调凭证类型](./../04service-callback/overview.md#credential-types)说明。

- `X-Shimo-Credential-Type`
- `X-Shimo-Token`
- `X-Shimo-Signature`

```json
{
  "kind": "System",
  "type": "endpointCallback",
  "appId": "your app id",
  "timestamp": 1669610727509, // 此事件产生的时间戳，单位毫秒
  "request": {
    // 错误回调请求的请求数据
    "headers": {
      "Accept": "application/json",
      "Content-Type": "application/json",
      "X-Shimo-APP-ID": "your app id",
      "X-Shimo-Credential-Type": "0", // 0 代表使用用户 token 发起的回调请求， 3 表示石墨 SDK 主动签名发起的回调请求
      "X-Shimo-Token": "your user token",
      "X-Shimo-Signature": "the signature generate by sdk"
    },
    "url": "http://your-callback-api-host/callback/users/current/info",
    "body": ""
  },
  "response": {
    // 错误回调请求的响应数据
    "headers": {
      "Connection": "keep-alive",
      "Content-Length": "9",
      "Content-Type": "text/plain; charset=utf-8",
      "Date": "Mon, 28 Nov 2022 04:45:27 GMT"
    },
    "status": 404,
    "body": "Not Found"
  },
  "errorMessage": "endpoint: request endpoint server failed with status code 0, code 10009, message: GET request failed (http://your-callback-api-host/callback/users/current/info) with status code: 404 "
}
```
