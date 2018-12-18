---
title: Microsoft 翻譯工具文字 API 翻譯自訂 | Microsoft Docs
description: 以慣用的術語和樣式，使用 Microsoft Translator Hub 建置您自己的電腦翻譯系統。
services: cognitive-services
author: Jann-Skotdal
manager: chriswendt1
ms.service: cognitive-services
ms.component: translator-text
ms.topic: article
ms.date: 05/10/2018
ms.author: v-jansko
ms.openlocfilehash: 1db22a414c41f338c4e7fd6ce9dc7ac739fa9237
ms.sourcegitcommit: 95d9a6acf29405a533db943b1688612980374272
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/23/2018
ms.locfileid: "35370231"
---
# <a name="customize-your-text-translations"></a>自訂文字翻譯

Microsoft 自訂翻譯工具預覽是 Microsoft 翻譯工具服務的功能，可讓使用者在使用 Microsoft 翻譯工具文字 API (僅限第 3 版) 翻譯文字時，自訂 Microsoft 翻譯工具的進階類神經機器翻譯。 

此功能在搭配使用[認知服務語音預覽](https://docs.microsoft.com/en-us/azure/cognitive-services/speech-service/)時，也可用來自訂語音翻譯。

## <a name="custom-translator"></a>自訂翻譯工具
利用自訂翻譯工具，您可以建置類神經翻譯系統，以了解您自己的企業和產業中使用的術語。 然後，自訂的翻譯系統會整合到現有的應用程式、工作流程和網站。 

### <a name="how-does-it-work"></a>運作方式
使用您之前翻譯的文件 (傳單、網頁、文件等) 來建置反映特定領域術語和樣式的翻譯系統，比一般翻譯系統更好。 使用者可以上傳 TMX、XLIFF、TXT、DOCX 和 XLSX 文件。  

系統也接受在文件層級上為平行處理、但在句子層級上尚未對齊的資料。 若使用者可以存取使用多種語言、但位於在不同文件中的相同內容版本，自訂翻譯工具就能夠自動比對文件之間的句子。  系統還可以使用其中一或二種語言的單語資料，補充平行訓練資料來改善翻譯。 

然後，您可以利用 category 參數，透過定期呼叫 Microsoft 翻譯工具文字 API 來使用自訂系統。

假設有適當的訓練資料類型和數量，透過使用自訂翻譯工具預期翻譯品質取得介於 5 到 10 之間或甚至更多的 BLEU 點數並不罕見。

如需根據可用資料之各種自訂層級的詳細資料，請參閱[自訂翻譯工具使用者指南](http://aka.ms/CustomTranslatorDocs)。


## <a name="microsoft-translator-hub"></a>Microsoft Translator Hub

舊版的 Microsoft Translator Hub 可用來翻譯統計機器翻譯。 [深入了解](https://www.microsoft.com/en-us/translator/hub.aspx) 

## <a name="custom-translator-versus-hub"></a>自訂翻譯工具與中樞

|   | **中心** | **自訂翻譯工具**|
|:-----|:----:|:----:|
|自訂功能狀態   | 正式運作  | 預覽 |
| 文字 API 版本  | 僅限第 2 版   | 僅限第 3 版 |
| SMT 自訂 | yes   | 否 | 
| NMT 自訂 | 否    | yes |
| 新的統一語音服務自訂 | 否    | yes | 
| [不追蹤](http://www.aka.ms/notrace) | yes   | yes | 

## <a name="collaborative-translations-framework"></a>共同作業翻譯架構

> [!NOTE]
> 自 2018 年 2 月 1 起，AddTranslation() 和 AddTranslationArray() 不再適用於翻譯工具文字 API 2.0 版。 這些方法將會失敗，不會寫入任何內容。 翻譯工具文字 API 3.0 版不支援這些方法。

>Translator Hub API 中提供類似功能。 請參閱 [https://hub.microsofttranslator.com/swagger](https://hub.microsofttranslator.com/swagger)。 

## <a name="next-steps"></a>後續步驟
> [!div class="nextstepaction"]
> [使用自訂翻譯工具設定自訂的語言系統](http://aka.ms/CustomTranslatorDocs)
