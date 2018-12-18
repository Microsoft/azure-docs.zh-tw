---
title: 使用 Azure Site Recovery 讓 VMware VM 能夠複寫至 Azure | Microsoft Docs
description: 本文說明如何設定使用 Azure Site Recovery 將 VMware VM 複寫至 Azure 的作業。
services: site-recovery
author: asgang
ms.service: site-recovery
ms.date: 07/06/2018
ms.topic: conceptual
ms.author: asgang
ms.openlocfilehash: 9a868b196a287b7a5121803136d3c0119f64d9fe
ms.sourcegitcommit: a06c4177068aafc8387ddcd54e3071099faf659d
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/09/2018
ms.locfileid: "37917018"
---
# <a name="enable-replication-to-azure-for-vmware-vms"></a>讓 VMware VM 能夠複寫至 Azure


本文說明如何啟用將內部部署 VMware VM 複寫至 Azure 的作業。

## <a name="prerequisites"></a>先決條件

本文假設您已經：

1.  [設定內部部署來源環境](vmware-azure-set-up-source.md)。
2.  [在 Azure 中設定目標環境](vmware-azure-set-up-target.md)。


## <a name="before-you-start"></a>開始之前
當您要複寫 VMware 虛擬機器時︰

* 您的 Azure 使用者帳戶必須具有特定[權限](site-recovery-role-based-linked-access-control.md#permissions-required-to-enable-replication-for-new-virtual-machines)，才能將新的虛擬機器複寫至 Azure。
* 系統會每隔 15 分鐘探索 VMware VM 一次。 在探索之後，可能需要 15 分鐘以上，VMware VM 才會出現在 Azure 入口網站中。 同樣地，當您加入新的 vCenter 伺服器或 vSphere 主機時，探索可能需要 15 分鐘以上。
* 虛擬機器上的環境變更 (例如 VMware 工具安裝) 會花上 15 分鐘以上的時間，才會在入口網站更新。
* 您可以在 [組態伺服器] 頁面上 vCenter 伺服器/vSphere 主機的 [上次連絡時間] 欄位中，查看 VMware VM 的上次探索時間。
* 若要新增要複寫的機器而不等候已排定的探索，請反白選取組態伺服器 (不要按它)，然後按一下 [重新整理] 按鈕。
* 當您啟用複寫時，如果機器已準備好，處理序伺服器會自動在其上安裝行動服務。


## <a name="enable-replication"></a>啟用複寫

1. 按一下 [步驟 2: 複寫應用程式]  >  [來源]。 第一次啟用複寫之後，請按一下保存庫中的 [+複寫]，以對其他機器啟用複寫。
2. 在 [來源] 頁面 > [來源] 中，選取組態伺服器。
3. 在 [機器類型] 中，選取 [虛擬機器] 或 [實體機器]。
4. 在 [vCenter/vSphere Hypervisor] 中，選取管理 vSphere 主機的 vCenter 伺服器，或選取主機。 如果您是複寫實體機器，則這個設定不相關。
5. 如果您尚未建立任何額外的處理序伺服器，則選取處理序伺服器，將是設定伺服器的名稱。 然後按一下 [確定] 。

    ![啟用複寫來源](./media/vmware-azure-enable-replication/enable-replication2.png)

6. 在 [目標] 中，選取您想要在其中建立容錯移轉虛擬機器的訂用帳戶和資源群組。 選擇您需要在 Azure 中針對容錯移轉虛擬機器使用的部署模型。

7. 選取您要用來複寫資料的 Azure 儲存體帳戶。 

    > [!NOTE]

    >   * 您可以選取進階或標準儲存體帳戶。 如果選取進階帳戶，您需要針對進行中的複寫記錄檔，額外指定一個標準儲存體帳戶。 帳戶必須位於與復原服務保存庫相同的區域中。
    >   * 如果您想使用不同的儲存體帳戶，您可以[建立一個](../storage/common/storage-create-storage-account.md)。 若要使用 Resource Manager 建立儲存體帳戶，請按一下 [新建]。 

8. 選取 Azure VM 在容錯移轉後啟動時所要啟動的 Azure 網路和子網路。 此網路必須位於與復原服務保存庫相同的區域中。 選取 [立即設定選取的機器]，將網路設定套用至您選取要進行保護的所有機器。 選取 [稍後設定] 以選取每部機器的 Azure 網路。 如果您沒有網路，則必須[建立一個](#set-up-an-azure-network)。 若要使用 Resource Manager 建立網路，請按一下 [新建]。 選取子網路 (如果適用的話)，然後按一下 [確定]。

    ![啟用複寫目標設定](./media/vmware-azure-enable-replication/enable-rep3.png)
9. 在 [虛擬機器] > [選取虛擬機器] 中，選取您要複寫的每部機器。 您只能選取可以啟用複寫的機器。 然後按一下 [確定] 。

    ![啟用複寫選取虛擬機器](./media/vmware-azure-enable-replication/enable-replication5.png)
10. 在 [屬性] > [設定屬性] 中，選取處理序伺服器要用來在電腦上自動安裝行動服務的帳戶。  
11. 依預設會複寫所有磁碟。 若要將磁碟從複寫中排除，按一下 [所有磁碟]，然後清除任何您不想要複寫的磁碟。  然後按一下 [確定] 。 您可以稍後再設定其他屬性。 [深入了解](vmware-azure-exclude-disk.md)排除磁碟。

    ![啟用複寫設定屬性](./media/vmware-azure-enable-replication/enable-replication6.png)

12. 在 [複寫設定] > [設定複寫設定] 中，確認已選取正確的複寫原則。 您可以在 [設定]  >  [複寫原則] > 原則名稱 > [編輯設定] 中修改複寫原則設定。 套用到原則的變更將會套用到複寫和新的機器。
13. 如果您想要將機器聚集成一個複寫群組，請啟用 [多部 VM 一致性]。 指定群組的名稱，然後按一下 [確定]。 

    > [!NOTE]

    >    * 複寫群組中的機器會一起複寫，並且在容錯移轉時會有共用的損毀一致和應用程式一致的復原點。
    >    * 將 VM 與實體伺服器一起收集，讓它們鏡像您的工作負載。 啟用多 VM 一致性可能會影響工作負載的效能。 僅在電腦執行相同的工作負載且需要一致性時，才應該使用它。

    ![啟用複寫](./media/vmware-azure-enable-replication/enable-replication7.png)
14. 按一下 [啟用複寫] 。 您可以在 [設定]  >  [作業]  >  [Site Recovery 作業] 中，追蹤 [啟用保護] 作業的進度。 執行 [完成保護] 作業之後，機器即準備好進行容錯移轉。

> [!NOTE]
> 如果機器已準備好進行推送安裝，則當啟用保護時，將會安裝行動服務元件。 在機器上安裝元件之後，保護工作會啟動並失敗。 在失敗之後，您需要手動將每一部電腦重新啟動。 重新啟動之後，保護作業再次開始，並且會發生初始複寫。
>
>

## <a name="view-and-manage-vm-properties"></a>檢視及管理 VM 屬性

接著確認來源機器的屬性。 請記住，Azure VM 名稱應該符合 [Azure 虛擬機器需求](vmware-physical-azure-support-matrix.md#replicated-machines)。

1. 按一下 [設定] > [複寫的項目]，然後選取機器。 [程式集] 頁面會顯示機器設定與狀態的相關資訊。
2. 在 [屬性] 中，您可以檢視 VM 的複寫和容錯移轉資訊。
3. 在 [計算和網路] > [計算屬性] 中，您可以指定 Azure VM 名稱和目標大小。 視需要修改名稱以符合 Azure 需求。

    ![計算和網路屬性](./media/vmware-azure-enable-replication/vmproperties.png)

4.  您可以選取電腦將成為其後置容錯移轉一部分的[資源群組](https://docs.microsoft.com/azure/virtual-machines/windows/infrastructure-resource-groups-guidelines)。 您可以在容錯移轉之前隨時變更這項設定。 容錯移轉後，如果將電腦移轉到不同的資源群組，則電腦的保護設定將會中斷。
5. 如果您的電腦必須是其中一個後置容錯移轉，您可以選取[可用性設定組](https://docs.microsoft.com/azure/virtual-machines/windows/infrastructure-availability-sets-guidelines)。 選取可用性設定組時，請記住︰

    * 只將屬於指定之資源群組的可用性設定組列出。  
    * 虛擬網路不同的電腦不可在相同的可用性設定組中。
    * 只有相同大小的虛擬機器可在相同的可用性設定組中。
5. 您也可以檢視和加入目標網路、子網路的相關資訊，以及將指派給 Azure VM 的 IP 位址。
6. 在 [磁碟] 中，您可以看見 VM 上將要複寫的作業系統和資料磁碟。

### <a name="configure-networks-and-ip-addresses"></a>設定網路和 IP 位址

- 您可以設定目標 IP 位址。 如果您未提供地址，容錯移轉的電腦就會使用 DHCP。 如果您設定的位址在容錯移轉時無法使用，容錯移轉就無法運作。 如果位址可用於測試容錯移轉網路，則相同的目標 IP 位址可用於測試容錯移轉。
- 網路介面卡的數目會視您指定給目標虛擬機器的大小而有所不同，如下所示：
    - 如果來源電腦上的網路介面卡數目小於或等於針對目標機器大小所允許的介面卡數目，則目標將具備與來源相同的介面卡數目。
    - 如果來源虛擬機器的介面卡數目超過針對目標大小所允許的數目，則使用目標大小的最大值。
    例如，如果來源電腦具有兩張網路介面卡，而目標電腦大小支援四張，目標電腦就會有兩張介面卡。 如果來源電腦具有兩張介面卡，但支援的目標大小僅支援一張介面卡，目標電腦就只會有一張介面卡。
    - 如果虛擬機器有多張網路介面卡，就會全部連線至相同的網路。 此外，清單中顯示的第一項就會變成 Azure 虛擬機器中的*預設*網路介面卡。

### <a name="azure-hybrid-benefit"></a>Azure Hybrid Benefit

微軟軟體保證客戶可以使用 Azure Hybrid Benefit，節省移轉至 Azure 之 Windows Server 電腦的授權成本，或使用 Azure 進行災害復原。 如果您有權使用 Azure Hybrid Benefit，可以將獲指派這項權益的虛擬機器指定為 Azure Site Recovery 在容錯移轉時建立的一部虛擬機器。 作法：
- 移至複寫之虛擬機器的 [計算與網路屬性] 區段。
- 回答問題，了解您是否具備 Windows Server 授權，讓您符合使用 Azure Hybrid Benefit 的資格。
- 選取核取方塊，確認您具備擁有軟體保證的合格 Windows Server 授權，可讓您在容錯移轉時建立的電腦上套用 Azure Hybrid Benefit。
- 儲存複寫機器的設定。

深入了解 [Azure Hybrid Benefit](https://aka.ms/azure-hybrid-benefit-pricing)。

## <a name="common-issues"></a>常見問題

* 各磁碟大小必須都小於 1 TB。
* OS 磁碟應該是基本磁碟而非動態磁碟。
* 針對第 2 代/已啟用 UEFI 的虛擬機器，作業系統系列應為 Windows，而且開機磁碟應小於 300 GB。

## <a name="next-steps"></a>後續步驟

保護完成且機器達到受保護的狀態之後，您可以嘗試[容錯移轉](site-recovery-failover.md)，以檢查應用程式是否會出現在 Azure 中。

如果您想要停用保護，請了解如何[清除註冊與保護設定](site-recovery-manage-registration-and-protection.md)。
