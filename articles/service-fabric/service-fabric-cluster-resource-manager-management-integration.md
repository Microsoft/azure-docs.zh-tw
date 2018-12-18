---
title: Service Fabric 叢集 Resource Manager - 管理整合 | Microsoft Docs
description: 叢集資源管理員和 Service Fabric 管理之間的整合點概觀。
services: service-fabric
documentationcenter: .net
author: masnider
manager: timlt
editor: ''
ms.assetid: 956cd0b8-b6e3-4436-a224-8766320e8cd7
ms.service: Service-Fabric
ms.devlang: dotnet
ms.topic: conceptual
ms.tgt_pltfrm: NA
ms.workload: NA
ms.date: 08/18/2017
ms.author: masnider
ms.openlocfilehash: 7a1bab75521730f7e80e5b86112bbb0aed129f88
ms.sourcegitcommit: ebb460ed4f1331feb56052ea84509c2d5e9bd65c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/24/2018
ms.locfileid: "42917869"
---
# <a name="cluster-resource-manager-integration-with-service-fabric-cluster-management"></a>叢集資源管理員與 Service Fabric 叢集管理整合
Service Fabric 叢集資源管理員不會促使 Service Fabric 升級，但有所關聯。 叢集資源管理員協助管理的第一種方法是追蹤所需的叢集狀態及其內部的服務。 當叢集資源管理員無法讓叢集處於所需的設定時，它會送出健全狀況報告。 例如，如果容量不足，叢集資源管理員會發出健康情況警告和錯誤，指出問題所在。 整合的另一方面必定與升級方式有關。 在升級期間，叢集資源管理員會稍微改變其行為。  

## <a name="health-integration"></a>健康狀態整合
叢集資源管理員會持續追蹤您為服務定義的規則。 對於節點上和叢集中的每個計量，以及整個叢集中的每個計量，其也會追蹤剩餘容量。 如果無法滿足這些規則，或容量不足，則會發出健康情況警告和錯誤。 例如，若節點超出容量，叢集資源管理員會移動服務來嘗試解決情況。 如果無法解決情況，則會發出健全狀況警告，指出哪一個節點超出容量和涉及哪些計量。

Resource Manager 健全狀況警告的另一個例子是違反放置條件約束。 例如，若您已定義放置條件約束 (例如 `“NodeColor == Blue”`)，而資源管理員偵測到該條件約束的違規情形時，會發出健康情況警告。 這適用於自訂條件約束和預設條件約束 (例如容錯網域和升級網域條件約束)。

以下是這類健康狀態報告的範例。 在此情況下，健全狀況報告是針對系統服務的其中一個資料分割。 健全狀況訊息指出該資料分割的複本暫時封裝至太少的升級網域。

```posh
PS C:\Users\User > Get-ServiceFabricPartitionHealth -PartitionId '00000000-0000-0000-0000-000000000001'


PartitionId           : 00000000-0000-0000-0000-000000000001
AggregatedHealthState : Warning
UnhealthyEvaluations  :
                        Unhealthy event: SourceId='System.PLB', Property='ReplicaConstraintViolation_UpgradeDomain', HealthState='Warning', ConsiderWarningAsError=false.

ReplicaHealthStates   :
                        ReplicaId             : 130766528804733380
                        AggregatedHealthState : Ok

                        ReplicaId             : 130766528804577821
                        AggregatedHealthState : Ok

                        ReplicaId             : 130766528854889931
                        AggregatedHealthState : Ok

                        ReplicaId             : 130766528804577822
                        AggregatedHealthState : Ok

                        ReplicaId             : 130837073190680024
                        AggregatedHealthState : Ok

HealthEvents          :
                        SourceId              : System.PLB
                        Property              : ReplicaConstraintViolation_UpgradeDomain
                        HealthState           : Warning
                        SequenceNumber        : 130837100116930204
                        SentAt                : 8/10/2015 7:53:31 PM
                        ReceivedAt            : 8/10/2015 7:53:33 PM
                        TTL                   : 00:01:05
                        Description           : The Load Balancer has detected a Constraint Violation for this Replica: fabric:/System/FailoverManagerService Secondary Partition 00000000-0000-0000-0000-000000000001 is
                        violating the Constraint: UpgradeDomain Details: UpgradeDomain ID -- 4, Replica on NodeName -- Node.8 Currently Upgrading -- false Distribution Policy -- Packing
                        RemoveWhenExpired     : True
                        IsExpired             : False
                        Transitions           : Ok->Warning = 8/10/2015 7:13:02 PM, LastError = 1/1/0001 12:00:00 AM
```

以下是此健康狀態訊息要告訴我們的事情︰

1. 所有複本本身均狀況良好：每個都有 AggregatedHealthState : Ok
2. 目前違反升級網域發佈條件約束。 這表示特定升級網域擁有這個磁碟的過多分割複本。
3. 哪個節點包含造成違規的複本。 在此案例中是名為「Node.8」的節點。
4. 無論此分割區是否正在升級 (「Currently Upgrading -- false」)
5. 此服務的發佈原則：「Distribution Policy -- Packing」。 這是由`RequireDomainDistribution`[放置原則](service-fabric-cluster-resource-manager-advanced-placement-rules-placement-policies.md#requiring-replica-distribution-and-disallowing-packing)控管。 「封裝」表示在此情況下_不_需要 DomainDistribution，因此可知道並未替該服務指定放置原則。 
6. 報告時間 - 2015 年 8 月 10 日下午 7:13:02

這種資訊會引出在生產環境中出現的警示，讓您知道已發生問題，也會用來偵測和停止不正確的升級。 在此情況下，我們需要查明 Resource Manager 為何一定要將複本封裝至升級網域。 例如，封裝是暫時性的，因為其他升級網域中的節點已關閉。

假設叢集資源管理員嘗試放置某些服務，但沒有任何可行的解決方案。 無法放置服務時，通常是下列其中一個原因所致：

1. 某個暫時性情況導致無法正確地放置此服務執行個體或複本
2. 未滿足服務的放置需求。

在這些案例中，叢集資源管理員的健康情況報告有助於判斷為什麼無法放置服務。 我們將此程序稱為「條件約束消除序列」。 在此期間，系統會逐步解說會影響服務與記錄的已設定條件約束，以及它們所消除的項目。 如此一來，當項目無法放置時，您可以看到哪些節點已經消除及消除原因。

## <a name="constraint-types"></a>條件約束類型
讓我們討論這些健康情況報告中每個不同的條件約束。 無法放置複本時，會看到與這些條件約束有關的健康情況訊息。

* **ReplicaExclusionStatic** 和 **ReplicaExclusionDynamic**：這些條件約束指出解決方案遭拒，因為來自相同分割區的兩個服務物件必須放置在相同節點上。 不允許這麼做是因為該節點的失敗會過度影響該分割區。 ReplicaExclusionStatic 和 ReplicaExclusionDynamic 幾乎是完全相同的規則，兩者之間的差異並無太大影響。 如果您看到的條件約束消除序列包含 ReplicaExclusionStatic 或 ReplicaExclusionDynamic 條件約束，則是叢集資源管理員認為節點不足。 這需要其他解決方案使用這些不被允許的無效位置。 序列中的其他條件約束通常會告訴我們為什麼要先消除節點。
* **PlacementConstraint**︰如果您看到此訊息，就表示我們已消除一些節點，因為它們不符合服務的放置條件約束。 我們會在此此訊息中描繪出目前所設定的放置條件約束。 如果您已定義放置條件約束，則這是正常情形。 不過，如果放置條件約束錯誤，而造成過多節點遭消除，您就應該注意。
* **NodeCapacity**︰這個條件約束表示叢集資源管理員無法將複本放在指出的節點上，因為這樣會導致這些節點超出容量。
* **Affinity**︰這個條件約束表示我們無法將複本放在受影響的節點上，因為這會導致違反同質性條件約束。 如需同質性的詳細資訊，請參閱[這篇文章](service-fabric-cluster-resource-manager-advanced-placement-rules-affinity.md)
* **FaultDomain** 和 **UpgradeDomain**︰如果將複本放在指出的節點上會導致複本封裝在特定的容錯網域或升級網域中，此條件約束就會消除節點。 [容錯與升級網域條件約束及產生的行為](service-fabric-cluster-resource-manager-cluster-description.md)
* **PreferredLocation**︰您通常不會看到這個條件約束導致從解決方案中移除節點，因為該條件約束預設僅限用於最佳化。 慣用的位置條件約束也會在升級期間出現。 在升級期間，它用於將服務移回它們在升級開始時所在的位置。

## <a name="blocklisting-nodes"></a>封鎖節點
封鎖節點時，叢集資源管理員會報告另一個健康情況訊息。 您可以將封鎖視為自動套用的暫時性條件約束。 啟動該服務類型的執行個體時，發生重複失敗的節點會遭封鎖。 節點是依照服務類型來封鎖。 某個服務類型的節點可能會被封鎖，另一個服務類型的節點則未被封鎖。 

通常會在開發期間發生封鎖：一些錯誤導致服務主機在啟動時當機， Service Fabric 多次嘗試建立服務主機，但是持續失敗。 經過多次嘗試後，節點被封鎖，而叢集資源管理員會嘗試在其他位置建立服務。 如果多個節點持續發生同樣的失敗，最終可能導致叢集中的所有有效節點被封鎖。 封鎖也會移除許多節點，導致數量不足以無法成功啟動服務來達到所需的級別。 您通常會看見叢集資源管理員出現其他錯誤或警告，顯示服務低於所需的複本或執行個體計數，也會看到健康情況訊息顯示出最先導致封鎖的失敗。

封鎖不是永久情況。 經過幾分鐘後，便會從封鎖清單中移除節點，且 Service Fabric 會再次啟動該節點上的服務。 如果服務持續失敗，會再次封鎖該服務類型的節點。 

### <a name="constraint-priorities"></a>條件約束優先順序

> [!WARNING]
> 不建議變更條件約束優先順序，因為可能對叢集造成顯著的負面影響。 下列提供預設條件約束優先順序及其行為的參考資訊。 
>

所有這些條件約束可能會讓您覺得：「嘿，對我的系統來說，預設網域條件約束是最重要的。 為了確保不會違反預設網域條件約束，我願意違反其他條件約束。」

可以使用不同的優先順序等級來設定條件約束。 它們是：

   - 「硬性」(0)
   - 「彈性」(1)
   - 「最佳化」(2)
   - 「關閉」(-1)。 
   
大部分的條件約束預設是設定為硬性條件約束。

通常不會變更條件約束的優先順序。 有時候需要變更條件約束的優先順序，通常是為了解決影響環境的其他錯誤或行為。 大致上，條件約束優先順序基礎結構的彈性可以運作的非常好，但不會經常需要用到。 在大部分情況下，一切都按照預設優先順序安排。 

優先順序等級不表示會_違反_指定的條件約束，也不表示會一律符合條件約束。 條件約束優先順序會定義強制執行條件約束的順序。 無法滿足所有條件約束時，會以優先順序決定權衡取捨。 除非環境中有其他正在進行的項目，否則通常可以滿足所有條件約束。 導致違反條件約束的一些情節範例包括衝突的條件約束，或大量的並行失敗。

在進階情況中，可以變更條件約束優先順序。 例如，假設為了解決節點容量問題，而必須確保一律違反同質性。 為了達到此目的，您可以將同質性條件約束的優先順序設為「軟式」(1)，並將容量條件約束維持設為「硬式」(0)。

下列組態檔中指定了不同條件約束的預設優先順序值：

ClusterManifest.xml

```xml
        <Section Name="PlacementAndLoadBalancing">
            <Parameter Name="PlacementConstraintPriority" Value="0" />
            <Parameter Name="CapacityConstraintPriority" Value="0" />
            <Parameter Name="AffinityConstraintPriority" Value="0" />
            <Parameter Name="FaultDomainConstraintPriority" Value="0" />
            <Parameter Name="UpgradeDomainConstraintPriority" Value="1" />
            <Parameter Name="PreferredLocationConstraintPriority" Value="2" />
        </Section>
```

獨立部署透過 ClusterConfig.json，Azure 託管叢集透過 Template.json：

```json
"fabricSettings": [
  {
    "name": "PlacementAndLoadBalancing",
    "parameters": [
      {
          "name": "PlacementConstraintPriority",
          "value": "0"
      },
      {
          "name": "CapacityConstraintPriority",
          "value": "0"
      },
      {
          "name": "AffinityConstraintPriority",
          "value": "0"
      },
      {
          "name": "FaultDomainConstraintPriority",
          "value": "0"
      },
      {
          "name": "UpgradeDomainConstraintPriority",
          "value": "1"
      },
      {
          "name": "PreferredLocationConstraintPriority",
          "value": "2"
      }
    ]
  }
]
```

## <a name="fault-domain-and-upgrade-domain-constraints"></a>容錯網域和升級網域的條件約束
叢集資源管理員想要保留遍佈於容錯網域和升級網域的服務。 它會將此設定為叢集資源管理員的引擎內的條件約束。 如需其使用方式及特定行為的詳細資訊，請參閱[叢集設定](service-fabric-cluster-resource-manager-cluster-description.md#fault-and-upgrade-domain-constraints-and-resulting-behavior)一文。

叢集資源管理員可能需要將一些複本封裝至升級網域，以處理升級、失敗或其他條件約束違規情形。 通常，只有當系統中發生許多失敗或其他問題，導致無法正確放置時，才會封裝到容錯網域或升級網域。 如果即使在這類情況下也想要避免進行封裝，可以利用`RequireDomainDistribution`[放置原則](service-fabric-cluster-resource-manager-advanced-placement-rules-placement-policies.md#requiring-replica-distribution-and-disallowing-packing)。 請注意，這可能會產生影響服務可用性和可靠性的副作用，請審慎考慮。

如果已正確設定環境，則會完全遵守所有條件約束，甚至在升級期間也是如此。 關鍵在於叢集資源管理員會監看您的條件約束， 在偵測到違規時會立即回報，並嘗試更正問題。

## <a name="the-preferred-location-constraint"></a>慣用的位置條件約束
PreferredLocation 條件約束稍有不同，因為它有兩種不同的用途。 這個條件約束的其中一種用法是在應用程式升級期間。 叢集資源管理員會在升級期間自動管理這個條件約束， 用來確定複本在升級完成時會傳回到初始位置。 PreferredLocation 條件約束的另一種用法與[`PreferredPrimaryDomain`放置原則](service-fabric-cluster-resource-manager-advanced-placement-rules-placement-policies.md)有關。 兩者都屬於最佳化，因此 PreferredLocation 條件約束是唯一預設為「最佳化」的條件約束。

## <a name="upgrades"></a>升級
在應用程式和叢集升級過程中，叢集資源管理員也有所幫助，它在這期間有兩項作業︰

* 確保不影響叢集的規則
* 嘗試協助順利地進行升級

### <a name="keep-enforcing-the-rules"></a>繼續強制執行規則
規則是需要注意的重點 – 在升級期間仍會強制執行嚴格的條件約束 (如放置條件約束和容量)。 放置條件約束可確保工作負載只在允許的位置執行，即在升級期間也一樣。 服務受到極大的條件約束時，升級可能需要較長的時間。 服務或執行服務的節點為了更新而關閉時，可能有少數幾個選項可供選擇。

### <a name="smart-replacements"></a>聰明取代
升級開始時，Resource Manager 會取得叢集目前配置方式的快照集。 每次升級網域完成時，會嘗試將該升級網域中的服務恢復為原始排列方式。 如此一來，在升級期間，服務最多會進行兩次轉換。 一次是從受影響的節點移出服務，另一次是將服務移入。 讓叢集或服務回到升級之前的狀態，也可確保升級不會影響叢集配置。 

### <a name="reduced-churn"></a>減少流失
升級期間發生還會發生另一件事，就是叢集資源管理員關閉平衡作業。 阻止平衡可避免對升級本身做出不必要的反應，例如將服務移入已清空而不必升級的節點。 如果此次升級是「叢集」升級，則整個叢集在升級期間不會處於平衡狀態。 條件約束檢查會持續作用，只會將計量主動平衡類型的移動停用。

### <a name="buffered-capacity--upgrade"></a>緩衝處理的容量和升級
通常，即使叢集整體受條件約束或接近滿載，您也希望升級完成。 在升級期間，叢集容量的管理比平常更重要。 根據升級網域的數目，叢集內展開升級時，5% 到 20% 的容量必須移轉。 工作必須移至別處。 [緩衝容量](service-fabric-cluster-resource-manager-cluster-description.md#buffered-capacity)的概念在此真正派上用場。 在正常作業期間，系統會採用緩衝處理的容量。 在升級期間，叢集資源管理員可能會視需要而使用到節點的所有容量 (消耗緩衝區)。

## <a name="next-steps"></a>後續步驟
* 從頭開始，並 [取得 Service Fabric 叢集資源管理員的簡介](service-fabric-cluster-resource-manager-introduction.md)
