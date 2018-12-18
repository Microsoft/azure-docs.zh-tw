---
title: 在 Azure Application Insights 中使用搜尋服務 | Microsoft Docs
description: 搜尋和篩選 Web 應用程式傳送的原始遙測。
services: application-insights
documentationcenter: ''
author: mrbullwinkle
manager: carmonm
ms.assetid: 2a437555-8043-45ec-937a-225c9bf0066b
ms.service: application-insights
ms.workload: tbd
ms.tgt_pltfrm: ibiza
ms.devlang: na
ms.topic: conceptual
ms.date: 09/20/2018
ms.author: mbullwin
ms.openlocfilehash: 1bf716d4e5a37739de2e820725449be2576054ea
ms.sourcegitcommit: 4ecc62198f299fc215c49e38bca81f7eb62cdef3
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "47038761"
---
# <a name="using-search-in-application-insights"></a>在 Application Insights 中使用搜尋
「搜尋」是 [Application Insights](app-insights-overview.md) 的一項功能，可用來尋找及探索個別遙測項目，例如頁面檢視、例外狀況或 Web 要求。 而您可以檢視所編寫的記錄追蹤和事件。

(若要對您的資料執行更複雜的查詢，請使用[分析](app-insights-analytics-tour.md)。)

## <a name="where-do-you-see-search"></a>「搜尋」在哪裡？

### <a name="in-the-azure-portal"></a>在 Azure 入口網站

從應用程式的 Application Insights [概觀] 刀鋒視窗中，就可以直接開啟診斷搜尋︰

![Open diagnostic search](./media/app-insights-diagnostic-search/001.png)

![診斷搜尋圖表的螢幕擷取畫面](./media/app-insights-diagnostic-search/002.png)

「診斷搜尋」的主體是遙測項目的清單 - 伺服器要求、頁面檢視、您編寫的自訂事件等等。 在清單上方是摘要圖表，顯示一般時間的事件計數。

按一下 [重新整理] 以取得新的事件。

### <a name="in-visual-studio"></a>在 Visual Studio 中

在 Visual Studio 中，也有 [Application Insights 搜尋] 視窗。 它最適合顯示您偵錯的應用程式所產生的遙測事件。 還可以顯示 Azure 入口網站上從您已發佈的應用程式收集的事件。

在 Visual Studio 中開啟 [搜尋] 視窗：

![Visual Studio 開啟 Application Insights 搜尋](./media/app-insights-diagnostic-search/32.png)

[搜尋] 視窗有類似 Web 入口網站的功能：

![Visual Studio Application Insights 搜尋視窗](./media/app-insights-diagnostic-search/34.png)

您開啟要求或頁面檢視時可以看到 [追蹤作業] 索引標籤。 「作業」是與單一要求或頁面檢視相關聯的一連串事件。 例如，相依性呼叫、例外狀況、追蹤記錄檔和自訂事件可能都是單一作業的一部分。 針對要求或頁面檢視，[追蹤作業] 索引標籤會以圖形顯示這些事件的時刻和持續時間。 

## <a name="inspect-individual-items"></a>檢查個別項目

選取任何遙測項目來查看重要欄位和相關項目。

![個別相依性要求的螢幕擷取畫面](./media/app-insights-diagnostic-search/003.png)

這將會啟動端對端交易詳細資料檢視：

![端對端交易詳細資料檢視的螢幕擷取畫面。](./media/app-insights-diagnostic-search/004.png)

## <a name="filter-event-types"></a>篩選事件類型
開啟 [篩選器] 分頁並選擇要查看的事件類型。 (如果之後想要還原您所開啟分頁的篩選器，請按一下 [重設]。)

![選擇篩選器並選取遙測類型](./media/app-insights-diagnostic-search/02-filter-req.png)

事件類型包括：

* **追蹤** - [診斷記錄](app-insights-asp-net-trace-logs.md)，包括 TrackTrace、log4Net、NLog 和 System.Diagnostic.Trace 呼叫。
* **要求** - 伺服器應用程式收到的 HTTP 要求，包括頁面、指令碼、影像、樣式檔案和資料。 這些事件是用來建立要求和回應概觀圖表。
* **頁面檢視** - [Web 用戶端傳送的遙測](app-insights-javascript.md)，用來建立頁面檢視報告。 
* **自訂事件** - 如果您插入 TrackEvent() 呼叫來[監視使用量](app-insights-api-custom-events-metrics.md)，則可以在這裡搜尋它們。
* **例外狀況** - [伺服器中未捕捉到的例外狀況](app-insights-asp-net-exceptions.md)，以及您使用 TrackException() 記錄的例外狀況。
* **相依性** - [從伺服器應用程式呼叫](app-insights-asp-net-dependencies.md)其他服務 (例如 REST API 或資料庫)，以及來自[用戶端程式碼](app-insights-javascript.md)的 AJAX 呼叫。
* **可用性** - [可用性測試](app-insights-monitor-web-app-availability.md)的結果。

## <a name="filter-on-property-values"></a>依據屬性值篩選
您可以依屬性的值篩選事件。 可用的屬性因您所選取的事件類型而定。 

例如，找出具有特定回應代碼的要求。 

![展開屬性並選擇值](./media/app-insights-diagnostic-search/03-response500.png)

如果未針對特定屬性選擇任何值，則相當於選擇全部的值。 這樣會關掉該屬性的篩選功能。

### <a name="narrow-your-search"></a>縮小搜尋
篩選器值右側的計數顯示目前篩選的集合中有多少個出現次數。 

在此範例中，很明顯是 'Rpt/Employees' 要求造成大多數的 500 錯誤：

![展開屬性並選擇值](./media/app-insights-diagnostic-search/04-failingReq.png)

## <a name="find-events-with-the-same-property"></a>尋找具有相同屬性的事件
尋找具有相同屬性值的所有項目：

![以滑鼠右鍵按一下屬性](./media/app-insights-diagnostic-search/12-samevalue.png)

## <a name="search-the-data"></a>搜尋資料

> [!NOTE]
> 若要撰寫更複雜的查詢，請從 [搜尋] 刀鋒視窗頂端開啟 [[分析](app-insights-analytics-tour.md)]。
> 

您可以搜尋任何屬性值中的字詞。 如果您已編寫具有屬性值的[自訂事件](app-insights-api-custom-events-metrics.md)，這特別實用。 

您可能想要設定時間範圍，因為對一小段時間範圍進行的搜尋較快速。 

![Open diagnostic search](./media/app-insights-diagnostic-search/appinsights-311search.png)

請搜尋完整單字，而不是子字串。 使用引號來括住的特殊字元。

| 字串 | 這樣「找不到」 | 這樣找得到 |
| --- | --- | --- |
| HomeController.About |home<br/>controller<br/>out | homecontroller<br/>about<br/>"homecontroller.about"|
|美國|Uni<br/>ted|united<br/>states<br/>united AND states<br/>"united states"

以下是您可以使用搜尋運算式：

| 範例查詢 | 效果 |
| --- | --- |
| `apple` |在日期範圍中尋找欄位含有 "apple" 這個字的所有事件 |
| `apple AND banana` <br/>`apple banana` |尋找同時含有這兩個字的事件。 請使用大寫 "AND"，而不是 "and"。 <br/>簡短格式。 |
| `apple OR banana` |尋找含有任一單字的事件。 請使用 "OR"，而不是 "or"。 |
| `apple NOT banana` |尋找含有一個字但不含另一個字的事件。 |

## <a name="sampling"></a>取樣
如果您的應用程式會產生大量遙測 (且您使用 ASP.NET SDK 版本 2.0.0-beta3 或更新版本)，則調適性取樣模組會只傳送事件代表性片段，自動減少傳送至入口網站的資料量。 不過，同一個要求相關的事件是以整組方式來選取或取消選取，方便您切換瀏覽相關事件。 

[了解取樣](app-insights-sampling.md)。

## <a name="create-work-item"></a>建立工作項目
您可以利用來自任何遙測項目的詳細資料，在 GitHub 或 Azure DevOps 中建立 Bug。 

![按一下 [新增工作項目]、編輯欄位，然後按一下 [確定]。](./media/app-insights-diagnostic-search/42.png)

當您第一次執行此動作時，系統會要求您設定與 Azure DevOps 組織和專案的連結。

![填入 Azure DevOps Services 的 URL 和專案名稱，然後按一下 [授權]](./media/app-insights-diagnostic-search/41.png)

(您也可以在 [工作項目] 刀鋒視窗上設定連結)。

## <a name="send-more-telemetry-to-application-insights"></a>傳送更多遙測到 Application Insights
除了 Application Insights SDK 傳送的內建遙測，您可以：

* 在 [.NET](app-insights-asp-net-trace-logs.md) 或 [Java](app-insights-java-trace-logs.md) 中，從您最喜愛的紀錄架構擷取記錄追蹤。 這表示您可以搜尋您的記錄追蹤，並將它們與頁面檢視、例外狀況和其他事件相互關聯。 
* [編寫程式碼](app-insights-api-custom-events-metrics.md)以傳送自訂事件、頁面檢視和例外狀況。 

[了解如何將記錄和自訂遙測傳送至 Application Insights](app-insights-asp-net-trace-logs.md)。

## <a name="questions"></a>問與答
### <a name="limits"></a>保留多少資料？

請參閱[限制摘要](app-insights-pricing.md#limits-summary)。

### <a name="how-can-i-see-post-data-in-my-server-requests"></a>我如何查看我的伺服器要求中的 POST 資料？
我們不會自動記錄 POST 資料，但您可以使用 [TrackTrace 或記錄呼叫](app-insights-asp-net-trace-logs.md)。 將 POST 資料放在訊息參數中。 您無法像篩選屬性一樣來篩選訊息，但大小限制比較長。

## <a name="video"></a>影片

> [!VIDEO https://channel9.msdn.com/events/Connect/2016/112/player]

## <a name="add"></a>後續步驟
* [在分析中撰寫複雜的查詢](app-insights-analytics-tour.md)
* [將記錄和自訂遙測傳送至 Application Insights](app-insights-asp-net-trace-logs.md)
* [設定可用性和回應性測試](app-insights-monitor-web-app-availability.md)
* [疑難排解](app-insights-troubleshoot-faq.md)
