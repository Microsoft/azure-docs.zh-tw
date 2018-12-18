---
author: wolfma61
ms.service: cognitive-services
ms.topic: include
ms.date: 09/24/2018
ms.author: wolfma
ms.openlocfilehash: 3508f809ab89188e46145df064cbb53ca78c8f9f
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "47021692"
---
<!-- N.B. no header, language-agnostic -->

Microsoft 認知服務[語音 SDK](~/articles/cognitive-services/speech-service/speech-sdk.md)可用來**從語音辨識意圖**，且受到認知服務 [Language Understanding 服務 (LUIS)](https://www.luis.ai/home) 的支援。

1. 使用 LUIS 訂用帳戶金鑰和[區域](~/articles/cognitive-services/speech-service/regions.md#regions-for-intent-recognition)作為參數，建立語音設定。 LUIS 訂用帳戶金鑰在服務文件中稱為**端點金鑰**。 您無法使用 LUIS 撰寫金鑰。 (請參閱本節稍後的注意事項。)

1. 從語音設定建立意圖辨識器。 如果您想要用預設麥克風以外的來源 (例如，音訊資料流或音訊檔案) 進行辨識，請提供音訊設定。

1. 取得以您的 **AppId** 為基礎的語言理解模型。 新增您所需的意圖。 

1. 繫結事件以便執行非同步作業 (如有需要)。 當辨識器具有過渡期和最終結果 (包括意圖) 時，它會接著呼叫事件處理常式。 如果您未繫結事件，則應用程式只會收到最終轉譯結果。

1. 開始進行意圖辨識。 針對一次性辨識 (例如命令或查詢辨識)，請使用 `RecognizeOnceAsync()` 方法。 此方法會傳回第一個辨識出來的語句。 針對長時間執行的辨識，請使用 `StartContinuousRecognitionAsync()` 方法。 請繫結事件，以產生非同步辨識結果。

請參閱使用語音 SDK 的意圖辨識案例所適用的下列程式碼片段。 請將範例中的值取代為您自己的 LUIS 訂用帳戶金鑰 (端點金鑰)、[訂用帳戶的區域](~/articles/cognitive-services/speech-service/regions.md#regions-for-intent-recognition)和意圖模型的 **AppId**。

> [!NOTE]
> 相較於語音 SDK 所支援的其他服務，意圖辨識需要特定的訂用帳戶金鑰 (LUIS 端點金鑰)。 如需意圖辨識技術的相關資訊，請參閱 [LUIS 網站](https://www.luis.ai)。 如需如何取得**端點金鑰**的相關資訊，請參閱[建立 LUIS 端點金鑰](https://docs.microsoft.com/azure/cognitive-services/LUIS/luis-how-to-azure-subscription#create-luis-endpoint-key)。
