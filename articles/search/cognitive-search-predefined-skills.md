---
title: 預先定義的資料擷取、自然語言、影像處理技能 (Azure 搜尋服務) | Microsoft Docs
description: 資料擷取、自然語言、影像處理認知等技能會在 Azure 搜尋服務管線中對原始內容新增語意和結構。
manager: pablocas
author: luiscabrer
ms.service: search
ms.devlang: NA
ms.topic: conceptual
ms.date: 05/01/2018
ms.author: luisca
ms.openlocfilehash: 870cf9629c7af8faee0ce5709199b64910b27ffb
ms.sourcegitcommit: e221d1a2e0fb245610a6dd886e7e74c362f06467
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 05/07/2018
ms.locfileid: "33786697"
---
# <a name="predefined-skills-for-content-enrichment-azure-search"></a>內容擴充的預先定義技能 (Azure 搜尋服務)

在本文中，您會了解認知搜尋所提供的認知技能。 *認知技能*是一種以某種方式轉換內容的作業。 它通常是一種擷取資料或推斷結構的元件，因此能提高我們對輸入資料的了解。 輸出幾乎都是以文字為基礎的。 *技能集*是定義擴充管線的技能集合。 

## <a name="predefined-skills"></a>預先定義的技能

有數種技能所取用或產生的內容都很具彈性。 一般而言，大部分的技能乃是依據預先定型的模型，這表示您無法使用自己的定型資料來定型模型。 如需建立自訂技能的指引，請參閱[如何定義自訂介面](cognitive-search-custom-skill-interface.md)和[範例：建立自訂技能](cognitive-search-create-custom-skill-example.md)。 下表列舉並說明 Microsoft 所提供的技能。 

| 技能 | 說明 |
|-------|-------------|
| [Microsoft.Skills.Text.KeyPhraseSkill](cognitive-search-skill-keyphrases.md) | 這項技能使用預先定型的模型，根據字詞位置、語言規則、與其他字詞的鄰近程度，以及字詞在來源資料中是否尋常來偵測重要詞彙。 |
| [Microsoft.Skills.Text.LanguageDetectionSkill](cognitive-search-skill-language-detection.md)  | 這項技能使用預先定型的模型，偵測所使用的語言 (每個文件一個語言識別碼)。 當相同的文字區段內使用多種語言時，輸出是主要使用語言的 LCID。|
| [Microsoft.Skills.Text.MergerSkill](cognitive-search-skill-textmerger.md) | 將一組欄位中的文字合併成單一欄位。  |
| [Microsoft.Skills.Text.NamedEntityRecognitionSkill](cognitive-search-skill-named-entity-recognition.md) | 這項技能使用預先定型的模型，為一組固定的分類 (人員、位置和組織) 建立實體。 |
| [Microsoft.Skills.Text.SentimentSkill](cognitive-search-skill-sentiment.md)  | 這項技能使用預先定型的模型，為每一筆記錄評定正分或負分的人氣。 分數介於 0 到 1 之間。 當無法偵測到人氣時的 Null 案例，以及對於視為中性的文字，兩者都會發生中性的分數。  |
| [Microsoft.Skills.Text.SplitSkill](cognitive-search-skill-textsplit.md) | 將文字分成多個頁面，讓您能夠以遞增方式來擴充或增加內容。 |
| [Microsoft.Skills.Vision.ImageAnalysisSkill](cognitive-search-skill-image-analysis.md) | 這項技能使用影像偵測演算法，以識別影像的內容並產生文字描述。 |
| [Microsoft.Skills.Vision.OcrSkill](cognitive-search-skill-ocr.md) | 光學字元辨識。 |
| [Microsoft.Skills.Util.ShaperSkill](cognitive-search-skill-shaper.md) | 將輸出對應到複雜類型 (多部分的資料類型，可用於全名、多行地址，或姓氏與個人識別碼的組合)。 |

## <a name="see-also"></a>另請參閱

+ [如何定義技能集](cognitive-search-defining-skillset.md)
+ [自訂技能介面定義](cognitive-search-custom-skill-interface.md)
+ [教學課程：擴充認知搜尋的編製索引](cognitive-search-tutorial-blob.md)