---
author: wolfma61
ms.service: cognitive-services
ms.topic: include
ms.date: 09/24/2018
ms.author: wolfma
ms.openlocfilehash: ab4795325f733d15403b53f027daa4fb8c824cc6
ms.sourcegitcommit: 5b8d9dc7c50a26d8f085a10c7281683ea2da9c10
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/26/2018
ms.locfileid: "47186188"
---
<!-- N.B. no header, language-agnostic -->

Microsoft 認知服務[語音 SDK](~/articles/cognitive-services/speech-service/speech-sdk.md) 可讓您以最簡單的方式在應用程式中使用**語音翻譯**。
SDK 提供服務的完整功能。 執行語音翻譯的基本程序包括下列步驟：

1. 建立語音翻譯設定，並提供語音服務訂用帳戶金鑰或授權權杖和[區域](~/articles/cognitive-services/speech-service/regions.md)作為參數。 請視需要變更設定。 例如，您可以設定來源和目標翻譯語言，以及指定要輸出文字還是語音。

1. 從語音翻譯設定中建立翻譯辨識器。 如果您想要用預設麥克風 (例如，音訊資料流或音訊檔案) 以外的來源進行辨識，請提供音訊設定。

1. 繫結事件以便執行非同步作業 (如有需要)。 當辨識器具有過渡期和最終結果，以及選用音訊輸出的整合事件時，它會接著呼叫事件處理常式。 否則，您的應用程式只會收到最終轉譯結果。

1. 開始辨識。 針對一次性翻譯請使用 `RecognizeOnceAsync()` 方法，這會傳回第一個可辨識的語句。 針對長時間執行的翻譯請使用 `StartContinuousRecognitionAsync()` 方法，並繫結非同步辨識結果的事件。

請參閱使用語音 SDK 的語音翻譯案例所適用的下列程式碼片段。

[!INCLUDE [Get a subscription key](cognitive-services-speech-service-get-subscription-key.md)]
