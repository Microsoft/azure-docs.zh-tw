---
title: 在 Fiddler 或 Postman 中探索 REST API (Azure 搜尋服務 REST) | Microsoft Docs
description: 如何使用 Fiddler 或 Postman 以對 Azure 搜尋服務發出 HTTP 要求和 REST API 呼叫。
author: HeidiSteen
manager: cgronlun
services: search
ms.service: search
ms.devlang: rest-api
ms.topic: quickstart
ms.date: 04/20/2018
ms.author: heidist
ms.openlocfilehash: eba41086da645c2ff5cee65f9395267227cb1c11
ms.sourcegitcommit: e2adef58c03b0a780173df2d988907b5cb809c82
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 04/28/2018
ms.locfileid: "32190180"
---
# <a name="explore-azure-search-rest-apis-using-fiddler-or-postman"></a>使用 Fiddler 或 Postman 探索 Azure 搜尋服務 REST API

其中一個探索 [Azure 搜尋服務 REST API](https://docs.microsoft.com/rest/api/searchservice) 的最簡單方式，是使用 Fiddler 或 Postman 來制訂 HTTP 要求並且檢查回應。 透過適當的工具與下列指示，您可以在撰寫任何程式碼前，先傳送要求並檢視回應。

> [!div class="checklist"]
> * 下載 Web API 測試工具
> * 取得您的搜尋服務之 API 金鑰和端點
> * 設定要求標頭
> * 建立索引
> * 載入索引
> * 搜尋索引

如果您沒有 Azure 訂用帳戶，請在開始前建立[免費帳戶](https://azure.microsoft.com/free/?WT.mc_id=A261C142F)，然後[註冊 Azure 搜尋服務](search-create-service-portal.md)。

## <a name="download-and-install-tools"></a>下載並安裝工具

下列工具廣泛用於 Web 開發，但是如果您熟悉其他工具，仍然應該套用本文中的指示。

+ [Postman 桌面應用程式](https://www.getpostman.com/)
+ [Telerik Fiddler](http://www.telerik.com/fiddler)

## <a name="get-the-api-key-and-endpoint"></a>取得 API 金鑰和端點

REST 呼叫需要服務 URL 和每個要求的存取金鑰。 搜尋服務是同時建立，因此如果您將 Azure 搜尋服務新增至您的訂用帳戶，請遵循下列步驟來取得必要的資訊：

1. 在 Azure 入口網站中，從儀表板開啟搜尋服務分頁，或在服務清單中[尋找服務](https://ms.portal.azure.com/#blade/HubsExtension/BrowseResourceBlade/resourceType/Microsoft.Search%2FsearchServices)。
2. 在 [概觀] > [基本功能] > [URL] 中取得端點。 範例端點看起來會像是 `https://my-service-name.search.windows.net`。
3. 在 [設定] > [金鑰] 中取得 API 金鑰。 如果您想要復原金鑰，有兩個管理員金鑰可供備援。 管理員金鑰會授與您的服務之寫入存取權，這是建立和載入索引的必要權限。 您可以針對寫入作業使用主要或次要金鑰。

## <a name="configure-request-headers"></a>設定要求標頭

每個工具會保存工作階段的要求標頭資訊，這表示您只需要輸入一次 URL 端點、API 版本、API 金鑰和內容類型。

完整 URL 看起來應該類似下列範例，只是您應該有效取代 **`my-app`** 預留位置名稱：`https://my-app.search.windows.net/indexes/hotels?api-version=2017-11-11`

服務 URL 組合會包含下列元素：

+ HTTPS 前置詞。
+ 服務 URL，從入口網站取得。
+ 資源，在您的服務上建立物件的作業。 在此步驟中，它是名為旅館的索引。
+ API 版本，必要的小寫字串，針對目前版本指定為 "?api-version=2017-11-11"。 [API 版本](search-api-versions.md)會定期更新。 在每個要求上包括 API 版本可讓您具備所使用之版本的完整控制權。  

要求標頭組合包含兩個元素，也就是前一節中所述的內容類型和 API 金鑰：

         content-type: application/json
         api-key: <placeholder>

### <a name="fiddler"></a>Fiddler

制訂如下列螢幕擷取畫面所示的要求。 選擇 **PUT** 作為動詞。 Fiddler 會新增 `User-Agent=Fiddler`。 您可以在其下的新行上貼上兩個額外的要求標頭。 使用服務的管理員存取金鑰，包含服務的內容類型和 API 金鑰。

![Fiddler 要求標頭][1]

> [!Tip]
> 您可以關閉 Web 流量以隱藏與您正在執行之工作不相關的外來 HTTP 活動。 在 Fiddler 中，請移至 [檔案] 功能表並且關閉 [擷取流量]。 

### <a name="postman"></a>postman

制訂如下列螢幕擷取畫面所示的要求。 選擇 **PUT** 作為動詞。 

![Postman 要求標頭][6]

## <a name="create-the-index"></a>建立索引

要求本文包含索引定義。 新增要求本文會完成要求 (產生您的索引)。

除了索引名稱之外，要求中最重要的元件是欄位集合。 欄位集合會定義索引結構描述。 在每個欄位中指定其類型。 字串欄位用於全文檢索搜尋，因此如果您需要讓該內容可搜尋，您可能想要將數值資料轉換為字串。

欄位上的屬性會決定允許的動作。 REST API 預設允許許多動作。 例如，預設所有字串都可搜尋、可擷取、可篩選，以及可 Facet。 通常，您只需要在必須關閉行為時設定屬性。 如需屬性的詳細資訊，請參閱[建立索引 (REST)](https://docs.microsoft.com/rest/api/searchservice/create-index)。

          {
         "name": "hotels",  
         "fields": [
           {"name": "hotelId", "type": "Edm.String", "key":true, "searchable": false},
           {"name": "baseRate", "type": "Edm.Double"},
           {"name": "description", "type": "Edm.String", "filterable": false, "sortable": false, "facetable": false},
           {"name": "hotelName", "type": "Edm.String"},
           {"name": "category", "type": "Edm.String"},
           {"name": "tags", "type": "Collection(Edm.String)"},
           {"name": "parkingIncluded", "type": "Edm.Boolean"},
           {"name": "smokingAllowed", "type": "Edm.Boolean"},
           {"name": "lastRenovationDate", "type": "Edm.DateTimeOffset"},
           {"name": "rating", "type": "Edm.Int32"},
           {"name": "location", "type": "Edm.GeographyPoint"}
          ]
         }


當您提交此要求時，您應該會取得 HTTP 201 回應，指出已成功建立索引。 您可以在網站中驗證此動作，但是請注意，入口網站分頁有重新整理間隔，因此可能需要一兩分鐘才會更新。

如果您收到 HTTP 504，請確認 URL 所指定的是 HTTPS。 如果您看見 HTTP 400 或 404，請查看要求本文以確認並沒有「複製-貼上」錯誤。 HTTP 403 通常表示 API 金鑰有問題 (可能是金鑰無效或是用來指定 API 金鑰的語法有問題)。

### <a name="fiddler"></a>Fiddler

將索引定義複製到要求本文，類似於以下螢幕擷取畫面，然後按一下左上方的 [執行] 以傳送完成的要求。

![Fiddler 要求本文][7]

### <a name="postman"></a>postman

將索引定義複製到要求本文，類似於以下螢幕擷取畫面，然後按一下左上方的 [傳送] 以傳送完成的要求。

![Postman 要求本文][8]

## <a name="load-documents"></a>載入文件

建立索引和填入索引是個別的步驟。 在 Azure 搜尋服務中，索引會包含所有可搜尋的資料，您可以提供來作為 JSON 文件。 若要檢閱此作業的 API，請參閱[新增、更新或刪除文件 (REST)](https://docs.microsoft.com/rest/api/searchservice/addupdate-or-delete-documents)。

+ 針對此步驟將動詞變更為 **POST**。
+ 變更端點以包含 `/docs/index`。 完整的 URL 應該看起來像是 `https://my-app.search.windows.net/indexes/hotels/docs/index?api-version=2017-11-11`
+ 保留要求標頭的原狀。 

[Request Body (要求本文)] 包含 4 個要新增到飯店索引的文件。

         {
         "value": [
         {
             "@search.action": "upload",
             "hotelId": "1",
             "baseRate": 199.0,
             "description": "Best hotel in town",
             "hotelName": "Fancy Stay",
             "category": "Luxury",
             "tags": ["pool", "view", "wifi", "concierge"],
             "parkingIncluded": false,
             "smokingAllowed": false,
             "lastRenovationDate": "2010-06-27T00:00:00Z",
             "rating": 5,
             "location": { "type": "Point", "coordinates": [-122.131577, 47.678581] }
           },
           {
             "@search.action": "upload",
             "hotelId": "2",
             "baseRate": 79.99,
             "description": "Cheapest hotel in town",
             "hotelName": "Roach Motel",
             "category": "Budget",
             "tags": ["motel", "budget"],
             "parkingIncluded": true,
             "smokingAllowed": true,
             "lastRenovationDate": "1982-04-28T00:00:00Z",
             "rating": 1,
             "location": { "type": "Point", "coordinates": [-122.131577, 49.678581] }
           },
           {
             "@search.action": "upload",
             "hotelId": "3",
             "baseRate": 279.99,
             "description": "Surprisingly expensive",
             "hotelName": "Dew Drop Inn",
             "category": "Bed and Breakfast",
             "tags": ["charming", "quaint"],
             "parkingIncluded": true,
             "smokingAllowed": false,
             "lastRenovationDate": null,
             "rating": 4,
             "location": { "type": "Point", "coordinates": [-122.33207, 47.60621] }
           },
           {
             "@search.action": "upload",
             "hotelId": "4",
             "baseRate": 220.00,
             "description": "This could be the one",
             "hotelName": "A Hotel for Everyone",
             "category": "Basic hotel",
             "tags": ["pool", "wifi"],
             "parkingIncluded": true,
             "smokingAllowed": false,
             "lastRenovationDate": null,
             "rating": 4,
             "location": { "type": "Point", "coordinates": [-122.12151, 47.67399] }
           }
          ]
         }

幾秒鐘後，您應該就會在工作階段清單中看見 HTTP 200 的回應。 這表示已成功建立文件。 

如果您收到 207，表示至少有一個文件上傳失敗。 如果您收到 404，則表示要求的標頭或本文有語法錯誤：確認您已變更端點以包含 `/docs/index`。

> [!Tip]
> 針對選取的資料來源，您可以選擇替代「索引子」方法，這個方法可簡化並減少編制索引所需的程式碼數量。 如需詳細資訊，請參閱[索引子作業](https://docs.microsoft.com/rest/api/searchservice/indexer-operations)。

### <a name="fiddler"></a>Fiddler

將動詞變更為 **POST**。 變更 URL 以包含 `/docs/index`。 將文件複製到要求本文，類似於下列螢幕擷取畫面，然後執行要求。

![Fiddler 要求裝載][9]

### <a name="postman"></a>postman

將動詞變更為 **POST**。 變更 URL 以包含 `/docs/index`。 將文件複製到要求本文，類似於下列螢幕擷取畫面，然後執行要求。

![Postman 要求裝載][10]

## <a name="query-the-index"></a>查詢索引
現在已載入索引和文件，您可以對其發出查詢。 如需此 API 的詳細資訊，請參閱[搜尋文件 (REST)](https://docs.microsoft.com/rest/api/searchservice/search-documents)  

+ 針對此步驟將動詞變更為 **GET**。
+ 變更端點以包含查詢參數，包括搜尋字串。 查詢 URL 看起來會像是 `https://my-app.search.windows.net/indexes/hotels/docs?search=motel&$count=true&api-version=2017-11-11`
+ 保留要求標頭的原狀

此查詢會搜尋 "motel" 一詞，並在搜尋結果中傳回文件計數。 在您按一下 [傳送] 之後，Postman 的要求和回應看起來應該類似下列螢幕擷取畫面。 狀態碼應為 200。

 ![Postman 查詢回應][11]

### <a name="tips-for-running-our-sample-queries-in-fiddler"></a>在 Fiddler 中執行範例查詢的提示

下列範例查詢來自[搜尋索引作業 (Azure 搜尋服務 API)](https://docs.microsoft.com/rest/api/searchservice/Search-Documents) 一文。 本文中有許多範例查詢包含空格，這在 Fiddler 中是不允許的。 因此，請先使用 + 字元取代空格，再貼到查詢字串中，然後再於 Fiddler 中嘗試該查詢：

**前面的空格會被取代 (in lastRenovationDate desc)：**

        GET /indexes/hotels/docs?search=*&$orderby=lastRenovationDate desc&api-version=2017-11-11

**後面的空格會以 + 取代 (in lastRenovationDate+desc)：**

        GET /indexes/hotels/docs?search=*&$orderby=lastRenovationDate+desc&api-version=2017-11-11

## <a name="query-index-properties"></a>查詢索引屬性
您也可以查詢系統資訊以取得文件計數和儲存體用量：`https://my-app.search.windows.net/indexes/hotels/stats?api-version=2017-11-11`

在 Postman 中，您的要求看起來應該與下方項目類似，回應會包括文件計數和使用的空格 (以位元組表示)。

 ![Postman 系統查詢][12]

請注意，API 版本語法不同。 針對此要求，使用 `?` 以附加 API 版本。 ？ 從查詢字串分隔 URL 路徑，& 會分隔查詢字串中的每個 'name=value' 組。 針對此查詢，API 版本是查詢字串中第一個也是唯一一個項目。

如需此 API 的詳細資訊，請參閱[取得索引統計資料 (REST)](https://docs.microsoft.com/rest/api/searchservice/get-index-statistics)。


### <a name="tips-for-viewing-index-statistic-in-fiddler"></a>在 Fiddler 中檢視索引統計資料的提示

在 Fiddler 中，依序按一下 [檢測器] 索引標籤和 [標頭] 索引標籤，然後選取 JSON 格式。 您應該會看到文件計數和儲存體大小 (以 KB計算)。

## <a name="next-steps"></a>後續步驟

REST 用戶端對於臨時探索相當寶貴，但是現在您知道 REST API 的運作方式，您可以繼續進行程式碼。 對於您的下一個步驟，請參閱下列連結：

+ [建立索引 (REST)](search-create-index-rest-api.md)
+ [匯入資料 (REST)](search-import-data-rest-api.md)
+ [搜尋索引 (REST)](search-query-rest-api.md)

<!--Image References-->
[1]: ./media/search-fiddler/fiddler-url.png
[2]: ./media/search-fiddler/AzureSearch_Fiddler2_PostDocs.png
[3]: ./media/search-fiddler/AzureSearch_Fiddler3_Query.png
[4]: ./media/search-fiddler/AzureSearch_Fiddler4_QueryResults.png
[5]: ./media/search-fiddler/AzureSearch_Fiddler5_QueryStats.png
[6]: ./media/search-fiddler/postman-url.png
[7]: ./media/search-fiddler/fiddler-request.png
[8]: ./media/search-fiddler/postman-request.png
[9]: ./media/search-fiddler/fiddler-docs.png
[10]: ./media/search-fiddler/postman-docs.png
[11]: ./media/search-fiddler/postman-query.png
[12]: ./media/search-fiddler/postman-system-query.png