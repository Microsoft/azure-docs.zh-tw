---
title: 使用 REST API 辨識語音
description: 了解如何在認知服務語音服務中使用語音轉換文字 API。
titleSuffix: Microsoft Cognitive Services
services: cognitive-services
author: erhopf
ms.service: cognitive-services
ms.component: speech-service
ms.topic: article
ms.date: 07/16/2018
ms.author: erhopf
ms.openlocfilehash: 4e87feae7e3321efe7adcdbf1302762f539f034d
ms.sourcegitcommit: c282021dbc3815aac9f46b6b89c7131659461e49
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/12/2018
ms.locfileid: "49167268"
---
# <a name="recognize-speech-by-using-the-rest-api"></a>使用 REST API 辨識語音

[!INCLUDE [Selector](../../../includes/cognitive-services-speech-service-how-to-recognize-speech-selector.md)]

您可以使用 REST API 及 HTTP POST 要求辨識簡短的語句。

如果您不是使用 [SDK](speech-sdk.md) 所支援的語言，則 REST API 是最簡單的語音辨識方式。 您會對服務端點所提出的 HTTP POST 要求，並且在要求本文中傳遞整個語句。 您會收到包含已辨識文字的回應。

> [!NOTE]
> 使用 REST API 時，語句會受限於 15 秒或更短。
> 若要辨識較長的語句，請參閱[語音 SDK](how-to-recognize-speech-csharp.md)。

如需 [語音轉換文字] REST API 的詳細資訊，請參閱 [REST API](rest-apis.md#speech-to-text) 一文。 若要查看運作中的 API，可從 GitHub 下載 [REST API 範例](https://github.com/Azure-Samples/SpeechToText-REST)。

## <a name="next-steps"></a>後續步驟

- 請參閱 [REST API 概觀](rest-apis.md)。
