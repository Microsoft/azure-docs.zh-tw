---
title: Bing 自訂搜尋：網站搜尋 | Microsoft Docs
description: 說明如何設定託管的 UI
services: cognitive-services
author: brapel
manager: ehansen
ms.service: cognitive-services
ms.component: bing-custom-search
ms.topic: article
ms.date: 09/28/2017
ms.author: v-brapel
ms.openlocfilehash: 2aec8ba969fb639f2d785a429441f6ed4bbf7dfb
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46987674"
---
# <a name="configure-your-hosted-ui-experience"></a>設定託管的 UI 體驗

設定您的自訂搜尋執行個體之後，您可以呼叫自訂搜尋 API 以取得搜尋結果，並在您的應用程式中加以顯示。 或者，如果您的應用程式是 Web 應用程式，您可以使用自訂搜尋所提供的託管 UI。   

## <a name="configure-custom-hosted-ui"></a>設定自訂託管 UI

若要為您的 Web 應用程式設定託管的 UI，請依照下列步驟執行：

1. 登入自訂搜尋[入口網站](https://customsearch.ai)。  
  
2. 按一下 [自訂搜尋] 執行個體。 若要建立執行個體，請參閱[建立您的第一個 Bing 自訂搜尋執行個體](quick-start.md)。  

3. 按一下 [託管的 UI] 索引標籤。  
  
4. 選取配置。
  
  - 搜尋列和結果 (預設值) &mdash; 此版面配置是傳統搜尋頁面，包含搜尋方塊與搜尋結果。
  - 僅結果 &mdash; 此版面配置只會顯示搜尋結果。 此版面配置不會顯示搜尋方塊。 您必須提供搜尋查詢，方法是在 JavaScript 程式碼片段或 HTML 端點連結中將查詢參數 (&q=\<查詢字串>) 新增到要求 URL。
  - Pop-Over &mdash; 此版面配置不會顯示搜尋方塊，只會在滑動重疊中顯示搜尋結果。
      
5. 選取色彩佈景主題。 可能的佈景主題： 
  
  - 傳統
  - 深色
  - 天藍

  按一下每個佈景主題，看看哪個最適合您的 Web 應用程式。 若需要微調色彩佈景主題來與您的 Web 應用程式完美整合，請按一下 [自訂佈景主題]。 並非所有色彩設定都會套用到所有版面配置佈景主題。 若要變更色彩，請在對應的文字方塊中輸入色彩的 RGB HEX 值 (例如 #366eb8)。 或者，按一下色彩按鈕並按一下適合的色度。 
  
  變更色彩之後，看看您的變更如何影響右邊的預覽範例。 您隨時可以按一下 [重設為預設]，以回到所選佈景主題的預設色彩。

  > [!NOTE]
  > 當您變更色彩佈景主題時，選取色彩的時候請考慮可存取性。

5. 在 [其他組態] 下方，為您的應用程式提供適當的值。 這些設定是選用的。 若要檢視套用或移除這些設定的效果，請查看右側的預覽窗格。 可用的組態選項包括：  
  
  - Web 搜尋組態：
    - 已啟用 Web 結果 &mdash; 決定是否啟用 Web 搜尋 (您會在頁面頂端看到 [Web] 索引標籤)。
    - 啟用自動建議 &mdash; 決定是否啟用自訂建議 (如需額外成本的相關資訊，請參閱[定價](https://azure.microsoft.com/pricing/details/cognitive-services/bing-custom-search/))。
    - 每一頁面的 Web 結果 &mdash; 一次要顯示的 Web 搜尋結果數目 (最大值為每頁 50 個結果)。
    - 影像標題 &mdash; 決定是否要隨著搜尋結果顯示影像。
  
    如果您按一下 [顯示進階設定]，將顯示下列設定。  
  
    - 醒目提示文字 &mdash; 決定是否以粗體顯示結果中包含的搜尋字詞。 
    - 連結目標 &mdash; 決定當使用者按一下搜尋結果時，是要在新的瀏覽器索引標籤 (空白) 中開啟網頁，或在相同的瀏覽器索引標籤 (本身) 中開啟網頁。 

  - 影像搜尋組態：
    - 已啟用影像結果 &mdash; 決定是否要啟用影像搜尋 (您會在頁面頂端看到 [影像] 索引標籤)。   
    - 每一頁面的影像結果 &mdash; 一次要顯示的影像搜尋結果數目 (最大值為每頁 150 個結果)。  
  
    如果您按一下 [顯示進階設定] 將顯示下列設定。  
  
    - 啟用篩選條件 &mdash; 新增使用者可用來篩選 Bing 傳回之影像的篩選條件。 例如，使用者可以篩選只顯示動畫 GIF 的結果。

  - 影片搜尋設定：
    - 已啟用影片結果 &mdash; 決定是否啟用影片搜尋 (您將會在頁面頂端看到 [影片] 索引標籤)。  
    - 每一頁面的影片結果 &mdash; 一次要顯示的影片搜尋結果數目 (最大值為每頁 150 個結果)。
  
    如果您按一下 [顯示進階設定]，將顯示下列設定。  
  
    - 啟用篩選條件 &mdash; 新增使用者可用來篩選 Bing 傳回之影片的篩選條件。 例如，使用者可以篩選特定解析度的影片結果，或篩選在過去 24 小時內探索到的影片。

  - 其他組態：
    - 頁面標題 &mdash; 顯示在搜尋結果頁面標題區域中的文字 (不適用於 Pop-Over 版面配置)。
    - 工具列佈景主題 &mdash; 決定搜尋結果頁面標題區域的背景色彩。  
  
    如果您按一下 [顯示進階設定]，將顯示下列設定。  
  
    - 搜尋方塊文字預留位置 &mdash; 在輸入之前顯示於搜尋方塊中的文字。
    - 標題連結 URL &mdash; 標題連結的目標。
    - 標誌 URL &mdash; 顯示於標題旁的影像。 
    - Favicon URL &mdash; 顯示於瀏覽器標題列中的圖示。  

    只有當您透過 HTML 端點取用託管的 UI 時，系統才會套用下列設定 (若使用 JavaScript 程式碼片段則不會套用)。
    
    - 頁面標題
    - 工具列佈景主題
    - 標題連結 URL
    - 標誌 URL
    - Favicon URL  
  
6. 輸入搜尋訂用帳戶金鑰，或從下拉式清單中選擇。 下拉式清單中會填入來自您 Azure 帳戶之訂用帳戶的金鑰。 請參閱[認知服務 API 帳戶](https://docs.microsoft.com/azure/cognitive-services/cognitive-services-apis-create-account)。  

7. 如果您已啟用自動建議，請輸入自動建議訂用帳戶金鑰，或從下拉式清單中選擇。 下拉式清單中會填入來自您 Azure 帳戶之訂用帳戶的金鑰。 自訂自動建議需要特定的訂用帳戶層，請參閱[定價](https://azure.microsoft.com/pricing/details/cognitive-services/bing-custom-search/)。

> [!NOTE]
> 當您對自訂託管 UI 組態進行變更時，右側的窗格會顯示所做變更的參考。 顯示的搜尋結果並不是您執行個體的實際結果。

[!INCLUDE[publish or revert](./includes/publish-revert.md)]

## <a name="consume-custom-ui"></a>使用自訂 UI

若要使用託管的 UI，請： 

- 在您的網頁中包含指令碼  
  
  ```html
  <html>
      <body>
          <script type="text/javascript" 
              id="bcs_js_snippet"
              src="https://ui.customsearch.ai /api/ux/rendering-js?customConfig=<YOUR-CUSTOM-CONFIG-ID>&market=en-US&safeSearch=Moderate&version=latest&q=">
          </script>
      </body>    
  </html>
  ```

- 或者，您也可以在網頁瀏覽器中使用下列 URL。   
  
  `https://ui.customsearch.ai/hosted?customConfig=YOUR-CUSTOM-CONFIG-ID`  
  
  > [!NOTE]
  > 視需要將下列的查詢參數加入 URL。 如需有關這些參數的資訊，請參閱[自訂搜尋 API](https://docs.microsoft.com/rest/api/cognitiveservices/bing-custom-search-api-v7-reference#query-parameters) \(英文\) 參考。
  >
  > - q
  > - mkt
  > - safesearch
  > - setlang

  > [!IMPORTANT]
  > 頁面無法顯示您的隱私權聲明或其他通知和條款。 對您的適用性可能有所不同。  

如需相關資訊 (包括您的自訂組態識別碼)，請移至 [生產] 索引標籤下的 [端點]。

## <a name="next-steps"></a>後續步驟

- [使用裝飾標記醒目提示文字](./hit-highlighting.md)
- [頁面網頁](./page-webpages.md)