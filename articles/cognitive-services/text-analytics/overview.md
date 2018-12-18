---
title: 什麼是文字分析？
titleSuffix: Azure Cognitive Services
description: Azure 認知服務中的「文字分析」包含情感分析、關鍵片語擷取、語言偵測和實體連結。
services: cognitive-services
author: ashmaka
manager: cgronlun
ms.service: cognitive-services
ms.component: text-analytics
ms.topic: overview
ms.date: 10/01/2018
ms.author: ashmaka
ms.openlocfilehash: fcc10723b2fb750b099f99999de01ab699d5d5e8
ms.sourcegitcommit: 55952b90dc3935a8ea8baeaae9692dbb9bedb47f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/09/2018
ms.locfileid: "48887763"
---
# <a name="what-is-text-analytics"></a>什麼是文字分析？

文字分析 API 是一種雲端式服務，可針對原始的文字提供進階的自然語言處理，同時包含四種主要功能：情感分析、關鍵片語擷取、語言偵測和實體連結。

此 API 由 [Microsoft 認知服務](https://docs.microsoft.com/azure/cognitive-services/)中的資源提供技術支援，是雲端中的機器學習和 AI 演算法所組成的集合，隨時可供您在開發專案中取用。

## <a name="capabilities-in-text-analytics"></a>文字分析中的功能

文字分析可能代表不同的意義，但在認知服務中，文字分析 API 可提供四種類型的分析，如下表所說明。

| 作業| 說明 | API |
|-----------|-------------|------|
|[**情感分析**](how-tos/text-analytics-how-to-sentiment-analysis.md) | 藉由分析原始文字而獲得有關於正面或負面情感的線索，以了解客戶對您的品牌或主題有何看法。 此 API 會為每份文件傳回 0 到 1 之間的情感分數，1 代表最正面的情感。<br /> 分析模型會使用大量文字主體和 Microsoft 的自然語言技術預先定型。 針對[選取的語言](text-analytics-supported-languages.md)，API 可對您所提供的任何原始文字進行分析及評分，並直接將結果傳回至呼叫端應用程式。 | [REST](https://westus.dev.cognitive.microsoft.com/docs/services/TextAnalytics.V2.0/operations/56f30ceeeda5650db055a3c9) <br /> [.NET](https://docs.microsoft.com/azure/cognitive-services/text-analytics/quickstarts/csharp#install-the-nuget-sdk-package)  |
|[**關鍵片語擷取**](how-tos/text-analytics-how-to-keyword-extraction.md) | 自動擷取關鍵片語，以快速識別重點。 例如，若輸入文字為 "The food was delicious and there were wonderful staff"，API 即會傳回主要討論要點："food" 和 "wonderful staff"。  | [REST](https://westus.dev.cognitive.microsoft.com/docs/services/TextAnalytics.V2.0/operations/56f30ceeeda5650db055a3c6) <br /> [.NET](https://docs.microsoft.com/azure/cognitive-services/text-analytics/quickstarts/csharp#install-the-nuget-sdk-package) |
|[**語言偵測**](how-tos/text-analytics-how-to-language-detection.md) | 偵測輸入文字是以何種語言撰寫的，並針對要求所提交的每份文件回報單一語言代碼，最多可達 120 種語言。 語言代碼各配有一個分數，表示分數的強度。 | [REST](https://westus.dev.cognitive.microsoft.com/docs/services/TextAnalytics.V2.0/operations/56f30ceeeda5650db055a3c7) <br />  [.NET](https://docs.microsoft.com/azure/cognitive-services/text-analytics/quickstarts/csharp#install-the-nuget-sdk-package) | 
|[**實體辨識 (預覽)**](how-tos/text-analytics-how-to-entity-linking.md) | 識別文字中的實體並分類為人員、位置、組織、日期/時間、數量、百分比、貨幣等等。 已知的實體也會辨識出來，並連結至網路上的其他資訊。 | [REST](https://westus.dev.cognitive.microsoft.com/docs/services/TextAnalytics-V2-1-Preview/operations/5ac4251d5b4ccd1554da7634) | 

## <a name="typical-workflow"></a>一般工作流程

工作流程很簡單：您提交資料以進行分析，並在程式碼中處理輸出。 分析器會依原狀使用，不需要額外的設定或自訂。

1. [註冊](https://docs.microsoft.com/azure/cognitive-services/cognitive-services-apis-create-account)[存取金鑰](how-tos/text-analytics-how-to-access-key.md)。 每次要求時都必須傳入此金鑰。

2. [編寫要求](how-tos/text-analytics-how-to-call-api.md#json-schema)，其中包含 JSON 格式的原始非結構化文字，作為您的資料。

3. 將要求發佈至註冊期間所建立的端點，並附上所需的資源：情感分析、關鍵片語擷取、語言偵測或實體識別。

4. 將回應串流處理或儲存至本機。 視要求之不同，結果可能是情感分數、擷取的關鍵片語集合或語言代碼。

輸出會根據識別碼以單一 JSON 文件的形式傳回，且附有您發佈的每個文字文件所產生的結果。 您可在後續分析和視覺化結果，或將結果分類為可操作的深入解析。

資料不會儲存在您的帳戶中。 文字分析 API 所執行的作業是無狀態的，這表示您提供的文字經處理後隨即傳回結果。

<a name="supported-languages"></a>

## <a name="supported-languages"></a>支援的語言

本節已移至個別的文章，以利說明。 如需此內容，請參閱[文字分析 API 中支援的語言](text-analytics-supported-languages.md)。

<a name="data-limits"></a>

## <a name="data-limits"></a>資料限制

所有的文字分析 API 端點均接受原始文字資料。 目前的限制是每個文件 5000 個字元；如果您需要分析較大的文件，您可以其分成較小的區塊。 如果您仍需要更高的限制，請[與我們連絡](https://azure.microsoft.com/overview/sales-number/)，以便共同討論您的需求。

| 限制 | 值 |
|------------------------|---------------|
| 單一文件的大小上限 | 依 `String.Length` 測量的 5,000 個字元。 |
| 整體要求的大小上限 | 1 MB |
| 要求中的文件數上限 | 1,000 份文件 |

速率限制為每分鐘呼叫 100 次。 請注意，您可以在單一呼叫中提交大量文件 (最多 1000 份文件)。

## <a name="unicode-encoding"></a>Unicode 編碼

針對文字表示法和字元計數計算，文字分析 API 會使用 Unicode 編碼。 要求可使用 UTF-8 和 UTF-16 來提交，兩者的字元計數並沒有明顯差異。 Unicode 字碼指標會作為字元長度的啟發學習法，且會被視為等同於文字分析資料限制的用途。 如果您使用 `String.Length` 取得字元計數，您所使用就是我們用來測量資料大小的相同方法。

## <a name="next-steps"></a>後續步驟

首先，試著使用[互動式示範](https://azure.microsoft.com/services/cognitive-services/text-analytics/)。 您可以貼上文字輸入 (最多 5,000 個字元) 以偵測語言 (最多 120 種)、計算情感分數，或擷取關鍵片語。 您不需要註冊。

當您準備好要直接呼叫 API 時：

+ [註冊](how-tos/text-analytics-how-to-signup.md)存取金鑰，並檢閱[呼叫 API](how-tos/text-analytics-how-to-call-api.md) 的步驟。

+ [快速入門](quickstarts/csharp.md)是以 C# 撰寫的 REST API 的呼叫適用的逐步解說。 請了解如何以最少的程式碼提交文字、選擇分析，以及檢視結果。

+ [API 參考文件](//go.microsoft.com/fwlink/?LinkID=759346)提供 API 的技術文件。 這些文件支援內嵌呼叫，因此您可以從每個文件頁面中呼叫 API。

+ [外部和社群內容](text-analytics-resource-external-community.md)提供部落格貼文清單，並提供影片示範如何搭配其他工具和技術來使用文字分析。

## <a name="see-also"></a>另請參閱

 [認知服務文件頁面](https://docs.microsoft.com/azure/cognitive-services/)
