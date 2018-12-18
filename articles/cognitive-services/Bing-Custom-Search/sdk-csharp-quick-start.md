---
title: 快速入門：自訂搜尋 SDK (C#)
titleSuffix: Azure Cognitive Services
description: 設定自訂搜尋 SDK C# 主控台應用程式。
services: cognitive-services
author: swhite-msft
manager: cgronlun
ms.service: cognitive-services
ms.component: bing-custom-search
ms.topic: quickstart
ms.date: 09/06/2018
ms.author: scottwhi
ms.openlocfilehash: 6dc10bc2dedfe99573b5ad646461e66827c6df9e
ms.sourcegitcommit: 74941e0d60dbfd5ab44395e1867b2171c4944dbe
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/15/2018
ms.locfileid: "49320164"
---
# <a name="quickstart-using-the-bing-custom-search-sdk-with-c"></a>快速入門：使用以 C# 撰寫的 Bing 自訂搜尋 SDK

Bing 自訂搜尋 SDK 會提供比 Bing 自訂搜尋 REST API 更簡單的程式設計模型。 本節將引導您完成使用 C# SDK 進行第一個自訂搜尋呼叫的作業。

## <a name="prerequisites"></a>必要條件

若要完成本快速入門，您需要：

- 已經可以使用的自訂搜尋執行個體。 請參閱[建立您的第一個 Bing 自訂搜尋執行個體](quick-start.md)。  
  
- 訂用帳戶金鑰。 您可以在啟用[免費試用版](https://azure.microsoft.com/try/cognitive-services/?api=bing-custom-search)時取得訂用帳戶金鑰，或者可以從 Azure 儀表板使用付費訂用帳戶金鑰 (請參閱[認知服務 API 帳戶](https://docs.microsoft.com/azure/cognitive-services/cognitive-services-apis-create-account))。  
  
- 已安裝 Visual Studio 2017。 如果您還沒有此產品，則可以先下載**免費**的 [Visual Studio 2017 Community Edition](https://www.visualstudio.com/downloads/)。  
  
- 已安裝 [NuGet 自訂搜尋](https://www.nuget.org/packages/Microsoft.Azure.CognitiveServices.Search.CustomSearch/1.2.0)套件。 在 Visual Studio 的 [方案總管] 中，以滑鼠右鍵按一下專案，然後從功能表選取 `Manage NuGet Packages`。 安裝 `Microsoft.Azure.CognitiveServices.Search.CustomSearch` 套件。

安裝 NuGet 自訂搜尋套件也會一併安裝下列組件：

* Microsoft.Rest.ClientRuntime
* Microsoft.Rest.ClientRuntime.Azure
* Newtonsoft.Json



## <a name="run-the-code"></a>執行程式碼

> [!TIP]
> 從 [GitHub](https://github.com/Azure-Samples/cognitive-services-dotnet-sdk-samples/tree/master/BingSearchv7/BingCustomWebSearch) 取得最新程式碼作為 Visual Studio 解決方案。

若要執行此範例，請遵循下列步驟：

1. 開啟 Visual Studio 2017。
  
2. 按一下 [檔案] 功能表，依序按一下 [新增]、[專案] 及 [Visual C# 主控台應用程式] 範本。
  
3. 將解決方案命名為 CustomSearchSolution，並瀏覽至要將它放入其中的資料夾。
  
4. 按一下 [確定] 以建立解決方案。  
  
4. 在 [方案總管] 中，以滑鼠右鍵按一下您的專案 (其具有與解決方案相同的名稱)，然後選取 `Manage NuGet Packages`。 在 NuGet 套件管理員中按一下 [瀏覽]。 在搜尋方塊中輸入 Microsoft.Azure.CognitiveServices.Search.CustomSearch，然後按 Enter。 選取該套件並按一下 [安裝]。  
  
4. 將下列程式碼複製到 Program.cs 檔案中。 將 **YOUR-SUBSCRIPTION-KEY** 與 **YOUR-CUSTOM-CONFIG-ID** 取代為您的訂用帳戶金鑰與設定識別碼。  
  
    ```csharp
    using System;
    using System.Linq;
    using Microsoft.Azure.CognitiveServices.Search.CustomSearch;

    namespace CustomSrchSDK
    {
        class Program
        {
            static void Main(string[] args)
            {

                var client = new CustomSearchAPI(new ApiKeyServiceClientCredentials("YOUR-SUBSCRIPTION-KEY"));

                try
                {
                    // This will look up a single query (Xbox).
                    var webData = client.CustomInstance.SearchAsync(query: "Xbox", customConfig: Int32.Parse("YOUR-CUSTOM-CONFIG-ID")).Result;
                    Console.WriteLine("Searched for Query# \" Xbox \"");

                    //WebPages
                    if (webData?.WebPages?.Value?.Count > 0)
                    {
                        // find the first web page
                        var firstWebPagesResult = webData.WebPages.Value.FirstOrDefault();

                        if (firstWebPagesResult != null)
                        {
                            Console.WriteLine("Number of webpage results {0}", webData.WebPages.Value.Count);
                            Console.WriteLine("First web page name: {0} ", firstWebPagesResult.Name);
                            Console.WriteLine("First web page URL: {0} ", firstWebPagesResult.Url);
                        }
                        else
                        {
                            Console.WriteLine("Couldn't find web results!");
                        }
                    }
                    else
                    {
                        Console.WriteLine("Didn't see any Web data..");
                    }
                }
                catch (Exception ex)
                {
                    Console.WriteLine("Encountered exception. " + ex.Message);
                }

                Console.WriteLine("Press any key to exit...");
                Console.ReadKey();
            }

        }
    }
    ```  
  
5. 建置並執行 (偵錯) 解決方案。 




## <a name="breaking-it-down"></a>細分內容

安裝 NuGet 自訂搜尋套件之後，您需要為它新增 using 指示詞。

```csharp
using Microsoft.Azure.CognitiveServices.Search.CustomSearch;
```

接下來，將自訂搜尋用戶端具現化，您可以使用它來發出搜尋要求。 請務必將 `YOUR-SUBSCRIPTION-KEY` 取代為您的金鑰。

```csharp
var client = new CustomSearchAPI(new ApiKeyServiceClientCredentials("YOUR-CUSTOM-SEARCH-KEY"));
```

現在您可以使用用戶端來傳送搜尋要求。 請務必將 `YOUR-CUSTOM-CONFIG-ID` 取代為執行個體的設定識別碼 (您可以在[自訂搜尋入口網站](https://www.customsearch.ai/)中找到該識別碼)。 此範例會搜尋 Xbox。 請視需要更新。

```csharp
var webData = client.CustomInstance.SearchAsync(query: "Xbox", customConfig: Int32.Parse("YOUR-CUSTOM-CONFIG-ID")).Result;
```

`SearchAsync` 方法會傳回 `WebData` 物件。 請使用 `WebData` 來逐一查看找到的任何 `WebPages`。 此程式碼會找出第一個網頁結果，並列印網頁的 `Name` 和 `URL`。

```csharp
//WebPages
if (webData?.WebPages?.Value?.Count > 0)
{
    // find the first web page
    var firstWebPagesResult = webData.WebPages.Value.FirstOrDefault();

    if (firstWebPagesResult != null)
    {
        Console.WriteLine("Webpage Results#{0}", webData.WebPages.Value.Count);
        Console.WriteLine("First web page name: {0} ", firstWebPagesResult.Name);
        Console.WriteLine("First web page URL: {0} ", firstWebPagesResult.Url);
    }
    else
    {
        Console.WriteLine("Couldn't find web results!");
    }
}
else
{
    Console.WriteLine("Didn't see any Web data..");
}

```


## <a name="next-steps"></a>後續步驟

請參閱 SDK 範例。 每個範例都包括讀我檔案，內含必要條件和安裝/執行範例的詳細資料。

* 開始使用 [.NET 範例](https://github.com/Azure-Samples/cognitive-services-dotnet-sdk-samples/tree/master/BingSearchv7) 
    * [Nuget 套件](https://www.nuget.org/packages/Microsoft.Azure.CognitiveServices.Search.CustomSearch/1.2.0)
    * 另請參閱 [.NET 程式庫](https://github.com/Azure/azure-sdk-for-net/tree/psSdkJson6/src/SDKs/CognitiveServices/dataPlane/Search/BingCustomSearch)，以了解定義和相依性。
* 開始使用 [NodeJS 範例](https://github.com/Azure-Samples/cognitive-services-node-sdk-samples) 
    * 另請參閱 [NodeJS 程式庫](https://github.com/Azure/azure-sdk-for-node/tree/master/lib/services/customSearch)，以了解定義和相依性。
* 開始使用 [Java 範例](https://github.com/Azure-Samples/cognitive-services-java-sdk-samples) 
    * 另請參閱 [Java 程式庫](https://github.com/Azure/azure-sdk-for-java/tree/master/cognitiveservices/azure-customsearch)，以了解定義和相依性。
* 開始使用 [Python 範例](https://github.com/Azure-Samples/cognitive-services-python-sdk-samples) 
    * 另請參閱 [Python 程式庫](https://github.com/Azure/azure-sdk-for-python/tree/master/azure-cognitiveservices-search-customsearch) ，以了解定義和相依性。

