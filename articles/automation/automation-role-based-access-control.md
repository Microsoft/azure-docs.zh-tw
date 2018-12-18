---
title: Azure 自動化中的角色型存取控制
description: 角色型存取控制 (RBAC) 可以啟用對 Azure 資源的存取權管理。 本文說明如何在 Azure 自動化中設定 RBAC。
keywords: 自動化 rbac, 角色型存取控制, azure rbac
services: automation
ms.service: automation
ms.component: shared-capabilities
author: georgewallace
ms.author: gwallace
ms.date: 05/17/2018
ms.topic: conceptual
manager: carmonm
ms.openlocfilehash: fe35f26fe73661901fa85da65e353fe66adf5d47
ms.sourcegitcommit: cc4fdd6f0f12b44c244abc7f6bc4b181a2d05302
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/25/2018
ms.locfileid: "47094212"
---
# <a name="role-based-access-control-in-azure-automation"></a>Azure 自動化中的角色型存取控制

角色型存取控制 (RBAC) 可以啟用對 Azure 資源的存取權管理。 您可以使用 [RBAC](../role-based-access-control/overview.md) 來區隔小組內的職責，僅授與使用者、群組和應用程式執行作業所需的存取權範圍。 使用 Azure 入口網站、Azure 命令列工具或 Azure 管理 API 的使用者，可以獲授與角色型存取權。

## <a name="roles-in-automation-accounts"></a>自動化帳戶中的角色

在 Azure 自動化中，於自動化帳戶範圍將適當的 RBAC 角色指派給使用者、群組和應用程式，即可授與存取權。 以下是自動化帳戶支援的內建角色：

| **角色** | **說明** |
|:--- |:--- |
| 擁有者 |「擁有者」角色可讓您存取「自動化」帳戶內的所有資源和動作，包括提供存取權給其他使用者、群組及應用程式來管理「自動化」帳戶。 |
| 參與者 |參與者角色可讓您管理各個項目，修改其他使用者的自動化帳戶存取權限除外。 |
| 讀取者 |讀取者角色可讓您檢視自動化帳戶中的所有資源，但無法進行任何變更。 |
| 自動化運算子 |自動化操作員角色可讓您檢視 Runbook 的名稱和屬性，以及在自動化帳戶中建立及管理所有 Runbook 的作業。 如果您想要保護認證資產和 Runbook 等自動化帳戶資源不被檢視或修改，但仍然允許組織的成員來執行這些 Runbook，這個角色會有所幫助。 |
|自動化作業運算子|「自動化作業操作員」角色可讓您建立和管理自動化帳戶中的所有 Runbook 作業。|
|自動化 Runbook 運算子|「自動化 Runbook 操作員」角色可讓您檢視 Runbook 的名稱和屬性。|
| Log Analytics 參與者 | 「Log Analytics 參與者」角色可讓您讀取所有監視資料和編輯監視設定。 編輯監視設定包括將 VM 延伸模組新增至 VM、讀取儲存體帳戶金鑰以便能夠設定從 Azure 儲存體收集記錄、建立及設定「自動化」帳戶、新增解決方案，以及設定所有 Azure 資源上的 Azure 診斷。|
| Log Analytics 讀者 | 「Log Analytics 讀者」角色可讓您檢視和搜尋所有監視資料，以及檢視監視設定。 這包括檢視所有 Azure 資源上的 Azure 診斷設定。 |
| 監視參與者 | 「監視參與者」角色可讓您讀取所有監視資料和更新監視設定。|
| 監視讀取器 | 「監視讀者」角色可讓您讀取所有監視資料。 |
| 使用者存取系統管理員 |使用者存取系統管理員角色可讓您管理 Azure 自動化帳戶的使用者存取。 |

## <a name="role-permissions"></a>角色權限

下表描述賦予每個角色的特定權限。 這可以包括會授與權限的 Actions，以及會限制權限的 NotActions。

### <a name="owner"></a>擁有者

「擁有者」可以管理所有項目，包括存取權。 下表說明針對此角色授與的權限：

|動作|說明|
|---|---|
|Microsoft.Automation/automationAccounts/|建立和管理所有類型的資源。|

### <a name="contributor"></a>參與者

「參與者」可以管理存取權以外的所有項目。 下表說明針對此角色授與和拒絕的權限：

|**動作**  |**說明**  |
|---------|---------|
|Microsoft.Automation/automationAccounts/|建立和管理所有類型的資源|
|**無法執行的動作**||
|Microsoft.Authorization/*/Delete| 刪除角色和角色指派。       |
|Microsoft.Authorization/*/Write     |  建立角色和角色指派。       |
|Microsoft.Authorization/elevateAccess/Action    | 無法建立「使用者存取系統管理員」。       |

### <a name="reader"></a>讀取者

「讀取者」角色可以檢視「自動化」帳戶中的所有資源，但無法進行任何變更。

|**動作**  |**說明**  |
|---------|---------|
|Microsoft.Automation/automationAccounts/read|檢視「自動化」帳戶中的所有資源。 |

### <a name="automation-operator"></a>自動化運算子

自動化操作員可在自動化帳戶中建立和管理作業，以及讀取所有 Runbook 的名稱和屬性。  注意：如果您想要控制操作員存取個別 Runbook，請不要設定此角色，而是改為搭配使用「自動化作業操作員」和「自動化 Runbook 操作員」角色。 下表說明針對此角色授與的權限：

|**動作**  |**說明**  |
|---------|---------|
|Microsoft.Authorization/*/read|讀取授權。|
|Microsoft.Automation/automationAccounts/hybridRunbookWorkerGroups/read|讀取混合式 Runbook 背景工作角色資源。|
|Microsoft.Automation/automationAccounts/jobs/read|列出 Runbook 的作業。|
|Microsoft.Automation/automationAccounts/jobs/resume/action|繼續執行暫停的作業。|
|Microsoft.Automation/automationAccounts/jobs/stop/action|取消進行中的作業。|
|Microsoft.Automation/automationAccounts/jobs/streams/read|讀取「作業串流」和「輸出」。|
|Microsoft.Automation/automationAccounts/jobs/output/read|取得作業的輸出。|
|Microsoft.Automation/automationAccounts/jobs/suspend/action|暫停進行中的作業。|
|Microsoft.Automation/automationAccounts/jobs/write|建立作業。|
|Microsoft.Automation/automationAccounts/jobSchedules/read|取得 Azure 自動化作業排程。|
|Microsoft.Automation/automationAccounts/jobSchedules/write|建立 Azure 自動化作業排程。|
|Microsoft.Automation/automationAccounts/linkedWorkspace/read|取得連結至自動化帳戶的工作區。|
|Microsoft.Automation/automationAccounts/read|取得 Azure 自動化帳戶。|
|Microsoft.Automation/automationAccounts/runbooks/read|取得 Azure 自動化 Runbook。|
|Microsoft.Automation/automationAccounts/schedules/read|取得 Azure 自動化排程資產。|
|Microsoft.Automation/automationAccounts/schedules/write|建立或更新 Azure 自動化排程資產。|
|Microsoft.Resources/subscriptions/resourceGroups/read      |讀取角色和角色指派。         |
|Microsoft.Resources/deployments/*      |建立和管理資源群組部署。         |
|Microsoft.Insights/alertRules/*      | 建立和管理警示規則。        |
|Microsoft.Support/* |建立和管理支援票證。|

### <a name="automation-job-operator"></a>自動化作業運算子

授與「自動化作業操作員」角色時會在「自動化」帳戶範圍內授與。 這可讓操作員有權建立和管理帳戶中所有 Runbook 作業。 下表說明針對此角色授與的權限：

|**動作**  |**說明**  |
|---------|---------|
|Microsoft.Authorization/*/read|讀取授權。|
|Microsoft.Automation/automationAccounts/jobs/read|列出 Runbook 的作業。|
|Microsoft.Automation/automationAccounts/jobs/resume/action|繼續執行暫停的作業。|
|Microsoft.Automation/automationAccounts/jobs/stop/action|取消進行中的作業。|
|Microsoft.Automation/automationAccounts/jobs/streams/read|讀取「作業串流」和「輸出」。|
|Microsoft.Automation/automationAccounts/jobs/suspend/action|暫停進行中的作業。|
|Microsoft.Automation/automationAccounts/jobs/write|建立作業。|
|Microsoft.Resources/subscriptions/resourceGroups/read      |  讀取角色和角色指派。       |
|Microsoft.Resources/deployments/*      |建立和管理資源群組部署。         |
|Microsoft.Insights/alertRules/*      | 建立和管理警示規則。        |
|Microsoft.Support/* |建立和管理支援票證。|

### <a name="automation-runbook-operator"></a>自動化 Runbook 運算子

授與「自動化 Runbook 運算子」角色時，會在 Runbook 範圍授與。 「自動化 Runbook 操作員」可檢視 Runbook 的名稱和屬性。  此角色與「自動化作業操作員」角色結合，可讓操作員也建立及管理 Runbook 的作業。 下表說明針對此角色授與的權限：

|**動作**  |**說明**  |
|---------|---------|
|Microsoft.Automation/automationAccounts/runbooks/read     | 列出 Runbook。        |
|Microsoft.Authorization/*/read      | 讀取授權。        |
|Microsoft.Resources/subscriptions/resourceGroups/read      |讀取角色和角色指派。         |
|Microsoft.Resources/deployments/*      | 建立和管理資源群組部署。         |
|Microsoft.Insights/alertRules/*      | 建立和管理警示規則。        |
|Microsoft.Support/*      | 建立和管理支援票證。        |

### <a name="log-analytics-contributor"></a>Log Analytics 參與者

「Log Analytics 參與者」角色可以讀取所有監視資料和編輯監視設定。 編輯監視設定包括將 VM 延伸模組新增至 VM、讀取儲存體帳戶金鑰以便能夠設定從「Azure 儲存體」收集記錄、建立及設定「自動化」帳戶、新增解決方案，以及設定所有 Azure 資源上的 Azure 診斷。 下表說明針對此角色授與的權限：

|**動作**  |**說明**  |
|---------|---------|
|*/read|讀取密碼以外的所有類型的資源。|
|Microsoft.Automation/automationAccounts/*|管理自動化帳戶。|
|Microsoft.ClassicCompute/virtualMachines/extensions/*|建立和管理虛擬機器延伸模組。|
|Microsoft.ClassicStorage/storageAccounts/listKeys/action|列出傳統儲存體帳戶金鑰。|
|Microsoft.Compute/virtualMachines/extensions/*|建立和管理傳統虛擬機器延伸模組。|
|Microsoft.Insights/alertRules/*|讀取/寫入/刪除警示規則。|
|Microsoft.Insights/diagnosticSettings/*|讀取/寫入/刪除診斷設定。|
|Microsoft.OperationalInsights/*|管理 Log Analytics。|
|Microsoft.OperationsManagement/*|管理工作區中的解決方案。|
|Microsoft.Resources/deployments/*|建立和管理資源群組部署。|
|Microsoft.Resources/subscriptions/resourcegroups/deployments/*|建立和管理資源群組部署。|
|Microsoft.Storage/storageAccounts/listKeys/action|列出儲存體帳戶金鑰。|
|Microsoft.Support/*|建立和管理支援票證。|

### <a name="log-analytics-reader"></a>Log Analytics 讀者

「Log Analytics 讀者」可以檢視和搜尋所有監視資料，以及檢視監視設定，包括檢視所有 Azure 資源上的 Azure 診斷設定。 下表說明針對此角色授與或拒絕的權限：

|**動作**  |**說明**  |
|---------|---------|
|*/read|讀取密碼以外的所有類型的資源。|
|Microsoft.OperationalInsights/workspaces/analytics/query/action|管理 Log Analytics 中的查詢。|
|Microsoft.OperationalInsights/workspaces/search/action|搜尋 Log Analytics 資料。|
|Microsoft.Support/*|建立和管理支援票證。|
|**無法執行的動作**| |
|Microsoft.OperationalInsights/workspaces/sharedKeys/read|無法讀取共用存取金鑰。|

### <a name="monitoring-contributor"></a>監視參與者

「監視參與者」可以讀取所有監視資料和更新監視設定。 下表說明針對此角色授與的權限：

|**動作**  |**說明**  |
|---------|---------|
|*/read|讀取密碼以外的所有類型的資源。|
|Microsoft.AlertsManagement/alerts/*|管理「警示」。|
|Microsoft.AlertsManagement/alertsSummary/*|管理「警示」儀表板。|
|Microsoft.Insights/AlertRules/*|管理警示規則。|
|Microsoft.Insights/components/*|管理 Application Insights 元件。|
|Microsoft.Insights/DiagnosticSettings/*|管理診斷設定。|
|Microsoft.Insights/eventtypes/*|列出訂用帳戶中的活動記錄檔事件 (管理事件)。 此權限適用於以程式設計方式存取和入口網站存取活動記錄檔。|
|Microsoft.Insights/LogDefinitions/*|此為使用者需要透過入口網站存取活動記錄檔時所需的權限。 列出活動記錄檔中的記錄檔分類。|
|Microsoft.Insights/MetricDefinitions/*|讀取度量定義 (可用資源的度量類型清單)。|
|Microsoft.Insights/Metrics/*|讀取資源的度量。|
|Microsoft.Insights/Register/Action|註冊 Microsoft Insights 提供者。|
|Microsoft.Insights/webtests/*|管理 Application Insights Web 測試。|
|Microsoft.OperationalInsights/workspaces/intelligencepacks/*|管理 Log Analytics 解決方案套件。|
|Microsoft.OperationalInsights/workspaces/savedSearches/*|管理 Log Analytics 已儲存的搜尋。|
|Microsoft.OperationalInsights/workspaces/search/action|搜尋 Log Analytics 工作區。|
|Microsoft.OperationalInsights/workspaces/sharedKeys/action|列出 Log Analytics 工作區的金鑰。|
|Microsoft.OperationalInsights/workspaces/storageinsightconfigs/*|管理 Log Analytics 儲存體深入解析設定。|
|Microsoft.Support/*|建立和管理支援票證。|
|Microsoft.WorkloadMonitor/workloads/*|管理「工作負載」。|

### <a name="monitoring-reader"></a>監視讀取器

「監視讀者」可以讀取所有監視資料。 下表說明針對此角色授與的權限：

|**動作**  |**說明**  |
|---------|---------|
|*/read|讀取密碼以外的所有類型的資源。|
|Microsoft.OperationalInsights/workspaces/search/action|搜尋 Log Analytics 工作區。|
|Microsoft.Support/*|建立和管理支援票證|

### <a name="user-access-administrator"></a>使用者存取系統管理員

「使用者存取系統管理員」可以管理使用者對 Azure 資源的存取權。 下表說明針對此角色授與的權限：

|**動作**  |**說明**  |
|---------|---------|
|*/read|讀取所有資源|
|Microsoft.Authorization/*|管理授權|
|Microsoft.Support/*|建立和管理支援票證|

## <a name="onboarding"></a>登入

下表說明針對變更追蹤或更新管理解決方案，讓虛擬機器上線所需的最基本必要權限。

### <a name="onboarding-from-a-virtual-machine"></a>從虛擬機器上線

|**Action**  |**權限**  |**最基本範圍**  |
|---------|---------|---------|
|寫入新的部署      | Microsoft.Resources/deployments/*          |訂用帳戶          |
|寫入新的資源群組      | Microsoft.Resources/subscriptions/resourceGroups/write        | 訂用帳戶          |
|建立新的預設「工作區」      | Microsoft.OperationalInsights/workspaces/write         | 資源群組         |
|建立新的「帳戶」      |  Microsoft.Automation/automationAccounts/write        |資源群組         |
|連結工作區和帳戶      |Microsoft.OperationalInsights/workspaces/write</br>Microsoft.Automation/automationAccounts/read|工作區</br>自動化帳戶
|建立解決方案      | Microsoft.OperationalInsights/workspaces/intelligencepacks/write |資源群組          |
|建立 MMA 延伸模組      | Microsoft.Compute/virtualMachines/write         | 虛擬機器         |
|建立已儲存的搜尋      | Microsoft.OperationalInsights/workspaces/write          | 工作區         |
|建立範圍設定      | Microsoft.OperationalInsights/workspaces/write          | 工作區         |
|將解決方案連結至範圍設定      | Microsoft.OperationalInsights/workspaces/intelligencepacks/write         | 解決方法         |
|上線狀態檢查 - 讀取工作區      | Microsoft.OperationalInsights/workspaces/read         | 工作區         |
|上線狀態檢查 - 讀取帳戶的已連結工作區屬性     | Microsoft.Automation/automationAccounts/read      | 自動化帳戶        |
|上線狀態檢查 - 讀取解決方案      | Microsoft.OperationalInsights/workspaces/intelligencepacks/read          | 解決方法         |
|上線狀態檢查 - 讀取 VM      | Microsoft.Compute/virtualMachines/read         | 虛擬機器         |
|上線狀態檢查 - 讀取帳戶      | Microsoft.Automation/automationAccounts/read  |  自動化帳戶   |

### <a name="onboarding-from-automation-account"></a>從自動化帳戶上線

|**Action**  |**權限** |**最基本範圍**  |
|---------|---------|---------|
|建立新的部署     | Microsoft.Resources/deployments/*        | 訂用帳戶         |
|建立新的資源群組     | Microsoft.Resources/subscriptions/resourceGroups/write         | 訂用帳戶        |
|AutomationOnboarding 刀鋒視窗 - 建立新的工作區     |Microsoft.OperationalInsights/workspaces/write           | 資源群組        |
|AutomationOnboarding 刀鋒視窗 - 讀取已連結的工作區     | Microsoft.Automation/automationAccounts/read        | 自動化帳戶       |
|AutomationOnboarding 刀鋒視窗 - 讀取解決方案     | Microsoft.OperationalInsights/workspaces/intelligencepacks/read         | 解決方法        |
|AutomationOnboarding 刀鋒視窗 - 讀取工作區     | Microsoft.OperationalInsights/workspaces/intelligencepacks/read        | 工作區        |
|為工作區和帳戶建立連結     | Microsoft.OperationalInsights/workspaces/write        | 工作區        |
|寫入 Shoebox 的帳戶      | Microsoft.Automation/automationAccounts/write        | 帳戶        |
|建立解決方案      | Microsoft.OperationalInsights/workspaces/intelligencepacks/write        | 資源群組         |
|建立/編輯已儲存的搜尋     | Microsoft.OperationalInsights/workspaces/write        | 工作區        |
|建立/編輯範圍設定     | Microsoft.OperationalInsights/workspaces/write        | 工作區        |
|將解決方案連結至範圍設定      | Microsoft.OperationalInsights/workspaces/intelligencepacks/write         | 解決方法         |
|**步驟 2 - 讓多個 VM 上線**     |         |         |
|VMOnboarding 刀鋒視窗 - 建立 MMA 延伸模組     | Microsoft.Compute/virtualMachines/write           | 虛擬機器        |
|建立/編輯已儲存的搜尋     | Microsoft.OperationalInsights/workspaces/write           | 工作區        |
|建立/編輯範圍設定  | Microsoft.OperationalInsights/workspaces/write   | 工作區|

## <a name="update-management"></a>更新管理

更新管理的觸角會遍及多項服務來提供其服務。 下表說明管理更新管理部署所需的權限：

|**Resource**  |**角色**  |**範圍**  |
|---------|---------|---------|
|自動化帳戶     | Log Analytics 參與者       | 自動化帳戶        |
|自動化帳戶    | 虛擬機器參與者        | 帳戶的「資源群組」        |
|Log Analytics 工作區     | Log Analytics 參與者| Log Analytics 工作區        |
|Log Analytics 工作區 |Log Analytics 讀者| 訂用帳戶|
|解決方法     |Log Analytics 參與者         | 解決方法|
|虛擬機器     | 虛擬機器參與者        | 虛擬機器        |

## <a name="configure-rbac-for-your-automation-account"></a>為您的自動化帳戶設定 RBAC

下一節將說明如何透過[入口網站](#configure-rbac-using-the-azure-portal)和 [PowerShell](#configure-rbac-using-powershell) 在您的自動化帳戶上設定 RBAC

### <a name="configure-rbac-using-the-azure-portal"></a>使用 Azure 入口網站設定 RBAC

1. 登入 [Azure 入口網站](https://portal.azure.com/)，並從 [自動化帳戶] 頁面開啟您的自動化帳戶。
2. 按一下左上角的 [存取控制 (IAM)] 控制項。 這會開啟 [存取控制 (IAM)] 頁面，您可以在其中新增新使用者、群組及應用程式來管理您的「自動化」帳戶，並檢視可以為「自動化」帳戶設定的現有角色。

   ![[存取] 按鈕](media/automation-role-based-access-control/automation-01-access-button.png)

#### <a name="add-a-new-user-and-assign-a-role"></a>加入新使用者並指派角色

1. 從 [存取控制 (IAM)] 頁面中，按一下 [+ 新增] 以開啟 [新增權限] 頁面，您可以在其中新增使用者、群組或應用程式，並將角色指派給他們。

2. 從可用角色的清單中選取角色。 您可以選擇「自動化」帳戶支援的任何可用內建角色，或已定義的任何自訂角色。

3. 在 [選取] 欄位中輸入您想要授與權限之使用者的使用者名稱。 從清單中選取使用者，然後按一下 [儲存]。

   ![新增使用者](media/automation-role-based-access-control/automation-04-add-users.png)

   現在您應該會看到使用者已新增至 [使用者] 頁面，並已獲指派所選取的角色

   ![列出使用者](media/automation-role-based-access-control/automation-05-list-users.png)

   您也可以從 [角色] 頁面指派角色給使用者。
4. 從 [存取控制 (IAM)] 頁面按一下 [角色]，以開啟 [角色] 頁面。 從這個頁面中，您可以檢視角色的名稱、指派給該角色的使用者和群組數目。

    ![從 [使用者] 頁面指派角色](media/automation-role-based-access-control/automation-06-assign-role-from-users-blade.png)

   > [!NOTE]
   > 設定角色型存取控制時，只能在「自動化」帳戶範圍設定，而無法在「自動化」帳戶以下的任何資源設定。

#### <a name="remove-a-user"></a>移除使用者

您可以針對未管理「自動化」帳戶的使用者，或不再為組織工作的使用者，移除存取權限。 以下是移除使用者的步驟：

1. 從 [存取控制 (IAM)] 頁面中，選取要移除的使用者，然後按一下 [移除]。
2. 按一下指派詳細資料窗格上的 [移除] 按鈕。
3. 按一下 [是]  以確認移除。

   ![移除使用者](media/automation-role-based-access-control/automation-08-remove-users.png)

### <a name="configure-rbac-using-powershell"></a>使用 PowerShell 設定 RBAC

使用下列 [Azure PowerShell Cmdlet](../role-based-access-control/role-assignments-powershell.md) 也可以將角色型存取設定到「自動化」帳戶：

[Get-AzureRmRoleDefinition](https://msdn.microsoft.com/library/mt603792.aspx) 會列出 Azure Active Directory 中可用的所有 RBAC 角色。 您可以使用這個命令搭配 [名稱] 屬性，列出特定角色可以執行的所有動作。

```azurepowershell-interactive
Get-AzureRmRoleDefinition -Name 'Automation Operator'
```

以下是範例輸出：

```azurepowershell
Name             : Automation Operator
Id               : d3881f73-407a-4167-8283-e981cbba0404
IsCustom         : False
Description      : Automation Operators are able to start, stop, suspend, and resume jobs
Actions          : {Microsoft.Authorization/*/read, Microsoft.Automation/automationAccounts/jobs/read, Microsoft.Automation/automationAccounts/jobs/resume/action,
                   Microsoft.Automation/automationAccounts/jobs/stop/action...}
NotActions       : {}
AssignableScopes : {/}
```

[Get-AzureRmRoleAssignment](https://msdn.microsoft.com/library/mt619413.aspx) 會列出在指定範圍的 Azure AD RBAC 角色指派。 如果沒有任何參數，此命令會傳回在訂用帳戶下所做的所有角色指派。 使用 **ExpandPrincipalGroups** 參數，列出指定使用者以及使用者為其成員之群組的存取權指派。
    **範例：** 使用下列命令來列出自動化帳戶中的所有使用者以及其角色。

```azurepowershell-interactive
Get-AzureRMRoleAssignment -scope '/subscriptions/<SubscriptionID>/resourcegroups/<Resource Group Name>/Providers/Microsoft.Automation/automationAccounts/<Automation account name>'
```

以下是範例輸出：

```powershell
RoleAssignmentId   : /subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/myResourceGroup/providers/Microsoft.Automation/automationAccounts/myAutomationAccount/provid
                     ers/Microsoft.Authorization/roleAssignments/cc594d39-ac10-46c4-9505-f182a355c41f
Scope              : /subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/myResourceGroup/providers/Microsoft.Automation/automationAccounts/myAutomationAccount
DisplayName        : admin@contoso.com
SignInName         : admin@contoso.com
RoleDefinitionName : Automation Operator
RoleDefinitionId   : d3881f73-407a-4167-8283-e981cbba0404
ObjectId           : 15f26a47-812d-489a-8197-3d4853558347
ObjectType         : User
```

[New-AzureRmRoleAssignment](https://msdn.microsoft.com/library/mt603580.aspx) 可將特定範圍的存取權指派給使用者、群組及應用程式。
    **範例：** 請使用下列命令來為「自動化帳戶」範圍內的使用者指派「自動化運算子」角色。

```azurepowershell-interactive
New-AzureRmRoleAssignment -SignInName <sign-in Id of a user you wish to grant access> -RoleDefinitionName 'Automation operator' -Scope '/subscriptions/<SubscriptionID>/resourcegroups/<Resource Group Name>/Providers/Microsoft.Automation/automationAccounts/<Automation account name>'
```

以下是範例輸出：

```azurepowershell
RoleAssignmentId   : /subscriptions/00000000-0000-0000-0000-000000000000/resourcegroups/myResourceGroup/Providers/Microsoft.Automation/automationAccounts/myAutomationAccount/provid
                     ers/Microsoft.Authorization/roleAssignments/25377770-561e-4496-8b4f-7cba1d6fa346
Scope              : /subscriptions/00000000-0000-0000-0000-000000000000/resourcegroups/myResourceGroup/Providers/Microsoft.Automation/automationAccounts/myAutomationAccount
DisplayName        : admin@contoso.com
SignInName         : admin@contoso.com
RoleDefinitionName : Automation Operator
RoleDefinitionId   : d3881f73-407a-4167-8283-e981cbba0404
ObjectId           : f5ecbe87-1181-43d2-88d5-a8f5e9d8014e
ObjectType         : User
```

使用 [Remove-AzureRmRoleAssignment](https://msdn.microsoft.com/library/mt603781.aspx) 從特定範圍移除所指定使用者、群組或應用程式的存取權。
    **範例：** 請使用下列命令，從「自動化」帳戶範圍內的「自動化操作員」角色中移除使用者。

```azurepowershell-interactive
Remove-AzureRmRoleAssignment -SignInName <sign-in Id of a user you wish to remove> -RoleDefinitionName 'Automation Operator' -Scope '/subscriptions/<SubscriptionID>/resourcegroups/<Resource Group Name>/Providers/Microsoft.Automation/automationAccounts/<Automation account name>'
```

在前述範例中，請以您的帳戶詳細資料取代 **sign in Id** \(登入識別碼\)**subscription Id** \(訂用帳戶識別碼\)**resource group name**\(資源群組名稱\) 及 **Automation account name** \(自動化帳戶名稱\)。 當系統提示您確認時選擇 [是]  ，然後再繼續移除使用者角色指派。

### <a name="user-experience-for-automation-operator-role---automation-account"></a>自動化操作員角色的使用者體驗 - 自動化帳戶

當獲指派自動化帳戶範圍之「自動化操作員」角色的使用者檢視獲指派的自動化帳戶時，只能檢視在該自動化帳戶中建立之 Runbook、Runbook 作業及排程的清單，但無法檢視其定義。 他們可以啟動、停止、暫停、繼續或排程 Runbook 作業。 使用者無法存取其他「自動化」資源，例如設定、混合式背景工作角色群組或 DSC 節點。

![無法存取資源](media/automation-role-based-access-control/automation-10-no-access-to-resources.png)

## <a name="configure-rbac-for-runbooks"></a>為 Runbook 設定 RBAC

Azure 自動化可讓您將 RBAC 指派給特定 Runbook。 若要達成目的，請執行下列指令碼，將使用者新增至特定 Runbook。 下列指令碼可由「自動化帳戶管理員」或「租用戶系統管理員」執行。

```azurepowershell-interactive
$rgName = "<Resource Group Name>" # Resource Group name for the Automation Account
$automationAccountName ="<Automation Account Name>" # Name of the Automation Account
$rbName = "<Name of Runbook>" # Name of the runbook
$userId = "<User ObjectId>" # Azure Active Directory (AAD) user's ObjectId from the directory

# Gets the Automation Account resource
$aa = Get-AzureRmResource -ResourceGroupName $rgName -ResourceType "Microsoft.Automation/automationAccounts" -ResourceName $automationAccountName

# Get the Runbook resource
$rb = Get-AzureRmResource -ResourceGroupName $rgName -ResourceType "Microsoft.Automation/automationAccounts/runbooks" -ResourceName "$automationAccountName/$rbName"

# The Automation Job Operator role only needs to be ran once per user.
New-AzureRmRoleAssignment -ObjectId $userId -RoleDefinitionName "Automation Job Operator" -Scope $aa.ResourceId

# Adds the user to the Automation Runbook Operator role to the Runbook scope
New-AzureRmRoleAssignment -ObjectId $userId -RoleDefinitionName "Automation Runbook Operator" -Scope $rb.ResourceId
```

執行後，使用者就可以登入 Azure 入口網站並檢視**所有資源**。 他們會在清單中看到以**自動化 Runbook 操作員**身分新增的 Runbook 。

![入口網站中的 Runbook RBAC](./media/automation-role-based-access-control/runbook-rbac.png)

### <a name="user-experience-for-automation-operator-role---runbook"></a>自動化操作員角色的使用者體驗 - Runbook

當獲指派 Runbook 範圍之「自動化操作員」角色的使用者檢視獲指派的 Runbook 時，只能啟動 Runbook 並檢視 Runbook 作業。

![只有啟動的存取權](media/automation-role-based-access-control/automation-only-start.png)

## <a name="next-steps"></a>後續步驟

* 如需針對 Azure 自動化設定 RBAC 的不同方式詳細資訊，請參閱 [使用 Azure PowerShell 管理 RBAC](../role-based-access-control/role-assignments-powershell.md)。
* 如需以不同方式啟動 Runbook 的詳細資訊，請參閱 [啟動 Runbook](automation-starting-a-runbook.md)
* 如需不同 Runbook 類型的詳細資訊，請參閱 [Azure 自動化 Runbook 類型](automation-runbook-types.md)
