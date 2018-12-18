---
title: 將監視和診斷新增到 Azure 虛擬機器 | Microsoft Docs
description: 使用 Azure Resource Manager 範本以建立具有 Azure 診斷擴充功能的新 Windows 虛擬機器。
services: virtual-machines-windows
documentationcenter: ''
author: sbtron
manager: jeconnoc
editor: ''
tags: azure-resource-manager
ms.assetid: 8cde8fe7-977b-43d2-be74-ad46dc946058
ms.service: virtual-machines-windows
ms.workload: infrastructure-services
ms.tgt_pltfrm: vm-windows
ms.devlang: na
ms.topic: article
ms.date: 05/31/2017
ms.author: saurabh
ms.custom: H1Hack27Feb2017
ms.openlocfilehash: 792a3401c483327eb7fb9fcd88039bc09025b3ef
ms.sourcegitcommit: d98d99567d0383bb8d7cbe2d767ec15ebf2daeb2
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 05/10/2018
ms.locfileid: "33944950"
---
# <a name="use-monitoring-and-diagnostics-with-a-windows-vm-and-azure-resource-manager-templates"></a>使用 Windows VM 和 Azure Resource Manager 範本的監視和診斷
Azure 診斷擴充功能會在以 Windows 為基礎的 Azure 虛擬機器上提供監視和診斷功能。 您可以將擴充功能納入為 Azure Resource Manager 範本的一部分，在虛擬機器上啟用這些功能。 請參閱 [使用 VM 延伸模組編寫 Azure 資源管理員範本](../windows/template-description.md#extensions) ，以取得將任何延伸模組納入為虛擬機器範本一部分的詳細資訊。 本文描述如何將 Azure 診斷延伸模組新增至 Windows 虛擬機器範本。  

## <a name="add-the-azure-diagnostics-extension-to-the-vm-resource-definition"></a>將 Azure 診斷延伸模組新增至 VM 資源定義
若要在 Windows 虛擬機器上啟用診斷擴充功能，您必須新增擴充功能作為 Resource Manager 範本中的 VM 資源。

針對簡單以資源管理員為基礎的虛擬機器，將延伸模組組態新增至虛擬機器的「資源」  陣列： 

```json
"resources": [
    {
        "name": "Microsoft.Insights.VMDiagnosticsSettings",
        "type": "extensions",
        "location": "[resourceGroup().location]",
        "apiVersion": "2015-06-15",
        "dependsOn": [
            "[concat('Microsoft.Compute/virtualMachines/', variables('vmName'))]"
        ],
        "tags": {
            "displayName": "AzureDiagnostics"
        },
        "properties": {
            "publisher": "Microsoft.Azure.Diagnostics",
            "type": "IaaSDiagnostics",
            "typeHandlerVersion": "1.5",
            "autoUpgradeMinorVersion": true,
            "settings": {
                "xmlCfg": "[base64(concat(variables('wadcfgxstart'), variables('wadmetricsresourceid'), variables('vmName'), variables('wadcfgxend')))]",
                "storageAccount": "[parameters('existingdiagnosticsStorageAccountName')]"
            },
            "protectedSettings": {
                "storageAccountName": "[parameters('existingdiagnosticsStorageAccountName')]",
                "storageAccountKey": "[listkeys(variables('accountid'), '2015-05-01-preview').key1]",
                "storageAccountEndPoint": "https://core.windows.net"
            }
        }
    }
]
```

另一個常見的慣例是在範本的根資源節點新增擴充功能組態，而不是在虛擬機器的資源節點底下定義。 使用這個方法時，您必須以「名稱」和「類型」值明確指定擴充功能與虛擬機器之間的階層式關係。 例如︰ 

```json
"name": "[concat(variables('vmName'),'Microsoft.Insights.VMDiagnosticsSettings')]",
"type": "Microsoft.Compute/virtualMachines/extensions",
```

延伸模組一律與虛擬機器相關聯，您可以直接在虛擬機器的資源節點底下定義，或是在基底層級定義並且使用階層式命名慣例，將其與虛擬機器產生關聯。

對於虛擬機器擴展集，延伸模組組態是在 *VirtualMachineProfile* 的 *extensionProfile* 屬性中指定。

*publisher* 屬性具有值 **Microsoft.Azure.Diagnostics**，*type* 屬性具有值 **IaaSDiagnostics**，唯一識別 Azure 診斷延伸模組。

*name* 屬性的值可用來參考資源群組中的延伸模組。 特別將其設為 **Microsoft.Insights.VMDiagnosticsSettings**，讓它可輕鬆地由 Azure 入口網站識別，確保監視圖表會在 Azure 入口網站中正確顯示。

*TypeHandlerVersion* 會指定您想要使用的延伸模組的版本。 將 autoUpgradeMinorVersion 次要版本設為 **true** 可確保您獲得可用的最新擴充功能次要版本。 強烈建議您一律將 *autoUpgradeMinorVersion* 設為永遠為 **true** ，讓您永遠可以使用具有所有新功能和錯誤修正的可用最新診斷延伸模組。 

*settings* 元素包含延伸模組的組態屬性，可以從延伸模組 (有時稱為公用組態) 設定和讀取。 xmlcfg 屬性包含以 xml 為基礎之組態的診斷記錄、效能計數器等等，這些項目是由診斷代理程式收集。 如需 xml 結構描述本身的詳細資訊，請參閱 [診斷組態結構描述](https://msdn.microsoft.com/library/azure/dn782207.aspx) 。 常見的作法是將實際的 xml 組態儲存為 Azure 資源管理員範本中的變數，然後再進行串連和 base64 編碼，以設定 *xmlcfg*的值。 請參閱 [診斷組態變數](#diagnostics-configuration-variables) 以深入了解如何在變數中儲存 xml。 StorageAccount 屬性會指定要向其傳輸診斷資料的儲存體帳戶名稱。 

*protectedSettings* (有時稱為私用組態) 中的屬性可以設定，但是無法在設定之後讀取。 protectedSettings 的唯寫本質讓它對於儲存像是儲存體帳戶 (寫入診斷資料的位置) 金鑰的密碼相當有用。    

## <a name="specifying-diagnostics-storage-account-as-parameters"></a>將診斷儲存體帳戶指定為參數
上述的診斷擴充功能 json 片段假設兩個參數 existingdiagnosticsStorageAccountName 和 existingdiagnosticsStorageResourceGroup，以指定將會儲存診斷資料的診斷儲存體帳戶。 將診斷儲存體帳戶指定為參數，可讓您輕鬆地跨不同環境變更診斷儲存體帳戶，例如，您可能需要使用不同診斷儲存體帳戶進行測試，並且使用另一個進行生產部署。  

```json
"existingdiagnosticsStorageAccountName": {
    "type": "string",
    "metadata": {
"description": "The name of an existing storage account to which diagnostics data is transfered."
    }        
},
"existingdiagnosticsStorageResourceGroup": {
    "type": "string",
    "metadata": {
"description": "The resource group for the storage account specified in existingdiagnosticsStorageAccountName"
      }
}
```

最佳做法是在不同於虛擬機器資源群組的其他資源群組中指定診斷儲存體帳戶。 資源群組可以視為是具有自己的存留期的部署單位，可以部署虛擬機器以及在新組態更新時重新部署，但是您可能想要跨這些虛擬機器部署繼續在相同的儲存體帳戶中儲存診斷資料。 在不同的資源中擁有儲存體帳戶可讓儲存體帳戶接受來自各種虛擬機器部署的資料，方便疑難排解各種版本之間的問題。

> [!NOTE]
> 如果您從 Visual Studio 建立 Windows 虛擬機器範本，預設儲存體帳戶可能會設定為使用相同的儲存體帳戶，這是虛擬機器 VHD 上傳的位置。 這是為了簡化 VM 的初始設定。 請重構範本以使用可作為參數傳入的不同儲存體帳戶。 
> 
> 

## <a name="diagnostics-configuration-variables"></a>診斷組態變數
上述的診斷延伸模組 json 程式碼片段會定義 *accountid* 變數，以簡化取得診斷儲存體的儲存體帳戶金鑰：   

```json
"accountid": "[concat('/subscriptions/', subscription().subscriptionId, '/resourceGroups/',parameters('existingdiagnosticsStorageResourceGroup'), '/providers/','Microsoft.Storage/storageAccounts/', parameters('existingdiagnosticsStorageAccountName'))]"
```

診斷延伸模組的 *xmlcfg* 屬性是使用串連在一起的多個變數進行定義。 這些變數值的格式為 xml，因此必須在設定 json 變數時正確逸出。

以下範例說明診斷組態 xml，它會收集標準系統層級效能計數器以及一些 Windows 事件記錄檔和診斷基礎結構記錄檔。 已正確逸出和格式化，因此組態可以直接貼到您的範本的變數區段。 請參閱 [診斷組態結構描述](https://msdn.microsoft.com/library/azure/dn782207.aspx) 以取得人類可讀取的組態 xml 的範例。

```json
"wadlogs": "<WadCfg> <DiagnosticMonitorConfiguration overallQuotaInMB=\"4096\" xmlns=\"http://schemas.microsoft.com/ServiceHosting/2010/10/DiagnosticsConfiguration\"> <DiagnosticInfrastructureLogs scheduledTransferLogLevelFilter=\"Error\"/> <WindowsEventLog scheduledTransferPeriod=\"PT1M\" > <DataSource name=\"Application!*[System[(Level = 1 or Level = 2)]]\" /> <DataSource name=\"Security!*[System[(Level = 1 or Level = 2)]]\" /> <DataSource name=\"System!*[System[(Level = 1 or Level = 2)]]\" /></WindowsEventLog>",
"wadperfcounters1": "<PerformanceCounters scheduledTransferPeriod=\"PT1M\"><PerformanceCounterConfiguration counterSpecifier=\"\\Processor(_Total)\\% Processor Time\" sampleRate=\"PT15S\" unit=\"Percent\"><annotation displayName=\"CPU utilization\" locale=\"en-us\"/></PerformanceCounterConfiguration><PerformanceCounterConfiguration counterSpecifier=\"\\Processor(_Total)\\% Privileged Time\" sampleRate=\"PT15S\" unit=\"Percent\"><annotation displayName=\"CPU privileged time\" locale=\"en-us\"/></PerformanceCounterConfiguration><PerformanceCounterConfiguration counterSpecifier=\"\\Processor(_Total)\\% User Time\" sampleRate=\"PT15S\" unit=\"Percent\"><annotation displayName=\"CPU user time\" locale=\"en-us\"/></PerformanceCounterConfiguration><PerformanceCounterConfiguration counterSpecifier=\"\\Processor Information(_Total)\\Processor Frequency\" sampleRate=\"PT15S\" unit=\"Count\"><annotation displayName=\"CPU frequency\" locale=\"en-us\"/></PerformanceCounterConfiguration><PerformanceCounterConfiguration counterSpecifier=\"\\System\\Processes\" sampleRate=\"PT15S\" unit=\"Count\"><annotation displayName=\"Processes\" locale=\"en-us\"/></PerformanceCounterConfiguration><PerformanceCounterConfiguration counterSpecifier=\"\\Process(_Total)\\Thread Count\" sampleRate=\"PT15S\" unit=\"Count\"><annotation displayName=\"Threads\" locale=\"en-us\"/></PerformanceCounterConfiguration><PerformanceCounterConfiguration counterSpecifier=\"\\Process(_Total)\\Handle Count\" sampleRate=\"PT15S\" unit=\"Count\"><annotation displayName=\"Handles\" locale=\"en-us\"/></PerformanceCounterConfiguration><PerformanceCounterConfiguration counterSpecifier=\"\\Memory\\% Committed Bytes In Use\" sampleRate=\"PT15S\" unit=\"Percent\"><annotation displayName=\"Memory usage\" locale=\"en-us\"/></PerformanceCounterConfiguration><PerformanceCounterConfiguration counterSpecifier=\"\\Memory\\Available Bytes\" sampleRate=\"PT15S\" unit=\"Bytes\"><annotation displayName=\"Memory available\" locale=\"en-us\"/></PerformanceCounterConfiguration><PerformanceCounterConfiguration counterSpecifier=\"\\Memory\\Committed Bytes\" sampleRate=\"PT15S\" unit=\"Bytes\"><annotation displayName=\"Memory committed\" locale=\"en-us\"/></PerformanceCounterConfiguration><PerformanceCounterConfiguration counterSpecifier=\"\\Memory\\Commit Limit\" sampleRate=\"PT15S\" unit=\"Bytes\"><annotation displayName=\"Memory commit limit\" locale=\"en-us\"/></PerformanceCounterConfiguration><PerformanceCounterConfiguration counterSpecifier=\"\\PhysicalDisk(_Total)\\% Disk Time\" sampleRate=\"PT15S\" unit=\"Percent\"><annotation displayName=\"Disk active time\" locale=\"en-us\"/></PerformanceCounterConfiguration>",
"wadperfcounters2": "<PerformanceCounterConfiguration counterSpecifier=\"\\PhysicalDisk(_Total)\\% Disk Read Time\" sampleRate=\"PT15S\" unit=\"Percent\"><annotation displayName=\"Disk active read time\" locale=\"en-us\"/></PerformanceCounterConfiguration><PerformanceCounterConfiguration counterSpecifier=\"\\PhysicalDisk(_Total)\\% Disk Write Time\" sampleRate=\"PT15S\" unit=\"Percent\"><annotation displayName=\"Disk active write time\" locale=\"en-us\"/></PerformanceCounterConfiguration><PerformanceCounterConfiguration counterSpecifier=\"\\PhysicalDisk(_Total)\\Disk Transfers/sec\" sampleRate=\"PT15S\" unit=\"CountPerSecond\"><annotation displayName=\"Disk operations\" locale=\"en-us\"/></PerformanceCounterConfiguration><PerformanceCounterConfiguration counterSpecifier=\"\\PhysicalDisk(_Total)\\Disk Reads/sec\" sampleRate=\"PT15S\" unit=\"CountPerSecond\"><annotation displayName=\"Disk read operations\" locale=\"en-us\"/></PerformanceCounterConfiguration><PerformanceCounterConfiguration counterSpecifier=\"\\PhysicalDisk(_Total)\\Disk Writes/sec\" sampleRate=\"PT15S\" unit=\"CountPerSecond\"><annotation displayName=\"Disk write operations\" locale=\"en-us\"/></PerformanceCounterConfiguration><PerformanceCounterConfiguration counterSpecifier=\"\\PhysicalDisk(_Total)\\Disk Bytes/sec\" sampleRate=\"PT15S\" unit=\"BytesPerSecond\"><annotation displayName=\"Disk speed\" locale=\"en-us\"/></PerformanceCounterConfiguration><PerformanceCounterConfiguration counterSpecifier=\"\\PhysicalDisk(_Total)\\Disk Read Bytes/sec\" sampleRate=\"PT15S\" unit=\"BytesPerSecond\"><annotation displayName=\"Disk read speed\" locale=\"en-us\"/></PerformanceCounterConfiguration><PerformanceCounterConfiguration counterSpecifier=\"\\PhysicalDisk(_Total)\\Disk Write Bytes/sec\" sampleRate=\"PT15S\" unit=\"BytesPerSecond\"><annotation displayName=\"Disk write speed\" locale=\"en-us\"/></PerformanceCounterConfiguration><PerformanceCounterConfiguration counterSpecifier=\"\\LogicalDisk(_Total)\\% Free Space\" sampleRate=\"PT15S\" unit=\"Percent\"><annotation displayName=\"Disk free space (percentage)\" locale=\"en-us\"/></PerformanceCounterConfiguration></PerformanceCounters>",
"wadcfgxstart": "[concat(variables('wadlogs'), variables('wadperfcounters1'), variables('wadperfcounters2'), '<Metrics resourceId=\"')]",
"wadmetricsresourceid": "[concat('/subscriptions/', subscription().subscriptionId, '/resourceGroups/', resourceGroup().name , '/providers/', 'Microsoft.Compute/virtualMachines/')]",
"wadcfgxend": "\"><MetricAggregation scheduledTransferPeriod=\"PT1H\"/><MetricAggregation scheduledTransferPeriod=\"PT1M\"/></Metrics></DiagnosticMonitorConfiguration></WadCfg>"
```

上述組態中的度量定義 xml 節點是重要的組態元素，因為它會定義如何彙總和儲存稍早在 PerformanceCounter 節點中的 xml 所定義之效能計數器。 

> [!IMPORTANT]
> 這些計量控制了 Azure 入口網站中的監視圖表與警示 。  若要在 Azure 入口網站中查看 VM 的監視資料，必須將具有 *resourceID* 和 **MetricAggregation** 的**計量**節點包含在您的 VM 的診斷組態中。 
> 
> 

以下範例會顯示度量定義的 xml： 

```xml
<Metrics resourceId="/subscriptions/subscription().subscriptionId/resourceGroups/resourceGroup().name/providers/Microsoft.Compute/virtualMachines/vmName">
    <MetricAggregation scheduledTransferPeriod="PT1H"/>
    <MetricAggregation scheduledTransferPeriod="PT1M"/>
</Metrics>
```

*resourceID* 屬性會唯一識別您的訂用帳戶中的虛擬機器。 請確定使用 subscription() 和 resourceGroup() 函式，這樣範本就會根據訂用帳戶和您要部署的資源群組自動更新這些值。

若您要在迴圈中建立多部虛擬機器，必須以 copyIndex() 函式填入 resourceID 值，以正確區分每個個別的 VM。 *xmlCfg* 值可以更新為支援此功能，如下所示：  

```json
"xmlCfg": "[base64(concat(variables('wadcfgxstart'), variables('wadmetricsresourceid'), concat(parameters('vmNamePrefix'), copyindex()), variables('wadcfgxend')))]", 
```

PT1M 及 PT1H 的 MetricAggregation 值分別表示超過一分鐘的彙總及超過一小時的彙總。

## <a name="wadmetrics-tables-in-storage"></a>儲存體中的 WADMetrics 資料表
上述的度量組態將會在您的診斷儲存體帳戶中產生具有下列命名慣例的資料表：

* **WADMetrics**：所有 WADMetrics 資料表的標準前置詞
* **PT1H** 或 **PT1M**：表示資料表包含超過 1 小時或 1 分鐘的彙總資料
* **P10D**：表示資料表包含資料表開始收集資料起 10 天的資料
* **V2S**：字串常數
* **yyyymmdd**：資料表開始收集資料的日期

範例：*WADMetricsPT1HP10DV2S20151108* 包含從 2015 年 11 月 11 日開始 10 天內，所有超過一小時的彙總計量資料    

每個 WADMetrics 資料表都包含下列資料行：

* **PartitionKey**︰分割區索引鍵是以 resourceID 值作為基礎所建構，可唯一識別 VM 資源。 例如：002Fsubscriptions:<subscriptionID>:002FresourceGroups:002F<ResourceGroupName>:002Fproviders:002FMicrosoft:002ECompute:002FvirtualMachines:002F<vmName>  
* **RowKey**：遵循下列格式：`<Descending time tick>:<Performance Counter Name>`。 遞減的時間刻度計算是最大時間刻度減去開始彙總期間時間。 例如，如果取樣期間是從 2015 年 11 月 10 日 00:00Hrs UTC 開始，則計算就是：`DateTime.MaxValue.Ticks - (new DateTime(2015,11,10,0,0,0,DateTimeKind.Utc).Ticks)`。 對於記憶體可用位元組的效能計數器，資料列索引鍵看起來會像：`2519551871999999999__:005CMemory:005CAvailable:0020Bytes`
* **CounterName**：效能計數器的名稱。 這符合 xml 設定中定義的 *counterSpecifier* 。
* **Maximum**：彙總期間效能計數器的最大值。
* **Minimum**：彙總期間效能計數器的最小值。
* **Total**：彙總期間報告之效能計數器的所有值加總。
* **Count**：針對效能計數器報告的值總數。
* **Average**：彙總期間效能計數器的平均 (總計/計數) 值。

## <a name="next-steps"></a>後續步驟
* 如需具有診斷擴充功能之 Windows 虛擬機器的完整範例範本，請參閱 [201-vm-monitoring-diagnostics-extension](https://github.com/Azure/azure-quickstart-templates/tree/master/201-vm-monitoring-diagnostics-extension)   
* 使用 [Azure PowerShell](../windows/ps-template.md) 或 [Azure 命令列](../linux/create-ssh-secured-vm-from-template.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json)部署 Azure Resource Manager 範本
* 深入了解 [編寫 Azure 資源管理員範本](../../resource-group-authoring-templates.md)
