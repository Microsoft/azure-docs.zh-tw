---
title: 使用 Windows Azure 診斷的 Azure Service Fabric 事件彙總 | Microsoft Docs
description: 了解如何使用 WAD 彙總及收集事件，來監視和診斷 Azure Service Fabric 叢集。
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
ms.date: 04/03/2018
ms.author: srrengar
ms.openlocfilehash: 38a026e8995bb7384c866dcd2f12588ca816009f
ms.sourcegitcommit: eb75f177fc59d90b1b667afcfe64ac51936e2638
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 05/16/2018
ms.locfileid: "34205768"
---
# <a name="event-aggregation-and-collection-using-windows-azure-diagnostics"></a>使用 Windows Azure 診斷的事件彙總和收集
> [!div class="op_single_selector"]
> * [Windows](service-fabric-diagnostics-event-aggregation-wad.md)
> * [Linux](service-fabric-diagnostics-event-aggregation-lad.md)
>
>

當您執行 Azure Service Fabric 叢集時，最好從中央位置的所有節點收集記錄檔。 將記錄檔集中在中央位置，可協助您分析並針對叢集或該叢集中執行之應用程式與服務的問題進行疑難排解。

上傳和收集記錄的其中一種方式就是使用「Windows Azure 診斷 (WAD)」延伸模組，此延伸模組可將記錄上傳到「Azure 儲存體」，也可以選擇將記錄傳送至 Azure Application Insights 或「事件中樞」。 您也可以使用外部程序來讀取儲存體中的事件，然後將它們放在 [Log Analytics](../log-analytics/log-analytics-service-fabric.md) 這類的分析平台產品或其他記錄剖析解決方案中。

## <a name="prerequisites"></a>先決條件
本文中使用下列工具：

* [Azure Resource Manager](../azure-resource-manager/resource-group-overview.md)
* [Azure PowerShell](/powershell/azure/overview)
* [Azure Resource Manager 範本](../virtual-machines/windows/extensions-diagnostics-template.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json)

## <a name="service-fabric-platform-events"></a>Service Fabric 平台事件
Service Fabric 會設定一些[現成的記錄通道](service-fabric-diagnostics-event-generation-infra.md)，其使用擴充功能預先設定下列通道，將監視和診斷資料傳送至儲存體資料表或其他位置：
  * [操作事件](service-fabric-diagnostics-event-generation-operational.md)：Service Fabric 平台所執行的較高層級作業。 範例包括建立應用程式和服務、節點狀態變更和升級資訊。 這些是以 Windows 事件追蹤 (ETW) 記錄的形式發出。
  * [Reliable Actors 程式設計模型事件](service-fabric-reliable-actors-diagnostics.md)
  * [Reliable Services 程式設計模型事件](service-fabric-reliable-services-diagnostics.md)

## <a name="deploy-the-diagnostics-extension-through-the-portal"></a>透過入口網站部署診斷擴充功能
收集記錄檔的第一個步驟是將診斷擴充功能部署在 Service Fabric 叢集中的虛擬機器擴展集節點上。 診斷擴充功能會收集每個 VM 上的記錄檔，並將它們上傳至您指定的儲存體帳戶。 下列步驟概述如何透過 Azure 入口網站和 Azure Resource Manager 範本對新的和現有叢集完成此操作。

### <a name="deploy-the-diagnostics-extension-as-part-of-cluster-creation-through-azure-portal"></a>透過 Azure 入口網站建立叢集時部署診斷延伸模組
建立您的叢集時，在叢集組態步驟中展開選用設定，並確定已將 [診斷] 設定為 [開啟] (預設值)。

![入口網站中用於建立叢集的 Azure 診斷設定](media/service-fabric-diagnostics-event-aggregation-wad/azure-enable-diagnostics-new.png)

在最後一個步驟中，我們強烈建議您下載範本，然後再**按一下 [建立]**。 如需詳細資訊，請參閱[使用 Azure Resource Manager 範本來設定 Service Fabric 叢集](service-fabric-cluster-creation-via-arm.md)。 您需要範本來變更要從中收集資料的通道 (如上方所列)。

![叢集範本](media/service-fabric-diagnostics-event-aggregation-wad/download-cluster-template.png)

現在，若您要在 Azure 儲存體中彙總事件，請[設定 Log Analytics](service-fabric-diagnostics-oms-setup.md) 以取得見解，並在 Log Analytics 入口網站中加以查詢

>[!NOTE]
>目前沒有任何方法可以篩選或清理已傳送至資料表的事件。 如果不實作從資料表移除事件的處理序，資料表將會繼續成長 (預設上限為 50 GB)。 如何變更此項目的指示會在[本文下方詳述](service-fabric-diagnostics-event-aggregation-wad.md#update-storage-quota)。 此外，我們可以提供具有 [Watchdog 範例](https://github.com/Azure-Samples/service-fabric-watchdog-service)中所執行資料清理服務的範例，除非您必須儲存超過 30 或 90 天時間範圍的記錄，否則建議您自行撰寫資料清理服務。

## <a name="deploy-the-diagnostics-extension-through-azure-resource-manager"></a>透過 Azure Resource Manager 來部署診斷擴充功能

### <a name="create-a-cluster-with-the-diagnostics-extension"></a>建立具有診斷擴充功能的叢集
若要使用 Resource Manager 建立叢集，您需要在建立叢集之前，將診斷組態 JSON 加入至完整的 Resource Manager 範本。 我們在 Resource Manager 範本範例中提供一個五 VM 叢集 Resource Manager 範本，且已在其中加入診斷設定。 您可以在 Azure 資源庫中的這個位置看到它： [具有診斷 Resource Manager 範本範例的五節點叢集](https://azure.microsoft.com/en-in/resources/templates/service-fabric-secure-cluster-5-node-1-nodetype/)。

若要查看 Resource Manager 範本中的 [診斷] 設定，請開啟 azuredeploy.json 檔案，並搜尋 **IaaSDiagnostics**。 若要使用這個範本建立叢集，請選取上一個連結所提供的 [部署到 Azure] 按鈕。

或者，您也可以下載資源管理員範例，對它進行變更，然後在 Azure PowerShell 視窗中使用 `New-AzureRmResourceGroupDeployment` 命令來使用修改過的範本建立叢集。 針對您傳遞給命令的參數，請參閱以下程式碼。 如需如何使用 PowerShell 部署資源群組的詳細資訊，請參閱[使用 Azure Resource Manager 範本部署資源群組](../azure-resource-manager/resource-group-template-deploy.md)。

### <a name="add-the-diagnostics-extension-to-an-existing-cluster"></a>將診斷擴充功能新增到現有叢集
如果您具有未部署診斷的現有叢集，您可以透過叢集範本來新增或更新它。 修改用來建立現有叢集的 Resource Manager 範本，或是以上述方式從入口網站下載範本。 執行下列工作來修改 template.json 檔案：

藉由新增儲存體資源到資源區段，以將其新增至範本。

```json
{
  "apiVersion": "2015-05-01-preview",
  "type": "Microsoft.Storage/storageAccounts",
  "name": "[parameters('applicationDiagnosticsStorageAccountName')]",
  "location": "[parameters('computeLocation')]",
  "sku": {
    "accountType": "[parameters('applicationDiagnosticsStorageAccountType')]"
  },
  "tags": {
    "resourceType": "Service Fabric",
    "clusterName": "[parameters('clusterName')]"
  }
},
```

 接下來，新增至參數區段中儲存體帳戶定義之後的位置，位於 `supportLogStorageAccountName` 之間。 以您偏好的儲存體帳戶名稱取代預留位置文字 storage account name goes here。

```json
    "applicationDiagnosticsStorageAccountType": {
      "type": "string",
      "allowedValues": [
        "Standard_LRS",
        "Standard_GRS"
      ],
      "defaultValue": "Standard_LRS",
      "metadata": {
        "description": "Replication option for the application diagnostics storage account"
      }
    },
    "applicationDiagnosticsStorageAccountName": {
      "type": "string",
      "defaultValue": "**STORAGE ACCOUNT NAME GOES HERE**",
      "metadata": {
        "description": "Name for the storage account that contains application diagnostics data from the cluster"
      }
    },
```
然後，透過在擴充功能陣列內新增下列內容，更新 template.json 檔案的 `VirtualMachineProfile` 區段。 請務必在開頭或結尾 (取決於其插入的位置) 加入逗點。

```json
{
    "name": "[concat(parameters('vmNodeType0Name'),'_Microsoft.Insights.VMDiagnosticsSettings')]",
    "properties": {
        "type": "IaaSDiagnostics",
        "autoUpgradeMinorVersion": true,
        "protectedSettings": {
        "storageAccountName": "[parameters('applicationDiagnosticsStorageAccountName')]",
        "storageAccountKey": "[listKeys(resourceId('Microsoft.Storage/storageAccounts', parameters('applicationDiagnosticsStorageAccountName')),'2015-05-01-preview').key1]",
        "storageAccountEndPoint": "https://core.windows.net/"
        },
        "publisher": "Microsoft.Azure.Diagnostics",
        "settings": {
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
            }
            }
        },
        "StorageAccount": "[parameters('applicationDiagnosticsStorageAccountName')]"
        },
        "typeHandlerVersion": "1.5"
    }
}
```

如所述修改 template.json 檔案之後，將 Resource Manager 範本重新發佈。 如果已匯出範本，執行 deploy.ps1 檔案將會重新發佈範本。 部署之後，請確認 **ProvisioningState** 為 **Succeeded**。

> [!TIP]
> 如果您要將容器部署至叢集，請讓 WAD 能夠挑選 Docker 統計資料，方法是將該設定新增至 **WadCfg > DiagnosticMonitorConfiguration** 區段。
>
>```json
>"DockerSources": {
>    "Stats": {
>        "enabled": true,
>        "sampleRate": "PT1M"
>    }
>},
>```

### <a name="update-storage-quota"></a>更新儲存體配額

因為由擴充功能填入的表格可成長到達到配額限制，因此建議您考慮降低配額大小。 預設值是 50 GB，並可在 `DiagnosticMonitorConfiguration` 下方，從 `overallQuotainMB` 欄位下的範本中設定

```json
"overallQuotaInMB": "50000",
```

## <a name="log-collection-configurations"></a>記錄收集設定
來自其他通道的記錄也可供收集，以下是一些您可以在適用於 Azure 中執行之叢集的範本中進行的最常見設定。

* 操作通道 - 基礎：預設啟用，由 Service Fabric 與叢集執行的高階作業，包括正在啟動之節點、正在部署之新應用程式或升級復原等的事件。如需事件清單，請參閱[操作通道事件](https://docs.microsoft.com/azure/service-fabric/service-fabric-diagnostics-event-generation-operational)。
  
```json
      scheduledTransferKeywordFilter: "4611686018427387904"
  ```
* 操作通道 - 詳細：這包括健康情況報告與負載平衡決策，再加上基礎操作通道中的所有項目。 這些事件是藉由使用健康情況或負載報告 API (例如 [ReportPartitionHealth](https://msdn.microsoft.com/library/azure/system.fabric.iservicepartition.reportpartitionhealth.aspx) 或 [ReportLoad](https://msdn.microsoft.com/library/azure/system.fabric.iservicepartition.reportload.aspx))，由系統或您的程式碼所產生的。 若要在 Visual Studio 的「診斷事件檢視器」中檢視這些事件，請將 "Microsoft-ServiceFabric:4:0x4000000000000008" 新增到 ETW 提供者清單中。

```json
      scheduledTransferKeywordFilter: "4611686018427387912"
  ```

* 資料和傳訊通道 - 基礎：除了詳細操作通道記錄以外，在傳訊 (目前僅限 ReverseProxy) 和資料路徑中產生的重要記錄和事件。 這些事件是 ReverseProxy 和所處理要求中的要求處理失敗及其他嚴重問題。 **這是我們建議採用的完整記錄方式**。 若要在 Visual Studio 的「診斷事件檢視器」中檢視這些事件，請將 "Microsoft-ServiceFabric:4:0x4000000000000010" 新增到 ETW 提供者清單中。

```json
      scheduledTransferKeywordFilter: "4611686018427387928"
  ```

* 資料和傳訊通道 - 詳細：詳細資訊通道，其中包含來自叢集及詳細操作通道中資料和傳訊的所有非重要記錄。 如需所有反向 Proxy 事件的詳細疑難排解，請參閱[反向 proxy 診斷指南](service-fabric-reverse-proxy-diagnostics.md)。  若要在 Visual Studio 的「診斷事件檢視器」中檢視這些事件，請將 "Microsoft-ServiceFabric:4:0x4000000000000020" 新增到 ETW 提供者清單中。

```json
      scheduledTransferKeywordFilter: "4611686018427387944"
  ```

>[!NOTE]
>此通道具有非常大量的事件，允許從這個詳細通道收集事件會導致快速產生大量追蹤，而會消耗儲存體容量。 請只在絕對必要時才開啟此功能。


若要啟用**基礎資料和傳訊通道** (我們建議採用的完整記錄方式)，您範本之 `WadCfg` 中的 `EtwManifestProviderConfiguration` 會看起來如下：

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
                "scheduledTransferKeywordFilter": "4611686018427387928",
                "scheduledTransferPeriod": "PT5M",
                "DefaultEvents": {
                  "eventDestination": "ServiceFabricSystemEventTable"
                }
              }
            ]
          }
        }
      },
```

## <a name="collect-from-new-eventsource-channels"></a>從新的 EventSource 通道收集

若要更新診斷以從新的 EventSource 通道 (代表您將要部署的新應用程式) 收集記錄，請執行先前所述的相同步驟，以針對現有叢集設定診斷。

在您使用 `New-AzureRmResourceGroupDeployment` PowerShell 命令套用組態更新之前，請先更新 template.json 檔案中的 `EtwEventSourceProviderConfiguration` 區段，以新增 EventSource 通道的項目。 在 Visual Studio 產生的 ServiceEventSource.cs 檔案中，事件來源的名稱定義為程式碼的一部分。

例如，如果您的事件資源名稱是 My-Eventsource，新增下列程式碼以將來自 My-Eventsource 的事件置於名稱為 MyDestinationTableName 的資料表中。

```json
        {
            "provider": "My-Eventsource",
            "scheduledTransferPeriod": "PT5M",
            "DefaultEvents": {
            "eventDestination": "MyDestinationTableName"
            }
        }
```

若要收集效能計數器或事件記錄檔，請以[使用 Azure Resource Manager 範本建立具有監視和診斷的 Windows 虛擬機器](../virtual-machines/windows/extensions-diagnostics-template.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json)中提供的範例來修改 Resource Manager 範本。 然後請重新發佈 Resource Manager 範本。

## <a name="collect-performance-counters"></a>收集效能計數器

若要收集叢集中的效能計量，請在叢集的 Resource Manager 範本中將效能計數器新增至 "WadCfg > DiagnosticMonitorConfiguration"。 如需修改 `WadCfg` 以收集特定效能計數器的步驟，請參閱[使用 WAD 進行效能監視](service-fabric-diagnostics-perf-wad.md)。 如需我們建議收集的效能計數器清單，請參閱 [Service Fabric 效能計數器](service-fabric-diagnostics-event-generation-perf.md)。
  
如果您要使用 Application Insights 接收 (如下節所述)，而且想要將這些計量顯示在 Application Insights 中，則請確定在 "sinks" 區段中新增接收名稱 (如上所示)。 這會將個別設定的效能計數器自動傳送至 Application Insights 資源。


## <a name="send-logs-to-application-insights"></a>將記錄傳送至 Application Insights

將監視和診斷資料傳送至 Application Insights (AI) 的作業，可以在設定 WAD 時進行。 如果您決定使用 AI 進行事件分析和視覺效果，請閱讀屬於 "WadCfg" 一部份的[如何設定 AI 接收](service-fabric-diagnostics-event-analysis-appinsights.md#add-the-ai-sink-to-the-resource-manager-template)。

## <a name="next-steps"></a>後續步驟

正確設定 Azure 診斷之後，您會從 ETW 和 EventSource 記錄中看到「儲存體」資料表中的資料。 如果您選擇使用 Log Analytics、Kibana 或未直接設定於 Resource Manager 範本的任何其他資料分析和視覺效果平台，請務必設定您要用來讀取這些儲存體資料表資料的平台。 對 Log Analytics 而言，這是非常一般的作業，且會在[事件和記錄分析](service-fabric-diagnostics-event-analysis-oms.md)中進行。 也就是說，Application Insights 是特殊案例，因為它可以在設定診斷延伸模組時設定；因此，如果您選擇使用 AI，則請參閱[適當的文章](service-fabric-diagnostics-event-analysis-appinsights.md)。

>[!NOTE]
>目前沒有任何方法可以篩選或清理已傳送至資料表的事件。 如果不實作從資料表移除事件的處理序，資料表將會繼續成長。 目前，我們可以提供具有 [Watchdog 範例](https://github.com/Azure-Samples/service-fabric-watchdog-service)中所執行資料清理服務的範例，除非您必須儲存超過 30 或 90 天時間範圍的記錄，否則建議您自行撰寫資料清理服務。

* [了解如何使用診斷擴充功能收集效能計數器或記錄檔](../virtual-machines/windows/extensions-diagnostics-template.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json)
* [使用 Application Insights 進行事件分析和視覺效果](service-fabric-diagnostics-event-analysis-appinsights.md)
* [使用 Log Analytics 進行事件分析和視覺效果](service-fabric-diagnostics-event-analysis-oms.md)
