---
title: Azure 上的語音轉換文字服務常見問題集
description: 以下是有關語音轉換文字最熱門問題的解答。
services: cognitive-services
author: PanosPeriorellis
ms.service: cognitive-services
ms.component: custom-speech
ms.topic: article
ms.date: 06/11/2018
ms.author: panosper
ms.openlocfilehash: 8d70c4a359c713d6c5f46423193e9c9e7e1f3baf
ms.sourcegitcommit: 068fc623c1bb7fb767919c4882280cad8bc33e3a
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/27/2018
ms.locfileid: "39282852"
---
# <a name="text-to-speech-frequently-asked-questions"></a>文字轉換語音常見問題集

如果您在此常見問題集中找不到問題的解答，請參閱[此處](support.md)的其他支援選項。

## <a name="general"></a>一般

**問**：標準與自訂語音模型之間有何差異？

**答**：標準語音模型 (也稱為 聲音音調) 已使用 Microsoft 所擁有的資料加以定型，並已在雲端中部署。 自訂語音模型可讓使用者調整平均模型，並根據說話者的聲音風格轉變音色和表達方式，或根據使用者準備的定型資料，為全新的模型定型。 現在有越來越多的客戶希望他們的聊天機器人有獨一無二的品牌化聲音。 自訂語音建置平台是最適合的選擇。

**問**：如果我想要使用標準語音模型，該從何處著手？

**答**：透過 HTTP 要求可提供超過 45 種語言的 80 餘種標準語音模型。 首先，您必須取得[訂用帳戶金鑰](https://docs.microsoft.com/en-us/azure/cognitive-services/speech-service/get-started)。 若要對預先部署的語音模型進行 REST 呼叫，請參閱[這裡的詳細資料](https://docs.microsoft.com/en-us/azure/cognitive-services/speech-service/rest-apis#text-to-speech)。

**問**：如果我想要使用自訂語音模型，API 是否與標準語音相同？

**答**：當您已經建立並部署自訂語音模型時，您會取得一個適用於您模型的唯一端點。 您必須在您的 HTTP 要求中指定端點，才能使用語音功能在您的應用程式中說話。 透過 REST API 可以使用相同的功能，因為文字轉換語音服務也適用於自訂端點。 了解如何[建立及使用您的自訂端點](https://docs.microsoft.com/en-us/azure/cognitive-services/speech-service/how-to-customize-voice-font#create-and-use-a-custom-endpoint)。

**問**：我需要準備定型資料來建立自己的自訂語音模型嗎？

**答**：您必須自行準備定型資料。 建立自訂語音模型需要有語音資料集合。 此集合包含一組語音錄製的音訊檔案，以及各音訊檔案文字記錄的文字檔。 數位語音的結果與定型資料的品質息息相關。 若要產生良好的 TTS 語音，請務必在備有高品質站立式麥克風的安靜房間內完成錄音。 一致的音量、讀出速率、讀出音調，甚至一致的語音表達方式是建立絕佳數位語音不可或缺的一部分。 我們強烈建議您應該在錄音室中錄製語音。
目前我們不提供線上錄音支援，也沒有任何錄音室建議。 針對格式需求，請參閱[如何準備錄音和文字記錄](https://docs.microsoft.com/en-us/azure/cognitive-services/speech-service/how-to-customize-voice-font#prepare-recordings-and-transcripts)
 
**問**：我應該使用哪些腳本錄製語音資料以進行自訂語音定型？ 

**答**：我們沒有限制錄音的腳本。 您可以使用自己的腳本錄製語音。 只要確定語音資料中有足夠的語音涵蓋範圍。 若要為自訂語音定型，您可以從少量的語音資料開始，可能是 50 個不同的句子 (需要說 3-5 分鐘)。 您提供的資料越多，您的聲音聽起來會越自然。 當您提供超過 2000 個句子的錄音 (大約說 3 到 4 小時) 時，您就可以開始為完整的聲音音調定型。 若要取得高品質的完整語音，您必須準備超過 6000 個句子的錄音 (大約說 8 到 10 小時)。  
我們提供額外的服務來協助您準備錄音的腳本。 相關事宜，請連絡[自訂語音客戶支援](mailto:customvoice@microsoft.com?subject=Inquiries%20about%20scripts%20generation%20for%20Custom%20Voice%20creation)。

**問**：如果我需要比預設值或入口網站所提供值更高的並行處理，該怎麼辦？

**答**：您可以使用 20 個並行要求的增量調升模型規模。 關於進階調整的相關事宜，請連絡[自訂語音客戶支援](mailto:customvoice@microsoft.com?subject=Inquiries%20about%20scripts%20generation%20for%20Custom%20Voice%20creation)。

**問**：我可以在本機下載並執行模型嗎？

**答**：您無法在本機下載並執行模型。

## <a name="next-steps"></a>後續步驟

* [疑難排解](troubleshooting.md)
* [版本資訊](releasenotes.md)
