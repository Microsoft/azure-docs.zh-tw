---
title: 知識探索服務 API 中的 Web API 介面 | Microsoft Docs
description: 使用 Web API 介面，在認知服務的知識探索服務 (KES) API 中打造豐富的語意搜尋體驗。
services: cognitive-services
author: bojunehsu
manager: stesp
ms.service: cognitive-services
ms.component: knowledge-exploration
ms.topic: article
ms.date: 03/26/2016
ms.author: paulhsu
ms.openlocfilehash: 16c5680eb4f249a5d37e6b90eea92cfff7090eef
ms.sourcegitcommit: 95d9a6acf29405a533db943b1688612980374272
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/23/2018
ms.locfileid: "35367951"
---
# <a name="web-api-interface"></a>Web API 介面
透過一組 Web API，可以裝載和存取知識探索服務所建立的模型檔案。  使用 [`host_service`](CommandLine.md#host_service-command) 命令可以將這些 API 裝載於本機電腦，或使用 [`deploy_service`](CommandLine.md#deploy_service-command) 命令將其部署到 Azure 雲端服務。  這兩種技術均可公開下列 API 端點：
* [*interpret*](interpretMethod.md) – 解譯自然語言查詢字串。 傳回已標註的解譯來獲得豐富的搜尋方塊自動完成體驗，進而預期使用者所鍵入的項目。
* [*評估*](evaluateMethod.md) – 評估並傳回結構化查詢運算式的輸出。
* [*calchistogram*](calchistogramMethod.md) – 針對結構化查詢運算式所傳回的物件計算屬性值長條圖。

一起使用這些 API，即可打造豐富的語意搜尋體驗。  假設有一個自然語言查詢字串，*解譯*方法會根據基礎文法和索引資料，利用結構化查詢運算式提供輸入查詢的註解式版本。  *評估*方法會評估結構化查詢運算式，並傳回相符的索引物件以供顯示。  *calchistogram* 方法會計算屬性值分佈，以啟用篩選和精簡搜尋功能。

**範例**

在學術出版品領域中，如果使用者輸入字串 "latent s"，*解譯*方法會提供一組有排名的解譯，並暗示使用者可能會搜尋關鍵字 "latent semantic analysis"，標題 "latent structure analysis" 或其他開頭為 "latent s" 的運算式。  這項資訊可用來快速將使用者指引至所需的搜尋結果。

在這個領域中，*評估*方法可用於從學術索引中擷取一組相符的出版品，而 *calchistogram* 方法可用於計算相符出版品的屬性值分佈，其可用來進一步篩選並精簡搜尋結果。

請注意，為了改善範例的可讀性，REST API 呼叫包含未經過 URL 編碼的字元 (例如空格)。 您的程式碼必須套用適當的 URL 編碼。
