---
title: DevOps 適用的 Azure Application Insights 概觀 | Microsoft Docs
description: 了解如何在 Dev Ops 環境中使用 Application Insights。
author: mrbullwinkle
services: application-insights
documentationcenter: ''
manager: carmonm
ms.assetid: 6ccab5d4-34c4-4303-9d3b-a0f1b11e6651
ms.service: application-insights
ms.workload: tbd
ms.tgt_pltfrm: ibiza
ms.devlang: na
ms.custom: mvc
ms.topic: overview
ms.date: 09/06/2018
ms.author: mbullwin
ms.openlocfilehash: 799593758bf24924d91d38bd6a626b945247183b
ms.sourcegitcommit: ebd06cee3e78674ba9e6764ddc889fc5948060c4
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/07/2018
ms.locfileid: "44050233"
---
# <a name="overview-of-application-insights-for-devops"></a>DevOps 適用的 Application Insights 概觀

透過 [Application Insights](app-insights-overview.md)，您可以迅速瞭解您的應用程式在作用中時如何執行和使用。 如果發生問題，它可讓您了解、協助您評估影響，以及協助您判斷原因。

以下是某個開發 Web 應用程式的小組的敘述：

* *「幾天前，我們部署了次要的 Hotfix。我們沒有執行廣泛測試階段，但很不幸地，有些未預期的變更被合併到裝載中，造成前端與後端之間的不相容。伺服器例外狀況隨即湧現，警報發出，我們便知道了這個情況。在按幾下 Application Insights 入口網站之後，我們從例外狀況堆疊取得了足夠的資訊，而得以縮小問題。我們立即復原並將損害降至最低。Application Insights 使得這個部分的開發作業週期變得非常輕鬆且可行。」*

在本文中，我們追隨 Fabrikam Bank 中一個開發線上銀行系統 (OBS) 的團隊，了解他們如何使用 Application Insights 快速回應客戶和進行更新。  

此團隊會處理下圖所示的 DevOps 循環：

![DevOps 循環](./media/app-insights-detect-triage-diagnose/00-devcycle.png)

送入其開發待處理項目 (工作清單) 的需求。 小組努力工作，經常交付可行的軟體 - 通常是對現有的應用程式進行改進和延伸。 上線的 app 則會經常更新新功能。 雖然 app 已經上線，但小組會透過 Application Insights 的協助監視其效能和使用情況。 這項 APM 資料會餵送回其開發待處理項目。

小組使用 Application Insights 密切監視上線 Web 應用程式的下列項目：

* 效能。 他們想要了解回應時間如何隨著要求計數變化；使用多少 CPU、網路、磁碟和其他資源；哪個應用程式程式碼降低效能；以及瓶頸所在。
* 失敗。 如果有例外狀況或失敗的要求，或如果效能計數器超出其舒適範圍，小組必須快速知道以便採取動作。
* 使用狀況。 當發行新功能時，小組想要知道其使用程度，以及使用者在使用上是否有任何問題。

接下來討論循環的回饋部分：

![偵測-分級-診斷](./media/app-insights-detect-triage-diagnose/01-pipe1.png)

## <a name="detect-poor-availability"></a>偵測可用性不佳
Marcela Markova 是 OBS 小組的資深開發人員，主導線上效能監視。 她設定了數項[可用性測試](app-insights-monitor-web-app-availability.md)：

* 應用程式主要登陸頁面的單一 URL 測試 http://fabrikambank.com/onlinebanking/。 她設定 HTTP 代碼 200 與文字「歡迎使用！」的準則。 如果此測試失敗，表示網路或伺服器發生嚴重錯誤，或可能有部署問題。 (或是有人變更了頁面上的「歡迎使用！」 訊息，但沒讓她知道。)
* 更深入的多步驟測試將會記錄並取得目前帳戶清單，檢查每個頁面上的一些重要詳細資料。 此測試會驗證對帳戶資料庫的連結運作中。 她使用虛構的客戶識別碼：其中少數幾個保留供測試目的。

透過設定這些測試，Marcela 能確信若要任何中斷情況，小組將快速知道。  

失敗在 Web 測試圖表上會以紅點顯示：

![顯示對前一個期間執行的 Web 測試](./media/app-insights-detect-triage-diagnose/04-webtests.png)

但更重要的是，任何失敗的相關警示會以電子郵件方式寄送給開發小組。 以該方式，他們幾乎可在所有客戶之前便得知該情況。

## <a name="monitor-performance"></a>監視效能
在 Application Insights 中的概觀頁面上，有一個顯示各種[重要度量](app-insights-web-monitor-performance.md)的圖表。

![概觀效能 KPI 圖表的螢幕擷取畫面](./media/app-insights-detect-triage-diagnose/overview-graphs.png)

瀏覽器頁面載入時間是從網頁直接傳送的遙測所衍生。 伺服器回應時間、伺服器要求計數和失敗的要求計數，都是在 Web 伺服器中測量，然後從該處傳送到 Application Insights。

Marcela 有些擔心伺服器回應圖形。 此圖表會顯示伺服器自收到使用者瀏覽器的 HTTP 要求，直到傳回回應這段期間的平均時間。 在這個圖表中看到差異並無不尋常，因為各系統的負載不同。 但在此情況下，要求數量些微增加與回應時間大幅增加似乎有某種關係。 這可能表示系統到達運作極限。

她將「伺服器」圖表打開：

![各種度量](./media/app-insights-detect-triage-diagnose/002-servers.png)

其中似乎沒有資源限制的徵兆，也許伺服器回應圖表中的起伏只是巧合。

## <a name="set-alerts-to-meet-goals"></a>設定警示以符合目標
儘管如此，她還是會多加留意回應時間。 如果它們變得太高，她想立即知道。

因此，她針對回應時間大於一般臨界值的情況設定了一個[警示](app-insights-metrics-explorer.md)。 這可以確保當回應時間變慢時她就會知道。

![加入警示分頁](./media/app-insights-detect-triage-diagnose/07-alerts.png)

您可以在其他各種不同的度量上設定警示。 例如，您可以在例外狀況計數變高或可用記憶體變低，或用戶端要求中有尖峰時收到電子郵件。

## <a name="stay-informed-with-smart-detection-alerts"></a>獲得有關智慧型偵測警示的資訊
隔天，確實收到了一封來自 Application Insights 的電子郵件。 但是開啟郵件之後，她發現並不是她所設定的回應時間警示。 而是郵件告知她失敗的要求 (也就是傳回 500 或更高數字之失敗代碼的要求) 數目突然提高。

發生失敗的要求時使用者會看到錯誤，通常是在程式碼中擲出例外狀況之後。 也許他們會看到訊息指出 「抱歉，我們現在無法更新您的詳細資料。 」 或者，極度尷尬的是，使用者的螢幕上會顯示堆疊傾印 (出自於 Web 伺服器禮貌回應)。

這個警示令人驚訝，因為她上次查看時，失敗的要求計數很低，完全不用擔心。 其中一小部分的失敗預期是在忙碌的伺服器中。

這也讓她稍微感到驚訝，因為她之前並不需要設定這個警示。 Application Insights 包含智慧型偵測。 它會自動調整至您 app 的一般失敗模式，並且「習慣」特定頁面、高負載或和其他計量連結的失敗。 只有當增加量超出預期的量時它才會發出警示。

![主動診斷電子郵件](./media/app-insights-detect-triage-diagnose/21.png)

這封電子郵件非常有幫助。 它不只是發出警示。 它也會進行許多分級和診斷工作。

它會顯示有多少客戶，以及哪些網頁或作業受到影響。 Marcela 可以決定是否需要動員整個小組來處理此問題，或者可以延後到下週再處理。

該電子郵件也顯示發生的特定例外狀況，甚至 - 更有趣的 - 是與對特定資料庫呼叫失敗關聯的失敗。 這解釋了為何 Marcela 的團隊即使最近沒有部署任何更新也會突然發生錯誤。

Marcella 會根據這封電子郵件偵測資料庫團隊的主管。 她發現他們在過去半小時釋出了 Hot Fix；而不巧的是，或許是基礎結構有些微小變更...

因此，在問題發生後的 15 分鐘內，甚至是在檢查紀錄之前，就已經開始修正問題。 不過，Marcela 按了一下連結來開啟 Application Insights。 此時直接開啟了一個失敗的要求，而且她可以在相依性呼叫的關聯清單中看到失敗的資料庫呼叫。

![失敗的要求](./media/app-insights-detect-triage-diagnose/23.png)

## <a name="detect-exceptions"></a>偵測例外狀況
只要一點點設定，就可以將 [例外狀況](app-insights-asp-net-exceptions.md) 自動報告給 Application Insights。 也可以在程式碼中呼叫 [TrackException()](app-insights-api-custom-events-metrics.md#trackexception) ，明確擷取這些例外狀況：  

    var telemetry = new TelemetryClient();
    ...
    try
    { ...
    }
    catch (Exception ex)
    {
       // Set up some properties:
       var properties = new Dictionary <string, string>
         {{"Game", currentGame.Name}};

       var measurements = new Dictionary <string, double>
         {{"Users", currentGame.Users.Count}};

       // Send the exception telemetry:
       telemetry.TrackException(ex, properties, measurements);
    }


Fabrikam 銀行小組制定出一律在發生例外狀況時傳送遙測的作法，除非有明顯的恢復。  

事實上，其策略甚至比此更為廣泛：他們會在每個客戶對要做的事感到挫折時便傳送遙測，而不論是否與程式碼中的例外狀況對應。 例如，如果外部的銀行內傳送系統因為某此作業原因 (不是客戶的錯誤) 而傳回「無法完成此交易」訊息，他們會追蹤該事件。

    var successCode = AttemptTransfer(transferAmount, ...);
    if (successCode < 0)
    {
       var properties = new Dictionary <string, string>
            {{ "Code", returnCode, ... }};
       var measurements = new Dictionary <string, double>
         {{"Value", transferAmount}};
       telemetry.TrackEvent("transfer failed", properties, measurements);
    }

TrackException 用來報告例外狀況，因為它會傳送堆疊的副本。 TrackEvent 用來報告其他事件。 您可以在診斷中附加可能有幫助的任何屬性。

例外狀況和事件會顯示在 [[診斷搜尋](app-insights-diagnostic-search.md)] 刀鋒視窗中。 您可以深入探索，以查看額外屬性和堆疊追蹤。

![在「診斷搜尋」中，請使用篩選器來顯示特定類型的資料](./media/app-insights-detect-triage-diagnose/appinsights-333facets.png)


## <a name="monitor-proactively"></a>主動監視
Marcela 不會無所事事等候警示。 在每次重新部署之後，她都會立即查看[回應時間](app-insights-web-monitor-performance.md) - 除了例外狀況計數之外，也查看整體數據和最緩慢的要求表。  

![回應時間圖及伺服器回應時間格線。](./media/app-insights-detect-triage-diagnose/response-time.png)

她可以評估每個部署的效能影響，通常是將每週與前一週比較。 如果突然有變慢的情況，她會將該情況向相關的開發人員反應。

## <a name="triage-issues"></a>分級問題
分級 - 評估問題的嚴重性和程度 - 是偵測後的第一個步驟。 我們是否應該在半夜打電話給小組？ 或是問題可以留到累積的工作中下一次方便的時候？ 分級時有一些重要的問題。

發生頻率為何？ [概觀] 分頁的圖表可提供問題的某些觀點。 例如，Fabrikam 應用程式在一個晚上產生了四個 Web 測試警示。 小組在早上查看圖表，可以發現確實出現一些紅點，但多數的測試仍是綠色。 深入探索可用性圖表，很明顯地，這所有的間歇性問題都來自一個測試位置。 這顯然是僅影響一個路徑的網路問題，而且很可能可自行解決。  

相反地，例外狀況計數或回應時間圖表中明顯且穩定的上升則明顯有其他問題需要注意。

實用的分級策略是「自己動手做」。 如果您遇到相同問題，就會知道它是真的。

哪個部分的使用者受到影響？ 若要獲得約略的答案，請將失敗率除以工作階段計數。

在緩慢回應的情況中，將回應最緩慢的要求表與每個頁面的使用頻率相比較。

封鎖案例的重要性如何？ 如果這是功能性問題，封鎖了特定的使用者劇本，有很大影響嗎？ 如果客戶無法支付帳單，便很嚴重；如果客戶無法變更其畫面色彩喜好設定，也可以稍候再解決。 事件或例外狀況的詳細資料或緩慢頁面的身分識別，會告知您客戶發生問題的位置。

## <a name="diagnose-issues"></a>診斷問題
診斷與偵測不太一樣。 開始追蹤程式碼之前，您應該對問題的原因、位置和發生時機有約略的構念。

**發生時機為何？** 事件和度量圖表提供的歷程檢視可讓您輕鬆將影響與可能原因產生相互關聯。 如果回應時間或例外狀況率中有間歇性的尖峰，請查看要求計數：如果尖峰是在相同時間，則可能是資源問題。 您需要指派更多 CPU 或記憶體嗎？ 或者它是無法管理負載的相依性？

**問題在於我們嗎？**  如果特定類型的要求效能突然下降 - 例如客戶想要對帳單時 - 則可能是外部子系統而非 Web 應用程式有問題。 在「計量瀏覽器」中，選取相依性失敗率和相依性期間率，並將其過去幾個小時或幾天的歷程記錄與您偵測到的問題比較。 如果有相互關聯的變更，則外部子系統可能是原因所在。  


![相依性失敗和對相依性呼叫期間的圖表](./media/app-insights-detect-triage-diagnose/11-dependencies.png)

有些緩慢相依性的問題是地理位置問題。 Fabrikam 銀行使用 Azure virtual 機器，並發現他們誤將 Web 伺服器和帳戶伺服器放置在不同國家/地區。 透過移轉其中一部伺服器獲得明顯的改善。

**我們做了什麼？** 如果問題似乎不在於相依性，而且如果不是持續有問題，則可能是由於最近的變更而導致。 度量和事件圖表提供的歷史觀點讓您輕鬆將任何突然的變更與部署產生相互關聯。 它可縮小問題的搜尋範圍。 若要識別應用程式程式碼中哪些行讓效能變慢，請啟用 Application Insights Profiler。 請參閱[使用 Application Insights 來即時分析 Azure Web Apps](./app-insights-profiler.md)。 啟用分析工具之後，您會看到類似下列的追蹤。 在此範例中，很容易就能注意到是方法 *GetStorageTableData* 造成問題。  

![App Insights Profiler 追蹤](./media/app-insights-detect-triage-diagnose/AppInsightsProfiler.png)

**發生什麼？** 有些問題很少發生，而且透過離線測試可能難以追蹤。 我們能做的是嘗試在問題發生時擷取錯誤。 您可以在例外狀況報告中檢查堆疊傾印。 此外，您可以使用喜好的記錄架構或使用 TrackTrace() 或 TrackEvent() 來編寫追蹤呼叫。  

Fabrikam 的帳戶間轉送發生間歇性問題，但只有某些帳戶類型有此情況。 為了更加了解發生的情況，他們在程式碼中的重要點插入了 TrackTrace() 呼叫，附加帳戶類型作為每個呼叫的內容。 那使得要在診斷搜尋中僅篩選掉這些追蹤更為輕鬆。 他們也將參數值附加為追蹤呼叫的屬性和測量。

## <a name="respond-to-discovered-issues"></a>回應所發現的問題
診斷問題之後，您可以製訂修正問題的計劃。 也許您需要復原最近的變更，或也許您可以繼續並修正它。 修正完成後，Application Insights 會告知您是否成功。  

Fabrikam 銀行的開發小組對效能測量採取較使用 Application Insights 之前更具結構的方法。

* 他們會在 Application Insights 概觀頁面就特定度量設定效能目標。
* 他們從頭為應用程式設計效能度量，例如透過「漏斗」測量使用者進度的度量。  

## <a name="monitor-user-activity"></a>監視使用者活動
當回應時間一直都不錯，而且例外狀況不多時，開發小組可以繼續往可用性的方向努力。 他們可以思考如何改善使用者體驗，以及如何鼓勵更多使用者達到想要的目標。

Application Insights 也可以用來了解使用者在應用程式內執行的動作。 執行順暢時，小組會想要得知哪些功能最受歡迎、使用者喜歡或感到有困難的部份，以及使用者回來的頻率。 這些資訊有助於將他們近期的工作排定優先順序。 而他們可以計劃測量每個功能的成功度，作為開發週期的一部份。

例如，使用者在網站上的典型使用者旅程是明確的「漏斗圖」。 許多客戶會研究不同類型的貸款利率。 少部分的客戶會繼續填寫報價單。 在取得報價單的客戶當中，有一部分會繼續，並取得貸款。

![頁面檢視計數](./media/app-insights-detect-triage-diagnose/funnel.png)

透過找出最多客戶放棄的位置，企業可以思考如何讓更多使用者通過漏斗。 在某些情況下，可能是使用者體驗 (UX) 失敗 - 例如，很難找到 [下一步] 按鈕，或者指示不太明顯。 更有可能是因為重要的商業理由放棄：可能是貸款利率太高。

無論任何原因，資料都可協助團隊了解使用者在做什麼。 此時可以插入更多追蹤呼叫，了解更多細節。 TrackEvent() 可以用來計算任何使用者動作，小至個別的按鈕點擊，大到如付清貸款等重要成果。

團隊一直都有使用者活動的相關資訊。 只是現在，每當設計出一個新功能時，就要思考如何取得其使用方式的意見反應。 團隊如果從一開始就在功能中設計追蹤呼叫， 就可以使用意見反應在每個開發週期中改進功能。

[深入了解如何追蹤使用量](app-insights-usage-overview.md)。

## <a name="apply-the-devops-cycle"></a>套用 DevOps 週期
以上就是一個小組如何使用 Application Insights 來不只是修正個別問題，還改善其開發週期的情況。 希望這已提供您一些概念，讓您了解 Application Insights 如何協助您在自己的應用程式中進行應用程式效能管理。

## <a name="video"></a>影片

> [!VIDEO https://channel9.msdn.com/events/Connect/2016/112/player]

## <a name="next-steps"></a>後續步驟
視您的應用程式特性而定，您可以從數種方式著手。 請挑選最適合您的方式：

* [ASP.NET Web 應用程式](app-insights-asp-net.md)
* [Java Web 應用程式](app-insights-java-get-started.md)
* [Node.js Web 應用程式](app-insights-nodejs.md)
* 裝載於 [IIS](app-insights-monitor-web-app-availability.md)、[J2EE](app-insights-java-live.md) 或 [Azure](app-insights-azure.md) 上的已部署應用程式。
* [網頁](app-insights-javascript.md) - 單頁應用程式或一般網頁 - 單獨使用此選項或作為任何伺服器選項以外的附加選項。
* 可從公用網際網路測試您應用程式的[可用性測試](app-insights-monitor-web-app-availability.md)。
