---
title: Azure ServiceFabric 診斷和監視功能 | Microsoft Docs
description: 本文將說明 Service Fabric Reliable ServiceRemoting 執行階段中的效能監視功能，例如其發出的效能計數器。
services: service-fabric
documentationcenter: .net
author: suchiagicha
manager: timlt
editor: suchiagicha
ms.assetid: 1c229923-670a-4634-ad59-468ff781ad18
ms.service: service-fabric
ms.devlang: dotnet
ms.topic: conceptual
ms.tgt_pltfrm: NA
ms.workload: NA
ms.date: 06/29/2017
ms.author: suchiagicha
ms.openlocfilehash: d462ba0955a362c27b786ee6a5670eec20c52a22
ms.sourcegitcommit: eb75f177fc59d90b1b667afcfe64ac51936e2638
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 05/16/2018
ms.locfileid: "34206414"
---
# <a name="diagnostics-and-performance-monitoring-for-reliable-service-remoting"></a>Reliable Service Remoting 的診斷和效能監視
Reliable ServiceRemoting 執行階段會發出[效能計數器](https://msdn.microsoft.com/library/system.diagnostics.performancecounter.aspx)。 這些項目提供深入了解 ServiceRemoting 的運作方式，並有助於疑難排解及效能監視。


## <a name="performance-counters"></a>效能計數器
Reliable ServiceRemoting 執行階段定義下列效能計數器類別：

| 類別 | 說明 |
| --- | --- |
| Service Fabric Service |Azure Service Fabric Service Remoting 的特定計數器，例如處理要求所花費的平均時間 |
| Service Fabric Service Method |Service Fabric Remoting Service 所實作方法特定的計數器，例如叫用服務方法的頻率 |

前述的類別每一個都有一或多個計數器。

Windows 作業系統中預設可用的 [Windows 效能監視器](https://technet.microsoft.com/library/cc749249.aspx) 應用程式可用於收集與檢視效能計數器資料。 [Azure 診斷](../cloud-services/cloud-services-dotnet-diagnostics.md) 是另一個收集效能計數器資料並上傳至 Azure 資料表的選項。

### <a name="performance-counter-instance-names"></a>效能計數器執行個體名稱
含大量 ServiceRemoting 服務或資料分割的叢集有大量的效能計數器執行個體。 效能計數器執行個體名稱可幫助識別效能計數器執行個體相關聯的特定資料分割與服務方法 (如果適用)。

#### <a name="service-fabric-service-category"></a>Service Fabric Service 類別
`Service Fabric Service`類別的計數器執行個體名稱格式如下：

`ServiceFabricPartitionID_ServiceReplicaOrInstanceId_ServiceRuntimeInternalID`

*ServiceFabricPartitionID* 是與效能計數器執行個體相關聯之 Service Fabric 資料分割識別碼的字串表示法。 資料分割識別碼是 GUID，其字串表示法是透過 [`Guid.ToString`](https://msdn.microsoft.com/library/97af8hh4.aspx) 方法與格式規範 "D" 所產生。

*ServiceReplicaOrInstanceId* 是與效能計數器執行個體相關聯之 Service Fabric 複本/執行個體識別碼的字串表示法。

*ServiceRuntimeInternalID* 是 Fabric Service 執行階段所產生 64 位元整數的字串表示法，供內部使用。 這包含在效能計數器執行個體名稱中，以確保其唯一性，並避免與其他效能計數器執行個體名稱衝突。 使用者不應該嘗試解譯效能計數器執行個體名稱的這個部分。

以下為屬於 `Service Fabric Service` 類別之計數器的計數器執行個體名稱範例：

`2740af29-78aa-44bc-a20b-7e60fb783264_635650083799324046_5008379932`

在前述範例中，`2740af29-78aa-44bc-a20b-7e60fb783264` 是 Service Fabric 資料分割識別碼的字串表示法，`635650083799324046` 是複本/執行個體識別碼的字串表示法，而 `5008379932` 是為了內部使用執行階段所產生的 64 位元識別碼。

#### <a name="service-fabric-service-method-category"></a>Service Fabric Service Method 類別
`Service Fabric Service Method`類別的計數器執行個體名稱格式如下：

`MethodName_ServiceRuntimeMethodId_ServiceFabricPartitionID_ServiceReplicaOrInstanceId_ServiceRuntimeInternalID`

*MethodName* 是效能計數器執行個體相關聯的服務方法名稱。 方法名稱的格式取決於 Fabric Service 執行階段某個邏輯，該邏輯會在名稱的可讀性與 Windows 上效能計數器執行個體名稱長度上限之間取得平衡。

*ServiceRuntimeMethodId* 是 Fabric Service 執行階段所產生 32 位元整數的字串表示法，供內部使用。 這包含在效能計數器執行個體名稱中，以確保其唯一性，並避免與其他效能計數器執行個體名稱衝突。 使用者不應該嘗試解譯效能計數器執行個體名稱的這個部分。

*ServiceFabricPartitionID* 是與效能計數器執行個體相關聯之 Service Fabric 資料分割識別碼的字串表示法。 資料分割識別碼是 GUID，其字串表示法是透過 [`Guid.ToString`](https://msdn.microsoft.com/library/97af8hh4.aspx) 方法與格式規範 "D" 所產生。

*ServiceReplicaOrInstanceId* 是與效能計數器執行個體相關聯之 Service Fabric 複本/執行個體識別碼的字串表示法。

*ServiceRuntimeInternalID* 是 Fabric Service 執行階段所產生 64 位元整數的字串表示法，供內部使用。 這包含在效能計數器執行個體名稱中，以確保其唯一性，並避免與其他效能計數器執行個體名稱衝突。 使用者不應該嘗試解譯效能計數器執行個體名稱的這個部分。

以下為屬於 `Service Fabric Service Method` 類別之計數器的計數器執行個體名稱範例：

`ivoicemailboxservice.leavemessageasync_2_89383d32-e57e-4a9b-a6ad-57c6792aa521_635650083804480486_5008380`

在前述範例中，`ivoicemailboxservice.leavemessageasync` 是方法名稱，`2` 是為了內部使用執行階段所產生的 32 位元識別碼，`89383d32-e57e-4a9b-a6ad-57c6792aa521` 是 Service Fabric 資料分割識別碼的字串表示法，`635650083804480486` 是 Service Fabric 複本/執行個體識別碼的字串表示法，而 `5008380` 是為了內部使用執行階段所產生的 64 位元識別碼。

## <a name="list-of-performance-counters"></a>效能計數器的清單
### <a name="service-method-performance-counters"></a>服務方法效能計數器

Reliable Service 執行階段會發佈與執行服務方法相關的下列效能計數器。

| 類別名稱 | 計數器名稱 | 說明 |
| --- | --- | --- |
| Service Fabric Service Method |叫用數目/秒 |每秒叫用服務方法的次數 |
| Service Fabric Service Method |每個叫用的平均毫秒數 |執行服務方法花費的時間 (單位為毫秒) |
| Service Fabric Service Method |擲回的例外狀況數/秒 |每秒服務方法擲回例外狀況的次數 |

### <a name="service-request-processing-performance-counters"></a>服務要求處理效能計數器
當用戶端透過服務 Proxy 物件叫用方法時，會造成要求訊息透過網路傳送至遠端服務。 服務會處理要求訊息，並傳送回應給用戶端。 Reliable ServiceRemoting 執行階段會發佈下列與服務要求處理相關的效能計數器。

| 類別名稱 | 計數器名稱 | 說明 |
| --- | --- | --- |
| Service Fabric Service |# of outstanding requests |服務中正在處理的要求數目 |
| Service Fabric Service |每個要求的平均毫秒數 |服務處理要求所花費的時間 (單位為毫秒) |
| Service Fabric Service |要求還原序列化的平均毫秒數 |當服務收到服務要求訊息時，將它還原序列化所花費的時間 (單位為毫秒) |
| Service Fabric Service |要求序列化的平均毫秒數 |在回應傳送至用戶端之前，序列化服務回應訊息所花費的時間 (單位為毫秒) |

## <a name="next-steps"></a>後續步驟
* [範例程式碼](https://github.com/Azure/servicefabric-samples)
* [PerfView 中的 EventSource 提供者](https://blogs.msdn.microsoft.com/vancem/2012/07/09/introduction-tutorial-logging-etw-events-in-c-system-diagnostics-tracing-eventsource/)
