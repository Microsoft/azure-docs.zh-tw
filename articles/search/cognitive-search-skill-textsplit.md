---
title: 文字分割認知搜尋技能 (Azure 搜尋服務) | Microsoft Docs
description: 您可以根據 Azure 搜尋服務擴充管線中的長度，將文字分成文字之區塊或頁面。
services: search
manager: pablocas
author: luiscabrer
ms.service: search
ms.devlang: NA
ms.workload: search
ms.topic: conceptual
ms.date: 05/01/2018
ms.author: luisca
ms.openlocfilehash: dbb9261cfce0a8437cfe76121fa16aa87c4b3393
ms.sourcegitcommit: e221d1a2e0fb245610a6dd886e7e74c362f06467
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 05/07/2018
ms.locfileid: "33786797"
---
#   <a name="text-split-cognitive-skill"></a>文字分割認知技能

**文字分割**技能將文字分成文字區塊。 您可以指定是否想要將文字分成句子或特定長度的頁面。 如果其他技能下游有最大文字長度需求，此技能特別有用。 

## <a name="odatatype"></a>@odata.type  
Microsoft.Skills.Text.SplitSkill 

## <a name="skill-parameters"></a>技能參數

這些參數會區分大小寫。

| 參數名稱     | 說明 |
|--------------------|-------------|
| textSplitMode      | 「頁面」或「句子」 | 
| maximumPageLength | 如果 textSplitMode 設為「頁面」，這是指由 `String.Length` 測量的最大頁面長度。 最小值為 100。 | 
| defaultLanguageCode   | (選用) 以下其中一個語言代碼：`da, de, en, es, fi, fr, it, ko, pt`。 預設值是英文 (en)。 幾點考量事項：<ul><li>如果您傳遞的是 languagecode-countrycode 格式，則只會使用該格式的 languagecode 部分。</li><li>如果語言不在前面的清單中，分割技能會在字元界限拆分文字。</li><li>提供語言代碼有助於避免將無空格語言 (例如中文、日文和韓文) 的字組切成一半。</li></ul>  |


## <a name="skill-inputs"></a>技能輸入

| 參數名稱       | 說明      |
|----------------------|------------------|
| text  | 要分割成子字串的文字。 |
| languageCode  | (選用) 文件的語言代碼。  |

## <a name="skill-outputs"></a>技能輸出 

| 參數名稱     | 說明 |
|--------------------|-------------|
| textItems | 已擷取的子字串陣列。 |


##  <a name="sample-definition"></a>範例定義

```json
{
    "@odata.type": "#Microsoft.Skills.Text.SplitSkill",
    "textSplitMode" : "pages", 
    "maximumPageLength": 1000,
    "defaultLanguageCode": "en",
    "inputs": [
        {
            "name": "text",
            "source": "/document/content"
        },
        {
            "name": "languageCode",
            "source": "/document/language"
        }
    ],
    "outputs": [
        {
            "name": "textItems",
            "targetName": "mypages"
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
                "text": "This is a the loan application for Joe Romero, he is a Microsoft employee who was born in Chile and then moved to Australia…",
                "languageCode": "en"
            }
        },
        {
            "recordId": "2",
            "data": {
                "text": "This is the second document, which will be broken into several pages...",
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
                "textItems": [
                    "This is the loan…",
                    "On the second page we…"
                ]
            }
        },
        {
            "recordId": "2",
            "data": {
                "textItems": [
                    "This is the second document...",
                    "On the second page of the second doc…"
                ]
            }
        }
    ]
}
```

## <a name="error-cases"></a>錯誤案例
如果不支援某語言，則會產生警告，而且會在字元界限分割文字。

## <a name="see-also"></a>另請參閱

+ [預先定義的技能](cognitive-search-predefined-skills.md) (英文)
+ [如何定義技能集](cognitive-search-defining-skillset.md) (英文)
