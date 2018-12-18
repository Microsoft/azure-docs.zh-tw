---
title: 透過新增虛擬機器擴展集來相應放大 Azure Service Fabric 叢集 | Microsoft Docs
description: 了解如何透過新增虛擬機器擴展集來調整 Service Fabric 叢集的規模。
services: service-fabric
documentationcenter: .net
author: rwike77
manager: timlt
editor: ''
ms.assetid: 5441e7e0-d842-4398-b060-8c9d34b07c48
ms.service: service-fabric
ms.devlang: dotnet
ms.topic: article
ms.tgt_pltfrm: NA
ms.workload: NA
ms.date: 08/21/2018
ms.author: ryanwi
ms.openlocfilehash: 8e1c194ea2ebc0e06918c8389c9ee6f72afb3e86
ms.sourcegitcommit: f1e6e61807634bce56a64c00447bf819438db1b8
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/24/2018
ms.locfileid: "42887783"
---
# <a name="scale-a-service-fabric-cluster-out-by-adding-a-virtual-machine-scale-set"></a>透過新增虛擬機器擴展集來調整 Service Fabric 叢集的規模
此文章說明如何透過將新的虛擬機器擴展集新增至現有的叢集，來調整 Azure Service Fabric 叢集的規模。 Service Fabric 叢集是一組由網路連接的虛擬或實體機器，可用來將您的微服務部署到其中並進行管理。 屬於叢集一部分的機器或 VM 都稱為節點。 虛擬機器擴展集是一個 Azure 計算資源，可以用來將一組虛擬機器當做一個集合加以部署和管理。 在 Azure 叢集中定義的每個節點類型，會[設定為不同的擴展集](service-fabric-cluster-nodetypes.md)。 隨後，您即可個別管理每個節點類型。 建立 Service Fabric 叢集之後，您可以垂直調整叢集節點類型 (變更節點的資源)、將節點類型 VM 的作業系統升級，或將新的虛擬機器擴展集新增至現有的叢集。  您可以隨時調整叢集，即使正在叢集上執行工作負載，也是如此。  在叢集進行調整時，您的應用程式也會自動調整。

> [!WARNING]
> 如果叢集健康情況不良，請勿開始變更主要的 NodeType VM SKU。 如果叢集健康情況不良，且您嘗試變更 VM SKU，您只會讓叢集變得更不穩定。
>
> 建議您不要變更擴展集/節點類型的 VM SKU，除非它以[銀級耐久性或更高級別](service-fabric-cluster-capacity.md#the-durability-characteristics-of-the-cluster)執行。 變更 VM SKU 大小是資料破壞性就地基礎結構作業。 如果沒有延遲或監視此變更的能力，作業可能會導致具狀態服務的資料遺失，或者甚至對於無狀態工作負載導致其他未預期作業問題。 這代表會載入您的主要節點類型 (其正在執行具狀態服務網狀架構系統服務) 或任何正在執行具狀態應用程式的節點類型。
>

## <a name="upgrade-the-size-and-operating-system-of-the-primary-node-type-vms"></a>將主要節點類型 VM 的大小和作業系統升級
若要將主要節點類型 VM 的 VM 大小和作業系統升級，其程序如下。  升級之後，主要節點類型 VM 的大小為標準 D4_V2，所執行的是 Windows Server 2016 Datacenter with Containers。

> [!WARNING]
> 在對生產叢集嘗試此程序之前，建議您先研究範本範例，並確認要對測試叢集所進行的程序。 叢集也會有一段時間無法使用。

1. 使用這些[範本](https://github.com/Azure/service-fabric-scripts-and-templates/blob/master/templates/nodetype-upgrade/Deploy-2NodeTypes-2ScaleSets.json)和[參數](https://github.com/Azure/service-fabric-scripts-and-templates/blob/master/templates/nodetype-upgrade/Deploy-2NodeTypes-2ScaleSets.parameters.json)檔案範例，為初始叢集部署兩個節點類型和兩個擴展集 (兩個節點類型各一個擴展集)。  這兩個擴展集的大小為標準 D2_V2，所執行的是 Windows Server 2012 R2 Datacenter。  等候叢集完成基準升級。   
2. 選擇性 - 在叢集中部署具狀態範例。
3. 在決定升級主要節點類型 VM 後，請使用這些[範本](https://github.com/Azure/service-fabric-scripts-and-templates/blob/master/templates/nodetype-upgrade/Deploy-2NodeTypes-3ScaleSets.json)和[參數](https://github.com/Azure/service-fabric-scripts-and-templates/blob/master/templates/nodetype-upgrade/Deploy-2NodeTypes-3ScaleSets.parameters.json)檔案範例對主要節點類型新增擴展集，讓主要節點類型現在具有兩個擴展集。  系統服務和使用者應用程式可在兩個不同擴展集內的 VM 之間進行遷移。  新擴展集 VM 的大小為標準 D4_V2，並執行 Windows Server 2016 Datacenter with Containers。  新的擴展集也會一起新增負載平衡器和公用 IP 位址。  
    若要在範本中尋找新的擴展集，請搜尋以 vmNodeType2Name 參數命名的 "Microsoft.Compute/virtualMachineScaleSets" 資源。  若要在主要節點類型中新增擴展集，請使用 [屬性] -> [virtualMachineProfile] -> [extensionProfile] -> [擴充功能] -> [屬性] -> [設定] -> [nodeTypeRef 設定]。
4. 檢查叢集健康情況，並確認所有節點均狀況良好。
5. 停用主要節點類型舊有擴展集內的節點，以便之後移除節點。 您可以將所有節點一併停用，這些作業便會排入佇列中。 請靜候所有節點停用，這可能需要一些時間。  隨著節點類型中的較舊節點停用，系統服務和種子節點會遷移到主要節點類型中的新擴展集 VM。
6. 從主要節點類型移除較舊的擴展集。
7. 移除與舊擴展集相關聯的負載平衡器。 在為新的擴展集設定新的公用 IP 位址和負載平衡器時，叢集會無法使用。  
8. 在變數中儲存與舊的主要節點類型擴展集相關聯的公用 IP 位址 DNS 設定，然後移除該公用 IP 位址。
9. 使用所刪除公用 IP 位址的 DNS 設定，來取代與新的主要節點類型擴展集相關聯的公用 IP 位址 DNS 設定。  叢集現在可再次連線了。
10. 從叢集中移除節點的節點狀態。  如果舊擴展集的持久性層級為銀級或金級，則系統會自動完成此步驟。
11. 如果您在上一個步驟中部署了具狀態應用程式，請確認應用程式可正常運作。

```powershell
# Variables.
$groupname = "sfupgradetestgroup"
$clusterloc="southcentralus"  
$subscriptionID="<your subscription ID>"

# sign in to your Azure account and select your subscription
Login-AzureRmAccount -SubscriptionId $subscriptionID 

# Create a new resource group for your deployment and give it a name and a location.
New-AzureRmResourceGroup -Name $groupname -Location $clusterloc

# Deploy the two node type cluster.
New-AzureRmResourceGroupDeployment -ResourceGroupName $groupname -TemplateParameterFile "C:\temp\cluster\Deploy-2NodeTypes-2ScaleSets.parameters.json" `
    -TemplateFile "C:\temp\cluster\Deploy-2NodeTypes-2ScaleSets.json" -Verbose

# Connect to the cluster and check the cluster health.
$ClusterName= "sfupgradetest.southcentralus.cloudapp.azure.com:19000"
$thumb="F361720F4BD5449F6F083DDE99DC51A86985B25B"

Connect-ServiceFabricCluster -ConnectionEndpoint $ClusterName -KeepAliveIntervalInSec 10 `
    -X509Credential `
    -ServerCertThumbprint $thumb  `
    -FindType FindByThumbprint `
    -FindValue $thumb `
    -StoreLocation CurrentUser `
    -StoreName My 

Get-ServiceFabricClusterHealth

# Deploy a new scale set into the primary node type.  Create a new load balancer and public IP address for the new scale set.
New-AzureRmResourceGroupDeployment -ResourceGroupName $groupname -TemplateParameterFile "C:\temp\cluster\Deploy-2NodeTypes-3ScaleSets.parameters.json" `
    -TemplateFile "C:\temp\cluster\Deploy-2NodeTypes-3ScaleSets.json" -Verbose

# Check the cluster health again. All 15 nodes should be healthy.
Get-ServiceFabricClusterHealth

# Disable the nodes in the original scale set.
$nodeNames = @("_NTvm1_0","_NTvm1_1","_NTvm1_2","_NTvm1_3","_NTvm1_4")

Write-Host "Disabling nodes..."
foreach($name in $nodeNames){
    Disable-ServiceFabricNode -NodeName $name -Intent RemoveNode -Force
}

Write-Host "Checking node status..."
foreach($name in $nodeNames){
 
    $state = Get-ServiceFabricNode -NodeName $name 

    $loopTimeout = 50

    do{
        Start-Sleep 5
        $loopTimeout -= 1
        $state = Get-ServiceFabricNode -NodeName $name
        Write-Host "$name state: " $state.NodeDeactivationInfo.Status
    }

    while (($state.NodeDeactivationInfo.Status -ne "Completed") -and ($loopTimeout -ne 0))
    

    if ($state.NodeStatus -ne [System.Fabric.Query.NodeStatus]::Disabled)
    {
        Write-Error "$name node deactivation failed with state" $state.NodeStatus
        exit
    }
}

# Remove the scale set
$scaleSetName="NTvm1"
Remove-AzureRmVmss -ResourceGroupName $groupname -VMScaleSetName $scaleSetName -Force
Write-Host "Removed scale set $scaleSetName"

$lbname="LB-sfupgradetest-NTvm1"
$oldPublicIpName="PublicIP-LB-FE-0"
$newPublicIpName="PublicIP-LB-FE-2"

# Store DNS settings of public IP address related to old Primary NodeType into variable 
$oldprimaryPublicIP = Get-AzureRmPublicIpAddress -Name $oldPublicIpName  -ResourceGroupName $groupname

$primaryDNSName = $oldprimaryPublicIP.DnsSettings.DomainNameLabel

$primaryDNSFqdn = $oldprimaryPublicIP.DnsSettings.Fqdn

# Remove Load Balancer related to old Primary NodeType. This will cause a brief period of downtime for the cluster
Remove-AzureRmLoadBalancer -Name $lbname -ResourceGroupName $groupname -Force

# Remove the old public IP
Remove-AzureRmPublicIpAddress -Name $oldPublicIpName -ResourceGroupName $groupname -Force

# Replace DNS settings of Public IP address related to new Primary Node Type with DNS settings of Public IP address related to old Primary Node Type
$PublicIP = Get-AzureRmPublicIpAddress -Name $newPublicIpName  -ResourceGroupName $groupname
$PublicIP.DnsSettings.DomainNameLabel = $primaryDNSName
$PublicIP.DnsSettings.Fqdn = $primaryDNSFqdn
Set-AzureRmPublicIpAddress -PublicIpAddress $PublicIP

# Check the cluster health
Get-ServiceFabricClusterHealth

# Remove node state for the deleted nodes.
foreach($name in $nodeNames){
    # Remove the node from the cluster
    Remove-ServiceFabricNodeState -NodeName $name -TimeoutSec 300 -Force
    Write-Host "Removed node state for node $name"
}
```

## <a name="adding-an-additional-scale-set-to-an-existing-cluster"></a>將額外的擴展集新增至現有的叢集
將新的 NodeType 虛擬機器擴展集新增至現有的叢集，與上述升級主要節點類型的作法類似，但您將不會使用相同的 NodeTypeRef、不會停用任何使用中的虛擬機器擴展集，也不會在不更新主要節點類型的情況下失去叢集可用性。 

NodeTypeRef 屬性會在虛擬機器擴展集 Service Fabric 擴充功能屬性內宣告：
```json
<snip>
"publisher": "Microsoft.Azure.ServiceFabric",
     "settings": {
     "clusterEndpoint": "[reference(parameters('clusterName')).clusterEndpoint]",
     "nodeTypeRef": "[parameters('vmNodeType2Name')]",
     "dataPath": "D:\\\\SvcFab",
     "durabilityLevel": "Silver",
<snip>
```

此外，您必須將這個新的節點類型新增至您的 Service Fabric 叢集資源：

```json
<snip>
"nodeTypes": [
      {
      "name": "[parameters('vmNodeType2Name')]",
      "applicationPorts": {
                "endPort": "[parameters('nt2applicationEndPort')]",
                "startPort": "[parameters('nt2applicationStartPort')]"
      },
      "clientConnectionEndpointPort": "[parameters('nt2fabricTcpGatewayPort')]",
      "durabilityLevel": "Silver",
       "ephemeralPorts": {
                "endPort": "[parameters('nt2ephemeralEndPort')]",
                "startPort": "[parameters('nt2ephemeralStartPort')]"
      },
      "httpGatewayEndpointPort": "[parameters('nt2fabricHttpGatewayPort')]",
      "isPrimary": false,
      "vmInstanceCount": "[parameters('nt2InstanceCount')]"
},
<snip>
```

## <a name="next-steps"></a>後續步驟
* 深入了解[應用程式延展性](service-fabric-concepts-scalability.md)。
* [將 Azure 叢集相應縮小或相應放大](service-fabric-tutorial-scale-cluster.md)。
* 使用 Fluent Azure 計算 SDK，[以程式設計方式調整 Azure 叢集](service-fabric-cluster-programmatic-scaling.md)。
* [將獨立叢集相應縮小或相應放大](service-fabric-cluster-windows-server-add-remove-nodes.md)。

