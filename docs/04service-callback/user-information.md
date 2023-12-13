---
title: 用户信息
sidebar_position: 5
---

## 获取当前用户信息{#user-current}

获取当前用户的 ID、名称、头像图片地址等信息，创建、编辑、导入时需使用此接口获取用户身份信息。

**请求地址**

*GET* /users/current/info

**HTTP Request Headers**

|**Header 名**|**值**|**说明**|
|:----|:----|:----|
|X-Shimo-Token|接入方提供的 Token|用于接入方对本次请求鉴权|

**HTTP Response Body**

|字段名|类型|值示例|说明|
|:----|:----|:----|:----|
|id|string|userid123|接入方系统的用户 ID|
|name|string|张三|接入方系统的用户名称|
|avatar|string|[http://fake.site/user-123.png](http://fake.site/user-123.png)|接入方系统的用户头像地址|
|email|string|[user123@fake.site](mailto:user123@fake.site)|接入方系统的用户邮箱|
|teamGuid|string|123|接入方系统的用户所属团队 ID，返回时需和 file.teamGuid 同时实现|

**Response Body Example**

```json
{
  "id": "userid123",
  "name": "张三",
  "avatar": "http://fake.site/user-123.png",
  "email": "user123@fake.site",
  "teamGuid": "123"
}
```

## 获取当前用户所在团队信息{#user-current-team}

获取当前用户所在的团队信息。

**请求地址**

*GET* /users/current/team

**HTTP Request Headers**

|**Header 名**|**值**|**说明**|
|:----|:----|:----|
|X-Shimo-Token|接入方提供的 Token|用于接入方对本次请求鉴权|

**HTTP Response Body**

|字段名|类型|值示例|说明|
|:----|:----|:----|:----|
|id|string|123|接入方系统的团队 ID|
|name|string|XXX公司效率团队|接入方系统的团队名称|
|memberCount|number|99|接入方系统的团队所有成员数量|

**Response Body Example**

```json
{
  "id": "123",
  "name": "XXX公司效率团队",
  "memberCount": 99
}
```

## 获取指定用户信息{#user-by-id}

获取用户的基本信息，ID、名称、头像图片地址等，在获取协作场景下的指定用户信息时会请求此接口。

**请求地址**

*GET* /users/{userId}

**HTTP Request Headers**

|**Header 名**|**值**|**说明**|
|:----|:----|:----|
|X-Shimo-Token|接入方提供的 Token|用于接入方对本次请求鉴权|

**HTTP Response Body**

|字段名|类型|值示例|说明|
|:----|:----|:----|:----|
|id|string|userid123|接入方系统的用户 ID|
|name|string|张三|接入方系统的用户名称|
|avatar|string|[http://fake.site/user-123.png](http://fake.site/user-123.png)|接入方系统的用户头像地址|
|email|string|[user123@fake.site](mailto:user123@fake.site)|接入方系统的用户邮箱|

**Response Body Example**

```json
{
  "id": "userid123",
  "name": "张三",
  "avatar": "http://fake.site/user-123.png",
  "email": "user123@fake.site"
}
```

## 获取用户水印信息{#user-watermark}

在用户打开或者预览文档时，会尝试获取当前用户的水印信息用于在页面上展示，接入方可选择返回业务中相应信息作为水印字段信息，也可按照格式返回空，页面上将隐藏水印信息。

**请求地址**

*GET* /users/{userId}/watermark

**HTTP Request Headers**

|**Header 名**|**值**|**说明**|
|:----|:----|:----|
|X-Shimo-Token|接入方提供的 Token|用于接入方对本次请求鉴权|

**HTTP Response Body**

|字段名|类型|值示例|说明|
|:----|:----|:----|:----|
|watermarks|Array|["自定义字段1", "自定义字段2", "自定义字段3", ...]|接入方返回的水印字段，以字符串数组的形式返回，每个字符串代表一行。返回空数组时，石墨编辑器将不会在编辑器上展示水印信息。|

:::info 提示

由于水印字段目前使用字符串传递，目前未限制此字段长度，为了更好的水印体验，建议返回字段遵循如下规则：

- `字段数量`不超过 `3` 个，即`水印内容`不超过 `3` 行
- `每个字段`不超过 `20` 个字，即`水印单行`内容不超过 `20` 字
若确实有需要，可按照自身需求调整`行数`或`单行字数`。

:::

**Response Body Example**

返回自定义水印信息用于展示

```json
{
  "watermarks": [ // 结果为字符串数组，按顺序每个单独字符串为一行显示
    "张三",
    "userid",
    "XX团队-部门-子部门"
  ]
}
```

返回空信息用于隐藏对应用户水印信息

```json
{
  "watermarks": [] // 空数组
}
```

## 批量获取用户信息{#users-by-ids}

获取多个指定用户的信息，ID、名称、头像图片地址等。如多人协作时，侧边栏历史展示时需要展示多个用户的相关信息，此时会请求此接口。

**请求地址**

*POST* /users/batch/get

**HTTP Request Headers**

|**Header 名**|**值**|**说明**|
|:----|:----|:----|
|X-Shimo-Token|接入方提供的 Token|用于接入方对本次请求鉴权|

**HTTP Request Body**

|字段名|类型|值示例|说明|
|:----|:----|:----|:----|
|ids|[]string|["user1","user2"]|用户 ID 列表|

**HTTP Response Body**

|字段名|类型|值示例|说明|
|:----|:----|:----|:----|
|items[0].id|string|userid123|接入方系统的用户的 ID|
|items[0].name|string|张三|接入方系统的用户名称|
|items[0].avatar|string|[http://fake.site/user-123.png](http://fake.site/user-123.png)|接入方系统的用户的头像地址|
|items[0].email|string|[user123@fake.site](mailto:user123@fake.site)|接入方系统的用户的邮箱|

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

## 获取用户部门路径{#user-department-paths}

获取指定用户的部门路径，若用户尚未分配部门，则返回空数组。

:::caution 说明

部门路径是指从用户所属部门到一级部门的路径节点列表。

:::

例如：

用户所在部门结构示例

```plain
XX 公司效率团队      // 团队层级
└── XX 研发部       // 一级部门
    └── 基础设置组
        └── 后端组  // 用户所属部门
```

该用户应返回的部门路径示例

```plain
[
  "XX 研发部",
  "基础设置组",
  "后端组"
]
```

**请求地址**

*GET* /users/{userId}/department-paths

**HTTP Request Headers**

|**Header 名**|**值**|**说明**|
|:----|:----|:----|
|X-Shimo-Token|接入方提供的 Token|用于接入方对本次请求鉴权|

**HTTP Response Body**

|字段名|类型|值示例|说明|
|:----|:----|:----|:----|
|list[0][0].id|string|123|接入方系统的部门 ID|
|list[0][0].name|string|XX 研发部|接入方系统的部门名称|

**Response Body Example**

用户仅属于一个部门

```json
[
  [
    {"id": "123", "name": "XX 研发部"},
    {"id": "456", "name": "基础设施组"},
    {"id": "789", "name": "后端组"}
  ]
]
```

用户属于多个部门

```json
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



