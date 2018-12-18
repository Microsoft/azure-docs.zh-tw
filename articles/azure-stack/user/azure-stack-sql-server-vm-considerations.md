---
title: Azure Stack 虛擬機器中的 SQL Server 效能最佳做法
description: 提供將 Microsoft Azure Stack 虛擬機器中的 SQL Server 效能最佳化的最佳做法。
services: azure-stack
documentationcenter: ''
author: mattbriggs
manager: femila
editor: ''
ms.assetid: ''
ms.service: azure-stack
ms.workload: na
pms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 05/31/2018
ms.author: mabrigg
ms.reviewer: anajod
ms.openlocfilehash: 00c67503f5b9e0027cbb62520e392f56420a75e6
ms.sourcegitcommit: 59fffec8043c3da2fcf31ca5036a55bbd62e519c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/04/2018
ms.locfileid: "34701630"
---
# <a name="optimize-sql-server-performance"></a>將 SQL Server 效能最佳化

本文提供將 Microsoft Azure Stack 虛擬機器中的 SQL Server 效能最佳化的指導方針。 在 Azure Stack 虛擬機器中執行 SQL Server 時，所使用的資料庫效能微調選項，應該和適用於內部部署伺服器環境中 SQL Server 的選項相同。 在 Azure Stack 雲端中，關聯式資料庫的效能取決於諸多因素。 這些因素包括虛擬機器的系列大小，以及資料磁碟的設定。

建立 SQL Server 映像時，[請考慮在 Azure Stack 入口網站中佈建虛擬機器](https://docs.microsoft.com/azure/virtual-machines/windows/sql/virtual-machines-windows-portal-sql-server-provision)。 從 Azure Stack 管理入口網站中的 [Marketplace 管理] 下載 SQL IaaS 擴充功能，然後下載您所選擇的 SQL 虛擬機器虛擬硬碟 (VHD)。 這些項目包括 SQL2014SP2、SQL2016SP1 及 SQL2017。

> [!NOTE]  
> 雖然本文描述的是如何使用全域 Azure 入口網站來佈建 SQL Server 虛擬機器，但文中的指導方針同樣適用於 Azure Stack，但有如下差異：SSD 無法作為作業系統磁碟、未提供受控磁碟，且儲存體設定有些微差異。

本文重心放在如何於 Azure Stack 虛擬機器上獲得「最佳的」SQL Server 效能。 如果工作負載需求不高，則不一定要遵循每個最佳化建議。 評估以下建議時，請考慮您的效能需求和工作負載模式。

> [!NOTE]  
> 如需 Azure 虛擬機器的 SQL Server 效能指導方針，請參閱[本文](https://docs.microsoft.com/azure/virtual-machines/windows/sql/virtual-machines-windows-sql-performance)。

## <a name="before-you-begin"></a>開始之前
下列檢查清單是為了讓您在 Azure Stack 虛擬機器上獲得最佳的 SQL Server 效能：


|領域|最佳化|
|-----|-----|
|虛擬機器大小 |[DS3](https://docs.microsoft.com/azure/azure-stack/user/azure-stack-vm-sizes) 或更高版本 (若為 SQL Server Enterprise Edition)。<br><br>[DS2](https://docs.microsoft.com/azure/azure-stack/user/azure-stack-vm-sizes) 或更高版本 (若為 SQL Server Standard Edition 和 Web Edition)。|
|儲存體 |使用支援[進階儲存體](https://docs.microsoft.com/azure/azure-stack/user/azure-stack-acs-differences)的虛擬機器系列。|
|磁碟 |使用至少兩個資料磁碟 (一個供記錄檔使用，一個供資料檔案和 TempDB 使用)，並根據容量需求選擇磁碟大小。 在 SQL Server 安裝期間，將預設資料檔案位置設定為這些磁碟。<br><br>避免使用作業系統或暫存磁碟作為資料儲存體或進行記錄。<br>使用「儲存空間」，分割多個 Azure 資料磁碟以提高 IO 輸送量。<br><br>以文件上記載的配置大小格式化。|
|I/O|針對資料檔案，啟用 [立即檔案初始化] 功能。<br><br>將資料庫的自動成長限制在合理且固定的小規模增量 (64 MB - 256 MB)。<br><br>停用資料庫上的 [自動壓縮] 功能。<br><br>將預設的備份和資料庫檔案位置設定在資料磁碟上，而非在作業系統磁碟上。<br><br>啟用鎖定的頁面。<br><br>套用 SQL Server Service Pack 和累積更新。|
|功能專屬|直接備份到 Blob 儲存體 (如果所使用的 SQL Server 版本可支援)。|
|||

如需有關「如何」和「為何」進行這些最佳化的詳細資訊，請檢閱下列各節提供的詳細資料與指導方針。

## <a name="virtual-machine-size-guidance"></a>虛擬機器大小指導方針

對於需要高效能的應用程式，建議您採用下列[虛擬機器大小](https://docs.microsoft.com/azure/azure-stack/user/azure-stack-vm-sizes)：

- **SQL Server Enterprise Edition：** DS3 或更高版本

- **SQL Server Standard Edition 和 Web Edition：** DS2 或更高版本

在使用 Azure Stack 時，DS 和 DS_v2 虛擬機器系列的效能沒有差異。

## <a name="storage-guidance"></a>儲存體指引

Azure Stack 中的 DS 系列 (以及 DSv2 系列) 虛擬機器可提供最大的作業系統磁碟和資料磁碟輸送量 (IOPS)。 無論所選磁碟的大小或類型為何，DS 或 DSv2 系列中的虛擬機器，可為作業系統磁碟提供最高 1,000 的 IOPS，並為每個資料磁碟提供最高 2,300 的 IOPS。

資料磁碟輸送量僅取決於虛擬機器系列。 您可以[參閱本文](https://docs.microsoft.com/azure/azure-stack/user/azure-stack-vm-sizes)，來找出每個虛擬機器系列的資料磁碟輸送量。

> [!NOTE]  
> 對於生產工作負載，請選取 DS 系列或 DSv2 系列的虛擬機器，以對作業系統磁碟和資料磁碟提供所能實現的最大 IOPS。

在 Azure Stack 中建立儲存體帳戶時，由於 Azure Stack 未提供異地複寫功能，因此異地複寫選項不會有作用。

## <a name="disks-guidance"></a>磁碟指引

Azure Stack 虛擬機器上有三種主要的磁碟類型︰

- **作業系統磁碟：** 在建立 Azure Stack 虛擬機器時，此平台會至少將一個磁碟 (標示為 **C** 磁碟機) 連結至虛擬機器作為作業系統磁碟。 此磁碟是以分頁 Blob 的形式儲存於儲存體的 VHD。

- **暫存磁碟：** Azure Stack 虛擬機器包含另一個稱為暫存磁碟的磁碟 (標示為 **D** 磁碟機)。 此磁碟位於可用於塗銷空間的節點上。

- **資料磁碟：** 您可以將其他磁碟連結至虛擬機器作為資料磁碟，這些磁碟將會以分頁 Blob 形式儲存於儲存體中。

下列各節說明使用這些不同磁碟的建議。

### <a name="operating-system-disk"></a>作業系統磁碟

作業系統磁碟是指可開機，並掛接為執行的作業系統版本，且標示為 **C** 磁碟機的 VHD。

### <a name="temporary-disk"></a>暫存磁碟

標示為 **D** 磁碟機的暫存磁碟機不會保留下來。 請勿將不願遺失的資料 (例如，使用者資料庫檔案或使用者交易記錄檔) 儲存在 **D** 磁碟機。

由於每個資料磁碟最多可提供高達 2,300 的 IOPS，因此建議您將 TempDB 儲存在資料磁碟上。

### <a name="data-disks"></a>資料磁碟

- **對資料和記錄檔使用資料磁碟。** 如果您未使用磁碟等量分割，請使用 2 個支援進階儲存體的虛擬機器資料磁碟，一個磁碟包含記錄檔，另一個則包含資料和 TempDB 檔案。 每個資料磁碟都會提供數個 IOPS 和頻寬 (MB/s) (取決於虛擬機器系列)，如 [Azure Stack 中支援的虛擬機器大小](https://docs.microsoft.com/azure/azure-stack/user/azure-stack-vm-sizes)一文所述。 如果您使用磁碟等量分割技術 (例如「儲存空間」)，請將所有資料和記錄檔放置在相同磁碟機 (包括 TempDB)。 此設定會讓您有最大的 IOPS 數目供 SQL Server 取用 (不論哪個檔案在哪個特定時間有需要)。

> [!NOTE]  
> 當您在入口網站中佈建 SQL Server 虛擬機器時，您可以選擇編輯儲存體設定。 根據您的設定，Azure Stack 會設定一或多個磁碟。 多個磁碟會合併成單一儲存體集區。 資料和記錄檔皆在此設定中。

- **磁碟等量分割︰** 如需更多的輸送量，您可以新增其他資料磁碟，並使用磁碟等量分割。 若要判斷所需的資料磁碟數目，請分析記錄檔以及資料和 TempDB 檔案所需的 IOPS 數目和頻寬。 請注意，IOPS 的限制是以每個資料磁碟為基礎的 (根據虛擬機器系列)，而不是以虛擬機器大小基礎的。 不過，網路頻寬限制則是以虛擬機器大小為基礎的。 如需詳細資訊，請參閱 [Azure Stack 中的虛擬機器大小](https://docs.microsoft.com/azure/azure-stack/user/azure-stack-vm-sizes)上的資料表。 請使用下列指引：

    - 若為 Windows Server 2012 或更新版本，請使用[儲存空間](https://technet.microsoft.com/library/hh831739.aspx)與下列指導方針：

        1.  將線上交易處理 (OLTP) 工作負載的間隔 (等量磁碟區大小) 設為 64 KB (65,536 位元組)，資料倉儲的工作負載則設為 256 KB (262,144 位元組)，以避免分割對齊錯誤影響效能。 必須使用 PowerShell 來設定。

        2.  設定資料行數目 = 實體磁碟數量。 設定超過 8 個磁碟時請使用 PowerShell (而非伺服器管理員 UI)。

            例如，下列 PowerShell 會建立新的儲存體集區，其間隔大小設定為 64 KB，且資料行數目設定為 2︰

          ```PowerShell  
          $PoolCount = Get-PhysicalDisk -CanPool $True
          $PhysicalDisks = Get-PhysicalDisk | Where-Object {$_.FriendlyName -like "*2" -or $_.FriendlyName -like "*3"}

          New-StoragePool -FriendlyName "DataFiles" -StorageSubsystemFriendlyName "Storage Spaces*" -PhysicalDisks $PhysicalDisks | New-VirtualDisk -FriendlyName "DataFiles" -Interleave 65536 -NumberOfColumns 2 -ResiliencySettingName simple –UseMaximumSize |Initialize-Disk -PartitionStyle GPT -PassThru |New-Partition -AssignDriveLetter -UseMaximumSize |Format-Volume -FileSystem NTFS -NewFileSystemLabel "DataDisks" -AllocationUnitSize 65536 -Confirm:$false
          ```

- 請根據您預期的負載量，決定與您的儲存體集區相關聯的磁碟數量。 請注意，各虛擬機器大小所允許連接的資料磁碟數量皆不同。 如需詳細資訊，請參閱 [Azure Stack 中支援的虛擬機器大小](https://docs.microsoft.com/azure/azure-stack/user/azure-stack-vm-sizes)。
- 為了獲得資料磁碟所能實現的最大 IOPS，建議您新增[虛擬機器大小](https://docs.microsoft.com/azure/azure-stack/user/azure-stack-vm-sizes)所支援的最大資料磁碟數目，並使用磁碟等量分割。
- **NTFS 配置單位大小：** 格式化資料磁碟時，建議您針對資料/記錄檔案和 TempDB，採用 64 KB 的配置單位大小。
- **磁碟管理做法︰** 移除資料磁碟時，請於變更期間停止 SQL Server 服務。 此外，請勿變更磁碟的快取設定，這麼做並不會改善效能。

> [!WARNING]  
> 如果在這些作業進行期間無法停止 SQL Server，可能會造成資料庫損毀。

## <a name="io-guidance"></a>I/O 指引

- 請考慮啟用 [立即檔案初始化] 功能，以減少配置初始檔案所需的時間。 若要發揮「立即檔案初始化」的優點，請將 **SE_MANAGE_VOLUME_NAME** 授與 SQL Server (MSSQLSERVER) 服務帳戶，並將該帳戶加入[執行磁碟區維護工作] 安全性原則。 如果您使用的是 Azure 的 SQL Server 平台映像，預設服務帳戶 (**NT Service\MSSQLSERVER**) 不會加入 [執行磁碟區維護工作] 安全性原則。 也就是說，SQL Server Azure 平台映像未啟用 [立即檔案初始化] 功能。 將 SQL Server 服務帳戶加入 [執行磁碟區維護工作]  安全性原則之後，請重新啟動 SQL Server 服務。 使用此功能時，可能有安全性考量。 如需詳細資訊，請參閱 [資料庫檔案初始化](https://msdn.microsoft.com/library/ms175935.aspx)。
- **自動成長**是在發生非預期成長時的應急方案。 請勿每天使用「自動成長」功能，管理資料和記錄成長。 若已使用自動成長功能，請透過 [大小] 參數預先放大檔案。
- 請確定已停用「自動壓縮」  ，以避免不必要的額外負荷，而對效能造成負面影響。
- 設定預設備份和資料庫檔案位置。 請使用本文中的建議，並在 [伺服器屬性] 視窗中進行變更。 如需指示，請參閱 [檢視或變更資料及記錄檔的預設位置 (SQL Server Management Studio)](https://msdn.microsoft.com/library/dd206993.aspx)。 下列螢幕擷取畫面示範進行這些變更的位置：

    > ![檢視或變更預設位置](./media/sql-server-vm-considerations/image1.png)

- 起用鎖定的頁面，以減少 IO 和任何分頁活動。 如需詳細資訊，請參閱 [啟用鎖定記憶體分頁選項 (Windows)](https://msdn.microsoft.com/library/ms190730.aspx)。

- 將資料檔案傳輸至 Azure Stack，或從 Azure Stack 往外傳輸時，請考慮將資料檔案壓縮。

## <a name="feature-specific-guidance"></a>特定功能的指引

有些部署作業可能會使用更進階的組態技術，提供額外的效能優點。 下列清單特別強調了一些可協助您達到更佳效能的 SQL Server 功能：

- **備份至 Azure** **儲存體。** 對 Azure Stack 虛擬機器中所執行的 SQL Server 執行備份時，可以使用「SQL Server 備份至 URL」。 此功能從 SQL Server 2012 SP1 CU2 開始提供，建議在備份至連接的資料磁碟時使用。

    使用 Azure 儲存體來進行備份或還原時，請遵循 [SQL Server 備份至 URL 的最佳做法和疑難排解](https://msdn.microsoft.com/library/jj919149.aspx)和[從儲存在 Microsoft Azure 中的備份進行還原](https://docs.microsoft.com/en-us/sql/relational-databases/backup-restore/restoring-from-backups-stored-in-microsoft-azure?view=sql-server-2017)中所提供的建議。 您也可以使用 [Azure 虛擬機器中的 SQL Server 自動備份](https://docs.microsoft.com/azure/virtual-machines/windows/sql/virtual-machines-windows-sql-automated-backup)，自動執行這些備份作業。

-   **備份至 Azure Stack 儲存體。** 您可以使用和「備份至 Azure 儲存體」類似的方式，來備份至 Azure Stack 儲存體。 當您在 SQL Server Management Studio (SSMS) 中建立備份時，必須手動輸入設定資訊。 您無法使用 SSMS 來建立儲存體容器或共用存取簽章。 SSMS 僅會連線至 Azure 訂用帳戶，不會連線至 Azure Stack 訂用帳戶。 相反地，您必須在 Azure Stack 入口網站中或使用 PowerShell，才能建立儲存體帳戶、容器和共用存取簽章。

    當您將資訊放入 [SQL Server 備份] 對話方塊時：

    ![SQL Server 備份](./media/sql-server-vm-considerations/image3.png)

    > [!NOTE]  
    > 共用存取簽章是得自 Azure Stack 入口網站的 SAS 權杖，字串中沒有前置 ‘?’。 in the string. 如果您從入口網站使用複製功能，則必須刪除前置 ‘?’ 才能讓權杖在 SQL Server 中生效。 for the token to work within SQL Server.

    在 SQL Server 中安裝和設定了備份目的地後，您就可以備份至 Azure Stack Blob 儲存體。

## <a name="next-steps"></a>後續步驟

[使用服務或為 Azure Stack 建置應用程式](azure-stack-considerations.md)