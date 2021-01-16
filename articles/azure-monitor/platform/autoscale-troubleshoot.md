---
title: 針對 Azure 自動調整進行疑難排解
description: 使用 Service Fabric、虛擬機器、Web Apps 和雲端服務中所使用的 Azure 自動調整來追蹤問題。
ms.topic: conceptual
ms.date: 11/4/2019
ms.subservice: autoscale
ms.openlocfilehash: 8c4589acd17e76d1341d5aceada67e565c8f8c37
ms.sourcegitcommit: 25d1d5eb0329c14367621924e1da19af0a99acf1
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 01/16/2021
ms.locfileid: "98251262"
---
# <a name="troubleshooting-azure-autoscale"></a>針對 Azure 自動調整進行疑難排解
 
Azure 監視器自動調整可協助您執行適當數量的資源來處理應用程式的負載。 它可讓您新增資源來處理增加的負載，也可以藉由移除閒置的資源來節省成本。 您可以根據您選擇的排程、固定日期時間或資源計量進行調整。 如需詳細資訊，請參閱 [自動調整總覽](autoscale-overview.md)。

自動調整服務會為您提供計量和記錄，以瞭解所發生的調整動作，以及導致這些動作的條件評估。 您可以找到下列問題的解答：

- 為什麼我的服務相應放大或縮小？
- 為什麼我的服務無法調整？
- 自動調整動作為何失敗？
- 為什麼自動調整動作需要花費時間進行調整？
  
## <a name="autoscale-metrics"></a>自動調整計量

自動調整會提供您 [四個度量](metrics-supported.md#microsoftinsightsautoscalesettings) 來瞭解其運作方式。 

- **觀察** 到的計量值-自動調整引擎所見或計算的度量值（您選擇的度量值）。 由於單一自動調整設定可以有多個規則，因此有多個計量來源，因此您可以使用「計量來源」作為維度來進行篩選。
- 計量 **閾值**-您設定來採取調整動作的閾值。 由於單一自動調整設定可以有多個規則，因此有多個計量來源，因此您可以使用「計量規則」作為維度來進行篩選。
- **觀察到的容量** -自動調整引擎所見的目標資源實例的有效數目。
- **已起始的調整動作** - 自動調整引擎所起始的相應放大和相應縮小動作數目。 您可以依相應放大和相應縮小動作進行篩選。

您可以使用 [計量瀏覽器](metrics-getting-started.md) ，在同一個位置繪製上述度量的圖表。 圖表應該會顯示：

  - 實際度量
  - 自動調整引擎所見/計算的度量
  - 調整規模動作的閾值
  - 容量變更 

## <a name="example-1---analyzing-a-simple-autoscale-rule"></a>範例 1-分析簡單的自動調整規則 

針對虛擬機器擴展集，我們有一個簡單的自動調整設定：

- 當集合的平均 CPU 百分比大於 70% 10 分鐘時相應放大 
- 當集合的 CPU 百分比小於5% 超過10分鐘時進行調整。 

讓我們從自動調整服務中檢查計量。
 
![螢幕擷取畫面顯示虛擬機器擴展集百分比 CPU 範例。](media/autoscale-troubleshoot/autoscale-vmss-CPU-ex-full-1.png)

![虛擬機器擴展集百分比 CPU 範例](media/autoscale-troubleshoot/autoscale-vmss-CPU-ex-full-2.png)

**_圖1a：虛擬機器擴展集的 CPU 計量百分比和自動調整設定的觀察計量值_* 計量

![計量閾值和觀察到的容量](media/autoscale-troubleshoot/autoscale-metric-threshold-capacity-ex-full.png)

_*_圖 1b-度量閾值和觀察到的容量_*_

在圖1b 中，相應放大規則的 _ 計量 *閾值** (淺藍線) 為70。  觀察到的 **容量** (深藍線) 顯示作用中的實例數目，目前為3。 

> [!NOTE]
> 您將需要依計量觸發規則維度 scale out 篩選計量 **閾值** (增加) 規則，以查看相應放大閾值，以及依規則中的尺規 (減少) 。 

## <a name="example-2---advanced-autoscaling-for-a-virtual-machine-scale-set"></a>範例 2-虛擬機器擴展集的 Advanced 自動調整

我們有一個自動調整設定，可讓虛擬機器擴展集資源根據其本身的計量 **輸出流量** 向外延展。 請注意，已檢查計量閾值的 [ **依實例劃分度量** ] 選項。 

調整動作規則為： 

如果 **每個實例的輸出流量** 值大於10，則自動調整規模服務應由1個實例相應放大。 

在此情況下，自動調整引擎觀察到的度量值會計算為實際的計量值除以實例的數目。 如果觀察到的計量值小於閾值，則不會起始相應放大動作。 
 
![螢幕擷取畫面顯示 [平均輸出流程] 頁面，其中包含虛擬機器擴展集自動調整度量圖表的範例。](media/autoscale-troubleshoot/autoscale-vmss-metric-chart-ex-1.png)

![虛擬機器擴展集自動調整計量圖表範例](media/autoscale-troubleshoot/autoscale-vmss-metric-chart-ex-2.png)

**_圖 2-虛擬機器擴展集自動調整計量圖表範例_* _

在 [圖 2] 中，您可以看到兩個度量圖表。 

上方的圖表會顯示 _ *輸出流程** 度量的實際值。 實際值為6。 

底部的圖表會顯示幾個值。 
 - **觀察到的度量值** (淺藍色) 為3，因為有2個作用中的實例，6除以2是3。 
 - **觀察到的容量** (紫色) 顯示自動調整引擎看到的實例計數。 
 - **標準閾值** (淺綠色) 設定為10。 

如果有多個調整規模動作規則，您可以使用 [計量瀏覽器] 圖表中的 [分割] 或 [ **新增篩選** ] 選項，根據特定來源或規則來查看度量。 如需分割計量圖表的詳細資訊，請參閱計量 [圖表的 Advanced 功能-分割](metrics-charts.md#apply-splitting)

## <a name="example-3---understanding-autoscale-events"></a>範例 3-瞭解自動調整事件

在 [自動調整設定] 畫面中，移至 [ **執行歷程記錄** ] 索引標籤以查看最近的調整動作。 此索引標籤也會顯示一段時間內 **觀察到的容量** 變更。 若要尋找所有自動調整動作的詳細資料，包括更新/刪除自動調整規模設定等作業，請查看活動記錄，並依自動調整規模作業篩選。

![自動調整設定執行歷程記錄](media/autoscale-troubleshoot/autoscale-setting-run-history-smaller.png)

## <a name="autoscale-resource-logs"></a>自動調整資源記錄

與任何其他 Azure 資源相同，自動調整服務會提供 [資源記錄](platform-logs-overview.md)。 有兩種記錄類別。

- **自動調整評估** -自動調整引擎會在每次執行檢查時，記錄每個單一條件評估的記錄專案。  專案包含觀察到的計量值、評估的規則，以及評估是否產生調整規模動作的詳細資料。

- **自動調整規模調整動作** -引擎記錄規模調整服務所起始的調整動作事件，以及這些縮放動作的結果 (成功、失敗，以及自動調整規模服務) 所看到的調整量。

如同任何 Azure 監視器支援的服務，您可以使用 [診斷設定](diagnostic-settings.md) 來路由傳送這些記錄：

- 至 Log Analytics 工作區以進行詳細分析
- 到事件中樞，然後再到非 Azure 工具
- 至您的 Azure 儲存體帳戶以進行封存  

![自動調整診斷設定](media/autoscale-troubleshoot/diagnostic-settings.png)

上圖顯示 Azure 入口網站自動調整診斷設定。 您可以在這裡選取 [診斷/資源記錄檔] 索引標籤，並啟用記錄檔收集和路由。 您也可以使用 REST API、CLI、PowerShell、Resource Manager 診斷設定的範本來執行相同的動作，方法是將資源類型選擇為 [ *AutoscaleSettings*]。 

## <a name="troubleshooting-using-autoscale-logs"></a>使用自動調整記錄進行疑難排解 

為了獲得最佳的疑難排解體驗，建議您在建立自動調整設定時，透過工作區將記錄路由傳送至 Azure 監視器記錄 (Log Analytics) 。 此程式會顯示在上一節的圖片中。 您可以使用 Log Analytics 來更妥善地驗證評估和調整動作。

設定自動調整記錄以傳送至 Log Analytics 工作區之後，您可以執行下列查詢來檢查記錄。 

若要開始使用，請嘗試此查詢以查看最近的自動調整評估記錄：

```Kusto
AutoscaleEvaluationsLog
| limit 50
```

或嘗試下列查詢來查看最近的調整動作記錄：

```Kusto
AutoscaleScaleActionsLog
| limit 50
```

請使用以下各節來解決這些問題。 

## <a name="a-scale-action-occurred-that-i-didnt-expect"></a>發生未預期的調整規模動作

先執行調整動作的查詢，以找出您感興趣的調整規模動作。 如果是最新的調整動作，請使用下列查詢：

```Kusto
AutoscaleScaleActionsLog
| take 1
```

從 [調整動作] 記錄檔中選取 [CorrelationId] 欄位。 使用 CorrelationId 找出正確的評估記錄。 執行下列查詢將會顯示評估為該調整動作的所有規則和條件。

```Kusto
AutoscaleEvaluationsLog
| where CorrelationId = "<correliationId>"
```

## <a name="what-profile-caused-a-scale-action"></a>哪些設定檔造成調整動作？

發生調整的動作，但您有重迭的規則和設定檔，而且需要追蹤造成動作的原因。 

找出調整規模動作的 correlationId (如範例 1) 所述，然後對評估記錄執行查詢，以深入瞭解此設定檔。

```Kusto
AutoscaleEvaluationsLog
| where CorrelationId = "<correliationId_Guid>"
| where ProfileSelected == true
| project ProfileEvaluationTime, Profile, ProfileSelected, EvaluationResult
```

您也可以使用下列查詢來更清楚地瞭解整個設定檔評估

```Kusto
AutoscaleEvaluationsLog
| where TimeGenerated > ago(2h)
| where OperationName contains == "profileEvaluation"
| project OperationName, Profile, ProfileEvaluationTime, ProfileSelected, EvaluationResult
```

## <a name="a-scale-action-did-not-occur"></a>未發生調整規模動作

我預期會有調整規模的動作，而不會發生。 可能沒有調整動作事件或記錄。

如果您使用以計量為基礎的規模調整規則，請參閱自動調整計量。 觀察到的計量值或觀察到的 **容量** 可能不是您預期的 **值**，因此不會引發調整規模規則。 您仍會看到評估，但不會顯示相應放大規則。 也有可能是因為冷卻時間持續發生調整動作。 

 在預期調整動作發生的期間內，檢查自動調整評估記錄。 檢查它所做的所有評估，以及它決定不觸發調整動作的原因。


```Kusto
AutoscaleEvaluationsLog
| where TimeGenerated > ago(2h)
| where OperationName == "MetricEvaluation" or OperationName == "ScaleRuleEvaluation"
| project OperationName, MetricData, ObservedValue, Threshold, EstimateScaleResult
```

## <a name="scale-action-failed"></a>調整動作失敗

在某些情況下，自動調整服務會採取調整動作，但系統決定不調整或無法完成調整動作。 您可以使用此查詢來尋找失敗的調整動作。

```Kusto
AutoscaleScaleActionsLog
| where ResultType == "Failed"
| project ResultDescription
```

建立警示規則，以取得自動調整動作或失敗的通知。 您也可以建立警示規則，以取得自動調整事件的通知。

## <a name="schema-of-autoscale-resource-logs"></a>自動調整資源記錄的架構

如需詳細資訊，請參閱 [自動調整資源記錄](autoscale-resource-log-schema.md)

## <a name="next-steps"></a>後續步驟
閱讀自動調整 [最佳作法](autoscale-best-practices.md)的資訊。 
