---
title: 安裝 Azure 備份伺服器 v2
description: Azure 備份伺服器 v2 可提供您經過強化的備份功能，讓您保護 VM、檔案和資料夾以及工作負載等項目。 了解如何安裝或升級為 Azure 備份伺服器 v2。
services: backup
author: markgalioto
manager: carmonm
ms.service: backup
ms.topic: conceptual
ms.date: 05/15/2017
ms.author: adigan
ms.openlocfilehash: a458a46f3775a593f369d5acb967fc90d61efde8
ms.sourcegitcommit: 4de6a8671c445fae31f760385710f17d504228f8
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/08/2018
ms.locfileid: "39628336"
---
# <a name="install-azure-backup-server-v2"></a>安裝 Azure 備份伺服器 v2

Azure 備份伺服器可協助保護您的虛擬機器 (VM)、工作負載以及檔案和資料夾等項目。 Azure 備份伺服器 v2 是以 Azure 備份伺服器 v1 作為建置基礎，並可提供您 v1 所沒有的新功能。 如需 v1 和 v2 的功能比較，請參閱 [Azure 備份伺服器保護矩陣](backup-mabs-protection-matrix.md)。 

備份伺服器 v2 中新增的功能是備份伺服器 v1 的升級。 不過，並非一定要有備份伺服器 v1 才能安裝備份伺服器 v2。 如果您想要從備份伺服器 v1 升級為備份伺服器 v2，請在備份伺服器保護伺服器上安裝備份伺服器 v2。 您現有的備份伺服器設定會保持不變。

您可以在 Windows Server 2016 或 Windows Server 2012 R2 上安裝備份伺服器 v2。 若要利用新功能 (例如 System Center 2016 Data Protection Manager 的新式備份儲存體)，您必須在 Windows Server 2016 上安裝備份伺服器 v2。 在安裝或升級為備份伺服器 v2 之前，請先閱讀[安裝必要條件](https://docs.microsoft.com/system-center/dpm/install-dpm#setup-prerequisites)。

> [!NOTE]
> Azure 備份伺服器具有和 System Center Data Protection Manager 一樣的程式碼基底。 備份伺服器 v1 相當於 Data Protection Manager 2012 R2。 備份伺服器 v2 相當於 Data Protection Manager 2016。 本文偶爾會參考 Data Protection Manager 文件。
>
>

## <a name="upgrade-backup-server-to-v2"></a>將備份伺服器升級為 v2
若要從備份伺服器 v1 升級為備份伺服器 v2，請確定您的安裝具有所需更新：

- 在受保護的伺服器上[更新保護代理程式](backup-mabs-upgrade-to-v2.md#update-the-data-protection-manager-protection-agent)。
- 將 Windows Server 2012 R2 升級為 Windows Server 2016。
- 在所有生產伺服器上升級 Azure 備份伺服器遠端系統管理員。
- 確定備份已設定為繼續執行而不要重新啟動生產伺服器。


### <a name="upgrade-steps-for-backup-server-v2"></a>備份伺服器 v2 的升級步驟

1. 在下載中心[下載升級安裝程式](https://go.microsoft.com/fwlink/?LinkId=626082)。

2. 在將設定精靈解壓縮之後，先確定您已選取 [執行 setup.exe]，再選取 [完成]。

  ![設定安裝程式 - 執行設定](./media/backup-mabs-upgrade-to-v2/run-setup.png)

3. 在 Microsoft Azure 備份伺服器精靈的 [安裝] 底下，選取 [Microsoft Azure 備份伺服器]。

  ![設定安裝程式 - 選取安裝](./media/backup-mabs-upgrade-to-v2/mabs-installer-s1.png)

4. 在 [歡迎使用] 頁面上檢閱警告，然後選取 [下一步]。

  ![設定安裝程式 - [歡迎使用] 頁面](./media/backup-mabs-upgrade-to-v2/mabs-installer-s2.png)

5. 設定精靈會執行必要條件檢查以確定您的環境是否可以升級。 在 [必要條件檢查] 頁面上，選取 [檢查]。

  ![設定安裝程式 - [必要條件檢查] 頁面](./media/backup-mabs-upgrade-to-v2/mabs-installer-s3-perform-checks.png)

6. 您的環境必須通過必要條件檢查。 如果您的環境未通過檢查，請記下所發生的問題，並加以修正。 然後，選取 [再檢查一次]。 通過必要條件檢查之後，選取 [下一步]。

  ![設定安裝程式 - [再檢查一次] 按鈕](./media/backup-mabs-upgrade-to-v2/mabs-installer-s4-pass-checks.png)

7. 在 [SQL 設定] 頁面上，選取 SQL 安裝的相關選項，然後選取 [檢查並安裝]。

  ![設定安裝程式 - [SQL 設定] 頁面](./media/backup-mabs-upgrade-to-v2/mabs-installer-s5-sql-settings.png)

  檢查可能需要幾分鐘的時間。 檢查完畢後，選取 [下一步]。

  ![設定安裝程式 - [SQL 設定] 的 [檢查並安裝] 按鈕](./media/backup-mabs-upgrade-to-v2/mabs-installer-s5a-check-and-fix-settings.png)

8. 在 [安裝設定] 頁面上，對備份伺服器安裝所在的位置或臨時位置進行任何變更。 選取 [下一步] 。

  ![設定安裝程式 - [安裝設定] 頁面](./media/backup-mabs-upgrade-to-v2/mabs-installer-s6-installation-settings.png)

9. 若要完成設定精靈，請選取 [完成]。

  ![設定安裝程式 - 完成](./media/backup-mabs-upgrade-to-v2/run-setup.png)

## <a name="add-storage-for-modern-backup-storage"></a>為新式備份儲存體新增儲存體

為了提升備份儲存體效率，備份伺服器 v2 新增了磁碟區支援。 備份伺服器 v1 和備份伺服器 v2 都支援磁碟。

### <a name="add-volumes-and-disks"></a>新增磁碟區和磁碟

如果您在 Windows Server 2016 上執行備份伺服器 v2，您可以使用磁碟區來儲存備份資料。 磁碟區可節省儲存空間並提高備份速度。 磁碟區是備份伺服器的新功能，因此您必須新增磁碟區。 

當您在備份伺服器中新增磁碟區時，您可以讓磁碟區有個好記的名稱。 對您想要命名的磁碟區選取 [易記名稱] 資料行。 如有必要，您可於稍後變更名稱。 您也可以使用 PowerShell 來新增或變更磁碟區的易記名稱。

若要在管理員主控台中新增磁碟區：

1. 在 Azure 備份伺服器管理員主控台中，選取 [管理] > [磁碟儲存體] > [新增]。

  ![開啟「新增磁碟儲存體」精靈](./media//backup-mabs-upgrade-to-v2/open-add-disk-storage-wizard.png)

  「新增磁碟儲存體」精靈隨即開啟。

2. 在 [新增磁碟儲存體] 頁面上，於 [可用磁碟區] 方塊中選取磁碟區，然後選取 [新增]。

3. 在 [已選取的磁碟區] 方塊中，為磁碟區輸入易記名稱，然後選取 [確定]。

  ![新增磁碟儲存體精靈 - 新增磁碟區](./media/backup-mabs-upgrade-to-v2/add-volume.png)

  如果您想要新增磁碟，該磁碟必須屬於具有舊式儲存體的保護群組。 您只能將這些磁碟用於這些保護群組。 如果備份伺服器所擁有的來源並沒有舊式保護功能，則磁碟不會列出。

  如需新增磁碟的詳細資訊，請參閱[新增磁碟以增加舊式儲存體](http://docs.microsoft.com/system-center/dpm/upgrade-to-dpm-2016#adding-disks-to-increase-legacy-storage)。 您不能對磁碟賦予易記名稱。


### <a name="assign-workloads-to-volumes"></a>將工作負載指派給磁碟區

在備份伺服器中，您會指定要將哪些工作負載指派給哪些磁碟區。 例如，您可以將支援大量每秒輸入/輸出作業 (IOPS) 的高度耗費資源磁碟區，設定為僅儲存需要頻繁且大量備份的工作負載。 舉例來說，具有交易記錄的 SQL Server。

#### <a name="update-dpmdiskstorage"></a>Update-DPMDiskStorage

若要更新備份伺服器儲存集區磁碟區的屬性，請使用 PowerShell Cmdlet **Update-DPMDiskStorage**。

語法：

`Parameter Set: Volume`

```
Update-DPMDiskStorage [-Volume] <Volume> [[-FriendlyName] <String> ] [[-DatasourceType] <VolumeTag[]> ] [-Confirm] [-WhatIf] [<CommonParameters>]
```

您使用 PowerShell 所進行的變更都會反映在 UI 中。


## <a name="protect-data-sources"></a>保護資料來源
若要開始保護資料來源，請建立保護群組。 下列步驟會特別指出「新增保護群組」精靈中有變更或新增的地方。

若要建立保護群組：

1. 在備份伺服器管理員主控台中選取 [保護]。

2. 在工具功能區中選取 [新增]。

  [建立新保護群組] 精靈隨即開啟。

  ![建立新保護群組精靈](./media/backup-mabs-upgrade-to-v2/create-a-protection-group-1.png)

3. 在 [歡迎] 頁面上，選取 [下一步]。

4. 在 [選取保護群組類型] 頁面上，選取您要建立的保護群組類型，然後選取 [下一步]。

  ![[選取保護群組類型] 頁面](./media/backup-mabs-upgrade-to-v2/create-a-protection-group-2.png)

5. [選取群組成員] 頁面上的 [可用成員] 方塊中會列出具有保護代理程式的成員。 在此範例中，選取磁碟區 D:\ 和 E:\,，然後將其新增至 [已選取的成員]。 選取 [下一步] 。

  ![[選取群組成員] 頁面](./media/backup-mabs-upgrade-to-v2/create-a-protection-group-3.png)

6. 在 [選取資料保護方式] 頁面上輸入**保護群組名稱**，選取保護方式，然後選取 [下一步]。 如果您想要短期保護，則請選取 [磁碟] 備份方式。

  ![[選取資料保護方式] 頁面](./media/backup-mabs-upgrade-to-v2/create-a-protection-group-4.png)

7. 在 [指定短期目標] 頁面上，選取 [保留範圍] 和 [同步處理頻率] 的詳細資料。 然後，選取 [下一步]。 (選擇性) 若要變更復原點的擷取排程，請選取 [修改]。

  ![[指定短期目標] 頁面](./media/backup-mabs-upgrade-to-v2/create-a-protection-group-5.png)

8. 在 [檢閱磁碟儲存體配置] 頁面上，檢閱您所選取的資料來源詳細資料，包括資料來源大小、要佈建的空間值和目標儲存體磁碟區。

  ![[檢閱磁碟儲存體配置] 頁面](./media/backup-mabs-upgrade-to-v2/create-a-protection-group-6.png)

  儲存體磁碟區是以工作負載磁碟區配置 (使用 PowerShell 來設定) 和可用儲存體為基礎。 您可以在下拉式功能表中選取其他磁碟區來變更儲存體磁碟區。 如果您變更 [目標儲存體] 的值，[可用磁碟儲存體] 的值會動態變更，以反映 [可用空間] 和 [佈建不足的空間] 底下的值。

  如果資料來源按計劃成長，[可用磁碟儲存體] 中的 [佈建不足的空間] 資料行的值會反映另外需要的儲存體數量。 請使用此值來協助規劃您的儲存需求，以便順利進行備份。 如果值為零，則在可預見的未來將不可能發生任何儲存問題。 如果值為零以外的數字，則表示所配置的儲存體不足 (根據您的保護原則和受保護成員的資料大小)。

  ![配置不足的磁碟儲存體](./media/backup-mabs-upgrade-to-v2/create-a-protection-group-7.png)

  若要完成保護群組的建立，請完成精靈。

## <a name="migrate-legacy-storage-to-modern-backup-storage"></a>將舊式儲存體移轉至新式備份儲存體
在安裝或升級為備份伺服器 v2，然後將作業系統升級為 Windows Server 2016 之後，請將保護群組更新為使用新式備份儲存體。 根據預設，系統不會變更保護群組。 保護群組會繼續依照一開始的設定方式運作。 

您可以選擇是否將保護群組更新為使用新式備份儲存體。 若要更新保護群組，請使用保留資料選項來停止保護所有資料來源。 然後，將資料來源新增至新的保護群組。

1. 在 System Center 2016 DPM 管理主控台中選取 [保護] 功能。 在 [保護群組成員] 清單中，以滑鼠右鍵按一下成員，然後選取 [停止保護成員]。

  ![停止保護成員](http://docs.microsoft.com/system-center/dpm/media/upgrade-to-dpm-2016/dpm-2016-stop-protection1.png)

2. 在 [從群組中移除] 對話方塊中，檢閱儲存集區中已使用的磁碟空間和可用空間。 預設值是讓復原點留在磁碟上，並讓復原點按照所關聯的保留原則來到期。 選取 [確定] 。

  如果您想要立即將已使用的磁碟空間歸還給可用的儲存集區，請選取 [刪除磁碟上的複本] 核取方塊，以刪除與該成員相關聯的備份資料 (與復原點)。

  ![[從群組中移除] 對話方塊](http://docs.microsoft.com/system-center/dpm/media/upgrade-to-dpm-2016/dpm-2016-retain-data.png)

3. 建立使用新式備份儲存體的保護群組。 納入未受保護的資料來源。


## <a name="add-disks-to-increase-legacy-storage"></a>新增磁碟以增加舊式儲存體

如果您想要在備份伺服器中使用舊式儲存體，您可能需要新增磁碟以增加舊式儲存體。 

若要新增磁碟儲存體：

1. 在 System Center 2016 DPM 管理主控台中選取 [管理] > [磁碟儲存體] > [新增]。

  ![[新增磁碟儲存體] 對話方塊](http://docs.microsoft.com/system-center/dpm/media/upgrade-to-dpm-2016/dpm-2016-add-disk-storage.png)

2. 在 [新增磁碟儲存體] 對話方塊中選取 [新增磁碟]。

3. 在可用磁碟清單中選取您要新增的磁碟。 選取 [新增]，然後選取 [確定]。

## <a name="update-the-data-protection-manager-protection-agent"></a>升級 Data Protection Manager 保護代理程式

備份伺服器會使用 System Center Data Protection Manager 保護代理程式來進行更新。 如果您要升級的保護代理程式未連線到網路，您將無法使用 Data Protection Manager 管理員主控台來完成已連線代理程式的升級。 您必須在非作用中的網域環境升級保護代理程式。 在用戶端電腦連線到網路之前，Data Protection Manager 管理員主控台會顯示保護代理程式更新受到擱置。

下列幾節會說明如何更新已連線用戶端電腦和未連線用戶端電腦的保護代理程式。

### <a name="update-a-protection-agent-for-a-connected-client-computer"></a>更新已連線用戶端電腦的保護代理程式

1. 在備份伺服器管理員主控台中選取 [管理] > [代理程式]。

2. 在顯示窗格中，選取您要更新保護代理程式的用戶端電腦。

  > [!NOTE]
  > [代理程式更新] 資料行會在各個受保護的電腦可以進行保護代理程式更新時予以指出。 在 [動作] 窗格中，只有當您已選取受保護的電腦且其可供更新時，才可使用 [更新] 動作。

3. 若要在選取的電腦上安裝已更新的保護代理程式，請在 [動作] 窗格中選取 [更新]。

### <a name="update-a-protection-agent-on-a-client-computer-that-is-not-connected"></a>在未連線用戶端電腦上更新保護代理程式

1. 在備份伺服器管理員主控台中選取 [管理] > [代理程式]。

2. 在顯示窗格中，選取您要更新保護代理程式的用戶端電腦。

  > [!NOTE]
  > [代理程式更新] 資料行會在各個受保護的電腦可以進行保護代理程式更新時予以指出。 在 [動作] 窗格中，除非已選取的受保護電腦可供更新，否則您無法使用 [更新] 動作。

3. 若要在選取的電腦上安裝已更新的保護代理程式，請選取 [更新]。

4. 對於未連線到網路的用戶端電腦，除非該電腦連線到網路，否則 [代理程式狀態] 資料行會顯示 [更新擱置中] 的狀態。

  當用戶端電腦連線到網路時，用戶端電腦的 [代理程式更新] 資料行會顯示 [更新中] 狀態。
  
### <a name="move-legacy-protection-groups-from-the-old-version-and-sync-the-new-version-with-azure"></a>將舊版保護群組從舊版本中移出，並透過 Azure 同步處理新版本

當 Azure 備份伺服器與 OS 皆更新時，您就可以使用新式備份儲存體來保護新的資料來源。 已受保護的資料來源會繼續和在 Azure 備份伺服器 (舊版) 時一樣受到保護。 新的保護群組全都會使用 Modern Backup Storage。

若要將資料來源從傳統的保護模式遷移到 Modern Backup Storage：

1.  在 Data Protection Manager 儲存體集區中新增磁碟區。 您也可以指派易記名稱，然後選取資料來源標記。

2. 對於傳統模式下的每個資料來源，停止資料來源保護。 然後，選取 [保留受保護的資料]。  這可讓您在移轉之後復原舊的復原點。

3. 建立新的保護群組。 然後，選取您想要使用新格式來儲存的資料來源。

  Data Protection Manager 會在本機的 Modern Backup Storage 磁碟區中，儲存舊版備份儲存體中的複本。
    > [!NOTE] 
    > 建立複本會顯示為復原後操作作業。

  然後，所有新的同步處理和復原點會儲存到 Modern Backup Storage 中。

  舊的復原點則會在過期時剪除。 舊的復原點遭到刪除時，便會釋放磁碟空間。

  當舊有儲存體中刪除了所有舊式磁碟區時，您便可以從 Azure 備份中移除磁碟。 然後，您可以從系統中移除磁碟。

4. 製作 Data Protection Manager 資料庫的備份。

  > [!IMPORTANT]
  > - 新的伺服器名稱必須與原始 Azure 備份伺服器執行個體的名稱相同。 如果您想要使用先前的儲存體集區和 Data Protection Manager 資料庫以保留復原點，就無法變更新 Azure 備份伺服器執行個體的名稱。
  > - 您必須有 Data Protection Manager 資料庫的備份。 您必須還原資料庫。

5. 關閉原始的 Azure 備份伺服器執行個體，或讓伺服器離線。

6. 在 Active Directory 中重設機器帳戶。

7. 在新機器上安裝 Windows Server 2016。 至於伺服器名稱，請使用與原始 Azure 備份伺服器執行個體相同的機器名稱。

8. 加入網域。

9. 安裝 Azure 備份伺服器 v2。 (從舊伺服器移除 Data Protection Manager 儲存體集區磁碟，並將這些磁碟匯入到新的伺服器)。

10. 還原您在步驟 4 中所建立的 Data Protection Manager 資料庫。

11. 將原始備份伺服器的儲存體連結到新伺服器。

12. 在 SQL Server 中，還原 Data Protection Manager 資料庫。

13. 在新伺服器上的系統管理員命令列中，使用 `cd` 移至 Microsoft Azure 備份安裝位置和 bin 資料夾。  

  範例：  
  C:\windows\system32>cd "c:\Program Files\Microsoft Azure Backup\DPM\DPM\bin\ to Azure Backup

14. 執行 `DPMSYNC -SYNC`。
  
  > [!NOTE]
  > 如果您已將「新」磁碟新增至 Data Protection Manager 儲存體集區，而不是移動舊磁碟，則請執行 `DPMSYNC -Reallocatereplica`。

## <a name="new-powershell-cmdlets-in-backup-server-v2"></a>備份伺服器 v2 中的新 PowerShell Cmdlet

當您在安裝 Azure 備份伺服器 v2 時，系統有兩個新的 Cmdlet 可供使用： 
* [Mount-DPMRecoveryPoint](https://technet.microsoft.com/library/mt787159.aspx)
* [Dismount-DPMRecoveryPoint](https://technet.microsoft.com/library/mt787158.aspx)

## <a name="next-steps"></a>後續步驟

了解如何準備您的伺服器或開始保護工作負載：
- [準備備份伺服器工作負載](backup-azure-microsoft-azure-backup.md)
- [使用備份伺服器來備份 VMware 伺服器](backup-azure-backup-server-vmware.md)
- [使用備份伺服器來備份 SQL Server](backup-azure-sql-mabs.md)
- [在備份伺服器中使用新式備份儲存體](backup-mabs-add-storage.md)

