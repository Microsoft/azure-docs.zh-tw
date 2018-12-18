---
pageTitle: Synonyms in Azure Search | Microsoft Docs
description: 使用同義字以擴充搜尋查詢的範圍
authors: mhko
services: search
ms.service: search
ms.devlang: rest-api
ms.topic: conceptual
ms.date: 04/20/2018
manager: jlembicz
ms.author: nateko
ms.openlocfilehash: 579d92f41e41cdb38d4a1eb0bb6e56ce4d4b2a45
ms.sourcegitcommit: cc4fdd6f0f12b44c244abc7f6bc4b181a2d05302
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/25/2018
ms.locfileid: "47093209"
---
# <a name="synonyms-in-azure-search"></a>Azure 搜尋服務的同義字

搜尋引擎中與對等詞彙相關聯的同義字，讓使用者不必實際提供詞彙，就能以隱含方式擴充查詢範圍。 例如，給定詞彙「狗」與關聯的同義字「犬科動物」和「小狗」，任何包含「狗」、「犬科動物」和「小狗」的文件都會包含在查詢範圍內。

在 Azure 搜尋服務中，同義字擴充在查詢同時就已完成。 您可以在不中斷現有作業的情況下，新增同義字地圖至服務中。 您無需重建索引，就可以將 synonymMaps 屬性新增至欄位定義。

## <a name="feature-availability"></a>功能可用性

最新的 api-version (api-version=2017-11-11) 支援同義字功能。 目前 Azure 入口網站並不支援此功能。

## <a name="how-to-use-synonyms-in-azure-search"></a>如何在 Azure 搜尋服務中使用同義字

Azure 搜尋服務是根據您定義並上傳至服務的同義字地圖，提供同義字支援。 這些地圖由獨立資源構成 (例如索引或資料資源)，且可以在您搜尋服務索引中的任何可搜尋欄位使用。

同義字地圖和索引會分開維護。 一旦您定義同義字地圖，並上傳至服務後，您可以透過在欄位定義中新增 **synonymMaps** 屬性，啟用同義字功能。 建立、 更新及刪除同義字地圖永遠是全文件的作業，這表示您無法以累加方式建立、 更新或刪除同義字地圖中的部分內容。 即便只是更新一個項目也需要重新載入。

將同義字整合至您的搜尋應用程式需要兩個步驟：

1.  透過下列的 API 將同義字地圖新增至您的搜尋服務。  

2.  在索引定義中設定要使用同義字地圖的可搜尋欄位。

### <a name="synonymmaps-resource-apis"></a>SynonymMaps 資源 API

#### <a name="add-or-update-a-synonym-map-under-your-service-using-post-or-put"></a>使用 POST 或 PUT 在您的服務中新增或更新同義字地圖。

同義地圖會透過 POST 或 PUT 上傳至服務。 每個規則都必須以新行字元 ('\n') 分隔。 在免費服務中，每個同義字地圖最多可以定義 5,000 條規則，而其他所有 SKU 最多可以定義 10,000 條規則。 每條規則最多可以有 20 個擴充詞彙。

同義字地圖必須使用 Apache Solr 格式，以下會加以說明。 如果您有現有的同義字字典使用的是不同格式，而您想直接使用此字典，請透過 [UserVoice](https://feedback.azure.com/forums/263029-azure-search) 讓我們知道。

您可以使用 HTTP POST 建立新的同義字地圖，如下列範例所示︰

    POST https://[servicename].search.windows.net/synonymmaps?api-version=2017-11-11
    api-key: [admin key]

    {  
       "name":"mysynonymmap",
       "format":"solr",
       "synonyms": "
          USA, United States, United States of America\n
          Washington, Wash., WA => WA\n"
    }

或者，您也可以使用 PUT，並在 URI 中指定同義字地圖名稱。 如果同義字地圖不存在，系統就會加以建立。

    PUT https://[servicename].search.windows.net/synonymmaps/mysynonymmap?api-version=2017-11-11
    api-key: [admin key]

    {  
       "format":"solr",
       "synonyms": "
          USA, United States, United States of America\n
          Washington, Wash., WA => WA\n"
    }

##### <a name="apache-solr-synonym-format"></a>Apache Solr 同義字格式

Solr 格式支援對等且明確的對應同義字。 對應規則需遵守 Apache Solr 的開放來源同義字篩選條件規格，規則如[關鍵字篩選條件](https://cwiki.apache.org/confluence/display/solr/Filter+Descriptions#FilterDescriptions-SynonymFilter)文件所述。 以下是對等同義字的樣本規則。
```
USA, United States, United States of America
```

根據上述規則，搜尋「USA」時，會擴充搜尋「USA」或「United States」以及「United States of America」。

明確的對應由箭號「=>」表示。 當指定時，符合「=>」左手邊搜尋查詢的詞彙序列，將會被右手邊的替代詞彙取代。 根據下列規則，搜尋查詢「Washington」、「Wash.」 或「WA」，都會重寫為「WA」。 明確對應只會套用在指定的方向，而且在此案例中，不會在查詢「WA」時重寫為「Washington」。
```
Washington, Wash., WA => WA
```

#### <a name="list-synonym-maps-under-your-service"></a>您服務中的同義字地圖清單。

    GET https://[servicename].search.windows.net/synonymmaps?api-version=2017-11-11
    api-key: [admin key]

#### <a name="get-a-synonym-map-under-your-service"></a>在您的服務中取得同義字地圖。

    GET https://[servicename].search.windows.net/synonymmaps/mysynonymmap?api-version=2017-11-11
    api-key: [admin key]

#### <a name="delete-a-synonyms-map-under-your-service"></a>刪除您服務中的同義字地圖。

    DELETE https://[servicename].search.windows.net/synonymmaps/mysynonymmap?api-version=2017-11-11
    api-key: [admin key]

### <a name="configure-a-searchable-field-to-use-the-synonym-map-in-the-index-definition"></a>在索引定義中設定要使用同義字地圖的可搜尋欄位。

您可以使用新的欄位屬性 **synonymMaps**，來指定要在可搜尋欄位中使用的同義字地圖。 同義字地圖屬於服務層級的資源，且服務中的任何索引欄位均可參考。

    POST https://[servicename].search.windows.net/indexes?api-version=2017-11-11
    api-key: [admin key]

    {
       "name":"myindex",
       "fields":[
          {
             "name":"id",
             "type":"Edm.String",
             "key":true
          },
          {
             "name":"name",
             "type":"Edm.String",
             "searchable":true,
             "analyzer":"en.lucene",
             "synonymMaps":[
                "mysynonymmap"
             ]
          },
          {
             "name":"name_jp",
             "type":"Edm.String",
             "searchable":true,
             "analyzer":"ja.microsoft",
             "synonymMaps":[
                "japanesesynonymmap"
             ]
          }
       ]
    }

可以為「Edm.String」或「Collection(Edm.String)」類型的可搜尋欄位指定 **synonymMaps**。

> [!NOTE]
> 每一個欄位僅能有一個同義字地圖。 如果您想要使用多個同義字地圖，請透過 [UserVoice](https://feedback.azure.com/forums/263029-azure-search) 讓我們知道。

## <a name="impact-of-synonyms-on-other-search-features"></a>同義字對其他搜尋功能的影響

同義字功能會在 OR 運算子中，使用同義字改寫原始的查詢。 基於這個原因，點閱數醒目提示與評分檔案，會將原始詞彙與同義字視為對等。

同義字功能適用於搜尋查詢，並不適用於篩選條件或面向。 同樣地，搜尋建議僅根據原始詞彙提供，同義字比對結果不會出現在回應中。

同義字擴充不適用於萬用字元搜尋詞彙；也不會擴充前置詞、模糊與 Regex 詞彙。

## <a name="tips-for-building-a-synonym-map"></a>建置同義字地圖的秘訣

- 簡潔、 設計良好的同義字地圖比詳盡的可能比對結果清單來得有效率。 過於龐大或複雜的字典需要花較長的時間剖析，且如果擴充至太多的同義字，會影響到查詢延遲。 與其猜測可能使用的詞彙，您可以透過[搜尋流量分析報告](search-traffic-analytics.md)取得實際使用的詞彙。

- 請啟用並使用這份報告作預備和驗證，以精確地判斷哪一個詞彙能夠有助於同義字配對，接著再使用此報告來驗證您的同義字地圖的確能產生較佳的結果。 在預先定義的報告中，「最常見的搜尋查詢」與「無結果的搜尋查詢」圖格能提供您必要資訊。

- 您可以為您的搜尋應用程式建立多個同義字地圖 (例如，如果您的應用程式支援多語言的客戶群，您可以建立不同語言的同義字地圖)。 目前，一個欄位只能使用一個同義字地圖。 您可以隨時更新欄位的 synonymMaps 屬性。

## <a name="next-steps"></a>後續步驟

- 如果您在開發 (非生產) 環境中有現有的索引，可以使用小型的字典進行試驗，看看增加同義字能如何改變搜尋體驗，包含對於評分檔案、點閱數醒目提示以及搜尋建議的影響。

- [啟用搜尋流量分析](search-traffic-analytics.md)，並使用預先定義的 Power BI 報告來了解哪些詞彙最常使用，與哪些詞彙沒有傳回文件。 有了這些深入解析加持後，您可以修改字典，為沒有產出結果，但應該在您的索引中得到文件的查詢包含同義字。
