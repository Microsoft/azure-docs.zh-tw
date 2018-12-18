---
title: LUIS (Language Understanding) 中的資料擷取概念
titleSuffix: Azure Cognitive Services
description: 了解可以從 Language Understanding (LUIS) 擷取哪些類型的資料
services: cognitive-services
author: diberry
manager: cgronlun
ms.service: cognitive-services
ms.component: language-understanding
ms.topic: article
ms.date: 09/10/2018
ms.author: diberry
ms.openlocfilehash: 39d36ee0c46d3e6954c3264f37f3f575130186b9
ms.sourcegitcommit: 7c4fd6fe267f79e760dc9aa8b432caa03d34615d
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/28/2018
ms.locfileid: "47434478"
---
# <a name="data-extraction"></a>資料擷取
LUIS 可讓您從使用者的自然語言語句取得資訊。 此資訊的擷取方式使得它可供程式、應用程式或 Chatbot 用來執行動作。 在下列各節中，您將透過 JSON 範例，了解從意圖和實體會傳回哪些資料。

最難擷取的資料是機器學習資料，因為它不是全文相符的資料。 機器學習[實體](luis-concept-entity-types.md)的資料擷取必須是[撰寫循環](luis-concept-app-iteration.md)的一部分，直到您確信收到預期的資料為止。

## <a name="data-location-and-key-usage"></a>資料位置和金鑰使用方式
LUIS 會從已發佈的[端點](luis-glossary.md#endpoint)提供資料。 **HTTPS 要求** (POST 或 GET) 除了包含一些額外的設定 (例如預備或生產環境) 之外，也包含語句。

`https://westus.api.cognitive.microsoft.com/luis/v2.0/apps/<appID>?subscription-key=<subscription-key>&verbose=true&timezoneOffset=0&q=book 2 tickets to paris`

當您編輯 LUIS 應用程式時，從該 LUIS 應用程式的 [Settings] \(設定\) 頁面，以及從您 URL 的一部分 (在 `/apps/` 之後)，都可以取得 `appID`。 `subscription-key` 是用來查詢您應用程式的端點金鑰。 在學習 LUIS 期間，雖然您可以使用免費的撰寫/入門金鑰，但請務必將端點金鑰變更為支援[預期的 LUIS 使用方式](luis-boundaries.md#key-limits)的金鑰。 `timezoneOffset` 單位為分鐘。

**HTTPS 回應**包含 LUIS 可以根據目前已發佈之預備或生產環境端點模型來判斷的所有意圖和實體資訊。 端點 URL 是在 [LUIS](luis-reference-regions.md) 網站、[管理] 區段、[金鑰和端點] 頁面上找到的。

## <a name="data-from-intents"></a>來自意圖的資料
主要資料是最高分的**意圖名稱**。 使用 `MyStore` [快速入門](luis-quickstart-intents-only.md) 時，端點回應為：

```JSON
{
  "query": "when do you open next?",
  "topScoringIntent": {
    "intent": "GetStoreInfo",
    "score": 0.984749258
  },
  "entities": []
}
```

|資料物件|資料類型|資料位置|值|
|--|--|--|--|
|意圖|字串|topScoringIntent.intent|"GetStoreInfo"|

如果您的 Chatbot 或 LUIS 呼叫應用程式會根據多個意圖分數進行決策，則請設定查詢字串參數 `verbose=true` 來傳回所有意圖的分數。 端點回應為：

```JSON
{
  "query": "when do you open next?",
  "topScoringIntent": {
    "intent": "GetStoreInfo",
    "score": 0.984749258
  },
  "intents": [
    {
      "intent": "GetStoreInfo",
      "score": 0.984749258
    },
    {
      "intent": "None",
      "score": 0.2040639
    }
  ],
  "entities": []
}
```

意圖會依最高分到最低分排序。

|資料物件|資料類型|資料位置|值|分數|
|--|--|--|--|:--|
|意圖|字串|intents[0].intent|"GetStoreInfo"|0.984749258|
|意圖|字串|intents[1].intent|"None"|0.0168218873|

如果您新增預先建置的定義域，則意圖名稱除了會指出意圖之外，也會指出該定義域，例如 `Utilties` 或 `Communication`：

```JSON
{
  "query": "Turn on the lights next monday at 9am",
  "topScoringIntent": {
    "intent": "Utilities.ShowNext",
    "score": 0.07842206
  },
  "intents": [
    {
      "intent": "Utilities.ShowNext",
      "score": 0.07842206
    },
    {
      "intent": "Communication.StartOver",
      "score": 0.0239675418
    },
    {
      "intent": "None",
      "score": 0.0168218873
    }],
  "entities": []
}
```

|網域|資料物件|資料類型|資料位置|值|
|--|--|--|--|--|
|公用事業|意圖|字串|intents[0].intent|"<b>Utilities</b>.ShowNext"|
|通訊|意圖|字串|intents[1].intent|<b>Communication</b>.StartOver"|
||意圖|字串|intents[2].intent|"None"|


## <a name="data-from-entities"></a>來自實體的資料
大多數 Chatbot 和應用程式都不僅僅只是需要意圖名稱。 這個額外的選擇性資料來自在語句中探索到的實體。 每個類型的實體會傳回與比對相關的不同資訊。

語句中的單一單字或片語可能會與多個實體相符。 在該情況下，系統會傳回每個相符的實體及其分數。

所有實體都會在端點回應的**實體**陣列中傳回：

```JSON
"entities": [
  {
    "entity": "bob jones",
    "type": "Name",
    "startIndex": 0,
    "endIndex": 8,
    "score": 0.473899543
  },
  {
    "entity": "3",
    "type": "builtin.number",
    "startIndex": 16,
    "endIndex": 16,
    "resolution": {
      "value": "3"
    }
  }
]
```

## <a name="tokenized-entity-returned"></a>傳回的 Token 化實體
有數個[文化特性](luis-language-support.md#tokenization)會傳回 `entity` 值已[Token 化](luis-glossary.md#token)的實體物件。 LUIS 在實體物件中傳回的 startIndex 和 endIndex 不會對應至新的 Token 化值，而是會對應至原始查詢，以便讓您透過程式設計方式擷取原始實體。 

例如，在德文中，`das Bauernbrot` 會經由 Token 化變成 `das bauern brot`。 系統會傳回 Token 化值 `das bauern brot`，而只要透過程式設計方式，即可從原始查詢的 startIndex 和 endIndex 判斷出原始值，將 `das Bauernbrot` 提供給您。

## <a name="simple-entity-data"></a>簡單實體資料

[簡單實體](luis-concept-entity-types.md)是一個機器學習值。 它可以是一個單字或片語。

`Bob Jones wants 3 meatball pho`

在先前的語句中，`Bob Jones` 被標記為簡單的 `Customer` 實體。

從端點傳回的資料會包含實體名稱、從語句探索到的文字，所探索文字的位置，以及分數：

```JSON
"entities": [
  {
  "entity": "bob jones",
  "type": "Customer",
  "startIndex": 0,
  "endIndex": 8,
  "score": 0.473899543
  }
]
```

|資料物件|實體名稱|值|
|--|--|--|
|簡單實體|"Customer"|"bob jones"|

## <a name="hierarchical-entity-data"></a>階層式實體資料

[階層式](luis-concept-entity-types.md)實體是機器學習實體，並可包含單字或片語。 子系會由內容來識別。 如果您要尋找全文相符的父子關係，請使用[清單](#list-entity-data)實體。

`book 2 tickets to paris`

在先前的語句中，`paris` 被標記為 `Location` 階層式實體的 `Location::ToLocation` 子系。

從端點傳回的資料會包含實體名稱和子系名稱、從語句探索到的文字，所探索文字的位置，以及分數：

```JSON
"entities": [
  {
    "entity": "paris",
    "type": "Location::ToLocation",
    "startIndex": 18,
    "endIndex": 22,
    "score": 0.6866132
  }
]
```

|資料物件|父系|子系|值|
|--|--|--|--|--|
|階層式實體|位置|ToLocation|"paris"|

## <a name="composite-entity-data"></a>複合實體資料
[複合](luis-concept-entity-types.md)實體是機器學習實體，並可包含單字或片語。 例如，思考一下與下列語句搭配之預先建置 `number` 與 `Location::ToLocation` 的複合實體：

`book 2 tickets to paris`

請注意，數字 `2` 與 ToLocation `paris` 之間有不屬於任何實體的單字。 [LUIS](luis-reference-regions.md) 網站中用於所標示語句的綠色底線表示複合實體。

![複合實體](./media/luis-concept-data-extraction/composite-entity.png)

複合實體會在 `compositeEntities` 陣列中傳回，而複合項目內的所有實體則會一併在 `entities` 陣列中傳回：

```JSON
  "entities": [
    {
      "entity": "paris",
      "type": "Location::ToLocation",
      "startIndex": 18,
      "endIndex": 22,
      "score": 0.956998169
    },
    {
      "entity": "2",
      "type": "builtin.number",
      "startIndex": 5,
      "endIndex": 5,
      "resolution": {
        "value": "2"
      }
    },
    {
      "entity": "2 tickets to paris",
      "type": "Order",
      "startIndex": 5,
      "endIndex": 22,
      "score": 0.7714499
    }
  ],
  "compositeEntities": [
    {
      "parentType": "Order",
      "value": "2 tickets to paris",
      "children": [
        {
          "type": "builtin.number",
          "value": "2"
        },
        {
          "type": "Location::ToLocation",
          "value": "paris"
        }
      ]
    }
  ]
```    

|資料物件|實體名稱|值|
|--|--|--|
|預先建置的實體 - number|"builtin.number"|"2"|
|階層式實體 - Location|"Location::ToLocation"|"paris"|

## <a name="list-entity-data"></a>清單實體資料

[清單](luis-concept-entity-types.md)實體不是機器學習實體。 它是全文相符的項目。 清單代表清單中的項目及這些項目的同義字。 LUIS 會將與任何清單中項目相符的項目，在回應中標示為實體。 一個同義字可以出現在多個清單中。

假設應用程式具有一個名為 `Cities` 的清單，其中可允許城市名稱的各種變化，包括機場城市 (Sea-tac)、機場代碼 (SEA)、郵遞區號 (98101) 及電話區碼 (206)。

|清單項目|項目同義字|
|---|---|
|Seattle|sea-tac、sea、98101、206、+1 |
|巴黎|cdg、roissy、ory、75001、1、+33|

`book 2 tickets to paris`

在上述語句中，`paris` 一字會對應至 `Cities` 清單實體中的 paris 項目。 此清單實體會同時比對項目的正規化名稱及項目同義字。

```JSON
"entities": [
  {
    "entity": "paris",
    "type": "Cities",
    "startIndex": 18,
    "endIndex": 22,
    "resolution": {
      "values": [
        "Paris"
      ]
    }
  }
]
```

使用 Paris 同義字的另一個範例語句：

`book 2 tickets to roissy`

```JSON
"entities": [
  {
    "entity": "roissy",
    "type": "Cities",
    "startIndex": 18,
    "endIndex": 23,
    "resolution": {
      "values": [
        "Paris"
      ]
    }
  }
]
```

## <a name="prebuilt-entity-data"></a>預先建置的實體資料
探索[預先建置](luis-concept-entity-types.md)實體時，會使用開放原始碼 [Recognizers-Text](https://github.com/Microsoft/Recognizers-Text) 專案，根據規則運算式比對進行探索。 預先建置的實體會在實體陣列中傳回，並使用前面加上 `builtin::` 的類別名稱。 以下文字是一個範例語句，其中含有所傳回的預先建置實體：

`Dec 5th send to +1 360-555-1212`

```JSON
"entities": [
    {
      "entity": "dec 5th",
      "type": "builtin.datetimeV2.date",
      "startIndex": 0,
      "endIndex": 6,
      "resolution": {
        "values": [
          {
            "timex": "XXXX-12-05",
            "type": "date",
            "value": "2017-12-05"
          },
          {
            "timex": "XXXX-12-05",
            "type": "date",
            "value": "2018-12-05"
          }
        ]
      }
    },
    {
      "entity": "1",
      "type": "builtin.number",
      "startIndex": 18,
      "endIndex": 18,
      "resolution": {
        "value": "1"
      }
    },
    {
      "entity": "360",
      "type": "builtin.number",
      "startIndex": 20,
      "endIndex": 22,
      "resolution": {
        "value": "360"
      }
    },
    {
      "entity": "555",
      "type": "builtin.number",
      "startIndex": 26,
      "endIndex": 28,
      "resolution": {
        "value": "555"
      }
    },
    {
      "entity": "1212",
      "type": "builtin.number",
      "startIndex": 32,
      "endIndex": 35,
      "resolution": {
        "value": "1212"
      }
    },
    {
      "entity": "5th",
      "type": "builtin.ordinal",
      "startIndex": 4,
      "endIndex": 6,
      "resolution": {
        "value": "5"
      }
    },
    {
      "entity": "1 360 - 555 - 1212",
      "type": "builtin.phonenumber",
      "startIndex": 18,
      "endIndex": 35,
      "resolution": {
        "value": "1 360 - 555 - 1212"
      }
    }
  ]
```

## <a name="regular-expression-entity-data"></a>規則運算式實體資料
探索[規則運算式](luis-concept-entity-types.md)實體時，會使用您建立實體時所提供的運算式，根據規則運算式比對進行探索。 使用 `kb[0-9]{6}` 作為規則運算式實體定義時，以下 JSON 回應是一個範例語句，其中含有 `When was kb123456 published?` 查詢的所傳回規則運算式實體：

```JSON
{
  "query": "when was kb123456 published?",
  "topScoringIntent": {
    "intent": "FindKBArticle",
    "score": 0.933641255
  },
  "intents": [
    {
      "intent": "FindKBArticle",
      "score": 0.933641255
    },
    {
      "intent": "None",
      "score": 0.04397359
    }
  ],
  "entities": [
    {
      "entity": "kb123456",
      "type": "KB number",
      "startIndex": 9,
      "endIndex": 16
    }
  ]
}
```

## <a name="extracting-names"></a>擷取名稱
從語句中取得名稱相當困難，因為名稱幾乎可以是字母與單字的任何組合。 視所要擷取的名稱類型而定，您會有數個選項。 這些並非規則，而比較像是指導方針。

### <a name="names-of-people"></a>人名
人名可依據語言和文化特性而有些微的格式。 請使用以名字和姓氏作為子系的階層式實體，或使用含有名字和姓氏角色的簡單實體。 請務必提供在語句的不同部分、在不同長度的語句中及在所有意圖 (包括 None 意圖) 的語句中使用名字和姓氏的範例。 請定期[檢閱](luis-how-to-review-endoint-utt.md)端點語句，以標記任何未正確預測的名稱。

### <a name="names-of-places"></a>地名
地名是已設定且已知的名稱，例如城市、郡縣、州、省及國家/地區。 如果您的應用程式使用一組已知的地點，請考慮使用清單實體。 如果您需要尋找所有地名，請建立一個簡單實體，然後提供各種範例。 請新增一個地名片語清單來強調地名在您應用程式中看起來的樣子。 請定期[檢閱](luis-how-to-review-endoint-utt.md)端點語句，以標記任何未正確預測的名稱。

### <a name="new-and-emerging-names"></a>全新和新興的名稱
有些應用程式需要能夠尋找全新和新興的名稱，例如產品或公司。 這是難度最高類型的資料擷取。 請從簡單實體開始著手，並新增一個片語清單。 請定期[檢閱](luis-how-to-review-endoint-utt.md)端點語句，以標記任何未正確預測的名稱。

## <a name="pattern-roles-data"></a>模式角色資料
角色是實體的內容相關差異。

```JSON
{
  "query": "move bob jones from seattle to redmond",
  "topScoringIntent": {
    "intent": "MoveAssetsOrPeople",
    "score": 0.9999998
  },
  "intents": [
    {
      "intent": "MoveAssetsOrPeople",
      "score": 0.9999998
    },
    {
      "intent": "None",
      "score": 1.02040713E-06
    },
    {
      "intent": "GetEmployeeBenefits",
      "score": 6.12244548E-07
    },
    {
      "intent": "GetEmployeeOrgChart",
      "score": 6.12244548E-07
    },
    {
      "intent": "FindForm",
      "score": 1.1E-09
    }
  ],
  "entities": [
    {
      "entity": "bob jones",
      "type": "Employee",
      "startIndex": 5,
      "endIndex": 13,
      "score": 0.922820568,
      "role": ""
    },
    {
      "entity": "seattle",
      "type": "Location",
      "startIndex": 20,
      "endIndex": 26,
      "score": 0.948008537,
      "role": "Origin"
    },
    {
      "entity": "redmond",
      "type": "Location",
      "startIndex": 31,
      "endIndex": 37,
      "score": 0.7047979,
      "role": "Destination"
    }
  ]
}
```

## <a name="patternany-entity-data"></a>Pattern.any 實體資料
Pattern.any 實體是在[模式](luis-concept-patterns.md)範本語句中使用的可變長度實體。

```JSON
{
  "query": "where is the form Understand your responsibilities as a member of the community and who needs to sign it after I read it?",
  "topScoringIntent": {
    "intent": "FindForm",
    "score": 0.999999464
  },
  "intents": [
    {
      "intent": "FindForm",
      "score": 0.999999464
    },
    {
      "intent": "GetEmployeeBenefits",
      "score": 4.883697E-06
    },
    {
      "intent": "None",
      "score": 1.02040713E-06
    },
    {
      "intent": "GetEmployeeOrgChart",
      "score": 9.278342E-07
    },
    {
      "intent": "MoveAssetsOrPeople",
      "score": 9.278342E-07
    }
  ],
  "entities": [
    {
      "entity": "understand your responsibilities as a member of the community",
      "type": "FormName",
      "startIndex": 18,
      "endIndex": 78,
      "role": ""
    }
  ]
}
```


## <a name="sentiment-analysis"></a>情感分析
如果已設定情感分析，LUIS JSON 回應就會包含情感分析。 若要深入了解情感分析，請參閱[文字分析](https://docs.microsoft.com/azure/cognitive-services/text-analytics/)文件。

### <a name="sentiment-data"></a>情感資料
情感資料是一個介於 1 與 0 之間的分數，指出資料的正面 (較接近 1) 或負面 (較接近 0) 情感。

當文化特性為 `en-us` 時，回應為：

```JSON
"sentimentAnalysis": {
  "label": "positive",
  "score": 0.9163064
}
```

針對所有其他文化特性，回應為：

```JSON
"sentimentAnalysis": {
  "score": 0.9163064
}
```


### <a name="key-phrase-extraction-entity-data"></a>關鍵片語擷取實體資料
關鍵片語擷取實體會傳回語句中[文字分析](https://docs.microsoft.com/azure/cognitive-services/text-analytics/)所提供的關鍵片語。

```JSON
{
  "query": "Is there a map of places with beautiful views on a favorite trail?",
  "topScoringIntent": {
    "intent": "GetJobInformation",
    "score": 0.764368951
  },
  "intents": [
    ...
  ],
  "entities": [
    {
      "entity": "beautiful views",
      "type": "builtin.keyPhrase",
      "startIndex": 30,
      "endIndex": 44
    },
    {
      "entity": "map of places",
      "type": "builtin.keyPhrase",
      "startIndex": 11,
      "endIndex": 23
    },
    {
      "entity": "favorite trail",
      "type": "builtin.keyPhrase",
      "startIndex": 51,
      "endIndex": 64
    }
  ]
}
```

## <a name="data-matching-multiple-entities"></a>與多個實體相符的資料
LUIS 會傳回在語句中探索到的所有實體。 因此，您的 Chatbot 可能需要根據結果進行決策。 一個語句可以包含許多實體：

`book me 2 adult business tickets to paris tomorrow on air france`

LUIS 端點可以探索不同實體中的相同資料：

```JSON
{
  "query": "book me 2 adult business tickets to paris tomorrow on air france",
  "topScoringIntent": {
    "intent": "BookFlight",
    "score": 1.0
  },
  "intents": [
    {
      "intent": "BookFlight",
      "score": 1.0
    },
    {
      "intent": "Concierge",
      "score": 0.04216196
    },
    {
      "intent": "None",
      "score": 0.03610297
    }
  ],
  "entities": [
    {
      "entity": "air france",
      "type": "Airline",
      "startIndex": 54,
      "endIndex": 63,
      "score": 0.8291798
    },
    {
      "entity": "adult",
      "type": "Category",
      "startIndex": 10,
      "endIndex": 14,
      "resolution": {
        "values": [
          "adult"
        ]
      }
    },
    {
      "entity": "paris",
      "type": "Cities",
      "startIndex": 36,
      "endIndex": 40,
      "resolution": {
        "values": [
          "Paris"
        ]
      }
    },
    {
      "entity": "tomorrow",
      "type": "builtin.datetimeV2.date",
      "startIndex": 42,
      "endIndex": 49,
      "resolution": {
        "values": [
          {
            "timex": "2018-02-21",
            "type": "date",
            "value": "2018-02-21"
          }
        ]
      }
    },
    {
      "entity": "paris",
      "type": "Location::ToLocation",
      "startIndex": 36,
      "endIndex": 40,
      "score": 0.9730773
    },
    {
      "entity": "2",
      "type": "builtin.number",
      "startIndex": 8,
      "endIndex": 8,
      "resolution": {
        "value": "2"
      }
    },
    {
      "entity": "business",
      "type": "Seat",
      "startIndex": 16,
      "endIndex": 23,
      "resolution": {
        "values": [
          "business"
        ]
      }
    },
    {
      "entity": "2 adult business",
      "type": "TicketSeatOrder",
      "startIndex": 8,
      "endIndex": 23,
      "score": 0.8784727
    }
  ],
  "compositeEntities": [
    {
      "parentType": "TicketSeatOrder",
      "value": "2 adult business",
      "children": [
        {
          "type": "Category",
          "value": "adult"
        },
        {
          "type": "builtin.number",
          "value": "2"
        },
        {
          "type": "Seat",
          "value": "business"
        }
      ]
    }
  ]
}
```

## <a name="next-steps"></a>後續步驟

請參閱[新增實體](luis-how-to-add-entities.md)，以深入了解如何將實體新增至 LUIS 應用程式。
