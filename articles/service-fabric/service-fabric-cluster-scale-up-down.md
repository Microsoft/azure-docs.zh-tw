---
title: 相應縮小或放大 Service Fabric 叢集 | Microsoft Docs
description: 設定每個節點類型/虛擬機器擴展集的自動調整規模規則，以相應縮小或放大 Service Fabric 叢集使其符合需求。 新增或移除 Service Fabric 叢集的節點
services: service-fabric
documentationcenter: .net
author: aljo-microsoft
manager: timlt
editor: ''
ms.assetid: aeb76f63-7303-4753-9c64-46146340b83d
ms.service: service-fabric
ms.devlang: dotnet
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 06/22/2017
ms.author: aljo
ms.openlocfilehash: d820898b1a0cc26d6832be9d302c74306fa4882f
ms.sourcegitcommit: 30c7f9994cf6fcdfb580616ea8d6d251364c0cd1
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/18/2018
ms.locfileid: "42146186"
---
# <a name="read-before-you-scale"></a>調整之前閱讀
調整計算資源來符合您的應用程式工作負載時，需要經過刻意規劃，生產環境幾乎一律需要超過一小時的時間才能完成規劃，而且會要求您了解您的工作負載和商務內容；事實上，如果您之前不曾執行這項活動，建議您先閱讀並了解 [Service Fabric 叢集容量規劃考量](https://docs.microsoft.com/en-us/azure/service-fabric/service-fabric-cluster-capacity)，再繼續進行本文件的其餘部分。 此建議是為了避免非預期的 LiveSite 問題，此外，也建議您成功測試您決定要對非商業執行環境執行的作業。 您可以隨時[報告實際執行問題，或要求 Azure 的付費支援](https://docs.microsoft.com/en-us/azure/service-fabric/service-fabric-support#report-production-issues-or-request-paid-support-for-azure)。 針對配置來執行這些作業且擁有適當內容的工程師，本文將說明調整作業，但您必須決定並了解哪些作業適合您的使用案例；例如要調整哪些資源 (CPU、儲存體、記憶體)、調整方向為何 (水平或垂直)，以及要執行哪些作業 (資源範本部署、入口網站、PowerShell/CLI)。

# <a name="scale-a-service-fabric-cluster-in-or-out-using-auto-scale-rules-or-manually"></a>使用自動調整規則或以手動方式相應縮小或放大 Service Fabric 叢集
虛擬機器擴展集是一個 Azure 計算資源，可以用來將一組虛擬機器當做一個集合加以部署和管理。 在 Service Fabric 叢集中定義的每個節點類型都會安裝為不同的虛擬機器擴展集。 然後每個節點類型可以獨立相應縮小或放大，可以開啟不同組的連接埠，並可以有不同的容量度量。 若要深入了解，請參閱 [Service Fabric 節點類型](service-fabric-cluster-nodetypes.md) 文件。 因為叢集中的 Service Fabric 節點類型是由後端的虛擬機器擴展集建立，所以您必須為每個節點類型/虛擬機器擴展集設定自動調整規則。

> [!NOTE]
> 您的訂用帳戶必須要有足夠的核心，來新增構構成此叢集的虛擬機器。 目前沒有模型驗證，所以如果達到任一配額限制，就會收到部署時間失敗。 此外，單一節點類型不能只是每個 VMSS 超過 100 個節點。 您可能必須新增 VMSS 來達成目標的縮放，且自動縮放比例無法自動新增 VMSS。 就地將 VMSS 新增至即時叢集是富有挑戰性的工作，而通常這會導致使用者利用建立時所佈建的適當節點類型來佈建新叢集；據此[規劃叢集容量](https://docs.microsoft.com/en-us/azure/service-fabric/service-fabric-cluster-capacity)。 
> 
> 

## <a name="choose-the-node-typevirtual-machine-scale-set-to-scale"></a>選擇要調整規模的節點類型/虛擬機器擴展集
目前，您不能使用入口網站指定虛擬機器擴展集的自動調整規則，所以請讓我們使用 Azure PowerShell (1.0+) 列出節點類型，然後向它們新增自動調整規則。

若要取得建立叢集的虛擬機器擴展集清單，請執行下列 Cmdlet：

```powershell
Get-AzureRmResource -ResourceGroupName <RGname> -ResourceType Microsoft.Compute/VirtualMachineScaleSets

Get-AzureRmVmss -ResourceGroupName <RGname> -VMScaleSetName <Virtual Machine scale set name>
```

## <a name="set-auto-scale-rules-for-the-node-typevirtual-machine-scale-set"></a>設定節點類型/虛擬機器擴展集的自動調整規模規則
如果您的叢集有多個節點類型，您就需要為每個要相應縮小或放大的節點類型/虛擬機器擴展集重複這項作業。 請先考慮一定要有的節點數目，再設定自動調整規模。 主要節點類型一定要有的節點數目下限是由您已選擇的可靠性層級決定。 深入了解 [可靠性層級](service-fabric-cluster-capacity.md)。

> [!NOTE]
> 將主要節點類型相應減少到小於最低數目，會造成叢集不穩定或關閉。 這可能導致應用程式和系統服務資料遺失。
> 
> 

自動調整規模功能目前不是由應用程式可能向 Service Fabric 報告的負載所驅動。 所以，您現在取得的自動調整只由每個虛擬機器擴展集執行個體所發出的效能計數器驅動。  

請遵循下列指示[設定每個虛擬機器擴展集的自動調整規模](../virtual-machine-scale-sets/virtual-machine-scale-sets-autoscale-overview.md)。

> [!NOTE]
> 在相應減少的案例中，除非節點類型有 Gold 或 Silver 的持久性層級，否則必須以適當的節點名稱呼叫 [Remove-ServiceFabricNodeState](https://docs.microsoft.com/powershell/module/servicefabric/remove-servicefabricnodestate) Cmdlet。針對 Bronze 持久性，不建議一次相應減少超過一個節點。
> 
> 

## <a name="manually-add-vms-to-a-node-typevirtual-machine-scale-set"></a>手動將 VM 加入節點類型/虛擬機器擴展集
請依照 [快速啟動範本庫](https://github.com/Azure/azure-quickstart-templates/tree/master/201-vmss-scale-existing) 的範例/指示變更每個 Nodetype 的 VM 數目。 

> [!NOTE]
> 新增 VM 需要時間，因此請不要期待馬上就會有結果。 所以請善加規劃增加容量的時間，在複本/服務執行個體可以使用 VM 容量之前有超過 10 分鐘的時間讓一切就定位。
> 
> 

## <a name="manually-remove-vms-from-the-primary-node-typevirtual-machine-scale-set"></a>手動從主要節點類型/虛擬機器擴展集移除 VM
> [!NOTE]
> Service Fabric 系統服務在叢集中是以主要節點類型執行。 所以請永遠不要關閉該節點類型的執行個體，或將該節點類型的執行個體數目相應減少到少於可靠性層級所需的數目。 請參閱 [可靠性層級的詳細資料](service-fabric-cluster-capacity.md)。 
> 
> 

您必須一次一個 VM 執行個體執行下列步驟。 這可讓系統服務 (以及您的具狀態服務) 在您要移除的 VM 執行個體上正常關閉，並且在其他節點上建立新複本。

1. 執行 [Disable-ServiceFabricNode](https://docs.microsoft.com/powershell/module/servicefabric/disable-servicefabricnode?view=azureservicefabricps) 加上 'RemoveNode' 可停用您要移除的節點 (該節點類型的最高執行個體)。
2. 執行 [Get-ServiceFabricNode](https://docs.microsoft.com/powershell/module/servicefabric/get-servicefabricnode?view=azureservicefabricps) 可確保節點已確實轉換為停用。 如果沒有，請等到節點停用。 您無法加快此步驟的速度。
3. 請依照[快速啟動範本庫](https://github.com/Azure/azure-quickstart-templates/tree/master/201-vmss-scale-existing)的範例/指示變更該節點類型的一個 VM。 移除的執行個體是最高的 VM 執行個體。 
4. 視需要重複步驟 1 到 3，但是請永遠不要將主要節點類型的執行個體數目相應減少到少於可靠性層級所需的數目。 請參閱 [可靠性層級的詳細資料](service-fabric-cluster-capacity.md)。 

## <a name="manually-remove-vms-from-the-non-primary-node-typevirtual-machine-scale-set"></a>手動從非主要節點類型/虛擬機器擴展集移除 VM
> [!NOTE]
> 如果是具狀態服務，您需要一些永遠啟動的節點來維持可用性，以及維持服務的狀態。 您至少需要與資料分割/服務的目標複本集計數相等的節點數目。 
> 
> 

您必須一次一個 VM 執行個體執行下列步驟。 這可讓系統服務 (以及您的具狀態服務) 在您要移除的 VM 執行個體上正常關閉，並且在其他位置建立新複本。

1. 執行 [Disable-ServiceFabricNode](https://docs.microsoft.com/powershell/module/servicefabric/disable-servicefabricnode?view=azureservicefabricps) 加上 'RemoveNode' 可停用您要移除的節點 (該節點類型的最高執行個體)。
2. 執行 [Get-ServiceFabricNode](https://docs.microsoft.com/powershell/module/servicefabric/get-servicefabricnode?view=azureservicefabricps) 可確保節點已確實轉換為停用。 如果沒有，請等到節點停用。 您無法加快此步驟的速度。
3. 請依照[快速啟動範本庫](https://github.com/Azure/azure-quickstart-templates/tree/master/201-vmss-scale-existing)的範例/指示變更該節點類型的一個 VM。 現在會移除最高的 VM 執行個體。 
4. 視需要重複步驟 1 到 3，但是請永遠不要將主要節點類型的執行個體數目相應減少到少於可靠性層級所需的數目。 請參閱 [可靠性層級的詳細資料](service-fabric-cluster-capacity.md)。

## <a name="behaviors-you-may-observe-in-service-fabric-explorer"></a>Service Fabric Explorer 可能出現的行為
當您相應增加叢集時，Service Fabric Explorer 會反映屬於叢集的節點數目 (虛擬機器擴展集執行個體)。  不過，當您相應減少叢集時，除非您以適當的節點名稱呼叫 [Remove-ServiceFabricNodeState](https://docs.microsoft.com/powershell/module/servicefabric/remove-servicefabricnodestate) Cmdlet，否則會看到已移除的節點/VM 執行個體顯示為健康狀態不良。   

以下是這種行為的說明。

Service Fabric Explorer 列出的節點會反映出 Service Fabric 系統服務 (特別是 FM) 所知叢集曾經有過/現有擁有的節點數目。 當您相應減少虛擬機器擴展集時會刪除 VM，但 FM 系統服務仍然認為節點 (也就是已刪除的對應 VM) 會回復。 所以 Service Fabric Explorer 會繼續顯示該節點 (雖然健全狀況狀態可能是錯誤或未知)。

為確保移除 VM 時也移除節點，您有兩個選項︰

1) 為叢集中的節點類型選擇 Gold 或 Silver 持久性層級，提供您基礎結構的整合。 它會在您相應減少時，從我們的系統服務 (FM) 狀態自動移除節點。
請參閱 [持久性層級的詳細資訊](service-fabric-cluster-capacity.md)

2) VM 執行個體一旦相應減少，您就必須呼叫 [Remove-ServiceFabricNodeState Cmdlet](https://docs.microsoft.com/powershell/module/servicefabric/remove-servicefabricnodestate)。

> [!NOTE]
> Service Fabric 叢集需要有一定數量的節點隨時都處於啟動，以維持可用性並維持狀態 - 稱為「維持仲裁」。 因此，除非您已先執行 [狀態的完整備份](service-fabric-reliable-services-backup-restore.md)，否則關閉叢集中的所有電腦通常並不安全。
> 
> 

## <a name="next-steps"></a>後續步驟
亦請參閱下列文章，了解規劃叢集容量、升級叢集和分割服務︰

* [規劃叢集容量](service-fabric-cluster-capacity.md)
* [叢集升級](service-fabric-cluster-upgrade.md)
* [分割具狀態服務以達最大規模](service-fabric-concepts-partitioning.md)

<!--Image references-->
[BrowseServiceFabricClusterResource]: ./media/service-fabric-cluster-scale-up-down/BrowseServiceFabricClusterResource.png
[ClusterResources]: ./media/service-fabric-cluster-scale-up-down/ClusterResources.png
