---
title: 關於語音翻譯
description: 語音翻譯功能概觀
titleSuffix: Microsoft Cognitive Services
services: cognitive-services
author: erhopf
ms.service: cognitive-services
ms.component: speech-service
ms.topic: article
ms.date: 04/28/2018
ms.author: erhopf
ms.openlocfilehash: f3c22589270ccee2c60ca5ea5b15047d9c81fabe
ms.sourcegitcommit: c282021dbc3815aac9f46b6b89c7131659461e49
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/12/2018
ms.locfileid: "49165432"
---
# <a name="about-the-speech-translation-api"></a>關於語音翻譯 API

Microsoft 語音 API 可讓您將端對端、即時、多國語言翻譯的語音新增至您的應用程式、工具和裝置。 相同的 API 可以用於語音轉換語音和語音轉換文字翻譯。

使用 Microsoft Translator Speech API，用戶端應用程式會將語音音訊串流至服務，且接收回結果串流。 這些結果包括來源語言中的已辨識文字，以及其在目標語言中的翻譯。 語調完成之前，可以提供暫時翻譯，而在完成時會提供最終翻譯。

您可以選擇備妥最終翻譯的合成音訊版本，以啟用直正的語音轉換語音翻譯。

語音翻譯 API 使用 WebSockets 通訊協定來提供用戶端與伺服器之間的全雙工通訊通道。 但是您不需要處理 WebSockets；語音 SDK 會為您處理。

語音翻譯 API 會使用採用各種 Microsoft 產品和服務的相同技術。 全球已有數千家公司在其應用程式和工作流程中使用此服務。

## <a name="about-the-technology"></a>關於技術

根據 Microsoft 翻譯引擎的基礎，有兩種不同的方法：統計機器翻譯 (SMT) 和類神經機器翻譯 (NMT)。 後者，採用類神經網路的人工智慧方法是較新的機器翻譯方式。 NMT 提供更好的翻譯；不只更為精確，也更為流暢和自然。 這項流暢性的主要原因在於 NMT 使用句子的完整內容來翻譯文字。

現在，針對最受歡迎的語言，Microsoft 已遷移至 NMT，而且只會針對不常使用的語言採用 SMT。 所有[可用於語音轉換語音翻譯的語言](language-support.md#speech-translation)都是採用 NMT 技術。 根據語言配對，語音轉換文字翻譯可能會使用 SMT 或 NMT。 若 NMT 支援目標語言，則完整翻譯採用 NMT 技術。 若 NMT 不支援目標語言，則翻譯是 NMT 與 SMT 兩者的混合，並使用英文作為兩種語言之間的「樞紐」。

模型之間的差異在於內部翻譯引擎。 終端使用者只會注意到改善的翻譯品質，特別是中文、日文和阿拉伯文。

> [!NOTE]
> 對深入了解翻譯引擎的技術感興趣嗎？ 請參閱[機器翻譯](https://www.microsoft.com/en-us/translator/mt.aspx)。

## <a name="next-steps"></a>後續步驟

* [試用認知服務](https://azure.microsoft.com/try/cognitive-services/)
* [了解如何以 C# 來轉譯語音](how-to-translate-speech-csharp.md)
* [了解如何以 C++ 來轉譯語音](how-to-translate-speech-cpp.md)
* [了解如何以 Java 來轉譯語音](how-to-translate-speech-java.md)
