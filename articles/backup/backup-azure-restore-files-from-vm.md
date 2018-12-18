---
title: Azure 備份︰從 Azure VM 備份復原檔案和資料夾
description: 從 Azure 虛擬機器的復原點復原檔案
services: backup
author: pvrk
manager: shivamg
keywords: 項目層級復原；從 Azure VM 備份檔案復原；從 Azure VM 中還原檔案
ms.service: backup
ms.topic: conceptual
ms.date: 8/22/2018
ms.author: pullabhk
ms.openlocfilehash: 1f3b81c31dc566e5e3011167eee00145f6791cb1
ms.sourcegitcommit: a62cbb539c056fe9fcd5108d0b63487bd149d5c3
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/22/2018
ms.locfileid: "42616904"
---
# <a name="recover-files-from-azure-virtual-machine-backup"></a>從 Azure 虛擬機器備份復原檔案

Azure 備份可從 Azure 虛擬機器 (VM) 備份 (又稱復原點) 還原 [Azure VM 和磁碟](./backup-azure-arm-restore-vms.md)。 本文說明如何從 Azure VM 備份來復原檔案和資料夾。 只有使用 Resource Manager 模型部署且受復原服務保存庫保護的 Azure VM，才能還原檔案和資料夾。

> [!Note]
> 這項功能適用於使用 Resource Manager 模型部署且受復原服務保存庫保護的 Azure VM。
> 不支援從加密 VM 備份的檔案復原。
>

## <a name="mount-the-volume-and-copy-files"></a>掛接磁碟區和複製檔案

若要從復原點還原檔案或資料夾，請移至虛擬機器，然後選擇所需的復原點。

1. 登入 [Azure 入口網站](http://portal.Azure.com)，按一下左窗格中的 [虛擬機器]。 從虛擬機器的清單中，選取虛擬機器以開啟該虛擬機器的儀表板。

2. 在虛擬機器的功能表中，按一下 [備份] 來開啟 [備份] 儀表板。

    ![開啟復原服務保存庫備份項目](./media/backup-azure-restore-files-from-vm/open-vault-for-vm.png)

3. 在 [備份] 儀表板功能表中，按一下 [檔案復原]。

    ![[檔案復原] 按鈕](./media/backup-azure-restore-files-from-vm/vm-backup-menu-file-recovery-button.png)

    [檔案復原] 功能表隨即開啟。

    ![[檔案復原] 功能表](./media/backup-azure-restore-files-from-vm/file-recovery-blade.png)

4. 從 [選取復原點] 下拉式選單中，選取包含您所需檔案的復原點。 根據預設，已選取最近的復原點。

5. 若要下載可從還原點複製檔案的軟體，請按一下 [下載可執行檔] \(適用於 Microsoft Azure VM\) 或 [下載指令碼] \(適用於 Linux Azure VM\)。

    ![產生的密碼](./media/backup-azure-restore-files-from-vm/download-executable.png)

    Azure 會將可執行檔或指令碼下載到本機電腦。

    ![可執行檔或指令碼的下載訊息](./media/backup-azure-restore-files-from-vm/run-the-script.png)

    若要以系統管理員的身分執行可執行檔或指令碼，建議先將下載項目儲存到電腦上。

6. 可執行檔或指令碼受到密碼保護，因此需輸入密碼。 在 [檔案復原] 功能表上，按一下 [複製] 按鈕以將密碼載入記憶體中。

    ![產生的密碼](./media/backup-azure-restore-files-from-vm/generated-pswd.png)

7. 於下載位置中 (通常是 [下載] 資料夾)，在可執行檔或指令碼上按一下滑鼠右鍵，並以系統管理員認證來執行。 當出現提示時，請輸入密碼或從記憶體中貼上密碼，然後按 Enter 鍵。 輸入有效的密碼後，指令碼會連線至復原點。

    ![[檔案復原] 功能表](./media/backup-azure-restore-files-from-vm/executable-output.png)

    如果您在具有限制存取的電腦上執行指令碼，請確定可存取︰

    - download.microsoft.com
    - 復原服務 URL (geo-name 是指復原服務保存庫所在的區域)
        - <https://pod01-rec2.geo-name.backup.windowsazure.com> (適用於 Azure 公用地區)
        - <https://pod01-rec2.geo-name.backup.windowsazure.cn> (適用於 Azure 中國)
        - <https://pod01-rec2.geo-name.backup.windowsazure.us> (適用於 Azure US Government)
        - <https://pod01-rec2.geo-name.backup.windowsazure.de> (適用於 Azure 德國)
    - 輸出連接埠 3260

    若為 Linux，指令碼需要 'open-iscsi' 和 'lshw' 元件來連接到復原點。 如果元件不存在於執行指令碼的電腦上，則指令碼會要求安裝元件的權限。 同意安裝必要的元件。

    需有 download.microsoft.com 的存取權才能下載元件，並用這些元件在執行指令碼的機器及復原點中的資料之間建立安全通道。

    您可以在任何具有與備份 VM 相的 (或相容) 作業系統的電腦上執行指令碼。 請參閱[相容作業系統資料表](backup-azure-restore-files-from-vm.md#system-requirements)以查看相容的作業系統。 如果受保護的 Azure 虛擬機器使用 Windows 儲存空間 (適用於 Windows Azure 虛擬機器) 或 LVM/RAID 陣列 (適用於 Linux 虛擬機器)，則您無法在同一部虛擬機器上執行可執行檔或指令碼。 請改為在其他具有相容作業系統的電腦上執行可執行或指令碼。

### <a name="identifying-volumes"></a>識別磁碟區

#### <a name="for-windows"></a>若為 Windows

當您執行可執行檔時，作業系統會掛接新磁碟區並指派磁碟機代號。 您可以使用 Windows 檔案總管或檔案總管來瀏覽這些磁碟機。 指派給磁碟區的磁碟機代號可能與原始虛擬機器為不同的代號，不過，系統會保留磁碟區名稱。 例如，如果原始虛擬機器上的磁碟區是 "Data Disk (E:`\`)"，則可以在本機電腦上將該磁碟區連結為 "Data Disk ('任何磁碟機代號':`\`)"。 瀏覽指令碼輸出中所提及的所有磁碟區，直到找出您的檔案/資料夾。  

   ![[檔案復原] 功能表](./media/backup-azure-restore-files-from-vm/volumes-attached.png)

#### <a name="for-linux"></a>若為 Linux

在 Linux 中，復原點的磁碟區會掛接到執行指令碼的資料夾。 連結的磁碟、磁碟區和對應的掛接路徑會相應顯示。 具有根層級存取權的使用者都看得到這些掛接路徑。 在指令碼輸出中瀏覽所述的磁碟區。

  ![Linux [檔案復原] 功能表](./media/backup-azure-restore-files-from-vm/linux-mount-paths.png)
  
## <a name="closing-the-connection"></a>關閉連線

在識別檔案並將它們複製到本機儲存體位置之後，移除 (或卸載) 其他磁碟機。 若要卸載磁碟機，請在 Azure 入口網站的 [檔案復原] 功能表中，按一下 [卸載磁碟]。

![取消掛接磁碟](./media/backup-azure-restore-files-from-vm/unmount-disks3.png)

一旦卸載磁碟後，您會收到一則訊息。 連線可能需要幾分鐘重新整理，以讓您可以移除磁碟。

在 Linux 中，復原點的連線嚴重損毀之後，作業系統並不會自動移除對應的掛接路徑。 這些掛接路徑會以「孤立」磁碟區的形式存在。雖然您看得見，但是存取/寫入檔案時會擲回錯誤。 可以手動移除它們。 指令碼執行時，會從任何先前的復原點識別任何這類現有的磁碟區，並在同意下將它們清除。

## <a name="special-configurations"></a>特殊的組態

### <a name="dynamic-disks"></a>動態磁碟

如果受保護 Azure VM 具有下列之一或兩者特性的磁碟區，則您無法在同一部 VM 上執行可執行的指令碼。

    - 跨多個磁碟的磁碟區 (合併或等量磁碟區)
    - 動態磁碟上的容錯磁碟區 (鏡像與 RAID-5 磁碟區)

請改為在其他具有相容作業系統的電腦上執行可執行的指令碼。

### <a name="windows-storage-spaces"></a>Windows 儲存空間

Windows 儲存空間是一種可將儲存體虛擬化的 Windows 技術。 您可以透過 Windows 儲存空間，集合一組符合業界標準的磁碟成為儲存體集區。 然後使用儲存體集區中的可用空間來建立虛擬磁碟 (稱為儲存空間)。

如果受保護 Azure VM 使用 Windows 儲存空間，則無法在同一部虛擬機器上執行可執行的指令碼。 相反地，在任何其他具有相容作業系統上的電腦執行可執行的指令碼。

### <a name="lvmraid-arrays"></a>LVM/RAID 陣列

在 Linux 中，邏輯磁碟區管理員 (LVM) 和/或軟體 RAID 陣列會用來管理多個磁碟的邏輯磁碟區。 如果受保護 Linux VM 使用 LVM 和/或 RAID 陣列，則無法在同一部虛擬機器上執行指令碼。 請改為在其他具有相容作業系統且支援受保護 VM 之檔案系統的電腦上執行指令碼。

下列指令碼輸出會顯示 LVM 和/或 RAID 陣列磁碟與磁碟區及其磁碟分割類型。

   ![Linux LVM 輸出功能表](./media/backup-azure-restore-files-from-vm/linux-LVMOutput.png)

若要讓這些磁碟分割上線，請執行以下小節中的命令。

#### <a name="for-lvm-partitions"></a>若為 LVM 磁碟分割

列出實體磁碟區之下的磁碟區群組名稱。

```bash
#!/bin/bash
$ pvs <volume name as shown above in the script output>
```

列出磁碟區群組中的所有邏輯磁碟區、名稱及其路徑。

```bash
#!/bin/bash
$ lvdisplay <volume-group-name from the pvs command’s results>
```

若要掛接邏輯磁碟區至您所選擇的路徑。

```bash
#!/bin/bash
$ mount <LV path> </mountpath>
```

#### <a name="for-raid-arrays"></a>若為 RAID 陣列

下列命令會顯示所有 RAID 磁碟的詳細資料。

```bash
#!/bin/bash
$ mdadm –detail –scan
```

 相關的 RAID 磁碟將顯示為 `/dev/mdm/<RAID array name in the protected VM>`

如果 RAID 磁碟具有實體磁碟區，請使用掛接命令。

```bash
#!/bin/bash
$ mount [RAID Disk Path] [/mountpath]
```

如果 RAID 磁碟上有設定其他 LVM，則請使用上述適用於 LVM 磁碟分割的程序，但將 RAID 磁碟名稱改為磁碟區名稱

## <a name="system-requirements"></a>系統需求

### <a name="for-windows-os"></a>若為 Windows OS

下表顯示伺服器和電腦作業系統之間的相容性。 復原檔案時，無法將檔案還原至之前或之後的作業系統版本。 例如，您無法將 Windows Server 2016 虛擬機器的檔案還原至 Windows Server 2012 或 Windows 8 電腦。 您可以將 VM 的檔案還原至相同的伺服器作業系統，或相容的用戶端作業系統。

|伺服器作業系統 | 相容的用戶端作業系統  |
| --------------- | ---- |
| Windows Server 2016    | Windows 10 |
| Windows Server 2012 R2 | Windows 8.1 |
| Windows Server 2012    | Windows 8  |
| Windows Server 2008 R2 | Windows 7   |

### <a name="for-linux-os"></a>若為 Linux 作業系統

在 Linux 中，用來還原檔案的電腦作業系統必須支援受保護虛擬機器的檔案系統。 選取要執行指令碼的電腦時，請確認該電腦具有相容的作業系統，且使用下表中列出的其中一個版本：

|Linux 作業系統 | 版本  |
| --------------- | ---- |
| Ubuntu | 12.04 和更新版本 |
| CentOS | 6.5 和更新版本  |
| RHEL | 6.7 和更新版本 |
| Debian | 7 和更新版本 |
| Oracle Linux | 6.4 和更新版本 |
| SLES | 12 以上 (含) |
| openSUSE | 42.2 和更新版本 |

指令碼也需要 Python 和 Bash 元件，才能夠執行並安全地連線至復原點。

|元件 | 版本  |
| --------------- | ---- |
| Bash | 4 和更新版本 |
| Python | 2.6.6 和更新版本  |
| TLS | 應支援 1.2  |

## <a name="troubleshooting"></a>疑難排解

如果您從虛擬機器復原檔案時遇到問題，請檢查下表中的其他資訊。

| 錯誤訊息 / 案例 | 可能的原因 | 建議的動作 |
| ------------------------ | -------------- | ------------------ |
| Exe 輸出︰連線到目標的例外狀況 |指令碼無法存取復原點    | 檢查電腦是否符合上述的存取需求。 |  
| Exe 輸出︰目標已透過 iSCSI 工作階段登入。 | 指令碼已在相同電腦上執行，並已附加磁碟機 | 復原點磁碟區已連接。 它們可能未使用原始 VM 的相同磁碟機代號裝載。 在檔案總管中的檔案瀏覽所有可用的磁碟區 |
| Exe 輸出︰這個指令碼無效，因為磁碟已透過入口網站/超過 12 小時限制卸載。*從入口網站下載新的指令碼。* |    已從入口網站卸載磁碟或超過 12 小時的限制 | 此特定 exe 目前無效，無法執行。 如果您想要存取該復原點時間的檔案，請瀏覽新 exe 的入口網站|
| 在執行 exe 的電腦上︰按一下 [卸載] 按鈕之後不會卸載新的磁碟區 | 電腦上的 iSCSI 啟動器沒有回應/重新整理與目標的連線並維護快取。 |  按一下 [卸載] 後，請稍候幾分鐘。 如果新磁碟區並未卸載，請瀏覽所有磁碟區。 瀏覽所有磁碟區會強制啟動器重新整理連線，且會卸載磁碟區，並出現磁碟無法使用的錯誤訊息。|
| Exe 輸出︰指令碼成功執行，但指令碼輸出上不會顯示「附加的新磁碟區」 |    這是暫時性的錯誤    | 磁碟區可能已經附加。 開啟檔案總管以瀏覽。 如果您每次都使用同一部電腦執行指令碼，請考慮重新啟動電腦，清單應該會顯示在後續的 exe 執行。 |
| Linux 特定︰無法檢視所需的磁碟區 | 執行指令碼所在電腦的作業系統可能無法辨識受保護 VM 的底層檔案系統 | 檢查復原點是否損毀一致還是檔案一致。 如果檔案一致，請在作業系統可辨識受保護 VM 檔案系統的其他電腦上執行指令碼 |
| Windows 特定︰無法檢視所需的磁碟區 | 已連接磁碟，但未設定磁碟區 | 從 [磁碟管理] 畫面上，找出與復原點相關的其他磁碟。 如果有任何這些磁碟處於離線狀態，請透過在磁碟上按一下滑鼠右鍵並按一下 [上線]，來嘗試使狀態成為上線|
