---
title: 使用 Site Recovery 將複寫的 VMware VM 和實體伺服器容錯移轉及容錯回復至 Azure | Microsoft Docs
description: 了解如何使用 Site Recovery，將 VMware VM 和實體伺服器容錯移轉到 Azure，然後容錯回復到內部部署網站
services: site-recovery
author: rayne-wiselman
manager: carmonm
ms.service: site-recovery
ms.topic: tutorial
ms.date: 09/11/2018
ms.author: raynew
ms.custom: MVC
ms.openlocfilehash: e9ed0ba8d24f30f67dbb315848dc4c260cae4f50
ms.sourcegitcommit: 794bfae2ae34263772d1f214a5a62ac29dcec3d2
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/11/2018
ms.locfileid: "44391363"
---
# <a name="fail-over-and-fail-back-vmware-vms-and-physical-servers-replicated-to-azure"></a>將複寫的 VMware VM 和實體伺服器容錯移轉及容錯回復至 Azure

本教學課程說明如何將 VMware VM 容錯移轉至 Azure。 在容錯移轉之後，您可以容錯回復至可用的內部部署網站。 在本教學課程中，您了解如何：

> [!div class="checklist"]
> * 確認要檢查的 VMware VM 屬性符合 Azure 需求
> * 執行容錯移轉到 Azure
> * 建立容錯回復的處理序伺服器和主要目標伺服器
> * 將 Azure VM 放到內部部署網站重新保護
> * 從 Azure 容錯移轉至內部部署網站
> * 重新保護內部部署 VM，再次開始複寫至 Azure

>[!NOTE]
>這些教學課程皆設計成顯示案例的最簡單部署路徑。 可能的話，會使用預設選項，而不會顯示所有可能的設定與路徑。 如果您想要深入了解測試容錯移轉的步驟，請參閱[操作指南](site-recovery-failover.md)。

這是本系列的第五個教學課程。 本教學課程假設您已經完成了先前教學課程中的工作。

1. [準備 Azure](tutorial-prepare-azure.md)
2. [準備內部部署 VMware](vmware-azure-tutorial-prepare-on-premises.md)
3. [設定災害復原](vmware-azure-tutorial.md)
4. [執行災害復原演練](tutorial-dr-drill-azure.md)
5. 除了上述的步驟，最好先針對此災害復原案例[檢閱架構](vmware-azure-architecture.md)。

## <a name="failover-and-failback"></a>容錯移轉和容錯回復

容錯移轉和容錯回復有四個階段：

1. **容錯移轉至 Azure**：將機器從內部部署網站容錯移轉至 Azure。
2. **重新保護 Azure VM**：重新保護 Azure VM，使其開始複寫回到內部部署 VMware VM。 內部部署 VM 會在重新保護期間關閉。 這有助於確保資料在複寫期間的一致性。
3. **容錯移轉至內部部署環境**：執行容錯移轉，以從 Azure 失敗容錯回復。
4. **重新保護內部部署 VM**：在資料容錯回復之後，請重新保護已容錯回復到的內部部署 VM，使其開始複寫至 Azure。

## <a name="verify-vm-properties"></a>驗證 VM 屬性

驗證 VM 屬性，並確定 VM 符合 [Azure 需求](vmware-physical-azure-support-matrix.md#replicated-machines)。

1. 在 [受保護的項目] 中，按一下 [複寫的項目] > VM。

2. 在 [複寫的項目] 窗格中，將會呈現 VM 資訊、健康情況狀態，以及最新可用復原點的摘要。 如需檢視詳細資訊，請按一下 [屬性]。

3. 在 [計算與網路] 中，您可以修改 Azure 的名稱、資源群組、目標大小、[可用性設定組](../virtual-machines/windows/tutorial-availability-sets.md)及[受控磁碟設定](#managed-disk-considerations)。

4. 您可以檢視及修改網路設定，包括在容錯移轉後 Azure VM 所在的網路/子網路，以及要指派給它的 IP 位址。

5. 在 [磁碟] 中，您可以看見 VM 上作業系統和資料磁碟的相關資訊。

## <a name="run-a-failover-to-azure"></a>執行容錯移轉到 Azure

1. 在 [設定] > [複寫的項目] 中，按一下 [VM] > [容錯移轉]。

2. 在 [容錯移轉] 中，選取要容錯移轉的目標**復原點**。 您可以使用下列其中一個選項：
   - **最新**：此選項會先處理所有傳送至 Site Recovery 的資料。 它會提供最低的 RPO (復原點目標)，因為在容錯移轉後建立的 Azure VM 具有在觸發容錯移轉時複寫到 Site Recovery 的所有資料。
   - **最近處理**：此選項會將 VM 容錯移轉到 Site Recovery 所處理的最新復原點。 此選項提供低 RTO (復原時間目標)，因為無須花費時間處理未處理的資料。
   - **最新應用程式一致**：此選項會將 VM 容錯移轉到 Site Recovery 所處理的最近應用程式一致復原點。
   - **自訂**：指定任何復原點。

3. 選取 [先將機器關機再開始容錯移轉]，以在觸發容錯移轉之前，嘗試將來源虛擬機器關機。 即使關機失敗，仍會繼續容錯移轉。 您可以 [作業] 頁面上追蹤容錯移轉進度。

在某些情況下，容錯移轉需要額外的處理，這會耗費約 8 到 10 分鐘的時間來完成。 對於使用 9.8 以前版本行動服務的 VMware 虛擬機器、實體伺服器、VMware Linux 虛擬機器、受保護作為實體伺服器的 Hyper-V 虛擬機器、未啟用 DHCP 服務的 VMware VM，以及不包含下列開機驅動程式的 VMware VM：storvsc、vmbus、storflt、intelide、atapi，您可能會發現**測試容錯移轉時間較長**。

> [!WARNING]
> **不要取消正在進行的容錯移轉**：在啟動容錯移轉之前，已停止 VM 複寫。
> 如果您取消正在進行的容錯移轉，容錯移轉會停止，但 VM 不會再次複寫。

## <a name="connect-to-failed-over-virtual-machine-in-azure"></a>連線到 Azure 中的容錯移轉虛擬機器

1. 如果您想要在容錯移轉後使用 RDP/SSH 連線到 Azure VM，請遵循資料表 ([這裡](site-recovery-test-failover-to-azure.md#prepare-to-connect-to-azure-vms-after-failover)) 中摘要說明的需求。
2. 容錯移轉之後，請移至虛擬機器，並[連線](../virtual-machines/windows/connect-logon.md)至虛擬機器來進行驗證。
3. 在驗證之後，按一下 [認可] 來完成容錯移轉後的虛擬機器復原點。 在認可之後，系統會刪除所有其他可用的復原點。 如此即完成容錯移轉活動。

>[!TIP]
> 如果您不滿意容錯移轉的虛擬機器，則**變更復原點**可協助您在容錯移轉之後選擇不同的復原點。 在**認可**之後，將無法再使用此選項。

請依照[這裡](site-recovery-failover-to-azure-troubleshoot.md)所述的步驟，針對容錯移轉後的任何連線問題進行疑難排解。

## <a name="preparing-for-reprotection-of-azure-vm"></a>準備重新保護 Azure VM

### <a name="create-a-process-server-in-azure"></a>在 Azure 中建立處理序伺服器

處理序伺服器會從 Azure VM 接收資料，並將資料傳送至內部部署網站。 低延遲網路必須位於處理序伺服器與受保護的 VM 之間。

- 基於測試目的，如果您有 Azure ExpressRoute 連線，則可以使用在設定伺服器上自動安裝的內部部署處理序伺服器 (內建的處理序伺服器)。
- 如果您有 VPN 連線，或正在生產環境中執行容錯回復，則必須將 Azure VM 設定為 Azure 型處理序伺服器，以進行容錯回復。
- 若要在 Azure 中設定處理序伺服器，請依照[本文](vmware-azure-set-up-process-server-azure.md)中的指示操作。

### <a name="configure-the-master-target-server"></a>設定主要目標伺服器

主要目標伺服器會在從 Azure 容錯回復期間接收並處理複寫資料。 依預設，會在內部部署設定伺服器上提供。 在本教學課程中，我們將使用預設的主要目標伺服器。

>[!NOTE]
>保護 Linux 型虛擬機器需要建立個別的主要目標伺服器。 請按一下[這裡](vmware-azure-install-linux-master-target.md)深入了解。

如果 VM 在 **vCenter 伺服器所管理的 ESXi 主機上**，則主要目標伺服器必須能存取 VM 的資料存放區 (VMDK)，以將複寫的資料寫入至 VM 磁碟。 確定 VM 資料存放區已掛接在主要目標的主機上，並具有讀寫權限。

如果 VM 不在 **vCenter 伺服器所管理的 ESXi** 上，則 Site Recovery 服務會在重新保護期間建立新的 VM。 此 VM 建立於您建立主要目標的 ESX 主機上。
VM 的硬碟必須位於可供主要目標伺服器執行所在主機存取的資料存放區中。

如果 VM **未使用 vCenter**，您應該先在主要目標伺服器執行所在的主機完成探索，才能重新保護機器。 這也適用於容錯回復實體伺服器。 另一個選項 (如果內部部署 VM 存在) 是先將其刪除，然後才執行容錯回復。 接著，容錯回復會在與主要目標 ESX 主機相同的主機上建立新的 VM。 當您容錯回復至替代位置時，資料將會復原到內部部署主要目標伺服器所使用的相同資料存放區和相同 ESX 主機上。

您無法使用主要目標伺服器的儲存體 vMotion。 如果您這麼做，容錯回復將無法運作，因為它無法使用磁碟。 請從 vMotion 清單中排除主要目標伺服器。

>[!Warning]
>如果使用不同的主要目標伺服器重新保護複寫群組，則伺服器無法提供共同的時間點。

## <a name="reprotect-azure-vms"></a>重新保護 Azure VM

重新保護 Azure VM 會將資料複寫到內部部署 VM。 當您從 Azure 中執行容錯移轉至內部部署 VM 時，這是必要的步驟。 請遵循下面提供的指示，以執行重新保護。

1. 在 [設定] > [複寫的項目] 中，以滑鼠右鍵按一下已容錯移轉的 VM > [重新保護]。
2. 在 [重新保護] 中，確認已選取 [Azure 到內部部署]。
3. 指定內部部署主要目標伺服器，以及處理序伺服器。
4. 在 [資料存放區] 中，選取您要將內部部署磁碟復原至的主要目標資料存放區。 如果系統已刪除 VM，則會在此資料存放區上建立新的磁碟。 如果磁碟已存在，則會忽略此設定，但您仍然需要指定一個值。
5. 選取主要目標保留磁碟機。 系統會自動選取容錯回復原則。
6. 按一下 [確定] 以開始重新保護。 隨即開始執行將虛擬機器從 Azure 複寫到內部部署網站的作業。 您可以在 [作業]  索引標籤上追蹤進度。
7. 當 VM 狀態在 [複寫的項目] 上變更為 [受保護] 之後，機器便就緒可容錯移轉至內部部署。

> [!NOTE]
> Azure VM 可以復原到現有的內部部署 VM 或替代位置。 閱讀[本文](concepts-types-of-failback.md)以深入了解。

## <a name="run-a-failover-from-azure-to-on-premises"></a>執行從 Azure 容錯移轉至內部部署環境

若要複寫回到內部部署環境，則需使用容錯回復原則。 此原則會在您建立可供複寫至 Azure 的複寫原則時自動建立：

- 此原則會自動與組態伺服器產生關聯。
- 無法修改此原則。
- 此原則的值如下：
    - RPO 臨界值 = 15 分鐘
    - 復原點保留 = 24 小時
    - 應用程式一致快照頻率 = 60 分鐘

執行容錯移轉，如下所示：

1. 在 [複寫的項目] 頁面上，以滑鼠右鍵按一下 [機器] > [容錯移轉]。
2. 在 [確認容錯移轉] 中，確認容錯移轉方向是從 Azure。
    ![failover-direction](media/vmware-azure-tutorial-failover-failback/failover-direction.PNG)
3. 選取您要用於容錯移轉的復原點。 應用程式一致復原點發生在最近的時間點之前，而且它將會導致一些資料遺失。

    >[!WARNING]
    >執行容錯移轉時，Site Recovery 會關閉 Azure VM，並啟動內部部署 VM。 機器將會停機一段時間，所以請選擇適當的時間。

4. 您可以在 [復原服務保存庫] > [監視與報告] > [Site Recovery 作業] 上追蹤作業進度。
5. 完成容錯移轉之後，以滑鼠右鍵按一下虛擬機器，然後按一下 [認可]。 這會觸發可移除 Azure VM 的作業。
6. 確認 Azure VM 已如預期般關閉。

## <a name="reprotect-on-premises-machines-to-azure"></a>將內部部署機器放到 Azure 重新保護

資料現在應該回到內部部署網站上，但不會複寫至 Azure。 您可以再次開始複寫至 Azure，如下所示：

1. 在 [保存庫] > [受保護項目] >[複寫的項目] 中，選取容錯回復的 VM，然後按一下 [重新保護]。
2. 選取要用於將已複寫資料傳送至 Azure 的處理序伺服器，然後按一下 [確定]。

重新保護完成之後，VM 會複寫回到 Azure，而您可以視需要執行容錯移轉。
