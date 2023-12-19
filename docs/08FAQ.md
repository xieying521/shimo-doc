---
title: FAQ
sidebar_position: 8
---

## 咨询问题

**Q:石墨文档中台支持导入什么格式的文档？**

A:请参考产品介绍中的 [文件导入支持的格式](./01overview.md)。

**Q:石墨文档中台是否必须要认证以后才能体验相关核心功能？**

A:是的，必须上传营业执照，认证合法的企业资质才能体验。

**Q:石墨文档中台最多可以创建多少个应用？**

A:测试应用为 1 个，正式应用不限数量。

## 使用问题

**Q:石墨文档中台有导出 ofd 文件的接口吗？**

A:目前不支持导出 ofd 格式。支持导出的文件格式请参考 [文件导出支持的格式](./01overview.md)。

**Q:石墨文档中台如何基于我的文件模板，新建文件？**

A:石墨文档中台目前不提供模板，可以新建一个文件作为模板，然后再创建副本。

**Q:石墨文档中台前端 SDK 有 API 文档吗？**

A:石墨提供了丰富的前端 API，使用文档正在编写中，敬请期待。

**Q:石墨文档中台如何查看只读状态下的在线文档的历史记录？**

A:请参考文档 [获取历史信息](./06API-document/interface-description/collaborative-editing.md#doc-sidebar-info)。

**Q:石墨文档中台插入书签后，书签内容能否替换为表格?**

A:石墨传统文档支持书签替换功能，可替换为纯文本或整篇传统文档，详细内容请参考 [替换书签内容](./06API-document/interface-description/collaborative-editing.md#replace-bookmark)。

**Q:石墨文档中台 API 如何设置只读和编辑?**

A:回调接口控制当前用户权限，请参考 [文件权限说明](./04service-callback/file-information.md#file-permission) 文档。

**Q:石墨文档中台在哪里可以获取文档 URL，便于接入方自定义参数？**

A:文档的url由接入方自己决定，只需要实例化 js-sdk 即可将石墨编辑器嵌入您的页面中。

**Q:石墨文档中台为什么打开的文档是只读模式？**

A:辛苦检查以下内容：
1. 检查回调接口的权限字段配置
2. 检查前端 js-sdk 接入时更新签名的策略，签名过期也会导致无法编辑

**Q:石墨文档中台如何设置书签？**

A:传统文档支持用户手动插入书签。

**Q:石墨文档中台是否有导出 pdf 的功能有服务端接口调用的方式？**

A:请参考 [导出文件](./06API-document/interface-description/file-operation.md#export-v1) 文档。

**Q:在石墨文档中台如何获取 excel 表格数据？**

A:请参考 [获取表格内容](./06API-document/interface-description/collaborative-editing.md#get-table-content) 文档。

**Q:石墨文档中台使用时，预览文档和协同文档用的同一个元数据回调接口，怎么确定它是预览功能还是协同功能呢？**

A:石墨服务开放平台有有两种文件类型：【预览文档】和【协同文档】且每种文件类型都是固定不变的。

其中【协同文档】既支持预览访问，也支持编辑访问，但【预览文档】只支持预览访问。所以当生成fileId的时候，这个文件是预览还是协同，就已经确定了。

**Q:创建了协同文档，但是无法打开预览页面，如何处理？**

A:需要确认以下信息：

1.确认文件元信息回调接口是否缺少字段

2.确认创建协同文档时填的 type 和回调接口中的 type 是否一致，例如 document 和 documentPro 就是两种类型

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

**Q:使用石墨文档时，如何保存文档保存？**

A:文档不用手动保存，在编辑的过程中文档会实时自动保存。

**Q:使用石墨文档时，如何保存文件的最新版本？**

A:通过 /events 回调接口的 [文件内容更新](./04service-callback/push-message.md#update-file) 实现。

**Q:石墨文档中台升级时，前端需要对接升级吗？**

A:建议使用最新的 shimo-js-sdk，具体操作请参考文档 [石墨JS-SDK安装](./05shimo-jssdk/installation.md)。

**Q:用户在获取 license 信息以及修改回调地址时，显示返回 400，该如何处理？**

A:请参考以下步骤进行操作：
1. 首先排查 signature 是否计算正确，是否过期
2. 然后检查 signature 过期时间是否设置过长（大于 5 分钟），获取 App 详情接口是严格接口，过期时间不能大于 5 分钟。

**Q:石墨文档中台如何把本地文件通过在线编辑器打开？**

A:可以使用  [导入文件](./06API-document/interface-description/file-operation.md#import-v1) 接口，将本地文件导入至文档中台后，对接 js-sdk，即可在页面中在线访问文档。

**Q:石墨文档中台如何获取 file_id？**

A:file_id 由接入方自行定义，传入石墨文档中台。

**Q:石墨文档中台是否支持水印功能？**

A:支持水印功能，接入方实现 [获取用户水印信息](./04service-callback/user-information.md#user-watermark) 接口后，可展示水印。

**Q:石墨文档中台在那里获取文件下载地址？**

A:可以调用 [导出文件](./06API-document/interface-description/file-operation.md#export-v1) 接口创建导出任务，然后通过 [获取导出进度](./06API-document/interface-description/file-operation.md#export-progress-v1) 接口获取文件下载地址。
