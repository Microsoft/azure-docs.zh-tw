---
title: 使用 Azure 備份伺服器，將工作負載備份至 Azure
description: 使用 Azure 備份伺服器來保護工作負載，或備份至 Azure 入口網站。
services: backup
author: PVRK
manager: shivamg
keywords: Azure 備份伺服器; 保護工作負載; 備份工作負載
ms.service: backup
ms.topic: conceptual
ms.date: 7/10/2018
ms.author: adigan
ms.openlocfilehash: 67243aca9f5f578402ff79422783148af53798c6
ms.sourcegitcommit: 0a84b090d4c2fb57af3876c26a1f97aac12015c5
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/11/2018
ms.locfileid: "38546012"
---
# <a name="install-and-configure-azure-backup-server"></a>安裝和設定 Azure 備份伺服器
> [!div class="op_single_selector"]
> * [Azure 備份伺服器](backup-azure-microsoft-azure-backup.md)
> * [SCDPM](backup-azure-dpm-introduction.md)
>
>

本文說明如何準備環境，以使用 Azure 備份伺服器來備份工作負載。 透過 Azure 備份伺服器，您將可從單一主控台保護 Hyper-V VM、Microsoft SQL Server、SharePoint Server、Microsoft Exchange 和 Windows 用戶端等應用程式工作負載。

> [!NOTE]
> Azure 備份伺服器現在可以保護 VMware VM，並提供改善的安全性功能。 依下列各節所述安裝此產品，並套用 Update 1 和最新的 Azure 備份代理程式。 若要深入了解使用 Azure 備份伺服器備份 VMware 伺服器，請參閱文件：[使用 Azure 備份伺服器備份 VMware 伺服器](backup-azure-backup-server-vmware.md)。 若要深入了解安全性功能，請參閱 [Azure 備份安全性功能文件](backup-azure-security-feature.md)。
>
>

您也可以保護基礎結構即服務 (IaaS) 工作負載，例如 Azure 中的 VM。

> [!NOTE]
> Azure 有兩種用來建立和使用資源的部署模型： [Resource Manager 和傳統](../azure-resource-manager/resource-manager-deployment-model.md)。 本文提供的資訊和程序可供還原使用 Resource Manager 模型部署的 VM。
>
>

Azure 備份伺服器承襲了 Data Protection Manager (DPM) 的大部分工作負載備份功能。 本文會連結至 DPM 文件，以說明其中的部分共用功能。 雖然 Azure 備份伺服器共用許多與 DPM 相同的功能。 Azure 備份伺服器並不會備份到磁帶，也無法與 System Center 整合。

## <a name="choose-an-installation-platform"></a>選擇安裝平台
要啟動並執行 Azure 備份伺服器，第一個步驟是設定 Windows Server。 伺服器可以位於 Azure 或內部部署中。

### <a name="using-a-server-in-azure"></a>使用 Azure 中的伺服器
選擇用來執行 Azure 備份伺服器的伺服器時，建議您從 Windows Server 2012 R2 Datacenter 或 Windows Server 2016 Datacenter 的資源庫映像開始。 即使您之前從未使用過 Azure， [在 Azure 入口網站中建立第一個 Windows 虛擬機器](../virtual-machines/virtual-machines-windows-hero-tutorial.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json)一文會提供教學課程讓您在 Azure 中開始使用建議的虛擬機器。 伺服器虛擬機器 (VM) 的最低建議需求應該是︰A2 標準，具備雙核心及 3.5 GB 的 RAM。

使用 Azure 備份伺服器保護工作負載有許多細節需要注意。 [將 DPM 安裝為 Azure 虛擬機器](https://technet.microsoft.com/library/jj852163.aspx)一文可協助說明這些細節。 在部署機器之前，請先確實閱讀此文章。

### <a name="using-an-on-premises-server"></a>使用內部部署伺服器
若不想在 Azure 中執行基本伺服器，您可以在 Hyper-V VM、VMware VM 或實體主機上執行伺服器。 伺服器硬體的最低建議需求是雙核心與 4 GB 的 RAM。 下表列出支援的作業系統：

| 作業系統 | 平台 | SKU |
|:--- | --- |:--- |
| Windows Server 2016 和最新的 SP |64 位元 |Standard、Datacenter、Essentials (MABS v2 和更新版本) |
| Windows Server 2012 R2 和最新的 SP |64 位元 |Standard、Datacenter、Foundation |
| Windows Server 2012 和最新的 SP |64 位元 |Datacenter、Foundation、Standard |
| Windows Storage Server 2012 R2 和最新的 SP |64 位元 |Standard、Workgroup |
| Windows Storage Server 2012 和最新的 SP |64 位元 |Standard、Workgroup |

您可以使用 Windows Server Deduplication 為 DPM 儲存體刪除重複資料。 深入了解在 Hyper-V VM 中部署時， [DPM 和重複資料刪除](https://technet.microsoft.com/library/dn891438.aspx) 如何搭配運作。

> [!NOTE]
> Azure 備份伺服器的設計目的是在專用、單一用途的伺服器上執行。 您無法在下列位置安裝 Azure 備份伺服器︰
> - 執行為網域控制站的電腦
> - 安裝應用程式伺服器角色所在的電腦
> - 本身是 System Center Operations Manager 管理群組的電腦
> - Exchange Server 執行所在的電腦
> - 本身是叢集節點的電腦

Azure 備份伺服器一律加入網域。 如果您打算將伺服器移到不同的網域，請先安裝 Azure 備份伺服器，然後將伺服器加入新網域。 若在部署後將現有的 Azure 備份伺服器機器移至新網域，該動作將「不受支援」 。

無論您是將備份資料傳送至 Azure，或保存在本機，都必須向復原服務保存庫註冊 Azure 備份伺服器。

[!INCLUDE [backup-create-rs-vault.md](../../includes/backup-create-rs-vault.md)]

### <a name="set-storage-replication"></a>設定儲存體複寫
儲存體複寫選項有異地備援儲存體和本地備援儲存體可供您選擇。 根據預設，復原服務保存庫會使用異地備援儲存體。 如果這個保存庫是您的主要保存庫，儲存體選項請保持設定為異地備援儲存體。 如果您想要更便宜但不持久的選項，請選擇本地備援儲存體。 在 [Azure 儲存體複寫概觀](../storage/common/storage-redundancy.md)中，深入了解[異地備援](../storage/common/storage-redundancy-grs.md)和[本地備援](../storage/common/storage-redundancy-lrs.md)儲存體選項。

若要編輯儲存體複寫設定︰

1. 選取保存庫以開啟保存庫儀表板和 [設定] 功能表。 如果 [設定] 功能表未開啟，請按一下保存庫儀表板中的 [所有設定]。
2. 在 [設定] 功能表上按一下 [備份基礎結構] > [備份組態]，開啟 [備份組態] 刀鋒視窗。 在 [備份組態]  功能表上，選擇保存庫的儲存體複寫選項。

    ![備份保存庫的清單](./media/backup-azure-vms-first-look-arm/choose-storage-configuration-rs-vault.png)

    選擇好保存庫的儲存體選項後，就可以開始建立 VM 與保存庫的關聯。 若要開始關聯，請探索及註冊 Azure 虛擬機器。

## <a name="software-package"></a>軟體封裝
### <a name="downloading-the-software-package"></a>下載軟體封裝
1. 登入 [Azure 入口網站](https://portal.azure.com/)。
2. 如果您已開啟復原服務保存庫，請繼續步驟 3。 如果您並未開啟復原服務保存庫，但位於 Azure 入口網站中，請在主功能表上按一下 [瀏覽]。

   * 在資源清單中輸入 **復原服務**。
   * 當您開始輸入時，清單將會根據您輸入的文字進行篩選。 當您看到 [復原服務保存庫] 時，請按一下它。

     ![建立復原服務保存庫的步驟 1](./media/backup-azure-microsoft-azure-backup/open-recovery-services-vault.png)

     隨即會出現 [復原服務保存庫] 清單。
   * 在 [復原服務保存庫] 清單中選取保存庫。

     選取的保存庫儀表板隨即開啟。

     ![開啟 [保存庫] 刀鋒視窗](./media/backup-azure-microsoft-azure-backup/vault-dashboard.png)
3. 根據預設，[設定] 刀鋒視窗隨即會開啟。 如果該刀鋒視窗未開啟，請按一下 [設定]  以開啟 [設定] 刀鋒視窗。

    ![開啟 [保存庫] 刀鋒視窗](./media/backup-azure-microsoft-azure-backup/vault-setting.png)
4. 按一下 [備份] 以開啟 [開始使用] 精靈。

    ![備份開始使用](./media/backup-azure-microsoft-azure-backup/getting-started-backup.png)

    在開啟的 [開始使用備份] 刀鋒視窗中，系統會自動選取 [備份目標]。

    ![Backup-goals-default-opened](./media/backup-azure-microsoft-azure-backup/getting-started.png)

5. 在 [備份目標] 刀鋒視窗中，從 [您的工作負載在何處執行] 功能表中，選取 [內部部署]。

    ![內部部署和做為目標的工作負載](./media/backup-azure-microsoft-azure-backup/backup-goals-azure-backup-server.png)

    從 [你想要備份哪些項目？] 下拉式功能表中，選取您要使用 Azure 備份伺服器保護的工作負載，然後按一下 [確定]。

    [開始使用備份功能] 精靈會切換 [準備基礎結構] 選項以將工作負載備份至 Azure。

   > [!NOTE]
   > 如果您只想要備份檔案和資料夾，我們建議使用 Azure 備份代理程式並遵循[初步了解：備份檔案和資料夾](backup-try-azure-backup-in-10-mins.md)一文中的指引。 如果您想要保護比檔案和資料夾更多的工作負載，或未來有擴充保護需求的規劃，請選取那些工作負載。
   >
   >

    ![開始使用精靈變更](./media/backup-azure-microsoft-azure-backup/getting-started-prep-infra.png)

6. 在開啟的 [準備基礎結構] 刀鋒視窗中，按一下可供安裝 Azure 備份伺服器和下載保存庫認證的 [下載] 連結。 在將 Azure 備份伺服器註冊到復原服務保存庫期間，您必須使用保存庫認證。 連結會使您進入可下載軟體套件的 [下載中心]。

    ![準備用於 Azure 備份伺服器的基礎結構](./media/backup-azure-microsoft-azure-backup/azure-backup-server-prep-infra.png)

7. 選取所有檔案，然後按 [下一步] 。 下載 [Microsoft Azure 備份] 下載頁面中的所有檔案，並將所有檔案放在相同的資料夾中。

    ![下載中心 1](./media/backup-azure-microsoft-azure-backup/downloadcenter.png)

    因為所有檔案的下載大小合計 > 3G，在 10Mbps 下載連結上可能需要 60 分鐘才能完成下載。

### <a name="extracting-the-software-package"></a>將軟體封裝解壓縮
下載所有檔案之後，按一下 [MicrosoftAzureBackupInstaller.exe] 。 這會啟動 [Microsoft Azure 備份安裝精靈]  ，並將安裝程式檔案解壓縮至您所指定的位置。 繼續執行精靈，然後按一下 [解壓縮]  按鈕，以開始解壓縮程序。

> [!WARNING]
> 至少 4 GB 的可用空間，才能將安裝程式檔案解壓縮。
>
>

![Microsoft Azure 備份安裝精靈](./media/backup-azure-microsoft-azure-backup/extract/03.png)

解壓縮程序完成之後，請選取可啟動剛解壓縮 setup.exe 的方塊，以開始安裝 Microsoft Azure 備份伺服器，然後按一下 [完成] 按鈕。

### <a name="installing-the-software-package"></a>安裝軟體封裝
1. 按一下 [Microsoft Azure 備份]  啟動安裝精靈。

    ![Microsoft Azure 備份安裝精靈](./media/backup-azure-microsoft-azure-backup/launch-screen2.png)
2. 在 [歡迎使用] 畫面上按 [下一步]  按鈕。 這會讓您進入 [必要條件檢查]  區段。 在此畫面上按一下 [檢查]，以判斷是否符合 Azure 備份伺服器的硬體和軟體必要條件。 如果完全符合所有必要條件，您會看到訊息指出機器符合需求。 按 [下一步]  按鈕。

    ![Azure 備份伺服器 - 歡迎使用和必要條件檢查](./media/backup-azure-microsoft-azure-backup/prereq/prereq-screen2.png)
3. Microsoft Azure 備份伺服器需要 SQL Server Standard。 此外，Azure 備份伺服器安裝套件在必要時會隨附適當的 SQL Server 二進位檔 (如果您不想要使用自己的 SQL)。 在進行新的 Azure 備份伺服器安裝時，您應該選擇 [在此安裝中安裝新的 SQL Server 執行個體]，然後按一下 [檢查並安裝] 按鈕。 成功安裝必要條件後，按 [下一步] 。

    ![Azure 備份伺服器 - SQL 檢查](./media/backup-azure-microsoft-azure-backup/sql/01.png)

    如果發生失敗且建議您重新啟動電腦，請依指示進行，然後按一下 [再檢查一次] 。

   > [!NOTE]
   > Azure 備份伺服器不會使用遠端 SQL Server 執行個體。 Azure 備份伺服器所使用的執行個體必須是本機的。
   >
   >
4. 提供 Microsoft Azure 備份伺服器檔案的安裝位置，按 [下一步] 。

    ![Microsoft Azure 備份必要條件 2](./media/backup-azure-microsoft-azure-backup/space-screen.png)

    備份至 Azure 需要暫存位置。 請確保暫存位置至少為打算備份至雲端的資料的 5%。 在磁碟保護方面，安裝完成之後必須設定獨立的磁碟。 如需有關存放集區的詳細資訊，請參閱 [設定存放集區和磁碟儲存體](https://technet.microsoft.com/library/hh758075.aspx)。
5. 為受限的本機使用者帳戶提供強式密碼，按 [下一步] 。

    ![Microsoft Azure 備份必要條件 2](./media/backup-azure-microsoft-azure-backup/security-screen.png)
6. 選取是否要使用 *Microsoft Update* 檢查更新，按 [下一步]。

   > [!NOTE]
   > 建議讓 Windows Update 重新導向至 Microsoft Update，此網站為 Windows 和 Microsoft Azure 備份伺服器等其他產品提供安全性和重要更新。
   >
   >

    ![Microsoft Azure 備份必要條件 2](./media/backup-azure-microsoft-azure-backup/update-opt-screen2.png)
7. 檢閱「設定值摘要」，按一下 [安裝]。

    ![Microsoft Azure 備份必要條件 2](./media/backup-azure-microsoft-azure-backup/summary-screen.png)
8. 安裝分階段執行。 在第一個階段中，會將 Microsoft Azure 復原服務代理程式安裝在伺服器上。 精靈也會檢查網際網路連線。 如果可連線至網際網路，您可以繼續安裝，否則必須提供 Proxy 詳細資料來連線到網際網路。

    下一個步驟是設定 Microsoft Azure 復原服務代理程式。 在設定的過程中，您將必須提供保存庫認證，以向復原服務保存庫註冊機器。 您也須提供複雜密碼來加密/解密 Azure 與您的內部部署之間所傳送的資料。 您可以自動產生複雜密碼，或提供您自己的複雜密碼，最少 16 個字元。 繼續執行精靈，直到代理程式完成設定。

    ![Azure 備份伺服器必要條件 2](./media/backup-azure-microsoft-azure-backup/mars/04.png)
9. Microsoft Azure 備份伺服器順利完成註冊後，整體安裝精靈會繼續安裝及設定 SQL Server 和 Azure 備份伺服器的元件。 SQL Server 元件安裝完成後，會安裝 Azure 備份伺服器元件。

    ![Azure 備份伺服器](./media/backup-azure-microsoft-azure-backup/final-install/venus-installation-screen.png)

當安裝步驟完成時，產品的桌面圖示也將一併建立完成。 只要按兩下該圖示，即可啟動產品。

### <a name="add-backup-storage"></a>新增備份儲存體
第一個備份複本會保存在連接至 Azure 備份伺服器機器的儲存體上。 如需有關新增磁碟的詳細資訊，請參閱 [設定存放集區和磁碟儲存體](https://technet.microsoft.com/library/hh758075.aspx)。

> [!NOTE]
> 即使您打算將資料傳送至 Azure，也必須新增備份儲存體。 在目前的「Azure 備份伺服器」架構中，「Azure 備份」保存庫會保存資料的「第二個」  複本，而本機儲存體則是保存第一個 (必要的) 備份複本。
>
>

## <a name="network-connectivity"></a>網路連線
Azure 備份伺服器需要連線至 Azure 備份服務，產品才能順利運作。 若要驗證機器是否連接至 Azure，請在Azure 備份伺服器 PowerShell 主控台中使用 ```Get-DPMCloudConnection``` Cmdlet。 如果 Cmdlet 的輸出為 TRUE，表示連線存在，否則沒有連線。

同時，Azure 訂用帳戶也必須處於狀況良好狀態。 若您想了解訂用帳戶狀態並加以管理，請登入 [訂用帳戶入口網站](https://account.windowsazure.com/Subscriptions)。

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
如果您有防火牆或 Proxy 以致無法存取 Azure，您必須將防火牆/Proxy 設定檔中的下列網域位址列入允許清單中：

* `http://www.msftncsi.com/ncsi.txt`
* \*.Microsoft.com
* \*.WindowsAzure.com
* \*.microsoftonline.com
* \*.windows.net

在 Azure 的連線已還原至 Azure 備份伺服器機器之後，可執行的作業將取決於 Azure 訂用帳戶狀態。 上表詳細列出機器「已連接」之後所允許之作業的相關資訊。

### <a name="handling-subscription-states"></a>處理訂用帳戶狀態
您可以將 Azure 訂用帳戶從「已過期」或「已取消佈建」狀態變更為「作用中」狀態。 不過，當狀態不是「作用中」 時，此動作會對產品的行為造成某些影響：

* 「已取消佈建」  的訂用帳戶在取消佈建的這段期間會失去功能。 切換為「作用中」 時，就會恢復產品的備份/還原功能。 此外，只要以夠大的保留期間來保存，也可以擷取本機磁碟上的備份資料。 不過，一旦訂用帳戶進入「已取消佈建」  狀態，Azure 中的備份資料便會遺失而無法復原。
* 「已過期」的訂用帳戶只有在恢復「作用中」狀態之前會失去功能。 任何針對訂用帳戶處於「已過期」  期間所排定的備份，都不會執行。

## <a name="troubleshooting"></a>疑難排解
如果 Microsoft Azure 備份伺服器在安裝階段 (或是備份或還原) 失敗且出現錯誤，請參閱這份[錯誤碼文件](https://support.microsoft.com/kb/3041338)，以取得詳細資訊。
您也可以參考 [Azure 備份相關的常見問題集](backup-azure-backup-faq.md)

## <a name="next-steps"></a>後續步驟
您可以在 Microsoft TechNet 網站上取得有關 [準備 DPM 的環境](https://technet.microsoft.com/library/hh758176.aspx) 的詳細資訊。 其中也包含可據以部署和使用 Azure 備份伺服器之支援組態的相關資訊。

請參閱這些文章，以深入了解使用 Microsoft Azure 備份伺服器來保護工作負載。

* [SQL Server 備份](backup-azure-backup-sql.md)
* [SharePoint 伺服器備份](backup-azure-backup-sharepoint.md)
* [替代伺服器備份](backup-azure-alternate-dpm-server.md)
