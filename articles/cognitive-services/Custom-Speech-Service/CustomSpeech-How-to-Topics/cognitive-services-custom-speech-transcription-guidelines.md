---
title: Azure 上自訂語音服務中的轉譯指導方針 | Microsoft Docs
description: 了解如何針對「認知服務」中的「自訂語音服務」準備您的資料。
services: cognitive-services
author: PanosPeriorellis
manager: onano
ms.service: cognitive-services
ms.component: custom-speech
ms.topic: article
ms.date: 02/08/2017
ms.author: panosper
ms.openlocfilehash: 6c64b89c2eb933e9bcc24bdc3097e7a0b2aab368
ms.sourcegitcommit: 1aacea6bf8e31128c6d489fa6e614856cf89af19
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/16/2018
ms.locfileid: "49342461"
---
# <a name="transcription-guidelines"></a>轉譯指導方針

[!INCLUDE [Deprecation note](../../../../includes/cognitive-services-custom-speech-deprecation-note.md)]

若要確保充分利用您的文字資料來自訂原音和語言模型，應該依循下列轉譯指導方針。 這些指導方針是語言特有的。

## <a name="text-normalization"></a>文字正規化

為了在自訂原音或語言模型時充分利用文字資料，必須將文字資料正規化，也就是轉換成系統可讀的標準明確形式。 本節說明匯入資料時「自訂語音服務」所執行的文字正規化，以及使用者在匯入資料之前必須先執行的文字正規化。

## <a name="inverse-text-normalization"></a>反向文字正規化

將「原始」未格式化的文字轉換回格式化文字 (亦即帶有大小寫和標點符號) 的過程稱為反向文字正規化 (ITN)。 ITN 會在「Microsoft 認知服務語音 API」傳回的結果上執行。 使用「自訂語音服務」部署的自訂端點會使用與「Microsoft 認知服務語音 API」相同的 ITN。 不過，此服務目前不支援自訂的 ITN，因此自訂語言模型所導入的新字詞在辨識結果中將不會格式化。

## <a name="transcription-guidelines-for-en-us"></a>en-US 的轉譯指導方針

上傳到此服務的文字資料應該只使用 ASCII 可列印字元集以純文字撰寫。 檔案的每一行應該都只包含單一話語的文字。

請務必避免使用 Unicode 標點符號字元。 如果是在文字處理程式中準備資料，或是從網頁集湊資料，便可能不小心發生這種情況。 請以適當的 ASCII 替代字元取代這些字元。 例如︰

| 應避免的 Unicode | ASCII 替代字元 |
|----- | ----- |
| “Hello world” (左右雙引號) | "Hello world" (雙引號) |
| John’s day (右單引號) | John's day (單引號) |

### <a name="text-normalization-performed-by-the-custom-speech-service"></a>自訂語音服務所執行的文字正規化

此服務會對以語言資料集形式匯入的資料，或對原音資料集的轉譯，執行下列文字正規化。 這包括

*   將所有文字轉換成小寫
*   移除文字內部單引號以外的所有標點符號
*   將數字擴充成口語形式，包括美元金額

以下是一些範例

| 原始文字 | 正規化之後 |
|----- | ----- |
| Starbucks Coffee | starbucks coffee |
| “Holy cow!” said Batman. | holy cow said batman |
| “What?” said Batman’s sidekick, Robin. | what said batman’s sidekick robin |
| Go get -em! | go get em |
| I’m double-jointed | i’m double jointed |
| 104 Main Street | one oh four main street |
| Tune to 102.7 | tune to one oh two point seven |
| Pi is about 3.14 | pi is about three point one four |
| It costs $3.14 | it costs three fourteen |

### <a name="text-normalization-required-by-users"></a>使用者所需的文字正規化

若要確保充分利用您的資料，應該先針對資料套用下列正規化規則，然後才匯入該資料。

*   縮寫應該以字組寫出以反映口語形式
*   非標準數值字串應該以字組寫出
*   含非字母字元或混合英數字字元的字組應該轉譯如發音
*   常見的縮略字可以保留單一實體形式，其中字母間不含句點或空格，但所有其他縮略字則應該以個別字母寫出，其中以單一空格分隔每個字母

以下是一些範例

| 原始文字 | 正規化之後 |
|----- | ----- |
| 14 NE 3rd Dr. | fourteen northeast third drive |
| Dr.Strangelove | Doctor Strangelove |
| James Bond 007 | james bond double oh seven |
| Ke$ha | Kesha |
| How long is the 2x4 | How long is the two by four |
| The meeting goes from 1-3pm | The meeting goes from one to three pm |
| my blood type is O+ | My blood type is O positive |
| water is H20 | water is H 2 O |
| play OU812 by Van Halen | play O U 8 1 2 by Van Halen |

## <a name="transcription-guidelines-for-zh-cn"></a>zh-CN 的轉譯指導方針

上傳到「自訂語音服務」的文字資料應該使用 **UTF-8 編碼 (包含 BOM)**。 檔案的每一行應該都只包含單一話語的文字。

請務必避免使用半形標點符號字元。 如果是在文字處理程式中準備資料，或是從網頁集湊資料，便可能不小心發生這種情況。 請以適當的全形替代字元取代這些字元。 例如︰

| 應避免的 Unicode | ASCII 替代字元 |
|----- | ----- |
| “你好” (左右雙引號) | "你好" (雙引號) |
| 需要什么帮助? (問號) | 需要什么帮助？ |

### <a name="text-normalization-performed-by-the-custom-speech-service"></a>自訂語音服務所執行的文字正規化

此語音服務會對以語言資料集形式匯入的資料，或對原音資料集的轉譯，執行下列文字正規化。 這包括

*   移除所有標點符號
*   將數字擴充成口語形式
*   將全形字母轉換成半形字母。
*   將所有英文字轉換成大寫

這裡有一些範例：

| 原始文字 | 正規化之後 |
|----- | ----- |
| 3.1415 | 三 点 一 四 一 五 |
| ￥3.5 | 三 元 五 角 |
| w f y z | W F Y Z |
| 1992年8月8日 | 一 九 九 二 年 八 月 八 日 |
| 你吃饭了吗 ? | 你 吃饭 了 吗 |
| 下午5:00的航班 | 下午 五点 的 航班 |
| 我今年21岁 | 我 今年 二十 一 岁 |

### <a name="text-normalization-required-by-users"></a>使用者所需的文字正規化

若要確保充分利用您的資料，應該先針對資料套用下列正規化規則，然後才匯入該資料。

*   縮寫應該以字組寫出以反映口語形式
*   此服務未涵蓋所有數值數量。 較可靠的方式是將數值字串以口語形式寫出

以下是一些範例

| 原始文字 | 正規化之後 |
|----- | ----- |
| 我今年21 | 我今年二十一 |
| 3号楼504 | 三号 楼 五 零 四 |

## <a name="transcription-guidelines-for-de-de"></a>de-DE 的轉譯指導方針

上傳到「自訂語音服務」的文字資料應該只使用 **UTF-8 編碼 (包含 BOM)**。 檔案的每一行應該都只包含單一話語的文字。

### <a name="text-normalization-performed-by-the-custom-speech-service"></a>自訂語音服務所執行的文字正規化

此服務會對以語言資料集形式匯入的資料，或對原音資料集的轉譯，執行下列文字正規化。 這包括

*   將所有文字轉換成小寫
*   移除所有標點符號，包括英文或德文引號 (可接受 "test"、'test'、“test„ 或 «test»)
*   捨棄包含任何特殊字元的任何資料列，包括：^ ¢ £ ¤ ¥ ¦ § © ª ¬ ® ° ± ² µ × ÿ Ø¬¬
*   將數字擴充成字組形式，包括美元或歐元金額
*   我們只接受 a、o、u 的變母音；其他則會以 "th" 取代或捨棄

以下是一些範例

| 原始文字 | 正規化之後 |
|----- | ----- |
| Frankfurter Ring | frankfurter ring |
| "Hallo, Mama!" sagt die Tochter. | hallo mama sagt die tochter |
| ¡Eine Frage! | eine frage |
| wir, haben | wir haben |
| Das macht $10 | das macht zehn dollars |


### <a name="text-normalization-required-by-users"></a>使用者所需的文字正規化

若要確保充分利用您的資料，應該先針對資料套用下列正規化規則，然後才匯入該資料。

*   小數點應該是 , 而非 . 例如 2,3% 而非 2.3%
*   小時與分鐘之間的時間分隔符號應該是 : 而非 .，例如 12:00 Uhr
*   'ca.', 'bzw.' 之類的縮寫 不會被取代。 建議您使用完整格式，以便擁有正確的發音。
*   已移除五個主要的數學運算子：+、-、\*/。
 建議您以其文字形式 plus、minus、mal、geteilt 加以取代。
*   這同樣適用於比較子 (=、<、>) - gleich、kleiner als、grösser als
*   使用 3/4 這類分數時，採用文字形式 'drei viertel' 而不採用 ¾
*   以文字形式 "Euro" 取代 € 符號


以下是一些範例

| 原始文字 | 在使用者的正規化之後 | 在系統的正規化之後
|--------  | ----- | -------- |
| Es ist 12.23Uhr | Es ist 12:23Uhr | es ist zwölf uhr drei und zwanzig uhr |
| {12.45} | {12,45} | zwölf komma vier fünf |
| 3 < 5 | 3 kleiner als 5 | drei kleiner als vier |
| 2 + 3 - 4 | 2 plus 3 minus 4 | zwei plus drei minus vier|
| Das macht 12€ | Das macht 12 Euros | das macht zwölf euros |



## <a name="next-steps"></a>後續步驟
* [如何使用自訂的語音轉換文字端點](cognitive-services-custom-speech-create-endpoint.md)
* 使用您的[自訂原音模型](cognitive-services-custom-speech-create-acoustic-model.md)來提升準確性
* 使用您的[自訂語言模型](cognitive-services-custom-speech-create-language-model.md)來提升準確性
