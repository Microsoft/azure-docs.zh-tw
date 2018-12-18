---
title: 語言支援 - Bing Web 搜尋 API
titleSuffix: Azure Cognitive Services
description: Bing 新聞搜尋 API 支援的自然語言、國家/地區和區域清單。
services: cognitive-services
author: v-jerkin
manager: cgronlun
ms.service: cognitive-services
ms.component: bing-web-search
ms.topic: article
ms.date: 09/25/2018
ms.author: erhopf
ms.openlocfilehash: c15e1ddd35e625a713ff569f26e9312d9dcd0bc8
ms.sourcegitcommit: 7c4fd6fe267f79e760dc9aa8b432caa03d34615d
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/28/2018
ms.locfileid: "47435649"
---
# <a name="language-and-region-support-for-the-bing-web-search-api"></a>Bing Web 搜尋 API 支援的語言和區域

Bing Web 搜尋 API 支援三十多個國家/地區，許多國家/地區具有一個以上的語言。 使用查詢指定國家/地區，有助於根據該國家/地區的興趣，縮小搜尋結果範圍。 結果可能包含 Bing 的連結，而且這些連結可能會根據指定的國家/地區或語言，將 Bing 的使用者體驗當地語系化。

您可以使用 `cc` 查詢參數指定國家/地區。 若指定了國家/地區，則必須使用 [`Accept-Language` 標頭](https://docs.microsoft.com/rest/api/cognitiveservices/bing-web-api-v7-reference#headers)指定一或多個語言代碼。 使用[市場資料表](#Markets)取得每個市場所支援的語言清單。

或者，您也可以使用 `mkt` 查詢參數和**市場**資料表的代碼來指定市場。 指定市場同時會指定國家/地區和慣用的語言。 您可以使用 `setLang` 查詢參數明確設定語言。

## <a name="countriesregions"></a>國家/區域

|國家/區域|代碼|
|-------|----|
|阿根廷|AR|
|澳大利亞|AU|
|奧地利|AT|
|比利時|BE|
|巴西|BR|
|加拿大|CA|
|智利|CL|
|丹麥|DK|
|芬蘭|FI|
|法國|FR|
|德國|DE|
|香港|HK|
|印度|IN|
|印尼|ID|
|義大利|IT|
|日本|JP|
|南韓|KR|
|馬來西亞|MY|
|墨西哥|MX|
|荷蘭|NL|
|紐西蘭|NZ|
|挪威|否|
|中國|CN|
|波蘭|PL|
|葡萄牙|PT|
|菲律賓|PH|
|俄羅斯|RU|
|沙烏地阿拉伯|SA|
|南非|ZA|
|西班牙|ES|
|瑞典|SE|
|瑞士|CH|
|台灣|TW|
|土耳其|TR|
|英國|GB|
|美國|US|

## <a name="markets"></a>市場

|國家/區域|語言|市場代碼|
|-------|--------|-----------|
|阿根廷|西班牙文|es-AR|
|澳大利亞|English|en-AU|
|奧地利|德文|de-AT|
|比利時|荷蘭文|nl-BE|
|比利時|法文|fr-BE|
|巴西|葡萄牙文|pt-BR|
|加拿大|English|en-CA|
|加拿大|法文|fr-CA|
|智利|西班牙文|es-CL|
|丹麥|丹麥文|da-DK|
|芬蘭|芬蘭文|fi-FI|
|法國|法文|fr-FR|
|德國|德文|de-DE|
|香港|繁體中文|zh-HK|
|印度|English|en-IN|
|印尼|English|en-ID|
|義大利|義大利文|it-IT|
|日本|日文|ja-JP|
|南韓|韓文|ko-KR|
|馬來西亞|English|en-MY|
|墨西哥|西班牙文|es-MX|
|荷蘭|荷蘭文|nl-NL|
|紐西蘭|English|en-NZ|
|挪威|挪威文|no-NO|
|中國|中文|zh-CN|
|波蘭|波蘭文|pl-PL|
|葡萄牙|葡萄牙文|pt-PT|
|菲律賓|English|en-PH|
|俄羅斯|俄文|ru-RU|
|沙烏地阿拉伯|阿拉伯文|ar-SA|
|南非|English|en-ZA|
|西班牙|西班牙文|es-ES|
|瑞典|瑞典文|sv-SE|
|瑞士|法文|fr-CH|
|瑞士|德文|de-CH|
|台灣|繁體中文|zh-TW|
|土耳其|土耳其文|tr-TR|
|英國|English|en-GB|
|美國|English|zh-TW|
|美國|西班牙文|es-US|
