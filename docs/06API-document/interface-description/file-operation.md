---
title: 文件操作
description: 如何使用API接口导入导出文件、创建预览
keywords: [石墨文档, 文档中台, 协同办公, 在线文档, 文件共享, 导入, 导出, 预览]
sidebar_position: 3

---

import Tabs from '@theme/Tabs';
import TabItem from '@theme/TabItem';


## 上传下载

### 上传附件

**步骤一：获取 Token**

POST https://shimo-domain/sdk/v2/api/uploader/token

> Body 请求参数

```json
[
  {
    "fileGuid": "string",
    "store": "string",
    "bucket": "string",
    "filename": "string",
    "fileSize": 0,
    "encrypt": "string"
  }
]
```

请求参数

|名称|位置|类型|必选|说明|
|---|---|---|---|---|
|X-Shimo-Signature|header|string| 是 |X-Shimo-Signature|
|X-Shimo-Token|header|string| 是 |X-Shimo-Token|
|Content-Type|header|string| 是 |application/json|
|body|body|array[object]| 否 |none|

> 返回示例

> 200 Response

```json
[
  {
    "fileFieldKey": "string",
    "formData": {
      "accessToken": "string",
      "download": "string"
    },
    "guid": "string",
    "images": "string",
    "key": "string",
    "serverUrl": "string",
    "url": "string",
  }
]
```

返回结果

|状态码|状态码含义|说明|数据模型|
|---|---|---|---|
|200|[OK](https://tools.ietf.org/html/rfc7231#section-6.3.1)|成功|Inline|

返回数据结构

状态码 **200**

|名称|类型|必选|约束|中文名|说明|
|---|---|---|---|---|---|
|fileFieldKey|string|false|none|file|第二步上传时 file 应该使用的 key name。固定值 file|
|formData|object|false|none|-|第二步上传时的 post form data|
|accessToken|string|true|none|-|第二步上传时候使用的 accessToken|
|download|string|true|none|-|控制生成的 url 是否携带 download=1 参数|
|guid|string|false|none|附件guid|附件guid|
|images|string|false|none|url|同 url，兼容使用|
|key|string|false|none|附件key|附件key|
|serverUrl|string|false|none|上传地址|下一步上传需要提交 post 表单到的服务器地址|
|url|string|false|none|url|附件访问 url|

**步骤二：执行上传**

POST https://shimo-domain/sdk/v2/api/uploader/upload

Body 请求参数

```yaml
accessToken: string
download: true, false
file: string

```

请求参数

|名称|位置|类型|必选|说明|
|---|---|---|---|---|
|Content-Type|header|string| 是 |表单|
|X-Shimo-Signature|header|string| 是 |X-Shimo-Signature|
|X-Shimo-Token|header|string| 是 |X-Shimo-Token|
|body|body|object| 否 |none|
|accessToken|body|string| 是 |第一步中获取到的accessToken|
|download|body|string| 否 |是否在返回结果url 中增加download 参数|
|file|body|string(binary)| 是 |文件|

> 返回示例

> 200 Response

```json
{
  "code": 0,
  "data": {
    "GUID": "string",
    "filename": "string",
    "key": "string",
    "mimeType": "string",
    "images": "string",
    "rawUrl": "string",
    "url": "string",
    "audio": null,
    "video": null,
    "width": "string",
    "height": "string",
    "size": 0
  }
}
```

返回结果

|状态码|状态码含义|说明|数据模型|
|---|---|---|---|
|200|[OK](https://tools.ietf.org/html/rfc7231#section-6.3.1)|成功|Inline|

返回数据结构

状态码 **200**

|名称|类型|必选|约束|中文名|说明|
|---|---|---|---|---|---|
|code|integer|true|none|-|none|
| data|object|true|none|-|none|
|GUID|string|true|none|GUID|附件GUID|
|filename|string|true|none|文件名称|文件名称|
| key|string|true|none|key|附件key|
|mimeType|string|true|none|mimeType|例如 application/gzip|
|images|string|true|none|同url|none|
|url|string|true|none|附件url|none|
|audio|null|true|none|音频信息|none|
|video|null|true|none|视频信息|例如 {"width":"640","height":"352","duration":"17.333333","fps":"","fileFormat":"","bitrate":"49710"}|
| width|string|true|none|图片宽度|none|
|height|string|true|none|图片高度|none|
|size|integer|true|none|大小|字节|

## 导入导出 {#import-export}

### 导入文件 {#import-v1}

导入文件过程是一个异步处理过程，接口请求成功仅代表导入服务收到并开始处理导入请求，实际是否成功需要轮询 [获取导入进度](#import-progress-v1) 接口查询导入任务是否完成。

:::caution 说明

此接口为新版导入文件接口，建议优先使用。新版导入文件接口要求接入方必须解析结果中的 taskId，并且在调用 [获取导入进度](#import-progress-v1) 时带上 taskId 参数。任务执行的最长时间为10分钟。

:::

**导入支持的文件格式**

| 石墨文件类型 | 支持导入的文件类型       |
| :----------- | :----------------------- |
| document     | .docx，.doc，.md，.txt   |
| documentPro  | .docx，.doc，.wps        |
| spreadsheet  | .xlsx，.xls，.csv, .xlsm |
| presentation | .pptx，.ppt              |

**请求地址**

_POST_ https://shimo-domain/sdk/v2/api/files/v1/import

**HTTP Request Body**

| 传参方式 | 参数名   | 类型   | 值示例                                        | 必选 | 说明                                                    |
|:---------|:---------|:-------|:----------------------------------------------|:----|:------------------------------------------------------|
| Form     | fileId   | string | file1001                                      | 是   | 服务商文件的唯一 ID                                     |
| Form     | type     | string | documentPro                                   | 是   | 需要导入的文件类型                                      |
| Form     | file     | file   |                                               | 否   | 需要导入的文件（如果不发送文件，就必须传fileUrl、fileName） |
| Form     | fileUrl  | string | <https://domain.com/files/download/test.docx> | 否   | 需要导入的文件下载地址（有参数file，可以不用传）           |
| Form     | fileName | string | test.docx                                     | 否   | 需要导入的文件名称（有参数file，可以不用传）               |

:::caution 说明

如无特别说明，导入文件时需要附带扩展名，否则可能解析失败，如「`测试文件.doc`」或使用 `FormData` 时 `filename` 字段中传入带扩展名的文件名，详细操作请参考 [MDN](https://developer.mozilla.org/zh-CN/docs/Web/API/FormData/append#%E7%A4%BA%E4%BE%8B)。

:::

**请求示例**
<Tabs
  defaultValue="curl"
  values={[
    { label: 'cURL', value: 'curl', },
    { label: 'Java', value: 'java', },
    { label: 'Golang', value: 'go', },
    { label: 'Node.js', value: 'js', }, 
    { label: 'PHP', value: 'php', },
  ]
}>
<TabItem value="curl">

```bash
curl --request POST 'https://shimo-domain/sdk/v2/api/files/v1/import' \
--form 'fileId="{importFileId}"' \
--form 'type="document"' \
--form 'file=@"/your/local/path/测试.docx"' \
--header 'X-Shimo-Signature: your_signature' \
--header 'X-Shimo-Token: your_token'
```

</TabItem>
<TabItem value="java">

```java
OkHttpClient client = new OkHttpClient().newBuilder()
  .build();
MediaType mediaType = MediaType.parse("multipart/form-data");
RequestBody body = new MultipartBody.Builder()
  .setType(MultipartBody.FORM)
  .addFormDataPart("fileId","{importFileId}")
  .addFormDataPart("type","document")
  .addFormDataPart("file","测试.docx",
    RequestBody.create(MediaType.parse("application/vnd.openxmlformats-officedocument.wordprocessingml.document"),
    new File("/your/local/path/测试.docx")))
  .build();
Request request = new Request.Builder()
  .url("https://shimo-domain/sdk/v2/api/files/v1/import")
  .method("POST", body)
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
  "bytes"
  "mime/multipart"
  "os"
  "path/filepath"
  "io"
  "net/http"
  "io/ioutil"
)

func main() {
  url := "https://shimo-domain/sdk/v2/api/files/v1/import"
  method := "POST"

  payload := &bytes.Buffer{}
  writer := multipart.NewWriter(payload)
  _ = writer.WriteField("fileId", "{importFileId}")
  _ = writer.WriteField("type", "document")
  file, errFile3 := os.Open("/your/local/path/测试.docx")
  defer file.Close()
  part3, errFile3 := writer.CreateFormFile("file",filepath.Base("/Users/yanjixiong/Downloads/Office文件/测试.docx"))
  _, errFile3 = io.Copy(part3, file)
  if errFile3 != nil {
    fmt.Println(errFile3)
    return
  }
  err := writer.Close()
  if err != nil {
    fmt.Println(err)
    return
  }

  client := &http.Client {
  }
  req, err := http.NewRequest(method, url, payload)
  if err != nil {
    fmt.Println(err)
    return
  }

  req.Header.Set("Content-Type", writer.FormDataContentType())
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
var FormData = require('form-data');
var fs = require('fs');
var data = new FormData();
data.append('fileId', '{importFileId}');
data.append('type', 'document');
data.append('file', fs.createReadStream('/your/local/path/测试.docx'));

var config = {
  method: 'post',
  url: 'https://shimo-domain/sdk/v2/api/files/v1/import',
  headers: {
    ...data.getHeaders(),
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
<TabItem value="php">

```php
<?php
require 'vendor/autoload.php';
use GuzzleHttp\Client;
use GuzzleHttp\Psr7\Request;
$client = new Client();
$headers = [
  'X-Shimo-Signature' => 'your_signature',
  'X-Shimo-Token' => 'your_token'
];
$options = [
  'multipart' => [
    [
      'name' => 'fileId',
      'contents' => '{importFileId}'
    ],
    [
      'name' => 'type',
      'contents' => 'document'
    ],
    [
      'name' => 'file',
      'contents' => Utils::tryFopen('/path/to/file', 'r'),
      'filename' => '/path/to/file',
      'headers'  => [
        'Content-Type' => '<Content-type header>'
      ]
    ]
]];
$request = new Request('POST', 'https://shimo-domain/sdk/v2/api/files/v1/import', $headers);
$res = $client->sendAsync($request, $options)->wait();
echo $res->getBody();



```

</TabItem>
</Tabs>

**HTTP Response Body**

| 字段名      | 类型   | 值示例           | 说明                                                                             |
|:------------|:-------|:-----------------|:-------------------------------------------------------------------------------|
| status      | number | 0                | 导入状态，非零值表示异常                                                          |
| message     | string |                  | 导入异常时的提示信息                                                             |
| data.taskId | string | TnJdQS8Wk70wNHuB | 导入任务的标识 ID，调用导入进度接口时，请带上该参数。导入失败时请提供此 ID 用于调试 |



### 获取导入进度 {#import-progress-v1}

:::info 提示

此接口为新版获取导入进度接口，建议优先使用。当任务完成之后，进度结果将会默认缓存 5 分钟，过期之后再次调用进度结果将会失败。

:::

**请求地址**

_POST_ https://shimo-domain/sdk/v2/api/files/v1/import/progress

**HTTP Request Parameters**

| 传参方式 | 参数名    | 类型   | 值示例                               | 必选 | 说明                                                                                                      |
|:---------|:----------|:-------|:-------------------------------------|:----|:--------------------------------------------------------------------------------------------------------|
| Query    | taskId    | string | TnJdQS8Wk70wNHuB                     | 是   | 导入文件接口返回的 taskId                                                                                 |

**请求示例**
<Tabs
  defaultValue="curl"
  values={[
    { label: 'cURL', value: 'curl', },
    { label: 'Java', value: 'java', },
    { label: 'Golang', value: 'go', },
    { label: 'Node.js', value: 'js', },
    { label: 'PHP', value: 'php', },
  ]
}>
<TabItem value="curl">

```bash
curl --request POST 'https://shimo-domain/sdk/v2/api/files/v1/import/progress?taskId={taskId}' \
--header 'X-Shimo-Signature: your_signature' \
--header 'X-Shimo-Token: your_token' \
--data-raw ''
```

</TabItem>
<TabItem value="java">

```java
OkHttpClient client = new OkHttpClient().newBuilder()
  .build();
MediaType mediaType = MediaType.parse("application/json");
RequestBody body = RequestBody.create(mediaType, "");
Request request = new Request.Builder()
  .url("https://shimo-domain/sdk/v2/api/files/v1/import/progress?taskId={taskId}")
  .method("POST", body)
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
  url := "https://shimo-domain/sdk/v2/api/files/v1/import/progress?taskId={taskId}"
  method := "POST"

  payload := strings.NewReader(``)

  client := &http.Client {
  }
  req, err := http.NewRequest(method, url, payload)
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
var data = '';

var config = {
  method: 'post',
  url: 'https://shimo-domain/sdk/v2/api/files/v1/import/progress?taskId={taskId}',
  headers: {
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
<TabItem value="php">

```php
<?php
require 'vendor/autoload.php';
use GuzzleHttp\Client;
use GuzzleHttp\Psr7\Request;
$client = new Client();
$headers = [
  'X-Shimo-Signature' => 'your_signature',
  'X-Shimo-Token' => 'your_token'
];
$request = new Request('POST', 'https://shimo-domain/sdk/v2/api/files/v1/import/progress?taskId={taskId}', $headers);
$res = $client->sendAsync($request)->wait();
echo $res->getBody();


```

</TabItem>
</Tabs>

**HTTP Response Body**

| 字段名        | 类型   | 值示例 | 说明                                 |
|:--------------|:-------|:-------|:-----------------------------------|
| status        | number | 0      | 导入状态，非零值表示异常              |
| message       | string |        | 导入异常时的提示信息                 |
| data.progress | number | 100    | 导入进度百分比，为 100 时表示导入完成 |

:::caution 说明

通过轮询该接口，直到返回的 `status == 0` 和 `data.progress == 100` 时表示`导入完成`，即可通过 「文件编辑」 接口进行查看或编辑。

:::

### 导出文件 {#export-v1}

将在线编辑文件导出为 Office 文件、PDF 等格式，不同套件支持的导出格式有所不同。

导出文件过程是一个异步处理过程，接口请求成功仅代表导出服务收到并开始处理导出请求，实际是否成功需要轮询 [获取导出进度](#export-progress-v1) 接口查询导出任务是否完成。当任务完成后，用「获取导出进度」返回的链接 ( `data.downloadUrl` 字段) 下载文件。

:::info 提示

此接口为新版导出文件接口，建议优先使用。新版导出文件接口要求接入方必须解析结果中的 taskId，并且在调用 [获取导出进度](#export-progress-v1) 时带上 taskId 参数。此接口允许同时将同一个石墨文件导出为不同的文件类型，例如：同时将轻文档导出为 PDF 和 Word 两个格式。任务执行的最长时间为 10 分钟。

:::

**导出支持的文件格式**

| 石墨文件类型 | 默认导出的文件类型 | 支持导出的文件类型 |
| :----------- | :----------------- | :----------------- |
| document     | docx               | docx, md, jpg, pdf |
| documentPro  | docx               | docx, pdf, wps     |
| spreadsheet  | xlsx               | xlsx               |
| presentation | pptx               | pptx, pdf          |

**请求地址**

_POST_ https://shimo-domain/sdk/v2/api/files/v1/export/{fileId}

**HTTP Request Body**

| 传参方式 | 参数名 | 类型   | 值示例 | 必选 | 说明                                      |
|:---------|:-------|:-------|:-------|:----|:----------------------------------------|
| Body     | type   | string | docx   | 否   | 需要导出的文件类型，不传时按照默认类型导出 |

**导出支持的文件格式**

| 石墨文件类型 | 默认导出的文件类型 | 支持导出的文件类型 |
|:-------------|:-------------------|:-------------------|
| document     | docx               | docx, md, jpg, pdf |
| documentPro  | docx               | docx, pdf, wps     |
| spreadsheet  | xlsx               | xlsx               |
| presentation | pptx               | pptx, pdf          |

**请求示例**
<Tabs
  defaultValue="curl"
  values={[
    { label: 'cURL', value: 'curl', },
    { label: 'Java', value: 'java', },
    { label: 'Golang', value: 'go', },
    { label: 'Node.js', value: 'js', },
    { label: 'PHP', value: 'php', },
  ]
}>
<TabItem value="curl">

```bash
curl --request POST 'https://shimo-domain/sdk/v2/api/files/v1/export/{exportFileId}' \
--header 'Content-Type: application/json' \
--header 'X-Shimo-Signature: your_signature' \
--header 'X-Shimo-Token: your_token' \
--data-raw '{"type": "docx"}'
```

</TabItem>
<TabItem value="java">

```java
OkHttpClient client = new OkHttpClient().newBuilder()
  .build();
MediaType mediaType = MediaType.parse("application/json");
RequestBody body = RequestBody.create(mediaType, "{\"type\": \"docx\"}");
Request request = new Request.Builder()
  .url("https://shimo-domain/sdk/v2/api/files/v1/export/{exportFileId}")
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
  url := "https://shimo-domain/sdk/v2/api/files/v1/export/{exportFileId}"
  method := "POST"

  payload := strings.NewReader(`{"type": "docx"}`)

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
  "type": "docx"
});

var config = {
  method: 'post',
  url: 'https://shimo-domain/sdk/v2/api/files/v1/export/{exportFileId}',
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
<TabItem value="php">

```php
<?php
require 'vendor/autoload.php';
use GuzzleHttp\Client;
use GuzzleHttp\Psr7\Request;
$client = new Client();
$headers = [
  'Content-Type' => 'application/json',
  'X-Shimo-Signature' => 'your_signature',
  'X-Shimo-Token' => 'your_token'
];
$body = '{
  "type": "docx"
}';
$request = new Request('POST', 'https://shimo-domain/sdk/v2/api/files/v1/export/{exportFileId}', $headers, $body);
$res = $client->sendAsync($request)->wait();
echo $res->getBody();


```

</TabItem>
</Tabs>

**HTTP Response Body**

| 字段名      | 类型   | 值示例                       | 说明                                                                          |
|:------------|:-------|:-----------------------------|:----------------------------------------------------------------------------|
| status      | number | 0                            | 导出状态，非零值表示异常                                                       |
| data.taskId | string | 3oo4vnBJgcG5HxMm:1:603:xmind | 导出任务的标识ID，调用导出进度接口时，请带上该参数，导出失败时请提供此ID用于调试 |
| message     | string |                              | 导出异常时的提示信息                                                          |

### 获取导出进度 {#export-progress-v1}

:::caution 说明

此接口为新版获取导入进度接口，建议优先使用。进度结果将会默认缓存 5 分钟，过期之后再次调用进度结果将会失败。

:::

**请求地址**

_POST_ https://shimo-domain/sdk/v2/api/files/v1/export/progress

**HTTP Request Parameters**

| 传参方式 | 参数名    | 类型   | 值示例                               | 必选 | 说明                                                                                                      |
|:---------|:----------|:-------|:-------------------------------------|:----|:--------------------------------------------------------------------------------------------------------|
| Query    | taskId    | string | 3oo4vnBJgcG5HxMm:1:603:xmind         | 是   | 导出文件接口返回的 taskId                                                                                 |

**请求示例**
<Tabs
  defaultValue="curl"
  values={[
    { label: 'cURL', value: 'curl', },
    { label: 'Java', value: 'java', },
    { label: 'Golang', value: 'go', },
    { label: 'Node.js', value: 'js', },
    { label: 'PHP', value: 'php', },
  ]
}>
<TabItem value="curl">

```bash
curl --request POST 'https://shimo-domain/sdk/v2/api/files/v1/export/progress?taskId={taskId}' \
--header 'X-Shimo-Signature: your_signature' \
--header 'X-Shimo-Token: your_token' \
--data-raw ''
```

</TabItem>
<TabItem value="java">

```java
OkHttpClient client = new OkHttpClient().newBuilder()
  .build();
MediaType mediaType = MediaType.parse("application/json");
RequestBody body = RequestBody.create(mediaType, "");
Request request = new Request.Builder()
  .url("https://shimo-domain/sdk/v2/api/files/v1/export/progress?taskId={taskId}")
  .method("POST", body)
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
  url := "https://shimo-domain/sdk/v2/api/files/v1/export/progress?taskId={taskId}"
  method := "POST"

  payload := strings.NewReader(``)

  client := &http.Client {
  }
  req, err := http.NewRequest(method, url, payload)
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
  method: 'POST',
  url: 'https://shimo-domain/sdk/v2/api/files/v1/export/progress?taskId={taskId}',
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
<TabItem value="php">

```php
<?php
require 'vendor/autoload.php';
use GuzzleHttp\Client;
use GuzzleHttp\Psr7\Request;
$client = new Client();
$headers = [
  'X-Shimo-Signature' => 'your_signature',
  'X-Shimo-Token' => 'your_token'
];
$request = new Request('POST', 'https://shimo-domain/sdk/v2/api/files/v1/export/progress?taskId={taskId}', $headers);
$res = $client->sendAsync($request)->wait();
echo $res->getBody();


```

</TabItem>
</Tabs>

**HTTP Response Body**

| 字段名           | 类型   | 值示例 | 说明                                 |
|:-----------------|:-------|:-------|:-----------------------------------|
| status           | number | 0      | 导出状态，非零值表示异常              |
| message          | string | - | 导出异常时的提示信息                 |
| data.progress    | number | 100    | 导出进度百分比，为 100 时表示导入完成 |
| data.downloadUrl | string | - | 导出文件的下载地址                   |

:::info 提示

通过轮询该接口，直到返回的 `data.progress == 100` 时表示导出完成。

:::

### 导出应用表格为 Excel {#export-table-sheets}

导出应用表格为 .xlsx 文件并返回该文件的下载地址。

**请求地址**

_POST_ https://shimo-domain/sdk/v2/api/files/export/table-sheets/{fileId}

**HTTP Request Parameters**

| 传参方式 | 参数名    | 类型   | 值示例                               | 必选 | 说明                                                                                                                                           |
|:---------|:----------|:-------|:-------------------------------------|:----|:---------------------------------------------------------------------------------------------------------------------------------------------|
| Path     | fileId    | string | qeK4Xdxvxg8jF5gz                     | 是   | 文件 ID 位于创建预览接口的 URL 路径中，此文件 ID 为接入服务商文件列表中的唯一 ID，石墨会根据此 ID 请求接入服务商的文件接口获取文件信息和下载地址 |

**请求示例**
<Tabs
  defaultValue="curl"
  values={[
    { label: 'cURL', value: 'curl', },
    { label: 'Java', value: 'java', },
    { label: 'Golang', value: 'go', },
    { label: 'Node.js', value: 'js', },
    { label: 'PHP', value: 'php', },
  ]
}>
<TabItem value="curl">

```bash
curl -X POST "https://shimo-domain/sdk/v2/api/files/export/table-sheets/{fileId}" \
--header 'X-Shimo-Signature: your_signature' \
--header 'X-Shimo-Token: your_token'
```

</TabItem>
<TabItem value="java">

```java
OkHttpClient client = new OkHttpClient().newBuilder()
  .build();
MediaType mediaType = MediaType.parse("application/json");
RequestBody body = RequestBody.create(mediaType, "");
Request request = new Request.Builder()
  .url("https://shimo-domain/sdk/v2/api/files/export/table-sheets/{fileId}")
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
  url := "https://shimo-domain/sdk/v2/api/files/export/table-sheets/{fileId}"
  method := "POST"

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
  method: 'POST',
  url: 'https://shimo-domain/sdk/v2/api/files/export/table-sheets/{fileId}',
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
<TabItem value="php">

```php
<?php
require 'vendor/autoload.php';
use GuzzleHttp\Client;
use GuzzleHttp\Psr7\Request;
$client = new Client();
$headers = [
  'X-Shimo-Signature' => 'your_signature',
  'X-Shimo-Token' => 'your_token'
];
$request = new Request('POST', 'https://shimo-domain/sdk/v2/api/files/export/table-sheets/{fileId}', $headers);
$res = $client->sendAsync($request)->wait();
echo $res->getBody();



```

</TabItem>
</Tabs>

**HTTP Response Body**

| 字段名      | 类型   | 值示例 | 说明                    |
|:------------|:-------|:-------|:----------------------|
| status      | number | 0      | 导出状态，非零值表示异常 |
| message     | string | - | 导出异常时的提示信息    |
| downloadUrl | string | - | .xlsx 文件下载地址      |

## 文件预览

:::caution 说明
此功能不支持编辑文档，如果要编辑文档，请参考 [协同文档](./collaborative-editing.md)。
:::

### 创建预览

同步创建预览任务,，如果创建失败则返回失败的原因。

**请求地址**

_POST_ https://shimo-domain/sdk/v2/api/cloud-files/{fileID}/create

**HTTP Request Parameters**

| 传参方式 | 参数名    | 类型   | 示例值                               | 必选 | 说明                                                                                                                                           |
|:---------|:----------|:-------|:-------------------------------------|:----|:---------------------------------------------------------------------------------------------------------------------------------------------|
| Path     | fileId    | string | qeK4Xdxvxg8jF5gz                     | 是   | 文件 ID 位于创建预览接口的 URL 路径中，此文件 ID 为接入服务商文件列表中的唯一 ID，石墨会根据此 ID 请求接入服务商的文件接口获取文件信息和下载地址 |

**请求示例**
<Tabs
  defaultValue="curl"
  values={[
    { label: 'cURL', value: 'curl', },
    { label: 'Java', value: 'java', },
    { label: 'Golang', value: 'go', },
    { label: 'Node.js', value: 'js', },
    { label: 'PHP', value: 'php', },
  ]
}>
<TabItem value="curl">

```bash
curl --request POST 'https://shimo-domain/sdk/v2/api/cloud-files/acd87e72-946c-4182-b864-f1a7e9c74ddb/create' \
--header 'X-Shimo-Signature: your_signature' \
--header 'X-Shimo-Token: your_token'
```

</TabItem>
<TabItem value="java">

```java
OkHttpClient client = new OkHttpClient().newBuilder()
  .build();
MediaType mediaType = MediaType.parse("text/plain");
RequestBody body = RequestBody.create(mediaType, "");
Request request = new Request.Builder()
  .url("https://shimo-domain/sdk/v2/api/cloud-files/acd87e72-946c-4182-b864-f1a7e9c74ddb/create")
  .method("POST", body)
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
  url := "https://shimo-domain/sdk/v2/api/cloud-files/acd87e72-946c-4182-b864-f1a7e9c74ddb/create"
  method := "POST"

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
  method: 'post',
  url: 'https://shimo-domain/sdk/v2/api/cloud-files/acd87e72-946c-4182-b864-f1a7e9c74ddb/create',
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
<TabItem value="php">

```php
<?php
require 'vendor/autoload.php';
use GuzzleHttp\Client;
use GuzzleHttp\Psr7\Request;
$client = new Client();
$headers = [
  'X-Shimo-Signature' => 'your_signature',
  'X-Shimo-Token' => 'your_token'
];
$request = new Request('POST', 'https://shimo-domain/sdk/v2/api/cloud-files/acd87e72-946c-4182-b864-f1a7e9c74ddb/create', $headers);
$res = $client->sendAsync($request)->wait();
echo $res->getBody();



```

</TabItem>
</Tabs>

**HTTP Response Body**

| 字段名  | 类型   | 值示例             | 说明                                                      |
|:--------|:-------|:-------------------|:--------------------------------------------------------|
| code    | string | 90028              | 创建预览结果状态码，空字符串代表创建成功， 非空代表创建失败 |
| message | string | file is converting | 创建失败错误信息                                          |

**Example**

```json
// 创建失败
{
  "code": "90042",
  "message": "文件格式不正确"
}
// 创建成功
{
  "code": "",
  "message": ""
}
```

### 访问预览

用于在浏览器上渲染出预览的页面， 需要嵌入Iframe使用。

对于同一个文件， 第一次调用该接口时， 后台会异步调用 [创建预览](#创建预览) 接口创建任务，创建成功后才会渲染出预览页面。 如果需要加快第一次打开文件的速度， 可在服务端提前调用 [创建预览](#创建预览) 接口创建预览。

**请求地址**

_GET_ https://shimo-domain/sdk/v2/api/cloud-files/{fileID}/page

**HTTP Request Parameters**

| 传参方式 | 参数名    | 类型   | 示例值                               | 必选 | 说明                                                                                                                                           |
|:---------|:----------|:-------|:-------------------------------------|:----|:---------------------------------------------------------------------------------------------------------------------------------------------|
| Path     | fileId    | string | qeK4Xdxvxg8jF5gz                     | 是   | 文件 ID 位于创建预览接口的 URL 路径中，此文件 ID 为接入服务商文件列表中的唯一 ID，石墨会根据此 ID 请求接入服务商的文件接口获取文件信息和下载地址 |
| Query    | lang      | string | zh-CN                                | 否   | 页面中的语言设置， 默认zh-CN， 可选 en-US，ja-JP                                                                                                  |

嵌入 iframe 使用

```html
<html>
  <head>
    <!-- some code -->
  </head>
  <body>
    <!-- your HTML elements -->
    <div id="shimo-sdk-preview" class="shimo-sdk-preview">
      <iframe
        src="https://shimo-domain/sdk/v2/api/cloud-files/acd87e72-946c-4182-b864-f1a7e9c74ddb/page?signature=your_signature&token=your_token&lang=zh-CN"
        style="border: none; overflow: hidden;"
      ></iframe>
    </div>
  </body>
</html>
```
