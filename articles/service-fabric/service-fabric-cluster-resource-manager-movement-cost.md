---
title: Service Fabric 叢集 Resource Manager：移動成本 | Microsoft Docs
description: Service Fabric 服務的移動成本概觀
services: service-fabric
documentationcenter: .net
author: masnider
manager: timlt
editor: ''
ms.assetid: f022f258-7bc0-4db4-aa85-8c6c8344da32
ms.service: Service-Fabric
ms.devlang: dotnet
ms.topic: conceptual
ms.tgt_pltfrm: NA
ms.workload: NA
ms.date: 08/18/2017
ms.author: masnider
ms.openlocfilehash: 74b61967a796fca22ab86918235f1def27a22f91
ms.sourcegitcommit: eb75f177fc59d90b1b667afcfe64ac51936e2638
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 05/16/2018
ms.locfileid: "34204918"
---
# <a name="service-movement-cost"></a>服務移動成本
「Service Fabric 叢集資源管理員」在嘗試判斷要對叢集進行哪些變更時會考量一個因素，就是這些變更的成本。 「成本」的概念是針對叢集可以改善多少來做取捨。 成本在移動服務進行平衡、重組和其他需求時納入考量因素。 目標是以最沒有干擾、最便宜的方式符合需求。 

移動服務會花費最少的 CPU 時間和網路頻寬。 對於具狀態服務，需要複製這些服務的狀態、耗用額外記憶體和磁碟。 將 Azure Service Fabric 叢集資源管理員帶來的解決方案成本降至最低，有助於確保不會花費不必要的叢集資源。 然而，也不想忽略可大幅改善叢集中資源配置的解決方案。

「叢集資源管理員」有兩種計算及限制成本的方式，在它嘗試管理叢集時亦是如此。 第一種機制只會計算進行的每次移動。 如果產生兩個平衡值 (分數) 幾乎相同的解決方案，「叢集資源管理員」就會採用成本最低 (移動總數) 的那一個。

此策略效果不錯。 但是若使用預設或靜態負載時，不太可能在任何複雜的系統中所有的移動都相等。 有些可能會昂貴許多。

## <a name="setting-move-costs"></a>設定移動成本 
您可以在服務建立時指定其預設移動成本：

PowerShell：

```posh
New-ServiceFabricService -ApplicationName $applicationName -ServiceName $serviceName -ServiceTypeName $serviceTypeName –Stateful -MinReplicaSetSize 3 -TargetReplicaSetSize 3 -PartitionSchemeSingleton -DefaultMoveCost Medium
```

C#： 

```csharp
FabricClient fabricClient = new FabricClient();
StatefulServiceDescription serviceDescription = new StatefulServiceDescription();
//set up the rest of the ServiceDescription
serviceDescription.DefaultMoveCost = MoveCost.Medium;
await fabricClient.ServiceManager.CreateServiceAsync(serviceDescription);
```

您也可以在服務建立之後，指定其移動成本或動態更新 MoveCost： 

PowerShell： 

```posh
Update-ServiceFabricService -Stateful -ServiceName "fabric:/AppName/ServiceName" -DefaultMoveCost High
```

C#：

```csharp
StatefulServiceUpdateDescription updateDescription = new StatefulServiceUpdateDescription();
updateDescription.DefaultMoveCost = MoveCost.High;
await fabricClient.ServiceManager.UpdateServiceAsync(new Uri("fabric:/AppName/ServiceName"), updateDescription);
```

## <a name="dynamically-specifying-move-cost-on-a-per-replica-basis"></a>以動態方式指定每個複本的移動成本

上述程式碼片段可以從服務本身外部一次指定整個服務的 MoveCost。 不過，當特定服務物件的移動成本隨著其生命週期變更時，移動成本最有用。 因為服務本身可能有在指定時間移動需要多少成本的最佳想法，所以有一個適用於服務的 API 可以報告執行階段期間的個別移動成本。 

C#：

```csharp
this.Partition.ReportMoveCost(MoveCost.Medium);
```

## <a name="impact-of-move-cost"></a>移動成本的影響
MoveCost 有四個層級：零、低、中和高。 除了零之外，MoveCosts 彼此具有相對性。 零移動成本表示移動是免費的，因此不應計入解決方案的分數。 將移動成本設定為 [高]，並「不」保證複本會待在一個地方。

<center>
![移動成本作為選取要移動之複本的因素][Image1]
</center>

MoveCost 可協助您在達成對等的平衡時，尋找整體導致最少中斷且最容易達成的解決方案。 服務的成本概念可相對於許多事項。 計算您的移動成本時最常見的因素為：

- 服務必須移動的狀態或資料量。
- 用戶端中斷連線的成本。 移動主要複本的成本通常高於移動次要複本的成本。
- 中斷執行中作業的成本。 某些資料存放區層級的作業，或是執行以回應用戶端呼叫的作業，它們的成本都很高。 在某個特定點之後，除非必要否則您不會想要停止它們。 因此，在作業持續期間，您可以提高此服務物件的移動成本以降低其移動的可能性。 當作業完成之後，您可以將成本設定回正常。

## <a name="enabling-move-cost-in-your-cluster"></a>在您的叢集中啟用移動成本
為了將更細微的 MoveCosts 納入考量，必須在叢集中啟用 MoveCost。 如果沒有此設定，則會使用計算移動的預設模式來計算 MoveCost，並且忽略 MoveCost 報告。


ClusterManifest.xml：

``` xml
        <Section Name="PlacementAndLoadBalancing">
            <Parameter Name="UseMoveCostReports" Value="true" />
        </Section>
```

獨立部署透過 ClusterConfig.json，Azure 託管叢集透過 Template.json：

```json
"fabricSettings": [
  {
    "name": "PlacementAndLoadBalancing",
    "parameters": [
      {
          "name": "UseMoveCostReports",
          "value": "true"
      }
    ]
  }
]
```

## <a name="next-steps"></a>後續步驟
- Service Fabric 叢集資源管理員會使用度量來管理叢集中的耗用量和容量。 若要深入了解度量及設定度量的方式，請參閱 [在 Service Fabric 中使用度量管理資源耗用量和負載](service-fabric-cluster-resource-manager-metrics.md)。
- 若要深入了解叢集資源管理員如何在叢集中進行管理和負載平衡，請查看 [平衡 Service Fabric 叢集](service-fabric-cluster-resource-manager-balancing.md)。

[Image1]:./media/service-fabric-cluster-resource-manager-movement-cost/service-most-cost-example.png
