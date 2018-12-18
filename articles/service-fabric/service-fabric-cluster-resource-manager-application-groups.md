---
title: Service Fabric 叢集 Resource Manager - 應用程式群組 | Microsoft Docs
description: Service Fabric 叢集 Resource Manager 中應用程式群組功能的概觀
services: service-fabric
documentationcenter: .net
author: masnider
manager: timlt
editor: ''
ms.assetid: 4cae2370-77b3-49ce-bf40-030400c4260d
ms.service: Service-Fabric
ms.devlang: dotnet
ms.topic: conceptual
ms.tgt_pltfrm: NA
ms.workload: NA
ms.date: 08/18/2017
ms.author: masnider
ms.openlocfilehash: c8bab609212c837802be6f70e7fc74df6b5eaf2e
ms.sourcegitcommit: af9cb4c4d9aaa1fbe4901af4fc3e49ef2c4e8d5e
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/11/2018
ms.locfileid: "44346248"
---
# <a name="introduction-to-application-groups"></a>應用程式群組簡介
Service Fabric 的叢集資源管理員通常會將負載平均分配到整個叢集 (透過[計量](service-fabric-cluster-resource-manager-metrics.md)表示)，以管理叢集資源。 Service Fabric 透過[容量](service-fabric-cluster-resource-manager-cluster-description.md)管理叢集的節點容量及整個叢集的容量。 計量和容量很適用於許多工作負載，但過度使用不同 Service Fabric 應用程式執行個體的模式，有時會引起其他需求。 例如，您可能想要：

- 保留叢集節點上的一些容量，用於具名應用程式執行個體內的服務
- 限制具名應用程式執行個體內的服務執行的節點總數 (而不是將這些服務散佈到整個叢集)
- 決定具名應用程式執行個體本身的容量，以限制服務數量或執行個體內服務的資源總耗用量

為了符合這些需求，Service Fabric 叢集資源管理員可支援名為應用程式群組的功能。

## <a name="limiting-the-maximum-number-of-nodes"></a>限制節點數目上限
應用程式容量的最簡單使用案例，是在需要將應用程式執行個體限制為特定的節點數目上限時。 這會將該應用程式執行個體中的所有服務合併到固定數目的機器上。 在嘗試預測或限制該具名應用程式執行個體中的服務所使用的實體資源時，適合使用合併。 

下圖顯示已定義和未定義節點數目上限的應用程式執行個體：

<center>
![定義節點數目上限的應用程式執行個體][Image1]
</center>

在左邊的範例中，應用程式沒有定義的節點數目上限，且提供有三項服務。 叢集資源管理員已將所有複本分散到六個可用的節點，在叢集中達到最佳平衡狀態 (預設行為)。 在右邊的範例中，我們看到相同的應用程式限制為三個節點。

控制此行為的參數稱為 MaximumNodes。 您可以在應用程式建立期間設定這個參數，也可以針對執行中的應用程式執行個體更新這個參數。

Powershell

``` posh
New-ServiceFabricApplication -ApplicationName fabric:/AppName -ApplicationTypeName AppType1 -ApplicationTypeVersion 1.0.0.0 -MaximumNodes 3
Update-ServiceFabricApplication –ApplicationName fabric:/AppName –MaximumNodes 5
```

C#

``` csharp
ApplicationDescription ad = new ApplicationDescription();
ad.ApplicationName = new Uri("fabric:/AppName");
ad.ApplicationTypeName = "AppType1";
ad.ApplicationTypeVersion = "1.0.0.0";
ad.MaximumNodes = 3;
await fc.ApplicationManager.CreateApplicationAsync(ad);

ApplicationUpdateDescription adUpdate = new ApplicationUpdateDescription(new Uri("fabric:/AppName"));
adUpdate.MaximumNodes = 5;
await fc.ApplicationManager.UpdateApplicationAsync(adUpdate);

```

在節點集內，叢集資源管理員無法保證哪些服務物件會放置在一起，或會使用哪些節點。

## <a name="application-metrics-load-and-capacity"></a>應用程式度量、負載和容量
應用程式群組也可讓您定義與特定具名應用程式執行個體相關聯的計量，以及應用程式執行個體在這些計量中的容量。 應用程式計量可讓您追蹤、保留及限制該應用程式執行個體內各項服務的資源耗用量。

每一個應用程式計量可以設定兩個值︰

- **應用程式容量總計** – 這項設定代表應用程式在特定計量中的容量總計。 如果在這個應用程式執行個體內建立任何新的服務會導致總負載超過此值，則叢集資源管理員不允許建立這些服務。 例如，假設應用程式執行個體的容量為 10，而且已有五個負載。 不允許建立預設負載總計為 10 的服務。
- **節點容量上限** – 這項設定指定在單一節點上的應用程式負載總計上限。 如果負載超過這個容量，叢集資源管理員會將複本移動到其他節點，使負載降低。


PowerShell：

``` posh
New-ServiceFabricApplication -ApplicationName fabric:/AppName -ApplicationTypeName AppType1 -ApplicationTypeVersion 1.0.0.0 -Metrics @("MetricName:Metric1,MaximumNodeCapacity:100,MaximumApplicationCapacity:1000")
```

C#：

``` csharp
ApplicationDescription ad = new ApplicationDescription();
ad.ApplicationName = new Uri("fabric:/AppName");
ad.ApplicationTypeName = "AppType1";
ad.ApplicationTypeVersion = "1.0.0.0";

var appMetric = new ApplicationMetricDescription();
appMetric.Name = "Metric1";
appMetric.TotalApplicationCapacity = 1000;
appMetric.MaximumNodeCapacity = 100;
ad.Metrics.Add(appMetric);
await fc.ApplicationManager.CreateApplicationAsync(ad);
```

## <a name="reserving-capacity"></a>保留容量
應用程式群組的另一個常見用途是確保叢集內的資源保留給指定的應用程式執行個體。 建立應用程式執行個體時，一定會保留空間。

即使發生下列情況，也會立即為應用程式保留叢集中的空間：
- 應用程式執行個體已建立，但是其中尚無任何服務
- 每當變更應用程式執行個體中的服務數量時 
- 服務存在，但是不耗用資源 

保留應用程式執行個體的資源，需要指定兩個額外參數：*MinimumNodes* 和 *NodeReservationCapacity*

- **MinimumNodes** - 定義應執行應用程式執行個體的節點數目下限。  
- **NodeReservationCapacity** - 此設定是針對應用程式的計量。 該值是在該應用程式中的服務執行的任何節點上，為該應用程式保留的計量數量。

結合 **MinimumNodes** 和 **NodeReservationCapacity** 可保證叢集中的應用程式獲得最小的負載保留。 如果叢集中的剩餘容量小於所需的保留總量，則無法建立應用程式。 

讓我們看看容量保留的範例︰

<center>
![定義保留容量的應用程式執行個體][Image2]
</center>

在左邊的範例中，應用程式並沒有定義任何應用程式容量。 叢集資源管理員會根據一般規則來平衡一切事物。

在右邊範例中，假設以下列設定建立 Application1：

- MinimumNodes 設為二
- 應用程式計量定義下列值
  - NodeReservationCapacity 為 20

Powershell

 ``` posh
 New-ServiceFabricApplication -ApplicationName fabric:/AppName -ApplicationTypeName AppType1 -ApplicationTypeVersion 1.0.0.0 -MinimumNodes 2 -Metrics @("MetricName:Metric1,NodeReservationCapacity:20")
 ```

C#

 ``` csharp
ApplicationDescription ad = new ApplicationDescription();
ad.ApplicationName = new Uri("fabric:/AppName");
ad.ApplicationTypeName = "AppType1";
ad.ApplicationTypeVersion = "1.0.0.0";
ad.MinimumNodes = 2;

var appMetric = new ApplicationMetricDescription();
appMetric.Name = "Metric1";
appMetric.NodeReservationCapacity = 20;

ad.Metrics.Add(appMetric);

await fc.ApplicationManager.CreateApplicationAsync(ad);
```

Service Fabric 為 Application1 保留了兩個節點上的容量，而且即使 Application1 內的服務當時並未耗用容量，也不允許 Application2 的服務耗用該容量。 保留的應用程式容量會被視為已耗用，且是取自該節點上和叢集內的剩餘容量。  保留的容量會立即從剩餘的叢集容量中扣除，但只有在至少一個服務物件置於節點上的情況下，才會從特定節點的容量中扣除保留的耗用量。 這後來的保留做法能夠更有彈性和更充分利用資源，因為只有在需要時才於節點上保留資源。

## <a name="obtaining-the-application-load-information"></a>取得應用程式負載資訊
若應用程式的「應用程式容量」由一個或多個計量定義，您可以從服務複本報告中取得每一個應用程式的彙總負載之相關資訊。

PowerShell：

``` posh
Get-ServiceFabricApplicationLoadInformation –ApplicationName fabric:/MyApplication1
```

C#

``` csharp
var v = await fc.QueryManager.GetApplicationLoadInformationAsync("fabric:/MyApplication1");
var metrics = v.ApplicationLoadMetricInformation;
foreach (ApplicationLoadMetricInformation metric in metrics)
{
    Console.WriteLine(metric.ApplicationCapacity);  //total capacity for this metric in this application instance
    Console.WriteLine(metric.ReservationCapacity);  //reserved capacity for this metric in this application instance
    Console.WriteLine(metric.ApplicationLoad);  //current load for this metric in this application instance
}
```

ApplicationLoad 查詢會傳回應用程式指定之「應用程式容量」的基本資訊。 這項資訊包含「節點下限」和「節點上限」資訊，以及應用程式目前佔用的數目。 也包含每個應用程式負載計量的相關資訊，包括︰

* 度量名稱：度量的名稱。
* 保留容量：針對此應用程式保留在叢集中的叢集容量。
* 應用程式負載︰此應用程式的子複本的總負載。
* 應用程式容量︰許可的應用程式負載值上限。

## <a name="removing-application-capacity"></a>移除應用程式容量
一旦針對應用程式設定應用程式容量參數，它們可以使用更新的應用程式 API 或 PowerShell Cmdlet 來移除。 例如︰

``` posh
Update-ServiceFabricApplication –Name fabric:/MyApplication1 –RemoveApplicationCapacity

```

此命令會從應用程式執行個體中移除所有「應用程式容量管理」參數， 包括 MinimumNodes、MaximumNodes 和應用程式的度量 (如果有)。 此命令會立即生效。 此命令完成後，叢集資源管理員會使用預設行為來管理應用程式。 您可以透過 `Update-ServiceFabricApplication`/`System.Fabric.FabricClient.ApplicationManagementClient.UpdateApplicationAsync()` 再次指定「應用程式容量」參數。

### <a name="restrictions-on-application-capacity"></a>應用程式容量的限制
應用程式容量參數有數個限制必須遵守。 如果發生驗證錯誤，則不會進行任何變更。

- 所有整數參數必須為非負數。
- MinimumNodes 不能大於 MaximumNodes。
- 如果已定義負載度量的容量，則它們必須遵守下列規則︰
  - 節點保留容量不能大於節點容量上限。 例如，您不能在節點上將 “CPU” 計量的容量限制為兩個單位，卻嘗試在每個節點上保留三個單位。
  - 如果已指定 MaximumNodes，則 MaximumNodes 和節點容量上限的乘積不能大於應用程式容量總計。 例如，假設負載計量 “CPU” 的「節點容量上限」設定為八。 另外也假設您將「節點上限」設定為 10。 在此情況下，此負載計量的「應用程式容量總計」必須大於 80。

在應用程式建立及更新期間，都會強制執行限制。

## <a name="how-not-to-use-application-capacity"></a>如何不使用應用程式容量
- 請勿嘗試使用「應用程式群組」功能將應用程式限制到「特定」的節點子集。 換句話說，您可以指定最多在五個節點上執行應用程式，但無法指定叢集中的哪五個特定節點。 您可以使用服務的放置條件約束，將應用程式限制到特定節點。
- 請勿使用「應用程式容量」來確保相同應用程式的兩個服務放在相同的節點上。 請改用[親和性](service-fabric-cluster-resource-manager-advanced-placement-rules-affinity.md)或[放置條件約束](service-fabric-cluster-resource-manager-cluster-description.md#node-properties-and-placement-constraints)。

## <a name="next-steps"></a>後續步驟
- 如需服務設定的詳細資訊，請[深入了解設定服務](service-fabric-cluster-resource-manager-configure-services.md)
- 若要了解叢集資源管理員如何管理並平衡叢集中的負載，請查看關於 [平衡負載](service-fabric-cluster-resource-manager-balancing.md)
- 從頭開始，並 [取得 Service Fabric 叢集 Resource Manager 的簡介](service-fabric-cluster-resource-manager-introduction.md)
- 如需度量通常如何運作的詳細資訊，請繼續閱讀 [Service Fabric 負載度量](service-fabric-cluster-resource-manager-metrics.md)
- 叢集資源管理員有許多描述叢集的選項。 若要深入了解這些選項，請參閱關於[描述 Service Fabric 叢集](service-fabric-cluster-resource-manager-cluster-description.md)一文

[Image1]:./media/service-fabric-cluster-resource-manager-application-groups/application-groups-max-nodes.png
[Image2]:./media/service-fabric-cluster-resource-manager-application-groups/application-groups-reserved-capacity.png
