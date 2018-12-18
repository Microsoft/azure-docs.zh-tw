---
title: Bing 自訂搜尋：搜尋自訂檢視 | Microsoft Docs
description: 描述如何搜尋網頁的自訂檢視
services: cognitive-services
author: brapel
manager: ehansen
ms.service: cognitive-services
ms.component: bing-custom-search
ms.topic: article
ms.date: 09/28/2017
ms.author: v-brapel
ms.openlocfilehash: 7a2db7881dfe7efedb1a83637a6281f786652958
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46964881"
---
# <a name="call-your-custom-search"></a>呼叫您的自訂搜尋

在您第一次呼叫自訂搜尋 API 取得執行個體的搜尋結果之前，必須先取得認知服務訂用帳戶金鑰。 若要取得自訂搜尋 API 的金鑰，請參閱[試用認知服務](https://azure.microsoft.com/try/cognitive-services/?api=bing-custom-search)。


## <a name="try-it-out"></a>試做

設定自訂的搜尋體驗之後，您可以從自訂搜尋入口網站中測試設定。 

1. 登入[自訂搜尋](https://customsearch.ai)。
2. 從您的執行個體清單中按一下 [自訂搜尋] 執行個體。
3. 按一下 [生產] 索引標籤。 
4. 在 [端點] 索引標籤下，選取端點 (例如 Web API)。 您的訂用帳戶會決定要顯示哪些端點 (請參閱[定價](https://azure.microsoft.com/pricing/details/cognitive-services/bing-custom-search/)以取得訂用帳戶選項)。 
5. 指定參數值。 

    下列是您可以設定的參數 (實際清單取決於選取的端點)。 如需有關這些參數的詳細資訊，請參閱[自訂搜尋 API](https://docs.microsoft.com/rest/api/cognitiveservices/bing-custom-search-api-v7-reference#query-parameters) \(英文\) 參考。

    - **查詢**：要搜尋的搜尋字詞。 僅適用於網頁、影像、影片與自動建議端點。
    - **自訂設定識別碼**：所選取自訂搜尋執行個體的設定識別碼。 這是唯讀欄位。
    - **市場**：結果來自的市場。 僅適用於網頁、影像、影片與裝載 UI 端點。
    - **訂用帳戶金鑰**：要用來測試的訂用帳戶金鑰。 您可以從下拉式清單中選取金鑰，或手動輸入一個。  
      
    按一下 [其他參數] 會顯示下列參數：  
      
    - **安全搜尋**：用來篩選成人內容網頁的篩選條件。 僅適用於網頁、影像、影片與裝載 UI 端點。
    - **使用者介面語言**：要用於使用者介面字串的語言。 例如，若您在裝載 UI 中啟用影像與影片， [影像] 與 [影片] 索引標籤會使用指定的語言。
    - **計數**：回應中傳回的搜尋結果數目。 僅適用於網頁、影像與影片端點。
    - **位移**：傳回結果之前要跳過的搜尋結果數目。 僅適用於網頁、影像與影片端點。

6. 指定所有必要選項之後，請按一下 [呼叫] 在右窗格中檢視 JSON 回應。 

如果您選取裝載 UI 端點，可以從底部窗格測試搜尋體驗。

## <a name="next-steps"></a>後續步驟

- [使用 C# 呼叫您的自訂檢視](./call-endpoint-csharp.md)
- [使用 Java 呼叫您的自訂檢視](./call-endpoint-java.md)
- [使用 NodeJs 呼叫您的自訂檢視](./call-endpoint-nodejs.md)
- [使用 Python 呼叫您的自訂檢視](./call-endpoint-python.md)

- [使用 C# SDK 呼叫您的自訂檢視](./sdk-csharp-quick-start.md)