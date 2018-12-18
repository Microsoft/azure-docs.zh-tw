---
title: 使用 Azure Site Recovery 將內部部署機器移轉至 Azure | Microsoft Docs
description: 此文章說明如何使用 Azure Site Recovery 將內部部署機器移轉至 Azure。
services: site-recovery
author: rayne-wiselman
ms.service: site-recovery
ms.topic: tutorial
ms.date: 09/12/2018
ms.author: raynew
ms.custom: MVC
ms.openlocfilehash: bd41244192efa1333bc90bec8c00f38aaaa7f612
ms.sourcegitcommit: c29d7ef9065f960c3079660b139dd6a8348576ce
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/12/2018
ms.locfileid: "44714984"
---
# <a name="migrate-on-premises-machines-to-azure"></a>將內部部署機器移轉至 Azure

除了針對商務持續性和災害復原 (BCDR) 使用 [Azure Site Recovery](site-recovery-overview.md) 服務來管理和協調內部部署電腦和 Azure VM 的災害復原之外，您也可以使用 Site Recovery 來管理內部部署機器到 Azure 的移轉。


此教學課程說明如何將內部部署 VM 和實體伺服器移轉至 Azure。 在此教學課程中，您了解如何：

> [!div class="checklist"]
> * 選取複寫目標
> * 設定來源和目標環境
> * 設定複寫原則
> * 啟用複寫
> * 執行測試移轉，確定一切都沒問題
> * 執行一次性容錯移轉至 Azure

這是本系列的第三個教學課程。 此教學課程假設您已經完成了先前教學課程中的工作：

1. [準備 Azure](tutorial-prepare-azure.md)
2. 準備內部部署 [VMware](vmware-azure-tutorial-prepare-on-premises.md) 或 [Hyper-V] (hyper-v-prepare-on-premises-tutorial.md) 伺服器。

開始之前，最好先針對災害復原檢閱 [VMware](vmware-azure-architecture.md) 或 [Hyper-V](hyper-v-azure-architecture.md) 架構。


## <a name="prerequisites"></a>先決條件

不支援並行虛擬驅動程式所匯出的裝置。


## <a name="create-a-recovery-services-vault"></a>建立復原服務保存庫

1. 登入 [Azure 入口網站](https://portal.azure.com) > [復原服務]。
2. 按一下 [建立群組] > [監視和管理] > [備份和 Site Recovery]。
3. 在 [名稱] 中，指定易記名稱 [ContosoVMVault]。 如果您有多個訂用帳戶，請選取適當的一個。
4. 建立資源群組 **ContosoRG**。
5. 指定 Azure 區域。 若要查看支援的區域，請參閱 [Azure Site Recovery 定價詳細資料](https://azure.microsoft.com/pricing/details/site-recovery/)。
6. 若要從儀表板快速存取保存庫，請按一下 [釘選到儀表板]，然後按一下 [建立]。

   ![新增保存庫](./media/migrate-tutorial-on-premises-azure/onprem-to-azure-vault.png)

新的保存庫會新增到主要 [復原服務保存庫] 頁面上 [所有資源] 之下的 [儀表板]。



## <a name="select-a-replication-goal"></a>選取複寫目標

選取您要複寫的項目以及您要複寫到的位置。
1. 按一下 [復原服務保存庫] > 保存庫。
2. 在 [資源功能表] 中，按一下 [Site Recovery] > [準備基礎結構] > [保護目標]。
3. 在 [保護目標] 中，選取您需要移轉的項目。
    - **VMware**：選取 [至 Azure] > [es, with VMWare vSphere Hypervisor] \(是，使用 VMware vSphere Hypervisor)。
    - **實體機器**：選取 [至 Azure] > [未虛擬化/其他]。
    - **Hyper-V**：選取 [至 Azure] > [Yes, with Hyper-V] \(是，使用 Hyper-V)。 如果 Hyper-V VM 是由 VMM 管理，請選取 [是]。


## <a name="set-up-the-source-environment"></a>設定來源環境

- [設定](vmware-azure-tutorial.md#set-up-the-source-environment) VMware VM 的來源環境。
- [設定](physical-azure-disaster-recovery.md#set-up-the-source-environment)實體伺服器的來源環境。
- [設定](hyper-v-azure-tutorial.md#set-up-the-source-environment) Hyper-V VM 的來源環境。

## <a name="set-up-the-target-environment"></a>設定目標環境

選取並確認目標資源。

1. 按一下 [準備基礎結構] > [目標]，然後選取您要使用的 Azure 訂用帳戶。
2. 指定 Resource Manager 部署模型。
3. Site Recovery 會檢查您是否有一或多個相容的 Azure 儲存體帳戶和網路。

## <a name="set-up-a-replication-policy"></a>設定複寫原則

- [設定 VMware VM 的複寫原則](vmware-azure-tutorial.md#create-a-replication-policy)。
- [設定實體伺服器的複寫原則](physical-azure-disaster-recovery.md#create-a-replication-policy)。
- [設定 Hyper-V VM 的複寫原則](hyper-v-azure-tutorial.md#set-up-a-replication-policy)。


## <a name="enable-replication"></a>啟用複寫

- [啟用 VMware VM 的複寫](vmware-azure-tutorial.md#enable-replication)。
- [啟用實體伺服器的複寫功能](physical-azure-disaster-recovery.md#enable-replication)。
- 啟用 Hyper-V VM 的複寫 ([使用](hyper-v-vmm-azure-tutorial.md#enable-replication)或[不使用 VMM](hyper-v-azure-tutorial.md#enable-replication))。


## <a name="run-a-test-migration"></a>執行測試移轉

執行[測試容錯移轉](tutorial-dr-drill-azure.md)至 Azure，確定一切都沒問題。


## <a name="migrate-to-azure"></a>移轉至 Azure

針對您要移轉的機器執行容錯移轉。

1. 在 [設定] > [複寫的項目] 中，按一下機器 > [容錯移轉]。
2. 在 [容錯移轉] 中，選取要容錯移轉的目標**復原點**。 選取最新的復原點。
3. 加密金鑰設定不適用於此案例。
4. 選取 [Shut down machine before beginning failover] \(先將機器關機再開始容錯移轉)。 Site Recovery 在觸發容錯移轉之前，會嘗試將虛擬機器關機。 即使關機失敗，仍會繼續容錯移轉。 您可以 [作業] 頁面上追蹤容錯移轉進度。
5. 確認 Azure VM 如預期般出現在 Azure 中。
6. 在 [複寫的項目] 中，以滑鼠右鍵按一下 VM > [完成移轉]。 這會完成移轉程序、停止 VM 的複寫功能，並停止 VM 的 Site Recovery 計費。

    ![完成移轉](./media/migrate-tutorial-on-premises-azure/complete-migration.png)


> [!WARNING]
> **不要取消正在進行的容錯移轉**：在啟動容錯移轉之前，已停止 VM 複寫。 如果您取消正在進行的容錯移轉，容錯移轉會停止，但 VM 不會再次複寫。

在某些情況下，容錯移轉需要額外的處理，這會耗費約 8 到 10 分鐘的時間來完成。 您可能注意到下列項目的測試容錯移轉時間較久：實體伺服器、VMware Linux 機器、未啟用 DHCP 服務的 VMware VM，以及沒有下列開機驅動程式的 VMware VM：storvsc、vmbus、storflt、intelide、atapi。

## <a name="after-migration"></a>移轉之後

將機器移轉至 Azure 之後，有一些您應該完成的步驟。

藉由使用[復原方案]( https://docs.microsoft.com/azure/site-recovery/site-recovery-runbook-automation)中的內建自動化指令碼功能，可將一些步驟自動化而成為移轉程序的一部分   


### <a name="post-migration-steps-in-azure"></a>Azure 中的移轉後步驟

- 執行任何移轉後應用程式調整，例如更新資料庫連接字串和 Web 伺服器設定。 
- 在現在於 Azure 中執行的已移轉應用程式上，執行最終的應用程式和移轉接受度測試。
- [Azure VM 代理程式](https://docs.microsoft.com/azure/virtual-machines/extensions/agent-windows)可管理 VM 與「Azure 網狀架構控制器」之間的互動。 它對某些 Azure 服務 (例如 Azure 備份、Site Recovery 及 Azure 安全性) 來說是必要的。
    - 如果您要移轉 VMware 機器和實體伺服器，「行動服務」安裝程式就會在 Windows 機器上安裝可用的 Azure VM 代理程式。 在 Linux VM 上，建議您在容錯移轉後安裝代理程式。 a
    - 如果您要將 Azure VM 移轉至次要區域，就必須在移轉前將 Azure VM 代理程式佈建在 VM 上。
    - 如果您要將 Hyper-V VM 移轉至 Azure，請在移轉後將 Azure VM 代理程式安裝在 Azure VM 上。
- 請從 VM 中手動移除任何 Site Recovery 提供者/代理程式。 如果您移轉 VMware VM 或實體伺服器，請從 Azure VM 中 [解除安裝行動服務][vmware-azure-install-mobility-service.md#uninstall-mobility-service-on-a-windows-server-computer]。
- 針對提升復原能力：
    - 使用「Azure 備份」服務來備份 Azure VM 以維護資料安全。 [深入了解]( https://docs.microsoft.com/azure/backup/quick-backup-vm-portal)。
    - 使用 Site Recovery 將 Azure VM 複寫至次要區域，讓工作負載保持執行且持續可供使用。 [深入了解](azure-to-azure-quickstart.md)。
- 針對提升安全性：
    - 使用「Azure 資訊安全中心」[Just-In-Time 系統管理]( https://docs.microsoft.com/azure/security-center/security-center-just-in-time)來鎖定並限制輸入流量存取
    - 使用[網路安全性群組](https://docs.microsoft.com/azure/virtual-network/security-overview)來限制傳送至管理端點的網路流量。
    - 部署 [Azure 磁碟加密](https://docs.microsoft.com/azure/security/azure-security-disk-encryption-overview)以協助保護磁碟，以及防止資料遭到竊取和受到未經授權的存取。
    - 深入了解如何[保護 IaaS 資源]( https://azure.microsoft.com/services/virtual-machines/secure-well-managed-iaas/ )，並瀏覽 [Azure 資訊安全中心](https://azure.microsoft.com/services/security-center/ )。
- 針對監視及管理：
    - 可考慮部署 [Azure 成本管理](https://docs.microsoft.com/azure/cost-management/overview)來監視資源使用情況和花費。

### <a name="post-migration-steps-on-premises"></a>內部部署環境中的移轉後步驟

- 將應用程式流量移至在已移轉之 Azure VM 執行個體上執行的應用程式。
- 從您的本機 VM 清查中移除內部部署 VM。
- 從本機備份中移除內部部署 VM。
- 更新任何內部文件，以顯示 Azure VM 的新位置和 IP 位址。


## <a name="next-steps"></a>後續步驟

在此教學課程中，您已將內部部署 VM 移轉至 Azure VM。 現在，您可以為 Azure VM [設定災害復原](azure-to-azure-replicate-after-migration.md)來復原至次要 Azure 區域。

  
