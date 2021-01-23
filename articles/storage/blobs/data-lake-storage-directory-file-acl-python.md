---
title: Azure Data Lake Storage Gen2 Python SDK for files & Acl
description: 在已啟用階層命名空間 (HNS) 的儲存體帳戶中，使用 Python [管理目錄] 和 [檔案和目錄存取控制清單 (ACL) 。
author: normesta
ms.service: storage
ms.date: 01/22/2021
ms.author: normesta
ms.topic: how-to
ms.subservice: data-lake-storage-gen2
ms.reviewer: prishet
ms.custom: devx-track-python
ms.openlocfilehash: 5036930c7bb49578582fbc1b347b11518579b53e
ms.sourcegitcommit: 6272bc01d8bdb833d43c56375bab1841a9c380a5
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 01/23/2021
ms.locfileid: "98740613"
---
# <a name="use-python-to-manage-directories-files-and-acls-in-azure-data-lake-storage-gen2"></a>使用 Python 來管理 Azure Data Lake Storage Gen2 中的目錄、檔案和 Acl

本文說明如何使用 Python 來建立和管理已啟用階層命名空間 (HNS) 之儲存體帳戶中的目錄、檔案和許可權。 

[套件 (Python 套件索引) ](https://pypi.org/project/azure-storage-file-datalake/)  | [範例](https://github.com/Azure/azure-sdk-for-python/tree/master/sdk/storage/azure-storage-file-datalake/samples)  | [API 參考](/python/api/azure-storage-file-datalake/azure.storage.filedatalake)  | [Gen1 至 Gen2 對應](https://github.com/Azure/azure-sdk-for-python/tree/master/sdk/storage/azure-storage-file-datalake/GEN1_GEN2_MAPPING.md)  | [提供意見](https://github.com/Azure/azure-sdk-for-python/issues)反應

## <a name="prerequisites"></a>Prerequisites

> [!div class="checklist"]
> * Azure 訂用帳戶。 請參閱[取得 Azure 免費試用](https://azure.microsoft.com/pricing/free-trial/)。
> * 已啟用階層命名空間 (HNS) 的儲存體帳戶。 遵循[下列](../common/storage-account-create.md)指示以建立帳戶。

## <a name="set-up-your-project"></a>設定專案

使用 [pip](https://pypi.org/project/pip/)安裝適用于 Python 的 Azure Data Lake Storage 用戶端程式庫。

```
pip install azure-storage-file-datalake
```

將這些 import 語句新增至程式碼檔案的頂端。

```python
import os, uuid, sys
from azure.storage.filedatalake import DataLakeServiceClient
from azure.core._match_conditions import MatchConditions
from azure.storage.filedatalake._models import ContentSettings
```

## <a name="connect-to-the-account"></a>連線到帳戶

若要使用本文中的程式碼片段，您必須建立代表儲存體帳戶的 **DataLakeServiceClient** 實例。 

### <a name="connect-by-using-an-account-key"></a>使用帳戶金鑰進行連接

這是連接到帳戶的最簡單方式。 

此範例會使用帳戶金鑰建立 **DataLakeServiceClient** 實例。

:::code language="python" source="~/azure-storage-snippets/blobs/howto/python/python-v12/crud_datalake.py" id="Snippet_AuthorizeWithKey":::
 
- 使用您的儲存體帳戶名稱取代 `storage_account_name` 預留位置值。

- 將 `storage_account_key` 預留位置值取代為您的儲存體帳戶存取金鑰。

### <a name="connect-by-using-azure-active-directory-ad"></a>使用 Azure Active Directory (AD) 連接

您可以使用 [適用于 Python 的 Azure 身分識別用戶端程式庫](https://pypi.org/project/azure-identity/) ，以 Azure AD 來驗證您的應用程式。

此範例會使用用戶端識別碼、用戶端密碼和租使用者識別碼來建立 **DataLakeServiceClient** 實例。  若要取得這些值，請參閱 [從 Azure AD 取得權杖，以授權用戶端應用程式的要求](../common/storage-auth-aad-app.md)。

:::code language="python" source="~/azure-storage-snippets/blobs/howto/python/python-v12/crud_datalake.py" id="Snippet_AuthorizeWithAAD":::

> [!NOTE]
> 如需更多範例，請參閱 [適用于 Python 的 Azure 身分識別用戶端程式庫](https://pypi.org/project/azure-identity/) 檔。

## <a name="create-a-container"></a>建立容器

容器可作為檔案的檔案系統。 您可以藉由呼叫 **FileSystemDataLakeServiceClient.create_file_system** 方法來建立一個。

這個範例會建立名為的容器 `my-file-system` 。

:::code language="python" source="~/azure-storage-snippets/blobs/howto/python/python-v12/crud_datalake.py" id="Snippet_CreateContainer":::

## <a name="create-a-directory"></a>建立目錄

藉由呼叫 **FileSystemClient.create_directory** 方法來建立目錄參考。

此範例會將名為 `my-directory` 的目錄新增至容器。 

:::code language="python" source="~/azure-storage-snippets/blobs/howto/python/python-v12/crud_datalake.py" id="Snippet_CreateDirectory":::

## <a name="rename-or-move-a-directory"></a>重新命名或移動目錄

藉由呼叫 **DataLakeDirectoryClient.rename_directory** 方法來重新命名或移動目錄。 傳遞所需目錄的路徑給參數。 

此範例會將子目錄重新命名為名稱 `my-subdirectory-renamed` 。

:::code language="python" source="~/azure-storage-snippets/blobs/howto/python/python-v12/crud_datalake.py" id="Snippet_RenameDirectory":::

## <a name="delete-a-directory"></a>刪除目錄

藉由呼叫 **DataLakeDirectoryClient.delete_directory** 方法來刪除目錄。

此範例刪除名為 `my-directory` 的目錄。  

:::code language="python" source="~/azure-storage-snippets/blobs/howto/python/python-v12/crud_datalake.py" id="Snippet_DeleteDirectory":::

## <a name="upload-a-file-to-a-directory"></a>將檔案上傳至目錄 

首先，藉由建立 **DataLakeFileClient** 類別的實例，在目標目錄中建立檔案參考。 藉由呼叫 **DataLakeFileClient.append_data** 方法來上傳檔案。 請務必藉由呼叫 **DataLakeFileClient.flush_data** 方法來完成上傳。

此範例會將文字檔上傳至名為的目錄 `my-directory` 。   

:::code language="python" source="~/azure-storage-snippets/blobs/howto/python/python-v12/crud_datalake.py" id="Snippet_UploadFile":::

> [!TIP]
> 如果您的檔案大小很大，您的程式碼就必須對 **DataLakeFileClient.append_data** 方法進行多個呼叫。 請考慮改為使用 **DataLakeFileClient.upload_data** 方法。 如此一來，您就可以在單一呼叫中上傳整個檔案。 

## <a name="upload-a-large-file-to-a-directory"></a>將大型檔案上傳至目錄

使用 **DataLakeFileClient.upload_data** 方法來上傳大型檔案，而不需要對 **DataLakeFileClient.append_data** 方法進行多個呼叫。

:::code language="python" source="~/azure-storage-snippets/blobs/howto/python/python-v12/crud_datalake.py" id="Snippet_UploadFileBulk":::

## <a name="download-from-a-directory"></a>從目錄下載 

開啟要寫入的本機檔案。 然後，建立代表您要下載之檔案的 **DataLakeFileClient** 實例。 呼叫 **DataLakeFileClient.read_file** 從檔案讀取位元組，然後將這些位元組寫入本機檔案。 

:::code language="python" source="~/azure-storage-snippets/blobs/howto/python/python-v12/crud_datalake.py" id="Snippet_DownloadFromDirectory":::

## <a name="list-directory-contents"></a>列出目錄內容

藉由呼叫 **FileSystemClient.get_paths** 方法來列出目錄內容，然後列舉結果。

此範例會列印每個子目錄和檔案的路徑，該目錄位於名為的目錄中 `my-directory` 。

:::code language="python" source="~/azure-storage-snippets/blobs/howto/python/python-v12/crud_datalake.py" id="Snippet_ListFilesInDirectory":::

## <a name="manage-access-control-lists-acls"></a> (Acl) 管理存取控制清單

您可以取得、設定及更新目錄和檔案的存取權限。

> [!NOTE]
> 如果您使用 Azure Active Directory (Azure AD) 來授與存取權，請確定已將 [儲存體 Blob 資料擁有者角色](../../role-based-access-control/built-in-roles.md#storage-blob-data-owner)指派給您的安全性主體。 若要深入了解如何套用 ACL 權限以及變更權限的效果，請參閱 [Azure Data Lake Storage Gen2 中的存取控制](./data-lake-storage-access-control.md)。

### <a name="manage-directory-acls"></a>管理目錄 Acl

藉由呼叫 **DataLakeDirectoryClient.get_access_control** 方法來取得目錄 (acl) 的存取控制清單，並藉由呼叫 **DataLakeDirectoryClient.set_access_control** 方法來設定 acl。

> [!NOTE]
> 如果您的應用程式使用 Azure Active Directory (Azure AD) 來授與存取權，請確定您的應用程式用來授與存取權的安全性主體已獲指派 [儲存體 Blob 資料擁有者角色](../../role-based-access-control/built-in-roles.md#storage-blob-data-owner)。 若要深入了解如何套用 ACL 權限以及變更權限的效果，請參閱 [Azure Data Lake Storage Gen2 中的存取控制](./data-lake-storage-access-control.md)。

這個範例會取得並設定名為之目錄的 ACL `my-directory` 。 字串 `rwxr-xrw-` 提供擁有使用者的讀取、寫入和執行許可權，僅提供擁有群組的讀取和執行許可權，並提供所有其他的讀取和寫入權限。

:::code language="python" source="~/azure-storage-snippets/blobs/howto/python/python-v12/ACL_datalake.py" id="Snippet_ACLDirectory":::

您也可以取得及設定容器根目錄的 ACL。 若要取得根目錄，請呼叫 **FileSystemClient._get_root_directory_client** 方法。

### <a name="manage-file-permissions"></a>管理檔案許可權

藉由呼叫 **DataLakeFileClient.get_access_control** 方法，取得檔案 (acl) 的存取控制清單，並藉由呼叫 **DataLakeFileClient.set_access_control** 方法來設定 acl。

> [!NOTE]
> 如果您的應用程式使用 Azure Active Directory (Azure AD) 來授與存取權，請確定您的應用程式用來授與存取權的安全性主體已獲指派 [儲存體 Blob 資料擁有者角色](../../role-based-access-control/built-in-roles.md#storage-blob-data-owner)。 若要深入了解如何套用 ACL 權限以及變更權限的效果，請參閱 [Azure Data Lake Storage Gen2 中的存取控制](./data-lake-storage-access-control.md)。

這個範例會取得並設定名為之檔案的 ACL `my-file.txt` 。 字串 `rwxr-xrw-` 提供擁有使用者的讀取、寫入和執行許可權，僅提供擁有群組的讀取和執行許可權，並提供所有其他的讀取和寫入權限。

:::code language="python" source="~/azure-storage-snippets/blobs/howto/python/python-v12/ACL_datalake.py" id="Snippet_FileACL":::

### <a name="set-an-acl-recursively"></a>以遞迴方式設定 ACL

您可以在父目錄的現有子專案上以遞迴方式新增、更新和移除 Acl，而不需要為每個子專案個別進行這些變更。 如需詳細資訊，請參閱 [設定 (acl 的存取控制清單) 以遞迴方式進行 Azure Data Lake Storage Gen2](recursive-access-control-lists.md)。

## <a name="see-also"></a>另請參閱

* [API 參考文件](/python/api/azure-storage-file-datalake/azure.storage.filedatalake)
* [套件 (Python 套件索引)](https://pypi.org/project/azure-storage-file-datalake/) \(英文\)
* [範例](https://github.com/Azure/azure-sdk-for-python/tree/master/sdk/storage/azure-storage-file-datalake/samples)
* [Gen1 至 Gen2 對應](https://github.com/Azure/azure-sdk-for-python/tree/master/sdk/storage/azure-storage-file-datalake/GEN1_GEN2_MAPPING.md)
* [已知問題](data-lake-storage-known-issues.md#api-scope-data-lake-client-library)
* [提供意見反應](https://github.com/Azure/azure-sdk-for-python/issues)