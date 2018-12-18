---
title: 認知搜尋管線中自訂技能的介面定義 (Azure 搜尋服務) | Microsoft Docs
description: Azure 搜尋服務的認知搜尋管線中，web-api 自訂技能的自訂資料擷取介面。
manager: pablocas
author: luiscabrer
services: search
ms.service: search
ms.devlang: NA
ms.topic: conceptual
ms.date: 08/14/2018
ms.author: luisca
ms.openlocfilehash: 2218a96702a02a32df18da9640ea9946d05acdb1
ms.sourcegitcommit: 1aedb52f221fb2a6e7ad0b0930b4c74db354a569
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/17/2018
ms.locfileid: "42140884"
---
# <a name="how-to-add-a-custom-skill-to-a-cognitive-search-pipeline"></a>如何將自訂技能新增至認知搜尋管線

Azure 搜尋服務中的[認知搜尋索引管線](cognitive-search-concept-intro.md)可從[預先定義的技能](cognitive-search-predefined-skills.md)以及自行建立並新增至管線的自訂技能來組合。 在本文中，了解如何建立自訂技能，公開介面以讓它包含在認知搜尋管線中。 

建置自訂技能可讓您插入內容獨有的轉換。 自訂技能會獨立執行，可套用在任何所需的擴充步驟。 例如，您可以定義欄位特定的自訂實體、建立自訂的分類模型以區分商務和財務合約和文件，或者新增語音辨識技能以深入觸及音訊檔案來了解相關內容。 如需逐步範例，請參閱[範例：建立自訂技能](cognitive-search-create-custom-skill-example.md)。

 無論您需要哪一種自訂功能，都有一個簡單且清楚的介面，可將自訂技能連接到擴充管線的其餘部分。 要包含[技能集](cognitive-search-defining-skillset.md)中的唯一需求，便是以在技能集內可整體取用的方式，接受輸入並發出輸出的能力。 本文著重於擴充管線所需的輸入和輸出格式。

## <a name="web-api-custom-skill-interface"></a>Web API 自訂技能介面

自訂 WebAPI 技能端點必須在 5 分鐘的時段內傳回回應。 索引管線是同步的，且如果未在該時段內收到回應，則索引會產生逾時錯誤。

目前與自訂技能互動的唯一機制是透過 Web API 介面。 Web API 的需求必須符合本節所描述的需求。

### <a name="1--web-api-input-format"></a>1.Web API 輸入格式

Web API 必須接受要處理的記錄陣列。 每一筆記錄都必須包含「屬性包」，也就是提供給 Web API 的輸入。 

假設您要建立簡單的擴充程式，以識別合約文字中提及的第一個日期。 在此範例中，技能接受單一輸入 *contractText* 做為合約文字。 技能也有單一輸出，也就是合約的日期。 為了讓擴充程式更有趣，以多部分複雜類型的形式來傳回此 *contractDate*。

您的 Web API 應該準備好接收一批輸入記錄。 *values* 陣列的每個成員都代表特定記錄的輸入。 每筆記錄都必須有下列元素：

+ *recordId* 成員，此為特定記錄的唯一識別碼。 當擴充程式傳回結果時，必須提供此 *recordId*，才能讓呼叫者比對記錄結果與輸入。

+ *data* 成員，基本上是每筆記錄的輸入欄位包。

為了更具體，根據上面的範例，您的 Web API 應該會預期如下的要求：

```json
{
    "values": [
      {
        "recordId": "a1",
        "data":
           {
             "contractText": 
                "This is a contract that was issues on November 3, 2017 and that involves... "
           }
      },
      {
        "recordId": "b5",
        "data":
           {
             "contractText": 
                "In the City of Seattle, WA on February 5, 2018 there was a decision made..."
           }
      },
      {
        "recordId": "c3",
        "data":
           {
             "contractText": null
           }
      }
    ]
}
```
事實上，您的服務可能會有數百筆或數千筆記錄的呼叫，而不是只像下面所顯示的三筆記錄。

### <a name="2-web-api-output-format"></a>2.Web API 輸出格式

輸出的格式是一組包含 *recordId* 和屬性包的記錄。 

```json
{
  "values": 
  [
      {
        "recordId": "b5",
        "data" : 
        {
            "contractDate":  { "day" : 5, "month": 2, "year" : 2018 }
        }
      },
      {
        "recordId": "a1",
        "data" : {
            "contractDate": { "day" : 3, "month": 11, "year" : 2017 }                    
        }
      },
      {
        "recordId": "c3",
        "data" : 
        {
        },
        "errors": [ { "message": "contractText field required "}   ],  
        "warnings": [ {"message": "Date not found" }  ]
      }
    ]
}
```

這個特定的範例只有一個輸出，但您可以輸出多個屬性。 

### <a name="errors-and-warning"></a>錯誤和警告

如上述範例所示，您可以針對每一筆記錄傳回錯誤和警告訊息。

## <a name="consuming-custom-skills-from-skillset"></a>取用技能集的自訂技能

當您建立 Web API 擴充程式時，可以在要求中描述 HTTP 標頭和參數。 以下程式碼片段示範如何在技能集定義中描述要求參數和 HTTP 標頭。

```json
{
    "skills": [
      {
        "@odata.type": "#Microsoft.Skills.Custom.WebApiSkill",
        "description": "This skill calls an Azure function, which in turn calls TA sentiment",
        "uri": "https://indexer-e2e-webskill.azurewebsites.net/api/DateExtractor?language=en",
        "context": "/document",
        "httpHeaders": {
            "DateExtractor-Api-Key": "foo"
        },
        "inputs": [
          {
            "name": "contractText",
            "source": "/document/content"
          }
        ],
        "outputs": [
          {
            "name": "contractDate",
            "targetName": "date"
          }
        ]
      }
  ]
}
```

## <a name="next-steps"></a>後續步驟

+ [範例：建立翻譯文字 API 的自訂技能](cognitive-search-create-custom-skill-example.md)
+ [如何定義技能集](cognitive-search-defining-skillset.md)
+ [建立技能集 (REST)](https://docs.microsoft.com/rest/api/searchservice/create-skillset)
+ [如何對應擴充的欄位](cognitive-search-output-field-mapping.md)
