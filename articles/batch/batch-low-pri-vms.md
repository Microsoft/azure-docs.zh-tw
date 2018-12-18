---
title: 在符合成本效益的低優先順序 VM 上執行 Azure Batch 工作負載 | Microsoft Docs
description: 了解如何佈建低優先順序的 VM，降低 Azure Batch 工作負載的成本。
services: batch
author: mscurrell
manager: jeconnoc
ms.assetid: dc6ba151-1718-468a-b455-2da549225ab2
ms.service: batch
ms.devlang: multiple
ms.topic: article
ms.workload: na
ms.date: 03/19/2018
ms.author: markscu
ms.openlocfilehash: d42cef944c3b971804ef1417a3877bf919784a02
ms.sourcegitcommit: e8f443ac09eaa6ef1d56a60cd6ac7d351d9271b9
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/12/2018
ms.locfileid: "35771077"
---
# <a name="use-low-priority-vms-with-batch"></a>使用低優先順序的 VM 搭配 Batch

Azure Batch 提供低優先順序的虛擬機器 (VM)，可降低 Batch 工作負載的成本。 低優先順序的 VM 能提供大量的計算能力，且使用成本非常低，從而實現新的 Batch 工作負載類型。
 
低優先順序的 VM 能善用 Azure 中的剩餘容量。 當您指定集區中的低優先順序 VM 時，Azure Batch 就會在有多餘的容量時加以使用。
 
使用低優先順序 VM 的代價是，這些 VM 可能無法用於配置，或可能隨時會有高優先順序的 VM 先佔，視可用容量而定。 基於這個理由，低優先順序的 VM 最適合特定類型的工作負載。 低優先順序的 VM 是用於批次和非同步處理的工作負載，這種工作負載的作業完成時間很有彈性，且工作會分散於許多 VM。
 
低優先順序的 VM 比起專用的 VM，能以大幅降低的價格提供。 如需定價詳細資料，請參閱 [Batch 定價](https://azure.microsoft.com/pricing/details/batch/)。

## <a name="use-cases-for-low-priority-vms"></a>低優先順序 VM 的使用案例

如果有低優先順序的 VM 的特性，哪些工作負載可以使用和無法使用它們？ 一般情況下，批次處理工作負載就很適合，因為作業會區分成許多平行的工作，或是會將許多作業相應放大並分散於許多 VM。

-   若要充分使用 Azure 中的剩餘容量，可將適當的作業相應放大。

-   VM 偶爾可能會無法使用或被優先佔用，這會導致作業容量降低，且可能會導致工作中斷及重新執行。 因此，作業在可使用的時間內必須是有彈性的，才會將作業加以執行。

-   如果工作較長的作業受到中斷，可能就會影響較大。 如果長時間執行的工作在執行時實作檢查點來儲存進度，就可降低中斷所造成的影響。 執行時間較短的工作通常最適合低優先順序的 VM，因為中斷的影響較小。

-   利用多個 VM 長時間執行的 MPI 作業並不太適合使用低優先順序的 VM，因為一個優先佔用的 VM 就可能會導致整個作業必須重新執行一次。

適用於低優先順序 VM 的一些批次處理使用案例的範例如下︰

-   **開發與測試**︰尤其是如果您正在開發大規模的解決方案，可節省可觀的成本。 所有的測試類型都能有所助益，但大規模的負載測試及迴歸測試都是很棒的用途。

-   **補充隨選容量**︰低優先順序的 VM 可用來補充一般的專用 VM - 在可使用時，作業就能加以調整並從而以較低的成本加速完成；在無法使用時，仍能使用專用 VM 的基準。

-   **彈性的作業執行時間**︰如果作業完成所需的時間有彈性，就能容許可能的容量下降；但如果增加低優先順序的 VM ，作業就能經常執行地更快，且成本更低。

有幾種方法可將 Batch 集區設為使用低優先順序的 VM，視作業執行時間的彈性而定︰

-   集區中只使用低優先順序的 VM。 在此情況下，Batch 會在可用時復原任何優先佔用的容量。 此種設定是執行作業最便宜的方式，因為只會使用低優先順序的 VM。

-   低優先順序的 VM 可以用來搭配專用 VM 的固定基準。 固定的專用 VM 數目可確保一律會有一些容量可保持作業進度。

-   可以將專用 VM 和低優先順序 VM 動態混用，如此就能在可用時單獨使用低成本低優先順序的 VM，但會視需要將原定價格的專用 VM 相應增加。 此設定可保留能夠維持作業進度的最少容量。

## <a name="batch-support-for-low-priority-vms"></a>Batch 支援低優先順序的 VM

Azure Batch 提供多項功能，能讓使用者輕鬆使用及受益於低優先順序的 VM：

-   Batch 集區可以同時包含專用 VM 和低優先順序的 VM。 您可以使用明確的調整大小作業或使用自動調整，在集區建立時指定每種類型的 VM 數目，或是針對現有集區隨時，將每種類型的 VM 數目加以變更。 無論集區中的 VM 類型為何，作業和工作提交可維持不變。 您也可以設定集區，以盡可能便宜的方式，完全使用低優先順序的 VM 來執行作業，但如果容量低於最小臨界值，就會運轉專用 VM，從而保持作業執行。

-   Batch 集區會自動搜尋低優先順序 VM 的目標數目。 如果 VM 被優先佔用，Batch 就會嘗試取代遺失的容量並回到目標。

-   當工作中斷時，Batch 會偵測到並自動將工作重新放入佇列以再次執行。

-   低優先順序的 VM 有不同於專用 VM 的 vCPU 配額。 
    低優先順序 VM 的配額高於專用 VM 的配額，因為低優先順序 VM 的成本較低。 如需詳細資訊，請參閱 [Batch 服務配額和限制](batch-quota-limit.md#resource-quotas)。    

> [!NOTE]
> [使用者訂用帳戶模式](batch-api-basics.md#account)中建立的 Batch 帳戶目前不支援低優先順序的 VM。
>

## <a name="create-and-update-pools"></a>建立和更新集區

Batch 集區可以同時包含專用 VM 和低優先順序的 VM (亦稱為計算節點)。 您可以為專用 VM 和低優先順序的 VM 設定計算節點的目標數目。 節點的目標數目會指定您在集區中想要的 VM 數目。

例如，使用目標為 5 個專用 VM 和 20 個低優先順序 VM 的 Azure 雲端服務來建立集區：

```csharp
CloudPool pool = batchClient.PoolOperations.CreatePool(
    poolId: "cspool",
    targetDedicatedComputeNodes: 5,
    targetLowPriorityComputeNodes: 20,
    virtualMachineSize: "Standard_D2_v2",
    cloudServiceConfiguration: new CloudServiceConfiguration(osFamily: "5") // WS 2016
);
```

使用目標為 5 個專用 VM 和 20 個低優先順序 VM 的 Azure 虛擬機器 (在此情況下為 Linux VM) 來建立集區：

```csharp
ImageReference imageRef = new ImageReference(
    publisher: "Canonical",
    offer: "UbuntuServer",
    sku: "16.04-LTS",
    version: "latest");

// Create the pool
VirtualMachineConfiguration virtualMachineConfiguration =
    new VirtualMachineConfiguration("batch.node.ubuntu 16.04", imageRef);

pool = batchClient.PoolOperations.CreatePool(
    poolId: "vmpool",
    targetDedicatedComputeNodes: 5,
    targetLowPriorityComputeNodes: 20,
    virtualMachineSize: "Standard_D2_v2",
    virtualMachineConfiguration: virtualMachineConfiguration);
```

您可以取得專用 VM 和低優先順序 VM 目前的節點數目：

```csharp
int? numDedicated = pool1.CurrentDedicatedComputeNodes;
int? numLowPri = pool1.CurrentLowPriorityComputeNodes;
```

集區節點的屬性可表示節點為專用的還是低優先順序的 VM：

```csharp
bool? isNodeDedicated = poolNode.IsDedicated;
```

當集區中的一或多個節點被優先佔用時，集區的列出節點作業仍會傳回那些節點。 低優先順序節點的目前數目會維持不變，但這些節點的狀態都已設定為**先佔**狀態。 Batch 會嘗試尋找取代 VM，如果成功，節點在變成可供工作執行前，會逐步變成**建立中**和**啟動中**狀態，就像新的節點一樣。

## <a name="scale-a-pool-containing-low-priority-vms"></a>調整包含低優先順序 VM 的集區

如同由專用 VM 單獨組成的集區，可藉由呼叫 Resize 方法或使用自動調整，調整包含低優先順序 VM 的集區。

調整集區大小的作業會採用第二個選擇性參數，可更新 **targetLowPriorityNodes** 的值：

```csharp
pool.Resize(targetDedicatedComputeNodes: 0, targetLowPriorityComputeNodes: 25);
```

集區自動調整公式支援低優先順序的 VM，如下所示︰

-   您可以取得或設定服務定義之 **$TargetLowPriorityNodes** 變數的值。

-   您可以取得服務定義之 **$CurrentLowPriorityNodes** 變數的值。

-   您可以取得服務定義之 **$PreemptedNodeCount** 變數的值。 
    此變數會傳回優先佔用狀態中的節點數目，並可依無法使用的優先佔用節點數目，將您的專用節點數目相應增加或相應減少。

## <a name="jobs-and-tasks"></a>工作 (Job) 和工作 (Task)

作業和工作幾乎都不需要針對低優先順序節點設定；唯一的支援如下所示︰

-   作業的 JobManagerTask 屬性都有新的屬性，**AllowLowPriorityNode**。 
    當這個屬性為 true 時，就可以在專用節點或低優先順序的節點上排程作業管理員工作。 如果這個屬性為 false，就只會在專用節點上排程作業管理員工作。

-   [環境變數](batch-compute-node-environment-variables.md)可供工作應用程式使用，因此它可以判斷是在低優先順序還是專用節點上執行。 環境變數是 AZ_BATCH_NODE_IS_DEDICATED。

## <a name="handling-preemption"></a>處理優先佔用

VM 可能偶爾會被優先佔用；當發生優先佔用時，Batch 會執行下列動作︰

-   優先佔用的 VM 都會將其狀態更新為**優先佔用**。
-   如果工作是在優先佔用的節點 VM 上執行，就會將這些工作重新排入佇列並再次執行。
-   VM 實際上會被刪除，導致遺失在 VM 上本機儲存的任何資料。
-   集區會繼續嘗試觸達可用的低優先順序節點之目標數目。 找到取代容量時，節點會保留其識別碼，但在可供工作排程使用之前，會先重新初始化，逐步變成**建立中**和**啟動中**狀態。
-   優先佔用計數會在 Azure 入口網站中作為計量提供使用。

## <a name="metrics"></a>度量

[Azure 入口網站](https://portal.azure.com)中有針對低優先順序節點提供的新計量。 這些計量包括：

- 低優先順序節點計數
- 低優先順序核心計數 
- 先占節點計數

若要檢視 Azure 入口網站中的計量：

1. 在入口網站中瀏覽至您的 Batch 帳戶，並檢視 Batch 帳戶的設定。
2. 從 [監視] 區段選取 [計量]。
3. 從 [可用的計量] 清單中選取您所需的計量。

![低優先順序節點的計量](media/batch-low-pri-vms/low-pri-metrics.png)

## <a name="next-steps"></a>後續步驟

* 請參閱 [適用於開發人員的 Batch 功能概觀](batch-api-basics.md)，這是任何準備使用 Batch 的人員不可或缺的資訊。 本文包含 Batch 服務資源 (例如集區、節點、作業和工作) 的詳細資訊，以及在建置 Batch 應用程式時可使用的許多 API 功能。
* 了解可用來建置 Batch 解決方案的 [Batch API 和工具](batch-apis-tools.md)。
