---
title: LUIS 中的資料轉換概念 - Language Understanding
titleSuffix: Azure Cognitive Services
description: 了解在 Language Understanding (LUIS) 中的預測之前如何改變語句
services: cognitive-services
author: diberry
manager: cgronlun
ms.service: cognitive-services
ms.component: language-understanding
ms.topic: article
ms.date: 09/10/2018
ms.author: diberry
ms.openlocfilehash: 3862a0dbd94b5764cf506b05201d8dc60430fc7d
ms.sourcegitcommit: 4ecc62198f299fc215c49e38bca81f7eb62cdef3
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "47038964"
---
# <a name="data-conversion-concepts-in-luis"></a>LUIS 中的資料轉換概念
LUIS 會使用認知服務語音服務，在預測之前將口語語句轉換成文字語句。 

## <a name="speech-to-intent-conversion-concepts"></a>語音意圖轉換概念
LUIS 的語音轉換文字功能可讓您將口頭語句傳送到端點並接收 LUIS 預測回應。 此流程整合了[語音](https://docs.microsoft.com/azure/cognitive-services/Speech)服務與 LUIS。 

### <a name="key-requirements"></a>重要需求
您不必為此整合建立 **Bing 語音 API** 金鑰。 在 Azure 入口網站中建立的 **Language Understanding** 金鑰適用於此整合。 請勿使用 LUIS 啟動器金鑰，該金鑰不適用於此整合。

### <a name="new-endpoint"></a>新增端點 
此整合建立了新的端點和[價格](luis-boundaries.md#key-limits)模式。 端點透過[語音 SDK](https://github.com/Azure-Samples/cognitive-services-speech-sdk) 能夠接受語音和文字語句，讓您將此端點當作單一端點。 

### <a name="quota-usage"></a>配額使用方式
相關資訊請參閱[重要限制](luis-boundaries.md#key-limits)。 

### <a name="data-retention"></a>資料保留
透過語音 SDK 傳送到端點的資料，無論是語音還是文字，都只能用來增強您的語音模式。 除了您的模式之外，不會在一般容量中增強語音或 LUIS。 刪除 LUIS 應用程式後，也會刪除保留的資料。

<!-- TBD: Machine translation conversion concepts -->

## <a name="next-steps"></a>後續步驟

> [!div class="nextstepaction"]
> [使用語音轉換文字](luis-tutorial-speech-to-intent.md)

