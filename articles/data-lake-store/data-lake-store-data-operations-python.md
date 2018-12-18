---
title: Python：Azure Data Lake Storage Gen1 的檔案系統作業 | Microsoft Docs
description: 了解如何利用 Python SDK 來使用 Data Lake Storage Gen1 檔案系統。
services: data-lake-store
documentationcenter: ''
author: nitinme
manager: jhubbard
editor: cgronlun
ms.service: data-lake-store
ms.devlang: na
ms.topic: conceptual
ms.date: 05/29/2018
ms.author: nitinme
ms.openlocfilehash: 33abaf7488579a501dc7e2d0b63645726b86c28b
ms.sourcegitcommit: 794bfae2ae34263772d1f214a5a62ac29dcec3d2
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/11/2018
ms.locfileid: "44390717"
---
# <a name="filesystem-operations-on-azure-data-lake-storage-gen1-using-python"></a>使用 Python 在 Azure Data Lake Storage Gen1 上進行檔案系統作業
> [!div class="op_single_selector"]
> * [.NET SDK](data-lake-store-data-operations-net-sdk.md)
> * [Java SDK](data-lake-store-get-started-java-sdk.md)
> * [REST API](data-lake-store-data-operations-rest-api.md)
> * [Python](data-lake-store-data-operations-python.md)
>
> 

在本文中，您會了解如何使用 Python SDK 在 Azure Data Lake Storage Gen1 上執行檔案系統作業。 如需有關如何使用 Python 在 Data Lake Storage Gen1 上執行帳戶管理作業的指示，請參閱[使用 Python 在 Data Lake Storage Gen1 上進行帳戶管理作業](data-lake-store-get-started-python.md)。

## <a name="prerequisites"></a>必要條件

* **Python**。 您可以從[這裡](https://www.python.org/downloads/)下載 Python。 本文使用 Python 3.6.2。

* **Azure 訂用帳戶**。 請參閱[取得 Azure 免費試用](https://azure.microsoft.com/pricing/free-trial/)。

* **Azure Data Lake Storage Gen1 帳戶**。 請遵循[使用 Azure 入口網站開始使用 Azure Data Lake Storage Gen1](data-lake-store-get-started-portal.md) 的指示。

## <a name="install-the-modules"></a>安裝模組

若要透過 Python 使用 Data Lake Storage Gen1，您需要安裝三個模組。

* `azure-mgmt-resource` 模組，這包括適用於 Active Directory 等等的 Azure 模組。
* `azure-mgmt-datalake-store` 模組，這包括 Azure Data Lake Storage Gen1 帳戶管理作業。 如需關於此模組的詳細資訊，請參閱 [azure-mgmt-datalake-store 模組參考](https://docs.microsoft.com/python/api/azure.mgmt.datalake.store?view=azure-python)。
* `azure-datalake-store` 模組，含有 Azure Data Lake Storage Gen1 檔案系統作業。 如需此模組的詳細資訊，請參閱 [azure-datalake-store 檔案系統模組參考](http://azure-datalake-store.readthedocs.io/en/latest/)。

使用下列命令來安裝新模組。

```
pip install azure-mgmt-resource
pip install azure-mgmt-datalake-store
pip install azure-datalake-store
```

## <a name="create-a-new-python-application"></a>建立新的 Python 應用程式

1. 在您選定的整合式開發環境 (IDE) 中建立新的 Python 應用程式，例如 **mysample.py**。

2. 加入以下文字行以匯入必要模組

    ```
    ## Use this only for Azure AD service-to-service authentication
    from azure.common.credentials import ServicePrincipalCredentials

    ## Use this only for Azure AD end-user authentication
    from azure.common.credentials import UserPassCredentials

    ## Use this only for Azure AD multi-factor authentication
    from msrestazure.azure_active_directory import AADTokenCredentials

    ## Required for Azure Data Lake Storage Gen1 account management
    from azure.mgmt.datalake.store import DataLakeStoreAccountManagementClient
    from azure.mgmt.datalake.store.models import DataLakeStoreAccount

    ## Required for Azure Data Lake Storage Gen1 filesystem management
    from azure.datalake.store import core, lib, multithread

    # Common Azure imports
    from azure.mgmt.resource.resources import ResourceManagementClient
    from azure.mgmt.resource.resources.models import ResourceGroup

    ## Use these as needed for your application
    import logging, getpass, pprint, uuid, time
    ```

3. 將變更儲存至 mysample.py。

## <a name="authentication"></a>驗證

在本節中，我們會討論向 Azure AD 進行驗證的各種方式。 可用的選項如下︰

* 如需了解應用程式的終端使用者驗證，請參閱[使用 Python 向 Data Lake Storage Gen1 進行終端使用者驗證](data-lake-store-end-user-authenticate-python.md)。
* 如需了解應用程式的服務對服務驗證，請參閱[使用 Python 向 Data Lake Storage Gen1 進行服務對服務驗證](data-lake-store-service-to-service-authenticate-python.md)。

## <a name="create-filesystem-client"></a>建立檔案系統用戶端

以下程式碼片段會先建立 Data Lake Storage Gen1 帳戶用戶端。 它會使用用戶端物件來建立 Data Lake Storage Gen1 帳戶。 最後，程式碼片段會建立檔案系統用戶端物件。

    ## Declare variables
    subscriptionId = 'FILL-IN-HERE'
    adlsAccountName = 'FILL-IN-HERE'

    ## Create a filesystem client object
    adlsFileSystemClient = core.AzureDLFileSystem(adlCreds, store_name=adlsAccountName)

## <a name="create-a-directory"></a>建立目錄

    ## Create a directory
    adlsFileSystemClient.mkdir('/mysampledirectory')

## <a name="upload-a-file"></a>上傳檔案


    ## Upload a file
    multithread.ADLUploader(adlsFileSystemClient, lpath='C:\\data\\mysamplefile.txt', rpath='/mysampledirectory/mysamplefile.txt', nthreads=64, overwrite=True, buffersize=4194304, blocksize=4194304)


## <a name="download-a-file"></a>下載檔案

    ## Download a file
    multithread.ADLDownloader(adlsFileSystemClient, lpath='C:\\data\\mysamplefile.txt.out', rpath='/mysampledirectory/mysamplefile.txt', nthreads=64, overwrite=True, buffersize=4194304, blocksize=4194304)

## <a name="delete-a-directory"></a>刪除目錄

    ## Delete a directory
    adlsFileSystemClient.rm('/mysampledirectory', recursive=True)

## <a name="next-steps"></a>後續步驟
* [使用 Python 在 Data Lake Storage Gen1 上進行帳戶管理作業](data-lake-store-get-started-python.md)。

## <a name="see-also"></a>另請參閱

* [Azure Data Lake Storage Gen1 Python (檔案系統) 參考](http://azure-datalake-store.readthedocs.io/en/latest)
* [與 Azure Data Lake Storage Gen1 相容的開放原始碼巨量資料應用程式](data-lake-store-compatible-oss-other-applications.md)
