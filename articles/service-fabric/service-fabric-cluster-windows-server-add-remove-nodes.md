---
title: 在獨立 Service Fabric 叢集中新增或移除節點 | Microsoft Docs
description: 了解如何在執行 Windows Server 的實體或虛擬電腦上 (無論是在內部部署或任何雲端) 對 Azure Service Fabric 叢集新增或移除節點。
services: service-fabric
documentationcenter: .net
author: dkkapur
manager: timlt
editor: ''
ms.assetid: bc6b8fc0-d2af-42f8-a164-58538be38d02
ms.service: service-fabric
ms.devlang: dotnet
ms.topic: conceptual
ms.tgt_pltfrm: NA
ms.workload: NA
ms.date: 11/02/2017
ms.author: dekapur
ms.openlocfilehash: b1ffe3510cf4f5e73b05572a482f49d529fca60d
ms.sourcegitcommit: eb75f177fc59d90b1b667afcfe64ac51936e2638
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 05/16/2018
ms.locfileid: "34212539"
---
# <a name="add-or-remove-nodes-to-a-standalone-service-fabric-cluster-running-on-windows-server"></a>在執行於 Windows Server 上的獨立 Service Fabric 叢集中新增或移除節點
[在 Windows Server 機器上建立獨立 Service Fabric 叢集](service-fabric-cluster-creation-for-windows-server.md)之後，您的業務需求可能會改變，因此您將需要在叢集中新增或移除節點。 本文提供可達成此目的的詳細步驟。 請注意，在本機開發叢集中，不支援新增/移除節點功能。

## <a name="add-nodes-to-your-cluster"></a>將節點新增至叢集

1. 遵循[規劃和準備 Service Fabric 叢集部署](service-fabric-cluster-creation-for-windows-server.md)中所述的步驟，準備您要新增至叢集的 VM/機器
2. 識別要用來新增此 VM/機器的容錯網域和升級網域
3. 透過遠端桌面 (RDP) 登入到要新增至叢集的 VM/機器
4. 複製或[下載適用於 Windows Server 之 Service Fabric 的獨立套件](http://go.microsoft.com/fwlink/?LinkId=730690)到此 VM/機器，然後將套件解壓縮
5. 以較高的權限 PowerShell，然後瀏覽至解壓縮套件的位置
6. 使用描述要新增之新節點的參數來執行 *AddNode.ps1* 指令碼。 下列範例會將名為 VM5 的新節點 (類型為 NodeType0 且 IP 位址為 182.17.34.52) 新增至 UD1 和 fd:/dc1/r0。 *ExistingClusterConnectionEndPoint* 是已在現有叢集中之節點的連線端點，這可以是叢集中「任何」節點的 IP 位址。

    ```
    .\AddNode.ps1 -NodeName VM5 -NodeType NodeType0 -NodeIPAddressorFQDN 182.17.34.52 -ExistingClientConnectionEndpoint 182.17.34.50:19000 -UpgradeDomain UD1 -FaultDomain fd:/dc1/r0 -AcceptEULA
    ```
    指令碼執行完成之後，您可以執行 [Get-ServiceFabricNode](/powershell/module/servicefabric/get-servicefabricnode?view=azureservicefabricps) Cmdlet，來檢查是否已新增新節點。

7. 為了確保叢集中不同節點間的一致性，您必須起始組態升級。 請執行 [Get-ServiceFabricClusterConfiguration](/powershell/module/servicefabric/get-servicefabricclusterconfiguration?view=azureservicefabricps) 來取得最新的組態檔，然後將剛新增的節點新增到 "Nodes" 區段。 此外，建議您一律備妥最新的叢集組態，以因應您需要以相同組態重新部署叢集的情況。

    ```
        {
            "nodeName": "vm5",
            "iPAddress": "182.17.34.52",
            "nodeTypeRef": "NodeType0",
            "faultDomain": "fd:/dc1/r0",
            "upgradeDomain": "UD1"
        }
    ```
8. 執行 [Start-ServiceFabricClusterConfigurationUpgrade](/powershell/module/servicefabric/start-servicefabricclusterconfigurationupgrade?view=azureservicefabricps) 來開始升級。

    ```
    Start-ServiceFabricClusterConfigurationUpgrade -ClusterConfigPath <Path to Configuration File>

    ```
    您可以在 Service Fabric Explorer 中監視升級進度。 或者，您也可以執行 [Get-ServiceFabricClusterUpgrade](/powershell/module/servicefabric/get-servicefabricclusterupgrade?view=azureservicefabricps)

### <a name="add-nodes-to-clusters-configured-with-windows-security-using-gmsa"></a>使用 gMSA 將節點新增至已設定 Windows 安全性的叢集
針對已設定「群組受控服務帳戶」(gMSA) (https://technet.microsoft.com/library/hh831782.aspx)) 的叢集，可以使用組態升級來新增新的節點：
1. 在任何現有的節點上執行 [Get-ServiceFabricClusterConfiguration](/powershell/module/servicefabric/get-servicefabricclusterconfiguration?view=azureservicefabricps) 來取得最新的組態檔，然後在 "Nodes" 區段中新增有關所要新增之新節點的詳細資料。 請確定新節點屬於相同的群組受控帳戶。 此帳戶應該是所有機器上的「系統管理員」。

    ```
        {
            "nodeName": "vm5",
            "iPAddress": "182.17.34.52",
            "nodeTypeRef": "NodeType0",
            "faultDomain": "fd:/dc1/r0",
            "upgradeDomain": "UD1"
        }
    ```
2. 執行 [Start-ServiceFabricClusterConfigurationUpgrade](/powershell/module/servicefabric/start-servicefabricclusterconfigurationupgrade?view=azureservicefabricps) 來開始升級。

    ```
    Start-ServiceFabricClusterConfigurationUpgrade -ClusterConfigPath <Path to Configuration File>
    ```
    您可以在 Service Fabric Explorer 中監視升級進度。 或者，您也可以執行 [Get-ServiceFabricClusterUpgrade](/powershell/module/servicefabric/get-servicefabricclusterupgrade?view=azureservicefabricps)

### <a name="add-node-types-to-your-cluster"></a>將節點類型新增至叢集
若要新增新的節點類型，請修改您的組態以在 "Properties" 底下的 "NodeTypes" 區段中包含新節點類型，然後使用 [Start-ServiceFabricClusterConfigurationUpgrade](/powershell/module/servicefabric/start-servicefabricclusterconfigurationupgrade?view=azureservicefabricps) 來開始組態升級。 升級完成之後，您便可以將此節點類型的新節點新增到您的叢集。

## <a name="remove-nodes-from-your-cluster"></a>從叢集移除節點
您可以使用組態升級，以下列方式將節點自叢集中移除：

1. 執行 [Get-ServiceFabricClusterConfiguration](/powershell/module/servicefabric/get-servicefabricclusterconfiguration?view=azureservicefabricps) 來取得最新的組態檔，然後將節點從 "Nodes" 區段中「移除」。
將 "NodesToBeRemoved" 參數新增至 "FabricSettings" 區段內的 "Setup" 區段。 "value" 應該是需要移除之節點的節點名稱清單 (以逗號分隔)。

    ```
         "fabricSettings": [
            {
            "name": "Setup",
            "parameters": [
                {
                "name": "FabricDataRoot",
                "value": "C:\\ProgramData\\SF"
                },
                {
                "name": "FabricLogRoot",
                "value": "C:\\ProgramData\\SF\\Log"
                },
                {
                "name": "NodesToBeRemoved",
                "value": "vm0, vm1"
                }
            ]
            }
        ]
    ```
2. 執行 [Start-ServiceFabricClusterConfigurationUpgrade](/powershell/module/servicefabric/start-servicefabricclusterconfigurationupgrade?view=azureservicefabricps) 來開始升級。

    ```
    Start-ServiceFabricClusterConfigurationUpgrade -ClusterConfigPath <Path to Configuration File>

    ```
    您可以在 Service Fabric Explorer 中監視升級進度。 或者，您也可以執行 [Get-ServiceFabricClusterUpgrade](/powershell/module/servicefabric/get-servicefabricclusterupgrade?view=azureservicefabricps)

> [!NOTE]
> 移除節點可能會起始多個升級作業。 有些節點帶有 `IsSeedNode=”true”` 標記標示，透過使用 `Get-ServiceFabricClusterManifest` 來查詢叢集資訊清單即可識別這些節點。 移除這類節點所需的時間可能比移除其他節點長，因為在這類案例中，需要將種子節點四處移動。 叢集必須至少維持 3 個主要節點類型節點。
> 
> 

### <a name="remove-node-types-from-your-cluster"></a>從叢集移除節點類型
移除節點之前，請仔細檢查是否有任何節點參考該節點類型。 請先移除這些節點，然後才移除對應的節點類型。 移除所有對應的節點之後，您便可以將該 NodeType 自叢集組態中移除，然後使用 [Start-ServiceFabricClusterConfigurationUpgrade](/powershell/module/servicefabric/start-servicefabricclusterconfigurationupgrade?view=azureservicefabricps) 來開始組態升級。


### <a name="replace-primary-nodes-of-your-cluster"></a>取代您叢集的主要節點
應以逐一取代主要節點的方式來執行，而不是以批次方式移除後再加入。


## <a name="next-steps"></a>後續步驟
* [獨立 Windows 叢集的組態設定](service-fabric-cluster-manifest.md)
* [使用 X509 憑證保護 Windows 上的獨立叢集](service-fabric-windows-cluster-x509-security.md)
* [建立具有執行 Windows 之 Azure VM 的獨立 Service Fabric 叢集](service-fabric-cluster-creation-with-windows-azure-vms.md)

