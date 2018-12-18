---
title: 深入了解如何使用 Azure Site Recovery 為內部部署電腦及 Azure 執行災害復原演練 | Microsoft 文件
description: 深入了解如何使用 Azure Site Recovery 為內部部署電腦及 Azure 執行災害復原演練
author: rayne-wiselman
manager: carmonm
ms.service: site-recovery
ms.topic: tutorial
ms.date: 10/10/2018
ms.author: raynew
ms.openlocfilehash: 3be3631d8d917fe9ff85e8471a35ac2ddece80b7
ms.sourcegitcommit: 4b1083fa9c78cd03633f11abb7a69fdbc740afd1
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/10/2018
ms.locfileid: "49078149"
---
# <a name="run-a-disaster-recovery-drill-to-azure"></a>執行 Azure 的災害復原演練

在本文中，我們會示範如何使用測試容錯移轉執行內部部署機器至 Azure 的災害復原演練。 此演練可驗證您複寫策略未遺失資料。

這是一系列中的第四個教學課程，說明如何為內部部署 VMware VM 或 Hyper-V VM 設定 Azure 的災害復原。

此教學課程假設您已完成前三個教學課程：
    - 在[第一個教學課程](tutorial-prepare-azure.md)中，我們針對 VMware 災害復原需求設定 Azure 元件。
    - 在[第二個教學課程](vmware-azure-tutorial-prepare-on-premises.md)中，我們為災害復原準備了內部部署元件，並檢閱必要條件。
    - 在[第三個教學課程](vmware-azure-tutorial.md)中，我們為內部部署 VMware VM 設定和啟用複寫。
    - 這些教學課程的設計訴求是顯示**案例的最簡單部署路徑**。 可能的話，會使用預設選項，而不會顯示所有可能的設定與路徑。 如果您想要深入了解測試容錯移轉的步驟，請參閱[作法指南](site-recovery-test-failover-to-azure.md)。

在本教學課程中，您將了解如何：

> [!div class="checklist"]
> * 為測試容錯移轉設定隔離網路
> * 準備在容錯移轉後連線到 Azure VM
> * 執行單一機器測試容錯移轉



## <a name="verify-vm-properties"></a>驗證 VM 屬性

執行測試容錯移轉之前，請先驗證 VM 屬性，並確定 [Hyper-V VM](hyper-v-azure-support-matrix.md#replicated-vms) 或 [VMware VM](vmware-physical-azure-support-matrix.md#replicated-machines) 遵守 Azure 需求。

1. 在 [受保護的項目] 中，按一下 [複寫的項目] 然後按一下 [VM]。
2. 在 [複寫的項目] 窗格中，將會呈現 VM 資訊、健康情況狀態，以及最新可用復原點的摘要。 如需檢視詳細資訊，請按一下 [屬性]。
3. 在 [計算與網路] 中，您可以修改 Azure 的名稱、資源群組、目標大小、可用性設定組及受控磁碟設定。
4. 您可以檢視及修改網路設定，包括在容錯移轉後 Azure VM 所在的網路/子網路，以及要指派給它的 IP 位址。
5. 在 [磁碟] 中，您可以看見 VM 上作業系統和資料磁碟的相關資訊。

## <a name="create-a-network-for-test-failover"></a>建立測試容錯移轉的網路

我們建議針對測試容錯移轉所選擇的網路，是與每部 VM [計算與網路] 設定中特定實際執行復原網站網路隔離的網路。 根據預設，當您建立 Azure 虛擬網路時，它會與其他網路隔離。 此測試網路應模擬您的實際執行網路︰

- 測試網路應該具有與實際執行網路相同數目的子網路。 子網路應具有相同的名稱。
- 測試網路應使用相同的 IP 位址範圍。
- 使用針對 [計算與網路] 設定中 DNS VM 指定的 IP 位址，來更新測試網路的 DNS。 如需詳細資訊，請參閱 [Active Directory 測試容錯移轉考量](site-recovery-active-directory.md#test-failover-considerations)。

## <a name="run-a-test-failover-for-a-single-vm"></a>執行單一 VM 測試容錯移轉

當您執行測試容錯移轉時，會發生下列情況：

1. 程式會進行必要條件檢查，以確保所有容錯移轉所需之條件都已準備就緒。
2. 容錯移轉會處理資料，以便建立 Azure VM。 如果您選取最新的復原點，則會根據資料建立復原點。
3. 將會使用先前步驟中處理的資料來建立 Azure VM。

執行測試容錯移轉，如下所示：

1. 在 [設定] > [複寫的項目] 中，按一下 [VM] > [+測試容錯移轉]。
2. 為本教學課程選取**最後處理的**復原點。 這會將 VM 容錯移轉至最後可用的時間點。 隨即顯示時間戳記。 使用此選項時，無須花費時間處理資料，因此它會提供低 RTO (復原時間目標)。
3. 在 [測試容錯移轉] 中，選取 Azure VM 在容錯移轉之後要連線的目標 Azure 網路。
4. 按一下 [確定]  即可開始容錯移轉。 您可以按一下 VM 開啟其屬性以追蹤進度。 或者，您也可以按一下保存庫名稱中的 **[測試容錯移轉]** 工作 > **[設定]** > **[工作]** >
    **[Site Recovery 工作]**。
5. 容錯移轉完成之後，複本 Azure VM 會出現在 Azure 入口網站> [虛擬機器] 中。 確認 VM 為適當的大小、其已連線到正確的網路，而且正在執行中。
6. 您現在應該能夠連線到 Azure 中複寫的 VM。
7. 若要刪除測試容錯移轉期間建立的 Azure VM，請按一下 VM 上的 [清除測試容錯移轉]。 在 [記事] 中，記錄並儲存關於測試容錯移轉的任何觀察。

在某些情況下，容錯移轉需要額外的處理，這會耗費約 8 到 10 分鐘的時間來完成。 您可能注意到下列項目的測試容錯移轉時間較久：VMware Linux 機器、未啟用 DHCP 服務的 VMware VM，以及沒有下列開機驅動程式的 VMware VM：storvsc、vmbus、storflt、intelide、atapi。

## <a name="prepare-to-connect-to-azure-vms-after-failover"></a>準備在容錯移轉後連接到 Azure VM

如果您想要在容錯移轉後使用 RDP/SSH 連線到 Azure VM，請遵循資料表 ([這裡](site-recovery-test-failover-to-azure.md#prepare-to-connect-to-azure-vms-after-failover)) 中摘要說明的需求。

請依照[這裡](site-recovery-failover-to-azure-troubleshoot.md)所述的步驟，針對容錯移轉後的任何連線問題進行疑難排解。

## <a name="next-steps"></a>後續步驟

> [!div class="nextstepaction"]
> [在內部部署 VMware VM 執行容錯移轉和容錯回復](vmware-azure-tutorial-failover-failback.md)。
> [在內部部署 Hyper-V VM 執行容錯移轉和容錯回復](hyper-v-azure-failover-failback-tutorial.md)。
