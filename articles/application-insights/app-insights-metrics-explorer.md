---
title: 在 Azure Application Insights 中探索計量 | Microsoft Docs
description: 如何解譯計量瀏覽器上的圖表，以及如何自訂計量瀏覽器刀鋒視窗。
services: application-insights
documentationcenter: ''
author: mrbullwinkle
manager: carmonm
ms.assetid: 1f471176-38f3-40b3-bc6d-3f47d0cbaaa2
ms.service: application-insights
ms.workload: tbd
ms.tgt_pltfrm: ibiza
ms.devlang: na
ms.topic: conceptual
ms.date: 03/08/2017
ms.author: mbullwin
ms.openlocfilehash: 66ef3330a435574405dbfb8b8c82d984bb877efe
ms.sourcegitcommit: e8f443ac09eaa6ef1d56a60cd6ac7d351d9271b9
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/12/2018
ms.locfileid: "35633453"
---
# <a name="exploring-metrics-in-application-insights"></a>在 Application Insights 中探索度量
[Application Insights][start] 中的度量為從您的應用程式傳送的遙測中的測量值和事件計數。 它們幫助您偵測效能問題，並觀察使用應用程式方式的趨勢。 標準度量的範圍很廣泛，而您也可以建立自己的自訂度量和事件。

度量和事件計數會顯示在彙總值 (例如總和、平均或計數) 的圖表中。

以下是一組範例圖表︰

![](./media/app-insights-metrics-explorer/01-overview.png)

您可在 Application Insights 入口網站中找到度量圖表。 在大部分情況下，您可以自訂它們，而且您可以將更多圖表新增至刀鋒視窗。 從 [概觀] 刀鋒視窗中，按一下以取得更詳細的圖表 (有「伺服器」之類的標題)，或按一下 [計量瀏覽器]，以開啟新的刀鋒視窗讓您建立自訂圖表。

## <a name="time-range"></a>時間範圍
您可以在任何刀鋒視窗上變更圖表或格線涵蓋的時間範圍。

![開啟 Azure 入口網站中應用程式的 [概觀] 刀鋒視窗](./media/app-insights-metrics-explorer/03-range.png)

如果您預期的部分資料尚未出現，請按一下 [重新整理]。 圖表本身會定期重新整理，但是時間範圍越大，間隔時間會越長。 資料從分析管線往圖表上顯示可能需要一些時間。

若要放大圖表的局部，請拖曳過該部分︰

![拖曳過圖表的一部分。](./media/app-insights-metrics-explorer/12-drag.png)

按一下 [復原縮放] 按鈕可將它還原。

## <a name="granularity-and-point-values"></a>資料粒度和點值
將滑鼠移至圖表上可顯示該點度量的值。

![將滑鼠移至圖表上](./media/app-insights-metrics-explorer/02-focus.png)

特定點的度量值會繼著前一個取樣間隔而彙總。

取樣間隔或「資料粒度」會顯示在刀鋒視窗的頂端。

![刀鋒視窗的標題。](./media/app-insights-metrics-explorer/11-grain.png)

您可以在時間範圍刀鋒視窗中調整資料粒度：

![刀鋒視窗的標題。](./media/app-insights-metrics-explorer/grain.png)

可提供的資料粒度取決於您選取的時間範圍。 明確的資料粒度可替代時間範圍內的「自動」資料粒度。


## <a name="editing-charts-and-grids"></a>編輯圖表和格線
若要對刀鋒視窗加入新圖表：

![在 [計量瀏覽器] 中，選擇 [加入圖表]](./media/app-insights-metrics-explorer/04-add.png)

選取現有或新圖表上的 [編輯]  來編輯其顯示的內容：

![選取一或多個度量](./media/app-insights-metrics-explorer/08-select.png)

您可以在圖表上顯示一個以上的度量，不過，可以一起顯示的組合有一些限制。 只要您選取一個度量，便會停用某些其他度量。

如果您編寫[自訂度量][track]到您的應用程式 (對 TrackMetric 和 TrackEvent 呼叫)，會在這裡予以列出。

## <a name="segment-your-data"></a>分段資料
您可以依照屬性分割度量 - 例如，比較使用不同作業系統的用戶端上的頁面檢視。

選取圖表或格線，將分組切換成開啟，並選取要分組依據的屬性：

![選取 [分組開啟]，然後在 [群組依據] 中選取屬性](./media/app-insights-metrics-explorer/15-segment.png)

> [!NOTE]
> 當您使用群組時，[區域] 及 [橫條圖] 類型會提供堆疊的顯示。 這適合於 [彙總] 方法是 [加總] 的情況。 但如果彙總類型是 [平均]，則選擇 [線條] 或[格線] 顯示類型。
>
>

如果您將自訂度量編寫至您的應用程式，並且它們包含[屬性值][track]，將可以在清單中選取屬性。

圖表是否對分段的資料來說太小？ 調整其高度：

![調整滑桿](./media/app-insights-metrics-explorer/18-height.png)

## <a name="aggregation-types"></a>彙總類型
旁邊的圖例通常預設會顯示圖表在這段期間的彙總值。 如果您將滑鼠停留在圖表上，它會顯示在該點的值。

圖表上的每個資料點是在先前取樣間隔或「資料粒度」中所收到的資料值彙總。 資料粒度會顯示在刀鋒視窗頂端，並隨著圖表的時幅而有所不同。

度量可以用不同方式彙總：

* **計數**是在取樣間隔中接收到的事件計數。 它用於事件，例如要求。 圖表的高度變化指出事件發生的速率變化。 但請注意，數字的值會隨變更取樣間隔而變更。
* **總和** ：將取樣間隔或圖表期間收到的所有資料點的值相加。
* **平均** ：將總和除以間隔期間收到的資料點數目。
* **唯一** ：計數會用於使用者及帳戶的計數。 在取樣間隔或圖表期間，圖形顯示在該時間看到的不同使用者的計數。
* **%** - 每個彙總百分比版本只會搭配分段圖表使用。 總計永遠會加總至 100%，圖表會顯示總計的不同元件之相對貢獻。

    ![百分比彙總](./media/app-insights-metrics-explorer/percentage-aggregation.png)

### <a name="change-the-aggregation-type"></a>變更彙總類型

![編輯圖表，然後選取彙總](./media/app-insights-metrics-explorer/05-aggregation.png)

當您建立新的圖表或所有度量皆已取消選取時，會顯示每個度量的預設方法：

![取消選取所有度量以查看預設值](./media/app-insights-metrics-explorer/06-total.png)

## <a name="pin-y-axis"></a>釘選 Y 軸 
根據預設，圖表會顯示資料範圍中從零到最大值的 Y 軸值，以提供值配量的視覺表示法。 但是在某些超過配量的情況下，以視覺化方式檢查值的些微變化可能有興趣。 對於類似這種的自訂，請使用 Y 軸範圍編輯功能，將 Y 軸的最小值或最大值釘選在所要的位置。
按一下 [進階設定] 核取方塊以顯示 [Y 軸範圍] 設定

![按一下 [進階設定]、選取 [自訂範圍]，然後指定最小值與最大值](./media/app-insights-metrics-explorer/y-axis-range.png)

## <a name="filter-your-data"></a>篩選資料
若只要查看選取的一組屬性值的度量：

![按一下 [篩選器]，然後檢查一些值](./media/app-insights-metrics-explorer/19-filter.png)

如果您不為特定屬性選取任何值，這與將它們全選相同：該屬性上沒有篩選器。

請注意每個屬性值旁的事件計數。 選取一個屬性的值時，會調整計數與其他屬性值。

篩選會套用至刀鋒視窗上的所有圖表。 如果您要將不同的篩選套用到不同的圖表，請建立並儲存不同的計量刀鋒視窗。 如果想要，您可以將不同刀鋒視窗中的圖表釘選到儀表板，以便並排查看兩者。

### <a name="remove-bot-and-web-test-traffic"></a>移除 Bot 和 Web 測試流量
使用篩選器**真實或綜合流量**並勾選**真實**。

您也可以依 **綜合流量的來源**篩選。

### <a name="to-add-properties-to-the-filter-list"></a>將屬性加入篩選器清單
您要根據自己選擇的類別篩選遙測嗎？ 例如，您可能將使用者劃分成不同的類別，且您想要依照這些類別來分割資料。

[建立您自己的屬性](app-insights-api-custom-events-metrics.md#properties)。 在 [遙測初始設定式](app-insights-api-custom-events-metrics.md#defaults) 中設定，以使其顯示在所有遙測中 - 包括不同 SDK 模組所傳送的標準遙測。

## <a name="edit-the-chart-type"></a>編輯圖表類型
請注意您可以在格線與圖形之間切換：

![選取格線或圖表，然後選擇圖表類型](./media/app-insights-metrics-explorer/16-chart-grid.png)

## <a name="save-your-metrics-blade"></a>儲存您的度量刀鋒視窗
建立一些圖表後，請將它們儲存為我的最愛。 如果您使用組織帳戶，可以選擇是否要將它與他小組成員分享。

![選擇 [我的最愛]](./media/app-insights-metrics-explorer/21-favorite-save.png)

若要再次查看分頁，請 **前往 [概觀] 分頁** ，並開啟 [我的最愛]：

![在 [概觀] 刀鋒視窗中，選擇 [我的最愛]](./media/app-insights-metrics-explorer/22-favorite-get.png)

如果儲存時選擇「相對」時間範圍，會以最新度量更新刀鋒視窗。 如果選擇「絕對」時間範圍，它會每次都顯示相同資料。

## <a name="reset-the-blade"></a>重設刀鋒視窗
如果您編輯刀鋒視窗，但之後想要回到原始儲存的集合，只需要按一下 [重設]。

![在 [計量瀏覽器] 上方的按鈕中](./media/app-insights-metrics-explorer/17-reset.png)

## <a name="live-metrics-stream"></a>即時計量串流

如需更加立即的遙測檢視，請開啟[即時串流](app-insights-live-stream.md)。 因為彙總程序，大部分的計量需要數分鐘才會出現。 相較之下，即時計量已針對低延遲進行最佳化。 

## <a name="set-alerts"></a>設定警示
若要在任何度量有不尋常的值時收到電子郵件通知，請加入警示。 您可以選擇將電子郵件傳送給帳戶管理員，或傳送給特定的電子郵件地址。

![在 [計量瀏覽器] 中，選擇 [警示規則]，然後選擇 [加入警示]](./media/app-insights-metrics-explorer/appinsights-413setMetricAlert.png)

[深入了解警示][alerts]。


## <a name="continuous-export"></a>連續匯出
如果您想要連續匯出資料，讓您能夠在外部加以處理，請考慮使用 [連續匯出](app-insights-export-telemetry.md)。

### <a name="power-bi"></a>Power BI
如果您希望更完整地檢視您的資料，您可以 [匯出至 Power BI](http://blogs.msdn.com/b/powerbi/archive/2015/11/04/explore-your-application-insights-data-with-power-bi.aspx)。

## <a name="analytics"></a>分析
[分析](app-insights-analytics.md) 是使用強大的查詢語言來分析遙測的更靈活方式。 如果您想要合併或計算計量的結果，或執行您應用程式近期效能的深入探索，請使用它。 

從計量圖表中，您可以按一下 [分析] 圖示，直接前往對應的分析查詢。

## <a name="troubleshooting"></a>疑難排解
*我看不到我的圖表上的任何資料。*

* 篩選會套用至刀鋒視窗上的所有圖表。 請確定，當您將焦點放在某個圖表時，您未在其他圖表上設定會排除所有資料的篩選。

    如果您想要在不同的圖表上設定不同的篩選，請在不同的刀鋒視窗中建立圖表，將它們儲存為個別的最愛圖表。 如果想要，您可以將這些圖表釘選到儀表板，以便並排查看兩者。
* 如果您依據計量上未定義的屬性將圖表分組，則圖表上不會有任何資料。 請嘗試清除 [分組依據]，或選擇不同的群組屬性。
* 效能資料 (CPU、IO 速率等等) 適用於 Java Web 服務、Windows 傳統型應用程式、[IIS Web 應用程式和服務 (若您安裝狀態監視器)](app-insights-monitor-performance-live-website-now.md) 和 [Azure 雲端服務](app-insights-azure.md)。 它不適用於 Azure 網站。

## <a name="video"></a>影片

> [!VIDEO https://channel9.msdn.com/events/Connect/2016/112/player]

## <a name="next-steps"></a>後續步驟
* [使用 Application Insights 監視使用情況](app-insights-web-track-usage.md)
* [使用診斷搜尋](app-insights-diagnostic-search.md)

<!--Link references-->

[alerts]: app-insights-alerts.md
[start]: app-insights-overview.md
[track]: app-insights-api-custom-events-metrics.md
