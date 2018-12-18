---
title: 使用 PowerShell 建立和設定 Log Analytics 工作區 | Microsoft Docs
description: Log Analytics 會使用來自內部部署或雲端基礎結構中之伺服器的資料。 您可以在 Azure 診斷產生電腦資料時，從 Azure 儲存體加以收集。
services: log-analytics
documentationcenter: ''
author: richrundmsft
manager: jochan
editor: ''
ms.assetid: 3b9b7ade-3374-4596-afb1-51b695f481c2
ms.service: log-analytics
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: powershell
ms.topic: conceptual
ms.date: 11/21/2016
ms.author: richrund
ms.component: ''
ms.openlocfilehash: f30114797c7ba62fde555487d3202de5edf48ecb
ms.sourcegitcommit: 5b8d9dc7c50a26d8f085a10c7281683ea2da9c10
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/26/2018
ms.locfileid: "47182343"
---
# <a name="manage-log-analytics-using-powershell"></a>使用 PowerShell 管理 Log Analytics
您可以從命令列或在指令碼中，使用 [Log Analytics PowerShell Cmdlet](https://docs.microsoft.com/powershell/module/azurerm.operationalinsights/) 在 Log Analytics 中執行各種功能。  您可以使用 PowerShell 執行的工作範例包括︰

* 建立工作區
* 新增或移除方案
* 匯入和匯出已儲存的搜尋
* 建立電腦群組
* 從已安裝 Windows 代理程式的電腦啟用 IIS 記錄檔收集功能
* 從 Linux 和 Windows 電腦收集效能計數器
* 在 Linux 電腦上收集 syslog 事件 
* 從 Windows 事件記錄檔收集事件
* 收集自訂事件記錄檔
* 將 Log Analytics 代理程式加入至 Azure 虛擬機器
* 設定 Log Analytics 將 Azure 診斷所收集的資料編製索引

本文提供兩個程式碼範例，示範您可以從 PowerShell 執行的一些功能。  關於其他功能，您可以參考 [Log Analytics PowerShell Cmdlet 參考文件](https://docs.microsoft.com/powershell/module/azurerm.operationalinsights/) 。

> [!NOTE]
> Log Analytics 在以前稱為 Operational Insights，這也是 Cmdlet 中使用此名稱的原因。
> 
> 

## <a name="prerequisites"></a>必要條件
這些範例可與 AzureRm.OperationalInsights 模組的 2.3.0 版或更新版本搭配運作。


## <a name="create-and-configure-a-log-analytics-workspace"></a>建立及設定 Log Analytics 工作區
下列指令碼範例說明如何：

1. 建立工作區
2. 列出可用的方案
3. 將方案加入至工作區
4. 匯入已儲存的搜尋
5. 匯出已儲存的搜尋
6. 建立電腦群組
7. 從已安裝 Windows 代理程式的電腦啟用 IIS 記錄檔收集功能
8. 從 Linux 電腦收集邏輯磁碟效能計數器 (% Used Inodes; Free Megabytes; % Used Space; Disk Transfers/sec; Disk Reads/sec; Disk Writes/sec)
9. 從 Linux 電腦收集 syslog 事件
10. 從 Windows 電腦的應用程式事件記錄檔收集錯誤和警告事件
11. 從 Windows 電腦收集記憶體可用 Mb 效能計數器
12. 收集自訂記錄檔 

```

$ResourceGroup = "oms-example"
$WorkspaceName = "log-analytics-" + (Get-Random -Maximum 99999) # workspace names need to be unique - Get-Random helps with this for the example code
$Location = "westeurope"

# List of solutions to enable
$Solutions = "Security", "Updates", "SQLAssessment"

# Saved Searches to import
$ExportedSearches = @"
[
    {
        "Category":  "My Saved Searches",
        "DisplayName":  "WAD Events (All)",
        "Query":  "Type=Event SourceSystem:AzureStorage ",
        "Version":  1
    },
    {        
        "Category":  "My Saved Searches",
        "DisplayName":  "Current Disk Queue Length",
        "Query":  "Type=Perf ObjectName=LogicalDisk InstanceName=\"C:\" CounterName=\"Current Disk Queue Length\"",
        "Version":  1
    }
]
"@ | ConvertFrom-Json

# Custom Log to collect
$CustomLog = @"
{
    "customLogName": "sampleCustomLog1", 
    "description": "Example custom log datasource", 
    "inputs": [
        { 
            "location": { 
            "fileSystemLocations": { 
                "windowsFileTypeLogPaths": [ "e:\\iis5\\*.log" ], 
                "linuxFileTypeLogPaths": [ "/var/logs" ] 
                } 
            }, 
        "recordDelimiter": { 
            "regexDelimiter": { 
                "pattern": "\\n", 
                "matchIndex": 0, 
                "matchIndexSpecified": true, 
                "numberedGroup": null 
                } 
            } 
        }
    ], 
    "extractions": [
        { 
            "extractionName": "TimeGenerated", 
            "extractionType": "DateTime", 
            "extractionProperties": { 
                "dateTimeExtraction": { 
                    "regex": null, 
                    "joinStringRegex": null 
                    } 
                } 
            }
        ] 
    }
"@

# Create the resource group if needed
try {
    Get-AzureRmResourceGroup -Name $ResourceGroup -ErrorAction Stop
} catch {
    New-AzureRmResourceGroup -Name $ResourceGroup -Location $Location
}

# Create the workspace
New-AzureRmOperationalInsightsWorkspace -Location $Location -Name $WorkspaceName -Sku Standard -ResourceGroupName $ResourceGroup

# List all solutions and their installation status
Get-AzureRmOperationalInsightsIntelligencePacks -ResourceGroupName $ResourceGroup -WorkspaceName $WorkspaceName

# Add solutions
foreach ($solution in $Solutions) {
    Set-AzureRmOperationalInsightsIntelligencePack -ResourceGroupName $ResourceGroup -WorkspaceName $WorkspaceName -IntelligencePackName $solution -Enabled $true
}

# List enabled solutions
(Get-AzureRmOperationalInsightsIntelligencePacks -ResourceGroupName $ResourceGroup -WorkspaceName $WorkspaceName).Where({($_.enabled -eq $true)})

# Import Saved Searches
foreach ($search in $ExportedSearches) {
    $id = $search.Category + "|" + $search.DisplayName
    New-AzureRmOperationalInsightsSavedSearch -ResourceGroupName $ResourceGroup -WorkspaceName $WorkspaceName -SavedSearchId $id -DisplayName $search.DisplayName -Category $search.Category -Query $search.Query -Version $search.Version
}

# Export Saved Searches
(Get-AzureRmOperationalInsightsSavedSearch -ResourceGroupName $ResourceGroup -WorkspaceName $WorkspaceName).Value.Properties | ConvertTo-Json 

# Create Computer Group based on a query
New-AzureRmOperationalInsightsComputerGroup -ResourceGroupName $ResourceGroup -WorkspaceName $WorkspaceName -SavedSearchId "My Web Servers" -DisplayName "Web Servers" -Category "My Saved Searches" -Query "Computer=""web*"" | distinct Computer" -Version 1

# Create a computer group based on names (up to 5000)
$computerGroup = """servername1.contoso.com"",""servername2.contoso.com"",""servername3.contoso.com"",""servername4.contoso.com"""
New-AzureRmOperationalInsightsComputerGroup -ResourceGroupName $ResourceGroup -WorkspaceName $WorkspaceName -SavedSearchId "My Named Servers" -DisplayName "Named Servers" -Category "My Saved Searches" -Query $computerGroup -Version 1

# Enable IIS Log Collection using agent
Enable-AzureRmOperationalInsightsIISLogCollection -ResourceGroupName $ResourceGroup -WorkspaceName $WorkspaceName

# Linux Perf
New-AzureRmOperationalInsightsLinuxPerformanceObjectDataSource -ResourceGroupName $ResourceGroup -WorkspaceName $WorkspaceName -ObjectName "Logical Disk" -InstanceName "*"  -CounterNames @("% Used Inodes", "Free Megabytes", "% Used Space", "Disk Transfers/sec", "Disk Reads/sec", "Disk Reads/sec", "Disk Writes/sec") -IntervalSeconds 20  -Name "Example Linux Disk Performance Counters"
Enable-AzureRmOperationalInsightsLinuxCustomLogCollection -ResourceGroupName $ResourceGroup -WorkspaceName $WorkspaceName

# Linux Syslog
New-AzureRmOperationalInsightsLinuxSyslogDataSource -ResourceGroupName $ResourceGroup -WorkspaceName $WorkspaceName -Facility "kern" -CollectEmergency -CollectAlert -CollectCritical -CollectError -CollectWarning -Name "Example kernal syslog collection"
Enable-AzureRmOperationalInsightsLinuxSyslogCollection -ResourceGroupName $ResourceGroup -WorkspaceName $WorkspaceName

# Windows Event
New-AzureRmOperationalInsightsWindowsEventDataSource -ResourceGroupName $ResourceGroup -WorkspaceName $WorkspaceName -EventLogName "Application" -CollectErrors -CollectWarnings -Name "Example Application Event Log"

# Windows Perf
New-AzureRmOperationalInsightsWindowsPerformanceCounterDataSource -ResourceGroupName $ResourceGroup -WorkspaceName $WorkspaceName -ObjectName "Memory" -InstanceName "*" -CounterName "Available MBytes" -IntervalSeconds 20 -Name "Example Windows Performance Counter"

# Custom Logs
New-AzureRmOperationalInsightsCustomLogDataSource -ResourceGroupName $ResourceGroup -WorkspaceName $WorkspaceName -CustomLogRawJson "$CustomLog" -Name "Example Custom Log Collection"

```

## <a name="configuring-log-analytics-to-index-azure-diagnostics"></a>設定 Log Analytics 來編製 Azure 診斷的索引
若要以無代理程式的方式監視 Azure 資源，資源需要啟用 Azure 診斷並將其設定為寫入至 Log Analytics 工作區。 此方法會將資料直接傳送到 Log Analytics，而且不需要將資料寫入儲存體帳戶。 支援的資源包括：

| 資源類型 | 記錄檔 | 度量 |
| --- | --- | --- |
| 應用程式閘道    | 是 | 是 |
| 自動化帳戶     | 是 | |
| Batch 帳戶          | 是 | 是 |
| Data Lake Analytics     | 是 | | 
| Data Lake Store         | 是 | |
| SQL 彈性集區        |     | 是 |
| 事件中樞命名空間     |     | 是 |
| IoT 中樞                |     | 是 |
| Key Vault               | 是 | |
| 負載平衡器          | 是 | |
| Logic Apps              | 是 | 是 |
| 網路安全性群組 | 是 | |
| Redis 快取             |     | 是 |
| 搜尋服務         | 是 | 是 |
| 服務匯流排命名空間   |     | 是 |
| SQL (v12)               |     | 是 |
| 網站               |     | 是 |
| Web 伺服器陣列        |     | 是 |

如需可用度量的詳細資訊，請參閱[支援 Azure 監視器的度量](../monitoring-and-diagnostics/monitoring-supported-metrics.md)。

如需可用記錄檔的詳細資訊，請參閱[支援的服務以及診斷記錄檔的結構描述](../monitoring-and-diagnostics/monitoring-diagnostic-logs-schema.md)。

```
$workspaceId = "/subscriptions/d2e37fee-1234-40b2-5678-0b2199de3b50/resourcegroups/oi-default-east-us/providers/microsoft.operationalinsights/workspaces/rollingbaskets"

$resourceId = "/SUBSCRIPTIONS/ec11ca60-1234-491e-5678-0ea07feae25c/RESOURCEGROUPS/DEMO/PROVIDERS/MICROSOFT.NETWORK/NETWORKSECURITYGROUPS/DEMO" 

Set-AzureRmDiagnosticSetting -ResourceId $resourceId -WorkspaceId $workspaceId -Enabled $true
```

您也可以使用前述 Cmdlet，來收集不同訂用帳戶中之資源的記錄檔。 因為您會提供建立記錄檔之資源和記錄檔所傳送至之工作區這兩個項目的識別碼，因此這個 Cmdlet 可跨訂用帳戶運作。


## <a name="configuring-log-analytics-to-index-azure-diagnostics-from-storage"></a>設定 Log Analytics 以對來自儲存體的 Azure 診斷編製索引
若要從傳統雲端服務或 Service Fabric 叢集的執行中執行個體內收集記錄檔資料，您必須先將資料寫入 Azure 儲存體。 然後，可將 Log Analytics 設定為從儲存體帳戶收集記錄檔。 支援的資源包括：

* 傳統雲端服務 (Web 和背景工作角色)
* Service Fabric 叢集

下列範例示範如何執行：

1. 列出現有的儲存體帳戶和 Log Analytics 檢索的資源來源位置
2. 建立組態以讀取儲存體帳戶
3. 更新新建立的組態以檢索其他位置的資料
4. 刪除新建立的組態

```
# validTables = "WADWindowsEventLogsTable", "LinuxsyslogVer2v0", "WADServiceFabric*EventTable", "WADETWEventTable" 
$workspace = (Get-AzureRmOperationalInsightsWorkspace).Where({$_.Name -eq "your workspace name"})

# Update these two lines with the storage account resource ID and the storage account key for the storage account you want to Log Analytics to  
$storageId = "/subscriptions/ec11ca60-1234-491e-5678-0ea07feae25c/resourceGroups/demo/providers/Microsoft.Storage/storageAccounts/wadv2storage"
$key = "abcd=="

# List existing insights
Get-AzureRmOperationalInsightsStorageInsight -ResourceGroupName $workspace.ResourceGroupName -WorkspaceName $workspace.Name

# Create a new insight
New-AzureRmOperationalInsightsStorageInsight -ResourceGroupName $workspace.ResourceGroupName -WorkspaceName $workspace.Name -Name "newinsight" -StorageAccountResourceId $storageId -StorageAccountKey $key -Tables @("WADWindowsEventLogsTable") -Containers @("wad-iis-logfiles")

# Update existing insight
Set-AzureRmOperationalInsightsStorageInsight -ResourceGroupName $workspace.ResourceGroupName -WorkspaceName $workspace.Name -Name "newinsight" -Tables @("WADWindowsEventLogsTable", "WADETWEventTable") -Containers @("wad-iis-logfiles")

# Remove the insight
Remove-AzureRmOperationalInsightsStorageInsight -ResourceGroupName $workspace.ResourceGroupName -WorkspaceName $workspace.Name -Name "newinsight" 

```

您也可以使用前述指令碼，來收集不同訂用帳戶中之儲存體帳戶的記錄檔。 因為您會提供儲存體帳戶資源識別碼和對應的存取金鑰，因此指令碼可跨訂用帳戶運作。 當您變更存取金鑰時，您需要更新儲存體深入解析使其擁有新的金鑰。


## <a name="next-steps"></a>後續步驟
* [檢閱 Log Analytics PowerShell Cmdlet](https://docs.microsoft.com/powershell/module/azurerm.operationalinsights/) 。

