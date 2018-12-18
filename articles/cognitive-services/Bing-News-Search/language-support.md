---
title: 語言支援 - Bing 新聞搜尋 API
titleSuffix: Azure Cognitive Services
description: Bing 新聞搜尋 API 支援的自然語言、國家/地區和區域清單。
services: cognitive-services
author: MikeDodaro
manager: cgronlun
ms.service: cognitive-services
ms.component: bing-news-search
ms.topic: conceptual
ms.date: 09/25/2018
ms.author: v-gedod
ms.openlocfilehash: a8f7ccdb199f934011b6d64d813eb36cc9dff952
ms.sourcegitcommit: 9eaf634d59f7369bec5a2e311806d4a149e9f425
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/05/2018
ms.locfileid: "48804543"
---
# <a name="language-and-region-support-for-the-bing-news-search-api"></a>Bing 新聞搜尋 API 支援的語言和區域

Bing 新聞搜尋 API 支援多個國家/地區，許多國家/地區具有一個以上的語言。 使用查詢指定國家/地區，主要是為了根據該國家/地區的興趣，縮小搜尋結果範圍。 此外，結果可能包含 Bing 的連結，而且這些連結可能會根據指定的國家/地區或語言，將 Bing 的使用者體驗當地語系化。

您可以使用 `cc` 查詢參數指定國家/地區。 如果您指定國家/地區，您也必須使用 `Accept-Language` HTTP 標頭指定一或多個語言代碼。 支援的語言視國家/地區而異；「市場」表格中提供針對每個國家/地區支援的語言。

或者，您可以使用 `mkt` 查詢參數和**市場**表格的代碼來指定市場。 指定市場同時會指定國家/地區和慣用的語言。 此時，`setLang` 查詢參數可能會設定為語言代碼，這通常與 `mkt` 所指定的語言相同，除非使用者偏好以另一種語言查看 Bing。

## <a name="supported-markets-for-news-search-endpoint"></a>支援新聞搜尋端點的市場

針對 `/news/search` 端點，下表列出您可以用來指定 `mkt` 查詢參數的市場代碼值。 Bing 只會傳回這些市場的內容。 清單會隨時變動。  

如需可在 `cc` 查詢參數中指定的國家/地區代碼，請參閱[國家/地區代碼](#countrycodes)。  

|國家/區域|語言|市場代碼|  
|---------------------|--------------|-----------------|
|丹麥|丹麥文|da-DK|
|奧地利|德文|de-AT|
|瑞士|德文|de-CH|
|德國|德文|de-DE|
|澳大利亞|English|en-AU|
|加拿大|English|en-CA|
|英國|English|en-GB|
|印尼|English|en-ID|
|愛爾蘭|English|en-IE|
|印度|English|en-IN|
|馬來西亞|English|en-MY|
|紐西蘭|English|en-NZ|
|菲律賓共和國|English|en-PH|
|新加坡|English|en-SG|
|美國|English|zh-TW|
|English|一般|en-WW|
|English|一般|en-XA|
|南非|English|en-ZA|
|阿根廷|西班牙文|es-AR|
|智利|西班牙文|es-CL|
|西班牙|西班牙文|es-ES|
|墨西哥|西班牙文|es-MX|
|美國|西班牙文|es-US|
|西班牙文|一般|es-XL|
|芬蘭|芬蘭文|fi-FI|  
|法國|法文|fr-BE|
|加拿大|法文|fr-CA|
|比利時|荷蘭文|nl-BE|
|瑞士|法文|fr-CH|
|法國|法文|fr-FR|  
|義大利|義大利文|it-IT|
|香港特別行政區|繁體中文|zh-HK|  
|台灣|繁體中文|zh-TW|
|日本|日文|ja-JP|  
|南韓|韓文|ko-KR|  
|荷蘭|荷蘭文|nl-NL|  
|中華人民共和國|中文|zh-CN|  
|巴西|葡萄牙文|pt-BR|
|俄羅斯|俄文|ru-RU|  
|瑞典|瑞典文|sv-SE|  
|土耳其|土耳其文|tr-TR|  

## <a name="supported-markets-for-news-endpoint"></a>支援新聞端點的市場
針對 `/news` 端點，下表列出您可以用來指定 `mkt` 查詢參數的市場代碼值。 Bing 只會傳回這些市場的內容。 清單會隨時變動。  

如需可在 `cc` 查詢參數中指定的國家/地區代碼，請參閱[國家/地區代碼](#countrycodes)。  

|國家/區域|語言|市場代碼|  
|---------------------|--------------|-----------------|
|丹麥|丹麥文|da-DK|
|德國|德文|de-DE|
|澳大利亞|English|en-AU|
|英國|English|en-GB|
|美國|English|zh-TW|
|English|一般|en-WW|
|智利|西班牙文|es-CL|
|墨西哥|西班牙文|es-MX|
|美國|西班牙文|es-US|
|芬蘭|芬蘭文|fi-FI|  
|加拿大|法文|fr-CA|
|法國|法文|fr-FR|  
|義大利|義大利文|it-IT|
|巴西|葡萄牙文|pt-BR|
|中華人民共和國|中文|zh-CN|

## <a name="supported-markets-for-news-trending-endpoint"></a>支援新聞趨勢端點的市場
針對 `/news/trendingtopics` 端點，下表列出您可以用來指定 `mkt` 查詢參數的市場代碼值。 Bing 只會傳回這些市場的內容。 清單會隨時變動。  

如需可在 `cc` 查詢參數中指定的國家/地區代碼，請參閱[國家/地區代碼](#countrycodes)。  

|國家/區域|語言|市場代碼|  
|---------------------|--------------|-----------------|
|德國|德文|de-DE|
|澳大利亞|English|en-AU|
|英國|English|en-GB|
|美國|English|zh-TW|
|加拿大|English|en-CA|
|印度|English|en-IN|
|法國|法文|fr-FR|
|加拿大|法文|fr-CA|
|巴西|葡萄牙文|pt-BR|
|中華人民共和國|中文|zh-CN|


<a name="countrycodes"></a>   
### <a name="country-codes"></a>國碼  

以下是可以在 `cc` 查詢參數中指定的國家/地區代碼。 清單會隨時變動。  

|國家/區域|國碼 (地區碼)|  
|---------------------|------------------|  
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
|香港特別行政區|HK|  
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
|中華人民共和國|CN|  
|波蘭|PL|  
|葡萄牙|PT|  
|菲律賓共和國|PH|  
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

## <a name="next-steps"></a>後續步驟
如需 Bing 新聞搜尋端點的詳細資訊，請參閱[新聞搜尋 API v7 參考](https://docs.microsoft.com/rest/api/cognitiveservices/bing-news-api-v7-reference) \(英文\)。
