---
title: 服務對服務驗證︰使用 Python 透過 Azure Active Directory 向 Azure Data Lake Storage Gen1 進行驗證 | Microsoft Docs
description: 了解如何使用 Python 透過 Azure Active Directory 向 Azure Data Lake Storage Gen1 完成服務對服務驗證
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
ms.openlocfilehash: a8e2cb090b2ee02301c54697db9edadeaebd9cd3
ms.sourcegitcommit: cf606b01726df2c9c1789d851de326c873f4209a
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/19/2018
ms.locfileid: "46295406"
---
# <a name="service-to-service-authentication-with-azure-data-lake-storage-gen1-using-python"></a>使用 Python 向 Azure Data Lake Storage Gen1 進行服務對服務驗證
> [!div class="op_single_selector"]
> * [使用 Java](data-lake-store-service-to-service-authenticate-java.md)
> * [使用 .NET SDK](data-lake-store-service-to-service-authenticate-net-sdk.md)
> * [使用 Python](data-lake-store-service-to-service-authenticate-python.md)
> * [使用 REST API](data-lake-store-service-to-service-authenticate-rest-api.md)
> 
>  

在本文中，您會了解如何使用 Python 向 Azure Data Lake Storage Gen1 進行服務對服務驗證。 如須使用 Python 向 Data Lake Storage Gen1 進行使用者驗證，請參閱[使用 Python 向 Data Lake Storage Gen1 進行使用者驗證](data-lake-store-end-user-authenticate-python.md)。


## <a name="prerequisites"></a>必要條件

* **Python**。 您可以從[這裡](https://www.python.org/downloads/)下載 Python。 本文使用 Python 3.6.2。

* **Azure 訂用帳戶**。 請參閱[取得 Azure 免費試用](https://azure.microsoft.com/pricing/free-trial/)。

* **建立 Azure Active Directory "Web" 應用程式**。 您必須先完成[使用 Azure Active Directory 向 Data Lake Storage Gen1 進行服務對服務驗證](data-lake-store-service-to-service-authenticate-using-active-directory.md)中的步驟。

## <a name="install-the-modules"></a>安裝模組

若要透過 Python 使用 Data Lake Storage Gen1，您需要安裝三個模組。

* `azure-mgmt-resource` 模組，這包括適用於 Active Directory 等等的 Azure 模組。
* `azure-mgmt-datalake-store` 模組包括 Data Lake Storage Gen1 帳戶管理作業。 如需關於此模組的詳細資訊，請參閱 [Azure Data Lake Storage Gen1 Management module reference (Azure Data Lake Storage Gen1 管理模組參考)](https://docs.microsoft.com/python/api/azure.mgmt.datalake.store?view=azure-python)。
* `azure-datalake-store` 模組包括 Data Lake Storage Gen1 檔案系統作業。 如需關於此模組的詳細資訊，請參閱 [azure-datalake-store Filesystem module reference (azure-datalake-store 檔案系統模組參考)](http://azure-datalake-store.readthedocs.io/en/latest/)。

使用下列命令來安裝新模組。

```
pip install azure-mgmt-resource
pip install azure-mgmt-datalake-store
pip install azure-datalake-store
```

## <a name="create-a-new-python-application"></a>建立新的 Python 應用程式

1. 在您選定的整合式開發環境 (IDE) 中建立新的 Python 應用程式，例如 **mysample.py**。

2. 新增以下程式碼片段以匯入必要模組

    ```
    ## Use this for Azure AD authentication
    from msrestazure.azure_active_directory import AADTokenCredentials

    ## Required for Data Lake Storage Gen1 account management
    from azure.mgmt.datalake.store import DataLakeStoreAccountManagementClient
    from azure.mgmt.datalake.store.models import DataLakeStoreAccount

    ## Required for Data Lake Storage Gen1 filesystem management
    from azure.datalake.store import core, lib, multithread

    # Common Azure imports
    import adal
    from azure.mgmt.resource.resources import ResourceManagementClient
    from azure.mgmt.resource.resources.models import ResourceGroup

    ## Use these as needed for your application
    import logging, getpass, pprint, uuid, time
    ```

3. 將變更儲存至 mysample.py。

## <a name="service-to-service-authentication-with-client-secret-for-account-management"></a>適用於帳戶管理的用戶端密碼服務對服務驗證

使用此程式碼片段向 Azure AD 進行驗證，以便在 Data Lake Storage Gen1 上進行帳戶管理作業，例如建立 Data Lake Storage Gen1 帳戶、刪除 Data Lake Storage Gen1 帳戶等。下列程式碼片段可供使用現有 Azure AD「Web 應用程式」應用程式的應用程式/服務主體的用戶端密碼，以非互動方式驗證您的應用程式。

    authority_host_uri = 'https://login.microsoftonline.com'
    tenant = '<TENANT>'
    authority_uri = authority_host_uri + '/' + tenant
    RESOURCE = 'https://management.core.windows.net/'
    client_id = '<CLIENT_ID>'
    client_secret = '<CLIENT_SECRET>'
    
    context = adal.AuthenticationContext(authority_uri, api_version=None)
    mgmt_token = context.acquire_token_with_client_credentials(RESOURCE, client_id, client_secret)
    armCreds = AADTokenCredentials(mgmt_token, client_id, resource=RESOURCE)

## <a name="service-to-service-authentication-with-client-secret-for-filesystem-operations"></a>適用於檔案系統作業的用戶端密碼服務對服務驗證

使用下列程式碼片段向 Azure AD 進行驗證，以便在 Data Lake Storage Gen1 上進行檔案系統作業，例如建立資料夾、上載檔案等。下列程式碼片段可供使用應用程式/服務主體的用戶端密碼，以非互動方式驗證您的應用程式。 請將此方法用於現有的 Azure AD「Web 應用程式」應用程式。

    tenant = '<TENANT>'
    RESOURCE = 'https://datalake.azure.net/'
    client_id = '<CLIENT_ID>'
    client_secret = '<CLIENT_SECRET>'
    
    adlCreds = lib.auth(tenant_id = tenant,
                    client_secret = client_secret,
                    client_id = client_id,
                    resource = RESOURCE)

<!-- ## Service-to-service authentication with certificate for account management

Use this snippet to authenticate with Azure AD for account management operations on Data Lake Storage Gen1 such as create a Data Lake Storage Gen1 account, delete a Data Lake Storage Gen1 account, etc. The following snippet can be used to authenticate your application non-interactively, using the certificate of an existing Azure AD "Web App" application. For instructions on how to create an Azure AD application, see [Create service principal with certificates](../azure-resource-manager/resource-group-authenticate-service-principal.md#create-service-principal-with-self-signed-certificate).

    authority_host_uri = 'https://login.microsoftonline.com'
    tenant = '<TENANT>'
    authority_uri = authority_host_uri + '/' + tenant
    resource_uri = 'https://management.core.windows.net/'
    client_id = '<CLIENT_ID>'
    client_cert = '<CLIENT_CERT>'
    client_cert_thumbprint = '<CLIENT_CERT_THUMBPRINT>'

    context = adal.AuthenticationContext(authority_uri, api_version=None)
    mgmt_token = context.acquire_token_with_client_certificate(resource_uri, client_id, client_cert, client_cert_thumbprint)
    credentials = AADTokenCredentials(mgmt_token, client_id) -->

## <a name="next-steps"></a>後續步驟
在本文中，您已了解如何使用 Python 向 Data Lake Storage Gen1 進行服務對服務驗證。 您現在可以看看下列文章，了解如何配合使用 Python 與 Data Lake Storage Gen1。

* [使用 Python 對 Data Lake Storage Gen1 進行帳戶管理作業](data-lake-store-get-started-python.md)
* [使用 Python 對 Data Lake Storage Gen1 進行資料作業](data-lake-store-data-operations-python.md)


