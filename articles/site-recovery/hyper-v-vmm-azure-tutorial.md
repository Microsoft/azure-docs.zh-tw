---
title: 使用 Azure Site Recovery 將 VMM 雲端中內部部署 Hyper-V VM 的災害復原設定至 Azure | Microsoft Docs
description: 了解如何使用 Azure Site Recovery 服務，將 System Center VMM 雲端中內部部署 Hyper-V VM 的災害復原設定至 Azure。
services: site-recovery
author: rayne-wiselman
ms.service: site-recovery
ms.topic: article
ms.date: 07/06/2018
ms.author: raynew
ms.custom: MVC
ms.openlocfilehash: a6be34d96ec833005ff191fe7477b117ec045a04
ms.sourcegitcommit: 248c2a76b0ab8c3b883326422e33c61bd2735c6c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/23/2018
ms.locfileid: "39213737"
---
# <a name="set-up-disaster-recovery-of-on-premises-hyper-v-vms-in-vmm-clouds-to-azure"></a>將 VMM 雲端中 Hyper-V VM 的災害復原設定至 Azure

[Azure Site Recovery](site-recovery-overview.md) 服務可藉由管理及協調內部部署電腦與 Azure 虛擬機器 (VM) 的複寫、容錯移轉及容錯回復，為您的災害復原策略做出貢獻。

本教學課程說明如何設定內部部署 Hyper-V VM 至 Azure 的災害復原。 本教學課程適用於由 System Center Virtual Machine Manager (VMM) 控管的 Hyper-V VM。 在本教學課程中，您了解如何：

> [!div class="checklist"]
> * 選取複寫來源和目標。
> * 設定來源複寫環境，包括內部部署 Site Recovery 元件和目標複寫環境。
> * 設定網路對應，以在 VMM VM 網路與 Azure 虛擬網路之間進行對應。
> * 建立複寫原則
> * 啟用 VM 複寫

這是本系列的第三個教學課程。 本教學課程假設您已經完成了先前教學課程中的工作：

1. [準備 Azure](tutorial-prepare-azure.md)
2. [準備內部部署 Hyper-V](tutorial-prepare-on-premises-hyper-v.md)

開始之前，最好先針對此災害復原案例[檢閱架構](concepts-hyper-v-to-azure-architecture.md)。



## <a name="select-a-replication-goal"></a>選取複寫目標

1. 在 [所有服務] > [復原服務保存庫] 中，按一下我們在這些教學課程中使用的保存庫名稱 **ContosoVMVault**。
2. 在 [使用者入門] 中，按一下 [Site Recovery]。 然後按一下 [準備基礎結構]
3. 在 [保護目標] > [您的電腦位於何處] 中，選取 [內部部署]。
4. 在 [您要將電腦複寫到何處] 中，選取 [複製到 Azure]。
5. 在 [您的電腦虛擬化了嗎] 中，選取 [是，使用 Hyper-V]。
6. 在 [您是否使用 System Center VMM]，選取 [是]。 然後按一下 [確定] 。

    ![複寫目標](./media/hyper-v-vmm-azure-tutorial/replication-goal.png)



## <a name="set-up-the-source-environment"></a>設定來源環境

設定來源環境時，您需要安裝 Azure Site Recovery Provider 和 Azure 復原服務代理程式，並在保存庫中註冊內部部署伺服器。 

1. 在 [準備基礎結構] 中，按一下 [來源]。
2. 在 [準備來源] 中，按一下 [+ VMM] 以新增 VMM 伺服器。 在 [新增伺服器] 中，檢查 [System Center VMM 伺服器] 是否出現在 [伺服器類型] 中。
3. 下載 Microsoft Azure Site Recovery 提供者的安裝程式。
4. 下載保存庫註冊金鑰。 您執行提供者安裝程式時需要此金鑰。 該金鑰在產生後會維持 5 天有效。
5. 下載復原服務代理程式。

    ![下載](./media/hyper-v-vmm-azure-tutorial/download-vmm.png)

### <a name="install-the-provider-on-the-vmm-server"></a>在 VMM 伺服器上將提供者解除安裝

1. 在 Azure Site Recovery Provider 安裝精靈 > **Microsoft Update** 中，選擇使用 Microsoft Update 來檢查 Provider 更新。
2. 在 [安裝] 中，接受 Provider 的預設安裝位置，然後按一下 [安裝]。 
3. 安裝之後，在 [Microsoft Azure Site Recovery 註冊精靈] > [保存庫設定] 中，按一下 [瀏覽]，然後在 [金鑰檔案] 中，選取您下載的保存庫金鑰檔。
4. 指定 Azure Site Recovery 訂用帳戶和保存庫名稱 (**ContosoVMVault**)。 為 VMM 伺服器指定易記名稱，以便在保存庫中加以識別。
5. 在 [Proxy 設定] 中，選取 [不使用 Proxy 而直接連線到 Azure Site Recovery]。
6. 接受用來加密資料的憑證預設位置。 當您容錯移轉時，即會將加密的資料解密。
7. 在 [同步雲端中繼資料] 中，選取 [請將雲端中繼資料同步至 Site Recovery 入口網站]。 這個動作只需要在每個伺服器上進行一次。 然後按一下 [註冊]。
8. 在保存庫中註冊伺服器之後，請按一下 [完成]。

註冊完成之後，Azure Site Recovery 會取出來自伺服器的中繼資料，而該 VMM 伺服器會顯示在 [Site Recovery 基礎結構] 中。

### <a name="install-the-recovery-services-agent"></a>安裝復原服務代理程式

在每部包含您想要複寫之 VM 的 Hyper-V 主機上安裝代理程式。

1. 在 [Microsoft Azure 復原服務代理程式安裝精靈] > [檢查必要條件] 中，按 [下一步]。 將自動安裝任何缺少的必要條件。
2. 在 [安裝設定] 中，接受安裝位置和快取位置。 快取磁碟機至少需要 5 GB 的儲存體。 我們建議磁碟機要有 600 GB 或更多的可用空間。 然後按一下 [安裝] 。
3. 在 [安裝] 中，當安裝完成時，請按一下 [關閉] 以完成精靈。

    ![安裝代理程式](./media/hyper-v-vmm-azure-tutorial/mars-install.png)
    

## <a name="set-up-the-target-environment"></a>設定目標環境

1. 按一下 [準備基礎結構] > [目標]。
2. 選取容錯移轉之後，將在其中建立 Azure VM 的訂用帳戶和資源群組 (**ContosoRG**)。
3. 選取 [資源管理員] 部署模型。

Site Recovery 會檢查您是否有一或多個相容的 Azure 儲存體帳戶和網路。


## <a name="configure-network-mapping"></a>設定網路對應

1. 在 [Site Recovery 基礎結構]  >  [網路對應]  >  [網路對應]中，按一下 [+網路對應] 圖示。
2. 在 [新增網路對應] 中，選取來源 VMM 伺服器。 選取 [Azure] 作為目標。
3. 在容錯移轉後確認訂用帳戶和部署模型。
4. 在 [來源網路] 中，選取來源內部部署 VM 網路。
5. 在 [目標網路] 中，選取 Azure 網路來放置容錯移轉之後建立的複本 Azure VM。 然後按一下 [確定] 。

    ![網路對應](./media/hyper-v-vmm-azure-tutorial/network-mapping-vmm.png)

## <a name="set-up-a-replication-policy"></a>設定複寫原則

1. 按一下 [準備基礎結構] > [複寫設定] > [+建立並產生關聯]。
2. 在 [建立及關聯原則] 中，指定原則名稱 **ContosoReplicationPolicy**。
3. 保留預設值設定，然後按一下 [確定]。
    - [複製頻率] 指出差異資料 (在初始複寫之後) 將會每隔五分鐘複寫一次。
    - [復原點保留] 指定每個復原點的保留時間範圍會是兩小時。
    - [應用程式一致快照頻率]  指出將會每小時建立一次包含應用程式一致快照集的復原點。
    - [初始複寫開始時間] 指出初始複寫將會立即開始。
    - **將儲存在 Azure 上的資料加密**：預設的**關閉**設定指出不會加密 Azure 中的待用資料。
4. 建立原則之後，請按一下 [確定]。 當您建立新的原則時，該原則會自動與 VMM 雲端產生關聯。

## <a name="enable-replication"></a>啟用複寫

1. 在 [複寫應用程式] 中，按一下 [來源]。 
2. 在 [來源] 中，選取 VMM 雲端。 然後按一下 [確定] 。
3. 在 [目標] 中，確認目標為 Azure、保存庫訂用帳戶，然後選取 [Resource Manager] 模型。
4. 選取 **contosovmsacct1910171607** 儲存體帳戶和 **ContosoASRnet** Azure 網路。
5. 在 [虛擬機器] > [選取] 中，選取您要複寫的 VM。 然後按一下 [確定] 。

 您可以在 [作業] > [Site Recovery 作業] 中，追蹤 [啟用保護] 動作的進度。 在**完成保護**作業完成後，即完成初始複寫，且 VM 已可進行容錯移轉。


## <a name="next-steps"></a>後續步驟
[執行災害復原演練](tutorial-dr-drill-azure.md)
