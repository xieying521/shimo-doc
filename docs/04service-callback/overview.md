---
title: 概述
sidebar_position: 1
---

石墨文档中台专注于对文档本身的预览、编辑以及协同功能，对接入方系统中的用户信息、组织架构、文件权限等方面都没有入侵。

基于此原则，接入方需要提供回调接口，让石墨文档中台可以获取文件元信息（包括文件下载地址，文件权限信息等）、当前用户信息、组织架构信息等，用于打通石墨文档中台与接入方文件系统。

## 回调凭证类型{#credential-types}

请求回调接口存在不同的触发场景，因此存在不同的凭证类型，通过回调请求的 HTTP Header 指定 `X-Shimo-Credential-Type`。

已知的凭证类型：

- `X-Shimo-Credential-Type` == `0`，此类型通常是由用户的操作行为触发，比如打开页面、编辑文件等，请求会通过 HTTP Header `X-Shimo-Token` 携带用户在浏览器操作时接入方颁发的 `token`。
- `X-Shimo-Credential-Type` == `3`, 此类型通常由石墨文档中台主动触发，如跨表格引用的更新等，请求会通过 HTTP Header `X-Shimo-Signature` 根据接入方当前的 `appId` `secret` 主动签名后供接入方回调进行校验，确认此请求是否来自石墨文档中台。
