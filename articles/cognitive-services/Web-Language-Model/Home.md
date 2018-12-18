---
title: Web 語言模型 API 的概觀
titleSuffix: Azure Cognitive Services
description: Microsoft 認知服務中的 Web 語言模型 API 可提供最先進的工具來處理自然語言。
services: cognitive-services
author: piyushbehre
manager: cgronlun
ms.service: cognitive-services
ms.component: web-language-model
ms.topic: overview
ms.date: 08/12/2016
ms.author: pibehre
ROBOTS: NOINDEX
ms.openlocfilehash: 79b126fc33175b7cd6df96ab07cd7b726d6065a7
ms.sourcegitcommit: f20e43e436bfeafd333da75754cd32d405903b07
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/17/2018
ms.locfileid: "49389933"
---
# <a name="what-is-the-web-language-model-api-preview"></a>什麼是 Web 語言模型 API？ (預覽)

> [!IMPORTANT]
> Web 語言模型預覽已於 2018 年 8 月 9 日解除委任。 我們建議使用 [Azure Machine Learning 文字分析模組](https://docs.microsoft.com/azure/machine-learning/studio-module-reference/text-analytics)來進行文字處理和分析。

Microsoft Web 語言模型 API 是 REST 架構的雲端服務，可提供最先進的工具來處理自然語言。 使用此 API，應用程式便可透過 Bing 在 en-US 市場所收集的 Web 規模語料庫將語言模型定型，以利用巨量資料的強大威力。

這些平滑降速的 N-gram 語言模型會在下列語料庫上定型，最多可支援到第 5 階的 Markov 鏈結：

- 網頁本文
- 網頁標題文字
- 網頁錨定文字
- Web 搜尋查詢文字

Web 語言模型 API 支援四個查閱作業：

1. 文字序列的聯結 (log10) 機率。
2. 指定了前面字組序列的單一文字條件式 (log10) 機率。
3. 最可能遵循指定文字順序的文字清單 (完成)。
4. 不包含空格的字串斷字。

## <a name="getting-started"></a>開始使用

1. 訂閱服務。
2. 下載 [SDK](https://www.github.com/microsoft/cognitive-weblm-windows)。
3. 執行 SDK 程式碼範例。
4. 請參閱 [API 參考](https://westus.dev.cognitive.microsoft.com/docs/services/55de9ca4e597ed1fd4e2f104)以取得端點的完整詳細資料，包括各種語言的程式碼片段。

## <a name="underlying-technology"></a>基礎技術

下列文件詳述這些語言模型的開發，使用這項服務的研究出版品中應該會加以引用：

- [An Overview of Microsoft Web N-gram Corpus and Applications](http://research.microsoft.com/apps/pubs/default.aspx?id=130762), NAACL-HLT 2010

按一下[此處](https://academic.microsoft.com/#/search?iq=And%28Ty%3D'0'%2CRId%3D2145833060%29&q=papers%20citing%20an%20overview%20of%20microsoft%20web%20n%20gram%20corpus%20and%20applications&filters=&from=0&sort=0)來取得引用此著作的最新文件清單。
