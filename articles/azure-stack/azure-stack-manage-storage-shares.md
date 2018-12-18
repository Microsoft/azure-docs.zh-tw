---
title: 管理 Azure Stack 中的儲存體容量 | Microsoft Docs
description: 監視和管理 Azure Stack 可用的儲存空間。
services: azure-stack
documentationcenter: ''
author: mattbriggs
manager: femila
editor: ''
ms.service: azure-stack
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: PowerShell
ms.topic: get-started-article
ms.date: 09/28/2018
ms.author: mabrigg
ms.reviewer: xiaofmao
ms.openlocfilehash: d979692ba32c596058d28c88f32e7e5872ab7ef9
ms.sourcegitcommit: f31bfb398430ed7d66a85c7ca1f1cc9943656678
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/28/2018
ms.locfileid: "47452321"
---
# <a name="manage-storage-capacity-for-azure-stack"></a>管理 Azure Stack 的儲存體容量 

*適用於：Azure Stack 整合系統和 Azure Stack 開發套件*

本文中的資訊可協助 Azure Stack 雲端操作員監視和管理其 Azure Stack 部署的儲存容量。 Azure Stack 儲存體基礎結構會配置 Azure Stack 部署的總儲存容量子集，以便用於**儲存體服務**。 儲存體服務會將租用戶的資料儲存在磁碟區上對應到部署節點的共用中。

身為雲端操作員，您可以使用的儲存體數量有限。 您實作的解決方案會定義儲存體數量。 當您使用多節點的解決方案時，解決方案是由 OEM 廠商所提供，或由您安裝 Azure Stack 開發套件的硬體所提供。

因為 Azure Stack 不支援儲存體容量擴充，所以務必[監視](#monitor-shares)可用的儲存體，以確保維護有效率的作業。  

當共用的剩餘可用容量有限時，請規劃[管理空間](#manage-available-space)以免共用用完容量。

用於管理容量的選項包括：
- 回收容量
- 移轉容器

當共用的使用率為 100% 時，儲存體服務便無法再對該共用運作。 若要取得還原共用作業的協助，請連絡 Microsoft 支援服務。

## <a name="understand-volumes-and-shares-containers-and-disks"></a>了解磁碟區和共用、容器和磁碟
### <a name="volumes-and-shares"></a>磁碟區和共用
*儲存體服務*會將可用的儲存體分割成相等的個別磁碟區，而這些磁碟區會配置用來保存租用戶資料。 磁碟區數目等於 Azure Stack 部署中的節點數目：

- 在包含四個節點的部署中，有四個磁碟區。 每個磁碟區都有單一共用。 在多重節點的部署上，如果節點已移除或運作失常，共用數目並不會縮減。
- 如果您使用 Azure Stack 開發人員套件，則會有具備單一共用的單一磁碟區。

因為儲存體服務共用專供儲存體服務使用，所以您不得直接修改、新增或移除共用上的任何檔案。 只有儲存體服務應處理這些磁碟區中儲存的檔案。

磁碟區上的共用可保存租用戶資料。 租用戶資料包括分頁 blob、區塊 blob、附加 blob、資料表、佇列、資料庫和相關中繼資料存放區。 因為儲存體物件 (blob 等) 都各自包含在單一共用中，所以每個物件的大小上限不能超過共用的大小。 新物件的大小上限取決於建立新物件時，在共用中仍未使用的空間容量。

當共用上的可用空間不足且[回收](#reclaim-capacity)空間的動作不成功或無法使用時，Azure Stack 雲端操作員可以在共用之間[移轉](#migrate-a-container-between) blob 容器。

- 如需有關容器和 blob 的詳細資訊，請參閱 Azure Stack 中重要功能和概念中的 [Blob 儲存體](azure-stack-key-features.md#blob-storage)。
- 如需租用戶使用者如何使用 Azure Stack 中 blob 儲存體的詳細資訊，請參閱 [Azure Stack 儲存體服務](/azure/azure-stack/user/azure-stack-storage-overview#azure-stack-storage-services)。


### <a name="containers"></a>容器
租用戶使用者可建立容器，接著用來存放 blob 資料。 當使用者決定要放置 blob 的容器時，儲存體服務會使用演算法來判斷哪個磁碟區要放置容器。 此演算法通常會選擇具有最多可用空間的磁碟區。  

將 blob 置於容器之後，該 blob 可以成長以使用更多空間。 隨著您新增 blob 且現有 blob 成長，磁碟區中保存該容器的可用空間會縮減。  

容器不限於單一共用。 當容器中合併的 blob 資料成長而使用 80% 或更多的可用空間時，容器會進入「溢位」模式。 處於溢位模式時，在該容器中建立的任何新 blob 會配置給具有足夠空間的不同磁碟區。 經過一段時間，溢位模式中的容器可以將 blob 分散於多個磁碟區。

使用 80%，而後使用磁碟區中 90% 的可用空間時，系統就會在 Azure Stack 管理員入口網站中引發警示。 雲端操作員應該檢閱可用的儲存體容量，並規劃重新平衡內容。 使用 100% 的磁碟時，儲存體服務會停止運作，而且沒有引發任何其他警示。

### <a name="disks"></a>磁碟
VM 磁碟包含作業系統磁碟，並由租用戶新增至容器。 VM 也可能有一或多個資料磁碟。 這兩種類型的磁碟會儲存為分頁 blob。 指引租用戶將每個磁碟放入不同的容器，以改善 VM 效能。
- 保存 VM 中磁碟 (分頁 blob) 的每個容器，會被視為擁有此磁碟之 VM 所連結的容器。
- 未保存 VM 中任何磁碟的容器則會被視為可用的容器。

用於在連結的容器上釋放空間的選項[有限](#move-vm-disks)。
> [!TIP]  
> 雲端操作員不會直接管理磁碟，而這些磁碟已連結至租用戶可能新增到容器的虛擬機器 (VM)。 不過，在規劃管理儲存體共用上的空間時，則可用來了解磁碟與容器和共用有何關聯。

## <a name="monitor-shares"></a>監視共用
使用 PowerShell 或管理員入口網站來監視共用，以便了解可用空間何時受限。 當您使用入口網站時，您會收到有關共用空間不足的警示。    

### <a name="use-powershell"></a>使用 PowerShell
身為雲端操作員，您可以使用 PowerShell **Get-AzsStorageShare** Cmdlet 來監視共用的儲存體容量。 Get-AzsStorageShare Cmdlet 會傳回每個共用上總計、已配置和可用的空間 (以位元組為單位)。   
![範例：傳回共用的可用空間](media/azure-stack-manage-storage-shares/free-space.png)

- [容量總計] 是共用上可用的空間總計 (以位元組為單位)。 此空間用於儲存體服務所維護的資料和中繼資料。
- [已使用容量] 是存放租用戶資料和相關中繼資料的檔案所使用的資料數量 (以位元組為單位)。

### <a name="use-the-administrator-portal"></a>使用管理員入口網站
身為雲端操作員，您可以使用管理員入口網站來檢視所有共用的儲存體容量。

1. 登入[管理入口網站](https://adminportal.local.azurestack.external)。
2. 選取 [所有服務]  >  [儲存體] 以開啟檔案共用清單，您可以在其中檢視使用量資訊。 

  ![範例：儲存體檔案共用](media/azure-stack-manage-storage-shares/storage-file-shares.png)

  - [總計] 是共用上可用的空間總計 (以位元組為單位)。 此空間用於儲存體服務所維護的資料和中繼資料。
  - [已使用] 是存放租用戶資料和相關中繼資料的檔案所使用的資料數量 (以位元組為單位)。

### <a name="storage-space-alerts"></a>儲存體空間警示
當您使用管理員入口網站時，您會收到有關共用空間不足的警示。

> [!IMPORTANT]
> 身為雲端操作員，讓共用不會達到完全使用狀態。 當共用的使用率為 100% 時，儲存體服務便無法再對該共用運作。 若要在已 100% 使用的共用上復原可用空間及還原作業，您必須連絡 Microsoft 支援服務。

**警告**：當檔案共用使用超過 80% 時，您會在管理員入口網站中收到「警告」警示：![範例：警告警示](media/azure-stack-manage-storage-shares/alert-warning.png)


**重大**：當檔案共用使用超過 90% 時，您會在管理員入口網站中收到「重大」警示：![範例：重大警示](media/azure-stack-manage-storage-shares/alert-critical.png)

**檢視詳細資料**：您可以在管理員入口網站中開啟警示的詳細資料，以檢視緩和選項：![範例：檢視警示詳細資料](media/azure-stack-manage-storage-shares/alert-details.png)


## <a name="manage-available-space"></a>管理可用的空間
必須釋放共用上的可用空間時，請先使用最不具侵略性的方法。 例如，嘗試在選擇移轉容器之前回收空間。  

### <a name="reclaim-capacity"></a>回收容量
*此選項適用於多重節點的部署和 Azure Stack 開發套件。*

您可以回收已刪除之租用戶帳戶所使用的容量。 當資料到達[保留期](azure-stack-manage-storage-accounts.md#set-the-retention-period)時，系統會自動回收此容量，或者您可以採取動作立即將它回收。

如需詳細資訊，請參閱「管理儲存體資源」中的[回收容量](azure-stack-manage-storage-accounts.md#reclaim)。

### <a name="migrate-a-container-between-volumes"></a>在磁碟區之間移轉容器
*此選項只適用於多重節點的部署。*

有些租用戶共用會由於租用戶使用模式，而使用比其他共用更多的空間。 結果可能是在相對未使用的其他共用之前，空間不足的共用。

您可以將某些 blob 容器手動移轉到不同的共用，嘗試在過度使用的共用上釋放空間。 您可以將數個較小容器移轉至有容量可保存所有容量的單一共用。 您可以使用移轉作業來移動「可用」容器。 可用容器是不包含 VM 磁碟的容器。   

移轉會合併新共用上的所有容器 blob。

- 如果容器已進入溢位模式且 blob 已置於其他磁碟區上，新的共用必須有足夠的容量可保存您移轉之容器的所有 blob。 這包括位於其他共用上的 blob。

- PowerShell Cmdlet Get-AzsStorageContainer 只會識別容器的初始磁碟區上使用中的空間。 此 Cmdlet 不會識別已置於其他磁碟區上的 blob 所使用的空間。 因此，容器的完整大小可能不明顯。 新共用上的容器彙總可能會使該新共用進入溢位狀況，以將資料置於其他共用。 因此，您可能需要再次重新平衡共用。

- 如果您缺少資源群組的權限，且無法使用 PowerShell 來查詢溢位資料的其他磁碟區，請與這些資源群組和容器的擁有者合作，以在移轉該資料之前了解要移轉的資料大小總計。  

> [!IMPORTANT]
> 容器的 blob 移轉是需要使用 PowerShell 的離線作業。 在移轉完成前，您正在移轉之容器的所有 blob 會保持離線狀態且無法使用。 您也應該避免在所有進行中的移轉完成之前升級 Azure Stack。

#### <a name="to-migrate-containers-using-powershell"></a>使用 PowerShell 來移轉容器
1. 確認您[已安裝並設定 Azure PowerShell](http://azure.microsoft.com/documentation/articles/powershell-install-configure/)。 如需詳細資訊，請參閱 [將 Azure PowerShell 與 Azure 資源管理員搭配使用](http://go.microsoft.com/fwlink/?LinkId=394767)。
2.  檢查容器以了解您打算移轉的共用上有哪些資料。 若要識別可在磁碟區中移轉的最佳候選容器，請使用 **Get-AzsStorageContainer** Cmdlet：

    ````PowerShell  
    $farm_name = (Get-AzsStorageFarm)[0].name
    $shares = Get-AzsStorageShare -FarmName $farm_name
    $containers = Get-AzsStorageContainer -ShareName $shares[0].ShareName -FarmName $farm_name
    ````
    接著，檢查 $containers：

    ````PowerShell
    $containers
    ````

    ![範例：$Containers](media/azure-stack-manage-storage-shares/containers.png)

3.  找出最佳目的地共用以保存您移轉的容器：

    ````PowerShell
    $destinationshares = Get-AzsStorageShare -SourceShareName
    $shares[0].ShareName -Intent ContainerMigration
    ````

    接著，檢查 $destinationshares：

    ````PowerShell 
    $destinationshares
    ````

    ![範例：$destination shares](media/azure-stack-manage-storage-shares/examine-destinationshares.png)

4. 開始移轉容器。 移轉不是同步進行。 如果您在第一次移轉完成之前開始移轉其他容器，請使用作業識別碼來追蹤每個容器的狀態。

  ````PowerShell
  $job_id = Start-AzsStorageContainerMigration -StorageAccountName $containers[0].Accountname -ContainerName $containers[0].Containername -ShareName $containers[0].Sharename -DestinationShareUncPath $destinationshares[0].UncPath -FarmName $farm_name
  ````

  接著，檢查 $jobId。 在下列範例中，以您想要檢查的作業識別碼取代 d62f8f7a-8b46-4f59-a8aa-5db96db4ebb0：

  ````PowerShell
  $jobId
  d62f8f7a-8b46-4f59-a8aa-5db96db4ebb0
  ````

5. 使用作業識別碼來檢查移轉作業的狀態。 當容器移轉完成時，**MigrationStatus** 會設定為 **Complete**。

  ````PowerShell 
  Get-AzsStorageContainerMigrationStatus -JobId $job_id -FarmName $farm_name
  ````

  ![範例：移轉狀態](media/azure-stack-manage-storage-shares/migration-status1.png)

6.  您可以取消進行中的移轉工作。 已取消的移轉作業會以非同步方式處理。 您可以使用 $jobid 追蹤取消作業：

  ````PowerShell
  Stop-AzsStorageContainerMigration -JobId $job_id -FarmName $farm_name
  ````

  ![範例：復原狀態](media/azure-stack-manage-storage-shares/rollback.png)

7. 您可以再次執行步驟 6 中的命令，直到狀態確認移轉作業為 **Canceled** 為止：  

    ![範例：已取消狀態](media/azure-stack-manage-storage-shares/cancelled.png)

### <a name="move-vm-disks"></a>移動 VM 磁碟
*此選項只適用於多重節點的部署。*

用於管理空間的最極端方法牽涉到虛擬機器磁碟的移動。 因為移動連結的容器 (包含 VM 磁碟的容器) 很複雜，請連絡 Microsoft 支援服務以完成此動作。

## <a name="next-steps"></a>後續步驟
深入了解如何[將虛擬機器提供給使用者](azure-stack-tutorial-tenant-vm.md)。
