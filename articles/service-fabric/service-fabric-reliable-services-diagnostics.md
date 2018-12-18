---
title: Azure Service Fabric 具狀態可靠服務診斷 | Microsoft Docs
description: Azure Service Fabric 具狀態可靠服務診斷功能
services: service-fabric
documentationcenter: .net
author: dkkapur
manager: timlt
editor: ''
ms.assetid: ae0e8f99-69ab-4d45-896d-1fa80ed45659
ms.service: Service-Fabric
ms.devlang: dotnet
ms.topic: conceptual
ms.tgt_pltfrm: NA
ms.workload: NA
ms.date: 8/24/2018
ms.author: dekapur
ms.openlocfilehash: a7ba92d871bb440b7b8c8a12c1e90f9aa10df3be
ms.sourcegitcommit: f6e2a03076679d53b550a24828141c4fb978dcf9
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/27/2018
ms.locfileid: "43105336"
---
# <a name="diagnostic-functionality-for-stateful-reliable-services"></a>具狀態 Reliable Services 診斷功能
Azure Service Fabric 具狀態可靠服務 StatefulServiceBase 類別會發出 [EventSource](https://msdn.microsoft.com/library/system.diagnostics.tracing.eventsource.aspx) 事件，可用於偵錯服務、提供執行階段運作情形的深入了解，以及協助疑難排解。

## <a name="eventsource-events"></a>EventSource 事件
具狀態可靠服務之 StatefulServiceBase 類別的 EventSource 名稱是 "Microsoft-ServiceFabric-Services"。 當 [Visual Studio 中正在偵錯](service-fabric-debugging-your-application.md)服務時，此事件來源的事件會出現在 [診斷事件](service-fabric-diagnostics-how-to-monitor-and-diagnose-services-locally.md#view-service-fabric-system-events-in-visual-studio)視窗中。

可協助您收集和/或檢視 EventSource 事件之工具和技術的範例包括 [PerfView](http://www.microsoft.com/download/details.aspx?id=28567)、[Azure 診斷](../cloud-services/cloud-services-dotnet-diagnostics.md)和 [Microsoft TraceEvent 程式庫](http://www.nuget.org/packages/Microsoft.Diagnostics.Tracing.TraceEvent)。

## <a name="events"></a>活動
| 事件名稱 | 事件識別碼 | Level | 事件說明 |
| --- | --- | --- | --- |
| StatefulRunAsyncInvocation |1 |資訊 |啟動服務 RunAsync 工作時發出 |
| StatefulRunAsyncCancellation |2 |資訊 |取消服務 RunAsync 工作時發出 |
| StatefulRunAsyncCompletion |3 |資訊 |完成服務 RunAsync 工作時發出 |
| StatefulRunAsyncSlowCancellation |4 |警告 |服務 RunAsync 工作花太多時間才能完成取消時發出 |
| StatefulRunAsyncFailure |5 |Error |服務 RunAsync 工作擲回例外狀況時發出 |

## <a name="interpret-events"></a>解譯事件
StatefulRunAsyncInvocation、StatefulRunAsyncCompletion，和 StatefulRunAsyncCancellation 事件可讓服務撰寫者了解服務的生命週期--以及服務啟動、取消或完成的時機。 此資訊可能適用於偵錯服務問題或了解服務生命週期。

服務撰寫者應密切注意 StatefulRunAsyncSlowCancellation 及 StatefulRunAsyncFailure 事件，因為它們表示服務的問題。

只要服務 RunAsync() 工作擲回例外狀況，就會發出 StatefulRunAsyncFailure。 通常，擲回例外狀況表示服務中的錯誤。 此外，例外狀況會導致服務失敗，因此它會移至另一個節點。 此作業的成本很高，且會在服務移動時延遲傳入的要求。 服務撰寫者應該判斷例外狀況的原因，並盡可能減少這種情況。

只要 RunAsync 工作的取消要求所花費的時間超過四秒，就會發出 StatefulRunAsyncSlowCancellation。 當服務完成取消所花費的時間太久，它會影響服務快速在另一個節點上重新啟動的能力。 此案例可能會影響服務的整體可用性。

## <a name="performance-counters"></a>效能計數器
Reliable Services 執行階段定義下列效能計數器類別：

| 類別 | 說明 |
| --- | --- |
| Service Fabric 異動複寫器 |Azure Service Fabric 異動複寫器特有的計數器 |
| Service Fabric TStore |Azure Service Fabric TStore 特有的計數器 |

Service Fabric 異動複寫器使用 [Reliable State Manager](service-fabric-reliable-services-reliable-collections-internals.md)，複寫所指定[複本](service-fabric-concepts-replica-lifecycle.md)中的異動。

Service Fabric TStore 是一個元件，在 [Reliable Collections](service-fabric-reliable-services-reliable-collections-internals.md) 中用來儲存及擷取索引鍵/值組。

Windows 作業系統中預設可用的 [Windows 效能監視器](https://technet.microsoft.com/library/cc749249.aspx) 應用程式可用於收集與檢視效能計數器資料。 [Azure 診斷](../cloud-services/cloud-services-dotnet-diagnostics.md) 是另一個收集效能計數器資料並上傳至 Azure 資料表的選項。

### <a name="performance-counter-instance-names"></a>效能計數器執行個體名稱
含大量可靠服務或可靠服務資料分割的叢集，將有大量的異動複寫器效能計數器執行個體。 TStore 效能計數器也是如此，但是只會乘以使用的可靠字典和可靠佇列數目。 效能計數器執行個體名稱有助於識別效能計數器執行個體相關聯的特定[磁碟分割](service-fabric-concepts-partitioning.md)、服務複本，而在 TStore 中還可識別狀態供應器。

#### <a name="service-fabric-transactional-replicator-category"></a>Service Fabric 異動複寫器類別
`Service Fabric Transactional Replicator`類別的計數器執行個體名稱格式如下：

`ServiceFabricPartitionId:ServiceFabricReplicaId`

*ServiceFabricPartitionId* 是與效能計數器執行個體相關聯 Service Fabric 資料分割識別碼的字串表示法。 資料分割識別碼是 GUID，其字串表示法是透過 [`Guid.ToString`](https://msdn.microsoft.com/library/97af8hh4.aspx) 與格式規範 "D" 所產生。

*ServiceFabricReplicaId* 是與某一可靠服務特定複本相關聯的識別碼。 複本識別碼包含在效能計數器執行個體名稱中，以確保其唯一性，並避免與相同資料分割所產生的其他效能計數器執行個體名稱發生衝突。 [這裡](service-fabric-concepts-replica-lifecycle.md)會詳細介紹複本以及它們在可靠服務中的角色。

下列的計數器執行個體名稱一般用於 `Service Fabric Transactional Replicator` 類別底下的計數器：

`00d0126d-3e36-4d68-98da-cc4f7195d85e:131652217797162571`

在上述範例中，`00d0126d-3e36-4d68-98da-cc4f7195d85e` 是 Service Fabric 資料分割識別碼的字串表示法，`131652217797162571` 則是複本識別碼。

#### <a name="service-fabric-tstore-category"></a>Service Fabric TStore 類別
`Service Fabric TStore`類別的計數器執行個體名稱格式如下：

`ServiceFabricPartitionId:ServiceFabricReplicaId:ServiceFabricStateProviderId_PerformanceCounterInstanceDifferentiator`

*ServiceFabricPartitionId* 是與效能計數器執行個體相關聯 Service Fabric 資料分割識別碼的字串表示法。 資料分割識別碼是 GUID，其字串表示法是透過 [`Guid.ToString`](https://msdn.microsoft.com/library/97af8hh4.aspx) 與格式規範 "D" 所產生。

*ServiceFabricReplicaId* 是與某一可靠服務特定複本相關聯的識別碼。 複本識別碼包含在效能計數器執行個體名稱中，以確保其唯一性，並避免與相同資料分割所產生的其他效能計數器執行個體名稱發生衝突。 [這裡](service-fabric-concepts-replica-lifecycle.md)會詳細介紹複本以及它們在可靠服務中的角色。

ServiceFabricStateProviderId 是識別碼，與可靠服務內的狀態供應器相關聯。 狀態供應器識別碼會包含在效能計數器執行個體名稱中，以便區別各個 TStore。

PerformanceCounterInstanceDifferentiator 是區別識別碼，與狀態供應器內的效能計數器執行個體相關聯。 區分碼會包含在效能計數器執行個體名稱中，以確保其唯一性，並避免與相同狀態供應器所產生的其他效能計數器執行個體名稱發生衝突。

下列的計數器執行個體名稱一般用於 `Service Fabric TStore` 類別底下的計數器：

`00d0126d-3e36-4d68-98da-cc4f7195d85e:131652217797162571:142652217797162571_1337`

在上述範例中，`00d0126d-3e36-4d68-98da-cc4f7195d85e` 是 Service Fabric 磁碟分割識別碼的字串代表、`131652217797162571` 是複本識別碼、`142652217797162571` 是狀態供應器識別碼，而 `1337` 是效能計數器執行個體區分碼。

### <a name="transactional-replicator-performance-counters"></a>異動複寫器效能計數器

Reliable Services 執行階段會發出 `Service Fabric Transactional Replicator` 類別底下的下列事件

 計數器名稱 | 說明 |
| --- | --- |
| Begin Txn Operations/sec | 每秒建立的新寫入異動數目。|
| Txn Operations/sec | 每秒在可靠集合上所執行的新增/更新/刪除作業數目。|
| Avg.Flush Latency (ms) | 異動複寫器每秒排清到磁碟的位元組數 |
| Throttled Operations/sec | 異動複寫器每秒因為節流而拒絕的作業數目。 |
| Avg.Transaction ms/Commit | 每一次異動的平均認可延遲 (毫秒) |
| Avg.Flush Latency (ms) | 異動複寫器起始的磁碟排清作業平均持續時間 (毫秒) |

### <a name="tstore-performance-counters"></a>TStore 效能計數器

Reliable Services 執行階段會發出 `Service Fabric TStore` 類別底下的下列事件

 計數器名稱 | 說明 |
| --- | --- |
| 項目計數 | 存放區中的項目數。|
| 磁碟大小 | 存放區檢查點檔案的磁碟大小總計 (位元組)。|
| 檢查點檔案寫入位元組/秒 | 最近檢查點檔案的每秒寫入位元組數。|
| 複製磁碟傳輸位元組/秒 | 存放區複製期間的每秒讀取 (主要複本上) 或寫入 (次要複本上) 磁碟位元組數。|

## <a name="next-steps"></a>後續步驟
[PerfView 中的 EventSource 提供者](https://blogs.msdn.microsoft.com/vancem/2012/07/09/introduction-tutorial-logging-etw-events-in-c-system-diagnostics-tracing-eventsource/)
