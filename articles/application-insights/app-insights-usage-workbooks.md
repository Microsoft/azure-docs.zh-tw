---
title: 調查使用方式資料並且與 Azure Application Insights 中的互動式活頁簿共用 | Microsoft docs
description: 您 Web 應用程式的使用者人口統計分析。
services: application-insights
documentationcenter: ''
author: mrbullwinkle
manager: carmonm
ms.service: application-insights
ms.workload: tbd
ms.tgt_pltfrm: ibiza
ms.devlang: multiple
ms.topic: article
ms.date: 06/12/2017
ms.author: mbullwin; daviste
ms.openlocfilehash: a871378b3e2cc0b34c925593c6f01952de3aa08e
ms.sourcegitcommit: 870d372785ffa8ca46346f4dfe215f245931dae1
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 05/08/2018
---
# <a name="investigate-and-share-usage-data-with-interactive-workbooks-in-application-insights"></a>調查使用方式資料並且與 Application Insights 中的互動式活頁簿共用

活頁簿將 [Azure Application Insights](app-insights-overview.md) 資料視覺效果、[分析查詢](app-insights-analytics.md) 及文字合併為互動式文件。 活頁簿可以由具有相同 Azure 資源存取權的其他小組成員編輯。 這表示用來建立活頁簿的查詢和控制項都可供讀取活頁簿的其他人使用，使其易於探索、擴充和檢查錯誤。

活頁簿對於下列案例很有用：

* 當您事先不知道下列感興趣的計量時，探索應用程式的使用方式：使用者數目、保留率、轉換率等等。不同於 Application Insights 中的其他使用方式分析工具，活頁簿可讓您合併多種類型的視覺效果與分析，使其更適合這種自由形式探索。
* 向小組說明新發行的功能如何執行，方法是顯示關鍵互動和其他計量的使用者計數。
* 與小組的其他成員分享應用程式中 A/B 實驗的結果。 您可以使用文字說明實驗的目標，然後顯示用來評估實驗的每個使用方式計量和分析查詢，以及每個計量是高於或低於目標的清楚圖說文字。
* 報告應用程式使用方式中斷的影響，合併資料、文字說明和下一步的討論，以防止未來的中斷。

> [!NOTE]
> 您的 Application Insights 資源必須包含分頁檢視或自訂事件，以便使用活頁簿。 [了解如何設定應用程式以使用 Application Insights JavaScript SDK 自動收集分頁檢視](app-insights-javascript.md)。
> 
> 

## <a name="editing-rearranging-cloning-and-deleting-workbook-sections"></a>編輯、重新排列、複製和刪除活頁簿區段

活頁簿是由下列區段組成：獨立的可編輯使用方式視覺效果、圖表、資料表、文字或分析查詢結果。

若要編輯活頁簿區段的內容，請按一下活頁簿區段右下方的 [編輯] 按鈕。

![Application Insights 活頁簿區段編輯控制項](./media/app-insights-usage-workbooks/editing-controls.png)

1. 當您完成編輯區段時，按一下區段左下角的 [完成編輯]。

2. 若要建立區段的複本，按一下 [複製此區段] 圖示。 建立重複的區段是逐一查看查詢而不會遺失先前反覆項目的絕佳方式。

3. 若要在活頁簿中將區段上移，按一下 [上移] 或 [下移] 圖示。

4. 若要永久移除區段，按一下 [移除] 圖示。

## <a name="adding-usage-data-visualization-sections"></a>新增使用方式資料視覺效果區段

活頁簿提供四種類型的內建使用方式分析視覺效果。 每個種類會回答應用程式使用方式的一般問題。 若要新增這四個區段以外的資料表和圖表，請新增分析查詢區段 (如下所示)。

若要將 [使用者]、[工作階段]、[事件] 或 [保留期] 區段新增至活頁簿，請使用活頁簿底端或任何區段底端的 [新增使用者] 或其他對應按鈕。

![活頁簿中的使用者區段](./media/app-insights-usage-workbooks/users-section.png)

[使用者] 區段會回答「有多少使用者檢視我的網站的某些分頁或使用某些功能？」

[工作階段] 區段會回答「使用者耗費多少工作階段來檢視我的網站的某些分頁或使用某些功能？」

[事件] 區段會回答「使用者檢視我的網站的某些分頁或使用某些功能多少次？」

這三個區段類型會提供相同的控制項和視覺效果集合：

* [深入了解編輯使用者、工作階段和事件區段](app-insights-usage-segmentation.md)
* 切換主要圖表、長條圖格線、自動深入解析及範例使用者視覺效果，方法是使用每個區段頂端的 [顯示圖表]、[顯示格線]、[顯示深入解析] 和 [這些使用者的範例] 核取方塊。

![活頁簿中的保留期區段](./media/app-insights-usage-workbooks/retention-section.png)

[保留期] 區段會回答「在某天或某週檢視某些分頁或使用某些功能的人員中，有多少人員在後續一天或一週回來？」

* [深入了解編輯保留期區段](app-insights-usage-retention.md)
* 切換選擇性整體保留期圖表，方法是使用區段頂端的 [顯示整體保留期圖表] 核取方塊。

## <a name="adding-application-insights-analytics-sections"></a>新增 Application Insights 分析區段

![活頁簿中的分析區段](./media/app-insights-usage-workbooks/analytics-section.png)

若要將 Application Insights 分析查詢區段新增至您的活頁簿，請使用活頁部底端或任何區段底端的 [新增分析查詢] 按鈕。

分析查詢區段可讓您將 Application Insights 資料的任意查詢新增至活頁簿。 這種彈性表示分析查詢區段應該是您在回答上述四個區段 (使用者、工作階段、事件和保留期) 以外的任何網站相關問題時的一時之選，例如：

* 在與拒絕使用方式相同的時間週期期間，您的網站擲回多少例外狀況？
* 檢視某些分頁之使用者的分頁載入時間散發是多久？
* 有多少使用者檢視您的網站上的某些分頁集合，但是未檢視其他分頁集合？ 如果您有叢集使用者使用網站功能的不同子集 (使用 `join` 運算子搭配 Log Analytics 查詢語言的 `kind=leftanti` 修飾詞)，這項功能相當有用。

使用 [Log Analytics 查詢語言參考](https://docs.loganalytics.io/)以深入了解撰寫查詢。

## <a name="adding-text-and-markdown-sections"></a>新增文字和 Markdown 區段

將標題、說明及註解新增至您的活頁簿，可協助將一組資料表和圖表變成敘述。 活頁簿中的文字區段支援 [Markdown 語法](https://daringfireball.net/projects/markdown/)以進行文字格式設定，例如標題、粗體、斜體和項目符號清單。

若要將文字區段新增至您的活頁簿，請使用活頁部底端或任何區段底端的 [新增文字] 按鈕。

## <a name="saving-and-sharing-workbooks-with-your-team"></a>儲存活頁簿並且與小組共用

活頁簿會儲存在 Application Insights 資源中，儲存在您私人使用的 [我的報表] 區段中，或是可供具有 Application Insights 資源存取權的每個人存取的 [共用報表] 區段中。 若要檢視資源中的所有活頁簿，請按一下動作列中的 [開啟] 按鈕。

若要共用目前在 [我的報表] 的活頁簿：

1. 按一下動作列中的 [開啟]
2. 按一下您想要共用之活頁簿旁邊的 [...] 按鈕
3. 按一下 [移至共用報表]。

若要使用連結或透過電子郵件共用活頁簿，請按一下動作列中的 [共用]。 請注意，連結的收件者需要在 Azure 入口網站中存取此資源的存取權，才能檢視活頁簿。 若要進行編輯，收件者至少需要資源的參與者權限。

若要將活頁簿的連結釘選到 Azure 儀表板：

1. 按一下動作列中的 [開啟]
2. 按一下您想要釘選之活頁簿旁邊的 [...] 按鈕
3. 按一下 [釘選到儀表板]。

## <a name="next-steps"></a>後續步驟

## <a name="next-steps"></a>後續步驟
- 若要啟用使用體驗，請開始傳送[「自訂事件」](https://docs.microsoft.com/azure/application-insights/app-insights-api-custom-events-metrics#trackevent) 或 [「頁面檢視」](https://docs.microsoft.com/azure/application-insights/app-insights-api-custom-events-metrics#page-views)。
- 如果您已傳送自訂事件或頁面檢視，請探索「使用量工具」，以了解使用者如何使用您的服務。
    - [使用者、工作階段、事件](app-insights-usage-segmentation.md)
    - [漏斗圖](usage-funnels.md)
    - [保留](app-insights-usage-retention.md)
    - [使用者流程](app-insights-usage-flows.md)
    - [新增使用者內容](app-insights-usage-send-user-context.md)
    
