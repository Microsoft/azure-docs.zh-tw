---
title: 使用 Site Recovery 將複寫的 Hyper-V VM 容錯移轉及容錯回復至 Azure | Microsoft Docs
description: 了解如何使用 Site Recovery，將 Hyper-V VM 容錯移轉至 Azure，然後容錯回復至內部部署網站
services: site-recovery
author: rayne-wiselman
manager: carmonm
ms.service: site-recovery
ms.topic: tutorial
ms.date: 10/10/2018
ms.author: raynew
ms.custom: MVC
ms.openlocfilehash: 31de654e6746cecf5aedabbfe481ab99b2aa3510
ms.sourcegitcommit: 4b1083fa9c78cd03633f11abb7a69fdbc740afd1
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/10/2018
ms.locfileid: "49076978"
---
# <a name="failover-and-failback-hyper-v-vms-replicated-to-azure"></a>將 Hyper-V VM 容錯移轉及容錯回復至 Azure

本教學課程說明如何將 Hyper-V VM 容錯移轉至 Azure。 在容錯移轉之後，您可以容錯回復至可用的內部部署網站。 在本教學課程中，您了解如何：

> [!div class="checklist"]
> * 確認要檢查的 Hyper-V VM 屬性符合 Azure 需求
> * 執行容錯移轉到 Azure
> * 從 Azure 容錯回復至內部部署
> * 反向複寫內部部署 VM 以再次開始複寫至 Azure

此教學課程是本系列的第五個教學課程。 其內容假設您已經完成了先前教學課程中的工作。    

1. [準備 Azure](tutorial-prepare-azure.md)
2. [準備內部部署 Hyper-V](tutorial-prepare-on-premises-hyper-v.md)
3. 設定適用於 [Hyper-V VM](tutorial-hyper-v-to-azure.md)，或適用於 [System Center VMM 雲端中所管理的 Hyper-V VM](tutorial-hyper-v-vmm-to-azure.md) 的災害復原
4. [執行災害復原演練](tutorial-dr-drill-azure.md)

## <a name="prepare-for-failover-and-failback"></a>準備容錯移轉和容錯回復

請確認 VM 上有沒有快照集，且在容錯回復期間已將內部部署 VM 關閉。 這有助於確保資料在複寫期間的一致性。 請勿在容錯回復期間開啟內部部署 VM。 

容錯移轉和容錯回復有三個階段：

1. **容錯移轉至 Azure**：從內部部署網站將 Hyper-V VM 容錯移轉至 Azure。
2. **容錯回復至內部部署**：在內部部署網站可供使用時，將 Azure VM 容錯回復至內部部署網站。 它會開始將資料從 Azure 同步處理至內部部署，並在完成時，於內部部署上啟動 VM。  
3. **反向複寫內部部署 VM**：容錯回復至內部部署之後，請反向複寫內部部署 VM，以開始將這些 VM 複寫至 Azure。

## <a name="verify-vm-properties"></a>驗證 VM 屬性

在容錯移轉之前，請驗證 VM 屬性，並確定 VM 符合 [Azure 需求](hyper-v-azure-support-matrix.md#replicated-vms)。

在 [受保護的項目] 中，按一下 [複寫的項目] > VM。

2. 在 [複寫的項目] 窗格中，將會呈現 VM 資訊、健康情況狀態，以及最新可用復原點的摘要。 如需檢視詳細資訊，請按一下 [屬性]。

3. 在 [計算與網路] 中，您可以修改 Azure 的名稱、資源群組、目標大小、[可用性設定組](../virtual-machines/windows/tutorial-availability-sets.md)，及管理磁碟設定。

4. 您可以檢視及修改網路設定，包括在容錯移轉後 Azure VM 所在的網路/子網路，以及要指派給它的 IP 位址。

5. 在 [磁碟] 中，您可以看見 VM 上作業系統和資料磁碟的相關資訊。

## <a name="failover-to-azure"></a>容錯移轉至 Azure

1. 在 [設定] > [複寫的項目] 中，按一下 [VM] > [容錯移轉]。
2. 在 [容錯移轉] 中，選取 [最新的] 復原點。 
3. 選取 [Shut down machine before beginning failover] \(先將機器關機再開始容錯移轉)。 Site Recovery 會嘗試先關閉來源 VM 後，再觸發容錯移轉。 即使關機失敗，仍會繼續容錯移轉。 您可以 [作業] 頁面上追蹤容錯移轉進度。
4. 驗證容錯移轉之後，請按一下 [認可]。 這會刪除所有可用的復原點。

> [!WARNING]
> **請勿取消進行中的容錯移轉**：如果您在進行時取消，容錯移轉會停止，但 VM 不會再次複寫。

## <a name="failback-azure-vm-to-on-premises-and-reverse-replicate-the-on-premises-vm"></a>將 Azure VM 容錯回復至內部部署並反向複寫內部部署 VM

基本上，容錯回復作業是從 Azure 容錯移轉至內部部署網站，而在反向複寫中，此作業會再次開始將 VM 從內部部署網站複寫至 Azure。

1. 在 [設定] > [複寫的項目] 中，按一下 VM > [規劃的容錯移轉]。
2. 在 [確認規劃的容錯移轉] 中，確認容錯移轉方向 (從 Azure)，並選取來源和目標位置。
3. 選取 [在容錯移轉前同步處理資料 (僅同步處理差異變更)]。 此選項會降低 VM 停機時間，因為它會在不關閉 VM 的情況下同步處理。
4. 起始容錯移轉。 您可以在 [工作]  索引標籤上追蹤容錯移轉進度。
5. 在完成初始資料同步處理且您已經準備好關閉 Azure VM 之後，請按一下 [作業] > 規劃的容錯移轉作業名稱 > [完成容錯移轉]。 這會將 Azure VM 關機、將最新變更轉送至內部部署，然後啟動內部部署 VM。
6. 登入內部部署 VM 以檢查它如預期般可用。
7. 內部部署 VM 現在處於「認可擱置」狀態。 按一下 [認可]。 這會刪除 Azure VM 和它的磁碟，並準備內部部署 VM 來反轉複寫方向。
若要開始將內部部署 VM 複寫至 Azure，請啟用 [反向複寫]。 這會觸發複寫關閉 Azure VM 之後所發生的差異變更。  
