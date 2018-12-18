---
title: 自動調整常用計量
description: 了解哪些度量常用於自動調整您的雲端服務、虛擬機器和 Web Apps。
author: anirudhcavale
services: azure-monitor
ms.service: azure-monitor
ms.topic: conceptual
ms.date: 12/6/2016
ms.author: ancav
ms.component: autoscale
ms.openlocfilehash: 48c53b1b0c037e6bcfea3be49fdd2110e1e694b3
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46970697"
---
# <a name="azure-monitor-autoscaling-common-metrics"></a>Azure 監視器自動調整的常用度量
Azure 監視器的自動調整可讓您根據遙測資料 (度量) 增加或減少執行中執行個體的數目。 本文件說明您可能會使用的常用度量。 在 Azure 入口網站中，您可以選擇要作為調整依據的資源度量。 不過，您也可以選擇其他資源的任何度量來做為調整依據。

Azure 監視器自動調整僅適用於[虛擬機器擴展集](https://azure.microsoft.com/services/virtual-machine-scale-sets/)、[雲端服務](https://azure.microsoft.com/services/cloud-services/)、[App Service - Web Apps](https://azure.microsoft.com/services/app-service/web/) 和 [API 管理服務](https://docs.microsoft.com/azure/api-management/api-management-key-concepts)。 其他 Azure 服務使用不同的調整方法。

## <a name="compute-metrics-for-resource-manager-based-vms"></a>針對以 Resource Manager 為基礎的 VM 來計算度量
根據預設，以 Resource Manager 為基礎的虛擬機器和虛擬機器擴展集會發出基本 (主機層級) 的度量。 此外，當您設定 Azure VM 和 VMSS 的診斷資料收集時，Azure 診斷擴充也會發出客體 OS 效能計數器 (通常稱為「客體 OS 度量」)。  您在自動調整規則中使用所有這些度量。

您可以使用 `Get MetricDefinitions` API/PoSH/CLI 來檢視 VMSS 資源的可用度量。

如果您使用 VM 擴展集，而且發現特定度量沒有列出來，可能是您的診斷擴充功能已「停用」它。

如果特定度量沒有取樣或以您想要的頻率傳輸，您可以更新診斷的組態設定。

如果發生上述任一種情況，請檢閱 PowerShell 相關的[使用 PowerShell 在執行 Windows 的虛擬機器中啟用 Azure 診斷](../virtual-machines/windows/ps-extensions-diagnostics.md)，設定和更新 Azure VM 診斷擴充以啟用該度量。 該文章也包含診斷組態檔的範例。

### <a name="host-metrics-for-resource-manager-based-windows-and-linux-vms"></a>以 Resource Manager 為基礎的 Windows 和 Linux VM 的主機度量
在 Windows 和 Linux 執行個體中，根據預設會針對 Azure VM 和 VMSS 發出下列的主機層級度量。 這些度量描述您的 Azure VM，然而是從 Azure VM 主機收集，而不是透過客體 VM 上安裝的代理程式收集。 您可以在自動調整規則中使用這些度量。

- [以 Resource Manager 為基礎的 Windows 和 Linux VM 的主機度量](monitoring-supported-metrics.md#microsoftcomputevirtualmachines)
- [以 Resource Manager 為基礎的 Windows 和 Linux VM 擴展集的主機度量](monitoring-supported-metrics.md#microsoftcomputevirtualmachinescalesets)

### <a name="guest-os-metrics-resource-manager-based-windows-vms"></a>客體 OS 度量以 Resource Manager 為基礎的 Windows VM
在 Azure 中建立 VM 時會使用診斷擴充來啟用診斷。 診斷擴充會發出一組取自 VM 內的度量。 這表示您可以關閉自動調整依預設不發出的度量。

您可以在 PowerShell 中使用下列命令產生度量清單。

```
Get-AzureRmMetricDefinition -ResourceId <resource_id> | Format-Table -Property Name,Unit
```

您可以建立下列度量的警示：

| 度量名稱 | 單位 |
| --- | --- |
| \Processor(_Total)\% Processor Time |百分比 |
| \Processor(_Total)\% Privileged Time |百分比 |
| \Processor(_Total)\% User Time |百分比 |
| \Processor Information(_Total)\Processor Frequency |Count |
| \System\Processes |Count |
| \Process(_Total)\Thread Count |Count |
| \Process(_Total)\Handle Count |Count |
| \Memory\% Committed Bytes In Use |百分比 |
| \Memory\Available Bytes |位元組 |
| \Memory\Committed Bytes |位元組 |
| \Memory\Commit Limit |位元組 |
| \Memory\Pool Paged Bytes |位元組 |
| \Memory\Pool Nonpaged Bytes |位元組 |
| \PhysicalDisk(_Total)\% Disk Time |百分比 |
| \PhysicalDisk(_Total)\% Disk Read Time |百分比 |
| \PhysicalDisk(_Total)\% Disk Write Time |百分比 |
| \PhysicalDisk(_Total)\每秒的磁碟傳輸數 |每秒計數 |
| \PhysicalDisk(_Total)\Disk Reads/sec |每秒計數 |
| \PhysicalDisk(_Total)\Disk Writes/sec |每秒計數 |
| \PhysicalDisk(_Total)\Disk Bytes/sec |每秒位元組 |
| \PhysicalDisk(_Total)\Disk Read Bytes/sec |每秒位元組 |
| \PhysicalDisk(_Total)\Disk Write Bytes/sec |每秒位元組 |
| \PhysicalDisk(_Total)\Avg.磁碟佇列長度 |Count |
| \PhysicalDisk(_Total)\Avg.磁碟讀取佇列長度 |Count |
| \PhysicalDisk(_Total)\Avg.磁碟寫入佇列長度 |Count |
| \LogicalDisk(_Total)\% Free Space |百分比 |
| \LogicalDisk(_Total)\Free Megabytes |Count |

### <a name="guest-os-metrics-linux-vms"></a>客體 OS 度量 Linux VM
當您在 Azure 中建立 VM 時，根據預設會使用診斷擴充來啟用診斷。

您可以在 PowerShell 中使用下列命令產生度量清單。

```
Get-AzureRmMetricDefinition -ResourceId <resource_id> | Format-Table -Property Name,Unit
```

 您可以建立下列度量的警示：

| 度量名稱 | 單位 |
| --- | --- |
| \Memory\AvailableMemory |位元組 |
| \Memory\PercentAvailableMemory |百分比 |
| \Memory\UsedMemory |位元組 |
| \Memory\PercentUsedMemory |百分比 |
| \Memory\PercentUsedByCache |百分比 |
| \Memory\PagesPerSec |每秒計數 |
| \Memory\PagesReadPerSec |每秒計數 |
| \Memory\PagesWrittenPerSec |每秒計數 |
| \Memory\AvailableSwap |位元組 |
| \Memory\PercentAvailableSwap |百分比 |
| \Memory\UsedSwap |位元組 |
| \Memory\PercentUsedSwap |百分比 |
| \Processor\PercentIdleTime |百分比 |
| \Processor\PercentUserTime |百分比 |
| \Processor\PercentNiceTime |百分比 |
| \Processor\PercentPrivilegedTime |百分比 |
| \Processor\PercentInterruptTime |百分比 |
| \Processor\PercentDPCTime |百分比 |
| \Processor\PercentProcessorTime |百分比 |
| \Processor\PercentIOWaitTime |百分比 |
| \PhysicalDisk\BytesPerSecond |每秒位元組 |
| \PhysicalDisk\ReadBytesPerSecond |每秒位元組 |
| \PhysicalDisk\WriteBytesPerSecond |每秒位元組 |
| \PhysicalDisk\TransfersPerSecond |每秒計數 |
| \PhysicalDisk\ReadsPerSecond |每秒計數 |
| \PhysicalDisk\WritesPerSecond |每秒計數 |
| \PhysicalDisk\AverageReadTime |秒 |
| \PhysicalDisk\AverageWriteTime |秒 |
| \PhysicalDisk\AverageTransferTime |秒 |
| \PhysicalDisk\AverageDiskQueueLength |Count |
| \NetworkInterface\BytesTransmitted |位元組 |
| \NetworkInterface\BytesReceived |位元組 |
| \NetworkInterface\PacketsTransmitted |Count |
| \NetworkInterface\PacketsReceived |Count |
| \NetworkInterface\BytesTotal |位元組 |
| \NetworkInterface\TotalRxErrors |Count |
| \NetworkInterface\TotalTxErrors |Count |
| \NetworkInterface\TotalCollisions |Count |

## <a name="commonly-used-web-server-farm-metrics"></a>常用的 Web (伺服器陣列) 度量
您也可以根據常用的 Web 伺服器度量 (如 Http 佇列長度) 執行自動調整。 它的計量名稱是 **HttpQueueLength**。  下一節會列出可用的伺服器陣列 (Web Apps) 度量。

### <a name="web-apps-metrics"></a>Web Apps 度量
您可以在 PowerShell 中使用下列命令產生 Web Apps 清單。

```
Get-AzureRmMetricDefinition -ResourceId <resource_id> | Format-Table -Property Name,Unit
```

您可以針對這些度量發出警示通知或以其為調整依據。

| 度量名稱 | 單位 |
| --- | --- |
| CpuPercentage |百分比 |
| MemoryPercentage |百分比 |
| DiskQueueLength |Count |
| HttpQueueLength |Count |
| BytesReceived |位元組 |
| BytesSent |位元組 |

## <a name="commonly-used-storage-metrics"></a>常用的儲存體度量
您可以「儲存體佇列長度」做為調整依據，它是儲存體佇列中的訊息數目。 儲存體佇列長度是特殊度量，臨界值是每個執行個體的訊息數。 比方說，如果有兩個執行個體，且臨界值設定為 100，當佇列中的訊息總數為 200 時，將會進行調整。 可能每個執行個體有 100 個訊息、120 和 80 個訊息，或合計最多 200 個或更多訊息的其他任何組合。

在 Azure 入口網站的 [設定] 刀鋒視窗中進行此設定。 若使用 VM 擴展集，您可以更新 Resource Manager 範本中的自動調整設定，改為使用 metricName 作為 ApproximateMessageCount，並傳遞儲存體佇列的識別碼作為 *metricResourceUri*。

例如，使用傳統儲存體帳戶時，自動調整設定 metricTrigger 會包含：

```
"metricName": "ApproximateMessageCount",
 "metricNamespace": "",
 "metricResourceUri": "/subscriptions/SUBSCRIPTION_ID/resourceGroups/RES_GROUP_NAME/providers/Microsoft.ClassicStorage/storageAccounts/STORAGE_ACCOUNT_NAME/services/queue/queues/QUEUE_NAME"
 ```

使用 (非傳統) 儲存體帳戶時，metricTrigger 會包含：

```
"metricName": "ApproximateMessageCount",
"metricNamespace": "",
"metricResourceUri": "/subscriptions/SUBSCRIPTION_ID/resourceGroups/RES_GROUP_NAME/providers/Microsoft.Storage/storageAccounts/STORAGE_ACCOUNT_NAME/services/queue/queues/QUEUE_NAME"
```

## <a name="commonly-used-service-bus-metrics"></a>常用的服務匯流排衡量標準
您可以依服務匯流排佇列長度做為調整依據，它是服務匯流排佇列中的訊息數目。 服務匯流排長度是特殊度量，臨界值是每個執行個體的訊息數。 比方說，如果有兩個執行個體，且臨界值設定為 100，當佇列中的訊息總數為 200 時，將會進行調整。 可能每個執行個體有 100 個訊息、120 和 80 個訊息，或合計最多 200 個或更多訊息的其他任何組合。

若使用 VM 擴展集，您可以更新 Resource Manager 範本中的自動調整設定，改為使用 metricName 作為 ApproximateMessageCount，並傳遞儲存體佇列的識別碼作為 *metricResourceUri*。

```
"metricName": "MessageCount",
 "metricNamespace": "",
"metricResourceUri": "/subscriptions/SUBSCRIPTION_ID/resourceGroups/RES_GROUP_NAME/providers/Microsoft.ServiceBus/namespaces/SB_NAMESPACE/queues/QUEUE_NAME"
```

> [!NOTE]
> 若使用服務匯流排，資源群組的概念不存在，但 Azure Resource Manager 會建立每個區域的預設資源群組。 此資源群組通常是 'Default-ServiceBus-[region]' 的格式。 例如，'Default-ServiceBus-EastUS'、'Default-ServiceBus-WestUS'、'Default-ServiceBus-AustraliaEast' 等。
>
>
