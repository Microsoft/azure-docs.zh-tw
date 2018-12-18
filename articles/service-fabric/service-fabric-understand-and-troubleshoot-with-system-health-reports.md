---
title: 使用系統健康狀態報告進行疑難排解 | Microsoft 疑難排解
description: 描述針對 Azure Service Fabric 元件及其使用量所傳送的健康狀態報告，以便對叢集或應用程式問題進行疑難排解
services: service-fabric
documentationcenter: .net
author: oanapl
manager: timlt
editor: ''
ms.assetid: 52574ea7-eb37-47e0-a20a-101539177625
ms.service: service-fabric
ms.devlang: dotnet
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 2/28/2018
ms.author: oanapl
ms.openlocfilehash: 8304790b5eba4679b0633641c82d57316e7f8ec4
ms.sourcegitcommit: eb75f177fc59d90b1b667afcfe64ac51936e2638
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 05/16/2018
ms.locfileid: "34210829"
---
# <a name="use-system-health-reports-to-troubleshoot"></a>使用系統健康狀態報告進行疑難排解
Azure Service Fabric 元件會針對現成叢集中的所有實體，提供系統健康情況報告。 [健康狀態資料存放區](service-fabric-health-introduction.md#health-store) 會根據系統報告來建立和刪除實體。 它也會將這些實體組織為階層以擷取實體的互動。

> [!NOTE]
> 若要了解健康狀態相關概念，請詳細閱讀 [Service Fabric 健康狀態模型](service-fabric-health-introduction.md)。
> 
> 

系統健康情況報告可顯示叢集和應用程式功能，並標記問題。 系統健康狀態報告會針對應用程式和服務來確認實體是否已實作，以及從 Service Fabric 的角度來確認其是行為是否正確。 這些報告並不會監控任何服務商務邏輯的健康情況，也不會偵測是否有無回應的處理程序。 使用者服務可使用其邏輯的特定資訊讓健康情況資料更豐富。

> [!NOTE]
> 使用者監視程式所傳送的健康情況報告只有在系統元件建立實體*之後*才會顯示。 刪除實體時，健康狀態資料存放區會自動刪除所有與其相關聯的健康情況報告。 建立實體的新執行個體時也是如此。 例如，在建立新的具狀態持續性服務複本執行個體時。 所有與舊執行個體相關聯的報告都會從存放區刪除及清除。
> 
> 

系統元件報告將由來源識別，它會以 **System.** 前置詞做為開頭 。 看門狗不能對來源使用相同的前置詞，因為含有無效參數的報告會遭到拒絕。

讓我們來看看部分系統報告，了解何者觸發了它們，以及如何修正它們所代表的潛在問題。

> [!NOTE]
> Service Fabric 會繼續以感興趣的條件來新增報告，藉此改善叢集和應用程式中狀況的可見性。 更多詳細資料可以增強現有的報告，有助於更快針對問題進行疑難排解。
> 
> 

## <a name="cluster-system-health-reports"></a>叢集系統健康狀態報告
健康狀態資料存放區中會自動建立叢集健康狀態實體。 如果一切正常運作，則不會有系統報告。

### <a name="neighborhood-loss"></a>網路上的芳鄰遺失
**System.Federation** 偵測到網路上的芳鄰遺失時，即會回報錯誤。 報告來自各別的節點，且節點識別碼會包含於屬性名稱中。 如果整個 Service Fabric 通道中有一個網路上的芳鄰遺失，您通常可預期會產生兩個能夠代表兩邊差距報告的事件。 如果有多個網路上的芳鄰遺失，將會產生更多的事件。

報告會將全域租用逾時指定為存留時間 (TTL)。 只要條件仍在作用中，就會在每半個 TTL 期間重新傳送一次報告。 事件到期時會自動移除。 到期時移除的行為可確保正確地從健康狀態資料存放區清除報告，即使報告節點已關閉也不例外。

* **SourceId**：System.Federation
* **Property**：開頭為 **Neighborhood**，且包含節點資訊。
* **後續步驟**：調查網路上的芳鄰遺失的原因。 例如，檢查叢集節點之間的通訊。

### <a name="rebuild"></a>重建

容錯移轉管理員 (FM) 服務管理叢集節點的相關資訊。 當 FM 失去其資料，然後進入資料遺失狀態時，它無法保證能擁有叢集節點的最新相關資訊。 在此情況下，系統會執行重建程序，而 System.FM 會從叢集中所有節點收集資料以重建其狀態。 有時候，由於網路或節點問題，重建可能會當機或停止。 容錯移轉管理員主節點 (FMM) 服務也可能發生同樣的情況。 FMM 是無狀態的系統服務，用於追蹤所有 FM 在叢集中的位置。 FMM 主節點一律為識別碼最接近 0 的節點。 如果卸除該節點，會觸發重建。
當先前的某個情況發生時，**System.FM** 或 **System.FMM** 會透過錯誤報告為它加上旗標。 重建可能會卡在兩個階段其中之一：

* **等候廣播**：FM/FMM 等候來自其他節點的廣播訊息回覆。

  * **後續步驟**：調查節點之間是否有網路連線問題。
* **等候節點**：FM/FMM 已經收到來自其他節點的廣播回覆，而且正在等候特定節點的回覆。 健康情況報告列出 FM/FMM 正在等候回應的節點。
   * **後續步驟**：調查 FM/FMM 與所列節點之間的網路連線。 調查每個列出的節點以尋找其他可能的問題。

* **SourceID**：System.FM 或 System.FMM
* **屬性**：重建。
* **後續步驟**：調查節點之間的網路連線，以及健康情況報告描述中列出之任何特定節點的狀態。

## <a name="node-system-health-reports"></a>節點系統健康狀態報告
System.FM(代表容錯移轉管理員服務) 是管理叢集節點相關資訊的授權單位。 每個節點都應該有一份來自 System.FM 的報告，以顯示其狀態。 移除節點狀態時會移除節點實體。 如需詳細資訊，請參閱 [RemoveNodeStateAsync](https://docs.microsoft.com/dotnet/api/system.fabric.fabricclient.clustermanagementclient.removenodestateasync)。

### <a name="node-updown"></a>節點運作中/關閉
當節點加入通道時，System.FM 會回報為 OK (節點已啟動且正在運作中)。 當節點離開通道時，則會回報錯誤 (節點已關閉進行升級，或只是發生故障)。 由健康狀態資料存放區建置的健康情況階層會根據 System.FM 節點報告，對部署的實體採取行動。 它會將節點視為所有已部署實體的虛擬父系。 如果 System.FM 回報指出該節點已啟動，且其執行個體與實體相關聯的執行個體相同，則該節點上已部署的實體將會透過查詢公開。 當 System.FM 回報節點已當作新執行個體關閉或重新啟動時，健康狀態資料存放區會自動清除僅能存在於已關閉節點或先前的節點執行個體上的已部署實體。

* **SourceId**：System.FM
* **Property**：State。
* **後續步驟**：如果節點已關閉來進行升級，應該會在升級後重新啟動。 在這種情況下，健康情況應切換回 OK。 如果節點沒有重新啟動或故障，就需要進一步調查問題。

以下範例說明帶有 OK (代表節點已啟動) 健全狀況狀態的 System.FM 事件：

```PowerShell
PS C:\> Get-ServiceFabricNodeHealth  _Node_0

NodeName              : _Node_0
AggregatedHealthState : Ok
HealthEvents          : 
                        SourceId              : System.FM
                        Property              : State
                        HealthState           : Ok
                        SequenceNumber        : 8
                        SentAt                : 7/14/2017 4:54:51 PM
                        ReceivedAt            : 7/14/2017 4:55:14 PM
                        TTL                   : Infinite
                        Description           : Fabric node is up.
                        RemoveWhenExpired     : False
                        IsExpired             : False
                        Transitions           : Error->Ok = 7/14/2017 4:55:14 PM, LastWarning = 1/1/0001 12:00:00 AM
```


### <a name="certificate-expiration"></a>憑證到期
**System.FabricNode** 會回報警告。 每個節點有三個憑證：**Certificate_cluster**、**Certificate_server** 及 **Certificate_default_client**。 如果過期時間至少超過兩週，報告健康狀態就是 OK。 如果過期時間是在兩週內，則報告類型會是 Warning。 這些事件的 TTL 是無限制的，只有節點離開叢集時才會被移除。

* **SourceId**：System.FabricNode
* **Property**：開頭為 **Certificate**，且包含關於憑證類型的詳細資訊。
* **後續步驟**：如果憑證即將到期，請更新憑證。

### <a name="load-capacity-violation"></a>負載容量違規
當 Service Fabric Load Balancer 偵測到節點容量違規時，就會回報警告。

* **SourceId**：System.PLB
* **Property**：開頭為 **Capacity**。
* **後續步驟**：檢查提供的計量，並檢視節點上的目前容量。

### <a name="node-capacity-mismatch-for-resource-governance-metrics"></a>節點容量與資源控管計量不符
如果叢集資訊清單中定義的節點容量大於資源控管計量 (記憶體和 CPU 核心) 的實際節點容量，則 System.Hosting 會回報警告。 當第一個使用[資源控管](service-fabric-resource-governance.md)的服務套件在指定的節點上註冊時，即會顯示健康情況報告。

* **SourceId**：System.Hosting
* **Property**：**ResourceGovernance**。
* **後續步驟**：這個問題可能會造成麻煩，因為控管的服務套件不會如預期般強制執行，而[資源控管](service-fabric-resource-governance.md)無法正常運作。 使用這些計量的正確節點容量來更新叢集資訊清單，或者不要指定它們，讓 Service Fabric 自動偵測可用的資源。

## <a name="application-system-health-reports"></a>應用程式系統健康狀態報告
System.CM(代表叢集管理員服務) 是管理應用程式相關資訊的授權單位。

### <a name="state"></a>State
已建立或更新應用程式時，System.CM 會回報為 OK。 刪除應用程式時，它會通知健康狀態資料存放區，以便從存放區將它移除。

* **SourceId**：System.CM
* **Property**：State。
* **後續步驟**：如果已建立或更新應用程式，它就應該包含叢集管理員健康情況報告。 否則，請藉由發出查詢來檢查應用程式的狀態。 例如，使用 PowerShell Cmdlet **Get-ServiceFabricApplication -ApplicationName** applicationName。

以下範例說明 **fabric:/WordCount** 應用程式上的狀態事件：

```PowerShell
PS C:\> Get-ServiceFabricApplicationHealth fabric:/WordCount -ServicesFilter None -DeployedApplicationsFilter None -ExcludeHealthStatistics

ApplicationName                 : fabric:/WordCount
AggregatedHealthState           : Ok
ServiceHealthStates             : None
DeployedApplicationHealthStates : None
HealthEvents                    : 
                                  SourceId              : System.CM
                                  Property              : State
                                  HealthState           : Ok
                                  SequenceNumber        : 282
                                  SentAt                : 7/13/2017 5:57:05 PM
                                  ReceivedAt            : 7/14/2017 4:55:10 PM
                                  TTL                   : Infinite
                                  Description           : Application has been created.
                                  RemoveWhenExpired     : False
                                  IsExpired             : False
                                  Transitions           : Error->Ok = 7/13/2017 5:57:05 PM, LastWarning = 1/1/0001 12:00:00 AM
```

## <a name="service-system-health-reports"></a>服務系統健康狀態報告
System.FM(代表容錯移轉管理員服務) 是管理服務相關資訊的授權單位。

### <a name="state"></a>State
已建立服務時，System.FM 會回報為 OK。 已刪除服務時，它會從健康狀態資料存放區刪除實體。

* **SourceId**：System.FM
* **Property**：State。

以下範例說明 **fabric:/WordCount/WordCountWebService** 服務上的狀態事件：

```PowerShell
PS C:\> Get-ServiceFabricServiceHealth fabric:/WordCount/WordCountWebService -ExcludeHealthStatistics


ServiceName           : fabric:/WordCount/WordCountWebService
AggregatedHealthState : Ok
PartitionHealthStates : 
                        PartitionId           : 8bbcd03a-3a53-47ec-a5f1-9b77f73c53b2
                        AggregatedHealthState : Ok
                        
HealthEvents          : 
                        SourceId              : System.FM
                        Property              : State
                        HealthState           : Ok
                        SequenceNumber        : 14
                        SentAt                : 7/13/2017 5:57:05 PM
                        ReceivedAt            : 7/14/2017 4:55:10 PM
                        TTL                   : Infinite
                        Description           : Service has been created.
                        RemoveWhenExpired     : False
                        IsExpired             : False
                        Transitions           : Error->Ok = 7/13/2017 5:57:18 PM, LastWarning = 1/1/0001 12:00:00 AM
```

### <a name="service-correlation-error"></a>服務相互關聯錯誤
**System.PLB** 在偵測到更新服務與建立同質鏈結的其他服務相互關聯時，就會回報錯誤。 更新成功時，就會清除報告。

* **SourceId**：System.PLB
* **Property**：**ServiceDescription**。
* **後續步驟**：檢查相互關聯的服務說明。

## <a name="partition-system-health-reports"></a>分割區系統健康狀態報告
System.FM(代表容錯移轉管理員服務) 是管理服務分割區相關資訊的授權單位。

### <a name="state"></a>State
已建立分割區且其狀況良好時，System.FM 會回報為 OK。 刪除分割區時，它會從健康狀態資料存放區刪除實體。

如果分割區低於最小複本計數，它會回報錯誤。 如果分割區高於最小複本計數，但低於目標複本計數，則會回報警告。 如果分割區處於仲裁遺失狀態，System.FM 會回報錯誤。

其他值得注意的事件包括：當重新設定花費的時間比預期長，或者建置的時間比預期長，則會回報警告。 建置和重新設定的預計時間可根據服務案例來設定。 例如，如果服務擁有 1 TB 的狀態 (例如 Azure SQL Database)，則建置的時間會比只有少量狀態的服務更長。

* **SourceId**：System.FM
* **Property**：State。
* **後續步驟**：如果健康狀態不是 OK，有可能是因為部分複本並沒有正確建立、開啟、提升為主要或次要的複本。 

如果描述說明仲裁遺失，則檢查已關閉之複本的詳細健康情況報告，並讓它們重新運作，有助於讓分割區重新上線。

如果描述說明分割區停滯於[重新設定](service-fabric-concepts-reconfiguration.md)，則主要複本上的健康情況報告會提供額外資訊。

對於其他 System.FM 健康情況報告，則會提供關於複本或是來自其他系統元件之分割區或服務的報告。 

下列範例說明這其中的部分報告。 

以下範例顯示狀況良好的分割區：

```PowerShell
PS C:\> Get-ServiceFabricPartition fabric:/WordCount/WordCountWebService | Get-ServiceFabricPartitionHealth -ExcludeHealthStatistics -ReplicasFilter None

PartitionId           : 8bbcd03a-3a53-47ec-a5f1-9b77f73c53b2
AggregatedHealthState : Ok
ReplicaHealthStates   : None
HealthEvents          : 
                        SourceId              : System.FM
                        Property              : State
                        HealthState           : Ok
                        SequenceNumber        : 70
                        SentAt                : 7/13/2017 5:57:05 PM
                        ReceivedAt            : 7/14/2017 4:55:10 PM
                        TTL                   : Infinite
                        Description           : Partition is healthy.
                        RemoveWhenExpired     : False
                        IsExpired             : False
                        Transitions           : Error->Ok = 7/13/2017 5:57:18 PM, LastWarning = 1/1/0001 12:00:00 AM
```

以下範例顯示低於目標複本計數之分割區的健康情況。 後續步驟是取得分割區描述，它將說明設定方式：**MinReplicaSetSize** 為 3，且 **TargetReplicaSetSize** 為 7。 接著取得叢集中的節點數目，在此案例中為 5。 因此，在此情況下，無法放置兩個複本，因為複本的目標數目大於可用節點的數目。

```PowerShell
PS C:\> Get-ServiceFabricPartition fabric:/WordCount/WordCountService | Get-ServiceFabricPartitionHealth -ReplicasFilter None -ExcludeHealthStatistics


PartitionId           : af2e3e44-a8f8-45ac-9f31-4093eb897600
AggregatedHealthState : Warning
UnhealthyEvaluations  : 
                        Unhealthy event: SourceId='System.FM', Property='State', HealthState='Warning', ConsiderWarningAsError=false.
                        
ReplicaHealthStates   : None
HealthEvents          : 
                        SourceId              : System.FM
                        Property              : State
                        HealthState           : Warning
                        SequenceNumber        : 123
                        SentAt                : 7/14/2017 4:55:39 PM
                        ReceivedAt            : 7/14/2017 4:55:44 PM
                        TTL                   : Infinite
                        Description           : Partition is below target replica or instance count.
                        fabric:/WordCount/WordCountService 7 2 af2e3e44-a8f8-45ac-9f31-4093eb897600
                          N/S Ready _Node_2 131444422260002646
                          N/S Ready _Node_4 131444422293113678
                          N/S Ready _Node_3 131444422293113679
                          N/S Ready _Node_1 131444422293118720
                          N/P Ready _Node_0 131444422293118721
                          (Showing 5 out of 5 replicas. Total available replicas: 5)
                        
                        RemoveWhenExpired     : False
                        IsExpired             : False
                        Transitions           : Error->Warning = 7/14/2017 4:55:44 PM, LastOk = 1/1/0001 12:00:00 AM
                        
                        SourceId              : System.PLB
                        Property              : ServiceReplicaUnplacedHealth_Secondary_af2e3e44-a8f8-45ac-9f31-4093eb897600
                        HealthState           : Warning
                        SequenceNumber        : 131445250939703027
                        SentAt                : 7/14/2017 4:58:13 PM
                        ReceivedAt            : 7/14/2017 4:58:14 PM
                        TTL                   : 00:01:05
                        Description           : The Load Balancer was unable to find a placement for one or more of the Service's Replicas:
                        Secondary replica could not be placed due to the following constraints and properties:  
                        TargetReplicaSetSize: 7
                        Placement Constraint: N/A
                        Parent Service: N/A
                        
                        Constraint Elimination Sequence:
                        Existing Secondary Replicas eliminated 4 possible node(s) for placement -- 1/5 node(s) remain.
                        Existing Primary Replica eliminated 1 possible node(s) for placement -- 0/5 node(s) remain.
                        
                        Nodes Eliminated By Constraints:
                        
                        Existing Secondary Replicas -- Nodes with Partition's Existing Secondary Replicas/Instances:
                        --
                        FaultDomain:fd:/4 NodeName:_Node_4 NodeType:NodeType4 UpgradeDomain:4 UpgradeDomain: ud:/4 Deactivation Intent/Status: None/None
                        FaultDomain:fd:/3 NodeName:_Node_3 NodeType:NodeType3 UpgradeDomain:3 UpgradeDomain: ud:/3 Deactivation Intent/Status: None/None
                        FaultDomain:fd:/2 NodeName:_Node_2 NodeType:NodeType2 UpgradeDomain:2 UpgradeDomain: ud:/2 Deactivation Intent/Status: None/None
                        FaultDomain:fd:/1 NodeName:_Node_1 NodeType:NodeType1 UpgradeDomain:1 UpgradeDomain: ud:/1 Deactivation Intent/Status: None/None
                        
                        Existing Primary Replica -- Nodes with Partition's Existing Primary Replica or Secondary Replicas:
                        --
                        FaultDomain:fd:/0 NodeName:_Node_0 NodeType:NodeType0 UpgradeDomain:0 UpgradeDomain: ud:/0 Deactivation Intent/Status: None/None
                        
                        
                        RemoveWhenExpired     : True
                        IsExpired             : False
                        Transitions           : Error->Warning = 7/14/2017 4:56:14 PM, LastOk = 1/1/0001 12:00:00 AM

PS C:\> Get-ServiceFabricPartition fabric:/WordCount/WordCountService | select MinReplicaSetSize,TargetReplicaSetSize

MinReplicaSetSize TargetReplicaSetSize
----------------- --------------------
                2                    7                        

PS C:\> @(Get-ServiceFabricNode).Count
5
```

下列範例顯示分割區的健康情況，此分割區因為使用者不接受 **RunAsync** 方法中的取消權杖而停滯於重新設定過程中。 調查任何標示為「主要 (P)」之複本的健康情況報告，有助於進一步深入探索問題。

```PowerShell
PS C:\utilities\ServiceFabricExplorer\ClientPackage\lib> Get-ServiceFabricPartitionHealth 0e40fd81-284d-4be4-a665-13bc5a6607ec -ExcludeHealthStatistics 


PartitionId           : 0e40fd81-284d-4be4-a665-13bc5a6607ec
AggregatedHealthState : Warning
UnhealthyEvaluations  : 
                        Unhealthy event: SourceId='System.FM', Property='State', HealthState='Warning', 
                        ConsiderWarningAsError=false.
                                               
HealthEvents          : 
                        SourceId              : System.FM
                        Property              : State
                        HealthState           : Warning
                        SequenceNumber        : 7
                        SentAt                : 8/27/2017 3:43:09 AM
                        ReceivedAt            : 8/27/2017 3:43:32 AM
                        TTL                   : Infinite
                        Description           : Partition reconfiguration is taking longer than expected.
                        fabric:/app/test1 3 1 0e40fd81-284d-4be4-a665-13bc5a6607ec
                          P/S Ready Node1 131482789658160654
                          S/P Ready Node2 131482789688598467
                          S/S Ready Node3 131482789688598468
                          (Showing 3 out of 3 replicas. Total available replicas: 3)                        
                        
                        For more information see: http://aka.ms/sfhealth
                        RemoveWhenExpired     : False
                        IsExpired             : False
                        Transitions           : Ok->Warning = 8/27/2017 3:43:32 AM, LastError = 1/1/0001 12:00:00 AM
```
這份健康情況報告顯示正處於重新設定之分割區的複本狀態： 

```
  P/S Ready Node1 131482789658160654
  S/P Ready Node2 131482789688598467
  S/S Ready Node3 131482789688598468
```

針對每個複本，健康情況報告包含：
- 先前的設定角色
- 目前的設定角色
- [複本狀態](service-fabric-concepts-replica-lifecycle.md)
- 複本執行所在的節點
- 複本識別碼

在如同範例的情況下，需要進一步調查。 針對上述範例中，複本開頭標示為 `Primary` 和 `Secondary` (131482789658160654 及 131482789688598467) 的每個個別複本，調查其健康情況。

### <a name="replica-constraint-violation"></a>複本條件約束違規
**System.PLB** 偵測到複本條件約束違規，且無法安置所有磁碟分割複本時，就會回報警告。 報告詳細資料會顯示哪些條件約束和屬性導致無法安置複本。

* **SourceId**：System.PLB
* **Property**：開頭為 **ReplicaConstraintViolation**。

## <a name="replica-system-health-reports"></a>複本系統健康狀態報告
**System.RA**(代表重新設定代理程式元件) 是複本狀態的授權單位。

### <a name="state"></a>State
System.RA 會在複本建立後回報 OK。

* **SourceId**：System.RA
* **Property**：State。

以下範例顯示狀況良好的複本：

```PowerShell
PS C:\> Get-ServiceFabricPartition fabric:/WordCount/WordCountService | Get-ServiceFabricReplica | where {$_.ReplicaRole -eq "Primary"} | Get-ServiceFabricReplicaHealth

PartitionId           : af2e3e44-a8f8-45ac-9f31-4093eb897600
ReplicaId             : 131444422293118721
AggregatedHealthState : Ok
HealthEvents          : 
                        SourceId              : System.RA
                        Property              : State
                        HealthState           : Ok
                        SequenceNumber        : 131445248920273536
                        SentAt                : 7/14/2017 4:54:52 PM
                        ReceivedAt            : 7/14/2017 4:55:13 PM
                        TTL                   : Infinite
                        Description           : Replica has been created._Node_0
                        RemoveWhenExpired     : False
                        IsExpired             : False
                        Transitions           : Error->Ok = 7/14/2017 4:55:13 PM, LastWarning = 1/1/0001 12:00:00 AM
```

### <a name="replicaopenstatus-replicaclosestatus-replicachangerolestatus"></a>ReplicaOpenStatus、ReplicaCloseStatus、ReplicaChangeRoleStatus
這個屬性用於表示在嘗試開啟複本、關閉複本或將複本從一個角色轉換為另一個角色時的警告或失敗。 如需詳細資訊，請參閱[複本生命週期](service-fabric-concepts-replica-lifecycle.md)。 失敗可能是在這段期間，從 API 呼叫或服務主機處理序損毀時擲回的例外狀況。 針對因為來自 C# 程式碼的 API 呼叫而導致的失敗，Service Fabric 將會在健康情況報告中新增例外狀況和堆疊追蹤。

這些健康情況警告會在本機重試該動作數次之後引發 (根據原則而定)。 Service Fabric 會重試該動作，直到達到閾值上限為止。 達到閾值上限之後，它可能會嘗試採取更正這種情況的動作。 這個嘗試可能會在放棄針對此節點採取動作時，導致這些警告被清除。 例如，如果複本無法在節點上開啟，Service Fabric 將會引發健康情況警告。 如果複本仍然無法開啟，Service Fabric 則會採取自我修復的動作。 此動作可能涉及在另一個節點上嘗試相同的作業。 這個嘗試會導致針對此複本引發的警告被清除。 

* **SourceId**：System.RA
* **Property**：**ReplicaOpenStatus**、**ReplicaCloseStatus** 和 **ReplicaChangeRoleStatus**。
* **後續步驟**：調查服務程式碼或損毀傾印來識別作業失敗的原因。

下列範例顯示從其 open 方法擲回 `TargetInvocationException` 之複本的健康情況。 描述包含失敗點 **IStatefulServiceReplica.Open**、例外狀況類型 **TargetInvocationException** 和堆疊追蹤。

```PowerShell
PS C:\> Get-ServiceFabricReplicaHealth -PartitionId 337cf1df-6cab-4825-99a9-7595090c0b1b -ReplicaOrInstanceId 131483509874784794


PartitionId           : 337cf1df-6cab-4825-99a9-7595090c0b1b
ReplicaId             : 131483509874784794
AggregatedHealthState : Warning
UnhealthyEvaluations  : 
                        Unhealthy event: SourceId='System.RA', Property='ReplicaOpenStatus', HealthState='Warning', 
                        ConsiderWarningAsError=false.
                        
HealthEvents          : 
                        SourceId              : System.RA
                        Property              : ReplicaOpenStatus
                        HealthState           : Warning
                        SequenceNumber        : 131483510001453159
                        SentAt                : 8/27/2017 11:43:20 PM
                        ReceivedAt            : 8/27/2017 11:43:21 PM
                        TTL                   : Infinite
                        Description           : Replica had multiple failures during open on _Node_0 API call: IStatefulServiceReplica.Open(); Error = System.Reflection.TargetInvocationException (-2146232828)
Exception has been thrown by the target of an invocation.
   at Microsoft.ServiceFabric.Replicator.RecoveryManager.d__31.MoveNext()
--- End of stack trace from previous location where exception was thrown ---
   at System.Runtime.CompilerServices.TaskAwaiter.ThrowForNonSuccess(Task task)
   at System.Runtime.CompilerServices.TaskAwaiter.HandleNonSuccessAndDebuggerNotification(Task task)
   at Microsoft.ServiceFabric.Replicator.LoggingReplicator.d__137.MoveNext()
--- End of stack trace from previous location where exception was thrown ---
   at System.Runtime.CompilerServices.TaskAwaiter.ThrowForNonSuccess(Task task)
   at System.Runtime.CompilerServices.TaskAwaiter.HandleNonSuccessAndDebuggerNotification(Task task)
   at Microsoft.ServiceFabric.Replicator.DynamicStateManager.d__109.MoveNext()
--- End of stack trace from previous location where exception was thrown ---
   at System.Runtime.CompilerServices.TaskAwaiter.ThrowForNonSuccess(Task task)
   at System.Runtime.CompilerServices.TaskAwaiter.HandleNonSuccessAndDebuggerNotification(Task task)
   at Microsoft.ServiceFabric.Replicator.TransactionalReplicator.d__79.MoveNext()
--- End of stack trace from previous location where exception was thrown ---
   at System.Runtime.CompilerServices.TaskAwaiter.ThrowForNonSuccess(Task task)
   at System.Runtime.CompilerServices.TaskAwaiter.HandleNonSuccessAndDebuggerNotification(Task task)
   at Microsoft.ServiceFabric.Replicator.StatefulServiceReplica.d__21.MoveNext()
--- End of stack trace from previous location where exception was thrown ---
   at System.Runtime.CompilerServices.TaskAwaiter.ThrowForNonSuccess(Task task)
   at System.Runtime.CompilerServices.TaskAwaiter.HandleNonSuccessAndDebuggerNotification(Task task)
   at Microsoft.ServiceFabric.Services.Runtime.StatefulServiceReplicaAdapter.d__0.MoveNext()

    For more information see: http://aka.ms/sfhealth
                        RemoveWhenExpired     : False
                        IsExpired             : False
                        Transitions           : Error->Warning = 8/27/2017 11:43:21 PM, LastOk = 1/1/0001 12:00:00 AM                        
```

下列範例顯示在關閉期間持續損毀的複本：

```PowerShell
C:>Get-ServiceFabricReplicaHealth -PartitionId dcafb6b7-9446-425c-8b90-b3fdf3859e64 -ReplicaOrInstanceId 131483565548493142


PartitionId           : dcafb6b7-9446-425c-8b90-b3fdf3859e64
ReplicaId             : 131483565548493142
AggregatedHealthState : Warning
UnhealthyEvaluations  : 
                        Unhealthy event: SourceId='System.RA', Property='ReplicaCloseStatus', HealthState='Warning', 
                        ConsiderWarningAsError=false.
                        
HealthEvents          : 
                        SourceId              : System.RA
                        Property              : ReplicaCloseStatus
                        HealthState           : Warning
                        SequenceNumber        : 131483565611258984
                        SentAt                : 8/28/2017 1:16:01 AM
                        ReceivedAt            : 8/28/2017 1:16:03 AM
                        TTL                   : Infinite
                        Description           : Replica had multiple failures during close on _Node_1. The application 
                        host has crashed.
                        
                        For more information see: http://aka.ms/sfhealth
                        RemoveWhenExpired     : False
                        IsExpired             : False
                        Transitions           : Error->Warning = 8/28/2017 1:16:03 AM, LastOk = 1/1/0001 12:00:00 AM
```

### <a name="reconfiguration"></a>Reconfiguration
這個屬性用來表示當執行[重新設定](service-fabric-concepts-reconfiguration.md)的複本偵測到重新設定已停止或停滯時。 這份健康情況報告可能位於其目前角色為主要的複本上，但交換主要重新設定的情況例外，其可能位於從主要降級為使用中次要的複本上。

重新設定可能基於下列其中一個原因而停滯：

- 本機複本上的動作 (與執行重新設定的複本相同的複本) 尚未完成。 在此情況下，從其他元件 (System.RAP 或 System.RE) 調查此複本上的健康情況報告可能會提供額外資訊。

- 遠端複本上的動作尚未完成。 健康情況報告中將列出動作擱置中的複本。 您應該在那些遠端複本的健康情況報告上進行進一步調查。 此外，在這個節點與遠端節點之間可能也有通訊問題。

在罕見的情況下，重新設定可能會因為這個節點與容錯移轉管理員服務之間的通訊或其他問題而停滯。

* **SourceId**：System.RA
* **Property**：Reconfiguration。
* **後續步驟**：根據健康情況報告的描述來調查本機或遠端複本。

下列範例會顯示本機複本上重新設定已停滯的健康情況報告。 在此範例中，這是因為服務未接受取消權杖的緣故。

```PowerShell
PS C:\> Get-ServiceFabricReplicaHealth -PartitionId 9a0cedee-464c-4603-abbc-1cf57c4454f3 -ReplicaOrInstanceId 131483600074836703


PartitionId           : 9a0cedee-464c-4603-abbc-1cf57c4454f3
ReplicaId             : 131483600074836703
AggregatedHealthState : Warning
UnhealthyEvaluations  : 
                        Unhealthy event: SourceId='System.RA', Property='Reconfiguration', HealthState='Warning', 
                        ConsiderWarningAsError=false.
                        
HealthEvents          : 
                        SourceId              : System.RA
                        Property              : Reconfiguration
                        HealthState           : Warning
                        SequenceNumber        : 131483600309264482
                        SentAt                : 8/28/2017 2:13:50 AM
                        ReceivedAt            : 8/28/2017 2:13:57 AM
                        TTL                   : Infinite
                        Description           : Reconfiguration is stuck. Waiting for response from the local replica
                        
                        For more information see: http://aka.ms/sfhealth
                        RemoveWhenExpired     : False
                        IsExpired             : False
                        Transitions           : Error->Warning = 8/28/2017 2:13:57 AM, LastOk = 1/1/0001 12:00:00 AM
```

下列範例會顯示重新設定已停滯，以等候兩個遠端複本回應的健康情況報告。 在此範例中，分割區上有三個複本，包括目前的主要複本。 

```Powershell
PS C:\> Get-ServiceFabricReplicaHealth -PartitionId  579d50c6-d670-4d25-af70-d706e4bc19a2 -ReplicaOrInstanceId 131483956274977415


PartitionId           : 579d50c6-d670-4d25-af70-d706e4bc19a2
ReplicaId             : 131483956274977415
AggregatedHealthState : Warning
UnhealthyEvaluations  : 
                        Unhealthy event: SourceId='System.RA', Property='Reconfiguration', HealthState='Warning', ConsiderWarningAsError=false.
                        
HealthEvents          : 
                        SourceId              : System.RA
                        Property              : Reconfiguration
                        HealthState           : Warning
                        SequenceNumber        : 131483960376212469
                        SentAt                : 8/28/2017 12:13:57 PM
                        ReceivedAt            : 8/28/2017 12:14:07 PM
                        TTL                   : Infinite
                        Description           : Reconfiguration is stuck. Waiting for response from 2 replicas
                        
                        Pending Replicas: 
                        P/I Down 40 131483956244554282
                        S/S Down 20 131483956274972403
                        
                        For more information see: http://aka.ms/sfhealth
                        RemoveWhenExpired     : False
                        IsExpired             : False
                        Transitions           : Error->Warning = 8/28/2017 12:07:37 PM, LastOk = 1/1/0001 12:00:00 AM
```

這份健康情況報告顯示重新設定已停滯，以等候來自兩個複本的回應： 

```
    P/I Down 40 131483956244554282
    S/S Down 20 131483956274972403
```

針對每個複本，提供了以下資訊：
- 先前的設定角色
- 目前的設定角色
- [複本狀態](service-fabric-concepts-replica-lifecycle.md)
- 節點識別碼
- 複本識別碼

解除封鎖重新設定：
- **down** 複本應該會重新運作。 
- **inbuild** 複本應該會完成建置並轉換為就緒。

### <a name="slow-service-api-call"></a>緩慢服務 API 呼叫
如果呼叫使用者服務程式碼所花費的時間超過設定的時間，**System.RAP** 和 **System.Replicator** 就會回報警告。 當呼叫完成時，警告就會被清除。

* **SourceId**：System.RAP 或 System.Replicator
* **Property**：緩慢 API 的名稱。 該說明會提供有關 API 擱置時間的詳細資訊。
* **後續步驟**：調查呼叫所花費時間超過預期的原因。

下列範例針對不接受 **RunAsync** 中之取消權杖的可靠服務，顯示來自 System.RAP 的健康情況態事件：

```PowerShell
PS C:\> Get-ServiceFabricReplicaHealth -PartitionId 5f6060fb-096f-45e4-8c3d-c26444d8dd10 -ReplicaOrInstanceId 131483966141404693


PartitionId           : 5f6060fb-096f-45e4-8c3d-c26444d8dd10
ReplicaId             : 131483966141404693
AggregatedHealthState : Warning
UnhealthyEvaluations  : 
                        Unhealthy event: SourceId='System.RA', Property='Reconfiguration', HealthState='Warning', ConsiderWarningAsError=false.
                        
HealthEvents          :                         
                        SourceId              : System.RAP
                        Property              : IStatefulServiceReplica.ChangeRole(S)Duration
                        HealthState           : Warning
                        SequenceNumber        : 131483966663476570
                        SentAt                : 8/28/2017 12:24:26 PM
                        ReceivedAt            : 8/28/2017 12:24:56 PM
                        TTL                   : Infinite
                        Description           : The api IStatefulServiceReplica.ChangeRole(S) on _Node_1 is stuck. Start Time (UTC): 2017-08-28 12:23:56.347.
                        RemoveWhenExpired     : False
                        IsExpired             : False
                        Transitions           : Error->Warning = 8/28/2017 12:24:56 PM, LastOk = 1/1/0001 12:00:00 AM
                        
```

屬性和文字會指出已停滯的 API。 針對不同已停滯 API 所採取的後續步驟皆不相同。 *IStatefulServiceReplica* 或 *IStatelessServiceInstance* 上的任何 API 通常都是服務程式碼中的錯誤 (Bug)。 下節說明如何將這些轉譯為 [Reliable Services 模型](service-fabric-reliable-services-lifecycle.md)：

- **IStatefulServiceReplica.Open**：此警告表示對 `CreateServiceInstanceListeners` 或 `ICommunicationListener.OpenAsync` 的呼叫，或者覆寫的 `OnOpenAsync` 是否已停滯。

- **IStatefulServiceReplica.Close** 和 **IStatefulServiceReplica.Abort**：最常見的案例是服務不接受傳遞至 `RunAsync` 的取消權杖。 也可能是那個 `ICommunicationListener.CloseAsync`，或者覆寫的 `OnCloseAsync` 是否已停滯。

- **IStatefulServiceReplica.ChangeRole(S)** 和 **IStatefulServiceReplica.ChangeRole(N)**：最常見的案例是服務不接受傳遞至 `RunAsync` 的取消權杖。

- **IStatefulServiceReplica.ChangeRole(P)**：最常見的案例是服務尚未從 `RunAsync` 傳回工作。

其他可能停滯的 API 呼叫均位於 **IReplicator** 介面上。 例如︰

- **IReplicator.CatchupReplicaSet**：此警告表示下列其中一種情況。 啟動的複本數不足。 若要了解是否為這種情況，請查看分割區中複本的複本狀態或 System.FM 健康情況報告，以進行停滯重新設定。 或者複本未認可作業。 PowerShell Cmdlet `Get-ServiceFabricDeployedReplicaDetail` 可用來判斷所有複本的進度。 問題出在其 `LastAppliedReplicationSequenceNumber` 值位於主要複本之 `CommittedSequenceNumber` 值後面的複本。

- **IReplicator.BuildReplica(<Remote ReplicaId>)**：此警告表示在建置程序發生問題。 如需詳細資訊，請參閱[複本生命週期](service-fabric-concepts-replica-lifecycle.md)。 可能是因為複寫器位址的設定不正確而造成。 如需詳細資訊，請參閱[設定具狀態可靠服務](service-fabric-reliable-services-configuration.md)和[在服務資訊清單中指定資源](service-fabric-service-manifest-resources.md)。 也可能是遠端節點上的問題。

### <a name="replicator-system-health-reports"></a>複寫器系統健康情況報告
**複寫佇列已滿：**
**System.Replicator** 會在複寫佇列已滿時回報警告。 在主要資料庫上，複寫佇列通常會因為一或多個次要複本太慢認可作業而排滿。 在次要複本上，這通常是因為服務緩慢而無法套用作業所造成。 當佇列有空間時，警告就會被清除。

* **SourceId**：System.Replicator
* **Property**：**PrimaryReplicationQueueStatus** 或 **SecondaryReplicationQueueStatus** (根據複本角色而定)。
* **後續步驟**：如果報表是在主要複本上，檢查叢集中節點之間的連線。 如果所有連線狀況良好，可能是至少一個具有高磁碟延遲時間的緩慢次要複本要套用作業。 如果報表是在次要複本上，請先檢查節點上的磁碟使用量和效能。 然後檢查從緩慢節點到主要複本的傳出連線。

**RemoteReplicatorConnectionStatus:**
**System.Replicator** 在主要複本上與次要 (遠端) 複寫器連線狀況不良時回報警告。 遠端複寫器的位址會顯示在報表的訊息中，讓您更方便地偵測是否有錯誤組態傳入或者複寫器之間是否有網路問題。

* **SourceId**：System.Replicator
* **Property**：**RemoteReplicatorConnectionStatus**。
* **後續步驟**：檢查錯誤訊息，並確定已正確設定遠端複寫器位址。 例如，如果遠端複寫器是以 "localhost" 接聽位址開啟的，就無法從外部連線。 如果位址看來正確無誤，請檢查主要節點與遠端位址之間的連線，以尋找任何可能的網路問題。

### <a name="replication-queue-full"></a>複寫佇列已滿
**System.Replicator** 會在複寫佇列已滿時回報警告。 在主要資料庫上，複寫佇列通常會因為一或多個次要複本太慢認可作業而排滿。 在次要複本上，這通常是因為服務緩慢而無法套用作業所造成。 當佇列有空間時，警告就會被清除。

* **SourceId**：System.Replicator
* **Property**：**PrimaryReplicationQueueStatus** 或 **SecondaryReplicationQueueStatus** (根據複本角色而定)。

### <a name="slow-naming-operations"></a>緩慢的命名作業
**System.NamingService** 會在命名作業執行時間太長而無法接受時，報告其主要複本的健康情況。 命名作業的範例為 [CreateServiceAsync](https://docs.microsoft.com/dotnet/api/system.fabric.fabricclient.servicemanagementclient.createserviceasync) 或 [DeleteServiceAsync](https://docs.microsoft.com/dotnet/api/system.fabric.fabricclient.servicemanagementclient.deleteserviceasync)。 您可以在 FabricClient 下找到更多方法。 例如，可以在[服務管理方法](https://docs.microsoft.com/dotnet/api/system.fabric.fabricclient.servicemanagementclient)或[屬性管理方法](https://docs.microsoft.com/dotnet/api/system.fabric.fabricclient.propertymanagementclient)底下找到這些方法。

> [!NOTE]
> 命名服務會將服務名稱解析為叢集中的位置。 使用者可以使用它來管理服務名稱和屬性。 它是 Service Fabric 資料分割保存的服務。 其中一個分割區代表*授權擁有者*，內含所有 Service Fabric 名稱和服務的中繼資料。 Service Fabric 名稱會對應至不同的分割區 (稱為*名稱擁有者*分割區)，讓服務可以擴充。 深入了解[命名服務](service-fabric-architecture.md)。
> 
> 

命名作業所花費的時間超出預期時，在負責處理該作業的命名服務分割區的主要複本上，該作業會標幟警告報告。 如果作業順利完成，就會清除警告。 如果作業完成但發生錯誤，健全狀況報告會包含錯誤的詳細資訊。

* **SourceId**：System.NamingService
* **Property**：開頭為前置詞 "**Duration_**"，並可識別緩慢作業和套用該作業的 Service Fabric 名稱。 例如，如果在名稱 **fabric:/MyApp/MyService** 建立服務花太多時間，其屬性就是 **Duration_AOCreateService.fabric:/MyApp/MyService**。 "AO" 會針對這個名稱和作業，指向命名分割區的角色。
* **後續步驟**︰查看命名作業失敗的原因。 每個作業都可能有不同的根本原因。 例如，刪除服務可能已停滯。 由於服務程式碼中的使用者錯誤 (Bug) 造成節點上的應用程式主機持續當機，而使得服務停滯。

以下範例顯示一個建立服務作業。 作業所花費的時間超過設定的期間。 "AO" 重試，並將工作傳送到 "NO"。 "NO" 完成最後一個作業但逾時。 在此情況下，"AO" 和 "NO" 角色有相同的主要複本。

```PowerShell
PartitionId           : 00000000-0000-0000-0000-000000001000
ReplicaId             : 131064359253133577
AggregatedHealthState : Warning
UnhealthyEvaluations  :
                        Unhealthy event: SourceId='System.NamingService', Property='Duration_AOCreateService.fabric:/MyApp/MyService', HealthState='Warning', ConsiderWarningAsError=false.

HealthEvents          :
                        SourceId              : System.RA
                        Property              : State
                        HealthState           : Ok
                        SequenceNumber        : 131064359308715535
                        SentAt                : 4/29/2016 8:38:50 PM
                        ReceivedAt            : 4/29/2016 8:39:08 PM
                        TTL                   : Infinite
                        Description           : Replica has been created.
                        RemoveWhenExpired     : False
                        IsExpired             : False
                        Transitions           : Error->Ok = 4/29/2016 8:39:08 PM, LastWarning = 1/1/0001 12:00:00 AM

                        SourceId              : System.NamingService
                        Property              : Duration_AOCreateService.fabric:/MyApp/MyService
                        HealthState           : Warning
                        SequenceNumber        : 131064359526778775
                        SentAt                : 4/29/2016 8:39:12 PM
                        ReceivedAt            : 4/29/2016 8:39:38 PM
                        TTL                   : 00:05:00
                        Description           : The AOCreateService started at 2016-04-29 20:39:08.677 is taking longer than 30.000.
                        RemoveWhenExpired     : True
                        IsExpired             : False
                        Transitions           : Error->Warning = 4/29/2016 8:39:38 PM, LastOk = 1/1/0001 12:00:00 AM

                        SourceId              : System.NamingService
                        Property              : Duration_NOCreateService.fabric:/MyApp/MyService
                        HealthState           : Warning
                        SequenceNumber        : 131064360657607311
                        SentAt                : 4/29/2016 8:41:05 PM
                        ReceivedAt            : 4/29/2016 8:41:08 PM
                        TTL                   : 00:00:15
                        Description           : The NOCreateService started at 2016-04-29 20:39:08.689 completed with FABRIC_E_TIMEOUT in more than 30.000.
                        RemoveWhenExpired     : True
                        IsExpired             : False
                        Transitions           : Error->Warning = 4/29/2016 8:39:38 PM, LastOk = 1/1/0001 12:00:00 AM
```

## <a name="deployedapplication-system-health-reports"></a>DeployedApplication 系統健康狀態報告
**System.Hosting** 是已部署實體的授權單位。

### <a name="activation"></a>啟用
當應用程式在節點上成功啟用時，System.Hosting 會回報為 OK。 否則，它會報告錯誤。

* **SourceId**：System.Hosting
* **Property**：**Activation**，包括首度發行版本。
* **後續步驟**：如果應用程式的狀況不佳，請調查啟用失敗的原因。

以下範例顯示成功啟用的情況：

```PowerShell
PS C:\> Get-ServiceFabricDeployedApplicationHealth -NodeName _Node_1 -ApplicationName fabric:/WordCount -ExcludeHealthStatistics

ApplicationName                    : fabric:/WordCount
NodeName                           : _Node_1
AggregatedHealthState              : Ok
DeployedServicePackageHealthStates : 
                                     ServiceManifestName   : WordCountServicePkg
                                     ServicePackageActivationId : 
                                     NodeName              : _Node_1
                                     AggregatedHealthState : Ok
                                     
HealthEvents                       : 
                                     SourceId              : System.Hosting
                                     Property              : Activation
                                     HealthState           : Ok
                                     SequenceNumber        : 131445249083836329
                                     SentAt                : 7/14/2017 4:55:08 PM
                                     ReceivedAt            : 7/14/2017 4:55:14 PM
                                     TTL                   : Infinite
                                     Description           : The application was activated successfully.
                                     RemoveWhenExpired     : False
                                     IsExpired             : False
                                     Transitions           : Error->Ok = 7/14/2017 4:55:14 PM, LastWarning = 1/1/0001 12:00:00 AM
```

### <a name="download"></a>下載
如果應用程式套件下載失敗，System.Hosting 會回報錯誤。

* **SourceId**：System.Hosting
* **Property**：**Download**，包括首度發行版本。
* **後續步驟**：調查節點上下載失敗的原因。

## <a name="deployedservicepackage-system-health-reports"></a>DeployedServicePackage 系統健康狀態報告
**System.Hosting** 是已部署實體的授權單位。

### <a name="service-package-activation"></a>服務封裝啟用
如果節點上的服務封裝成功啟用，System.Hosting 會回報為 OK。 否則，它會報告錯誤。

* **SourceId**：System.Hosting
* **Property**：Activation。
* **後續步驟**：調查啟用失敗的原因。

### <a name="code-package-activation"></a>程式碼封裝啟用
如果啟用成功，System.Hosting 會針對每個程式碼套件回報為 OK。 如果啟用失敗，它會依設定回報警告。 如果 **CodePackage** 無法啟用，或者因為錯誤數超過 **CodePackageHealthErrorThreshold** 的設定而結束，則 Hosting 會回報錯誤。 如果服務封裝包含多個程式碼封裝，就會針對每個封裝產生啟用報告。

* **SourceId**：System.Hosting
* **Property**：使用前置詞 **CodePackageActivation**，並以 CodePackageActivation:CodePackageName:SetupEntryPoint/EntryPoint 的形式包含程式碼套件的名稱和進入點。 例如，**CodePackageActivation:Code:SetupEntryPoint**。

### <a name="service-type-registration"></a>服務類型註冊
如果已經成功註冊服務類型，則 System.Hosting 會回報為 OK。 如果註冊未及時完成 (使用 **ServiceTypeRegistrationTimeout** 來設定)，則會回報錯誤。 如果執行階段已關閉，則會從節點取消註冊服務類型，且主機會回報警告。

* **SourceId**：System.Hosting
* **Property**：使用前置詞 **ServiceTypeRegistration**，並包含服務類型名稱。 例如，**ServiceTypeRegistration:FileStoreServiceType**。

以下顯示顯示狀況良好的已部署服務封裝：

```PowerShell
PS C:\> Get-ServiceFabricDeployedServicePackageHealth -NodeName _Node_1 -ApplicationName fabric:/WordCount -ServiceManifestName WordCountServicePkg


ApplicationName            : fabric:/WordCount
ServiceManifestName        : WordCountServicePkg
ServicePackageActivationId : 
NodeName                   : _Node_1
AggregatedHealthState      : Ok
HealthEvents               : 
                             SourceId              : System.Hosting
                             Property              : Activation
                             HealthState           : Ok
                             SequenceNumber        : 131445249084026346
                             SentAt                : 7/14/2017 4:55:08 PM
                             ReceivedAt            : 7/14/2017 4:55:14 PM
                             TTL                   : Infinite
                             Description           : The ServicePackage was activated successfully.
                             RemoveWhenExpired     : False
                             IsExpired             : False
                             Transitions           : Error->Ok = 7/14/2017 4:55:14 PM, LastWarning = 1/1/0001 12:00:00 AM
                             
                             SourceId              : System.Hosting
                             Property              : CodePackageActivation:Code:EntryPoint
                             HealthState           : Ok
                             SequenceNumber        : 131445249084306362
                             SentAt                : 7/14/2017 4:55:08 PM
                             ReceivedAt            : 7/14/2017 4:55:14 PM
                             TTL                   : Infinite
                             Description           : The CodePackage was activated successfully.
                             RemoveWhenExpired     : False
                             IsExpired             : False
                             Transitions           : Error->Ok = 7/14/2017 4:55:14 PM, LastWarning = 1/1/0001 12:00:00 AM
                             
                             SourceId              : System.Hosting
                             Property              : ServiceTypeRegistration:WordCountServiceType
                             HealthState           : Ok
                             SequenceNumber        : 131445249088096842
                             SentAt                : 7/14/2017 4:55:08 PM
                             ReceivedAt            : 7/14/2017 4:55:14 PM
                             TTL                   : Infinite
                             Description           : The ServiceType was registered successfully.
                             RemoveWhenExpired     : False
                             IsExpired             : False
                             Transitions           : Error->Ok = 7/14/2017 4:55:14 PM, LastWarning = 1/1/0001 12:00:00 AM
```

### <a name="download"></a>下載
如果服務套件下載失敗，System.Hosting 會回報錯誤。

* **SourceId**：System.Hosting
* **Property**：**Download**，包括首度發行版本。
* **後續步驟**：調查節點上下載失敗的原因。

### <a name="upgrade-validation"></a>升級驗證
如果在升級期間驗證失敗，或是節點的升級失敗，System.Hosting 會回報錯誤。

* **SourceId**：System.Hosting
* **Property**：使用前置詞 **FabricUpgradeValidation**，並包含升級版本。
* **Description**：指出發生的錯誤。

### <a name="undefined-node-capacity-for-resource-governance-metrics"></a>未針對資源控管計量定義的節點容量
如果叢集資訊清單中未定義節點容量且已關閉自動偵測的組態，則 System.Hosting 會回報警告。 每當使用[資源控管](service-fabric-resource-governance.md)的服務套件在指定的節點上註冊時，Service Fabric 將會引發健康情況警告。

* **SourceId**：System.Hosting
* **Property**：**ResourceGovernance**。
* **後續步驟**：克服此問題的較佳方式是變更叢集資訊清單，以啟用可用資源的自動偵測。 另一種方式是使用為這些計量正確指定的節點容量來更新叢集資訊清單。

## <a name="next-steps"></a>後續步驟
* [檢視 Service Fabric 健康狀態報告](service-fabric-view-entities-aggregated-health.md)

* [如何回報和檢查服務健全狀況](service-fabric-diagnostics-how-to-report-and-check-service-health.md)

* [在本機上監視及診斷服務](service-fabric-diagnostics-how-to-monitor-and-diagnose-services-locally.md)

* [Service Fabric 應用程式升級](service-fabric-application-upgrade.md)

