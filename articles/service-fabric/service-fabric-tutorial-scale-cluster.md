---
title: 在 Azure 中調整 Service Fabric 叢集 | Microsoft Docs
description: 在本教學課程中，您會了解如何在 Azure 中快速調整 Service Fabric 叢集。
services: service-fabric
documentationcenter: .net
author: rwike77
manager: timlt
editor: ''
ms.assetid: ''
ms.service: service-fabric
ms.devlang: dotNet
ms.topic: tutorial
ms.tgt_pltfrm: NA
ms.workload: NA
ms.date: 010/01/2018
ms.author: ryanwi
ms.custom: mvc
ms.openlocfilehash: 1af4cdb361c1db378991201fc42f17dcbf67fe67
ms.sourcegitcommit: 1981c65544e642958917a5ffa2b09d6b7345475d
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/03/2018
ms.locfileid: "48238760"
---
# <a name="tutorial-scale-a-service-fabric-cluster-in-azure"></a>教學課程：在 Azure 中調整 Service Fabric 叢集

本教學課程為系列課程的第二個部分，解說如何相應放大和縮小現有叢集。 完成時，您將知道如何調整叢集，以及如何清除任何剩餘的資源。

在本教學課程中，您了解如何：

> [!div class="checklist"]
> * 讀取叢集節點計數
> * 新增叢集節點 (相應放大)
> * 移除叢集節點 (相應縮小)

在本教學課程系列中，您將了解如何：
> [!div class="checklist"]
> * 在 Azure 上，使用範本建立安全的 [Windows 叢集](service-fabric-tutorial-create-vnet-and-windows-cluster.md)或 [Linux 叢集](service-fabric-tutorial-create-vnet-and-linux-cluster.md)
> * 將叢集相應縮小或相應放大
> * [升級叢集的執行階段](service-fabric-tutorial-upgrade-cluster.md)
> * [刪除叢集](service-fabric-tutorial-delete-cluster.md)

## <a name="prerequisites"></a>必要條件

開始進行本教學課程之前：

* 如果您沒有 Azure 訂用帳戶，請建立[免費帳戶](https://azure.microsoft.com/free/?WT.mc_id=A261C142F)
* 安裝 [Azure PowerShell 模組 4.1 版或更新版本](https://docs.microsoft.com/powershell/azure/install-azurerm-ps)或 [Azure CLI](/cli/azure/install-azure-cli)。
* 在 Azure 上，建立安全的 [Windows 叢集](service-fabric-tutorial-create-vnet-and-windows-cluster.md)或 [Linux 叢集](service-fabric-tutorial-create-vnet-and-linux-cluster.md)
* 如果您部署 Windows 叢集，請設定 Windows 開發環境。 安裝 [Visual Studio 2017](http://www.visualstudio.com) 和 **Azure 開發**、**ASP.NET 和 Web 開發**以及 **.NET Core 跨平台開發**工作負載。  然後設定 [.NET 開發環境](service-fabric-get-started.md)。
* 如果您部署 Linux 叢集，請在 [Linux](service-fabric-get-started-linux.md) 或 [MacOS](service-fabric-get-started-mac.md) 上設定 Java 開發環境。  安裝 [Service Fabric CLI](service-fabric-cli.md)。

## <a name="sign-in-to-azure"></a>登入 Azure

請先登入您的 Azure 帳戶並選取您的訂用帳戶，再執行 Azure 命令。

```powershell
Connect-AzureRmAccount
Get-AzureRmSubscription
Set-AzureRmContext -SubscriptionId <guid>
```

```azurecli
az login
az account set --subscription <guid>
```

## <a name="connect-to-the-cluster"></a>連接到叢集

若要成功完成教學課程的這個部分，您需要連線到 Service Fabric 叢集和虛擬機器擴展集 (其中裝載叢集)。 虛擬機器擴展集是在 Azure 裝載 Service Fabric 的 Azure 資源。

您連接到叢集時，您可以查詢叢集的資訊。 您可以使用叢集，了解叢集注意到哪些節點。 在下列程式碼中，連接到叢集的動作使用此系列的第一個部分中建立的同一個憑證。 請確定您將 `$endpoint` 和 `$thumbprint` 變數設定為您的值。

```powershell
$endpoint = "<mycluster>.southcentralus.cloudapp.azure.com:19000"
$thumbprint = "63EB5BA4BC2A3BADC42CA6F93D6F45E5AD98A1E4"

Connect-ServiceFabricCluster -ConnectionEndpoint $endpoint `
          -KeepAliveIntervalInSec 10 `
          -X509Credential -ServerCertThumbprint $thumbprint `
          -FindType FindByThumbprint -FindValue $thumbprint `
          -StoreLocation CurrentUser -StoreName My

Get-ServiceFabricClusterHealth
```

```azurecli
sfctl cluster select --endpoint https://aztestcluster.southcentralus.cloudapp.azure.com:19080 \
--pem ./aztestcluster201709151446.pem --no-verify
```

完成連線後，您可以使用命令取得叢集中每個節點的狀態。 對於 **PowerShell**，使用 `Get-ServiceFabricClusterHealth` 命令，對於 **sfctl** 請使用 `sfctl cluster select` 命令。

## <a name="scale-out"></a>相應放大

您相應放大時，會將多個虛擬機器執行個體新增到擴展集。 這些執行個體會成為 Service Fabric 使用的節點。 擴展集有多個新增的執行個體 (透過相應放大) 時，Service Fabric 會知道並自動回應。 下列程式碼會按照名稱取得擴展集，並將擴展集的**容量**增加 1。

```powershell
$scaleset = Get-AzureRmVmss -ResourceGroupName SFCLUSTERTUTORIALGROUP -VMScaleSetName nt1vm
$scaleset.Sku.Capacity += 1

Update-AzureRmVmss -ResourceGroupName $scaleset.ResourceGroupName -VMScaleSetName $scaleset.Name -VirtualMachineScaleSet $scaleset
```

這段程式碼將容量設定為 6。

```azurecli
# Get the name of the node with
az vmss list-instances -n nt1vm -g sfclustertutorialgroup --query [*].name

# Use the name to scale
az vmss scale -g sfclustertutorialgroup -n nt1vm --new-capacity 6
```

## <a name="scale-in"></a>相應縮小

相應縮小與相應放大相同，只不過您使用較低的**容量**值。 您相應縮小擴展集時，會從擴展集移除虛擬機器執行個體。 一般來說，Service Fabric 不會察覺發生什麼事，而會認為節點消失。 Service Fabric 接著會報告叢集狀況不良的狀態。 若要避免這種錯誤狀態，您必須通知 Service Fabric 您預期節點消失。

### <a name="remove-the-service-fabric-node"></a>移除 Service Fabric 節點

> [!NOTE]
> 此部分僅適用於 *Bronze* 持久性層。 如需持久性的詳細資訊，請參閱 [Service Fabric 叢集容量規劃][durability]。

為了讓叢集節點在升級和容錯網域之間平均分配，因而讓它們的使用率更加平均，應該先移除最近建立的節點。 換句話說，節點的移除順序應該與建立順序相反。 最近建立的節點具有最大的 `virtual machine scale set InstanceId` 屬性值。 下列程式碼範例會傳回最近建立的節點。

```powershell
Get-ServiceFabricNode | Sort-Object { $_.NodeName.Substring($_.NodeName.LastIndexOf('_') + 1) } -Descending | Select-Object -First 1
```

```azurecli
sfctl node list --query "sort_by(items[*], &name)[-1]"
```

Service Fabric 叢集必須知道將移除此節點。 您需要採取三個步驟：

1. 停用節點，以免節點再成為資料的複本。  
PowerShell：`Disable-ServiceFabricNode`  
sfctl：`sfctl node disable`

2. 停止節點，以便 Service Fabric 執行階段正常關閉，而且應用程式取得終止要求。  
PowerShell：`Start-ServiceFabricNodeTransition -Stop`  
sfctl：`sfctl node transition --node-transition-type Stop`

2. 從叢集移除該節點。  
PowerShell：`Remove-ServiceFabricNodeState`  
sfctl：`sfctl node remove-state`

這三個步驟套用至節點之後，即可從擴展集移除節點。 如果在 [bronze][durability] 之外使用任何持久性層，則移除擴展集執行個體時，將完成這些步驟。

下列程式碼區塊會取得、停用、停止最後建立的節點，並從叢集移除該節點。

```powershell
#### After you've connected.....
# Get the node that was created last
$node = Get-ServiceFabricNode | Sort-Object { $_.NodeName.Substring($_.NodeName.LastIndexOf('_') + 1) } -Descending | Select-Object -First 1

# Node details for the disable/stop process
$nodename = $node.NodeName
$nodeid = $node.NodeInstanceId

$loopTimeout = 10

# Run disable logic
Disable-ServiceFabricNode -NodeName $nodename -Intent RemoveNode -TimeoutSec 300 -Force

$state = Get-ServiceFabricNode | Where-Object NodeName -eq $nodename | Select-Object -ExpandProperty NodeStatus

while (($state -ne [System.Fabric.Query.NodeStatus]::Disabled) -and ($loopTimeout -ne 0))
{
    Start-Sleep 5
    $loopTimeout -= 1
    $state = Get-ServiceFabricNode | Where-Object NodeName -eq $nodename | Select-Object -ExpandProperty NodeStatus
    Write-Host "Checking state... $state found"
}

# Exit if the node was unable to be disabled
if ($state -ne [System.Fabric.Query.NodeStatus]::Disabled)
{
    Write-Error "Disable failed with state $state"
}
else
{
    # Stop node
    $stopid = New-Guid
    Start-ServiceFabricNodeTransition -Stop -OperationId $stopid -NodeName $nodename -NodeInstanceId $nodeid -StopDurationInSeconds 300

    $state = (Get-ServiceFabricNodeTransitionProgress -OperationId $stopid).State
    $loopTimeout = 10

    # Watch the transaction
    while (($state -eq [System.Fabric.TestCommandProgressState]::Running) -and ($loopTimeout -ne 0))
    {
        Start-Sleep 5
        $state = (Get-ServiceFabricNodeTransitionProgress -OperationId $stopid).State
        Write-Host "Checking state... $state found"
    }

    if ($state -ne [System.Fabric.TestCommandProgressState]::Completed)
    {
        Write-Error "Stop transaction failed with $state"
    }
    else
    {
        # Remove the node from the cluster
        Remove-ServiceFabricNodeState -NodeName $nodename -TimeoutSec 300 -Force
    }
}
```

在下方的 **sfctl** 程式碼中，會使用下列命令來取得最後建立節點的 **node-name** 值：`sfctl node list --query "sort_by(items[*], &name)[-1].name"`

```azurecli
# Inform the node that it is going to be removed
sfctl node disable --node-name _nt1vm_5 --deactivation-intent 4 -t 300

# Stop the node using a random guid as our operation id
sfctl node transition --node-instance-id 131541348482680775 --node-name _nt1vm_5 --node-transition-type Stop --operation-id c17bb4c5-9f6c-4eef-950f-3d03e1fef6fc --stop-duration-in-seconds 14400 -t 300

# Remove the node from the cluster
sfctl node remove-state --node-name _nt1vm_5
```

> [!TIP]
> 使用下列 **sfctl** 查詢，檢查每個步驟的狀態
>
> **檢查停用狀態**
> `sfctl node list --query "sort_by(items[*], &name)[-1].nodeDeactivationInfo"`
>
> **檢查停止狀態**
> `sfctl node list --query "sort_by(items[*], &name)[-1].isStopped"`
>

### <a name="scale-in-the-scale-set"></a>相應縮小擴展集

已從叢集移除 Service Fabric 節點之後，即可相應縮小虛擬機器擴展集。 在下列範例中，擴展集容量減少 1。

```powershell
$scaleset = Get-AzureRmVmss -ResourceGroupName SFCLUSTERTUTORIALGROUP -VMScaleSetName nt1vm
$scaleset.Sku.Capacity -= 1

Update-AzureRmVmss -ResourceGroupName SFCLUSTERTUTORIALGROUP -VMScaleSetName nt1vm -VirtualMachineScaleSet $scaleset
```

這段程式碼將容量設定為 5。

```azurecli
# Get the name of the node with
az vmss list-instances -n nt1vm -g sfclustertutorialgroup --query [*].name

# Use the name to scale
az vmss scale -g sfclustertutorialgroup -n nt1vm --new-capacity 5
```

## <a name="next-steps"></a>後續步驟

在本教學課程中，您已了解如何：

> [!div class="checklist"]
> * 讀取叢集節點計數
> * 新增叢集節點 (相應放大)
> * 移除叢集節點 (相應縮小)

接下來，請前往下列教學課程，了解如何升級叢集執行階段。
> [!div class="nextstepaction"]
> [升級叢集的執行階段](service-fabric-tutorial-upgrade-cluster.md)

[durability]: service-fabric-cluster-capacity.md#the-durability-characteristics-of-the-cluster
