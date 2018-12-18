---
title: Azure 中的 SQL Server 效能指導方針 | Microsoft Docs
description: 提供將 Microsoft Azure VM 中 SQL Server 效能最佳化的指導方針。
services: virtual-machines-windows
documentationcenter: na
author: rothja
manager: craigg
editor: ''
tags: azure-service-management
ms.assetid: a0c85092-2113-4982-b73a-4e80160bac36
ms.service: virtual-machines-sql
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: vm-windows-sql-server
ms.workload: iaas-sql-server
ms.date: 09/26/2018
ms.author: jroth
ms.openlocfilehash: 0119c6642d68db6a90af07395882e620b1af08c6
ms.sourcegitcommit: d1aef670b97061507dc1343450211a2042b01641
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/27/2018
ms.locfileid: "47394944"
---
# <a name="performance-guidelines-for-sql-server-in-azure-virtual-machines"></a>Azure 虛擬機器中的 SQL Server 效能指導方針

## <a name="overview"></a>概觀

本文提供在「Microsoft Azure 虛擬機器」中最佳化 SQL Server 效能的指引。 在 Azure 虛擬機器中執行 SQL Server 時，我們建議您繼續使用相同的資料庫效能微調選項，這些選項適用於內部部署伺服器環境中的 SQL Server。 不過，公用雲端中關聯式資料庫的效能優劣取決於許多因素，例如虛擬機器的大小和資料磁碟的組態。

[在 Azure 入口網站中佈建的 SQL Server 映像](quickstart-sql-vm-create-portal.md)會遵循存一般的儲存體組態最佳做法 (如需關於如何設定儲存體的詳細資訊，請參閱 [SQL Server VM 的儲存體組態](virtual-machines-windows-sql-server-storage-configuration.md))。 佈建之後，請考慮套用本文所討論的其他最佳化作業。 依據您的工作負載做選擇，並透過測試進行確認。

> [!TIP]
> 您通常必須在最佳化成本與最佳化效能之間做出取捨。 本文的主題為如何讓 Azure VM 上的 SQL Server 達到最佳  效能。 如果您的工作負載需求較低，可能就不需要採用下列每一項最佳化條件。 評估以下建議時，請考量您的效能需求、成本和工作負載模式。

## <a name="quick-check-list"></a>快速檢查清單

下列快速檢查清單能協助您讓 Azure 虛擬機器上的 SQL Server 達到最佳效能：

| 領域 | 最佳化 |
| --- | --- |
| [VM 大小](#vm-size-guidance) |SQL Enterprise 版的 [DS3_v2](../sizes-general.md) 或更高版本。<br/><br/>SQL Standard 和 Web 版的 [DS2_v2](../sizes-general.md) 或更高版本。 |
| [儲存體](#storage-guidance) |使用[進階儲存體](../premium-storage.md)。 標準儲存體只建議用於開發/測試。<br/><br/>將[儲存體帳戶](../../../storage/common/storage-create-storage-account.md)和 SQL Server VM 置於同一個區域。<br/><br/>停用儲存體帳戶上的 Azure [異地備援儲存體](../../../storage/common/storage-redundancy.md) (異地複寫)。 |
| [磁碟](#disks-guidance) |使用至少 2 個 [P30 磁碟](../premium-storage.md#scalability-and-performance-targets) (1 個用於記錄檔，另 1 個用於資料檔案，包括 TempDB)。<br/><br/>避免使用作業系統或暫存磁碟作為資料儲存體或進行記錄。<br/><br/>啟用裝載資料檔案和 TempDB 資料檔案的磁碟上的 [讀取快取] 功能。<br/><br/>不要啟用裝載記錄檔案的磁碟上的 [快取] 功能。<br/><br/>重要事項︰變更 Azure VM 磁碟的快取設定時，停止 SQL Server 服務。<br/><br/>分割多個 Azure 資料磁碟，以提高 IO 輸送量。<br/><br/>以文件上記載的配置大小格式化。 |
| [I/O](#io-guidance) |啟用 [資料庫頁面壓縮] 功能　。<br/><br/>針對資料檔案，啟用 [立即檔案初始化] 功能。<br/><br/>限制資料庫上的 [自動成長] 功能。<br/><br/>停用資料庫上的 [自動壓縮] 功能。<br/><br/>將所有的資料庫 (包括系統資料庫) 移到資料磁碟。<br/><br/>將 SQL Server 的錯誤記錄檔和追蹤檔案目錄移至資料磁碟。<br/><br/>設定預設備份和資料庫檔案位置。<br/><br/>啟用鎖定的頁面。<br/><br/>套用 SQL Server 效能修正程式。 |
| [特定功能](#feature-specific-guidance) |直接備份至 Blob 儲存體。 |

如需有關「如何」和「為何」進行這些最佳化的詳細資訊，請檢閱下列各節提供的詳細資料與指引。

## <a name="vm-size-guidance"></a>VM 大小指引

對於需要高效能的應用程式，建議您採用下列 [虛擬機器大小](../sizes.md)：

* **SQL Server Enterprise 版：** DS3_v2 或更高版本
* **SQL Server Standard 和 Web 版**：DS2_v2 或更高版本

[DSv2 系列](../sizes-general.md#dsv2-series)的 VM 支援進階儲存體，是獲得最佳效能的建議選擇。 此處建議的大小只是基準，您選擇的實際機器大小仍以您的工作負載需求為準。 DSv2 系列的 VM 是一般目的 VM，適合用於各種工作負載，其他機器大小則是專為特定工作負載類型而經過最佳化。 舉例來說，[M 系列](../sizes-memory.md#m-series)提供最高的 vCPU 計數及記憶體，適用於最大型的 SQL Server 工作負載。 [GS 系列](../sizes-memory.md#gs-series)和 [DSv2 系列 11-15](../sizes-memory.md#dsv2-series-11-15) 則為大型記憶體需求經過最佳化。 這兩種系列都可以[有限的核心大小](../../windows/constrained-vcpu.md)提供，為計算需求較低的工作負載節省花費。 [Ls 系列](../sizes-storage.md)機器為高磁碟輸送量與 IO 經過最佳化。 請務必考量您的特定 SQL Server 工作負載，並應用在您的 VM 系列與大小選項。

## <a name="storage-guidance"></a>儲存體指引

DS 系列 (以及 DSv2 系列和 GS 系列) VM 支援 [進階儲存體](../premium-storage.md)。 針對所有生產環境工作負載，建議使用進階儲存體。

> [!WARNING]
> 標準儲存體具有不同的延遲和頻寬，並建議僅用於開發/測試工作負載。 這包括新的標準 SSD 儲存體。 生產工作負載應該使用進階儲存體。

此外，我們建議您在存放 SQL Server 虛擬機器的同一個資料中心內建立 Azure 儲存體帳戶，以減少傳輸延遲的狀況。 建立儲存體帳戶時，請停用 [異地複寫] 功能，因為跨多個磁碟時無法保證寫入順序一致。 請考慮另一個方法，在兩個 Azure 資料中心之間設定 SQL Server 災害復原技術。 如需詳細資訊，請參閱 [Azure 虛擬機器中的 SQL Server 高可用性和災害復原](virtual-machines-windows-sql-high-availability-dr.md)。

## <a name="disks-guidance"></a>磁碟指引

Azure VM 上有三種主要的磁碟類型︰

* **OS 磁碟**：建立「Azure 虛擬機器」時，平台會至少將一個磁碟 (標示為 **C** 磁碟機) 連接至 VM 作為您的作業系統磁碟。 此磁碟是以分頁 Blob 的形式儲存於儲存體的 VHD。
* **暫存磁碟**：「Azure 虛擬機器」包含另一個稱為暫存磁碟的磁碟 (標示為 **D**: 磁碟機)。 此磁碟位於可用於塗銷空間的節點上。
* **資料磁碟**：您也可以將其他磁碟連接至虛擬機器作為資料磁碟，這些磁碟將會以分頁 Blob 形式儲存於儲存體中。

下列各節說明使用這些不同磁碟的建議。

### <a name="operating-system-disk"></a>作業系統磁碟

作業系統磁碟是指可開機，並掛接為執行的作業系統版本，且標示為 **C** 磁碟機的 VHD。

作業系統磁碟上的預設快取原則是 **讀取/寫入**。 對於需要高效能的應用程式，我們建議您使用資料磁碟取代作業系統磁碟。 請參閱下面的＜資料磁碟＞一節。

### <a name="temporary-disk"></a>暫存磁碟

標示為 **D**: 磁碟機的暫存磁碟機不會保存至 Azure blob 儲存體。 請勿將使用者資料庫檔案或使用者交易記錄檔儲存在 **D**: 磁碟機。

D 系列、Dv2 系列和 G 系列 VM 的暫存磁碟機皆為 SSD 式。 如果您的工作負載大量使用 TempDB (例如用於暫存物件或複雜的聯結)，將 TempDB 儲存在 **D** 磁碟機可能會產生較高的 TempDB 輸送量和較低的 TempDB 延遲。 如需範例案例，請參閱下列部落格文章中的 TempDB 討論：[Azure VM 上的 SQL Server 適用的儲存體組態指導方針](https://blogs.msdn.microsoft.com/sqlserverstorageengine/2018/09/25/storage-configuration-guidelines-for-sql-server-on-azure-vm/)。

對於支援「進階儲存體」的 VM (DS 系列、DSv2 系列與 GS 系列)，建議您將 TempDB 儲存在支援「進階儲存體」且啟用讀取快取的磁碟上。 這項建議有一個例外，如果 TempDB 使用方式是密集寫入，您可以將 TempDB 儲存在本機的 **D** 磁碟機 (在這些機器大小上也是 SSD 型) 上以達到更高的效能。

### <a name="data-disks"></a>資料磁碟

* **將資料磁碟用於資料檔和記錄檔**：如果您未使用磁碟等量分割，請使用 2 個進階儲存體 [P30 磁碟](../premium-storage.md#scalability-and-performance-targets)，一個磁碟包含記錄檔，另一個則包含資料和 TempDB 檔案。 每個進階儲存體磁碟會根據其大小提供數個 IOPS 和頻寬 (MB/s)，如[針對磁碟使用進階儲存體](../premium-storage.md)一文所述。 如果您使用磁碟等量化技術 (例如儲存空間)，您將會有兩個集區 (一個用於記錄檔，另一個用於資料檔案)，而達到最佳效能。 不過，如果您打算使用 SQL Server 容錯移轉叢集執行個體 (FCI)，則必須設定一個集區。

   > [!TIP]
   > 如需不同磁碟和工作負載組態的測試結果，請參閱下列部落格文章：[Azure VM 上的 SQL Server 適用的儲存體組態指導方針](https://blogs.msdn.microsoft.com/sqlserverstorageengine/2018/09/25/storage-configuration-guidelines-for-sql-server-on-azure-vm/)。

   > [!NOTE]
   > 當您在入口網站中佈建 SQL Server VM 時，您可以選擇編輯儲存體組態。 根據您的設定，Azure 會設定一或多個磁碟。 多個磁碟會利用串接合併成單一儲存體集區。 資料和記錄檔皆在此設定中。 如需詳細資訊，請參閱 [SQL Server VM 的儲存體組態](virtual-machines-windows-sql-server-storage-configuration.md)。

* **磁碟等量分割**︰如需更多的輸送量，您可以新增其他資料磁碟，並使用「磁碟等量分割」。 為了判斷資料磁碟的數目，您需要分析記錄檔以及資料和 TempDB 檔案所需的 IOPS 和頻寬數目。 請注意，不同的 VM 大小在支援的 IOPS 和頻寬數目上有不同的限制，請參閱每個 [VM 大小](../sizes.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json)的 IOPS 相關表格。 請使用下列指引：

  * 若為 Windows 8/Windows Server 2012 以上版本，請遵循下列指導方針使用[儲存空間](https://technet.microsoft.com/library/hh831739.aspx)：

      1. 將 OLTP 工作負載的間隔 (等量磁碟區大小) 設為 64 KB (65536 位元組)，資料倉儲的工作負載則設為 256 KB (262144 位元組)，以避免分割對齊錯誤影響效能。 必須使用 PowerShell 來設定。
      2. 設定資料行數目 = 實體磁碟數量。 設定 8 個以上的磁碟 (不是伺服器管理員 UI) 時，使用 PowerShell。 

    例如，下列 PowerShell 會建立新的儲存體集區，其間隔大小為 64 KB，且資料行數目為 2︰

    ```powershell
    $PoolCount = Get-PhysicalDisk -CanPool $True
    $PhysicalDisks = Get-PhysicalDisk | Where-Object {$_.FriendlyName -like "*2" -or $_.FriendlyName -like "*3"}

    New-StoragePool -FriendlyName "DataFiles" -StorageSubsystemFriendlyName "Storage Spaces*" -PhysicalDisks $PhysicalDisks | New-VirtualDisk -FriendlyName "DataFiles" -Interleave 65536 -NumberOfColumns 2 -ResiliencySettingName simple –UseMaximumSize |Initialize-Disk -PartitionStyle GPT -PassThru |New-Partition -AssignDriveLetter -UseMaximumSize |Format-Volume -FileSystem NTFS -NewFileSystemLabel "DataDisks" -AllocationUnitSize 65536 -Confirm:$false 
    ```

  * 對於 Windows 2008 R2 之前的版本，可以使用動態磁碟 (OS 分割的磁碟區)，且等量磁碟區的大小一律為 64 KB。 請注意，Windows 8/Windows Server 2012 已不再提供此選項。 如需相關資訊，請參閱 [虛擬磁碟服務正轉換為 Windows 存放管理 API](https://msdn.microsoft.com/library/windows/desktop/hh848071.aspx)中的支援聲明。

  * 如果您要搭配使用[儲存空間直接存取 (S2D)](/windows-server/storage/storage-spaces/storage-spaces-direct-in-vm) 與 [SQL Server 容錯移轉叢集執行個體](virtual-machines-windows-portal-sql-create-failover-cluster.md)，您必須設定單一集區。 請注意，雖然可以在單一集區上建立不同的磁碟區，但這些磁碟區將共用相同特性，例如相同的快取原則。

  * 請根據您預期的負載量，決定與您的儲存體集區相關聯的磁碟數量。 請注意，各 VM 大小所允許連接的資料磁碟數量皆不同。 如需相關資訊，請參閱[虛擬機器的大小](../sizes.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json)。

  * 如果您不是使用「進階儲存體」(開發/測試案例)，建議您新增您 [VM 大小](../sizes.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json) 所支援的最大數目資料磁碟，並使用「磁碟等量分割」。

* **快取原則**：請注意，下列快取原則皆是根據您的儲存體組態來建議。

  * 如果您針對資料和記錄檔使用不同磁碟，請在裝載資料檔和 TempDB 資料檔案的資料磁碟上啟用讀取快取。 這會帶來明顯的效能優勢。 請勿在裝載記錄檔的磁碟上啟用快取，這樣會導致效能小幅降低。

  * 如果您要在單一儲存體集區中使用磁碟等量化，大部分的工作負載將會從讀取快取中獲益。 如果您為記錄檔和資料檔案使用個別的儲存體集區，請僅在資料檔的儲存體集區上啟用讀取快取。 針對某些大量寫入的工作負載，沒有快取可能會達到更好的效能。 這只能透過測試來判斷。

  * 上述建議適用於進階儲存體磁碟。 如果您並非使用進階儲存體，請勿啟用任何資料磁碟上的任何快取功能。

  * 如需有關設定磁碟快取功能的指示，請參閱下列文章。 對於傳統 (ASM) 部署模型，請參閱：[Set-azureosdisk](https://msdn.microsoft.com/library/azure/jj152847) 和 [Set-azuredatadisk](https://msdn.microsoft.com/library/azure/jj152851.aspx)。 對於 Azure Resource Manager 部署模型，請參閱：[Set-AzureRMOSDisk](https://docs.microsoft.com/powershell/module/azurerm.compute/set-azurermvmosdisk?view=azurermps-4.4.1) 和 [Set-AzureRMVMDataDisk](https://docs.microsoft.com/powershell/module/azurerm.compute/set-azurermvmdatadisk?view=azurermps-4.4.1)。

     > [!WARNING]
     > 變更 Azure VM 磁碟的快取設定時，請停止 SQL Server 服務，以避免任何發生資料庫損毀的可能性。

* **NTFS 配置單位大小**：格式化資料磁碟時，建議您針對資料/記錄檔案和 TempDB，採用 64 KB 的配置單位大小。

* **磁碟管理最佳做法**︰移除資料磁碟或變更其快取類型時，請於變更期間停止 SQL Server 服務。 當 OS 磁碟上的快取設定發生變更時，Azure 會停止 VM、變更快取類型，然後重新啟動 VM。 當資料磁碟的快取設定發生變更時，系統不會停止 VM，但在變更期間會從 VM 卸離資料磁碟，然後再重新連接。

  > [!WARNING]
  > 如果在這些作業期間無法停止 SQL Server 服務，可能就會造成資料庫損毀。

## <a name="io-guidance"></a>I/O 指引

* 平行處理應用程式和要求時，進階儲存體可以達到最佳效能。 進階儲存體是專為 IO 佇列深度大於 1 的案例所設計，所以您會發現單一執行緒的序列要求 (即使其為儲存密集型) 的效能只有微幅提升，或沒有提升。 舉例來說，這可能會影響效能分析工具 (如 SQLIO) 的單一執行緒測試結果。

* 請考慮使用 [資料庫頁面壓縮](https://msdn.microsoft.com/library/cc280449.aspx) 功能，其有助於改善 I/O 密集型工作負載的效能。 不過，資料壓縮可能會增加資料庫伺服器的 CPU 使用量。

* 請考慮啟用 [立即檔案初始化] 功能，以減少配置初始檔案所需的時間。 若要發揮「立即檔案初始化」的優點，請將 SE_MANAGE_VOLUME_NAME 授與 SQL Server (MSSQLSERVER) 服務帳戶，並將該帳戶加入[執行磁碟區維護工作] 安全性原則。 如果您使用的是 Azure 的 SQL Server 平台映像，預設服務帳戶 (NT Service\MSSQLSERVER) 不會加入 [執行磁碟區維護工作] 安全性原則。 也就是說，SQL Server Azure 平台映像未啟用 [立即檔案初始化] 功能。 將 SQL Server 服務帳戶加入 [執行磁碟區維護工作]  安全性原則之後，請重新啟動 SQL Server 服務。 使用此功能時，可能有安全性考量。 如需詳細資訊，請參閱 [資料庫檔案初始化](https://msdn.microsoft.com/library/ms175935.aspx)。

*  只是發生非預期成長的應變方案。 請勿每天使用「自動成長」功能，管理資料和記錄成長。 若已使用「自動成長」功能，請透過 大小參數預先放大檔案。

* 請確定已停用「自動壓縮」  ，以避免不必要的額外負荷，而對效能造成負面影響。

* 將所有的資料庫 (包括系統資料庫) 移到資料磁碟。 如需詳細資訊，請參閱 [移動系統資料庫](https://msdn.microsoft.com/library/ms345408.aspx)。

* 將 SQL Server 的錯誤記錄檔和追蹤檔案目錄移至資料磁碟。 在 SQL Server 組態管理員中，以滑鼠右鍵按一下您的 SQL Server 執行個體並選取內容，即可完成。 您可以在 [啟動參數]  索引標籤中變更錯誤記錄和追蹤檔案設定。若要指定「傾印目錄」，則是在 [進階]  索引標籤中指定。下列螢幕擷取畫面顯示錯誤記錄啟動參數的位置。

    ![SQL ErrorLog 螢幕擷取畫面](./media/virtual-machines-windows-sql-performance/sql_server_error_log_location.png)

* 設定預設備份和資料庫檔案位置。 請使用本文中的建議，並在 [伺服器屬性] 視窗中進行變更。 如需指示，請參閱 [檢視或變更資料及記錄檔的預設位置 (SQL Server Management Studio)](https://msdn.microsoft.com/library/dd206993.aspx)。 下列螢幕擷取畫面示範進行這些變更的位置。

    ![SQL 資料記錄和備份檔案](./media/virtual-machines-windows-sql-performance/sql_server_default_data_log_backup_locations.png)
* 起用鎖定的頁面，以減少 IO 和任何分頁活動。 如需詳細資訊，請參閱 [啟用鎖定記憶體分頁選項 (Windows)](https://msdn.microsoft.com/library/ms190730.aspx)。

* 如果您執行的是 SQL Server 2012，請安裝 Service Pack 1 累計更新 10。 此更新包含一個修正程式，可修正在 SQL Server 2012 中執行 select into 暫存資料表陳述式時，I/O 效能不佳的狀況。 如需相關資訊，請參閱此 [知識庫文章](http://support.microsoft.com/kb/2958012)。

* 將資料檔案傳輸至 Azure，或從 Azure 往外傳輸時，請考慮先壓縮所有資料檔案。

## <a name="feature-specific-guidance"></a>特定功能的指引

有些部署作業可能會使用更進階的組態技術，提供額外的效能優點。 下列清單特別強調了一些可協助您達到更佳效能的 SQL Server 功能：

* **備份至 Azure 儲存體**：備份在 Azure 虛擬機器中執行的 SQL Server 時，可以使用 [SQL Server 備份至 URL](https://msdn.microsoft.com/library/dn435916.aspx)。 此功能從 SQL Server 2012 SP1 CU2 開始提供，建議在備份至連接的資料磁碟時使用。 在備份至 Azure 儲存體 (或從中還原) 時，請依照 [SQL Server 備份至 URL 的最佳作法和疑難排解，以及從儲存在 Azure 儲存體中的備份還原](https://msdn.microsoft.com/library/jj919149.aspx)中提供的建議執行。 您也可以使用 [Azure 虛擬機器中的 SQL Server 自動備份](virtual-machines-windows-sql-automated-backup.md)，自動執行這些備份作業。

    對於 SQL Server 2012 之前的版本，您可以使用 [將 SQL Server 備份至 Azure 工具](https://www.microsoft.com/download/details.aspx?id=40740)。 這項工具可以透過多個備份等量磁碟區目標協助您提高備份的輸送量。

* **Azure 中的 SQL Server 資料檔案**：從 SQL Server 2014 開始提供 [Azure 中的 SQL Server 資料檔](https://msdn.microsoft.com/library/dn385720.aspx)這項新功能。 在 Azure 中執行具有資料檔案的 SQL Server 的效能與使用 Azure 資料磁碟的效能特性相當。

## <a name="next-steps"></a>後續步驟

如需儲存體和效能的詳細資訊，請參閱 [Azure VM 上的 SQL Server 適用的儲存體組態指導方針](https://blogs.msdn.microsoft.com/sqlserverstorageengine/2018/09/25/storage-configuration-guidelines-for-sql-server-on-azure-vm/)

如需安全性的最佳作法，請參閱 [Azure 虛擬機器中的 SQL Server 安全性考量](virtual-machines-windows-sql-security.md)。

請檢閱 [Azure 虛擬機器上的 SQL Server 概觀](virtual-machines-windows-sql-server-iaas-overview.md)中其他「SQL Server 虛擬機器」的相關文章。 如果您有 SQL Server 虛擬機器的相關問題，請參閱[常見問題集](virtual-machines-windows-sql-server-iaas-faq.md)。
