---
title: 情感認知搜尋技能 (Azure 搜尋服務) | Microsoft Docs
description: 在 Azure 搜尋服務擴充管線中的文字擷取情感。
services: search
manager: pablocas
author: luiscabrer
ms.service: search
ms.devlang: NA
ms.workload: search
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.date: 05/01/2018
ms.author: luisca
ms.openlocfilehash: 1e4028c3a810de41efe217e6dd4347fc3bc6bf16
ms.sourcegitcommit: 1b561b77aa080416b094b6f41fce5b6a4721e7d5
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/17/2018
ms.locfileid: "45730879"
---
#   <a name="sentiment-cognitive-skill"></a>情感認知技能

**情感**技能會依循正負面連續性評估非結構化文字，並且針對每個記錄傳回介於 0 與 1 之間的數值分數。 接近 1 的分數表示正面情感；接近 0 的分數表示負面情感。

> [!NOTE]
> 認知搜尋目前為公開預覽狀態。 技能集執行、映像擷取及正規化目前為免費提供。 我們將在不久後宣布這些功能的定價。 

## <a name="odatatype"></a>@odata.type  
Microsoft.Skills.Text.SentimentSkill

## <a name="data-limits"></a>資料限制
記錄的大小上限應該是 5000 個字元 (以 `String.Length` 為測量單位)。 如果您需要先分割資料，再將該資料傳送至情感分析器，請使用[文字分割技能](cognitive-search-skill-textsplit.md)。


## <a name="skill-parameters"></a>技能參數

這些參數會區分大小寫。

| 參數名稱 |                      |
|----------------|----------------------|
| defaultLanguageCode | (選用) 要套用至未明確指定語言之文件的語言代碼。 <br/> 請參閱[支援語言的完整清單](../cognitive-services/text-analytics/text-analytics-supported-languages.md) |

## <a name="skill-inputs"></a>技能輸入 

| 輸入名稱 | 說明 |
|--------------------|-------------|
| text | 要分析的文字。|
| languageCode  |  (選用) 此字串表示記錄的語言。 如果未指定此屬性，則預設值為 "en"。 <br/>請參閱[支援語言的完整清單](../cognitive-services/text-analytics/text-analytics-supported-languages.md)。|

## <a name="skill-outputs"></a>技能輸出

| 輸出名稱 | 說明 |
|--------------------|-------------|
| 分數 | 介於 0 與 1 之間的值代表所分析文字的情感。 接近 0 的值具有負面情感、接近 0.5 的值具有中性情感，而接近 1 的值具有正面情感。|


##  <a name="sample-definition"></a>範例定義

```json
{
    "@odata.type": "#Microsoft.Skills.Text.SentimentSkill",
    "inputs": [
        {
            "name": "text",
            "source": "/document/content"
        },
        {
            "name": "languageCode",
            "source": "/document/languagecode"
        }
    ],
    "outputs": [
        {
            "name": "score",
            "targetName": "mySentiment"
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
            "data": {
                "text": "I had a terrible time at the hotel. The staff was rude and the food was awful.",
                "languageCode": "en"
            }
        }
    ]
}
```


##  <a name="sample-output"></a>範例輸出

```json
{
    "values": [
        {
            "recordId": "1",
            "data": {
                "score": 0.01
            }
        }
    ]
}
```

## <a name="notes"></a>注意
如果是空的，這些記錄不會傳回情感分數。

## <a name="error-cases"></a>錯誤案例
如果不支援語言，則會產生錯誤，並且不會傳回情感分數。

## <a name="see-also"></a>另請參閱

+ [預先定義的技能](cognitive-search-predefined-skills.md)
+ [如何定義技能集](cognitive-search-defining-skillset.md) (英文)