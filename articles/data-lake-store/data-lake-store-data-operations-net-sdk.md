---
title: .NET SDK：Azure Data Lake Storage Gen1 的檔案系統作業 | Microsoft Docs
description: 使用 Azure Data Lake Storage Gen1 的 .NET SDK 在 Data Lake Storage Gen1 上執行檔案系統作業，例如建立資料夾等。
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
ms.openlocfilehash: 71ddbc2363075b721bfbd418bd29e5154baba866
ms.sourcegitcommit: 794bfae2ae34263772d1f214a5a62ac29dcec3d2
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/11/2018
ms.locfileid: "44391482"
---
# <a name="filesystem-operations-on-azure-data-lake-storage-gen1-using-net-sdk"></a>使用 .NET SDK 在 Azure Data Lake Storage Gen1 上進行檔案系統作業
> [!div class="op_single_selector"]
> * [.NET SDK](data-lake-store-data-operations-net-sdk.md)
> * [Java SDK](data-lake-store-get-started-java-sdk.md)
> * [REST API](data-lake-store-data-operations-rest-api.md)
> * [Python](data-lake-store-data-operations-python.md)
>
>

在本文中，您會了解如何使用 .NET SDK 在 Data Lake Storage Gen1 上執行檔案系統作業。 檔案系統作業包括在 Data Lake Storage Gen1 帳戶中建立資料夾、上傳檔案、下載檔案等。

如需有關如何使用 .NET SDK 在 Data Lake Storage Gen1 上執行帳戶管理作業的指示，請參閱[使用.NET SDK 在 Data Lake Storage Gen1 上進行帳戶管理作業](data-lake-store-get-started-net-sdk.md)。

## <a name="prerequisites"></a>必要條件
* **Visual Studio 2013、2015 或 2017**。 以下指示使用 Visual Studio 2017。

* **Azure 訂用帳戶**。 請參閱[取得 Azure 免費試用](https://azure.microsoft.com/pricing/free-trial/)。

* **Azure Data Lake Storage Gen1 帳戶**。 如需有關如何建立帳戶的指示，請參閱[開始使用 Azure Data Lake Storage Gen1](data-lake-store-get-started-portal.md)

## <a name="create-a-net-application"></a>建立 .NET 應用程式
[GitHub](https://github.com/Azure-Samples/data-lake-store-adls-dot-net-get-started/tree/master/AdlsSDKGettingStarted) 上可用的程式碼範例會逐步引導您完成在存放區中建立檔案，串連檔案、下載檔案，以及刪除存放區中一些檔案的程序。 本節文章會逐步解說程式碼的主要部分。

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

      * `Microsoft.Azure.DataLake.Store` - 本教學課程使用 v1.0.0。
      * `Microsoft.Rest.ClientRuntime.Azure.Authentication` - 本教學課程使用 v2.3.1。
    
    關閉 [NuGet 套件管理員]。

6. 開啟 **Program.cs**，刪除現有的程式碼，然後納入下列陳述式以新增命名空間的參考。

        using System;
        using System.IO;using System.Threading;
        using System.Linq;
        using System.Text;
        using System.Collections.Generic;
        using System.Security.Cryptography.X509Certificates; // Required only if you are using an Azure AD application created with certificates
                
        using Microsoft.Rest;
        using Microsoft.Rest.Azure.Authentication;
        using Microsoft.Azure.DataLake.Store;
        using Microsoft.IdentityModel.Clients.ActiveDirectory;

7. 宣告變數如下，並提供預留位置的值。 此外，請確定電腦上有此處提供的本機路徑和檔案名稱。

        namespace SdkSample
        {
            class Program
            {
                private static string _adlsg1AccountName = "<DATA-LAKE-STORAGE-GEN1-NAME>.azuredatalakestore.net";        
            }
        }

在本文的其餘章節中，您可以了解如何使用可用的 .NET 方法來執行一些作業，例如驗證、檔案上載等。

## <a name="authentication"></a>驗證

* 如需讓應用程式進行使用者驗證，請參閱[使用 .NET SDK 向 Data Lake Storage Gen1 進行使用者驗證](data-lake-store-end-user-authenticate-net-sdk.md)。
* 如需讓應用程式進行服務對服務驗證，請參閱[使用 .NET SDK 向 Data Lake Storage Gen1 進行服務對服務驗證](data-lake-store-service-to-service-authenticate-net-sdk.md)。


## <a name="create-client-object"></a>建立用戶端物件
下列程式碼片段會建立 Data Lake Storage Gen1 檔案系統用戶端物件，以便對服務發出要求。

    // Create client objects
    AdlsClient client = AdlsClient.CreateClient(_adlsg1AccountName, adlCreds);

## <a name="create-a-file-and-directory"></a>建立檔案和目錄
將下列程式碼片段新增至應用程式。 這個程式碼片段會新增與檔案任何不存在的父目錄。

    // Create a file - automatically creates any parent directories that don't exist
    // The AdlsOutputStream preserves record boundaries - it does not break records while writing to the store
    using (var stream = client.CreateFile(fileName, IfExists.Overwrite))
    {
        byte[] textByteArray = Encoding.UTF8.GetBytes("This is test data to write.\r\n");
        stream.Write(textByteArray, 0, textByteArray.Length);

        textByteArray = Encoding.UTF8.GetBytes("This is the second line.\r\n");
        stream.Write(textByteArray, 0, textByteArray.Length);
    }

## <a name="append-to-a-file"></a>附加到檔案
下列程式碼片段會將資料附加至 Data Lake Storage Gen1 帳戶中的現有檔案。

    // Append to existing file
    using (var stream = client.GetAppendStream(fileName))
    {
        byte[] textByteArray = Encoding.UTF8.GetBytes("This is the added line.\r\n");
        stream.Write(textByteArray, 0, textByteArray.Length);
    }

## <a name="read-a-file"></a>讀取檔案
下列程式碼片段會讀取 Data Lake Storage Gen1 中的檔案內容。

    //Read file contents
    using (var readStream = new StreamReader(client.GetReadStream(fileName)))
    {
        string line;
        while ((line = readStream.ReadLine()) != null)
        {
            Console.WriteLine(line);
        }
    }

## <a name="get-file-properties"></a>取得檔案屬性
下列程式碼片段會傳回與檔案或目錄相關聯的屬性。

    // Get file properties
    var directoryEntry = client.GetDirectoryEntry(fileName);
    PrintDirectoryEntry(directoryEntry);

`PrintDirectoryEntry` 方法的定義可隨著 [Github 上](https://github.com/Azure-Samples/data-lake-store-adls-dot-net-get-started/tree/master/AdlsSDKGettingStarted)的範例一起提供。 

## <a name="rename-a-file"></a>重新命名檔案
下列程式碼片段會重新命名 Data Lake Storage Gen1 帳戶中的現有檔案。

    // Rename a file
    string destFilePath = "/Test/testRenameDest3.txt";
    client.Rename(fileName, destFilePath, true);

## <a name="enumerate-a-directory"></a>列舉目錄
下列程式碼片段會列舉 Data Lake Storage Gen1 帳戶中的目錄

    // Enumerate directory
    foreach (var entry in client.EnumerateDirectory("/Test"))
    {
        PrintDirectoryEntry(entry);
    }

`PrintDirectoryEntry` 方法的定義可隨著 [Github 上](https://github.com/Azure-Samples/data-lake-store-adls-dot-net-get-started/tree/master/AdlsSDKGettingStarted)的範例一起提供。

## <a name="delete-directories-recursively"></a>以遞迴方式刪除目錄
下列程式碼片段會以遞迴方式刪除目錄，以及其所有子目錄。

    // Delete a directory and all its subdirectories and files
    client.DeleteRecursive("/Test");

## <a name="samples"></a>範例
以下是幾個有關如何使用 Data Lake Storage Gen1 Filesystem SDK 的範例。
* [GitHub 上的基本範例](https://github.com/Azure-Samples/data-lake-store-adls-dot-net-get-started/tree/master/AdlsSDKGettingStarted)
* [Github 上的進階範例](https://github.com/Azure-Samples/data-lake-store-adls-dot-net-samples)

## <a name="see-also"></a>另請參閱
* [使用.NET SDK 在 Data Lake Storage Gen1 上進行帳戶管理作業](data-lake-store-get-started-net-sdk.md)
* [Data Lake Storage Gen1 .NET SDK 參考](https://docs.microsoft.com/dotnet/api/overview/azure/data-lake-store?view=azure-dotnet)

## <a name="next-steps"></a>後續步驟
* [保護 Data Lake Storage Gen1 中的資料](data-lake-store-secure-data.md)
