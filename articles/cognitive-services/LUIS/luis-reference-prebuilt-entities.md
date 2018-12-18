---
title: 預先建置實體 - LUIS
titleSuffix: Azure Cognitive Services
description: 本文包含 Language Understanding (LUIS) 中所包含預先建置的實體清單。
services: cognitive-services
author: diberry
manager: cgronlun
ms.service: cognitive-services
ms.component: language-understanding
ms.topic: article
ms.date: 09/24/2018
ms.author: diberry
ms.openlocfilehash: 6e38718f14fa5c7cb8455d6c6e7bd5421d70b553
ms.sourcegitcommit: 4ecc62198f299fc215c49e38bca81f7eb62cdef3
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "47034480"
---
# <a name="entities-per-culture"></a>根據文化特性的實體

Language Understanding (LUIS) 提供預先建置的實體。 當您的應用程式中包含預先建置的實體時，LUIS 就會在端點回應中包含對應的實體預測。 所有範例語句也都會標示該實體。 預先建置實體的行為**無法**修改。 除非另有註明，否則預先建置的實體在所有 LUIS 應用程式地區設定 (文化特性) 中均適用。 下表列出支援各種文化特性的預先建置實體。

預先建置的實體   |   英文 (美國)<br>```En-us```   |   法文 (法國)<br>```fr-FR```   |   義大利文 (義大利)<br>```it-IT```   |   西班牙文 (西班牙)<br>```es-ES```   |   中文<br>```zh-CN```   |   德文<br>```de-DE```   |   葡萄牙文 (巴西)<br>```pt-BR```   |   日文 (日本)<br>```ja-JP```   |   韓文 (韓國)<br>```ko-kr```   | 法文 (加拿大)<br>```fr-CA```   |   西班牙文 (墨西哥)<br>```es-MX```   |   荷蘭文 (荷蘭)<br>```nl-NL```   |
------|:------:|------|------|------|------|------|------|------|------|------|------|------|
[年齡](luis-reference-prebuilt-age.md)：<br>年<br>月<br>week<br>day   |    ✔   |    ✔   |    ✔   |    ✔   |    ✔   |    ✔   |    ✔   |    ✔   |   -   |   -   |   -   |   -   |
[貨幣](luis-reference-prebuilt-currency.md)：<br>貨幣<br>小數單位 (例如：貨幣單位)  |    ✔   |    ✔   |    ✔   |    ✔   |    ✔   |    ✔   |    ✔   |    ✔   |   -   |   -   |   -   |   -   |
[DatetimeV2](luis-reference-prebuilt-datetimev2.md)：<br>日期<br>daterange<br>分析<br>時間範圍   |    ✔   |   ✔   |   -   |   ✔   |    ✔   |   -   |   ✔   |   -   |   -   |   -   |   -   |   -   |
[維度](luis-reference-prebuilt-dimension.md)：<br>磁碟區<br>區域<br>重量<br>資訊 (例如：位元/位元組)<br>長度 (例如：公尺)<br>速度 (例如：英哩/小時)  |    ✔   |    ✔   |    ✔   |    ✔   |    ✔   |    ✔   |    ✔   |    ✔   |   -   |   -   |   -   |   -   |
[電子郵件](luis-reference-prebuilt-email.md)   |    ✔   |   -   |   -   |   -   |   -   |   -   |   -   |   -   |   -   |   -   |   -   |   -   |
[GeographyV2](luis-reference-prebuilt-geographyV2.md)   |    ✔   |   -   |   -   |   -   |   -   |   -   |   -   |   -   |   -   |   -   |   -   |   -   |
[KeyPhrase](luis-reference-prebuilt-keyphrase.md)   |    ✔   |   ✔   |   ✔   |   ✔   |   -   |   ✔   |   ✔   |   ✔   |   ✔   |   ✔   |   ✔   |   ✔   |
[Number](luis-reference-prebuilt-number.md)   |    ✔   |    ✔   |    ✔   |    ✔   |    ✔   |    ✔   |    ✔   |    ✔   |   -   |   -   |   -   |   -   |
[序數](luis-reference-prebuilt-ordinal.md)   |    ✔   |    ✔   |    ✔   |    ✔   |    ✔   |    ✔   |    ✔   |    ✔   |   -   |   -   |   -   |   -   |
[百分比](luis-reference-prebuilt-percentage.md)   |    ✔   |    ✔   |    ✔   |    ✔   |    ✔   |    ✔   |    ✔   |    ✔   |   -   |   -   |   -   |   -   |
[PersonName](luis-reference-prebuilt-person.md)   |    ✔   |    -   |    -   |    -   |    ✔   |    -   |    -   |    -   |   -   |   -   |   -   |   -   |
[Phonenumber](luis-reference-prebuilt-phonenumber.md)   |    ✔   |   -   |   -   |   -   |   -   |   -   |   -   |   -   |   -   |   -   |   -   |   -   |
[溫度](luis-reference-prebuilt-temperature.md)：<br>華氏<br>克式<br>蘭氏<br>德氏<br>攝氏   |    ✔   |    ✔   |    ✔   |    ✔   |    ✔   |    ✔   |    ✔   |    ✔   |   -   |   -   |   -   |   -   |
[URL](luis-reference-prebuilt-url.md)   |    ✔   |   -   |   -   |   -   |   -   |   -   |   -   |   -   |   -   |   -   |   -   |   -   |

請參閱[已被取代的預先建置實體](luis-reference-prebuilt-deprecated.md)的注意事項

KeyPhrase 不適用於葡萄牙文 (巴西) 的所有子文化特性 - ```pt-BR```。

<!--
## Examples of prebuilt entities in en-us culture
The following table lists prebuilt entities with example data and the return values.

Prebuilt entity   |   Example utterance   |   JSON
------|------|------|
 ```builtin.age```   |   ```100 year old```   |```{ "type": "builtin.age", "entity": "100 year old" }```|  
 ```builtin.age```   |   ```19 years old```   |```{ "type": "builtin.age", "entity": "19 years old" }```|
 ```builtin.currency```     |   ```1000.00 US dollars```   |```{ "type": "builtin.currency", "entity": "1000.00 us dollars" }```
 ```builtin.currency```     |   ```$ 67.5 B```   |```{ "type": "builtin.currency", "entity": "$ 67.5" }```|
 ```builtin.datetimeV2``` | See [builtin.datetimeV2](luis-reference-prebuilt-datetimev2.md) | See [builtin.datetimeV2](luis-reference-prebuilt-datetimev2.md) |
 ```builtin.dimension```     |   ```2 miles```   |```{ "type": "builtin.dimension", "entity": "2 miles" }```|
 ```builtin.dimension```     |  ```650 square kilometers```   |```{ "type": "builtin.dimension", "entity": "650 square kilometers" }```|
 ```builtin.email```     |  ```patti.owens@microsoft.com```   |```{ "type": "builtin.email", "entity": "patti.owens@microsoft.com" }```|
 ```builtin.number```     |  ```ten```   |``` { "type": "builtin.number", "entity": "ten" } ```|
 ```builtin.number```     |   ```3.1415```   |```  { "type": "builtin.number", "entity": "3 . 1415" }``` |
 ```builtin.ordinal```     |   ```first```   |```{ "type": "builtin.ordinal", "entity": "first" }``` |
 ```builtin.ordinal```     |   ```10th```   | ```{ "type": "builtin.ordinal", "entity": "10th" }``` |  
 ```builtin.percentage```   |   ```The stock price increase by 7 $ this year```   |```{ "type": "builtin.percentage", "entity": "7 %" }```|
 ```builtin.phonenumber```   |   ```my mobile is 00 44 161 1234567```   |```{ "type": "builtin.phonenumber", "entity": "00 44 161 1234567" }```|
 ```builtin.temperature```     |   ```10 degrees celsius```   | ```{ "type": "builtin.temperature", "entity": "10 degrees celcius" }```|   
 ```builtin.temperature```     |   ```78 F```   |```{ "type": "builtin.temperature", "entity": "78 f" }```|
 ```builtin.url```     |   ```http://www.luis.ai is a great cognitive service```   |```{ "type": "builtin.url", "entity": "http://www.luis.ai" }```|
-->

## <a name="contribute-to-prebuilt-entity-cultures"></a>參與預先建置的實體文化特性
預先建置的實體是在 Recognizers-Text 開放原始碼專案中開發的。 [參與](https://github.com/Microsoft/Recognizers-Text)此專案。 此專案包含每個文化特性的貨幣適用的範例。 

## <a name="next-steps"></a>後續步驟

了解 [number](luis-reference-prebuilt-number.md)、[datetimeV2](luis-reference-prebuilt-datetimev2.md) 和 [currency](luis-reference-prebuilt-currency.md) 實體相關資訊。 
