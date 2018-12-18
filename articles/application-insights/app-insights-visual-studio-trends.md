---
title: 在 Visual Studio 中分析趨勢 | Microsoft Docs
description: 在 Visual Studio 中分析、 視覺化及探索您的 Application Insights 遙測的趨勢。
services: application-insights
documentationcenter: .net
author: mrbullwinkle
manager: carmonm
ms.assetid: 3150c6fc-2691-44f6-a290-fc5cd68e692a
ms.service: application-insights
ms.custom: vs-azure
ms.workload: azure-vs
ms.tgt_pltfrm: ibiza
ms.devlang: na
ms.topic: conceptual
ms.date: 03/17/2017
ms.reviewer: daviste
ms.author: mbullwin
ms.openlocfilehash: 17303186b176c6d14b1c895d17d21c0f29cdcdcf
ms.sourcegitcommit: fab878ff9aaf4efb3eaff6b7656184b0bafba13b
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/22/2018
ms.locfileid: "42441784"
---
# <a name="analyzing-trends-in-visual-studio"></a>在 Visual Studio 中分析趨勢 | Microsoft Azure
「Application Insights 趨勢」工具會以視覺化方式呈現 Web 應用程式的重要遙測事件如何隨著時間變更，協助您快速識別問題和異常。 將您連結至更詳細的診斷資訊，「趨勢」可協助您改善應用程式效能、追蹤例外狀況的原因，以及揭露您的自訂事件情資。

![範例趨勢視窗](./media/app-insights-visual-studio-trends/app-insights-trends-hero-750.png)

## <a name="configure-your-web-app-for-application-insights"></a>針對 Application Insights 設定您的 Web 應用程式

如果您尚未這麼做，請[針對 Application Insights 設定您的 Web 應用程式](app-insights-overview.md)。 這可讓它將遙測資料傳送至 Application Insights 入口網站。 趨勢工具會從那裡讀取遙測資料。

在 Visual Studio 2015 Update 3 及更新版本中可取得 Application Insights 趨勢。

## <a name="open-application-insights-trends"></a>開啟 Application Insights 趨勢
若要開啟 [Application Insights 趨勢] 視窗：

* 從 Application Insights 工具列按鈕，選擇 [探索遙測趨勢] ，或
* 從專案操作功能表，選擇 [Application Insights] > [探索遙測趨勢]，或
* 從 Visual Studio 功能表列，選擇 [檢視] > [其他視窗] > [Application Insights 趨勢]。

您可能會看到選取資源的提示。 按一下 [選取資源] ，登入 Azure 訂用帳戶，然後從您要分析遙測趨勢的清單中選擇 Application Insights 資源。

## <a name="choose-a-trend-analysis"></a>選擇趨勢分析
![趨勢分析的常見類型功能表](./media/app-insights-visual-studio-trends/app-insights-trends-1-750.png)

首先選擇其中一個通用趨勢分析 (共 5 個)，而這些通用趨勢分析會各自分析過去 24 小時內的資料︰

* **調查伺服器要求的效能問題** - 對您的服務所做的要求 (依回應時間分組)
* **分析伺服器要求中的錯誤** - 對您的服務所做的要求 (依 HTTP 回應碼分組)
* **檢查應用程式中的例外狀況** - 您的服務的例外狀況 (依例外狀況類型分組)
* **檢查應用程式相依項目的效能** - 您的服務所呼叫的服務 (依回應時間分組)
* **檢查您的自訂事件** - 您為服務設定的自訂事件 (依事件類型分組)。

稍後可從 [趨勢] 視窗左上角的 [檢視常見的遙測分析類型]  按鈕取得這些預先建置的分析。

## <a name="visualize-trends-in-your-application"></a>以視覺化方式呈現您的應用程式的趨勢
Application Insights 趨勢會從您的應用程式的遙測建立時間序列視覺效果。 每一個時間序列視覺效果會顯示某些時間範圍內一種類型的遙測 (依該遙測的其中一個屬性分組)。 例如，您可能想要檢視過去 24 小時內的伺服器要求 (依源自的國家/地區分組)。 在此範例中，視覺效果上的每個泡泡會代表在一小時期間內某個國家/區域的伺服器要求計數。

使用視窗頂端的控制項來調整您可檢視的遙測類型。 首先，選擇您感興趣的遙測類型︰

* **遙測類型** - 伺服器要求、例外狀況、相依性或自訂事件
* **時間範圍** - 從過去 30 分鐘到過去 3 天的任何時間
* **分組依據** - 例外狀況類型、問題識別碼、國家/區域等等。

然後，按一下 [分析遙測]  來執行查詢。

若要在視覺效果中的泡泡之間巡覽︰

* 按一下以選取泡泡，該泡泡會更新視窗底部的篩選器，而且只彙整在特定期間內發生的事件
* 按兩下泡泡以瀏覽至「搜尋」工具，並查看在該時間內發生的所有個別遙測事件
* 按住 CTRL 鍵並按一下泡泡，以在視覺效果中取消選取該泡泡。

> [!TIP]
> 「趨勢」和「搜尋」工具一起運作，可協助您在數千個遙測事件中找出您服務的問題原因。 例如，如果您的客戶在某個下午通知您的應用程式較無回應，請開始使用「趨勢」。 分析在過去幾個小時內對您的服務所做的要求 (依回應時間分組)。 查看是否有非常大量的慢速要求。 然後連按兩下該泡泡來移至「搜尋」工具 (已篩選成這些要求事件)。 在「搜尋」中，您可以探索這些要求的內容並巡覽至相關程式碼來解決此問題。
> 
> 

## <a name="filter"></a>Filter
使用視窗底部的篩選控制項，探索更明確的趨勢。 若要套用篩選器，請按一下其名稱。 您可以快速切換不同的篩選器，以探索可能隱藏在遙測的特定維度中的趨勢。 如果您在一個維度 (如例外狀況類型) 中套用一個篩選器，則其他維度中的篩選器即使呈現灰色，仍然可加以點選。若要取消套用篩選器，請再按一次。 按住 CTRL 鍵並按一下滑鼠，以選取相同維度中的多個篩選器。

![趨勢篩選器](./media/app-insights-visual-studio-trends/TrendsFiltering-750.png)

如果您想要套用多個篩選器，該怎麼辦？ 

1. 套用第一個篩選器。 
2. 按一下第一個篩選器的維度名稱旁邊的 [套用選取的篩選器並再次查詢]  按鈕。 這只會重新查詢您的遙測中符合第一個篩選條件的事件。 
3. 套用第二個篩選器。 
4. 重複此程序以在遙測的特定子集中尋找趨勢。 例如，名為 "GET Home/Index"「且」來自德國「且」收到 500 回應碼的伺服器要求。 

若要取消套用上述其中一個篩選器，請按一下維度的 [移除選取的篩選器並再次查詢]  按鈕。

![多個篩選器](./media/app-insights-visual-studio-trends/TrendsFiltering2-750.png)

## <a name="find-anomalies"></a>尋找異常
「趨勢」工具會醒目提示異常的事件泡泡 (相較於同一個時間序列中的其他泡泡)。 在 [檢視類型] 下拉式清單中，選擇 [時間值區中的計數 (醒目提示異常)] 或 [時間值區中的百分比 (醒目提示異常)]。 紅色泡泡為異常。 異常的定義是計數/百分比超過 2.1 乘以在過去兩個時間週期 (如果您正在檢視過去 24 小時的資料，則為 48 小時) 內發生之計數/百分比的標準差的泡泡。

![彩色的點表示異常](./media/app-insights-visual-studio-trends/TrendsAnomalies-750.png)

> [!TIP]
> 在小型泡泡的時間序列中尋找可能看起來大小相似的離群值時，醒目提示異常特別實用。  
> 
> 

## <a name="next"></a>後續步驟
|  |  |
| --- | --- |
| **[在 Visual Studio 中使用 Application Insights](app-insights-visual-studio.md)**<br/>搜尋遙測、查看 CodeLens 中的資料，以及設定 Application Insights。 盡在 Visual Studio 中。 |![以滑鼠右鍵按一下專案，然後選擇 [Application Insights]、[搜尋]](./media/app-insights-visual-studio-trends/34.png) |
| **[新增更多測試](app-insights-asp-net-more.md)**<br/>監視使用狀況、可用性、相依性、例外狀況。 整合來自記錄架構的追蹤。 撰寫自訂遙測。 |![Visual Studio](./media/app-insights-visual-studio-trends/64.png) |
| **[使用 Application Insights 入口網站](app-insights-dashboards.md)**<br/>儀表板、功能強大的診斷和分析工具、警示、即時的應用程式相依性對應，以及遙測匯出等功能。 |![Visual Studio](./media/app-insights-visual-studio-trends/62.png) |

