---
title: 搜索功能
sidebar_position: 7
---

## 获取与文件相关的用户列表{#users-recent}

此接口用于 @ 功能用户补全。 在文件中输入 @ 时，展示下拉菜单，选择提及的用户，接入方可根据自己系统情况返回，例如：和当前用户最相关的用户列表、和当前文件最相关的用户列表、或返回空数组。

**请求地址**

*GET* /search/users/recent?fileId=file12345

**HTTP Query Parameters**

|参数名|值|说明|
|:----|:----|:----|
|fileId|file12345|表示当前查询来自接入方的某个文件 ID|

**HTTP Request Headers**

|**Header 名**|**值**|**说明**|
|:----|:----|:----|
|X-Shimo-Token|接入方提供的 Token|用于接入方对本次请求鉴权|

**HTTP Response Body**

|字段名|类型|值示例|说明|
|:----|:----|:----|:----|
|items[0].id|string|userid123|接入方系统的用户的 ID|
|items[0].name|string|张三|接入方系统的用户名称|
|items[0].avatar|string|[http://fake.site/user-123.png](http://fake.site/user-123.png)|接入方系统的用户头像地址|
|items[0].email|string|[user123@fake.site](mailto:user123@fake.site)|接入方系统的用户邮箱|

**Response Body Example**

```plain
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

## 获取与文件相关的文件列表{#files-recent}

此接口用于 @ 功能文件补全。 在文件中输入 @ 时，展示下拉菜单，选择提及的文件，接入方可根据自己系统情况返回，例如：和当前用户最相关的文件列表、和当前文件最相关的文件列表、或返回空数组。

### 支持返回 @ 第三方文件

:::caution 说明

由于第三方文件无法进行协作，仅支持在 SDK 中插入 @ 时保留引用，因此需要达到点击后跳转至第三方页面，需要满足以下条件：

1. file 信息 `type` 字段值为 `file`，参考如下 Response Body Example。

2. file 信息增加 `fullUrl` 字段作为跳转至接入方系统的完整地址，参考如下 Response Body Example。

3. 前端使用 `shimo-js-sdk` 时在调用 `connect` 需要实现 `openLink` 方法用于控制编辑器内点击链接时的跳转行为。

:::

**请求地址**

*GET* /search/files/recent?fileId=file12345

**HTTP Query Parameters**

|参数名|值|说明|
|:----|:----|:----|
|fileId|file12345|表示当前查询来自接入方的某个文件 ID|

**HTTP Request Headers**

|**Header 名**|**值**|**说明**|
|:----|:----|:----|
|X-Shimo-Token|接入方提供的 Token|用于接入方对本次请求鉴权|

**HTTP Response Body**

|字段名|类型|值示例|说明|
|:----|:----|:----|:----|
|items[0].id|string|ba13551165cc5066|接入方系统中的文档 ID|
|items[0].name|string|file title|文档标题|
|items[0].type|string|documentPro|文档类型列表参考 [创建文档](./../06API-document/interface-description/collaborative-editing.md#create-collab-file)，若为第三方存储的非石墨协作文件，则固定传 file|
|items[0].creatorId|string|1|接入方文件的创建者用户 ID|
|items[0].createdAt|string|2021-08-01T00:00:00Z|接入方记录的文件创建时间，`UTC` 时间（0 时区）， 格式为 `2022-01-01T09:00:01Z`|
|items[0].updatedAt|string|2021-08-02T00:00:00Z|接入方记录的文件最后更新时间，`UTC` 时间（0 时区）， 格式为 `2022-01-01T09:00:01Z`|
|items[0].views|number|100|接入方统计的文件阅读次数，用于在文档信息中展示阅读次数。若接入方未返回，则默认显示为 1|
|items[0].permissions|object| - |请参考 [文件权限说明](./../04service-callback/file-information.md#file-permission)|
|items[0].fullUrl|string|[https://customer-system.com/path/to/file](https://customer-system.com/path/to/file)|访问文件的完整地址|

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
    "views": 0,
    "fullUrl": "https://customer-system.com/path/to/file" // 仅在文件仅为接入方自身系统的文件而非协同文档时需要传递
   }
]
```

## 按关键字搜索文件和用户列表{#search-by-keyword}

此接口用 @ 功能搜索，获取下拉菜单的结果使用。 输入 @ 时，如需要根据关键字搜索用户或文件，通过请求接入方系统返回对应的文件列表、用户列表；如不需要可将对应字段返回空数组。

### 支持返回 @ 第三方文件

:::caution 说明

由于第三方文件无法进行协作，仅支持在 SDK 中插入 @ 时保留引用，因此需要达到点击后跳转至第三方页面，需要满足以下条件：

1. file 信息 `type` 字段值为 `file`，参考如下 Response Example

2. file 信息增加 `fullUrl` 字段作为跳转至接入方系统的完整地址，参考如下 Response Example

3. 前端使用 `shimo-js-sdk` 时在调用 `connect` 需要实现 `openLink` 方法用于控制编辑器内点击链接时的跳转行为。

::: 

**请求地址**

*POST* /search

**HTTP Request Headers**

|**Header 名**|**值**|**说明**|
|:----|:----|:----|
|X-Shimo-Token|接入方提供的 Token|用于接入方对本次请求鉴权|

**HTTP Request Body**

|字段名|类型|值示例|说明|
|:----|:----|:----|:----|
|fileId|string|fileid1234|接入方系统的文件 ID|
|keyword|string|test|文件名或者用户名的关键字|
|page|number|0|搜索的结果第几页|
|pageSize|number|6|搜索结果每页数量|
|type|string|file_name,recent_contact,collaborator,team_member,department|指定搜索请求需要搜索的结果包括哪些类型，按照传入的类型按需返回不同的搜索结果。通过英文逗号 , 分割后根据命中的类型返回相应的结果。|

### `type` 字段说明

:::caution 说明

* `file_name` 表示通过关键字搜索文件，对应 Response 中 `files` 结果。
* `recent_contact` 表示通过关键字搜索比较相关用户，对应 Response 中 `recentUsers` 结果。
* `collaborator` 表示通过关键字搜索协作者用户，对应 Response 中 `collaborators` 结果。
* `team_member` 表示通过关键字搜索团队成员，对应 Response 中 `teamMembers` 结果。
* `department` 表示通过关键字搜索部门，对应 Response 中 `department` 结果。

:::

**Request Body Example**

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

|字段名|类型|值示例|说明|
|:----|:----|:----|:----|
|files.count|number|3|搜索到结果的总条数，包括所有分页结果的总和|
|files.page|number|0|当前为第几页，第如第一页的值为 0|
|files.pageSize|number|6|每一页分页的结果的数量|
|files.pageCount|number|1|分搜索结果的总条数按照分页大小计算后的总页数|
|files.results[0].id|string|ba13551165cc5066|接入方系统中的文档 ID|
|files.results[0].name|string|file title|文档标题|
|files.results[0].type|string|documentPro|文档类型列表参考 [创建文档](./../06API-document/interface-description/collaborative-editing.md#create-collab-file)，若为第三方存储的非石墨协作文件，则固定传 `file`|
|files.results[0].creatorId|string|1|接入方文件的创建者用户 ID|
|files.results[0].createdAt|string|2021-08-01T00:00:00Z|接入方记录的文件创建时间，`UTC` 时间（0 时区）， 格式为 `2022-01-01T09:00:01Z`|
|files.results[0].updatedAt|string|2021-08-02T00:00:00Z|接入方记录的文件最后更新时间，`UTC` 时间（0 时区）， 格式为 `2022-01-01T09:00:01Z`|
|files.results[0].views|number|100|接入方统计的文件阅读次数，用于在文档信息中展示阅读次数。若接入方未返回，则默认显示为 1|
|files.results[0].permissions|object|    | 请参考 [文件权限说明](./../04service-callback/file-information.md#file-permission) |
|files.results[0].fullUrl|string|[https://customer-system.com/path/to/file](https://customer-system.com/path/to/file)|访问文件的完整地址|
|recentUsers.count|number|3|搜索到结果的总条数，包括所有分页结果的总和|
|recentUsers.page|number|0|当前为第几页，第如第一页的值为 0|
|recentUsers.pageSize|number|6|每一页分页的结果的数量|
|recentUsers.pageCount|number|1|分搜索结果的总条数按照分页大小计算后的总页数|
|recentUsers.results[0].id|string|userid123|接入方系统的用户的 ID|
|recentUsers.results[0].name|string|张三|接入方系统的用户名称|
|recentUsers.results[0].avatar|string|[http://fake.site/user-123.png](http://fake.site/user-123.png)|接入方系统的用户的头像地址|
|recentUsers.results[0].email|string|[user123@fake.site](mailto:user123@fake.site)|接入方系统的用户的邮箱|
|collaborators.count|number|3|搜索到结果的总条数，包括所有分页结果的总和|
|collaborators.page|number|0|当前为第几页，第如第一页的值为 0|
|collaborators.pageSize|number|6|每一页分页的结果的数量|
|collaborators.pageCount|number|1|分搜索结果的总条数按照分页大小计算后的总页数|
|collaborators.results[0].id|string|userid123|接入方系统的用户的 ID|
|collaborators.results[0].name|string|张三|接入方系统的用户名称|
|collaborators.results[0].avatar|string|[http://fake.site/user-123.png](http://fake.site/user-123.png)|接入方系统的用户的头像地址|
|collaborators.results[0].email|string|[user123@fake.site](mailto:user123@fake.site)|接入方系统的用户的邮箱|
|teamMembers.count|number|3|搜索到结果的总条数，包括所有分页结果的总和|
|teamMembers.page|number|0|当前为第几页，第如第一页的值为 0|
|teamMembers.pageSize|number|6|每一页分页的结果的数量|
|teamMembers.pageCount|number|1|分搜索结果的总条数按照分页大小计算后的总页数|
|teamMembers.results[0].id|string|userid123|接入方系统的用户的 ID|
|teamMembers.results[0].name|string|张三|接入方系统的用户名称|
|teamMembers.results[0].avatar|string|[http://fake.site/user-123.png](http://fake.site/user-123.png)|接入方系统的用户的头像地址|
|teamMembers.results[0].email|string|[user123@fake.site](mailto:user123@fake.site)|接入方系统的用户的邮箱|
|department.count|number|3|搜索到结果的总条数，包括所有分页结果的总和|
|department.page|number|0|当前为第几页，第如第一页的值为 0|
|department.pageSize|number|6|每一页分页的结果的数量|
|department.pageCount|number|1|分搜索结果的总条数按照分页大小计算后的总页数|
|department.results[0].id|string|123|接入方系统的部门 ID|
|department.results[0].name|string|张三|接入方系统的部门名称|
|department.results[0].allMemberCount|number|10|接入方系统的当前部门成员总数|
|department.results[0].parentDepartments|Array|    |接入方系统当前部门数据对应的所有父级部门，顺序 从高到底 ，不包括 团队 层级，从一级部门开始，若已是一级部门，则返回空数组。由于在 表格锁定 场景搜索部门时可能遇到同名部门，可以通过所有父级部门来帮助区分部门，若无需区分，可选择不返回父级部门列表。|
|department.results[0].parentDepartments[0].id|string|    |接入方系统中当前部门父级部门的 ID|
|department.results[0].parentDepartments[0].name|name|    |接入方系统中当前部门父级部门的名称|

**Response Body Example**

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
        "views": 0,
        "fullUrl": "https://customer-system.com/path/to/file" // 仅在文件仅为接入方自身系统的文件而非协同文档时需要传递
       }
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

