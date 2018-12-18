---
title: 使用 Azure Site Recovery 將 VM 從 AWS 移轉至 Azure | Microsoft Docs
description: 本文說明如何使用 Azure Site Recovery，將 Amazon Web Services (AWS) 中執行的 Windows VM 移轉至 Azure。
services: site-recovery
author: rayne-wiselman
manager: carmonm
ms.service: site-recovery
ms.topic: tutorial
ms.date: 10/10/2018
ms.author: raynew
ms.custom: MVC
ms.openlocfilehash: 04e7506562d29e37abb65b7a760fbc9bd707e2c6
ms.sourcegitcommit: 4b1083fa9c78cd03633f11abb7a69fdbc740afd1
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/10/2018
ms.locfileid: "49078931"
---
# <a name="migrate-amazon-web-services-aws-vms-to-azure"></a>將 Amazon Web Services (AWS) VM 移轉至 Azure

本教學課程將教導您如何使用 Azure Site Recovery，將 Amazon Web Services (AWS) 虛擬機器 (VM) 移轉至 Azure VM。 將 AWS EC2 執行個體移轉至 Azure 時，VM 會被視為實體的內部部署電腦。 在本教學課程中，您了解如何：

> [!div class="checklist"]
> * 驗證必要條件
> * 準備 Azure 資源
> * 準備要移轉的 AWS EC2 執行個體
> * 部署設定伺服器
> * 啟用 VM 複寫
> * 測試容錯移轉以確定一切都沒問題
> * 執行一次性容錯移轉至 Azure

如果您沒有 Azure 訂用帳戶，請在開始前建立 [免費帳戶](https://azure.microsoft.com/pricing/free-trial/) 。

## <a name="prerequisites"></a>必要條件
- 確定您想要移轉的 VM 正執行支援的 OS 版本。 支援的版本包括： 
  - Windows Server 2016 
  - Windows Server 2012 R2
  - Windows Server 2012 
  - 64 位元版本的 Windows Server 2008 R2 SP1 或更新版本
  - Red Hat Enterprise Linux 6.7 (僅限 HVM 虛擬化執行個體)，且具有 Citrix PV 或 AWS PV 驅動程式。 *不*支援執行 RedHat PV 驅動程式的執行個體。
 - 行動服務必須安裝在您要複寫的每個 VM 上。 

    > [!IMPORTANT]
    > 當您啟用 VM 的複寫功能時，Site Recovery 會自動安裝此服務。 若要自動安裝，您必須在 EC2 執行個體上準備一個可供 Site Recovery 用來存取 VM 的帳戶。 您可以使用網域帳戶或本機帳戶。 
    > - 若是 Linux VM，此帳戶應該是來源 Linux 伺服器上的根使用者。 
    > - 若是 Windows VM，如果您不使用網域帳戶，請停用本機電腦上的遠端使用者存取控制：
    >
    >      在登錄的 **HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System** 下，新增 DWORD 項目 **LocalAccountTokenFilterPolicy** 並將其值設為 **1**。

- 另一個可作為 Site Recovery 設定伺服器的 EC2 執行個體。 此執行個體必須執行 Windows Server 2012 R2。

## <a name="prepare-azure-resources"></a>準備 Azure 資源

Azure 中必須有幾個資源可供移轉的 EC2 執行個體使用。 其中包括儲存體帳戶、保存庫及虛擬網路。

### <a name="create-a-storage-account"></a>建立儲存體帳戶

複寫機器的映像會保留在 Azure 儲存體中。 當您從內部部署容錯移轉至 Azure 時，會從儲存體建立 Azure VM。

1. 在 [Azure 入口網站](https://portal.azure.com)的左側功能表中，選取 [建立資源] > [儲存體] > [儲存體帳戶]。
2. 輸入儲存體帳戶的名稱。 在這些教學課程中，我們使用名稱 **awsmigrated2017**。 名稱必須：
    - 在 Azure 中是唯一的
    - 介於 3 到 24 個字元之間
    - 只包含數字和小寫字母
3. 保留 [部署模型]、[帳戶種類]、[效能] 和 [需要安全傳輸] 的預設值。
5. 針對 [複寫]選取預設值 [RA-GRS]。
6. 選取您要用於本教學課程的訂用帳戶。
7. 在 [資源群組] 中，選取 [新建]。 在此範例中，我們使用 **migrationRG** 作為資源群組名稱。
8. 在 [位置] 中，選取 [西歐]。
9. 選取 [建立]  以建立儲存體帳戶。

### <a name="create-a-vault"></a>建立保存庫

1. 在 [Azure 入口網站](https://portal.azure.com)中，選取 [所有服務]。 搜尋而後選取 [復原服務保存庫]。
2. 在 Microsoft Azure 復原服務的保存庫頁面上，選取 [新增]。
3. 針對 [名稱]，輸入 **myVault**。
4. 針對 [訂用帳戶]，選取您要使用的訂用帳戶。
4. 針對 [資源群組]，選取 [使用現有的]，然後選取 [migrationRG]。
5. 在 [位置] 中，選取 [西歐]。
5. 選取 [釘選到儀表板]，即可從儀表板快速存取新的保存庫。
7. 當您完成時，選取 [建立]。

若要查看新的保存庫，請移至 [儀表板] > [所有資源]。 新的保存庫也會出現在 [復原服務保存庫] 主頁面上。

### <a name="set-up-an-azure-network"></a>設定 Azure 網路

移轉 (容錯移轉) 之後建立的 Azure VM 會加入此 Azure 網路。

1. 在 [Azure 入口網站](https://portal.azure.com)中，選取 **[建立資源]**>**[網路]** >
    **[虛擬網路]**。
3. 針對 [名稱]，輸入 **myMigrationNetwork**。
4. 保留 [位址空間] 的預設值。
5. 針對 [訂用帳戶]，選取您要使用的訂用帳戶。
6. 針對 [資源群組]，選取 [使用現有的]，然後選取 [migrationRG]。
7. 在 [位置] 中，選取 [西歐]。
8. 在 [子網路] 之下，保留 [名稱] 和 [IP 範圍] 的預設值。
9. 讓 [服務端點] 選項保持停用。
10. 當您完成時，選取 [建立]。

## <a name="prepare-the-infrastructure"></a>準備基礎結構

在 Azure 入口網站的保存庫頁面上，於 [使用者入門] 區段中選取 [Site Recovery]，然後選取 [準備基礎結構]。 完成下列步驟。

### <a name="1-protection-goal"></a>1：保護目標

在 [保護目標] 頁面上，選取下列值：

|    |  |
|---------|-----------|
| 您的電腦位於何處? |選取 [內部部署]。|
| 您要將電腦複寫到何處? |選取 [至 Azure]。|
| 您的電腦虛擬化了嗎? |選取 [未虛擬化 / 其他]。|

當您完成時，選取 [確定] 以移至下一個區段。

### <a name="2-prepare-source"></a>2：準備來源

在 [準備來源] 頁面上，選取 [+ 組態伺服器]。

1. 使用執行 Windows Server 2012 R2 的 EC2 執行個體建立設定伺服器，並註冊到復原保存庫。
2. 在作為設定伺服器的 EC2 執行個體 VM 上設定 Proxy，使其能夠存取[服務 URL](site-recovery-support-matrix-to-azure.md)。
3. 下載 [Microsoft Azure Site Recovery 整合安裝](http://aka.ms/unifiedinstaller_wus)。 您可以先將它下載到本機電腦，再複製到作為組態伺服器使用的 VM。
4. 選取 [下載] 按鈕，以下載保存庫註冊金鑰。 將已下載的檔案複製到作為組態伺服器使用的 VM。
5. 在 VM 上，以滑鼠右鍵按一下您下載的 Microsoft Azure Site Recovery 整合安裝的安裝程式，然後選取 [以系統管理員身分執行]。

    1. 在 [開始之前] 下，選取 [安裝設定伺服器和處理序伺服器]，然後選取 [下一步]。
    2. 在 [第三方軟體授權] 中，選取 [我接受第三方授權合約]，然後選取 [下一步]。
    3. 在 [註冊] 中，選取 [瀏覽]，然後移至您放置保存庫註冊金鑰檔的位置。 選取 [下一步] 。
    4. 在 [網際網路設定] 中，選取 [不使用 Proxy 伺服器而連線到 Azure Site Recovery]，然後選取 [下一步]。
    5. [檢查必要條件] 頁面會檢查數個項目。 完成時，選取 [下一步]。
    6. 在 [MySQL 組態] 中，提供必要的密碼，然後選取 [下一步]。
    7. 在 [環境詳細資料] 中，選取 [否]。 您不需要保護 VMware 機器。 然後，選取 [下一步]。
    8. 在 [安裝位置] 中，選取 [下一步] 接受預設值。
    9. 在 [選取網路] 中，選取 [下一步] 接受預設值。
    10. 在 [摘要] 中，選取 [安裝]。
    11. [安裝進度] 會顯示安裝程序相關資訊。 完成時，選取 [完成]。 視窗會顯示重新開機的相關訊息。 選取 [確定] 。 接下來，視窗會顯示有關組態伺服器連線複雜密碼的訊息。 將複雜密碼複製到剪貼簿，並將它儲存在安全的位置。
6. 在 VM 上，執行 cspsconfigtool.exe，以在設定伺服器上建立一或多個管理帳戶。 在您想要移轉的 EC2 執行個體上，請確定管理帳戶具有系統管理員權限。

組態伺服器設定完成時，請回到入口網站，並選取您剛才針對 [組態伺服器] 建立的伺服器。 選取 [確定]，前往「3：準備目標」。

### <a name="3-prepare-target"></a>3：準備目標

在本節中，針對您稍早在本教學課程的[準備 Azure 資源](#prepare-azure-resources)中所建立的資源，輸入相關資訊。

1. 在 [訂用帳戶] 中，選取您原先用於[準備 Azure](tutorial-prepare-azure.md) 教學課程的 Azure 訂用帳戶。
2. 選取 [Resource Manager]  做為部署模型。
3. Site Recovery 會確認您是否有一或多個相容的 Azure 儲存體帳戶和網路。 這些應該是您稍早在本教學課程的[準備 Azure 資源](#prepare-azure-resources)中所建立的資源。
4. 完成後，選取 [確定]。


### <a name="4-prepare-replication-settings"></a>4：準備複寫設定

您必須先建立複寫原則，才能啟用複寫。

1. 選取 [複寫並產生關聯]。
2. 在 [名稱] 中，輸入 **myReplicationPolicy**。
3. 保留其餘的預設設定，然後選取 [確定] 以建立原則。 新原則會自動與設定伺服器產生關聯。

### <a name="5-select-deployment-planning"></a>5：選取部署規劃

在 [完成部署規劃了嗎] 中，選取 [稍後再進行]，然後選取 [確定]。

當您完成 [準備基礎結構] 下的所有五個區段時，選取 [確定]。


## <a name="enable-replication"></a>啟用複寫

針對您想要移轉的每部 VM 啟用複寫。 啟用複寫時，Site Recovery 會自動安裝行動服務。

1. 移至 [Azure 入口網站](htts://portal.azure.com)。
1. 在保存庫頁面的 [使用者入門] 下，選取 [Site Recovery]。
2. 在 [內部部署電腦與 Azure VM] 下，選取 [步驟 1: 複寫應用程式]。 以下列資訊完成精靈頁面。 完成時在每個頁面上選取 [確定]：
    - 1：設定來源

    |  |  |
    |-----|-----|
    | 來源： | 選取 [內部部署]。|
    | 來源位置：| 輸入組態伺服器 EC2 執行個體的名稱。|
    |機器類型： | 選取 [實體機器]。|
    | 處理序伺服器： | 從下拉式清單中選取設定伺服器。|

    - 2：設定目標

    |  |  |
    |-----|-----|
    | 目標： | 保留預設值。|
    | 訂用帳戶： | 選取您已使用的訂用帳戶。|
    | 容錯移轉後的資源群組：| 使用您在[準備 Azure 資源](#prepare-azure-resources)中所建立的資源群組。|
    | 容錯移轉後的部署模型： | 選擇 **資源管理員**。|
    | 儲存體帳戶： | 選取您在[準備 Azure 資源](#prepare-azure-resources)中所建立的儲存體帳戶。|
    | Azure 網路： | 選取 [立即設定選取的機器]。|
    | 容錯移轉後的 Azure 網路： | 選擇您在[準備 Azure 資源](#prepare-azure-resources)中所建立的網路。|
    | 子網路： | 在下拉式清單中選取 [預設值]。|

    - 3：選取實體機器

      選取 [實體機器]，然後針對您想要移轉的 EC2 執行個體，輸入 [名稱]、[IP 位址] 和 [OS 類型] 的值。 選取 [確定] 。

    - 4：設定屬性

      選取您在組態伺服器上建立的帳戶，然後選取 [確定]。

    - 5：設定複寫設定

      請確定在下拉式清單中選取的複寫原則是 **myReplicationPolicy**，然後選取 [確定]。

3. 當精靈完成時，選取 [啟用複寫]。

若要追蹤 [啟用保護] 作業的進度，請移至 [監視與報告] > [作業] > [Site Recovery 作業] 中。 執行 [完成保護] 作業之後，機器即準備好進行容錯移轉。        

啟用 VM 複寫時，可能需要 15 分鐘或更久的時間，變更才會生效並顯示在入口網站中。

## <a name="run-a-test-failover"></a>執行測試容錯移轉

當您執行測試容錯移轉時，會發生下列事件：

- 系統會執行必要條件檢查，確保容錯移轉所需的所有條件都已備妥。
- 容錯移轉會處理資料，以便建立 Azure VM。 如果您選取最新的復原點，則會根據資料建立復原點。
- 使用先前步驟中處理的資料來建立 Azure VM。

在入口網站中，執行測試容錯移轉：

1. 在保存庫的頁面上，移至 [受保護的項目] > [複寫的項目]。 選取 VM，然後選取 [測試容錯移轉]。
2. 選取要用於容錯移轉的復原點：
    - **最近處理**：將 VM 容錯移轉到 Site Recovery 所處理的最新復原點。 隨即顯示時間戳記。 使用此選項時，無須花費時間處理資料，因此它會提供低復原時間目標 (RTO)。
    - **最近的應用程式一致**：此選項會將所有 VM 容錯移轉到最新的應用程式一致復原點。 隨即顯示時間戳記。
    - **自訂**：選取任何復原點。

3. 在 [測試容錯移轉] 中，選取 Azure VM 在容錯移轉之後要連線的目標 Azure 網路。 這應該是您在[準備 Azure 資源](#prepare-azure-resources)中所建立的網路。
4. 選取 [確定]  即可開始容錯移轉。 若要追蹤進度，請選取 VM 來檢視其屬性。 或者，可以選取保存庫頁面上的 [測試容錯移轉] 作業。 若要這樣做，請選取 [監視與報告] > [作業] >  [Site Recovery 作業]。
5. 容錯移轉完成時，Azure VM 複本會出現在 Azure 入口網站中。 若要檢視 VM，請選取 [虛擬機器]。 確保 VM 為適當的大小、其已連線到正確的網路，而且正在執行中。
6. 您現在應該能夠連線到 Azure 中複寫的 VM。
7. 若要刪除測試容錯移轉期間建立的 Azure VM，請選取復原計劃中的 [清除測試容錯移轉]。 在 [記事] 中，記錄並儲存關於測試容錯移轉的任何觀察。

在某些情況下，容錯移轉需要額外的處理。 處理需要 8 到 10 分鐘才能完成。

## <a name="migrate-to-azure"></a>移轉至 Azure

對 EC2 執行個體執行實際的容錯移轉，以移轉至 Azure VM：

1. 在 [受保護的項目] > [複寫的項目] 中，選取 AWS 執行個體，然後選取 [容錯移轉]。
2. 在 [容錯移轉] 中，選取容錯移轉的目標 [復原點]。 選取最新的復原點，然後開始容錯移轉。 您可以 [作業] 頁面上追蹤容錯移轉進度。
1. 確保 VM 出現在 [複寫的項目] 中。
2. 以滑鼠右鍵按一下每個 VM，然後選取 [完成移轉]。 這會完成移轉程序、停止 AWS VM 的複寫，並停止 VM 的 Site Recovery 計費。

    ![完成移轉](./media/migrate-tutorial-aws-azure/complete-migration.png)

> [!WARNING]
> 不要取消正在進行的容錯移轉。 在啟動容錯移轉之前，已停止 VM 複寫。 如果您取消正在進行的容錯移轉，容錯移轉會停止，但 VM 不會再次複寫。  


## <a name="next-steps"></a>後續步驟

在本文中，您已了解如何將 AWS EC2 執行個體移轉至 Azure VM。 若要深入了解 Azure VM，請繼續 Windows VM 的教學課程。

> [!div class="nextstepaction"]
> [Azure Windows 虛擬機器教學課程](../virtual-machines/windows/tutorial-manage-vm.md)
