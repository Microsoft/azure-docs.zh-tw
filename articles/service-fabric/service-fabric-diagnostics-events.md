---
title: Azure Service Fabric 事件 | Microsoft Docs
description: 了解協助您監視 Azure Service Fabric 叢集的現成 Service Fabric 事件。
services: service-fabric
documentationcenter: .net
author: dkkapur
manager: timlt
editor: ''
ms.assetid: ''
ms.service: service-fabric
ms.devlang: dotnet
ms.topic: conceptual
ms.tgt_pltfrm: NA
ms.workload: NA
ms.date: 04/25/2018
ms.author: dekapur
ms.openlocfilehash: b9372c806eab1b0ca69ba078d972b076c8a7d6f6
ms.sourcegitcommit: eb75f177fc59d90b1b667afcfe64ac51936e2638
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 05/16/2018
ms.locfileid: "34212705"
---
# <a name="service-fabric-events"></a>Service Fabric 事件 

Service Fabric 平台會撰寫好幾種結構化事件，呈現叢集內發生的重要作業活動。 不論是叢集升級或複本放置決策等，都包括在內。 Service Fabric 公開的每個事件，都與叢集中的以下其中一個實體相對應：
* 叢集
* Application
* 服務
* 資料分割
* 複本 
* 容器

查看平台公開的完整事件清單 - [Service Fabric 事件清單](service-fabric-diagnostics-event-generation-operational.md)。

以下是幾個重要的情節範例，您應該可以在叢集中看到這些情節的事件。 
1. 節點生命週期事件：隨著節點上線、下線、啟動/停用或重新啟動，平台會藉由公開事件來指出發生什麼情況，同時協助您辨別機器本身是否發生任何問題，或是否曾透過 SF 呼叫 API 來修改節點狀態。
1. 叢集升級：當叢集升級 (SF 版本或組態變更) 時，您可以看見升級起始、在每個 UD 間展開及完成 (或復原)。 
1. 應用程式升級：與叢集升級相似，當升級展開時會產生一組詳盡的事件。 這些事件能用來得知升級的排程、升級目前的狀態，以及事件的整體順序。 它們能用來回顧哪些升級已成功推出。
1. 應用程式/服務的部署/刪除：每個應用程式、服務及容器建立或刪除時，都會產生相應的事件。
1. 分割區移動 (重新組態)：每當具狀態分割區經歷重新組態 (複本集發生變更) 時，平台會記錄事件。 如果您正在嘗試了解分割區複本集變更的頻率，或是追蹤哪個節點在任何時間點執行主要複本，這項功能將能派上用場。
1. Chaos 事件：使用 Service Fabric 的 [Chaos](service-fabric-controlled-chaos.md) 服務時，每當服務啟動或停止，或是在系統中插入錯誤時，您都可以看到事件。
1. 健康情況事件：每當警告或錯誤健康情況報告建立時、實體恢復良好健康情況時，或健康情況報告到期時，Service Fabric 都會公開健康情況事件。 這些事件特別有助於追蹤實體的歷史健康情況統計資料。 

## <a name="how-to-access-events"></a>如何存取事件

存取 Service Fabric 事件的方法有好幾個：
* 透過作業通道。 這些事件可以透過 Azure 診斷延伸模組收集，再傳送到儲存體資料表以供使用，或擷取到 OMS Log Analytics 之類的工具。 當叢集啟用「診斷」時，Azure 診斷代理程式會部署在您的叢集上，而且預設會設定為從作業通道讀入記錄。 深入了解如何設定 [Azure 診斷代理程式](service-fabric-diagnostics-event-aggregation-wad.md)修改叢集的診斷設定，以選取更多記錄或效能計數器。 
* 透過讓您直接查詢叢集的 EventStore 服務 Rest API，或透過 Service Fabric 用戶端程式庫。 請參閱[查詢 EventStore API 以查詢叢集事件](service-fabric-diagnostics-eventstore-query.md)。

## <a name="next-steps"></a>後續步驟
* 監視叢集的詳細資訊 - [監視叢集和平台](service-fabric-diagnostics-event-generation-infra.md)。
* 深入了解 EventStore 服務 - [EventStore 服務概觀](service-fabric-diagnostics-eventstore.md)
