---
title: .NET SDK：Azure Data Lake Storage Gen1 的帳戶管理作業 | Microsoft Docs
description: 使用 Azure Data Lake Storage Gen1 的 .NET SDK 在 Data Lake Storage Gen1 中執行帳戶管理作業
services: data-lake-store
documentationcenter: ''
author: nitinme
manager: jhubbard
editor: cgronlun
ms.assetid: ea57d5a9-2929-4473-9d30-08227912aba7
ms.service: data-lake-store
ms.devlang: na
ms.topic: conceptual
ms.date: 05/29/2018
ms.author: nitinme
ms.openlocfilehash: 2ed9f534c0eb27601243428f8e4b9d95db5d16b0
ms.sourcegitcommit: f10653b10c2ad745f446b54a31664b7d9f9253fe
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/18/2018
ms.locfileid: "46123907"
---
# <a name="account-management-operations-on-azure-data-lake-storage-gen1-using-net-sdk"></a>使用 .NET SDK 對 Azure Data Lake Storage Gen1 進行帳戶管理作業。
> [!div class="op_single_selector"]
> * [.NET SDK](data-lake-store-get-started-net-sdk.md)
> * [REST API](data-lake-store-get-started-rest-api.md)
> * [Python](data-lake-store-get-started-python.md)
>
>

在本文中，您會了解如何使用 .NET SDK 對 Azure Data Lake Storage Gen1 執行帳戶管理作業。 帳戶管理作業包括建立 Data Lake Storage Gen1 帳戶、列出 Azure 訂用帳戶中的帳戶、刪除帳戶等。

如需有關如何使用 .NET SDK 對 Data Lake Storage Gen1 執行資料管理作業的指示，請參閱[使用 .NET SDK 對 Data Lake Storage Gen1 進行檔案系統作業](data-lake-store-data-operations-net-sdk.md)。

## <a name="prerequisites"></a>必要條件
* **Visual Studio 2013、2015 或 2017**。 以下指示使用 Visual Studio 2017。

* **Azure 訂用帳戶**。 請參閱[取得 Azure 免費試用](https://azure.microsoft.com/pricing/free-trial/)。

## <a name="create-a-net-application"></a>建立 .NET 應用程式
1. 開啟 Visual Studio，建立主控台應用程式。
2. 從 [檔案] 功能表中，按一下 [新增]，再按 [專案]。
3. 在 [ **新增專案**] 中，輸入或選取下列值：

   | 屬性 | 值 |
   | --- | --- |
   | 類別 |範本/Visual C#/Windows |
   | 範本 |主控台應用程式 |
   | 名稱 |CreateADLApplication |
4. 按一下 [確定]  以建立專案。
5. 將 NuGet 套件新增至您的專案。

   1. 在方案總管中以滑鼠右鍵按一下專案名稱，然後按一下 [ **管理 NuGet 封裝**]。
   2. 在 [NuGet 套件管理員] 索引標籤中，確定 [套件來源] 設為 [nuget.org]，且已選取 [包含發行前版本] 核取方塊。
   3. 搜尋並安裝下列 NuGet 封裝：

      * `Microsoft.Azure.Management.DataLake.Store` - 本教學課程使用 v2.1.3-preview。
      * `Microsoft.Rest.ClientRuntime.Azure.Authentication` - 本教學課程使用 v2.2.12。

        ![新增 NuGet 來源](./media/data-lake-store-get-started-net-sdk/data-lake-store-install-nuget-package.png "建立新的 Azure Data Lake 帳戶")
   4. 關閉 [NuGet 套件管理員]。
6. 開啟 **Program.cs**，刪除現有的程式碼，然後納入下列陳述式以新增命名空間的參考。

        using System;
        using System.IO;
        using System.Linq;
        using System.Text;
        using System.Threading;
        using System.Collections.Generic;
        using System.Security.Cryptography.X509Certificates; // Required only if you are using an Azure AD application created with certificates
                
        using Microsoft.Rest;
        using Microsoft.Rest.Azure.Authentication;
        using Microsoft.Azure.Management.DataLake.Store;
        using Microsoft.Azure.Management.DataLake.Store.Models;
        using Microsoft.IdentityModel.Clients.ActiveDirectory;

7. 宣告變數，並提供預留位置的值。 此外，請確定電腦上有您提供的本機路徑和檔案名稱。

        namespace SdkSample
        {
            class Program
            {
                private static DataLakeStoreAccountManagementClient _adlsClient;
                
                private static string _adlsAccountName;
                private static string _resourceGroupName;
                private static string _location;
                private static string _subId;

                private static void Main(string[] args)
                {
                    _adlsAccountName = "<DATA-LAKE-STORAGE-GEN1-NAME>.azuredatalakestore.net"; 
                    _resourceGroupName = "<RESOURCE-GROUP-NAME>"; 
                    _location = "East US 2";
                    _subId = "<SUBSCRIPTION-ID>";                    
                }
            }
        }

在本文的其餘章節中，您可以了解如何使用可用的 .NET 方法來執行一些作業，例如驗證、檔案上載等。

## <a name="authentication"></a>驗證

* 如需讓應用程式進行使用者驗證，請參閱[使用 .NET SDK 向 Data Lake Storage Gen1 進行使用者驗證](data-lake-store-end-user-authenticate-net-sdk.md)。
* 如需讓應用程式進行服務對服務驗證，請參閱[使用 .NET SDK 向 Data Lake Storage Gen1 進行服務對服務驗證](data-lake-store-service-to-service-authenticate-net-sdk.md)。

## <a name="create-client-object"></a>建立用戶端物件
下列程式碼片段會建立 Data Lake Storage Gen1 帳戶用戶端物件，此物件用於對服務發出帳戶管理要求，例如建立帳戶、刪除帳戶等。

    // Create client objects and set the subscription ID
    _adlsClient = new DataLakeStoreAccountManagementClient(armCreds) { SubscriptionId = _subId };
    
## <a name="create-a-data-lake-storage-gen1-account"></a>建立 Data Lake Storage Gen1 帳戶
下列程式碼片段會在您建立 Data Lake Storage Gen1 帳戶用戶端物件時所提供的 Azure 訂用帳戶中，建立 Data Lake Storage Gen1 帳戶。

    // Create Data Lake Storage Gen1 account
    var adlsParameters = new DataLakeStoreAccount(location: _location);
    _adlsClient.Account.Create(_resourceGroupName, _adlsAccountName, adlsParameters);

## <a name="list-all-data-lake-storage-gen1-accounts-within-a-subscription"></a>列出訂用帳戶內的所有 Data Lake Storage Gen1 帳戶
將下列方法新增至您的類別定義。 下列程式碼片段會列出指定 Azure 訂用帳戶內的所有 Data Lake Storage Gen1 帳戶。

    // List all Data Lake Storage Gen1 accounts within the subscription
    public static List<DataLakeStoreAccountBasic> ListAdlStoreAccounts()
    {
        var response = _adlsClient.Account.List(_adlsAccountName);
        var accounts = new List<DataLakeStoreAccountBasic>(response);

        while (response.NextPageLink != null)
        {
            response = _adlsClient.Account.ListNext(response.NextPageLink);
            accounts.AddRange(response);
        }

        return accounts;
    }

## <a name="delete-a-data-lake-storage-gen1-account"></a>刪除 Data Lake Storage Gen1 帳戶
下列程式碼片段會刪除您之前建立的 Data Lake Storage Gen1 帳戶。

    // Delete Data Lake Storage Gen1 account
    _adlsClient.Account.Delete(_resourceGroupName, _adlsAccountName);

## <a name="see-also"></a>另請參閱
* [使用 .NET SDK 對 Data Lake Storage Gen1 進行檔案系統作業](data-lake-store-data-operations-net-sdk.md)
* [Data Lake Storage Gen1 .NET SDK 參考](https://docs.microsoft.com/dotnet/api/overview/azure/data-lake-store?view=azure-dotnet)

## <a name="next-steps"></a>後續步驟
* [保護 Data Lake Storage Gen1 中的資料](data-lake-store-secure-data.md)
