---
title: Azure Application Insights 中的儀表板與導覽 | Microsoft Docs
description: 建立重要 APM 圖表和查詢的檢視。
services: application-insights
documentationcenter: ''
author: mrbullwinkle
manager: carmonm
ms.assetid: 39b0701b-2fec-4683-842a-8a19424f67bd
ms.service: application-insights
ms.workload: tbd
ms.tgt_pltfrm: ibiza
ms.devlang: multiple
ms.topic: conceptual
ms.date: 03/14/2017
ms.author: mbullwin
ms.openlocfilehash: 8161fda80b5fa498f9321371c9ad2c8a2d97441a
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46962414"
---
# <a name="navigation-and-dashboards-in-the-application-insights-portal"></a>Application Insights 入口網站中的導覽與儀表板
[在您的專案上設定 Application Insights](app-insights-overview.md) 之後，有關應用程式效能和使用情況的遙測資料會出現在 [Azure 入口網站](https://portal.azure.com)之專案的 Application Insights 資源中。

## <a name="find-your-telemetry"></a>尋找遙測
登入 [Azure 入口網站](https://portal.azure.com)，並瀏覽至您為應用程式建立的 Application Insights 資源。

![按一下 [瀏覽]，選取 [Application Insights]，然後選取您的應用程式。](./media/app-insights-dashboards/00-start.png)

應用程式的 [概觀] 刀鋒視窗 (頁面) 會顯示您應用程式的關鍵診斷資料，且為入口網站其他功能的閘道。

![檢視您的遙測的主要路由](./media/app-insights-dashboards/010-oview.png)

您可以自訂任何圖表和格線，並將它們釘選到儀表板。 如此一來，您就可以在中央儀表板上結合來自不同應用程式的重要遙測資料。

## <a name="dashboards"></a>儀表板
您登入 [Microsoft Azure 入口網站](https://portal.azure.com) 之後最先看到的是儀表板。 您可以在這裡將所有 Azure 資源中對您最重要的圖表結合在一起，包括來自 [Azure Application Insights](app-insights-overview.md) 的遙測。

![自訂的儀表板。](./media/app-insights-dashboards/31.png)

1. **瀏覽至特定資源**，例如 Application Insights 中您的應用程式︰使用左列。
2. **回到目前的儀表板**，或切換至其他最近的檢視：使用左上方的下拉式功能表。
3. **切換儀表板**︰使用儀表板標題上的下拉式功能表
4. 在儀表板工具列中**建立、編輯和共用儀表板**。
5. **編輯儀表板**：將滑鼠停留在圖格上，然後使用其頂端列加以移動、自訂或移除。

## <a name="add-to-a-dashboard"></a>加入儀表板中
當您尋找特別感興趣的刀鋒視窗或一組圖表時，您可將其釘選一份到儀表板。 您將會在下次返回該處時見到它。

![若要釘選圖表，可將滑鼠停留在其上方，然後按一下標頭中的 [...]。](./media/app-insights-dashboards/33.png)

1. 將圖表釘選到儀表板。 圖表的複本會出現在儀表板上。
2. 將整個刀鋒視窗釘選到儀表板 - 它會在儀表板上顯示為圖格以供您點選。
3. 按一下左上角以返回目前的儀表板。 然後您可以使用下拉式功能表，返回目前的檢視。

請注意，圖表會分組為圖格︰一個圖格中可包含多個圖表。 您將整個圖格釘選至儀表板。

圖表會以取決於圖表時間範圍的頻率自動重新整理：

* 長達 1 小時的時間範圍：每隔 5 分鐘重新整理一次
* 介於 1-24 小時的時間範圍：每隔 15 分鐘重新整理一次
* 超過 24 小時的時間範圍：(時間範圍)/60

### <a name="pin-any-query-in-analytics"></a>在分析中釘選任何查詢
您也可以[釘選分析](../log-analytics/query-language/get-started-analytics-portal.md)圖表到[共用](#share-dashboards-with-your-team)儀表板。 這可讓您新增任意查詢以及標準度量的圖表。 

系統會每小時自動重新計算結果。 按一下圖表上的 [重新整理] 圖示，即可立即重新計算。 (瀏覽器重新整理並不會重新計算)。

## <a name="adjust-a-tile-on-the-dashboard"></a>調整儀表板上的圖格
圖格在儀表板上之後，您就可以進行調整。

![暫留在一個圖表，以進行編輯。](./media/app-insights-dashboards/36.png)

1. 將圖表加入圖格中。
2. 設定度量、分組依據的維度，和圖表的樣式 (資料表、圖形)。
3. 拖曳圖表加以放大；按一下 [復原] 按鈕來重設時間範圍；在圖格上設定圖表的篩選屬性。
4. 設定圖格的標題。

從計量瀏覽器刀鋒釘選的圖格，編輯選項會比從 [概觀] 刀鋒視窗釘選的圖格多。

您釘選的原始圖格不會受到您的編輯影響。

## <a name="switch-between-dashboards"></a>切換儀表板
您可以儲存多個儀表板並在之間進行切換。 當您釘選圖表或刀鋒視窗時，即會將它們加入目前的儀表板。

![若要在儀表板間進行切換，按一下 [儀表板]，然後選取已儲存的儀表板。 若要建立並儲存新的儀表板，按一下 [新增]。 若要重新排列，按一下 [編輯]。](./media/app-insights-dashboards/32.png)

例如，您可能必須使用一個儀表板，在小組室中以全螢幕顯示，並使用另一個儀表板進行一般開發。

在儀表板上，刀鋒視窗會顯示為磚：按一下即可移至刀鋒視窗。 圖表會複製其原始位置中的圖表。

![按一下圖格以開啟它所代表的刀鋒視窗](./media/app-insights-dashboards/35.png)

## <a name="share-dashboards"></a>共用儀表板
建立儀表板之後，您可以與其他使用者共用它。

![在儀表板標題中按一下 [共用]](./media/app-insights-dashboards/41.png)

深入了解 [角色和存取控制](app-insights-resources-roles-access-control.md)。

## <a name="create-dashboards-programmatically"></a>以程式設計方式建立儀表板
您可以使用 [Azure Resource Manager](https://docs.microsoft.com/azure/azure-portal/azure-portal-dashboards-create-programmatically) 與簡單的 JSON 編輯器，將建立儀表板自動化。

## <a name="app-navigation"></a>應用程式導覽
概觀刀鋒視窗是您的應用程式詳細資訊的閘道。

* **任何圖表或圖格** - 按一下任何圖格或圖表，以查看更多關於其顯示內容的詳細資訊。

### <a name="overview-blade-buttons"></a>概觀刀鋒視窗按鈕
![概觀刀鋒視窗頂端導覽列](./media/app-insights-dashboards/app-overview-top-nav.png)

* [**計量瀏覽器**](app-insights-metrics-explorer.md) - 建立自己的效能和使用量圖表。
* [**搜尋**](app-insights-diagnostic-search.md) - 調查事件的特定執行個體，例如要求、例外狀況或記錄檔案追蹤。
* [**分析**](app-insights-analytics.md) - 遙測的強大查詢。
* **時間範圍** - 調整刀鋒視窗上所有圖表所顯示的範圍。
* **刪除** - 刪除此應用程式的 Application Insights 資源。 您應該也從您的應用程式程式碼中移除 Application Insights 套件，或編輯您應用程式中的[檢測金鑰](app-insights-create-new-resource.md#copy-the-instrumentation-key)，將遙測導向至不同的 Application Insights 資源。

### <a name="essentials-tab"></a>Essentials 索引標籤
* [檢測金鑰](app-insights-create-new-resource.md#copy-the-instrumentation-key) - 識別此應用程式資源。

### <a name="app-navigation-bar"></a>應用程式導覽列
![左導覽列](./media/app-insights-dashboards/app-left-nav-bar.png)

* **概觀** - 返回應用程式概觀刀鋒視窗。
* **活動記錄檔** - 警示與 Azure 系統管理事件。
* [**存取控制**](app-insights-resources-roles-access-control.md) - 將存取權提供給小組成員和其他人。
* [**標籤**](../azure-resource-manager/resource-group-using-tags.md) - 使用標籤將您的應用程式與其他應用程式分組。

調查

* [**應用程式對應**](app-insights-app-map.md) - 顯示應用程式元件的使用中對應，其衍生自相依性資訊。
* [**智慧型偵測**](app-insights-proactive-diagnostics.md) - 檢閱最近的效能警示。
* [**即時串流**](app-insights-live-stream.md) - 一組固定且幾近即時的計量，在部署新組建或偵錯時非常實用。
* [**可用性 / Web 測試**](app-insights-monitor-web-app-availability.md) - 從世界各地將一般要求傳送至您的 Web 應用程式。*
* [**失敗、效能**](app-insights-web-monitor-performance.md) - 對您的應用程式提出的要求和您的應用程式對[相依項目](app-insights-asp-net-dependencies.md)提出的要求，其例外狀況、失敗率和回應時間。
* [**效能**](app-insights-web-monitor-performance.md) - 回應時間、相依性回應時間。
* [伺服器](app-insights-web-monitor-performance.md) - 效能計數器。 當您 [安裝狀態監視器](app-insights-monitor-performance-live-website-now.md)時適用。
* **瀏覽器** - 頁面檢視和 AJAX 效能。 當您 [檢測網頁](app-insights-javascript.md)時適用。
* **使用量** - 頁面檢視、使用者和工作階段計數。 當您 [檢測網頁](app-insights-javascript.md)時適用。

設定

* **開始使用** - 內嵌教學課程。
* **屬性** - 檢測金鑰、訂用帳戶和資源識別碼。
* [警示](app-insights-alerts.md) - 計量警示組態。
* [連續匯出](app-insights-export-telemetry.md) - 設定將遙測匯出至 Azure 儲存體。
* [效能測試](app-insights-monitor-web-app-availability.md#performance-tests) - 設定您的網站上的模擬負載。
* [配額和價格](app-insights-pricing.md)和[擷取取樣](app-insights-sampling.md)。
* **API 存取** - 建立[版本註解](app-insights-annotations.md)以及資料存取 API。
* [**工作項目**](app-insights-diagnostic-search.md#create-work-item) - 連線到工作追蹤系統，讓您可以在檢查遙測時建立 Bug。

設定

* [**鎖定**](../azure-resource-manager/resource-group-lock-resources.md) -鎖定 Azure 資源
* [**自動化指令碼**](app-insights-powershell.md) - 匯出 Azure 資源的定義，如此您就能用其做為範本來建立新資源。


## <a name="video"></a>影片

> [!VIDEO https://channel9.msdn.com/events/Connect/2016/112/player]

## <a name="next-steps"></a>後續步驟

|  |  |
| --- | --- |
| [計量瀏覽器](app-insights-metrics-explorer.md)<br/>篩選與分割計量 |![搜尋範例](./media/app-insights-dashboards/64.png) |
| [診斷搜尋](app-insights-diagnostic-search.md)<br/>尋找和檢查事件、相關的事件，並建立 Bug |![搜尋範例](./media/app-insights-dashboards/61.png) |
| [分析](app-insights-analytics.md)<br/>功能強大的查詢語言 |![搜尋範例](./media/app-insights-dashboards/63.png) |
