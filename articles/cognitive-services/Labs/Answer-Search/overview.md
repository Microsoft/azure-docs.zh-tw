---
title: 什麼是專案答案搜尋？
titlesuffix: Azure Cognitive Services
description: 專案答案搜尋簡介。
services: cognitive-services
author: mikedodaro
manager: cgronlun
ms.service: cognitive-services
ms.component: project-answer-search
ms.topic: overview
ms.date: 04/13/2018
ms.author: rosh
ms.openlocfilehash: 5658054b3cc77db20edd64f6c560ee5d4a58eb46
ms.sourcegitcommit: 55952b90dc3935a8ea8baeaae9692dbb9bedb47f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/09/2018
ms.locfileid: "48883715"
---
# <a name="what-is-project-answer-search"></a>什麼是專案答案搜尋？
專案答案搜尋 API 使用 Bing v7 端點取得疑問查詢的答案。 「地球的圓周有多長？」之類的問題 傳回事實資訊的答案。  人、地或事的查詢會傳回查詢所識別的實體有關的資訊。 這些情節適用於對話聊天機器人、傳訊應用程式、讀取器之類的應用程式。  

查詢會依據查詢情節傳回回應：一律傳回網頁，而且傳回相關的[事實](fact-queries.md)及/或[實體](entity-queries.md)。

## <a name="endpoint"></a>端點
若要取得問題的答案或人、地或事的資訊，請將要求傳送至答案搜尋 API 端點。 若有多個要求，請使用標頭和 URL 參數。  包含 *Ocp-Apim-Subscription-Key* 和有效的驗證權杖。  需要市場參數。 目前僅支援 `en-us` 市場。

下列查詢會取得問題的答案：「地球的圓周有多長？」

GET：
````
https://api.labs.cognitive.microsoft.com/answerSearch/v7.0/search?q=what+is+circumference+of+the=earth?&mkt=en-us

````

需要 URL 參數 `q=` 才能指定搜尋的物件。

## <a name="response-object"></a>回應物件

回應包括 HTTP 標頭、網頁、事實及/或實體。

````
BingAPIs-TraceId: AB2E75C998614ADB8EBF5110DF648298
X-MSEdge-ClientID: 1E48FC4F7B8768C80B14F7997A106906
BingAPIs-SessionId: 0504DDD6DAE84861A4842306F8DA7A58
BingAPIs-Market: en-US
X-MSEdge-Ref: Ref A: AB2E75C998614ADB8EBF5110DF648298 Ref B: CO1EDGE0322 Ref C: 2018-04-19T19:57:13Z

JSON Response:

{
  "_type": "SearchResponse",
  "queryContext": {
    "originalQuery": "what is the circumference of earth"
  },
  "webPages": {
    "webSearchUrl": "https://www.bing.com/search?q\u003dwhat+is+the+circumference+of+earth",
    "totalEstimatedMatches": 217000,
    "value": [
      {
        "id": "https://www.bingapis.com/api/v7/#WebPages.0",
        "name": "Circumference of the Earth - Universe Today",
        "url": "https://www.universetoday.com/26461/circumference-of-the-earth/",
        "isFamilyFriendly": true,
        "displayUrl": "https://www.universetoday.com/26461/circumference-of-the-earth",
        "snippet": "The circumference of the Earth in kilometers is 40,075 km, and the circumference of the Earth in miles is 24,901. In other words, if you could drive your car around the equator of the Earth (yes, even over the oceans), youâ€™d put on an extra 40,075 km on the odometer.",
        "deepLinks": [
          {
            "name": "About Earth",
            "url": "https://www.universetoday.com/14382/10-interesting-facts-about-planet-earth/"
          }
        ],
        "dateLastCrawled": "2018-04-12T14:13:00.0000000Z",
        "language": "en"
      },
      {
        "id": "https://www.bingapis.com/api/v7/#WebPages.1",
        "name": "Earth - Wikipedia",
        "url": "https://en.wikipedia.org/wiki/Earth",
        "about": [
          {
            "name": "Earth"
          },
          {
            "name": "Earth"
          }
        ],
        "isFamilyFriendly": true,
        "displayUrl": "https://en.wikipedia.org/wiki/Earth",
        "snippet": "Circumference: 40 075.017 km equatorial (24 901.461 mi) ... Earth is the third planet from the Sun and the only object in the Universe known to harbor life.",
        "deepLinks": [
          {
            "name": "Moon",
            "url": "https://en.wikipedia.org/wiki/Moon"
          },
          {
            "name": "Planet",
            "url": "https://en.wikipedia.org/wiki/Planet"
          },
          {
            "name": "Quasi-Satellites",
            "url": "https://en.wikipedia.org/wiki/Quasi-satellite"
          },
          {
            "name": "World Population",
            "url": "https://en.wikipedia.org/wiki/World_population"
          },
   . . .

    ]
  },
  "entities": {
    "value": [
      {
        "id": "https://www.bingapis.com/api/v7/#Entities.0",
        "contractualRules": [
          {
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
            "text": "Wikipedia",
            "url": "http://en.wikipedia.org/wiki/Earth"
          },
          {
            "_type": "ContractualRules/MediaAttribution",
            "targetPropertyName": "image",
            "mustBeCloseToContent": true,
            "url": "http://en.wikipedia.org/wiki/Earth"
          }
        ],
        "webSearchUrl": "https://www.bing.com/entityexplore?q\u003dEarth\u0026filters\u003dsid:%226ddb3372-4801-5567-321e-e8a53bd774a4%22\u0026elv\u003dAXXfrEiqqD9r3GuelwApulpmymQx!ODfuQu*veOQHkvP0!Zbvi5F5tVcMSDJvDEWiQWwrdueYTtIszgj03oFQHykYYLYgq3q5!Sf00QxXGIS",
        "name": "Earth",
        "image": {
          "name": "Earth",
          "thumbnailUrl": "https://www.bing.com/th?id\u003dA3ab623665ab412f386c162bd29f0683a\u0026w\u003d110\u0026h\u003d110\u0026c\u003d7\u0026rs\u003d1\u0026qlt\u003d80\u0026cdv\u003d1\u0026pid\u003d16.1",
          "provider": [
            {
              "_type": "Organization",
              "url": "http://en.wikipedia.org/wiki/Earth"
            }
          ],
          "hostPageUrl": "http://upload.wikimedia.org/wikipedia/commons/9/97/The_Earth_seen_from_Apollo_17.jpg",
          "width": 110,
          "height": 110,
          "sourceWidth": 799,
          "sourceHeight": 800
        },
        "description": "Earth is the third planet from the Sun and the only object in the Universe known to harbor life. According to radiometric dating and other sources of evidence, Earth formed over 4.5 billion years ago. Earth\u0027s gravity interacts with other objects in space, especially the Sun and the Moon, Earth\u0027s only natural satellite. Earth revolves around the Sun in 365.26 days, a period known as an Earth year. During this time, Earth rotates about its axis about 366.26 times.",
        "entityPresentationInfo": {
          "entityScenario": "DominantEntity",
          "entityTypeHints": [
            "Generic"
          ]
        },
        "bingId": "6ddb3372-4801-5567-321e-e8a53bd774a4"
      }
    ]
  },
  "facts": {
    "id": "https://www.bingapis.com/api/v7/#Facts",
    "contractualRules": [
      {
        "_type": "ContractualRules/LinkAttribution",
        "text": "www.universetoday.com/26461/circumference-of-the-earth/",
        "url": "http://www.universetoday.com/26461/circumference-of-the-earth/"
      }
    ],
    "attributions": [
      {
        "providerDisplayName": "www.universetoday.com/26461/circumference-of-the-earth/",
        "seeMoreUrl": "http://www.universetoday.com/26461/circumference-of-the-earth/"
      }
    ],
    "value": [
      {
        "description": "The circumference of the Earth in kilometers is 40,075 km, and the circumference of the Earth in miles is 24,901. In other words, if you could drive your car around the equator of the Earth (yes, even over the oceans), youâ€™d put on an extra 40,075 km on the odometer.",
        "subjectName": ""
      }
    ]
  },
  "rankingResponse": {
    "mainline": {
      "items": [
        {
          "answerType": "Facts",
          "value": {
            "id": "https://www.bingapis.com/api/v7/#Facts"
          }
        },
        {
          "answerType": "WebPages",
          "resultIndex": 0,
          "value": {
            "id": "https://www.bingapis.com/api/v7/#WebPages.0"
          }
        },
        {
          "answerType": "WebPages",
          "resultIndex": 1,
          "value": {
            "id": "https://www.bingapis.com/api/v7/#WebPages.1"
          }
        },
        {
          "answerType": "WebPages",
          "resultIndex": 2,
          "value": {
            "id": "https://www.bingapis.com/api/v7/#WebPages.2"
          }
        },
        
        . . . 
      ]
    },
    "sidebar": {
      "items": [
        {
          "answerType": "Entities",
          "resultIndex": 0,
          "value": {
            "id": "https://www.bingapis.com/api/v7/#Entities.0"
          }
        }
      ]
    }
  }
}

````

## <a name="terms-of-use"></a>使用規定
專案答案搜尋和專案視訊趨勢都受限於 [Bing 搜尋使用和顯示需求](use-display-requirements.md)。

您或代表您的第三方可能無法使用、保留、儲存、快取、共用或散發 URL 預覽 API 的任何資料，以進行測試、開發、訓練、散發或提供給任何非 Microsoft 服務或功能。 

## <a name="throttling-requests"></a>節流要求

[!INCLUDE [cognitive-services-bing-throttling-requests](../../../../includes/cognitive-services-bing-throttling-requests.md)]


## <a name="data-attribution"></a>資料屬性  

專案答案搜尋回應包含第三方所擁有的資訊。 您會負責確定使用適當，例如遵守任何您使用者體驗可能依賴的 Creative Commons 授權。  
  
若回應或結果包含 `contractualRules`、`attributions` 或 `provider` 欄位，您必須屬性化資料。 若回應不包含上述任何欄位，則不需要任何屬性。 若回應包含 `contractualRules` 欄位以及 `attributions` 和/或 `provider` 欄位，您必須使用契約規則來屬性化資料。  
  
下列範例顯示包含 MediaAttribution 契約規則的實體，以及包含 `provider` 欄位的影像。 MediaAttribution 規則會識別影像作為規則的目標，因此您會忽略影像的 `provider` 欄位，並改用 MediaAttribution 規則來提供屬性。  
  
```  
        "value" : [{
            "contractualRules" : [
                . . .
                {
                    "_type" : "ContractualRules\/MediaAttribution",
                    "targetPropertyName" : "image",
                    "mustBeCloseToContent" : true,
                    "url" : "http:\/\/en.wikipedia.org\/wiki\/Space_Needle"
                }
            ],
            . . .
            "image" : {
                "name" : "Space Needle",
                "thumbnailUrl" : "https:\/\/www.bing.com\/th?id=A46378861201...",
                "provider" : [{
                    "_type" : "Organization",
                    "url" : "http:\/\/en.wikipedia.org\/wiki\/Space_Needle"
                }],
                "hostPageUrl" : "http:\/\/www.citydictionary.com\/Uploaded...",
                "width" : 110,
                "height" : 110
            },
            . . .
        }]
```  
  
若契約規則包含 `targetPropertyName` 欄位，該規則只會套用至目標欄位。 否則，該規則會套用至包含 `contractualRules` 欄位的父物件。  
  
  
在下列範例中，`LinkAttribution` 規則包含 `targetPropertyName` 欄位，因此該規則會套用至 `description` 欄位。 針對套用至特定欄位的規則，您必須在目標資料之後立即加入一行，其中包含提供者網站的超連結。 例如，若要屬性化描述，請在描述文字之後立即加入一行，其中包含提供者網站的超連結。在本例中，請建立 contoso.com 的連結。  
  
```  
"entities" : {  
    "value" : [{  
            . . .  
            "description" : "Peyton Williams Manning is a former American....",  
            . . .  
            "contractualRules" : [{  
                    "_type" : "ContractualRules\/LinkAttribution",  
                    "targetPropertyName" : "description",  
                    "mustBeCloseToContent" : true,  
                    "text" : "en.wikipedia.org",  
                    "url" : "http:\/\/www.bing.com\/cr?IG=B8AD73..."  
                 },  
            . . .  
  
```  

### <a name="license-attribution"></a>授權屬性  

如果契約規則清單包含 [LicenseAttribution](https://docs.microsoft.com/rest/api/cognitiveservices/bing-entities-api-v7-reference#licenseattribution) 規則，您必須在緊接於套用授權之內容後面的該行上顯示通知。 `LicenseAttribution` 規則使用 `targetPropertyName` 欄位來識別套用授權的屬性。  
  
以下示範內含 `LicenseAttribution` 規則的範例。  
  
![授權屬性](./media/licenseattribution.png)  
  
您顯示的授權通知必須包括網站的超連結，其中包含授權的相關資訊。 一般而言，您會將授權名稱設為超連結。 例如，如果通知是 **CC-BY-SA 授權文字**，而 CC-BY-SA 是授權名稱，您會將 CC-BY-SA 設為超連結。  
  
### <a name="link-and-text-attribution"></a>連結和文字屬性  

[LinkAttribution](reference.md#linkattribution) 和 [TextAttribution](reference.md#textattribution) 規則通常可用來識別資料的提供者。 `targetPropertyName` 欄位會識別套用規則的欄位。  
  
若要屬性化提供者，請在套用屬性的內容 (例如目標欄位) 之後立即加入一行。 該行應該清楚標示，指出提供者是資料的來源。 例如，「資料來自：en.wikipedia.org」。 針對 `LinkAttribution` 規則，您必須建立提供者網站的超連結。  
  
以下示範內含 `LinkAttribution` 和 `TextAttribution` 規則的範例。  
  
![連結文字屬性](./media/linktextattribution.png)  

### <a name="media-attribution"></a>媒體屬性  

如果實體包含影像，而且您會顯示該影像，您必須提供提供者網站的點選連結。 若實體包含 [MediaAttribution](reference.md#mediaattribution) 規則，請使用規則的 URL 建立點選連結。 否則，請使用包含在影像 `provider` 欄位中的 URL 建立點選連結。  
  
以下示範內含影像 `provider` 欄位和契約規則的範例。 由於範例包含契約規則，因此您會忽略影像的 `provider` 欄位並套用 `MediaAttribution` 規則。  
  
![媒體屬性](./media/mediaattribution.png)  

## <a name="next-steps"></a>後續步驟
- [C# 快速入門](c-sharp-quickstart.md)
- [Java 快速入門](java-quickstart.md)
- [Node 快速入門](node-quickstart.md)
- [Python 快速入門](python-quickstart.md)
