---
title: 使用 Azure Application Insights 診斷效能問題 | Microsoft Docs
description: 使用 Azure Application Insights 來尋找並診斷應用程式效能問題的教學課程。
services: application-insights
keywords: ''
author: mrbullwinkle
ms.author: mbullwin
ms.date: 09/18/2017
ms.service: application-insights
ms.custom: mvc
ms.topic: tutorial
manager: carmonm
ms.openlocfilehash: 4ce4c9e2479c8d570766169ce5094dcc2b4bc511
ms.sourcegitcommit: 58c5cd866ade5aac4354ea1fe8705cee2b50ba9f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/24/2018
ms.locfileid: "42812866"
---
# <a name="find-and-diagnose-performance-issues-with-azure-application-insights"></a>使用 Azure Application Insights 尋找並診斷效能問題

Azure Application Insights 會從您的應用程式收集遙測，以協助分析其作業和效能。  使用此資訊可識別可能發生的問題，或識別對使用者影響最大的應用程式增強功能。  本教學課程會引導您完成以下程序：分析應用程式的伺服器元件效能，以及從用戶端角度分析效能。  您會了解如何：

> [!div class="checklist"]
> * 識別伺服器端作業的效能
> * 分析伺服器作業，以判斷效能緩慢的根本原因
> * 識別最慢的用戶端作業
> * 使用查詢語言分析頁面檢視的詳細資料


## <a name="prerequisites"></a>必要條件

若要完成本教學課程：

- 使用下列工作負載安裝 [Visual Studio 2017](https://www.visualstudio.com/downloads/)：
    - ASP.NET 和 Web 開發
    - Azure 開發
- 將 .NET 應用程式部署至 Azure，並[啟用 Application Insights SDK](app-insights-asp-net.md)。
- [啟用應用程式的 Application Insights Profiler](app-insights-profiler.md#installation)。

## <a name="log-in-to-azure"></a>登入 Azure
在 [https://portal.azure.com](https://portal.azure.com) 上登入 Azure 入口網站。

## <a name="identify-slow-server-operations"></a>識別緩慢的伺服器作業
Application Insights 會收集您應用程式中不同作業的效能詳細資料。 藉由識別持續時間最長的作業，您可以診斷潛在的問題，或為進行中的開發設定最適合的目標，以改善應用程式的整體效能。

1. 選取 [Application Insights]，然後選取您的訂用帳戶。  
1. 若要開啟 [效能] 面板，選取 [調查] 功能表下方的 [效能]，或按一下 [伺服器回應時間] 圖形。

    ![效能](media/app-insights-tutorial-performance/performance.png)

2. [效能] 面板會顯示應用程式每個作業的計數和平均持續時間。  您可以使用這項資訊來識別對使用者影響最大的作業。 在此範例中，**GET Customers/Details** 和 **GET Home/Index** 是可能的調查候選項目，因其持續時間和呼叫數目相對較高。  其他作業的持續時間可能更高，但因呼叫次數不頻繁，改善後的影響不大。  

    ![[效能] 面板](media/app-insights-tutorial-performance/performance-blade.png)

3. 此圖形目前顯示選取的作業在一段時間內的平均持續時間。 您可以切換到第 95 個百分位數，以找出效能問題。 將您感興趣的作業釘選至圖形以新增它們。  這裡顯示有些尖峰值得調查。  減少此圖形的時間範圍，以進一步隔離它。

    ![釘選作業](media/app-insights-tutorial-performance/pin-operations.png)

4.  右側的效能面板會顯示所選作業不同要求的持續時間分佈。  請將時間範圍縮小為從第 95 個百分位數開始。 從「前 3 大相依性」深入解析卡片您可以很快地看出，外部相依性很可能是交易緩慢的主因。  按一下附有範例數目的按鈕，以檢視範例清單。 接著，您可以選取任何範例以檢視交易詳細資料。

    ![持續時間分佈](media/app-insights-tutorial-performance/duration-distribution.png)

5.  您可以一眼看出，交易的總持續時間最主要來自於 Fabrikamaccount Azure 資料表。 您也可以查看導致交易失敗的例外狀況。 您可以按一下清單中的任何項目，以在右側查看其詳細資料。 [深入了解交易診斷體驗](app-insights-transaction-diagnostics.md)

    ![作業詳細資料](media/app-insights-tutorial-performance/operation-details.png)
    

6.  **Profiler** 可顯示為作業執行的實際程式碼和每個步驟所需的時間，以協助您深入進行程式碼層級的診斷。 因為 Profiler 是定期執行，某些作業可能不具有追蹤。  隨著時間累積，將會有更多作業具有追蹤。  若要針對作業啟動 Profiler，請按一下 [Profiler 追蹤]。
5.  追蹤會顯示每個作業的個別事件，讓您可以診斷整體作業持續時間的根本原因。  按一下持續時間最長之前幾個範例的其中一個。
6.  按一下 [顯示最忙碌路徑] 來反白顯示對於作業的總持續時間影響最深之事件的特定路徑。  在此範例中，您可以看到最慢的呼叫是來自 FabrikamFiberAzureStorage.GetStorageTableData 方法。 花費最多時間的部分是 CloudTable.CreateIfNotExist 方法。 如果每次呼叫函式時都會執行此程式碼，就會使用不必要的網路呼叫和 CPU 資源。 修正程式碼的最佳方式，是將這一行放在只執行一次的某些啟動方法中。 

    ![Profiler 詳細資料](media/app-insights-tutorial-performance/profiler-details.png)

7.  畫面頂端的 [效能提示] 證實了過長的持續時間是由於等待。  按一下 [等待] 連結以存取關於解譯不同事件類型的文件。

    ![效能提示](media/app-insights-tutorial-performance/performance-tip.png)

8.  如需進一步的分析，您可以按一下 [下載 .etl 追蹤]，將追蹤下載至 Visual Studio。

## <a name="use-analytics-data-for-server"></a>使用伺服器的分析資料
Application Insights Analytics 提供豐富的查詢語言，可讓您分析 Application Insights 收集的所有資料。  您可以使用這項功能來深入分析要求和效能資料。

1. 返回 [作業詳細資料] 面板，然後按一下 [Analytics] 按鈕。

    ![[Analytics] 按鈕](media/app-insights-tutorial-performance/server-analytics-button.png)

2. Application Insights Analytics 隨即開啟，面板中每個檢視各有一個查詢。  您可以依原狀執行這些查詢，或根據您的需求作修改。  第一個查詢顯示這項作業過去的持續時間。

    ![分析](media/app-insights-tutorial-performance/server-analytics.png)


## <a name="identify-slow-client-operations"></a>識別緩慢的用戶端作業
除了識別要最佳化的伺服器處理序，Application Insights 還可以從用戶端瀏覽器的角度來進行分析。  這可協助您識別用戶端元件是否有改善空間，甚至識別不同瀏覽器或不同位置的問題。

1. 選取 [調查] 下方的 [瀏覽器] 以開啟瀏覽器摘要。  這樣會從瀏覽器的角度提供各種應用程式遙測的視覺效果摘要。

    ![瀏覽器摘要](media/app-insights-tutorial-performance/browser-summary.png)

2.  向下捲動至 [哪些是我最慢的頁面?]。  這樣會顯示在您的應用程式中用戶端載入耗時最久的頁面清單。  您可以使用這項資訊，將對使用者影響最大的頁面依照優先順序排列。
3.  按一下其中一個以開啟 [頁面檢視] 面板。  在此範例中，**/FabrikamProd** 頁面顯示過長的平均持續時間。  [頁面檢視] 面板提供有關此頁面的詳細資料，包括不同持續時間範圍的明細。

    ![頁面檢視](media/app-insights-tutorial-performance/page-view.png)

4.  按一下最高的持續時間，以查看這些要求的詳細資料。  然後按一下個別的要求以檢視要求頁面之用戶端的詳細資料，包括瀏覽器的類型和其位置。  這項資訊有助於判斷是否有和特定類型的用戶端相關的效能問題。

    ![要求詳細資料](media/app-insights-tutorial-performance/request-details.png)

## <a name="use-analytics-data-for-client"></a>將分析資料用於用戶端
就像針對伺服器效能所收集的資料一樣，Application Insights 可透過 Analytics 來深入分析所有用戶端資料。

1. 返回瀏覽器摘要，然後按一下 [Analytics] 圖示。

    ![[Analytics] 圖示](media/app-insights-tutorial-performance/client-analytics-icon.png)

2. Application Insights Analytics 隨即開啟，面板中每個檢視各有一個查詢。 第一個查詢顯示不同的頁面檢視過去的持續時間。

    ![分析](media/app-insights-tutorial-performance/client-analytics.png)

3.  「智慧型診斷」是 Application Insights Analytics 的一項功能，可識別資料中的唯一模式。  當您按一下折線圖中「智慧型診斷」點時，將會執行相同的查詢，但不涵蓋造成異常的記錄。  這些記錄的詳細資料顯示在查詢的註解區段，讓您可以針對導致過長持續時間的頁面檢視，識別這些頁面檢視的屬性。

    ![智慧型診斷](media/app-insights-tutorial-performance/client-smart-diagnostics.png)


## <a name="next-steps"></a>後續步驟
由於您已學會如何識別執行階段例外狀況，請前進到下一個教學課程，了解如何建立警示以回應失敗。

> [!div class="nextstepaction"]
> [針對應用程式健康情況發出警示](app-insights-tutorial-alert.md)
