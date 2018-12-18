---
title: Service Fabric 叢集 Resource Manager 中的節流 | Microsoft Docs
description: 了解如何設定 Service Fabric 叢集 Resource Manager 提供的節流。
services: service-fabric
documentationcenter: .net
author: masnider
manager: timlt
editor: ''
ms.assetid: 4a44678b-a5aa-4d30-958f-dc4332ebfb63
ms.service: Service-Fabric
ms.devlang: dotnet
ms.topic: conceptual
ms.tgt_pltfrm: NA
ms.workload: NA
ms.date: 08/18/2017
ms.author: masnider
ms.openlocfilehash: e9db1070066a2a02b72b5cc051e59d8b04dc9928
ms.sourcegitcommit: eb75f177fc59d90b1b667afcfe64ac51936e2638
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 05/16/2018
---
# <a name="throttling-the-service-fabric-cluster-resource-manager"></a>對 Service Fabric 叢集資源管理員進行節流
即使您已經正確設定叢集資源管理員，也可以中斷叢集。 例如，叢集可能會同時發生節點和容錯網域失敗，如果這些失敗是在升級時發生會造成什麼後果？ 叢集資源管理員會不斷嘗試讓一切恢復正常，耗用叢集的資源來試著重新組織並修正叢集。 節流有助於提供最後防線，讓叢集利用資源來穩定系統，也就是讓節點恢復運作、修復網路磁碟分割，以及部署經過修正的程式碼。

為了協助達成這幾種情況，Service Fabric 叢集資源管理員包含幾種節流。 這些節流功能個個都是強大無比的工具。 一般來說，如果您沒有仔細規劃並測試，就請不要對其進行變更。

如果您變更叢集資源管理員的節流功能，請針對您預期的實際負載來調整它們。 您可能會認定，即使使用節流會在某些情況下導致叢集需要更長的時間才能穩定下來，但您還是需要備有節流功能。 您必須進行測試才能決定正確的節流值。 節流值必須夠高，才能讓叢集在合理的時間內對變更做出回應，但也要夠低，才能真正避免過度耗用資源。 

由於客戶環境中的資源已經有限，因此我們發現他們大多會實施節流措施。 例如，限制個別節點的網路頻寬，或由於輸送量有限而不讓磁碟同時建置許多具狀態複本。 若不實施節流措施，將會有大量作業爭用這些資源，從而導致作業失敗或速度變慢。 有鑑於此，客戶被迫實施了節流措施，卻也知道叢集需要更久的時間才能達到穩定狀態。 客戶也了解進行節流時，最後可能會在整體可靠性降低的情況下運作。


## <a name="configuring-the-throttles"></a>設定節流

Service Fabric 有兩種機制可供節制移動的複本數目。 Service Fabric 5.7 之前就存在的預設機制是以允許移動的數量固定不變的方式來進行節流。 但這種方式不適用於所有規模的叢集。 對於大型叢集來說尤其如此，這種機制的預設值可能會太小，而導致即使這樣的值有其必要的情況下大幅降低平衡速度，然而這種機制對於較小的叢集來說卻沒有效果。 這個舊機制已由百分比式的節流所取代，後者對於服務和節點數目經常變更的動態叢集會有較好的調整效果。

這些節流值是以叢集中複本數目的百分比作為根據。 百分比式的節流能夠表達這樣的規則：「不要在為期 10 分鐘的間隔內移動超過 10% 的複本」(舉例)。

百分比式節流的組態設定如下：

  - GlobalMovementThrottleThresholdPercentage - 叢集中隨時允許移動的數目上限，以「叢集中複本總數的百分比」來表示。 0 表示沒有限制。 預設值為 0。 如果您同時指定此設定和 GlobalMovementThrottleThreshold，則系統會使用更保守的限制。
  - GlobalMovementThrottleThresholdPercentageForPlacement - 放置階段允許移動的數目上限，以「叢集中複本總數的百分比」來表示。 0 表示沒有限制。 預設值為 0。 如果您同時指定此設定和 GlobalMovementThrottleThresholdForPlacement，則系統會使用更保守的限制。
  - GlobalMovementThrottleThresholdPercentageForBalancing - 平衡階段允許移動的數目上限，以「叢集中複本總數的百分比」來表示。 0 表示沒有限制。 預設值為 0。 如果您同時指定此設定和 GlobalMovementThrottleThresholdForBalancing，則系統會使用更保守的限制。

在指定節流百分比時，請以 0.05 來指定 5%。 用來控管這些節流的間隔是 GlobalMovementThrottleCountingInterval，此設定是以秒為單位來指定。


``` xml
<Section Name="PlacementAndLoadBalancing">
     <Parameter Name="GlobalMovementThrottleThresholdPercentage" Value="0" />
     <Parameter Name="GlobalMovementThrottleThresholdPercentageForPlacement" Value="0" />
     <Parameter Name="GlobalMovementThrottleThresholdPercentageForBalancing" Value="0" />
     <Parameter Name="GlobalMovementThrottleCountingInterval" Value="600" />
</Section>
```

獨立部署透過 ClusterConfig.json，Azure 託管叢集透過 Template.json：

```json
"fabricSettings": [
  {
    "name": "PlacementAndLoadBalancing",
    "parameters": [
      {
          "name": "GlobalMovementThrottleThresholdPercentage",
          "value": "0.0"
      },
      {
          "name": "GlobalMovementThrottleThresholdPercentageForPlacement",
          "value": "0.0"
      },
      {
          "name": "GlobalMovementThrottleThresholdPercentageForBalancing",
          "value": "0.0"
      },
      {
          "name": "GlobalMovementThrottleCountingInterval",
          "value": "600"
      }
    ]
  }
]
```

### <a name="default-count-based-throttles"></a>預設計數型節流
我們提供了這項資訊，以免您還有較舊的叢集或是仍在已升級的叢集中保有這些組態。 一般情況下，建議您使用上述的百分比式節流來取代這些節流。 百分比式節流依預設會停用，因此這些節流仍舊會是叢集的預設節流，除非您將它們停用，並以百分比式的節流來加以取代。 

  - GlobalMovementThrottleThreshold – 這項設定會控制叢集在一段時間內的移動總數。 這段時間的長度會以 GlobalMovementThrottleCountingInterval 來指定，單位為秒。 GlobalMovementThrottleThreshold 的預設值為 1000，GlobalMovementThrottleCountingInterval 的預設值則為 600。
  - MovementPerPartitionThrottleThreshold – 這項設定會控制任何服務分割區在一段時間內的移動總數。 這段時間的長度會以 MovementPerPartitionThrottleCountingInterval 來指定，單位為秒。 MovementPerPartitionThrottleThreshold 的預設值為 50，MovementPerPartitionThrottleCountingInterval 的預設值則為 600。

這些節流的組態會遵循和百分比式節流相同的模式。

## <a name="next-steps"></a>後續步驟
- 若要了解叢集資源管理員如何管理並平衡叢集中的負載，請查看關於 [平衡負載](service-fabric-cluster-resource-manager-balancing.md)
- 叢集資源管理員有許多描述叢集的選項。 若要深入了解這些選項，請參閱關於[描述 Service Fabric 叢集](service-fabric-cluster-resource-manager-cluster-description.md)一文
