---
title: 認知搜尋文件資源 (Azure 搜尋服務) | Microsoft Docs
description: 與 Azure 搜尋服務中之認知搜尋工作負載相關的文章、教學課程、範例和部落格文章的標註清單。
services: search
manager: cgronlun
author: HeidiSteen
ms.service: search
ms.devlang: NA
ms.topic: conceptual
ms.date: 05/04/2018
ms.author: heidist
ms.openlocfilehash: bad73d2bf8bdf4adee4ce6e40550e506a35a2083
ms.sourcegitcommit: c282021dbc3815aac9f46b6b89c7131659461e49
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/12/2018
ms.locfileid: "49165660"
---
# <a name="documentation-resources-for-cognitive-search-workloads"></a>認知搜尋工作負載的文件資源

認知搜尋 (現在位於公開預覽版中) 是 Azure 搜尋索引中的新擴充圖層，可在非文字來源和無差異文字中尋找潛在資訊，並將其轉換成 Azure 搜尋服務中可全文檢索搜尋的內容。

下列文章是認知搜尋的完整文件。

## <a name="getting-started"></a>開始使用
+ [什麼是認知搜尋？](cognitive-search-concept-intro.md)
+ [快速入門：在入口網站試用認知搜尋](cognitive-search-quickstart-blob.md)
+ [教學課程：了解認知搜尋 API](cognitive-search-tutorial-blob.md)
+ [範例：建立自訂技能](cognitive-search-create-custom-skill-example.md)

## <a name="how-to-guidance"></a>做法指引
+ [如何定義技能集](cognitive-search-defining-skillset.md)
+ [如何參考技能集中的註釋](cognitive-search-concept-annotations-syntax.md)
+ [如何將欄位對應到索引](cognitive-search-output-field-mapping.md)
+ [如何處理影像並從影像擷取資訊](cognitive-search-concept-image-scenarios.md)
+ [如何重建 Azure 搜尋服務索引](search-howto-reindex.md)
+ [如何定義自訂技能介面](cognitive-search-custom-skill-interface.md)
+ [疑難排解秘訣](cognitive-search-concept-troubleshooting.md)

## <a name="reference"></a>參考

+ [預先定義的技能](cognitive-search-predefined-skills.md)
  + [Microsoft.Skills.Text.KeyPhraseSkill](cognitive-search-skill-keyphrases.md)
  + [Microsoft.Skills.Text.LanguageDetectionSkill](cognitive-search-skill-language-detection.md)
  + [Microsoft.Skills.Text.NamedEntityRecognitionSkill](cognitive-search-skill-named-entity-recognition.md)
  + [Microsoft.Skills.Text.MergeSkill](cognitive-search-skill-textmerger.md)
  + [Microsoft.Skills.Text.SplitSkill](cognitive-search-skill-textsplit.md)
  + [Microsoft.Skills.Text.SentimentSkill](cognitive-search-skill-sentiment.md)
  + [Microsoft.Skills.Vision.ImageAnalysisSkill](cognitive-search-skill-image-analysis.md)
  + [Microsoft.Skills.Vision.OcrSkill](cognitive-search-skill-ocr.md)
  + [Microsoft.Skills.Util.ShaperSkill](cognitive-search-skill-shaper.md)

+ [預覽 REST API](search-api-2017-11-11-preview.md)
  + [建立技能集 (api-version=2017-11-11-Preview)](https://docs.microsoft.com/rest/api/searchservice/create-skillset)
  + [建立索引子 (api-version=2017-11-11-Preview)](https://docs.microsoft.com/rest/api/searchservice/create-indexer)

## <a name="see-also"></a>另請參閱

+ [Azure 搜尋服務 REST API](https://docs.microsoft.com/rest/api/searchservice/)
+ [Azure 搜尋服務中的索引子](search-indexer-overview.md)
+ [何謂 Azure 搜尋服務？](search-what-is-azure-search.md)
