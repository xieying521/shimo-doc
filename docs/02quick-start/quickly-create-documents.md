---
title: 10分钟创建协同文档
description: 10分钟创建协同文档
keywords: [石墨文档, 文档中台, 协同办公, 在线文档, 文件共享, 接口, 文档元信息]
sidebar_position: 3
---

如何快速创建一个 word 文档，并在石墨编辑器中使用，请参考下列步骤。

### 步骤 1：接入方生成文件 id

例如：my-new-file-u98213
这个 id 完全由接入方决定是什么，建议使用 uuid 等方式避免重复。

### 步骤 2：接入方提供回调接口

接入方要先实现两个接口供石墨服务端使用，接口必须公网可访问。

- 文档元信息接口
  提供一个公网可访问的 HTTP 接口，使公网 GET 请求访问。
  http://your-endpoint-url.com/files/my-new-file-u98213 时，结果为：

```json
{
  "id": "my-new-file-u98213",
  "name": "第一个协同文档",
  "type": "documentPro",
  "permissions": {
    "commentable": true,
    "editable": true,
    "readable": true,
    "copyable": true,
    "exportable": true
  },
  "views": 100,
  "creatorId": "1",
  "createdAt": "2021-08-01T00:00:00Z",
  "updatedAt": "2021-08-02T00:00:00Z",
  "teamGuid": "123"
}
```

- 当前用户信息接口
  提供一个公网可访问的 HTTP 接口，使公网 GET 请求访问
  http://your-endpoint-url.com/users/current/info 时，结果为：

```json
{
  "id": "userid123",
  "name": "张三",
  "avatar": "http://fake.site/user-123.png",
  "email": "user123@fake.site",
  "teamGuid": "123"
}
```

可前往：https://apifox.com/apidoc/shared-3a4a51a7-9180-4efb-bf8d-2849ace4369e 调试回调接口。

### 步骤 3：创建协同文档

POST 调用接口 https://office.shimoapi.com/sdk/v2/api/files ，
请求参数分为 query 和 body 两个部分。

- query 参数包括：appId，signature，token。

  - appId：license 的 appId。
  - signature：签名算法算出的结果。
  - token：接入方提供的 token ，在石墨请求接入方的接口时，会放到 HTTP Headers X-Shimo-Token 中作为值进行传递。

- body 使用 json 格式，参数包括：type，fileId。
  - type：documentPro
  - fileId：my-new-file-u98213

请求示例：

```
curl --location --request POST 'https://office.shimoapi.com/sdk/v2/api/files?appId=yourShimoAppId&signature=yourSignatureString&token=yourTokenString' \
--header 'Content-Type: application/json' \
--data-raw '{"type": "document","fileId": "my-new-file-u98213"}'
```

可前往：https://apifox.com/apidoc/shared-1c5de2c8-4eb6-43e0-908a-663cc7b2bbb5 调试石墨 API 接口。

### 步骤 4：访问协同文档

写一个最简单的 html 页面，包含一个父容器用来承载石墨的 iframe。

Shimo-js-sdk 地址：https://github.com/shimohq/shimo-js-sdk

1. 使用 npm view 和 npm pack 下载代码包 (.tgz 格式).
   命令行中输入：

```shell
npm view shimo-js-sdk
npm pack shimo-js-sdk
```

会下载 shimo-js-sdk 的代码压缩包，例如 shimo-js-sdk-1.2.1.tgz。
解压这个文件，会得到名为 dist 的文件夹。

2. 新建项目文件夹 shimo-doc-test。

3. 将下载下来的代码包解压，并将 shimo-js-sdk 文件夹放在 shimo-doc-test 文件夹下。

4. 在文件夹内新建两个文件，分别为 index.js 和 index.html，它们的内容如下：

index.js

```js
const { connect, FileType } = window.ShimoJSSDK;
const fileId = "my-new-file-u98213"; // 刚才定义的文件id

// const { signature, token } = await getCredentialsFromServer() // 从您的后端服务获取用于石墨鉴权的签名和 token
const signature = "{your-calculated-signature}"; // 测试时直接写在这里
const token = "{your-customized-token}"; // 测试时直接写在这里

connect({
  debug: true,
  fileId: fileId,
  endpoint: "https://office.shimoapi.com/sdk/v2/api",
  signature: signature,
  token: token,
  container: document.querySelector("#iframe-container"), // iframe 挂载的目标容器元素id
}).then((sdk) => {
  // sdk 即为 ShimoSDK 实例
  // console.log(sdk)
});
```

index.html

```html
<head>
  Hello Shimo
</head>
<div id="iframe-container"></div>
<script src="./shimo-js-sdk/dist/index.js"></script>
<script src="index.js"></script>
```

5. 新建一个文件夹叫 shimo-js-sdk，并将步骤 1 中得到的 dist 文件夹放入其中，此时目录结构如下：

```
shimo-doc-test/
├── index.js
└── index.html
└── shimo-js-sdk/
    └── dist/
```

双击用浏览器打开 index.html 文件，页面中将出现石墨的编辑器（iframe 方式挂载在配置的目标容器下），此时可编辑该协同文档。
