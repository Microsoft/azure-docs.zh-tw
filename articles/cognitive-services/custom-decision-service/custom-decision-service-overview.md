---
title: 什麼是自訂決策服務？
titlesuffix: Azure Cognitive Services
description: 此文章提供自訂決策服務的概觀。
services: cognitive-services
author: alekh
manager: cgronlun
ms.service: cognitive-services
ms.component: custom-decision-service
ms.topic: overview
ms.date: 05/08/2018
ms.author: slivkins
ms.openlocfilehash: 273f2965a0fcaaa729175c5232da1aba69589eec
ms.sourcegitcommit: ce526d13cd826b6f3e2d80558ea2e289d034d48f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/19/2018
ms.locfileid: "46364198"
---
# <a name="what-is-custom-decision-service"></a>什麼是自訂決策服務？

在典型的 Web 或行動應用程式中，前端頁面會連結至數篇文章或其他類型的內容。 前端頁面載入時，可能會要求自訂決策服務對該前端頁面上包含的文章進行排名。 因此，當使用者點選某篇文章時，第二個要求將可傳送至自訂決策服務，以記錄該使用者決策的結果。

您只需要內容的 RSS 摘要，並將數行 JavaScript 加入至您的應用程式，即可輕鬆使用自訂決策服務。

自訂決策服務會將您的內容轉換為機器學習的特徵。 系統會使用這些特徵，就文字、影像、視訊和整體情緒等方面了解您的內容。 它也會使用其他數種 [Microsoft 認知服務](https://www.microsoft.com/cognitive-services)，像是[實體連結](../entitylinking/home.md)、[文字分析](../text-analytics/overview.md)、[情緒](../emotion/home.md)和[電腦視覺](../computer-vision/home.md)等。

自訂決策服務的一些常見使用案例包括：

* 個人化新聞網站上的文章
* 個人化媒體入口網站上的視訊內容
* 最佳化廣告版位或廣告所導向的網頁
* 排名購物網站上的推薦項目。

自訂決策服務目前為*免費公開預覽版*。 它可以個人化網站或應用程式上的文章清單。 英文內容的特徵擷取功效最佳。 對於其他語言則提供[有限的功能](../text-analytics/overview.md)，例如西班牙文、法文、德文、葡萄牙文和日文。 此文件將在有新功能可供使用時修訂。

自訂決策服務適用於不在內容個人化區域中的應用程式。 這些應用程式可能適合使用自訂預覽。 請[與我們連絡](https://azure.microsoft.com/overview/sales-number/)以了解詳情。

## <a name="api-usage-modes"></a>API 使用模式

自訂決策服務可套用至網頁與行動應用程式。 API 可從瀏覽器或應用程式呼叫。 兩者的 API 使用方式很類似，但在某些方面有些許不同。

## <a name="glossary-of-terms"></a>詞彙解釋

數個經常出現在此文件中的詞彙包括：

* **動作集**：自訂決策服務要排名的內容項目集合。 此集合可指定為 *RSS* 或 *Atom* 端點。
* **排名**：對自訂決策服務的每個要求會指定一或多個動作集。 系統會從這些集合中選擇所有內容選項，並依排名順序傳回，以進行回應。
* **回呼函式**：此函式 (由您指定) 會呈現您 UI 中的內容。 內容會依據自訂決策服務所傳回的排名順序來排序。
* **報酬**：此量值代表使用者對呈現內容的回應程度。 自訂決策服務會藉由點擊次數來測量使用者回應。 點擊次數會透過在您的應用程式中插入的自訂程式碼回報給系統。

## <a name="next-steps"></a>後續步驟

* 向自訂決策服務[註冊您的應用程式](custom-decision-service-get-started-register.md)
* 開始最佳化[網頁](custom-decision-service-get-started-browser.md)或[智慧型手機應用程式](custom-decision-service-get-started-app.md)。
* 請參閱 [API 參考](custom-decision-service-api-reference.md)以深入了解提供的功能。