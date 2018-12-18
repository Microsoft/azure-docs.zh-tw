---
title: 智慧型偵測 - 效能異常 | Microsoft Docs
description: Application Insights 會執行您應用程式遙測的智慧型分析，並且警告您有潛在的問題。 這項功能不需要進行任何設定。
services: application-insights
documentationcenter: windows
author: mrbullwinkle
manager: carmonm
ms.assetid: 6acd41b9-fbf0-45b8-b83b-117e19062dd2
ms.service: application-insights
ms.workload: tbd
ms.tgt_pltfrm: ibiza
ms.devlang: na
ms.topic: conceptual
ms.date: 05/04/2017
ms.reviewer: antonfr
ms.author: mbullwin
ms.openlocfilehash: 8c34696f81e40b4db13e6868f6bdaa1b2a8e7a2a
ms.sourcegitcommit: cc4fdd6f0f12b44c244abc7f6bc4b181a2d05302
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/25/2018
ms.locfileid: "47095458"
---
# <a name="smart-detection---performance-anomalies"></a>智慧型偵測 - 效能異常

[Application Insights](app-insights-overview.md) 會自動分析 Web 應用程式的效能，並且可以警告您有關潛在的問題。 您會讀取到此訊息，可能是因為您收到一個我們的智慧型偵測通知。

除了設定您 Application Insights 的應用程式 (在 [ASP.NET](app-insights-asp-net.md) 上、[Java](app-insights-java-get-started.md) 或 [Node.js](app-insights-nodejs.md)，以及在[網頁程式碼](app-insights-javascript.md)中) 以外，這項功能不需要特殊設定。 當您的應用程式產生足夠的遙測時，就會是在作用中。

## <a name="when-would-i-get-a-smart-detection-notification"></a>何時會取得智慧型偵測通知？

Application Insights 偵測到您的應用程式以下列其中一種方式降低效能︰

* **回應時間降低** - 您的應用程式回應要求速度已開始較以往更慢。 變更可能很快速，例如因為您的最新部署中有一個迴歸。 或者它可能是逐漸進行的，可能是由於記憶體流失所造成。 
* **相依性持續時間降低** - 您的應用程式會呼叫 REST API、資料庫或其他相依性。 相依性回應較以往更慢。
* **效能變慢模式** - 您應用程式出現的效能問題只會影響部分要求。 例如，頁面在某一類型瀏覽器上的載入速度低於其他類型瀏覽器；或是某一部特定伺服器服務要求的速度較慢。 目前，我們的演算法會查看頁面載入時間、要求回應時間，和相依性回應時間。  

智慧型偵測需要可用磁碟區中至少 8 天的遙測，才能建立一般效能的基準。 因此，您的應用程式在這段時間內執行之後，任何嚴重的問題都會發出通知。


## <a name="does-my-app-definitely-have-a-problem"></a>我的應用程式絕對有問題嗎？

否，通知並不表示您的應用程式一定有問題。 這只是一個建議，您可以仔細探究其中的問題。

## <a name="how-do-i-fix-it"></a>如何修正問題？

通知會包含診斷資訊。 以下是範例：


![以下是伺服器回應時間降低偵測的範例](./media/app-insights-proactive-diagnostics/server_response_time_degradation.png)

1. **分級**。 通知會顯示受影響的使用者人數或作業數。 這可協助您將優先順序指派給此問題。
2. **範圍**。 此問題是否會影響所有流量，還是只會影響某些頁面？ 它是否限制為特定的瀏覽器或位置？ 可以從通知取得這項資訊。
3. **診斷**。 通常，通知中的診斷資訊會建議問題的本質。 例如，如果要求率很高時回應時間變慢，表示您的伺服器或相依性已超載。 

    否則，在 Application Insights 中開啟 [效能] 刀鋒視窗。 您可以在此處找到[分析工具](app-insights-profiler.md)資料。 如果擲回例外狀況，則您也可以嘗試[快照集偵錯工具](app-insights-snapshot-debugger.md)。



## <a name="configure-email-notifications"></a>設定電子郵件通知

依預設會啟用智慧型偵測通知，並將這些通知傳送給具有 [Application Insights 資源的擁有者、參與者和讀取者存取權](app-insights-resources-roles-access-control.md)的人員。 若要變更這種情況，請按一下電子郵件通知中的 [設定]，或開啟 Application Insights 中的 [智慧型偵測] 設定。 
  
  ![智慧型偵測設定](./media/app-insights-proactive-diagnostics/smart_detection_configuration.png)
  
  * 您可以使用智慧型偵測電子郵件中的 [取消訂閱] 連結，停止接收電子郵件通知。

每個 Application Insights 資源每天僅限一個關於智慧型偵測效能異常的電子郵件。 只有在當天偵測到至少一個新的問題時，才會傳送電子郵件。 您不會重複收到任何訊息。 

## <a name="faq"></a>常見問題集

* *Microsoft 人員會看到我的資料嗎？*
  * 否。 服務完全是自動的。 只有您會收到通知。 您的資料是 [不公開的](app-insights-data-retention-privacy.md)。
* *你們會分析 Application Insights 收集的所有資料嗎？*
  * 目前尚未。 我們目前會分析要求回應時間、相依性回應時間和頁面載入時間。 我們後續的未來展望中將有其他計量的分析。

* 這適用於哪些類型的應用程式？
  * 會在產生適當遙測的任何應用程式中偵測到這些降低。 如果您在 Web 應用程式中安裝了 Application Insights，就會自動追蹤要求及相依性。 但在後端服務或其他應用程式中，如果您將呼叫插入 [TrackRequest()](app-insights-api-custom-events-metrics.md#trackrequest) 或 [TrackDependency](app-insights-api-custom-events-metrics.md#trackdependency)，則智慧型偵測會以相同的方式運作。

* 我可以建立自己的異常偵測規則或自訂現有的規則嗎？

  * 還不行，但是您可以︰
    * [設定警示](app-insights-alerts.md)，使其在計量超出臨界值時通知您。
    * [將遙測匯出](app-insights-export-telemetry.md)至[資料庫](app-insights-code-sample-export-sql-stream-analytics.md)或[至 PowerBI](app-insights-export-power-bi.md)，以自行分析它。
* *執行分析的頻率為何？*

  * 我們每天都會根據前一天的遙測執行分析 (UTC 時區中全天)。
* *那麼，這可以取代[計量警示](app-insights-alerts.md)嗎？*
  * 否。  我們不保證能偵測到您可能認為異常的每項行為。


* 如果我完全不回應通知，是否會收到提醒？
  * 不會，每個問題您只會收到一次訊息。 如果問題仍然存在，就會在 [智慧型偵測摘要] 刀鋒視窗中進行更新。
* *我遺失了電子郵件。在入口網站中哪裡可以找到通知？*
  * 在應用程式的 Application Insights 概觀中，按一下 [智慧型偵測] 磚。 您可以在這裡找到最多 90 天前的所有通知。

## <a name="how-can-i-improve-performance"></a>如何改善效能？
您可從自己的經驗得知，對網站使用者而言，回應緩慢和失敗是最大挫折之一。 因此，請務必解決問題。

### <a name="triage"></a>分級
首先，這很重要嗎？ 如果頁面的載入速度一直很慢，但是只有 1% 的網站台使用者必須查看該網頁，您或許有更重要的事項需要考慮。 另一方面，如果只有 1% 的使用者開啟該網頁，但它每次都擲回例外狀況，這可能就是值得調查的問題。

使用影響敘述 (受影響的使用者或流量的 %) 作為一般指南，但請留意該敘述並不是全部的詳情。 蒐集其他證據進行確認。

請考慮這個問題的參數。 如果與地理位置有關，請設定包括該地區的 [可用性測試](app-insights-monitor-web-app-availability.md) ：該地區可能只有網路問題。

### <a name="diagnose-slow-page-loads"></a>診斷頁面載入緩慢
問題出在哪裡？ 伺服器是否回應太慢、頁面是否很長，或瀏覽器必須執行很多工作才能顯示頁面？

開啟 [瀏覽器] 計量刀鋒視窗。 分段顯示的瀏覽器頁面載入時間可顯示時間的進度。 

* 如果 [傳送要求時間]  太久，不是伺服器回應速度緩慢，就是要求是含有大量資料的文章。 查看 [效能計量](app-insights-web-monitor-performance.md#metrics) 以調查回應時間。
* 設定 [相依性追蹤](app-insights-asp-net-dependencies.md) 以查看速度慢是否是外部服務或您的資料庫所造成。
* 如果 [接收回應]  是主導因素，您的頁面和其相依組件 (JavaScript、CSS 及影像等，而非以非同步方式載入的資料) 會很長。 設定 [可用性測試](app-insights-monitor-web-app-availability.md)，而且務必設定載入相依組件的選項。 當您取得一些結果時，請開啟結果的詳細資料並將它展開，以查看不同檔案的載入時間。
* [用戶端處理時間]  過長表示指令碼執行速度很慢。 如果原因不明顯，請考慮加入一些時間計時程式碼並在 trackMetric 呼叫中傳送時間。

### <a name="improve-slow-pages"></a>改善慢速網頁
Web 上有改善您的伺服器回應和頁面載入時間的完整建議，因此我們不會嘗試這次重複說明。 以下是您可能已知道的一些祕訣，這只是為提醒您：

* 由大型檔案造成的緩慢載入：以非同步方式載入指令碼和其他組件。 使用指令碼統合。 將主頁面分成可個別載入其資料的 Widget。 不要對長資料表傳送純舊式 HTML：使用指令碼要求 JSON 或其他壓縮格式的資料，然後就地填滿資料表。 有一些絕佳的架構可協助進行這一切。 (當然，也必須承擔大型指令碼)。
* 降低伺服器相依性：考慮您的元件的地理位置。 比方說，如果您使用 Azure，請確定 Web 伺服器和資料庫位於相同的區域中。 查詢是否會擷取超過所需的資訊？ 快取或批次處理是否有所幫助？
* 容量問題：查看回應時間和要求計數的伺服器計量。 如果回應時間尖峰與要求計數尖峰不成比例，有可能是您的伺服器已被過度使用。


## <a name="server-response-time-degradation"></a>伺服器回應時間降低

回應時間降低通知會告訴您︰

* 相較於這項作業一般回應時間的回應時間。
* 受影響的使用者人數。
* 偵測到當天和 7 天前這項作業的平均回應時間和第 90 百分位回應時間。 
* 偵測到當天和 7 天前這項作業的要求計數。
* 這項作業之降低與相關相依性之降低間的相互關聯。 
* 連結可幫助您診斷問題。
  * 分析工具追蹤可協助您檢視作業花費時間之處 (如果在偵測期間收集到這項作業的分析工具追蹤範例，則可使用連結)。 
  * 您可以在計量瀏覽器中的效能報告將這項作業的時間範圍/篩選條件進行交叉分析。
  * 搜尋此呼叫可檢視特定的呼叫屬性。
  * 報告失敗 - 如果計數 > 1，表示這項作業中的失敗可能造成效能降低。

## <a name="dependency-duration-degradation"></a>相依性持續時間降低

現代化應用程式採用越來越多微服務設計的方法，在許多情況下，會造成外部服務的重度可靠性。 例如，如果您的應用程式需仰賴某些資料平台，或即使您建立自己的 Bot 服務，可能還是需要一些認知服務提供者，才能讓您的 Bot 以更人性化的方式進行互動，且 Bot 也需要一些可提取回答的資料存放服務。  

範例相依性降低通知︰

![以下是相依性持續時間降低偵測的範例](./media/app-insights-proactive-diagnostics/dependency_duration_degradation.png)

請注意，它會告訴您︰

* 相較於這項作業一般回應時間的持續時間
* 受影響的使用者人數
* 偵測到當天和 7 天前此相依性的平均持續時間和第 90 百分位持續時間
* 偵測到當天和 7 天前的相依性呼叫數目
* 連結可幫助您診斷問題
  * 計量瀏覽器中此相依性的效能報告
  * 搜尋此相依性呼叫可檢視呼叫屬性
  * 報告失敗 - 如果計數 > 1，表示偵測期間失敗的相依性呼叫可能造成持續時間降低。 
  * 使用計算此相依性持續時間和計數的查詢來開啟分析  

## <a name="smart-detection-of-slow-performing-patterns"></a>效能變慢模式的智慧型偵測 

Application Insights 會尋找可能只會影響某部分使用者，或只在某些情況下影響使用者的效能問題。 例如，通知關於頁面在某一類型瀏覽器上的載入速度低於其他類型瀏覽器，或特定伺服器服務要求的速度較慢。 它同時可以探索與屬性組合相關的問題，例如在某地區中使用特定作業系統的用戶端，頁面載入速度緩慢的問題。  

這類異常狀況很難只藉由調查資料來偵測，但比您想像的更為常見。 通常只在您的客戶抱怨時才會浮出檯面。 但那時就太晚了：受影響的使用者已經轉而選擇您的對手！

目前，我們的演算法會查看頁面載入時間、伺服器上的要求回應時間，和相依性回應時間。  

您不需設定任何臨界值或設定規則。 機器學習服務和資料採礦演算法會用來偵測異常模式。

![按一下電子郵件警示中的連結，可在 Azure 中開啟診斷報告](./media/app-insights-proactive-performance-diagnostics/03.png)

* **時間**顯示偵測到問題的時間。
* [對象] 說明：

  * 偵測到的問題；
  * 我們發現的事件集的特性顯示了問題行為。
* 表格會比較效能差的事件集和所有其他事件的平均行為。

按下連結以開啟 [計量瀏覽器]，搜尋相關報告、篩選緩慢執行的事件集的時間和屬性。

修改時間範圍和篩選器可探索遙測。

## <a name="next-steps"></a>後續步驟
這些診斷工具可協助您檢查來自您的應用程式的遙測︰

* [分析工具](app-insights-profiler.md) 
* [快照集偵錯工具](app-insights-snapshot-debugger.md)
* [分析](app-insights-analytics-tour.md)
* [分析智慧型診斷](app-insights-analytics-diagnostics.md)

智慧型偵測是完全自動的。 但是，或許您會想要再設定一些警示？

* [手動設定的度量警示](app-insights-alerts.md)
* [可用性 Web 測試](app-insights-monitor-web-app-availability.md)
