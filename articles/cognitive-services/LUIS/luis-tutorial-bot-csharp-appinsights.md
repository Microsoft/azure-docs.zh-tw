---
title: 使用 C# 取自 LUIS 的 Application Insights 資料
titleSuffix: Azure Cognitive Services
description: 使用 C# 建置與 LUIS 應用程式和 Application Insights 整合的 Bot。
services: cognitive-services
author: diberry
manager: cgronlun
ms.service: cognitive-services
ms.component: language-understanding
ms.topic: article
ms.date: 09/24/2018
ms.author: diberry
ms.openlocfilehash: 83ad70e1242af1e01af06206a3a141f455072a44
ms.sourcegitcommit: 4ecc62198f299fc215c49e38bca81f7eb62cdef3
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "47038947"
---
# <a name="add-luis-results-to-application-insights"></a>將 LUIS 結果新增至 Application Insights
本教學課程將 LUIS 回應資訊新增至 [Application Insights](https://azure.microsoft.com/services/application-insights/) 遙測資料存放區。 在您擁有該資料之後，即可使用 Kusto 語言或 PowerBi 查詢它，以即時分析、彙總以及報告意圖和語調實體。 此分析可協助您判斷應該新增還是編輯 LUIS 應用程式的意圖和實體。 

此 Bot 是使用 Bot Framework 3.x 和 Azure Web 應用程式 Bot 所建置的。

在本教學課程中，您了解如何：

> [!div class="checklist"]
* 將 Application Insights 新增至 Web 應用程式 Bot
* 將 LUIS 查詢結果擷取並傳送至 Application Insights
* 查詢 Application Insights 中的最高意圖、分數和語句

## <a name="prerequisites"></a>必要條件

* **[上一個教學課程](luis-csharp-tutorial-build-bot-framework-sample.md)** 中開啟 Application Insights 的 LUIS Web 應用程式 Bot。 
* 您的電腦本機上已安裝 [Visual Studio 2017](https://www.visualstudio.com/downloads/)。

> [!Tip]
> 如果您還沒有訂用帳戶，則可以註冊[免費帳戶](https://azure.microsoft.com/free/)。

本教學課程中的所有程式碼都是位於 [LUIS-Samples github 存放庫](https://github.com/Microsoft/LUIS-Samples/tree/master/documentation-samples/tutorial-web-app-bot-application-insights/csharp)，而且與本教學課程建立關聯的每一行都會加上 `//LUIS Tutorial:` 註解。 

## <a name="review-luis-web-app-bot"></a>檢閱 LUIS Web 應用程式 Bot
本教學課程假設您的程式碼與下面類似，或您已完成[其他教學課程](luis-csharp-tutorial-build-bot-framework-sample.md)： 

   [!code-csharp[Web app bot with LUIS](~/samples-luis/documentation-samples/tutorial-web-app-bot/csharp/BasicLuisDialog.cs "Web app bot with LUIS")]

## <a name="application-insights-in-web-app-bot"></a>在 Web 應用程式 Bot 中新增 Application Insights
目前，在 Web 應用程式 Bot 服務建立過程中新增的 Application Insights 服務會收集 Bot 的一般狀態遙測。 它不會收集 LUIS 回應資訊。 若要分析及改善 LUIS，您需要 LUIS 回應資訊。  

若要擷取 LUIS 回應，Web 應用程式 Bot 需要針對專案安裝及設定 **[Application Insights](https://www.nuget.org/packages/Microsoft.ApplicationInsights/)**。 

## <a name="download-web-app-bot"></a>下載 Web 應用程式 Bot
使用 [Visual Studio 2017](https://www.visualstudio.com/downloads/)為 Web 應用程式 Bot 來新增及設定 Application Insights。 若要在 Visual Studio 中使用 Web 應用程式 Bot，請下載 Web 應用程式 Bot 程式碼。

1. 在 Azure 入口網站中，針對 Web 應用程式 Bot選取 [建置]。

    ![在入口網站中選取建置](./media/luis-tutorial-bot-csharp-appinsights/download-build-menu.png)

2. 選取 [下載 zip 檔案] 並等等檔案備妥為止。

    ![下載 zip 檔](./media/luis-tutorial-bot-csharp-appinsights/download-link.png)

3. 在快顯視窗中選取 [下載 zip 檔案]。 請記住在您電腦上的位置，您在下一節中需要用到它。

    ![下載 zip 檔案快顯視窗](./media/luis-tutorial-bot-csharp-appinsights/download-popup.png)

## <a name="open-solution-in-visual-studio-2017"></a>在 Visual Studio 2017 中開啟解決方案

1. 將檔案解壓縮到資料夾中。 

2. 開啟 Visual Studio 2017 並開啟解決方案檔案 `Microsoft.Bot.Sample.LuisBot.sln`。 如果出現安全性警告，請選取 [確定]。

    ![在 Visual Studio 2017 中開啟解決方案](./media/luis-tutorial-bot-csharp-appinsights/vs-2017-security-warning.png)

3. Visual Studio 必須將相依性新增至解決方案。 在 [解決方案總管] 中，以滑鼠右鍵按一下 [參考]，然後選取 [管理 NuGet 套件...]。 

    ![Manage NuGet packages](./media/luis-tutorial-bot-csharp-appinsights/vs-2017-manage-nuget-packages.png)

4. NuGet 套件管理員會顯示已安裝的套件清單。 選取黃色列中的 [還原]。 等候還原程序完成。

    ![還原 NuGet 套件](./media/luis-tutorial-bot-csharp-appinsights/vs-2017-restore-packages.png)

## <a name="add-application-insights-to-the-project"></a>將 Application Insights 新增至專案
在 Visual Studio 中安裝及設定 Application Insights。 

1. 在 Visual Studio 2017 中，選取頂端功能表上的 [專案]，然後選取 [新增 Application Insights 遙測...]。

2. 在 [Application Insights 設定] 視窗中，選取 [開始免費使用]

    ![開始設定 Application Insights](./media/luis-tutorial-bot-csharp-appinsights/vs-2017-configure-app-insights.png)

3. 向 Application Insights 註冊您的應用程式。 您必須輸入 Azure 入口網站認證。 

4. Visual Studio 會將 Application Insights 新增至專案，並顯示其當下的狀態。 

    ![Application Insights 狀態](./media/luis-tutorial-bot-csharp-appinsights/vs-2017-adding-application-insights-to-project.png)

    處理完成時，[Application Insights 設定] 會顯示進度狀態。 

    ![Application Insights 進度狀態](./media/luis-tutorial-bot-csharp-appinsights/vs-2017-configured-application-insights-to-project.png)

    [啟用追蹤收集] 為紅色，表示並未啟用。 本教學課程不會使用此功能。 

## <a name="build-and-resolve-errors"></a>建置及解決錯誤

1. 選取 [建置] 功能表，然後選取 [重建解決方案]，以建置解決方案。 等待建置完成。 

2. 如果建置因為 `CS0104` 錯誤而失敗，您必須加以修正。 在 `Controllers` 資料夾的 `MessagesController.cs file` 中，在活動類型前面加上連接器類型，以修正模稜兩可的 `Activity` 類型使用方式。 若要這樣做，請將行 22 和 36 上的名稱 `Activity` 從 `Activity` 變更為 `Connector.Activity`。 再次建置解決方案。 應該不再有建置錯誤。

    該檔案的完整來源是：

    [!code-csharp[MessagesController.cs file](~/samples-luis/documentation-samples/tutorial-web-app-bot-application-insights/csharp/MessagesController.cs "MessagesController.cs file")]

## <a name="publish-project-to-azure"></a>將專案發佈到 Azure
**Application Insights** 套件現在位於專案中，並已在 Azure 入口網站中針對您的認證正確設定。 專案變更必須發佈回到 Azure。

1. 在 [方案總管] 中，以滑鼠右鍵按一下專案名稱，然後選取 [發佈]。

    ![將專案發佈至入口網站](./media/luis-tutorial-bot-csharp-appinsights/vs-2017-publish.png)

2. 在 [發佈] 視窗中，選取 [建立新設定檔]。

    ![將專案發佈至入口網站](./media/luis-tutorial-bot-csharp-appinsights/vs-2017-publish-1.png)

3. 選取 [匯入設定檔]，然後選取 [確定]。

    ![將專案發佈至入口網站](./media/luis-tutorial-bot-csharp-appinsights/vs-2017-publish-2.png)

4. 在 [匯入發佈設定檔] 視窗中，瀏覽至您的專案資料夾，瀏覽至 `PostDeployScripts` 資料夾，選取以 `.PublishSettings` 結尾的檔案，然後選取 `Open`。 您現在已針對這個專案設定發佈。 

5. 選取 [發佈] 按鈕，將您的本機原始程式碼發佈到 Bot 服務。 [輸出]  視窗會顯示狀態。 已在 Azure 入口網站中完成本教學課程的其餘部分。 關閉 Visual Studio 2017。 

## <a name="open-three-browser-tabs"></a>開啟三個瀏覽器索引標籤
在 Azure 入口網站中，尋找 Web 應用程式 Bot 並加以開啟。 下列步驟使用 Web 應用程式 Bot 的三個不同檢視。 在瀏覽器中開啟三個個別的索引標籤可能比較容易： 
  
>  * 在網路聊天中測試
>  * 建置/開啟線上程式碼編輯器 -> App Service 編輯器
>  * App Service 編輯器/開啟 Kudu 主控台 -> 診斷主控台

## <a name="modify-basicluisdialogcs-code"></a>修改 BasicLuisDialog.cs 程式碼

1. 在 [App Service 編輯器] 瀏覽器索引標籤中，開啟 `BasicLuisDialog.cs` 檔案。

2. 在現有 `using` 行下方，新增下列 NuGet 相依性：

   [!code-csharp[Add using statement](~/samples-luis/documentation-samples/tutorial-web-app-bot-application-insights/csharp/BasicLuisDialog.cs?range=11-12 "Add using statement")]

3. 新增 `LogToApplicationInsights` 函式：

   [!code-csharp[Add the LogToApplicationInsights function](~/samples-luis/documentation-samples/tutorial-web-app-bot-application-insights/csharp/BasicLuisDialog.cs?range=61-92 "Add the LogToApplicationInsights function")]

    Application Insights 檢測金鑰已在 Web 應用程式名為 `BotDevInsightsKey` 的 Bot 應用程式設定中。 

    此函式的最後一行會將資料新增至 Application Insights。 追蹤的名稱是 `LUIS`，這是此 Web 應用程式 Bot 所收集的任何其他遙測資料以外的唯一名稱。 所有屬性的前面也會加上 `LUIS_`，因此您可以查看本教學課程新增哪些資料 (相較於 Web 應用程式 Bot 所提供的資訊)。

4. 在 `ShowLuisResult` 函式頂端呼叫 `LogToApplicationInsights` 函式：

   [!code-csharp[Use the LogToApplicationInsights function](~/samples-luis/documentation-samples/tutorial-web-app-bot-application-insights/csharp/BasicLuisDialog.cs?range=114-115 "Use the LogToApplicationInsights function")]

## <a name="build-web-app-bot"></a>建置 Web 應用程式 Bot
1. 以兩種方式之一建置 Web 應用程式 Bot。 第一個方法是以滑鼠右鍵按一下 [App Service 編輯器] 中的 `build.cmd`，然後選取 [從主控台執行]。 主控台的輸出會顯示 `Finished successfully.` 並完成。

2. 如果未順利完成，您需要開啟主控台，瀏覽到指令碼，並使用下列步驟加以執行。 在 [App Service 編輯器] 的頂端藍色列上，選取您的 Bot 名稱，然後選取下拉式清單中的 [開啟 Kudu 主控台]。

    ![開啟 Kudu 主控台](./media/luis-tutorial-bot-csharp-appinsights/open-kudu-console.png)

3. 在主控台視窗中，輸入下列命令：

    ```
    cd site\wwwroot && build.cmd
    ```

    等待建置完成並顯示 `Finished successfully.`

## <a name="test-the-web-app-bot"></a>測試 Web 應用程式 Bot

1. 若要測試 Web 應用程式 Bot，請使用入口網站中的 [在網路聊天中測試] 功能。 

2. 輸入片語 `Coffee bar on please`。  

    ![在聊天中測試 Web 應用程式 Bot](./media/luis-tutorial-bot-csharp-appinsights/test-in-web-chat.png)

3. 您應會看到聊天機器人回應並無差異。 變更會將資料傳送至 Application Insights，而不在 Bot 回應中。 輸入多一些語句，Application Insights 中就會有多一些資料：

```
Please deliver a pizza
Turn off all the lights
Turn on the hall light
```

## <a name="view-luis-entries-in-application-insights"></a>在 Application Insights 中檢視 LUIS 項目
開啟 Application Insights 以查看 LUIS 項目。 

1. 在入口網站中，選取 [所有資源]，然後依 Web 應用程式 Bot 名稱篩選。 按一下類型為 **Application Insights** 的資源。 Application Insights 的圖示為燈泡。 

    ![搜尋應用程式見解](./media/luis-tutorial-bot-csharp-appinsights/portal-service-list-app-insights.png)

2. 當資源開啟時，按一下最右側面板中放大鏡的**搜尋**圖示。 即會顯示右側新面板。 根據找到的遙測資料量，此面板可能需要一秒才會顯示。 搜尋 `LUIS`。 這份清單的範圍會縮小為只有使用本教學課程所新增的 LUIS 查詢結果。

    ![搜尋追蹤](./media/luis-tutorial-bot-csharp-appinsights/portal-service-list-app-insights-search-luis-trace.png)

3. 選取頂端項目。 新的視窗會顯示更多詳細資料 (包括最右側 LUIS 查詢的自訂資料)。 資料包括最上層意圖和其分數。

    ![檢閱追蹤項目](./media/luis-tutorial-bot-csharp-appinsights/portal-service-list-app-insights-search-luis-trace-item.png)

    當您完成時，請選取最右上方的 **X** 回到相依性項目清單。 


> [!Tip]
> 若您想要儲存相依性清單，並於稍後返回，則請按一下 [...More] \(...其他\)，然後按一下 [儲存我的最愛]。

## <a name="query-application-insights-for-intent-score-and-utterance"></a>查詢 Application Insights 中的意圖、分數和語句
Application Insights 可讓您使用 [Kusto](https://docs.microsoft.com/azure/application-insights/app-insights-analytics#query-data-in-analytics) 語言查詢資料，以及將它匯出至 [PowerBI](https://powerbi.microsoft.com)。 

1. 按一下相依性清單頂端且於篩選方塊上方的 [Analytics]。 

    ![[Analytics] 按鈕](./media/luis-tutorial-bot-csharp-appinsights/portal-service-list-app-insights-search-luis-analytics-button.png)

2. 即會開啟新的視窗，而其頂端有查詢視窗，其下則有資料表視窗。 若您先前使用過資料庫，則這種排列方式十分熟悉。 查詢包括過去 24 小時以 `LUIS` 名稱開始的所有項目。 **CustomDimensions** 資料行具有 LUIS 查詢結果作為名稱/值組。

    ![預設分析報告](./media/luis-tutorial-bot-csharp-appinsights/analytics-query-1.png)

3. 若要拉出最上層意圖、分數和語句，請在查詢視窗中於最後一行正上方新增下列項目：

    ```SQL
    | extend topIntent = tostring(customDimensions.LUIS_topScoringIntent)
    | extend score = todouble(customDimensions.LUIS_topScoringIntentScore)
    | extend utterance = tostring(customDimensions.LUIS_query)
    ```

4. 執行查詢。 捲動至資料表中的最右側。 有新的最上層意圖、分數和語句資料行可用。 按一下最上層意圖資料行排序。

    ![自訂分析報告](./media/luis-tutorial-bot-csharp-appinsights/analytics-query-2.png)


深入了解 [Kusto 查詢語言](https://docs.microsoft.com/azure/log-analytics/query-language/get-started-queries)或[將資料匯出至 PowerBi](https://docs.microsoft.com/azure/application-insights/app-insights-export-power-bi)。 


## <a name="learn-more-about-bot-framework"></a>深入了解 Bot Framework
深入了解 [Bot Framework](https://dev.botframework.com/)。

## <a name="next-steps"></a>後續步驟

您可能想要新增至 Application Insights 資料的其他資訊包括應用程式識別碼、版本識別碼、上次模型變更日期、最後訓練日期、最後發行日期。 這些值可以擷取自端點 URL (應用程式識別碼和版本識別碼) 或[撰寫 API](https://westus.dev.cognitive.microsoft.com/docs/services/5890b47c39e2bb17b84a55ff/operations/5890b47c39e2bb052c5b9c3d) 呼叫，然後設定於 Web 應用程式 Bot 設定並從該處提取。  

若您將相同端點訂用帳戶用於多個 LUIS 應用程式，則也應該包括訂用帳戶識別碼以及指出它為共用金鑰的屬性。 

> [!div class="nextstepaction"]
> [深入了解範例語句](luis-how-to-add-example-utterances.md)
