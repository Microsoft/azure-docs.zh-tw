---
title: 支援的語言 - Translator Speech API
titlesuffix: Azure Cognitive Services
description: 檢視翻譯工具語音 API 支援的語言。
services: cognitive-services
author: Jann-Skotdal
manager: cgronlun
ms.service: cognitive-services
ms.component: translator-speech
ms.topic: conceptual
ms.date: 3/5/2018
ms.author: v-jansko
ms.openlocfilehash: dd1c60a2c2ad88abf686e89972d29768ef600e4e
ms.sourcegitcommit: 1aacea6bf8e31128c6d489fa6e614856cf89af19
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/16/2018
ms.locfileid: "49344552"
---
# <a name="languages-supported-by-the-translator-speech-api"></a>翻譯工具語音 API 支援的語言

[!INCLUDE [Deprecation note](../../../includes/cognitive-services-translator-speech-deprecation-note.md)]

語音翻譯支援以下語言。 如果語音翻譯兩種語言都支援，則可以使用語音轉換語音或語音轉換文字。 如果語音翻譯不支援目標語言，則只能使用語音轉換文字翻譯。 

| 語音語言    |
|:----------- |
| 阿拉伯文 (現代標準)      |
| 中文 (普通話)      |
| English      |
| 法文      |
| 德文      |
| 義大利文      |
| 日文      |
| 葡萄牙文 (巴西)     |
| 俄文      |
| 西班牙文      | 

翻譯工具語音 API 支援以下語言作為語音轉換文字翻譯的目標語言。 

| 文字語言    | 語言代碼 |
|:----------- |:-------------:|
| 南非荷蘭文      | `af`          |
| 阿拉伯文       | `ar`          |
| 孟加拉文      | `bn`          |
| 波士尼亞文 (拉丁文)      | `bs`          |
| 保加利亞文      | `bg`          |
| 粵語 (繁體中文)      | `yue`          |
| 卡達隆尼亞文      | `ca`          |
| 簡體中文      | `zh-Hans`          | 
| 繁體中文      | `zh-Hant`          |
| 克羅埃西亞文      | `hr`          |
| 捷克文      | `cs`          |
| 丹麥文      | `da`          |
| 荷蘭文      | `nl`          |
| English      | `en`          |
| 愛沙尼亞文      | `et`          |
| 斐濟文      | `fj`          |
| 菲律賓文      | `fil`          |
| 芬蘭文      | `fi`          |
| 法文      | `fr`          |
| 德文      | `de`          |
| 希臘文      | `el`          |
| 海地克裏奧爾文      | `ht`          |
| 希伯來文      | `he`          |
| 北印度文      | `hi`          |
| 白苗文      | `mww`          |
| 匈牙利文      | `hu`          |
|冰島文|`is`          |
| 印尼文      | `id`          |
| 義大利文      | `it`          |
| 日文      | `ja`          |
| 史瓦希里文      | `sw`          |
| 克林貢文      | `tlh`          |
| 克林貢文 (plqaD)      | `tlh-Qaak`          |
| 韓文      | `ko`          |
| 拉脫維亞文      | `lv`          |
| 立陶宛文      | `lt`          |
| 馬達加斯加文      | `mg`          |
| 馬來文      | `ms`          |
| 馬爾他文      | `mt`          |
| 挪威文      | `nb`          |
| 波斯文      | `fa`          |
| 波蘭文      | `pl`          |
| 葡萄牙文      | `pt`          |
| 奎雷塔洛歐多米文      | `otq`          |
| 羅馬尼亞文      | `ro`          |
| 俄文      | `ru`          |
| 薩摩亞文      | `sm`          |
| 塞爾維亞文 (斯拉夫)      | `sr-Cyrl`          |
| 塞爾維亞文 (拉丁)      | `sr-Latn`          |
| 斯洛伐克文     | `sk`          |
| 斯洛維尼亞文      | `sl`          |
| 西班牙文      | `es`          |
| 瑞典文      | `sv`          |
| 大溪地文      | `ty`          |
| 坦米爾文      | `ta`          |
| 泰文      | `th`          |
| 東加文      | `to`          |
| 土耳其文      | `tr`          |
| 烏克蘭文      | `uk`          |
| 烏都文      | `ur`          |
| 越南文      | `vi`          |
| 威爾斯文      | `cy`          |
| 猶加敦馬雅文      | `yua`          |

## <a name="access-the-list-programmatically"></a>以程式設計方式存取清單

您可以使用語言資源以程式設計方式存取支援的語言清單。 此清單提供語言代碼，以及該語言的英文名稱或任何其他受支援語言的名稱。 當我們提供新語言時，翻譯工具語音服務會自動更新此清單。

語言資源會傳回語音、文字和文字轉換語音的支援語言清單。 語言資源不需要驗證。

[瀏覽 API 參考以試用語言方法](languages-reference.md)

## <a name="access-the-list-on-the-microsoft-translator-website"></a>存取 Microsoft Translator 網站上的清單

如需快速一覽所有語言，Microsoft Translator 網站顯示了翻譯工具文字與語音 API 支援的所有語言。 這份清單不含開發人員特定資訊，例如語言代碼。

[查看語言清單](https://www.microsoft.com/translator/languages.aspx) \(英文\) 
