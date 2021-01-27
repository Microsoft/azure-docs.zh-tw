---
title: 將來自 Azure 監視器的資料視覺化 | Microsoft Docs
description: 概要介紹用以將儲存在 Azure 監視器中的計量和記錄資料視覺化的可用方法。
ms.subservice: ''
ms.topic: conceptual
author: bwren
ms.author: bwren
ms.date: 01/25/2021
ms.openlocfilehash: bc59c639856de6a5ce5adda1f78266eb7e4d5dd3
ms.sourcegitcommit: aaa65bd769eb2e234e42cfb07d7d459a2cc273ab
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 01/27/2021
ms.locfileid: "98881226"
---
# <a name="visualizing-data-from-azure-monitor"></a>將來自 Azure 監視器的資料視覺化
本文會概要介紹用以將儲存在 Azure 監視器中的記錄和計量資料視覺化的可用方法。

圖表影像等視覺效果可協助分析監視資料，以深入鑽研問題，並找出其中的模式。 根據您所使用的工具，還可以選擇是否要與組織內外的其他使用者共用視覺效果。

## <a name="workbooks"></a>活頁簿
[活頁簿](./platform/workbooks-overview.md)是互動式的文件，可以提供對團隊內部的資料、調查和共同作業的深入解析。 顯見活頁簿實用性的具體範例，在於疑難排解指南和事件事後分析。

![圖表會顯示活頁簿中數個頁面的螢幕擷取畫面，包括分析網頁檢視、使用方式，以及花費在頁面上的時間。](media/visualizations/workbook.png)

### <a name="advantages"></a>優點
- 支援計量和記錄。
- 支援啟用互動式報告的參數，選取表格中的元素，即會動態更新相關聯的圖表和視覺效果。
- 文件式流程。
- 適用於個人或共用活頁簿的選項。
- 簡單、方便共同作業的編寫體驗。
- 範本支援以 GitHub 為基礎的範本資源庫。

### <a name="limitations"></a>限制
- 無自動重新整理功能。
- 並未提供類似儀表板的密集配置，導致活頁簿較不適合做為單一管理平台使用。 專為提供更深入的見解。


## <a name="azure-dashboards"></a>Azure 儀表板
[Azure 儀表板](../azure-portal/azure-portal-dashboards.md)是 Azure 的主要儀表板設計技術。 儀表板特別適合用來提供 Azure 基礎結構和服務的單一管理平台，以利使用者迅速找出重要問題所在。

![螢幕擷取畫面顯示 Azure 儀表板的範例，其中包含可自訂的資訊。](media/visualizations/dashboard.png)

以下是建立儀表板的影片逐步解說。

> [!VIDEO https://www.microsoft.com/en-us/videoplayer/embed/RE4AslH]

### <a name="advantages"></a>優點
- 深入整合至 Azure。 視覺效果可以釘選到多個 Azure 頁面的儀表板，包括 [計量瀏覽器](platform/metrics-charts.md)、 [Log Analytics](log-query/log-analytics-overview.md)和 [Application Insights](app/app-insights-overview.md)。
- 支援計量和記錄。
- 結合多個來源的資料，包括[Application Insights](app/app-insights-overview.md)中[計量瀏覽器](platform/metrics-charts.md)、[記錄查詢](log-query/log-query-overview.md)和[對應](app/app-map.md)和可用性的輸出。
- 個人或共用儀表板的選項。 [ (AZURE RBAC) 與 azure 角色型存取控制](../role-based-access-control/overview.md)整合。
- 自動重新整理。 計量的重新整理端看時間範圍而定，最少五分鐘。 記錄每小時都會重新整理一次，並藉由在指定的視覺效果上按一下 [重新整理] 圖示，或重新整理整個儀表板，依需求提供手動重新整理的選項。
- 使用時間戳記和自訂參數以參數化計量儀表板。
- 彈性的版面配置選項。
- 全螢幕模式。


### <a name="limitations"></a>限制
- 對記錄分析視覺效果的控制有限，不支援資料表。 資料數列的總數限制為50，並在 _其他_ 值區下分組進一步的資料序列。
- 未提供對記錄圖表的自訂參數支援。
- 記錄圖表僅限於過去 30 天。
- 記錄圖表只能釘選到共用儀表板。
- 沒有與儀表板資料的互動功能。
- 內容相關的向下鑽研受限。


## <a name="power-bi"></a>Power BI
[Power BI](https://powerbi.microsoft.com/documentation/powerbi-service-get-started/) 特別適合用來建立以業務為核心的儀表板和報告，對於長期 KPI 趨勢的分析報告也相當合用。 您可以將[記錄查詢的結果匯入](platform/powerbi.md) Power BI 資料集，以便運用其功能，例如，可合併不同來源的資料，以及共用網路和行動裝置上的資料。

![Power BI](media/visualizations/power-bi.png)

### <a name="advantages"></a>優點
- 豐富的視覺效果。
- 廣泛的互動功能，包括放大和交叉篩選。
- 易於在整個組織內共用。
- 能與來自多個資料來源的其他資料整合。
- 在 Cube 中快取結果，效能更佳。


### <a name="limitations"></a>限制
- 支援記錄，但不支援計量。
- 沒有 Azure 整合。 無法透過 Azure Resource Manager 管理儀表板和模型。
- 需要將查詢結果匯入 Power BI 模型中進行設定。 對於結果大小和重新整理有所限制。
- 每天限制的資料重新整理次數為 8 次。


## <a name="grafana"></a>Grafana
[Grafana](https://grafana.com/) 在操作儀表板中，是表現相當出色的開放平台。 特別適用於偵測、隔離、分級事件。 您可以將 [Grafana Azure 監視器資料來源外掛程式](platform/grafana-plugin.md)新增至您的 Azure 訂用帳戶，以使其可視覺化 Azure 計量資料。

![顯示 Grafana 視覺效果的螢幕擷取畫面。](media/visualizations/grafana.png)

### <a name="advantages"></a>優點
- 豐富的視覺效果。
- 豐富的資料來源生態系統。
- 包括縮放的資料互動功能。
- 支援參數。

### <a name="limitations"></a>限制
- 沒有 Azure 整合。 無法透過 Azure Resource Manager 管理儀表板和模型。
- 需付費支援額外的 Grafana 基礎結構，或 Grafana 雲端需要額外費用。


## <a name="build-your-own-custom-application"></a>建置您自己的自訂應用程式
您可以使用任何 REST 用戶端，透過 Azure 監視器的 API 存取其中的記錄資料和計量資料，如此即可建置自訂的網站和應用程式。

### <a name="advantages"></a>優點
- 在 UI、視覺效果、互動功能，以及功能方面具有完全的靈活性。
- 將計量和記錄資料與其他資料來源結合。

### <a name="disadvantages"></a>缺點
- 需要大量的工程工作。


## <a name="azure-monitor-views"></a>Azure 監視器檢視

> [!IMPORTANT]
> 檢視正逐漸被取代。 如需將檢視轉換至活頁簿的指引，請參閱 [Azure 監視器檢視設計工具至活頁簿轉換指南](platform/view-designer-conversion-overview.md)。

[Azure 監視器中的檢視](platform/view-designer.md)可讓您使用記錄資料建立自訂視覺效果。 [監視解決方案](insights/solutions.md)會使用這些視覺效果呈現收集到的資料。


![螢幕擷取畫面顯示 [容器監視解決方案] 磚，以及當您選取時開啟的詳細 Azure 監視器視圖。](media/visualizations/view.png)

### <a name="advantages"></a>優點
- 豐富的記錄資料視覺效果。
- 可匯出和匯入檢視，以將之傳送給其他資源群組和訂用帳戶。
- 可使用工作區和監視解決方案，整合至 Azure 監視器管理模型。
- 提供自訂參數的[篩選條件](platform/view-designer-filters.md)。
- 互動式，可支援多層級的向下鑽研能力 (可從一個檢視鑽研至另一個檢視)

### <a name="limitations"></a>限制
- 支援記錄，但不支援計量。
- 沒有任何個人的檢視。 具有工作區存取權限的使用者均可使用。
- 無自動重新整理功能。
- 版面配置選項有限。
- 不支援跨多個工作區和 Application Insights 應用程式進行查詢。
- 查詢的回應大小限制為 8MB，查詢執行時間為 110 秒。

## <a name="next-steps"></a>後續步驟
- 了解 [ Azure 監視器收集的資料](platform/data-platform.md)。
- 了解 [Azure 儀表板](../azure-portal/azure-portal-dashboards.md)。
- 瞭解 [計量瀏覽器](platform/metrics-getting-started.md)
- 了解[活頁簿](./platform/workbooks-overview.md)。
- 了解[將記錄資料匯入 Power BI](./platform/powerbi.md)。
- 了解 [Grafana Azure 監視器資料來源外掛程式](./platform/grafana-plugin.md)。
- 了解 [Azure 監視器中的檢視](platform/view-designer.md)。

