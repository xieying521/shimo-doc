---
sidebar_position: 4
toc_max_heading_level: 4
description: >-
  Then, you need to apply for the appId and secret, and after obtaining them, you can obtain the app details and update the app callback address. When you connect to the interface, you need to understand or prepare the signature, parameters, web page, and callback interface.
  The process includes calling the Create API to create a graphite document, introducing the graphite JSSDK, initializing the editor, creating a file preview, and supporting file types and extension formats. You need to pay attention to HTTP
  Response Header and File Extension fields.
---

# Start the connection

:::caution Note
⚠️ Make sure you have read [`接入须知`](./concepts/concepts.md#接入须知)it. 

⚠️ Make sure to call as specified , `地址``方法`, ,`HTTP Header` ,
:::

## Application `appId``secret`

Contact Graphite to obtain the and for docking with Graphite SDK `appId` `secret`, and you can perform the following operations after obtaining it:

- [Get app details](./apis.md#get-app-detail)
  - Used to view details about the app
- [Update the callback address of the app](apis#update-app-endpoint-url)
  - Update the callback address of the access party requested by the Graphite SDK (the update takes about 1 minute to take effect)

The accessing party needs to keep the obtained `appId` and `secret` . 

## Collaborative documentation

Before you start accessing the collaboration document, you need to understand or prepare the following items:

- The signature** (see **Signature Implementation Details[) parameter ](./resources.md#签名)that needs to be provided for access 
- The web page **developed by the accessing party ** is used to host the graphite editor
- Develop a number of **callback interfaces**
  - It is used to obtain the corresponding information in the Graphite SDK `协作` or `预览` scenes, and the callback interfaces that need to be implemented for common functions refer to [ the list of interfaces related to common function scenarios](./impl.md#features-api-list)

:::tip tip
⚠️ To achieve the most basic document collaboration, the access party needs to develop at least the following two interfaces:

- [`获取文件元信息`](./impl.md#获取文件元信息-协同文档)
- [`获取当前用户信息`](./impl.md#获取当前用户信息)

If you want to use the full functionality, implement all of the [`接入方实现接口`](./impl.md) features  or implement [`常见功能场景相关接口清单`](./impl.md#features-api-list) the features required by the department.
:::

### Access process

1. Call the Create API  to [create a graphite document](./apis.md#create-collab-file)
2. In the web page developed by the accessing party, the [graphite JSSDK is introduced](./resources.md#js-sdk)
3. Follow the  prompts to initialize the editor according to[ the README ](https://www.npmjs.com/package/shimo-js-sdk)in the JSSDK npm package 

## File preview

The online preview function refers to the `docx``xlsx` `pptx` conversion of Office files such as , etc., into a form that can be previewed and viewed in the browser. 

:::caution Note
⚠️ This feature does not support editing documents, if you want to edit documents, please refer to [`协同文档`](#协同文档).
:::

### Support format {#preview-supported-types}

| file type   | Supported extension formats                                           |
| :--------- | :--------------------------------------------------------- |
| Word file  | `.doc` `.docx` `.wps` `.wpt`                               |
| Excel file | `.xls` `.xlsx` `.xlsm` `.csv`                              |
| PPT file   | `.ppt` `.pptx`                                             |
| PDF file   | `.pdf` `.rtf`                                              |
| TXT file   | `.txt`                                                     |
| Audio files  | `.mp3` `.mp4`                                              |
| Picture file  | `.jpg` `.jpeg` `.png` `.gif` `.bmp` `.svg` `.heic` `.heif` |
| Other documents  | `.ofd` `.rtf` `.md`                                        |

Before you start previewing the access file, you need to know or prepare the following items:

- [Implementation details of the signature](./resources.md#签名)

#### Access instructions

1. To create a preview task[ by using ](./apis.md#创建预览)Create Preview, you need to [ specify the file download address in the downloadUrl field of](./impl.md#获取文件元信息-文件预览) the API to get file meta information, for example`http(s)://{domain}/static/files/2017.docx`, the graphite server will download the file from this address and process it. Make sure that the address can be accessed by the graphite server. 

1. Preview the file by [visiting the preview page](./apis.md#访问预览). 

### HTTP Response Header requirements

**⚠️ Note:**

In order to ensure that the file type can be correctly recognized, the HTTP response header when the download address is accessed ** must ** ensure one of the following conditions (if the return is incorrect, the preview will fail due to the incorrect identification of the file type).

#### Content-Disposition

The HTTP Response**Header **needs to **return the correct Content-Disposition information** (see [MDN Content-Disposition](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Headers/Content-Disposition)), as follows:

```
Content-Disposition: attachment; filename="yourfilename.docx"
```

#### Content-Type

The HTTP Response**Header **needs to **return the correct Content-Type information**, as follows

![Image](./img/sdk-preview-header.png)

Several types of Content-Type examples

| Extension | The corresponding Content-Type value                                                    |
| :----- | :------------------------------------------------------------------------ |
| Doc    | application/msword                                                        |
| docx   | application/vnd.openxmlformats-officedocument.wordprocessingml.document   |
| xlsx   | application/vnd.openxmlformats-officedocument.spreadsheetml.sheet         |
| xls    | application/vnd.ms-excel                                                  |
| ppt    | application/vnd.ms-powerpoint                                             |
| pptx   | application/vnd.openxmlformats-officedocument.presentationml.presentation |
| pdf    | application/pdf                                                           |
| txt    | text/plain                                                                |

### File extension field

The  extension information of the file is returned in [ the Field of  Get ](./impl.md#获取文件元信息-文件预览)File Meta Information API - File Preview`ext`. 

As:

The preview file download address (`downloadUrl`) is  , `https://your-domain/static/files/1234` and the accessing party cannot return any of the following header information when requesting the download, which is used by the Graphite SDK to identify the file type:

- `Content-Type`
- `Content-Disposition`

This can be done via the field returned in the meta information `ext` . 

If you `ext: "docx"` tell the Graphite SDK that the extension of this file is recognized as a Word file, see the following callback interface example highlighted line. 

```json title="通过回调接口中的 ext 字段指定文件类型"
{
  "id": "ba13551165cc5066",
  "name": "示例文档.docx",
  "type": "file",
  "permissions": {
    "readable": true
  },
  "downloadUrl": "http://example.com/download/test.docx",
  // highlight-next-line
  "ext": "docx" // 用于标识文件类型
}
```
