---
title: Azure Service Fabric 事件清單 | Microsoft Docs
description: Azure Service Fabric 所提供的事件完整清單可協助監控叢集。
services: service-fabric
documentationcenter: .net
author: srrengar
manager: timlt
editor: ''
ms.assetid: ''
ms.service: service-fabric
ms.devlang: dotnet
ms.topic: reference
ms.tgt_pltfrm: NA
ms.workload: NA
ms.date: 04/25/2018
ms.author: dekapur
ms.openlocfilehash: 03dac03405588ba00a0f8ca5b127956c40853e36
ms.sourcegitcommit: 0bb8db9fe3369ee90f4a5973a69c26bff43eae00
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/08/2018
ms.locfileid: "48868508"
---
# <a name="list-of-service-fabric-events"></a>Service Fabric 事件清單 

Service Fabric 會公開一組主要的叢集事件，以 [Service Fabric 事件](service-fabric-diagnostics-events.md)的方式向您通知叢集的狀態。 這些是以 Service Fabric 在您的節點和叢集上執行的動作，或以叢集擁有者/操作者所做的管理決定為基礎。 您可以透過查詢叢集中的 [EventStore](service-fabric-diagnostics-eventstore.md)，或透過操作通道存取這些事件。 在 Windows 電腦中，操作通道也會與 EventLog 連結，因此您可以在事件檢視器中看到 Service Fabric 事件。 

>[!NOTE]
>如需低於 6.2 版叢集的 Service Fabric 事件清單，請參閱下一節。 

以下是平台中所有可用事件的清單，依它們所對應的實體排序。

## <a name="cluster-events"></a>叢集事件

**叢集升級事件**

| EventId | 名稱 | 說明 |來源 (工作) | Level | 版本 |
| --- | --- | --- | --- | --- | --- |
| 29627 | ClusterUpgradeStarted | 已啟動叢集更新 | CM | 資訊 | 1 |
| 29628 | ClusterUpgradeCompleted | 已完成叢集更新| CM | 資訊 | 1 |
| 29629 | ClusterUpgradeRollbackStarted | 已開始復原叢集更新 | CM | 資訊 | 1 |
| 29630 | ClusterUpgradeRollbackCompleted | 已完成復原叢集更新 | CM | 資訊 | 1 |
| 29631 | ClusterUpgradeDomainCompleted | 已在叢集升級期間完成網域升級 | CM | 資訊 | 1 |

## <a name="node-events"></a>節點事件

**節點生命週期事件** 

| EventId | 名稱 | 說明 |來源 (工作) | Level | 版本 |
| --- | --- | ---| --- | --- | --- |
| 18602 | NodeDeactivateCompleted | 已完成停用節點 | FM | 資訊 | 1 |
| 18603 | NodeUp | 叢集偵測到節點已啟動 | FM | 資訊 | 1 |
| 18604 | NodeDown | 叢集偵測到節點已關閉 |  FM | 資訊 | 1 |
| 18605 | NodeAddedToCluster | 已將新的節點新增至叢集 | FM | 資訊 | 1 |
| 18606 | NodeRemovedFromCluster | 已從叢集中移除節點 | FM | 資訊 | 1 |
| 18607 | NodeDeactivateStarted | 已啟動停用節點作業 | FM | 資訊 | 1 |
| 25620 | NodeOpening | 節點正在啟動。 節點生命週期的第一個階段 | FabricNode | 資訊 | 1 |
| 25621 | NodeOpenSucceeded | 節點已成功啟動 | FabricNode | 資訊 | 1 |
| 25622 | NodeOpenFailed | 節點無法啟動 | FabricNode | 資訊 | 1 |
| 25623 | NodeClosing | 節點正在關閉。 節點生命週期最後階段的開始 | FabricNode | 資訊 | 1 |
| 25624 | NodeClosed | 已成功關閉節點 | FabricNode | 資訊 | 1 |
| 25625 | NodeAborting | 正在開始強制關閉節點 | FabricNode | 資訊 | 1 |
| 25626 | NodeAborted | 已強制關閉節點 | FabricNode | 資訊 | 1 |

## <a name="application-events"></a>應用程式事件

**應用程式生命週期事件**

| EventId | 名稱 | 說明 |來源 (工作) | Level | 版本 |
| --- | --- | ---| --- | --- | --- |
| 29620 | ApplicationCreated | 已建立新的應用程式 | CM | 資訊 | 1 |
| 29625 | ApplicationDeleted | 已刪除現有的應用程式 | CM | 資訊 | 1 |
| 23083 | ApplicationProcessExited | 已結束應用程式中的處理程序 | 裝載 | 資訊 | 1 |

**應用程式升級事件**

| EventId | 名稱 | 說明 |來源 (工作) | Level | 版本 |
| --- | --- | ---| --- | --- | --- |
| 29621 | ApplicationUpgradeStarted | 已啟動應用程式更新 | CM | 資訊 | 1 |
| 29622 | ApplicationUpgradeCompleted | 已完成應用程式更新 | CM | 資訊 | 1 |
| 29623 | ApplicationUpgradeRollbackStarted | 已開始復原應用程式更新 |CM | 資訊 | 1 |
| 29624 | ApplicationUpgradeRollbackCompleted | 已完成復原應用程式更新 | CM | 資訊 | 1 |
| 29626 | ApplicationUpgradeDomainCompleted | 已在應用程式升級期間完成網域升級 | CM | 資訊 | 1 |

## <a name="service-events"></a>服務事件

**服務生命週期事件**

| EventId | 名稱 | 說明 |來源 (工作) | Level | 版本 |
| --- | --- | ---| --- | --- | --- |
| 18657 | ServiceCreated | 已建立新服務 | FM | 資訊 | 1 |
| 18658 | ServiceDeleted | 已刪除現有服務 | FM | 資訊 | 1 |

## <a name="partition-events"></a>分割事件

**分割移動事件**

| EventId | 名稱 | 說明 |來源 (工作) | Level | 版本 |
| --- | --- | ---| --- | --- | --- |
| 18940 | PartitionReconfigured | 已完成重新設定分割 | RA | 資訊 | 1 |

## <a name="container-events"></a>容器事件

**容器生命週期事件** 

| EventId | 名稱 | 說明 |來源 (工作) | Level | 版本 |
| --- | --- | ---| --- | --- | --- |
| 23074 | ContainerActivated | 容器已啟動 | 裝載 | 資訊 | 1 |
| 23075 | ContainerDeactivated | 容器已停止 | 裝載 | 資訊 | 1 |
| 23082 | ContainerExited | 容器已結束 - 查看 UnexpectedTermination 旗標 | 裝載 | 資訊 | 1 |

## <a name="health-reports"></a>健康狀態報告

**叢集健康情況報表事件**

| EventId | 名稱 | 說明 |來源 (工作) | Level | 版本 |
| --- | --- | --- | --- | --- | --- |
| 54428 | ClusterNewHealthReport | 有新的叢集健康情況報告可用 | HM | 資訊 | 1 |
| 54437 | ClusterHealthReportExpired | 現有的叢集健康情況報告已過期 | HM | 資訊 | 1 |

**節點健康情況報表事件**

| EventId | 名稱 | 說明 |來源 (工作) | Level | 版本 |
| --- | --- | ---| --- | --- | --- |
| 54423 | NodeNewHealthReport | 有新的節點健康情況報告可用 | HM | 資訊 | 1 |
| 54432 | NodeHealthReportExpired | 現有的節點健康情況報告已過期 | HM | 資訊 | 1 |

**應用程式健康情況報表事件**

| EventId | 名稱 | 說明 |來源 (工作) | Level | 版本 |
| --- | --- | ---| --- | --- | --- |
| 54425 | ApplicationNewHealthReport | 已建立新的應用程式健康情況報告。 這適用於已解除部署的應用程式。 | HM | 資訊 | 1 |
| 54426 | DeployedApplicationNewHealthReport | 已建立新的已部署應用程式健康情況報告 | HM | 資訊 | 1 |
| 54427 | DeployedServicePackageNewHealthReport | 已建立新的已部署服務健康情況報告 | HM | 資訊 | 1 |
| 54434 | ApplicationHealthReportExpired | 現有的應用程式健康情況報告已過期 | HM | 資訊 | 1 |
| 54435 | DeployedApplicationHealthReportExpired | 現有的已部署應用程式健康情況報告已過期 | HM | 資訊 | 1 |
| 54436 | DeployedServicePackageHealthReportExpired | 現有的已部署服務健康情況報告已過期 | HM | 資訊 | 1 |

**服務健康情況報表事件**

| EventId | 名稱 | 說明 |來源 (工作) | Level | 版本 |
| --- | --- | ---| --- | --- | --- |
| 54424 | ServiceNewHealthReport | 已建立新的服務健康情況報告 | HM | 資訊 | 1 |
| 54433 | ServiceHealthReportExpired | 現有的服務健康情況報告已過期 | HM | 資訊 | 1 |

**分割健康情況報表事件**

| EventId | 名稱 | 說明 |來源 (工作) | Level | 版本 |
| --- | --- | ---| --- | --- | --- |
| 54422 | PartitionNewHealthReport | 已建立新的分割健康情況報告 | HM | 資訊 | 1 |
| 54431 | PartitionHealthReportExpired | 現有的分割健康情況報告已過期 | HM | 資訊 | 1 |

**複本健康情況報表事件**

| EventId | 名稱 | 說明 |來源 (工作) | Level | 版本 |
| --- | --- | ---| --- | --- | --- |
| 54429 | StatefulReplicaNewHealthReport | 已建立具狀態的複本健康情況報告 | HM | 資訊 | 1 |
| 54430 | StatelessInstanceNewHealthReport | 已建立新的具狀態執行個體健康情況報告 | HM | 資訊 | 1 |
| 54438 | StatefulReplicaHealthReportExpired | 現有的具狀態複本健康情況報告已過期 | HM | 資訊 | 1 |
| 54439 | StatelessInstanceHealthReportExpired | 現有的無狀態執行個體健康情況報告已過期 | HM | 資訊 | 1 |

## <a name="chaos-testing-events"></a>混沌測試事件 

**混沌工作階段事件**

| EventId | 名稱 | 說明 |來源 (工作) | Level | 版本 |
| --- | --- | ---| --- | --- | --- |
| 50021 | ChaosStarted | 已啟動混沌測試工作階段 | 可測試性 | 資訊 | 1 |
| 50023 | ChaosStopped | 已停止混沌測試工作階段 | 可測試性 | 資訊 | 1 |

**混沌節點事件**

| EventId | 名稱 | 說明 |來源 (工作) | Level | 版本 |
| --- | --- | ---| --- | --- | --- |
| 50033 | ChaosNodeRestartScheduled | 已排定將節點重新啟動為混沌測試工作階段的一部分 | 可測試性 | 資訊 | 1 |
| 50087 | ChaosNodeRestartCompleted | 已完成將節點重新啟動為混沌測試工作階段的一部分 | 可測試性 | 資訊 | 1 |

**混沌應用程式事件**

| EventId | 名稱 | 說明 |來源 (工作) | Level | 版本 |
| --- | --- | ---| --- | --- | --- |
| 50053 | ChaosCodePackageRestartScheduled | 已在混沌測試工作階段期間排定程式碼套件重新啟動 | 可測試性 | 資訊 | 1 |
| 50101 | ChaosCodePackageRestartCompleted | 已在混沌測試工作階段期間完成程式碼套件重新啟動 | 可測試性 | 資訊 | 1 |

**混沌分割事件**

| EventId | 名稱 | 說明 |來源 (工作) | Level | 版本 |
| --- | --- | ---| --- | --- | --- |
| 50069 | ChaosPartitionPrimaryMoveScheduled | 已排定將主要磁碟分割移為混沌測試工作階段的一部分 | 可測試性 | 資訊 | 1 |
| 50077 | ChaosPartitionSecondaryMoveScheduled | 已排定將次要磁碟分割移為混沌測試工作階段的一部分 | 可測試性 | 資訊 | 1 |
| 65003 | PartitionPrimaryMoveAnalysis | 已可更深入地分析主要磁碟分割的移動 | 可測試性 | 資訊 | 1 |

**混沌複本事件**

| EventId | 名稱 | 說明 |來源 (工作) | Level | 版本 |
| --- | --- | ---| --- | --- | --- |
| 50047 | ChaosReplicaRestartScheduled | 已將複本重新啟動排定為混沌測試工作階段的一部份 | 可測試性 | 資訊 | 1 |
| 50051 | ChaosReplicaRemovalScheduled | 已將複本移除排定為混沌測試工作階段的一部份 | 可測試性 | 資訊 | 1 |
| 50093 | ChaosReplicaRemovalCompleted | 已完成將複本移除作為混沌測試工作階段的一部份 | 可測試性 | 資訊 | 1 |

## <a name="other-events"></a>其他事件

**相互關連事件**

| EventId | 名稱 | 說明 |來源 (工作) | Level | 版本 |
| --- | --- | ---| --- | --- | --- |
| 65011 | CorrelationOperational | 已偵測到相互關聯 | 可測試性 | 資訊 | 1 |

## <a name="events-prior-to-version-62"></a>6.2 版之前的事件

以下是 Service Fabric 6.2 版之前所提供事件的完整清單。

| EventId | 名稱 | 來源 (工作) | 等級 |
| --- | --- | --- | --- |
| 25620 | NodeOpening | FabricNode | 資訊 |
| 25621 | NodeOpenedSuccess | FabricNode | 資訊 |
| 25622 | NodeOpenedFailed | FabricNode | 資訊 |
| 25623 | NodeClosing | FabricNode | 資訊 |
| 25624 | NodeClosed | FabricNode | 資訊 |
| 25625 | NodeAborting | FabricNode | 資訊 |
| 25626 | NodeAborted | FabricNode | 資訊 |
| 29627 | ClusterUpgradeStart | CM | 資訊 |
| 29628 | ClusterUpgradeComplete | CM | 資訊 |
| 29629 | ClusterUpgradeRollback | CM | 資訊 |
| 29630 | ClusterUpgradeRollbackComplete | CM | 資訊 |
| 29631 | ClusterUpgradeDomainComplete | CM | 資訊 |
| 23074 | ContainerActivated | 裝載 | 資訊 |
| 23075 | ContainerDeactivated | 裝載 | 資訊 |
| 29620 | ApplicationCreated | CM | 資訊 |
| 29621 | ApplicationUpgradeStart | CM | 資訊 |
| 29622 | ApplicationUpgradeComplete | CM | 資訊 |
| 29623 | ApplicationUpgradeRollback | CM | 資訊 |
| 29624 | ApplicationUpgradeRollbackComplete | CM | 資訊 |
| 29625 | ApplicationDeleted | CM | 資訊 |
| 29626 | ApplicationUpgradeDomainComplete | CM | 資訊 |
| 18566 | ServiceCreated | FM | 資訊 |
| 18567 | ServiceDeleted | FM | 資訊 |

## <a name="next-steps"></a>後續步驟

* 深入了解 Service Fabric 中整個[平台層級的事件產生](service-fabric-diagnostics-event-generation-infra.md)
* 修改您的 [Azure 診斷](service-fabric-diagnostics-event-aggregation-wad.md)設定以收集更多記錄
* [設定 Application Insights](service-fabric-diagnostics-event-analysis-appinsights.md) 以查看您的作業通道記錄
