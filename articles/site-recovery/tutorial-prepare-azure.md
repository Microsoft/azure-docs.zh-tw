---
title: 建立要搭配 Azure Site Recovery 使用的資源 | Microsoft Docs
description: 了解如何準備 Azure，以使用 Azure Site Recovery 來進行內部部署電腦的複寫。
services: site-recovery
author: rayne-wiselman
ms.service: site-recovery
ms.topic: tutorial
ms.date: 10/10/2018
ms.author: raynew
ms.custom: MVC
ms.openlocfilehash: 4a92bfc9d682346c874ce91fc9835558402bae84
ms.sourcegitcommit: 4b1083fa9c78cd03633f11abb7a69fdbc740afd1
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/10/2018
ms.locfileid: "49078982"
---
# <a name="prepare-azure-resources-for-replication-of-on-premises-machines"></a>準備 Azure 資源以進行內部部署機器的複寫

 [Azure Site Recovery](site-recovery-overview.md) 藉由確保您的商務應用程式可在計劃性與非計劃性中斷期間持續啟動並執行，來提供商務持續性和災害復原 (BCDR) 策略。 Site Recovery 會管理並協調內部部署機器和 Azure 虛擬機器 (VM) 的災害復原，包括複寫、容錯移轉和復原。

本文是一系列中的第一個教學課程，說明如何為內部部署 VM 設定災害復原。 這與您是否要保護內部部署 VMware VM、Hyper-V VM 或實體伺服器有關。

這些教學課程皆設計成顯示案例的最簡單部署路徑。 可能的話，會使用預設選項，而不會顯示所有可能的設定與路徑。 

本文說明如何在您想要將內部部署 VM (Hyper-V 或 VMware) 或 Windows/Linux 實體伺服器複寫至 Azure 時，準備好 Azure 元件。 在本教學課程中，您了解如何：

> [!div class="checklist"]
> * 確認您的 Azure 帳戶具有複寫權限。
> * 建立 Azure 儲存體帳戶。 複寫機器的映像會儲存在其中。
> * 建立復原服務保存庫。 保存庫會保存 VM 的中繼資料和組態資訊，以及其他複寫元件。
> * 設定 Azure 網路。 當 Azure VM 在容錯移轉後建立時，會加入此 Azure 網路。

如果您沒有 Azure 訂用帳戶，請在開始前建立 [免費帳戶](https://azure.microsoft.com/pricing/free-trial/) 。

## <a name="sign-in-to-azure"></a>登入 Azure

登入 [Azure 入口網站](http://portal.azure.com)。

## <a name="verify-account-permissions"></a>確認帳戶權限

如果您剛建立免費的 Azure 帳戶，您就是您的訂用帳戶管理員。 如果您不是訂用帳戶管理員，請與系統管理員合作以指派您所需的權限。 若要啟用新虛擬機器的複寫，您必須具有權限才能：

- 在所選的資源群組中建立 VM。
- 在所選的虛擬網路中建立 VM。
- 寫入所選取的儲存體帳戶。

若要完成這些工作，應將虛擬機器參與者內建角色指派至您的帳戶。 此外，若要管理保存庫中的 Site Recovery 作業，應將 Site Recovery 參與者內建角色指派至您的帳戶。

## <a name="create-a-storage-account"></a>建立儲存體帳戶

複寫機器的映像會保留在 Azure 儲存體中。 當您從內部部署容錯移轉至 Azure 時，會從儲存體建立 Azure VM。 儲存體帳戶與復原服務保存庫必須位於相同的區域。 在本教學課程中，我們使用「西歐」。

1. 在 [Azure 入口網站](https://portal.azure.com)功能表中，選取 [建立資源] > [儲存體] > [儲存體帳戶 - Blob、檔案、資料表、佇列]。
2. 在 [建立儲存體帳戶] 上，輸入帳戶的名稱。 在這些教學課程中，我們會使用 **contosovmsacct1910171607**。 您所選取的名稱必須是 Azure 中的唯一名稱，並介於 3 到 24 個字元之間，而且只能包含數字和小寫字母。
3. 在 [部署模型] 中選取 [Resource Manager]。
4. 在 [帳戶種類] 中選取 [儲存體 (一般用途 v1)]。 請勿選取 Blob 儲存體。
5. 在 [複寫] 中，針對儲存體備援選取預設的 [讀取權限異地備援儲存體]。 我們將 [需要安全傳輸] 保留為 [停用]。
6. 在 [效能] 中，選取 [標準]，並在 [存取層] 中選擇 [經常性] 的預設選項。
7. 在 [訂用帳戶] 中，選取您要在其中建立新儲存體帳戶的訂用帳戶。
8. 在 [資源群組] 中，輸入新的資源群組。 Azure 資源群組是在其中部署與管理 Azure 資源的邏輯容器。 在這些教學課程中，我們會使用 **ContosoRG**。
9. 在 [位置] 中，選取儲存體帳戶的地理位置。 

   ![建立儲存體帳戶](media/tutorial-prepare-azure/create-storageacct.png)

9. 選取 [建立]  以建立儲存體帳戶。

## <a name="create-a-recovery-services-vault"></a>建立復原服務保存庫

1. 在 Azure 入口網站中，選取 [建立群組] > [儲存體] > [備份和 Site Recovery (OMS)]。
2. 在 [ **名稱**] 中，輸入保存庫的易記識別名稱。 在這一組教學課程中，我們會使用 **ContosoVMVault**。
3. 在 [資源群組] 中，我們使用 **contosoRG**。
4. 在 [位置] 中。 我們使用**西歐**。
5. 若要從儀表板快速存取保存庫，請按一下 [釘選到儀表板] > [建立]。

   ![建立新保存庫](./media/tutorial-prepare-azure/new-vault-settings.png)

   新的保存庫會出現在 [儀表板] > [所有資源] 上，以及 [復原服務保存庫] 主頁面上。

## <a name="set-up-an-azure-network"></a>設定 Azure 網路

當 Azure VM 在容錯移轉後從儲存體建立時，會加入此網路。

1. 在 [Azure 入口網站](https://portal.azure.com)中，選取 [建立資源] > [網路] > [虛擬網路]。
2. 我們保留 [資源管理員] 作為部署模型。
3. 在 [名稱] 中，輸入網路名稱。 此名稱必須是 Azure 資源群組中的唯一名稱。 在本教學課程中，我們使用 **ContosoASRnet**。
4. 指定要在其中建立網路的資源群組。 我們使用現有的資源群組 **contosoRG**。
5. 在 [位址範圍] 中，輸入網路範圍 **10.0.0.0/24**。 此網路中，我們不會使用子網路。
6. 在 [訂用帳戶] 中，選取要在其中建立網路的訂用帳戶。
7. 在 [位置] 中，選取 [西歐]。 此網路必須位於與復原服務保存庫相同的區域中。
8. 我們保留基本 DDoS 保護的預設選項，且網路上沒有任何服務端點。
9. 按一下頁面底部的 [新增] 。

   ![建立虛擬網路](media/tutorial-prepare-azure/create-network.png)

   虛擬網路的建立會耗用幾秒鐘時間。 建立後，您會在 Azure 入口網站儀表板中看到它。

## <a name="useful-links"></a>實用的連結

- [了解](https://docs.microsoft.com/azure/virtual-network/virtual-networks-overview) Azure 網路。
- [了解](https://docs.microsoft.com/azure/storage/common/storage-introduction#types-of-storage-accounts) Azure 儲存體的類型。
- [進一步了解](https://docs.microsoft.com/azure/storage/common/storage-redundancy-grs#read-access-geo-redundant-storage)儲存體的儲存體備援和[安全傳輸](https://docs.microsoft.com/azure/storage/common/storage-require-secure-transfer)。



## <a name="next-steps"></a>後續步驟

> [!div class="nextstepaction"]
> [準備內部部署 VMware 基礎結構以進行 Azure 的災害復原](tutorial-prepare-on-premises-vmware.md)
