---
title: 概述
description: 介绍API的功能及使用方法
keywords: [石墨文档, 文档中台, 协同办公, 在线文档, 文件共享, API, 文档处理, 鉴权]
sidebar_position: 1
---

石墨文档中台提供了丰富的API接口用于文档处理，例如创建协同文档、获取表格内容等。下面从系统接口、协同编辑和文件操作三个维度，详细阐述API接口的请求地址、请求参数、请求示例等，让开发者快速熟悉和使用石墨API接口，实现业务功能需求。

:::caution 说明

- 如无特别说明，所有请求均为 `HTTP`，请求头和返回头的 `Content-Type` 均为 `application/json`。
- `https://shimo-domain` 指石墨 SDK 服务器域名，请联系石墨人员获取此域名。
- 路由中的参数值如果有特殊符号，建议将值进行转义，JavaScript 转义方法请参考 [`MDN`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/encodeURIComponent?fileGuid=Wr3DVn8lO4HE2kJQ)。
- 如无特殊说明，此页面下所有接口均需要鉴权，详细操作请参考 [鉴权方式](./../06API-document/interface-description/system-interface.md#auth-with-querystring)。

:::