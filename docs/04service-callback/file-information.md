---
title: 文件信息
sidebar_position: 4
---
## 文件权限说明{#file-permission}

文件权限是对文件的进行访问控制和操作控制的一种机制，包括读取权限、写入权限和执行权限。在石墨SDK中，文件权限主要由Permissions 字段控制。

|**字段名**|**类型**|**说明**|
|:----|:----|:----|
|commentable|boolean|用户是否可以评论文档。|
|editable|boolean|用户是否可以编辑文档。|
|readable|boolean|用户是否可以访问文档|
|copyable|boolean|用户是否可以复制文档内容。当 editable 值域为 true 时，copyable 一定为 true。|
|exportable|boolean|用户是否可以导出文件。|
|manageable|boolean|用户是否可以管理文档，例如删除、移动等操作。|

## 获取当前用户的文件列表{#list-files}

在添加跨表格公式、合并工作表等操作时，选择用户最近使用的文件列表会请求此接口。

**请求地址**

*GET* /files

**HTTP Request Headers**

|**Header 名**|**值**|**说明**|
|:----|:----|:----|
|X-Shimo-Token|接入方提供的 Token|用于接入方对本次请求鉴权|

**HTTP Query Parameters**

|**字段名**|**类型**|**值示例**|**说明**|
|:----|:----|:----|:----|
|limit|number|123|文件列表最大条目数 ID|
|orderBy|string|created_at|文件列表按文件创建时间或最后修改时间排序，可选created_at、 updated_at|

**HTTP Response Body**

|字段名|类型|值示例|说明|
|:----|:----|:----|:----|
|items[0].id|string|ba13551165cc5066|接入方系统中的文档 ID|
|items[0].name|string|file title|文档标题|
|items[0].type|string|documentPro|文档类型列表请参考 [创建文档](./../06API-document/interface-description/collaborative-editing.md#create-collab-file)，若为第三方存储的非石墨协作文件，则固定传 `file`|
|items[0].creatorId|string|1|接入方文件的创建者用户 ID|
|items[0].createdAt|string|2021-08-01T00:00:00Z|接入方记录的文件创建时间，`UTC` 时间（0 时区）， 格式为 `2022-01-01T09:00:01Z`|
|items[0].updatedAt|string|2021-08-02T00:00:00Z|接入方记录的文件最后更新时间，`UTC` 时间（0 时区）， 格式为 `2022-01-01T09:00:01Z`|
|items[0].views|number|100|接入方统计的文件阅读次数，用于在文档信息中展示阅读次数。若接入方未返回，则默认显示为 1|
|items[0].permissions|object|-|请参考 [文件权限说明](#file-permission)|

**Response Body Example**

```json
[
  {
    "id": "ba13551165cc5066",
    "name": "示例表格",
    "type": "spreadsheet",
    "permissions": {
      "commentable": true,
      "editable": false,
      "readable": true
    },
    "creatorId": "1",
    "createdAt": "2021-08-01T00:00:00Z", // UTC 时间，0 时区，在北京时间基础上减 8 小时
    "updatedAt": "2021-08-02T00:00:00Z", // UTC 时间，0 时区，在北京时间基础上减 8 小时
    "views": 100
  },
  {
    "id": "ba13551165cc5066",
    "name": "示例文档",
    "type": "document",
    "permissions": {
      "commentable": true,
      "editable": false,
      "readable": true
    },
    "creatorId": "1",
    "createdAt": "2021-08-01T00:00:00Z", // UTC 时间，0 时区，在北京时间基础上减 8 小时
    "updatedAt": "2021-08-02T00:00:00Z", // UTC 时间，0 时区，在北京时间基础上减 8 小时
    "views": 100
   },
  {
    "id": "ba13551165cc5066",
    "name": "接入方自己的文件.docx",
    "type": "file",
    "permissions": {
      "commentable": false,
      "editable": false,
      "readable": true
    },
    "creatorId": "1",
    "createdAt": "2021-08-01T00:00:00Z", // UTC 时间，0 时区，在北京时间基础上减 8 小时
    "updatedAt": "2021-08-02T00:00:00Z", // UTC 时间，0 时区，在北京时间基础上减 8 小时
    "views": 0
   },
]
```

## 获取文件元信息-协同文档{#file-meta-collab}

获取文件的基本信息、权限等，创建、导入、编辑文件时会请求此接口。

**请求地址**

*GET* /files/{fileId}

**HTTP Request Headers**

|**Header 名**|**值**|**说明**|
|:----|:----|:----|
|X-Shimo-Token|接入方提供的 Token|用于接入方对本次请求鉴权|

**HTTP Response Body**

|字段名|类型|值示例|说明|
|:----|:----|:----|:----|
|id|string|123|接入方系统中的文档 ID|
|name|string|file title|文档标题|
|type|string|documentPro|文档类型列表请参考 [创建文档](./../06API-document/interface-description/collaborative-editing.md#create-collab-file)|
|creatorId|string|1|接入方文件的创建者用户 ID|
|createdAt|string|2021-08-01T00:00:00Z|接入方记录的文件创建时间，`UTC` 时间（0 时区）， 格式为 `2022-01-01T09:00:01Z`|
|updatedAt|string|2021-08-02T00:00:00Z|接入方记录的文件最后更新时间，`UTC` 时间（0 时区）， 格式为 `2022-01-01T09:00:01Z`|
|views|number|100|接入方统计的文件阅读次数，用于在文档信息中展示阅读次数。若接入方未返回，则默认显示为 1|
|permissions|object|    |请参考 [文件权限说明](#file-permission)|
|teamGuid|string|123|接入方文件所属团队 ID，非必需。若返回此字段，则 user.teamGuid 字段也应实现并返回|

:::caution 说明

`manageable` 权限说明：建议至少给文件的创建者指定 `manageable: true` 权限，若接入方允许有其他人拥有管理权限，则可自行设计实现。

使用到 `manageable` 权限的场景如下：

* 调用石墨 SDK 删除文件接口时，需要 `manageable` 为 `true`
* 进行表格锁定操作时，若当前用户具有 `manageable` 权限为 `true` 时，则会展示为管理者
* `更新` 或 `删除` 他人创建的版本时，当前用户对于此文件的权限需要 `manageable` 为 `true` 时，才允许操作，否则会被拒绝
* 表格导出时，若当前用户对于此文件权限满足 `manageable` 为 `true` 时，导出的文件中会包含表格锁定的内容，否则导出文件内容不包含锁定内容。

:::

**Response Body Example**

```json
{
  "id": "ba13551165cc5066",
  "name": "示例文档",
  "type": "document",
  "permissions": {
    "commentable": true,
    "editable": true,
    "readable": true,
    "copyable": true,
    "exportable": true
  },
  "views": 100,
  "creatorId": "1",
  "createdAt": "2021-08-01T00:00:00Z", // UTC 时间，0 时区，在北京时间基础上减 8 小时
  "updatedAt": "2021-08-02T00:00:00Z", // UTC 时间，0 时区，在北京时间基础上减 8 小时
  "teamGuid": "123" // 接入方文件所属团队 ID
}
```

## 获取文件元信息-文件预览{#file-meta-preview}

获取基本信息、下载地址，预览文件时会请求此接口。

:::caution 说明

此接口地址与 [获取文件元信息-协同文档](#获取文件元信息-协同文档) 相同，但返回结果不同；可能需要接入方自行区分文件是石墨协同文档还是普通文件。

:::

**请求地址**

*GET* /files/{fileId}

**HTTP Request Headers**

|**Header 名**|**值**|**说明**|
|:----|:----|:----|
|X-Shimo-Token|接入方提供的 Token|用于接入方对本次请求鉴权|

**HTTP Response Body**

|字段名|类型|值示例|说明|
|:----|:----|:----|:----|
|id|string|123|接入方系统中的文件 ID|
|name|string|file title|文件标题|
|type|string|file|目前使用 `file `表示第三方文件类型|
|permissions|object| - |请参考 [文件权限说明](#file-permission)|
|downloadUrl|string|[http://fake.site/sample.docx](http://fake.site/sample.docx)|用于预览 Office 文件时下载文件内容|
|ext|string|docx|当石墨请求下载地址 (`downloadUrl`) 返回的 HTTP Response Header 中 `Content-Type` 或 `Content-Disposition` 接入方未返回可识别的信息时，此字段用于标识此文件的扩展名。不需要带" `.`"，若文件为`测试文件.docx` ，则应传 `docx`，若返回 `.docx `可能识别失败。|

:::caution 说明

- 请求下载时接入方无法返回 `Content-Type` 以及 `Content-Disposition` 信息用于石墨 SDK 识别文件类型，此时通过元信息中返回的 `ext` 字段告知石墨 SDK 此文件的扩展名。例如 `docx` 识别为 Word 文件。

- `ext` 字段不带 `.` ，例如文件为 `测试文件.docx` ，应返回 `docx`。

- 出于安全性考虑，2021-12-01 之后开始，预览时获取文件信息需要返回 `permissions` 信息用于验证当前用户是否具有相应权限。

:::

**Response Body Example**

```json
{
  "id": "ba13551165cc5066",
  "name": "示例文档.docx",
  "type": "file",
  "permissions": {
    "readable": true
  },
  "downloadUrl": "http://example.com/download/test.docx", // 文件预览时用到，协同文档留空
  "ext": "docx" // 若下载地址返回的 Content-Type 或 Content-Disposition 可识别则此字段可选
}
```

>注：石墨服务器会下载`downloadUrl`中的文件进行解析。可能需要设置指定的 Header 供石墨来选择以何种方式解析。更多细节可以参考 [Header 要求](https://sdk.shimo.im/v2/docs/roll#http-response-header-%E8%A6%81%E6%B1%82)。
## 获取文件的协作者列表{#file-collborators}

用于获取此协作文件的协作者（接入方可自行设置哪些用户为协作者）用户列表信息，如评论实时同步给其他用户时用于获取协作者列表发送评论事件

:::caution 说明

在表格锁定功能获取协作者列表时接入方必须指明返回的用户是否为 `管理者`，用于在锁定面板中展示，管理者无法被其他人锁定。未正确返回此字段可能导致不同用户看到的协作者列表中管理者不一致。

:::

**请求地址**

*GET* /files/{fileId}/collaborators

**HTTP Request Headers**

|**Header 名**|**值**|**说明**|
|:----|:----|:----|
|X-Shimo-Token|接入方提供的 Token|用于接入方对本次请求鉴权|

**HTTP Response Body**

|字段名|类型|值示例|说明|
|:----|:----|:----|:----|
|items[0].id|string|userid123|接入方系统的用户的 ID|
|items[0].name|string|张三|接入方系统的用户名称|
|items[0].avatar|string|[http://fake.site/user-123.png](http://fake.site/user-123.png)|接入方系统的用户的头像地址|
|items[0].email|string|[user123@fake.site](mailto:user123@fake.site)|接入方系统的用户的邮箱|
|items[0].isManager|boolean|true|该用户是否为此文件的管理者|

**Response Body****Example**

```json
[
  {
    "id": "user1",
    "name": "testuser1",
    "avatar": "http://your-domain/avatar-url",
    "email": "xxx@shimo.im",
    "isManager": true  // 指定此用户是否此文件的管理者
  },
  {
    "id": "user2",
    "name": "testuser2",
    "avatar": "http://your-domain/avatar-url",
    "email": "xxx2@shimo.im",
    "isManager": false  // 指定此用户是否此文件的管理者
  }
]
```

## 获取接入方指定文件的完整访问地址{#file-url}

通过接入方文件 ID 获取接入方完整的访问地址，此接口通常为协同文档场景下使用。

如轻文档文件中存在 @ 其他文件的关联，此文件导出为 docx 时，文件中的 @ 关联会转换为石墨 SDK 的跳转链接，当访问此跳转链接时，石墨 SDK 会尝试通过 `fileId` 获取接入方系统的完整访问地址进行重定向。

:::caution 说明

接口由石墨 SDK 主动发起，出于安全考虑，建议对此接口传递的 HTTP Header `X-Shimo-Signature` 进行校验，签名算法与接入方生成签名的方法一致。

:::

**请求地址**

*POST* /files/{fileId}/url

**HTTP Request Headers**

|Header 名|值|说明|
|:----|:----|:----|
|X-Shimo-Signature|-|石墨根据 AppId、AppSecret 以及 fileId 签名出的 Signature 字符串，用于接入方校验请求来源合法性|

**HTTP Request Body**

无

**Response Body****Example**

```plain
{
  "url": "https://your-server-domain/path/to/url"
}
```

## 获取文件元信息-协同文档自动任务{#file-meta-admin}

自动任务获取基本信息（目前只有跨表格引用任务）时会请求此接口。

:::caution 说明

接口由石墨 SDK 主动发起，出于安全考虑，建议对此接口传递的 HTTP Header `X-Shimo-Signature` 进行校验，签名算法与接入方生成签名的方法一致。

:::

**请求地址**

*GET* /admin/files/{fileId}

**HTTP Request Headers**

|Header 名|值|说明|
|:----|:----|:----|
|X-Shimo-Credential-Type|3|无法提供 token 的 [回调凭证类型](./../04service-callback/overview.md#credential-types)|
|X-Shimo-Signature|-|石墨根据服务商的 AppId 以及 Secret 信息签名的Signature，payload 中包含 `fileId` ，若与请求参数中不一致说明非合法请求|

**HTTP Response Body**

|字段名|类型|值示例|说明|
|:----|:----|:----|:----|
|id|string|123|接入方系统中的文档 ID|
|name|string|file title|文档标题|
|type|string|documentPro|文档类型列表参考 [创建文档](./../06API-document/interface-description/collaborative-editing.md#create-collab-file)|
|creatorId|string|1|接入方文件的创建者用户 ID|
|createdAt|string|2021-08-01T00:00:00Z|接入方记录的文件创建时间，`UTC` 时间（0 时区）， 格式为 `2022-01-01T09:00:01Z`|
|updatedAt|string|2021-08-02T00:00:00Z|接入方记录的文件最后更新时间，UTC 时间（0 时区）， 格式为 `2022-01-01T09:00:01Z`|
|views|number|100|接入方统计的文件阅读次数，用于在文档信息中展示阅读次数。若接入方未返回，则默认显示为 1|
|permissions|object| - |请参考 [文件权限说明](#file-permission)，`readable` 需为 `true`|

**Response Body Example**

```json
{
  "id": "ba13551165cc5066",
  "name": "无标题表格",
  "type": "spreadsheet",
  "permissions": {
    "commentable": true,
    "editable": true,
    "readable": true,
    "copyable": true,
    "exportable": true
  },
  "views": 100,
  "creatorId": "1",
  "createdAt": "2021-08-01T00:00:00Z", // UTC 时间，0 时区，在北京时间基础上减 8 小时
  "updatedAt": "2021-08-02T00:00:00Z" // UTC 时间，0 时区，在北京时间基础上减 8 小时
}
```

## 根据指定用户获取文件元信息-协同文档{#file-meta-by-uid}

石墨 SDK 需要主动获取文件基本信息（目前只有跨表格引用场景）时会请求此接口。

:::caution 说明

接口由石墨 SDK 主动发起，出于安全考虑，建议对此接口传递的 HTTP Header `X-Shimo-Signature` 进行校验，签名算法与接入方生成签名的方法一致。

:::

**请求地址**

*GET* /admin/files/{fileId}/by-user-id

**HTTP Request Headers**

|Header 名|值|说明|
|:----|:----|:----|
|X-Shimo-Credential-Type|3|无法提供 token 的 [回调凭证类型](./../04service-callback/overview.md#credential-types)|
|X-Shimo-Signature|-|石墨根据服务商的 AppId 以及 AppSecret 信息签名的 Signature，Payload 中包含 `fileId`，`userId`。若与请求参数中不一致说明非合法请求|

**HTTP Query Parameters**

|参数名|值|说明|
|:----|:----|:----|
|userId|user12345|表示当前查询需要获取服务商的某个用户 ID 的文件权限|

**HTTP Response Body**

|字段名|类型|值示例|说明|
|:----|:----|:----|:----|
|id|string|123|接入方系统中的文档 ID|
|name|string|file title|文档标题|
|type|string|documentPro|文档类型列表参考 [创建文档](./../06API-document/interface-description/collaborative-editing.md#create-collab-file)|
|creatorId|string|1|接入方文件的创建者用户 ID|
|createdAt|string|2021-08-01T00:00:00Z|接入方记录的文件创建时间，`UTC` 时间（0 时区）， 格式为 `2022-01-01T09:00:01Z`|
|updatedAt|string|2021-08-02T00:00:00Z|接入方记录的文件最后更新时间，`UTC` 时间（0 时区）， 格式为 `2022-01-01T09:00:01Z`|
|views|number|100|接入方统计的文件阅读次数，用于在文档信息中展示阅读次数。若接入方未返回，则默认显示为 1|
|permissions|object|    |请参考 [文件权限说明](#file-permission)，`readable` 需为 `true`|

**Response Body Example**

```json
{
  "id": "ba13551165cc5066",
  "name": "无标题表格",
  "type": "spreadsheet",
  "permissions": {
    "commentable": true,
    "editable": true,
    "readable": true,
    "copyable": true,
    "exportable": true
  },
  "views": 100,
  "creatorId": "1",
  "createdAt": "2021-08-01T00:00:00Z", // UTC 时间，0 时区，在北京时间基础上减 8 小时
  "updatedAt": "2021-08-02T00:00:00Z" // UTC 时间，0 时区，在北京时间基础上减 8 小时
}
```

