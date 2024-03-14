---
sidebar_position: 8
description: 'This document lists eight types of error code comparisons, including invalid endpoint addresses, obtaining specified file information, current user information, and specified user information. '
---

# Error code comparison
## The access party implements interface error codes
| Error codes   | Error code description | solution|
| :------ | :----- |:----- |
| 10001    | An invalid endpoint address | Confirm that the endpoint callback address prefix of the app is set correctly, and use the`更新应用信息` API to update `endpointUrl` the field |
| 10002    | Obtain the specified file information 404 | Check the access `文件信息`party interface implementation |
| 10003    | Get current user information 404 | Check the access `当前用户`party interface implementation |
| 10004    | Get the specified user information 404 | Check the access `指定用户信息`party interface implementation |
| 10005    | Get the list of user information 404 | Check the access `用户列表信息`party interface implementation |
| 10006    | Get a list of 404 users with collaboration privileges | Check the access `协作权限用户列表`party interface implementation |
| 10007    | Search for the recent contact list 404 | Check the access `获取最近用户`party interface implementation |
| 10008    | Search for the list of recently used files 404 | Check the access `获取最近使用文件`party interface implementation |
| 10009    | An HTTP error occurs on the callback interface of the access party, and the status code is >= 400 | Check the interface of the access party |
| 10010    | Invalid normal HTTP status code, only 200 is recognized | Check the interface implementation of the access party, and the status code of 200 is considered to be normal |
| 10011    | Not legitimate JSON data | Check whether the accessing party returns data correctly as described in the document `JSON 格式数据``字段结构``字段类型` |
| 10012    | File data is not as expected | If the file information does not meet expectations, check the access party`文件信息` interface |
| 10013    | The FileType data is not as expected |  If `获取文件类型`the interface returns does not meet expectations, check whether the implementation of the access party is consistent with the description in the document |
| 10014    | User data is not as expected | `用户信息`If the interface does not meet expectations, check whether the interface implemented by the accessing party is consistent with the relevant documentation |
| 10015    | A network error occurred when requesting the access party to call back the interface | `endpointUrl` When a request is made to the assembled accessor callback interface, a network error is encountered, and whether the Graphite SDK cluster environment cannot be connected to `endpointUrl` the callback address of the accessor pointed to by the domain name or IP, such as the inability to access the public network, network configuration, and other factors  |

## Error codes for application-related APIs
| Error codes   | Error code description | solution|
| :------ | :----- |:----- |
| 20001    | Error in creating app parameters | Check the parameters as prompted |
| 20002    | Failed to create an app | If the occurrence can be ignored, please contact the graphite personnel if it is repeated |
| 20003    | Update app parameter error |Check the parameters as prompted |
| 20004    | The update app failed | If the occurrence can be ignored, please contact the graphite personnel if it is repeated |
| 20005    | The app doesn't exist | Check the URL parameter application ID |

## Error codes on account-related APIs
| Error codes   | Error code description | solution|
| :------ | :----- |:----- |
| 30001    | The parameters for creating an access account are incorrect | Check the parameters as prompted |
| 30002    | Failed to create an access account | If the occurrence can be ignored, please contact the graphite personnel if it is repeated |
| 30003    | The login parameters are incorrect | Check the parameters as prompted |
| 30004    | Login verification failed | If the occurrence can be ignored, please contact the graphite personnel if it is repeated |
| 30005    | Failed to generate tokens | If the occurrence can be ignored, please contact the graphite personnel if it is repeated |
| 30006    | The account and password do not match | Please check whether the account number and password are entered correctly |
| 30007    | The password parameter is incorrect | Check the parameters as prompted |
| 30008    | Failed to change the password | If the occurrence can be ignored, please contact the graphite personnel if it is repeated |

## Error codes related to collaborative documents
| Error codes   | Error code description | solution|
| :------ | :----- |:----- |
| 70001    | The file parameter is incorrect | Check the parameters as prompted |
| 70003    | Failed to initialize file contents | If the occurrence can be ignored, please contact the graphite personnel if it is repeated |
| 70004    | Failed to create file information | If the occurrence can be ignored, please contact the graphite personnel if it is repeated |
| 70005    | Failed to create file history | If the occurrence can be ignored, please contact the graphite personnel if it is repeated |
| 70006    | Initialization of collaborative document content failed or was not initialized | Check whether the following are the following:1. When creating a collaborative document fails, the unprocessed error opens the document directly with the file ID. Need to deal with errors 2. Without waiting for the import task to end, open the document directly with the file ID. Need to get progress until success or failure 3. The object storage service provided by the access party is unavailable |
| 70007    | The content of the initialization collaboration document is abnormal | Please contact graphite personnel when appearing |
| 70008    | Unrecognized file type | Please check the file type |
| 70012    | The file corresponding to the file ID already exists | Please check the file ID |
| 70013    | The file does not exist | Confirm that you can `创建协同文档` access this collaboration document `导入文件` after success or `获取导入进度` after completion. If you encounter a Create Failure (HTTP Status >= 400) or an Import Error (Import File or Get Import Progress Failed), handle it accordingly and should not open the file after the failure |
| 70015    | The source file content was obtained abnormally | If it can be ignored and repeated, please contact graphite personnel for troubleshooting |
| 70016    | The target file already exists, but the Create Copy task was not found | Recurrence: It is recommended to recreate the copy after deleting the file using the target file ID |
| 70017    | Failed to create a copy of the target file | When this error occurs, Graphite deletes the target file association record |
| 70018    | An error is reported when you get the task of creating a replica | If it happens occasionally, you can ignore it and try again, and if it happens, please contact graphite personnel to troubleshoot |
| 70019    | The Create Replica task is in progress | You need to try again to get the latest status |
| 70020    | Unknown quest task status | If it happens occasionally, you can ignore it and try again, and if it happens, please contact graphite personnel to troubleshoot |
| 60010    | The Get Sidebar History parameter is incorrect | Check the parameters as prompted |
| 60011    | Get all historical failures | If the occurrence can be ignored, please contact the graphite personnel if it is repeated |
| 60012    | Historical content errors | If the occurrence can be ignored, please contact the graphite personnel if it is repeated |
| 60014    | Failed to obtain user information created in history | If the occurrence can be ignored, please contact the graphite personnel if it is repeated |
| 100002    | Failed to get version list | If the occurrence can be ignored, please contact the graphite personnel if it is repeated |
| 100003    | Failed to obtain user information for version creation | If the occurrence can be ignored, please contact the graphite personnel if it is repeated |

## File preview
| Error codes   | Error code description | solution|
| :------ | :----- |:----- |
| 90028         | The preview is already being created     | See if you can create multiple previews of the same file at the same time |
| 90031         | Preview import exceptions        | If it happens occasionally, you can try again |
| 90032         | Preview import failed        | Check if the file format is correct |
| 90033         | Preview types are not supported     | Preview types are not supported |
| 90035         | The preview file failed to download     | Check whether the download link returned by the file meta information callback API is normal |
| 90040         | Unknown bug          | For details about the error, see the message field returned, which can be retried |
| 90042         | Files with passwords        | Check if the file has a password |
| 90043         | The preview image is not formatted correctly  | Check the file format |
| 90050, 90052  | The file size is too large           | Try splitting the file before previewing it |
| 90054         | The preview build timed out        | If it happens occasionally, you can try again |

## Import and export
| Error codes   | Error code description | solution|
| :------ | :----- |:----- |
| 120001    | The import parameters are incorrect | Check the parameters as prompted |
| 120002    | The import failed | If the occurrence can be ignored, please contact the graphite personnel if it is repeated |
| 120003    | Import permission error | Please check that you are logged in and that your current account has permissions on the current file |
| 120004    | The import progress query parameter is incorrect | Check the parameters as prompted |
| 120005    | The import progress query failed | If the occurrence can be ignored, please contact the graphite personnel if it is repeated |
| 120006    | The export parameter is incorrect | Check the parameters as prompted |
| 120007    | The export failed | If the occurrence can be ignored, please contact the graphite personnel if it is repeated |
| 120008    | The export progress parameter is incorrect | Check the parameters as prompted |
| 120009    | The export progress query failed | If the occurrence can be ignored, please contact the graphite personnel if it is repeated |
| 120010    | Import progress not found | Check whether an import task has been created for the file |
| 120011    | The import progress result has timed out | The import task has expired and must be recreated |
| 120012    | The exported source file type is not supported | Please check the export formats that are supported by the current document |
| 120013    | Files cannot be exported to their current format | Please export to other formats |
| 120015    | The file has been imported | Please upload the file again and import it again |
| 120016    | Files cannot be imported | Please check the file name as well as the file type |
| 120017    | The import task is not created | Create an import task first |
| 120018    | The import file is too large | Try splitting the file before importing it |
| 120019    | The import file size is invalid | Please check if the import file is empty |
| 120020    | There are no export permissions | Check whether you have export permissions for the file |
| 120022    | The export task has not been created, or has expired | Please recreate the export task |
| 120023    | The export result failed to be parsed or has expired | Please recreate the export task|
| 120024    | The import result failed to be parsed or has expired | Please recreate the import task |
| 120502    | The form is password protected | Please unencrypt and import again |
| 120503    | The parsed data of the table is greater than 80Mbit/s, which exceeds the platform limit. The import could not be successful | Try splitting the table before importing it |
| 120504    | There are broken internal links after the table is parsed | Please delete the internal link before importing it |
| 120505    | The table is too large and the parsing fails | Check whether there is redundant data in the table, and try to delete the redundant blank rows and columns before importing |
| 120506    | Tables are in Strict Open XML format and cannot be imported | Please save it as an Excel workbook (*.xlsx) before importing |
| 120507    | The file format and extension are invalid, and the table cannot be opened | Please check your documents |
| 120508    | Tables are in XLSB format and do not support import | Open Save as Excel Workbook (.xlsx) format before importing |
| 120509    | The number of table cells has exceeded 2 million | Try splitting the table before importing it |
| 120510    | The total number of table cells has exceeded 5 million | Try splitting the table before importing it  |
| 120511    | Import/export exceptions | Please re-import/export |
| 120512    | The import file is too large and the data parsing is abnormal | Try splitting the file before importing it |
| 120513    | The maximum Excel row limit has been reached and cannot be exported | Please remove redundant blank lines |
| 120514    | The export fails due to the file size being too large or the number of images | Split it into multiple tables and export them one by one |
| 120515    | The sheet name cannot contain special characters | Please remove the / \\ ? * : ' ] [ and so on |
| 120516    | The service is currently busy | Please try it later |
| 120517    | The document password is incorrect | Please re-enter |
| 120518    | The limit of the export image format has been exceeded | Please export to other formats |
