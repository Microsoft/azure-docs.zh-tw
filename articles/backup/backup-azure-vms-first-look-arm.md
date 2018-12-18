---
title: 初步了解：使用復原服務保存庫保護 Azure VM
description: 使用復原服務保存庫保護 Azure VM。 使用 Resource Manager 部署的 VM、傳統部署的 VM 和進階儲存體 VM、已在受控磁碟中加密 VM 的備份來保護您的資料。 建立和註冊復原服務保存庫。 在 Azure 中註冊 VM、建立原則和保護 VM。
services: backup
author: markgalioto
manager: carmonm
keyword: backups; vm backup
ms.service: backup
ms.topic: conceptual
ms.date: 08/01/2018
ms.author: markgal
ms.custom: H1Hack27Feb2017
keywords: 備份; VM 備份
ms.openlocfilehash: c3bb198a65f907080023af707080f60c9c1fccbf
ms.sourcegitcommit: d4c076beea3a8d9e09c9d2f4a63428dc72dd9806
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/01/2018
ms.locfileid: "39398884"
---
# <a name="back-up-azure-virtual-machines-to-recovery-services-vault"></a>將 Azure 虛擬機器備份到復原服務保存庫

這篇文章說明如何從虛擬機器作業功能表或復原服務保存庫設定虛擬機器的安全防護。 復原服務保存庫可保護︰

* Azure Resource Manager 部署的 VM
* 傳統 VM
* 標準儲存體 VM
* 進階儲存體 VM
* 在受控磁碟上執行的 VM
* 使用 Azure 磁碟加密來加密的 VM
* 使用 VSS 之 Windows VM 和使用自訂快照前與快照後指令碼之 Linux VM 的應用程式一致備份

如需保護進階儲存體 VM 的詳細資訊，請參閱[備份和還原進階儲存體 VM](backup-introduction-to-azure-backup.md#using-premium-storage-vms-with-azure-backup)一文。 如需受控磁碟 VM 支援的詳細資訊，請參閱[備份及還原受控磁碟上的 VM](backup-introduction-to-azure-backup.md#using-managed-disk-vms-with-azure-backup)。 如需有關 Linux VM 備份之前置和後置指令碼架構的詳細資訊，請參閱[使用前置指令碼和後置指令碼的應用程式一致 Linux VM 備份](https://docs.microsoft.com/azure/backup/backup-azure-linux-app-consistent)。

若要深入了解可備份和無法備份哪些項目，請參閱[預備好備份 Azure VM 的環境](backup-azure-arm-vms-prepare.md#limitations-when-backing-up-and-restoring-a-vm)。

> [!NOTE]
> 備份服務會建立與 VM 資源群組不同的資源群組，來儲存還原點集合。 建議客戶請勿鎖定建立給備份服務使用的資源群組。
備份服務建立的資源群組命名格式為：AzureBackupRG_`<Geo>`_`<number>`
<br>例如：AzureBackupRG_northeurope_1
>
>

根據您想要保護的虛擬機器數目，您可以從不同的起始點開始。 如果您想要以單一作業備份多個虛擬機器，請移至復原服務保存庫，然後[從保存庫儀表板起始備份工作](backup-azure-vms-first-look-arm.md#configure-the-backup-job-from-the-recovery-services-vault)。 如果您想要備份單一虛擬機器，便可以[從 VM 作業功能表起始備份作業](backup-azure-vms-first-look-arm.md#configure-the-backup-job-from-the-vm-operations-menu)。

## <a name="configure-the-backup-job-from-the-vm-operations-menu"></a>從 VM 作業功能表設定備份作業

使用下列步驟，從虛擬機器作業功能表設定備份作業。 下列步驟僅適用於 Azure 入口網站中的虛擬機器。

1. 登入 [Azure 入口網站](https://portal.azure.com/)。
2. 在 [中樞] 功能表上按一下 [所有服務]，然後在 [篩選] 對話方塊中輸入**虛擬機器**。 當您輸入時，會一邊篩選資源清單。 當您看到虛擬機器時，請選取它。

  ![顯示如何從所有服務瀏覽至虛擬機器的螢幕擷取畫面](./media/backup-azure-vms-first-look-arm/open-vm-from-hub.png)

  訂用帳戶中的虛擬機器 (VM) 清單隨即出現。

  ![訂用帳戶中的 VM 清單隨即出現。](./media/backup-azure-vms-first-look-arm/list-of-vms.png)

3. 從清單中選取要備份的 VM。

  ![訂用帳戶中的 VM 清單隨即出現。](./media/backup-azure-vms-first-look-arm/list-of-vms-selected.png)

  當您選取 VM 時，虛擬機器清單便會轉移至左邊，並開啟虛擬機器管理功能表和虛擬機器儀表板。

4. 在 VM 管理功能表的 [作業] 區段中，按一下 [備份]。 </br>

  ![VM 管理功能表中的備份選項](./media/backup-azure-vms-first-look-arm/vm-management-menu.png)

  [啟用備份] 功能表隨即開啟。

  ![VM 管理功能表中的備份選項](./media/backup-azure-vms-first-look-arm/vm-menu-enable-backup.png)

5. 在復原服務保存庫區域中，按一下 [選取現有]，然後從下拉式清單中選擇保存庫。

  ![啟用備份精靈](./media/backup-azure-vms-first-look-arm/vm-menu-enable-backup-small.png)

  如果沒有復原服務保存庫，或您想要使用新的保存庫，請按一下 [新建]，並提供新保存庫的名稱。 已在與虛擬機器相同的資源群組和相同區域中建立新的保存庫。 如果您想要使用不同的值建立復原服務保存庫，請參閱如何[建立復原服務保存庫](backup-azure-vms-first-look-arm.md#create-a-recovery-services-vault-for-a-vm)一節。

6. 從 [選擇備份原則] 功能表選取一個原則。 所選原則的詳細資料會顯示在下拉式功能表底下。

  如果您想建立新原則或編輯現有的原則，請按一下 [建立 (或編輯) 新原則] 以開啟備份原則編輯器。 如需定義備份原則的指示，請參閱 [定義備份原則](backup-azure-vms-first-look-arm.md#defining-a-backup-policy)。 若要將變更儲存至備份原則，並返回 [啟用備份] 功能表，請按一下 [確定]。

  ![選取備份原則](./media/backup-azure-vms-first-look-arm/set-backup-policy.png)

7. 若要套用復原服務保存庫與備份原則至虛擬機器，請按一下 [啟用備份] 以部署原則。 部署原則以讓它與保存庫和虛擬機器產生關聯。

  ![啟用備份按鈕](./media/backup-azure-vms-first-look-arm/vm-management-menu-enable-backup-button.png)

8. 您可以透過出現在入口網站的通知來追蹤組態進度。 下列範例顯示部署已經開始。

  ![啟用備份通知](./media/backup-azure-vms-first-look-arm/vm-management-blade-enable-backup-notification.png)

9. 設定進度完成後，在 VM 管理功能表上，按一下 [備份] 以開啟 [備份] 功能表並檢視可用的詳細資料。

  ![VM 備份項目檢視](./media/backup-azure-vms-first-look-arm/backup-item-view-update.png)

  在完成初始備份之前，[上次備份狀態] 會顯示為 [警告 (待執行初始備份)]。 若要查看下一個排定的備份工作何時發生，請在 [摘要] 下按一下原則的名稱。 [備份原則] 功能表隨即開啟，並顯示已排定備份的時間。

10. 若要保護虛擬機器，請按一下 [立即備份]。 

  ![按一下 [立即備份] 以執行初始備份](./media/backup-azure-vms-first-look-arm/backup-now-update.png)

  [立即備份] 功能表隨即開啟。 

  ![顯示 [立即備份] 刀鋒視窗](./media/backup-azure-vms-first-look-arm/backup-now-blade-short.png)

11. 在 [立即備份] 功能表上，按一下日曆圖示，使用日曆控制項選取此復原點保留的最後一天，然後按一下 [確定]。

  ![設定 [立即備份] 復原點保留的最後一天](./media/backup-azure-vms-first-look-arm/backup-now-blade-calendar.png)

  部署通知可讓您知道已觸發備份工作，而且您可以在 [備份工作] 頁面上監視作業的進度。

## <a name="configure-the-backup-job-from-the-recovery-services-vault"></a>設定復原服務保存庫中的備份工作
若要設定備份作業，請完成下列步驟。

1. 建立虛擬機器的復原服務保存庫。
2. 使用 Azure 入口網站選取案例、設定備份原則，以及識別要保護的項目。
3. 執行初始備份。

## <a name="create-a-recovery-services-vault-for-a-vm"></a>建立 VM 的復原服務保存庫。
復原服務保存庫是一個實體，會儲存歷來建立的所有備份和復原點。 復原服務保存庫也包含套用至受保護 VM 的備份原則。

> [!NOTE]
> 備份 VM 是本機程序。 您無法將某個區域的 VM 備份到另一個區域的復原服務保存庫。 因此，若 Azure 區域具有要備份的 VM，該區域至少必須有一個復原服務保存庫。
>
>

若要建立復原服務保存庫：

1. 如果您尚未這麼做，請使用 Azure 訂用帳戶登入 [Azure 入口網站](https://portal.azure.com/)。
2. 在 [中樞] 功能表上按一下 [所有服務]，然後在 [篩選] 對話方塊中輸入**復原服務**。 當您輸入時，會一邊篩選資源清單。 當您在清單中看到 [復原服務保存庫] 時，請按一下它。

    ![建立復原服務保存庫的步驟 1](./media/backup-try-azure-backup-in-10-mins/open-rs-vault-list.png) <br/>

    如果訂用帳戶中有復原服務保存庫，則會列出保存庫。

    ![建立復原服務保存庫的步驟 2](./media/backup-azure-vms-first-look-arm/list-of-rs-vault.png)
3. 在 [復原服務保存庫] 功能表上，按一下 [新增]。

    ![建立復原服務保存庫的步驟 2](./media/backup-try-azure-backup-in-10-mins/rs-vault-menu.png)

    [復原服務保存庫] 功能表隨即開啟，並提示您提供 [名稱]、[訂用帳戶]、[資源群組] 和 [位置]。

    ![建立復原服務保存庫的步驟 3](./media/backup-try-azure-backup-in-10-mins/rs-vault-step-3.png)

4. 在 [名稱] 中，輸入易記名稱來識別保存庫。 必須是 Azure 訂用帳戶中唯一的名稱。 輸入包含 2 到 50 個字元的名稱。 該名稱必須以字母開頭，而且只可以包含字母、數字和連字號。

5. 在 [訂用帳戶] 區段中，使用下拉式功能表來選擇 Azure 訂用帳戶。 如果您只使用一個訂用帳戶，該訂用帳戶會出現，您可以跳到下一個步驟。 如果您不確定要使用哪個訂用帳戶，請使用預設 (或建議) 的訂用帳戶。 只有在您的組織帳戶與多個 Azure 訂用帳戶相關聯時，才會有多個選擇。

6. 在 [資源群組] 區段中︰

    * 如果您想建立新的資源群組，請選取 [新建]。
    或
    * 選取 [使用現有的]﹐然後按一下下拉式功能表，以查看可用的資源群組清單。

  如需資源群組的完整資訊，請參閱 [Azure Resource Manager 概觀](../azure-resource-manager/resource-group-overview.md)。

7. 按一下 [位置]  以選取保存庫的地理區域。 此選項會決定您的備份資料要傳送到哪個地理區域。

  > [!IMPORTANT]
  > 如果您不確定 VM 的所在區域，請關閉保存庫建立對話方塊，並移至入口網站的虛擬機器清單。 如果您在多個區域中有虛擬機器，請在每個區域中建立復原服務保存庫。 請先在第一個區域建立保存庫，再進入下一個區域。 儲存備份資料時，不需要指定儲存體帳戶，復原服務保存庫和「Azure 備份」服務會自動處理儲存體。
  >

8. 按一下 [復原服務保存庫] 功能表底部的 [建立]。

    建立復原服務保存庫可能需要一些時間。 請監視入口網站右上方區域中的狀態通知。 保存庫一旦建立好，就會出現在 [復原服務保存庫] 的清單中。 在數分鐘之後﹐如果您沒有看到您的保存庫，請按一下 [重新整理]。

    ![按一下 [重新整理] 按鈕。](./media/backup-try-azure-backup-in-10-mins/refresh-button.png)</br>

    一旦在復原服務保存庫清單中看到您的保存庫，您即可開始設定儲存體備援。

現在您已建立好保存庫，接下來要了解如何設定儲存體複寫。

### <a name="set-storage-replication"></a>設定儲存體複寫
儲存體複寫選項有異地備援儲存體和本地備援儲存體可供您選擇。 根據預設，保存庫具有異地備援儲存體。 如果復原服務保存庫是您的主要備份，請將儲存體複寫選項設為異地備援儲存體。 如果您想要更便宜但不持久的選項，請選擇本地備援儲存體。 在 [Azure 儲存體複寫概觀](../storage/common/storage-redundancy.md)中，深入了解[異地備援](../storage/common/storage-redundancy-grs.md)和[本地備援](../storage/common/storage-redundancy-lrs.md)儲存體選項。

若要編輯儲存體複寫設定︰

1. 從 [復原服務保存庫] 功能表，選取新的保存庫。

  ![從復原服務保存庫清單中選取新的保存庫](./media/backup-try-azure-backup-in-10-mins/rs-vault-list.png)

  當您選取該保存庫時，[設定] 功能表 (頂端有保存庫名稱) 和保存庫儀表板隨即開啟。

  ![檢視新保存庫的儲存體組態](./media/backup-try-azure-backup-in-10-mins/set-storage-configuration-update.png)

2. 在新保存庫的 [管理] 功能表中，使用垂直滑桿捲動至 [管理] 區段，然後按一下 [備份基礎結構] 以開啟 [備份基礎結構] 功能表。
 
   ![為新保存庫設定儲存體組態](./media/backup-try-azure-backup-in-10-mins/set-storage-config-bkup-infra.png)

3. 在 [備份基礎結構] 功能表中，按一下 [備份設定] 開啟 [備份設定] 功能表。

    ![為新保存庫設定儲存體組態](./media/backup-try-azure-backup-in-10-mins/set-storage-open-infra.png)
4. 為保存庫選擇適當的儲存體複寫選項。

    ![儲存體組態選項](./media/backup-try-azure-backup-in-10-mins/choose-storage-configuration.png)

    根據預設，保存庫具有異地備援儲存體。 如果您使用 Azure 做為主要的備份儲存體端點，請繼續使用 [異地備援]。 如果您未使用 Azure 做為主要的備份儲存體端點，則選擇 [本地備援]，以減少 Azure 儲存體成本。 在此[儲存體備援概觀](../storage/common/storage-redundancy.md)中，深入了解[異地備援](../storage/common/storage-redundancy-grs.md)和[本地備援](../storage/common/storage-redundancy-lrs.md)儲存體選項。


## <a name="select-a-backup-goal-set-policy-and-define-items-to-protect"></a>選取備份目標、設定原則及定義要保護的項目
在向保存庫註冊 VM 前，請先執行探索程序，以確保能夠識別任何新增至訂用帳戶的新虛擬機器。 此程序會在 Azure 中查詢訂用帳戶中的虛擬機器清單，以及其他資訊，例如雲端服務名稱、區域等。 在 Azure 入口網站中，案例是指您要放入復原服務保存庫中的項目。 原則是復原點擷取頻率和時間的排程。 原則也會包含復原點的保留範圍。

1. 如果您已開啟復原服務保存庫，請繼續步驟 2。 否則，請按一下 [所有服務]。 輸入**復原服務**，然後按一下 [復原服務保存庫]。

    ![建立復原服務保存庫的步驟 1](./media/backup-try-azure-backup-in-10-mins/open-rs-vault-list.png) <br/>

    復原服務保存庫清單隨即出現。

    ![復原服務保存庫清單檢視](./media/backup-azure-arm-vms-prepare/rs-list-of-vaults.png)

    在復原服務保存庫清單中，選取保存庫以開啟其儀表板。

     ![開啟保存庫功能表](./media/backup-azure-arm-vms-prepare/new-vault-settings-blade.png)

2. 在 [保存庫儀表板] 功能表中，按一下 [備份] 以開啟 [備份] 功能表。

    ![開啟 [備份] 功能表](./media/backup-azure-arm-vms-prepare/backup-button.png)

    [備份] 和 [備份目標] 功能表隨即開啟。

    ![開啟 [案例] 功能表](./media/backup-azure-arm-vms-prepare/select-backup-goal-1.png)
3. 在 [備份目標] 功能表中，從 [您的工作負載在何處執行] 下拉式功能表中，選取 Azure。 從 [欲備份的項目] 下拉式清單，選擇 [虛擬機器]，然後按一下 [確定]。

    這些動作會向保存庫註冊 VM 擴充功能。 [備份目標] 功能表隨即關閉，然後開啟 [備份原則] 功能表。

    ![開啟 [案例] 功能表](./media/backup-azure-arm-vms-prepare/select-backup-goal-2.png)

4. 在 [備份原則] 功能表中，選取您要套用至保存庫的備份原則。

    ![選取備份原則](./media/backup-azure-arm-vms-prepare/setting-rs-backup-policy-new.png)

    預設原則的詳細資料便會列在下拉式功能表之下。 如果您想要建立原則，請在下拉式功能表中選取 [建立新的]  。 如需定義備份原則的指示，請參閱 [定義備份原則](backup-azure-vms-first-look-arm.md#defining-a-backup-policy)。
    按一下 [確定] 讓備份原則與保存庫建立關聯。

    [備份原則] 功能表隨即關閉，然後開啟 [選取虛擬機器] 功能表。
5. 在 [選取虛擬機器] 功能表中，選擇要與指定原則建立關聯的虛擬機器，然後按一下 [確定]。

    ![選取工作負載](./media/backup-azure-arm-vms-prepare/select-vms-to-backup.png)

    選取的虛擬機器便會接受驗證。 如果沒看到應該要有的虛擬機器，請確認其位於和復原服務保存庫相同的 Azure 位置，且尚未受到保護。 復原服務保存庫的位置會顯示在保存庫儀表板上。

6. 現在您已定義保存庫的所有設定，接下來在 [備份] 功能表中按一下 [啟用備份]，以將原則部署至保存庫和 VM。 部署備份原則不會建立虛擬機器的初始復原點。

    ![啟用備份](./media/backup-azure-arm-vms-prepare/vm-validated-click-enable.png)

成功啟用備份後，備份原則就會依排程執行。 不過，請繼續初始化第一個備份工作。

## <a name="initial-backup"></a>初始備份
在虛擬機器上部署好備份原則後，並不表示您已備份好資料。 根據預設，第一個排定的備份 (如備份原則中所定義) 即為初始備份。 在執行初始備份之前，[備份作業] 功能表上的 [上次備份狀態] 會顯示為 [警告 (待執行初始備份)]。

![待備份](./media/backup-azure-vms-first-look-arm/initial-backup-not-run.png)

除非您的初始備份預計會馬上開始，否則建議您執行 [立即備份] 。

若要執行初始備份作業：

1. 在保存庫儀表板中，按一下 [備份項目] 下的數字，或按一下 [備份項目] 圖格。 <br/>
  ![設定圖示](./media/backup-azure-vms-first-look-arm/rs-vault-config-vm-back-up-now-1.png)

  [備份項目] 功能表隨即開啟。

  ![備份項目](./media/backup-azure-vms-first-look-arm/back-up-items-list.png)

2. 在 [備份項目] 功能表中，選取該項目。

  ![設定圖示](./media/backup-azure-vms-first-look-arm/back-up-items-list-selected.png)

  [備份項目] 清單隨即開啟。 <br/>

  ![備份作業已觸發](./media/backup-azure-vms-first-look-arm/backup-items-not-run.png)

3. 在 [備份項目] 清單上，按一下省略符號 **...** 以開啟操作功能表。

  ![內容功能表](./media/backup-azure-vms-first-look-arm/context-menu.png)

  操作功能表隨即出現。

  ![內容功能表](./media/backup-azure-vms-first-look-arm/context-menu-small.png)

4. 在操作功能表上，按一下 [立即備份]。

  ![內容功能表](./media/backup-azure-vms-first-look-arm/context-menu-small-backup-now.png)

  [立即備份] 功能表隨即開啟。

  ![顯示 [立即備份] 功能表](./media/backup-azure-vms-first-look-arm/backup-now-blade-short.png)

5. 在 [立即備份] 功能表上，按一下日曆圖示，使用日曆控制項選取此復原點保留的最後一天，然後按一下 [備份]。

  ![設定 [立即備份] 復原點保留的最後一天](./media/backup-azure-vms-first-look-arm/backup-now-blade-calendar.png)

  部署通知可讓您知道已觸發備份工作，而且您可以在 [備份工作] 頁面上監視作業的進度。 根據您的 VM 大小，建立初始備份可能需要花一點時間。

6. 若要在保存庫儀表板上檢視或追蹤初始備份的狀態，請在 [備份工作] 圖格上，按一下 [進行中]。

  ![備份工作圖格](./media/backup-azure-vms-first-look-arm/open-backup-jobs-1.png)

  [備份作業] 功能表隨即開啟。

  ![備份工作圖格](./media/backup-azure-vms-first-look-arm/backup-jobs-in-jobs-view-1.png)

  在 [備份作業] 功能表中，您可以看到所有作業的狀態。 檢查您 VM 的備份作業是否仍在進行中或已經完成。 當備份工作完成時，狀態會是「完成」 。

  > [!NOTE]
  > 在備份工作進行時，Azure 備份服務會對每個 VM 中的備份擴充功能發出命令，以排清所有寫入並取得一致的快照。
  >
  >


[!INCLUDE [backup-create-backup-policy-for-vm](../../includes/backup-create-backup-policy-for-vm.md)]

## <a name="install-the-vm-agent-on-the-virtual-machine"></a>在虛擬機器中安裝 VM 代理程式
如果需要便會提供此資訊。 Azure VM 代理程式必須安裝在 Azure 虛擬機器上，備份擴充功能才能運作。 不過，如果 VM 是建立自 Azure 資源庫，則 VM 代理程式已存在於虛擬機器上。 從內部部署資料中心移轉的 VM 不會安裝 VM 代理程式。 在這種情況下，必須安裝 VM 代理程式。 如果您在備份 Azure VM 時遇到問題，請先確定已在虛擬機器上正確安裝 Azure VM 代理程式 (請參閱下表)。 如果您建立自訂 VM，請先安裝 VM 代理程式，再佈建虛擬機器。

深入了解 [VM 代理程式](https://go.microsoft.com/fwLink/?LinkID=390493&clcid=0x409)和[如何安裝](../virtual-machines/windows/classic/manage-extensions.md?toc=%2fazure%2fvirtual-machines%2fwindows%2fclassic%2ftoc.json)。

下表提供適用於 Windows 和 Linux VM 之 VM 代理程式的其他資訊。

| **作業** | **Windows** | **Linux** |
| --- | --- | --- |
| 安裝 VM 代理程式 |<li>下載並安裝 [代理程式 MSI](http://go.microsoft.com/fwlink/?LinkID=394789&clcid=0x409)。 您需要有系統管理員權限，才能完成安裝。 <li>[更新 VM 屬性](http://blogs.msdn.com/b/mast/archive/2014/04/08/install-the-vm-agent-on-an-existing-azure-vm.aspx) 以表示已安裝代理程式。 |<li> 從 GitHub 安裝最新的 [Linux 代理程式](https://github.com/Azure/WALinuxAgent) 。 您需要有系統管理員權限，才能完成安裝。 <li> [更新 VM 屬性](http://blogs.msdn.com/b/mast/archive/2014/04/08/install-the-vm-agent-on-an-existing-azure-vm.aspx) 以表示已安裝代理程式。 |
| 更新 VM 代理程式 |更新 VM 代理程式與重新安裝 [VM 代理程式二進位檔](http://go.microsoft.com/fwlink/?LinkID=394789&clcid=0x409)一樣簡單。 <br>確定在更新 VM 代理程式時，沒有任何執行中的備份作業。 |請遵循[更新 Linux VM 代理程式](../virtual-machines/linux/update-agent.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json)的指示。 <br>確定在更新 VM 代理程式時，沒有任何執行中的備份作業。 |
| 驗證 VM 代理程式安裝 |<li>瀏覽至 Azure VM 中的 C:\WindowsAzure\Packages 資料夾。 <li>您應該會發現 WaAppAgent.exe 檔案已存在。<li> 在該檔案上按一下滑鼠右鍵，前往 [屬性]，然後選取 [詳細資料] 索引標籤。[產品版本] 欄位應為 2.6.1198.718 或更高版本。 |N/A |

### <a name="backup-extension"></a>備份擴充功能
虛擬機器上安裝了 VM 代理程式後，Azure 備份服務就會在 VM 代理程式上安裝備份擴充功能。 Azure 備份服務無需使用者介入，即可順暢地升級和修補備份擴充功能。

備份服務會安裝備份擴充功能，即使 VM 並未執行。 執行中的 VM 提供了取得應用程式一致復原點的絕佳機會。 不過，即使 VM 已關閉而無法安裝擴充功能，Azure 備份服務仍會繼續備份 VM。 這種類型的備份稱為離線 VM，復原點是「當機時保持一致」。

## <a name="troubleshooting-information"></a>疑難排解資訊
如果您在完成本文中的某些工作時遇到問題，請參閱 [疑難排解指引](backup-azure-vms-troubleshoot.md)。

## <a name="pricing"></a>價格
備份 Azure VM 的成本是根據受保護執行個體的數目。 如需受保護執行個體的定義，請參閱[什麼是受保護執行個體](backup-introduction-to-azure-backup.md#what-is-a-protected-instance)。 如需計算備份虛擬機器成本的範例，請參閱[如何計算受保護執行個體](backup-azure-vms-introduction.md#calculating-the-cost-of-protected-instances)。 請參閱 Azure 備份定價頁面以取得[備份定價](https://azure.microsoft.com/pricing/details/backup/)的相關資訊。

## <a name="questions"></a>有疑問嗎？
如果您有問題，或希望我們加入任何功能，請 [傳送意見反應給我們](http://aka.ms/azurebackup_feedback)。
