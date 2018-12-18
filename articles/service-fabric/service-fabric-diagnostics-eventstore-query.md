---
title: 在 Azure Service Fabric 叢集中使用 EventStore API 查詢叢集事件 | Microsoft Docs
description: 了解如何使用 Azure Service Fabric EventStore API 來查詢平台事件
services: service-fabric
documentationcenter: .net
author: dkkapur
manager: timlt
editor: ''
ms.assetid: ''
ms.service: service-fabric
ms.devlang: dotNet
ms.topic: conceptual
ms.tgt_pltfrm: NA
ms.workload: NA
ms.date: 04/25/2018
ms.author: dekapur
ms.openlocfilehash: 5c184841602f269555ce2196ef660faba14dbf8a
ms.sourcegitcommit: eb75f177fc59d90b1b667afcfe64ac51936e2638
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 05/16/2018
ms.locfileid: "34204605"
---
# <a name="query-eventstore-apis-for-cluster-events"></a>針對叢集事件查詢 EventStore API

本文內容涵蓋如何查詢於 Service Fabric 6.2 版和更新版本中提供的 EventStore API。如果您想要深入了解 EventStore 服務，請參閱 [EventStore 服務概觀](service-fabric-diagnostics-eventstore.md)。 目前，EventStore 服務只能存取過去 7 天 (這是以您叢集的診斷資料保留原則為基礎) 的資料。

>[!NOTE]
>截至 Service Fabric 6.2 版為止， EventStore API 目前為預覽版本，僅適用於在 Azure 上執行的 Windows 叢集。 我們正在將這個功能移植到 Linux 和我們的獨立叢集。

EventStore API 可以直接透過 REST 端點存取，或以程式設計方式存取。 視查詢而定，需要有幾個參數才能收集到正確資料。 這些參數通常包括：
* `api-version`：您使用的 EventStore API 版本
* `StartTimeUtc`：定義想查詢期間的開始時間
* `EndTimeUtc`：查詢期間的結束時間

除了這些參數之外，還有幾個選擇性參數可用，例如：
* `timeout`：覆寫用於執行要求作業的預設 60 秒逾時
* `eventstypesfilter`：這可以提供篩選特定事件類型的選項
* `ExcludeAnalysisEvents`：不要傳回「分析」事件。 根據預設，EventStore 查詢會在可能的情況下傳回「分析」事件；分析事件為資訊較豐富的操作通道事件，其中包含比一般 Service Fabric 事件更廣泛的其他內容或資訊，並提供更深入的資訊。
* `SkipCorrelationLookup`：不查詢叢集中可能的相關聯事件。 根據預設，EventStore 將會嘗試在跨叢集事件之間建立關聯，並盡可能將您的事件連結在一起。 

您可以對叢集中的每個實體查詢事件。 您也可以針對某一類型的所有實體查詢事件。 例如，您可以查詢特定節點的事件，或叢集中所有節點的事件。 目前您可以查詢事件的實體集合為 (包含查詢的結構方式)：
* 叢集：`/EventsStore/Cluster/Events`
* 多個節點：`/EventsStore/Nodes/Events`
* 單一節點：`/EventsStore/Nodes/<NodeName>/$/Events`
* 多個應用程式：`/EventsStore/Applications/Events`
* 單一應用程式：`/EventsStore/Applications/<AppName>/$/Events`
* 多個服務：`/EventsStore/Services/Events`
* 單一服務：`/EventsStore/Services/<ServiceName>/$/Events`
* 多個分割區：`/EventsStore/Partitions/Events`
* 單一分割區：`/EventsStore/Partitions/<PartitionID>/$/Events`
* 多個複本：`/EventsStore/Partitions/<PartitionID>/$/Replicas/Events`
* 單一複本：`/EventsStore/Partitions/<PartitionID>/$/Replicas/<ReplicaID>/$/Events`

>[!NOTE]
>參考應用程式或服務名稱時，查詢不需要包括 "fabric:/" 前置詞。 此外，如果您的應用程式或服務名稱包含 "/"，請將它變更為 "~"，查詢才能運作。 例如，如果您的應用程式顯示為 "fabric:/App1/FrontendApp"，應用程式特定查詢的結構就會是 `/EventsStore/Applications/App1~FrontendApp/$/Events`。
>此外，現今的服務健康情況報表會顯示在相對應的應用程式底下，因此您可以查詢正確應用程式實體的 `DeployedServiceHealthReportCreated` 事件。 

## <a name="query-the-eventstore-via-rest-api-endpoints"></a>透過 REST API 端點查詢 EventStore

您可以將 `GET` 要求設為 `<your cluster address>/EventsStore/<entity>/Events/`，以直接透過 REST 端點查詢 EventStore。

例如，若要查詢 `2018-04-03T18:00:00Z` 和 `2018-04-04T18:00:00Z` 之間的所有叢集事件，您的要求看起來會像這樣：

```
Method: GET 
URL: http://mycluster:19080/EventsStore/Cluster/Events?api-version=6.2-preview&StartTimeUtc=2018-04-03T18:00:00Z&EndTimeUtc=2018-04-04T18:00:00Z
```

這可能會在如果沒有事件可用的情況下傳回錯誤，或者如果查詢成功，您就會看到以 json 傳回的事件：

```json
Response: 200
Body:
[
  {
    "Kind": "ClusterUpgradeStart",
    "CurrentClusterVersion": "0.0.0.0:",
    "TargetClusterVersion": "6.2:1.0",
    "UpgradeType": "Rolling",
    "RollingUpgradeMode": "UnmonitoredAuto",
    "FailureAction": "Manual",
    "EventInstanceId": "090add3c-8f56-4d35-8d57-a855745b6064",
    "TimeStamp": "2018-04-03T20:18:59.4313064Z",
    "HasCorrelatedEvents": false
  },
  {
    "Kind": "ClusterUpgradeDomainComplete",
    "TargetClusterVersion": "6.2:1.0",
    "UpgradeState": "RollingForward",
    "UpgradeDomains": "(0 1 2)",
    "UpgradeDomainElapsedTimeInMs": "78.5288",
    "EventInstanceId": "090add3c-8f56-4d35-8d57-a855745b6064",
    "TimeStamp": "2018-04-03T20:19:59.5729953Z",
    "HasCorrelatedEvents": false
  },
  {
    "Kind": "ClusterUpgradeDomainComplete",
    "TargetClusterVersion": "6.2:1.0",
    "UpgradeState": "RollingForward",
    "UpgradeDomains": "(3 4)",
    "UpgradeDomainElapsedTimeInMs": "0",
    "EventInstanceId": "090add3c-8f56-4d35-8d57-a855745b6064",
    "TimeStamp": "2018-04-03T20:20:59.6271949Z",
    "HasCorrelatedEvents": false
  },
  {
    "Kind": "ClusterUpgradeComplete",
    "TargetClusterVersion": "6.2:1.0",
    "OverallUpgradeElapsedTimeInMs": "120196.5212",
    "EventInstanceId": "090add3c-8f56-4d35-8d57-a855745b6064",
    "TimeStamp": "2018-04-03T20:20:59.8134457Z",
    "HasCorrelatedEvents": false
  }
]
```

我們可以看到在 `2018-04-03T18:00:00Z` 和 `2018-04-04T18:00:00Z` 之間，此叢集在第一次啟動時便順利完成第一次升級：於 `"OverallUpgradeElapsedTimeInMs": "120196.5212"`的時間內，從 `"CurrentClusterVersion": "0.0.0.0:"` 升級為 `"TargetClusterVersion": "6.2:1.0"`。

## <a name="query-the-eventstore-programmatically"></a>以程式設計方式查詢 EventStore

您也能透過 [Service Fabric 用戶端程式庫](https://docs.microsoft.com/dotnet/api/overview/azure/service-fabric?view=azure-dotnet#client-library)，以程式設計方式查詢 EventStore。

在您設定好 Service Fabric 用戶端之後，就可以透過以下方式存取 EventStore 以查詢事件：` sfhttpClient.EventStore.<request>`

以下是一個透過 `GetClusterEventListAsync` 函式，要求 `2018-04-03T18:00:00Z` 和 `2018-04-04T18:00:00Z` 之間所有叢集事件的範例要求。

```csharp
var sfhttpClient = ServiceFabricClientFactory.Create(clusterUrl, settings);

var clstrEvents = sfhttpClient.EventsStore.GetClusterEventListAsync(
    "2018-04-03T18:00:00Z",
    "2018-04-04T18:00:00Z")
    .GetAwaiter()
    .GetResult()
    .ToList();
```

## <a name="sample-scenarios-and-queries"></a>範例案例和查詢

以下是幾個範例，說明您可以如何呼叫 Event Store REST API 以了解叢集狀態。

*叢集升級：*

若要查看您的叢集在上一週最後一次成功升級或嘗試升級的時間，您可以透過查詢 EventStore 中的 "ClusterUpgradeComplete" 事件，來查詢最近完成叢集升級的 API：`https://mycluster.cloudapp.azure.com:19080/EventsStore/Cluster/Events?api-version=6.2-preview&starttimeutc=2017-04-22T17:01:51Z&endtimeutc=2018-04-29T17:02:51Z&EventsTypesFilter=ClusterUpgradeComplete`

*叢集升級問題：*

同樣地，如果最近的叢集升級出現問題，您可以查詢叢集實體的所有事件。 您將會看到各種不同事件，包括升級的起始和升級成功完成的每個 UD。 您也將會看到復原開始時間點的事件和相對應的健康情況事件。 以下是用於完成此動作的查詢：`https://mycluster.cloudapp.azure.com:19080/EventsStore/Cluster/Events?api-version=6.2-preview&starttimeutc=2017-04-22T17:01:51Z&endtimeutc=2018-04-29T17:02:51Z`

*節點狀態變更：*

若要查看您節點狀態於過去幾天的變更，例如節點啟動或停止，或是被啟用或停用 (透過平台、混沌服務或使用者輸入)，請使用以下查詢：`https://mycluster.cloudapp.azure.com:19080/EventsStore/Nodes/Events?api-version=6.2-preview&starttimeutc=2017-04-22T17:01:51Z&endtimeutc=2018-04-29T17:02:51Z`

*應用程式事件：*

您也可以追蹤最近的應用程式部署和升級。 使用以下查詢來查看您叢集中的所有應用程式相關事件：`https://mycluster.cloudapp.azure.com:19080/EventsStore/Applications/Events?api-version=6.2-preview&starttimeutc=2017-04-22T17:01:51Z&endtimeutc=2018-04-29T17:02:51Z`

*應用程式的健康情況歷程記錄：*

除了單純查看應用程式生命週期事件之外，您也可以查看特定應用程式之健康情況的歷程記錄資料。 您可以透過指定要收集其資料的應用程式名稱來達到此目的。 使用此查詢來取得所有應用程式健康情況事件：`https://mycluster.cloudapp.azure.com:19080/EventsStore/Applications/myApp/$/Events?api-version=6.2-preview&starttimeutc=2018-03-24T17:01:51Z&endtimeutc=2018-03-29T17:02:51Z&EventsTypesFilter=ProcessApplicationReport`. 如果您想要包括可能已經過期 (超出其存留時間 (TTL)) 的健康狀態事件，請將 `,ExpiredDeployedApplicationEvent` 新增至查詢的結尾處，以篩選兩種類型的事件。

*"myApp" 中所有服務的健康情況歷程記錄：*

目前，服務的健康情況報表事件會在相對應應用程式實體底下顯示為 `DeployedServiceHealthReportCreated` 事件。 若要查看您的服務如何為 "App1" 執行作業，請使用以下查詢：`https://winlrc-staging-10.southcentralus.cloudapp.azure.com:19080/EventsStore/Applications/myapp/$/Events?api-version=6.2-preview&starttimeutc=2017-04-22T17:01:51Z&endtimeutc=2018-04-29T17:02:51Z&EventsTypesFilter=DeployedServiceHealthReportCreated`

*分割區重新設定：*

若要查看您叢集中所發生的所有分割區移動，請查詢 `ReconfigurationCompleted` 事件。 這可以協助您在叢集中診斷問題時，找出有哪些工作負載於哪些特定時間在哪一個節點上執行。 以下是能夠達到上述目的的查詢範例：`https://mycluster.cloudapp.azure.com:19080/EventsStore/Partitions/Events?api-version=6.2-preview&starttimeutc=2018-04-22T17:01:51Z&endtimeutc=2018-04-29T17:02:51Z&EventsTypesFilter=PartitionReconfigurationCompleted`

*混沌服務：*

有一個能在混沌服務啟動或停止時，於叢集層級公開的事件。 若要查看近期使用混沌服務的資訊，請使用以下查詢：`https://mycluster.cloudapp.azure.com:19080/EventsStore/Cluster/Events?api-version=6.2-preview&starttimeutc=2017-04-22T17:01:51Z&endtimeutc=2018-04-29T17:02:51Z&EventsTypesFilter=ChaosStarted,ChaosStopped`

