---
title: 升級至 Azure VM 備份堆疊 V2
description: VM 備份堆疊、Azure Resource Manager 部署模型的升級流程與常見問題集
services: backup
author: trinadhk
manager: vijayts
tags: azure-resource-manager, virtual-machine-backup
ms.service: backup
ms.topic: conceptual
ms.date: 8/1/2018
ms.author: trinadhk
ms.openlocfilehash: 1021900620272cc5476d8972daf9d7e0a161797a
ms.sourcegitcommit: d4c076beea3a8d9e09c9d2f4a63428dc72dd9806
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/01/2018
ms.locfileid: "39397996"
---
# <a name="upgrade-to-azure-vm-backup-stack-v2"></a>升級至 Azure VM 備份堆疊 V2

升級至虛擬機器 (VM) 備份堆疊的 Resource Manager 部署模型提供了下列增強功能：

* 能夠查看在備份作業中所建立的快照集 (可用於復原)，而不需等到資料轉送完成。 如此可減少觸發還原之前，將快照集複製到保存庫的等候時間。 此外，也能免去備份進階 VM 時的額外儲存需求 (第一次備份除外)。  

* 將快照集於本機上保存七天，以減少備份與還原時間。

* 支援最多 4 TB 磁碟大小。

* 還原時，可以使用非受控 VM 的原始儲存體帳戶。 即使 VM 的磁碟分散於多個儲存體帳戶，仍然具有此功能。 它能夠針對各種不同的 VM 設定，加快還原作業的速度。
    > [!NOTE]
    > 此功能不同於覆寫原始 VM。 
    >

## <a name="whats-changing-in-the-new-stack"></a>新堆疊有何改變？
目前的備份作業包含兩個階段：
1.  建立 VM 快照集。 
2.  將 VM 快照集傳輸到 Azure 備份保存庫。 

復原點只會在第 1 和第 2 階段完成之後才建立。 在新堆疊中，快照集一完成就會建立復原點。 您也可以使用相同的還原流程，由此復原點來還原。 您可以在 Azure 入口網站中，使用「快照集」復原點類型來識別此復原點。 快照集傳輸至保存庫後，復原點類型會變更為「快照集與保存庫」。 

![VM 備份堆疊 Resource Manager 部署模型的備份作業 -- 儲存體與保存庫](./media/backup-azure-vms/instant-rp-flow.jpg) 

根據預設，快照集會保存 7 天。 此功能可以讓來自這些快照集的還原作業更快完成。 它可以減少將資料從保存庫複製到客戶儲存體帳戶所需的時間。 

## <a name="considerations-before-upgrade"></a>升級前的考量

* VM 備份堆疊為單向升級，所有備份都會進入此流程。 由於變更發生於訂用帳戶層級，因此所有 VM 都會進入此流程。 所有新增加的功能將會建置在相同堆疊之上。 您目前無法在原則層級控制堆疊。

* 快照集會儲存在本機上，以利於建立復原點，也可加快還原作業的速度。 因此，您將看見儲存體成本會隨著在七天期間內取得的快照集而增減。

* 增量快照會儲存為分頁 Blob。 使用非受控磁碟的所有客戶，將必須為儲存在客戶本機儲存體帳戶中的快照集付費 7 天。 因為由受控 VM 備份所使用的還原點集合會於基礎儲存體層級使用 Blob 快照集，因此針對受控磁碟，您將會看見對應到 [Blob 快照集定價](https://docs.microsoft.com/rest/api/storageservices/understanding-how-snapshots-accrue-charges) \(英文\) 的成本，而且它們會持續累加。 

* 如果您從快照集復原點還原進階 VM，便會在建立 VM 時使用暫存位置。

* 對於進階儲存體帳戶，針對立即復原點取得的快照集會計入配置空間的 10 TB 限制。

## <a name="upgrade"></a>升級
### <a name="the-azure-portal"></a>Azure 入口網站
如果您使用 Azure 入口網站，將會在保存庫儀表板上看到通知。 此通知乃是關於大型磁碟支援，以及備份和還原速度的改進。 或者，您可以前往保存庫的 [屬性] 頁面以取得升級選項。

![VM 備份堆疊 Resource Manager 部署模型的備份作業 -- 支援通知](./media/backup-azure-vms/instant-rp-banner.png) 

若要開啟畫面以升級至新的堆疊，請選取橫幅。 

![VM 備份堆疊 Resource Manager 部署模型的備份作業 -- 升級](./media/backup-azure-vms/instant-rp.png) 

### <a name="powershell"></a>PowerShell
從提高權限的 PowerShell 終端機執行下列 Cmdlet：
1.  登入您的 Azure 帳戶： 

    ```
    PS C:> Connect-AzureRmAccount
    ```

2.  選取您要註冊的訂用帳戶：

    ```
    PS C:>  Get-AzureRmSubscription –SubscriptionName "Subscription Name" | Select-AzureRmSubscription
    ```

3.  註冊此訂用帳戶：

    ```
    PS C:>  Register-AzureRmProviderFeature -FeatureName "InstantBackupandRecovery" –ProviderNamespace Microsoft.RecoveryServices
    ```

## <a name="verify-that-the-upgrade-is-finished"></a>驗證已完成升級
從提高權限的 PowerShell 終端機執行下列 Cmdlet︰

```
Get-AzureRmProviderFeature -FeatureName "InstantBackupandRecovery" –ProviderNamespace Microsoft.RecoveryServices
```

如果顯示 "Registered" (已註冊)，則代表您的訂用帳戶已升級至 VM 備份堆疊 Resource Manager 部署模型。

## <a name="frequently-asked-questions"></a>常見問題集

下列問題與解答均收集自論壇與客戶問題。

### <a name="will-upgrading-to-v2-impact-current-backups"></a>升級至 V2 將會影響目前的備份嗎？

如果您升級至 V2，並不會影響到您目前的備份，而且不需重新設定您的環境。 升級和您的備份環境都會繼續正常運作。

### <a name="what-does-it-cost-to-upgrade-to-azure-vm-backup-stack-v2"></a>升級至 Azure VM 備份堆疊 V2 的成本為何？

將堆疊升級至 V2 不需要任何成本。 快照集會儲存在本機上，以加快建立復原點和還原作業的速度。 因此，您將看見儲存體成本會隨著在七天期間內取得的快照集而增減。

### <a name="does-upgrading-to-stack-v2-increase-the-premium-storage-account-snapshot-limit-by-10-tb"></a>升級至堆疊 V2 是否可將進階儲存體帳戶的快照集限制提高到 10 TB？

否，每個儲存體帳戶的總計快照集限制仍維持 10TB。 

### <a name="in-premium-storage-accounts-do-snapshots-taken-for-instant-recovery-point-occupy-the-10-tb-snapshot-limit"></a>在進階儲存體帳戶中，針對立即復原點取得的快照集是否會佔用 10 TB 的快照集限制？

是，對於進階儲存體帳戶，針對立即復原點取得的快照集會佔用 10 TB 的配置空間。

### <a name="how-does-the-snapshot-work-during-the-seven-day-period"></a>快照集在七天期間內的運作方式為何？ 

每天都會取得新的快照集。 有七個個別的快照集。 服務**不會**在第一天取得複本，然後在接下來的六天內新增變更。

### <a name="is-a-v2-snapshot-an-incremental-snapshot-or-full-snapshot"></a>V2 快照集是增量快照集，還是完整的快照集？

增量快照集適用於非受控磁碟。 針對受控磁碟，Azure 備份所建立的還原點集合會使用 Blob 快照集，因此會持續累加。 
