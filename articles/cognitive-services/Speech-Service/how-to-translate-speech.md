---
title: 利用語音服務翻譯語音 | Microsoft Docs
description: 了解如何使用語音服務中的語音翻譯。
titleSuffix: Microsoft Cognitive Services
services: cognitive-services
author: v-jerkin
manager: noellelacharite
ms.service: cognitive-services
ms.component: speech-service
ms.topic: article
ms.date: 07/16/2018
ms.author: v-jerkin
ms.openlocfilehash: 6acfcf0ae8ab4c63e6cc943f93da6b947f3d118c
ms.sourcegitcommit: 0b05bdeb22a06c91823bd1933ac65b2e0c2d6553
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/17/2018
ms.locfileid: "39071091"
---
# <a name="translate-speech-using-speech-service"></a>利用語音服務翻譯語音

[語音 SDK](speech-sdk.md) 是在您的應用程式中使用語音翻譯的最簡單方式。 SDK 提供服務的完整功能。 執行語音翻譯的基本程序包括下列步驟：

1. 建立語音處理站，並提供語音服務訂用帳戶金鑰和[區域](regions.md)或授權權杖。 您也會設定來源和目標翻譯語言，以及指定是否要輸出文字或語音。

2. 從處理站取得辨識器。 如需翻譯，選取翻譯辨識器。 (其他辨識器適用於「語音轉換文字」。)根據您使用的音訊來源，翻譯辨識器有各種類型。

4. 繫結事件以便執行非同步作業 (如有需要)。 當辨識器具有過渡期和最終結果時，它會呼叫事件處理常式。 否則，您的應用程式會收到最終翻譯結果。

5. 開始辨識及翻譯。

# <a name="sdk-samples"></a>SDK 範例

如需最新的範例集，請參閱[認知服務語音 SDK 範例 GitHub 存放庫](https://aka.ms/csspeech/samples) (英文)。

# <a name="next-steps"></a>後續步驟

- [試用認知服務](https://azure.microsoft.com/try/cognitive-services/)
- [以 C# 辨識語音](quickstart-csharp-dotnet-windows.md)
