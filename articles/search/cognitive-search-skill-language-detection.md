---
title: 語言偵測認知搜尋技能 (Azure 搜尋服務) | Microsoft Docs
description: 評估非結構化文字，並針對每筆記錄傳回語言識別項，其分數表示 Azure 搜尋服務擴充管線中分析的強度。
services: search
manager: pablocas
author: luiscabrer
ms.service: search
ms.devlang: NA
ms.workload: search
ms.topic: conceptual
ms.date: 05/01/2018
ms.author: luisca
ms.openlocfilehash: 443ac895085053b7c4c876c3deecaa1943c9f506
ms.sourcegitcommit: 4e5ac8a7fc5c17af68372f4597573210867d05df
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/20/2018
ms.locfileid: "39171053"
---
#   <a name="language-detection-cognitive-skill"></a>語言偵測認知技能

**語言偵測**技能會偵測輸入文字的語言，並針對要求所提交的每份文件回報單一語言代碼，最多可達 120 種語言。 語言代碼各配有一個分數，表示分析的強度。

當您需要提供文字的語言作為其他技能的輸入時 (例如[情感分析技能](cognitive-search-skill-sentiment.md)或[文字分割技能](cognitive-search-skill-textsplit.md))，這項功能就特別有用。

## <a name="odatatype"></a>@odata.type  
Microsoft.Skills.Text.LanguageDetectionSkill

## <a name="data-limits"></a>資料限制
記錄的大小上限應該是 50,000 個字元 (以 `String.Length` 為測量單位)。 如果您需要先分割資料，再將該資料傳送至情感分析器，您可以使用[文字分割技能](cognitive-search-skill-textsplit.md)。

## <a name="skill-inputs"></a>技能輸入

這些參數會區分大小寫。

| 輸入     | 說明 |
|--------------------|-------------|
| text | 要分析的文字。|

## <a name="skill-outputs"></a>技能輸出

| 輸出名稱    | 說明 |
|--------------------|-------------|
| languageCode | 所識別語言的 ISO 6391 語言代碼。 例如，"en"。 |
| languageName | 語言的名稱。 例如，"English"。 |
| 分數 | 0 到 1 之間的值。 正確識別語言的可能性。 如果句子有混合的語言，分數可能會小於 1。  |

##  <a name="sample-definition"></a>範例定義

```json
 {
    "@odata.type": "#Microsoft.Skills.Text.LanguageDetectionSkill ",
    "inputs": [
      {
        "name": "text",
        "source": "/document/text"
      }
    ],
    "outputs": [
      {
        "name": "languageCode",
        "targetName": "myLanguageCode"
      },
      {
        "name": "languageName",
        "targetName": "myLanguageName"
      },
      {
        "name": "score",
        "targetName": "myLanguageScore"
      }

    ]
  }
```

##  <a name="sample-input"></a>範例輸入

```json
{
    "values": [
      {
        "recordId": "1",
        "data":
           {
             "text": "Glaciers are huge rivers of ice that ooze their way over land, powered by gravity and their own sheer weight. "
           }
      },
      {
        "recordId": "2",
        "data":
           {
             "text": "Estamos muy felices de estar con ustedes."
           }
      }
    ]
```


##  <a name="sample-output"></a>範例輸出

```json
{
    "values": [
      {
        "recordId": "1",
        "data":
            {
              "languageCode": "en",
              "languageName": "English",
              "score": 1,
            }
      },
      {
        "recordId": "2",
        "data":
            {
              "languageCode": "es",
              "languageName": "Spanish",
              "score": 1,
            }
      }
    ]
}
```


## <a name="error-cases"></a>錯誤案例
如果文字以不支援的語言表示，則會產生錯誤，而且不會傳回語言識別項。

## <a name="see-also"></a>另請參閱

+ [預先定義的技能](cognitive-search-predefined-skills.md)
+ [如何定義技能集](cognitive-search-defining-skillset.md) (英文)
