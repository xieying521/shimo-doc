---
sidebar_position: 6
toc_max_heading_level: 4
---

# 接入方实现接口

:::caution
⚠️ 如果没有特别说明，所有请求均为 `HTTP`，请求头和返回头的 `Content-Type` 均为 `application/json`。

⚠️ 接口里的 `{xxxx}` 表示需要用户在接口地址里使用自己的值来进行替换，比如 `/users/{userId}` 的真实调用地址可能是 `/users/123`，在接口描述里会将此参数以 `userId` 的形式进行描述。

⚠️ /admin 前缀的接口，主要由用户动作产生的间接请求，可能无法携带用户身份或者用户身份无关，如：用户A 创建了表格1引用了表格2并分享给用户B，当用户B访问表格1时，需要检查用户A是否还有权限访问表格2。

:::

## 概览

接入方在使用石墨 SDK 协同功能时，需要按需实现以下回调接口，如下表中为当前接入石墨 SDK 时已知场景下需要实现接口的完整列表以及简要说明。

若接入方仅关心**部分功能**，可参考 [常见功能场景相关接口清单](#features-api-list) 部分。

| 接口                                                                | 接口用途                                    | 说明                                                                                                                                                                          |
|:--------------------------------------------------------------------|:----------------------------------------|:----------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `GET` [/files](#list-files)                                         | 获取当前用户的文件列表                      | 添加跨表格公式、合并工作表等选择用户最近使用的文件列表会请求此接口                                                                                                             |
| `GET` [/files/{fileId}](#file-meta-collab)                          | 获取文件元信息-协同文档                     | 获取基本信息、权限等，创建、导入、编辑会请求此接口                                                                                                                                |
| `GET` [/files/{fileId}](#file-meta-preview)                         | 获取文件元信息-文件预览                     | 获取基本信息、下载地址，预览文件时会请求此接口                                                                                                                                  |
| `GET` [/files/{fileId}/collaborators](#file-collborators)           | 获取当前的协作用户                          | 用于获取此协作文件的协作者（接入方可自己控制哪些用户为协作者）用户列表信息，如评论实时同步给其他用户时用于获取协作者列表发送评论事件                                             |
| `POST` [/files/{fileId}/url](#file-url)                             | 获取接入方指定文件的完整访问地址            | 在文件中 @ 其他文件时，在将此文件导出后的文件中会保留一个石墨 SDK 的跳转链接地址，打开链接时，会通过此接口获取当前完整的接入方访问链接，获取后会重定向到接入方文件地址            |
| `GET` [/admin/files/{fileId}](#file-meta-admin)                     | 获取文件元信息-协同文档自动任务             | 自动任务获取基本信息（目前只有跨表格引用任务）时会请求此接口                                                                                                                    |
| `GET` [/admin/files/{fileId}/by-user-id](#file-meta-by-uid)         | 根据指定用户获取文件元信息-协同文档自动任务 | 石墨 SDK 需要主动获取文件基本信息（目前只有跨表格引用场景）时会请求此接口                                                                                                       |
| `GET` [/users/current/info](#user-current)                          | 获取当前用户信息                            | 获取当前用户的 ID、名称、头像图片地址等信息，创建、编辑、导入时需使用此接口获取用户身份信息                                                                                        |
| `GET` [/users/current/team](#user-current-team)                     | 获取当前用户所在团队信息                    | 获取当前用户所在团队的 ID、名称、团队成员总数                                                                                                                                   |
| `GET` [/users/{userId}](#user-by-id)                                | 获取用户信息                                | 提供用户的基本信息，ID、名称、头像图片地址等，在获取协作场景下的指定用户信息时会请求此接口                                                                                        |
| `GET` [/users/{userId}/watermark](#user-watermark)                  | 获取用户水印信息                            | 获取指定用户的水印信息，可根据自身业务系统需要获取返回需要的水印字段或返回空                                                                                                   |
| `GET` [/users/{userId}/department-paths](#user-department-paths)    | 获取用户部门路径                            | 获取指定用户的部门路径，可根据自身业务系统需要获取返回需要的部门路径或返回空                                                                                                   |
| `POST` [/users/batch/get](#users-by-ids)                            | 批量获取用户信息                            | 提供多个指定用户的信息，ID、名称、头像图片地址等。如多人协作时，侧边栏历史展示时需要展示多个用户的相关信息，此时会请求此接口                                                        |
| `GET` [/teams/{teamId}/members](#team-members)                      | 获取团队下的成员列表                        | 获取指定团队的成员列表，根据分页信息查询列表结果                                                                                                                               |
| `GET` [/departments/{departmentId}](#department-info)               | 获取部门信息                                | 获取指定部门的信息，包括部门 ID、名称、部门成员总数                                                                                                                              |
| `GET` [/departments/{departmentId}/children](#children-departments) | 获取部门的下级部门节点                      | 获取指定部门的下级部门节点，包括部门信息 ID、名称、部门成员总数                                                                                                                  |
| `GET` [/departments/{departmentId}/members](#department-members)    | 获取部门下的成员分页列表                    | 获取指定部门下的成员分页列表，获取当前分页的用户信息列表以及总成员数                                                                                                           |
| `GET` [/search/users/recent](#users-recent)                         | 获取与文件相关的用户列表                    | 用于在文件中输入 @ 时，为展示下拉菜单，选择提及的用户，由于此时没有输入关键字，接入方可根据自己系统情况返回，如：和当前用户最相关的用户列表、和当前文件最相关的用户列表、或返回空数组 |
| `GET` [/search/files/recent](#files-recent)                         | 获取与文件相关的文件列表                    | 用于在文件中输入 @ 时，展示下拉菜单，选择提及的文件，由于此时没有输入关键字，接入方可根据自己系统情况返回，如：和当前用户最相关的文件列表、和当前文件最相关的文件列表、或返回空数组   |
| `POST` [/search](#search-by-keyword)                                | 根据关键字搜索文件或用户                    | 用于输入 @ 时，如需要根据关键字，通过请求接入方系统返回对应的文件列表、用户列表，如不需要可将对应字段返回空数组。表格锁定添加人员时涉及此接口                                      |
| `POST` [/events](#events)                                           | 推送石墨 SDK 相关事件                       | 由服务商选择性处理需要关心的事件数据。如当有人对评论、讨论、提及等进行操作时，部分操作会触发此事件                                                                                |


## 常见功能场景相关接口清单 {#features-api-list}

##### 使用协同文档进行编辑时
  - `GET` [/files/{fileId}](#file-meta-collab) 获取元信息-协同文档
  - `GET` [/users/current/info](#user-current) 获取当前用户信息
  - `GET` [/users/{userId}](#user-by-id) 获取用户信息
  - `POST` [/users/batch/get](#users-by-ids) 批量获取用户信息
  - `GET` [/files/{fileId}/collaborators](#file-collborators) 获取当前的协作用户
  - `GET` [/users/{userId}/watermark](#user-watermark)  获取用户水印信息
##### 使用在协同文档的 `内容` 、`评论`、`讨论` 中提及(@)人、提及(@)文件功能时 {#feature-mention-at}
  - `GET` [/search/users/recent](#users-recent) 获取与文件相关的用户列表
  - `GET` [/search/files/recent](#files-recent) 获取与文件相关的文件列表
  - `POST` [/search](#search-by-keyword)  根据关键字搜索文件或用户
##### 使用协同文档 `导入` 或 `导出` 功能时
  - `GET` [/files/{fileId}](#file-meta-collab) 获取元信息-协同文档
  - `GET` [/users/current/info](#user-current) 获取当前用户信息
  - `POST` [/files/{fileId}/url](#file-url) 获取接入方指定文件的完整访问地址
##### 使用专业表格跨表格公式、合并工作表时 {#feature-sheets}
:::tip 提示

由于跨表格、合并工作表仅针对**表格**文件，因此搜索文件列表的接口会过滤出 `type: spreadsheet` 类型文件，若无此类型文件，列表可能为空。

:::
  - `GET` [/files](#list-files) 获取当前用户的文件列表
  - `GET` [/search/files/recent](#files-recent) 获取与文件相关的文件列表
  - `GET` [/admin/files/{fileId}/by-user-id](#file-meta-by-uid) 根据指定用户获取文件元信息-协同文档自动任务
  - `GET` [/admin/files/{fileId}](#file-meta-admin) 获取文件元信息-协同文档自动任务
##### 使用专业表格锁定功能时
  - `GET` [/files/{fileId}/collaborators](#file-collborators) 获取当前的协作用户
  - `POST` [/search](#search-by-keyword) 根据关键字搜索文件或用户
##### 第三方文件预览
  - `GET` [/files/{fileId}](#file-meta-preview) 获取文件元信息-文件预览
  - `GET` [/users/current/info](#user-current) 获取当前用户信息
##### 需要通过石墨协同文档事件定制业务系统功能时
  - `POST` [/events](#events) 推送石墨 SDK 相关事件
##### 在提及 at 或表格锁定时，选择部门或部门成员 {#feature-team-department}
相关概念参考 [团队与部门](./concepts/team_department.md)
  - `GET` [/users/current/info](#user-current) 获取当前用户信息
  - `GET` [/users/current/team](#user-current-team) 获取当前用户所在团队信息
  - `GET` [/users/{userId}/department-paths](#user-department-paths) 获取用户部门路径
  - `GET` [/teams/{teamId}/members](#team-members) 获取团队下的成员列表
  - `GET` [/departments/{departmentId}](#department-info) 获取部门信息
  - `GET` [/departments/{departmentId}/children](#children-departments) 获取部门的下级部门节点
  - `GET` [/departments/{departmentId}/members](#department-members) 获取部门下的成员分页列表
  - `POST` [/search](#search-by-keyword) 根据关键字搜索文件、用户、团队成员、部门

## 文件

### 基础概念

#### 文件权限 {#file-permissions}

Permissions 字段

| 字段名      | 类型    | 说明                                  |
|:------------|:--------|:------------------------------------|
| commentable | boolean | 该用户是否可以进行评论                |
| editable    | boolean | 该用户是否可以进行编辑                |
| readable    | boolean | 该用户是否可以可以访问该文档          |
| copyable    | boolean | 该用户是否可以可以复制文档内容。`editable` 为 `true` 时，`copyable` 一定为 `true`。 |
| exportable  | boolean | 该用户是否可以进行导出                |
| manageable  | boolean | 该用户是否可以管理文档，如进行删除操作 |

### 获取当前用户的文件列表 {#list-files}

添加跨表格公式、合并工作表等选择用户最近使用的文件列表会请求此接口

**请求地址**

_GET_ /files

**HTTP Request Headers**

| Header 名     | 值 | 说明                                        |
|:--------------|:---|:------------------------------------------|
| X-Shimo-Token |    | 服务商提供的 token，用于服务商对该次请求鉴权 |

**HTTP Query Parameters**

| 字段名  | 类型   | 值示例     | 说明                                                             |
|:--------|:-------|:-----------|:---------------------------------------------------------------|
| limit   | number | 123        | 最大条目数 ID                                                    |
| orderBy | string | created_at | 结果按文件创建时间或最后修改时间排序，可选created_at、  updated_at |

**HTTP Response Body**

| 字段名               | 类型   | 值示例               | 说明                                                                                              |
|:---------------------|:-------|:---------------------|:------------------------------------------------------------------------------------------------|
| items[0].id          | string | ba13551165cc5066     | 服务商系统中的文档 ID                                                                             |
| items[0].name        | string | file title           | 文档标题                                                                                          |
| items[0].type        | string | documentPro          | 文档类型列表参考[创建文档](./apis.md#create-collab-file)，若为第三方存储的非石墨协作文件，则固定传 `file` |
| items[0].creatorId   | string | 1                    | 接入方文件的创建者用户 ID                                                                         |
| items[0].createdAt   | string | 2021-08-01T00:00:00Z | 接入方记录的文件创建时间，`UTC` 时间（0 时区）， 格式为 `2022-01-01T09:00:01Z`                        |
| items[0].updatedAt   | string | 2021-08-02T00:00:00Z | 接入方记录的文件最后更新时间，`UTC` 时间（0 时区）， 格式为 `2022-01-01T09:00:01Z`                    |
| items[0].views       | number | 100                  | 接入方统计的文件阅读次数，用于在文档信息中展示阅读次数。若接入方未返回，则默认显示为 1               |
| items[0].permissions | object |                      | [用户对文件权限信息](#file-permissions)                                                                   |

Response Example

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
  ​},
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
  ​},
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
  ​},
]
```

### 获取文件元信息-协同文档 {#file-meta-collab}

获取基本信息、权限等，创建、导入、编辑会请求此接口

**请求地址**

_GET_ /files/{fileId}

**HTTP Request Headers**

| Header 名     | 值 | 说明                                        |
|:--------------|:---|:------------------------------------------|
| X-Shimo-Token |    | 服务商提供的 token，用于服务商对该次请求鉴权 |

**HTTP Response Body**

| 字段名      | 类型   | 值示例               | 说明                                                                                |
|:------------|:-------|:---------------------|:----------------------------------------------------------------------------------|
| id          | string | 123                  | 服务商系统中的文档 ID                                                               |
| name        | string | file title           | 文档标题                                                                            |
| type        | string | documentPro          | 文档类型列表参考[创建文档](./apis.md#create-collab-file)                                  |
| creatorId   | string | 1                    | 接入方文件的创建者用户 ID                                                           |
| createdAt   | string | 2021-08-01T00:00:00Z | 接入方记录的文件创建时间，`UTC` 时间（0 时区）， 格式为 `2022-01-01T09:00:01Z`          |
| updatedAt   | string | 2021-08-02T00:00:00Z | 接入方记录的文件最后更新时间，`UTC` 时间（0 时区）， 格式为 `2022-01-01T09:00:01Z`      |
| views       | number | 100                  | 接入方统计的文件阅读次数，用于在文档信息中展示阅读次数。若接入方未返回，则默认显示为 1 |
| permissions | object |                      | [用户对文件权限信息](#file-permissions)                                                      |
| teamGuid    | string | 123                  | 接入方文件所属团队 ID，非必需。若返回此字段，则 user.teamGuid 字段也应实现并返回                    |

:::caution
**`manageable` 权限说明**

⚠️ 建议至少给文件的创建者指定 `manageable: true` 权限，若接入方允许有其他人拥有管理权限，则可自行设计实现。

使用到 `manageable` 权限的场景如下：
- 调用石墨 SDK 删除文件接口时，需要 `manageable` 为 `true`
- 进行表格锁定操作时，若当前用户具有 `manageable` 权限为 `true` 时，则会展示为管理者
- `更新` 或 `删除` 他人创建的版本时，当前用户对于此文件的权限需要 `manageable` 为 `true` 时，才允许操作，否则会被拒绝
- 表格导出时，若当前用户对于此文件权限满足 `manageable` 为 `true` 时，导出的文件中会包含表格锁定的内容，否则导出文件内容不包含锁定内容。
:::

Example

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



### 获取文件元信息-文件预览 {#file-meta-preview}

获取基本信息、下载地址，预览文件时会请求此接口

> ⚠️ 此接口地址与「获取文件元信息-协同文档」相同，但返回结果不同；可能需要接入方自行区分文件是石墨协同文档还是普通文件。

**请求地址**

_GET_ /files/{fileId}

**HTTP Request Headers**

| Header 名     | 值 | 说明                                        |
|:--------------|:---|:------------------------------------------|
| X-Shimo-Token |    | 服务商提供的 token，用于服务商对该次请求鉴权 |

**HTTP Response Body**

| 字段名      | 类型   | 值示例                       | 说明                                                                                                                                                                                                                                               |
|:------------|:-------|:-----------------------------|:-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| id          | string | 123                          | 服务商系统中的文件 ID                                                                                                                                                                                                                              |
| name        | string | file title                   | 文件标题                                                                                                                                                                                                                                           |
| type        | string | file                         | 目前使用 `file` 表示第三方文件类型                                                                                                                                                                                                                 |
| permissions | object |                              | [用户对文件权限信息](#file-permissions)                                                                                                                                                                                                      |
| downloadUrl | string | http://fake.site/sample.docx | 用于预览 Office 文件时下载文件内容                                                                                                                                                                                                                 |
| ext         | string | docx                         | 当石墨请求下载地址 (`downloadUrl`) 返回的 HTTP Response Header 中 `Content-Type` 或 `Content-Disposition` 接入方未返回可识别的信息时，此字段用于标识此文件的扩展名。不需要带 `.`，若文件为 `测试文件.docx` ，则应传 `docx`，若返回 `.docx` 可能识别失败 |

:::caution
⚠️  请求下载时接入方无法返回 `Content-Type` 以及 `Content-Disposition` 信息用于石墨 SDK 识别文件类型，此时通过元信息中返回的 `ext` 字段告知石墨 SDK 此文件的扩展名。例如 `docx` 识别为 Word 文件。

`ext` 字段不带 `.` ，例如文件为 `测试文件.docx` ，应返回 `docx`。
:::

:::caution
⚠️ 出于安全性考虑，2021-12-01 之后开始，预览时获取文件信息需要返回 `permissions` 信息用于验证当前用户是否具有相应权限。
:::

Example

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

> 注：石墨服务器会下载`downloadUrl`中的文件进行解析。可能需要设置指定的 Header 供石墨来选择以何种方式解析。更多细节可以参考 [Header 要求](./roll.md#http-response-header-要求)。


### 获取文件的协作者列表 {#file-collborators}

用于获取此协作文件的协作者（接入方可自己控制哪些用户为协作者）用户列表信息，如评论实时同步给其他用户时用于获取协作者列表发送评论事件

:::caution
⚠️ 在表格锁定功能获取协作者列表时接入方**必须**指明返回的用户是否为 `管理者`，用于在锁定面板中展示，管理者无法被其他人锁定。未正确返回此字段可能导致不同用户看到的协作者列表中管理者不一致。
:::

**请求地址**

_GET_ /files/{fileId}/collaborators

**HTTP Request Headers**

| Header 名     | 值 | 说明                                        |
|:--------------|:---|:------------------------------------------|
| X-Shimo-Token |    | 服务商提供的 token，用于服务商对该次请求鉴权 |

**HTTP Response Body**

| 字段名             | 类型    | 值示例                        | 说明                       |
|:-------------------|:--------|:------------------------------|:-------------------------|
| items[0].id        | string  | userid123                     | 服务商系统的用户的 ID      |
| items[0].name      | string  | 张三                          | 服务商系统的用户名称       |
| items[0].avatar    | string  | http://fake.site/user-123.png | 服务商系统的用户的头像地址 |
| items[0].email     | string  | user123@fake.site             | 服务商系统的用户的邮箱     |
| items[0].isManager | boolean | true                          | 该用户是否为此文件的管理者 |

Response Body Example

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

### 获取接入方指定文件的完整访问地址 {#file-url}

通过接入方文件 ID 获取接入方完整的访问地址，此接口通常为协同文档场景下使用。

如轻文档文件中存在 @ 其他文件的关联，此文件导出为 docx 时，文件中的 @ 关联会转换为石墨 SDK 的跳转链接，当访问此跳转链接时，石墨 SDK 会尝试通过 `fileId` 获取接入方系统的完整访问地址进行重定向。

:::caution
⚠️ 接口由石墨 SDK 主动发起，出于安全考虑，建议对此接口传递的 HTTP Header `X-Shimo-Signature` 进行校验，签名算法与接入方生成签名的方法一致。
:::

**请求地址**

_POST_ /files/{fileId}/url

**HTTP Request Headers**

| Header 名         | 值 | 说明                                                                                        |
|:------------------|:---|:------------------------------------------------------------------------------------------|
| X-Shimo-Signature |    | 石墨根据 AppId、AppSecret 以及 fileId 签名出的 Signature 字符串，用于接入方校验请求来源合法性 |

**HTTP Request Body**

无

**HTTP Response Example**

```json
{
  "url": "https://your-server-domain/path/to/url"
}
```

### 获取文件元信息-协同文档自动任务 {#file-meta-admin}

自动任务获取基本信息（目前只有跨表格引用任务）时会请求此接口。

:::caution
⚠️ 接口由石墨 SDK 主动发起，出于安全考虑，建议对此接口传递的 HTTP Header `X-Shimo-Signature` 进行校验，签名算法与接入方生成签名的方法一致。
:::

**请求地址**

_GET_ /admin/files/{fileId}

**HTTP Request Headers**

| Header 名               | 值 | 说明                                                                                                               |
|:------------------------|:---|:-----------------------------------------------------------------------------------------------------------------|
| X-Shimo-Credential-Type | 3  | 无法提供 token 的 [回调凭证类型](./resources.md##credential-types)                                                 |
| X-Shimo-Signature       |    | 石墨根据服务商的 AppId 以及 Secret 信息签名的Signature，payload 中包含 `fileId` ，若与请求参数中不一致说明非合法请求 |


**HTTP Response Body**

| 字段名      | 类型   | 值示例               | 说明                                                                                |
|:------------|:-------|:---------------------|:----------------------------------------------------------------------------------|
| id          | string | 123                  | 服务商系统中的文档 ID                                                               |
| name        | string | file title           | 文档标题                                                                            |
| type        | string | documentPro          | 文档类型列表参考[创建文档](./apis.md#create-collab-file)                                  |
| creatorId   | string | 1                    | 接入方文件的创建者用户 ID                                                           |
| createdAt   | string | 2021-08-01T00:00:00Z | 接入方记录的文件创建时间，`UTC` 时间（0 时区）， 格式为 `2022-01-01T09:00:01Z`          |
| updatedAt   | string | 2021-08-02T00:00:00Z | 接入方记录的文件最后更新时间，`UTC` 时间（0 时区）， 格式为 `2022-01-01T09:00:01Z`      |
| views       | number | 100                  | 接入方统计的文件阅读次数，用于在文档信息中展示阅读次数。若接入方未返回，则默认显示为 1 |
| permissions | object |                      | [用户对文件权限信息](#file-permissions)，`readable` 需为 `true`                               |

Example

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

### 根据指定用户获取文件元信息-协同文档 {#file-meta-by-uid}

石墨 SDK 需要主动获取文件基本信息（目前只有跨表格引用场景）时会请求此接口

:::caution
⚠️ 接口由石墨 SDK 主动发起，出于安全考虑，建议对此接口传递的 HTTP Header `X-Shimo-Signature` 进行校验，签名算法与接入方生成签名的方法一致。
:::

**请求地址**

_GET_ /admin/files/{fileId}/by-user-id

**HTTP Request Headers**

| Header 名               | 值 | 说明                                                                                                                           |
|:------------------------|:---|:-----------------------------------------------------------------------------------------------------------------------------|
| X-Shimo-Credential-Type | 3  | 无法提供 token 的 [回调凭证类型](./resources.md##credential-types)                                                             |
| X-Shimo-Signature       |    | 石墨根据服务商的 AppId 以及 AppSecret 信息签名的 Signature，Payload 中包含 `fileId`,`userId` 若与请求参数中不一致说明非合法请求 |


**HTTP Query Parameters**

| 参数名 | 值        | 说明                                               |
|:-------|:----------|:-------------------------------------------------|
| userId | user12345 | 表示当前查询需要获取服务商的哪个用户 ID 的文件权限 |

**HTTP Response Body**

| 字段名      | 类型   | 值示例               | 说明                                                                                |
|:------------|:-------|:---------------------|:----------------------------------------------------------------------------------|
| id          | string | 123                  | 服务商系统中的文档 ID                                                               |
| name        | string | file title           | 文档标题                                                                            |
| type        | string | documentPro          | 文档类型列表参考 [创建文档](./apis.md#create-collab-file)                                 |
| creatorId   | string | 1                    | 接入方文件的创建者用户 ID                                                           |
| createdAt   | string | 2021-08-01T00:00:00Z | 接入方记录的文件创建时间，`UTC` 时间（0 时区）， 格式为 `2022-01-01T09:00:01Z`          |
| updatedAt   | string | 2021-08-02T00:00:00Z | 接入方记录的文件最后更新时间，`UTC` 时间（0 时区）， 格式为 `2022-01-01T09:00:01Z`      |
| views       | number | 100                  | 接入方统计的文件阅读次数，用于在文档信息中展示阅读次数。若接入方未返回，则默认显示为 1 |
| permissions | object |                      | [用户对文件权限信息](#file-permissions)，`readable` 需为 `true`                                          |

Example

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



## 用户

### 获取当前用户信息 {#user-current}

获取当前用户的 ID、名称、头像图片地址等信息，创建、编辑、导入时需使用此接口获取用户身份信息

**请求地址**

_GET_ /users/current/info

**HTTP Request Headers**

| Header 名     | 值 | 说明                                        |
|:--------------|:---|:------------------------------------------|
| X-Shimo-Token |    | 服务商提供的 token，用于服务商对该次请求鉴权 |

**HTTP Response Body**

| 字段名   | 类型   | 值示例                        | 说明                          |
|:---------|:-------|:------------------------------|:----------------------------|
| id       | string | userid123                     | 服务商系统的用户的 ID         |
| name     | string | 张三                          | 服务商系统的用户名称          |
| avatar   | string | http://fake.site/user-123.png | 服务商系统的用户的头像地址    |
| email    | string | user123@fake.site             | 服务商系统的用户的邮箱        |
| teamGuid | string | 123                           | 服务商系统的用户的所属团队 ID，返回时需和 file.teamGuid 同时实现 |

Response Body Example

```json
{
  "id": "userid123",
  "name": "张三",
  "avatar": "http://fake.site/user-123.png",
  "email": "user123@fake.site",
  "teamGuid": "123"
}
```

### 获取当前用户所在团队信息 {#user-current-team}

获取当前用户所在团队信息

**请求地址**

_GET_ /users/current/team

**HTTP Request Headers**

| Header 名     | 值 | 说明                                        |
|:--------------|:---|:------------------------------------------|
| X-Shimo-Token |    | 服务商提供的 token，用于服务商对该次请求鉴权 |

**HTTP Response Body**

| 字段名      | 类型   | 值示例          | 说明                           |
|:------------|:-------|:----------------|:-----------------------------|
| id          | string | 123             | 服务商系统的团队的 ID          |
| name        | string | XXX公司效率团队 | 服务商系统的团队的名称         |
| memberCount | number | 99              | 服务商系统的团队的所有成员数量 |

Response Body Example

```json
{
  "id": "123",
  "name": "XXX公司效率团队",
  "memberCount": 99
}
```

### 获取指定用户信息 {#user-by-id}

提供用户的基本信息，ID、名称、头像图片地址等，在获取协作场景下的指定用户信息时会请求此接口

**请求地址**

_GET_ /users/{userId}

**HTTP Request Headers**

| Header 名     | 值 | 说明                                        |
|:--------------|:---|:------------------------------------------|
| X-Shimo-Token |    | 服务商提供的 token，用于服务商对该次请求鉴权 |

**HTTP Response Body**

| 字段名 | 类型   | 值示例                        | 说明                       |
|:-------|:-------|:------------------------------|:-------------------------|
| id     | string | userid123                     | 服务商系统的用户的 ID      |
| name   | string | 张三                          | 服务商系统的用户名称       |
| avatar | string | http://fake.site/user-123.png | 服务商系统的用户的头像地址 |
| email  | string | user123@fake.site             | 服务商系统的用户的邮箱     |

Response Body Example

```json
{
  "id": "userid123",
  "name": "张三",
  "avatar": "http://fake.site/user-123.png",
  "email": "user123@fake.site"
}
```

### 获取用户水印信息 {#user-watermark}

在用户打开或者预览文档时，会尝试获取当前用户的水印信息用于在页面上展示，接入方可选择返回业务中相应信息作为水印字段信息，也可按照格式返回空，页面上将隐藏水印信息。

**请求地址**

_GET_ /users/{userId}/watermark

**HTTP Request Headers**

| Header 名     | 值 | 说明                                        |
|:--------------|:---|:------------------------------------------|
| X-Shimo-Token |    | 服务商提供的 token，用于服务商对该次请求鉴权 |

**HTTP Response Body**

| 字段名     | 类型  | 值示例                                             | 说明                                                                                                                        |
|:-----------|:------|:------------------------------------------------|:--------------------------------------------------------------------------------------------------------------------------|
| watermarks | Array | ["自定义字段1", "自定义字段2", "自定义字段3", ...] | 服务商返回的水印字段，以字符串数组的形式返回，每个字符串代表一行。返回**空数组**时，石墨编辑器将**不会**在编辑器上展示水印信息。 |

:::tip
**水印字段的内容建议**

由于水印字段目前使用字符串传递，目前未限制此字段长度，为了更好的水印体验，建议返回字段遵循如下规则：
1. `字段数量`不超过 `3` 个，即`水印内容`不超过 `3` 行
2. `每个字段`不超过 `20` 个字，即`水印单行`内容不超过 `20` 字

若确实有需要，可按照自身需求调整`行数`或`单行字数`
:::

Response Body Example

```json title="返回自定义水印信息用于展示"
{
  "watermarks": [ // 结果为字符串数组，按顺序每个单独字符串为一行显示
    "张三",
    "userid",
    "XX团队-部门-子部门"
  ]
}
```

```json title="返回空信息用于隐藏对应用户水印信息"
{
  "watermarks": [] // 空数组
}
```

### 批量获取用户信息 {#users-by-ids}

提供多个指定用户的信息，ID、名称、头像图片地址等。如多人协作时，侧边栏历史展示时需要展示多个用户的相关信息，此时会请求此接口

**请求地址**

_POST_ /users/batch/get

**HTTP Request Headers**

| Header 名     | 值 | 说明                                        |
|:--------------|:---|:------------------------------------------|
| X-Shimo-Token |    | 服务商提供的 token，用于服务商对该次请求鉴权 |

**HTTP Request Body**

| 字段名 | 类型     | 值示例            | 说明         |
|:-------|:---------|:------------------|:-----------|
| ids    | []string | ["user1","user2"] | 用户 ID 列表 |

**HTTP Response Body**

| 字段名          | 类型   | 值示例                        | 说明                       |
|:----------------|:-------|:------------------------------|:-------------------------|
| items[0].id     | string | userid123                     | 服务商系统的用户的 ID      |
| items[0].name   | string | 张三                          | 服务商系统的用户名称       |
| items[0].avatar | string | http://fake.site/user-123.png | 服务商系统的用户的头像地址 |
| items[0].email  | string | user123@fake.site             | 服务商系统的用户的邮箱     |

Response Body Example

```json
[
  {
    "id": "userid123",
    "name": "张三",
    "avatar": "http://fake.site/user-123.png",
    "email": "user123@fake.site"
  },
  {
    "id": "userid1234",
    "name": "李四",
    "avatar": "http://fake.site/user-123.png",
    "email": "user123@fake.site"
  }
]
```

### 获取用户部门路径 {#user-department-paths}

获取指定用户的的部门路径，若用户尚未分配部门，则返回空数组

:::info 备注

**部门路径**

是指 **从用户所属部门** 到 **一级部门** 的路径节点列表

:::

例如：

```json title="用户所在部门结构示例"
XX 公司效率团队      // 团队层级
└── XX 研发部       // 一级部门
    └── 基础设置组
        └── 后端组  // 用户所属部门
```

```json title="该用户应返回的部门路径示例"
[
  "XX 研发部",
  "基础设置组",
  "后端组"
]
```

**请求地址**

_GET_ /users/{userId}/department-paths

**HTTP Request Headers**

| Header 名     | 值 | 说明                                        |
|:--------------|:---|:------------------------------------------|
| X-Shimo-Token |    | 服务商提供的 token，用于服务商对该次请求鉴权 |

**HTTP Response Body**

| 字段名          | 类型   | 值示例    | 说明                   |
|:----------------|:-------|:----------|:---------------------|
| list[0][0].id   | string | 123       | 服务商系统的部门的 ID  |
| list[0][0].name | string | XX 研发部 | 服务商系统的部门的名称 |

**Response Body Example**

```json title="用户仅属于一个部门"
[
  [
    {"id": "123", "name": "XX 研发部"},
    {"id": "456", "name": "基础设施组"},
    {"id": "789", "name": "后端组"}
  ]
]
```

```json title="用户仅属于多个部门"
[
  [
    {"id": "123", "name": "XX 研发部"},
    {"id": "456", "name": "基础设施组"},
    {"id": "1789", "name": "后端组"}
  ],
  [
    {"id": "123", "name": "XX 研发部"},
    {"id": "456", "name": "基础设施组"},
    {"id": "2789", "name": "前端端组"}
  ]
]
```

## 团队和部门

### 获取团队下的成员列表 {#team-members}

**请求地址**

_GET_ /teams/{teamGuid}/members?pagination=true&page={page}&pageSize={pageSize}

**HTTP Request Headers**

| Header 名     | 值 | 说明                                        |
|:--------------|:---|:------------------------------------------|
| X-Shimo-Token |    | 服务商提供的 token，用于服务商对该次请求鉴权 |

**HTTP Query Parameters**

| 参数名     | 类型    | 值   | 说明           |
|:-----------|:--------|:-----|:-------------|
| pagination | boolean | true | 是否为分页查询 |
| page       | number  | 1    | 分页查询第几页 |
| pageSize   | number  | 10   | 分页大小       |

**HTTP Response Body**

| 字段名          | 类型   | 值示例                        | 说明                       |
|:----------------|:-------|:------------------------------|:-------------------------|
| items[0].id     | string | userid123                     | 服务商系统的用户的 ID      |
| items[0].name   | string | 张三                          | 服务商系统的用户名称       |
| items[0].avatar | string | http://fake.site/user-123.png | 服务商系统的用户的头像地址 |
| items[0].email  | string | user123@fake.site             | 服务商系统的用户的邮箱     |

**Response Body Example**

```json
[
  {
    "id": "userid123",
    "name": "张三",
    "avatar": "http://fake.site/user-123.png",
    "email": "user123@fake.site"
  },
  {
    "id": "userid1234",
    "name": "李四",
    "avatar": "http://fake.site/user-123.png",
    "email": "user123@fake.site"
  }
]
```

### 获取部门信息 {#department-info}

:::caution 提示

当部门 ID 为根节点 ID 时，以特殊形式传递部门 ID ，参考 **[`特殊部门 ID 说明`](./concepts/team_department.md#depatment-id-tips)**。

:::

**请求地址**

_GET_ /departments/{departmentId}

**HTTP Request Headers**

| Header 名     | 值 | 说明                                        |
|:--------------|:---|:------------------------------------------|
| X-Shimo-Token |    | 服务商提供的 token，用于服务商对该次请求鉴权 |

**HTTP Response Body**

| 字段名         | 类型   | 值示例 | 说明                           |
|:---------------|:-------|:-------|:-----------------------------|
| id             | string | 123    | 服务商系统的部门 ID            |
| name           | string | 研发部 | 服务商系统的部门名称           |
| allMemberCount | number | 99     | 服务商当前部门下的所有成员数量 |

**Response Body Example**

```json
{
  "id": "123",
  "name": "研发部",
  "allMemberCount": 99
}
```

### 获取部门的下级部门节点 {#children-departments}

:::caution 提示

当部门 ID 为根节点 ID 时，以特殊形式传递部门 ID ，参考 **[`特殊部门 ID 说明`](./concepts/team_department.md#depatment-id-tips)**。

:::

**请求地址**

_GET_ /departments/{departmentId}/children

**HTTP Request Headers**

| Header 名     | 值 | 说明                                        |
|:--------------|:---|:------------------------------------------|
| X-Shimo-Token |    | 服务商提供的 token，用于服务商对该次请求鉴权 |

**HTTP Response Body**

| 字段名                  | 类型   | 值示例 | 说明                           |
|:------------------------|:-------|:-------|:-----------------------------|
| items[0].id             | string | 123    | 服务商系统的部门 ID            |
| items[0].name           | string | 研发部 | 服务商系统的部门名称           |
| items[0].allMemberCount | number | 99     | 服务商当前部门下的所有成员数量 |

**Response Body Example**

```json
[
  {
    "id": "123",
    "name": "研发部",
    "allMemberCount": 99
  },
  {
    "id": "124",
    "name": "财务部",
    "allMemberCount": 10
  }
]
```

### 获取部门下的成员分页列表 {#department-members}

:::caution 提示

当部门 ID 为根节点 ID 时，以特殊形式传递部门 ID ，参考 **[`特殊部门 ID 说明`](./concepts/team_department.md#depatment-id-tips)**。

:::

**请求地址**

_GET_ /departments/{departmentId}/members?page={page}&pageSize={pageSize}

**HTTP Request Headers**

| Header 名     | 值 | 说明                                        |
|:--------------|:---|:------------------------------------------|
| X-Shimo-Token |    | 服务商提供的 token，用于服务商对该次请求鉴权 |

**HTTP Query Parameters**

| 参数名   | 类型   | 值 | 说明           |
|:---------|:-------|:---|:-------------|
| page     | number | 1  | 分页查询第几页 |
| pageSize | number | 20 | 分页大小       |

**HTTP Response Body**

| 字段名            | 类型   | 值示例                        | 说明                         |
|:------------------|:-------|:------------------------------|:---------------------------|
| total             | number | 20                            | 服务商系统的部门下的成员总数 |
| members[0].id     | string | userid123                     | 服务商系统的用户的 ID        |
| members[0].name   | string | 张三                          | 服务商系统的用户名称         |
| members[0].avatar | string | http://fake.site/user-123.png | 服务商系统的用户的头像地址   |
| members[0].email  | string | user123@fake.site             | 服务商系统的用户的邮箱       |

**Response Body Example**

```json
{
  "total": 20,
  "members": [
    {
      "id": "userid123",
      "name": "张三",
      "avatar": "http://fake.site/user-123.png",
      "email": "user123@fake.site"
    },
    {
      "id": "userid1234",
      "name": "李四",
      "avatar": "http://fake.site/user-123.png",
      "email": "user123@fake.site"
    }
  ]
}
```

## 搜索

### 获取与文件相关的用户列表 {#users-recent}

此接口用 @ 功能，进行补全使用。
用于在文件中输入 @ 时，展示下拉菜单，选择提及的用户，接入方可根据自己系统情况返回，如：和当前用户最相关的用户列表、和当前文件最相关的用户列表、或返回空数组

**请求地址**

_GET_ /search/users/recent?fileId=file12345

**HTTP Query Parameters**

| 参数名 | 值        | 说明                                |
|:-------|:----------|:----------------------------------|
| fileId | file12345 | 表示当前查询来自服务商的哪个文件 ID |

**HTTP Request Headers**

| Header 名     | 值 | 说明                                        |
|:--------------|:---|:------------------------------------------|
| X-Shimo-Token |    | 服务商提供的 token，用于服务商对该次请求鉴权 |

**HTTP Response Body**

| 字段名          | 类型   | 值示例                        | 说明                       |
|:----------------|:-------|:------------------------------|:-------------------------|
| items[0].id     | string | userid123                     | 服务商系统的用户的 ID      |
| items[0].name   | string | 张三                          | 服务商系统的用户名称       |
| items[0].avatar | string | http://fake.site/user-123.png | 服务商系统的用户的头像地址 |
| items[0].email  | string | user123@fake.site             | 服务商系统的用户的邮箱     |

Response Body Example

```json
[
  {
    "id": "userid123",
    "name": "张三",
    "avatar": "http://fake.site/user-123.png",
    "email": "user123@fake.site"
  },
  {
    "id": "userid456",
    "name": "李四",
    "avatar": "http://fake.site/user-123.png",
    "email": "user456@fake.site"
  }
]
```

### 获取与文件相关的文件列表 {#files-recent}

此接口用 @ 功能，进行补全使用。
用于在文件中输入 @ 时，展示下拉菜单，选择提及的文件，接入方可根据自己系统情况返回，如：和当前用户最相关的文件列表、和当前文件最相关的文件列表、或返回空数组

#### 支持返回 @ 第三方文件

:::tip

由于第三方文件无法进行协作，仅支持在 SDK 中插入 @ 时保留引用，因此需要达到点击后跳转至第三方页面，需要满足以下条件：
1. file 信息 `type` 字段值为 `file`，参考如下 Response Example
2. file 信息增加 `fullUrl` 字段作为跳转至接入方系统的完整地址，参考如下 Response Example
3. 前端使用 `shimo-js-sdk` 时在调用 `connect` 需要实现 `openLink` 方法用于控制编辑器内点击链接时的跳转行为。
:::

**请求地址**

_GET_ /search/files/recent?fileId=file12345

**HTTP Query Parameters**

| 参数名 | 值        | 说明                                |
|:-------|:----------|:----------------------------------|
| fileId | file12345 | 表示当前查询来自服务商的哪个文件 ID |

**HTTP Request Headers**

| Header 名     | 值 | 说明                                        |
|:--------------|:---|:------------------------------------------|
| X-Shimo-Token |    | 服务商提供的 token，用于服务商对该次请求鉴权 |

**HTTP Response Body**

| 字段名               | 类型   | 值示例                                   | 说明                                                                                              |
|:---------------------|:-------|:-----------------------------------------|:------------------------------------------------------------------------------------------------|
| items[0].id          | string | ba13551165cc5066                         | 服务商系统中的文档 ID                                                                             |
| items[0].name        | string | file title                               | 文档标题                                                                                          |
| items[0].type        | string | documentPro                              | 文档类型列表参考[创建文档](./apis.md#create-collab-file)，若为第三方存储的非石墨协作文件，则固定传 `file` |
| items[0].creatorId   | string | 1                                        | 接入方文件的创建者用户 ID                                                                         |
| items[0].createdAt   | string | 2021-08-01T00:00:00Z                     | 接入方记录的文件创建时间，`UTC` 时间（0 时区）， 格式为 `2022-01-01T09:00:01Z`                        |
| items[0].updatedAt   | string | 2021-08-02T00:00:00Z                     | 接入方记录的文件最后更新时间，`UTC` 时间（0 时区）， 格式为 `2022-01-01T09:00:01Z`                    |
| items[0].views       | number | 100                                      | 接入方统计的文件阅读次数，用于在文档信息中展示阅读次数。若接入方未返回，则默认显示为 1               |
| items[0].permissions | object |                                          | [用户对文件权限信息](#file-permissions)                                                                                      |
| items[0].fullUrl     | string | https://customer-system.com/path/to/file | 访问文件的完整地址                                                                                |

Response Body Example

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
  ​},
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
  ​},
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
    "views": 0,
    "fullUrl": "https://customer-system.com/path/to/file" // 仅在文件仅为接入方自身系统的文件而非协同文档时需要传递
  ​}
]
```

### 按关键字搜索文件和用户列表 {#search-by-keyword}

此接口用 @ 功能搜索，获取下拉菜单的结果使用。
用于输入 @ 时，如需要根据关键字搜索用户或文件，通过请求接入方系统返回对应的文件列表、用户列表，如不需要可将对应字段返回空数组

#### 支持返回 @ 第三方文件

:::tip 说明

由于第三方文件无法进行协作，仅支持在 SDK 中插入 @ 时保留引用，因此需要达到点击后跳转至第三方页面，需要满足以下条件：
1. file 信息 `type` 字段值为 `file`，参考如下 Response Example
2. file 信息增加 `fullUrl` 字段作为跳转至接入方系统的完整地址，参考如下 Response Example
3. 前端使用 `shimo-js-sdk` 时在调用 `connect` 需要实现 `openLink` 方法用于控制编辑器内点击链接时的跳转行为。
:::

**请求地址**

_POST_ /search

**HTTP Request Headers**

| Header 名     | 值 | 说明                                        |
|:--------------|:---|:------------------------------------------|
| X-Shimo-Token |    | 服务商提供的 token，用于服务商对该次请求鉴权 |

**HTTP Request Body**

| 字段名   | 类型   | 值示例                                                       | 说明                                                                                                                             |
|:---------|:-------|:-------------------------------------------------------------|:-------------------------------------------------------------------------------------------------------------------------------|
| fileId   | string | fileid1234                                                   | 服务商系统的文件 ID                                                                                                              |
| keyword  | string | test                                                         | 文件名或者用户名的关键字                                                                                                         |
| page     | number | 0                                                            | 搜索的结果第几页                                                                                                                 |
| pageSize | number | 6                                                            | 搜索结果每页数量                                                                                                                 |
| type     | string | file_name,recent_contact,collaborator,team_member,department | 指定搜索请求需要搜索的结果包括哪些类型，按照传入的类型按需返回不同的搜索结果。通过英文逗号 `,` 分割后根据命中的类型返回相应的结果。 |

#### `type` 字段说明

:::tip 说明

- `file_name` 表示通过关键字搜索文件，对应 Response 中 `files` 结果。
- `recent_contact` 表示通过关键字搜索比较相关用户，对应 Response 中 `recentUsers` 结果。
- `collaborator` 表示通过关键字搜索协作者用户，对应 Response 中 `collaborators` 结果。
- `team_member` 表示通过关键字搜索团队成员，对应 Response 中 `teamMembers` 结果。
- `department` 表示通过关键字搜索部门，对应 Response 中 `department` 结果。

:::

**HTTP Request Body Example**

```json
{
  "fileId": "file1234",
  "keyword": "test",
  "page": 0,
  "pageSize": 6,
  "type": "file_name,recent_contact,collaborator,team_member,department"
}
```

**HTTP Response Body**

| 字段名                                          | 类型   | 值示例                                   | 说明                                                                                                                                                                                                                                                 |
|:------------------------------------------------|:-------|:-----------------------------------------|:---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| files.count                                     | number | 3                                        | 搜索到结果的总条数，包括所有分页结果的总和                                                                                                                                                                                                            |
| files.page                                      | number | 0                                        | 当前为第几页，第如第一页的值为 0                                                                                                                                                                                                                      |
| files.pageSize                                  | number | 6                                        | 每一页分页的结果的数量                                                                                                                                                                                                                               |
| files.pageCount                                 | number | 1                                        | 分搜索结果的总条数按照分页大小计算后的总页数                                                                                                                                                                                                         |
| files.results[0].id                             | string | ba13551165cc5066                         | 服务商系统中的文档 ID                                                                                                                                                                                                                                |
| files.results[0].name                           | string | file title                               | 文档标题                                                                                                                                                                                                                                             |
| files.results[0].type                           | string | documentPro                              | 文档类型列表参考[创建文档](./apis.md#create-collab-file)，若为第三方存储的非石墨协作文件，则固定传 `file`                                                                                                                                                    |
| files.results[0].creatorId                      | string | 1                                        | 接入方文件的创建者用户 ID                                                                                                                                                                                                                            |
| files.results[0].createdAt                      | string | 2021-08-01T00:00:00Z                     | 接入方记录的文件创建时间，`UTC` 时间（0 时区）， 格式为 `2022-01-01T09:00:01Z`                                                                                                                                                                           |
| files.results[0].updatedAt                      | string | 2021-08-02T00:00:00Z                     | 接入方记录的文件最后更新时间，`UTC` 时间（0 时区）， 格式为 `2022-01-01T09:00:01Z`                                                                                                                                                                       |
| files.results[0].views                          | number | 100                                      | 接入方统计的文件阅读次数，用于在文档信息中展示阅读次数。若接入方未返回，则默认显示为 1                                                                                                                                                                  |
| files.results[0].permissions                    | object |                                          | [用户对文件权限信息](#file-permissions)                                                                                                                                                                                                                   |
| files.results[0].fullUrl                        | string | https://customer-system.com/path/to/file | 访问文件的完整地址                                                                                                                                                                                                                                   |
| recentUsers.count                               | number | 3                                        | 搜索到结果的总条数，包括所有分页结果的总和                                                                                                                                                                                                            |
| recentUsers.page                                | number | 0                                        | 当前为第几页，第如第一页的值为 0                                                                                                                                                                                                                      |
| recentUsers.pageSize                            | number | 6                                        | 每一页分页的结果的数量                                                                                                                                                                                                                               |
| recentUsers.pageCount                           | number | 1                                        | 分搜索结果的总条数按照分页大小计算后的总页数                                                                                                                                                                                                         |
| recentUsers.results[0].id                       | string | userid123                                | 服务商系统的用户的 ID                                                                                                                                                                                                                                |
| recentUsers.results[0].name                     | string | 张三                                     | 服务商系统的用户名称                                                                                                                                                                                                                                 |
| recentUsers.results[0].avatar                   | string | http://fake.site/user-123.png            | 服务商系统的用户的头像地址                                                                                                                                                                                                                           |
| recentUsers.results[0].email                    | string | user123@fake.site                        | 服务商系统的用户的邮箱                                                                                                                                                                                                                               |
| collaborators.count                             | number | 3                                        | 搜索到结果的总条数，包括所有分页结果的总和                                                                                                                                                                                                            |
| collaborators.page                              | number | 0                                        | 当前为第几页，第如第一页的值为 0                                                                                                                                                                                                                      |
| collaborators.pageSize                          | number | 6                                        | 每一页分页的结果的数量                                                                                                                                                                                                                               |
| collaborators.pageCount                         | number | 1                                        | 分搜索结果的总条数按照分页大小计算后的总页数                                                                                                                                                                                                         |
| collaborators.results[0].id                     | string | userid123                                | 服务商系统的用户的 ID                                                                                                                                                                                                                                |
| collaborators.results[0].name                   | string | 张三                                     | 服务商系统的用户名称                                                                                                                                                                                                                                 |
| collaborators.results[0].avatar                 | string | http://fake.site/user-123.png            | 服务商系统的用户的头像地址                                                                                                                                                                                                                           |
| collaborators.results[0].email                  | string | user123@fake.site                        | 服务商系统的用户的邮箱                                                                                                                                                                                                                               |
| teamMembers.count                               | number | 3                                        | 搜索到结果的总条数，包括所有分页结果的总和                                                                                                                                                                                                            |
| teamMembers.page                                | number | 0                                        | 当前为第几页，第如第一页的值为 0                                                                                                                                                                                                                      |
| teamMembers.pageSize                            | number | 6                                        | 每一页分页的结果的数量                                                                                                                                                                                                                               |
| teamMembers.pageCount                           | number | 1                                        | 分搜索结果的总条数按照分页大小计算后的总页数                                                                                                                                                                                                         |
| teamMembers.results[0].id                       | string | userid123                                | 服务商系统的用户的 ID                                                                                                                                                                                                                                |
| teamMembers.results[0].name                     | string | 张三                                     | 服务商系统的用户名称                                                                                                                                                                                                                                 |
| teamMembers.results[0].avatar                   | string | http://fake.site/user-123.png            | 服务商系统的用户的头像地址                                                                                                                                                                                                                           |
| teamMembers.results[0].email                    | string | user123@fake.site                        | 服务商系统的用户的邮箱                                                                                                                                                                                                                               |
| department.count                                | number | 3                                        | 搜索到结果的总条数，包括所有分页结果的总和                                                                                                                                                                                                            |
| department.page                                 | number | 0                                        | 当前为第几页，第如第一页的值为 0                                                                                                                                                                                                                      |
| department.pageSize                             | number | 6                                        | 每一页分页的结果的数量                                                                                                                                                                                                                               |
| department.pageCount                            | number | 1                                        | 分搜索结果的总条数按照分页大小计算后的总页数                                                                                                                                                                                                         |
| department.results[0].id                        | string | 123                                      | 服务商系统的部门 ID                                                                                                                                                                                                                                  |
| department.results[0].name                      | string | 张三                                     | 服务商系统的部门名称                                                                                                                                                                                                                                 |
| department.results[0].allMemberCount            | number | 10                                       | 服务商系统的当前部门成员总数                                                                                                                                                                                                                         |
| department.results[0].parentDepartments         | Array  |                                          | 服务商系统当前部门数据对应的所有父级部门，顺序 **`从高到底`** ，不包括 `团队` 层级，从一级部门开始，若已是一级部门，则返回空数组。由于在 `表格锁定` 场景搜索部门时可能遇到同名部门，可以通过所有父级部门来帮助区分部门，若无需区分，可选择不返回父级部门列表。 |
| department.results[0].parentDepartments[0].id   | string |                                          | 服务商系统中当前部门父级部门的 ID                                                                                                                                                                                                                    |
| department.results[0].parentDepartments[0].name | name   |                                          | 服务商系统中当前部门父级部门的名称                                                                                                                                                                                                                   |


**HTTP Response Example**

```json
{
  "files": { // 按文件名搜索的文件
    "count": 3,
    "page": 0,
    "pageSize": 6,
    "pageCount": 3,
    "results": [
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
      ​},
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
      ​},
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
        "views": 0,
        "fullUrl": "https://customer-system.com/path/to/file" // 仅在文件仅为接入方自身系统的文件而非协同文档时需要传递
      ​}
    ]
  },
  "recentUsers": { // 最近联系人等
    "count": 2,
    "page": 0,
    "pageSize": 6,
    "pageCount": 2,
    "results": [
      {
        "id": "userid123",
        "name": "张三",
        "avatar": "http://fake.site/user-123.png",
        "email": "user123@fake.site"
      },
      {
        "id": "userid456",
        "name": "李四",
        "avatar": "http://fake.site/user-123.png",
        "email": "userid456@fake.site"
      }
    ]
  },
  "collaborators": { // 协作者
    "count": 2,
    "page": 0,
    "pageSize": 6,
    "pageCount": 1,
    "results": [
      {
        "id": "userid123",
        "name": "张三",
        "avatar": "http://fake.site/user-123.png",
        "email": "user123@fake.site"
      },
      {
        "id": "userid456",
        "name": "李四",
        "avatar": "http://fake.site/user-123.png",
        "email": "userid456@fake.site"
      }
    ]
  },
  "teamMembers": { // 团队成员
    "count": 2,
    "page": 0,
    "pageSize": 6,
    "pageCount": 1,
    "results": [
      {
        "id": "userid123",
        "name": "张三",
        "avatar": "http://fake.site/user-123.png",
        "email": "user123@fake.site"
      },
      {
        "id": "userid456",
        "name": "李四",
        "avatar": "http://fake.site/user-123.png",
        "email": "userid456@fake.site"
      }
    ]
  },
  "department": {
    "count": 2,
    "page": 0,
    "pageSize": 6,
    "pageCount": 1,
    "results": [
      {
        "id": "2",
        "name": "后端组",
        "allMemberCount": 9,
        "parentDepartments": [
          {
            "id": "3",
            "name": "XXX 事业部" // 一级部门
          },
          {
            "id": "4",
            "name": "YYY 产品部"
          }
        ]
      },
      {
        "id": "userid456",
        "name": "基础服务后端组", // 已经是一级部门
        "allMemberCount": 20,
        "parentDepartments": []
      }
    ]
  }
}
```


## 推送石墨 SDK 相关事件 {#events}

此接口用于用户在石墨文件触发特定行为时，向服务商发送通知。

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

_POST_ /events

**HTTP Request Headers**

| Header 名         | 值 | 说明                                        |
|:------------------|:---|:------------------------------------------|
| X-Shimo-Token     |    | 服务商提供的 token，用于服务商对该次请求鉴权 |
| X-Shimo-Sdk-Event |    | 用于识别具体的事件类型                      |

请求参数和请求体根据不同事件有所不同。

### **评论 （Comment）**

在 创建评论、更新评论、删除评论、关闭评论 时，会触发相应事件推送。

**HTTP Request Headers**

| Header 名         | 值      | 说明                   |
|:------------------|:--------|:---------------------|
| X-Shimo-Sdk-Event | Comment | 用于识别具体的事件类型 |

#### 轻文档

##### 添加评论

```json
{
  "kind": "comment",
  "type": "comment", // 预留评论下的小分类，目前就 comment
  "action": "create", // 业务操作
  "fileId": "file1", // 接入方文件 ID
  "userId": "user1", // 接入方用户 ID
  "comment": {
    "guid": "wS2uTQBMuujrwaGZ",    // 评论 ID
    "content": "这是评论内容",       // 评论内容
    "userIds": ["user1", "user2"], // 评论中若有 at 用户，此参数会传递一份，也可通过 mention_at 事件中获取
    "selectionGuid": "comment-Ynimup3kBXoxwF4j",  // 评论划词高亮区域的 ID
    "selectionContent": "123123123123"
  }
}
```

##### 删除评论

```json
{
  "kind":"comment",
  "type":"comment",
  "action":"delete",
  "fileId":"5b4f80b2aeb9d47d",
  "userId":"1",
  "deleteComment":{
    "guid":"EGhnSobUZWujKG2g", // 评论 ID
    "selectionGuid":"comment-Ynimup3kBXoxwF4j" // 划词高亮区域 ID
  }
}
```

##### 结束评论

```json
{
  "kind":"comment",
  "type":"comment",
  "action":"closeComments",
  "fileId":"5b4f80b2aeb9d47d",
  "userId":"1",
  "closeComment":{
    "selectionGuid":"comment-Ynimup3kBXoxwF4j" // 划词高亮区域 ID
  }
}
```

#### 表格

##### 添加评论

```json
{
  "kind":"comment",
  "type":"comment",
  "action":"create",
  "fileId":"6ee9f8da39498a33",
  "userId":"1",
  "comment":{
    "guid":"8dkDnFH6HPaGPqD1", // 评论 ID
    "selectionTitle":"MODOC",  // 评论所在表格 sheet ID
    "selectionGuid":"comment-OWTDgRUtF1LDoLwh", // 评论所在位置 ID
    "content":"测试表格评论", // 评论内容
    "userIds":["user1", "user2"] // at 的用户列表，没有则空
  }
}
```

##### 删除评论

```json
{
  "kind":"comment",
  "type":"comment",
  "action":"delete",
  "fileId":"6ee9f8da39498a33",
  "userId":"1",
  "deleteComment":{
    "guid":"bfdbJyDNlctJXcnw", // 评论 ID
    "selectionGuid":"comment-OWTDgRUtF1LDoLwh" // 评论位置区域 ID
  }
}
```

##### 结束评论

```json
{
  "kind":"comment",
  "type":"comment",
  "action":"closeComments",
  "fileId":"6ee9f8da39498a33",
  "userId":"1",
  "closeComment":{
    "selectionGuid":"comment-OWTDgRUtF1LDoLwh" // 评论位置区域 ID
  }
}
```

#### 传统文档

##### 添加评论

```json
{
  "kind":"comment",
  "type":"comment",
  "action":"create",
  "fileId":"691f6abc72f769a4",
  "userId":"1",
  "comment":{
    "guid":"ZwOJjlTDHcPda0V4", // 评论 ID
    "selectionGuid":"comment-Y29tbWVudD05Ng==", // 划词高亮区域 ID
    "content":"3123123", // 评论内容
    "userIds":["user1"] // 评论中 at 的用户，若无则为空
  }
}
```

##### 更新评论

```json
{
  "kind":"comment",
  "type":"comment",
  "action":"update",
  "fileId":"691f6abc72f769a4",
  "userId":"1",
  "comment":{
    "guid":"ZwOJjlTDHcPda0V4", // 评论 ID
    "selectionGuid":"comment-Y29tbWVudD05Ng==", // 划词高亮区域 ID
    "content":"312312345666", // 更新的评论内容
    "userIds":["user1"]  // 评论中 at 的用户，若无则为空
  }
}
```

##### 删除评论

```json
{
  "kind":"comment",
  "type":"comment",
  "action":"delete",
  "fileId":"691f6abc72f769a4",
  "userId":"1",
  "deleteComment":{
    "guid":"JrJWv9VuuOWlEo64", // 评论 ID
    "selectionGuid":"comment-Y29tbWVudD05Ng==" // 划词高亮区域 ID
  }
}
```

#### 幻灯片

##### 添加评论

```json
{
  "kind":"comment",
  "type":"comment",
  "action":"create",
  "fileId":"6ee9f8da39498a33",
  "userId":"1",
  "comment":{
    "guid":"Ddg3iIcz1nfqJSo9", // 评论 ID
    "selectionTitle":"s2f6cb94",  // 评论所在幻灯片页面 ID
    "selectionGuid":"comment-rrsIAvomFD3girqU", // 评论所在位置 ID
    "content":"测试评论", // 评论内容
    "userIds":[] // 幻灯片暂不支持 at 用户，为空
  }
}
```

##### 删除评论

```json
{
  "kind":"comment",
  "type":"comment",
  "action":"delete",
  "fileId":"6ee9f8da39498a33",
  "userId":"1",
  "deleteComment":{
    "guid":"Ddg3iIcz1nfqJSo9", // 评论 ID
    "selectionGuid":"comment-rrsIAvomFD3girqU" // 评论位置区域 ID
  }
}
```

##### 结束评论

```json
{
  "kind":"comment",
  "type":"comment",
  "action":"closeComments",
  "fileId":"6ee9f8da39498a33",
  "userId":"1",
  "closeComment":{
    "selectionGuid":"comment-rrsIAvomFD3girqU" // 评论位置区域 ID
  }
}
```

#### 应用表格

##### 添加评论

```json
{
  "kind":"comment",
  "type":"comment",
  "action":"create",
  "fileId":"794821fa1c537c88",
  "userId":"5",
  "comment":{
    "guid":"vi3sZCU3SPmYkaPL",
    "selectionTitle":"tiAtgfY7iRH", // 工作表 ID
    "selectionGuid":"comment-5CZl3gADOnG", // 评论位置
    "selectionContent":"JAuXsJM8obN:标题", //
    "content":"123123123 @yanjixiong ",
    "userIds":["5"]
  }
}
```

```json title="对于评论的回复评论"
{
  "kind": "comment",
  "type": "comment",
  "action": "create",
  "fileId": "794821fa1c537c88",
  "userId": "5",
  "comment": {
    "guid": "wY2THFlt3HoxbpMI",      // 评论 ID
    "selectionTitle": "tiAtgfY7iRH", // 工作表 ID
    "selectionGuid": "comment-r3Zmnqa1lm2", // 评论位置
    "content": "123132",             // 评论内容
    "userIds": [],
    "replyTo": "JIyRqgGY3GzyNj0d",   // 回复的评论 ID
    "replyComment": {
      "userId": "5",                 // 被回复评论的创建者
      "commentGuid": "JIyRqgGY3GzyNj0d", // 被回复评论的 ID
      "selectionTitle": "tiAtgfY7iRH", // 工作表 ID
      "selectionGuid": "comment-r3Zmnqa1lm2" // 评论位置
    }
  }
}
```

##### 删除评论

```json
{
  "kind":"comment",
  "type":"comment",
  "action":"delete",
  "fileId":"794821fa1c537c88",
  "userId":"5",
  "deleteComment":{
    "guid":"vi3sZCU3SPmYkaPL", // 评论 ID
    "selectionGuid":"comment-5CZl3gADOnG" // 评论位置
  }
}
```

### **讨论 （Discussion）**

在 创建讨论消息 时，会触发相应事件推送。

**HTTP Request Headers**

| Header 名         | 值         | 说明                   |
|:------------------|:-----------|:---------------------|
| X-Shimo-Sdk-Event | Discussion | 用于识别具体的事件类型 |

#### 轻文档

##### 发送讨论消息

```json
{
  "kind": "discussion",
  "type": "discussion", // 预留讨论下的小分类，目前就 discussion
  "action": "create",
  "fileId": "5b4f80b2aeb9d47d", // 接入方文件 ID
  "userId": "user1",            // 接入方用户 ID
  "discussion": {
    "id": "61f0bc916014eb0006c85c6f", // 讨论消息 ID
    "unixus": 1643166865906215, // 时间，微秒
    "content": "123",
    "positionId": ""            // 预留字段，目前无用
  }
}
```

### **提及（@ MentionAt）**

在 评论、讨论 消息中 @ 一个或多个用户时，将会触发此事件。

**被 @ 的用户 ID 列表，**由**SDK 在事件数据中**传递给接入方**。**

**HTTP Request Headers**

| Header 名         | 值        | 说明                   |
|:------------------|:----------|:---------------------|
| X-Shimo-Sdk-Event | MentionAt | 用于识别具体的事件类型 |

#### 轻文档

##### 在评论中 at

HTTP Request Body

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

##### 在讨论中 at

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

##### 在正文中 at

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

#### 表格

暂不支持讨论

##### 在评论中 at

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

##### 在正文中 at

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

#### 传统文档

暂不支持讨论

##### 在评论中 at

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

##### 在正文中 at

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

#### 幻灯片

暂不支持 at

#### 应用表格

暂不支持讨论

##### 在评论中 at

```json
{
  "kind":"mention_at",
  "type":"comment",
  "action":"create",
  "fileId":"794821fa1c537c88",
  "userId":"5",
  "comment":{
    "guid":"61p3ftQ8XuE31oba", // 评论 ID
    "selectionTitle":"tiAtgfY7iRH", // 工作表 ID
    "selectionGuid":"comment-5ehKQW140LZ", // 评论位置 ID
    "selectionContent":"JAuXsJM8obN:标题",
    "content":"12312312313 @zhangsan", // 评论 at 内容
    "userIds":["2"]   // 提及用户
  }
}
```

##### 在正文中 at

```json
{
  "kind":"mention_at",
  "type":"mention_at",
  "action":"create",
  "fileId":"794821fa1c537c88",
  "userId":"5",
  "mentionAt":{
    "guid":"tiAtgfY7iRHthWE9C0jIFcskBkByFcwVAhFQMQVEgj0B",
    "userId":"1" // 提及用户
  }
}
```


### **日期提醒 (DateMention)**

在文件中插入日期提醒规则时，会将规则信息推送至接入方，接入方可按需根据创建、更新、删除事件实现业务系统中的定时提醒功能。

:::caution
⚠️ 石墨 SDK 会在页面上操作后 **`立即`** 将 `创建`、`修改`、`删除` 事件推送至接入方，定时提醒功能需要业务方系统实现，石墨 SDK 不会在目标时间达到时发送任何事件通知。
:::

**HTTP Request Headers**

| Header 名         | 值          | 说明                   |
|:------------------|:------------|:---------------------|
| X-Shimo-Sdk-Event | DateMention | 用于识别具体的事件类型 |

#### 轻文档

HTTP Request Body

##### 创建

```json
{
  "kind": "mention",
  "type": "date_mention",
  "action": "create",
  "createData": {
    "id":"MlRFslp55Mrt19Iq",        // 提醒 ID
    "fileId":"ac4ce108419f103c",    // 文件 ID
    "authorId":"12",                // 创建者用户 ID
    "content":"<a>2022年1月19日 </a>", // 提醒文字
    "remindUserIds":["12"],         // 提醒用户
    "remindAt":"2021-12-07T15:00:00Z", // 提醒时间
    "positionId": "12"              // 预留字段，暂无用途
  }
}
```

##### 修改

```json
{
  "kind": "mention",
  "type": "date_mention",
  "action": "update",
  "updateData":{
    "id":"MlRFslp55Mrt19Iq",          // 提醒 ID
    "content":"<a>2022年1月20日 </a>", // 提醒文字
    "remindUserIds":["user1"],        // 提醒用户
    "remindAt":"2021-12-07T15:00:00Z", // 提醒时间
    "positionId": "12"                // 预留字段，暂无用途
  }
}
```

##### 删除

```json
{
  "kind": "mention",
  "type": "date_mention",
  "action": "remove",
  "removeData":{
    "id":"MlRFslp55Mrt19Iq" // 提醒 ID
  }
}
```

#### 表格

HTTP Request Body

##### 创建

```json
{
  "kind": "mention",
  "type": "date_mention",
  "action": "create",
  "createData": {
    "id":"MlRFslp55Mrt19Iq",        // 提醒 ID
    "fileId":"ac4ce108419f103c",    // 文件 ID
    "authorId":"12",                // 创建者用户 ID
    "content":"123",                // 提醒文字，若未设置文字，则为空
    "remindUserIds":["12"],         // 提醒用户
    "remindAt":"2021-12-07T15:00:00Z", // 提醒时间
    "positionId":"MODOC-QjUpDSt3swY6Tlir" // 预留字段，暂无用途
  }
}
```

##### 修改

```json
{
  "kind": "mention",
  "type": "date_mention",
  "action": "update",
  "updateData":{
    "id":"MlRFslp55Mrt19Iq",        // 提醒 ID
    "content":"12313",              // 提醒文字
    "remindUserIds":["12"],         // 提醒用户
    "remindAt":"2021-12-07T15:00:00Z", // 提醒时间
    "positionId":"MODOC-QjUpDSt3swY6Tlir" // 预留字段，暂无用途
  }
}
```

##### 删除

```json
{
  "kind": "mention",
  "type": "date_mention",
  "action": "remove",
  "removeData":{
    "id":"MlRFslp55Mrt19Iq" // 提醒 ID
  }
}
```

#### 传统文档

HTTP Request Body

##### 创建

```json
{
  "kind": "mention",
  "type": "date_mention",
  "action": "create",
  "createData": {
    "id":"MlRFslp55Mrt19Iq",        // 提醒 ID
    "fileId":"ac4ce108419f103c",    // 文件 ID
    "authorId":"12",                // 创建者用户 ID
    "content":"2022年01月19日, 周三", // 提醒文字
    "remindAt":"2021-12-07T15:00:00Z", // 提醒时间
    "positionId":"36"                // 预留字段，暂无用途
  }
}
```

##### 修改

```json
{
  "kind": "mention",
  "type": "date_mention",
  "action": "update",
  "updateData":{
    "id": "MlRFslp55Mrt19Iq",        // 提醒 ID
    "content": "2022年01月21日, 周五 19:56", // 提醒文字
    "remindAt": "2021-12-07T15:00:00Z", // 提醒时间
    "positionId":"36"                // 预留字段，暂无用途
  }
}
```

##### 删除

```json
{
  "kind": "mention",
  "type": "date_mention",
  "action": "remove",
  "removeData":{
    "id":"MlRFslp55Mrt19Iq" // 提醒 ID
  }
}
```

#### 幻灯片

暂不支持

### **文件内容更新 (FileContent)**

在文件有用户进行修改并被服务器处理成功后，将会触发此事件。

**HTTP Request Headers**

| Header 名               | 值                                   | 说明                                                                                           |
|:------------------------|:-------------------------------------|:---------------------------------------------------------------------------------------------|
| X-Shimo-Credential-Type | 3                                    | 无法提供 token 的 [回调凭证类型](./resources.md##credential-types)                             |
| X-Shimo-Sdk-Event       | FileContent                          | 用于识别具体的事件类型                                                                         |
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
### **文档协作者协同状态变化 (Collaborator)**

当一个文档当前参与协作的用户发生改变时，推送此事件。

事件可通过 `action` 的值区分用户行为：
- `enter` 网络无异常情况下，进入文档，触发此事件
- `leave` 用户关闭文档或网络断开时，触发此事件

**HTTP Request Headers**

| Header 名               | 值                                   | 说明                                                                                           |
|:------------------------|:-------------------------------------|:---------------------------------------------------------------------------------------------|
| X-Shimo-Credential-Type | 3                                    | 无法提供 token 的 [回调凭证类型](./resources.md##credential-types)                             |
| X-Shimo-Sdk-Event       | Collaborator                         | 用于识别具体的事件类型                                                                         |
| X-Shimo-Signature       | ebc1cde3-9b57-4962-883d-54302d428600 | 当石墨无法提供接入方 token 时，使用接入方对应 appId, secret 主动生成 signature 用于回调接口校验 |

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

### **版本 (Revision)** {#events-revision}

当有用户创建、修改、删除版本并被服务器处理成功后，推送此事件。

事件可通过 `action` 的值区分用户行为：
- `create` 用户创建版本时，触发此事件
- `update` 用户修改版本时，触发此事件
- `delete` 用户删除版本时，触发此事件

**HTTP Request Headers**

| Header 名         | 值       | 说明                   |
|:------------------|:---------|:---------------------|
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

### **系统事件 (System)** {#events-system}

推送 SDK 系统级别的事件，目前为实验性功能，未来可能会有调整或变化。

**HTTP Request Headers**

| Header 名         | 值       | 说明                   |
|:------------------|:---------|:---------------------|
| X-Shimo-Sdk-Event | System | 用于识别具体的事件类型 |

#### (实验性) 回调请求错误

当 SDK 请求接入方接口异常时，将 `请求数据` 和 `响应数据` 以及 SDK 错误通过此消息推送至接入方接口，用于及时发现接入方自身回调接口问题。

:::caution 使用说明

此事件目前仅包含**部分请求**的错误信息，暂未包含所有场景，不建议依赖此事件作为回调是否正常的依据。

:::

**HTTP Request Body**

Request Headers 中的相关信息参考 [回调凭证类型](./resources.md#credential-types) 说明
- `X-Shimo-Credential-Type`
- `X-Shimo-Token`
- `X-Shimo-Signature`

```json
{
  "kind": "System",
  "type": "endpointCallback",
  "appId": "your app id",
  "timestamp": 1669610727509, // 此事件产生的时间戳，单位毫秒
  "request": {                // 错误回调请求的请求数据
    "headers": {
      "Accept": "application/json",
      "Content-Type": "application/json",
      "X-Shimo-APP-ID": "your app id",
      "X-Shimo-Credential-Type": "0",      // 0 代表使用用户 token 发起的回调请求， 3 表示石墨 SDK 主动签名发起的回调请求
      "X-Shimo-Token": "your user token",
      "X-Shimo-Signature": "the signature generate by sdk"
    },
    "url": "http://your-callback-api-host/callback/users/current/info",
    "body": ""
  },
  "response": {               // 错误回调请求的响应数据
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
