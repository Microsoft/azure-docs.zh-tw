---
title: 文字合併認知搜尋技能 (Azure 搜尋服務) | Microsoft Docs
description: 將一組欄位中的文字合併成一個合併欄位。 在 Azure 搜尋服務擴充管線中使用這項認知技能。
services: search
manager: pablocas
author: luiscabrer
ms.service: search
ms.devlang: NA
ms.workload: search
ms.topic: conceptual
ms.date: 05/01/2018
ms.author: luisca
ms.openlocfilehash: 5387eeacc78875ac0f38f96a6c83fb3f5791775e
ms.sourcegitcommit: c282021dbc3815aac9f46b6b89c7131659461e49
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/12/2018
ms.locfileid: "49167609"
---
#    <a name="text-merge-cognitive-skill"></a>文字合併認知技能

**文字合併**技能可將一組欄位中的文字合併成單一欄位。 

> [!NOTE]
> 認知搜尋目前為公開預覽狀態。 技能集執行、映像擷取及正規化目前為免費提供。 我們將在不久後宣告這些功能的定價。 

## <a name="odatatype"></a>@odata.type  
Microsoft.Skills.Text.MergeSkill

## <a name="skill-parameters"></a>技能參數

這些參數會區分大小寫。

| 參數名稱     | 說明 |
|--------------------|-------------|
| insertPreTag  | 要在每次插入之前包含的字串。 預設值為 `" "`。 若要省略空間，請將值設定為 `""`。  |
| insertPostTag | 要在每次插入之後包含的字串。 預設值為 `" "`。 若要省略空間，請將值設定為 `""`。  |


##  <a name="sample-input"></a>範例輸入
提供此技能之有用輸入的 JSON 文件可以是：

```json
{
    "values": [
      {
        "recordId": "1",
        "data":
           {
             "text": "The brown fox jumps over the dog" ,
             "itemsToInsert": ["quick", "lazy"],
             "offsets": [3, 28],
           }
      }
    ]
}
```

##  <a name="sample-output"></a>範例輸出
這個範例顯示前一個輸入的輸出，並假設 *insertPreTag* 設定為 `" "`，且 *insertPostTag* 設定為 `""`。 

```json
{
    "values": [
      {
        "recordId": "1",
        "data":
           {
             "mergedText": "The quick brown fox jumps over the lazy dog" 
           }
      }
    ]
}
```

## <a name="extended-sample-skillset-definition"></a>擴充的範例技能集定義

使用文字合併的常見案例是將影像的文字表示 (來自 OCR 技能的文字，或影像標題) 合併至文件的內容欄位。 

下列範例技能集使用 OCR 技能來擷取文件中內嵌影像的文字。 接下來，它會建立 *merged_text* 欄位以包含每個影像的原始和已進行 OCR 處理的文字。 

```json
{
  "description": "Extract text from images and merge with content text to produce merged_text",
  "skills":
  [
    {
        "name": "OCR skill",
        "description": "Extract text (plain and structured) from image.",
        "@odata.type": "#Microsoft.Skills.Vision.OcrSkill",
        "context": "/document/normalized_images/*",
        "defaultLanguageCode": "en",
        "detectOrientation": true,
        "inputs": [
          {
            "name": "image",
            "source": "/document/normalized_images/*"
          }
        ],
        "outputs": [
          {
            "name": "text"
          }
        ]
    },
    {
      "@odata.type": "#Microsoft.Skills.Text.MergeSkill",
      "description": "Create merged_text, which includes all the textual representation of each image inserted at the right location in the content field.",
      "context": "/document",
      "insertPreTag": " ",
      "insertPostTag": " "
      "inputs": [
        {
          "name":"text", "source": "/document/content"
        },
        {
          "name": "itemsToInsert", "source": "/document/normalized_images/*/text"
        },
        {
          "name":"offsets", "source": "/document/normalized_images/*/contentOffset" 
        }
      ],
      "outputs": [
        {
          "name": "mergedText", "targetName" : "merged_text"
        }
      ]
    }
  ]
}
```
上述範例假設存在正規化影像欄位。 若要產生正規化影像欄位，請將索引子定義中的 *imageAction* 設定設定為 *generateNormalizedImages*，如下所示：

```json
{  
   //...rest of your indexer definition goes here ... 
  "parameters":{  
      "configuration":{  
         "dataToExtract":"contentAndMetadata",
         "imageAction":"generateNormalizedImages"
      }
   }
}
```

## <a name="see-also"></a>另請參閱

+ [預先定義的技能](cognitive-search-predefined-skills.md)
+ [如何定義技能集](cognitive-search-defining-skillset.md) (英文)
+ [建立索引子 (REST)](https://docs.microsoft.com/rest/api/searchservice/create-indexer)
