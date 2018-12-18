---
title: Microsoft Translator Text API - 移轉至 V3 | Microsoft Docs
description: 了解如何將 Translator Text API 從 V2 移轉至 V3。
services: cognitive-services
author: Jann-Skotdal
manager: chriswendt1
ms.service: cognitive-services
ms.technology: translator
ms.topic: article
ms.date: 03/27/2018
ms.author: v-jansko
ms.openlocfilehash: 16fec351af5b5b3875657ee244c18f305311d965
ms.sourcegitcommit: 95d9a6acf29405a533db943b1688612980374272
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/23/2018
ms.locfileid: "35370243"
---
# <a name="microsoft-translator-text-api-v2-to-v3-migration"></a>Microsoft Translator Text API V2 到 V3 的移轉

Microsoft Translator 團隊已發行第 3 版 (V3) 的文字翻譯 API。 本版包含對 Microsoft Translator 服務傳送和接收資料的新功能、汰用方法和新格式。 本文件提供將應用程式變更為使用 V3 的資訊。 V2 將在 2018 年 4 月 30 日被取代，並將於 2019 年 4 月 30 日中止。

本文件結尾處包含多個有用的連結，以供您深入了解。

## <a name="summary-of-features"></a>功能摘要

* 無追蹤 - 在 V3 中，Azure 入口網站中的所有定價層都會套用「無追蹤」。 這表示 Microsoft 將不會儲存任何提交至 V3 API 的文字。
* JSON - XML 已由 JSON 取代。 所有傳送至服務和從服務接收的資料均採用 JSON 格式。
* 單一要求中的多目標語言 -「翻譯」方法可接受在單一要求中使用多個翻譯「目標」語言。 例如，單一要求可用英文作為「來源」語言，並以德文、西班牙文和日文或其他任何語言群組作為「目標」語言。
* 雙語字典 - API 中新增了雙語字典方法。 此方法包含「查閱」和「範例」。
* 音譯 - API 中新增了音譯方法。 此方法會將一個指令碼中的字組和句子 (例如 阿拉伯文) 轉換為另一個指令碼 (例如 拉丁文)。
* 語言 - 新的「語言」方法提供 JSON 格式的語言資訊，可與「翻譯」、「字典」和「音譯」等方法搭配使用。
* 新的翻譯功能 - 「翻譯」方法中新增了新功能，用以支援在 V2 API 中作為個別方法的某些功能。 例如，TranslateArray 就是其中之一。
* 口語方法 - Microsoft Translator API 已不再支援文字轉語音功能。 文字轉語音功能可在 Microsoft Azure 認知服務 Bing 語音 API 中使用。

下列 V2 和 V3 方法清單列出將提供 V2 隨附功能的 V3 方法和 API。

| V2 API 方法   | V3 API 相容性 |
|:----------- |:-------------|
| 翻譯     | 翻譯          |
| TranslateArray      | 翻譯          |
| GetLanguageNames      | 語言          |
| GetLanguagesForTranslate     | 語言        |
| GetLanguagesForSpeak      | 認知服務語音 API         |
| 口語     | 認知服務語音 API          |
| 偵測     | 偵測         |
| DetectArray     | 偵測         |
| AddTranslation     | Microsoft Translator HUB API         |
| AddTranslationArray    | Microsoft Translator HUB API          |
| BreakSentences      | BreakSentence         |
| GetTranslations      | 不再支援功能         |
| GetTranslationsArray      | 不再支援功能         |

## <a name="move-to-json-format"></a>移轉至 JSON 格式

Microsoft Translator Text Translation V2 可接受及傳回 XML 格式的資料。 在 V3 中，所有使用 API 傳送和接收的資料均採用 JSON 格式。 在 V3 中將不再接受或傳回 XML。 

此變更將對為 V2 文字翻譯 API 撰寫的應用程式產生若干層面的影響。 例如：語言 API 會傳回文字翻譯、音譯和兩個字典方法的語言資訊。 您可以在單一呼叫中要求所有方法的所有語言資訊，或是個別要求。

語言方法不需要驗證，您只要按一下以下連結，即可用 JSON 格式檢視 V3 的所有語言資訊：

[https://api.cognitive.microsofttranslator.com/languages?api-version=3.0&scope=translation,dictionary,transliteration](https://api.cognitive.microsofttranslator.com/languages?api-version=3.0&scope=translation,dictionary,transliteration)

## <a name="authentication-key"></a>驗證金鑰

V3 將接受您用於 V2 的驗證金鑰。 您不需要取得新的訂用帳戶。 在整整一年的移轉期間，您都能夠在應用程式中混用 V2 及 V3，而讓您在將 V2-XML 移轉至 V3-JSON 的同時，能夠更輕鬆地發行新版本。

## <a name="pricing-model"></a>定價模型

Microsoft Translator V3 的定價方式與 V2 相同，即依字元計價，包含空格在內。 V3 中的新功能針對哪些字元需計入收費的方式做了一些變更。

| V3 方法   | 需計費的字元 |
|:----------- |:-------------|
| 語言     | 未提交任何字元就不會計算，而不會產生費用。          |
| 翻譯     | 計費將取決於提交了多少個字元進行翻譯，以及這些字元翻譯成多少種語言。 若提交了 50 個字元，且要求 5 種語言，則會以 50x5 計算。           |
| 音譯     | 會計算提交以進行音譯的字元數。         |
| 字典查閱和範例     | 會計算針對字典查閱和範例而提交的字元數。         |
| BreakSentence     | 不收費。       |
| 偵測     | 不收費。      |

## <a name="v3-end-points"></a>V3 結束點

全域

* api.cognitive.microsofttranslator.com


## <a name="v3-api-text-translations-methods"></a>V3 API 文字翻譯方法

[語言](reference/v3-0-languages.md)

[翻譯](reference/v3-0-translate.md)

[音譯](reference/v3-0-transliterate.md)

[BreakSentence](reference/v3-0-break-sentence.md)

[偵測](reference/v3-0-detect.md)

[字典/查閱](reference/v3-0-dictionary-lookup.md)

[字典/範例](reference/v3-0-dictionary-examples.md)

## <a name="customization"></a>自訂

Microsoft Translator V3 依預設會使用類神經機器翻譯。 因此，它無法與僅支援傳統統計機器翻譯的 Microsoft Translator Hub 搭配使用。 類神經翻譯現在已可使用自訂翻譯工具進行自訂。 [深入了解如何自訂類神經機器翻譯](customization.md)

使用 V3 文字 API 的類神經翻譯不支援使用標準類別 (smt、語音、文字、generalnn)。


## <a name="links"></a>連結

* [Microsoft 隱私權原則](https://privacy.microsoft.com/privacystatement)
* [Microsoft Azure 法律資訊](https://azure.microsoft.com/support/legal)
* [線上服務條款](http://www.microsoftvolumelicensing.com/DocumentSearch.aspx?Mode=3&DocumentTypeId=31)

## <a name="next-steps"></a>後續步驟

> [!div class="nextstepaction"]
> [檢視 V3.0 文件](reference/v3-0-reference.md)
