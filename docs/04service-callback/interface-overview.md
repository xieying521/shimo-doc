---
title: 接口总览
sidebar_position: 3
---


石墨开放与服务平台提供了一系列接口，帮助企业对接石墨在线协同能力。

:::caution 说明

- 如果没有特别说明，所有请求均为 `HTTP`，请求头和返回头的 `Content-Type` 均为 `application/json`。
- 接口里的 `{xxxx}` 表示需要用户在接口地址里使用自己的值来进行替换，比如 `/users/{userId}` 的真实调用地址可能是 `/users/123`，在接口描述里会将此参数以 `userId` 的形式进行描述。
- /admin 前缀的接口，主要由用户动作产生的间接请求，可能无法携带用户身份或者用户身份无关，如：用户A 创建了表格1引用了表格2并分享给用户B，当用户B访问表格1时，需要检查用户A是否还有权限访问表格2。

:::

|接口|接口用途|说明|
|:----|:----|:----|
|GET [/files](./../04service-callback/file-information.md#list-files)|获取当前用户的文件列表|添加跨表格公式、合并工作表等选择用户最近使用的文件列表会请求此接口|
|GET [/files/{fileId}](./../04service-callback/file-information.md#file-meta-collab)|获取文件元信息-协同文档|获取基本信息、权限等，创建、导入、编辑会请求此接口|
|GET [/files/{fileId}](./../04service-callback/file-information.md#file-meta-preview)|获取文件元信息-文件预览|获取基本信息、下载地址，预览文件时会请求此接口|
|GET [/files/{fileId}/collaborators](./../04service-callback/file-information.md#file-collborators)|获取当前的协作用户|用于获取此协作文件的协作者（接入方可自己控制哪些用户为协作者）用户列表信息，如评论实时同步给其他用户时用于获取协作者列表发送评论事件|
|POST [/files/{fileId}/url](./../04service-callback/file-information.md#file-url)|获取接入方指定文件的完整访问地址|在文件中 @ 其他文件时，在将此文件导出后的文件中会保留一个石墨 SDK 的跳转链接地址，打开链接时，会通过此接口获取当前完整的接入方访问链接，获取后会重定向到接入方文件地址|
|GET [/admin/files/{fileId}](./../04service-callback/file-information.md#file-meta-admin)|获取文件元信息-协同文档自动任务|自动任务获取基本信息（目前只有跨表格引用任务）时会请求此接口|
|GET [/admin/files/{fileId}/by-user-id](./../04service-callback/file-information.md#file-meta-by-uid)|根据指定用户获取文件元信息-协同文档|石墨 SDK 需要主动获取文件基本信息（目前只有跨表格引用场景）时会请求此接口|
|GET [/users/current/info](./../04service-callback/user-information.md#user-current)|获取当前用户信息|获取当前用户的 ID、名称、头像图片地址等信息，创建、编辑、导入时需使用此接口获取用户身份信息|
|GET [/users/current/team](./../04service-callback/user-information.md#user-current-team)|获取当前用户所在团队信息|获取当前用户所在团队的 ID、名称、团队成员总数|
|GET [/users/{userId}](./../04service-callback/user-information.md#user-by-id)|获取指定用户信息|提供用户的基本信息，ID、名称、头像图片地址等，在获取协作场景下的指定用户信息时会请求此接口|
|GET [/users/{userId}/watermark](./../04service-callback/user-information.md#user-watermark)|获取用户水印信息|获取指定用户的水印信息，可根据自身业务系统需要获取返回需要的水印字段或返回空|
|GET [/users/{userId}/department-paths](./../04service-callback/user-information.md#user-department-paths)|获取用户部门路径|获取指定用户的部门路径，可根据自身业务系统需要获取返回需要的部门路径或返回空|
|POST [/users/batch/get](./../04service-callback/user-information.md#users-by-ids)|批量获取用户信息|提供多个指定用户的信息，ID、名称、头像图片地址等。如多人协作时，侧边栏历史展示时需要展示多个用户的相关信息，此时会请求此接口|
|GET [/teams/{teamId}/members](./../04service-callback/team-information.md#team-members)|获取团队下的成员列表|获取指定团队的成员列表，根据分页信息查询列表结果|
|GET [/departments/{departmentId}](./../04service-callback/team-information.md#department-info)|获取部门信息|获取指定部门的信息，包括部门 ID、名称、部门成员总数|
|GET [/departments/{departmentId}/children](./../04service-callback/team-information.md#children-departments)|获取部门的下级部门节点|获取指定部门的下级部门节点，包括部门信息 ID、名称、部门成员总数|
|GET [/departments/{departmentId}/members](./../04service-callback/team-information.md#department-members)|获取部门下的成员分页列表|获取指定部门下的成员分页列表，获取当前分页的用户信息列表以及总成员数|
|GET [/search/users/recent](./../04service-callback/search-function.md#users-recent)|获取与文件相关的用户列表|用于在文件中输入 @ 时，为展示下拉菜单，选择提及的用户，由于此时没有输入关键字，接入方可根据自己系统情况返回，如：和当前用户最相关的用户列表、和当前文件最相关的用户列表、或返回空数组|
|GET [/search/files/recent](./../04service-callback/search-function.md#files-recent)|获取与文件相关的文件列表|用于在文件中输入 @ 时，展示下拉菜单，选择提及的文件，由于此时没有输入关键字，接入方可根据自己系统情况返回，如：和当前用户最相关的文件列表、和当前文件最相关的文件列表、或返回空数组|
|POST [/search](./../04service-callback/search-function.md#search-by-keyword)|根据关键字搜索文件或用户|用于输入 @ 时，如需要根据关键字，通过请求接入方系统返回对应的文件列表、用户列表，如不需要可将对应字段返回空数组。表格锁定添加人员时涉及此接口|
|POST [/events](./../04service-callback/push-message.md)|推送石墨 SDK 相关事件|由接入商选择性处理需要关心的事件数据。如当有人对评论、讨论、提及等进行操作时，部分操作会触发此事件|

