---
title: Service Fabric 叢集 Resource Manager - 放置原則 | Microsoft Docs
description: Service Fabric 服務的其他放置原則和規則的概觀
services: service-fabric
documentationcenter: .net
author: masnider
manager: timlt
editor: ''
ms.assetid: 5c2d19c6-dd40-4c4b-abd3-5c5ec0abed38
ms.service: Service-Fabric
ms.devlang: dotnet
ms.topic: conceptual
ms.tgt_pltfrm: NA
ms.workload: NA
ms.date: 08/18/2017
ms.author: masnider
ms.openlocfilehash: 66c51b08884c9d7a4d522c94f7b81774ec7a8bda
ms.sourcegitcommit: 266fe4c2216c0420e415d733cd3abbf94994533d
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/01/2018
ms.locfileid: "34641997"
---
# <a name="placement-policies-for-service-fabric-services"></a>Service Fabric 服務的放置原則
放置原則是在一些較罕見的特定情況下可用來掌管服務放置的額外規則。 這些情況的一些例子如下︰

- 您的 Service Fabric 叢集跨越一段地理距離 (例如多個內部部署資料中心) 或跨越多個 Azure 區域
- 您的環境跨越多個地緣政治或法定管制區，或是在其他一些情況下，您有需要強制執行的政策界限
- 由於距離很大，或是使用較慢或較不可靠的網路連結，而有通訊效能或延遲考量
- 您需要盡最大努力確保特定工作負載與其他工作負載共置，或放置在客戶附近

上述大部分需求會與叢集的實體配置 (以叢集的容錯網域表示) 一致。 

可協助解決這些情況的進階放置原則包括：

1. 無效的網域
2. 所需的網域
3. 慣用的網域
4. 不允許封裝複本

以下大部分控制可透過節點屬性和放置條件約束來設定，但其中有一些比較複雜。 為了簡化起見，Service Fabric 叢集資源管理員提供這些額外的放置原則。 放置原則可以依個別具名服務執行個體來設定， 還可以動態更新。

## <a name="specifying-invalid-domains"></a>指定無效的網域
**InvalidDomain** 放置原則可讓您指定某個容錯網域對特定服務是無效的。 此原則可確保特定的服務絕對不會在特定的區域中執行，例如，基於地緣政治或公司政策的緣故。 您可以透過個別原則指定多個無效的網域。

<center>
![無效的網域範例][Image1]
</center>

程式碼：

```csharp
ServicePlacementInvalidDomainPolicyDescription invalidDomain = new ServicePlacementInvalidDomainPolicyDescription();
invalidDomain.DomainName = "fd:/DCEast"; //regulations prohibit this workload here
serviceDescription.PlacementPolicies.Add(invalidDomain);
```

PowerShell：

```posh
New-ServiceFabricService -ApplicationName $applicationName -ServiceName $serviceName -ServiceTypeName $serviceTypeName –Stateful -MinReplicaSetSize 3 -TargetReplicaSetSize 3 -PartitionSchemeSingleton -PlacementPolicy @("InvalidDomain,fd:/DCEast”)
```
## <a name="specifying-required-domains"></a>指定所需的網域
所需的網域放置原則要求服務只能存在於指定的網域中。 您可以透過個別原則指定多個所需的網域。

<center>
![所需的網域範例][Image2]
</center>

程式碼：

```csharp
ServicePlacementRequiredDomainPolicyDescription requiredDomain = new ServicePlacementRequiredDomainPolicyDescription();
requiredDomain.DomainName = "fd:/DC01/RK03/BL2";
serviceDescription.PlacementPolicies.Add(requiredDomain);
```

PowerShell：

```posh
New-ServiceFabricService -ApplicationName $applicationName -ServiceName $serviceName -ServiceTypeName $serviceTypeName –Stateful -MinReplicaSetSize 3 -TargetReplicaSetSize 3 -PartitionSchemeSingleton -PlacementPolicy @("RequiredDomain,fd:/DC01/RK03/BL2")
```

## <a name="specifying-a-preferred-domain-for-the-primary-replicas-of-a-stateful-service"></a>指定具狀態服務之主要複本的慣用網域
「慣用的主要網域」會指定要放置「主要」複本的容錯網域。 若一切狀況良好，「主要」複本最後會在這個網域中。 如果網域或「主要」複本失敗或關閉，則「主要」複本會移至某些其他位置，在理想狀況下會是相同的網域。 如果這個新的位置並非慣用的網域，叢集資源管理員會儘快將它移回慣用的網域。 當然，此設定僅適用於具狀態服務。 如果叢集跨越 Azure 區域或多個資料中心，但具有選擇放置在特定位置的服務，此原則最有用。 讓「主要」複本靠近其使用者或其他服務有助於縮短延遲，特別是針對「主要」複本預設所處理的讀取作業。

<center>
![慣用的主要網域和容錯移轉][Image3]
</center>

```csharp
ServicePlacementPreferPrimaryDomainPolicyDescription primaryDomain = new ServicePlacementPreferPrimaryDomainPolicyDescription();
primaryDomain.DomainName = "fd:/EastUS/";
serviceDescription.PlacementPolicies.Add(primaryDomain);
```

PowerShell：

```posh
New-ServiceFabricService -ApplicationName $applicationName -ServiceName $serviceName -ServiceTypeName $serviceTypeName –Stateful -MinReplicaSetSize 3 -TargetReplicaSetSize 3 -PartitionSchemeSingleton -PlacementPolicy @("PreferredPrimaryDomain,fd:/EastUS")
```

## <a name="requiring-replica-distribution-and-disallowing-packing"></a>要求複本散佈，且不允許封裝
叢集狀況良好時，複本「通常」會散佈在容錯和升級網域之間。 不過，指定資料分割的多個複本有時會暫時封裝到單一網域。 例如，假設叢集有九個節點在三個容錯網域中 (fd:/0、fd:/1 和 fd:/2)。 假設您的服務有三個複本。 假設 fd:/1 和 fd:/2 中用於這些複本的節點停止運作。 叢集資源管理員通常會選擇這些相同容錯網域中的其他節點。 在此情況下，假設因為容量問題，這些網域中的其他節點都無效。 如果叢集資源管理員為這些複本建立替代項目，則必須選擇 fd:/0 中的節點。 不過，「這樣做」會導致違反容錯網域條件約束。 封裝複本會增加整個複本集停止運作或遺失的可能性。 

> [!NOTE]
> 如需一般情況下條件約束和條件約束優先順序的詳細資訊，請參閱[本主題](service-fabric-cluster-resource-manager-management-integration.md#constraint-priorities)。
>

如果您曾看過類似「`The Load Balancer has detected a Constraint Violation for this Replica:fabric:/<some service name> Secondary Partition <some partition ID> is violating the Constraint: FaultDomain`」的健康狀態訊息，表示您已遇到此狀況或類似的情況。 通常只有一或兩個複本會暫時封裝在一起。 只要少於指定網域中複本的法定數目，就是安全的。 封裝很罕見，但可能發生，這些情況通常是暫時性，因為節點會恢復。 如果節點持續關閉，而且叢集資源管理員需要建置替代項目，理想的容錯網域中通常會有其他節點可用。

某些工作負載即使封裝至更少的網域，也一律都有達到目標數目的複本。 這些工作負載打賭會同時全面發生永久性網域故障，通常可以復原本機狀態。 其他工作負載會提早停工，而不想冒著更正或遺失資料的風險。 大部分生產工作負載執行時會使用三個以上的複本、三個以上的容錯網域，以及每個容錯網域的許多有效節點。 因此，預設行為預設會允許網域封裝。 預設行為允許一般平衡和容錯移轉處理這些極端情況，即使這表示暫時的網域封裝也一樣。

如果您想要對指定的工作負載停用這類封裝，您可以在服務上指定 `RequireDomainDistribution` 原則。 設定此原則時，叢集資源管理員可確保來自相同資料分割的兩個複本不會在相同的容錯網域或升級網域中執行。

程式碼：

```csharp
ServicePlacementRequireDomainDistributionPolicyDescription distributeDomain = new ServicePlacementRequireDomainDistributionPolicyDescription();
serviceDescription.PlacementPolicies.Add(distributeDomain);
```

PowerShell：

```posh
New-ServiceFabricService -ApplicationName $applicationName -ServiceName $serviceName -ServiceTypeName $serviceTypeName –Stateful -MinReplicaSetSize 3 -TargetReplicaSetSize 3 -PartitionSchemeSingleton -PlacementPolicy @("RequiredDomainDistribution")
```

現在，是否可針對未跨越地理區域的叢集中的服務使用這些組態？ 可以，但也沒有充分的理由。 除非情況要求，否則請避免所需、無效和慣用的網域設定。 如果試著強制在單一機架中執行指定的工作負載，或偏好本機叢集的某些區段而不是其他區段，這樣並沒有任何意義。 不同的硬體設定應該分散至容錯網域，並透過一般放置條件約束和節點屬性來處理。

## <a name="next-steps"></a>後續步驟
- 如需服務設定的詳細資訊，請[深入了解設定服務](service-fabric-cluster-resource-manager-configure-services.md)

[Image1]:./media/service-fabric-cluster-resource-manager-advanced-placement-rules-placement-policies/cluster-invalid-placement-domain.png
[Image2]:./media/service-fabric-cluster-resource-manager-advanced-placement-rules-placement-policies/cluster-required-placement-domain.png
[Image3]:./media/service-fabric-cluster-resource-manager-advanced-placement-rules-placement-policies/cluster-preferred-primary-domain.png
