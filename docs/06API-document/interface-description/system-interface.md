---
title: 系统接口
sidebar_position: 1
---

import Tabs from '@theme/Tabs';
import TabItem from '@theme/TabItem';

## 鉴权方式{#auth-with-querystring}

### Query string 鉴权方式

为石墨和接入方提供的身份识别的接口。

|参数名|类型|值示例|必选|说明|
|:----|:----|:----|:----|:----|
|token|string|eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9|是|接入方提供的 Token，在石墨请求接入方的接口时，会放到 HTTP Headers X-Shimo-Token 中作为值进行传递|
|signature|string| -                                    |是|接入方根据签名方法，将从石墨获取的 AppId 和AppSecret 生成的字符串，详细操作请参考 [签名方式](./../../02quick-start/signature.md) 。|
|appId|string|ebc1cde3-9b57-4962-883d-54302d428600|否|已废弃，可传可不传。接入方从石墨获取的 AppId|

**请求示例：**

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
var axios = require('axios');

var config = {
  method: 'GET',
  url: 'https://shimo-domain/sdk/v2/api/fake-api?signature=your_signature&token=your_token'
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

### 获取App详情

App 详细信息，包括 app 名称、可使用文件类型、已激活用户数、用户总数、席位数、license 有效时间、回调地址信息。

:::info 提示

由于 App API 的特殊性，为了安全考虑，需要在 [生成签名](./../../02quick-start/signature.md)  时附加上 `"scope": "license"` 字段。

:::

**请求地址：**

*GET* [https://shimo-domain/sdk/v2/api/license/apps/](https://shimo-domain/sdk/v2/api/license/apps/){appId}

**请求示例：**

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
var axios = require('axios');

axios({
	"method": "GET",
	"url": "https://shimo-domain/sdk/v2/api/license/apps/yourShimoAppId",
  headers: {
    'X-Shimo-Signature': 'your_signature',
    'X-Shimo-Token': 'your_token'
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



**HTTP状态码**

HTTP 状态码用来表明特定 HTTP 请求是否成功完成。当状态码为200时，表示请求成功。

**HTTP Response Body**

|**类型**|**字段名**|**说明**|**值示例**|
|:----|:----|:----|:----|
|string|appName|应用名|your app name|
|Array|availableFileTypes|可用石墨套件列表|["document","documentPro","spreadsheet","presentation","table"]|
|number|activatedUserCount|已激活席位用户数|43|
|number|userCount|用户数总数，包含已激活、已禁用和未使用的用户总数，仅“已激活 ”数量占用席位。|50|
|number|memberLimit|license 席位限制用户数，即“已激活 ”用户数量最大限制|99|
|string|validFrom|license 生效时间|2020-12-31T16:00:00Z|
|string|validUntil|license 有效期截止时间|2022-12-30T16:00:00Z|
|string|endpointUrl|接入方回调地址|[http://your-domain](http://your-domain)|

**响应示例**

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

### 更新App回调地址

用于更新接入方App的回调地址。

:::info 提示

由于 App API 的特殊性，为了安全考虑，需要在 [生成签名](./../../02quick-start/signature.md)  时附加上 `"scope": "license"` 字段。

:::

**请求地址**

*PUT* [https://shimo-domain/sdk/v2/api/license/apps/](https://shimo-domain/sdk/v2/api/license/apps/){appId}/endpoint-url

**HTTP Request Body**

|字段名|类型|值示例|必选|说明|
|:----|:----|:----|:----|:----|
|url|string|[http://your-domain](http://your-domain)|是|需要更新的服务商回调地址|

**请求示例：**

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
var axios = require('axios');
var data = JSON.stringify({
  url: "http://your-domain"
});

var config = {
  method: 'PUT',
  url: 'https://shimo-domain/sdk/v2/api/license/apps/yourShimoAppId/endpoint-url',
  headers: {
    'Content-Type': 'application/json',
    'X-Shimo-Signature': 'your_signature',
    'X-Shimo-Token': 'your_token'
  },
  data : data
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
当状态码为204时，表示创建成功。

**HTTP Response Body**

空

## 席位管理

### 用户席位状态说明

| **值域** | **席位状态** | **说明**                                                     |
| :------- | :----------- | :----------------------------------------------------------- |
| 1        | 激活         | 仅“激活”状态的用户占用席位数。                               |
| 0        | 已禁用       | “已禁用”状态的用户必须重新激活才可以使用。                   |
| -1       | 未启用       | 1、若接入方已集成实现`@ 人 / 提及人`相关功能时，在调用接入方搜索接口时，可能会返回石墨 SDK 中未关联的用户，由于需要在石墨编辑器中的 at 下拉列表中展示此类用户，石墨 SDK 会首先进行建立用户关联操作，此时用户的状态将会初始化为`未启用`状态，此状态 **`不会占用`** 席位数量。 <br />2、当用户处于未启用状态时，当通过此用户身份进行如下操作时，将会自动激活，成为 `激活` 状态，此时占用相应席位，若席位不足，则会被拒绝访问:<br /><br />创建文件、创建副本、访问编辑页 导入、导出文件、预览文件、调用表格内容操作接口、获取文件侧边栏历史信息、版本信息、获取文件纯文本信息、获取文件 at 人列表、获取文件评论数据<br /> |

### 获取用户列表和席位状态

**请求地址**

*GET* http(s)://shimo-domain/sdk/v2/api/license/users

**HTTP Request Parameters**

|传参方式|参数名|类型|值示例|必选|说明|
|:----|:----|:----|:----|:----|:----|
|Query|page|number|1|是|请求页码|
|Query|size|number|30|是|单页返回用户数量|

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
var axios = require('axios');

var config = {
  method: 'get',
  url: 'https://shimo-domain/sdk/v2/api/license/users?page=1&size=30',
  headers: {
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
</Tabs>



**HTTP Response Body**

|字段名|类型|值示例|说明|
|:----|:----|:----|:----|
|userId|string|1|用户 ID|
|createdAt|string|2021-05-20T14:56:53+08:00|创建时间|
|status|number|1|请参考 [用户席位状态说明](#用户席位状态说明)。|

**响应示例**

```json
[
  {
    "userId": "1",
    "createdAt": "2021-05-20T14:56:53+08:00",
    "status": 1
  }
]
```
### 激活用户席位[​](https://sdk.shimo.im/v2/docs/apis/base#%E6%BF%80%E6%B4%BB%E7%94%A8%E6%88%B7%E5%B8%AD%E4%BD%8D)

**请求地址**

*POST* http(s)://shimo-domain/sdk/v2/api/license/users/activate

**HTTP Request Body**

|字段名|类型|值示例|必选|说明|
|:----|:----|:----|:----|:----|
|userIds|[]string|["1", "2"]|是|需激活席位的用户 ID 列表|

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
var axios = require('axios');
var data = JSON.stringify({
  "userIds": [
    "1",
    "2"
  ]
});

var config = {
  method: 'post',
  url: 'https://shimo-domain/sdk/v2/api/license/users/activate',
  headers: {
    'Content-Type': 'application/json',
    'X-Shimo-Signature': 'your_signature',
    'X-Shimo-Token': 'your_token'
  },
  data : data
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



**HTTP状态码**
当状态码为204时，表示操作成功。

**HTTP Response Body**

空

### 取消用户席位[​](https://sdk.shimo.im/v2/docs/apis/base#%E5%8F%96%E6%B6%88%E7%94%A8%E6%88%B7%E5%B8%AD%E4%BD%8D)

被取消席位的 `userId` 只能通过重新激活才可以使用。

**请求地址**

*POST* http(s)://shimo-domain/sdk/v2/api/license/users/deactivate

**HTTP Request Body**

|字段名|类型|值示例|必选|说明|
|:----|:----|:----|:----|:----|
|userIds|[]string|["1", "2"]|是|需取消席位的用户 ID 列表|

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
var axios = require('axios');
var data = JSON.stringify({
  "userIds": [
    "1",
    "2"
  ]
});

var config = {
  method: 'post',
  url: 'https://shimo-domain/sdk/v2/api/license/users/deactivate',
  headers: {
    'Content-Type': 'application/json',
    'X-Shimo-Signature': 'your_signature',
    'X-Shimo-Token': 'your_token'
  },
  data : data
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



**HTTP状态码**
当状态码为204时，表示操作成功。

**HTTP Response Body**

空

### 批量设置用户席位[​](https://sdk.shimo.im/v2/docs/apis/base#batch-set-user-status)

**请求地址**

*POST* http(s)://shimo-domain/sdk/v2/api/license/users/set-status

**HTTP Request Body**

|字段名|类型|值示例|必选|说明|
|:----|:----|:----|:----|:----|
|userIds|[]string|["1", "2"]|是|需修改席位状态的用户 ID 列表|
|status|number|-1|是|请参考 [用户席位状态说明](#用户席位状态说明)。|

:::caution 说明

设置为 `-1 未启用` 并不能阻止用户继续使用石墨 SDK，比如用户打开的石墨 SDK 页面并没有关闭，signature 和 token 仍有效，即使设置为 `-1 未启用`，只要用户未关闭的页面发起了请求，状态也会被自动设置为 `1 激活`。

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
var axios = require('axios');
var data = JSON.stringify({
  "userIds": [
    "1",
    "2"
  ],
  "status": 1
});

var config = {
  method: 'POST',
  url: 'https://shimo-domain/sdk/v2/api/license/users/set-status',
  headers: {
    'Content-Type': 'application/json',
    'X-Shimo-Signature': 'your_signature',
    'X-Shimo-Token': 'your_token'
  },
  data : data
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

**HTTP状态码**
当状态码为204时，表示操作成功。

**HTTP Response Body**

空
