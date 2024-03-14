---
sidebar_position: 1
description: 'Graphite Editor SDK integrates with IM, cloud disk and OA, approval and other process office systems, which can convert local files into online files, and organize online files by constructing folders, tags and other systems. '
---

# Brief introduction

## What is the Graphite SDK

- Graphite SDK is a component that separates graphite documents, tables, slides and other editors from the whole site product and can be integrated separately by third-party systems (such as IM and cloud disks).
- The Graphite SDK does not pay attention to "information other than content" such as user accounts, file permissions, file addresses, and file paths. This information is maintained by a third-party system, and can be called by the SDK when used;
- Graphite SDK supports:
  - real-time multiplayer collaboration;
  - Save the body content, including text, images, comments, and other elements, and record the editors of these contents;
  - Save the editing history and support version backtracking;
  - Import, export;
  - preview the file;

## Graphite Editor SDK access form

- Integration with IM;

  - Local Office files sent in IM can be converted into online files with a certain permission range (for example, group members can edit) by using the import capability of the Graphite Editor SDK. Group members can access links to enable real-time collaboration between multiple people. The file itself belongs to a group space or an enterprise cloud disk;
  - In IM, you can directly create a blank online file and invite group members to participate in collaboration.
  - In IM, you can select the online file in the cloud disk to send it (the entry is the same as the sending file, which is actually the sending link);
  - In the Graphite Editor SDK, you can send files to IM chats and give group members corresponding permissions (docking required);

- Integrate with cloud disks;
  - In the cloud disk, you can create, import, and delete online files, organize online files by constructing folders, tags, and other systems, and participate in the setting of permissions together with ordinary files.
  - In the cloud disk, you can forward online files to IM and give group members corresponding permissions (docking is required);

![Image](./img/sdk-intro3.png)

_Legend - Cloud disk integration_

- Integrate with OA, approval, and other process-based office systems;
  - In various process office systems, in addition to "add local files", you can also add "add online files" to add links to online files as attachments to the process. When the data is transferred to each node, the members of the node can be automatically granted online file permissions.
  - In order to prevent online documents from being tampered with during the approval process, you can set the permission to only read or comment, and if a node in the process requires multiple people to collaborate on editing (mostly seen in the information collection at the beginning of the process, such as forms), you can also open the corresponding permissions;
