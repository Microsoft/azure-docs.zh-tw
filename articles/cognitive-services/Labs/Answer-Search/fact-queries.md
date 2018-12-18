---
title: 快速入門：專案答案搜尋事實查詢
titlesuffix: Azure Cognitive Services
description: 使用專案答案搜尋進行事實查詢
services: cognitive-services
author: mikedodaro
manager: cgronlun
ms.service: cognitive-services
ms.component: project-answer-search
ms.topic: quickstart
ms.date: 04/16/2018
ms.author: rosh
ms.openlocfilehash: ae8a42b95df70cff9cb845d4d553fbbb33cbdec9
ms.sourcegitcommit: 0bb8db9fe3369ee90f4a5973a69c26bff43eae00
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/08/2018
ms.locfileid: "48868328"
---
# <a name="quickstart-query-for-facts"></a>快速入門：查詢事實

如果查詢的是日期或可識別的知識之類的事實，回應可能包含 `facts` 答案。 事實答案包含從網路文件中的段落擷取的相關結果。  這些查詢一律回傳網頁，而且[事實](fact-queries.md)及/或[實體](entity-queries.md)與查詢有關。

valentines+2016、when+is+ramadan 之類的查詢均被視為日期相關的查詢。 如果 Bing 判定查詢與日期相關，則回應包含 `facts` 答案。 

下列範例是與日期相關的 `facts` 答案。 

**查詢：**
````
https://labsportalppe.azure-api.net/answerSearch/v7.0/search?q=valentines+2016

````

**回應：**`subjectName` 欄位包含使用者查詢的顯示版本，您可以在顯示事實時使用這個版本為標籤。 如果查詢字串是 valentines+2016，Bing 可能將該字串變更為 Valentine's Day 2016。 描述欄位包含事實。

````
{   
    "_type" : "SearchResponse",   
    "queryContext" : {   
        "originalQuery" : "valentines 2016" 
    },   
    "facts" : {   
        "id" : "https:\/\/www.bingapis.com\/api\/v7\/#Facts",   
        "value" : [{   
            "description" : "Valentine's Day was on Sunday, February 14, 2016.",   
            "subjectName" : "Valentine's Day 2016"   
        }]   
    },   
    "rankingResponse" : {   
        "mainline" : {   
            "items" : [{   
                "answerType" : "Facts",   
                "value" : {   
                    "id" : "https:\/\/www.bingapis.com\/api\/v7\/knowledge\/#Facts"                   }   
            }]   
        }   
    }   
}   

````

查詢「Why is the sky blue?」\(天空為什麼是藍色？\) 回傳知識相關答案的範例。

**查詢：**

````
https://api.labs.cognitive.microsoft.com/answerSearch/v7.0/search?q=why+is+the+sky+blue

````

**回應：**`value/description` 欄位包含查詢要求的知識或資訊。

````
  "facts": {
    "id": "https://www.bingapis.com/api/v7/#Facts",
    "contractualRules": [
      {
        "_type": "ContractualRules/LinkAttribution",
        "text": "en.wikipedia.org/wiki/Diffuse_sky_radiation",
        "url": "http://en.wikipedia.org/wiki/Diffuse_sky_radiation"
      },
      {
        "_type": "ContractualRules/LinkAttribution",
        "text": "spaceplace.nasa.gov/blue-sky/en/",
        "url": "http://spaceplace.nasa.gov/blue-sky/en/"
      }
    ],
    "attributions": [
      {
        "providerDisplayName": "en.wikipedia.org/wiki/Diffuse_sky_radiation",
        "seeMoreUrl": "http://en.wikipedia.org/wiki/Diffuse_sky_radiation"
      },
      {
        "providerDisplayName": "spaceplace.nasa.gov/blue-sky/en/",
        "seeMoreUrl": "http://spaceplace.nasa.gov/blue-sky/en/"
      }
    ],
    "value": [
      {
        "image": {
          "webSearchUrl": "https://www.bing.com/images/search?view=detailv2&FORM=OIIRPO&q=&id=B632272C4E934D7F0B18790300B2DE34E7676C7A&simid=608045681196075791&iss=eqna",
          "name": "Why is the sky blue in the sky?",
          "thumbnailUrl": "https://www.bing.com/th?id=ODE.858093005&w=140&h=140&c=8&rs=1&qlt=100&pid=3.1",
          "isFamilyFriendly": true
        },
        "description": "When sunlight reaches Earth's atmosphere, it is scattered in all directions by all the gases and particles in the air. Blue is scattered by air molecules more than other colors because it travels as shorter, smaller waves. This is why we see a blue sky most of the time.\n\nCloser to the horizon, the sky fades to a lighter blue or white. The sunlight reaching us from low in the sky has passed through even more air than the sunlight reaching us from overhead. As the sunlight has passed through all this air, the air molecules have scattered and rescattered the blue light many times in many directions. Also, the surface of Earth has reflected and scattered the light. All this scattering mixes the colors together again so we see more white and less blue.",
        "subjectName": "Why is the sky blue in the sky?",
        "primaryData": [
          "The atmosphere scatters more blue light"
        ]
      }
    ]
  },

````

## <a name="tabular-data"></a>表格式資料
在某些情況下，事實可以當做 `_type: StructuredValue/TabularData` 回傳。 下列查詢會取得將咖啡和茶的對比資訊列出的表格式資料。

````
https://labsportalppe.azure-api.net/answerSearch/v7.0/search?q=coffee+vs+tea&mkt=en-us 

````
`facts` 結果包含下列資料列和資料格：
````
    "value": [
      {
        "subjectName": "Coffee vs. Tea",
        "richCaption": {
          "_type": "StructuredValue/TabularData",
          "header": [
            "",
            "Coffee",
            "Tea"
          ],
          "rows": [
            {
              "cells": [
                {
                  "text": "Part of plant used"
                },
                {
                  "text": "Bean"
                },
                {
                  "text": "Leaf"
                }
              ]
            },
            {
              "cells": [
                {
                  "text": "Caffeine Content"
                },
                {
                  "text": "80-185 mg per 8 ounce cup depending upon the brew and the..."
                },
                {
                  "text": "15 - 70 mg per cup"
                }
              ]
            },
            {
              "cells": [
                {
                  "text": "Types of Consumption"
                },
                {
                  "text": "Drip Coffee"
                },
                {
                  "text": "White Tea"
                }
              ]
            },
            {
              "cells": [
                {
                  "text": "Additions"
                },
                {
                  "text": "Sugar"
                },
                {
                  "text": "Milk"
                }
              ]
            }
          ],
          "seeMoreUrl": {
            "text": "8 more rows",
            "url": "http://www.diffen.com/difference/Coffee_vs_Tea"
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
