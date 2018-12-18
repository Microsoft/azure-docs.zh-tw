---
title: 什麼是翻譯工具文字 API？
titlesuffix: Azure Cognitive Services
description: 將翻譯工具文字 API 整合到您的應用程式、網站、工具與其他解決方案，以提供多語言使用者體驗。
services: cognitive-services
author: noellelacharite
manager: cgronlun
ms.service: cognitive-services
ms.component: translator-text
ms.topic: overview
ms.date: 05/10/2018
ms.author: nolachar
ms.openlocfilehash: c0004960b7a234d374ec0ae1bdc2f6576b197705
ms.sourcegitcommit: f20e43e436bfeafd333da75754cd32d405903b07
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/17/2018
ms.locfileid: "49385019"
---
# <a name="what-is-translator-text-api"></a>什麼是翻譯工具文字 API？

翻譯工具文字 API 可以無縫整合到您的應用程式、網站、工具或其他解決方案，提供[超過 60 種語言的](languages.md)多語言使用者體驗。 它可用於任何硬體平台上，並可與任何作業系統搭配使用，以執行文字轉文字的語言翻譯。

翻譯工具文字 API 是由雲端中機器學習與 AI 演算法所組成之 [Azure 認知服務 API](https://docs.microsoft.com/azure/#pivot=products&panel=ai) 集合的一部分，隨時可供在您的開發專案中取用。

## <a name="about-microsoft-translator"></a>關於 Microsoft Translator

Microsoft Translator 是一個雲端式機器翻譯服務。 此服務的核心是 Translator Text API 和 [Translator Speech API](https://docs.microsoft.com/azure/cognitive-services/speech-service/speech-translation)，這些 API 為各種 Microsoft 產品和服務提供技術支援，並由全球數千個企業運用在其應用程式和工作流程中，使其內容得以觸及全球用戶。

語音翻譯也可透過[認知服務語音預覽](https://docs.microsoft.com/azure/cognitive-services/speech-service/)取得，其中包含現有的 Translator Speech API、Bing 語音 API 和 Custom Speech Service (預覽)，並結合成為統合並且可完整自訂的服務。  

深入了解 [Microsoft Translator 服務](https://www.microsoft.com/en-us/translator/home.aspx) \(英文\)

## <a name="language-customization"></a>自訂語言

自訂翻譯工具是核心 Microsoft Translator 服務的擴充功能，與 Translator Text API 搭配使用時，可協助您自訂類神經翻譯系統以及改善特定術語和風格的翻譯。

透過自訂翻譯工具，您將可建置翻譯系統，以處理您自己的企業或產業中使用的術語。 其後，您自訂的翻譯系統將可透過一般 Microsoft Translator Text API，使用類別參數跨多種類型的裝置輕易整合到您現有的應用程式、工作流程和網站中。 

深入了解[語言自訂](customization.md)

## <a name="microsoft-translator-neural-machine-translation"></a>Microsoft Translator 類神經機器翻譯

類神經機器翻譯 (NMT) 是高品質 AI 技術架構機器翻譯的新標準。 它取代了已在 2010 年代中期達到品質高原期的舊有統計機器翻譯 (SMT) 技術。

NMT 所提供的翻譯之所以優於 SMT，不僅僅是從原始翻譯品質評分的觀點來看，也因為其翻譯聽起來更流暢、更人性化。 這個流暢性的主要原因在於 NMT 使用句子的完整內容來翻譯單字。 SMT 則只採用每個字前後幾個字的緊鄰內容。

NMT 模型位於 API 核心，使用者並不會看到這些模型。 唯一可見的差異是翻譯品質的提升，特別是中文、日文和阿拉伯文等語言。 

深入了解 [NMT 的運作方式](https://www.microsoft.com/en-us/translator/mt.aspx#nnt) \(英文\)

## <a name="next-steps"></a>後續步驟

- 閱讀[定價詳細資料](https://azure.microsoft.com/pricing/details/cognitive-services/translator-text-api/)。

- [註冊](translator-text-how-to-signup.md)存取金鑰。

- [快速入門](quickstarts/csharp.md)是以 C# 撰寫的 REST API 的呼叫適用的逐步解說。 了解如何以最少的程式碼將文字從一種語言翻譯成另一種語言。

- [API 參考文件](https://docs.microsoft.com/azure/cognitive-services/Translator/reference/v3-0-reference)提供 API 的技術文件。

## <a name="see-also"></a>另請參閱

- [認知服務文件頁面](https://docs.microsoft.com/azure/#pivot=products&panel=ai)
- [認知服務產品頁面](https://azure.microsoft.com/services/cognitive-services/)
- [解決方案和定價資訊](https://www.microsoft.com/en-us/translator/default.aspx) \(英文\)
