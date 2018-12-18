---
title: 動作項目診斷和監視功能 | Microsoft Docs
description: 本文將說明 Service Fabric Reliable Actors 執行階段中的診斷與效能監視功能，包括其發出的事件與效能計數器。
services: service-fabric
documentationcenter: .net
author: abhishekram
manager: timlt
editor: vturecek
ms.assetid: 1c229923-670a-4634-ad59-468ff781ad18
ms.service: service-fabric
ms.devlang: dotnet
ms.topic: conceptual
ms.tgt_pltfrm: NA
ms.workload: NA
ms.date: 10/26/2017
ms.author: abhisram
ms.openlocfilehash: 03c8bf129bcb8bf1b8af4bb7d189f1a38ba8f503
ms.sourcegitcommit: 6cf20e87414dedd0d4f0ae644696151e728633b6
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/06/2018
ms.locfileid: "34809085"
---
# <a name="diagnostics-and-performance-monitoring-for-reliable-actors"></a>Reliable Actors 的診斷和效能監視
Reliable Actors 執行階段會發出 [EventSource](https://msdn.microsoft.com/library/system.diagnostics.tracing.eventsource.aspx) 事件與[效能計數器](https://msdn.microsoft.com/library/system.diagnostics.performancecounter.aspx)。 這些項目提供深入了解執行階段的運作方式，並有助於疑難排解及效能監視。

## <a name="eventsource-events"></a>EventSource 事件
Reliable Actors 執行階段的 EventSource 提供者名稱為 "Microsoft-ServiceFabric-Actors"。 當 [Visual Studio 中正在偵錯](service-fabric-debugging-your-application.md)動作項目應用程式時，事件來源中的事件會出現在 [診斷事件](service-fabric-diagnostics-how-to-monitor-and-diagnose-services-locally.md#view-service-fabric-system-events-in-visual-studio) 視窗中。

可協助您收集和/或檢視 EventSource 事件的工具和技術範例包括 [PerfView](http://www.microsoft.com/download/details.aspx?id=28567)、[Azure 診斷](../cloud-services/cloud-services-dotnet-diagnostics.md)、[語意記錄](https://msdn.microsoft.com/library/dn774980.aspx)和 [Microsoft TraceEvent 程式庫](http://www.nuget.org/packages/Microsoft.Diagnostics.Tracing.TraceEvent)。

### <a name="keywords"></a>關鍵字
所有屬於 Reliable Actor EventSource 的事件皆有一或多個相關聯的關鍵字。 這會啟動篩選所選的事件。 以下為已定義的關鍵字位元。

| Bit | 說明 |
| --- | --- |
| 0x1 |可彙總 Fabric 動作項目執行階段作業的重要事件集。 |
| 0x2 |說明動作項目方法呼叫的事件集。 如需詳細資訊，請參閱[動作項目簡介主題](service-fabric-reliable-actors-introduction.md)。 |
| 0x4 |與動作項目狀態相關的事件集。 如需詳細資訊，請參閱 [動作項目狀態管理](service-fabric-reliable-actors-state-management.md)主題。 |
| 0x8 |與動作項目的回合式並行相關的事件集。 如需詳細資訊，請參閱 [並行](service-fabric-reliable-actors-introduction.md#concurrency)主題。 |

## <a name="performance-counters"></a>效能計數器
Reliable Actor 執行階段定義下列效能計數器類別。

| 類別 | 說明 |
| --- | --- |
| Service Fabric 動作項目 |Azure Service Fabric 動作項目特定的計數器，例如儲存動作項目狀態花費的時間 |
| Service Fabric 動作項目方法 |Service Fabric 動作項目所實作方法特定的計數器，例如叫用動作項目方法的頻率 |

上述的類別每一個都有一或多個計數器。

Windows 作業系統中預設可用的 [Windows 效能監視器](https://technet.microsoft.com/library/cc749249.aspx) 應用程式可用於收集與檢視效能計數器資料。 [Azure 診斷](../cloud-services/cloud-services-dotnet-diagnostics.md) 是另一個收集效能計數器資料並上傳至 Azure 資料表的選項。

### <a name="performance-counter-instance-names"></a>效能計數器執行個體名稱
含大量動作項目服務或動作項目服務資料分割的叢集將有大量的動作項目效能計數器執行個體。 效能計數器執行個體名稱可幫助識別效能計數器執行個體相關聯的特定 [資料分割](service-fabric-reliable-actors-platform.md#service-fabric-partition-concepts-for-actors) 與動作項目方法 (如果適用)。

#### <a name="service-fabric-actor-category"></a>Service Fabric 動作項目類別
`Service Fabric Actor`類別的計數器執行個體名稱格式如下：

`ServiceFabricPartitionID_ActorsRuntimeInternalID`

*ServiceFabricPartitionID* 是與效能計數器執行個體相關聯之 Service Fabric 資料分割識別碼的字串表示法。 資料分割識別碼是 GUID，其字串表示法是透過 [`Guid.ToString`](https://msdn.microsoft.com/library/97af8hh4.aspx) 方法與格式規範 "D" 所產生。

*ActorRuntimeInternalID* 是 Fabric 動作項目執行階段所產生 64 位元整數的字串表示法，供內部使用。 這包含在效能計數器執行個體名稱中，以確保其唯一性，並避免與其他效能計數器執行個體名稱衝突。 使用者不應該嘗試解譯效能計數器執行個體名稱的這個部分。

以下為屬於 `Service Fabric Actor` 類別之計數器的計數器執行個體名稱範例：

`2740af29-78aa-44bc-a20b-7e60fb783264_635650083799324046`

在上述範例中，`2740af29-78aa-44bc-a20b-7e60fb783264` 是 Service Fabric 資料分割識別碼的字串表示法，`635650083799324046` 是為了內部使用執行階段所產生的 64 位元識別碼。

#### <a name="service-fabric-actor-method-category"></a>Service Fabric 動作項目方法類別
`Service Fabric Actor Method`類別的計數器執行個體名稱格式如下：

`MethodName_ActorsRuntimeMethodId_ServiceFabricPartitionID_ActorsRuntimeInternalID`

*MethodName* 是效能計數器執行個體相關聯的動作項目方法名稱。 方法名稱的格式取決於 Fabric Actor 執行階段某個邏輯，該邏輯會在名稱的可讀性與 Windows 上效能計數器執行個體名稱長度上限之間取得平衡。

*ActorsRuntimeMethodId* 是 Fabric 動作項目執行階段所產生 32 位元整數的字串表示法，供內部使用。 這包含在效能計數器執行個體名稱中，以確保其唯一性，並避免與其他效能計數器執行個體名稱衝突。 使用者不應該嘗試解譯效能計數器執行個體名稱的這個部分。

*ServiceFabricPartitionID* 是與效能計數器執行個體相關聯之 Service Fabric 資料分割識別碼的字串表示法。 資料分割識別碼是 GUID，其字串表示法是透過 [`Guid.ToString`](https://msdn.microsoft.com/library/97af8hh4.aspx) 方法與格式規範 "D" 所產生。

*ActorRuntimeInternalID* 是 Fabric 動作項目執行階段所產生 64 位元整數的字串表示法，供內部使用。 這包含在效能計數器執行個體名稱中，以確保其唯一性，並避免與其他效能計數器執行個體名稱衝突。 使用者不應該嘗試解譯效能計數器執行個體名稱的這個部分。

以下為屬於 `Service Fabric Actor Method` 類別之計數器的計數器執行個體名稱範例：

`ivoicemailboxactor.leavemessageasync_2_89383d32-e57e-4a9b-a6ad-57c6792aa521_635650083804480486`

在上述範例中，`ivoicemailboxactor.leavemessageasync` 是方法名稱，`2` 是為了執行階段內部使用所產生的 32 位元識別碼，`89383d32-e57e-4a9b-a6ad-57c6792aa521` 是 Service Fabric 資料分割識別碼的字串表示法，`635650083804480486` 是為了執行階段內部使用所產生的 64 位元識別碼。

## <a name="list-of-events-and-performance-counters"></a>事件與效能計數器清單
### <a name="actor-method-events-and-performance-counters"></a>動作項目方法事件與效能計數器
Reliable Actors 執行階段會發出下列與 [動作項目方法](service-fabric-reliable-actors-introduction.md)相關的事件。

| 事件名稱 | 事件識別碼 | Level | 關鍵字 | 說明 |
| --- | --- | --- | --- | --- |
| ActorMethodStart |7 |詳細資訊 |0x2 |動作項目執行階段即將叫用動作項目方法。 |
| ActorMethodStop |8 |詳細資訊 |0x2 |動作項目方法已經完成執行。 亦即，執行階段對動作項目方法的非同步呼叫已經傳回，且由動作項目方法傳回的工作已經完成。 |
| ActorMethodThrewException |9 |警告 |0x3 |執行動作項目方法期間擲回例外狀況，無論是在對動作項目方法的執行階段非同步呼叫期間，或執行動作項目方法傳回的工作期間。 此事件表示需要調查的動作項目程式碼發生某種失敗。 |

Reliable Actor 執行階段會發佈與執行動作項目方法相關的下列效能計數器。

| 類別名稱 | 計數器名稱 | 說明 |
| --- | --- | --- |
| Service Fabric 動作項目方法 |叫用數目/秒 |每秒叫用動作項目服務方法的次數 |
| Service Fabric 動作項目方法 |每個叫用的平均毫秒數 |執行動作項目服務方法花費的時間 (單位為毫秒) |
| Service Fabric 動作項目方法 |擲回的例外狀況數/秒 |每秒動作項目服務方法擲回例外狀況的次數 |

### <a name="concurrency-events-and-performance-counters"></a>並行事件與效能計數器
Reliable Actor 執行階段會發出下列與 [並行](service-fabric-reliable-actors-introduction.md#concurrency)相關的事件。

| 事件名稱 | 事件識別碼 | Level | 關鍵字 | 說明 |
| --- | --- | --- | --- | --- |
| ActorMethodCallsWaitingForLock |12 |詳細資訊 |0x8 |在動作項目每個新回合開始時會寫入此事件。 其包含擱置中的動作項目呼叫數目，這些呼叫正等待取得強制執行回合式並行的各動作項目鎖定。 |

Reliable Actor 執行階段會發佈下列與並行相關的效能計數器。

| 類別名稱 | 計數器名稱 | 說明 |
| --- | --- | --- |
| Service Fabric 動作項目 |# of actor calls waiting for actor lock |擱置中的動作項目呼叫數目，這些呼叫正等待取得強制執行回合式並行的各動作項目鎖定 |
| Service Fabric 動作項目 |每個 Lock Wait 的平均毫秒數 |取得強制執行回合式並行的各動作項目鎖定所虛的時間 (單位為毫秒) |
| Service Fabric 動作項目 |保留動作項目鎖定的平均毫秒數 |保留每個動作項目鎖定的時間 (單位為毫秒) |

### <a name="actor-state-management-events-and-performance-counters"></a>動作項目狀態管理事件與效能計數器
Reliable Actor 執行階段會發出下列與 [動作項目狀態管理](service-fabric-reliable-actors-state-management.md)相關的事件。

| 事件名稱 | 事件識別碼 | Level | 關鍵字 | 說明 |
| --- | --- | --- | --- | --- |
| ActorSaveStateStart |10 |詳細資訊 |0x4 |動作項目執行階段即將儲存動作項目狀態。 |
| ActorSaveStateStop |11 |詳細資訊 |0x4 |動作項目執行階段已完成儲存動作項目狀態。 |

Reliable Actor 執行階段會發佈下列與動作項目狀態管理相關的效能計數器。

| 類別名稱 | 計數器名稱 | 說明 |
| --- | --- | --- |
| Service Fabric 動作項目 |每個儲存狀態作業的平均毫秒數 |儲存動作項目狀態花費的時間 (單位為毫秒) |
| Service Fabric 動作項目 |每個載入狀態作業的平均毫秒數 |載入動作項目狀態花費的時間 (單位為毫秒) |

### <a name="events-related-to-actor-replicas"></a>與動作項目複本相關的事件
Reliable Actors 執行階段會發出下列與 [動作項目複本](service-fabric-reliable-actors-platform.md#service-fabric-partition-concepts-for-actors)相關的事件。

| 事件名稱 | 事件識別碼 | Level | 關鍵字 | 說明 |
| --- | --- | --- | --- | --- |
| ReplicaChangeRoleToPrimary |1 |資訊 |0x1 |動作項目複本已將角色變為主要。 這意指將在此複本內建立此資料分割的動作項目。 |
| ReplicaChangeRoleFromPrimary |2 |資訊 |0x1 |動作項目複本已將角色變為非主要。 這意指在此複本內不再為此資料分割建立動作項目。 沒有新的要求將傳遞到已在此複本內建立的動作項目。 任何進行中的要求完成後，將終結動作項目。 |

### <a name="actor-activation-and-deactivation-events-and-performance-counters"></a>動作項目啟動與停用事件與效能計數器
Reliable Actor 執行階段會發出下列與 [動作項目啟動與停用](service-fabric-reliable-actors-lifecycle.md)相關的事件。

| 事件名稱 | 事件識別碼 | Level | 關鍵字 | 說明 |
| --- | --- | --- | --- | --- |
| ActorActivated |5 |資訊 |0x1 |動作項目已啟動。 |
| ActorDeactivated |6 |資訊 |0x1 |動作項目已停用。 |

Reliable Actor 執行階段會發佈下列與動作項目啟用和停用相關的效能計數器。

| 類別名稱 | 計數器名稱 | 說明 |
| --- | --- | --- |
| Service Fabric 動作項目 |平均 OnActivateAsync 毫秒數 |執行 OnActivateAsync 方法花費的時間 (單位為毫秒) |

### <a name="actor-request-processing-performance-counters"></a>動作項目要求處理效能計數器
當用戶端透過動作項目 proxy 物件叫用方法時，會造成要求訊息透過網路傳送至動作項目服務。 服務會處理要求訊息，並傳送回應給用戶端。 Reliable Actor 執行階段會發佈下列與動作項目要求處理相關的效能計數器。

| 類別名稱 | 計數器名稱 | 說明 |
| --- | --- | --- |
| Service Fabric 動作項目 |# of outstanding requests |服務中正在處理的要求數目 |
| Service Fabric 動作項目 |每個要求的平均毫秒數 |服務處理要求所花費的時間 (單位為毫秒) |
| Service Fabric 動作項目 |要求還原序列化的平均毫秒數 |當服務收到動作項目要求訊息時，將它還原序列化所花費的時間 (單位為毫秒) |
| Service Fabric 動作項目 |要求序列化的平均毫秒數 |在回應傳送至用戶端之前，序列化動作項目回應訊息所花費的時間 (單位為毫秒) |

## <a name="next-steps"></a>後續步驟
* [Reliable Acto 如何使用 Service Fabric 平台](service-fabric-reliable-actors-platform.md)
* [動作項目 API 參考文件](https://msdn.microsoft.com/library/azure/dn971626.aspx)
* [範例程式碼](https://github.com/Azure-Samples/service-fabric-dotnet-getting-started)
* [PerfView 中的 EventSource 提供者](https://blogs.msdn.microsoft.com/vancem/2012/07/09/introduction-tutorial-logging-etw-events-in-c-system-diagnostics-tracing-eventsource/)
