---
sidebar_position: 2
toc_max_heading_level: 4
description: >-
  石墨 SDK 支持两种鉴权方式,包括接入服务商提供的token和用户身份信息。
  接入サービス商使用从石墨获取的AppId和AppSecret,根据签名方法生成的字符串。
  接出服务商从石微获取的appid,获取app详情,更新app回调地址,创建席位管理,获取用户列表和席位状态。
---

# 系统接口

## 鉴权方式 {#auth-methods}

<!-- 石墨 SDK 支持两种鉴权方式： -->

<!-- - [header](#auth-with-header)，推荐使用，文档中示例代码主要用此方式。 -->

- [query string](#auth-with-querystring)

<!-- ## Header 鉴权方式 {#auth-with-header}

| 参数名    | 类型   | 值示例                                  | 必选 | 说明                                                                            |
|:----------|:-------|:-------------------------------------|:----|:------------------------------------------------------------------------------|
| X-Shimo-Token     | string | eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9 | 是   | 接入服务商提供的 token，在石墨请求接入服务商的接口时，会作为 `X-Shimo-Token` 的值放进 HTTP Header 里          |
| X-Shimo-Signature | string | 参考[签名方法](../resources.md#signature)  | 是   | 接入服务商，使用从石墨获取的 AppId 、AppSecret 根据签名方法生成的字符串，详见[签名](../resources#signature)一节 |

**请求示例**

<Tabs
  defaultValue="curl"
  values={[
    { label: 'cURL', value: 'curl', },
    { label: 'Java', value: 'java', },
    { label: 'Golang', value: 'go', },
    { label: 'Node.js', value: 'js', },
  ]
}>
<TabItem value="curl">

```bash
curl --request GET 'https://shimo-domain/sdk/v2/api/fake-api' \
--header 'Content-Type: application/json' \
--header 'X-Shimo-Signature: your_signature' \
--header 'X-Shimo-Token: your_token'
```

</TabItem>
<TabItem value="java">

```java
OkHttpClient client = new OkHttpClient().newBuilder()
  .build();
Request request = new Request.Builder()
  .url("https://shimo-domain/sdk/v2/api/fake-api")
  .addHeader("Content-Type", "application/json")
  .addHeader("X-Shimo-Signature", "your_signature")
  .addHeader("X-Shimo-Token", "your_token")
  .build();
Response response = client.newCall(request).execute();
```

</TabItem>
<TabItem value="go">

```go
package main

import (
  "fmt"
  "strings"
  "net/http"
  "io/ioutil"
)

func main() {
  url := "https://shimo-domain/sdk/v2/fake-api"
  method := "GET"

  client := &http.Client {
  }
  req, err := http.NewRequest(method, url, nil)
  if err != nil {
    fmt.Println(err)
    return
  }

  req.Header.Set("Content-Type", "application/json")
  req.Header.Set("X-Shimo-Signature", "your_signature")
  req.Header.Set("X-Shimo-Token", "your_token")

  res, err := client.Do(req)
  if err != nil {
    fmt.Println(err)
    return
  }
  defer res.Body.Close()

  body, err := ioutil.ReadAll(res.Body)
  if err != nil {
    fmt.Println(err)
    return
  }
  fmt.Println(string(body))
}
```

</TabItem>
<TabItem value="js">

```js
var axios = require('axios');

var config = {
  method: 'GET',
  url: 'https://shimo-domain/sdk/v2/api/fake-api',
  headers: {
    'Content-Type': 'application/json',
    'X-Shimo-Signature': 'your_signature',
    'X-Shimo-Token': 'your_token'
  }
};

axios(config)
  .then(function (response) {
    console.log(JSON.stringify(response.data));
  })
  .catch(function (error) {
    console.log(error);
  });
```

</TabItem>
</Tabs> -->

### Query string 鉴权方式 {#auth-with-querystring}

| 参数名    | 类型   | 值示例                               | 必选 | 说明                                                                                                            |
| :-------- | :----- | :----------------------------------- | :--- | :-------------------------------------------------------------------------------------------------------------- |
| token     | string | eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9 | 是   | 接入服务商提供的 token，在石墨请求接入服务商的接口时，会放到 HTTP Headers X-Shimo-Token 中作为值进行传递        |
| signature | string | 参考签名方法一节                     | 是   | 接入服务商，使用从石墨获取的 AppId 、AppSecret 根据签名方法生成的字符串，详见[签名](../resources#signature)一节 |
| appId     | string | ebc1cde3-9b57-4962-883d-54302d428600 | 否   | **已废弃，可传可不传**。接入服务商从石墨获取的 AppId                                                            |

**请求示例**

<Tabs
defaultValue="curl"
values={[
{ label: 'cURL', value: 'curl', },
{ label: 'Java', value: 'java', },
{ label: 'Golang', value: 'go', },
{ label: 'Node.js', value: 'js', },
]
}>
<TabItem value="curl">

```bash
curl --request GET 'https://shimo-domain/sdk/v2/api/fake-api?signature=your_signature&token=your_token'
```

</TabItem>
<TabItem value="java">

```java
OkHttpClient client = new OkHttpClient().newBuilder()
  .build();
Request request = new Request.Builder()
  .url("https://shimo-domain/sdk/v2/api/fake-api?signature=your_signature&token=your_token")
  .build();
Response response = client.newCall(request).execute();
```

</TabItem>
<TabItem value="go">

```go
package main

import (
  "fmt"
  "strings"
  "net/http"
  "io/ioutil"
)

func main() {
  url := "https://shimo-domain/sdk/v2/api/fake-api?signature=your_signature&token=your_token"
  method := "GET"

  client := &http.Client {
  }
  req, err := http.NewRequest(method, url, nil)
  if err != nil {
    fmt.Println(err)
    return
  }

  res, err := client.Do(req)
  if err != nil {
    fmt.Println(err)
    return
  }
  defer res.Body.Close()

  body, err := ioutil.ReadAll(res.Body)
  if err != nil {
    fmt.Println(err)
    return
  }
  fmt.Println(string(body))
}
```

</TabItem>
<TabItem value="js">

```js
var axios = require("axios");

var config = {
  method: "GET",
  url: "https://shimo-domain/sdk/v2/api/fake-api?signature=your_signature&token=your_token",
};

axios(config)
  .then(function (response) {
    console.log(JSON.stringify(response.data));
  })
  .catch(function (error) {
    console.log(error);
  });
```

</TabItem>
</Tabs>

## 账号与应用管理

### 获取 app 详情 {#get-app-detail}

获取 app 详细信息，包括 app 名称、可使用文件类型、已激活用户数、用户总数、席位数、license 有效时间、回调地址信息。

:::caution 提醒

由于 app API 的特殊性，为了安全考虑，需要您在[生成签名](../resources.md#签名)时附加上 `"scope": "license"` 字段

:::

**请求地址**

_GET_ https://shimo-domain/sdk/v2/api/license/apps/{appId}

**请求示例**
<Tabs
defaultValue="curl"
values={[
{ label: 'cURL', value: 'curl', },
{ label: 'Java', value: 'java', },
{ label: 'Golang', value: 'go', },
{ label: 'Node.js', value: 'js', },
]
}>
<TabItem value="curl">

```bash
curl "https://shimo-domain/sdk/v2/api/license/apps/yourShimoAppId" \
--header 'X-Shimo-Signature: your_signature' \
--header 'X-Shimo-Token: your_token'
```

</TabItem>
<TabItem value="java">

```java
OkHttpClient client = new OkHttpClient().newBuilder()
  .build();
Request request = new Request.Builder()
  .url("https://shimo-domain/sdk/v2/api/license/apps/yourShimoAppId")
  .addHeader("X-Shimo-Signature", "your_signature")
  .addHeader("X-Shimo-Token", "your_token")
  .build();
Response response = client.newCall(request).execute();
```

</TabItem>
<TabItem value="go">

```go
package main

import (
	"fmt"
	"io/ioutil"
	"net/http"
)

func main() {
  url := "https://shimo-domain/sdk/v2/api/license/apps/yourShimoAppId"
  method := "GET"

  client := &http.Client {
  }
  req, err := http.NewRequest(method, url, nil)
  if err != nil {
    fmt.Println(err)
    return
  }

  req.Header.Set("X-Shimo-Signature", "your_signature")
  req.Header.Set("X-Shimo-Token", "your_token")

  res, err := client.Do(req)
  if err != nil {
    fmt.Println(err)
    return
  }
  defer res.Body.Close()

  body, err := ioutil.ReadAll(res.Body)
  if err != nil {
    fmt.Println(err)
    return
  }
  fmt.Println(string(body))
}
```

</TabItem>
<TabItem value="js">

```js
var axios = require("axios");

axios({
  method: "GET",
  url: "https://shimo-domain/sdk/v2/api/license/apps/yourShimoAppId",
  headers: {
    "X-Shimo-Signature": "your_signature",
    "X-Shimo-Token": "your_token",
  },
})
  .then(function (response) {
    console.log(JSON.stringify(response.data));
  })
  .catch(function (error) {
    console.log(error);
  });
```

</TabItem>
</Tabs>

**HTTP 状态码**

| 状态码 | 说明     |
| :----- | :------- |
| 200    | 返回成功 |

**HTTP Response Body**

| 字段名             | 类型   | 值示例                                                          | 说明                                                                            |
| :----------------- | :----- | :-------------------------------------------------------------- | :------------------------------------------------------------------------------ |
| appName            | string | your app name                                                   | 应用名                                                                          |
| availableFileTypes | Array  | ["document","documentPro","spreadsheet","presentation","table"] | 可用石墨套件列表                                                                |
| activatedUserCount | number | 43                                                              | 已激活席位用户数                                                                |
| userCount          | number | 50                                                              | 用户数总数，包含`已激活`、`已禁用`、`未使用` 用户总数，仅 `已激活` 数量占用席位 |
| memberLimit        | number | 99                                                              | license 席位限制用户数，即 `已激活` 用户数量最大限制                            |
| validFrom          | string | 2020-12-31T16:00:00Z                                            | license 生效时间                                                                |
| validUntil         | string | 2022-12-30T16:00:00Z                                            | license 有效期截止时间                                                          |
| endpointUrl        | string | http://your-domain                                              | 服务商回调地址                                                                  |

Example

```json
{
  "appName": "your app name",
  "availableFileTypes": [
    "document",
    "documentPro",
    "spreadsheet",
    "presentation",
    "table"
  ],
  "activatedUserCount": 43,
  "userCount": 50,
  "memberLimit": 99,
  "validFrom": "2020-12-31T16:00:00Z",
  "validUntil": "2022-12-30T16:00:00Z",
  "endpointUrl": "http://your-domain"
}
```

### 更新 app 回调地址 {#update-app-endpoint-url}

用于更新 app 对应服务商回调地址

:::caution 提醒

由于 app API 的特殊性，为了安全考虑，需要您在[生成签名](../resources.md#签名)时附加上 `"scope": "license"` 字段

:::

**请求地址**

_PUT_ https://shimo-domain/sdk/v2/api/license/apps/{appId}/endpoint-url

**HTTP Request Body**

| 字段名 | 类型   | 值示例             | 必选 | 说明                     |
| :----- | :----- | :----------------- | :--- | :----------------------- |
| url    | string | http://your-domain | 是   | 需要更新的服务商回调地址 |

**请求示例**
<Tabs
defaultValue="curl"
values={[
{ label: 'cURL', value: 'curl', },
{ label: 'Java', value: 'java', },
{ label: 'Golang', value: 'go', },
{ label: 'Node.js', value: 'js', },
]
}>
<TabItem value="curl">

```bash
curl -X PUT "https://shimo-domain/sdk/v2/api/license/apps/yourShimoAppId/endpoint-url" \
--header "Content-Type: application/json" \
--header "X-Shimo-Signature: your_signature" \
--header 'X-Shimo-Token: your_token' \
--data-raw $'{"url":"http://your-domain"}'
```

</TabItem>
<TabItem value="java">

```java
OkHttpClient client = new OkHttpClient().newBuilder()
  .build();
MediaType mediaType = MediaType.parse("application/json");
RequestBody body = RequestBody.create(mediaType, "{\"url\":\"http://your-domain\"}");
Request request = new Request.Builder()
  .url("https://shimo-domain/sdk/v2/api/license/apps/yourShimoAppId/endpoint-url")
  .method("PUT", body)
  .addHeader("Content-Type", "application/json")
  .addHeader("X-Shimo-Signature", "your_signature")
  .addHeader("X-Shimo-Token", "your_token")
  .build();
Response response = client.newCall(request).execute();
```

</TabItem>
<TabItem value="go">

```go
package main

import (
  "fmt"
  "strings"
  "net/http"
  "io/ioutil"
)

func main() {
  url := "https://shimo-domain/sdk/v2/api/license/apps/yourShimoAppId/endpoint-url"
  method := "PUT"

  payload := strings.NewReader(`{"url":"http://your-domain"}`)

  client := &http.Client {
  }
  req, err := http.NewRequest(method, url, payload)

  if err != nil {
    fmt.Println(err)
    return
  }
  req.Header.Set("Content-Type", "application/json")
  req.Header.Set("X-Shimo-Signature", "your_signature")
  req.Header.Set("X-Shimo-Token", "your_token")

  res, err := client.Do(req)
  if err != nil {
    fmt.Println(err)
    return
  }
  defer res.Body.Close()

  body, err := ioutil.ReadAll(res.Body)
  if err != nil {
    fmt.Println(err)
    return
  }
  fmt.Println(string(body))
}
```

</TabItem>
<TabItem value="js">

```js
var axios = require("axios");
var data = JSON.stringify({
  url: "http://your-domain",
});

var config = {
  method: "PUT",
  url: "https://shimo-domain/sdk/v2/api/license/apps/yourShimoAppId/endpoint-url",
  headers: {
    "Content-Type": "application/json",
    "X-Shimo-Signature": "your_signature",
    "X-Shimo-Token": "your_token",
  },
  data: data,
};

axios(config)
  .then(function (response) {
    console.log(JSON.stringify(response.data));
  })
  .catch(function (error) {
    console.log(error);
  });
```

</TabItem>
</Tabs>

**HTTP 状态码**

| 状态码 | 说明     |
| :----- | :------- |
| 204    | 创建成功 |

**HTTP Response Body**

空

## 席位管理

:::caution
由于席位管理 API 的特殊性，为了安全考虑，需要您在[生成签名](../resources.md#签名)时附加上 `"scope": "license"` 字段。
:::

### 用户席位状态说明 {#user-status-desc}

目前有三种席位状态：

- `1` 激活
- `0` 已禁用
- `-1` 未启用

**`-1 未启用` 状态何时会存在？**

若接入方已集成实现[`@ 人 / 提及人`](../impl#feature-mention-at)相关功能时，在调用接入方搜索接口时，可能会返回石墨 SDK 中未关联的用户，由于需要在石墨编辑器中的 at 下拉列表中展示此类用户，石墨 SDK 会首先进行建立用户关联操作，此时用户的状态将会初始化为`未启用`状态，此状态 **`不会占用`** 席位数量。

**`-1 未启用` 状态自动激活**

当用户处于未启用状态时，当通过此用户身份进行如下操作时，将会自动激活，成为 `激活` 状态，此时占用相应席位，若席位不足，则会被拒绝访问:

- 创建文件
- 创建副本
- 访问编辑页
- 导入、导出文件
- 预览文件
- 调用表格内容操作接口
- 获取文件侧边栏历史信息、版本信息
- 获取文件纯文本信息
- 获取文件 at 人列表
- 获取文件评论数据

### 获取用户列表和席位状态

**请求地址**

_GET_ http(s)://shimo-domain/sdk/v2/api/license/users

**HTTP Request Parameters**

| 传参方式 | 参数名 | 类型   | 值示例 | 必选 | 说明             |
| :------- | :----- | :----- | :----- | :--- | :--------------- |
| Query    | page   | number | 1      | 是   | 请求页码         |
| Query    | size   | number | 30     | 是   | 单页返回用户数量 |

**请求示例**
<Tabs
defaultValue="curl"
values={[
{ label: 'cURL', value: 'curl', },
{ label: 'Java', value: 'java', },
{ label: 'Golang', value: 'go', },
{ label: 'Node.js', value: 'js', },
]
}>
<TabItem value="curl">

```bash
curl --request GET 'https://shimo-domain/sdk/v2/api/license/users?page=1&size=30' \
--header 'X-Shimo-Signature: your_signature' \
--header 'X-Shimo-Token: your_token'
```

</TabItem>
<TabItem value="java">

```java
OkHttpClient client = new OkHttpClient().newBuilder()
  .build();
Request request = new Request.Builder()
  .url("https://shimo-domain/sdk/v2/api/license/users?page=1&size=30")
  .method("GET", null)
  .addHeader("X-Shimo-Signature", "your_signature")
  .addHeader("X-Shimo-Token", "your_token")
  .build();
Response response = client.newCall(request).execute();
```

</TabItem>
<TabItem value="go">

```go
package main

import (
  "fmt"
  "net/http"
  "io/ioutil"
)

func main() {
  url := "https://shimo-domain/sdk/v2/api/license/users?page=1&size=30"
  method := "GET"

  client := &http.Client {
  }
  req, err := http.NewRequest(method, url, nil)
  if err != nil {
    fmt.Println(err)
    return
  }

  req.Header.Set("X-Shimo-Signature", "your_signature")
  req.Header.Set("X-Shimo-Token", "your_token")

  res, err := client.Do(req)
  if err != nil {
    fmt.Println(err)
    return
  }
  defer res.Body.Close()

  body, err := ioutil.ReadAll(res.Body)
  if err != nil {
    fmt.Println(err)
    return
  }
  fmt.Println(string(body))
}
```

</TabItem>
<TabItem value="js">

```js
var axios = require("axios");

var config = {
  method: "get",
  url: "https://shimo-domain/sdk/v2/api/license/users?page=1&size=30",
  headers: {
    "X-Shimo-Signature": "your_signature",
    "X-Shimo-Token": "your_token",
  },
};

axios(config)
  .then(function (response) {
    console.log(JSON.stringify(response.data));
  })
  .catch(function (error) {
    console.log(error);
  });
```

</TabItem>
</Tabs>

**HTTP Response Body**

响应体示例：

```json
[
  {
    "userId": "1",
    "createdAt": "2021-05-20T14:56:53+08:00",
    "status": 1
  }
]
```

| 字段名    | 类型   | 值示例                    | 说明                                    |
| :-------- | :----- | :------------------------ | :-------------------------------------- |
| userId    | string | 1                         | 用户 ID                                 |
| createdAt | string | 2021-05-20T14:56:53+08:00 | 创建时间                                |
| status    | number | 1                         | 见[用户席位状态说明](#user-status-desc) |

### 激活用户席位

**请求地址**

_POST_ http(s)://shimo-domain/sdk/v2/api/license/users/activate

**HTTP Request Body**

请求体示例：

| 字段名  | 类型     | 值示例     | 必选 | 说明                       |
| :------ | :------- | :--------- | :--- | :------------------------- |
| userIds | []string | ["1", "2"] | 是   | 需要激活席位的用户 ID 列表 |

**请求示例**
<Tabs
defaultValue="curl"
values={[
{ label: 'cURL', value: 'curl', },
{ label: 'Java', value: 'java', },
{ label: 'Golang', value: 'go', },
{ label: 'Node.js', value: 'js', },
]
}>
<TabItem value="curl">

```bash
curl --request POST 'https://shimo-domain/sdk/v2/api/license/users/activate' \
--header 'Content-Type: application/json' \
--header 'X-Shimo-Signature: your_signature' \
--header 'X-Shimo-Token: your_token' \
--data-raw '{"userIds":["1","2"]}'
```

</TabItem>
<TabItem value="java">

```java
OkHttpClient client = new OkHttpClient().newBuilder()
  .build();
MediaType mediaType = MediaType.parse("application/json");
RequestBody body = RequestBody.create(mediaType, "{\"userIds\":[\"1\",\"2\"]}");
Request request = new Request.Builder()
  .url("https://shimo-domain/sdk/v2/api/license/users/activate")
  .method("POST", body)
  .addHeader("Content-Type", "application/json")
  .addHeader("X-Shimo-Signature", "your_signature")
  .addHeader("X-Shimo-Token", "your_token")
  .build();
Response response = client.newCall(request).execute();
```

</TabItem>
<TabItem value="go">

```go
package main

import (
  "fmt"
  "strings"
  "net/http"
  "io/ioutil"
)

func main() {
  url := "https://shimo-domain/sdk/v2/api/license/users/activate"
  method := "POST"

  payload := strings.NewReader(`{"userIds":["1","2"]}`)

  client := &http.Client {
  }
  req, err := http.NewRequest(method, url, payload)
  if err != nil {
    fmt.Println(err)
    return
  }

  req.Header.Set("Content-Type", "application/json")
  req.Header.Set("X-Shimo-Signature", "your_signature")
  req.Header.Set("X-Shimo-Token", "your_token")

  res, err := client.Do(req)
  if err != nil {
    fmt.Println(err)
    return
  }
  defer res.Body.Close()

  body, err := ioutil.ReadAll(res.Body)
  if err != nil {
    fmt.Println(err)
    return
  }
  fmt.Println(string(body))
}
```

</TabItem>
<TabItem value="js">

```js
var axios = require("axios");
var data = JSON.stringify({
  userIds: ["1", "2"],
});

var config = {
  method: "post",
  url: "https://shimo-domain/sdk/v2/api/license/users/activate",
  headers: {
    "Content-Type": "application/json",
    "X-Shimo-Signature": "your_signature",
    "X-Shimo-Token": "your_token",
  },
  data: data,
};

axios(config)
  .then(function (response) {
    console.log(JSON.stringify(response.data));
  })
  .catch(function (error) {
    console.log(error);
  });
```

</TabItem>
</Tabs>

**HTTP Response Body**

空

**HTTP 状态码**

| 状态码 | 说明     |
| :----- | :------- |
| 204    | 操作成功 |

### 取消用户席位

**被取消席位的 `userId` 只能通过重新激活才可以使用。**

**请求地址**

_POST_ http(s)://shimo-domain/sdk/v2/api/license/users/deactivate

**HTTP Request Body**

请求体示例：

| 字段名  | 类型     | 值示例     | 必选 | 说明                       |
| :------ | :------- | :--------- | :--- | :------------------------- |
| userIds | []string | ["1", "2"] | 是   | 需要取消席位的用户 ID 列表 |

**请求示例**
<Tabs
defaultValue="curl"
values={[
{ label: 'cURL', value: 'curl', },
{ label: 'Java', value: 'java', },
{ label: 'Golang', value: 'go', },
{ label: 'Node.js', value: 'js', },
]
}>
<TabItem value="curl">

```bash
curl --request POST 'https://shimo-domain/sdk/v2/api/license/users/deactivate' \
--header 'Content-Type: application/json' \
--header 'X-Shimo-Signature: your_signature' \
--header 'X-Shimo-Token: your_token' \
--data-raw '{"userIds":["1","2"]}'
```

</TabItem>
<TabItem value="java">

```java
OkHttpClient client = new OkHttpClient().newBuilder()
  .build();
MediaType mediaType = MediaType.parse("application/json");
RequestBody body = RequestBody.create(mediaType, "{\"userIds\":[\"1\",\"2\"]}");
Request request = new Request.Builder()
  .url("https://shimo-domain/sdk/v2/api/license/users/deactivate")
  .method("POST", body)
  .addHeader("Content-Type", "application/json")
  .addHeader("X-Shimo-Signature", "your_signature")
  .addHeader("X-Shimo-Token", "your_token")
  .build();
Response response = client.newCall(request).execute();
```

</TabItem>
<TabItem value="go">

```go
package main

import (
  "fmt"
  "strings"
  "net/http"
  "io/ioutil"
)

func main() {
  url := "https://shimo-domain/sdk/v2/api/license/users/deactivate"
  method := "POST"

  payload := strings.NewReader(`{"userIds":["1","2"]}`)

  client := &http.Client {
  }
  req, err := http.NewRequest(method, url, payload)
  if err != nil {
    fmt.Println(err)
    return
  }

  req.Header.Set("Content-Type", "application/json")
  req.Header.Set("X-Shimo-Signature", "your_signature")
  req.Header.Set("X-Shimo-Token", "your_token")

  res, err := client.Do(req)
  if err != nil {
    fmt.Println(err)
    return
  }
  defer res.Body.Close()

  body, err := ioutil.ReadAll(res.Body)
  if err != nil {
    fmt.Println(err)
    return
  }
  fmt.Println(string(body))
}
```

</TabItem>
<TabItem value="js">

```js
var axios = require("axios");
var data = JSON.stringify({
  userIds: ["1", "2"],
});

var config = {
  method: "post",
  url: "https://shimo-domain/sdk/v2/api/license/users/deactivate",
  headers: {
    "Content-Type": "application/json",
    "X-Shimo-Signature": "your_signature",
    "X-Shimo-Token": "your_token",
  },
  data: data,
};

axios(config)
  .then(function (response) {
    console.log(JSON.stringify(response.data));
  })
  .catch(function (error) {
    console.log(error);
  });
```

</TabItem>
</Tabs>

**HTTP Response Body**

空

**HTTP 状态码**

| 状态码 | 说明     |
| :----- | :------- |
| 204    | 操作成功 |

### 批量设置用户席位 {#batch-set-user-status}

**请求地址**

_POST_ http(s)://shimo-domain/sdk/v2/api/license/users/set-status

**HTTP Request Body**

请求体示例：

| 字段名  | 类型     | 值示例     | 必选 | 说明                                    |
| :------ | :------- | :--------- | :--- | :-------------------------------------- |
| userIds | []string | ["1", "2"] | 是   | 需要修改席位状态的用户 ID 列表          |
| status  | number   | -1         | 是   | 见[用户席位状态说明](#user-status-desc) |

:::caution 注意
设置为 `-1 未启用` 并不能阻止用户继续使用石墨 SDK，比如用户打开的石墨 SDK 页面并没有关闭，signature 和 token 仍有效，即使设置为 `-1 未启用`，只要用户未关闭的页面发起了请求，状态也会被自动设置为 `1 激活`。
:::

**请求示例**
<Tabs
defaultValue="curl"
values={[
{ label: 'cURL', value: 'curl', },
{ label: 'Java', value: 'java', },
{ label: 'Golang', value: 'go', },
{ label: 'Node.js', value: 'js', },
]
}>
<TabItem value="curl">

```bash
curl -X POST 'https://shimo-domain/sdk/v2/api/license/users/set-status' \
--header 'Content-Type: application/json' \
--header 'X-Shimo-Signature: your_signature' \
--header 'X-Shimo-Token: your_token' \
--data-raw '{"userIds":["1","2"],"status":1}'
```

</TabItem>
<TabItem value="java">

```java
OkHttpClient client = new OkHttpClient().newBuilder()
  .build();
MediaType mediaType = MediaType.parse("application/json");
RequestBody body = RequestBody.create(mediaType, "{\"userIds\":[\"1\",\"2\"],\"status\":1}");
Request request = new Request.Builder()
  .url("https://shimo-domain/sdk/v2/api/license/users/set-status")
  .method("POST", body)
  .addHeader("Content-Type", "application/json")
  .addHeader("X-Shimo-Signature", "your_signature")
  .addHeader("X-Shimo-Token", "your_token")
  .build();
Response response = client.newCall(request).execute();
```

</TabItem>
<TabItem value="go">

```go
package main

import (
  "fmt"
  "strings"
  "net/http"
  "io/ioutil"
)

func main() {
  url := "https://shimo-domain/sdk/v2/api/license/users/set-status"
  method := "POST"

  payload := strings.NewReader(`{"userIds":["1","2"],"status":1}`)

  client := &http.Client {
  }
  req, err := http.NewRequest(method, url, payload)
  if err != nil {
    fmt.Println(err)
    return
  }

  req.Header.Set("Content-Type", "application/json")
  req.Header.Set("X-Shimo-Signature", "your_signature")
  req.Header.Set("X-Shimo-Token", "your_token")

  res, err := client.Do(req)
  if err != nil {
    fmt.Println(err)
    return
  }
  defer res.Body.Close()

  body, err := ioutil.ReadAll(res.Body)
  if err != nil {
    fmt.Println(err)
    return
  }
  fmt.Println(string(body))
}
```

</TabItem>
<TabItem value="js">

```js
var axios = require("axios");
var data = JSON.stringify({
  userIds: ["1", "2"],
  status: 1,
});

var config = {
  method: "POST",
  url: "https://shimo-domain/sdk/v2/api/license/users/set-status",
  headers: {
    "Content-Type": "application/json",
    "X-Shimo-Signature": "your_signature",
    "X-Shimo-Token": "your_token",
  },
  data: data,
};

axios(config)
  .then(function (response) {
    console.log(JSON.stringify(response.data));
  })
  .catch(function (error) {
    console.log(error);
  });
```

</TabItem>
</Tabs>

**HTTP Response Body**

空

**HTTP 状态码**

| 状态码 | 说明     |
| :----- | :------- |
| 204    | 操作成功 |
