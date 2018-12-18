---
title: 查詢索引 (REST API - Azure 搜尋服務) | Microsoft Docs
description: 在 Azure 搜尋服務中建立搜尋查詢，並使用搜尋參數來篩選及排序搜尋結果。
author: brjohnstmsft
manager: jlembicz
ms.author: brjohnst
services: search
ms.service: search
ms.topic: quickstart
ms.date: 04/20/2018
ms.openlocfilehash: 035dc4ac349513867253e5593e01fab4fec62f6b
ms.sourcegitcommit: e2adef58c03b0a780173df2d988907b5cb809c82
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 04/28/2018
ms.locfileid: "32186506"
---
# <a name="query-your-azure-search-index-using-the-rest-api"></a>使用 REST API 查詢 Azure 搜尋服務索引
> [!div class="op_single_selector"]
>
> * [概觀](search-query-overview.md)
> * [入口網站](search-explorer.md)
> * [.NET](search-query-dotnet.md)
> * [REST](search-query-rest-api.md)
>
>

本文說明如何使用 [Azure 搜尋服務 REST API](https://docs.microsoft.com/rest/api/searchservice/) 查詢索引。

在開始閱讀本逐步解說前，請先[建立好 Azure 搜尋服務索引](search-what-is-an-index.md)，並[在索引中填入資料](search-what-is-data-import.md)。 如需有關背景資訊，請參閱[全文檢索搜尋如何在 Azure 搜尋服務中運作](search-lucene-query-architecture.md)。

## <a name="identify-your-azure-search-services-query-api-key"></a>識別 Azure 搜尋服務的查詢 API 金鑰
針對 Azure 搜尋服務 REST API 所執行之每個搜尋作業的主要元件是針對所佈建之服務產生的 API 金鑰  。 擁有有效的金鑰就能為每個要求在傳送要求之應用程式與處理要求之服務間建立信任。

1. 若要尋找服務的 API 金鑰，您可以登入 [Azure 入口網站](https://portal.azure.com/)
2. 前往 Azure 搜尋服務的刀鋒視窗。
3. 按一下 [金鑰] 圖示

您的服務有「系統管理金鑰」和「查詢金鑰」。

* 主要和次要系統管理金鑰  會授與所有作業的完整權限，包括管理服務以及建立和刪除索引、索引子與資料來源的能力。 由於有兩個金鑰，因此如果您決定重新產生主要金鑰，您可以繼續使用次要金鑰，反之亦然。
* 查詢金鑰  會授與索引和文件的唯讀存取權，且通常會分派給發出搜尋要求的用戶端應用程式。

若要查詢索引，您可以使用其中一個查詢金鑰。 系統管理金鑰也可以用於進行查詢，但是您應該在應用程式的程式碼中使用查詢金鑰，因為查詢金鑰更加符合 [最低權限準則](https://en.wikipedia.org/wiki/Principle_of_least_privilege)。

## <a name="formulate-your-query"></a>編寫查詢
有兩種方式可以 [使用 REST API 搜尋索引](https://docs.microsoft.com/rest/api/searchservice/Search-Documents)。 其中一種方式是發出 HTTP POST 要求，並將查詢參數定義在要求主體的 JSON 物件中。 另一種方式是發出 HTTP GET 要求，並將查詢參數定義在要求 URL 中。 在查詢參數的大小方面，POST 比 GET 擁有[更寬鬆的限制](https://docs.microsoft.com/rest/api/searchservice/Search-Documents)。 因此，除非您的情況特殊導致使用 GET 會比較方便，否則建議您使用 POST。

不管是 POST 還是 GET，您都需要在要求 URL 中提供服務名稱、索引名稱和適當的 API 版本 (本文件發行時的最新 API 版本是 `2017-11-11`)。 若為 GET，URL 結尾的「查詢字串」是您用來提供查詢參數的位置。 URL 的格式如下所示：

    https://[service name].search.windows.net/indexes/[index name]/docs?[query string]&api-version=2017-11-11

POST 的格式同上，但查詢字串參數中只有 API 版本。

#### <a name="example-queries"></a>查詢範例
以下是幾個名為 "hotels" 之索引的查詢範例。 範例中會同時顯示 GET 和 POST 格式的查詢。

在整個索引中搜尋 'budget' 一詞，並只傳回 `hotelName` 欄位：

```
GET https://[service name].search.windows.net/indexes/hotels/docs?search=budget&$select=hotelName&api-version=2017-11-11

POST https://[service name].search.windows.net/indexes/hotels/docs/search?api-version=2017-11-11
{
    "search": "budget",
    "select": "hotelName"
}
```

將篩選套用到索引，以尋找每晚房價低於美金 $150 元的旅館，並傳回 `hotelId` 和 `description`：

```
GET https://[service name].search.windows.net/indexes/hotels/docs?search=*&$filter=baseRate lt 150&$select=hotelId,description&api-version=2017-11-11

POST https://[service name].search.windows.net/indexes/hotels/docs/search?api-version=2017-11-11
{
    "search": "*",
    "filter": "baseRate lt 150",
    "select": "hotelId,description"
}
```

搜尋整個索引，依特定欄位 (`lastRenovationDate`) 以遞減順序排序，取前兩個結果，並只顯示 `hotelName` 和 `lastRenovationDate`：

```
GET https://[service name].search.windows.net/indexes/hotels/docs?search=*&$top=2&$orderby=lastRenovationDate desc&$select=hotelName,lastRenovationDate&api-version=2017-11-11

POST https://[service name].search.windows.net/indexes/hotels/docs/search?api-version=2017-11-11
{
    "search": "*",
    "orderby": "lastRenovationDate desc",
    "select": "hotelName,lastRenovationDate",
    "top": 2
}
```

## <a name="submit-your-http-request"></a>提交 HTTP 要求
現在您已將查詢編寫為 HTTP 要求 URL (若為 GET) 或主體 (若為 POST) 的一部分，接下來您可以定義要求標頭並提交查詢。

#### <a name="request-and-request-headers"></a>要求和要求標頭
您必須為 GET 定義兩個要求標頭，若為 POST 則必須定義三個：

1. `api-key` 標頭必須設為您在上面的步驟 I 中找到的查詢金鑰。 您也可以使用系統管理金鑰作為 `api-key` 標頭，但建議您使用查詢金鑰，因為其可獨佔授與索引和文件的唯讀存取權。
2. `Accept` 標頭必須設為 `application/json`。
3. 僅對於 POST，`Content-Type` 標頭也應該設為 `application/json`。

請參閱以下內容來取得使用 Azure 搜尋服務 REST API 搜尋 "hotels" 索引的 HTTP GET 要求，其使用簡單的查詢來搜尋 "motel" 一詞：

```
GET https://[service name].search.windows.net/indexes/hotels/docs?search=motel&api-version=2017-11-11
Accept: application/json
api-key: [query key]
```

以下是相同的查詢範例，但這次使用的是 HTTP POST：

```
POST https://[service name].search.windows.net/indexes/hotels/docs/search?api-version=2017-11-11
Content-Type: application/json
Accept: application/json
api-key: [query key]

{
    "search": "motel"
}
```

成功的查詢要求會產生 `200 OK` 的狀態碼，且搜尋結果會在回應主體中以 JSON 形式傳回。 以下是上述查詢結果的可能樣貌，此處假設 "hotels" 索引是以 [在 Azure 搜尋服務中使用 REST API 匯入資料](search-import-data-rest-api.md) 中的範例資料填入 (請注意，為了清楚起見，JSON 已經過格式化)。

```JSON
{
    "value": [
        {
            "@search.score": 0.59600675,
            "hotelId": "2",
            "baseRate": 79.99,
            "description": "Cheapest hotel in town",
            "description_fr": "Hôtel le moins cher en ville",
            "hotelName": "Roach Motel",
            "category": "Budget",
            "tags":["motel", "budget"],
            "parkingIncluded": true,
            "smokingAllowed": true,
            "lastRenovationDate": "1982-04-28T00:00:00Z",
            "rating": 1,
            "location": {
                "type": "Point",
                "coordinates": [-122.131577, 49.678581],
                "crs": {
                    "type":"name",
                    "properties": {
                        "name": "EPSG:4326"
                    }
                }
            }
        }
    ]
}
```

若要深入了解，請瀏覽 [搜尋文件](https://docs.microsoft.com/rest/api/searchservice/Search-Documents)的＜回應＞一節。 如需失敗時可能傳回的其他 HTTP 狀態碼詳細資訊，請參閱 [HTTP 狀態碼 (Azure 搜尋服務)](https://docs.microsoft.com/rest/api/searchservice/HTTP-status-codes)。
