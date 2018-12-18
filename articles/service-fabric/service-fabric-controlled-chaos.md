---
title: 在 Service Fabric 叢集中引發混亂 | Microsoft Docs
description: 使用錯誤注射與叢集分析服務的 API 來管理叢集中的混亂。
services: service-fabric
documentationcenter: .net
author: motanv
manager: anmola
editor: motanv
ms.assetid: 2bd13443-3478-4382-9a5a-1f6c6b32bfc9
ms.service: service-fabric
ms.devlang: dotnet
ms.topic: conceptual
ms.tgt_pltfrm: NA
ms.workload: NA
ms.date: 02/05/2018
ms.author: motanv
ms.openlocfilehash: 3774ae0572a87129fb089064cec9bb7957a98c22
ms.sourcegitcommit: 7827d434ae8e904af9b573fb7c4f4799137f9d9b
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/18/2018
ms.locfileid: "39113234"
---
# <a name="induce-controlled-chaos-in-service-fabric-clusters"></a>在 Service Fabric 叢集中引發受控制的混亂
雲端基礎結構之類的大型分散式系統本身並不可靠。 Azure Service Fabric 可讓開發人員在不可靠的基礎結構之上撰寫可靠的分散式服務。 若要在不可靠的基礎結構之上撰寫健全的分散式服務，開發人員需要能夠測試其服務的穩定性，同時不可靠的基礎結構會因錯誤而經歷複雜的狀態轉換。

[錯誤插入與叢集分析服務](https://docs.microsoft.com/azure/service-fabric/service-fabric-testability-overview) (亦稱為「錯誤分析服務」) 讓開發人員能夠引發錯誤來測試其服務。 這些針對性模擬錯誤，例如[重新啟動分割區](https://docs.microsoft.com/powershell/module/servicefabric/start-servicefabricpartitionrestart?view=azureservicefabricps)，可幫助您練習最常見的狀態轉換。 但針對性模擬錯誤會因為定義而有偏差，因此可能會遺漏只會出現在難以預測、冗長且複雜的狀態轉換順序中的問題。 如需無偏差測試，您可以使用混亂。

混亂會以很長的時間在整個叢集上模擬定期、交錯的錯誤 (包括非失誤性和失誤性錯誤)。 非失誤性錯誤是由一組 Service Fabric API 呼叫所組成。舉例來說，重新啟動複本錯誤是非失誤性錯誤，因為這是複本上由開啟所接續的關閉。 移除複本、移動主要複本，以及移動次要複本是由混亂所運用的其他非失誤性錯誤。 失誤性錯誤為處理程序結束，例如重新啟動節點和重新啟動程式碼封裝。 

設定混亂的比率和錯誤類型後，即可透過 C#、Powershell 或 REST API 啟動混亂，以開始在叢集和您的服務中產生錯誤。 您可以將混亂設定為執行一段指定的時間 (例如一小時)，混亂在那段時間之後會自動停止，您也可以隨時呼叫 StopChaos API (C#、Powershell 或 REST) 來停止它。

> [!NOTE]
> 目前來說，混亂只會引發安全的錯誤，這表示如果沒有外部錯誤，絕不會發生仲裁遺失或資料遺失。
>

混亂執行時，會產生不同事件來擷取目前執行的狀態。 例如，ExecutingFaultsEvent 包含混亂已決定正在該反覆運算中執行的所有錯誤。 ValidationFailedEvent 包含在驗證叢集期間所發現驗證失敗 (健康情況或穩定性問題) 的詳細資料。 您可以叫用 GetChaosReport API (C#、Powershell 或 REST) 以取得混亂執行的報告。 這些事件保存在[可靠的字典](https://docs.microsoft.com/azure/service-fabric/service-fabric-reliable-services-reliable-collections)中，其中有由兩個組態決定的截斷原則：MaxStoredChaosEventCount (預設值為 25000) 及 StoredActionCleanupIntervalInSeconds (預設值為 3600)。 每個 StoredActionCleanupIntervalInSeconds 混亂檢查及最新 MaxStoredChaosEventCount 事件以外的所有事件皆會自可靠字典中清除。

## <a name="faults-induced-in-chaos"></a>混亂中引發的錯誤
混亂會在整個 Service Fabric 叢集中產生錯誤，並將在幾個月或幾年內看到的錯誤壓縮成幾小時。 交錯錯誤和高錯誤率的組合，會尋找可能會在其他情形下遺漏的極端狀況。 這個混亂練習可以大幅提升服務的程式碼品質。

混亂會引發下列類別的錯誤︰

* 重新啟動節點
* 重新啟動已部署的程式碼封裝
* 移除複本
* 重新啟動複本
* 移動主要複本 (可設定)
* 移動次要複本 (可設定)

混亂會多次反覆執行。 每次反覆運算都包含指定期間的錯誤和叢集驗證。 您可以設定讓叢集穩定和驗證成功的所需時間。 如果在叢集驗證中發現失敗，則混亂會產生並保留一個 ValidationFailedEvent，包含 UTC 時間戳記與失敗詳細資料。 例如，考慮一個設為執行 1 小時且最多有 3 個並行錯誤的混亂執行個體。 混亂會引發三個錯誤，然後驗證叢集健康狀態。 它會重複執行上一個步驟，直到透過 StopChaosAsync API 或經過一小時後就會明確停止。 如果叢集在任何反覆運算中變成健康情況不佳 (也就是在傳入的 MaxClusterStabilizationTimeout 內未變成穩定或未變成狀況良好)，則混亂會產生 ValidationFailedEvent。 此事件表示發生了錯誤，且可能需要進一步調查。

若要取得混亂引發的錯誤，您可以使用 GetChaosReport API (Powershell、C# 或 REST)。 API 會根據傳入接續權杖或傳入的時間範圍取得混亂報告的下個區段。 您可以指定 ContinuationToken 取得混亂報告的下個區段，或者您可以透過 StartTimeUtc 與 EndTimeUtc 指定時間範圍，但您無法在同一個呼叫中同時指定 ContinuationToken 與時間範圍。 當混亂事件超過 100 個時，系統會分區段傳回混亂報告，每個區段中包含的混亂事件不超過 100 個。

## <a name="important-configuration-options"></a>重要的組態選項
* **TimeToRun**：混亂在成功完成前的總執行時間。 您可以在混亂執行 TimeToRun 這段時間之前透過 StopChaos API 停止混亂。

* **MaxClusterStabilizationTimeout**：在產生 ValidationFailedEvent 前等候叢集健康情況變為良好的時間上限。 這段等候時間是為了減少叢集在復原時所承擔的負載。 執行的檢查為：
  * 叢集健康狀態是否正常
  * 服務健康狀態是否正常
  * 服務分割區是否達到目標複本集大小
  * 沒有 InBuild 複本存在
* **MaxConcurrentFaults**：每個反覆運算中引發的最大並行錯誤數。 數字愈大，混亂愈積極，叢集經歷的容錯移轉與狀態轉換也更複雜。 

> [!NOTE]
> 無論 *MaxConcurrentFaults* 值多大，混亂都能保證在缺少外部錯誤的狀況下，不會有仲裁遺失或資料遺失。
>

* **EnableMoveReplicaFaults**：啟用或停用造成主要或次要複本移動的錯誤。 預設會啟用這些錯誤。
* **WaitTimeBetweenIterations**︰反覆運算之間要等候的時間量。 亦即，在已執行一輪的錯誤且已完成對應的叢集健康情況驗證後，混亂將暫停的時間。 值愈大，平均錯誤插入率愈低。
* **WaitTimeBetweenFaults**︰單一反覆運算中兩個連續錯誤之間的等候時間長度。 值愈大，錯誤的並行程度 (或錯誤之間的重疊度) 愈低。
* **ClusterHealthPolicy**︰叢集健康情況原則用於驗證混亂反覆運算之間的叢集健康情況。 如果叢集健康情況發生錯誤，或如果在錯誤執行期間發生未預期的例外狀況，則混亂會先等待 30 分鐘，再執行下一個健康情況檢查，讓叢集有時間復原。
* **內容**：(string, string) 類型索引鍵-值組的集合。 此對應可用於記錄混亂執行的相關資訊。 此類組合不能超過 100 個，且每個字串 (索引鍵或值) 最多為 4095 個字元長。 此對應由混亂執行的起始者設定，以選擇性地儲存特定執行的相關內容。
* **ChaosTargetFilter**：此篩選可用來將混亂錯誤的目標設定為僅針對特定節點類型或特定應用程式執行個體。 如果未使用 ChaosTargetFilter，混亂會針對所有叢集實體引發錯誤。 如果使用 ChaosTargetFilter，則混亂只會針對符合 ChaosTargetFilter 規格的實體引發錯誤。 NodeTypeInclusionList 和 ApplicationInclusionList 只允許集合聯集語意。 換句話說，您不能指定 NodeTypeInclusionList 和 ApplicationInclusionList 的交集。 例如，您不能指定「只有在此應用程式位於該節點類型上時才對它引發錯誤」。 若 NodeTypeInclusionList 或 ApplicationInclusionList 中包含某個實體，則該實體就不能使用 ChaosTargetFilter 排除。 即使 applicationX 並未出現在 ApplicationInclusionList 中，但在某些混亂反覆項目中也能對 applicationX 引發錯誤，因為它正好位於包括在 NodeTypeInclusionList 中的 nodeTypeY 的節點上。 如果 NodeTypeInclusionList 和 ApplicationInclusionList 都是 Null 或空白，則會擲回 ArgumentException。
    * **NodeTypeInclusionList**：要包含在混亂錯誤中的節點類型清單。 針對這些節點類型的節點，會啟用所有錯誤類型 (重新啟動節點、重新啟動程式碼封裝、移除複本、重新啟動複本、移動主要複本，以及移動次要複本)。 如果某個節點類型 (假設為 NodeTypeX) 未出現在 NodeTypeInclusionList 中，則系統將永遠不會針對 NodeTypeX 的節點啟用節點層級錯誤 (例如 NodeRestart)，但如果 ApplicationInclusionList 中的某個應用程式正好位於 NodeTypeX 的節點上，則仍會針對 NodeTypeX 啟用程式碼封裝和複本錯誤。 您最多可在此清單中包含 100 個節點類型名稱，若要增加此數目，則必須針對 MaxNumberOfNodeTypesInChaosTargetFilter 設定進行設定升級。
    * **ApplicationInclusionList**：要包含在混亂錯誤中的應用程式 URI 清單。 屬於這些應用程式之服務的所有複本，都適用於由混亂所引發的複本錯誤 (重新啟動複本、移除複本、移動主要複本和移動次要複本)。 只有在程式碼封裝只裝載這些應用程式的複本時，混亂才可以將程式碼封裝重新啟動。 如果某個應用程式未出現在此清單中，系統仍然可能在某些混亂反覆項目中針對它引發錯誤，前提是該應用程式位於某個包含在 NodeTypeInclusionList 中之節點類型的節點上。 不過，如果 applicationX 因位置限制而繫結至 nodeTypeY，且 applicationX 未出現在 ApplicationInclusionList，而且 nodeTypeY 未出現在 NodeTypeInclusionList 中，則系統將永遠不會對 applicationX 引發錯誤。 您最多可在此清單中包含 1000 個應用程式名稱，若要增加此數目，則必須針對 MaxNumberOfApplicationsInChaosTargetFilter 設定進行設定升級。

## <a name="how-to-run-chaos"></a>如何執行混亂

```csharp
using System;
using System.Collections.Generic;
using System.Threading.Tasks;
using System.Fabric;

using System.Diagnostics;
using System.Fabric.Chaos.DataStructures;

class Program
{
    private class ChaosEventComparer : IEqualityComparer<ChaosEvent>
    {
        public bool Equals(ChaosEvent x, ChaosEvent y)
        {
            return x.TimeStampUtc.Equals(y.TimeStampUtc);
        }
        public int GetHashCode(ChaosEvent obj)
        {
            return obj.TimeStampUtc.GetHashCode();
        }
    }

    static void Main(string[] args)
    {
        var clusterConnectionString = "localhost:19000";
        using (var client = new FabricClient(clusterConnectionString))
        {
            var startTimeUtc = DateTime.UtcNow;

            // The maximum amount of time to wait for all cluster entities to become stable and healthy. 
            // Chaos executes in iterations and at the start of each iteration it validates the health of cluster entities. 
            // During validation if a cluster entity is not stable and healthy within MaxClusterStabilizationTimeoutInSeconds, Chaos generates a validation failed event.
            var maxClusterStabilizationTimeout = TimeSpan.FromSeconds(30.0);

            var timeToRun = TimeSpan.FromMinutes(60.0);

            // MaxConcurrentFaults is the maximum number of concurrent faults induced per iteration. 
            // Chaos executes in iterations and two consecutive iterations are separated by a validation phase. 
            // The higher the concurrency, the more aggressive the injection of faults -- inducing more complex series of states to uncover bugs. 
            // The recommendation is to start with a value of 2 or 3 and to exercise caution while moving up.
            var maxConcurrentFaults = 3;

            // Describes a map, which is a collection of (string, string) type key-value pairs. The map can be used to record information about
            // the Chaos run. There cannot be more than 100 such pairs and each string (key or value) can be at most 4095 characters long.
            // This map is set by the starter of the Chaos run to optionally store the context about the specific run.
            var startContext = new Dictionary<string, string>{{"ReasonForStart", "Testing"}};

            // Time-separation (in seconds) between two consecutive iterations of Chaos. The larger the value, the lower the fault injection rate.
            var waitTimeBetweenIterations = TimeSpan.FromSeconds(10);

            // Wait time (in seconds) between consecutive faults within a single iteration. 
            // The larger the value, the lower the overlapping between faults and the simpler the sequence of state transitions that the cluster goes through. 
            // The recommendation is to start with a value between 1 and 5 and exercise caution while moving up.
            var waitTimeBetweenFaults = TimeSpan.Zero;

            // Passed-in cluster health policy is used to validate health of the cluster in between Chaos iterations. 
            var clusterHealthPolicy = new ClusterHealthPolicy
            {
                ConsiderWarningAsError = false,
                MaxPercentUnhealthyApplications = 100,
                MaxPercentUnhealthyNodes = 100
            };

            // All types of faults, restart node, restart code package, restart replica, move primary replica, and move secondary replica will happen
            // for nodes of type 'FrontEndType'
            var nodetypeInclusionList = new List<string> { "FrontEndType"};

            // In addition to the faults included by nodetypeInclusionList, 
            // restart code package, restart replica, move primary replica, move secondary replica faults will happen for 'fabric:/TestApp2'
            // even if a replica or code package from 'fabric:/TestApp2' is residing on a node which is not of type included in nodeypeInclusionList.
            var applicationInclusionList = new List<string> { "fabric:/TestApp2" };

            // List of cluster entities to target for Chaos faults.
            var chaosTargetFilter = new ChaosTargetFilter
            {
                NodeTypeInclusionList = nodetypeInclusionList,
                ApplicationInclusionList = applicationInclusionList
            };

            var parameters = new ChaosParameters(
                maxClusterStabilizationTimeout,
                maxConcurrentFaults,
                true, /* EnableMoveReplicaFault */
                timeToRun,
                startContext,
                waitTimeBetweenIterations,
                waitTimeBetweenFaults,
                clusterHealthPolicy) {ChaosTargetFilter = chaosTargetFilter};

            try
            {
                client.TestManager.StartChaosAsync(parameters).GetAwaiter().GetResult();
            }
            catch (FabricChaosAlreadyRunningException)
            {
                Console.WriteLine("An instance of Chaos is already running in the cluster.");
            }

            var filter = new ChaosReportFilter(startTimeUtc, DateTime.MaxValue);

            var eventSet = new HashSet<ChaosEvent>(new ChaosEventComparer());

            string continuationToken = null;

            while (true)
            {
                ChaosReport report;
                try
                {
                    report = string.IsNullOrEmpty(continuationToken)
                        ? client.TestManager.GetChaosReportAsync(filter).GetAwaiter().GetResult()
                        : client.TestManager.GetChaosReportAsync(continuationToken).GetAwaiter().GetResult();
                }
                catch (Exception e)
                {
                    if (e is FabricTransientException)
                    {
                        Console.WriteLine("A transient exception happened: '{0}'", e);
                    }
                    else if(e is TimeoutException)
                    {
                        Console.WriteLine("A timeout exception happened: '{0}'", e);
                    }
                    else
                    {
                        throw;
                    }

                    Task.Delay(TimeSpan.FromSeconds(1.0)).GetAwaiter().GetResult();
                    continue;
                }

                continuationToken = report.ContinuationToken;

                foreach (var chaosEvent in report.History)
                {
                    if (eventSet.Add(chaosEvent))
                    {
                        Console.WriteLine(chaosEvent);
                    }
                }

                // When Chaos stops, a StoppedEvent is created.
                // If a StoppedEvent is found, exit the loop.
                var lastEvent = report.History.LastOrDefault();

                if (lastEvent is StoppedEvent)
                {
                    break;
                }

                Task.Delay(TimeSpan.FromSeconds(1.0)).GetAwaiter().GetResult();
            }
        }
    }
}
```

```powershell
$clusterConnectionString = "localhost:19000"
$timeToRunMinute = 60

# The maximum amount of time to wait for all cluster entities to become stable and healthy. 
# Chaos executes in iterations and at the start of each iteration it validates the health of cluster entities. 
# During validation if a cluster entity is not stable and healthy within MaxClusterStabilizationTimeoutInSeconds, Chaos generates a validation failed event.
$maxClusterStabilizationTimeSecs = 30

# MaxConcurrentFaults is the maximum number of concurrent faults induced per iteration. 
# Chaos executes in iterations and two consecutive iterations are separated by a validation phase. 
# The higher the concurrency, the more aggressive the injection of faults -- inducing more complex series of states to uncover bugs. 
# The recommendation is to start with a value of 2 or 3 and to exercise caution while moving up.
$maxConcurrentFaults = 3

# Time-separation (in seconds) between two consecutive iterations of Chaos. The larger the value, the lower the fault injection rate.
$waitTimeBetweenIterationsSec = 10

# Wait time (in seconds) between consecutive faults within a single iteration. 
# The larger the value, the lower the overlapping between faults and the simpler the sequence of state transitions that the cluster goes through. 
# The recommendation is to start with a value between 1 and 5 and exercise caution while moving up.
$waitTimeBetweenFaultsSec = 0

# Passed-in cluster health policy is used to validate health of the cluster in between Chaos iterations. 
$clusterHealthPolicy = new-object -TypeName System.Fabric.Health.ClusterHealthPolicy
$clusterHealthPolicy.MaxPercentUnhealthyNodes = 100
$clusterHealthPolicy.MaxPercentUnhealthyApplications = 100
$clusterHealthPolicy.ConsiderWarningAsError = $False

# Describes a map, which is a collection of (string, string) type key-value pairs. The map can be used to record information about
# the Chaos run. There cannot be more than 100 such pairs and each string (key or value) can be at most 4095 characters long.
# This map is set by the starter of the Chaos run to optionally store the context about the specific run.
$context = @{"ReasonForStart" = "Testing"}

#List of cluster entities to target for Chaos faults.
$chaosTargetFilter = new-object -TypeName System.Fabric.Chaos.DataStructures.ChaosTargetFilter
$chaosTargetFilter.NodeTypeInclusionList = new-object -TypeName "System.Collections.Generic.List[String]"

# All types of faults, restart node, restart code package, restart replica, move primary replica, and move secondary replica will happen
# for nodes of type 'FrontEndType'
$chaosTargetFilter.NodeTypeInclusionList.AddRange( [string[]]@("FrontEndType") )
$chaosTargetFilter.ApplicationInclusionList = new-object -TypeName "System.Collections.Generic.List[String]"

# In addition to the faults included by nodetypeInclusionList, 
# restart code package, restart replica, move primary replica, move secondary replica faults will happen for 'fabric:/TestApp2'
# even if a replica or code package from 'fabric:/TestApp2' is residing on a node which is not of type included in nodeypeInclusionList.
$chaosTargetFilter.ApplicationInclusionList.Add("fabric:/TestApp2")

Connect-ServiceFabricCluster $clusterConnectionString

$events = @{}
$now = [System.DateTime]::UtcNow

Start-ServiceFabricChaos -TimeToRunMinute $timeToRunMinute -MaxConcurrentFaults $maxConcurrentFaults -MaxClusterStabilizationTimeoutSec $maxClusterStabilizationTimeSecs -EnableMoveReplicaFaults -WaitTimeBetweenIterationsSec $waitTimeBetweenIterationsSec -WaitTimeBetweenFaultsSec $waitTimeBetweenFaultsSec -ClusterHealthPolicy $clusterHealthPolicy -ChaosTargetFilter $chaosTargetFilter

while($true)
{
    $stopped = $false
    $report = Get-ServiceFabricChaosReport -StartTimeUtc $now -EndTimeUtc ([System.DateTime]::MaxValue)

    foreach ($e in $report.History) {

        if(-Not ($events.Contains($e.TimeStampUtc.Ticks)))
        {
            $events.Add($e.TimeStampUtc.Ticks, $e)
            if($e -is [System.Fabric.Chaos.DataStructures.ValidationFailedEvent])
            {
                Write-Host -BackgroundColor White -ForegroundColor Red $e
            }
            else
            {
                Write-Host $e
                # When Chaos stops, a StoppedEvent is created.
                # If a StoppedEvent is found, exit the loop.
                if($e -is [System.Fabric.Chaos.DataStructures.StoppedEvent])
                {
                    return
                }
            }
        }
    }

    Start-Sleep -Seconds 1
}
```
