---
title: Azure Service Fabric 叢集調整 | Microsoft Docs
description: 了解 Service Fabric 叢集的相應縮小、相應放大、相應增加或相應減少。
services: service-fabric
documentationcenter: .net
author: rwike77
manager: timlt
editor: aljo
ms.assetid: 5441e7e0-d842-4398-b060-8c9d34b07c48
ms.service: service-fabric
ms.devlang: dotnet
ms.topic: conceptual
ms.tgt_pltfrm: NA
ms.workload: NA
ms.date: 04/09/2018
ms.author: ryanwi
ms.openlocfilehash: f9e3f190decdc907cf57a0235b9d7142081bd2f1
ms.sourcegitcommit: eb75f177fc59d90b1b667afcfe64ac51936e2638
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 05/16/2018
ms.locfileid: "34208024"
---
# <a name="scaling-service-fabric-clusters"></a>調整 Service Fabric 叢集
Service Fabric 叢集是一組由網路連接的虛擬或實體機器，可用來將您的微服務部署到其中並進行管理。 屬於叢集一部分的機器或 VM 都稱為節點。 叢集有可能包含數千個節點。 在建立 Service Fabric 叢集之後，您可以水平調整叢集 (變更節點數目)，或以垂直方式調整 (變更節點的資源)。  您可以隨時調整叢集，即使正在叢集上執行工作負載，也是如此。  在叢集進行調整時，您的應用程式也會自動調整。

為什麼要調整叢集？ 應用程式需求會隨著時間而變更。  您可能需要增加叢集資源以因應增加的應用程式工作負載或網路流量，或是在需要下降時減少叢集資源。

### <a name="scaling-in-and-out-or-horizontal-scaling"></a>相應縮小和相應放大，或水平調整
變更叢集中的節點數目。  在新的節點加入叢集後，[叢集資源管理員](service-fabric-cluster-resource-manager-introduction.md)會將服務移至這些節點，而降低現有節點上的負載。  若叢集的資源未有效率地使用，您也可以減少節點數目。  當節點退出叢集時，服務即會從這些節點中移出，而其餘節點上的負載將會增加。  減少在 Azure 中執行之叢集中的節點數目可以節省成本，因為您只需對您所使用的 VM 數目付費，而不是對這些 VM 上的工作負載付費。  

- 優點：無限制的調整 (理論上)。  如果您的應用程式是針對延展性進行設計，您可以藉由新增更多節點而達到無限制的成長。  雲端環境中的工具使得節點的新增或移除更為簡便，因此您可以輕鬆地調整容量，且只需為使用的資源付費。  
- 缺點：應用程式必須[針對延展性進行設計](service-fabric-concepts-scalability.md)。  應用程式資料庫與持續性也可能需要仰賴更多架構工作來進行調整。  不過，Service Fabric 具狀態服務中的[可靠集合](service-fabric-reliable-services-reliable-collections.md)大幅降地了調整應用程式資料的難度。

### <a name="scaling-up-and-down-or-vertical-scaling"></a>相應增加和相應放大，或垂直調整 
變更叢集中節點的資源 (CPU、記憶體或儲存體)。
- 優點：軟體和應用程式架構保持不變。
- 缺點：有限的調整，因為您可以在個別節點上增加的資源有數目上的限制。 停機時間，因為您必須使實體或虛擬機器離線，才能新增或移除資源。

## <a name="scaling-an-azure-cluster-in-or-out"></a>將 Azure 叢集相應縮小或相應放大
虛擬機器擴展集是一個 Azure 計算資源，可以用來將一組虛擬機器當做一個集合加以部署和管理。 在 Azure 叢集中定義的每個節點類型，會[設定為不同的擴展集](service-fabric-cluster-nodetypes.md)。 然後每個節點類型可以獨立相應縮小或放大，可以開啟不同組的連接埠，並可以有不同的容量度量。 

在調整 Azure 叢集時，請記住下列指導方針︰
- 執行生產工作負載的主要節點類型，應一律具有五個或更多節點。
- 執行具狀態生產工作負載的非主要節點類型，應一律具有五個或更多節點。
- 執行無狀態生產工作負載的非主要節點類型，應一律具有兩個或更多節點。
- 任何具有金級或銀級[持久性層級](service-fabric-cluster-capacity.md#the-durability-characteristics-of-the-cluster)的節點類型，應一律具有五個或更多節點。
- 請勿從節點類型中移除隨機 VM 執行個體/節點，而應一律使用虛擬機器擴展集的相應減少功能。 刪除隨機 VM 執行個體可能會對系統進行負載平衡的能力造成負面影響。
- 如果使用自動調整規則，請適當設定規則，使系統逐一對節點進行相應縮小 (移除 VM 執行個體)。 一次相應減少超過一個執行個體並不安全。

由於叢集中的 Service Fabric 節點類型是由後端的虛擬機器擴展集所組成，因此您可以[設定自動調整規則或手動調整](service-fabric-cluster-scale-up-down.md)每個節點類型/虛擬機器擴展集。

### <a name="programmatic-scaling"></a>以程式設計方式調整
在許多案例中，[以手動方式或透過自動調整規則調整叢集](service-fabric-cluster-scale-up-down.md)都是不錯的解決方案。 但在更為進階的案例中，可能就不適用。 這些方法的可能缺點包括︰

- 手動調整需要登入系統，並明確地要求調整作業。 如果調整作業需要經常進行或難以預料會在何時進行，這個方法可能就不是合適的解決方案。
- 自動調整規則在從虛擬機器擴展集內移除執行個體時，並不會自動從相關聯的 Service Fabric 叢集移除對於該節點的認識，除非該節點類型的持久性等級為銀級或金級。 自動調整規則會作用在擴展集層級 (而非 Service Fabric 層級)，所以自動調整規則會直接移除 Service Fabric 節點，而未將其正常關閉。 以這種方式粗糙地移除節點，會在相應縮小作業完成後留下「準刪除」的 Service Fabric 節點狀態。 個人 (或服務) 必須定期清除 Service Fabric 叢集中的已移除節點狀態。
- 持久性等級為金級或銀級的節點類型會自動清除已移除的節點，因此不需要額外的清除動作。
- 雖然自動調整規則支援[許多計量](../monitoring-and-diagnostics/insights-autoscale-common-metrics.md)，但這組計量的數量仍然有限。 如果您的案例所需要的調整是以該組計量所未涵蓋的一些計量為基礎，則自動調整規則可能不是很好的選擇。

處理 Service Fabric 調整的方式取決於您的案例。 如果是不常見的調整，以手動方式新增或移除節點的能力或許就綽綽有餘。 若是更複雜的案例，能以程式設計方式進行調整的自動調整規則和 SDK 則可提供更強大的替代方案。

Azure API 的存在可讓應用程式以程式設計方式使用虛擬機器擴展集和 Service Fabric 叢集。 如果現有自動調整選項不適用於您的案例，這些 API 可讓您實作自訂調整邏輯。 

若要實作此「自製」自動調整功能，有一個方法是將新的無狀態服務新增至 Service Fabric 應用程式以管理調整作業。 建立自己的調整服務就能對應用程式的調整行為握有最高程度的控制力與自訂能力。 這很適合用於需要精確控制應用程式相應縮小或放大之時機或方式的案例。不過，伴隨此控制能力而來的是程式碼會變得複雜。 使用這種方式就表示您必須擁有調整程式碼 (此程式碼很複雜)。 在服務的 `RunAsync` 方法內，有一組觸發程序可以判斷是否需要調整 (包括檢查最大叢集大小和調整冷卻等參數)。   

用於虛擬機器擴展集互動 (檢查目前的虛擬機器執行個體數目並加以修改) 的 API 是 [Fluent Azure 管理計算程式庫](https://www.nuget.org/packages/Microsoft.Azure.Management.Compute.Fluent/)。 Fluent 計算程式庫可提供方便使用的 API 來與虛擬機器擴展集互動。  若要與 Service Fabric 叢集本身互動，請使用 [System.Fabric.FabricClient](/dotnet/api/system.fabric.fabricclient)。

不過，調整程式碼不需要以服務的形式在想要調整的叢集中執行。 `IAzure` 和 `FabricClient` 都能遠端連線到其相關聯的 Azure 資源，因此，調整服務可以輕鬆地成為主控台應用程式或從 Service Fabric 應用程式外部執行的 Windows 服務。

根據這些限制，您可能會想要[實作自訂能力更大的自動調整模型](service-fabric-cluster-programmatic-scaling.md)。


## <a name="scaling-a-standalone-cluster-in-or-out"></a>將獨立叢集相應縮小或相應放大
獨立叢集可讓您在內部部署 Service Fabric 叢集，或是在您選擇的雲端提供者中部署。  節點類型包含實體機器或虛擬機器，視您的部署而定。 相較於在 Azure 中執行的叢集，調整獨立叢集的程序會稍微複雜一些。  您必須手動變更叢集中的節點數目，然後執行叢集組態升級。

移除節點可能會起始多個升級作業。 有些節點會標示 `IsSeedNode=”true”` 標記，而可藉由使用 [Get-ServiceFabricClusterManifest](/powershell/module/servicefabric/get-servicefabricclustermanifest) 來查詢叢集資訊清單而加以識別。 移除這類節點所需的時間可能比移除其他節點長，因為在這類案例中，需要將種子節點四處移動。 叢集必須至少維持三個主要節點類型的節點。

在調整獨立叢集時，請記住下列指導方針︰
- 應以逐一取代主要節點的方式來執行，而不是以批次方式移除後再加入。
- 移除節點之前，請檢查是否有任何節點參考該節點類型。 請先移除這些節點，然後才移除對應的節點類型。 移除所有對應的節點之後，您便可以將該 NodeType 自叢集組態中移除，然後使用 [Start-ServiceFabricClusterConfigurationUpgrade](/powershell/module/servicefabric/start-servicefabricclusterconfigurationupgrade) 來開始組態升級。

如需詳細資訊，請參閱[調整獨立叢集](service-fabric-cluster-windows-server-add-remove-nodes.md)。

## <a name="scaling-an-azure-cluster-up-or-down"></a>將 Azure 叢集相應增加或相應減少
虛擬機器擴展集是一個 Azure 計算資源，可以用來將一組虛擬機器當做一個集合加以部署和管理。 在 Azure 叢集中定義的每個節點類型，會[設定為不同的擴展集](service-fabric-cluster-nodetypes.md)。 隨後，您即可個別管理每個節點類型。  要將一個節點類型相應增加或相應減少，必須在擴展集中變更虛擬機器執行個體的 SKU。 

> [!WARNING]
> 建議您不要變更擴展集/節點類型的 VM SKU，除非它以[銀級耐久性或更高級別](service-fabric-cluster-capacity.md#the-durability-characteristics-of-the-cluster)執行。 變更 VM SKU 大小是資料破壞性就地基礎結構作業。 如果沒有延遲或監視此變更的能力，作業可能會導致具狀態服務的資料遺失，或者甚至對於無狀態工作負載導致其他未預期作業問題。 
>

在調整 Azure 叢集時，請記住下列指導方針︰
- 如果相應減少主要節點類型，您絕對不應將其相應減少超過[可靠性層級](service-fabric-cluster-capacity.md#the-reliability-characteristics-of-the-cluster)允許的程度。

將節點類型相應增加或相應減少的程序，會根據它是非主要還是主要節點類型而有所不同。

### <a name="scaling-non-primary-node-types"></a>調整非主要節點類型
以您所需的資源建立新的節點類型。  更新執行中服務的放置條件約束，以包含新的節點類型。  請漸次 (一次一個) 將舊節點類型執行個體的執行個體計數縮減為零，使叢集的可靠性不受影響。  當舊的節點類型移除時，服務會逐漸移轉至新的節點類型。

### <a name="scaling-the-primary-node-type"></a>調整主要節點類型
建議您不要變更主要節點類型的 VM SKU。 如果您需要更多叢集容量，建議您新增更多執行個體。 

如果此做法不可行，您可以從舊叢集建立新的叢集並[還原應用程式狀態](service-fabric-reliable-services-backup-restore.md) (如果適用的話)。 您不需要還原任何系統服務狀態，它們會在您部署應用程式到新叢集時重新建立。 如果您之前只在叢集上執行無狀態應用程式，那麼您要做的只有部署應用程式到新叢集，不需還原任何東西。 如果您決定採取不支援的做法，並想要變更 VM SKU，請修改虛擬機器擴展集模型定義以反映新的 SKU。 如果您的叢集只有單一節點類型，請確保所有具狀態應用程式會及時回應所有[服務複本生命週期事件](service-fabric-reliable-services-lifecycle.md) (例如當組建中的複本陷入停滯)，且您的服務複本重建期間為少於五分鐘 (針對 Silver 持久性層級)。 

## <a name="next-steps"></a>後續步驟
* 深入了解[應用程式延展性](service-fabric-concepts-scalability.md)。
* [將 Azure 叢集相應縮小或相應放大](service-fabric-tutorial-scale-cluster.md)。
* 使用 Fluent Azure 計算 SDK，[以程式設計方式調整 Azure 叢集](service-fabric-cluster-programmatic-scaling.md)。
* [將獨立叢集相應縮小或相應放大](service-fabric-cluster-windows-server-add-remove-nodes.md)。

