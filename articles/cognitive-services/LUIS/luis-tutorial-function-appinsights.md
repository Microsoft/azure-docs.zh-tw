---
title: 使用 C# 取自 LUIS 的 Application Insights 資料
titleSuffix: Azure Cognitive Services
description: 使用 Node.js 建置與 LUIS 應用程式和 Application Insights 整合的 Bot。
services: cognitive-services
author: diberry
manager: cgronlun
ms.service: cognitive-services
ms.component: language-understanding
ms.topic: article
ms.date: 09/24/2018
ms.author: diberry
ms.openlocfilehash: 342c4f98d854d94426c3e4a1eb79bb2a6adffaad
ms.sourcegitcommit: 4ecc62198f299fc215c49e38bca81f7eb62cdef3
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "47037995"
---
# <a name="add-luis-results-to-application-insights"></a>將 LUIS 結果新增至 Application Insights
本教學課程將 LUIS 要求和回應資訊新增至 [Application Insights](https://azure.microsoft.com/services/application-insights/) 遙測資料存放區。 在您擁有該資料之後，即可使用 Kusto 語言或 PowerBi 查詢它，以即時分析、彙總以及報告意圖和語句實體。 此分析可協助您判斷應該新增還是編輯 LUIS 應用程式的意圖和實體。

此 Bot 是使用 Bot Framework 3.x 和 Azure Web 應用程式 Bot 所建置的。

在本教學課程中，您了解如何：

> [!div class="checklist"]
* 將 Application Insights 程式庫新增至 Web 應用程式 Bot
* 將 LUIS 查詢結果擷取並傳送至 Application Insights
* 查詢 Application Insights 中的最高意圖、分數和語句

## <a name="prerequisites"></a>必要條件

* **[上一個教學課程](luis-nodejs-tutorial-build-bot-framework-sample.md)** 中開啟 Application Insights 的 LUIS Web 應用程式 Bot。 

> [!Tip]
> 若您還沒有訂用帳戶，則可以註冊[免費帳戶](https://azure.microsoft.com/free/)。

本教學課程中的所有程式碼都是位於 [LUIS-Samples github 存放庫](https://github.com/Microsoft/LUIS-Samples/tree/master/documentation-samples/tutorial-web-app-bot-application-insights/nodejs)，而且與本教學課程建立關聯的每一行都會加上 `//APPINSIGHT:` 註解。 

## <a name="web-app-bot-with-luis"></a>含 LUIS 的 Web 應用程式 Bot
本教學課程假設您的程式碼與下面類似，或您已完成[其他教學課程](luis-nodejs-tutorial-build-bot-framework-sample.md)： 

   [!code-javascript[Web app bot with LUIS](~/samples-luis/documentation-samples/tutorial-web-app-bot/nodejs/app.js "Web app bot with LUIS")]

## <a name="add-application-insights-library-to-web-app-bot"></a>將 Application Insights 程式庫新增至 Web 應用程式 Bot
目前，此 Web 應用程式 Bot 中所使用的 Application Insights 服務會收集 Bot 的一般狀態遙測。 它不會收集您需要檢查與修正意圖和實體的 LUIS 要求和回應資訊。 

若要擷取 LUIS 要求和回應，Web 應用程式 Bot 需要在 **app.js** 檔案中安裝和設定 **[Application Insights](https://www.npmjs.com/package/applicationinsights)** NPM 套件。 接著意圖對話處理常式需要將 LUIS 要求和回應資訊傳送給 Application Insights。 

1. 在 Azure 入口網站中，於 Web 應用程式 Bot 服務中，選取 [Bot Management] \(Bot 管理\) 區段下方的 [建置]。 

    ![搜尋應用程式見解](./media/luis-tutorial-appinsights/build.png)

2. 新的瀏覽器索引標籤與 App Service 編輯器隨即開啟。 選取頂端列中的應用程式名稱，然後選取 [Open Kudu Console] \(開啟 Kudu 主控台\)。 

    ![搜尋應用程式見解](./media/luis-tutorial-appinsights/kudu-console.png)

3. 在主控台中，輸入下列命令以安裝 Application Insights 和 Underscore 套件：

    ```
    cd site\wwwroot && npm install applicationinsights && npm install underscore
    ```

    ![搜尋應用程式見解](./media/luis-tutorial-appinsights/npm-install.png)

    等候安裝套件：

    ```
    luisbot@1.0.0 D:\home\site\wwwroot
    `-- applicationinsights@1.0.1 
      +-- diagnostic-channel@0.2.0 
      +-- diagnostic-channel-publishers@0.2.1 
      `-- zone.js@0.7.6 
    
    npm WARN luisbot@1.0.0 No repository field.
    luisbot@1.0.0 D:\home\site\wwwroot
    +-- botbuilder-azure@3.0.4
    | `-- azure-storage@1.4.0
    |   `-- underscore@1.4.4 
    `-- underscore@1.8.3 
    ```

    您已完成 kudu 主控台瀏覽器索引標籤。

## <a name="capture-and-send-luis-query-results-to-application-insights"></a>將 LUIS 查詢結果擷取並傳送至 Application Insights
1. 在 App Service 編輯器瀏覽器索引標籤中，開啟 **app.js** 檔案。

2. 在現有 `requires` 行下方，新增下列 NPM 程式庫：

   [!code-javascript[Add NPM packages to app.js](~/samples-luis/documentation-samples/tutorial-web-app-bot-application-insights/nodejs/app.js?range=12-16 "Add NPM packages to app.js")]

3. 建立 Application Insights 物件，並使用 Web 應用程式 Bot 應用程式設定 **BotDevInsightsKey**： 

   [!code-javascript[Create the Application Insights object](~/samples-luis/documentation-samples/tutorial-web-app-bot-application-insights/nodejs/app.js?range=68-80 "Create the Application Insights object")]

4. 新增 **appInsightsLog** 函式：

   [!code-javascript[Add the appInsightsLog function](~/samples-luis/documentation-samples/tutorial-web-app-bot-application-insights/nodejs/app.js?range=82-109 "Add the appInsightsLog function")]

    此函式的最後一行就是將資料新增至 Application Insights 的位置。 事件的名稱是 **LUIS-results**，這是此 Web 應用程式 Bot 所收集之任何其他遙測資料以外的唯一名稱。 

5. 使用 **appInsightsLog** 函式。 您可以將它新增至每個意圖對話方塊：

   [!code-javascript[Use the appInsightsLog function](~/samples-luis/documentation-samples/tutorial-web-app-bot-application-insights/nodejs/app.js?range=117-118 "Use the appInsightsLog function")]

6. 若要測試 Web 應用程式 Bot，請使用 [Test in Web Chat] \(在網路聊天中測試\) 功能。 因為所有工作都是在 Application Insights 中進行，而不是在 Bot 回應中，所以您應該看不出有任何差異。

## <a name="view-luis-entries-in-application-insights"></a>在 Application Insights 中檢視 LUIS 項目
開啟 Application Insights 以查看 LUIS 項目。 

1. 在入口網站中，選取 [所有資源]，然後依 Web 應用程式 Bot 名稱篩選。 按一下類型為 **Application Insights** 的資源。 Application Insights 的圖示為燈泡。 

    ![搜尋應用程式見解](./media/luis-tutorial-appinsights/search-for-app-insights.png)



2. 當資源開啟時，按一下最右側面板中放大鏡的**搜尋**圖示。 即會顯示右側新面板。 根據找到的遙測資料量，此面板可能需要一秒才會顯示。 搜尋 `LUIS-results`，然後按鍵盤上的 Enter 鍵。 這份清單的範圍會縮小為只有使用本教學課程所新增的 LUIS 查詢結果。

    ![相依性篩選](./media/luis-tutorial-appinsights/app-insights-filter.png)

3. 選取頂端項目。 新的視窗會顯示更多詳細資料 (包括最右側 LUIS 查詢的自訂資料)。 資料包括最上層意圖和其分數。

    ![相依性詳細資料](./media/luis-tutorial-appinsights/app-insights-detail.png)

    當您完成時，請選取最右上方的 **X** 回到相依性項目清單。 


> [!Tip]
> 若您想要儲存相依性清單，並於稍後返回，則請按一下 [...More] \(...其他\)，然後按一下 [儲存我的最愛]。

## <a name="query-application-insights-for-intent-score-and-utterance"></a>查詢 Application Insights 中的意圖、分數和語句
Application Insights 可讓您使用 [Kusto](https://docs.microsoft.com/azure/application-insights/app-insights-analytics#query-data-in-analytics) 語言查詢資料，以及將它匯出至 [PowerBI](https://powerbi.microsoft.com)。 

1. 按一下相依性清單頂端且於篩選方塊上方的 [Analytics]。 

    ![[Analytics] 按鈕](./media/luis-tutorial-appinsights/analytics-button.png)

2. 即會開啟新的視窗，而其頂端有查詢視窗，其下則有資料表視窗。 若您先前使用過資料庫，則這種排列方式十分熟悉。 查詢包括過去 24 小時以 `LUIS-results` 名稱開始的所有項目。 **CustomDimensions** 資料行具有 LUIS 查詢結果作為名稱/值組。

    ![Analytics 查詢視窗](./media/luis-tutorial-appinsights/analytics-query-window.png)

3. 若要拉出最上層意圖、分數和語句，請在查詢視窗中於最後一行正上方新增下列項目：

    ```SQL
    | extend topIntent = tostring(customDimensions.LUIS_intent_intent)
    | extend score = todouble(customDimensions.LUIS_intent_score)
    | extend utterance = tostring(customDimensions.LUIS_text)
    ```

4. 執行查詢。 捲動至資料表中的最右側。 有新的最上層意圖、分數和語句資料行可用。 按一下最上層意圖資料行排序。

    ![分析最上層意圖](./media/luis-tutorial-appinsights/app-insights-top-intent.png)


深入了解 [Kusto 查詢語言](https://docs.microsoft.com/azure/log-analytics/query-language/get-started-queries)或[將資料匯出至 PowerBi](https://docs.microsoft.com/azure/application-insights/app-insights-export-power-bi)。 

## <a name="next-steps"></a>後續步驟

您可能想要新增至 Application Insights 資料的其他資訊包括應用程式識別碼、版本識別碼、上次模型變更日期、最後訓練日期、最後發行日期。 這些值可以擷取自端點 URL (應用程式識別碼和版本識別碼) 或[撰寫 API](https://westus.dev.cognitive.microsoft.com/docs/services/5890b47c39e2bb17b84a55ff/operations/5890b47c39e2bb052c5b9c3d) 呼叫，然後設定於 Web 應用程式 Bot 設定並從該處提取。  

若您將相同端點訂用帳戶用於多個 LUIS 應用程式，則也應該包括訂用帳戶識別碼以及指出它為共用金鑰的屬性。 

> [!div class="nextstepaction"]
> [深入了解範例語句](luis-how-to-add-example-utterances.md)
