---
title: Azure Service Fabric 程式設計調整 | Microsoft Docs
description: 以程式設計方式根據自訂觸發程序相應縮小或放大 Azure Service Fabric 叢集
services: service-fabric
documentationcenter: .net
author: mjrousos
manager: jonjung
editor: ''
ms.assetid: ''
ms.service: service-fabric
ms.devlang: dotnet
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 01/23/2018
ms.author: mikerou
ms.openlocfilehash: ff02f79321823e42c25897e9de30dfbb6fac46b0
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46949605"
---
# <a name="scale-a-service-fabric-cluster-programmatically"></a>以程式設計方式調整 Service Fabric 叢集 

在 Azure 中執行的 Service Fabric 叢集是建立在虛擬機器擴展集之上。  [叢集調整](./service-fabric-cluster-scale-up-down.md)描述可如何以手動方式或使用自動調整規則來調整 Service Fabric 叢集。 本文描述如何使用 Fluent Azure 計算 SDK 來管理認證及相應縮小或相應放大叢集，這是更進階的案例。 如需概觀，請閱讀[以程式設計方式協調 Azure 調整作業](service-fabric-cluster-scaling.md#programmatic-scaling)。 

## <a name="manage-credentials"></a>管理認證
撰寫服務來處理調整的其中一項挑戰是，服務必須能夠不經互動式登入程序就存取虛擬機器擴展集資源。 如果調整服務是要修改自己的 Service Fabric 應用程式，存取 Service Fabric 叢集是很容易的事，但需要有認證才能存取擴展集。 若要登入，您可以使用以 [Azure CLI](https://github.com/azure/azure-cli) 建立的[服務主體](https://docs.microsoft.com/cli/azure/create-an-azure-service-principal-azure-cli)。

您可以透過下列步驟來建立服務主體︰

1. 以具有虛擬機器擴展集存取權的使用者身分登入 Azure CLI (`az login`)
2. 使用 `az ad sp create-for-rbac` 建立服務主體
    1. 記下 appId (在其他地方稱為「用戶端識別碼」)、名稱、密碼及租用戶，以供稍後使用。
    2. 您還需要訂用帳戶識別碼，若要檢視此識別碼，請使用 `az account list`

Fluent 計算程式庫可以使用這些認證來登入 (請注意，`IAzure` 這類核心 Fluent Azure 類型是在 [Microsoft.Azure.Management.Fluent](https://www.nuget.org/packages/Microsoft.Azure.Management.Fluent/) 套件中)，如下所示：

```csharp
var credentials = new AzureCredentials(new ServicePrincipalLoginInformation {
                ClientId = AzureClientId,
                ClientSecret = 
                AzureClientKey }, AzureTenantId, AzureEnvironment.AzureGlobalCloud);
IAzure AzureClient = Azure.Authenticate(credentials).WithSubscription(AzureSubscriptionId);

if (AzureClient?.SubscriptionId == AzureSubscriptionId)
{
    ServiceEventSource.Current.ServiceMessage(Context, "Successfully logged into Azure");
}
else
{
    ServiceEventSource.Current.ServiceMessage(Context, "ERROR: Failed to login to Azure");
}
```

登入之後，可以透過 `AzureClient.VirtualMachineScaleSets.GetById(ScaleSetId).Capacity` 來查詢擴展集的執行個體計數。

## <a name="scaling-out"></a>相應放大
使用 Fluent Azure 計算 SDK 時，只需要幾個呼叫就能將執行個體新增至虛擬機器擴展集

```csharp
var scaleSet = AzureClient.VirtualMachineScaleSets.GetById(ScaleSetId);
var newCapacity = (int)Math.Min(MaximumNodeCount, scaleSet.Capacity + 1);
scaleSet.Update().WithCapacity(newCapacity).Apply(); 
``` 

或者，也可以使用 PowerShell Cmdlet 來管理虛擬機器擴展集大小。 [`Get-AzureRmVmss`](https://docs.microsoft.com/powershell/module/azurerm.compute/get-azurermvmss) 可以擷取虛擬機器擴展集物件。 目前的容量可透過 `.sku.capacity` 屬性來取得。 將容量變更為所需的值之後，便可以使用 [`Update-AzureRmVmss`](https://docs.microsoft.com/powershell/module/azurerm.compute/update-azurermvmss) 命令來更新 Azure 中的虛擬機器擴展集。

和手動新增節點時一樣，若要啟動新的 Service Fabric 節點，您只需要新增擴展集執行個體，因為擴展集範本所含有的擴充功能可自動將新的執行個體加入 Service Fabric 叢集。 

## <a name="scaling-in"></a>相應縮小

相應縮小與相應放大類似。實際的虛擬機器擴展集變更幾乎完全相同。 但就如先前所討論的，Service Fabric 只會自動清除持久性為金級或銀級的已移除節點。 因此，在銅級持久性的相應縮小案例中，就必須與 Service Fabric 叢集互動，以關閉要移除的節點，然後移除其狀態。

關閉節點的準備工作涉及尋找要移除的節點 (最近新增的虛擬機器擴展集執行個體) 並加以停用。 虛擬機器擴展集執行個體會以其新增的順序編號，因此可以藉由比較節點名稱中的數字尾碼 (其會比對基礎的虛擬機器擴展集執行個體名稱) 來找到較新的節點。 

```csharp
using (var client = new FabricClient())
{
    var mostRecentLiveNode = (await client.QueryManager.GetNodeListAsync())
        .Where(n => n.NodeType.Equals(NodeTypeToScale, StringComparison.OrdinalIgnoreCase))
        .Where(n => n.NodeStatus == System.Fabric.Query.NodeStatus.Up)
        .OrderByDescending(n =>
        {
            var instanceIdIndex = n.NodeName.LastIndexOf("_");
            var instanceIdString = n.NodeName.Substring(instanceIdIndex + 1);
            return int.Parse(instanceIdString);
        })
        .FirstOrDefault();
```

在找到要移除的節點後，即可使用和稍早相同的 `FabricClient` 執行個體和 `IAzure` 執行個體來加以停用並移除。

```csharp
var scaleSet = AzureClient.VirtualMachineScaleSets.GetById(ScaleSetId);

// Remove the node from the Service Fabric cluster
ServiceEventSource.Current.ServiceMessage(Context, $"Disabling node {mostRecentLiveNode.NodeName}");
await client.ClusterManager.DeactivateNodeAsync(mostRecentLiveNode.NodeName, NodeDeactivationIntent.RemoveNode);

// Wait (up to a timeout) for the node to gracefully shutdown
var timeout = TimeSpan.FromMinutes(5);
var waitStart = DateTime.Now;
while ((mostRecentLiveNode.NodeStatus == System.Fabric.Query.NodeStatus.Up || mostRecentLiveNode.NodeStatus == System.Fabric.Query.NodeStatus.Disabling) &&
        DateTime.Now - waitStart < timeout)
{
    mostRecentLiveNode = (await client.QueryManager.GetNodeListAsync()).FirstOrDefault(n => n.NodeName == mostRecentLiveNode.NodeName);
    await Task.Delay(10 * 1000);
}

// Decrement VMSS capacity
var newCapacity = (int)Math.Max(MinimumNodeCount, scaleSet.Capacity - 1); // Check min count 

scaleSet.Update().WithCapacity(newCapacity).Apply(); 
```

與相應放大規模時相同，如果偏好使用指令碼處理方法，在這裡也可以使用修改虛擬機器擴展集容量的 PowerShell Cmdlet。 移除虛擬機器執行個體後，就可以移除 Service Fabric 節點狀態。

```csharp
await client.ClusterManager.RemoveNodeStateAsync(mostRecentLiveNode.NodeName);
```

## <a name="next-steps"></a>後續步驟

若要開始實作您自己的自動調整邏輯，請先熟悉下列概念和實用的 API：

- [以手動方式或透過自動調整規則進行調整](./service-fabric-cluster-scale-up-down.md)
- [適用於 .NET 的 Fluent Azure 管理程式庫](https://github.com/Azure/azure-sdk-for-net/tree/Fluent) (適用於與 Service Fabric 叢集的基礎虛擬機器擴展集互動)
- [System.Fabric.FabricClient](https://docs.microsoft.com/dotnet/api/system.fabric.fabricclient) (適用於與 Service Fabric 叢集及其節點互動)
