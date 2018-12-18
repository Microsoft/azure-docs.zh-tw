---
title: 上傳資料 (REST API - Azure 搜尋服務) | Microsoft Docs
description: 了解如何使用 REST API 將資料上傳至 Azure 搜尋服務中的索引。
author: brjohnstmsft
manager: jlembicz
ms.author: brjohnst
services: search
ms.service: search
ms.devlang: rest-api
ms.topic: quickstart
ms.date: 04/20/2018
ms.openlocfilehash: 53b20c9db7efe1f8876eec7c0167dc151aa38786
ms.sourcegitcommit: e2adef58c03b0a780173df2d988907b5cb809c82
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 04/28/2018
---
# <a name="upload-data-to-azure-search-using-the-rest-api"></a>使用 REST API 將資料上傳至 Azure 搜尋服務
> [!div class="op_single_selector"]
>
> * [概觀](search-what-is-data-import.md)
> * [.NET](search-import-data-dotnet.md)
> * [REST](search-import-data-rest-api.md)
>
>

本文將說明如何使用 [Azure 搜尋服務 REST API](https://docs.microsoft.com/rest/api/searchservice/) 將資料匯入 Azure 搜尋服務索引。

在開始閱讀本逐步解說前，請先 [建立好 Azure 搜尋服務索引](search-what-is-an-index.md)。

若要使用 REST API 將文件推送至索引，您會發出 HTTP POST 要求至您的索引 URL 端點。 HTTP 要求主體是包含要新增、修改或刪除之文件的 JSON 物件。

## <a name="identify-your-azure-search-services-admin-api-key"></a>識別 Azure 搜尋服務的系統管理 API 金鑰
使用 REST API 對服務發出 HTTP 要求時，每個  API 要求都必須包含針對您佈建的搜尋服務所產生的 API 金鑰。 擁有有效的金鑰就能為每個要求在傳送要求之應用程式與處理要求之服務間建立信任。

1. 若要尋找服務的 API 金鑰，您可以登入 [Azure 入口網站](https://portal.azure.com/)
2. 前往 Azure 搜尋服務的刀鋒視窗。
3. 按一下 [金鑰] 圖示。

服務會有系統管理金鑰和查詢金鑰。

* 主要和次要系統管理金鑰  會授與所有作業的完整權限，包括管理服務以及建立和刪除索引、索引子與資料來源的能力。 由於有兩個金鑰，因此如果您決定重新產生主要金鑰，您可以繼續使用次要金鑰，反之亦然。
* 查詢金鑰  會授與索引和文件的唯讀存取權，且通常會分派給發出搜尋要求的用戶端應用程式。

主要或次要系統管理金鑰都可用於將資料匯入索引。

## <a name="decide-which-indexing-action-to-use"></a>決定要使用的索引編製動作
使用 REST API 時，您會發行具有 JSON 要求主體的 HTTP POST 要求到 Azure 搜尋服務索引的端點 URL。 HTTP 要求主體中的 JSON 物件會包含名為 "value" 的單一 JSON 陣列，陣列中則有代表想要新增至索引、更新或刪除之文件的 JSON 物件。

"value" 陣列中的每個 JSON 物件代表要編製索引的文件。 這些物件每一個都含有文件的索引鍵，並且會指定所需的索引編製動作 (上傳、合併、刪除等)。 依據您在以下動作中所做的選擇，每個文件內只需包含某些欄位：

| @search.action | 說明 | 每個文件的必要欄位 | 注意 |
| --- | --- | --- | --- |
| `upload` |`upload` 動作類似「upsert」，如果是新文件，就會插入該文件，如果文件已經存在，就會更新/取代它。 |索引鍵以及其他任何您想要定義的欄位 |在更新/取代現有文件時，要求中未指定的欄位會將其欄位設定為 `null`。 即使先前已將欄位設定為非 null 值也是一樣。 |
| `merge` |使用指定的欄位更新現有文件。 如果文件不存在於索引中，合併就會失敗。 |索引鍵以及其他任何您想要定義的欄位 |您在合併中指定的任何欄位將取代文件中現有的欄位。 這包括類型 `Collection(Edm.String)`的欄位。 例如，如果文件包含欄位 `tags` 且值為 `["budget"]`，而您使用值 `["economy", "pool"]` 針對 `tags` 執行合併，則 `tags` 欄位最後的值會是 `["economy", "pool"]`。 而不會是 `["budget", "economy", "pool"]`。 |
| `mergeOrUpload` |如果含有指定索引鍵的文件已經存在於索引中，則此動作的行為會類似 `merge`。 如果文件不存在，其行為會類似新文件的 `upload` 。 |索引鍵以及其他任何您想要定義的欄位 |- |
| `delete` |從索引中移除指定的文件。 |僅索引鍵 |您指定的所有欄位 (索引鍵欄位除外) 都將被忽略。 如果您想要從文件中移除個別欄位，請改用 `merge` ，而且只需明確地將該欄位設為 null。 |

## <a name="construct-your-http-request-and-request-body"></a>建構 HTTP 要求和要求本文
既然您已收集好索引動作的必要欄位值，您可以開始建構實際的 HTTP 要求和 JSON 要求主體以匯入資料。

#### <a name="request-and-request-headers"></a>要求和要求標頭
您需要在 URL 中提供服務名稱、索引名稱 (在本例中為 "hotels") 以及適當的 API 版本 (本文件發行時的最新 API 版本是 `2017-11-11` )。 您也需要定義 `Content-Type` 和 `api-key` 要求標頭。 請對後者使用服務的其中一個系統管理金鑰。

    POST https://[search service].search.windows.net/indexes/hotels/docs/index?api-version=2017-11-11
    Content-Type: application/json
    api-key: [admin key]

#### <a name="request-body"></a>要求本文
```JSON
{
    "value": [
        {
            "@search.action": "upload",
            "hotelId": "1",
            "baseRate": 199.0,
            "description": "Best hotel in town",
            "description_fr": "Meilleur hôtel en ville",
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
            "description_fr": "Hôtel le moins cher en ville",
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
            "@search.action": "mergeOrUpload",
            "hotelId": "3",
            "baseRate": 129.99,
            "description": "Close to town hall and the river"
        },
        {
            "@search.action": "delete",
            "hotelId": "6"
        }
    ]
}
```

在本案例中，我們會使用 `upload`、`mergeOrUpload` 和 `delete` 做為搜尋動作。

假設此 "hotels" 索引範例已填入幾份文件。 請留意我們在使用 `mergeOrUpload` 時是如何不必指定所有可能的文件欄位，以及在使用 `delete` 時如何僅指定文件索引鍵 (`hotelId`)。

另請注意，每個索引編製要求中最多只能包含 1000 份文件 (或 16 MB)。

## <a name="understand-your-http-response-code"></a>了解 HTTP 回應碼
#### <a name="200"></a>200
成功提交索引編製要求後，您會收到狀態碼為 `200 OK`的 HTTP 回應。 HTTP 回應的 JSON 主體如下所示：

```JSON
{
    "value": [
        {
            "key": "unique_key_of_document",
            "status": true,
            "errorMessage": null
        },
        ...
    ]
}
```

#### <a name="207"></a>207
至少有一個項目未成功建立索引時會傳回狀態碼 `207` 。 HTTP 回應的 JSON 主體會包含未成功之文件的相關資訊。

```JSON
{
    "value": [
        {
            "key": "unique_key_of_document",
            "status": false,
            "errorMessage": "The search service is too busy to process this document. Please try again later."
        },
        ...
    ]
}
```

> [!NOTE]
> 這通常表示搜尋服務的負載即將達到索引編製要求會開始傳回 `503` 回應的臨界點。 在此情況下，強烈建議您先讓用戶端程式碼退回並稍候一會，然後再重試。 這可讓系統有時間復原，以增加未來之要求的成功機會。 快速重試要求只會讓這種情況持續下去。
>
>

#### <a name="429"></a>429
當您已經超過每個索引的文件數量配額時，將會傳回 `429` 的狀態碼。

#### <a name="503"></a>503
如果要求中的所有項目皆未成功建立索引，將會傳回 `503` 的狀態碼。 此錯誤表示系統負載過重，因此目前無法處理要求。

> [!NOTE]
> 在此情況下，強烈建議您先讓用戶端程式碼退回並稍候一會，然後再重試。 這可讓系統有時間復原，以增加未來之要求的成功機會。 快速重試要求只會讓這種情況持續下去。
>
>

如需文件動作和成功/錯誤回應的詳細資訊，請參閱 [加入、更新或刪除文件](https://docs.microsoft.com/rest/api/searchservice/AddUpdate-or-Delete-Documents)。 如需失敗時可能傳回的其他 HTTP 狀態碼詳細資訊，請參閱 [HTTP 狀態碼 (Azure 搜尋服務)](https://docs.microsoft.com/rest/api/searchservice/HTTP-status-codes)。

## <a name="next-steps"></a>後續步驟
在填入 Azure 搜尋服務索引後，您就可以開始發出查詢來搜尋文件。 如需詳細資料，請參閱 [查詢 Azure 搜尋服務索引](search-query-overview.md) 。
