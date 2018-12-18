---
title: Azure 監視器資源群組深入解析 | Microsoft Docs
description: 透過 Azure 監視器了解您的分散式應用程式與服務在資源群組層級的健康狀態與效能
services: azure-monitor
author: NumberByColors
manager: carmonm
ms.service: azure-monitor
ms.workload: tbd
ms.tgt_pltfrm: ibiza
ms.devlang: multiple
ms.topic: conceptual
ms.date: 08/29/2018
ms.reviewer: mbullwin
ms.author: daviste
ms.openlocfilehash: 723006d37ed0570e32790a0bb70a3dce5a87ade8
ms.sourcegitcommit: 0c64460a345c89a6b579b1d7e273435a5ab4157a
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/31/2018
ms.locfileid: "43346580"
---
# <a name="monitor-resource-groups-with-azure-monitor-preview"></a>使用 Azure 監視器監視資源群組 (預覽)

新式的應用程式通常透過許多離散組件共同合作提供服務，因此複雜且高度分散。 體認到這種複雜性，Azure 監視器可以針對資源群組提供監視深入解析。 如此便能夠將個別資源碰到的任何問題加以分級與診斷，同時就資源群組&mdash;及您的應用程式&mdash;整體的健康狀態與效能提供內容。

## <a name="access-insights-for-resource-groups"></a>存取資源群組的深入解析

1. 從左側導覽列中選取 [資源群組]。
2. 選擇其中一個您想要探索的資源群組。 (如果您有大量的資源群組，則依訂用帳戶篩選有時可能會有幫助。)
3. 若要存取資源群組的深入解析，請按一下任何資源群組左側功能表中的 [深入解析]。

![資源群組深入解析概觀頁面螢幕擷取畫面](.\media\resource-group-insights\0001-overview.png)

## <a name="resources-with-active-alerts-and-health-issues"></a>具有作用中警示與健康狀態問題的資源

概觀頁面會顯示有多少個警示已引發與仍為作用中，以及每個資源目前的「Azure 資源健康狀態」。 此資訊可協助您快速找出任何碰到問題的資源。 警示可幫助您偵測程式碼及基礎結構設定方式中的問題。 「Azure 資源健康狀態」會呈現 Azure 平台本身的問題，而非個別應用程式特有的問題。

![Azure 資源健康狀態窗格螢幕擷取畫面](.\media\resource-group-insights\0002-overview.png)

### <a name="azure-resource-health"></a>Azure 資源健康狀態

若要顯示「Azure 資源健康狀態」，請勾選表格上方的 [顯示 Azure 資源健康狀態]。 此欄預設為隱藏，以幫助頁面快速載入。

![資源健康狀態圖表已新增螢幕擷取畫面](.\media\resource-group-insights\0003-overview.png)

依預設，資源會依應用程式層與資源類型分組。 [應用程式層] 是資源類型的簡單分類，只存在於資源群組深入解析概觀頁面的內容中。 還有與應用程式程式碼、運算基礎結構、網路功能、儲存體 + 資料庫相關的資源類型。 管理工具會取得自己的應用程式層，每個其他資源會歸類為屬於 [其他] 應用程式層。 此群組可幫助您快速了解應用程式的哪些子系統是健康與不健康的。

## <a name="diagnose-issues-in-your-resource-group"></a>診斷您資源群組中的問題

資源群組深入解析頁面還提供其他數種工具，幫助您診斷問題

   |         |          |
   | ---------------- |:-----|
   | [**警示**](https://docs.microsoft.com/azure/monitoring-and-diagnostics/monitoring-overview-unified-alerts)      |  檢視、建立與管理警示。 |
   | [**計量**](https://docs.microsoft.com/azure/monitoring-and-diagnostics/monitoring-overview-metrics) | 視覺化與探索以計量為基礎的資料。    |
   | [**活動記錄**](https://docs.microsoft.com/azure/monitoring-and-diagnostics/monitoring-overview-activity-logs) | 發生在 Azure 的訂用帳戶層級事件。  |
   | [**應用程式對應**](https://docs.microsoft.com/azure/application-insights/app-insights-app-map) | 瀏覽分散式應用程式的拓撲，以找出效能瓶頸或失敗熱點。 |

## <a name="failures-and-performance"></a>失敗與效能

如果您注意到應用程式執行速度緩慢，或使用者回報錯誤，該怎麼辦？ 要搜尋所有資源以隔離問題，是相當耗時的。

[效能] 與 [失敗] 索引標籤會將許多常見資源類型的效能與失敗診斷檢視結合在一起，讓搜尋程序簡化。

大多數的資源類型會開啟「Azure 監視器活頁簿」範本資源庫。 您建立的每個資源庫都能加以自訂、儲存、與您的小組共用，並能在未來重新使用以診斷類似的問題。

### <a name="investigate-failures"></a>調查失敗

若要測試 [失敗] 索引標籤，請選取左側功能表中 [調查] 下方的 [失敗]。

在您做出選擇之後，左側功能表列會變更，以提供新的選項。

![失敗概觀窗格螢幕擷取畫面](.\media\resource-group-insights\00004-failures.png)

若有選擇 App Service，您會看到「Azure 監視器活頁簿」範本資源庫。

![應用程式活頁簿資源庫螢幕擷取畫面](.\media\resource-group-insights\0005-failure-insights-workbook.png)

選擇 [失敗深入解析] 的範本將開啟活頁簿。

![失敗報告螢幕擷取畫面](.\media\resource-group-insights\0006-failure-visual.png)

您可以選取任何資料列。 選取項目便會顯示在圖形化的詳細資料檢視中。

![失敗詳細資料螢幕擷取畫面](.\media\resource-group-insights\0007-failure-details.png)

活頁簿會將建立自訂報告與視覺效果的困難工作抽出，成為可輕鬆取用的格式。 雖然某些使用者可能只想要調整預先建置的參數，但活頁簿是完全可自訂的。

若要了解此活頁簿內部運作的方式，請選取頂端列中的 [編輯]。

![其他編輯選項螢幕擷取畫面](.\media\resource-group-insights\0008-failure-edit.png)

活頁簿各種元素的附近有一些 [編輯] 方塊。 選取作業表格下方的 [編輯] 方塊。

![編輯方塊螢幕擷取畫面](.\media\resource-group-insights\0009-failure-edit-graph.png)

這會顯示驅動表格視覺化的基礎 Log Analytics 查詢。

 ![記錄分析查詢視窗的螢幕擷取畫面](.\media\resource-group-insights\0010-failure-edit-query.png)

您可以直接修改查詢。 或者，您可以使用它做為參考，並在設定您自己的參數化活頁簿時借用它做為參考。

### <a name="investigate-performance"></a>調查效能

效能會提供自己的活頁簿資源庫。 對於 App Service，預先建置的「應用程式效能」活頁簿會提供下列檢視：

 ![效能檢視螢幕擷取畫面](.\media\resource-group-insights\0011-performance.png)

在此情況下，如果您選取編輯，您將會看見這一組由「Azure 監視器計量」技術支援的視覺效果。

 ![含「Azure 計量」的效能檢視螢幕擷取畫面](.\media\resource-group-insights\0012-performance-metrics.png)

## <a name="next-steps"></a>後續步驟

- [Azure 監視器活頁簿](https://docs.microsoft.com/azure/application-insights/app-insights-usage-workbooks)
- [Azure 資源健康狀態](https://docs.microsoft.com/azure/service-health/resource-health-overview)
- [Azure 監視器警示](https://docs.microsoft.com/azure/monitoring-and-diagnostics/monitoring-overview-unified-alerts)
