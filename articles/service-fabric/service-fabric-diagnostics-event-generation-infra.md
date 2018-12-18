---
title: Azure Service Fabric 平台層級監視 | Microsoft Docs
description: 了解用來監視和診斷 Azure Service Fabric 叢集的平台層級事件和記錄。
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
ms.openlocfilehash: 31f23e3f8e792c6b61870c640f99ec3392a940d3
ms.sourcegitcommit: eb75f177fc59d90b1b667afcfe64ac51936e2638
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 05/16/2018
ms.locfileid: "34211169"
---
# <a name="monitoring-the-cluster-and-platform"></a>監視叢集和平台

請務必在平台層級進行監視，以判斷硬體和叢集是否如預期運作。 雖然 Service Fabric 可在硬體故障時讓應用程式繼續執行，但是您仍需要診斷錯誤是發生在應用程式中，還是發生在基礎結構中。 您也應該監視叢集，更妥善地規劃容量，以協助決定如何新增或移除硬體。

透過 EventStore 和各種現成的記錄通道，Service Fabric 能公開好幾種結構化的平台事件 (稱為 [Service Fabric 事件](service-fabric-diagnostics-events.md))。 

EventStore 能讓您以實體為單位存取叢集事件 (實體包括叢集、節點、應用程式、服務、分割區、複本及容器)，並且透過 REST API 和 Service Fabric 用戶端程式庫公開事件。 請使用 EventStore 來監視開發/測試叢集，以及了解生產叢集在特定時間點的狀態。 若要深入瞭解此資訊，請參閱 [EventStore 概觀](service-fabric-diagnostics-eventstore.md)。

Service Fabric 另提供以下現成記錄通道，供您設定生產叢集的監視管線：

* [**作業**](service-fabric-diagnostics-event-generation-operational.md)  
由 Service Fabric 與叢集執行的高階作業，包括即將進行的節點、正在部署的新應用程式，或升級復原等事件。

* **作業 - 詳細**  
健康情況報告和負載平衡決策。

* **資料和傳訊**  
在傳訊 (目前僅限 ReverseProxy) 和資料路徑 (可靠的服務模型) 中產生的重要記錄和事件。

* **資料和傳訊 - 詳細**  
詳細資訊通道，其中包含叢集中資料和傳訊的所有非重要記錄 (此通道有非常大量的事件)。

除此以外，還提供了兩個結構化 EventSource 通道，以及我們為了支援而收集的記錄。

* [Reliable Services 事件](service-fabric-reliable-services-diagnostics.md)  
程式設計模型特定事件。

* [Reliable Actors 事件](service-fabric-reliable-actors-diagnostics.md)  
程式設計模型特定的事件和效能計數器。

* 支援記錄  
由 Service Fabric 產生的系統記錄僅供我們在提供支援時使用。

這些各種通道涵蓋建議的大部分平台層級記錄。 若要改善平台層級記錄，請考慮深入了解健康情況模型並新增自訂健康情況報告，以及新增自訂**效能計數器**來即時了解對叢集上服務和應用程式的影響。

若要充分利用這些記錄，強烈建議在建立叢集期間啟用「診斷」。 透過開啟診斷，在部署叢集時，Windows Azure 診斷可以認可 Operational、Reliable Services 和 Reliable actors 通道，並儲存資料，如[使用 Azure 診斷彙總事件](service-fabric-diagnostics-event-aggregation-wad.md)的進一步說明。

## <a name="azure-service-fabric-health-and-load-reporting"></a>Azure Service Fabric 健全狀況和負載報告

Service Fabric 有自己的健全狀況模型，詳述於下列文件：

- [Service Fabric 健全狀況監視簡介](service-fabric-health-introduction.md)
- [回報和檢查服務健全狀況](service-fabric-diagnostics-how-to-report-and-check-service-health.md)
- [新增自訂 Service Fabric 健康狀態報告](service-fabric-report-health.md)
- [檢視 Service Fabric 健康狀態報告](service-fabric-view-entities-aggregated-health.md)

對於服務運作的許多層面而言，健全狀況監視不可或缺。 當 Service Fabric 執行具名的應用程式升級時，健全狀況監視尤其重要。 當服務的每個升級網域完成升級並提供給客戶之後，升級網域必須通過健全狀況檢查，才能繼續部署至下一個升級網域。 如果無法達到良好的健全狀態，部署就會復原，讓應用程式保持已知的良好狀態。 雖然在服務復原之前會影響一些客戶，但大部分客戶都不會遇到問題。 此外，很快就會開始解決，無需等待操作人員採取行動。 在程式碼中納入越多健全狀況檢查，服務越能夠從部署問題中恢復。

服務健全狀況的另一個層面是來自服務的報告計量。 計量是用來平衡資源使用量，在 Service Fabric 中很重要。 計量也可以當做系統健全狀況的指標。 例如，您的應用程式可能有許多服務，而每個執行個體會報告每秒要求數 (RPS) 計量。 如果有一項服務比其他服務使用更多資源，Service Fabric 會在叢集內移動服務執行個體，嘗試維護平衡的資源使用量。 如需資源耗用量運作方式的詳細說明，請參閱[在 Service Fabric 中使用計量管理資源耗用量和負載](service-fabric-cluster-resource-manager-metrics.md)。

計量也可協助您深入探索服務的運作方式。 經過一段時間，您可以使用計量，檢查服務是否在預期的參數內運作。 比方說，如果趨勢指出星期一早上 9 點的平均 RPS 是 1,000，您可能將健全狀況報告設定為當 RPS 低於 500 或高於 1,500 時發出警示。 一切可能都沒問題，但您可能需要檢查一下，確保客戶擁有絕佳的體驗。 您的服務可以定義一組計量，供健全狀況檢查時報告，但又不影響叢集的資源平衡。 若要這樣做，請將計量權數設為零。 我們建議您在開始時將所有計量的加權設為零，並且在確定您了解計量加權會如何影響叢集的資源平衡前，都不要提高權數。

> [!TIP]
> 請勿使用太多的加權計量。 很難了解服務執行個體為了平衡而移動的原因。 少數幾個計量就很夠用！

任何能夠指出應用程式健全狀況和效能的資訊，都適合納入計量和健全狀況報告中。 CPU 效能計數器可以告訴您節點的使用情況，但無法讓您知道某個特定服務是否健康，因為可能有多項服務在單一節點上執行。 不過，像是 RPS、處理的項目和求延遲等計量，都可以指出特定服務的健全狀況。

## <a name="service-fabric-support-logs"></a>Service Fabric 支援記錄

如果您需要連絡 Microsoft 支援服務以協助處理您的 Azure Service Fabric 叢集，通常都需要提供支援記錄。 如果您的叢集裝載於 Azure，建立叢集時會自動設定和收集支援記錄。 記錄會儲存在叢集資源群組中的專用儲存體帳戶。 儲存體帳戶沒有固定名稱，但您會在帳戶中看到名稱以 fabric 開頭的 blob 容器和資料表。 如需有關為獨立叢集設定記錄收集的資訊，請參閱[建立和管理獨立 Azure Service Fabric 叢集](service-fabric-cluster-creation-for-windows-server.md)和[獨立 Windows 叢集的組態設定](service-fabric-cluster-manifest.md)。 若為獨立 Service Fabric 執行個體，記錄應該傳送至本機檔案共用。 您**需要**擁有這些記錄才能取得支援，而這些記錄只限 Microsoft 客戶支援小組使用。

## <a name="measuring-performance"></a>測量效能

叢集的測量效能將協助您了解如何處理負載，以及進行調整叢集的決策 (請深入查看調整 [Azure 上](service-fabric-cluster-scale-up-down.md)或[內部部署](service-fabric-cluster-windows-server-add-remove-nodes.md)上的叢集)。 相較於您或應用程式和服務可能已採取的動作，在未來分析記錄時，效能資料也十分有用。 

如需使用 Service Fabric 時要收集的效能計數器清單，請參閱 [Service Fabric 中的效能計數器](service-fabric-diagnostics-event-generation-perf.md)。

以下是您可以設定收集叢集效能資料的兩個常用方式：

* **使用代理程式**  
這是從電腦收集效能的慣用方法，因為代理程式通常會有一份可收集的效能計量清單，而且選擇您要收集或變更的計量是相當簡單的程序。 請閱讀[如何設定 Service Fabric 的 OMS](service-fabric-diagnostics-event-analysis-oms.md) 和[設定 OMS Windows 代理程式](../log-analytics/log-analytics-windows-agent.md)文章以深入了解 OMS 代理程式，而這類監視代理程式可以挑選叢集 VM 和已部署容器的效能資料。

* **設定診斷以將效能計數器寫入資料表中**  
針對 Azure 上的叢集，這表示變更 Azure 診斷設定來反映叢集中 VM 的適當效能計數器，並在您要部署任何容器時，讓它反映 Docker 統計資料。 請閱讀在 Service Fabric 中設定 [WAD 中的效能計數器](service-fabric-diagnostics-event-aggregation-wad.md)，以設定效能計數器集合。

## <a name="next-steps"></a>後續步驟

需要先彙總記錄和事件，才能將它們傳送到任何分析平台。 請閱讀 [EventFlow](service-fabric-diagnostics-event-aggregation-eventflow.md) 和 [WAD](service-fabric-diagnostics-event-aggregation-wad.md) 以深入了解一些建議的選項。
