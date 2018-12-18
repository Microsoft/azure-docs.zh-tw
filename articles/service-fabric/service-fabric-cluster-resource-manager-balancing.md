---
title: 平衡 Azure Service Fabric 叢集 | Microsoft Docs
description: 使用 Azure Service Fabric 叢集資源管理員平衡叢集的簡介。
services: service-fabric
documentationcenter: .net
author: masnider
manager: timlt
editor: ''
ms.assetid: 030b1465-6616-4c0b-8bc7-24ed47d054c0
ms.service: Service-Fabric
ms.devlang: dotnet
ms.topic: conceptual
ms.tgt_pltfrm: NA
ms.workload: NA
ms.date: 08/18/2017
ms.author: masnider
ms.openlocfilehash: 5d2f195c50750a5c7685f62c909f77b2960613e6
ms.sourcegitcommit: eb75f177fc59d90b1b667afcfe64ac51936e2638
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 05/16/2018
ms.locfileid: "34213141"
---
# <a name="balancing-your-service-fabric-cluster"></a>平衡 Service Fabric 叢集
「Service Fabric 叢集資源管理員」支援動態負載變更、因應節點或服務的新增或移除。 它也會自動修正條件約束違規，以及主動重新平衡叢集。 但是這些動作執行的頻率，以及觸發它們的項目是什麼？

叢集資源管理員所執行的工作有三個不同的類別。 如下：

1. 放置 - 這個階段涉及安置任何遺漏的具狀態複本或無狀態執行個體。 放置包含新服務，也包含處理已失敗的具狀態複本或無狀態執行個體。 刪除和捨棄複本或執行個體都是在這裡處理。
2. 條件約束檢查 – 這個階段會檢查並更正系統內不同放置條件約束 (規則) 的違規情形。 規則範例包括像是確保節點不超出容量，以及符合服務的放置條件約束。
3. 平衡 - 這個階段會根據為不同計量設定的所需平衡層級，查看是否有必要使用重新平衡。 如果有必要，就會嘗試在叢集中尋找更平衡的安排方式。

## <a name="configuring-cluster-resource-manager-timers"></a>設定叢集資源管理員計時器
與平衡相關的第一組控制項是一組計時器。 這些計時器控管叢集資源管理員檢查叢集並且採取矯正措施的頻率。

「叢集資源管理員」可執行的每個不同類型修正，都是由控管其頻率的不同計時器所控制。 當每個計時器啟動時，便會排程工作。 根據預設，Resource Manager 會：

* 每 1/10 秒掃描一次其狀態並套用更新 (例如記錄某個節點已關閉)
* 設定放置檢查旗標 
* 設定每秒條件約束檢查旗標
* 每 5 秒設定一次平衡旗標。

控管這些計時器的設定範例如下：

ClusterManifest.xml：

``` xml
        <Section Name="PlacementAndLoadBalancing">
            <Parameter Name="PLBRefreshGap" Value="0.1" />
            <Parameter Name="MinPlacementInterval" Value="1.0" />
            <Parameter Name="MinConstraintCheckInterval" Value="1.0" />
            <Parameter Name="MinLoadBalancingInterval" Value="5.0" />
        </Section>
```

獨立部署透過 ClusterConfig.json，Azure 託管叢集透過 Template.json：

```json
"fabricSettings": [
  {
    "name": "PlacementAndLoadBalancing",
    "parameters": [
      {
          "name": "PLBRefreshGap",
          "value": "0.10"
      },
      {
          "name": "MinPlacementInterval",
          "value": "1.0"
      },
      {
          "name": "MinConstraintCheckInterval",
          "value": "1.0"
      },
      {
          "name": "MinLoadBalancingInterval",
          "value": "5.0"
      }
    ]
  }
]
```

目前「叢集資源管理員」只會依序一次執行這些動作其中之一。 這就是為什麼我們將這些計時器稱為「最小間隔」，計時器關閉時所採取的動作稱為「設定旗標」。 例如，「叢集資源管理員」在平衡叢集之前，會先處理要建立服務的擱置中要求。 如您所見，「叢集資源管理員」會依據指定的預設時間間隔，掃描它需要經常執行的一切項目。 通常這表示在每個步驟所做的一組變更通常很小。 經常進行少量變更可讓「叢集資源管理員」快速因應叢集中發生的情況。 由於許多相同類型的事件易於同時發生，因此預設計時器提供一些批次處理。 

例如，當節點失敗時，他們可以一次對整個容錯網域執行這個動作。 所有失敗會在 *PLBRefreshGap* 之後的下一個狀態更新期間擷取。 更正會在下列位置、條件約束檢查以及平衡執行期間決定。 「叢集資源管理員」預設不會將叢集中數小時的變更整個掃描一遍，然後嘗試一次處理所有變更。 這麼做會導致變換量激增。

「叢集資源管理員」也需要一些其他資訊，才能判斷叢集是否處於不平衡的狀態。 因此，我們有其他兩個的設定︰「平衡臨界值」和「活動臨界值」。

## <a name="balancing-thresholds"></a>平衡臨界值
平衡臨界值是觸發重新平衡的主要控制項。 計量的平衡臨界值是一個_比率_。 如果負載最多之節點的計量負載除以負載最少之節點的負載量超過該計量的*平衡臨界值*，此叢集就會被視為不平衡。 因此，下一次「叢集資源管理員」進行檢查時，就會觸發平衡作業。 MinLoadBalancingInterval 計時器會定義當需要重新平衡時，「叢集資源管理員」的檢查頻率。 檢查並不意謂著有發生任何事情。 

平衡臨界值會根據每個度量定義為叢集定義的一部分。 如需有關計量的詳細資訊，請參閱[這篇文章](service-fabric-cluster-resource-manager-metrics.md)。

ClusterManifest.xml

```xml
    <Section Name="MetricBalancingThresholds">
      <Parameter Name="MetricName1" Value="2"/>
      <Parameter Name="MetricName2" Value="3.5"/>
    </Section>
```

獨立部署透過 ClusterConfig.json，Azure 託管叢集透過 Template.json：

```json
"fabricSettings": [
  {
    "name": "MetricBalancingThresholds",
    "parameters": [
      {
          "name": "MetricName1",
          "value": "2"
      },
      {
          "name": "MetricName2",
          "value": "3.5"
      }
    ]
  }
]
```

<center>
![平衡臨界值範例][Image1]
</center>

在此範例中，每個服務皆取用一單位的某個計量。 在上半部的範例中，節點的負載上限為 5，而下限為 2。 假設此計量的平衡臨界值為 3。 由於叢集中的比率是 5/2 = 2.5，小於指定的平衡臨界值 3，因此叢集處於平衡狀態。 當「叢集資源管理員」進行檢查時，不會觸發任何平衡作業。

在下半部的範例中，節點的負載上限為 10，而下限為 2，所以比率為 5。 5 大於該計量的指定平衡臨界值 3。 因此，下一次引發平衡計時器時，便會排定執行重新平衡。 在類似這樣的情況中，有些負載通常會分散到 Node3。 由於「Service Fabric 叢集資源管理員」並不使用窮盡方法，因此有些負載也可能分散到 Node2。 

<center>
![平衡臨界值範例動作][Image2]
</center>

> [!NOTE]
> 「平衡」會處理兩個不同的策略來管理叢集中的負載。 叢集資源管理員使用的預設策略是在叢集的節點之間分散負載。 其他的策略是[重組](service-fabric-cluster-resource-manager-defragmentation-metrics.md)。 重組是在相同平衡執行期間執行。 平衡和重組策略可以用於相同叢集中不同的計量。 服務可以同時有平衡和重組計量。 對於重組計量，當叢集中的負載比率_低於_平衡臨界值時，會觸發重新平衡。 
>

使數據低於平衡臨界值並不是一個明確的目標。 平衡臨界值只是*觸發程序*。 當平衡執行時，叢集資源管理員會判斷可以進行哪些增強功能，如果有的話。 因為平衡搜尋開始並不代表任何項目移動。 有時候叢集過於受到修正的限制而不平衡。 或者，改進需要的移動太[昂貴](service-fabric-cluster-resource-manager-movement-cost.md))。

## <a name="activity-thresholds"></a>活動臨界值
有時候，雖然節點處於相對的不平衡狀態，但叢集中的負載「總量」  卻很低。 缺乏負載可能是暫時性的下跌情況，或是因為叢集是新的且才剛啟動而已。 不論是上述哪一種情況，您可能都不想花時間平衡叢集，因為能獲得的好處很少。 如果叢集進行平衡作業，您將需要花費網路和計算資源將東西四處移動，但卻不會產生任何大型的「絕對」差異。 為了避免不必要的移動，出現了另一種控制方式，稱為「活動臨界值」。 「活動臨界值」可讓您為活動指定某種絕對下限。 如果沒有任何節點超出此臨界值，則即使達到「平衡臨界值」，也不會觸發平衡作業。

假設我們為這個計量保留平衡臨界值 3。 同時假設我們有活動臨界值 1536。 在第一個案例中，根據「平衡臨界值」，叢集是處於不平衡狀態，但沒有任何節點達到「活動臨界值」，因此不會發生任何事情。 在下半部的範例中，Node1 超出「活動臨界值」。 由於同時超出該計量的「平衡臨界值」和「活動臨界值」，因此會排定平衡作業。 讓我們看看下圖的範例： 

<center>
![活動臨界值範例][Image3]
</center>

如同平衡臨界值，活動臨界值會透過叢集定義根據每個度量進行定義︰

ClusterManifest.xml

``` xml
    <Section Name="MetricActivityThresholds">
      <Parameter Name="Memory" Value="1536"/>
    </Section>
```

獨立部署透過 ClusterConfig.json，Azure 託管叢集透過 Template.json：

```json
"fabricSettings": [
  {
    "name": "MetricActivityThresholds",
    "parameters": [
      {
          "name": "Memory",
          "value": "1536"
      }
    ]
  }
]
```

平衡臨界值和活動臨界值都與特定計量繫結 - 只有同時超出同一計量的「平衡臨界值」和「活動臨界值」時，才會觸發平衡作業。

> [!NOTE]
> 若未指定，計量的平衡臨界值就是 1，而活動臨界值是 0。 這表示叢集 Resource Manager 會嘗試使該計量對指定負載保持完美的平衡。 如果您是使用自訂計量，建議您明確地定義自己的計量平衡與活動臨界值。 
>

## <a name="balancing-services-together"></a>一起平衡服務
叢集是否不平衡牽涉到整個叢集的決策。 不過，我們修正此種情況的方法是將個別的服務複本和執行個體四處移動。 這很合理，對吧？ 如果記憶體堆疊在某一個節點上，可能是由多個複本或執行個體所造成。 若要修正不平衡的狀態，可能需要移動所有使用不平衡計量的具狀態複本或無狀態執行個體。

有時候本身並非不平衡的服務會被移動 (請記住稍早關於邏輯和全域加權的討論)。 為什麼當服務的計量平衡時服務會移動？ 看看以下範例：

- 假設有四個服務：Service1、Service2、Service3 及 Service4。 
- Service1 報告計量 Metric1 和 Metric2。 
- Service2 報告計量 Metric2 和 Metric3。 
- Service3 報告計量 Metric3 和 Metric4。
- Service4 報告計量 Metric99。 

您應該可以看出這個範例要表達什麼：有鏈結！ 我們並非實際上擁有 4 個獨立的服務，而是有 3 個相關的服務，以及一個獨立的服務。

<center>
![將服務一起平衡][Image4]
</center>

因為這個鏈結，所以計量 1-4 若發生不平衡，可能會導致屬於服務 1-3 的複本或執行個體四處移動。 我們也知道計量 1、2 或 3 若發生不平衡，並不會導致 Service4 中發生移動。 這麼做並沒有必要，因為將屬於 Service4 的複本或執行個體四處移動完全不會影響計量 1-3 的平衡。

叢集資源管理員會自動找出相關的服務。 新增、移除或變更服務的計量，可能會影響它們的關聯性。 例如，在兩次執行平衡作業之間，可能已經更新 Service2 來移除 Metric2。 這會中斷 Service1 和 Service2 之間的鏈結。 現在，您擁有的是三個相關服務群組，而非兩個︰

<center>
![將服務一起平衡][Image5]
</center>

## <a name="next-steps"></a>後續步驟
* 度量是 Service Fabric 叢集資源管理員管理叢集中的耗用量和容量的方式。 若要深入了解計量及其設定方式，請查看[這篇文章](service-fabric-cluster-resource-manager-metrics.md)
* 移動成本是向叢集資源管理員發出訊號，表示移動某些服務會比較貴的其中一種方式。 如需有關移動成本的詳細資訊，請參閱[這篇文章](service-fabric-cluster-resource-manager-movement-cost.md)
* 叢集資源管理員有數個為減緩叢集的流失而可以設定的節流。 這些節流通常不是必要的，但若有需要，您可以參閱 [這裡](service-fabric-cluster-resource-manager-advanced-throttling.md)

[Image1]:./media/service-fabric-cluster-resource-manager-balancing/cluster-resrouce-manager-balancing-thresholds.png
[Image2]:./media/service-fabric-cluster-resource-manager-balancing/cluster-resource-manager-balancing-threshold-triggered-results.png
[Image3]:./media/service-fabric-cluster-resource-manager-balancing/cluster-resource-manager-activity-thresholds.png
[Image4]:./media/service-fabric-cluster-resource-manager-balancing/cluster-resource-manager-balancing-services-together1.png
[Image5]:./media/service-fabric-cluster-resource-manager-balancing/cluster-resource-manager-balancing-services-together2.png
