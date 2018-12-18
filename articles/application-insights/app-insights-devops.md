---
title: Web 應用程式效能監視 - Azure Application Insights | Microsoft Docs
description: Application Insights 如何融入 DevOps 循環
services: application-insights
documentationcenter: ''
author: mrbullwinkle
manager: carmonm
ms.assetid: 479522a9-ff5c-471e-a405-b8fa221aedb3
ms.service: application-insights
ms.workload: tbd
ms.tgt_pltfrm: ibiza
ms.devlang: multiple
ms.topic: conceptual
ms.date: 03/14/2017
ms.author: mbullwin
ms.openlocfilehash: e77389411e52853efb52252a17c8612f0480fa61
ms.sourcegitcommit: f3bd5c17a3a189f144008faf1acb9fabc5bc9ab7
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/10/2018
ms.locfileid: "44298637"
---
# <a name="deep-diagnostics-for-web-apps-and-services-with-application-insights"></a>使用 Application Insights 深入診斷 Web 應用程式
## <a name="why-do-i-need-application-insights"></a>我為什麼需要 Application Insights 呢？
Application Insights 會監視您正在執行的 Web 應用程式。 它會告訴您有關失敗和效能的問題，並協助您分析客戶如何使用您的應用程式。 它適用於在許多平台 (ASP.NET、J2EE、Node.js ...) 上執行的應用程式，並可裝載於雲端或內部部署。 

![傳遞 Web 應用程式的複雜度層面](./media/app-insights-devops/010.png)

在執行時現代應用程式加以監視十分重要。 最重要的是，您想要在大部分客戶之前偵測到失敗。 您也會想要找出並修正效能問題，儘管這並不嚴重，但或許有某些動作速度變慢，或對您的使用者造成些許不便。 當您滿意系統的執行時，會想要知道使用者正利用它來執行哪些動作︰他們正在使用最新功能嗎？ 是否已成功使用它？

現代 Web 應用程式是以持續傳遞週期來開發的︰發行新功能或改進功能；觀察其對於使用者的適用程度；根據該知識來規劃下一個增量開發。 此週期的關鍵部分是觀察階段。 Application Insights 提供工具來監視 Web 應用程式的效能和使用方式。

此程序最重要的方面是診斷和判斷。 如果應用程式失敗，則會造成商務損失。 因此，監視架構的主要角色是以可靠的方式偵測失敗、立即通知您，並為您呈現診斷問題所需的資訊。 這就是 Application Insights 的確實功用。

### <a name="where-do-bugs-come-from"></a>Bug 來自何處？
Web 系統中的失敗通常是由組態問題或其許多元件之間的互動不良所引起。 因此，處理即時網站事件時的首要任務就是找出問題所在︰原因出自於哪些元件或關聯性？

我們之中有些人 (灰白頭髮的那些人) 可能還記得那個電腦程式只在一部電腦中執行的單純年代。 開發人員會在交付之前徹底地測試該程式；但交付之後，就很少會再看見或聽到任何有關該程式的事。 使用者必須忍受殘留的 Bug 長達數年之久。 

但現在的情況已變得非常不同。 您的應用程式要在眾多不同的裝置上執行，而且很難保證在每一個裝置上都會有完全相同的行為。 在雲端中裝載應用程式，表示 Bug 可以快速修正，但這也表示競賽持續進行，並預期新功能會以更頻繁的間隔出現。 

在這些情況下，確實控制 Bug 計數的唯一方式是將單位測試自動化。 您無法在每次交付時手動重新測試一切。 單元測試現在是建置程序中常見的一部分。 像是 Xamarin Test Cloud 的工具可藉由在多個瀏覽器版本中提供自動化 UI 測試來提供協助。 這些測試制度讓我們能夠期望在應用程式內發現 Bug 的速度可以保持在最低限度。

典型的 Web 應用程式有許多即時元件。 除了用戶端 (在瀏覽器或裝置應用程式中) 和 Web 伺服器以外，還可能是實質的後端處理。 後端或許是元件的管線，或是鬆散的共同作業項目集合。 此外，它們其中有許多都不在您的控制內 - 它們是您所需的外部服務。

在這類組態中，很難以經濟實惠的方式測試或預見每個可能的失敗模式 (除了即時系統本身)。 

### <a name="questions-"></a>問題 ...
當我們開發 Web 系統時，會自問數個問題：

* 我的應用程式會當機嗎？ 
* 實際發生什麼事？ - 如果要求失敗，我想要知道問題出在哪裡。 我們需要追蹤事件 ...
* 我的應用程式速度夠快嗎？ 回應典型的要求需要多久的時間？
* 伺服器可以處理負載嗎？ 當要求的速率增加時，可以維持穩定的回應時間嗎？
* 如何回應是我的相依性 - REST API、資料庫，以及我的應用程式呼叫的其他元件。 尤其當系統變慢時，是我的元件所造成，還是我正在接收來自其他人的慢速回應？
* 我的應用程式是否正在運作？ 全世界都可以看到它嗎？ 讓我知道它是否會停止...
* 根本原因為何？ 失敗是在我的元件中，還是相依性的問題？ 是通訊問題嗎？
* 有多少使用者受到影響？ 如果我有一個以上要解決的問題，哪一個問題最重要？

## <a name="what-is-application-insights"></a>什麼是 Application Insights？
![Application Insights 的基本工作流程](./media/app-insights-devops/020.png)

1. Application Insights 會檢測您的應用程式，並在應用程式執行時，傳送相關的遙測。 您可以在應用程式中建置 Application Insights SDK，或者可以在執行階段運用檢測。 前者是更有彈性的方法，因為您可以將自己的遙測新增到一般模組。
2. 遙測會傳送到 Application Insights 入口網站，且在其中儲存並處理它 (雖然 Application Insights 裝載於 Microsoft Azure 中，但是它可以監視任何 Web 應用程式，而不只是 Azure 應用程式)。
3. 遙測會以事件的圖表和表格形式來呈現。

遙測有兩種主要類型︰彙總與原始執行個體。 

* 例如，執行個體資料包含您的 Web 應用程式所收到的要求報告。 您可以使用 Application Insights 入口網站中的搜尋工具，尋找並檢查要求的詳細資料。 執行個體所包含的資料如下：您的應用程式要花費多少時間來回應要求，以及要求的 URL、用戶端的大約位置和其他資料。
* 彙總的資料包含每個單位時間的事件計數，因此您可以將要求速率與回應時間進行比較。 它也包含像是要求回應時間的計量資訊。

資料的主要類別如下：

* 對您應用程式的要求 (通常是 HTTP 要求)，其中含有 URL 上的資料、回應時間，以及成功或失敗。
* 相依性 - 您的應用程式所進行的 REST 和 SQL 呼叫，也包含 URI、回應時間和成功
* 例外狀況，包括堆疊追蹤。
* 頁面檢視資料，此資料來自使用者的瀏覽器。
* 計量資訊 (例如效能計數器)，以及您自行撰寫的計量。 
* 您可以用來追蹤商務事件的自訂事件
* 用來偵錯的記錄追蹤。

## <a name="case-study-real-madrid-fc"></a>案例研究：西班牙皇家馬德里隊
[皇家馬德里足球俱樂部](http://www.realmadrid.com/) 的 Web 服務可服務世界各地大約 450 百萬名球迷。 球迷可透過 Web 瀏覽器和俱樂部的行動應用程式來存取此服務。 球迷不只可預約門票，也能存取比賽結果、選手及即將舉行的比賽相關資訊和影片剪輯。 他們會使用像是得分進球數等篩選條件進行搜尋。 另外還提供社交媒體的連結。 使用者體驗已高度個人化，並設計為可讓球迷參與的雙向溝通。

此解決方案[是一個 Microsoft Azure 上應用程式和服務的系統](https://www.microsoft.com/en-us/enterprise/microsoftcloud/realmadrid.aspx)。 延展性是關鍵需求︰流量變化無常，而且可在比賽期間和接近比賽時達到非常高的量。

對於皇家馬德里而言，監視系統效能是非常重要的。 Azure Application Insights 能夠提供整個系統的完整檢視，確保提供既可靠又高等級的服務。 

俱樂部也能對球迷進行深入了解︰他們的所在位置 (只有 3% 在西班牙)、他們對於球員、歷史記錄和即將舉行的比賽感興趣的程度為何，以及他們如何回應每場比賽結果。

此遙測資料中大多數都會自動收集，而不需要新增任何程式碼，這可簡化解決方案並降低操作複雜度。  對於皇家馬德里而言，Application Insights 每個月可處理 38 億個遙測點。

皇家馬德里會使用 Power BI 模組來檢視其遙測。

![Application Insights 遙測的 Power BI 檢視](./media/app-insights-devops/080.png)

## <a name="smart-detection"></a>智慧型偵測
[主動診斷](app-insights-proactive-diagnostics.md)是一項最新功能。 您不需提供任何特殊組態，Application Insights 就會自動偵測並警示您，關於您應用程式中異常提高的失敗率。 它的智慧型功能足以忽略偶爾失敗的背景，而且也只會依要求中的增加比率按比例提高。 例如，如果您依存的其中一個服務中發生失敗，或者如果您剛部署的新組建並沒有運作得很好，則當您查看電子郵件之後立即就能了解相關資訊 (而且會提供 Webhook，讓您能夠觸發其他應用程式)。

這項功能的另一個層面會針對遙測執行每日的深入分析，以找出很難發現的異常效能模式。 例如，它會找到與特定地理區域或特定瀏覽器版本相關聯的慢速效能。

在這兩種情況下，警示不只會通知您它發現的徵兆，還會為您提供有助於診斷問題所需的資料，例如相關的例外狀況報告。

![來自主動診斷的電子郵件](./media/app-insights-devops/030.png)

客戶 Samtec 說：「在最近的功能快速轉換期間，我們發現低於調整規模的資料庫已到達它的資源限制並導致逾時。 主動偵測警示會在我們分類問題時逐字顯現，非常接近即時公告。 這個與 Azure 平台相結合的警示，幾乎可協助我們即時修正問題。 總停機時間 < 10 分鐘。」

## <a name="live-metrics-stream"></a>即時計量串流
部署最新組建是令人焦慮的體驗。 如果有任何問題，您會希望立即了解它們，讓您能夠在必要時取消部署。 即時計量串流可為您提供大約 1 秒延遲的關鍵計量。

![即時計量](./media/app-insights-devops/040.png)

並可讓您立即檢查任何失敗或例外狀況的樣本。

![即時失敗事件](./media/app-insights-devops/live-stream-failures.png)

## <a name="application-map"></a>應用程式對應
應用程式對應會自動探索您的應用程式拓撲 (在其上方放置效能資訊)，讓您能夠輕易地在整個分散式環境中識別出效能瓶頸和有問題的流程。 它可讓您探索 Azure 服務上的應用程式相依性。 您可以藉由了解問題是否與程式碼相關或與相依性相關，並從單一位置向內切入到相關診斷體驗，來將問題分類。 例如，您的應用程式可能會因為 SQL 層的效能降低而失敗。 利用應用程式對應，您可以立即看見它並鑽研 SQL 索引建議程式或 Query Insights 體驗。

![應用程式對應](./media/app-insights-devops/050.png)

## <a name="application-insights-analytics"></a>Application Insights 分析
使用 [分析](app-insights-analytics.md)，您就能利用功能強大的 SQL 式語言撰寫任意查詢。  將各種不同的觀點連結在一起時，跨整個應用程式堆疊的診斷就會變得很容易，而您可以詢問正確的問題，以便將服務效能與商務計量和客戶體驗相互關聯。 

您可以查詢您儲存在入口網站中的所有遙測執行個體和未經處理的計量資料。 語言包括篩選、聯結、彙總及其他作業。 您可以計算欄位並執行統計分析。 目前提供表格式和圖形視覺效果。

![分析查詢和結果圖表](./media/app-insights-devops/025.png)

例如，很容易：

* 依客戶層區隔應用程式的要求效能資來了解他們的體驗。
* 在即時網站調查期間，搜尋特定錯誤碼或自訂的事件名稱。
* 向下切入特定客戶的應用程式使用方式，以了解需要和採用哪些功能。
* 針對特定使用者追蹤工作階段與回應時間，讓支援和營運團隊能夠提供即時客戶支援。
* 判斷常用的應用程式功能來回答應優先考慮的功能問題。

客戶 DNN 說：「Application Insights 為我們提供了方程式中遺漏的部分，以便我們能夠視需要結合、排序、查詢及篩選資料。 讓我們的團隊可以使用他們自己的創意和體驗，利用功能強大的查詢語言來尋找資料，讓我們能夠得到相關見解，並解決我們甚至不曾知道我們發生的問題。 許多有趣的答案都是來自開頭為『我想知道如果...』的問題。」

## <a name="development-tools-integration"></a>開發工具整合
### <a name="configuring-application-insights"></a>設定 Application Insights
Visual Studio 和 Eclipse 提供工具，可為您正在開發的專案設定正確的 SDK 封裝。 有一個功能表命令可新增 Application Insights。

如果您需要使用追蹤紀錄架構 (例如 Log4N、NLog 或 System.Diagnostics.Trace)，則您可以選擇將記錄傳送到 Application Insights 以及另一個遙測，因此可輕鬆地將追蹤與要求、相依性呼叫及例外狀況相互關聯。

### <a name="search-telemetry-in-visual-studio"></a>在 Visual Studio 中搜尋遙測
開發並偵錯功能時，您可以使用 Web 入口網站中的相同搜尋功能，在 Visual Studio 中直接搜尋遙測。

此外，當 Application Insights 記錄例外狀況時，您可以在 Visual Studio 中檢視資料點，並直接跳到相關的程式碼。

![Visual Studio 搜尋](./media/app-insights-devops/060.png)

偵錯期間，您可以選擇將遙測保留於開發電腦中，在 Visual Studio 中加以檢視，但不會將它傳送到入口網站。 此本機選項可避免與生產環境的遙測偵錯混合在一起。

### <a name="build-annotations"></a>建置註解
如果您使用 Azure DevOps 來建置和部署應用程式，則入口網站中的圖表上會顯示部署註解。 如果您的最新版本會在計量上產生任何效果，則會變得很明顯。

![建置註解](./media/app-insights-devops/070.png)

### <a name="work-items"></a>工作項目
產生警示時，Application Insights 可以在您的工作追蹤系統中自動建立工作項目。

## <a name="but-what-about"></a>但是，您認為...？
* [隱私權和儲存體](app-insights-data-retention-privacy.md) - 您的遙測會保留於 Azure 安全伺服器上。
* 效能 - 影響很低。 遙測會進行批次處理。
* [價格](app-insights-pricing.md) - 您可以開始免費使用，當您處於低用量期間可繼續使用。


## <a name="video"></a>影片

> [!VIDEO https://channel9.msdn.com/events/Connect/2016/112/player]

## <a name="next-steps"></a>後續步驟
開始使用 Application Insights 很簡單。 主要選項包括：

* 檢測已在執行的 Web 應用程式。 這可為您提供所有內建的效能遙測。 其適用於 [Java](app-insights-java-live.md) 和 [IIS 伺服器](app-insights-monitor-performance-live-website-now.md)，以及 [Azure Web 應用程式](app-insights-azure.md)。
* 在開發期間檢測您的專案。 您可以針對 [ASP.NET](app-insights-asp-net.md) 或 [Java](app-insights-java-get-started.md) 應用程式，以及 [Node.js](app-insights-nodejs.md) 和許多[其他類型](app-insights-platforms.md)執行此動作。 
* 藉由新增簡短的程式碼片段來檢測 [任何網頁](app-insights-javascript.md) 。

