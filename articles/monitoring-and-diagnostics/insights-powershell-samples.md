---
title: Azure 監視器 PowerShell 快速入門範例
description: 使用 PowerShell 存取 Azure 監視器的功能，例如自動調整、警示、webhook 和搜尋活動記錄檔。
author: rboucher
services: azure-monitor
ms.service: azure-monitor
ms.topic: conceptual
ms.date: 2/14/2018
ms.author: robb
ms.component: ''
ms.openlocfilehash: 5566b169b3d4503746d12789addf4eb682d16733
ms.sourcegitcommit: 74941e0d60dbfd5ab44395e1867b2171c4944dbe
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/15/2018
ms.locfileid: "49321609"
---
# <a name="azure-monitor-powershell-quick-start-samples"></a>Azure 監視器 PowerShell 快速入門範例
本文說明可協助您存取 Azure 監視器 功能的範例 PowerShell 命令。

> [!NOTE]
> 自 2016 年 9 月 25 日起，「Azure 監視器」是以前所謂「Azure Insights」的新名稱。 不過，命名空間及下列命令中仍包含 "insights" 一字。
> 
> 

## <a name="set-up-powershell"></a>設定 PowerShell
設定要在電腦上執行的 PowerShell (如果您還未設定)。 如需詳細資訊，請參閱[如何安裝及設定 PowerShell](/powershell/azure/overview)。

## <a name="examples-in-this-article"></a>本文中的範例
本文中的範例將說明如何使用Azure 監視器 Cmdlet。 您也可以在 [Azure 監視器 Cmdlet](https://docs.microsoft.com/powershell/module/azurerm.insights) 檢閱整個 Azure 監視器 PowerShell Cmdlet 清單。

## <a name="sign-in-and-use-subscriptions"></a>登入和使用訂用帳戶
首先，登入您的 Azure 訂用帳戶。

```PowerShell
Connect-AzureRmAccount
```

您會看到登入畫面。 一旦登入之後，就會顯示您的帳戶、TenantID 和預設的訂用帳戶識別碼。 所有 Azure Cmdlet 都會在您的預設訂用帳戶內容中運作。 若要檢視您具有存取權的訂用帳戶的清單，請使用下列命令：

```PowerShell
Get-AzureRmSubscription
```

若要將使用中的內容變更為不同的訂用帳戶，請使用下列命令：

```PowerShell
Set-AzureRmContext -SubscriptionId <subscriptionid>
```


## <a name="retrieve-activity-log-for-a-subscription"></a>擷取訂用帳戶的活動記錄檔
使用 `Get-AzureRmLog` Cmdlet。  以下是一些常見的範例。

取得此時間/日期迄今的記錄檔項目︰

```PowerShell
Get-AzureRmLog -StartTime 2016-03-01T10:30
```

取得某個時間/日期範圍間的記錄檔項目︰

```PowerShell
Get-AzureRmLog -StartTime 2015-01-01T10:30 -EndTime 2015-01-01T11:30
```

取得特定資源群組中的記錄檔項目︰

```PowerShell
Get-AzureRmLog -ResourceGroup 'myrg1'
```

從特定的資源提供者取得某個時間/日期範圍間的記錄檔項目︰

```PowerShell
Get-AzureRmLog -ResourceProvider 'Microsoft.Web' -StartTime 2015-01-01T10:30 -EndTime 2015-01-01T11:30
```

取得包含特定呼叫者的所有記錄檔項目︰

```PowerShell
Get-AzureRmLog -Caller 'myname@company.com'
```

下列命令會擷取活動記錄檔中的最後 1000 個事件︰

```PowerShell
Get-AzureRmLog -MaxEvents 1000
```

`Get-AzureRmLog` 支援其他許多參數。 如需詳細資訊，請參閱 `Get-AzureRmLog` 參考。

> [!NOTE]
> `Get-AzureRmLog` 只提供 15 天的歷程記錄。 使用 **-MaxEvents** 參數可讓您查詢超過 15 天的前 N 個事件。 若要存取 15 天前的事件，請使用 REST API 或 SDK (使用 SDK 的 C# 範例)。 如果您未包含 **StartTime**，則預設值是 **EndTime** 減去一小時。 如果您未包含 **EndTime**，則預設值是目前的時間。 所有時間都是採用 UTC 格式。
> 
> 

## <a name="retrieve-alerts-history"></a>擷取警示歷程記錄
若要檢視所有警示事件，您可以使用下列範例查詢 Azure Resource Manager 記錄檔。

```PowerShell
Get-AzureRmLog -Caller "Microsoft.Insights/alertRules" -DetailedOutput -StartTime 2015-03-01
```

若要檢視特定警示規則的歷程記錄，您可以使用 `Get-AzureRmAlertHistory` Cmdlet，傳入警示規則的資源識別碼。

```PowerShell
Get-AzureRmAlertHistory -ResourceId /subscriptions/s1/resourceGroups/rg1/providers/microsoft.insights/alertrules/myalert -StartTime 2016-03-1 -Status Activated
```

`Get-AzureRmAlertHistory` Cmdlet 支援多各種參數。 如需詳細資訊，請參閱 [Get AlertHistory](https://msdn.microsoft.com/library/mt282453.aspx)。

## <a name="retrieve-information-on-alert-rules"></a>擷取警示規則的相關資訊
下列所有命令都會處理一個名為 "montest" 的資源群組。

檢視警示規則的所有屬性︰

```PowerShell
Get-AzureRmAlertRule -Name simpletestCPU -ResourceGroup montest -DetailedOutput
```

擷取資源群組上的所有警示︰

```PowerShell
Get-AzureRmAlertRule -ResourceGroup montest
```

擷取針對目標資源設定的所有警示規則。 例如，在 VM 上設定的所有警示規則。

```PowerShell
Get-AzureRmAlertRule -ResourceGroup montest -TargetResourceId /subscriptions/s1/resourceGroups/montest/providers/Microsoft.Compute/virtualMachines/testconfig
```

`Get-AzureRmAlertRule` 支援其他參數。 如需詳細資訊，請參閱 [Get AlertRule](https://msdn.microsoft.com/library/mt282459.aspx) 。

## <a name="create-metric-alerts"></a>建立計量警示
您可以使用 `Add-AlertRule` Cmdlet 建立、更新或停用警示規則。

您可以分別使用 `New-AzureRmAlertRuleEmail` 和 `New-AzureRmAlertRuleWebhook` 建立電子郵件和 Webhook 屬性。 在警示規則 Cmdlet 中，將這些屬性當做動作，指派給警示規則的 [動作] 屬性。

下表描述使用計量建立警示所使用的參數和值。

| 參數 | value |
| --- | --- |
| 名稱 |simpletestdiskwrite |
| 此警示規則的位置 |美國東部 |
| ResourceGroup |montest |
| TargetResourceId |/subscriptions/s1/resourceGroups/montest/providers/Microsoft.Compute/virtualMachines/testconfig |
| 所建立警示的 MetricName |\PhysicalDisk(_Total)\Disk Writes/sec。請參閱 `Get-MetricDefinitions` Cmdlet 以了解如何擷取確切的計量名稱 |
| operator |GreaterThan |
| 臨界值 (此計量的計數/秒） |1 |
| WindowSize (hh:mm:ss 格式) |00:05:00 |
| 彙總工具 (在此情況為計量的統計資料，其使用平均計數) |平均值 |
| 自訂電子郵件 (字串陣列) |'foo@example.com','bar@example.com' |
| 傳送電子郵件給擁有者、參與者和讀者 |-SendToServiceOwners |

建立電子郵件動作

```PowerShell
$actionEmail = New-AzureRmAlertRuleEmail -CustomEmail myname@company.com
```

建立 Webhook 動作

```PowerShell
$actionWebhook = New-AzureRmAlertRuleWebhook -ServiceUri https://example.com?token=mytoken
```

在傳統 VM 上建立 CPU 百分比計量的警示規則

```PowerShell
Add-AzureRmMetricAlertRule -Name vmcpu_gt_1 -Location "East US" -ResourceGroup myrg1 -TargetResourceId /subscriptions/s1/resourceGroups/myrg1/providers/Microsoft.ClassicCompute/virtualMachines/my_vm1 -MetricName "Percentage CPU" -Operator GreaterThan -Threshold 1 -WindowSize 00:05:00 -TimeAggregationOperator Average -Actions $actionEmail, $actionWebhook -Description "alert on CPU > 1%"
```

擷取警示規則

```PowerShell
Get-AzureRmAlertRule -Name vmcpu_gt_1 -ResourceGroup myrg1 -DetailedOutput
```

如果指定屬性的警示規則已存在，Add alert Cmdlet 也會更新規則。 若要停用警示規則，請加上參數 **-DisableRule**。

## <a name="get-a-list-of-available-metrics-for-alerts"></a>取得可用警示計量的清單
您可以使用 `Get-AzureRmMetricDefinition` Cmdlet 檢視特定資源的所有計量的清單。

```PowerShell
Get-AzureRmMetricDefinition -ResourceId <resource_id>
```

下列範例會產生包含計量名稱及其單位的資料表。

```PowerShell
Get-AzureRmMetricDefinition -ResourceId <resource_id> | Format-Table -Property Name,Unit
```

`Get-AzureRmMetricDefinition` 的可用選項完整清單可在 [Get MetricDefinitions](https://msdn.microsoft.com/library/mt282458.aspx)取得。

## <a name="create-and-manage-activity-log-alerts"></a>建立和管理活動記錄警示
您可以使用 `Set-AzureRmActivityLogAlert` Cmdlet 來設定活動記錄警示。 活動記錄警示需要您先將您的條件定義為條件的字典，然後建立會使用這些條件的警示。

```PowerShell

$condition1 = New-AzureRmActivityLogAlertCondition -Field 'category' -Equal 'Administrative'
$condition2 = New-AzureRmActivityLogAlertCondition -Field 'operationName' -Equal 'Microsoft.Compute/virtualMachines/write'
$additionalWebhookProperties = New-Object "System.Collections.Generic.Dictionary``2[System.String,System.String]"
$additionalWebhookProperties.Add('customProperty', 'someValue')
$actionGrp1 = New-AzureRmActionGroup -ActionGroupId '/subscriptions/<subid>/providers/Microsoft.Insights/actiongr1' -WebhookProperty $additionalWebhookProperties
Set-AzureRmActivityLogAlert -Location 'Global' -Name 'alert on VM create' -ResourceGroupName 'myResourceGroup' -Scope '/subscriptions/<subid>' -Action $actionGrp1 -Condition $condition1, $condition2

```

其他的 Webhook 屬性是選擇性的。 您可以使用 `Get-AzureRmActivityLogAlert` 來取回活動記錄警示的內容。

## <a name="create-and-manage-autoscale-settings"></a>建立和管理自動調整設定
Web 應用程式、VM、雲端服務或虛擬機器擴展集之類的資源只能設定一個自動調整設定。
不過，每個自動調整設定都可以有多個設定檔。 例如，一個用於以效能為基礎的調整設定檔，以及一個用於以排程為基礎的設定檔。 每個設定檔都可以設定多個規則。 如需有關自動調整的詳細資訊，請參閱 [如何自動調整應用程式](../cloud-services/cloud-services-how-to-scale-portal.md)。

以下是要使用的步驟：

1. 建立規則。
2. 建立將您先前所建立的規則對應到設定檔的設定檔。
3. 選擇性︰設定 Webhook 和電子郵件的屬性，以建立自動調整的通知。
4. 藉由對應您在先前步驟中建立的設定檔和通知，使用目標資源上的名稱建立自動調整設定。

下列範例將示範如何使用 CPU 使用率計量，為 Windows 作業系統的虛擬機器擴展集建立自動調整設定。

首先，建立相應放大的規則，讓執行個體計數增加。

```PowerShell
$rule1 = New-AzureRmAutoscaleRule -MetricName "Percentage CPU" -MetricResourceId /subscriptions/s1/resourceGroups/big2/providers/Microsoft.Compute/virtualMachineScaleSets/big2 -Operator GreaterThan -MetricStatistic Average -Threshold 60 -TimeGrain 00:01:00 -TimeWindow 00:10:00 -ScaleActionCooldown 00:10:00 -ScaleActionDirection Increase -ScaleActionValue 1
```        

接著，建立相應縮小的規則，讓執行個體計數減少。

```PowerShell
$rule2 = New-AzureRmAutoscaleRule -MetricName "Percentage CPU" -MetricResourceId /subscriptions/s1/resourceGroups/big2/providers/Microsoft.Compute/virtualMachineScaleSets/big2 -Operator GreaterThan -MetricStatistic Average -Threshold 30 -TimeGrain 00:01:00 -TimeWindow 00:10:00 -ScaleActionCooldown 00:10:00 -ScaleActionDirection Decrease -ScaleActionValue 1
```

然後，建立規則的設定檔。

```PowerShell
$profile1 = New-AzureRmAutoscaleProfile -DefaultCapacity 2 -MaximumCapacity 10 -MinimumCapacity 2 -Rules $rule1,$rule2 -Name "My_Profile"
```

建立 Webhook 屬性。

```PowerShell
$webhook_scale = New-AzureRmAutoscaleWebhook -ServiceUri "https://example.com?mytoken=mytokenvalue"
```

建立自動調整設定的通知屬性，包括您先前建立的電子郵件與 Webhook。

```PowerShell
$notification1= New-AzureRmAutoscaleNotification -CustomEmails ashwink@microsoft.com -SendEmailToSubscriptionAdministrators SendEmailToSubscriptionCoAdministrators -Webhooks $webhook_scale
```

最後，建立自動調整設定來新增您先前建立的設定檔。 

```PowerShell
Add-AzureRmAutoscaleSetting -Location "East US" -Name "MyScaleVMSSSetting" -ResourceGroup big2 -TargetResourceId /subscriptions/s1/resourceGroups/big2/providers/Microsoft.Compute/virtualMachineScaleSets/big2 -AutoscaleProfiles $profile1 -Notifications $notification1
```

如需有關管理自動調整設定的詳細資訊，請參閱 [Get-AutoscaleSetting](https://msdn.microsoft.com/library/mt282461.aspx)。

## <a name="autoscale-history"></a>自動調整歷程記錄
下列範例示範如何檢視最新的自動調整和警示的事件。 使用活動記錄檔搜尋檢視自動調整歷程記錄。

```PowerShell
Get-AzureRmLog -Caller "Microsoft.Insights/autoscaleSettings" -DetailedOutput -StartTime 2015-03-01
```

您可以使用 `Get-AzureRmAutoScaleHistory` Cmdlet 擷取自動調整歷程記錄。

```PowerShell
Get-AzureRmAutoScaleHistory -ResourceId /subscriptions/s1/resourceGroups/myrg1/providers/microsoft.insights/autoscalesettings/myScaleSetting -StartTime 2016-03-15 -DetailedOutput
```

如需詳細資訊，請參閱 [Get-AutoscaleHistory](https://msdn.microsoft.com/library/mt282464.aspx)。

### <a name="view-details-for-an-autoscale-setting"></a>檢視自動調整設定的詳細資料
您可以使用 `Get-Autoscalesetting` Cmdlet 擷取有關自動調整設定的詳細資訊。

下列範例會顯示資源群組 'myrg1' 中所有自動調整設定的詳細資料。

```PowerShell
Get-AzureRmAutoscalesetting -ResourceGroup myrg1 -DetailedOutput
```

下列範例會顯示資源群組 'myrg1' 中所有自動調整設定的詳細資料，尤其是名為 'MyScaleVMSSSetting' 的自動調整設定。

```PowerShell
Get-AzureRmAutoscalesetting -ResourceGroup myrg1 -Name MyScaleVMSSSetting -DetailedOutput
```

### <a name="remove-an-autoscale-setting"></a>移除自動調整設定
您可以使用 `Remove-Autoscalesetting` Cmdlet 刪除自動調整設定。

```PowerShell
Remove-AzureRmAutoscalesetting -ResourceGroup myrg1 -Name MyScaleVMSSSetting
```

## <a name="manage-log-profiles-for-activity-log"></a>管理活動記錄檔的記錄檔設定檔
您可以建立*記錄檔設定檔*，並將資料從活動記錄檔匯出至儲存體帳戶，而且您可以為其設定資料保留期。 或者，您也可以將資料串流至事件中樞。 此功能目前處於預覽狀態，而且每個訂用帳戶只能建立一個記錄檔設定檔。 您可以使用下列 Cmdlet 搭配您目前的訂用帳戶來建立和管理記錄檔設定檔。 您也可以選擇特定的訂用帳戶。 PowerShell 預設為目前的訂用帳戶，但是您隨時可以使用 `Set-AzureRmContext`加以變更。 您可以設定活動記錄檔，將資料路由至任何儲存體帳戶或該訂用帳戶內的事件中樞。 資料會以 JSON 格式的 Blob 檔案寫入。

### <a name="get-a-log-profile"></a>取得記錄檔設定檔
若要擷取現有的記錄檔設定檔，請使用 `Get-AzureRmLogProfile` Cmdlet。

### <a name="add-a-log-profile-without-data-retention"></a>新增沒有資料保留期的記錄檔設定檔
```PowerShell
Add-AzureRmLogProfile -Name my_log_profile_s1 -StorageAccountId /subscriptions/s1/resourceGroups/myrg1/providers/Microsoft.Storage/storageAccounts/my_storage -Locations global,westus,eastus,northeurope,westeurope,eastasia,southeastasia,japaneast,japanwest,northcentralus,southcentralus,eastus2,centralus,australiaeast,australiasoutheast,brazilsouth,centralindia,southindia,westindia
```

### <a name="remove-a-log-profile"></a>移除記錄檔設定檔
```PowerShell
Remove-AzureRmLogProfile -name my_log_profile_s1
```

### <a name="add-a-log-profile-with-data-retention"></a>新增包含資料保留期的記錄檔設定檔
您可以使用保留資料的天數，以正整數指定 **-RetentionInDays** 屬性。

```PowerShell
Add-AzureRmLogProfile -Name my_log_profile_s1 -StorageAccountId /subscriptions/s1/resourceGroups/myrg1/providers/Microsoft.Storage/storageAccounts/my_storage -Locations global,westus,eastus,northeurope,westeurope,eastasia,southeastasia,japaneast,japanwest,northcentralus,southcentralus,eastus2,centralus,australiaeast,australiasoutheast,brazilsouth,centralindia,southindia,westindia -RetentionInDays 90
```

### <a name="add-log-profile-with-retention-and-eventhub"></a>新增包含保留期與 EventHub 的記錄檔設定檔
除了將資料路由至儲存體帳戶之外，您也可以將它串流到事件中樞。 在此預覽版本中，儲存體帳戶設定是強制性的，但事件中樞設定則是選擇性的。

```PowerShell
Add-AzureRmLogProfile -Name my_log_profile_s1 -StorageAccountId /subscriptions/s1/resourceGroups/myrg1/providers/Microsoft.Storage/storageAccounts/my_storage -serviceBusRuleId /subscriptions/s1/resourceGroups/Default-ServiceBus-EastUS/providers/Microsoft.ServiceBus/namespaces/mytestSB/authorizationrules/RootManageSharedAccessKey -Locations global,westus,eastus,northeurope,westeurope,eastasia,southeastasia,japaneast,japanwest,northcentralus,southcentralus,eastus2,centralus,australiaeast,australiasoutheast,brazilsouth,centralindia,southindia,westindia -RetentionInDays 90
```

## <a name="configure-diagnostics-logs"></a>設定診斷記錄檔
許多 Azure 服務提供額外的記錄檔和遙測，可執行下列一或多個作業： 
 - 設定將資料儲存在 Azure 儲存體帳戶
 - 傳送至事件中樞
 - 傳送至 Log Analytics 工作區。 

作業只能在資源層級執行。 儲存體帳戶或事件中樞應該存在於進行診斷設定所在目標資源的相同區域中。

### <a name="get-diagnostic-setting"></a>取得診斷設定
```PowerShell
Get-AzureRmDiagnosticSetting -ResourceId /subscriptions/s1/resourceGroups/myrg1/providers/Microsoft.Logic/workflows/andy0315logicapp
```

停用診斷設定

```PowerShell
Set-AzureRmDiagnosticSetting -ResourceId /subscriptions/s1/resourceGroups/myrg1/providers/Microsoft.Logic/workflows/andy0315logicapp -StorageAccountId /subscriptions/s1/resourceGroups/Default-Storage-WestUS/providers/Microsoft.Storage/storageAccounts/mystorageaccount -Enable $false
```

啟用不含保留期的診斷設定

```PowerShell
Set-AzureRmDiagnosticSetting -ResourceId /subscriptions/s1/resourceGroups/myrg1/providers/Microsoft.Logic/workflows/andy0315logicapp -StorageAccountId /subscriptions/s1/resourceGroups/Default-Storage-WestUS/providers/Microsoft.Storage/storageAccounts/mystorageaccount -Enable $true
```

啟用含保留期的診斷設定

```PowerShell
Set-AzureRmDiagnosticSetting -ResourceId /subscriptions/s1/resourceGroups/myrg1/providers/Microsoft.Logic/workflows/andy0315logicapp -StorageAccountId /subscriptions/s1/resourceGroups/Default-Storage-WestUS/providers/Microsoft.Storage/storageAccounts/mystorageaccount -Enable $true -RetentionEnabled $true -RetentionInDays 90
```

針對特定的記錄檔類別，啟用含保留期的診斷設定

```PowerShell
Set-AzureRmDiagnosticSetting -ResourceId /subscriptions/s1/resourceGroups/insights-integration/providers/Microsoft.Network/networkSecurityGroups/viruela1 -StorageAccountId /subscriptions/s1/resourceGroups/myrg1/providers/Microsoft.Storage/storageAccounts/sakteststorage -Categories NetworkSecurityGroupEvent -Enable $true -RetentionEnabled $true -RetentionInDays 90
```

啟用事件中樞的診斷設定

```PowerShell
Set-AzureRmDiagnosticSetting -ResourceId /subscriptions/s1/resourceGroups/insights-integration/providers/Microsoft.Network/networkSecurityGroups/viruela1 -serviceBusRuleId /subscriptions/s1/resourceGroups/Default-ServiceBus-EastUS/providers/Microsoft.ServiceBus/namespaces/mytestSB/authorizationrules/RootManageSharedAccessKey -Enable $true
```

啟用 Log Analytics 的診斷設定

```PowerShell
Set-AzureRmDiagnosticSetting -ResourceId /subscriptions/s1/resourceGroups/insights-integration/providers/Microsoft.Network/networkSecurityGroups/viruela1 -WorkspaceId /subscriptions/s1/resourceGroups/insights-integration/providers/providers/microsoft.operationalinsights/workspaces/myWorkspace -Enabled $true

```

請注意，工作區識別碼屬性會接受工作區的資源識別碼。 您可以使用下列命令取得 Log Analytics 工作區的資源識別碼：

```PowerShell
(Get-AzureRmOperationalInsightsWorkspace).ResourceId

```

這些命令可以合併以將資料傳送至多個目的地。
