---
title: Service Fabric 叢集 Resource Manager 簡介 | Microsoft Docs
description: Service Fabric 叢集資源管理員簡介。
services: service-fabric
documentationcenter: .net
author: masnider
manager: timlt
editor: ''
ms.assetid: cfab735b-923d-4246-a2a8-220d4f4e0c64
ms.service: Service-Fabric
ms.devlang: dotnet
ms.topic: conceptual
ms.tgt_pltfrm: NA
ms.workload: NA
ms.date: 08/18/2017
ms.author: masnider
ms.openlocfilehash: f25a422385abfcdb7020eb7477c0ae2ee55cd8fb
ms.sourcegitcommit: eb75f177fc59d90b1b667afcfe64ac51936e2638
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 05/16/2018
ms.locfileid: "34210591"
---
# <a name="introducing-the-service-fabric-cluster-resource-manager"></a>Service Fabric 叢集資源管理員簡介
以傳統方式管理 IT 系統或線上服務，意味著要讓特定實體或虛擬機器專屬於這些特定的服務或系統。 服務已建構為層次。 有「Web」層和「資料」或「儲存體」層。 應用程式會有訊息層，要求在其中流入或流出，以及專用於快取的一組機器。 每個層次或工作負載類型都有專屬的特定機器︰資料庫會取得一些專屬的機器，Web 伺服器也會取得一些。 如果特定類型的工作負載造成執行它的機器太忙碌，請以該層次的相同設定新增更多機器。 不過，並非所有工作負載都可以如此輕易地相應放大 - 特別是您通常會以較大型機器來取代機器的資料層。 簡單。 如果某台電腦失敗，則在還原該電腦之前，整體應用程式中的那一個部分會以較低容量來執行。 仍然相當簡單 (但不一定有趣)。

然而現在服務和軟體架構的世界已經變更。 應用程式採用相應放大設計更加常見。 使用容器或微服務 (或兩者) 來建置應用程式很常見。 現在，您可能仍然只有少數機器，它們不只執行工作負載的單一執行個體。 它們甚至可能同時執行多個不同的工作負載。 您現在有很多不同類型的服務 (但都沒有充分利用整部機器的資源)，這些服務可能有數百個不同的執行個體。 每個具名執行個體有一或多個執行個體或複本來支援高可用性 (HA)。 根據這些工作負載的大小，以及它們的忙碌程度，您可能會發現自己具有數百部或數千部機器。 

突然間，管理您的環境並不像管理一些專屬於單一類型工作負載的電腦一樣簡單。 您的伺服器是虛擬且不再擁有名稱 (畢竟您的心態已從[寵物到牛群](http://www.slideshare.net/randybias/architectures-for-open-and-scalable-clouds/20)完全轉換)。 關於電腦的設定會變少，而關於服務本身的設定會增加。 專用於工作負載單一執行個體的硬體已經是過去的產物。 服務本身已經變成小型分散式系統，跨越多個較小型商用硬體。

因為您的應用程式不再是分散到數個層的巨石陣，您現在有更多組合可以運用。 由誰決定什麼類型的工作負載可在哪些硬體上執行，或可執行多少工作負載？ 哪些工作負載可在相同的硬體上運作得更好，以及哪些會發生衝突？ 當機器關機時，您如何知道該機器上正在執行什麼項目？ 由誰負責確保工作負載會再次開始執行？ 您要等待 (虛擬？) 電腦恢復正常運作，或者您的工作負載會自動容錯移轉至其他電腦並且持續執行？ 是否需要使用者介入？ 在這個環境中升級？

當開發人員和操作員在這個環境中作業時，我們想要一些協助來解決這種複雜性。 招募更多人來嘗試掩飾複雜性並非正確解答，那麼，我們應該怎麼做？

## <a name="introducing-orchestrators"></a>Orchestrator 簡介
「Orchestrator」是一種軟體的一般用詞，可協助系統管理員管理這些類型的環境。 Orchestrator 是可接受像「我想要在環境中執行此服務的五個複本」這種要求的元件。 無論發生什麼事，它嘗試讓環境符合所需的狀態。

Orchestrator (不是人類) 是當機器失敗或工作負載基於某些意外因素而終止時，要迅速採取動作的事物。 大部分 Orchestrator 不是只處理失敗而已。 其他功能還包括管理新的部署、處理升級，以及處理資源耗用量和管理。 所有 Orchestrator 基本上會維護環境中理想的設定狀態。 您想要能夠告知 Orchestrator 您的期望，並讓它能夠執行繁重的工作。 以 Mesos、Docker Datacenter/Docker Swarm、Kubernetes 和 Service Fabric 為基礎的 Aurora，全都是 Orchestrator 的例子。 正在主動開發這些 Orchestrator，以符合生產環境中的實際工作負載需求。 

## <a name="orchestration-as-a-service"></a>協調流程即服務
叢集資源管理員是處理 Service Fabric 中協調流程的系統元件。 叢集資源管理員的工作可分成三個部分︰

1. 強制執行規則
2. 將您的環境最佳化
3. 協助其他程序

如需了解叢集資源管理員的運作方式，請觀賞下列 Microsoft Virtual Academy 影片︰<center><a target="_blank" href="https://mva.microsoft.com/en-US/training-courses/building-microservices-applications-on-azure-service-fabric-16747?l=d4tka66yC_5706218965">
<img src="./media/service-fabric-cluster-resource-manager-introduction/ConceptsAndDemoVid.png" WIDTH="360" HEIGHT="244">
</a></center>

### <a name="what-it-isnt"></a>哪些不是它的性質
在傳統的多層式架構應用程式中，總是有[負載平衡器](https://en.wikipedia.org/wiki/Load_balancing_(computing))。 這通常是網路負載平衡 (NLB) 或應用程式負載平衡器 (ALB)，根據它在網路堆疊中的位置而定。 某些負載平衡器是以硬體為基礎 (例如 F5 的 BigIP 供應項目)，有些則是以軟體為基礎 (例如 Microsoft 的 NLB)。 在其他環境中，您可能會在這個角色中看到像 HAProxy、nginx、Istio 或 Envoy 這樣的項目。 在這些架構中，負載平衡的工作是確保無狀態工作負載收到 (大約) 相同的工作量。 平衡負載策略各不相同。 某些平衡器會將每個不同的呼叫傳送至不同的伺服器。 有些則是提供工作階段關聯/綁定。 更進階的平衡器會使用實際負載估計或報告，根據預期的成本和目前的電腦工作負載來路由傳送呼叫。

網路平衡器或訊息路由器會嘗試確保 Web/背景工作角色層維持大致平衡。 平衡資料層的策略不相同，並且相依於資料存放庫。 平衡資料層依賴資料分區化、快取、受控檢視、預存程序，與其他存放區特定機制。

雖然這其中有些策略很有趣，但 Service Fabric 叢集資源管理員並非任何像是網路負載平衡器或快取的功能。 網路負載平衡器會藉由分散前端之間的流量來平衡前端。 Service Fabric 叢集資源管理員有不同的策略。 基本上，Service Fabric 會將「服務」移至最需要它們的地方，流量或負載會隨之轉向。 例如，可能將服務移至目前閒置的節點，因為那裡的服務未執行太多工作。 節點會閒置可能是因為原本存在的服務已刪除或移至別處。 再舉一個例子，叢集資源管理員也可能從電腦中移出服務。 可能是電腦即將升級，或因為其中執行的服務突然增加耗用量而負載過重。 或者，可能會增加服務的資源需求。 如此一來，此機器上的資源就不足以繼續執行。 

因為叢集資源管理員負責移動服務，相較於您在網路負載平衡器中發現的功能，它包含一組不同的功能。 這是因為網路負載平衡器將網路流量傳送到服務已在其中的位置，即使該位置對於執行服務本身來說並不理想。 Service Fabric 叢集資源管理員採用完全不同的策略，以確保有效率地利用叢集中的資源。

## <a name="next-steps"></a>後續步驟
- 如需叢集資源管理員內的架構和資訊流程的相關資訊，請參閱[這篇文章](service-fabric-cluster-resource-manager-architecture.md)
- 叢集資源管理員有許多描述叢集的選項。 若要深入了解計量，請參閱關於[描述 Service Fabric 叢集](service-fabric-cluster-resource-manager-cluster-description.md)一文
- 如需有關設定服務的詳細資訊，請參閱[深入了解設定服務](service-fabric-cluster-resource-manager-configure-services.md)(service-fabric-cluster-resource-manager-configure-services.md)
- 度量是 Service Fabric 叢集資源管理員管理叢集中的耗用量和容量的方式。 若要深入了解計量及其設定方式，請查看[這篇文章](service-fabric-cluster-resource-manager-metrics.md)
- 叢集資源管理員會搭配 Service Fabric 的管理功能使用。 若要深入了解該整合，請閱讀 [這篇文章](service-fabric-cluster-resource-manager-management-integration.md)
- 若要了解叢集資源管理員如何管理並平衡叢集中的負載，請查看關於 [平衡負載](service-fabric-cluster-resource-manager-balancing.md)
