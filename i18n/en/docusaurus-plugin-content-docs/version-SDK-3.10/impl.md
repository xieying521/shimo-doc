---
sidebar_position: 6
toc_max_heading_level: 4
description: >-
  For security reasons, starting from 2021-12-01, you need to return permissions information to verify whether the current user has the corresponding permissions. Get the current user
  You need to use this API to obtain user identity information when creating, editing, and importing information such as ID, name, and avatar image address. while
  If the SDK requests an interface to the access site is abnormal, the request data, response data, and SDK errors are pushed to the access side through this message.
---

# The access party implements the interface

:::caution
⚠️ Unless otherwise specified, all requests are  , `HTTP`and both the request header and the return header `Content-Type` are `application/json` . 

⚠️ The representation in the interface `{xxxx}` requires the user to replace it with its own value in the interface address, for example, `/users/{userId}` the  real call address of may be  , `/users/123`and this parameter will be described in the form of in the interface description `userId` . 

⚠️ For example, if user A creates Table 1 and references Table 2 and shares it with User B, when User B accesses Table 1, you need to check whether User A still has permission to access Table 2.

:::

## Overview

The following table shows a complete list of APIs that need to be implemented in known scenarios when the Graphite SDK is accessed, as shown in the following table.

If the accessing party is only interested in** some functions**, refer to  the [List of Interfaces in Common Function Scenarios](#features-api-list). 

| interface                                                                | Purpose of the interface                                    | illustrate                                                                                                                                                                                  |
| :------------------------------------------------------------------ | :------------------------------------------ | :------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `GET` [/files](#list-files)                                         | Get a list of files for the current user                      | This API is requested to add cross-table formulas, merge worksheets, and so on, and select a list of files that the user has recently used                                                                                                                    |
| `GET` [/files/{fileId}](#file-meta-collab)                          | Get File Meta Information - Collaborate Document                     | This API is requested to obtain basic information and permissions, such as creation, import, and editing                                                                                                                                    |
| `GET` [/files/{fileId}](#file-meta-preview)                         | Get file meta information - file preview                     | This API is requested when you get the basic information, download address, and preview the file                                                                                                                                        |
| `GET` [/files/{fileId}/collaborators](#file-collborators)           | Get the current collaborating user                          | Used to obtain the collaborator list information of the collaboration file (the accessing party can control which users are collaborators), such as the comment is synchronized to other users in real time, and it is used to obtain the collaborator list and send comment events                                                  |
| `POST` [/files/{fileId}/url](#file-url)                             | Obtain the complete access address of the file specified by the access party            | When @ other files in the file, a jump link address of the Graphite SDK will be retained in the file after the file is exported, and when the link is opened, the current complete access link will be obtained through this interface, and the access link will be redirected to the access party file address after obtainment                |
| `GET` [/admin/files/{fileId}](#file-meta-admin)                     | Get File Meta Information - Collaborative Document Automated Task             | This API is requested when an automatic task fetches basic information (currently only cross-table reference tasks).                                                                                                                          |
| `GET` [/admin/files/{fileId}/by-user-id](#file-meta-by-uid)         | Obtain file meta information according to the specified user - automatic task of collaborative documents | This API is requested when the Graphite SDK needs to actively obtain the basic information of the file (currently only cross-table reference scenarios).                                                                                                             |
| `GET` [/users/current/info](#user-current)                          | Get the current user information                            | Obtain the ID, name, and address of the current user, and use this API to obtain the user's identity information when creating, editing, and importing                                                                                           |
| `GET` [/users/current/team](#user-current-team)                     | Get information about the team the current user belongs to                    | Get the ID, name, and total number of team members of the team to which the current user belongs                                                                                                                                         |
| `GET` [/users/{userId}](#user-by-id)                                | Obtain user information                                | This API is requested when you obtain the basic information of the user, such as the ID, name, and avatar image address, and obtains the specified user information in the collaboration scenario                                                                                            |
| `GET` [/users/{userId}/watermark](#user-watermark)                  | Obtain the user watermark information                            | To obtain the watermark information of a specified user, you can obtain the required watermark field or return empty according to the needs of your business system                                                                                                          |
| `GET` [/users/{userId}/department-paths](#user-department-paths)    | Obtain the user's department path                            | Obtain the department path of a specified user, and obtain the required department path or return empty according to the needs of your business system                                                                                                          |
| `POST` [/users/batch/get](#users-by-ids)                            | Obtain user information in batches                            | Provide information about multiple specified users, such as ID, name, and avatar image address. If you want to display the information of multiple users when you want to display the history of multiple users in the sidebar, this API is requested                                                          |
| `GET` [/teams/{teamId}/members](#team-members)                      | Get a list of members under the team                        | Obtain the list of members of a specified team and query the list results based on the pagination information                                                                                                                                      |
| `GET` [/departments/{departmentId}](#department-info)               | Get departmental information                                | Get information about a specified department, including department ID, name, total number of department members                                                                                                                                   |
| `GET` [/departments/{departmentId}/children](#children-departments) | Gets the sub-department nodes of the department                      | Obtain the subordinate department nodes of the specified department, including the department ID, name, and total number of department members                                                                                                                       |
| `GET` [/departments/{departmentId}/members](#department-members)    | Get a paginated list of members under the department                    | Obtain the paginated list of members in a specified department, and obtain the list of users in the current pagination and the total number of members                                                                                                                  |
| `GET` [/search/users/recent](#users-recent)                         | Get a list of users related to the file                    | When entering @ in a file, in order to display the drop-down menu, select the mentioned user, because there is no keyword entered at this time, the accessing party can return according to its own system situation, such as: the list of users most related to the current user, the list of users most related to the current file, or return an empty array |
| `GET` [/search/files/recent](#files-recent)                         | Get a list of files related to the file                    | When entering @ in a file, display the drop-down menu, select the mentioned file, because there is no keyword entered at this time, the accessor can return according to its own system situation, such as: the list of files most relevant to the current user, the list of files most related to the current file, or return an empty array   |
| `POST` [/search](#search-by-keyword)                                | Search for files or users based on keywords                    | When entering @, if you need to return the corresponding file list and user list by requesting the access party system according to the keyword, you can return the corresponding field to an empty array if you don't need it. This interface is involved when adding people to a table lock                                         |
| `POST` [/events](#events)                                           | Push Graphite SDK related events                       | The service provider selectively processes the event data that needs to be cared for. For example, when someone takes action on a comment, discussion, mentions, etc., some actions will trigger this event                                                                                    |

## Features-api-list {#features-api-list}

##### When editing using a collaborative document

- `GET` [/files/{fileId}](#file-meta-collab) to get meta-info-synergy documents
- `GET` [/users/current/info](#user-current) to get the current user information
- `GET` [/users/{userId}](#user-by-id) to get user information
- `POST` [/users/batch/get](#users-by-ids) to obtain user information in batches
- `GET` [/files/{fileId}/goddes](#file-collborators) 
- `GET` [/users/{userId}/watermark](#user-watermark) to obtain the user's watermark information

#####  `内容``评论` {#feature-mention-at} when using the @ and @ mentioned (@) file functions in`讨论` the collaboration document 

- `GET` [/search/users/recent](#users-recent) to get a list of users associated with the file
- `GET` [/search/files/recent](#files-recent) to get a list of files related to the file
- `POST` [/search](#search-by-keyword) searches for files or users based on keywords

##### When using the Collaborative Document `导入` or `导出` Feature

- `GET` [/files/{fileId}](#file-meta-collab) to get meta-info-synergy documents
- `GET` [/users/current/info](#user-current) to get the current user information
- `POST` [/files/{fileId}/url](#file-url) to obtain the complete access address of the file specified by the access party

##### {#feature-sheets} when using professional tables across table formulas, when merging worksheets

:::tip tip

Since the cross-tabular and merge worksheets are only for** tabular **files, the interface for searching the file list will filter out `type: spreadsheet` files of type , without which the  list may be empty. 

:::

- `GET` [/files](#list-files) to get a list of files for the current user
- `GET` [/search/files/recent](#files-recent) to get a list of files related to the file
- `GET` [/admin/files/{fileId}/by-user-id](#file-meta-by-uid) Obtain file meta information based on the specified user - automatic task of collaborative documents
- `GET` [/admin/files/{fileId}](#file-meta-admin) Obtain file meta information - Collaborate with document automatic tasks

##### When using the professional form lock feature

- `GET` [/files/{fileId}/goddes](#file-collborators) 
- `POST` [/search](#search-by-keyword) searches for files or users based on keywords

##### Third-party file previews

- `GET` [/files/{fileId}](#file-meta-preview) to get file meta information - file preview
- `GET` [/users/current/info](#user-current) to get the current user information

##### When you need to customize the functions of the business system through graphite collaborative document events

- `POST` [/events](#events) pushes events related to the Graphite SDK

##### When mentioning at or table locking, select the department or department member {#feature-team-department}

Related concepts refer to [Teams & Departments](./concepts/team_department.md)

- `GET` [/users/current/info](#user-current) to get the current user information
- `GET` [/users/current/team](#user-current-team) to obtain information about the team to which the current user belongs
- `GET` [/users/{userId}/department-paths](#user-department-paths) 获取用户部门路径
- `GET` [/teams/{teamId}/members](#team-members) to get a list of members under the team
- `GET` [/departments/{departmentId}](#department-info) 获取部门信息
- `GET` [/departments/{departmentId}/children](#children-departments) 获取部门的下级部门节点
- `GET` [/departments/{departmentId}/members](#department-members) 获取部门下的成员分页列表
- `POST` [/search](#search-by-keyword) searches for files, users, team members, departments based on keywords

## file

### Basic concepts

#### File permissions {#file-permissions}

Permissions 字段

| The name of the field      | type    | illustrate                                                                                |
| :---------- | :------ | :---------------------------------------------------------------------------------- |
| Commentable | boolean | Whether the user can comment                                                              |
| editable    | boolean | Whether the user can make edits                                                              |
| readable    | boolean | Whether the user can access the document                                                        |
| copyable    | boolean | Whether the user can copy the content of the document. `editable` last `true` ,`copyable` must be `true` . |
| Exportable  | boolean | Whether the user can export                                                              |
| manageable  | boolean | Whether the user can manage the document, such as deleting it                                              |

### Get the file list of the current user {#list-files}

This API is requested to add cross-table formulas, merge worksheets, and so on, and select a list of files that the user has recently used

**The address of the request**

_GET_ /files

**HTTP Request Headers**

| Header 名     | value  | illustrate                                         |
| :------------ | :-- | :------------------------------------------- |
| X-Shimo-Token |     | The token provided by the service provider is used for the service provider to authenticate the request |

**HTTP Query Parameters**

| The name of the field  | type   | Examples of values     | illustrate                                                               |
| :------ | :----- | :--------- | :----------------------------------------------------------------- |
| limit   | number | 123        | Maximum number of articles ID                                                      |
| orderBy | string | created_at | The results are sorted by when the file was created or last modified, and you can choose created_at, updated_at |

**HTTP Response Body**

| The name of the field               | type   | Examples of values               | illustrate                                                                                                      |
| :------------------- | :----- | :------------------- | :-------------------------------------------------------------------------------------------------------- |
| items[0].id          | string | BA13551165CC5066     | The ID of the document in the service provider's system                                                                                     |
| items[0].name        | string | file title           | The title of the document                                                                                                  |
| items[0].type        | string | documentPro          | The list of document types refers to[ creating a document](./apis.md#create-collab-file), and if it is a non-graphite collaboration file stored by a third party, it will be fixed and `file` uploaded|
| items[0].creatorId   | string | 1                    | The user ID of the creator of the ingestion file                                                                                 |
| items[0].createdAt   | string | 2021-08-01T00:00:00Z | The time when the file was created recorded by the access party`UTC` (0 time zone) in the format `2022-01-01T09:00:01Z`of                            |
| items[0].updatedAt   | string | 2021-08-02T00:00:00Z | The last update time of the file recorded by the access party,`UTC` the time (0 time zone), and the format is `2022-01-01T09:00:01Z`                        |
| items[0].views       | number | 100                  | The number of times a file is read by the ingestor and is used to display the number of views in the document. If the accessing party does not return, it will be displayed as 1 by default                    |
| items[0].permissions | object |                      | [Information about the user's permissions on the file](#file-permissions)                                                                   |

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

### Get File Meta Info-Collaborate Document {#file-meta-collab}

This API is requested to obtain basic information and permissions, such as creation, import, and editing

**The address of the request**

_FÁ_ / files / {fileId}

**HTTP Request Headers**

| Header 名     | value  | illustrate                                         |
| :------------ | :-- | :------------------------------------------- |
| X-Shimo-Token |     | The token provided by the service provider is used for the service provider to authenticate the request |

**HTTP Response Body**

| The name of the field      | type   | Examples of values               | illustrate                                                                                   |
| :---------- | :----- | :------------------- | :------------------------------------------------------------------------------------- |
| id          | string | 123                  | The ID of the document in the service provider's system                                                                  |
| name        | string | file title           | The title of the document                                                                               |
| type        | string | documentPro          | For a list of document types, refer to[ Creating a Document](./apis.md#create-collab-file)                               |
| creatorId   | string | 1                    | The user ID of the creator of the ingestion file                                                              |
| createdAt   | string | 2021-08-01T00:00:00Z | The time when the file was created recorded by the access party`UTC` (0 time zone) in the format `2022-01-01T09:00:01Z`of         |
| updatedAt   | string | 2021-08-02T00:00:00Z | The last update time of the file recorded by the access party,`UTC` the time (0 time zone), and the format is `2022-01-01T09:00:01Z`     |
| views       | number | 100                  | The number of times a file is read by the ingestor and is used to display the number of views in the document. If the accessing party does not return, it will be displayed as 1 by default |
| permissions | object |                      | [Information about the user's permissions on the file](#file-permissions)                                                |
| teamGuid    | string | 123                  | The ID of the team to which the accessing file belongs, optional. If this field is returned, the user.teamGuid field should also be implemented and returned       |

:::caution
**`manageable` 权限说明**

⚠️ It is recommended to specify at least the permission to the creator of the file `manageable: true` , and if the accessing party allows others to have administrative rights, you can design and implement it yourself. 

 The following scenarios are used to use permissions:`manageable` 

- When calling the Graphite SDK to delete a file, you need to `manageable` be `true`
- When you lock a table, if the current user has `manageable` permission  of  , the `true` current user is displayed as a manager
- `更新` or  a `删除` version created by someone else, the current user's permission on the file needs to `manageable`  be `true` only allowed to be operated, otherwise it will be denied
- When exporting a table, if the current user has the permission for this file `manageable` `true` , the exported file will contain the locked content of the table, otherwise the exported file content does not contain the locked content.
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

### Get File Meta Info-File Preview {#file-meta-preview}

This API is requested when you get the basic information, download address, and preview the file

> ⚠️ This API address is the same as that of "Get File Meta Information - Collaborative Document", but the return result is different, and the accessing party may need to distinguish whether the file is a graphite collaborative document or a common file.

**The address of the request**

_FÁ_ / files / {fileId}

**HTTP Request Headers**

| Header 名     | value  | illustrate                                         |
| :------------ | :-- | :------------------------------------------- |
| X-Shimo-Token |     | The token provided by the service provider is used for the service provider to authenticate the request |

**HTTP Response Body**

| The name of the field      | type   | Examples of values                       | illustrate                                                                                                                                                                                                                                                    |
| :---------- | :----- | :--------------------------- | :------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| id          | string | 123                          | The file ID in the service provider's system                                                                                                                                                                                                                                   |
| name        | string | file title                   | File title                                                                                                                                                                                                                                                |
| type        | string | file                         | Currently used to `file` denote a third-party file type                                                                                                                                                                                                                      |
| permissions | object |                              | [Information about the user's permissions on the file](#file-permissions)                                                                                                                                                                                                                 |
| downloadUrl | string | http://fake.site/sample.docx | Used to download the contents of an Office file when previewing it                                                                                                                                                                                                                      |
| ext         | string | docx                         |  This field is used to identify the extension of this file when no identifiable information `downloadUrl`is returned in the HTTP Response Header  or by the accessing party returned by the `Content-Type` graphite request download address (`Content-Disposition`). There is no need to bring `.`, if the file is  , `测试文件.docx` it should be passed`docx`, if it is returned, it `.docx` may fail to recognize |

:::caution
⚠️ When requesting download, the accessor cannot return `Content-Type` and  the `Content-Disposition` information is used for the Graphite SDK to identify the file type, and the `ext` Graphite SDK is informed of the extension of the file through the field returned in the meta information. For example`docx`,  is recognized as a Word file. 

`ext` The field does not have `.` , e.g. the file is , `测试文件.docx` and should be returned `docx`.
:::

:::caution
⚠️ For security reasons, starting after 2021-12-01, you need to return `permissions` information to verify whether the current user has the appropriate permissions to obtain file information during preview.
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

> Note: The graphite server will download `downloadUrl`the file for parsing. You may need to set up a designated header for graphite to choose how to parse. For more details, please refer to [the Header Requirements](./roll.md#http-response-header-要求). 

### Get a list of collaborators for a file {#file-collborators}

Used to obtain the collaborator list information of the collaboration file (the accessing party can control which users are collaborators), such as the comment is synchronized to other users in real time, and it is used to obtain the collaborator list and send comment events

:::caution
⚠️ When obtaining the list of collaborators in the table lock function, the accessing party** must **indicate whether the returned user is  , `管理者`which is used to display in the lock panel, and the manager cannot be locked by others. Not returning this field correctly can result in inconsistent managers in the list of collaborators seen by different users.
:::

**The address of the request**

_GET_ /files/{fileId}/collaborators

**HTTP Request Headers**

| Header 名     | value  | illustrate                                         |
| :------------ | :-- | :------------------------------------------- |
| X-Shimo-Token |     | The token provided by the service provider is used for the service provider to authenticate the request |

**HTTP Response Body**

| The name of the field             | type    | Examples of values                        | illustrate                       |
| :----------------- | :------ | :---------------------------- | :------------------------- |
| items[0].id        | string  | userid123                     | The ID of the user of the service provider's system      |
| items[0].name      | string  | Tom                          | The user name of the service provider's system       |
| items[0].avatar    | string  | http://fake.site/user-123.png | The avatar address of the user of the service provider's system |
| items[0].email     | string  | user123@fake.site             | The email address of the user of the service provider's system     |
| items[0].isManager | boolean | true                          | Whether the user is the administrator of this file |

Response Body Example

```json
[
  {
    "id": "user1",
    "name": "testuser1",
    "avatar": "http://your-domain/avatar-url",
    "email": "xxx@shimo.im",
    "isManager": true // 指定此用户是否此文件的管理者
  },
  {
    "id": "user2",
    "name": "testuser2",
    "avatar": "http://your-domain/avatar-url",
    "email": "xxx2@shimo.im",
    "isManager": false // 指定此用户是否此文件的管理者
  }
]
```

### Obtain the complete access address of the file specified by the accessing party {#file-url}

Obtain the complete access address of the accessing party through the accessing party file ID, which is usually used in collaborative document scenarios.

If there are @ associations with other files in the light document file, when the file is exported as docx, the @ association in the file will be converted into a jump link of the Graphite SDK, and when this jump link is accessed, the Graphite SDK will try to `fileId` redirect by obtaining the full access address of the access system. 

:::caution
⚠️ The interface is initiated by the Graphite SDK, and for security reasons, it is recommended to verify the HTTP header passed by this interface `X-Shimo-Signature` , and the signature algorithm is consistent with the method used by the accessing party to generate the signature.
:::

**The address of the request**

_POST_ /files/{fileId}/url

**HTTP Request Headers**

| Header 名         | value  | illustrate                                                                                          |
| :---------------- | :-- | :-------------------------------------------------------------------------------------------- |
| X-Shimo-Signature |     | The signature string signed by Graphite based on AppId, AppSecret and fileId is used by the accessing party to verify the validity of the request source |

**HTTP Request Body**

not

**HTTP Response Example**

```json
{
  "url": "https://your-server-domain/path/to/url"
}
```

### Get File Meta Information - Collaborative Document Automatic Task {#file-meta-admin}

This API is requested when an automatic task fetches basic information (currently only cross-table reference tasks).

:::caution
⚠️ The interface is initiated by the Graphite SDK, and for security reasons, it is recommended to verify the HTTP header passed by this interface `X-Shimo-Signature` , and the signature algorithm is consistent with the method used by the accessing party to generate the signature.
:::

**The address of the request**

_GET_ /admin/files/{fileId}

**HTTP Request Headers**

| Header 名               | value  | illustrate                                                                                                                  |
| :---------------------- | :-- | :-------------------------------------------------------------------------------------------------------------------- |
| X-Shimo-Credential-Type | 3   | Unable to provide the [callback credential type](./resources.md##credential-types) for the token                                                    |
| X-Shimo-Signature       |     | Graphite's signature is based on the AppId and Secret information of the service provider, and the payload contains  , `fileId` if it is inconsistent with the request parameters, it means that it is not a legitimate request |

**HTTP Response Body**

| The name of the field      | type   | Examples of values               | illustrate                                                                                   |
| :---------- | :----- | :------------------- | :------------------------------------------------------------------------------------- |
| id          | string | 123                  | The ID of the document in the service provider's system                                                                  |
| name        | string | file title           | The title of the document                                                                               |
| type        | string | documentPro          | For a list of document types, refer to[ Creating a Document](./apis.md#create-collab-file)                               |
| creatorId   | string | 1                    | The user ID of the creator of the ingestion file                                                              |
| createdAt   | string | 2021-08-01T00:00:00Z | The time when the file was created recorded by the access party`UTC` (0 time zone) in the format `2022-01-01T09:00:01Z`of         |
| updatedAt   | string | 2021-08-02T00:00:00Z | The last update time of the file recorded by the access party,`UTC` the time (0 time zone), and the format is `2022-01-01T09:00:01Z`     |
| views       | number | 100                  | The number of times a file is read by the ingestor and is used to display the number of views in the document. If the accessing party does not return, it will be displayed as 1 by default |
| permissions | object |                      | [The user's permission information on the file](#file-permissions)`readable` needs to be `true`                        |

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

### Obtain file meta information based on a specified user - Collaborative Document {#file-meta-by-uid}

This API is requested when the Graphite SDK needs to actively obtain the basic information of the file (currently only cross-table reference scenarios).

:::caution
⚠️ The interface is initiated by the Graphite SDK, and for security reasons, it is recommended to verify the HTTP header passed by this interface `X-Shimo-Signature` , and the signature algorithm is consistent with the method used by the accessing party to generate the signature.
:::

**The address of the request**

_GET_ /admin/files/{fileId}/by-user-id

**HTTP Request Headers**

| Header 名               | value  | illustrate                                                                                                                            |
| :---------------------- | :-- | :------------------------------------------------------------------------------------------------------------------------------ |
| X-Shimo-Credential-Type | 3   | Unable to provide the [callback credential type](./resources.md##credential-types) for the token                                                              |
| X-Shimo-Signature       |     | Graphite's signature is based on the AppId and AppSecret information of the service provider, and the Payload contains `fileId` , if it `userId` is inconsistent with the request parameters, it means that the request is not legitimate |

**HTTP Query Parameters**

| Several participants | value        | illustrate                                               |
| :----- | :-------- | :------------------------------------------------- |
| userId | user12345 | Indicates the file permission of which user ID of the service provider needs to be obtained for the current query |

**HTTP Response Body**

| The name of the field      | type   | Examples of values               | illustrate                                                                                   |
| :---------- | :----- | :------------------- | :------------------------------------------------------------------------------------- |
| id          | string | 123                  | The ID of the document in the service provider's system                                                                  |
| name        | string | file title           | The title of the document                                                                               |
| type        | string | documentPro          | For a list of document types, refer to [Creating a Document](./apis.md#create-collab-file)                              |
| creatorId   | string | 1                    | The user ID of the creator of the ingestion file                                                              |
| createdAt   | string | 2021-08-01T00:00:00Z | The time when the file was created recorded by the access party`UTC` (0 time zone) in the format `2022-01-01T09:00:01Z`of         |
| updatedAt   | string | 2021-08-02T00:00:00Z | The last update time of the file recorded by the access party,`UTC` the time (0 time zone), and the format is `2022-01-01T09:00:01Z`     |
| views       | number | 100                  | The number of times a file is read by the ingestor and is used to display the number of views in the document. If the accessing party does not return, it will be displayed as 1 by default |
| permissions | object |                      | [The user's permission information on the file](#file-permissions)`readable` needs to be `true`                        |

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

## user

### Get current user information {#user-current}

Obtain the ID, name, and address of the current user, and use this API to obtain the user's identity information when creating, editing, and importing

**The address of the request**

_GET_ /users/current/info

**HTTP Request Headers**

| Header 名     | value  | illustrate                                         |
| :------------ | :-- | :------------------------------------------- |
| X-Shimo-Token |     | The token provided by the service provider is used for the service provider to authenticate the request |

**HTTP Response Body**

| The name of the field   | type   | Examples of values                        | illustrate                                                             |
| :------- | :----- | :---------------------------- | :--------------------------------------------------------------- |
| id       | string | userid123                     | The ID of the user of the service provider's system                                            |
| name     | string | Tom                          | The user name of the service provider's system                                             |
| avatar   | string | http://fake.site/user-123.png | The avatar address of the user of the service provider's system                                       |
| email    | string | user123@fake.site             | The email address of the user of the service provider's system                                           |
| teamGuid | string | 123                           | The team ID of the user of the service provider system needs to be returned at the same time as file.teamGuid |

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

### Get the team information of the current user {#user-current-team}

Get information about the team the current user belongs to

**The address of the request**

_GET_ /users/current/team

**HTTP Request Headers**

| Header 名     | value  | illustrate                                         |
| :------------ | :-- | :------------------------------------------- |
| X-Shimo-Token |     | The token provided by the service provider is used for the service provider to authenticate the request |

**HTTP Response Body**

| The name of the field      | type   | Examples of values           | illustrate                           |
| :---------- | :----- | :--------------- | :----------------------------- |
| id          | string | 123              | The ID of the team in the service provider's system          |
| name        | string | XXX Company Efficiency Team | The name of the team in the service provider system         |
| memberCount | number | 99               | The number of all members of the team in the service provider system |

Response Body Example

```json
{
  "id": "123",
  "name": "XXX公司效率团队",
  "memberCount": 99
}
```

### Obtaining the Specified User Information {#user-by-ID}

This API is requested when you obtain the basic information of the user, such as the ID, name, and avatar image address, and obtains the specified user information in the collaboration scenario

**The address of the request**

_GET_ /users/{userId}

**HTTP Request Headers**

| Header 名     | value  | illustrate                                         |
| :------------ | :-- | :------------------------------------------- |
| X-Shimo-Token |     | The token provided by the service provider is used for the service provider to authenticate the request |

**HTTP Response Body**

| The name of the field | type   | Examples of values                        | illustrate                       |
| :----- | :----- | :---------------------------- | :------------------------- |
| id     | string | userid123                     | The ID of the user of the service provider's system      |
| name   | string | Tom                          | The user name of the service provider's system       |
| avatar | string | http://fake.site/user-123.png | The avatar address of the user of the service provider's system |
| email  | string | user123@fake.site             | The email address of the user of the service provider's system     |

Response Body Example

```json
{
  "id": "userid123",
  "name": "张三",
  "avatar": "http://fake.site/user-123.png",
  "email": "user123@fake.site"
}
```

### Get user watermark information {#user-watermark}

When the user opens or previews the document, it will try to obtain the watermark information of the current user for display on the page, and the accessing party can choose to return the corresponding information in the service as the watermark field information, or return empty according to the format, and the watermark information will be hidden on the page.

**The address of the request**

_GET_ /users/{userId}/watermark

**HTTP Request Headers**

| Header 名     | value  | illustrate                                         |
| :------------ | :-- | :------------------------------------------- |
| X-Shimo-Token |     | The token provided by the service provider is used for the service provider to authenticate the request |

**HTTP Response Body**

| The name of the field     | type  | Examples of values                                                | illustrate                                                                                                                             |
| :--------- | :---- | :---------------------------------------------------- | :------------------------------------------------------------------------------------------------------------------------------- |
| watermarks | Array | ["Custom Field 1", "Custom Field 2", "Custom Field 3", ...] | The watermark field returned by the service provider is returned in the form of an array of strings, each of which represents a row. When an** empty array is returned**, the graphite editor will** not **display the watermark information on the editor. |

:::tip
**Suggested content for watermark fields**

Since the watermark field currently uses string passing, there is no limit to the length of this field, for a better watermark experience, we recommend that the return fields follow the following rules:

1. `字段数量`No more than `3` one, that is,`水印内容` no more than `3` one row
2. `每个字段`No more than `20` one word, that is,`水印单行` no more than `20` one word

If you really need it, you can adjust it according to your own needs`行数` or`单行字数`
:::

Response Body Example

```json title="返回自定义水印信息用于展示"
{
  "watermarks": [
    // 结果为字符串数组，按顺序每个单独字符串为一行显示
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

### Get user information in bulk {#users-by-ids}

Provide information about multiple specified users, such as ID, name, and avatar image address. If you want to display the information of multiple users when you want to display the history of multiple users in the sidebar, this API is requested

**The address of the request**

_POST_ /users/batch/get

**HTTP Request Headers**

| Header 名     | value  | illustrate                                         |
| :------------ | :-- | :------------------------------------------- |
| X-Shimo-Token |     | The token provided by the service provider is used for the service provider to authenticate the request |

**HTTP Request Body**

| The name of the field | type     | Examples of values            | illustrate         |
| :----- | :------- | :---------------- | :----------- |
| ids    | []string | ["user1","user2"] | A list of user IDs |

**Request Body Example**

```json
{
  "ids": ["user1", "user2"]
}
```

**HTTP Response Body**

| The name of the field          | type   | Examples of values                        | illustrate                       |
| :-------------- | :----- | :---------------------------- | :------------------------- |
| items[0].id     | string | userid123                     | The ID of the user of the service provider's system      |
| items[0].name   | string | Tom                          | The user name of the service provider's system       |
| items[0].avatar | string | http://fake.site/user-123.png | The avatar address of the user of the service provider's system |
| items[0].email  | string | user123@fake.site             | The email address of the user of the service provider's system     |

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

### Obtain the directory of the user department {#user-department-paths}

Obtain the department path of the specified user, and return an empty array if the user has not been assigned a department

:::info Notes

**Department paths**

Refers  to ** the list of path nodes **from the department to which the user belongs** to the **first-level department

:::

For example:

```json title="用户所在部门结构示例"
XX 公司效率团队      // 团队层级
└── XX 研发部       // 一级部门
    └── 基础设置组
        └── 后端组  // 用户所属部门
```

```json title="该用户应返回的部门路径示例"
["XX 研发部", "基础设置组", "后端组"]
```

**The address of the request**

_GET_ /users/{userId}/department-paths

**HTTP Request Headers**

| Header 名     | value  | illustrate                                         |
| :------------ | :-- | :------------------------------------------- |
| X-Shimo-Token |     | The token provided by the service provider is used for the service provider to authenticate the request |

**HTTP Response Body**

| The name of the field          | type   | Examples of values    | illustrate                   |
| :-------------- | :----- | :-------- | :--------------------- |
| list[0][0].id   | string | 123       | The ID of the department of the service provider's system  |
| list[0][0].name | string | XX R&D Department | The name of the department of the service provider's system |

**Response Body Example**

```json title="用户仅属于一个部门"
[
  [
    { "id": "123", "name": "XX 研发部" },
    { "id": "456", "name": "基础设施组" },
    { "id": "789", "name": "后端组" }
  ]
]
```

```json title="用户仅属于多个部门"
[
  [
    { "id": "123", "name": "XX 研发部" },
    { "id": "456", "name": "基础设施组" },
    { "id": "1789", "name": "后端组" }
  ],
  [
    { "id": "123", "name": "XX 研发部" },
    { "id": "456", "name": "基础设施组" },
    { "id": "2789", "name": "前端端组" }
  ]
]
```

## Teams and departments

### Get the list of members under a team {#team-members}

**The address of the request**

_GET_ /teams/{teamGuid}/members?pagination=true&page={page}&pageSize={pageSize}

**HTTP Request Headers**

| Header 名     | value  | illustrate                                         |
| :------------ | :-- | :------------------------------------------- |
| X-Shimo-Token |     | The token provided by the service provider is used for the service provider to authenticate the request |

**HTTP Query Parameters**

| Several participants     | type    | value   | illustrate           |
| :--------- | :------ | :--- | :------------- |
| pagination | boolean | true | Whether it is a paginated query |
| page       | number  | 1    | Paginate the pages of the query |
| pageSize   | number  | 10   | The size of the pagination       |

**HTTP Response Body**

| The name of the field          | type   | Examples of values                        | illustrate                       |
| :-------------- | :----- | :---------------------------- | :------------------------- |
| items[0].id     | string | userid123                     | The ID of the user of the service provider's system      |
| items[0].name   | string | Tom                          | The user name of the service provider's system       |
| items[0].avatar | string | http://fake.site/user-123.png | The avatar address of the user of the service provider's system |
| items[0].email  | string | user123@fake.site             | The email address of the user of the service provider's system     |

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

### Get department information {#department-info}

:::caution tip

When the department ID is the root node ID, pass the department ID in a special form, reference **[`特殊部门 ID 说明`](./concepts/team_department.md#depatment-id-tips)**. 

:::

**The address of the request**

_GET_ /departments/{departmentId}

**HTTP Request Headers**

| Header 名     | value  | illustrate                                         |
| :------------ | :-- | :------------------------------------------- |
| X-Shimo-Token |     | The token provided by the service provider is used for the service provider to authenticate the request |

**HTTP Response Body**

| The name of the field         | type   | Examples of values | illustrate                           |
| :------------- | :----- | :----- | :----------------------------- |
| id             | string | 123    | The department ID of the service provider's system            |
| name           | string | R&D Department | The name of the department of the service provider system           |
| allMemberCount | number | 99     | The number of all members under the current department of the service provider |

**Response Body Example**

```json
{
  "id": "123",
  "name": "研发部",
  "allMemberCount": 99
}
```

### Obtain the lower-level nodes of the department {#children-departments}

:::caution tip

When the department ID is the root node ID, pass the department ID in a special form, reference **[`特殊部门 ID 说明`](./concepts/team_department.md#depatment-id-tips)**. 

:::

**The address of the request**

_GET_ /departments/{departmentId}/children

**HTTP Request Headers**

| Header 名     | value  | illustrate                                         |
| :------------ | :-- | :------------------------------------------- |
| X-Shimo-Token |     | The token provided by the service provider is used for the service provider to authenticate the request |

**HTTP Response Body**

| The name of the field                  | type   | Examples of values | illustrate                           |
| :---------------------- | :----- | :----- | :----------------------------- |
| items[0].id             | string | 123    | The department ID of the service provider's system            |
| items[0].name           | string | R&D Department | The name of the department of the service provider system           |
| items[0].allMemberCount | number | 99     | The number of all members under the current department of the service provider |

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

### Gets the member page list under the department {#department-members}

:::caution tip

When the department ID is the root node ID, pass the department ID in a special form, reference **[`特殊部门 ID 说明`](./concepts/team_department.md#depatment-id-tips)**. 

:::

**The address of the request**

_GET_ /departments/{departmentId}/members?page={page}&pageSize={pageSize}

**HTTP Request Headers**

| Header 名     | value  | illustrate                                         |
| :------------ | :-- | :------------------------------------------- |
| X-Shimo-Token |     | The token provided by the service provider is used for the service provider to authenticate the request |

**HTTP Query Parameters**

| Several participants   | type   | value  | illustrate           |
| :------- | :----- | :-- | :------------- |
| page     | number | 1   | Paginate the pages of the query |
| pageSize | number | 20  | The size of the pagination       |

**HTTP Response Body**

| The name of the field            | type   | Examples of values                        | illustrate                         |
| :---------------- | :----- | :---------------------------- | :--------------------------- |
| total             | number | 20                            | The total number of members under the department of the service provider system |
| members[0].id     | string | userid123                     | The ID of the user of the service provider's system        |
| members[0].name   | string | Tom                          | The user name of the service provider's system         |
|[0]Members.Awador| string | http://fake.site/user-123.png | The avatar address of the user of the service provider's system   |
| members[0].email  | string | user123@fake.site             | The email address of the user of the service provider's system       |

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

## Search

### Get a list of users related to a file {#users-recent}

This interface is used with the @ function for completion.
When entering @ in a file, display the drop-down menu, select the mentioned user, and the accessor can return it according to your system situation, such as: the list of users most relevant to the current user, the list of users most related to the current file, or an empty array

**The address of the request**

_GET_ /search/users/recent?fileId=file12345

**HTTP Query Parameters**

| Several participants | value        | illustrate                                |
| :----- | :-------- | :---------------------------------- |
| fileId | file12345 | Indicates the file ID of the service provider from which the current query is coming from |

**HTTP Request Headers**

| Header 名     | value  | illustrate                                         |
| :------------ | :-- | :------------------------------------------- |
| X-Shimo-Token |     | The token provided by the service provider is used for the service provider to authenticate the request |

**HTTP Response Body**

| The name of the field          | type   | Examples of values                        | illustrate                       |
| :-------------- | :----- | :---------------------------- | :------------------------- |
| items[0].id     | string | userid123                     | The ID of the user of the service provider's system      |
| items[0].name   | string | Tom                          | The user name of the service provider's system       |
| items[0].avatar | string | http://fake.site/user-123.png | The avatar address of the user of the service provider's system |
| items[0].email  | string | user123@fake.site             | The email address of the user of the service provider's system     |

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

### Get a list of files related to a file {#files-recent}

This interface is used with the @ function for completion.
When entering @ in a file, display the drop-down menu, select the mentioned file, and the accessor can return it according to its own system situation, such as: the list of files most related to the current user, the list of files most related to the current file, or return an empty array

#### Support for returning @ third-party files

:::tip

Since third-party files cannot be collaborated, only the reference can be retained when @ is inserted in the SDK, so the following conditions need to be met when clicking to jump to the third-party page:

1. The file information `type` field value is  , `file`refer to the following Response Example
2. The file information field is added `fullUrl` as the complete address to jump to the access system, refer to the following Response Example
3. When used by the front-end, `shimo-js-sdk` the `connect` Required Implementation `openLink` method is called to control the jump behavior when a link is clicked within the editor.
:::

**The address of the request**

_GET_ /search/files/recent?fileId=file12345

**HTTP Query Parameters**

| Several participants | value        | illustrate                                |
| :----- | :-------- | :---------------------------------- |
| fileId | file12345 | Indicates the file ID of the service provider from which the current query is coming from |

**HTTP Request Headers**

| Header 名     | value  | illustrate                                         |
| :------------ | :-- | :------------------------------------------- |
| X-Shimo-Token |     | The token provided by the service provider is used for the service provider to authenticate the request |

**HTTP Response Body**

| The name of the field               | type   | Examples of values                                   | illustrate                                                                                                      |
| :------------------- | :----- | :--------------------------------------- | :-------------------------------------------------------------------------------------------------------- |
| items[0].id          | string | BA13551165CC5066                         | The ID of the document in the service provider's system                                                                                     |
| items[0].name        | string | file title                               | The title of the document                                                                                                  |
| items[0].type        | string | documentPro                              | The list of document types refers to[ creating a document](./apis.md#create-collab-file), and if it is a non-graphite collaboration file stored by a third party, it will be fixed and `file` uploaded|
| items[0].creatorId   | string | 1                                        | The user ID of the creator of the ingestion file                                                                                 |
| items[0].createdAt   | string | 2021-08-01T00:00:00Z                     | The time when the file was created recorded by the access party`UTC` (0 time zone) in the format `2022-01-01T09:00:01Z`of                            |
| items[0].updatedAt   | string | 2021-08-02T00:00:00Z                     | The last update time of the file recorded by the access party,`UTC` the time (0 time zone), and the format is `2022-01-01T09:00:01Z`                        |
| items[0].views       | number | 100                                      | The number of times a file is read by the ingestor and is used to display the number of views in the document. If the accessing party does not return, it will be displayed as 1 by default                    |
| items[0].permissions | object |                                          | [Information about the user's permissions on the file](#file-permissions)                                                                   |
| items[0].fullUrl     | string | https://customer-system.com/path/to/file | The full address of the access file                                                                                        |

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

### Search file and user lists by keyword {#search-by-keyword}

This interface searches with the @ function to get the results of the drop-down menu to use.
When entering @, if you need to search for users or files based on keywords, the system returns the corresponding file list and user list through the requesting party, and if you do not need it, you can return the corresponding fields to an empty array

#### Support for returning @ third-party files

:::tip Description

Since third-party files cannot be collaborated, only the reference can be retained when @ is inserted in the SDK, so the following conditions need to be met when clicking to jump to the third-party page:

1. The file information `type` field value is  , `file`refer to the following Response Example
2. The file information field is added `fullUrl` as the complete address to jump to the access system, refer to the following Response Example
3. When used by the front-end, `shimo-js-sdk` the `connect` Required Implementation `openLink` method is called to control the jump behavior when a link is clicked within the editor.
:::

**The address of the request**

_POST_ /search

**HTTP Request Headers**

| Header 名     | value  | illustrate                                         |
| :------------ | :-- | :------------------------------------------- |
| X-Shimo-Token |     | The token provided by the service provider is used for the service provider to authenticate the request |

**HTTP Request Body**

| The name of the field   | type   | Examples of values                                                       | illustrate                                                                                                                                |
| :------- | :----- | :----------------------------------------------------------- | :---------------------------------------------------------------------------------------------------------------------------------- |
| fileId   | string | filed1234                                                   | The file ID of the service provider's system                                                                                                                 |
| keyword  | string | test                                                         | The keyword of the file name or user name                                                                                                            |
| page     | number | 0                                                            | The results of the search on the first page                                                                                                                    |
| pageSize | number | 6                                                            | The number of search results per page                                                                                                                    |
| type     | string | file_name,recent_contact,collaborator,team_member,department | Specify which types of search results the search request needs to search, and return different search results on demand based on the type incoming. After splitting by comma, `,` the corresponding result is returned according to the type of hit. |

#### `type` Description of the field

:::tip Description

- `file_name` Indicates that the file is searched by keyword, corresponding to the result in Response `files` . 
- `recent_contact` Indicates the comparison of relevant users through keyword search, corresponding to the results in the response `recentUsers` . 
- `collaborator` Indicates a search for collaborator users by keyword, corresponding to the result in Response `collaborators` . 
- `team_member` Indicates that team members are searched by keyword, corresponding to the results in Response `teamMembers` . 
- `department` Indicates the search department by keyword, corresponding to the result in Response `department` . 

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

| The name of the field                                          | type   | Examples of values                                   | illustrate                                                                                                                                                                                                                                                           |
| :---------------------------------------------- | :----- | :--------------------------------------- | :------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| files.count                                     | number | 3                                        | The total number of search results, including the sum of all paginated results                                                                                                                                                                                                                     |
| files.page                                      | number | 0                                        | The current number of pages is 0, such as the value of the first page                                                                                                                                                                                                                               |
| files.pageSize                                  | number | 6                                        | The number of results for each page of pagination                                                                                                                                                                                                                                         |
| files.pageCount                                 | number | 1                                        | The total number of pages in a search result is calculated based on the pagination size                                                                                                                                                                                                                   |
| files.results[0].id                             | string | BA13551165CC5066                         | The ID of the document in the service provider's system                                                                                                                                                                                                                                          |
| files.results[0].name                           | string | file title                               | The title of the document                                                                                                                                                                                                                                                       |
| files.results[0].type                           | string | documentPro                              | The list of document types refers to[ creating a document](./apis.md#create-collab-file), and if it is a non-graphite collaboration file stored by a third party, it will be fixed and uploaded `file`                                                                                                                                                      |
| files.results[0].creatorId                      | string | 1                                        | The user ID of the creator of the ingestion file                                                                                                                                                                                                                                      |
| files.results[0].createdAt                      | string | 2021-08-01T00:00:00Z                     | The time when the file was created recorded by the access party`UTC` (0 time zone) in the format `2022-01-01T09:00:01Z`of                                                                                                                                                                                 |
| files.results[0].updatedAt                      | string | 2021-08-02T00:00:00Z                     | The last update time of the file recorded by the access party,`UTC` the time (0 time zone), and the format is `2022-01-01T09:00:01Z`                                                                                                                                                                             |
| files.results[0].views                          | number | 100                                      | The number of times a file is read by the ingestor and is used to display the number of views in the document. If the accessing party does not return, it will be displayed as 1 by default                                                                                                                                                                         |
| files.results[0].permissions                    | object |                                          | [Information about the user's permissions on the file](#file-permissions)                                                                                                                                                                                                                        |
| files.results[0].fullUrl                        | string | https://customer-system.com/path/to/file | The full address of the access file                                                                                                                                                                                                                                             |
| recentUsers.count                               | number | 3                                        | The total number of search results, including the sum of all paginated results                                                                                                                                                                                                                     |
| Receptors.page                                | number | 0                                        | The current number of pages is 0, such as the value of the first page                                                                                                                                                                                                                               |
| recentUsers.pageSize                            | number | 6                                        | The number of results for each page of pagination                                                                                                                                                                                                                                         |
| recentUsers.pageCount                           | number | 1                                        | The total number of pages in a search result is calculated based on the pagination size                                                                                                                                                                                                                   |
| recentUsers.results[0].id                       | string | userid123                                | The ID of the user of the service provider's system                                                                                                                                                                                                                                          |
| recentUsers.results[0].name                     | string | Tom                                     | The user name of the service provider's system                                                                                                                                                                                                                                           |
|Receptions.resorts.avatar[0]| string | http://fake.site/user-123.png            | The avatar address of the user of the service provider's system                                                                                                                                                                                                                                     |
| recentUsers.results[0].email                    | string | user123@fake.site                        | The email address of the user of the service provider's system                                                                                                                                                                                                                                         |
| collaborators.count                             | number | 3                                        | The total number of search results, including the sum of all paginated results                                                                                                                                                                                                                     |
| collaborators.page                              | number | 0                                        | The current number of pages is 0, such as the value of the first page                                                                                                                                                                                                                               |
| collaborators.pageSize                          | number | 6                                        | The number of results for each page of pagination                                                                                                                                                                                                                                         |
| collaborators.pageCount                         | number | 1                                        | The total number of pages in a search result is calculated based on the pagination size                                                                                                                                                                                                                   |
| collaborators.results[0].id                     | string | userid123                                | The ID of the user of the service provider's system                                                                                                                                                                                                                                          |
| collaborators.results[0].name                   | string | Tom                                     | The user name of the service provider's system                                                                                                                                                                                                                                           |
|Collaborative.resorts.avatar[0]| string | http://fake.site/user-123.png            | The avatar address of the user of the service provider's system                                                                                                                                                                                                                                     |
| collaborators.results[0].email                  | string | user123@fake.site                        | The email address of the user of the service provider's system                                                                                                                                                                                                                                         |
| teamMembers.count                               | number | 3                                        | The total number of search results, including the sum of all paginated results                                                                                                                                                                                                                     |
| teamMembers.page                                | number | 0                                        | The current number of pages is 0, such as the value of the first page                                                                                                                                                                                                                               |
| teamMembers.pageSize                            | number | 6                                        | The number of results for each page of pagination                                                                                                                                                                                                                                         |
| teamMembers.pageCount                           | number | 1                                        | The total number of pages in a search result is calculated based on the pagination size                                                                                                                                                                                                                   |
| teamMembers.results[0].id                       | string | userid123                                | The ID of the user of the service provider's system                                                                                                                                                                                                                                          |
| teamMembers.results[0].name                     | string | Tom                                     | The user name of the service provider's system                                                                                                                                                                                                                                           |
|Timmers.resorts.avatar[0]| string | http://fake.site/user-123.png            | The avatar address of the user of the service provider's system                                                                                                                                                                                                                                     |
| teamMembers.results[0].email                    | string | user123@fake.site                        | The email address of the user of the service provider's system                                                                                                                                                                                                                                         |
| department.count                                | number | 3                                        | The total number of search results, including the sum of all paginated results                                                                                                                                                                                                                     |
| department.page                                 | number | 0                                        | The current number of pages is 0, such as the value of the first page                                                                                                                                                                                                                               |
| department.pageSize                             | number | 6                                        | The number of results for each page of pagination                                                                                                                                                                                                                                         |
| department.pageCount                            | number | 1                                        | The total number of pages in a search result is calculated based on the pagination size                                                                                                                                                                                                                   |
| department.results[0].id                        | string | 123                                      | The department ID of the service provider's system                                                                                                                                                                                                                                            |
| department.results[0].name                      | string | Tom                                     | The name of the department of the service provider system                                                                                                                                                                                                                                           |
| department.results[0].allMemberCount            | number | 10                                       | The total number of current department members of the service provider system                                                                                                                                                                                                                                   |
| department.results[0].parentDepartments         | Array  |                                          | All parent departments corresponding to the current department data of the service provider system, in order, **`从高到底`** excluding `团队` levels, starting from the first-level department, if it is already a first-level department, an empty array is returned. Since you `表格锁定` may encounter departments with the same name when searching for departments in the scene, you can use all parent departments to help distinguish departments, and if you don't need to do so, you can choose not to return to the list of parent departments. |
| department.results[0].parentDepartments[0].id   | string |                                          | The ID of the parent department of the current department in the service provider's system                                                                                                                                                                                                                              |
| department.results[0].parentDepartments[0].name | name   |                                          | The name of the parent department of the current department in the service provider system                                                                                                                                                                                                                             |

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

## Push Graphite SDK related events {#events}

This interface is used to send a notification to the service provider when a graphite file triggers a specific behavior.

Events that will be pushed so far:

- Comment
- Discussion
- MentionAt (MentionAt @人)
- DateMention (DateMention)
- File Content Update (FileContent)
- Document collaborator collaboration status changes
- Revision
- System Events
  - The callback request is incorrect

**The address of the request**

_POST_ /events

**HTTP Request Headers**

| Header 名         | value  | illustrate                                         |
| :---------------- | :-- | :------------------------------------------- |
| X-Shimo-Token     |     | The token provided by the service provider is used for the service provider to authenticate the request |
| X-Shimo-Sdk-Event |     | Used to identify specific event types                       |

Request parameters and request bodies vary depending on the event.

### **Comment （Comment）**

When a comment is created, a comment is updated, a comment is deleted, or a comment is closed, and the corresponding event push is triggered.

**HTTP Request Headers**

| Header 名         | value      | illustrate                   |
| :---------------- | :------ | :--------------------- |
| X-Shimo-Sdk-Event | Comment | Used to identify specific event types |

#### Light documents

##### Add a comment

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

##### Delete the comment

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

##### End comment

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

#### form

##### Add a comment

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

##### Delete the comment

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

##### End comment

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

#### Traditional documentation

##### Add a comment

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

##### Update the review

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

##### Delete the comment

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

#### slide

##### Add a comment

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

##### Delete the comment

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

##### End comment

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

#### Application tables

##### Add a comment

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

```json title="对于评论的回复评论"
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

##### Delete the comment

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

### ** Discussion （Discussion）**

When a discussion message is created, an event push is triggered.

**HTTP Request Headers**

| Header 名         | value         | illustrate                   |
| :---------------- | :--------- | :--------------------- |
| X-Shimo-Sdk-Event | Discussion | Used to identify specific event types |

#### Light documents

##### Send a discussion message

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

### **MentionAt(@MentionAt)**

This event is triggered when one or more users are @ in a Comment, Discussion message.

**A list of @ user IDs, which** are passed to the accessing party **by the SDK in the event data**. **

**HTTP Request Headers**

| Header 名         | value        | illustrate                   |
| :---------------- | :-------- | :--------------------- |
| X-Shimo-Sdk-Event | MentionAt | Used to identify specific event types |

#### Light documents

##### In the comments at

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

##### In the discussion at

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

##### In the body at

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

#### form

Discussion is not supported at this time

##### In the comments at

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

##### In the body at

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

#### Traditional documentation

Discussion is not supported at this time

##### In the comments at

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

##### In the body at

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

#### slide

AT is not supported

#### Application tables

Discussion is not supported at this time

##### In the comments at

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

##### In the body at

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

### **DateMention (DateMention)**

When a date reminder rule is inserted into a file, the rule information is pushed to the access party, so that the access party can implement the regular reminder function in the business system based on the creation, update, and deletion events as needed.

:::caution
⚠️ The Graphite SDK will **`立即`**  push `创建`the , , and`修改``删除` events to the accessing party after the operation on the page, and the regular reminder function needs to be implemented by the business system, and the Graphite SDK will not send any event notification when the target time is reached.
:::

**HTTP Request Headers**

| Header 名         | value          | illustrate                   |
| :---------------- | :---------- | :--------------------- |
| X-Shimo-Sdk-Event | DateMention | Used to identify specific event types |

#### Light documents

HTTP Request Body

##### create

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

##### revise

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

##### Delete

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

#### form

HTTP Request Body

##### create

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

##### revise

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

##### Delete

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

#### Traditional documentation

HTTP Request Body

##### create

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

##### revise

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

##### Delete

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

#### slide

Not at this time

### **File Content Update (FileContent)**

This event is triggered when the file is modified by a user and processed by the server.

**HTTP Request Headers**

| Header 名               | value                                   | illustrate                                                                                            |
| :---------------------- | :----------------------------------- | :---------------------------------------------------------------------------------------------- |
| X-Shimo-Credential-Type | 3                                    | Unable to provide the [callback credential type](./resources.md##credential-types) for the token                              |
| X-Shimo-Sdk-Event       | FileContent                          | Used to identify specific event types                                                                          |
| X-Shimo-Signature       | ebc1cde3-9b57-4962-883d-54302d428600 | When graphite cannot provide the accessor token, the corresponding appId and secret of the accessor are used to generate a signature for callback interface verification |

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

### **Document Collaborator Collaboration Status Change**

This event is pushed when the current users of a document are changed.

Events can `action` distinguish user behavior by the value of

- `enter` If there is no abnormality in the network, enter the document and trigger this event
- `leave` This event is triggered when a user closes a document or when the network is disconnected

**HTTP Request Headers**

| Header 名               | value                                   | illustrate                                                                                            |
| :---------------------- | :----------------------------------- | :---------------------------------------------------------------------------------------------- |
| X-Shimo-Credential-Type | 3                                    | Unable to provide the [callback credential type](./resources.md##credential-types) for the token                              |
| X-Shimo-Sdk-Event       | Collaborator                         | Used to identify specific event types                                                                          |
| X-Shimo-Signature       | ebc1cde3-9b57-4962-883d-54302d428600 | When graphite cannot provide the accessor token, the corresponding appId and secret of the accessor are used to generate a signature for callback interface verification |

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

### **Revision (Revision)** {#events-revision}

This event is pushed when a user creates, modifies, or deletes a version and is successfully processed by the server.

Events can `action` distinguish user behavior by the value of

- `create` This event is triggered when a user creates a version
- `update` This event is triggered when a user modifies a version
- `delete` This event is triggered when a user deletes a version

**HTTP Request Headers**

| Header 名         | value       | illustrate                   |
| :---------------- | :------- | :--------------------- |
| X-Shimo-Sdk-Event | Revision | Used to identify specific event types |

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

### **System event {** #events-system}

Push SDK system-level events, which are currently experimental and subject to future adjustments or changes.

**HTTP Request Headers**

| Header 名         | value     | illustrate                   |
| :---------------- | :----- | :--------------------- |
| X-Shimo-Sdk-Event | System | Used to identify specific event types |

#### The (experimental) callback request is incorrect

When the SDK requests that the accessing interface is abnormal, `请求数据` the SDK  pushes and and `响应数据` SDK errors to the accessing interface through this message to discover the accessing party's own callback interface problems in a timely manner. 

:::caution Instructions for use

This event contains only** some requested **error messages, and does not include all scenarios, so it is not recommended to rely on this event as the basis for whether the callback is normal. 

:::

**HTTP Request Body**

For more information about Request Headers, see [Callback Credential Types](./resources.md#credential-types) 

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
