---
title: 語言支援 - LUIS
titleSuffix: Azure Cognitive Services
description: LUIS 在服務內有各種不同的功能。 並非所有功能都有相同的語言地位。 請確定您有興趣的功能支援您所針對的語言文化特性。 LUIS 應用程式是特定文化特性，一旦設定就無法變更。
services: cognitive-services
author: diberry
manager: cgronlun
ms.service: cognitive-services
ms.component: language-understanding
ms.topic: article
ms.date: 09/26/2018
ms.author: diberry
ms.openlocfilehash: ce0a2f36ca11c704062deb28bf47c45a91a32222
ms.sourcegitcommit: 7c4fd6fe267f79e760dc9aa8b432caa03d34615d
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/28/2018
ms.locfileid: "47435644"
---
# <a name="language-and-region-support-for-luis"></a>LUIS 支援的語言與區域

LUIS 在服務內有各種不同的功能。 並非所有功能都有相同的語言地位。 請確定您有興趣的功能支援您所針對的語言文化特性。 LUIS 應用程式是特定文化特性，一旦設定就無法變更。

## <a name="multi-language-luis-apps"></a>多語言 LUIS 應用程式

如果您需要多語言 LUIS 用戶端應用程式 (例如聊天機器人)，您會有幾個選項可用。 如果 LUIS 支援所有語言，請針對每種語言開發 LUIS 應用程式。 每個 LUIS 應用程式都有唯一的應用程式識別碼和端點記錄。 如果您必須為 LUIS 不支援的語言提供語言理解，您可以使用 [Microsoft 翻譯工具 API](../Translator/translator-info-overview.md)將語句翻譯成支援的語言、將語句提交至 LUIS 端點，以及接收所產生的分數。

## <a name="languages-supported"></a>支援的語言

LUIS 可理解下列語言的語句：

| 語言 |地區設定  |  預建網域 | 預建實體 | 片語建議 | \**[文字分析](https://docs.microsoft.com/azure/cognitive-services/text-analytics/text-analytics-supported-languages)<br>(情感和<br>關鍵字)|
|--|--|:--:|:--:|:--:|:--:|
| 美式英文 |`en-US` | ✔ | ✔  |✔|✔|
| 法文 (加拿大) |`fr-CA` |-|   -   |-|✔|
| *[中文](#chinese-support-notes) |`zh-CN` | ✔ | ✔ |✔|-|
| 荷蘭文 |`nl-NL` |-|  -   |-|✔|
| 法文 (法國) |`fr-FR` |-| ✔ |✔ |✔|
| 德文 |`de-DE` |-| ✔ |✔ |✔|
| 義大利文 |`it-IT` |-| ✔ |✔|✔|
| *[日文](#japanese-support-notes) |`ja-JP` |-| ✔ |✔|僅限關鍵片語|
| 韓文 |`ko-KR` |-|   -   |-|僅限關鍵片語|
| 葡萄牙文 (巴西) |`pt-BR` |-| ✔ |✔ |並非所有的次文化特性|
| 西班牙文 (西班牙) |`es-ES` |-| ✔ |✔|✔|
| 西班牙文 (墨西哥)|`es-MX` |-|  -   |✔|✔|


語言支援會因[預建實體](luis-reference-prebuilt-entities.md)和[網域](luis-reference-prebuilt-domains.md)而有所不同。

### <a name="chinese-support-notes"></a>*中文支援附註

 - 在 `zh-cn` 文化特性中，LUIS 預期會有簡體中文字元集，而不是繁體字元集。
 - 意圖、實體、功能和規則運算式的名稱可能採用中文或羅馬字元。
 - 如需 `zh-cn` 文化特性中支援哪些預建網域的相關資訊，請參閱[網域參考](luis-reference-prebuilt-domains.md)。
<!--- When writing regular expressions in Chinese, do not insert whitespace between Chinese characters.-->

### <a name="japanese-support-notes"></a>*日文支援附註

 - 因為 LUIS 不提供語法分析而無法理解 Keigo (敬語) 與非正式日文之間的差異，所以您必須將不同的正式層級合併為您應用程式的訓練範例。
     - でございます 與 です 不同。
     - です 與 だ 不同。

### <a name="text-analytics-support-notes"></a>\*\*文字分析支援附註
文字分析包含 keyPhrase 預先建置的實體和情感分析。 只有葡萄牙文支援次文化特性：`pt-PT` 和 `pt-BR`。 主要文化特性層級支援其他所有的文化特性。 深入了解文字分析[支援的語言](https://docs.microsoft.com/azure/cognitive-services/text-analytics/text-analytics-supported-languages)。

### <a name="speech-api-supported-languages"></a>語音 API 支援的語言
請參閱語音[支援的語言](https://docs.microsoft.com/azure/cognitive-services/Speech/api-reference-rest/supportedlanguages##interactive-and-dictation-mode)，以取得語音聽寫模式語言。

### <a name="bing-spell-check-supported-languages"></a>Bing 拼字檢查支援的語言
如需支援的語言清單和狀態，請參閱 Bing 拼字檢查[支援的語言](https://docs.microsoft.com/azure/cognitive-services/bing-spell-check/bing-spell-check-supported-languages)。

## <a name="rare-or-foreign-words-in-an-application"></a>應用程式中的罕見或外來字
在 `en-us` 文化特性中，LUIS 會學習辨識大部分的英文字，包括俚語。 在 `zh-cn` 文化特性中，LUIS 會學習辨識大部分的中文字元。 如果您使用 `en-us` 中的罕見字組或 `zh-cn` 中的字元，而且您發現 LUIS 似乎無法辨識該字組或字元，您可以將該字組或字元新增到[片語清單功能](luis-how-to-add-features.md)。 例如，應用程式文化特性外部的字組 (也就是外來字組) 應新增至片語清單功能。 此片語清單應標記為不可互換，表示一些罕見字組會構成 LUIS 應學習辨識的類別，但這些字組並不是同義字或可彼此互換。

### <a name="hybrid-languages"></a>混合式語言
混合式語言結合來自兩個文化特性 (例如英文和中文) 的文字。 LUIS 中不支援這些語言，因為應用程式是以單一文化特性為基礎。

## <a name="tokenization"></a>Token 化
為了執行機器學習，LUIS 根據文化特性將語句分成數個[語彙基元](luis-glossary.md#token)。

|語言|  每個空格或特殊字元 | 字元層級|複合字組|[傳回的 Token 化實體](luis-concept-data-extraction.md#tokenized-entity-returned)
|--|:--:|:--:|:--:|:--:|
|中文||✔||✔|
|荷蘭文|||✔|✔|
|英文 (en-us)|✔ ||||
|法文 (fr-FR)|✔||||
|法文 (fr-CA)|✔||||
|德文|||✔|✔|
|義大利文|✔||||
|日文||||✔|
|韓文||✔||✔|
|葡萄牙文 (巴西)|✔||||
|西班牙文 (es-ES)|✔||||
|西班牙文 (es-MX)|✔||||
