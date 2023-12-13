---
title: 团队和部门
sidebar_position: 6
---
## 获取团队下的成员列表{#team-members}

**请求地址**

*GET* /teams/{teamGuid}/members?pagination=true&page={page}&pageSize={pageSize}

**HTTP Request Headers**

|**Header 名**|**值**|**说明**|
|:----|:----|:----|
|X-Shimo-Token|接入方提供的 Token|用于接入方对本次请求鉴权|

**HTTP Query Parameters**

|参数名|类型|值|说明|
|:----|:----|:----|:----|
|pagination|boolean|true|是否为分页查询|
|page|number|1|分页查询第几页|
|pageSize|number|10|分页大小|

**HTTP Response Body**

|字段名|类型|值示例|说明|
|:----|:----|:----|:----|
|items[0].id|string|userid123|接入方系统的用户 ID|
|items[0].name|string|张三|接入方系统的用户名称|
|items[0].avatar|string|[http://fake.site/user-123.png](http://fake.site/user-123.png)|接入方系统的用户头像地址|
|items[0].email|string|[user123@fake.site](mailto:user123@fake.site)|接入方系统的用户邮箱|

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

## 获取部门信息{#department-info}

:::info 提示

当部门 ID 为根节点 ID 时，以特殊形式传递部门 ID ，参考 `特殊部门 ID 说明`。

:::

**请求地址**

*GET* /departments/{departmentId}

**HTTP Request Headers**

|**Header 名**|**值**|**说明**|
|:----|:----|:----|
|X-Shimo-Token|接入方提供的 Token|用于接入方对本次请求鉴权|

**HTTP Response Body**

|字段名|类型|值示例|说明|
|:----|:----|:----|:----|
|id|string|123|接入方系统的部门 ID|
|name|string|研发部|接入方系统的部门名称|
|allMemberCount|number|99|接入方当前部门下的所有成员数量|

**Response Body Example**

```json
{
  "id": "123",
  "name": "研发部",
  "allMemberCount": 99
}
```

## 获取部门的下级部门节点{#children-departments}

:::info 提示

当部门 ID 为根节点 ID 时，以特殊形式传递部门 ID ，参考 `特殊部门 ID 说明`。

:::

**请求地址**

*GET* /departments/{departmentId}/children

**HTTP Request Headers**

|**Header 名**|**值**|**说明**|
|:----|:----|:----|
|X-Shimo-Token|接入方提供的 Token|用于接入方对本次请求鉴权|

**HTTP Response Body**

|字段名|类型|值示例|说明|
|:----|:----|:----|:----|
|items[0].id|string|123|接入方系统的部门 ID|
|items[0].name|string|研发部|接入方系统的部门名称|
|items[0].allMemberCount|number|99|接入方当前部门下的所有成员数量|

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

## 获取部门下的成员分页列表{#department-members}

:::info 提示

当部门 ID 为根节点 ID 时，以特殊形式传递部门 ID ，参考 `特殊部门 ID 说明`。

:::

**请求地址**

*GET* /departments/{departmentId}/members?page={page}&pageSize={pageSize}

**HTTP Request Headers**

|**Header 名**|**值**|**说明**|
|:----|:----|:----|
|X-Shimo-Token|接入方提供的 Token|用于接入方对本次请求鉴权|

**HTTP Query Parameters**

|参数名|类型|值|说明|
|:----|:----|:----|:----|
|page|number|1|分页查询第几页|
|pageSize|number|20|分页大小|

**HTTP Response Body**

|字段名|类型|值示例|说明|
|:----|:----|:----|:----|
|total|number|20|接入方系统部门下的成员总数|
|members[0].id|string|userid123|接入方系统的用户的 ID|
|members[0].name|string|张三|接入方系统的用户名称|
|members[0].avatar|string|[http://fake.site/user-123.png](http://fake.site/user-123.png)|接入方系统的用户头像地址|
|members[0].email|string|[user123@fake.site](mailto:user123@fake.site)|接入方系统的用户邮箱|

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

