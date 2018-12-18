---
title: 在 Azure Application Insights 中建立自訂儀表板 | Microsoft Docs
description: 使用 Azure Application Insights 建立自訂 KPI 儀表板的教學課程。
keywords: ''
services: application-insights
author: mrbullwinkle
ms.author: mbullwin
ms.date: 09/20/2017
ms.service: application-insights
ms.custom: mvc
ms.topic: tutorial
manager: carmonm
ms.openlocfilehash: 50d2545d5145f1d93a1ea9fed3e4f98b474d41b2
ms.sourcegitcommit: b32d6948033e7f85e3362e13347a664c0aaa04c1
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 02/13/2018
ms.locfileid: "29178187"
---
# <a name="create-custom-kpi-dashboards-using-azure-application-insights"></a>使用 Azure Application Insights 建立自訂 KPI 儀表板

您可以在 Azure 入口網站中建立多個儀表板，其中每個儀表板會針對來自不同資源群組和訂用帳戶的多重 Azure 資源，包含磚視覺化資料。  從 Azure Application Insights 釘選不同的圖表和檢視來建立自訂的儀表板，就能提供應用程式健康情況和效能的完整概況。  本教學課程會引導您建立自訂的儀表板，其內容包含來自 Azure Application Insights 的多種類型資料和視覺效果。  您會了解如何：

> [!div class="checklist"]
> * 在 Azure 中建立自訂的儀表板
> * 從磚庫新增磚
> * 將 Application Insights 中的標準計量加入儀表板 
> * 將自訂計量圖表 Application Insights 加入儀表板
> * 將 Analytics 查詢的結果加入儀表板 



## <a name="prerequisites"></a>先決條件

若要完成本教學課程：

- 將 .NET 應用程式部署至 Azure，並[啟用 Application Insights SDK](app-insights-asp-net.md)。 

## <a name="log-in-to-azure"></a>登入 Azure
登入 Azure 入口網站，網址為 [https://portal.azure.com](https://portal.azure.com)。

## <a name="create-a-new-dashboard"></a>建立新的儀表板
單一儀表板可包含來自多個應用程式、資源群組和訂用帳戶的資源。  請從為您的應用程式建立新的儀表板，以開始本教學課程。  

2.  在入口網站的主畫面上，選取 [新增儀表板]。

    ![新增儀表板](media/app-insights-tutorial-dashboards/new-dashboard.png)

3. 輸入儀表板的名稱。
4. 從 [磚庫] 內各式各樣的磚當中挑選要新增至您儀表板的磚。  除了從磚庫新增磚以外，您還可直接從 Application Insights 將圖表和其他檢視釘選至儀表板。
5. 找到 [Markdown] 磚，然後拖曳至您的儀表板。  這個磚可讓您加入 Markdown 格式的文字，適合用來將描述性文字加入儀表板。
6. 將文字加入磚的屬性，然後調整其在儀表板畫布上的大小。
    
    ![編輯 Markdown 磚](media/app-insights-tutorial-dashboards/edit-markdown.png)

6. 按一下畫面頂端的 [完成自訂] 以結束磚自訂模式，然後按一下 [發行變更] 以儲存變更。

    ![含有 Markdown 磚的儀表板](media/app-insights-tutorial-dashboards/dashboard-01.png)


## <a name="add-health-overview"></a>新增健康情況概觀
只有靜態文字的儀表板沒那麼有趣，因此，現在要從 Application Insights 新增磚來顯示關於您應用程式的資訊。  您可以從 [磚庫] 新增 Application Insights 磚，或是直接從 Application Insights 畫面釘選它們。  這可讓您先設定您所熟悉的圖表和檢視，再將它們釘選至儀表板。  請先從新增應用程式的標準健康情況概觀開始。  這不需要設定，而且在儀表板中可以自訂的程度最少。


1. 在 Azure 功能表中選取 [Application Insights] ，然後選取您的應用程式。
2. 在 [概觀時間軸] 中，選取操作功能表，然後按一下 [釘選到儀表板]。  這樣會將磚新增到您所檢視的最後一個儀表板。  

    ![釘選 [概觀時間軸]](media/app-insights-tutorial-dashboards/pin-overview-timeline.png)
 
3. 在畫面頂端，按一下 [檢視儀表板] 以返回儀表板。
4. [概觀時間軸] 現在已新增至儀表板。  按一下並將它拖曳到適當的位置，然後按一下 [完成自訂] 和 [發行變更]。  您的儀表板現在已經有一個包含有用資訊的磚。

    ![含有 [概觀時間軸] 的儀表板](media/app-insights-tutorial-dashboards/dashboard-02.png)



## <a name="add-custom-metric-chart"></a>新增自訂的計量圖表
[計量] 面板可讓您將 Application Insights 長期以來所收集的計量圖表化，並提供選擇性的篩選和群組。  和 Application Insights 中的其他元素一樣，此圖表也可新增到儀表板。  要這樣做，您必須先進行一些自訂。

1. 在 Azure 功能表中選取 [Application Insights] ，然後選取您的應用程式。
1. 選取 [計量]。  
2. 已經有一個建立好的空白圖表，系統將提示您新增計量。  將計量新增至圖表，然後選擇性地加入篩選和群組。  下列範例顯示依成功狀態分組的伺服器要求數目。  這樣會提供一個含有成功和失敗要求的執行中檢視。

    ![新增計量](media/app-insights-tutorial-dashboards/metrics-chart.png)

4. 選取圖表的操作功能表，然後選取 [釘選到儀表板]。  這樣會將檢視新增到您所檢視的最後一個儀表板。

    ![釘選計量圖表](media/app-insights-tutorial-dashboards/pin-metrics-chart.png)

3. 在畫面頂端，按一下 [檢視儀表板] 以返回儀表板。

4. 時間軸計量圖表現在已新增至儀表板。 按一下並將它拖曳到適當的位置，然後依序按一下 [完成自訂] 和 [發行變更]。 

    ![含有計量的儀表板](media/app-insights-tutorial-dashboards/dashboard-03.png)


## <a name="metrics-explorer"></a>計量瀏覽器
[計量瀏覽器] 與 [計量] 類似，但新增至儀表板時可以自訂的程度高出許多。  將計量圖表化所使用的項目，取決於您特定的喜好設定和需求。

1. 在 Azure 功能表中選取 [Application Insights] ，然後選取您的應用程式。
1. 選取 [計量瀏覽器]。 
2. 按一下以編輯圖表，然後選取一或多個計量，或選取詳細的設定。  此範例顯示追蹤平均頁面回應時間的折線圖。
3. 按一下右上角的釘選圖示以將圖表新增到儀表板，然後將它拖曳到適當的位置。

    ![計量瀏覽器](media/app-insights-tutorial-dashboards/metrics-explorer.png)

4. [計量瀏覽器] 磚在新增到儀表板後可進行更多自訂。  以滑鼠右鍵按一下磚，然後選取 [編輯標題] 以新增自訂標題。  視需要繼續進行其他自訂。

    ![含有 [計量瀏覽器] 的儀表板](media/app-insights-tutorial-dashboards/dashboard-04a.png)

5. 您現在已將 [計量瀏覽器] 圖表新增至儀表板。

    ![含有 [計量瀏覽器] 的儀表板](media/app-insights-tutorial-dashboards/dashboard-04.png)

## <a name="add-analytics-query"></a>新增 Analytics 查詢
Azure Application Insights Analytics 提供豐富的查詢語言，可讓您分析 Application Insights 收集的所有資料。  如同圖表和其他檢視，您可以將 Analytics 查詢的輸出新增到儀表板。   

Azure Applications Insights Analytics 是分開的服務，因此您必須共用儀表板，儀表板才能包含 Analytics 查詢。  當您共用 Azure 儀表板時，便是將它發行為 Azure 資源，這樣就能將它提供給其他使用者和資源。  

1. 在儀表板畫面的頂端，按一下 [共用]。

    ![發佈儀表板](media/app-insights-tutorial-dashboards/publish-dashboard.png)

2. 將 [儀表板名稱] 維持不變，然後選取 [訂用帳戶名稱] 以共用儀表板。  按一下 [發佈] 。  儀表板現在已可供其他服務和訂用帳戶使用。  您也可以定義應具有儀表板存取權的特定使用者。
1. 在 Azure 功能表中選取 [Application Insights] ，然後選取您的應用程式。
2. 按一下畫面頂端的 [Analytics] 以開啟分析入口網站。

    ![啟動 Analytics](media/app-insights-tutorial-dashboards/start-analytics.png)

3. 輸入下列查詢會傳回最常要求的前 10 個頁面和其要求計數：

    ```
    requests
    | summarize count() by name
    | sort by count_ desc
    | take 10 
    ```

4. 按一下 [執行] 以驗證查詢的結果。
5. 按一下釘選圖示，然後選取您的儀表板名稱。  有別於先前的步驟使用的是最後一個儀表板，這個選項之所以讓您選取儀表板，是因為 Analytics 主控台是分開的服務，因此需從所有可用的共用儀表板中選取。

    ![釘選 Analytics 查詢](media/app-insights-tutorial-dashboards/analytics-pin.png)

5. 返回儀表板之前，請新增另一個查詢，但這次將它以圖形方式呈現，以觀察在儀表板中視覺化 Analytics 查詢的不同方式。  請從下列查詢開始，此查詢會彙總有最多例外狀況的前 10 個作業。

    ```
    exceptions
    | summarize count() by operation_Name
    | sort by count_ desc
    | take 10 
    ```

6. 選取 [圖表]，然後變更為 [環圈圖] 以視覺化輸出。

    ![Analytics 圖表](media/app-insights-tutorial-dashboards/analytics-chart.png)

6. 按一下釘選圖示以將圖表釘選至儀表板，這次請選取連結以返回儀表板。
4. 您所選格式的查詢結果現在已新增到您的儀表板。  按一下並將每個項目拖曳到適當的位置，然後按一下 [完成編輯]。
5. 以滑鼠右鍵按一下各個磚並選取 [編輯標題]，以提供描述性標題。

    ![含有 Analytics 的儀表板](media/app-insights-tutorial-dashboards/dashboard-05.png)

5. 按一下 [發行變更] 以將變更認可到您的儀表板。您的儀表板現在包含了各種來自 Application Insights 的圖表和視覺效果。


## <a name="next-steps"></a>後續步驟
由於您已學會如何建立自訂儀表板，請看看含有個案研究的其餘 Application Insights 說明文件。

> [!div class="nextstepaction"]
> [深入診斷](app-insights-devops.md)
