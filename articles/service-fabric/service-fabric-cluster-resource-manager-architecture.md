---
title: Resource Manager 架構 | Microsoft Docs
description: Service Fabric 叢集資源管理員的架構概觀。
services: service-fabric
documentationcenter: .net
author: masnider
manager: timlt
editor: ''
ms.assetid: 6c4421f9-834b-450c-939f-1cb4ff456b9b
ms.service: Service-Fabric
ms.devlang: dotnet
ms.topic: conceptual
ms.tgt_pltfrm: NA
ms.workload: NA
ms.date: 08/18/2017
ms.author: masnider
ms.openlocfilehash: 48da92be0eef1154b490fb4829363598d6d66569
ms.sourcegitcommit: eb75f177fc59d90b1b667afcfe64ac51936e2638
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 05/16/2018
ms.locfileid: "34211424"
---
# <a name="cluster-resource-manager-architecture-overview"></a>叢集資源管理員架構概觀
Service Fabric 叢集資源管理員是在叢集中執行的中央服務。 它會管理叢集中服務的所需狀態，特別是關於資源耗用和任何放置規則。 

若要管理叢集中的資源，Service Fabric 叢集資源管理員必須具備幾個部分的資訊：

- 目前有哪些服務
- 每個服務的目前 (或預設) 資源耗用量 
- 剩餘叢集容量 
- 叢集中節點的容量 
- 每個節點上所耗用的資源數量

特定服務的資源耗用量會隨時間而改變，服務通常會關心一種以上的資源。 在不同的服務之間，可能會同時測量真正實物和實體資源。 服務可能追蹤實體計量，例如記憶體和磁碟耗用量。 服務可能較關心邏輯計量，例如 "WorkQueueDepth" 或 "TotalRequests"。 在相同叢集中可以使用邏輯與實體計量。 計量可以跨多個服務共用，或是限定於特定服務。

## <a name="other-considerations"></a>其他考量
叢集的擁有者和操作員可以與服務和應用程式作者不同，或至少是同一人身兼多職。 當您開發應用程式時，您知道它所需的一些事項。 您估計了將會耗用的資源，以及服務的部署方式有何不同。 例如，Web 層需要在公開到網際網路的節點上執行，而資料庫服務則不應該這麼做。 另舉一例，Web 服務可能會受到 CPU 和網路限制，而資料層服務則比較在意記憶體和磁碟耗用量。 但是，在生產環境中處理該服務現場情況的人，或管理服務升級的人，有不同的作業，需要不同的工具。 

叢集和服務都是動態的：

- 在叢集中的節點數目可能增加和縮減
- 不同大小和類型的節點可能轉瞬即逝
- 服務可能建立、移除和變更所需的資源配置和放置規則
- 升級或其他管理作業可以在基礎結構層級上應用程式的叢集內展開
- 隨時都可能發生問題。

## <a name="cluster-resource-manager-components-and-data-flow"></a>叢集資源管理員元件和資料流程
叢集資源管理員必須追蹤每個服務的需求，以及這些服務中每個服務物件的資源耗用量。 叢集資源管理員有兩個概念性元件︰在每個節點上執行的代理程式和一個容錯服務。 每個節點上的代理程式會追蹤服務的負載報告、將報告彙總，並定期回報。 叢集資源管理員服務會彙總來自本機代理程式的資訊，並根據目前設定做出反應。

讓我們看看下圖︰

<center>
![資源平衡器架構][Image1]
</center>

執行階段可能發生許多變化。 例如，假設某些服務耗用的資源數量改變、某些服務失敗，以及某些節點加入和離開叢集。 節點上的所有變更會彙總，並定期傳送到叢集資源管理員服務 (1，2)，它們會在其中再次彙總、分析及儲存。 每隔幾秒鐘，服務就會查看變更，並判斷是否需要採取任何動作 (3)。 例如，它可能會注意到某些空的節點已新增至叢集。 如此一來，它會決定要將某些服務移至這些節點。 叢集資源管理員可能也會注意到特定節點是超載的，或者某些服務已失敗或刪除，而在別處釋放資源。

讓我們看看以下圖表，並看看接下來會發生什麼情況。 假設叢集資源管理員判斷需要變更。 它會與其他系統服務 (尤其是容錯移轉管理員) 進行協調，以進行必要的變更。 接著將必要的命令傳送至適當的節點 (4)。 例如，假設資源管理員注意到節點 5 已超載，因此決定要將服務 B 從節點 5 移至節點 4。 重新設定 (5) 結束時，叢集看起來像這樣︰

<center>
![資源平衡器架構][Image2]
</center>

## <a name="next-steps"></a>後續步驟
- 叢集資源管理員有許多描述叢集的選項。 若要深入了解這些選項，請參閱關於[描述 Service Fabric 叢集](./service-fabric-cluster-resource-manager-cluster-description.md)一文
- 叢集資源管理員的主要責任是重新平衡叢集，以及強制執行放置規則。 如需有關設定這些行為的詳細資訊，請參閱[平衡 Service Fabric 叢集](./service-fabric-cluster-resource-manager-balancing.md)

[Image1]:./media/service-fabric-cluster-resource-manager-architecture/Service-Fabric-Resource-Manager-Architecture-Activity-1.png
[Image2]:./media/service-fabric-cluster-resource-manager-architecture/Service-Fabric-Resource-Manager-Architecture-Activity-2.png
