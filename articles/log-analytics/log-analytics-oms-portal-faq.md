---
title: Log Analytics 使用者從 OMS 入口網站轉換到 Azure 入口網站的常見問題 | Microsoft Docs
description: Log Analytics 使用者從 OMS 入口網站轉換到 Azure 入口網站的常見問題解答。
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
ms.date: 09/25/2018
ms.author: bwren
ms.component: ''
ms.openlocfilehash: d1ad629ff5c6909256baa94573d045a8c2c09fb5
ms.sourcegitcommit: ad08b2db50d63c8f550575d2e7bb9a0852efb12f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/26/2018
ms.locfileid: "47223826"
---
# <a name="common-questions-for-transition-from-oms-portal-to-azure-portal-for-log-analytics-users"></a>Log Analytics 使用者從 OMS 入口網站轉換到 Azure 入口網站的常見問題
Log Analytics 一開始會使用其本身的入口網站 (稱為 OMS 入口網站) 來管理其設定和分析收集到的資料。  這個入口網站中的所有功能都已移至 Azure 入口網站，並將繼續進行開發。

本文將解答使用者進行這項轉換時的常見問題。  如果您曾在 OMS 入口網站中使用 Log Analytics，則可以在這裡找到解答，了解如何在 Azure 入口網站中執行相同的工作。

## <a name="do-i-need-to-migrate-anything"></a>我需要移轉任何項目嗎？
否。 Log Analytics 本身沒有進行任何變更，所以沒要任何項目需要移轉。 唯一變更的項目是用來存取它的介面。 事實上，您現在可以使用 Azure 入口網站來存取和您目前在 OMS 入口網站中使用的相同工作區、解決方案、檢視和記錄搜尋。

## <a name="where-do-i-find-log-analytics-in-azure"></a>在哪裡可以找到 Azure 中的 Log Analytics？
在 [https://portal.azure.com](https://portal.azure.com) 上登入 Azure 入口網站。  按一下 [所有服務]，然後在資源清單中鍵入 **Log Analytics**。 選取 [Log Analytics]，然後選取您的工作區。 工作區的 [摘要] 頁面隨即出現。

![Log Analytics 工作區](media/log-analytics-new-portal/log-analytics.png)

## <a name="how-do-i-manage-permissions"></a>如何管理權限？
如果您沒有 Azure 入口網站中 Log Analytics 工作區的存取權，則需要使用 [Azure 角色型存取](../active-directory/role-based-access-control-configure.md)來設定您的權限。 如需管理工作區權限的詳細資料，請參閱[管理工作區](../log-analytics/log-analytics-manage-access.md#manage-accounts-and-users)。 如需管理警示權限的資訊，請參閱[開始使用 Azure 監視器的角色、權限和安全性](../monitoring-and-diagnostics/monitoring-roles-permissions-security.md)。

## <a name="how-do-i-create-a-new-workspace"></a>如何建立新工作區？ 
從 Azure 入口網站的工作區清單中，按一下工作區清單中的 [新增]。  如需完整詳細資料，請參閱[在 Azure 入口網站中建立 Log Analytics 工作區](../log-analytics/log-analytics-quick-create-workspace.md)。

![概觀分頁](media/log-analytics-new-portal/new-workspace.png)

## <a name="where-is-my-overview-page"></a>概觀頁面在何處？
OMS 入口網站的主畫面會顯示下列磚：工作區中已安裝的所有管理解決方案，以及任何您已建立的自訂檢視。 Azure 入口網站中也會有相同的檢視。 從工作區中選取 [工作區摘要]。

![概觀分頁](media/log-analytics-new-portal/overview.png)

## <a name="how-do-i-open-log-search-and-view-designer"></a>如何開啟記錄搜尋和檢視設計工具？
[記錄搜尋] 和 [檢視設計工具] 都位於 Azure 入口網站的主頁面及工作區的左側功能表中 (緊接在 [概觀] 旁邊)。

## <a name="where-do-i-find-settings"></a>在哪裡可以找到設定？
OMS 入口網站之 [設定] 區段中的許多設定，都位於 Azure 入口網站工作區的 [進階設定] 功能表中。

![進階設定](media/log-analytics-new-portal/advanced-settings.png)

下列各節提供如何存取 OMS 入口網站的 [設定] 區段先前提供之設定的完整清單。

### <a name="accounts"></a>帳戶 
帳戶設定在 Azure 入口網站中的不同地方進行管理，如下表所述。

| OMS 入口網站中的設定 | Azure 入口網站中的對等項目 |
|:---|:---|
| 自動化帳戶 | 工作區的 [自動化帳戶] 功能表。 |
| Azure 訂用帳戶與數據方案 | 工作區的 [定價層] 功能表。 |
| 管理使用者 | 使用 Azure 角色型存取來[管理工作區的權限](#how-do-i-manage-permissions)。 |
| 工作區資訊 | 工作區的 [OMS 工作區] 功能表的可用資訊。 |

### <a name="alerts"></a>警示
根據 Log Analytics 查詢的警示規則現在是以[統一的警示體驗](#how-do-i-create-and-manage-alerts)進行管理。 

### <a name="computer-groups"></a>電腦群組
在工作區的 [進階設定] 功能表中管理電腦群組。 

### <a name="connected-sources"></a>連線的來源
在工作區的 [進階設定] 功能表中管理大部分的 [連線的來源] 設定。 下表提供這個功能表每個區段的詳細資料。

| OMS 入口網站中的設定 | Azure 入口網站中的對等項目 |
|:---|:---|
| Windows 伺服器   | 工作區的 [進階設定] 功能表。 |
| Linux 伺服器   | 工作區的 [進階設定] 功能表。 |
| Azure 儲存體     | 工作區的 [進階設定] 功能表。 |
| System Center     | 工作區的 [進階設定] 功能表。 |
| Office 365        | 如需設定詳細資料，請參閱 [Office 365 管理解決方案的文件](../operations-management-suite/oms-solution-office-365.md)。 |
| Windows 遙測 | 解決方案的 [設定] 功能表。 如需設定的詳細資料，請參閱 [Azure 入口網站中的 Windows 分析](/windows/deployment/update/windows-analytics-azure-portal)。 |
| ITSM 連接器    | 如需使用 Log Analytics 與 ITSM 服務連線的指示，請參閱[將 ITSM 產品/服務與 IT 服務管理連接器連線](../log-analytics/log-analytics-itsmc-connections.md)。 |

### <a name="data"></a>資料
在工作區的 [進階設定] 功能表中管理大部分的 [資料] 設定。 下表提供這個功能表每個區段的詳細資料。

| OMS 入口網站中的設定 | Azure 入口網站中的對等項目 |
|:---|:---|
| Windows 事件記錄檔           | 工作區的 [進階設定] 功能表。 |
| Windows 效能計數器 | 工作區的 [進階設定] 功能表。 |
| Linux 效能計數器   | 工作區的 [進階設定] 功能表。 |
| IIS 記錄檔                     | 工作區的 [進階設定] 功能表。 |
| 自訂欄位                | 工作區的 [進階設定] 功能表。 |
| 自訂記錄                  | 工作區的 [進階設定] 功能表。 |
| syslog                       | 工作區的 [進階設定] 功能表。 |
| Application Insights         | 由於 Log Analytics 和 Application Insights 共用相同的資料引擎，現在已淘汰此解決方案。  |
| Windows 檔案追蹤        | Azure 自動化中的 [變更追蹤] 功能表。 如需詳細資料，請參閱[使用變更追蹤解決方案來追蹤環境中的變更](../automation/automation-change-tracking.md)。 |
| Windows 登錄追蹤        | Azure 自動化中的 [變更追蹤] 功能表。 如需詳細資料，請參閱[使用變更追蹤解決方案來追蹤環境中的變更](../automation/automation-change-tracking.md)。 |
| Linux 檔案追蹤          | Azure 自動化中的 [變更追蹤] 功能表。 如需詳細資料，請參閱[使用變更追蹤解決方案來追蹤環境中的變更](../automation/automation-change-tracking.md)。 |

### <a name="solutions"></a>解決方案
在工作區的 [解決方案] 功能表中管理解決方案。 

## <a name="how-do-i-install-and-remove-management-solutions"></a>如何安裝及移除管理解決方案？
在 OMS 入口網站中，您可從 [方案庫] 安裝管理解決方案，並從 [設定] 移除這些解決方案。 在 Azure 入口網站中，請從 Azure Marketplace [ 安裝管理解決方案](../monitoring/monitoring-solutions.md#install-a-management-solution)， 並從已安裝的解決方案清單中[移除解決方案](../monitoring/monitoring-solutions.md#remove-a-management-solution)。

## <a name="how-do-i-create-and-manage-alerts"></a>如何建立及管理警示？
根據 Log Analytics 查詢的警示規則現在是以[統一的警示體驗](../monitoring-and-diagnostics/monitor-alerts-unified-usage.md)進行管理。 如需在 Azure 入口網站中設定和使用警示的詳細資料，請參閱[如何將警示從 Log Analytics 延伸到 Azure 警示](../monitoring-and-diagnostics/monitoring-alerts-extend-tool.md)。

## <a name="how-do-i-access-my-dashboards"></a>如何存取我的儀表板？
已淘汰 Log Analytics 中的[儀表板](../log-analytics/log-analytics-dashboards.md)。  您可以使用[檢視設計工具](../log-analytics/log-analytics-view-designer.md)將 Log Analytics 中的資料視覺化，此檢視設計工具具有額外的功能，可將查詢和檢視釘選到 Azure 的儀表板中。

## <a name="how-do-i-check-my-usage"></a>如何查看我的使用量？
現在，您可以藉由選取工作區中的 [使用量和估計成本]，輕鬆地檢視及管理您的 Log Analytics 使用量和成本。

![使用量和估計成本](media/log-analytics-new-portal/usage.png)


## <a name="can-i-still-use-the-classic-portal"></a>仍然可以使用傳統入口網站嗎？
在有限的時間內，您仍然可以使用您自己的工作區名稱，透過此 URL 存取入口網站： https://\<您的工作區名稱\>.portal.mms.microsoft.com。 但還是建議您使用 Azure 入口網站，並透過 LAUpgradeFeedback@microsoft.com 提供對任何重大問題的意見反應。

## <a name="next-steps"></a>後續步驟

- 使用 Azure 入口網站[尋找並安裝管理解決方案](../monitoring/monitoring-solutions.md)。
- 了解 [Azure 入口網站中的記錄搜尋](log-analytics-log-search-portals.md)。