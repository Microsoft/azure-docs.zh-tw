---
title: 什麼是翻譯工具語音服務？
titleSuffix: Azure Cognitive Services
description: 使用翻譯工具語音服務 API 將語音轉換語音及語音轉換文字翻譯新增至您的應用程式。
services: cognitive-services
author: Jann-Skotdal
manager: cgronlun
ms.service: cognitive-services
ms.component: translator-speech
ms.topic: overview
ms.date: 3/5/2018
ms.author: v-jansko
ms.openlocfilehash: 77e60fe39f7cbb985ee0e7ed2785805e80c389ae
ms.sourcegitcommit: 1aacea6bf8e31128c6d489fa6e614856cf89af19
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/16/2018
ms.locfileid: "49341135"
---
# <a name="what-is-translator-speech-api"></a>什麼是翻譯工具語音 API？

[!INCLUDE [Deprecation note](../../../includes/cognitive-services-translator-speech-deprecation-note.md)]

您可以使用翻譯工具語音 API，為應用程式、工具或任何需要多語言語音翻譯的解決方案，新增端對端的即時語音翻譯，而無須考慮目標 OS 或開發語言為何。 此 API 同時適用於語音轉換語音及語音轉換文字翻譯。

翻譯工具文字 API 是一個 Azure 服務，是由雲端中機器學習和 AI 演算法所組成之 [Azure 認知服務 API 集合](https://docs.microsoft.com/azure/#pivot=products&panel=cognitive)的一部分，隨時可供在您的開發專案中取用。

使用翻譯工具語音 API 時，用戶端應用程式會將語音音訊串流至服務，然後接收文字型和音訊型的結果資料流，包括來源語言中的已辨識文字及其在目標語言中的翻譯。 產生文字結果時，是透過將採用深度類神經網路技術的「自動語音辨識」(ASR) 套用至傳入的音訊資料流來產生。 原始 ASR 輸出經過稱為 TrueText 的新技術進一步改善，而能夠更密切地反映使用者意圖。 例如，TrueText 會移除話語中不流暢的部分 (嗯和咳嗽)、重複單字，並還原適當的標點符號和大小寫。 此外，也包含了遮罩或排除粗話的功能。 辨識和翻譯引擎皆特別經過訓練來處理交談語音。 

翻譯工具語音服務使用無聲偵測來判斷語句的結束。 在語音活動暫停之後，服務會串流來傳回已完成語句的最終結果。 此服務也可以送回部分結果，這可提供進行中語句的中繼辨識和翻譯。 

針對語音轉換語音翻譯，此服務提供從目標語言口語文字合成語音 (文字轉換語音) 的功能。 文字轉換語音音訊會以用戶端所指定的格式建立。 可用的格式包括 WAV 和 MP3。

翻譯工具語音 API 使用 WebSocket 通訊協定來提供用戶端與伺服器之間的全雙工通訊通道。 

## <a name="about-microsoft-translator"></a>關於 Microsoft Translator
Microsoft Translator 是一個雲端式機器翻譯服務。 此服務的核心是 [Translator Text API] (https://www.microsoft.com/en-us/translator/translatorapi.aspx) 和 Translator Speech API，這些 API 為各種 Microsoft 產品和服務提供技術支援，並由全球數千個企業運用在它們應用程式和工作流程中，使它們的內容得以觸及全球用戶。

深入了解 [Microsoft Translator 服務](https://www.microsoft.com/en-us/translator/home.aspx) \(英文\)

## <a name="microsoft-translator-neural-machine-translation-nmt"></a>Microsoft Translator 類神經機器翻譯 (NMT)
翻譯工具語音 API 同時使用傳統統計式機器翻譯 (SMT) 和更新的類神經機器翻譯 (NMT) 來提供翻譯。

統計機器翻譯在效能改善上已達到瓶頸。 對於使用 SMT 的一般系統來說，已無法再以任何顯著的方式改善翻譯品質。 一項新的 AI 型翻譯技術正挾著「類神經網路」(NN) 趁勢而起。

NMT 提供的翻譯比 SMT 更好的原因，不僅僅是從原始翻譯品質計分的觀點來看，也因為其翻譯聽起來更流暢、更人性化。 這個流暢性的主要原因在於 NMT 使用句子的完整內容來翻譯單字。 SMT 則只採用每個字前後幾個字的緊鄰內容。

NMT 模型位於 API 核心，使用者並不會看到這些模型。 唯一明顯的差異在於：
* 翻譯品質提升，特別是針對中文、日文及阿拉伯文等語言
* 與現有的中樞自訂功能 (可與 Microsoft Translator Text API 搭配使用) 不相容

所有支援的語音翻譯語言皆採用 NMT 技術。 因此，所有語音轉換語音翻譯都使用 NMT。 

語音轉換文字翻譯則可能視語言配對而定，使用 NMT 與 SMT 的組合。 如果目標語言是 NMT 支援的語言，完整翻譯就會採用 NMT 技術。 如果目標語言不是 NMT 支援的語言，翻譯就會是 NMT 與 SMT 兩者的混合，其中會使用英文作為兩種語言之間的「樞紐」。 

檢視 [Microsoft.com](https://www.microsoft.com/en-us/translator/languages.aspx) \(英文\) 上支援的語言。 

深入了解 [NMT 的運作方式](https://www.microsoft.com/en-us/translator/mt.aspx#nnt) \(英文\)

## <a name="next-steps"></a>後續步驟

> [!div class="nextstepaction"]
> [註冊](translator-speech-how-to-signup.md)

> [!div class="nextstepaction"]
> [開始編寫程式碼](quickstarts/csharp.md)

## <a name="see-also"></a>另請參閱
- [認知服務文件頁面](https://docs.microsoft.com/azure/#pivot=products&panel=cognitive)
- [認知服務產品頁面](https://azure.microsoft.com/services/cognitive-services/)
- [解決方案和定價資訊](https://www.microsoft.com/en-us/translator/home.aspx) \(英文\) 
