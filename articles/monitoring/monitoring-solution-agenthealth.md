---
title: OMS 中的代理程式健全狀況解決方案 | Microsoft Docs
description: 本文旨在協助您了解如何使用這個解決方案來監視直接向 OMS 或 System Center Operations Manager 回報之代理程式的健全狀況。
services: operations-management-suite
documentationcenter: ''
author: MGoedtel
manager: carmonm
editor: ''
ms.assetid: ''
ms.service: operations-management-suite
ms.workload: tbd
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 03/19/2017
ms.author: magoedte
ms.openlocfilehash: 8a6275748c82fbb448a767af690121abbd1f669a
ms.sourcegitcommit: 756f866be058a8223332d91c86139eb7edea80cc
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/02/2018
ms.locfileid: "37347604"
---
#  <a name="agent-health-solution-in-oms"></a>OMS 中的代理程式健全狀況解決方案
對於直接向 OMS 工作區或連線到 OMS 之 System Center Operations Manager 管理群組回報的所有代理程式，OMS 中的代理程式健全狀況解決方案可協助您了解哪些代理程式沒有回應且正在提交作業資料。  您可以也追蹤已部署的代理程式數目，其散佈地區，並執行其他查詢，以留意 Azure、其他雲端環境或內部部署中部署之代理程式的散佈情形。    

## <a name="prerequisites"></a>先決條件
部署這個解決方案之前，請確認您目前支援向 OMS 工作區回報或向與 OMS 工作區整合之 [Operations Manager 管理群組](../log-analytics/log-analytics-om-agents.md)回報的 [Windows 代理程式](../log-analytics/log-analytics-windows-agent.md)。    

## <a name="solution-components"></a>方案元件
此解決方案包含下列已新增到您的工作區以及直接連線之代理程式或 Operations Manager 連線之管理群組的資源。

### <a name="management-packs"></a>管理組件
如果 System Center Operations Manager 管理群組已連線到 OMS 工作區，則下列管理組件會安裝在 Operations Manager 中。  新增此解決方案之後，這些管理組件也會安裝在直接連線的 Windows 電腦上。 這些管理組件不需要進行任何設定或管理。

* Microsoft System Center Advisor HealthAssessment Direct Channel Intelligence Pack (Microsoft.IntelligencePacks.HealthAssessmentDirect)
* Microsoft System Center Advisor HealthAssessment Server Channel Intelligence Pack (Microsoft.IntelligencePacks.HealthAssessmentViaServer)。  

如需有關方案管理組件如何更新的詳細資訊，請參閱 [將 Operations Manager 連接到 Log Analytics](../log-analytics/log-analytics-om-agents.md)。

## <a name="configuration"></a>組態
使用[新增解決方案](../log-analytics/log-analytics-add-solutions.md)中所述的程序，將代理程式健全狀況解決方案新增至您的 OMS 工作區。 不需要進一步的組態。


## <a name="data-collection"></a>資料收集
### <a name="supported-agents"></a>支援的代理程式
下表描述此方案支援的連接來源。

| 連接的來源 | 支援 | 說明 |
| --- | --- | --- |
| Windows 代理程式 | yes | 系統會從直接 Windows 代理程式收集活動訊號事件。|
| System Center Operations Manager 管理群組 | yes | 系統會每隔 60 秒從向管理群組回報之代理程式收集活動訊號事件，然後轉送到 Log Analytics。 不需要直接從 Operations Manager 代理程式連線到 Log Analytics。 活動訊號事件資料會從管理群組轉送至 Log Analytics 存放庫。|

## <a name="using-the-solution"></a>使用解決方案
當您將解決方案新增至 OMS 工作區時，[代理程式健全狀況]  圖格會新增至 OMS 儀表板。 此圖格會顯示過去 24 小時內的代理程式總數和沒有回應的代理程式數目。<br><br> ![儀表板上的代理程式健全狀況圖格](./media/monitoring-solution-agenthealth/agenthealth-solution-tile-homepage.png)

按一下 [代理程式健全狀況] 圖格，以開啟 [代理程式健全狀況] 儀表板。  此儀表板包含下表中的資料行。 每個資料行依計數列出前十個事件，這幾個事件符合該資料行中指定時間範圍的準則。 您可以選取每個資料行右下角的 [查看全部] ，或按一下資料行標頭，以執行記錄搜尋來提供完整清單。

| 欄 | 說明 |
|--------|-------------|
| 不同時間的代理程式計數 | Linux 和 Windows 代理程式為期七天的代理程式計數趨勢。|
| 沒有回應的代理程式計數 | 在過去 24 小時內尚未傳送活動訊號的代理程式清單。|
| 依 OS 類型分配 | 劃分您的環境中有多少個 Windows 和 Linux 代理程式。|
| 依代理程式版本分配 | 劃分您的環境中安裝的不同代理程式版本，以及每個版本的計數。|
| 依代理程式類別分配 | 劃分傳送活動訊號事件的不同代理程式類別：直接代理程式、OpsMgr 代理程式或 OpsMgr 管理伺服器。|
| 依管理群組分配 | 劃分您的環境中不同的 SCOM 管理群組。|
| 代理程式的地理位置 | 劃分您擁有代理程式的不同國家/地區，以及在每個國家/地區安裝的代理程式總計數。|
| 已安裝的閘道計數 | 已安裝 OMS 閘道的伺服器數目，以及這些伺服器的清單。|

![代理程式健全狀況儀表板範例](./media/monitoring-solution-agenthealth/agenthealth-solution-dashboard.png)  

## <a name="log-analytics-records"></a>Log Analytics 記錄
此解決方案會在 OMS 存放庫中建立一種類型的記錄。  

### <a name="heartbeat-records"></a>活動訊號記錄
系統會建立類型為 [活動訊號] 的記錄。  這些記錄具有下表中的屬性。  

| 屬性 | 說明 |
| --- | --- |
| 類型 | *活動訊號*|
| 類別 | 值為 [直接代理程式]、[SCOM 代理程式] 或 [SCOM 管理伺服器]。|
| 電腦 | 電腦名稱。|
| OSType | Windows 或 Linux 作業系統。|
| OSMajorVersion | 作業系統主要版本。|
| OSMinorVersion | 作業系統次要版本。|
| 版本 | OMS 代理程式或 Operations Manager 代理程式版本。|
| SCAgentChannel | 值為 [Direct] 和/或 [SCManagementServer]。|
| IsGatewayInstalled | 如果已安裝 OMS 閘道，值為 true，否則值為 false。|
| ComputerIP | 電腦的 IP 位址。|
| RemoteIPCountry | 電腦部署所在的地理位置。|
| ManagementGroupName | Operations Manager 管理群組的名稱。|
| SourceComputerId | 電腦的唯一識別碼。|
| RemoteIPLongitude | 電腦的地理位置經度。|
| RemoteIPLatitude | 電腦的地理位置緯度。|

向 Operations Manager 管理伺服器回報的每個代理程式將會傳送兩個活動訊號，而 SCAgentChannel 屬性的值將同時包含 **Direct** 和 **SCManagementServer** (視您已在自己的 OMS 訂用帳戶中啟用哪些 Analytics 資料來源和解決方案而定)。 請回想一下，解決方案中的資料會從 Operations Manager 管理伺服器直接傳送至 OMS Web 服務，或者因為代理程式上收集的資料量，所以會從代理程式直接傳送至 OMS Web 服務。 若為具有 **SCManagementServer** 值的活動訊號事件，ComputerIP 值為管理伺服器的 IP 位址，因為資料實際上由它上傳。  若為 SCAgentChannel 設為 [Direct] 的活動訊號，此值為代理程式的公用 IP 位址。  

## <a name="sample-log-searches"></a>記錄搜尋範例
下表提供此解決方案所收集之記錄的記錄搜尋範例。

| 查詢 | 說明 |
|:---|:---|
| Heartbeat &#124; distinct Computer |代理程式總數 |
| Heartbeat &#124; summarize LastCall = max(TimeGenerated) by Computer &#124; where LastCall < ago(24h) |過去 24 小時內沒有回應的代理程式計數 |
| Heartbeat &#124; summarize LastCall = max(TimeGenerated) by Computer &#124; where LastCall < ago(15m) |過去 15 分鐘內沒有回應的代理程式計數 |
| Heartbeat &#124; where TimeGenerated > ago(24h) and Computer in ((Heartbeat &#124; where TimeGenerated > ago(24h) &#124; distinct Computer)) &#124; summarize LastCall = max(TimeGenerated) by Computer |在線上的電腦 (過去 24 小時內) |
| Heartbeat &#124; where TimeGenerated > ago(24h) and Computer !in ((Heartbeat &#124; where TimeGenerated > ago(30m) &#124; distinct Computer)) &#124; summarize LastCall = max(TimeGenerated) by Computer |在過去 30 分鐘內離線的代理程式總數 (針對過去 24 小時) |
| Heartbeat &#124; summarize AggregatedValue = dcount(Computer) by OSType |依 OSType 取得一段時間內的代理程式數目趨勢|
| Heartbeat &#124; summarize AggregatedValue = dcount(Computer) by OSType |依 OS 類型分配 |
| Heartbeat &#124; summarize AggregatedValue = dcount(Computer) by Version |依代理程式版本分配 |
| Heartbeat &#124; summarize AggregatedValue = count() by Category |依代理程式類別分配 |
| Heartbeat &#124; summarize AggregatedValue = dcount(Computer) by ManagementGroupName | 依管理群組分配 |
| Heartbeat &#124; summarize AggregatedValue = dcount(Computer) by RemoteIPCountry |代理程式的地理位置 |
| Heartbeat &#124; where iff(isnotnull(toint(IsGatewayInstalled)), IsGatewayInstalled == true, IsGatewayInstalled == "true") == true &#124; distinct Computer |已安裝的 OMS 閘道數目 |




## <a name="next-steps"></a>後續步驟

* 如需有關從 Log Analytics 產生的警示的詳細資料，請深入了解 [Log Analytics 中的警示](../log-analytics/log-analytics-alerts.md) 。
