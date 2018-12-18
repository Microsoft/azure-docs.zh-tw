---
title: Azure Service Fabric 中的複本與執行個體 | Microsoft Docs
description: 了解複本和執行個體 - 它們的功能及生命週期
services: service-fabric
documentationcenter: .net
author: appi101
manager: anuragg
editor: ''
ms.assetid: d5ab75ff-98b9-4573-a2e5-7f5ab288157a
ms.service: service-fabric
ms.devlang: dotnet
ms.topic: conceptual
ms.tgt_pltfrm: NA
ms.workload: NA
ms.date: 01/10/2018
ms.author: aprameyr
ms.openlocfilehash: 7f8638365b40395a5dd82457c40e5c15209ba1a7
ms.sourcegitcommit: eb75f177fc59d90b1b667afcfe64ac51936e2638
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 05/16/2018
ms.locfileid: "34211383"
---
# <a name="replicas-and-instances"></a>複本和執行個體 
本文概述具狀態服務之複本及無狀態服務之執行個體的生命週期。

## <a name="instances-of-stateless-services"></a>無狀態服務的執行個體
無狀態服務的執行個體是在叢集其中一個節點上執行之服務邏輯的複本。 分割區內的執行個體以其 **InstanceId** 作為唯一識別碼。 下圖為執行個體生命週期的模型：

![執行個體生命週期](./media/service-fabric-concepts-replica-lifecycle/instance.png)

### <a name="inbuild-ib"></a>建置中 (IB)
叢集資源管理員決定執行個體的位置後，它就會進入此生命週期的狀態。 執行個體會在節點上啟動。 應用程式主機啟動時，會建立及開啟執行個體。 啟動完成後，執行個體會轉換成就緒狀態。 

如果應用程式主機或此執行個體的節點當機，便會轉換成已卸除的狀態。

### <a name="ready-rd"></a>就緒 (RD)
在就緒狀態中，執行個體在節點上已啟動並執行中。 如果這個執行個體是可靠服務，便已叫用 **RunAsync**。 

如果應用程式主機或此執行個體的節點當機，便會轉換成已卸除的狀態。

### <a name="closing-cl"></a>關閉中 (CL)
在關閉中的狀態，Azure Service Fabric 正在關閉此節點上執行個體的流程。 有許多種原因會造成關閉的狀況，例如應用程式升級、負載平衡或正在刪除服務。 關閉完成之後，它會轉換成已卸除狀態。

### <a name="dropped-dd"></a>已卸除 (DD)
在已卸除狀態中，執行個體不會於節點上執行。 此時，Service Fabric 會維護此執行個體相關的中繼資料 (最終也會刪除)。

> [!NOTE]
> 使用 `Remove-ServiceFabricReplica` 中的 **ForceRemove** 選項，便可能由任何狀態轉換為已卸除狀態。
>

## <a name="replicas-of-stateful-services"></a>具狀態服務的複本
具狀態服務的複本是在叢集其中一個節點上執行之服務邏輯的複本。 此外，此複本會保留一份服務狀態的複本。 有兩個相關的概念描述具狀態複本的生命週期和行為：
- 複本生命週期
- 複本角色

以下討論描述持續性具狀態服務。 對於揮發性 (或記憶體內部) 具狀態服務，關閉狀態等同於已卸除狀態。

![複本生命週期](./media/service-fabric-concepts-replica-lifecycle/replica.png)

### <a name="inbuild-ib"></a>建置中 (IB)
建置中複本是正在建立中或已準備好加入複本集的複本。 視複本角色而定，IB 會具有不同的語意。 

如果建置中複本的應用程式主機或節點損毀，則會轉換成關閉狀態。

   - **主要建置中複本**：主要建置中複本是分割區的第一個複本。 這個複本通常發生分割區正在建立時。 當分割區所有的複本重新啟動或已卸除時，也會引發主要建置中複本。

   - **IdleSecondary 建置中複本**：這些是叢集資源管理員建立的新複本，或是已關閉且需要新增回複本集的現有複本。 這些複本在由主要複本植入或建置後，才能夠加入複本集作為 ActiveSecondary，並參與作業的仲裁認知。

   - **ActiveSecondary 建置中複本**：這個狀態能在某些查詢中觀察到。 它是複本集未變更，但必須建置複本的最佳化狀況。 複本本身依照正常的狀態機器轉換 (如複本角色一節中所述)。

### <a name="ready-rd"></a>就緒 (RD)
就緒複本是正在參與複寫與作業仲裁認知的複本。 就緒狀態適用於主要複本和使用中次要複本。

如果就緒複本的應用程式主機或節點損毀，則會轉換成關閉狀態。

### <a name="closing-cl"></a>關閉中 (CL)
複本在下列情況下進入關閉中狀態：

- **正在關閉複本的程式碼**：Service Fabric 可能需要關閉複本的執行中程式碼。 關閉的原因很多。 例如，可能是因為應用程式、網狀架構或基礎結構升級時，或是因為複本所報告的錯誤。 當複本關閉完成時，複本會轉換成關閉狀態。 與儲存在磁碟上的這個複本相關聯的持續性狀態不會清除。

- **將複本從叢集中移除**：Service Fabric 可能需要移除持續性狀態，並關閉複本執行中的程式碼。 關閉的原因有很多，如負載平衡。

### <a name="dropped-dd"></a>已卸除 (DD)
在已卸除狀態中，執行個體不會於節點上執行。 在節點上也沒有留下狀態。 此時，Service Fabric 會維護此執行個體相關的中繼資料 (最終也會刪除)。

### <a name="down-d"></a>關閉 (D)
在關閉狀態中，複本程式碼未執行，但複本的持續性狀態會存在於該節點上。 有許多原因會造成複本關閉，例如節點關閉中、複本程式碼損毀、應用程式升級、複本錯誤等。

Service Fabric 會視需要開啟關閉的複本，例如在節點上完成升級時等等。

複本角色在關閉狀態中是不相關的。

### <a name="opening-op"></a>開啟中 (OP)
當 Service Fabric 必須將複本再次啟動時，關閉的複本會進入開啟中狀態。 例如，這個狀態可能在應用程式在節點上完成程式碼升級後發生。 

如果開啟中複本的應用程式主機或節點損毀，則會轉換成關閉狀態。

複本角色在開啟中狀態中是不相關的。

### <a name="standby-sb"></a>待命 (SB)
待命複本是已關閉然後開啟之持續性服務的複本。 Service Fabric 如果必須將另一個複本新增至複本集時，會使用此複本 (因為複本已經有狀態的一部分，可加速建置流程)。 StandByReplicaKeepDuration 到期之後，會捨棄待命複本。

如果待命複本的應用程式主機或節點損毀，則會轉換成關閉狀態。

複本角色在待機狀態中是不相關的。

> [!NOTE]
> 任何未關閉或卸除的複本會視為*啟動*。
>

> [!NOTE]
> 使用 `Remove-ServiceFabricReplica` 中的 **ForceRemove** 選項，便可能由任何狀態轉換為已卸除狀態。
>

## <a name="replica-role"></a>複本角色 
複本角色可決定其在複本集中的功能：

- **主要 (P)**：複本集中會有一個主要複本負責執行讀取與寫入作業。 
- **ActiveSecondary (S)**：這些是從主要複本接收狀態更新、套用更新並傳送認知的複本。 複本集內有多個作用中次要複本。 這些次要複本的數目會決定服務可處理的錯誤數目。
- **IdleSecondary (I)**：這些複本正由主要複本建立中。 它們需要先接收來自主要複本的狀態，才能夠升階至作用中次要複本。 
- **無 (N)**：這些複本在複本集沒有責任。
- **未知 (U)**：這是複本在從 Service Fabric 接收任何 **ChangeRole** API 呼叫前的初始角色。

下圖說明複本角色轉換及一些轉換會發生的範例情節：

![複本角色](./media/service-fabric-concepts-replica-lifecycle/role.png)

- U -> P：建立新的主要複本。
- U -> I：建立新的閒置複本。
- U -> N：刪除待命複本。
- I -> S：將閒置次要複本升階為作用中次要複本，讓其認知對仲裁有貢獻。
- I -> P：將閒置次要複本升階為主要複本。 這會在閒置次要複本為主要複本的正確候選對象時，於特殊重新設定下發生。
- I -> N：刪除閒置次要複本。
- I -> P：將作用中次要複本升階為主要複本。 這可能是因為叢集資源管理員起始的主要複本容錯移轉或主要複本移動。 例如，為了回應應用程式升級或負載平衡等情況。
- S -> N：刪除作用中次要複本。
- P -> S：將主要複本降級。 這可能是因為叢集資源管理員起始的主要複本移動。 例如，為了回應應用程式升級或負載平衡等情況。
- P -> N：刪除主要複本。

> [!NOTE]
> 更高階的程式設計模型 (例如 [Reliable Actors](service-fabric-reliable-actors-introduction.md) 和 [Reliable Services](service-fabric-reliable-services-introduction.md)) 會對開發人員隱藏複本角色的概念。 在 Actors 中，角色的概念並非必要。 在 Services 中，大多數的案例均會予以簡化。
>

## <a name="next-steps"></a>後續步驟
如需有關 Service Fabric 概念的詳細資訊，請參閱下列文章：

[Reliable Services 生命週期 - C#](service-fabric-reliable-services-lifecycle.md)

