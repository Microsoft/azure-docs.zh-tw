---
title: 使用 Azure 備份伺服器備份工作負載
description: 在本文中，了解如何準備環境，以使用 Microsoft Azure 備份伺服器 (MABS) 來保護及備份工作負載。
ms.topic: conceptual
ms.date: 11/13/2018
ms.openlocfilehash: d476c228a619f03f798c1a2cd6854a8d603c3637
ms.sourcegitcommit: 04297f0706b200af15d6d97bc6fc47788785950f
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 01/28/2021
ms.locfileid: "98987017"
---
# <a name="install-and-upgrade-azure-backup-server"></a>安裝及升級 Azure 備份伺服器

> [!div class="op_single_selector"]
>
> * [Azure 備份伺服器](backup-azure-microsoft-azure-backup.md)
> * [SCDPM](backup-azure-dpm-introduction.md)
>
>

> 適用於：MABS v3。 (不再支援 MABS v2。 如果您使用的版本早于 MABS v3，請升級至最新版本。 ) 

本文說明如何準備環境，以使用 Microsoft Azure 備份伺服器 (MABS) 來備份工作負載。 透過 Azure 備份伺服器，您將可從單一主控台保護 Hyper-V VM、Microsoft SQL Server、SharePoint Server、Microsoft Exchange 和 Windows 用戶端等應用程式工作負載。

> [!NOTE]
> Azure 備份伺服器現在可以保護 VMware VM，並提供改善的安全性功能。 依下列各節所述安裝此產品，並安裝最新的 Azure 備份代理程式。 若要深入了解使用 Azure 備份伺服器備份 VMware 伺服器，請參閱文件：[使用 Azure 備份伺服器備份 VMware 伺服器](backup-azure-backup-server-vmware.md)。 若要深入了解安全性功能，請參閱 [Azure 備份安全性功能文件](backup-azure-security-feature.md)。
>
>

Azure VM 中部署的 MABS 可以在 Azure 中備份 VM，但是應在相同的網域中才能啟用備份作業。 備份 Azure VM 與備份內部部署 VM 的程序相同，不過在 Azure 中部署 MABS 有一些限制。 如需限制詳細資訊，請參閱 [DPM 作為 Azure 虛擬機器](/system-center/dpm/install-dpm#setup-prerequisites)

> [!NOTE]
> Azure 有兩種用來建立和使用資源的部署模型：[Resource Manager 和傳統](../azure-resource-manager/management/deployment-models.md)。 本文提供的資訊和程序可供還原使用 Resource Manager 模型部署的 VM。
>
>

Azure 備份伺服器承襲了 Data Protection Manager (DPM) 的大部分工作負載備份功能。 本文會連結至 DPM 文件，以說明其中的部分共用功能。 雖然 Azure 備份伺服器與 DPM 共用許多相同的功能，但 Azure 備份伺服器不會備份到磁帶，也不會與 System Center 整合。

## <a name="choose-an-installation-platform"></a>選擇安裝平台

要啟動並執行 Azure 備份伺服器，第一個步驟是設定 Windows Server。 伺服器可以位於 Azure 或內部部署中。

* 若要保護內部部署工作負載，MABS 伺服器必須位於內部部署環境。
* 若要保護 Azure VM 上執行的工作負載，MABS 伺服器必須位於 Azure 中，以 Azure VM 的形式執行。

### <a name="using-a-server-in-azure"></a>使用 Azure 中的伺服器

選擇伺服器執行 Azure 備份伺服器時，建議您從 Windows Server 2016 Datacenter 或 Windows Server 2019 Datacenter 的資源庫映射開始。 即使您之前從未使用過 Azure， [在 Azure 入口網站中建立第一個 Windows 虛擬機器](../virtual-machines/windows/quick-create-portal.md?toc=/azure/virtual-machines/windows/toc.json)一文會提供教學課程讓您在 Azure 中開始使用建議的虛擬機器。 伺服器虛擬機器 (VM) 的最低建議需求應該是︰搭配四核心和 8 GB RAM 的 Standard_A4_v2。

使用 Azure 備份伺服器保護工作負載有許多細節需要注意。 [MABS 的保護矩陣](./backup-mabs-protection-matrix.md)有助於說明這些細節。 在部署機器之前，請先確實閱讀此文章。

### <a name="using-an-on-premises-server"></a>使用內部部署伺服器

如果您不想要在 Azure 中執行基底伺服器，您可以在 Hyper-v VM、VMware VM 或實體主機上執行伺服器。 伺服器硬體的最低建議需求是雙核心與 8 GB 的 RAM。 下表列出支援的作業系統：

| 作業系統 | 平台 | SKU |
|:--- | --- |:--- |
| Windows Server 2019 |64 位元 |Standard、Datacenter、Essentials |
| Windows Server 2016 和最新的 SP |64 位元 |Standard、Datacenter、Essentials  |

您可以使用 Windows Server Deduplication 為 DPM 儲存體刪除重複資料。 深入了解在 Hyper-V VM 中部署時， [DPM 和重複資料刪除](/system-center/dpm/deduplicate-dpm-storage) 如何搭配運作。

> [!NOTE]
> Azure 備份伺服器的設計目的是在專用、單一用途的伺服器上執行。 您無法在下列上安裝 Azure 備份伺服器：
>
> * 執行為網域控制站的電腦
> * 安裝應用程式伺服器角色所在的電腦
> * System Center Operations Manager 管理伺服器的電腦
> * Exchange Server 執行所在的電腦
> * 屬於叢集節點的電腦
>
> Windows Server Core 或 Microsoft Hyper-V Server 不支援安裝 Azure 備份伺服器。

Azure 備份伺服器一律加入網域。 如果您打算將伺服器移到不同的網域，請先安裝 Azure 備份伺服器，然後將伺服器加入新網域。 若在部署後將現有的 Azure 備份伺服器機器移至新網域，該動作將「不受支援」 。

無論您是將備份資料傳送至 Azure，或保存在本機，都必須向復原服務保存庫註冊 Azure 備份伺服器。

[!INCLUDE [backup-create-rs-vault.md](../../includes/backup-create-rs-vault.md)]

### <a name="set-storage-replication"></a>設定儲存體複寫

儲存體複寫選項有異地備援儲存體和本地備援儲存體可供您選擇。 根據預設，復原服務保存庫會使用異地備援儲存體。 如果這個保存庫是您的主要保存庫，儲存體選項請保持設定為異地備援儲存體。 如果您想要更便宜但不持久的選項，請選擇本地備援儲存體。 在[Azure 儲存體複寫總覽](../storage/common/storage-redundancy.md)中，深入瞭解[地理區域冗余](../storage/common/storage-redundancy.md#geo-redundant-storage)、[本機冗余](../storage/common/storage-redundancy.md#locally-redundant-storage)和[區域冗余](../storage/common/storage-redundancy.md#zone-redundant-storage)儲存體選項。

若要編輯儲存體複寫設定︰

1. 從 [復原 **服務保存庫** ] 窗格中，選取新的保存庫。 在 [ **設定** ] 區段下，選取 [  **屬性**]。
2. 在 [ **屬性**] 中的 [ **備份** 設定] 下，選取 [ **更新**]。

3. 選取儲存體複寫類型，然後選取 [**儲存]。**

     ![為新保存庫設定儲存體組態](./media/backup-create-rs-vault/recovery-services-vault-backup-configuration.png)

## <a name="software-package"></a>軟體封裝

### <a name="downloading-the-software-package"></a>下載軟體封裝

1. 登入 [Azure 入口網站](https://portal.azure.com/)。
2. 如果您已經開啟復原服務保存庫，請繼續進行步驟3。 如果您沒有開啟復原服務保存庫，但位於 Azure 入口網站的主功能表上，請選取 **[流覽]**。

   * 在資源清單中輸入 **復原服務**。
   * 當您開始輸入時，清單將會根據您輸入的文字進行篩選。 當您看到 [復原服務保存庫] 時，請選取它。

     ![建立復原服務保存庫的步驟1](./media/backup-azure-microsoft-azure-backup/open-recovery-services-vault.png)

     隨即會出現 [復原服務保存庫] 清單。
   * 在 [復原服務保存庫] 清單中選取保存庫。

     選取的保存庫儀表板隨即開啟。

     ![保存庫儀表板](./media/backup-azure-microsoft-azure-backup/vault-dashboard.png)
3. 預設會開啟 [ **設定** ] 窗格。 如果已關閉，請選取 [ **設定** ] 以開啟 [設定] 窗格。

    ![設定窗格](./media/backup-azure-microsoft-azure-backup/vault-setting.png)
4. 選取 [ **備份** ] 以開啟開始使用 wizard。

    ![備份開始使用](./media/backup-azure-microsoft-azure-backup/getting-started-backup.png)

    在開啟 [備份] 窗格的 [ **開始使用** ] 中，將會自動選取 **備份目標** 。

    ![Backup-goals-default-opened](./media/backup-azure-microsoft-azure-backup/getting-started.png)

5. 在 [ **備份目標** ] 窗格中，從 [ **工作負載** 的執行位置] 功能表上選取 [內部 **部署**]。

    ![內部部署和做為目標的工作負載](./media/backup-azure-microsoft-azure-backup/backup-goals-azure-backup-server.png)

    從 [ **您要備份什麼？** ] 下拉式功能表中，選取您要使用 Azure 備份伺服器保護的工作負載，然後選取 **[確定]**。

    [開始使用備份功能] 精靈會切換 [準備基礎結構] 選項以將工作負載備份至 Azure。

   > [!NOTE]
   > 如果您只想要備份檔案和資料夾，我們建議使用 Azure 備份代理程式並遵循[初步了解：備份檔案和資料夾](./backup-windows-with-mars-agent.md)一文中的指引。 如果您要保護的不只是檔案和資料夾，或您打算在未來擴充保護需求，請選取這些工作負載。
   >
   >

    ![開始使用精靈變更](./media/backup-azure-microsoft-azure-backup/getting-started-prep-infra.png)

6. 在開啟的 [ **準備基礎結構** ] 窗格中，選取 [安裝 Azure 備份伺服器的 **下載** 連結，並下載保存庫認證。 您會在向復原服務保存庫 Azure 備份伺服器註冊期間使用保存庫認證。 連結會使您進入可下載軟體套件的 [下載中心]。

    ![準備用於 Azure 備份伺服器的基礎結構](./media/backup-azure-microsoft-azure-backup/azure-backup-server-prep-infra.png)

7. 選取所有檔案，然後選取 **[下一步]**。 下載「Microsoft Azure 備份」下載頁面中的所有檔案，並將所有檔案放在相同的資料夾中。

    ![下載中心 1](./media/backup-azure-microsoft-azure-backup/downloadcenter.png)

    因為所有檔案的下載大小都是 > 3 GB，所以在 10 Mbps 的下載連結上，下載完成最多可能需要60分鐘的時間。

### <a name="extracting-the-software-package"></a>將軟體封裝解壓縮

下載所有檔案之後，請選取 [ **MicrosoftAzureBackupInstaller.exe**]。 這會啟動 [Microsoft Azure 備份安裝精靈]  ，並將安裝程式檔案解壓縮至您所指定的位置。 繼續執行嚮導，然後選取 [ **解壓縮** ] 按鈕以開始解壓縮程式。

> [!WARNING]
> 至少 4 GB 的可用空間，才能將安裝程式檔案解壓縮。
>
>

![設定解壓縮檔案以供安裝](./media/backup-azure-microsoft-azure-backup/extract/03.png)

解壓縮程式完成之後，請核取此方塊以啟動全新解壓縮的 *setup.exe* ，開始安裝 Microsoft Azure 備份 Server，然後選取 [ **完成]** 按鈕。

### <a name="installing-the-software-package"></a>安裝軟體封裝

1. 選取 [ **Microsoft Azure 備份** ] 以啟動安裝程式。

    ![Microsoft Azure 備份安裝精靈](./media/backup-azure-microsoft-azure-backup/launch-screen2.png)
2. 在 [歡迎畫面上，選取 [ **下一步]** 按鈕。 這會讓您進入 [必要條件檢查]  區段。 在此畫面上選取 [ **檢查** ]，判斷是否已符合 Azure 備份伺服器的硬體和軟體必要條件。 如果成功符合所有必要條件，您會看到一則訊息，指出電腦符合需求。 選取 [下一步] 按鈕。

    ![Azure 備份伺服器 - 歡迎使用和必要條件檢查](./media/backup-azure-microsoft-azure-backup/prereq/prereq-screen2.png)
3. Azure 備份伺服器安裝封裝隨附適當的必要 SQL Server 二進位檔。 開始新的 Azure 備份伺服器安裝時，請選取 [ **使用此安裝程式安裝 SQL Server 的新實例** ] 選項，然後選取 [ **檢查並安裝** ] 按鈕。 成功安裝必要條件之後，請選取 **[下一步]**。

    >[!NOTE]
    >如果您想要使用自己的 SQL Server，支援的 SQL Server 版本為 SQL Server 2014 SP1 或更新版本、2016 和 2017。  所有 SQL Server 版本都應該是 Standard 或 Enterprise 64 位元。
    >Azure 備份伺服器無法與遠端 SQL Server 實例搭配運作。 Azure 備份伺服器所使用的執行個體必須是本機的。 如果您是使用現有的 SQL server 進行 MABS，MABS 安裝程式只支援使用 *已命名* 的 sql server 實例。

    ![Azure 備份伺服器 - SQL 檢查](./media/backup-azure-microsoft-azure-backup/sql/01.png)

    如果因重新開機電腦的建議而發生失敗，請選取 [ **檢查**]。 如果有任何 SQL 設定問題，請根據 SQL 指導方針重新設定 SQL，然後使用現有的 SQL 實例重試安裝/升級 MABS。

   **手動設定**

   當您使用自己的 SQL 執行個體時，請務必將 builtin\Administrators 新增至 Master 資料庫的系統管理員角色。

    **SQL 2017 的 SSRS 設定**

    當您使用自己的 SQL 2017 實例時，您需要手動設定 SSRS。 設定 SSRS 之後，請確保 SSRS 的 IsInitialized 屬性設定為 True。 當這個屬性設定為 True 時，MABS 會假設 SSRS 已設定好，並且會跳過 SSRS 設定。

    將下列值使用於 SSRS 設定：
    * 服務帳戶：「使用內建帳戶」應為網路服務
    * Web 服務 URL：應 ReportServer_ 「虛擬目錄」\<SQLInstanceName>
    * 資料庫： DatabaseName 應為 ReportServer $\<SQLInstanceName>
    * 入口網站 URL：應 Reports_ 「虛擬目錄」\<SQLInstanceName>

    [深入了解](/sql/reporting-services/report-server/configure-and-administer-a-report-server-ssrs-native-mode) SSRS 設定。

    > [!NOTE]
    > 針對用作 MABS 資料庫的 SQL Server 進行授權時，由 [Microsoft 線上服務條款](https://www.microsoft.com/licensing/product-licensing/products) (OST) 所控管。 根據 OST，與 MABS 配套的 SQL Server 只能用作 MABS 的資料庫。

4. 提供安裝 Microsoft Azure 備份 server 檔案的位置，然後選取 **[下一步]**。

    ![提供安裝檔案的位置](./media/backup-azure-microsoft-azure-backup/space-screen.png)

    備份至 Azure 需要暫存位置。 請確保暫存位置至少為打算備份至雲端的資料的 5%。 在磁碟保護方面，安裝完成之後必須設定獨立的磁碟。 如需有關儲存集區的詳細資訊，請參閱 [準備資料存放區](/system-center/dpm/plan-long-and-short-term-data-storage)。

    磁片儲存體的容量需求主要取決於受保護資料的大小、每日復原點大小、預期的磁片區資料成長率及保留範圍目標。 建議您將磁片儲存體的大小調整為受保護資料的兩倍。 這是假設 10% 受保護資料大小的每日復原點大小以及 10 天的保留範圍。 若要取得良好的大小預估，請參閱 [DPM Capacity Planner](https://www.microsoft.com/download/details.aspx?id=54301)。 

5. 為受限的本機使用者帳戶提供強式密碼，然後選取 **[下一步]**。

    ![提供強式密碼](./media/backup-azure-microsoft-azure-backup/security-screen.png)
6. 選取您是否要使用 *Microsoft Update* 檢查更新，然後選取 **[下一步]**。

   > [!NOTE]
   > 建議讓 Windows Update 重新導向至 Microsoft Update，此網站為 Windows 和 Microsoft Azure 備份伺服器等其他產品提供安全性和重要更新。
   >
   >

    ![Microsoft Update 選擇加入](./media/backup-azure-microsoft-azure-backup/update-opt-screen2.png)
7. 檢查 *設定的摘要* ，然後選取 [ **安裝**]。

    ![設定摘要](./media/backup-azure-microsoft-azure-backup/summary-screen.png)
8. 安裝分階段執行。 在第一個階段中，會將 Microsoft Azure 復原服務代理程式安裝在伺服器上。 精靈也會檢查網際網路連線。 如果有網際網路連線能力，您可以繼續進行安裝。 如果沒有，您需要提供 proxy 詳細資料，以連接到網際網路。

    下一個步驟是設定 Microsoft Azure 復原服務代理程式。 在設定過程中，您必須提供保存庫認證，以向復原服務保存庫註冊電腦。 您也會提供複雜密碼來加密/解密在 Azure 與您的內部部署之間傳送的資料。 您可以自動產生複雜密碼，或提供您自己的複雜密碼，最少 16 個字元。 繼續執行精靈，直到代理程式完成設定。

    ![註冊伺服器嚮導](./media/backup-azure-microsoft-azure-backup/mars/04.png)
9. Microsoft Azure 備份伺服器順利完成註冊後，整體安裝精靈會繼續安裝及設定 SQL Server 和 Azure 備份伺服器的元件。 SQL Server 元件安裝完成後，會安裝 Azure 備份伺服器元件。

    ![Azure 備份伺服器設定進度](./media/backup-azure-microsoft-azure-backup/final-install/venus-installation-screen.png)

當安裝步驟完成時，產品的桌面圖示也將一併建立完成。 按兩下圖示以啟動產品。

### <a name="add-backup-storage"></a>新增備份儲存體

第一個備份複本會保存在連接至 Azure 備份伺服器機器的儲存體上。 如需有關新增磁碟的詳細資訊，請參閱 [設定存放集區和磁碟儲存體](./backup-mabs-add-storage.md)。

> [!NOTE]
> 即使您打算將資料傳送至 Azure，也必須新增備份儲存體。 在目前的「Azure 備份伺服器」架構中，「Azure 備份」保存庫會保存資料的「第二個」  複本，而本機儲存體則是保存第一個 (必要的) 備份複本。
>
>

### <a name="install-and-update-the-data-protection-manager-protection-agent"></a>安裝及更新 Data Protection Manager 保護代理程式

MABS 會使用 System Center Data Protection Manager 保護代理程式。 [這裡的步驟](/system-center/dpm/deploy-dpm-protection-agent)可在保護伺服器上安裝保護代理程式。

下列各節說明如何更新用戶端電腦的保護代理程式。

1. 在備份伺服器管理員主控台中選取 [管理] > [代理程式]。

2. 在顯示窗格中，選取您要更新保護代理程式的用戶端電腦。

   > [!NOTE]
   > [代理程式更新] 資料行會在各個受保護的電腦可以進行保護代理程式更新時予以指出。 在 [動作] 窗格中，只有當您已選取受保護的電腦且其可供更新時，才可使用 [更新] 動作。
   >
   >

3. 若要在選取的電腦上安裝已更新的保護代理程式，請在 [動作] 窗格中選取 [更新]。

4. 對於未連線到網路的用戶端電腦，除非該電腦連線到網路，否則 [代理程式狀態] 資料行會顯示 [更新擱置中] 的狀態。

   在用戶端電腦連線到網路之後，用戶端電腦的 [代理程式更新] 資料行會顯示 [更新中] 狀態。

## <a name="move-mabs-to-a-new-server"></a>將 MABS 移至新的伺服器

如果您需要將 MABS 移至新伺服器，同時保留儲存體，請遵循步驟。 只有所有資料都位於新式備份儲存體時，才能這麼做。

  > [!IMPORTANT]
  >
  > * 新的伺服器名稱必須與原始 Azure 備份伺服器實例具有相同的名稱。 如果您想要使用先前的儲存體集區和 MABS 資料庫 (DPMDB) 以保留復原點，就無法變更新 Azure 備份伺服器執行個體的名稱。
  > * 您必須擁有 MABS 資料庫 (DPMDB) 的備份。 您將需要它來還原資料庫。

1. 在顯示窗格中，選取您要更新保護代理程式的用戶端電腦。
2. 關閉原始的 Azure 備份伺服器或使其離線。
3. 在 Active Directory 中重設機器帳戶。
4. 在新電腦上安裝伺服器2016，並為它提供與原始 Azure 備份伺服器相同的電腦名稱稱。
5. 加入網域。
6. 安裝 Azure 備份伺服器 V3 或更新版本 (從舊伺服器移動 MABS 存放集區磁片，然後匯入) 。
7. 還原在步驟 1 取得的 DPMDB。
8. 將原始備份伺服器的儲存體連結到新伺服器。
9. 從 SQL 還原 DPMDB。
10. 在新的伺服器上，以系統管理員) 的身分執行 CMD (。 移至 Microsoft Azure 備份安裝位置和 bin 資料夾

    路徑範例： `C:\windows\system32>cd "c:\Program Files\Microsoft Azure Backup\DPM\DPM\bin\"`

11. 若要連接到 Azure 備份，請執行 `DPMSYNC -SYNC`

    如果 **您已將磁片新增** 到 DPM 存放集區，而不是移動舊的磁片，請執行 `DPMSYNC -Reallocatereplica` 。

## <a name="network-connectivity"></a>網路連線

Azure 備份伺服器需要連線至 Azure 備份服務，產品才能順利運作。 若要驗證機器是否連接至 Azure，請在Azure 備份伺服器 PowerShell 主控台中使用 ```Get-DPMCloudConnection``` Cmdlet。 如果 Cmdlet 的輸出為 TRUE，表示連線存在，否則就不會有連線能力。

同時，Azure 訂用帳戶也必須處於狀況良好狀態。 若要了解訂用帳戶狀態並加以管理，請登入[訂用帳戶入口網站](https://account.windowsazure.com/Subscriptions)。

在您了解 Azure 連線和 Azure 訂用帳戶的狀態後，您可以使用下表來確認提供的備份/還原功能會受到哪些影響。

| 連線狀態 | Azure 訂閱 | 備份至 Azure | 備份到磁碟 | 從 Azure 還原 | 從磁碟還原 |
| --- | --- | --- | --- | --- | --- |
| 連線 |Active |允許 |允許 |允許 |允許 |
| 連線 |已過期 |已停止 |已停止 |允許 |允許 |
| 連線 |已取消佈建 |已停止 |已停止 |已停止且已刪除 Azure 復原點 |已停止 |
| 連線中斷 > 15 天 |Active |已停止 |已停止 |允許 |允許 |
| 連線中斷 > 15 天 |已過期 |已停止 |已停止 |允許 |允許 |
| 連線中斷 > 15 天 |已取消佈建 |已停止 |已停止 |已停止且已刪除 Azure 復原點 |已停止 |

### <a name="recovering-from-loss-of-connectivity"></a>從連線中斷的情況復原

如果您的電腦具有有限的網際網路存取權，請確定電腦或 proxy 上的防火牆設定允許下列 Url 和 IP 位址：

* URL
  * `www.msftncsi.com`
  * `*.Microsoft.com`
  * `*.WindowsAzure.com`
  * `*.microsoftonline.com`
  * `*.windows.net`
  * `www.msftconnecttest.com`
* IP 位址
  * 20.190.128.0/18
  * 40.126.0.0/18

如果您是使用 ExpressRoute Microsoft 對等互連，請選取下列服務/區域：

* Azure Active Directory (12076:5060)
* 根據復原服務保存庫的位置 (Microsoft Azure 區域) 
* 根據您的復原服務保存庫位置 Azure 儲存體 () 

如需詳細資料，請造訪 [ExpressRoute 路由需求](../expressroute/expressroute-routing.md)。

在 Azure 的連線已還原至 Azure 備份伺服器機器之後，可執行的作業將取決於 Azure 訂用帳戶狀態。 上表詳細列出機器「已連接」之後所允許之作業的相關資訊。

### <a name="handling-subscription-states"></a>處理訂用帳戶狀態

您可以將 Azure 訂用帳戶從 *過期* 或 *取消布建* 狀態為「*作用中」狀態。* 不過，當狀態 *為非作用* 中時，這會對產品行為造成一些影響：

* *取消布建* 訂用帳戶在取消布建的期間內失去了功能。 切換為「作用中」 時，就會恢復產品的備份/還原功能。 此外，只要以夠大的保留期間來保存，也可以擷取本機磁碟上的備份資料。 不過，一旦訂用帳戶進入「已取消佈建」  狀態，Azure 中的備份資料便會遺失而無法復原。
* 「已過期」的訂用帳戶只有在恢復「作用中」狀態之前會失去功能。 排程于訂閱 *過期* 期間的任何備份都不會執行。

## <a name="upgrade-mabs"></a>升級 MABS

使用下列程序來升級 MABS。

### <a name="upgrade-from-mabs-v2-to-v3"></a>從 MABS V2 升級至 V3

> [!NOTE]
>
> MABS V2 不是安裝 MABS V3 的先決條件。 不過，您只能從 MABS V2 升級到 MABS V3。

使用下列步驟來升級 MABS：

1. 若要從 MABS V2 升級到 MABS V3，請視需要將作業系統升級到 Windows Server 2016 或 Windows Server 2019。

2. 升級您的伺服器。 步驟類似於[安裝](#install-and-upgrade-azure-backup-server)。 不過，針對 SQL 設定，您可以選擇將 SQL 實例升級至 SQL 2017，或使用您自己的 SQL server 2017 實例。

   > [!NOTE]
   >
   > 當您的 SQL 實例正在升級時，請勿結束。 即將結束將會卸載 SQL reporting 實例，因此嘗試重新升級 MABS 將會失敗。

   > [!IMPORTANT]
   >
   >  在 SQL 2017 升級過程中，我們會備份 SQL 加密金鑰並將報告服務解除安裝。 升級 SQL Server 之後，便會安裝報告服務 (14.0.6827.4788) 並還原加密金鑰。
   >
   > 手動設定 SQL 2017 時，請參閱安裝指示之下的「SQL 2017 的 SSRS 設定」。

3. 在受保護的伺服器上更新保護代理程式。
4. 備份應會繼續，而不需重新啟動您的生產伺服器。
5. 您現在可以開始保護您的資料。 如果您要升級到 Modern Backup Storage，在保護時，您也可以選擇想要儲存備份的磁片區，然後在 [已布建的空間] 底下檢查。 [深入了解](backup-mabs-add-storage.md)。

## <a name="troubleshooting"></a>疑難排解

如果 Microsoft Azure 備份伺服器在安裝階段 (或是備份或還原) 失敗且出現錯誤，請參閱這份[錯誤碼文件](https://support.microsoft.com/kb/3041338)，以取得詳細資訊。
您也可以參考 [Azure 備份相關的常見問題集](backup-azure-backup-faq.md)

## <a name="next-steps"></a>後續步驟

您可以在這裡取得有關 [準備 DPM 環境](/system-center/dpm/prepare-environment-for-dpm)的詳細資訊。 其中也包含可據以部署和使用 Azure 備份伺服器之支援組態的相關資訊。 您可以使用一系列的 [PowerShell Cmdlet](/powershell/module/dataprotectionmanager/) 來執行各種作業。

請參閱這些文章，以深入了解使用 Microsoft Azure 備份伺服器來保護工作負載。

* [SQL Server 備份](backup-azure-backup-sql.md)
* [SharePoint 伺服器備份](backup-azure-backup-sharepoint.md)
* [替代伺服器備份](backup-azure-alternate-dpm-server.md)
