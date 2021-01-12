---
title: Azure Data Lake Storage Gen2 JAVA SDK for files & Acl
description: 使用適用于 JAVA 的 Azure 儲存體程式庫，在已啟用階層命名空間 (HNS) 的儲存體帳戶中，管理目錄和檔案和目錄存取控制清單 (ACL) 。
author: normesta
ms.service: storage
ms.date: 01/11/2021
ms.custom: devx-track-java
ms.author: normesta
ms.topic: how-to
ms.subservice: data-lake-storage-gen2
ms.reviewer: prishet
ms.openlocfilehash: 1cc6954569c509c977634a8e1cdd52c5c55b2100
ms.sourcegitcommit: 48e5379c373f8bd98bc6de439482248cd07ae883
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 01/12/2021
ms.locfileid: "98108122"
---
# <a name="use-java-to-manage-directories-files-and-acls-in-azure-data-lake-storage-gen2"></a>使用 JAVA 管理 Azure Data Lake Storage Gen2 中的目錄、檔案和 Acl

本文說明如何使用 JAVA 來建立和管理已啟用階層命名空間 (HNS) 之儲存體帳戶中的目錄、檔案和許可權。 

[封裝 (Maven) ](https://search.maven.org/artifact/com.azure/azure-storage-file-datalake)  | [範例](https://github.com/Azure/azure-sdk-for-java/tree/master/sdk/storage/azure-storage-file-datalake)  | [API 參考](/java/api/overview/azure/storage-file-datalake-readme)  | [Gen1 至 Gen2 對應](https://github.com/Azure/azure-sdk-for-java/tree/master/sdk/storage/azure-storage-file-datalake/GEN1_GEN2_MAPPING.md)  | [提供意見](https://github.com/Azure/azure-sdk-for-java/issues)反應

## <a name="prerequisites"></a>必要條件

> [!div class="checklist"]
> * Azure 訂用帳戶。 請參閱[取得 Azure 免費試用](https://azure.microsoft.com/pricing/free-trial/)。
> * 已啟用階層命名空間 (HNS) 的儲存體帳戶。 遵循[下列](../common/storage-account-create.md)指示以建立帳戶。

## <a name="set-up-your-project"></a>設定專案

若要開始使用，請開啟 [此頁面](https://search.maven.org/artifact/com.azure/azure-storage-file-datalake) ，並尋找最新版本的 JAVA 程式庫。 然後，在文字編輯器中開啟 *pom.xml* 檔案。 加入參考該版本的相依性專案。

如果您打算使用 Azure Active Directory (AD) 來驗證用戶端應用程式，請將相依性新增至 Azure Secret 用戶端程式庫。 請參閱  [將秘密用戶端程式庫套件新增至您的專案](https://github.com/Azure/azure-sdk-for-java/tree/master/sdk/identity/azure-identity#adding-the-package-to-your-project)。

接下來，將這些 imports 語句新增至您的程式碼檔案。

```java
import com.azure.storage.common.StorageSharedKeyCredential;
import com.azure.storage.file.datalake.DataLakeDirectoryClient;
import com.azure.storage.file.datalake.DataLakeFileClient;
import com.azure.storage.file.datalake.DataLakeFileSystemClient;
import com.azure.storage.file.datalake.DataLakeServiceClient;
import com.azure.storage.file.datalake.DataLakeServiceClientBuilder;
import com.azure.storage.file.datalake.models.ListPathsOptions;
import com.azure.storage.file.datalake.models.PathItem;
import com.azure.storage.file.datalake.models.AccessControlChangeCounters;
import com.azure.storage.file.datalake.models.AccessControlChangeResult;
import com.azure.storage.file.datalake.models.AccessControlType;
import com.azure.storage.file.datalake.models.PathAccessControl;
import com.azure.storage.file.datalake.models.PathAccessControlEntry;
import com.azure.storage.file.datalake.models.PathPermissions;
import com.azure.storage.file.datalake.models.PathRemoveAccessControlEntry;
import com.azure.storage.file.datalake.models.RolePermissions;
import com.azure.storage.file.datalake.options.PathSetAccessControlRecursiveOptions;
```

## <a name="connect-to-the-account"></a>連線到帳戶 

若要使用本文中的程式碼片段，您必須建立代表儲存體帳戶的 **DataLakeServiceClient** 實例。 

### <a name="connect-by-using-an-account-key"></a>使用帳戶金鑰進行連接

這是連接到帳戶的最簡單方式。 

此範例會使用帳戶金鑰建立 **DataLakeServiceClient** 實例。

:::code language="java" source="~/azure-storage-snippets/blobs/howto/Java/Java-v12/src/main/java/com/datalake/manage/Authorize_DataLake.java" id="Snippet_AuthorizeWithKey":::

### <a name="connect-by-using-azure-active-directory-azure-ad"></a>使用 Azure Active Directory (Azure AD 進行連接) 

您可以使用 [適用于 JAVA 的 Azure 身分識別用戶端程式庫](https://github.com/Azure/azure-sdk-for-java/tree/master/sdk/identity/azure-identity) ，透過 Azure AD 來驗證您的應用程式。

此範例會使用用戶端識別碼、用戶端密碼和租使用者識別碼來建立 **DataLakeServiceClient** 實例。  若要取得這些值，請參閱 [從 Azure AD 取得權杖，以授權用戶端應用程式的要求](../common/storage-auth-aad-app.md)。

:::code language="java" source="~/azure-storage-snippets/blobs/howto/Java/Java-v12/src/main/java/com/datalake/manage/Authorize_DataLake.java" id="Snippet_AuthorizeWithAzureAD":::

> [!NOTE]
> 如需更多範例，請參閱 [適用于 JAVA 的 Azure 身分識別用戶端程式庫](https://github.com/Azure/azure-sdk-for-java/tree/master/sdk/identity/azure-identity) 檔。


## <a name="create-a-container"></a>建立容器

容器可作為檔案的檔案系統。 您可以藉由呼叫 **DataLakeServiceClient. createFileSystem** 方法來建立一個。

這個範例會建立名為的容器 `my-file-system` 。 

:::code language="java" source="~/azure-storage-snippets/blobs/howto/Java/Java-v12/src/main/java/com/datalake/manage/CRUD_DataLake.java" id="Snippet_CreateFileSystem":::

## <a name="create-a-directory"></a>建立目錄

藉由呼叫 **DataLakeFileSystemClient. createDirectory** 方法來建立目錄參考。

此範例會將名為 `my-directory` 的目錄新增至容器，然後新增名為的子目錄 `my-subdirectory` 。 

:::code language="java" source="~/azure-storage-snippets/blobs/howto/Java/Java-v12/src/main/java/com/datalake/manage/CRUD_DataLake.java" id="Snippet_CreateDirectory":::

## <a name="rename-or-move-a-directory"></a>重新命名或移動目錄

藉由呼叫 DataLakeDirectoryClient 來重新命名或移動目錄 **。重新命名** 方法。 傳遞所需目錄的路徑給參數。 

此範例會將子目錄重新命名為名稱 `my-subdirectory-renamed` 。

:::code language="java" source="~/azure-storage-snippets/blobs/howto/Java/Java-v12/src/main/java/com/datalake/manage/CRUD_DataLake.java" id="Snippet_RenameDirectory":::

此範例會將名為的目錄移 `my-subdirectory-renamed` 至名為之目錄的子目錄 `my-directory-2` 。 

:::code language="java" source="~/azure-storage-snippets/blobs/howto/Java/Java-v12/src/main/java/com/datalake/manage/CRUD_DataLake.java" id="Snippet_MoveDirectory":::

## <a name="delete-a-directory"></a>刪除目錄

藉由呼叫 **DataLakeDirectoryClient. deleteWithResponse** 方法來刪除目錄。

此範例刪除名為 `my-directory` 的目錄。   

:::code language="java" source="~/azure-storage-snippets/blobs/howto/Java/Java-v12/src/main/java/com/datalake/manage/CRUD_DataLake.java" id="Snippet_DeleteDirectory":::

## <a name="upload-a-file-to-a-directory"></a>將檔案上傳至目錄

首先，藉由建立 **DataLakeFileClient** 類別的實例，在目標目錄中建立檔案參考。 藉由呼叫 **DataLakeFileClient. append** 方法來上傳檔案。 請務必藉由呼叫 **DataLakeFileClient. FlushAsync** 方法來完成上傳。

此範例會將文字檔上傳至名為的目錄 `my-directory` 。

:::code language="java" source="~/azure-storage-snippets/blobs/howto/Java/Java-v12/src/main/java/com/datalake/manage/CRUD_DataLake.java" id="Snippet_UploadFile":::

> [!TIP]
> 如果您的檔案大小很大，您的程式碼就必須對 **DataLakeFileClient 附加** 方法進行多次呼叫。 請考慮改為使用 **DataLakeFileClient. uploadFromFile** 方法。 如此一來，您就可以在單一呼叫中上傳整個檔案。 
>
> 如需範例，請參閱下一節。

## <a name="upload-a-large-file-to-a-directory"></a>將大型檔案上傳至目錄

使用 **DataLakeFileClient. uploadFromFile** 方法上傳大型檔案，而不需要對 **DataLakeFileClient. append** 方法進行多次呼叫。

:::code language="java" source="~/azure-storage-snippets/blobs/howto/Java/Java-v12/src/main/java/com/datalake/manage/CRUD_DataLake.java" id="Snippet_UploadFileBulk":::

## <a name="download-from-a-directory"></a>從目錄下載

首先，建立代表您要下載之檔案的 **DataLakeFileClient** 實例。 使用 **DataLakeFileClient** 讀取方法來讀取檔案。 使用任何 .NET 檔案處理 API，將資料流程中的位元組儲存至檔案。 

:::code language="java" source="~/azure-storage-snippets/blobs/howto/Java/Java-v12/src/main/java/com/datalake/manage/CRUD_DataLake.java" id="Snippet_DownloadFile":::

## <a name="list-directory-contents"></a>列出目錄內容

此範例會列印名為的目錄中每個檔案的名稱 `my-directory` 。

:::code language="java" source="~/azure-storage-snippets/blobs/howto/Java/Java-v12/src/main/java/com/datalake/manage/CRUD_DataLake.java" id="Snippet_ListFilesInDirectory":::

## <a name="manage-access-control-lists-acls"></a> (Acl) 管理存取控制清單

您可以取得、設定及更新目錄和檔案的存取權限。

> [!NOTE]
> 如果您使用 Azure Active Directory (Azure AD) 來授與存取權，請確定已將 [儲存體 Blob 資料擁有者角色](../../role-based-access-control/built-in-roles.md#storage-blob-data-owner)指派給您的安全性主體。 若要深入了解如何套用 ACL 權限以及變更權限的效果，請參閱 [Azure Data Lake Storage Gen2 中的存取控制](./data-lake-storage-access-control.md)。

### <a name="manage-a-directory-acl"></a>管理目錄 ACL

此範例會取得並設定名為之目錄的 ACL `my-directory` 。 此範例會提供擁有使用者的讀取、寫入和執行許可權，僅提供擁有群組的讀取和執行許可權，並提供所有其他的讀取權限。

> [!NOTE]
> 如果您的應用程式使用 Azure Active Directory (Azure AD) 來授與存取權，請確定您的應用程式用來授與存取權的安全性主體已獲指派 [儲存體 Blob 資料擁有者角色](../../role-based-access-control/built-in-roles.md#storage-blob-data-owner)。 若要深入了解如何套用 ACL 權限以及變更權限的效果，請參閱 [Azure Data Lake Storage Gen2 中的存取控制](./data-lake-storage-access-control.md)。

:::code language="java" source="~/azure-storage-snippets/blobs/howto/Java/Java-v12/src/main/java/com/datalake/manage/ACL_DataLake.java" id="Snippet_ManageDirectoryACLs":::

您也可以取得及設定容器根目錄的 ACL。 若要取得根目錄，請將空字串 (`""`) 傳遞至 **DataLakeFileSystemClient. getDirectoryClient** 方法。

### <a name="manage-a-file-acl"></a>管理檔案 ACL

這個範例會取得名為的檔案 ACL，然後設定其 ACL `upload-file.txt` 。 此範例會提供擁有使用者的讀取、寫入和執行許可權，僅提供擁有群組的讀取和執行許可權，並提供所有其他的讀取權限。

> [!NOTE]
> 如果您的應用程式使用 Azure Active Directory (Azure AD) 來授與存取權，請確定您的應用程式用來授與存取權的安全性主體已獲指派 [儲存體 Blob 資料擁有者角色](../../role-based-access-control/built-in-roles.md#storage-blob-data-owner)。 若要深入了解如何套用 ACL 權限以及變更權限的效果，請參閱 [Azure Data Lake Storage Gen2 中的存取控制](./data-lake-storage-access-control.md)。

:::code language="java" source="~/azure-storage-snippets/blobs/howto/Java/Java-v12/src/main/java/com/datalake/manage/ACL_DataLake.java" id="Snippet_ManageFileACLs":::

### <a name="set-an-acl-recursively"></a>以遞迴方式設定 ACL

您可以在父目錄的現有子專案上以遞迴方式新增、更新和移除 Acl，而不需要為每個子專案個別進行這些變更。 如需詳細資訊，請參閱 [設定 (acl 的存取控制清單) 以遞迴方式進行 Azure Data Lake Storage Gen2](recursive-access-control-lists.md)。

## <a name="see-also"></a>另請參閱

* [API 參考文件](/java/api/overview/azure/storage-file-datalake-readme)
* [套件 (Maven)](https://search.maven.org/artifact/com.azure/azure-storage-file-datalake) \(英文\)
* [範例](https://github.com/Azure/azure-sdk-for-java/tree/master/sdk/storage/azure-storage-file-datalake)
* [Gen1 至 Gen2 對應](https://github.com/Azure/azure-sdk-for-java/tree/master/sdk/storage/azure-storage-file-datalake/GEN1_GEN2_MAPPING.md)
* [已知問題](data-lake-storage-known-issues.md#api-scope-data-lake-client-library)
* [提供意見反應](https://github.com/Azure/azure-sdk-for-java/issues)