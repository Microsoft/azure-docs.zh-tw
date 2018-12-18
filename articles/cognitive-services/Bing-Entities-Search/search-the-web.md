---
title: 什麼是 Bing 實體搜尋？ | Microsoft Docs
description: 了解如何使用 Bing 實體搜尋 API 來搜尋網路上的實體和地點。
services: cognitive-services
author: swhite-msft
manager: ehansen
ms.assetid: 0B54E747-61BF-42AA-8788-E25D63F625FC
ms.service: cognitive-services
ms.component: bing-entity-search
ms.topic: article
ms.date: 07/06/2016
ms.author: scottwhi
ms.openlocfilehash: 275430bc6ee8f935978243e61f68713974648189
ms.sourcegitcommit: e0a678acb0dc928e5c5edde3ca04e6854eb05ea6
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/13/2018
ms.locfileid: "39008105"
---
# <a name="what-is-bing-entity-search"></a>什麼是 Bing 實體搜尋？

Bing 實體搜尋 API 會將搜尋查詢傳送至 Bing，並取得包含實體和地點的結果。 地點結果包含餐廳、旅館或其他本地商家。 若查詢指定本地商家名稱或要求商務類型 (例如我附近的餐廳)，Bing 會傳回地點。 若查詢指定已知人物、地點 (旅遊景點、縣/市、國家/地區等) 或事項，Bing 會傳回實體。

## <a name="suggesting--using-search-terms"></a>建議和使用搜尋字詞

若您提供使用者可在其中輸入其搜尋字詞的搜尋方塊，請使用 [Bing 自動建議 API](../bing-autosuggest/get-suggested-search-terms.md) 來改善使用經驗。 API 會根據部分搜尋字詞傳回建議的查詢字串，作為使用者類型。

在使用者輸入其搜尋字詞之後，URL 會先將此字詞編碼，再設定 [q](https://docs.microsoft.com/rest/api/cognitiveservices/bing-entities-api-v7-reference#query) 查詢參數。 例如，若使用者輸入 *Marcus Appel*，請將 `q` 設定為 *Marcus+Appel* 或 *Marcus%20Appel*。

若搜尋字詞包含拼字錯誤，搜尋回應會包含 [QueryContext](https://docs.microsoft.com/rest/api/cognitiveservices/bing-entities-api-v7-reference#querycontext) 物件。 物件會顯示 Bing 用於搜尋的原始拼字和已更正的拼字。

```json
"queryContext": {
    "originalQuery": "hollo wrld",
    "alteredQuery": "hello world",
    "alterationOverrideQuery": "+hollo wrld",
    "adultIntent": false
}
```

## <a name="requesting-entities"></a>要求實體

如需範例要求，請參閱[建立您的第一個要求](./quick-start.md)。

## <a name="the-response"></a>回應

回應包含 [SearchResponse](https://docs.microsoft.com/rest/api/cognitiveservices/bing-entities-api-v7-reference#searchresponse) 物件。 若 Bing 找到相關的實體或地點，物件會包含 `entities` 欄位、`places` 欄位或兩者。 否則，回應物件不包含任一欄位。
> [!NOTE]
> 實體回應支援多個市場，但地點回應支援僅美國公司地點。 

`entities` 欄位是包含 [Entity](https://docs.microsoft.com/rest/api/cognitiveservices/bing-entities-api-v7-reference#entity) 物件清單的 [EntityAnswer](https://docs.microsoft.com/rest/api/cognitiveservices/bing-entities-api-v7-reference#entityanswer) 物件 (請參閱 `value` 欄位)。 此清單可能包含單一主控實體、多個去除混淆實體或兩者。 

主控實體是 Bing 認為符合要求的唯一實體 (不會有哪個實體符合要求的模稜兩可情況)。 若可能有多個實體符合要求，清單會包含多個去除混淆實體。 例如，若要求使用電影系列的通用標題，清單可能會包含去除混淆實體。 但若要求指定系列中的特定標題，清單可能會包含單一主控實體。

實體包含已知的名人，例如歌手、演員、運動員、模特兒等；地點和地標，例如瑞尼爾山和林肯紀念堂；以及事項，例如香蕉、黃金貴賓犬、書籍或電影標題。 [entityPresentationInfo](https://docs.microsoft.com/rest/api/cognitiveservices/bing-entities-api-v7-reference#entitypresentationinfo) 欄位包含識別實體類型的提示。 例如，若它是人物、電影、動物或景點。 如需可能類型的清單，請參閱 [Entity Types](https://docs.microsoft.com/rest/api/cognitiveservices/bing-entities-api-v7-reference#entity-types) (實體類型)

```json
"entityPresentationInfo": {
    "entityScenario": "DominantEntity",
    "entityTypeHints": ["Attraction"],
    "entityTypeDisplayHint": "Mountain"
}, ...
```

以下顯示包含主控和去除混淆實體的回應。

```json
{
    "_type": "SearchResponse",
    "queryContext": {
        "originalQuery": "Mount Rainier"
    },
    "entities": {
        "value": [{
            "contractualRules": [{
                "_type": "ContractualRules/LicenseAttribution",
                "targetPropertyName": "description",
                "mustBeCloseToContent": true,
                "license": {
                    "name": "CC-BY-SA",
                    "url": "http://creativecommons.org/licenses/by-sa/3.0/"
                },
                "licenseNotice": "Text under CC-BY-SA license"
            },
            {
                "_type": "ContractualRules/LinkAttribution",
                "targetPropertyName": "description",
                "mustBeCloseToContent": true,
                "text": "contoso.com",
                "url": "http://contoso.com/mount_rainier"
            },
            {
                "_type": "ContractualRules/MediaAttribution",
                "targetPropertyName": "image",
                "mustBeCloseToContent": true,
                "url": "http://contoso.com/mount-rainier"
            }],
            "webSearchUrl": "https://www.bing.com/search?q=Mount%20Rainier...",
            "name": "Mount Rainier",
            "url": "http://www.northwindtraders.com/",
            "image": {
                "name": "Mount Rainier",
                "thumbnailUrl": "https://www.bing.com/th?id=A4ae343983daa4...",
                "provider": [{
                    "_type": "Organization",
                    "url": "http://contoso.com/mount_rainier"
                }],
                "hostPageUrl": "http://contoso.com/commons/7/72/mount_rain...",
                "width": 110,
                "height": 110
            },
            "description": "Mount Rainier is 14,411 ft tall and the highest mountain...",
            "entityPresentationInfo": {
                "entityScenario": "DominantEntity",
                "entityTypeHints": ["Attraction"]
            },
            "bingId": "38b9431e-cf91-93be-0584-c42a3ecbfdc7"
        },
        {
            "contractualRules": [{
                "_type": "ContractualRules/MediaAttribution",
                "targetPropertyName": "image",
                "mustBeCloseToContent": true,
                "url": "http://contoso.com/mount_rainier_national_park"
            }],
            "webSearchUrl": "https://www.bing.com/search?q=Mount%20Rainier%20National...",
            "name": "Mount Rainier National Park",
            "url": "http://worldwideimporters.com/",
            "image": {
                "name": "Mount Rainier National Park",
                "thumbnailUrl": "https://www.bing.com/th?id=A91bdc5a1b648a695a39...",
                "provider": [{
                    "_type": "Organization",
                    "url": "http://contoso.com/mount_rainier_national_park"
                }],
                "hostPageUrl": "http://contoso.com/en/7/7a...",
                "width": 50,
                "height": 50
            },
            "description": "Mount Rainier National Park is a United States National Park...",
            "entityPresentationInfo": {
                "entityScenario": "DisambiguationItem",
                "entityTypeHints": ["Organization"]
            },
            "bingId": "29d4b681-227a-3924-7bb1-8a54e8666b8c"
        }]
    }
}
```

此實體包含 `name`、`description` 和 `image` 欄位。 當您在使用者體驗中顯示這些欄位時，您必須屬性化這些欄位。 `contractualRules` 欄位包含您必須套用的屬性清單。 契約規則會識別套用屬性的欄位。 如需套用屬性的資訊，請參閱[屬性](#data-attribution)。

```json
"contractualRules": [{
    "_type": "ContractualRules/LicenseAttribution",
    "targetPropertyName": "description",
    "mustBeCloseToContent": true,
    "license": {
        "name": "CC-BY-SA",
        "url": "http://creativecommons.org/licenses/by-sa/3.0/"
    },
    "licenseNotice": "Text under CC-BY-SA license"
},
{
    "_type": "ContractualRules/LinkAttribution",
    "targetPropertyName": "description",
    "mustBeCloseToContent": true,
    "text": "contoso.com",
    "url": "http://contoso.com/wiki/Mount_Rainier"
},
{
    "_type": "ContractualRules/MediaAttribution",
    "targetPropertyName": "image",
    "mustBeCloseToContent": true,
    "url": "http://contoso.com/wiki/Mount_Rainier"
}], ...
```

當您顯示實體資訊 (名稱、描述和影像) 時，您也必須使用 `webSearchUrl` 欄位中的 URL 連結至包含實體的 Bing 搜尋結果頁面。


`places` 欄位是包含 [Place](https://docs.microsoft.com/rest/api/cognitiveservices/bing-entities-api-v7-reference#place) 物件清單的 [LocalEntityAnswer](https://docs.microsoft.com/rest/api/cognitiveservices/bing-entities-api-v7-reference#localentityanswer) 物件 (請參閱 `value` 欄位)。 清單包含一或多個符合要求的本地實體。

地點包含餐廳、旅館或本地商家。 [entityPresentationInfo](https://docs.microsoft.com/rest/api/cognitiveservices/bing-entities-api-v7-reference#entitypresentationinfo) 欄位包含識別本地實體類型的提示。 清單包含地點、本地商家、餐廳等提示清單。 陣列中的每個後續提示都會縮小實體的類型。 如需可能類型的清單，請參閱 [Entity Types](https://docs.microsoft.com/rest/api/cognitiveservices/bing-entities-api-v7-reference#entity-types) (實體類型)

```json
"entityPresentationInfo": {
    "entityScenario": "ListItem",
    "entityTypeHints": ["Place",
    "LocalBusiness",
    "Restaurant"]
}, ...
```
> [!NOTE]
> 實體回應支援多個市場，但地點回應支援僅美國公司地點。 

本地感知實體查詢 (例如「我附近的餐廳」) 需要使用者的位置以提供精確的結果。 您的要求應該一律使用 X-Search-Location 和 X-MSEdge-ClientIP 標頭來指定使用者的位置。 若 Bing 認為使用者的位置對查詢有利，則會將 [QueryContext](https://docs.microsoft.com/rest/api/cognitiveservices/bing-entities-api-v7-reference#querycontext) 的 `askUserForLocation` 欄位設定為 **true**。 

```json
{
    "_type": "SearchResponse",
    "queryContext": {
        "originalQuery": "Sinful Bakery and Cafe",
        "askUserForLocation": true
    },
    ...
}
```

地點結果包含地點的名稱、地址、電話號碼和實體網站的 URL。 當您顯示實體資訊時，您也必須使用 `webSearchUrl` 欄位中的 URL 連結至包含實體的 Bing 搜尋結果頁面。

```json
"places": {
    "value": [{
        "_type": "Restaurant",
        "webSearchUrl": "https://www.bing.com/search?q=Sinful%20Bakery...",
        "name": "Liberty's Delightful Sinful Bakery & Cafe",
        "url": "http://libertysdelightfulsinfulbakeryandcafe.com/",
        "entityPresentationInfo": {
            "entityScenario": "ListItem",
            "entityTypeHints": ["Place",
            "LocalBusiness",
            "Restaurant"]
        },
        "address": {
            "addressLocality": "Seattle",
            "addressRegion": "WA",
            "postalCode": "98112",
            "addressCountry": "US",
            "neighborhood": "Madison Park"
        },
        "telephone": "(800) 555-1212"
    }]
}
```

> [!NOTE]
> 您或代表您的第三方可能無法使用、保留、儲存、快取、共用或散發實體 API 的任何資料，以進行測試、開發、訓練、散發或提供給任何非 Microsoft 服務或功能。  

## <a name="data-attribution"></a>資料屬性

Bing 實體 API 回應包含第三方所擁有的資訊。 您會負責確定使用適當，例如遵守任何您使用者體驗可能依賴的 Creative Commons 授權。

若回應或結果包含 `contractualRules`、`attributions` 或 `provider` 欄位，您必須屬性化資料。 若回應不包含上述任何欄位，則不需要任何屬性。 若回應包含 `contractualRules` 欄位以及 `attributions` 和/或 `provider` 欄位，您必須使用契約規則來屬性化資料。

下列範例顯示包含 MediaAttribution 契約規則的實體，以及包含 `provider` 欄位的影像。 MediaAttribution 規則會識別影像作為規則的目標，因此您會忽略影像的 `provider` 欄位，並改用 MediaAttribution 規則來提供屬性。  

```json
"value": [{
    "contractualRules": [
        ...
        {
            "_type": "ContractualRules/MediaAttribution",
            "targetPropertyName": "image",
            "mustBeCloseToContent": true,
            "url": "http://contoso.com/mount_rainier"
        }
    ],
    ...
    "image": {
        "name": "Mount Rainier",
        "thumbnailUrl": "https://www.bing.com/th?id=A46378861201...",
        "provider": [{
            "_type": "Organization",
            "url": "http://contoso.com/mount_rainier"
        }],
        "hostPageUrl": "http://www.graphicdesigninstitute.com/Uploaded...",
        "width": 110,
        "height": 110
    },
    ...
}]
```

若契約規則包含 `targetPropertyName` 欄位，該規則只會套用至目標欄位。 否則，該規則會套用至包含 `contractualRules` 欄位的父物件。

在下列範例中，`LinkAttribution` 規則包含 `targetPropertyName` 欄位，因此該規則會套用至 `description` 欄位。 針對套用至特定欄位的規則，您必須在目標資料之後立即加入一行，其中包含提供者網站的超連結。 例如，若要屬性化描述，請在描述文字之後立即加入一行，其中包含提供者網站的超連結。在本例中，請建立 contoso.com 的連結。

```json
"entities": {
    "value": [{
            ...
            "description": "Marcus Appel is a former American....",
            ...
            "contractualRules": [{
                    "_type": "ContractualRules/LinkAttribution",
                    "targetPropertyName": "description",
                    "mustBeCloseToContent": true,
                    "text": "contoso.com",
                    "url": "http://contoso.com/cr?IG=B8AD73..."
                 },
            ...
  
```

### <a name="license-attribution"></a>授權屬性

若契約規則清單包含 [LicenseAttribution](https://docs.microsoft.com/rest/api/cognitiveservices/bing-entities-api-v7-reference#licenseattribution) 規則，您必須在緊接於套用授權之內容後面的該行上顯示通知。 `LicenseAttribution` 規則使用 `targetPropertyName` 欄位來識別套用授權的屬性。

以下示範內含 `LicenseAttribution` 規則的範例。

![授權屬性](./media/cognitive-services-bing-entities-api/licenseattribution.png)

您顯示的授權通知必須包括網站的超連結，其中包含授權的相關資訊。 一般而言，您會將授權名稱設為超連結。 例如，若通知是 **CC-BY-SA 授權文字**，而 CC-BY-SA 是授權名稱，您會將 CC-BY-SA 設為超連結。

### <a name="link-and-text-attribution"></a>連結和文字屬性

[LinkAttribution](https://docs.microsoft.com/rest/api/cognitiveservices/bing-entities-api-v7-reference#linkattribution) 和 [TextAttribution](https://docs.microsoft.com/rest/api/cognitiveservices/bing-entities-api-v7-reference#textattribution) 規則通常可用來識別資料的提供者。 `targetPropertyName` 欄位會識別套用規則的欄位。

若要屬性化提供者，請在套用屬性的內容 (例如目標欄位) 之後立即加入一行。 該行應該清楚標示，指出提供者是資料的來源。 例如，"Data from: contoso.com"。 針對 `LinkAttribution` 規則，您必須建立提供者網站的超連結。

以下示範內含 `LinkAttribution` 和 `TextAttribution` 規則的範例。

![連結文字屬性](./media/cognitive-services-bing-entities-api/linktextattribution.png)

### <a name="media-attribution"></a>媒體屬性

若實體包含影像，而且您會顯示該影像，您必須提供提供者網站的點選連結。 若實體包含 [MediaAttribution](https://docs.microsoft.com/rest/api/cognitiveservices/bing-entities-api-v7-reference#mediaattribution) 規則，請使用規則的 URL 建立點選連結。 否則，請使用包含在影像 `provider` 欄位中的 URL 建立點選連結。

以下示範內含影像 `provider` 欄位和契約規則的範例。 由於範例包含契約規則，因此您會忽略影像的 `provider` 欄位並套用 `MediaAttribution` 規則。

![媒體屬性](./media/cognitive-services-bing-entities-api/mediaattribution.png)

### <a name="search-or-search-like-experience"></a>搜尋或類似搜尋體驗

如同使用 Bing Web 搜尋 API，Bing 實體搜尋 API 只能作為直接使用者查詢或搜尋的結果，或是作為邏輯上可解譯為使用者搜尋要求之應用程式或體驗內的動作結果。 為了方便說明，以下是可接受之搜尋或類似搜尋體驗的一些範例。

- 使用者直接在應用程式的搜尋方塊中輸入查詢
- 使用者選取特定文字或影像，並要求「更多資訊」或「其他資訊」
- 使用者詢問搜尋 Bot 特定主題的相關資訊
- 使用者詳述圖像式搜尋類型案例中的特定物件或實體

若您不確定您的體驗是否可視為類似搜尋體驗，建議您與 Microsoft 確認。

## <a name="throttling-requests"></a>節流要求

[!INCLUDE [cognitive-services-bing-throttling-requests](../../../includes/cognitive-services-bing-throttling-requests.md)]

## <a name="next-steps"></a>後續步驟

若要快速開始使用您的第一個要求，請參閱[建立您的第一個要求](./quick-start.md)。

請熟悉 [Bing 實體搜尋 API v7](https://docs.microsoft.com/rest/api/cognitiveservices/bing-entities-api-v7-reference) 參考。 此參考包含您用來要求搜尋結果的標頭和查詢參數。 它也包含回應物件的定義。 

若要改善使用者搜尋方塊體驗，請參閱 [Bing 自動建議 API](../bing-autosuggest/get-suggested-search-terms.md)。 當使用者輸入其查詢字詞時，您可以呼叫此 API 以取得其他人所使用的相關查詢字詞。

務必閱讀 [Bing 使用和顯示需求](./use-display-requirements.md)，您才不會違反任何有關使用搜尋結果的規則。