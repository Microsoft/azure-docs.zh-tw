---
title: 教學課程：建立自訂搜尋網頁 - Bing 自訂搜尋
titlesuffix: Azure Cognitive Services
description: 描述如何設定自訂搜尋執行個體，並將它整合到網頁。
services: cognitive-services
author: brapel
manager: cgronlun
ms.service: cognitive-services
ms.component: bing-custom-search
ms.topic: tutorial
ms.date: 10/16/2017
ms.author: v-brapel
ms.openlocfilehash: 3e892131a0109d2fff924940542b5d8b2b701950
ms.sourcegitcommit: 6f59cdc679924e7bfa53c25f820d33be242cea28
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/05/2018
ms.locfileid: "48815370"
---
# <a name="tutorial-build-a-custom-search-web-page"></a>教學課程：建置自訂搜尋網頁

Bing 自訂搜尋可讓您針對感興趣的主題，建立量身訂做的搜尋經驗。 例如，如果您的武術網站提供搜尋體驗，可以指定 Bing 搜尋的網域、子網站及網頁。 您的使用者會看到針對他們感興趣的內容而量身訂做的搜尋結果，不需要逐頁查看可能包含不相關內容的一般搜尋結果。 

本教學課程示範如何設定自訂搜尋執行個體，並將它整合到新的網頁。

涵蓋的工作包括：

> [!div class="checklist"]
> - 建立自訂搜尋執行個體
> - 加入使用中的項目
> - 加入已封鎖的項目
> - 加入已釘選的項目
> - 將自訂搜尋整合到網頁

## <a name="prerequisites"></a>必要條件

- 若要依照本教學課程，您需要 Bing 自訂搜尋 API 的訂用帳戶金鑰。  若要取得金鑰，請參閱[試用認知服務](https://azure.microsoft.com/try/cognitive-services/?api=bing-custom-search)。
- 如果尚未安裝 Visual Studio 2017，您可以下載並使用**免費的** [Visual Studio 2017 Community 版本](https://www.visualstudio.com/downloads/)。

## <a name="create-a-custom-search-instance"></a>建立自訂搜尋執行個體

建立 Bing 自訂搜尋執行個體：

1. 開啟網際網路瀏覽器。  
  
2. 瀏覽至自訂搜尋[入口網站](https://customsearch.ai)。  
  
3. 使用 Microsoft 帳戶 (MSA) 登入入口網站。 如果您沒有 MSA，請按一下 [建立 Microsoft 帳戶]。 如果這是您第一次使用入口網站，它會要求您提供存取資料的權限。 按一下 [是] 。  
  
4. 登入之後，按一下 [新增自訂搜尋]。 在 [建立新的自訂搜尋執行個體] 視窗中，輸入有意義的名稱且名稱能夠描述搜尋傳回的內容類型。 您可以隨時變更名稱。  
  
  ![[建立新的自訂搜尋執行個體] 方塊的螢幕擷取畫面](../media/newCustomSrch.png)  
  
5. 按一下 [確定]，指定 URL，以及是否要包括 URL 的子頁面：  
  
  ![URL 定義頁面的螢幕擷取畫面](../media/newCustomSrch1-a.png)  


## <a name="add-active-entries"></a>加入使用中的項目

若要包括來自特定網站或 URL 的結果，請將它們加入到 [使用中] 索引標籤。

1.  在 [設定] 頁面上，按一下 [使用中] 索引標籤，並輸入要在搜尋中包括的一或多個網站的 URL。

    ![定義編輯器 [使用中] 索引標籤的螢幕擷取畫面](../media/customSrchEditor.png)

2.  若要確認您的執行個體會傳回結果，請在右邊的預覽窗格中輸入查詢。 Bing 只會針對已編製索引的公用網站傳回結果。

## <a name="add-blocked-entries"></a>加入已封鎖的項目

若要排除來自特定網站或 URL 的結果，請將它們加入到 [已封鎖] 索引標籤。

1. 在 [設定] 頁面上，按一下 [已封鎖] 索引標籤，並輸入要從搜尋中排除的一或多個網站的 URL。

    ![定義編輯器 [已封鎖] 索引標籤的螢幕擷取畫面](../media/blockedCustomSrch.png)


2. 若要確認您的執行個體不會傳回來自已封鎖網站的結果，請在右邊的預覽窗格中輸入查詢。 

## <a name="add-pinned-entries"></a>加入已釘選的項目

若要將特定網頁釘選到搜尋結果的頂端，請將網頁和查詢字詞加入到 [已釘選] 索引標籤。[已釘選] 索引標籤包含一份網頁與查詢字詞配對的清單，指定對於特定查詢要顯示為第一個結果的網頁。 只有當使用者的查詢字串符合釘選的查詢字串 (以釘選的比對狀況為基礎) 時，才會釘選網頁。 [閱讀更多](../define-your-custom-view.md#pin-to-top)。

1. 在 [設定] 頁面上，按一下 [已釘選] 索引標籤，並輸入網頁與應可傳回為第一個結果的查詢字詞。  
  
2. 根據預設，使用者的查詢字串必須完全符合您釘選的查詢字串，Bing 才能傳回網頁做為第一個結果。 若要變更比對條件，請編輯釘選 (按一下鉛筆圖示) 並按一下 [查詢比對條件] 欄中的 [精確]，然後選取適合您應用程式的比對條件。  
  
    ![定義編輯器 [已釘選] 索引標籤的螢幕擷取畫面](../media/pinnedCustomSrch.png)
  
3. 若要確認您的執行個體會將指定的網頁傳回為第一個結果，請在右邊的預覽窗格中輸入您釘選的查詢字詞。

## <a name="configure-hosted-ui"></a>設定託管的 UI

自訂搜尋提供一個託管的 UI 來呈現自訂搜尋執行個體的 JSON 回應。 定義您的 UI 體驗：

1. 按一下 [託管的 UI] 索引標籤。  
  
2. 選取配置。  
  
  ![[託管的 UI] 選取配置步驟的螢幕擷取畫面](./media/custom-search-hosted-ui-select-layout.png)  
  
3. 選取色彩佈景主題。  
  
  ![[託管的 UI] 選取色彩佈景主題的螢幕擷取畫面](./media/custom-search-hosted-ui-select-color-theme.png)  

  若需要微調色彩佈景主題以便與您的 Web 應用程式完美整合，請按一下 [自訂佈景主題]。 並非所有色彩設定都會套用到所有版面配置佈景主題。 若要變更色彩，請在對應的文字方塊中輸入色彩的 RGB HEX 值 (例如 #366eb8)。 或者，按一下色彩按鈕並按一下適合您的陰影。 選取色彩時務必考慮可存取性。
  
  ![[託管的 UI] 自訂色彩佈景主題的螢幕擷取畫面](./media/custom-search-hosted-ui-customize-color-theme.png)  

  
4. 指定其他設定選項。  
  
  ![[託管的 UI] 其他設定步驟的螢幕擷取畫面](./media/custom-search-hosted-ui-additional-configurations.png)  
  
  若要取得進階設定，請按一下 [顯示進階設定]。 此會新增設定，例如新增*連結目標*到 Web 搜尋選項、新增*啟用篩選*到影像與影片選項，並新增 [搜尋方塊文字預留位置] 到 [雜項] 選項。

  ![[託管的 UI] 進階設定步驟的螢幕擷取畫面](./media/custom-search-hosted-ui-advanced-configurations.png)  
  
5. 從下拉式清單選取您的訂用帳戶金鑰。 或者，您可以手動輸入訂用帳戶金鑰。 如需有關取得金鑰的詳細資訊，請參閱[嘗試認知服務](https://azure.microsoft.com/try/cognitive-services/?api=bing-custom-search-api)。  
  
  ![[託管的 UI] 其他設定步驟的螢幕擷取畫面](./media/custom-search-hosted-ui-subscription-key.png)

[!INCLUDE[publish or revert](../includes/publish-revert.md)]

<a name="consuminghostedui"></a>
## <a name="consuming-hosted-ui"></a>使用託管的 UI

有兩種方式可以取用託管的 UI。  

- 選項 1：將提供的 JavaScript 程式碼片段整合到您的應用程式中。
- 選項 2：使用提供的 HTML 端點。

本教學課程的其餘部分說明**選項 1：Javascript 程式碼片段**。  

## <a name="set-up-your-visual-studio-solution"></a>設定您的 Visual Studio 方案

1. 在電腦上開啟 **Visual Studio**。  
  
2. 從 [檔案] 功能表中，選取 [新增]，然後選擇 [專案]。  
  
3. 在 [新增專案] 視窗中，選取 [Visual C# / Web / ASP.NET Core Web 應用程式]，命名您的專案，然後按一下 [確定]。  
  
  ![[新增專案] 視窗的螢幕擷取畫面](./media/custom-search-new-project.png)  
  
4. 在 [新增 ASP.NET Core Web 應用程式] 視窗中，選取 [Web 應用程式]，然後按一下 [確定]。  
  
  ![[新增專案] 視窗的螢幕擷取畫面](./media/custom-search-new-webapp.png)  

## <a name="edit-indexcshtml"></a>編輯 Index.cshtml

1. 在 [方案總管] 中，展開 [頁面]，然後按兩下 [index.cshtml] 以開啟該檔案。  
  
  ![方案總管中展開頁面並選取了 index.cshtml 的螢幕擷取畫面](./media/custom-search-visual-studio-webapp-solution-explorer-index.png)  
  
2. 在 index.cshtml 中，刪除第 7 行以後的所有內容 (含第 7 行)。  
  
  ```razor
  @page
  @model IndexModel
  @{
      ViewData["Title"] = "Home page";
  }    
  ```  
  
3. 加入一個分行符號元素和要做為容器的 div。  
  
  ```html
  @page
  @model IndexModel
  @{
      ViewData["Title"] = "Home page";
  }
  <br />
  <div id="customSearch"></div>
  ```  
  
4. 在 [託管的 UI] 頁面中，向下捲動到名為 [取用 UI] 的區段。 按一下 [端點] 以存取 JavaScript 片段。 您也可以按一下 [生產] 並按一下 [託管的 UI] 索引標籤。
  
  <!-- Get new screenshot after prod gets new bits
  ![Screenshot of the Hosted UI save button](./media/custom-search-hosted-ui-consuming-ui.png)  
  -->
  
5. 將指令碼元素貼到您加入的容器中。  
  
  ``` html
  @page
  @model IndexModel
  @{
      ViewData["Title"] = "Home page";
  }
  <br />
  <div id="customSearch">
      <script type="text/javascript" 
          id="bcs_js_snippet"
          src="https://ui.customsearch.ai /api/ux/rendering-js?customConfig=<YOUR-CUSTOM-CONFIG-ID>&market=en-US&safeSearch=Moderate&version=latest&q=">
      </script>
  </div>
  ```  
  
6. 在 [方案總管] 中，以滑鼠右鍵按一下 [wwwroot]，然後按一下 [在瀏覽器中檢視]。  
  
  ![從方案總管的 [wwwroot] 快顯功能表選取 [在瀏覽器中檢視] 的螢幕擷取畫面](./media/custom-search-webapp-view-in-browser.png)  

新的自訂搜尋網頁看起來應該如下：

![自訂搜尋網頁的螢幕擷取畫面](./media/custom-search-webapp-browse-index.png)

執行搜尋會呈現像下面的結果：

![自訂搜尋結果的螢幕擷取畫面](./media/custom-search-webapp-results.png)

## <a name="next-steps"></a>後續步驟

> [!div class="nextstepaction"]
> [呼叫 Bing 自訂搜尋端點 (C#)](../call-endpoint-csharp.md)