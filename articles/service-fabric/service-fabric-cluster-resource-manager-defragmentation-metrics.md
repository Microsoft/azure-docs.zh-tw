---
title: Azure Service Fabric 中度量的重組 | Microsoft Docs
description: 使用重組或封裝作為 Service Fabric 中度量策略的概觀
services: service-fabric
documentationcenter: .net
author: masnider
manager: timlt
editor: ''
ms.assetid: e5ebfae5-c8f7-4d6c-9173-3e22a9730552
ms.service: Service-Fabric
ms.devlang: dotnet
ms.topic: conceptual
ms.tgt_pltfrm: NA
ms.workload: NA
ms.date: 08/18/2017
ms.author: masnider
ms.openlocfilehash: a35ae5933729615d634359e64e31d43536d81431
ms.sourcegitcommit: eb75f177fc59d90b1b667afcfe64ac51936e2638
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 05/16/2018
ms.locfileid: "34205071"
---
# <a name="defragmentation-of-metrics-and-load-in-service-fabric"></a>度量的重組和 Service Fabric 中的負載
Service Fabric 叢集資源管理員對於管理叢集中負載計量的預設策略是分散負載。 確保平均使用節點，以避免忙碌和閒置位置，導致爭用和浪費的資源。 就故障情況下幸存而言，分散工作負載是最安全的，因為這可確保不會因為故障而使指定的工作負載損失慘重。 

Service Fabric 叢集資源管理員支援管理負載的不同策略，也就是重組。 重組表示合併計量，而不是嘗試將計量的使用量分散到叢集中。 合併完全逆轉預設平衡策略 – 叢集資源管理員嘗試增加偏差，而不是將計量負載的平均標準偏差最小化。

## <a name="when-to-use-defragmentation"></a>使用重組的時機
分散叢集中的負載，會耗用每個節點上的一些資源。 某些工作負載會建立極龐大且耗用大部分或所有節點的服務。 在這些情況下，有可能在建立大型工作負載時，任何節點上都沒有可以執行的足夠空間。 大型工作負載在 Service Fabric 中並不是問題；在這些情況下，叢集資源管理員會決定需要重組叢集，以騰出空間給這個大型工作負載。 不過，同時此工作負載必須等待在叢集中排程。

如果要通過許多服務和狀態，則大型工作負載會經過很常的時間才放入叢集中。 如果叢集中的其他工作負載也很大，因而移動時間很久，可能就會發生這種情形。 Service Fabric 小組是模擬此案例來測量建立時間。 我們發現，只要叢集使用率超過 30% 到 50% 之間，建立大型服務所花費的時間就會更久。 為了解決這種情況，我們引進重組當作平衡策略。 我們發現，對於大型工作負載，特別是建立時間很重要的工作負載，重組確實有助於將新的工作負載放入叢集中排程。

您可以設定重組計量，讓叢集資源管理員主動嘗試將服務的負載壓縮至較少的節點。 這有助於確保幾乎永遠有空間容納更大型的服務，而不需要重新組織叢集。 不需要重新組織叢集可以讓建立大型工作負載更快速。

大部分的人不需要重組。 服務通常很小，因此在叢集中不難找到空間給它們。 重新組織可行時，同樣地可以快速執行，因為大部分服務很小，而且可以快速且平行地移動。 不過，如果您有大型的服務且需要儘速建立，則重組策略就適合您。 我們接下來將討論使用重組的權衡取捨。 

## <a name="defragmentation-tradeoffs"></a>重組權衡取捨
重組會放大故障的影響力，因為在故障節點上執行的服務更多。 重組也會增加成本，因為叢集中的資源必須保留，等候大型工作負載的建立。

下圖提供兩個叢集的視覺表示法，其中一個已重組，另一個未重組。 

<center>
![比較平衡和重組叢集][Image1]
</center>

在平衡情況下，請注意放置其中一個最大服務物件所需的移動次數。 在重組的叢集中，大型工作負載可以放置於四或五個節點，而不需要等待其他服務移動。

## <a name="defragmentation-pros-and-cons"></a>重組的優缺點
因此，有哪些其他概念性的代價？ 以下是要考慮事項的一覽表︰

| 重組優點 | 重組缺點 |
| --- | --- |
| 能夠更快速建立大型服務 |將負載集中到較少數的節點，提高爭用 |
| 在建立期間啟用較低的資料移動 |失敗會影響更多服務，並導致更多流失 |
| 能夠豐富描述需求和空間的回收 |較複雜的整體資源管理組態 |

您可以在相同叢集中混用重組計量和一般計量。 叢集資源管理員會嘗試儘可能合併重組計量，而分散其他計量。 混合重組和平衡策略的結果取決於許多因素，包括：
  - 平衡計量數目與重組計量數目
  - 是否有任何服務同時使用兩種類型的計量 
  - 計量權數
  - 目前的計量負載
  
需要實驗來判斷所需的確切組態。 我們建議先徹底測量您的工作負載，然後再於生產中啟用重組計量。 在相同服務中混合重組和平衡計量時，更是如此。 

## <a name="configuring-defragmentation-metrics"></a>設定重組度量
設定重組計量是叢集中的全域決策，而您可以選取個別的計量進行重組。 下列設定程式碼片段示範如何設定重組的計量。 在此情況下，"Metric1" 設定為重組計量，"Metric2" 則繼續進行一般平衡。 

ClusterManifest.xml：

```xml
<Section Name="DefragmentationMetrics">
    <Parameter Name="Metric1" Value="true" />
    <Parameter Name="Metric2" Value="false" />
</Section>
```

獨立部署透過 ClusterConfig.json，Azure 託管叢集透過 Template.json：

```json
"fabricSettings": [
  {
    "name": "DefragmentationMetrics",
    "parameters": [
      {
          "name": "Metric1",
          "value": "true"
      },
      {
          "name": "Metric2",
          "value": "false"
      }
    ]
  }
]
```


## <a name="next-steps"></a>後續步驟
- 叢集資源管理員有許多描述叢集的選項。 若要深入了解這些選項，請參閱關於[描述 Service Fabric 叢集](service-fabric-cluster-resource-manager-cluster-description.md)一文
- 度量是 Service Fabric 叢集資源管理員管理叢集中的耗用量和容量的方式。 若要深入了解計量及其設定方式，請查看[這篇文章](service-fabric-cluster-resource-manager-metrics.md)

[Image1]:./media/service-fabric-cluster-resource-manager-defragmentation-metrics/balancing-defrag-compared.png
