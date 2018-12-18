---
title: OMS 入口網站移至 Azure | Microsoft 文件
description: OMS 入口網站目前終止服務，連同其所有功能將移至 Azure 入口網站。 本文將提供此次轉換的詳細資料。
services: log-analytics
documentationcenter: ''
author: bwren
manager: carmonm
editor: ''
ms.service: log-analytics
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 08/27/2018
ms.author: bwren
ms.component: ''
ms.openlocfilehash: 6e3026ea3aca897ab63f4ad7d72eac438fdb7199
ms.sourcegitcommit: f20e43e436bfeafd333da75754cd32d405903b07
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/17/2018
ms.locfileid: "49389106"
---
# <a name="oms-portal-moving-to-azure"></a>OMS 入口網站移至 Azure

> [!NOTE]
> 本文適用於 Azure 公用雲端和政府雲端，除非另有指示。

Azure 入口網站為所有 Azure 服務的中樞，提供豐富的管理體驗，例如用以建立資源關聯的儀表板、用以尋找資源的智慧型搜尋，以及用以管理資源的標記等功能。 為了整合及簡化監控和管理工作流程，我們已陸續將 OMS 入口網站的功能加入 Azure 入口網站。 現在 OMS 入口網站的所有功能都已納入 Azure 入口網站。 事實上，流量分析等一些新功能僅能在 Azure 入口網站中使用。 您將能夠在 Azure 入口網站中，完成您在 OMS 入口網站中所執行的動作以及其他更多項目。 如果您尚未這麼做，請立即開始使用 Azure 入口網站！

**OMS 入口網站將於 2019 年 1 月 15 日正式淘汰。** 我們很高興移至 Azure 入口網站，並預期此次轉換將會很簡單。 但我們了解任何變更均有其難度，而且可能會對您造成困擾。 如有任何問題、意見反應或考量，請傳送至 **LAUpgradeFeedback@microsoft.com**。 本文其餘部分將介紹關鍵案例和此次轉換的藍圖。

## <a name="what-is-changing"></a>變更內容為何？ 
下列變更會隨著 OMS 入口網站的淘汰一併宣布。 並將在下列各節中逐一詳細說明這些變更。

- 您[只可以在 Azure 入口網站中](#new-workspaces)建立新的工作區。
- 新的警示管理體驗會[取代「警示管理」解決方案](#changes-to-alerts)。
- 現在要在 Azure 入口網站中，使用 Azure 角色型存取控制來進行[使用者存取管理](#user-access-and-role-migration)。
- 您將[不再需要 Application Insights Connector](#application-insights-connector-and-solution)，因為可以透過跨工作區的查詢，來使用相同的功能。
- [OMS 行動裝置應用程式](#oms-mobile-app)即將淘汰。 
- 將由透過流量分析解決方案所提供的增強功能，[取代 NSG 解決方案](#azure-network-security-group-analytics)。
- System Center Operations Manager 與 Log Analytics 的新連線需要[更新的管理組件](#system-center-operations-manager)。
- 如需[更新管理](../automation/automation-update-management.md)變更的詳細資訊，請參閱[將您的 OMS 更新部署移轉至 Azure](../automation/migrate-oms-update-deployments.md)。


## <a name="what-should-i-do-now"></a>我現在該怎麼辦？
雖然大部分功能都可以繼續運作而不用執行任何移轉，但是您需要執行下列工作：

- 您需要[遷移您的使用者權限](#user-access-and-role-migration)至 Azure 入口網站。
- 如需轉移更新管理解決方案的詳細資訊，請參閱[將您的 OMS 更新部署移轉至 Azure](../automation/migrate-oms-update-deployments.md)。

如需有關如何轉換到 Azure 入口網站的資訊，請參閱 [Log Analytics 使用者從 OMS 入口網站轉換至 Azure 入口網站的常見問題](../log-analytics/log-analytics-oms-portal-faq.md)。 如有任何意見反應、問題或疑慮，請傳送至 **LAUpgradeFeedback@microsoft.com**。

## <a name="user-access-and-role-migration"></a>使用者存取權和角色的移轉
Azure 入口網站存取管理功能，比在 OMS 入口網站中的存取管理功能更為豐富而強大。 請參閱[管理工作區](log-analytics-manage-access.md#manage-accounts-and-users)，以瞭解 Log Analytics 中存取管理的詳細資料。

> [!NOTE]
> 本文的舊版指出權限會自動從 OMS 入口網站轉換至 Azure 入口網站。 我們已捨棄此自動轉換的計劃，您必須自行執行轉換。

您可能已經在 Azure 入口網站中具有適當存取權，在此情況下，您不需要進行任何變更。 但是有可能您沒有適當的存取權，在此情況下，您的系統管理員必須為您指派權限。

- 您在 OMS 入口網站中具有唯讀使用者權限，但是在 Azure 入口網站中沒有任何權限。 
- 您在 OMS 入口網站中具有參與者權限，但是在 Azure 入口網站中只有讀取者存取權。
 
在這兩種情況下，您的系統管理員都必須手動為您指派下表中的適當角色。 我們建議您在資源群組或訂用帳戶層級指派此角色。  這兩種情況的更精準指引即將推出。

| OMS 入口網站權限 | Azure 角色 |
|:---|:---|
| 唯讀 | Log Analytics 讀者 |
| 參與者 | Log Analytics 參與者 |
| 系統管理員 | 擁有者 | 
 

## <a name="new-workspaces"></a>新工作區
您無法再使用 OMS 入口網站來建立新的工作區。 若要在 Azure 入口網站中建立新工作區，請遵循[在 Azure 入口網站中建立 Log Analytics 工作區](log-analytics-quick-create-workspace.md)的指引。

## <a name="changes-to-alerts"></a>警示的變更

### <a name="alert-extension"></a>警示擴充功能  

> [!NOTE]
> 公用雲端的警示功能現在已完全擴充至 Azure 入口網站。 您可以在 OMS 入口網站中檢視現有的警示規則，但只能在 Azure 入口網站中管理這些警示規則。 警示將在 2018 年 10 月從 Azure 政府雲端開始擴充至 Azure 入口網站中。

警示已[擴充至 Azure 入口網站](../monitoring-and-diagnostics/monitoring-alerts-extend.md)。 一旦完成後，警示的管理動作將只能在 Azure 入口網站中使用。 現有警示會繼續列在 OMS 入口網站中。 若您藉由使用 Log Analytics 警示 REST API 或 Log Analytics 警示資源範本以程式設計的方式存取警示，則必須使用動作群組，而非您 API 呼叫、Azure Resource Manager 範本及 PowerShell 命令中的動作。

### <a name="alert-management-solution"></a>警示管理解決方案
您可以使用 [Azure 監視器的統一警示介面](../monitoring-and-diagnostics/monitoring-overview-unified-alerts.md)，而不用[警示管理解決方案](log-analytics-solution-alert-management.md)，即可以視覺化方式檢視和管理您的通知。 這個新體驗會從 Azure 內多個來源彙總警示，包括 Log Analytics 的記錄檔警示。 您可以看到警示的分佈、透過智慧型群組 充分利用對相關警示的自動化分組功能，以及套用豐富的篩選器同時檢視多個訂閱的警示。 所有這些功能自 2018 年 6 月 4 日起均可提供預覽。 警示管理解決方案無法在 Azure 入口網站使用。 

只要為工作區安裝解決方案，警示管理解決方案  (具有警報類型的記錄) 所收集的資料就會繼續存在於 Log Analytics 中。 自 2018 年 8 月起，將啟用從統一警示進入工作區的警示串流，取代此項功能。 預期有部分結構描述將會變更，並將在日後公告。

## <a name="oms-mobile-app"></a>OMS 行動裝置應用程式
OMS 行動裝置應用程式將會連同 OMS 入口網站一併終止。 您不必使用 OMS 行動裝置應用程式來存取有關 IT 基礎結構、 儀表板和已儲存查詢的資訊，而可以直接從您的行動裝置瀏覽器中存取 Azure 入口網站。 若要收到警示，請設定 [Azure 動作群組](../monitoring-and-diagnostics/monitoring-action-groups.md)，即可以簡訊 或語音電話的形式接收通知

## <a name="application-insights-connector-and-solution"></a>Application Insights Connector 與解決方案
[Application Insights Connector](log-analytics-app-insights-connector.md) 可用來將 Application Insights 資料移入 Log Analytics 工作區。 若要啟用整個基礎結構和應用程式資料的可見度，則需要複製此資料。

有了[跨資源查詢](log-analytics-cross-workspace-search.md)的支援，就不再需要複製資料。 因此，將取代現有的 Application Insights 解決方案。 從 10 月開始，您將無法從新的 Application Insights 資源連結到 Log Analytics 工作區。 現有的連結和儀表板將會繼續運作到 2019 年 1 月 15 日為止。


## <a name="azure-network-security-group-analytics"></a>Azure 網路安全性群組分析
[Azure 網路安全性群組分析解決方案](log-analytics-azure-networking-analytics.md#azure-network-security-group-analytics-solution-in-log-analytics)將由最近推出的[流量分析](https://azure.microsoft.com/blog/traffic-analytics-in-preview/)所取代，讓您清楚掌握雲端網路上使用者和應用程式的活動。 流量分析可協助您稽核貴組織的網路活動、保護應用程式和資料、將工作負載效能最佳化並保持符合規範。 

此解決方案可分析 NSG 流程記錄，並提供下列深入見解。

- 在 Azure 與網際網路、公用雲端區域、VNET 以及子網路之間的網路流量。
- 您網路上的應用程式和通訊協定，而不需要 Sniffer 或專用的流量收集設備。
- 發言最多者、通訊流量高的應用程式、雲端中的 VM 對話、流量作用區。
- 跨 VNET 的流量來源和目的地，關鍵業務服務和應用程式之間的相互關係。
- 安全性包括 惡意流量、對網際網路開放的連接埠、嘗試存取網際網路的應用程式或 VM。
- 容量使用率，這可協助您解決過度佈建或使用量過低的問題。

您可以繼續仰賴診斷設定將 NSG 記錄檔傳送到 Log Analytics，如此一來，您現有已儲存的搜尋、 警示、 儀表板將可繼續運作。 在進一步通知之前，已安裝解決方案的客戶可以繼續使用。 自 9 月 5 日起，「網路安全性群組分析」解決方案將從市集中移除，並透過社群以 [Azure 快速入門範本](https://azure.microsoft.com/resources/templates/?resourceType=Microsoft.Operationalinsights)的方式提供。

## <a name="system-center-operations-manager"></a>System Center Operations Manager
如果您已經[將 Operations Manager 管理群組連線到 Log Analytics](log-analytics-om-agents.md)，則此服務將繼續運作，無須變更。 然而，針對新連線，您必須依照[使用 Microsoft System Center Operations Manager 管理組件設定 Operations Management Suite](https://blogs.technet.microsoft.com/momteam/2018/07/25/microsoft-system-center-operations-manager-management-pack-to-configure-operations-management-suite/) 中的指引。

## <a name="next-steps"></a>後續步驟
- 如需有關從 OMS 入口網站移至 Azure 入口網站的指引，請參閱 [Log Analytics 使用者從 OMS 入口網站轉換至 Azure 入口網站的常見問題](log-analytics-oms-portal-faq.md)。