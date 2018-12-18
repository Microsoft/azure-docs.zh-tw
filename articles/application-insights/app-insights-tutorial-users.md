---
title: 在 Azure Application Insights 中了解您的客戶 | Microsoft Docs
description: 此教學課程說明如何藉由 Azure Application Insights，以了解客戶如何使用您的應用程式。
keywords: ''
services: application-insights
author: mrbullwinkle
ms.author: mbullwin
ms.date: 09/20/2017
ms.service: application-insights
ms.custom: mvc
ms.topic: tutorial
manager: carmonm
ms.openlocfilehash: db61c300ad82270e59d315fa3372d9e4390c7a21
ms.sourcegitcommit: 93902ffcb7c8550dcb65a2a5e711919bd1d09df9
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 11/09/2017
ms.locfileid: "24099016"
---
# <a name="use-azure-application-insights-to-understand-how-customers-are-using-your-application"></a>藉由 Azure Application Insights，以了解客戶如何使用您的應用程式

Azure Application Insights 會收集使用資訊，以協助您了解使用者如何與您的應用程式互動。  本教學課程將逐一介紹可用於分析此資訊的各種不同資源。  您將了解如何：

> [!div class="checklist"]
> * 針對存取應用程式的使用者，分析這些使用者的相關詳細資料
> * 使用工作階段資訊來分析客戶如何使用您的應用程式
> * 定義漏斗圖，以比較期望的使用者活動和實際的活動 
> * 建立活頁簿，以將視覺效果和查詢合併成單一文件
> * 將類似的使用者分為一組以共同分析
> * 了解哪些使用者會返回您的應用程式
> * 調查使用者如何瀏覽您的應用程式


## <a name="prerequisites"></a>先決條件

若要完成本教學課程：

- 使用下列工作負載安裝 [Visual Studio 2017](https://www.visualstudio.com/downloads/)：
    - ASP.NET 和 Web 開發
    - Azure 開發
- 下載並安裝 [Visual Studio 快照集偵錯工具](http://aka.ms/snapshotdebugger) \(英文\)。
- 將 .NET 應用程式部署至 Azure，並[啟用 Application Insights SDK](app-insights-asp-net.md)。 
- [從您的應用程式傳送遙測](app-insights-usage-overview.md#send-telemetry-from-your-app)以新增自訂事件/頁面檢視
- 傳送[使用者內容](https://docs.microsoft.com/azure/application-insights/app-insights-usage-send-user-context)以追蹤使用者過去的活動，並充分運用使用方式功能。

## <a name="log-in-to-azure"></a>登入 Azure
登入 Azure 入口網站，網址為 [https://portal.azure.com](https://portal.azure.com)。

## <a name="get-information-about-your-users"></a>取得使用者的相關資訊
[使用者] 面板可透過各種方式讓您了解有關使用者的重要詳細資料。 使用此面板可了解的資訊包括：使用者從何處連線、用戶端的詳細資料，以及他們所存取的應用程式領域等。 

1. 選取 [Application Insights]，然後選取您的訂用帳戶。
2. 選取功能表中的 [使用者]。
3. 預設檢視會顯示過去 24 小時連線到您的應用程式的唯一使用者數目。  您可以變更時間範圍，並設定各種其他準則以篩選這項資訊。

    ![查詢產生器](media\app-insights-tutorial-users\QueryBuilder.png)

6. 按一下 [期間] 下拉式清單，並變更為 7 天的時間範圍。  這會增加面板中不同圖表包含的資料。

    ![變更時間範圍](media\app-insights-tutorial-users\TimeRange.png)

4. 按一下 [分割依據] 下拉式清單，以在圖形中加入依使用者屬性分類的明細。  選取 [國家或地區]。  圖形包含相同的資料，但可讓您檢視每個國家/地區的使用者數目明細。

    ![國家或地區圖形](media\app-insights-tutorial-users\CountryorRegion.png)

5. 將游標放在圖表中不同列的上方，注意每個國家/地區的計數僅反映該列所代表的時間範圍。
6. 看看右邊執行使用者資料分析的 [Insights] 資料行。  這裡提供的資訊包括該期間內的唯一工作階段數目，以及含有表示使用者資料之一般內容的記錄 

    ![[Insights] 資料行](media\app-insights-tutorial-users\insights.png)


## <a name="analyze-user-sessions"></a>分析使用者工作階段
[工作階段] 面板和 [使用者] 面板類似。  [使用者] 可協助您了解存取您應用程式之使用者的詳細資料，[工作階段] 可協助您了解這些使用者使用應用程式的方式。  

1. 選取功能表中的 [工作階段]。
2. 看看圖形，注意您有和 [使用者] 面板中一樣的選項可以篩選和細分資料。

    ![工作階段查詢產生器](media\app-insights-tutorial-users\SessionsBuilder.png)

3. 右側的 [這些工作階段的範例] 窗格列出包含大量事件的工作階段。  這些有趣的工作階段可以作分析。

    ![這些工作階段的範例](media\app-insights-tutorial-users\SessionsSample.png)

4. 按一下其中一個工作階段以檢視其 [工作階段時間軸]，其中顯示工作階段中的每個動作。  這可協助您識別資訊，例如有大量例外狀況的工作階段。

    ![工作階段時間軸](media\app-insights-tutorial-users\SessionsTimeline.png)

## <a name="group-together-similar-users"></a>將類似的使用者分為一組
[世代] 是依據類似特性分組的使用者。  在其他面板中使用世代來篩選資料，可以分析特定的使用者群組。  例如，您可能只想分析完成購物的使用者。

1.  選取功能表中的 [世代]。
2.  按一下 [新增] 以建立新的世代。
3.  選取 [使用過的人] 下拉式清單並選取一個動作。  只有在報告的這個時間範圍內執行此動作的使用者會包含在內。

    ![執行指定動作的世代](media\app-insights-tutorial-users\CohortsDropdown.png)

4.  選取功能表中的 [使用者]。
5.  在 [顯示] 下拉式清單中，選取您剛才建立的世代。  圖形的資料將僅限於那些使用者。

    ![使用者工具中的世代](media\app-insights-tutorial-users\UsersCohort.png)


## <a name="compare-desired-activity-to-reality"></a>比較期望的和實際的活動
先前的面板均著重在應用程式使用者的活動，而 [漏斗圖] 則著重在您想要使用者進行的活動。  漏斗圖代表應用程式中的一組步驟，以及在步驟之間移動的使用者百分比。  例如，您可以建立漏斗圖，來測量連線到應用程式搜尋產品的使用者百分比。  您就能看到將該產品放入購物車的使用者百分比，還有完成購物的使用者百分比。

1. 選取功能表中的 [漏斗圖]，然後按一下 [新增]。 

    ![](media\app-insights-tutorial-users\funnelsnew.png)

2. 輸入 [漏斗圖名稱]。
3. 針對每個步驟選取一個動作，以建立至少包含兩個步驟的漏斗圖。  此動作清單是根據 Application Insights 所收集的使用方式資料建立的。

    ![](media\app-insights-tutorial-users\funnelsedit.png)

4. 按一下 [儲存] 儲存漏斗圖，然後檢視其結果。  漏斗圖右邊的視窗顯示第一個活動前和最後一個活動後最常見的事件，以協助您了解特定序列周圍的使用者傾向。

    ![](media\app-insights-tutorial-users\funnelsright.png)


## <a name="learn-which-customers-return"></a>了解哪些客戶會返回
[保留期] 可協助您了解哪些使用者重新回到您的應用程式。  

1. 選取功能表中的 [保留期]。
2. 根據預設，這項分析資訊包含了執行任一動作後又返回執行任一動作的使用者。  您可以將這個篩選條件變更為任一篩選條件，例如只包含完成購物後又返回的使用者。

    ![](media\app-insights-tutorial-users\retentionquery.png)

3. 符合準則的返回使用者，以有著不同持續時間的圖形化和表格格式顯示。  這個典型的模式象徵返回的使用者數目隨著時間逐漸下降。  一個時段到下一時段若出現驟降的趨勢，可能需要關注。 

    ![](media\app-insights-tutorial-users\retentiongraph.png)

## <a name="analyze-user-navigation"></a>分析使用者瀏覽
[使用者流程] 會將使用者在您應用程式的頁面和功能之間瀏覽的方式視覺化。  這可協助您回答以下問題：例如，使用者通常從特定的頁面移往何處、他們通常如何結束您的應用程式，以及是否有任何動作會定期重複。

1.  選取功能表中的 [使用者流程]。
2.  按一下 [新增] 以建立新的使用者流程，然後按一下 [編輯] 以編輯其詳細資料。
3.  將 [時間範圍] 增加至 7 天，然後選取一個初始事件。  流程將會追蹤以該事件開始的使用者工作階段。

    ![](media\app-insights-tutorial-users\flowsedit.png)

4.  顯示使用者流程，您可以看到不同的使用者路徑和其工作階段計數。  藍線表示使用者在目前的動作之後所執行的動作。  紅線表示使用者工作階段結束。

    ![](media\app-insights-tutorial-users\flows.png)

5.  若要將某事件從流程中移除，按一下該動作角落的 **x**，然後按一下 [建立圖表]。  重繪後的圖形會移除該事件所有的執行個體。  按一下 [編輯]，查看該事件現在是否已新增至 [排除的事件]。

    ![](media\app-insights-tutorial-users\flowsexclude.png)

## <a name="consolidate-usage-data"></a>合併使用方式資料
[活頁簿] 可將資料視覺效果、Analytics 查詢及文字合併成互動式文件。  您可以使用活頁簿來彙整常見的使用方式資訊、合併來自特定事件的資訊，或向您的小組回報應用程式的使用方式。

1.  選取功能表中的 [活頁簿]。
2.  按一下 [新增] 以建立新的活頁簿。
3.  提供的現有查詢會以橫條圖顯示前一天的所有使用方式資料。  您可以使用這個查詢、以手動方式編輯它，或按一下 [範例查詢] 以選取其他有用的查詢。

    ![](media\app-insights-tutorial-users\samplequeries.png)

4.  按一下 [完成編輯]。
5.  按一下上方窗格中的 [編輯]，以編輯活頁簿頂端的文字。  文字會使用 Markdown 格式化。

    ![](media\app-insights-tutorial-users\markdown.png)

6.  按一下 [新增使用者]，以新增含有使用者資訊的圖形。  視需要編輯圖形的詳細資料，然後按一下 [完成編輯] 儲存它。


## <a name="next-steps"></a>後續步驟
由於您已學會如何分析使用者，請前進到下一個教學課程，了解如何建立自訂儀表板以結合此資訊和關於您應用程式的其他有用資料。

> [!div class="nextstepaction"]
> [建立自訂儀表板](app-insights-tutorial-dashboards.md)
