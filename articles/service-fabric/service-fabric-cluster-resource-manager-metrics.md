---
title: 使用計量來管理 Azure Service Fabric 應用程式負載 | Microsoft Docs
description: 了解如何在 Service Fabric 中設定及使用計量，以管理服務資源耗用量。
services: service-fabric
documentationcenter: .net
author: masnider
manager: timlt
editor: ''
ms.assetid: 0d622ea6-a7c7-4bef-886b-06e6b85a97fb
ms.service: Service-Fabric
ms.devlang: dotnet
ms.topic: conceptual
ms.tgt_pltfrm: NA
ms.workload: NA
ms.date: 08/18/2017
ms.author: masnider
ms.openlocfilehash: 7a7d3ad59d743287e5fe13c52c6c6a1a115d53f3
ms.sourcegitcommit: ebd06cee3e78674ba9e6764ddc889fc5948060c4
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/07/2018
ms.locfileid: "44053307"
---
# <a name="managing-resource-consumption-and-load-in-service-fabric-with-metrics"></a>在 Service Fabric 中使用度量管理資源耗用量和負載
*計量*是您的服務所關切的資源，且是由叢集中的節點提供。 計量就是任何您想要管理，以便改善或監視服務效能的項目。 例如，您可能會監看記憶體耗用量以得知您的服務是否為多載。 另一個用法是，了解服務是否能夠移至記憶體限制較小的其他位置，以取得更佳的效能。

像記憶體、磁碟 及 CPU 使用率等項目都是計量的範例。 這些計量是實體計量，也就是對應至節點上需要管理之實體資源的資源。 計量也可以是 (且通常是) 邏輯計量。 邏輯計量的範例是 “MyWorkQueueDepth”、"MessagesToProcess" 或 "TotalRecords" 之類的項目。 邏輯計量是由應用程式定義，並且間接對應到某些實體資源耗用量。 邏輯計量是常見的，因為很難以每個服務為基礎測量及報告實體資源耗用量。 測量及報告您自己的實體計量有點複雜，這也是為什麼 Service Fabric 會提供一些預設計量。

## <a name="default-metrics"></a>預設度量
假設您想要開始撰寫和部署您的服務。 此時，您不知道它會取用哪些實體或邏輯資源。 沒關係！ 未指定其他計量時，Service Fabric 叢集資源管理員會使用某些預設計量。 如下：

  - PrimaryCount - 節點上主要複本的計數 
  - ReplicaCount - 節點上具狀態複本的總數
  - Count - 節點上所有服務物件 (無狀態和具狀態) 的計數

| 計量 | 無狀態執行個體負載 | 具狀態次要負載 | 具狀態主要負載 | Weight |
| --- | --- | --- | --- | --- |
| PrimaryCount |0 |0 |1 |高 |
| ReplicaCount |0 |1 |1 |中 |
| Count |1 |1 |1 |低 |


針對基本工作負載，預設計量會提供叢集中工作的適當分配。 在下列範例中，讓我們看看建立兩個服務以及依賴預設計量以進行平衡時會發生什麼情況。 第一個服務是帶有三個資料分割且目標複本集大小為三的具狀態服務。 第二個服務是帶有一個資料分割且執行個體計數為三的無狀態服務。

以下是您將看到的情況：

<center>
![使用預設計量的叢集配置][Image1]
</center>

注意事項：
  - 具狀態服務的主要複本會分散到數個節點
  - 相同資料分割的複本是在不同節點上
  - 主要複本與次要複本的總數分佈在叢集中
  - 服務物件的總數平均配置於每個節點

很好！

預設計量是個很好的起點。 不過，預設計量只能做到這個程度。 例如︰您選擇的資料分割配置導致所有資料分割完全平均使用的可能性為何？ 所指定服務的負載在一段時間內保持不變或只是現在跨多個資料分割相同的機率為何？

您可以僅使用預設計量來執行。 不過，這樣通常表示叢集使用率較低，比您想要的更不平均。 這是因為預設計量不是調適性的，而是假設所有項目都相等。 例如，忙碌的主要複本和一個不忙碌的其他複本都會對 PrimaryCount 計量貢獻 "1"。 在最糟的情況下，使用預設計量也可能導致節點被過度排定，而造成效能問題。 如果您對充分利用叢集並避免效能問題感興趣，您必須使用自訂計量和動態負載報告。

## <a name="custom-metrics"></a>自訂度量
當您建立服務時，可以為每個具名服務執行個體設定計量。

任何計量都有一些描述它的屬性：名稱、權數及預設負載。

* 計量名稱：計量的名稱。 從「資源管理員」的觀點來看，計量名稱是叢集內計量的唯一識別碼。
* Weight︰計量權數定義了就此服務而言，此計量相對於其他計量的重要程度。
* 預設負載：預設負載會依據服務是無狀態或具狀態服務來以不同方式表示。
  * 就無狀態服務而言，每個計量都有名為 DefaultLoad 的單一屬性
  * 針對具狀態服務，您需定義：
    * PrimaryDefaultLoad：當此計量作為「主要」複本時，此服務將取用的預設量
    * SecondaryDefaultLoad：當此計量作為「次要」複本時，此服務將取用的預設量

> [!NOTE]
> 如果您定義自訂計量並_同時_也想要使用預設計量，您必須_明確地_加回預設計量並且定義它們的權數和值。 這是因為您必須定義預設計量與自訂計量之間的關聯性。 例如，也許您會在意 ConnectionCount 或 WorkQueueDepth 更甚於主要分配。 根據預設，PrimaryCount 計量的權數是「高」，所以當您新增其他計量時想要將它降低為「中」，以確保它們優先。
>

### <a name="defining-metrics-for-your-service---an-example"></a>為您的服務定義計量 - 範例
假設您想要下列設定：

  - 您的服務報告名為 "ConnectionCount” 的計量
  - 您也想要使用預設計量 
  - 您已經完成一些測量，而知道該服務的主要複本通常佔用 20 單位的 "ConnectionCount"。
  - 而次要複本則佔用 5 單位的 "ConnectionCount"
  - 您知道就管理這項特定服務的效能而言，"ConnectionCount" 是最重要的計量
  - 但您仍然希望主要複本達到平衡。 無論如何，平衡主要複本通常是個不錯的主意。 這有助於防止遺失某些節點或容錯網域而影響它隨附的大多數主要複本。 
  - 否則，預設計量是好的

以下是您以該計量組態建立服務時將撰寫的程式碼：

程式碼：

```csharp
StatefulServiceDescription serviceDescription = new StatefulServiceDescription();
StatefulServiceLoadMetricDescription connectionMetric = new StatefulServiceLoadMetricDescription();
connectionMetric.Name = "ConnectionCount";
connectionMetric.PrimaryDefaultLoad = 20;
connectionMetric.SecondaryDefaultLoad = 5;
connectionMetric.Weight = ServiceLoadMetricWeight.High;

StatefulServiceLoadMetricDescription primaryCountMetric = new StatefulServiceLoadMetricDescription();
primaryCountMetric.Name = "PrimaryCount";
primaryCountMetric.PrimaryDefaultLoad = 1;
primaryCountMetric.SecondaryDefaultLoad = 0;
primaryCountMetric.Weight = ServiceLoadMetricWeight.Medium;

StatefulServiceLoadMetricDescription replicaCountMetric = new StatefulServiceLoadMetricDescription();
replicaCountMetric.Name = "ReplicaCount";
replicaCountMetric.PrimaryDefaultLoad = 1;
replicaCountMetric.SecondaryDefaultLoad = 1;
replicaCountMetric.Weight = ServiceLoadMetricWeight.Low;

StatefulServiceLoadMetricDescription totalCountMetric = new StatefulServiceLoadMetricDescription();
totalCountMetric.Name = "Count";
totalCountMetric.PrimaryDefaultLoad = 1;
totalCountMetric.SecondaryDefaultLoad = 1;
totalCountMetric.Weight = ServiceLoadMetricWeight.Low;

serviceDescription.Metrics.Add(connectionMetric);
serviceDescription.Metrics.Add(primaryCountMetric);
serviceDescription.Metrics.Add(replicaCountMetric);
serviceDescription.Metrics.Add(totalCountMetric);

await fabricClient.ServiceManager.CreateServiceAsync(serviceDescription);
```

PowerShell：

```posh
New-ServiceFabricService -ApplicationName $applicationName -ServiceName $serviceName -ServiceTypeName $serviceTypeName –Stateful -MinReplicaSetSize 3 -TargetReplicaSetSize 3 -PartitionSchemeSingleton –Metric @("ConnectionCount,High,20,5”,"PrimaryCount,Medium,1,0”,"ReplicaCount,Low,1,1”,"Count,Low,1,1”)
```

> [!NOTE]
> 上述範例與本文件的其餘部分描述以每個具名服務為基礎來管理計量。 也可以在服務_類型_層級定義您的服務計量。 這會透過在您的服務資訊清單中指定它們來完成。 不建議定義類型層級度量有幾個原因。 第一個原因是，計量名稱經常是環境特定的。 除非有確實的就地合約，否則您無法確定一個環境中的計量 "Cores" 不是其他環境中的 "MiliCores" 或 "CoReS"。 如果您的計量在資訊清單中定義，您必須為每個環境建立資訊清單。 這通常會導致只有些許差異的不同資訊清單擴散，進而造成管理困難。  
>
> 計量負載通常會為每個具名服務執行個體指派。 例如，假設您為 CustomerA 建立一個服務執行個體，該客戶計劃只會輕度使用。 同時假設您為 CustomerB 建立另一個執行個體，該客戶有更大的工作負載。 在此情況下，您可能會想要調整這些服務的預設負載。 如果您有透過資訊清單定義的計量和負載，並且想要支援此案例，則對於每個客戶需要不同的應用程式和服務類型。 在服務建立時定義的值會覆寫資訊清單中定義的值，因此您可以用來設定特定預設值。 不過，這麼做會使資訊清單中宣告的值不符合服務實際執行的值。 這會導致混淆。 
>

提醒您，如果您只想要使用預設計量，您就完全不必碰觸計量集合，或在建立服務時執行任何特殊動作。 未定義其他計量時，會自動使用預設計量。 

現在，我們再更詳細地瀏覽這些設定，並且討論其影響的行為。

## <a name="load"></a>載入
定義計量的整個重點在於代表某個負載。 「負載」係指特定節點上的某個服務執行個體或複本取用的特定計量數量。 隨時可以設定負載。 例如︰

  - 建立服務時，可以定義負載。 這稱為_預設負載_。
  - 計量資訊，包括建立服務之後服務可以更新的預設負載。 這稱為_更新服務_。 
  - 指定資料分割的負載可以重設為該服務的預設值。 這稱為_重設資料分割負載_。
  - 負載可以每個服務物件為基礎，在執行階段以動態方式報告。 這稱為_報告負載_。 
  
這些策略全部可以在相同服務的存留期內使用。 

## <a name="default-load"></a>預設負載
預設負載係指此服務的每個服務物件 (無狀態執行個體或具狀態複本) 取用的計量數量。 叢集資源管理員會將這個數字用於服務物件的負載，直到它接收其他資訊，例如動態負載報告。 針對簡單服務，預設負載是靜態定義。 預設負載永遠不會更新，並且在服務的存留期使用。 預設負載非常適用於簡單的容量規劃案例，其中特定數量的資源是供不同的工作負載專用，不會變更。

> [!NOTE]
> 如需有關容量管理和定義叢集中節點容量的詳細資訊，請參閱[這篇文章](service-fabric-cluster-resource-manager-cluster-description.md#capacity)。
> 

「叢集資源管理員」允許具狀態服務為其「主要」複本和「次要」複本指定不同的預設負載。 而無狀態服務則只能指定一個值，套用至所有執行個體。 就具狀態服務而言，主要和次要複本的預設負載通常不同，因為複本在每個角色中是執行不同類型的工作。 例如，主要複本通常會同時為讀取和寫入 (以及大多數運算負擔) 提供服務，而次要複本則不會。 主要複本的預設負載通常高於次要複本的預設負載。 實際數字應該取決於您自己的測量。

## <a name="dynamic-load"></a>動態負載
假設您已經執行服務達一段時間。 藉由一些監視，您已注意到：

1. 所指定服務的某些資料分割或執行個體比其他資料分割或執行個體耗用更多資源
2. 某些服務的負載會隨著時間改變。

有很多原因可能造成這些類型的負載變動。 例如，不同的服務或資料分割會與具有不同需求的不同客戶相關聯。 由於服務的工作量會隨著一天的行程而異，所以也可以變更負載。 不論是哪種原因，通常沒有任何單一數字可供您用來作為預設值。 如果您想要最大限度的利用叢集，更是如此。 您為預設負載挑選的任何值有時都會出錯。 不正確的預設負載會導致「叢集資源管理員」配置的資源不是太多就是不足。 結果就是即使「叢集資源管理員」認為叢集已達平衡狀態，但您的節點卻有使用率太高或太低的情況。 由於預設負載提供初始放置的某些資訊，因此它們仍有其實用性，但是它們並無法反映實際工作負載的完整面貌。 為了準確地擷取變動的資源需求，「叢集資源管理員」允許每個服務物件在執行階段期間更新自己的負載。 這稱為動態負載報告功能。

動態負載報告可讓複本或執行個體在其存留期中調整其配置/回報的計量負載。 閒置且未執行任何工作的服務複本或執行個體通常會回報使用少量的指定計量。 忙碌的複本或執行個體則會回報使用較多的資源。

依據每一複本或執行個體報告負載可讓「叢集資源管理員」重新組織叢集內的個別服務物件。 重新組織服務可協助確保它們取得所需的資源。 忙碌的服務可有效地從其他閒置或執行較少工作的複本或執行個體「回收」資源。

在「可靠服務」中，動態報告負載的程式碼看起來會像這樣：

程式碼：

```csharp
this.Partition.ReportLoad(new List<LoadMetric> { new LoadMetric("CurrentConnectionCount", 1234), new LoadMetric("metric1", 42) });
```

服務可以報告在建立時為它定義的任何計量。 如果服務報告的計量負載並非設定為使用，Service Fabric 會忽略該報告。 如果相同時間有其他報告的有效計量，系統就會接受這些報告。 服務程式碼可以測量並報告它知道的所有計量，而操作員則可以指定要使用的的計量設定，而不需變更服務程式碼。 

### <a name="updating-a-services-metric-configuration"></a>更新服務的計量設定
與服務相關聯的計量清單和這些計量的屬性，可以在服務存留時動態地更新。 這樣可以進行實驗並具有彈性。 非常有用的某些範例為：

  - 針對特定服務停用具有錯誤報告的計量
  - 根據想要的行為重新設定計量的權數
  - 只有在程式碼已經部署並透過其他機制進行驗證之後，才啟用新的計量
  - 根據觀察到的行為和耗用量變更服務的預設負載

變更計量設定的主要 API 在 C# 中是 `FabricClient.ServiceManagementClient.UpdateServiceAsync`，在 PowerShell 中是 `Update-ServiceFabricService`。 您使用這些 API 指定的任何資訊都會立即取代服務的現有計量資訊。 

## <a name="mixing-default-load-values-and-dynamic-load-reports"></a>混用預設負載值和動態負載報告
預設負載和動態負載可以用於相同的服務。 當服務利用預設負載和動態負載報告時，預設負載會作為預估值，直到動態報告出現為止。 預設負載相當好，因為它提供一些可供「叢集資源管理員」使用的資訊。 預設負載可讓「叢集資源管理員」在建立服務物件時，就將它們放在適當的位置。 如果未提供任何預設負載資訊，就會以隨機方式有效率地放置服務。 當負載報告較晚到達時，初始隨機放置經常錯誤，叢集資源管理員必須移動服務。

讓我們採用先前的範例，看看當我們加入一些自訂負載和動態負載報告功能時會發生什麼狀況。 在此範例中，我們使用 “MemoryInMb” 作為範例計量。

> [!NOTE]
> 記憶體是 Service Fabric 可以進行[資源管理](service-fabric-resource-governance.md)的其中一個系統計量，您自行報告通常有難度。 實際上我們不期待您報告記憶體耗用量。在這裡使用記憶體是協助了解叢集資源管理員的功能。
>

讓我們假設一開始使用下列命令建立了具狀態服務︰

PowerShell：

```posh
New-ServiceFabricService -ApplicationName $applicationName -ServiceName $serviceName -ServiceTypeName $serviceTypeName –Stateful -MinReplicaSetSize 3 -TargetReplicaSetSize 3 -PartitionSchemeSingleton –Metric @("MemoryInMb,High,21,11”,"PrimaryCount,Medium,1,0”,"ReplicaCount,Low,1,1”,"Count,Low,1,1”)
```

提醒您，此語法是 ("MetricName, MetricWeight, PrimaryDefaultLoad, SecondaryDefaultLoad")。

讓我們看看可能的叢集配置看起來如何︰

<center>
![使用預設和自訂計量平衡的叢集][Image2]
</center>

有幾件事值得一提：

* 資料分割內的次要複本可以有自己的負載
* 整體計量看起來已達平衡。 就記憶體而言，最大和最小負載之間的比例為 1.75 (負載最高的節點是 N3，最低的是 N2，而 28/16 = 1.75)。

還有一些我們仍需說明的事項：

* 何者決定 1.75 的比率是否合理？ 「叢集資源管理員」如何知道已經做得夠好還是有待加強？
* 何時發生平衡？
* 記憶體的權數「很高」是什麼意思？

## <a name="metric-weights"></a>度量權數
追蹤各個不同服務的相同計量相當重要。 該全域檢視可讓「叢集資源管理員」追蹤叢集中的耗用量、平衡各個節點之間的耗用量，以及確保節點不會超出容量。 不過，服務在相同計量的重要性方面可能有不同的檢視。 此外，在具有許多計量和許多服務的叢集中，可能並非所有計量都有完美平衡的解決方案。 「叢集資源管理員」應該如何處理這些情況？

計量權數可讓「叢集資源管理員」在沒有完美解答時，決定如何平衡叢集。 計量權數也可讓「叢集資源管理員」以不同的方式平衡特定服務。 度量可以有四個不同的權數層級︰零、低、中和高。 在考量項目是否平衡時，權數為「零」的計量並沒有任何貢獻。 不過，其負載對容量管理仍有所貢獻。 具有「零」權數的計量仍然相當有用，並且經常作為服務行為和效能監視的一部分。 [這篇文章](service-fabric-diagnostics-event-generation-infra.md)提供有關使用計量進行監視，以及診斷服務的詳細資訊。 

叢集中不同計量權數的實際影響在於「叢集資源管理員」會產生不同的解決方案。 計量權數會告訴「叢集資源管理員」某些計量比其他計量重要。 當沒有完美的解決方案時，「叢集資源管理員」可以偏好選擇能更有效平衡較高權數計量的解決方案。 如果服務認為特定計量不重要，它可能會發現對該計量的使用不平衡。 這可讓另一項服務取得對其重要之某些計量的平均分配。

讓我們看看一些負載報告的範例，以及不同的計量權數如何在叢集中造成不同的配置。 在此範例中，我們看到切換計量的相對權數會導致「叢集資源管理員」建立不同的服務佈局。

<center>
![計量權數範例及其對平衡解決方案的影響][Image3]
</center>

在此範例中，有四個不同的服務，它們都針對兩個不同計量 (MetricA 和 MetricB) 報告不同值。 在其中一個案例中，所有定義 MetricA 的服務是最重要的 (權數 = 高)，MetricB 則是較不重要的 (權數 = 低)。 因此，我們看到「叢集資源管理員」是以讓 MetricA 比 MetricB 更加平衡的方式來設置服務。 「更加平衡」表示 MetricA 具有比 MetricB 較低的標準差。 在第二個案例中，我們將計量權數反轉。 結果就是「叢集資源管理員」會交換服務 A 與 B，以便產生 MetricB 比 MetricA 更加平衡的配置。

> [!NOTE]
> 計量權數會決定叢集資源管理員應該如何平衡，但是不會決定平衡應該何時發生。 如需有關平衡的詳細資訊，請參閱[這篇文章](service-fabric-cluster-resource-manager-balancing.md)
>

### <a name="global-metric-weights"></a>全域度量權數
假設 Services 將 MetricA 定義權數「高」，且 ServiceB 將 MetricA 的權數設定為「低」或「零」。 最後使用的實際權數是什麼？

針對每個計量都會追蹤多個權數。 第一個權數是在建立服務時針對計量定義的權數。 其他權數是全域權數，它會自動計算。 「叢集資源管理員」在計算解決方案的分數時會同時使用這兩種權數。 將這兩個權數列入考量很重要。 這可讓叢集資源管理員根據自己的優先順序平衡每個服務，也可確保會正確地配置整個叢集。

如果「叢集資源管理員」既不在意全域平衡也不在意區域平衡，會發生什麼狀況？ 建構全域平衡的解決方案雖然簡單，但是會導致個別服務的資源平衡不佳。 在以下範例中，讓我們看看僅使用預設計量設定的服務，並了解如果只考量全域平衡，會發生什麼狀況：

<center>
![全域唯一解決方案的影響][Image4]
</center>

在上半部的範例中，只考量了全域平衡，叢集整體上的確達到平衡。 所有節點的主要複本計數和複本總數都相同。 不過，如果您查看此配置的實際影響，就不是那麼理想︰遺失任何節點都會對特定工作負載帶來不成比例的影響，因為這會取出其所有主要複本。 例如，如果第一個節點發生失敗，圓形服務之三個不同資料分割的三個主要複本將會全部遺失。 相反地，「三角形」和「六邊形」服務的資料分割遺失複本。 這樣不會造成中斷，只是必須復原關閉的複本。

在下半部的範例中，「叢集資源管理員」已同時根據全域和個別服務平衡來分配複本。 在計算解決方案的分數時，它會將大部分權數給予全域解決方案，而將一部分 (可設定) 給予個別的服務。 計算計量的全域平衡時，是根據每項服務之計量權數的平均值來計算。 平衡每項服務時，是根據其自己已定義的計量權數來平衡。 這可確保根據服務本身的要求，在服務彼此之間達到平衡。 因此，如果相同的第一個節點發生失敗，失敗會分散在所有服務的所有資料分割。 對每個資料分割的影響都一樣。

## <a name="next-steps"></a>後續步驟
- 如需有關設定服務的詳細資訊，請參閱[深入了解設定服務](service-fabric-cluster-resource-manager-configure-services.md)(service-fabric-cluster-resource-manager-configure-services.md)
- 定義重組度量是合併 (而不是擴增) 節點上負載的一種方式。若要了解如何設定重組，請參閱 [這篇文章](service-fabric-cluster-resource-manager-defragmentation-metrics.md)
- 若要了解叢集資源管理員如何管理並平衡叢集中的負載，請查看關於 [平衡負載](service-fabric-cluster-resource-manager-balancing.md)
- 從頭開始，並 [取得 Service Fabric 叢集資源管理員的簡介](service-fabric-cluster-resource-manager-introduction.md)
- 移動成本是向叢集資源管理員發出訊號，表示移動某些服務會比較貴的其中一種方式。 若要深入了解移動成本，請參閱 [這篇文章](service-fabric-cluster-resource-manager-movement-cost.md)

[Image1]:./media/service-fabric-cluster-resource-manager-metrics/cluster-resource-manager-cluster-layout-with-default-metrics.png
[Image2]:./media/service-fabric-cluster-resource-manager-metrics/Service-Fabric-Resource-Manager-Dynamic-Load-Reports.png
[Image3]:./media/service-fabric-cluster-resource-manager-metrics/cluster-resource-manager-metric-weights-impact.png
[Image4]:./media/service-fabric-cluster-resource-manager-metrics/cluster-resource-manager-global-vs-local-balancing.png
