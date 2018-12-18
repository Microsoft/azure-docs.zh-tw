---
title: Azure Service Fabric - 使用 Windows Azure 診斷延伸模組進行效能監視 | Microsoft Docs
description: 使用 Windows Azure 診斷收集 Azure Service Fabric 叢集的效能計數器。
services: service-fabric
documentationcenter: .net
author: srrengar
manager: timlt
editor: ''
ms.assetid: ''
ms.service: service-fabric
ms.devlang: dotnet
ms.topic: conceptual
ms.tgt_pltfrm: NA
ms.workload: NA
ms.date: 03/26/2018
ms.author: srrengar
ms.openlocfilehash: f99206fe673f69c78bf130026207ed58344ccea5
ms.sourcegitcommit: 7ad9db3d5f5fd35cfaa9f0735e8c0187b9c32ab1
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/27/2018
ms.locfileid: "39324420"
---
# <a name="performance-monitoring-with-the-windows-azure-diagnostics-extension"></a>使用 Windows Azure 診斷延伸模組進行效能監視

本文件說明如何以 Windows 叢集的 Windows Azure 診斷 (WAD) 延伸模組逐步設定效能計數器集合。 若為 Linux 叢集，請設定 [OMS 代理程式](service-fabric-diagnostics-oms-agent.md)以收集您節點的效能計數器。 

 > [!NOTE]
> 應該在您的叢集上部署 WAD 延伸模組，以便這些步驟產生效用。 如果未設定，請參閱[使用 Windows Azure 診斷的事件彙總和收集](service-fabric-diagnostics-event-aggregation-wad.md)。  

## <a name="collect-performance-counters-via-the-wadcfg"></a>透過 WadCfg 收集效能計數器

若要透過 WAD 收集效能計數器，您需要適當地修改叢集 Resource Manager 範本中的設定。 請按照下列步驟來新增您想要收集至範本的效能計數器，並執行資源管理員資源升級。

1. 尋找您的叢集範本中的 WAD 設定 - 尋找 `WadCfg`。 您將在 `DiagnosticMonitorConfiguration` 下新增效能計數器以進行收集。

2. 將下列區段新增至您的 `DiagnosticMonitorConfiguration` 即可進行設定，以收集效能計數器。 

    ```json
    "PerformanceCounters": {
        "scheduledTransferPeriod": "PT1M",
        "PerformanceCounterConfiguration": []
    }
    ```

    `scheduledTransferPeriod` 所定義的是收集的計數器值傳輸到 Azure 儲存體資料表和任何已設定的接收之頻率。 

3. 將您想要收集的效能計數器新增至上一步中宣告的 `PerformanceCounterConfiguration`。 您想要收集的每個計數器皆可透過 `counterSpecifier`、`sampleRate`、`unit`、`annotation`，以及任何相關的 `sinks` 定義。

*處理器時間總計* (CPU 用於處理作業的時間量) 和*每秒 Service Fabric 動作項目方法引動過程* (其中一個 Service Fabric 自訂效能計數器) 的計數器組態範例如下。 請參閱 [Reliable Actor 效能計數器](service-fabric-reliable-actors-diagnostics.md#list-of-events-and-performance-counters)和 [Reliable Service 效能計數器](service-fabric-reliable-serviceremoting-diagnostics.md#list-of-performance-counters)，取得 Service Fabric 自訂效能計數器的完整清單。

 ```json
 "WadCfg": {
         "DiagnosticMonitorConfiguration": {
           "overallQuotaInMB": "50000",
           "EtwProviders": {
             "EtwEventSourceProviderConfiguration": [
               {
                 "provider": "Microsoft-ServiceFabric-Actors",
                 "scheduledTransferKeywordFilter": "1",
                 "scheduledTransferPeriod": "PT5M",
                 "DefaultEvents": {
                   "eventDestination": "ServiceFabricReliableActorEventTable"
                 }
               },
               {
                 "provider": "Microsoft-ServiceFabric-Services",
                 "scheduledTransferPeriod": "PT5M",
                 "DefaultEvents": {
                   "eventDestination": "ServiceFabricReliableServiceEventTable"
                 }
               }
             ],
             "EtwManifestProviderConfiguration": [
               {
                 "provider": "cbd93bc2-71e5-4566-b3a7-595d8eeca6e8",
                 "scheduledTransferLogLevelFilter": "Information",
                 "scheduledTransferKeywordFilter": "4611686018427387904",
                 "scheduledTransferPeriod": "PT5M",
                 "DefaultEvents": {
                   "eventDestination": "ServiceFabricSystemEventTable"
                 }
               }
             ]
           },
           "PerformanceCounters": {
                 "scheduledTransferPeriod": "PT1M",
                 "PerformanceCounterConfiguration": [
                     {
                         "counterSpecifier": "\\Processor(_Total)\\% Processor Time",
                         "sampleRate": "PT1M",
                         "unit": "Percent",
                         "annotation": [
                         ],
                         "sinks": ""
                     },
                     {
                         "counterSpecifier": "\\Service Fabric Actor Method(*)\\Invocations/Sec",
                         "sampleRate": "PT1M",
                     }
                 ]
             }
         }
       },
  ```

 可視需求修改計數器的採樣速率。 其格式為 `PT<time><unit>`；因此，如果您希望每秒收集計數器，應該設定 `"sampleRate": "PT15S"`。

 >[!NOTE]
 >雖然您可以使用 `*` 指定類似命名的效能計數器群組，但透過接收來傳送任何計數器 (至 Application Insights) 則需要單獨宣告。 

4. 新增需要收集的適用效能計數器之後，需要升級叢集資源，以便讓這些變更反映在正在執行的叢集中。 儲存修改的 `template.json`，並開啟 PowerShell。 您可以使用 `New-AzureRmResourceGroupDeployment` 升級叢集。 呼叫需要資源群組的名稱、更新的範本檔案，以及參數檔案，並提示資源管理員對您更新的資源進行適當的變更。 一旦登入您的帳戶並進入正確的訂用帳戶，請使用下列命令以執行升級：

    ```sh
    New-AzureRmResourceGroupDeployment -ResourceGroupName <ResourceGroup> -TemplateFile <PathToTemplateFile> -TemplateParameterFile <PathToParametersFile> -Verbose
    ```

5. 升級完成積存後 (大約需要 15-45 分鐘)，WAD 應該會收集效能計數器，並將其傳送至在與您的叢集有觀的儲存體帳戶中名稱為 WADPerformanceCountersTable 的表格。 藉由[將 AI 接收新增至 Resource Manager 範本](service-fabric-diagnostics-event-analysis-appinsights.md#add-the-ai-sink-to-the-resource-manager-template)，查看 Application Insights 中的效能計數器。

## <a name="next-steps"></a>後續步驟
* 為您的叢集收集更多效能計數器。 請參閱[效能度量](service-fabric-diagnostics-event-generation-perf.md)，以取得您應該收集的計數器清單。
* [透過 Windows VM 與 Azure Resource Manager 範本使用監視和診斷](../virtual-machines/windows/extensions-diagnostics-template.md)，對 `WadCfg` 做進一步的修改，包括設定診斷資料要傳送到的其他儲存體帳戶。
