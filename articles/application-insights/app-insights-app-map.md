---
title: Azure Application Insights 的應用程式對應 | Microsoft Docs
description: 使用應用程式對應監視複雜的應用程式拓撲
services: application-insights
documentationcenter: ''
author: mrbullwinkle
manager: carmonm
ms.assetid: 3bf37fe9-70d7-4229-98d6-4f624d256c36
ms.service: application-insights
ms.workload: tbd
ms.tgt_pltfrm: ibiza
ms.devlang: na
ms.topic: conceptual
ms.date: 06/14/2018
ms.reviewer: sdash
ms.author: mbullwin
ms.openlocfilehash: 9b39eef5accec4764f61ab31dd894d368242ee3d
ms.sourcegitcommit: cc4fdd6f0f12b44c244abc7f6bc4b181a2d05302
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/25/2018
ms.locfileid: "47094645"
---
# <a name="application-map-triage-distributed-applications"></a>應用程式對應：對分散式應用程式進行分級

應用程式對應可協助您找出分散式應用程式所有元件的效能瓶頸或失敗熱點。 對應上的每個節點各代表應用程式元件或其相依性；並具有健康情況 KPI 和警示狀態。 您可以從任何元件逐一點選至更詳細的診斷，例如 Application Insights 事件。 如果您的應用程式使用 Azure 服務，您也可以逐一點選 Azure 診斷，例如 SQL 資料庫建議程式的建議。

## <a name="what-is-a-component"></a>什麼是元件？

元件是分散式/微服務應用程式中可獨立部署的組件。 開發人員和作業小組能在程式碼層級檢視或存取這些應用程式元件所產生的遙測資料。 

* 元件不同於 SQL、EventHub 等小組/組織可能無法存取的「可觀察」外部相依性 (程式碼或遙測資料)。
* 元件能在任何數量的伺服器/角色/容器執行個體上執行。
* 元件可以是不同的 Application Insights 檢測金鑰 (即使訂用帳戶不同)，也可以是回報給單一 Application Insights 檢測金鑰的不同角色。 預覽對應體驗會顯示元件 (不論元件的設定方式為何)。

## <a name="composite-application-map"></a>複合應用程式對應

您可以查看跨相關應用程式元件多個層級的完整應用程式拓撲。 元件可以是不同的 Application Insights 資源，或是單一資源中的不同角色。 應用程式對應會尋找元件，方法是遵循已安裝 Application Insights SDK 之伺服器之間所發出的 HTTP 相依性呼叫。 

這項體驗一開始會漸進地探索元件。 首次載入應用程式對應時會觸發一組查詢，以探索與此元件相關的元件。 在探索到應用程式中的元件時，左上角的按鈕會依探索到的元件數目進行更新。 

當您按一下 [更新對應元件] 時，系統便會使用目前為止所探索的所有元件來重新整理對應。

如果所有元件都是單一 Application Insights 資源內的角色，則不需要進行此探索步驟。 這類應用程式一開始會載入所有元件。

![應用程式對應螢幕擷取畫面](media/app-insights-app-map/001.png)

這項新體驗的重要目標之一，是要能夠以視覺化方式顯示含有數百個元件的複雜拓撲。

按一下任何元件，即可查看相關深入資訊，並前往該元件的效能和失敗分級體驗。

![飛出視窗](media/app-insights-app-map/application-map-001.png)

### <a name="investigate-failures"></a>調查失敗

選取 [調查失敗] 以啟動 [失敗] 窗格。

![調查失敗按鈕的螢幕擷取畫面](media/app-insights-app-map/investigate-failures.png)

![失敗體驗的螢幕擷取畫面](media/app-insights-app-map/failures.png)

### <a name="investigate-performance"></a>調查效能

若要針對效能問題進行疑難排解，請選取 [調查效能]

![調查效能按鈕的螢幕擷取畫面](media/app-insights-app-map/investigate-performance.png)

![效能體驗的螢幕擷取畫面](media/app-insights-app-map/performance.png)

### <a name="go-to-details"></a>前往詳細資料

選取 [移至詳細資料] 以探索端對端交易體驗，其可提供對呼叫堆疊層級進行的檢視。

![移至詳細資料按鈕的螢幕擷取畫面](media/app-insights-app-map/go-to-details.png)

![端對端交易詳細資料的螢幕擷取畫面](media/app-insights-app-map/end-to-end-transaction.png)

### <a name="view-in-analytics"></a>在 Analytics 中檢視

若要進一步查詢及調查應用程式資料，請按一下 [在 Analytics 中檢視]。

![在 Analytics 中檢視按鈕的螢幕擷取畫面](media/app-insights-app-map/view-in-analytics.png)

![Analytics 體驗的螢幕擷取畫面](media/app-insights-app-map/analytics.png)

### <a name="alerts"></a>警示

若要檢視作用中警示和導致警示觸發的基礎規則，請選取 [警示]。

![警示按鈕的螢幕擷取畫面](media/app-insights-app-map/alerts.png)

![Analytics 體驗的螢幕擷取畫面](media/app-insights-app-map/alerts-view.png)

## <a name="video"></a>影片

> [!VIDEO https://channel9.msdn.com/events/Connect/2016/112/player] 

## <a name="feedback"></a>意見反應
請透過入口網站的意見反應選項提供意見反應。

![MapLink-1 影像](./media/app-insights-app-map/13.png)

## <a name="next-steps"></a>後續步驟

* [Azure 入口網站](https://portal.azure.com)
