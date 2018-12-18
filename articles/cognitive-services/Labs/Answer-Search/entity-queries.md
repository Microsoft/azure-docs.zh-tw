---
title: 快速入門：專案答案搜尋實體查詢
titlesuffix: Azure Cognitive Services
description: 使用專案答案搜尋進行實體查詢
services: cognitive-services
author: mikedodaro
manager: cgronlun
ms.service: cognitive-services
ms.component: project-answer-search
ms.topic: quickstart
ms.date: 04/16/2018
ms.author: rosh
ms.openlocfilehash: 0845f491772b905599bb60e8ec555d14b6d6b15f
ms.sourcegitcommit: 55952b90dc3935a8ea8baeaae9692dbb9bedb47f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/09/2018
ms.locfileid: "48883596"
---
# <a name="quickstart-query-for-entities"></a>快速入門：查詢實體

如果查詢要求人員、位置或事項的相關資訊，則回應可能包含 `entities` 答案。  查詢一律會傳回網頁，且[事實](fact-queries.md)及/或[實體](entity-queries.md)會與查詢有關。

實體支援三種查詢案例： 
-   DominantEntity — 只有一個符合使用者查詢和意圖的實體。 例如，「太空針塔」即為 DominantEntity 案例。 
-   去除混淆 — 有多個符合使用者查詢和意圖的實體，且使用者可以選取正確的實體。 例如，查詢「冰與火之歌」即為去除混淆案例，會傳回電視節目及書籍系列。 
-   清單 — 有多個符合使用者查詢和意圖的實體。 例如，查詢「瀕臨絕種物種清單」即為清單案例，會傳回格式化為要在資料列和資料格中顯示的表格式值。 
 
若要判斷查詢案例，請使用 `entities` 物件的 `queryScenario` 欄位。 實體所包含的資料取決於實體的類型。 雖然實體包含相同的基本資訊，但某些實體 (例如旅遊景點或旅遊書籍) 會包含其他屬性。 包含其他屬性的實體會有 `_type` 欄位，其中包含序列化程式所使用的提示。 下列實體包含其他屬性： 
-   書籍 
-   MusicRecording 
-   Person 
-   景點 
 
若要決定回應所包含的實體類型，請使用 `entityTypeHints` 欄位，如 Bill Gates 的下列查詢所示。
````
        },
        "description": "Bill Gates is an American business man and philanthropist, co-founder of Microsoft",
        "entityPresentationInfo": {
          "entityScenario": "DominantEntity",
          "entityTypeHints": [
            "Person"
          ]
        },
        "bingId": "6d7d66a7-2cb8-0ae9-637c-f81fd749dc9a"
      }
````
以下是「太空針塔」的查詢：
````
https://api.labs.cognitive.microsoft.com/answerSearch/v7.0/search?q=space+needle&mkt=en-us
````
回應會包含 `entities` 答案。 請注意 `entityScenario` 和 `entityTypeHints` 欄位。 
````
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
            "url": "http://en.wikipedia.org/wiki/Space_Needle"
          },
          {
            "_type": "ContractualRules/MediaAttribution",
            "targetPropertyName": "image",
            "mustBeCloseToContent": true,
            "url": "http://en.wikipedia.org/wiki/Space_Needle"
          }
        ],
        "webSearchUrl": "https://www.bing.com/entityexplore?q\u003dSpace+Needle\u0026filters\u003dsid:%22f8dd5b08-206d-2554-6e4a-893f51f4de7e%22\u0026elv\u003dAXXfrEiqqD9r3GuelwApulpmymQx!ODfuQu*veOQHkvP0!Zbvi5F5tVcMSDJvDEWiQWwrdueYTtIszgj03oFQHykYYLYgq3q5!Sf00QxXGIS",
        "name": "Space Needle",
        "image": {
          "name": "Space Needle",
          "thumbnailUrl": "https://www.bing.com/th?id\u003dA15d336cf119b9b5c7e0ab37e271421d3\u0026w\u003d110\u0026h\u003d110\u0026c\u003d7\u0026rs\u003d1\u0026qlt\u003d80\u0026cdv\u003d1\u0026pid\u003d16.1",
          "provider": [
            {
              "_type": "Organization",
              "url": "http://en.wikipedia.org/wiki/Space_Needle"
            }
          ],
          "hostPageUrl": "http://upload.wikimedia.org/wikipedia/commons/2/23/Space_Needle_2011-07-04.jpg",
          "width": 110,
          "height": 110,
          "sourceWidth": 152,
          "sourceHeight": 300
        },
        "description": "The Space Needle is an observation tower in Seattle, Washington, a landmark of the Pacific Northwest, and an icon of Seattle. It was built in the Seattle Center for the 1962 World\u0027s Fair, which drew over 2.3 million visitors, when nearly 20,000 people a day used its elevators.",
        "entityPresentationInfo": {
          "entityScenario": "DominantEntity",
          "entityTypeHints": [
            "Attraction"
          ]
        },
        "bingId": "f8dd5b08-206d-2554-6e4a-893f51f4de7e"
      }
    ]
  },
````

查詢可以傳回相關清單。

**查詢：** 下列查詢會尋找瀕臨絕種物的種清單：

````
https://api.labs.cognitive.microsoft.com/answerSearch/v7.0/search?q=list+of+endangered+species

````

**回應：** 回應會包含已格式化而顯示為表格式值的清單：
````
  "facts": {
    "id": "https://www.bingapis.com/api/v7/#Facts",
    "contractualRules": [
      {
        "_type": "ContractualRules/LinkAttribution",
        "text": "www.worldwildlife.org/species/directory?direction=desc&sort=e…",
        "url": "https://www.worldwildlife.org/species/directory?direction=desc&sort=extinction_status"
      }
    ],
    "attributions": [
      {
        "providerDisplayName": "www.worldwildlife.org/species/directory?direction=desc&sort=e…",
        "seeMoreUrl": "https://www.worldwildlife.org/species/directory?direction=desc&sort=extinction_status"
      }
    ],
    "value": [
      {
        "subjectName": "Species Directory",
        "richCaption": {
          "_type": "StructuredValue/TabularData",
          "header": [
            "Common name",
            "Scientific name",
            "Conservation status ↓"
          ],
          "rows": [
            {
              "cells": [
                {
                  "_type": "Properties/Link",
                  "text": "Amur Leopard",
                  "url": "https://www.bing.com/https//www.worldwildlife.org/species/amur-leopard"
                },
                {
                  "text": "Panthera pardus orientalis"
                },
                {
                  "text": "Critically Endangered"
                }
              ]
            },
            {
              "cells": [
                {
                  "_type": "Properties/Link",
                  "text": "Black Rhino",
                  "url": "https://www.bing.com/https//www.worldwildlife.org/species/black-rhino"
                },
                {
                  "text": "Diceros bicornis"
                },
                {
                  "text": "Critically Endangered"
                }
              ]
            },
            {
              "cells": [
                {
                  "_type": "Properties/Link",
                  "text": "Bornean Orangutan",
                  "url": "https://www.bing.com/https//www.worldwildlife.org/species/bornean-orangutan"
                },
                {
                  "text": "Pongo pygmaeus"
                },
                {
                  "text": "Critically Endangered"
                }
              ]
            },
            {
              "cells": [
                {
                  "_type": "Properties/Link",
                  "text": "Cross River Gorilla",
                  "url": "https://www.bing.com/https//www.worldwildlife.org/species/cross-river-gorilla"
                },
                {
                  "text": "Gorilla gorilla diehli"
                },
                {
                  "text": "Critically Endangered"
                }
              ]
            }
          ],
          "seeMoreUrl": {
            "text": "46 more rows",
            "url": "https://www.worldwildlife.org/species/directory?direction=desc&sort=extinction_status"
          }
        }
      }
    ]
  },

````


## <a name="next-steps"></a>後續步驟
- [C# 快速入門](c-sharp-quickstart.md)
- [Java 快速入門](java-quickstart.md)
- [Node 快速入門](node-quickstart.md)
- [Python 快速入門](python-quickstart.md)