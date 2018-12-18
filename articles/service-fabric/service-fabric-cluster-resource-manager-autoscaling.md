---
title: Azure Service Fabric 自動調整服務和容器 | Microsoft Docs
description: Azure Service Fabric 可讓您為服務和容器設定自動調整原則。
services: service-fabric
documentationcenter: .net
author: radicmilos
manager: ''
editor: nipuzovi
ms.assetid: ab49c4b9-74a8-4907-b75b-8d2ee84c6d90
ms.service: service-fabric
ms.devlang: dotNet
ms.topic: conceptual
ms.tgt_pltfrm: NA
ms.workload: NA
ms.date: 04/17/2018
ms.author: miradic
ms.openlocfilehash: db4f83d0d407ad3d9e895759ea2a687662f5620a
ms.sourcegitcommit: ebd06cee3e78674ba9e6764ddc889fc5948060c4
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/07/2018
ms.locfileid: "44053290"
---
# <a name="introduction-to-auto-scaling"></a>自動調整簡介
自動調整是 Service Fabric 的一項額外功能，可根據服務所回報的負載，或根據其資源使用情況，來動態調整服務。 自動調整提供絕佳的彈性，並可讓您依需求佈建額外的服務執行個體或分割區。 整個自動調整程序是自動化並在背景執行的，在您於服務上設定原則之後，即無須在服務層級進行手動調整作業。 您可以在建立服務時，或隨時藉由更新服務，來開啟自動調整功能。

其中一個適合使用自動調整的常見案例，就是當特定服務上的負載隨著時間而有所不同的時候。 例如，服務 (例如閘道) 可以根據處理連入要求所需的資源數量進行調整。 讓我們來看看這些調整規則可能看起來像什麼樣子的範例：
* 如果所有閘道執行個體平均使用的核心數都超過 2 個，則再多新增一個執行個體來相應放大閘道服務規模。 每小時都執行一次此操作，但執行個體總數一律不要超過 7 個。
* 如果所有閘道執行個體平均使用的核心數都少於 0.5 個，則移除一個執行個體來相應縮小閘道服務規模。 每小時都執行一次此操作，但執行個體總數一律不要少於 3 個。

容器和一般 Service Fabric 服務都支援自動調整功能。 本文的其餘部分說明調整原則、啟用或停用自動調整的方法，並提供有關如何使用此功能的範例。

## <a name="describing-auto-scaling"></a>描述自動調整
您可以為 Service Fabric 叢集內的每個服務定義自動調整原則。 每個調整原則都由兩個部分組成：
* **調整觸發程序**描述何時將執行服務調整。 系統會定期檢查觸發程序中所定義的條件，以判斷是否應該相應放大服務規模。

* **調整機制**描述觸發調整時，將如何執行調整。 只有在符合觸發程序中的條件時，才會套用機制。

目前支援的所有觸發程序都可以與[邏輯負載計量](service-fabric-cluster-resource-manager-metrics.md)或實體計量 (例如 CPU 或記憶體使用量) 搭配運作。 兩種方式不論是哪一種，Service Fabric 都會監視針對計量回報的負載，並且會定期評估觸發程序以判斷是否需要進行調整。

目前針對自動調整功能支援兩種機制。 第一種適用於無狀態服務，或適用於藉由新增或移除[執行個體](service-fabric-concepts-replica-lifecycle.md)來進行自動調整的容器。 不論是具狀態服務還是無狀態服務，都可以藉由新增或移除具名的服務[分割區](service-fabric-concepts-partitioning.md)來執行自動調整。

> [!NOTE]
> 目前僅支援每一服務一個調整原則。

## <a name="average-partition-load-trigger-with-instance-based-scaling"></a>搭配執行個體型調整的平均分割區負載觸發程序
第一種觸發程序所根據的是無狀態服務分割區中的執行個體負載。 計量負載會先經過平滑處理以取得分割區每個執行個體的負載，然後從分割區的所有執行個體中算出這些值的平均值。 決定服務調整時機的因素有三個：

* 「負載閾值下限」是一個決定何時應該**相應縮小**服務規模的值。 如果分割區的所有執行個體平均負載低於此值，就會相應縮小服務規模。
* 「負載閾值上限」是一個決定何時應該**相應放大**服務規模的值。如果分割區的所有執行個體平均負載高於此值，就會相應放大服務規模。
* 「調整間隔」會決定檢查觸發程序的頻率。 檢查觸發程序之後，如果需要調整，就會套用此機制。 如果不需要調整，則不會執行任何動作。 在這兩種情況下，在調整間隔再次到期之前，都不會再次檢查觸發程序。

此觸發程序只能與無狀態服務 (無狀態容器或 Service Fabric 服務) 搭配使用。 如果服務有多個分割區，將會針對每個分割區個別評估觸發程序，而且每個分割區會獨立套用指定的機制。 因此，在此情況下，根據分割區的負載，服務可能會同時有一些分割區將相應放大規模，有一些將相應縮小規模，以及一些將完全不調整。

唯一能夠與此觸發程序搭配使用的機制是 PartitionInstanceCountScaleMechanism。 決定此機制套用方式的因素有三個：
* 「調整增量」會決定當觸發機制時，要新增或移除多少個執行個體。
* 「最大執行個體計數」會定義調整的上限。 如果分割區的執行個體數目達到此限制，則不論負載情況為何，服務都不會相應放大規模。 您可以指定 -1 的值來省略此限制，在此情況下，服務將儘可能相應放大規模 (以叢集內可用的節點數目為限)。
* 「最小執行個體計數」會定義調整的下限。 如果分割區的執行個體數目達到此限制，則不論負載情況為何，服務都不會相應縮小規模。

## <a name="setting-auto-scaling-policy"></a>設定自動調整原則

### <a name="using-application-manifest"></a>使用應用程式資訊清單
``` xml
<LoadMetrics>
<LoadMetric Name="MetricB" Weight="High"/>
</LoadMetrics>
<ServiceScalingPolicies>
<ScalingPolicy>
    <AveragePartitionLoadScalingTrigger MetricName="MetricB" LowerLoadThreshold="1" UpperLoadThreshold="2" ScaleIntervalInSeconds="100"/>
    <InstanceCountScalingMechanism MinInstanceCount="3" MaxInstanceCount="4" ScaleIncrement="1"/>
</ScalingPolicy>
</ServiceScalingPolicies>
```
### <a name="using-c-apis"></a>使用 C# API
```csharp
FabricClient fabricClient = new FabricClient();
StatelessServiceDescription serviceDescription = new StatelessServiceDescription();
//set up the rest of the ServiceDescription
AveragePartitionLoadScalingTrigger trigger = new AveragePartitionLoadScalingTrigger();
PartitionInstanceCountScaleMechanism mechanism = new PartitionInstanceCountScaleMechanism();
mechanism.MaxInstanceCount = 3;
mechanism.MinInstanceCount = 1;
mechanism.ScaleIncrement = 1;
trigger.MetricName = "servicefabric:/_CpuCores";
trigger.ScaleInterval = TimeSpan.FromMinutes(20);
trigger.LowerLoadThreshold = 1.0;
trigger.UpperLoadThreshold = 2.0;
ScalingPolicyDescription policy = new ScalingPolicyDescription(mechanism, trigger);
serviceDescription.ScalingPolicies.Add(policy);
//as we are using scaling on a resource this must be exclusive service
//also resource monitor service needs to be enabled
serviceDescription.ServicePackageActivationMode = ServicePackageActivationMode.ExclusiveProcess
await fabricClient.ServiceManager.CreateServiceAsync(serviceDescription);
```
### <a name="using-powershell"></a>使用 PowerShell
```posh
$mechanism = New-Object -TypeName System.Fabric.Description.PartitionInstanceCountScaleMechanism
$mechanism.MinInstanceCount = 1
$mechanism.MaxInstanceCount = 6
$mechanism.ScaleIncrement = 2
$trigger = New-Object -TypeName System.Fabric.Description.AveragePartitionLoadScalingTrigger
$trigger.MetricName = "servicefabric:/_CpuCores"
$trigger.LowerLoadThreshold = 0.3
$trigger.UpperLoadThreshold = 0.8
$trigger.ScaleInterval = New-TimeSpan -Minutes 10
$scalingpolicy = New-Object -TypeName System.Fabric.Description.ScalingPolicyDescription
$scalingpolicy.ScalingMechanism = $mechanism
$scalingpolicy.ScalingTrigger = $trigger
$scalingpolicies = New-Object 'System.Collections.Generic.List[System.Fabric.Description.ScalingPolicyDescription]'
$scalingpolicies.Add($scalingpolicy)
#as we are using scaling on a resource this must be exclusive service
#also resource monitor service needs to be enabled
Update-ServiceFabricService -Stateless -ServiceName "fabric:/AppName/ServiceName" -ScalingPolicies $scalingpolicies
```

## <a name="average-service-load-trigger-with-partition-based-scaling"></a>搭配分割區型調整的平均服務負載觸發程序
第二種觸發程序乃是根據一個服務的所有分割區負載。 計量負載會先經過平滑處理以取得分割區每個複本或執行個體的負載。 就具狀態服務而言，分割區的負載被視為是主要複本的負載，而就無狀態服務而言，分割區的負載則是分割區所有執行個體的平均負載。 系統會從服務的所有分割區中算出這些值的平均值，然後使用此值來觸發自動調整。 與前一個機制相同，決定服務調整時機的因素有三個：

* 「負載閾值下限」是一個決定何時應該**相應縮小**服務規模的值。 如果服務的所有分割區平均負載低於此值，就會相應縮小服務規模。
* 「負載閾值上限」是一個決定何時應該**相應放大**服務規模的值。如果服務的所有分割區平均負載高於此值，就會相應放大服務規模。
* 「調整間隔」會決定檢查觸發程序的頻率。 檢查觸發程序之後，如果需要調整，就會套用此機制。 如果不需要調整，則不會執行任何動作。 在這兩種情況下，在調整間隔再次到期之前，都不會再次檢查觸發程序。

此觸發程序可以同時與具狀態和無狀態服務搭配使用。 唯一能夠與此觸發程序搭配使用的機制是 AddRemoveIncrementalNamedParitionScalingMechanism。 當服務相應放大規模時，就會新增一個分割區，而當服務相應縮小規模時，則會移除其中一個現有的分割區。 建立或更新服務時，會檢查一些限制，而如果不符合下列條件，服務的建立/更新就會失敗：
* 必須針對服務使用具名分割區配置。
* 分割區名稱必須是連續的整數數字，例如 "0"、"1"...
* 第一個分割區名稱必須是 "0"。

例如，如果服務一開始建立時有三個分割區，則唯一有效的可能分割區名稱會是 "0"、"1" 和 "2"。

所執行的實際自動調整作業也會遵守此命名配置：
* 如果目前的服務分割區命名為 "0"、"1" 和 "2"，則為了相應放大規模而將新增的分割區會命名為 "3"。
* 如果目前的服務分割區命名為 "0"、"1" 和 "2"，則為了相應縮小規模而將移除的分割區會是名為 "2" 的分割區。

與藉由新增或移除執行個體來使用調整的機制相同，決定此機制套用方式的參數有三個：
* 「調整增量」會決定當觸發機制時，要新增或移除多少個分割區。
* 「最大分割區計數」會定義調整的上限。 如果服務的分割區數目達到此限制，則不論負載情況為何，服務都不會相應放大規模。 您可以指定 -1 的值來省略此限制，在此情況下，服務將儘可能相應放大規模 (以叢集的實際容量為限)。
* 「最小執行個體計數」會定義調整的下限。 如果服務的分割區數目達到此限制，則不論負載情況為何，服務都不會相應縮小規模。

> [!WARNING] 
> 當 AddRemoveIncrementalNamedParitionScalingMechanism 搭配具狀態服務使用時，Service Fabric 會在**沒有任何通知或警告**的情況下新增或移除分割區。 觸發縮放機制後，就不會執行資料的重新分割。 在相應增加作業中，新的分割區會是空的，而在相應減少作業中，**分割區會與它所包含的所有資料一起刪除**。

## <a name="setting-auto-scaling-policy"></a>設定自動調整原則

### <a name="using-application-manifest"></a>使用應用程式資訊清單
``` xml
<ServiceScalingPolicies>
    <ScalingPolicy>
        <AverageServiceLoadScalingTrigger MetricName="servicefabric:/_MemoryInMB" LowerLoadThreshold="300" UpperLoadThreshold="500" ScaleIntervalInSeconds="600"/>
        <AddRemoveIncrementalNamedParitionScalingMechanism MinPartitionCount="1" MaxPartitionCount="3" ScaleIncrement="1"/>
    </ScalingPolicy>
</ServiceScalingPolicies>
```
### <a name="using-c-apis"></a>使用 C# API
```csharp
FabricClient fabricClient = new FabricClient();
StatefulServiceUpdateDescription serviceUpdate = new StatefulServiceUpdateDescription();
AveragePartitionLoadScalingTrigger trigger = new AverageServiceLoadScalingTrigger();
PartitionInstanceCountScaleMechanism mechanism = new AddRemoveIncrementalNamedParitionScalingMechanism();
mechanism.MaxPartitionCount = 4;
mechanism.MinPartitionCount = 1;
mechanism.ScaleIncrement = 1;
//expecting that the service already has metric NumberOfConnections
trigger.MetricName = "NumberOfConnections";
trigger.ScaleInterval = TimeSpan.FromMinutes(15);
trigger.LowerLoadThreshold = 10000;
trigger.UpperLoadThreshold = 20000;
ScalingPolicyDescription policy = new ScalingPolicyDescription(mechanism, trigger);
serviceUpdate.ScalingPolicies = new List<ScalingPolicyDescription>;
serviceUpdate.ScalingPolicies.Add(policy);
await fabricClient.ServiceManager.UpdateServiceAsync(new Uri("fabric:/AppName/ServiceName"), serviceUpdate);
```
### <a name="using-powershell"></a>使用 PowerShell
```posh
$mechanism = New-Object -TypeName System.Fabric.Description.AddRemoveIncrementalNamedParitionScalingMechanism
$mechanism.MinPartitionCount = 1
$mechanism.MaxPartitionCount = 3
$mechanism.ScaleIncrement = 2
$trigger = New-Object -TypeName System.Fabric.Description.AverageServiceLoadScalingTrigger
$trigger.MetricName = "servicefabric:/_MemoryInMB"
$trigger.LowerLoadThreshold = 5000
$trigger.UpperLoadThreshold = 10000
$trigger.ScaleInterval = New-TimeSpan -Minutes 25
$scalingpolicy = New-Object -TypeName System.Fabric.Description.ScalingPolicyDescription
$scalingpolicy.ScalingMechanism = $mechanism
$scalingpolicy.ScalingTrigger = $trigger
$scalingpolicies = New-Object 'System.Collections.Generic.List[System.Fabric.Description.ScalingPolicyDescription]'
$scalingpolicies.Add($scalingpolicy)
#as we are using scaling on a resource this must be exclusive service
#also resource monitor service needs to be enabled
New-ServiceFabricService -ApplicationName $applicationName -ServiceName $serviceName -ServiceTypeName $serviceTypeName –Stateful -TargetReplicaSetSize 3 -MinReplicaSetSize 2 -HasPersistedState true -PartitionNames @("0","1") -ServicePackageActivationMode ExclusiveProcess -ScalingPolicies $scalingpolicies
```

## <a name="auto-scaling-based-on-resources"></a>根據資源自動調整

讓資源監視服務根據實際的資源進行調整

``` json
"fabricSettings": [
...      
],
"addonFeatures": [
    "ResourceMonitorService"
],
```
有兩個代表實際實體資源的計量。 其中一個是 servicefabric:/_CpuCores，代表實際的 CPU 使用量 (因此 0.5 代表半個核心)，另一個是 servicefabric:/_MemoryInMB，代表記憶體使用量 (以 MB 為單位)。
ResourceMonitorService 會負責追蹤使用者服務的 CPU 和記憶體使用量。 此服務會套用加權移動平均，以說明可能的短期尖峰情況。 針對 Windows 上的容器化和非容器化應用程式，以及 Linux 上的容器化應用程式，都可支援資源監視。 只有針對在[專屬處理序模型](service-fabric-hosting-model.md#exclusive-process-model)中啟用的服務，才會啟用資源上的自動調整。

## <a name="next-steps"></a>後續步驟
深入了解[應用程式延展性](service-fabric-concepts-scalability.md)。
