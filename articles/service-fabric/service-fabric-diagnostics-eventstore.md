---
title: Azure Service Fabric 事件存放區 | Microsoft Docs
description: 深入了解 Azure Service Fabric 的 EventStore
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
ms.openlocfilehash: 1d235d5a75975c8d58cad1bbde0f16df2b1b3e7a
ms.sourcegitcommit: eb75f177fc59d90b1b667afcfe64ac51936e2638
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 05/16/2018
ms.locfileid: "34206511"
---
# <a name="eventstore-service-overview"></a>EventStore 服務概觀

>[!NOTE]
>截至 Service Fabric 6.2 版為止， EventStore API 目前為預覽版本，僅適用於在 Azure 上執行的 Windows 叢集。 我們正在將這個功能和我們的獨立叢集移植到 Linux。

## <a name="overview"></a>概觀

於 6.2 版引進的 EventStore 服務是 Service Fabric 中的監視服務，能讓您了解叢集或工作負載在指定時間點的狀態。 EventStore 服務能透過具備呼叫功能的 API 公開 Service Fabric 事件。 這些 EventStore API 能讓您直接查詢叢集，取得叢集中任何實體的診斷資料，因此應該用來協助您：
* 診斷開發或測試中的問題，或在使用監視管線的場合中診斷問題
* 確認叢集是否能正確處理您針對叢集採取的管理動作

若要查看 EventStore 中可用事件的完整清單，請參閱 [Service Fabric 事件](service-fabric-diagnostics-event-generation-operational.md)。

您可以查詢 EventStore 服務，找出叢集中每個實體和實體類型適用的事件。 這表示您可以查詢以下層級的事件：
* 叢集：所有叢集層級事件
* 節點：所有節點層級事件
* 節點：某個節點特有的事件 (根據 `nodeName`)
* 應用程式：所有應用程式層級事件
* 應用程式：某個應用程式特有的事件
* 服務：來自叢集中所有服務的事件
* 服務：來自特定服務的事件
* 分割區：來自所有分割區的事件
* 分割區：來自某個特定分割區的事件
* 複本：來自所有複本 / 執行個體的事件
* 複本：來自某個特定複本 / 執行個體的事件


EventStore 服務還能將叢集中的事件相互關聯。 透過同時間寫入的事件，您可以追溯到彼此影響的相異實體；EventStore 服務能連結這些事件，協助您找出叢集中活動的原因。 例如，如果某個應用程式在未引發變更的情況下成為不良狀況，EventStore 也會查看平台公開的其他事件，因而可能會將這個狀況與 `NodeDown` 事件相互關聯。 這有助於縮短偵測失敗及分析根本原因的時間。

若要開始使用 EventStore 服務，請參閱[查詢 EventStore API 以查詢叢集事件](service-fabric-diagnostics-eventstore-query.md)。

## <a name="next-steps"></a>後續步驟
* Service Fabric 的監視和診斷概觀 - [Service Fabric 的監視和診斷](service-fabric-diagnostics-overview.md)
* 深入了解監視叢集 - [監視叢集和平台](service-fabric-diagnostics-event-generation-infra.md)。