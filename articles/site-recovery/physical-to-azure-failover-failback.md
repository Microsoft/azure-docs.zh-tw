---
title: 使用 Site Recovery 將實體伺服器容錯移轉及容錯回復至 Azure | Microsoft Docs
description: 了解如何使用 Site Recovery，將實體伺服器容錯移轉到 Azure，然後容錯回復到內部部署網站
services: site-recovery
author: rayne-wiselman
ms.service: site-recovery
ms.topic: article
ms.date: 09/11/2018
ms.author: raynew
ms.openlocfilehash: 4036ab6e62f4738f4b2906eb7571dc5d0e972988
ms.sourcegitcommit: 794bfae2ae34263772d1f214a5a62ac29dcec3d2
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/11/2018
ms.locfileid: "44391142"
---
# <a name="fail-over-and-fail-back-physical-servers-replicated-to-azure"></a>將複寫的實體伺服器容錯移轉及容錯回復至 Azure

本教學課程說明如何將實體伺服器容錯移轉至 Azure。 在容錯移轉之後，您可以將伺服器容錯回復至可用的內部部署網站。

## <a name="preparing-for-failover-and-failback"></a>準備容錯移轉和容錯回復

使用站台復原複寫至 Azure 的實體伺服器，僅可以容錯回復為 VMware VM。 您需要 VMware VM 基礎結構，才能容錯回復。

容錯移轉和容錯回復有四個階段：

1. **容錯移轉至 Azure**：將機器從內部部署網站容錯移轉至 Azure。
2. **重新保護 Azure VM**：重新保護 Azure VM，使其開始複寫回到內部部署 VMware VM。
3. **容錯移轉至內部部署環境**：執行容錯移轉，以從 Azure 失敗容錯回復。
4. **重新保護內部部署 VM**：在資料容錯回復之後，請重新保護已容錯回復到的內部部署 VMware VM，使其開始複寫至 Azure。

## <a name="verify-server-properties"></a>驗證伺服器屬性

驗證伺服器屬性，並確定伺服器符合 Azure VM 的 [Azure 需求](vmware-physical-azure-support-matrix.md#replicated-machines)。

1. 在 [受保護的項目]中，按一下 [複寫的項目]，然後選取機器。

2. 在 [複寫的項目] 窗格中，將會呈現機器資訊、健康情況狀態，以及最新可用復原點的摘要。 如需檢視詳細資訊，請按一下 [屬性]。
3. 在 [計算與網路] 中，您可以修改 Azure 的名稱、資源群組、目標大小、[可用性設定組](../virtual-machines/windows/tutorial-availability-sets.md)及[受控磁碟設定](#managed-disk-considerations)。
4. 您可以檢視及修改網路設定，包括在容錯移轉後 Azure VM 所在的網路/子網路，以及要指派給它的 IP 位址。
5. 在 [磁碟] 中，您可以看見機器作業系統和資料磁碟的相關資訊。

## <a name="run-a-failover-to-azure"></a>執行容錯移轉到 Azure

1. 在 [設定] > [複寫的項目] 中，按一下機器 > [容錯移轉]。
2. 在 [容錯移轉] 中，選取容錯移轉的目標**復原點**。 您可以使用下列其中一個選項：
   - **最新**：此選項會先處理所有傳送至 Site Recovery 的資料。 它會提供最低的 RPO (復原點目標)，因為在容錯移轉後建立的 Azure VM 具有在觸發容錯移轉時複寫到 Site Recovery 的所有資料。
   - **最近處理**：此選項會將機器容錯移轉到 Site Recovery 所處理的最新復原點。 此選項提供低 RTO (復原時間目標)，因為無須花費時間處理未處理的資料。
   - **最新應用程式一致**：此選項會將機器容錯移轉到 Site Recovery 所處理的最近應用程式一致復原點。
   - **自訂**：指定任何復原點。

3. 如果想在觸發容錯移轉之前，讓 Site Recovery 嘗試將來源 VM 關機，請選取 [先將機器關機再開始容錯移轉]。 即使關機失敗，仍會繼續容錯移轉。 您可以 [作業] 頁面上追蹤容錯移轉進度。
4. 如果您已準備好連線到 Azure VM，請進行連線以在容錯移轉之後加以驗證。
5. 驗證之後，請 [認可] 容錯移轉。 這會刪除所有可用的復原點。

> [!WARNING]
> 請勿取消正在進行中的容錯移轉。 容錯移轉開始之前，就會停止機器複寫。 如果您取消容錯移轉，容錯移轉會隨即停止，但機器不會重新複寫一次。
> 若是實體伺服器，其他的容錯移轉可能需要處理約 8 到 10 分鐘才能完成。

## <a name="prepare-to-connect-to-azure-vms-after-failover"></a>準備在容錯移轉後連接到 Azure VM

如果您想要在容錯移轉後使用 RDP/SSH 連線到 Azure VM，請遵循資料表 ([這裡](site-recovery-test-failover-to-azure.md#prepare-to-connect-to-azure-vms-after-failover)) 中摘要說明的需求。

請依照[這裡](site-recovery-failover-to-azure-troubleshoot.md)所述的步驟，對容錯移轉後的連線問題進行疑難排解。

## <a name="create-a-process-server-in-azure"></a>在 Azure 中建立處理序伺服器

處理序伺服器會從 Azure VM 接收資料，並將資料傳送至內部部署網站。 低延遲網路必須位於處理序伺服器與受保護的機器之間。

- 基於測試目的，如果您有 Azure ExpressRoute 連線，您可以使用在組態伺服器上自動安裝的內部部署處理序伺服器。
- 如果您有 VPN 連線，或正在生產環境中執行容錯回復，則必須將 Azure VM 設定為以 Azure 為基礎的處理序伺服器進行容錯回復。
- 若要在 Azure 中設定處理序伺服器，請依照[本文](vmware-azure-set-up-process-server-azure.md)中的指示操作。

## <a name="configure-the-master-target-server"></a>設定主要目標伺服器

根據預設，主要目標伺服器會接收容錯回復資料。 它會在內部部署組態伺服器上執行。

- 如果您容錯回復的 VMware VM 在 VMware vCenter 伺服器所管理的 ESXi 主機上，則主要目標伺服器必須能存取 VM 的資料存放區 (VMDK)，以將複寫的資料寫入至 VM 磁碟。 確定 VM 資料存放區已掛接在主要目標的主機上，並具有讀寫權限。
- 如果 ESXi 主機不受 vCenter 伺服器管理，則 Site Recovery 服務會在重新保護期間建立新的 VM。 此 VM 建立於您建立主要目標 VM 的 ESX 主機上。 VM 的硬碟必須位於可供主要目標伺服器執行所在主機存取的資料存放區中。
- 針對您容錯回復的實體機器，您應該先完成主要目標伺服器執行所在主機的探索，才能重新保護機器。
- 另一個選項 (如果內部部署 VM 已存在以進行容錯回復) 是先將其刪除，然後才執行容錯回復。 接著，容錯回復會在與主要目標 ESX 主機相同的主機上建立新的 VM。 當您容錯回復至替代位置時，資料將會復原到內部部署主要目標伺服器所使用的相同資料存放區和相同 ESX 主機上。
- 您無法使用主要目標伺服器的儲存體 vMotion。 如果您這麼做，容錯回復將無法運作，因為它無法使用磁碟。 請從 vMotion 清單中排除主要目標伺服器。

## <a name="reprotect-azure-vms"></a>重新保護 Azure VM

此程序假設內部部署 VM 無法使用，而且您將它放到替代位置重新保護。

1. 在 [設定] > [已複寫的項目] 中，以滑鼠右鍵按一下已容錯移轉的 VM > [重新保護]。
2. 在 [重新保護] 中，確認已選取 [Azure 到內部部署]。
3. 指定內部部署主要目標伺服器，以及處理序伺服器。

4. 在 [資料存放區] 中，選取您要將內部部署磁碟復原至的主要目標資料存放區。 當內部部署 VM 遭到刪除且您需要建立新的磁碟時，請使用此選項。 如果磁碟已存在，則會忽略此設定，但您仍然需要指定一個值。
5. 選取主要目標保留磁碟機。 系統會自動選取容錯回復原則。
6. 按一下 [確定] 以開始重新保護。 隨即開始執行將虛擬機器從 Azure 複寫到內部部署網站的作業。 您可以在 [作業]  索引標籤上追蹤進度。

> [!NOTE]
> 若要將 Azure VM 復原到現有的內部部署 VM，請在主要目標伺服器的 ESXi 主機上掛接內部部署虛擬機器的資料存放區 (具有讀寫權限)。


## <a name="run-a-failover-from-azure-to-on-premises"></a>執行從 Azure 容錯移轉至內部部署環境

若要複寫回到內部部署環境，則需使用容錯回復原則。 此原則會在您建立可供複寫至 Azure 的複寫原則時自動建立：

- 此原則會自動與組態伺服器產生關聯。
- 無法修改此原則。
- 此原則的值如下：
    - RPO 臨界值 = 15 分鐘
    - 復原點保留 = 24 小時
    - 應用程式一致快照頻率 = 60 分鐘

執行容錯移轉，如下所示：

1. 在 [已複寫的項目] 頁面上，以滑鼠右鍵按一下機器 > [非計劃性容錯移轉]。
2. 在 [確認容錯移轉] 中，確認容錯移轉方向是從 Azure。

3. 選取您要用於容錯移轉的復原點。 應用程式一致復原點發生在最近的時間點之前，而且它將會導致一些資料遺失。 執行容錯移轉時，Site Recovery 會關閉 Azure VM，並啟動內部部署 VM。 機器將會停機一段時間，所以請選擇適當的時間。
4. 在機器上按一下滑鼠右鍵，然後按一下 [認可]。 這會觸發可移除 Azure VM 的作業。
5. 確認 Azure VM 已如預期般關閉。


## <a name="reprotect-on-premises-machines-to-azure"></a>將內部部署機器放到 Azure 重新保護

資料現在應該回到內部部署網站上，但不會複寫至 Azure。 您可以再次開始複寫至 Azure，如下所示：

1. 在保存庫 > [設定] > > [複寫項目] 中，選取已容錯回復的 VM，然後按一下 [重新保護]。
2. 選取用於將複寫的資料傳送至 Azure 的處理序伺服器，然後按一下 [確定]。

重新保護完成之後，VM 會複寫回到 Azure，而您可以視需要執行容錯移轉。
