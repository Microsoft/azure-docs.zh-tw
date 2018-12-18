---
title: 使用 Azure Site Recovery 設定內部部署 Hyper-V VM (不含 VMM) 至 Azure 的災害復原 | Microsoft Docs
description: 了解如何使用 Azure Site Recovery 服務來設定內部部署 Hyper-V VM (不含 VMM) 至 Azure 的災害復原。
services: site-recovery
author: rayne-wiselman
manager: carmonm
ms.service: site-recovery
ms.topic: tutorial
ms.date: 10/10/2018
ms.author: raynew
ms.custom: MVC
ms.openlocfilehash: 978b79fbd35178191ff5c95b8897045d00c7a272
ms.sourcegitcommit: 4b1083fa9c78cd03633f11abb7a69fdbc740afd1
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/10/2018
ms.locfileid: "49078031"
---
# <a name="set-up-disaster-recovery-of-on-premises-hyper-v-vms-to-azure"></a>設定 Hyper-V VM 至 Azure 的災害復原

[Azure Site Recovery](site-recovery-overview.md) 服務可藉由管理及協調內部部署電腦與 Azure 虛擬機器 (VM) 的複寫、容錯移轉及容錯回復，為您的災害復原策略做出貢獻。

本教學課程說明如何設定內部部署 Hyper-V VM 至 Azure 的災害復原。 本教學課程適用於不是由 System Center Virtual Machine Manager (VMM) 控管的 Hyper-V VM。 在本教學課程中，您了解如何：

> [!div class="checklist"]
> * 選取複寫來源和目標。
> * 設定來源複寫環境，包括內部部署 Site Recovery 元件和目標複寫環境。
> * 建立複寫原則。
> * 啟用 VM 複寫。

這是本系列的第三個教學課程。 本教學課程假設您已經完成了先前教學課程中的工作：

1. [準備 Azure](tutorial-prepare-azure.md)
2. [準備內部部署 Hyper-V](tutorial-prepare-on-premises-hyper-v.md)

開始之前，最好先針對此災害復原案例[檢閱架構](concepts-hyper-v-to-azure-architecture.md)。

## <a name="select-a-replication-goal"></a>選取複寫目標


1. 在 [所有服務] > [復原服務保存庫] 中，選取在前一個教學課程中備妥的保存庫 **ContosoVMVault**。
2. 在 [使用者入門] 中，按一下 [Site Recovery]。 然後按一下 [準備基礎結構]
3. 在 [保護目標] > [您的電腦位於何處] 中，選取 [內部部署]。
4. 在 [您要將電腦複寫到何處] 中，選取 [複製到 Azure]。
5. 在 [您是否使用 System Center VMM 來管理 Hyper-V 主機]，選取 [否]。 然後按一下 [確定] 。

    ![複寫目標](./media/hyper-v-azure-tutorial/replication-goal.png)

## <a name="confirm-deployment-planning"></a>確認部署規劃

當您規劃大型部署時，您應確定完成 [Hyper-V 複寫的部署規劃](hyper-v-deployment-planner-overview.md)。 針對教學課程的目的，在 [完成部署規劃了嗎?] 中，從下拉式清單選取 [稍後再進行]。

![部署規劃](./media/hyper-v-azure-tutorial/deployment-planning.png)

## <a name="set-up-the-source-environment"></a>設定來源環境

若要設定來源環境，您可以建立 Hyper-V 網站並將 Hyper-V 主機新增至該網站。 然後下載 Azure Site Recovery Provider 和 Azure 復原服務代理程式並安裝於每部主機上，然後在保存庫中註冊 Hyper-V 網站。 

1. 在 [準備基礎結構] 中，按一下 [來源]。
2. 按一下 [+Hyper-V 網站]，並指定在前一個教學課程中建立的網站名稱 **ContosoHyperVSite**。

    ![Hyper-V 站台](./media/hyper-v-azure-tutorial/hyperv-site.png)

3. 建立網站之後，按一下 [+Hyper-V 伺服器]。

    ![Hyper-V 伺服器](./media/hyper-v-azure-tutorial/hyperv-server.png)

4. 下載 Provider 安裝程式檔案。
6. 下載保存庫註冊金鑰。 您需要此金鑰才能執行提供者安裝程式。 該金鑰在產生後會維持 5 天有效。

    ![下載 Provider](./media/hyper-v-azure-tutorial/download.png)
    

### <a name="install-the-provider"></a>安裝 Provider

在您新增至 **ContosoHyperVSite** 網站的每部 Hyper-V 主機上執行 Provider 安裝程式檔案 (AzureSiteRecoveryProvider.exe)。 安裝程式會在每部 Hyper-V 主機上安裝 Azure Site Recovery Provider 和復原服務代理程式。

1. 在 Azure Site Recovery Provider 安裝精靈 > **Microsoft Update** 中，選擇使用 Microsoft Update 來檢查 Provider 更新。
2. 在 [安裝] 中，接受 Provider 和代理程式的預設安裝位置，然後按一下 [安裝]。
3. 安裝之後，在 Microsoft Azure Site Recovery 註冊精靈 > [保存庫設定] 中，按一下 [瀏覽]，然後在 [金鑰檔案] 中，選取您下載的保存庫金鑰檔。 
4. 指定 Azure Site Recovery 訂用帳戶、保存庫名稱 (**ContosoVMVault**)，以及 Hyper-V 伺服器所屬的 Hyper-V 網站 (**ContosoHyperVSite**)。
5. 在 [Proxy 設定] 中，選取 [不使用 Proxy 而直接連線到 Azure Site Recovery]。
6. 在保存庫中註冊伺服器之後，請於 [註冊] 中按一下 [完成]。

Azure Site Recovery 會取出來自 Hyper-V 伺服器的中繼資料，而該伺服器會顯示在 [Site Recovery 基礎結構] > [Hyper-V 主機] 中。 此程序可能需要 30 分鐘的時間。


## <a name="set-up-the-target-environment"></a>設定目標環境

選取並確認目標資源。 

1. 按一下 [準備基礎結構] > [目標]。
2. 選取容錯移轉之後，將在其中建立 Azure VM 的訂用帳戶和資源群組 **ContosoRG**。
3. 選取 [資源管理員] 部署模型。

Site Recovery 會檢查您是否有一或多個相容的 Azure 儲存體帳戶和網路。


## <a name="set-up-a-replication-policy"></a>設定複寫原則

> [!NOTE]
> 在 Hyper-V 至 Azure 的複寫原則中，15 分鐘的複製頻率選項已遭淘汰，改為 5 分鐘和 30 秒的複製頻率設定。 使用 15 分鐘複製頻率的複寫原則，將會自動更新為使用 5 分鐘的複製頻率設定。 相較於 15 分鐘的複製頻率，5 分鐘和 30 秒的複製頻率選項可提供改善的複寫效能和更佳的復原點目標，其對於頻寬使用量和資料傳輸量的影響最低。

1. 按一下 [準備基礎結構] > [複寫設定] > [+建立並產生關聯]。
2. 在 [建立及關聯原則] 中，指定原則名稱 **ContosoReplicationPolicy**。
3. 保留預設值設定，然後按一下 [確定]。
    - [複製頻率] 指出差異資料 (在初始複寫之後) 將會每隔五分鐘複寫一次。
    - [復原點保留] 指定每個復原點的保留時間範圍會是兩小時。
    - [應用程式一致快照頻率]  指出將會每小時建立一次包含應用程式一致快照集的復原點。
    - [初始複寫開始時間]  指出初始複寫將會立即開始。
4. 建立原則之後，請按一下 [確定]。 建立的新原則會自動與指定的 Hyper-V 網站 (**ContosoHyperVSite**) 產生關聯

    ![複寫原則](./media/hyper-v-azure-tutorial/replication-policy.png)


## <a name="enable-replication"></a>啟用複寫


1. 在 [複寫應用程式] 中，按一下 [來源]。 
2. 在 [來源] 中，選取 [ContosoHyperVSite] 網站。 然後按一下 [確定] 。
3. 在 [目標] 中，確認目標為 Azure、保存庫訂用帳戶，以及 [資源管理員] 部署模型。
4. 選取在前一個教學課程中為所複寫資料建立的 **contosovmsacct1910171607** 儲存體帳戶，以及 Azure VM 在容錯移轉之後所在的 **ContosoASRnet** 網路。
5. 在 [虛擬機器] > [選取] 中，選取您要複寫的 VM。 然後按一下 [確定] 。

 您可以在 [作業] > [Site Recovery 作業] 中，追蹤 [啟用保護] 動作的進度。 在**完成保護**作業完成後，即完成初始複寫，虛擬機器已可進行容錯移轉。

## <a name="next-steps"></a>後續步驟
[執行災害復原演練](tutorial-dr-drill-azure.md)
