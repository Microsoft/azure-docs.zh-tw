---
title: Azure 搜尋服務的同義字 C# 教學課程 | Microsoft Docs
description: 在本教學課程中，會將同義字功能新增至 Azure 搜尋服務中的索引。
manager: cgronlun
author: HeidiSteen
services: search
ms.service: search
ms.topic: tutorial
ms.date: 07/10/2018
ms.author: heidist
ms.openlocfilehash: 8340c4dc2a855911073905a3aea93e19fc7b520d
ms.sourcegitcommit: df50934d52b0b227d7d796e2522f1fd7c6393478
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/12/2018
ms.locfileid: "38990556"
---
# <a name="tutorial-add-synonyms-for-azure-search-in-c"></a>Azure 搜尋服務的同義字 C# 教學課程

同義字可藉由比對在語意上視為等於輸入詞彙的詞彙來展開查詢。 例如，您可能希望 "car" 比對包含 "automobile" 或 "vehicle" 詞彙的文件。 

在 Azure 搜尋服務中，同義字會定義於*同義字對應*中，透過*對應規則*讓相等的詞彙產生關聯。 本教學課程涵蓋基本步驟，利用現有的索引新增及使用同義字。 您會了解如何：

> [!div class="checklist"]
> * 建立和張貼對應規則來啟用同義字 
> * 在查詢字串中參考同義字對應

您可以建立多個同義字對應、將它們張貼為可供任何索引使用的全服務資源，然後參照哪一個要在欄位層級使用。 查詢時，除了搜尋索引，Azure 搜尋服務會查閱同義字對應 (如果在查詢中使用的欄位上指定一個同義字對應)。

> [!NOTE]
> 最新的 API 和 SDK 版本 (api-version=2017-11-11、SDK 5.0.0 版) 支援同義字。 目前 Azure 入口網站並不支援此功能。 如果 Azure 入口網站的同義字支援對您很有用，請在 [UserVoice](https://feedback.azure.com/forums/263029-azure-search) 上提供您的意見反應

## <a name="prerequisites"></a>必要條件

教學課程包含下列需求︰

* [Visual Studio](https://www.visualstudio.com/downloads/)
* [Azure 搜尋服務](search-create-service-portal.md)
* [Microsoft.Azure.Search .NET 程式庫](https://aka.ms/search-sdk)
* [如何從 .NET 應用程式使用 Azure 搜尋服務](https://docs.microsoft.com/azure/search/search-howto-dotnet-sdk)

## <a name="overview"></a>概觀

之前與之後查詢會示範同義字的值。 在本教學課程中，會使用可執行查詢並傳回範例索引結果的範例應用程式。 範例應用程式會建立名為 "hotels" 並已填入兩份文件的小型索引。 此應用程式會使用未出現在索引中的詞彙和詞句來執行搜尋查詢，啟用同義字功能，然後再次發出相同的搜尋。 下列程式碼示範整體流程。

```csharp
  static void Main(string[] args)
  {
      SearchServiceClient serviceClient = CreateSearchServiceClient();

      Console.WriteLine("{0}", "Cleaning up resources...\n");
      CleanupResources(serviceClient);

      Console.WriteLine("{0}", "Creating index...\n");
      CreateHotelsIndex(serviceClient);

      ISearchIndexClient indexClient = serviceClient.Indexes.GetClient("hotels");

      Console.WriteLine("{0}", "Uploading documents...\n");
      UploadDocuments(indexClient);

      ISearchIndexClient indexClientForQueries = CreateSearchIndexClient();

      RunQueriesWithNonExistentTermsInIndex(indexClientForQueries);

      Console.WriteLine("{0}", "Adding synonyms...\n");
      UploadSynonyms(serviceClient);
      EnableSynonymsInHotelsIndex(serviceClient);
      Thread.Sleep(10000); // Wait for the changes to propagate

      RunQueriesWithNonExistentTermsInIndex(indexClientForQueries);

      Console.WriteLine("{0}", "Complete.  Press any key to end application...\n");

      Console.ReadKey();
  }
```
[如何從 .NET 應用程式使用 Azure 搜尋服務](https://docs.microsoft.com/azure/search/search-howto-dotnet-sdk)會說明用來建立及填入範例索引的步驟。

## <a name="before-queries"></a>「之前」查詢

在 `RunQueriesWithNonExistentTermsInIndex` 中，使用 "five star"、"internet" 和 "economy AND hotel" 發出搜尋查詢。
```csharp
Console.WriteLine("Search the entire index for the phrase \"five star\":\n");
results = indexClient.Documents.Search<Hotel>("\"five star\"", parameters);
WriteDocuments(results);

Console.WriteLine("Search the entire index for the term 'internet':\n");
results = indexClient.Documents.Search<Hotel>("internet", parameters);
WriteDocuments(results);

Console.WriteLine("Search the entire index for the terms 'economy' AND 'hotel':\n");
results = indexClient.Documents.Search<Hotel>("economy AND hotel", parameters);
WriteDocuments(results);
```
這兩份經過檢索的文件都不包含這些詞彙，所以我們會從第一個 `RunQueriesWithNonExistentTermsInIndex` 取得下列輸出。
~~~
Search the entire index for the phrase "five star":

no document matched

Search the entire index for the term 'internet':

no document matched

Search the entire index for the terms 'economy' AND 'hotel':

no document matched
~~~

## <a name="enable-synonyms"></a>啟用同義字

啟用同義字的程序包含兩步驟。 我們會先定義和上傳同義字規則，然後再設定要使用這些規則的欄位。 `UploadSynonyms` 和 `EnableSynonymsInHotelsIndex` 會簡要說明此程序。

1. 將同義字對應新增到您的搜尋服務。 在 `UploadSynonyms` 中，我們會在同義字對應'desc-synonymmap' 中定義四個規則並上傳至服務。
```csharp
    var synonymMap = new SynonymMap()
    {
        Name = "desc-synonymmap",
        Format = "solr",
        Synonyms = "hotel, motel\n
                    internet,wifi\n
                    five star=>luxury\n
                    economy,inexpensive=>budget"
    };

    serviceClient.SynonymMaps.CreateOrUpdate(synonymMap);
```
同義字對應必須符合開放原始碼標準 `solr` 格式。 [Azure 搜尋服務中的同義字](search-synonyms.md)的 `Apache Solr synonym format`一節會說明此格式。

2. 設定可搜尋的欄位，以使用索引定義中的同義字對應。 在 `EnableSynonymsInHotelsIndex` 中，我們會將 `synonymMaps` 屬性設定為新上傳的同義字對應名稱，以在 `category` 和 `tags` 兩個欄位上啟用同義字。
```csharp
  Index index = serviceClient.Indexes.Get("hotels");
  index.Fields.First(f => f.Name == "category").SynonymMaps = new[] { "desc-synonymmap" };
  index.Fields.First(f => f.Name == "tags").SynonymMaps = new[] { "desc-synonymmap" };

  serviceClient.Indexes.CreateOrUpdate(index);
```
當您新增同義字對應時，不需要重建索引。 您可以將同義字對應新增到您的服務，然後修改任何索引中的現有欄位定義，以使用新的同義字對應。 新增屬性對於索引可用性沒有任何影響。 停用欄位的同義字也是如此。 您只要將 `synonymMaps` 屬性設定為空白清單。
```csharp
  index.Fields.First(f => f.Name == "category").SynonymMaps = new List<string>();
```

## <a name="after-queries"></a>「之後」查詢

上傳同義字對應並將索引更新為使用同義字對應之後，第二個 `RunQueriesWithNonExistentTermsInIndex` 就會呼叫下列輸出︰

~~~
Search the entire index for the phrase "five star":

Name: Fancy Stay        Category: Luxury        Tags: [pool, view, wifi, concierge]

Search the entire index for the term 'internet':

Name: Fancy Stay        Category: Luxury        Tags: [pool, view, wifi, concierge]

Search the entire index for the terms 'economy' AND 'hotel':

Name: Roach Motel       Category: Budget        Tags: [motel, budget]
~~~
第一個查詢會尋找 `five star=>luxury` 規則所產生的文件。 第二個查詢會使用 `internet,wifi` 展開搜尋，而第三個查詢會同時使用 `hotel, motel` 和 `economy,inexpensive=>budget` 來尋找相符的文件。

新增同義字會全然改變搜尋經驗。 在本教學課程中，即使我們索引中的文件有關聯，原始查詢也無法傳回有意義的結果。 啟用同義字，我們就可以展開索引以包含常用的詞彙，而不需變更索引中的基礎資料。

## <a name="sample-application-source-code"></a>範例應用程式的原始程式碼
您可以在 [GitHub](https://github.com/Azure-Samples/search-dotnet-getting-started/tree/master/DotNetHowToSynonyms) 上尋找本逐步解說中所用範例應用程式的完整原始程式碼。

## <a name="clean-up-resources"></a>清除資源

在完成教學課程後，最快速的清除方式是刪除包含 Azure 搜尋服務的資源群組。 您現在可以刪除資源群組，以永久刪除當中所包含的所有項目。 在入口網站中，資源群組名稱位在 Azure 搜尋服務的 [概觀] 頁面上。

## <a name="next-steps"></a>後續步驟

本教學課程示範以 C# 程式碼[同義字 REST API](https://aka.ms/rgm6rq) 來建立及張貼對應規則，然後呼叫查詢上的同義字對應。 您可以在 [.NET SDK](https://docs.microsoft.com/dotnet/api/microsoft.azure.search) 和 [REST API](https://docs.microsoft.com/rest/api/searchservice/) 參考文件中找到其他資訊。

> [!div class="nextstepaction"]
> [如何在 Azure 搜尋服務中使用同義字](search-synonyms.md)
