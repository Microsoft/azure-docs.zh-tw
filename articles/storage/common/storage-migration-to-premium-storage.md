---
title: 將 VM 移轉至 Azure 進階儲存體 | Microsoft Docs
description: 將您現有的 VM 移轉至 Azure 進階儲存體。 「進階儲存體」可針對在「Azure 虛擬機器」上執行且需要大量 I/O 的工作負載，提供高效能、低延遲的磁碟支援。
services: storage
author: yuemlu
ms.service: storage
ms.topic: article
ms.date: 06/27/2017
ms.author: yuemlu
ms.component: common
ms.openlocfilehash: c6256fc209a4ffa5308dc3b24794f8295c57f4ef
ms.sourcegitcommit: 9819e9782be4a943534829d5b77cf60dea4290a2
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/06/2018
ms.locfileid: "39521773"
---
# <a name="migrating-to-azure-premium-storage-unmanaged-disks"></a>移轉至 Azure 進階儲存體 (非受控磁碟)

> [!NOTE]
> 本文討論如何將使用非受控標準磁碟的 VM，移轉至使用非受控進階磁碟的 VM。 建議您對新 VM 使用 Azure 受控磁碟，並將先前的非受控磁碟轉換成受控磁碟。 受控磁碟會處理基礎儲存體帳戶，您不需要再處理。 如需詳細資訊，請參閱[受控磁碟概觀](../../virtual-machines/windows/managed-disks-overview.md)。
>

針對執行時需要大量 I/O 之工作負載的虛擬機器，「Azure 進階儲存體」可提供高效能、低延遲的磁碟支援。 您可以將應用程式的 VM 磁碟移轉到「Azure 進階儲存體」，以利用這些磁碟的速度和效能。

本指南的目的，在於協助「Azure 進階儲存體」的新使用者做好準備，以便從他們目前的系統順利轉換到「進階儲存體」。 本指南說明此程序中的三個重要元件︰

* [規劃移轉至進階儲存體](#plan-the-migration-to-premium-storage)
* [準備並將虛擬硬碟 (VHD) 複製到進階儲存體](#prepare-and-copy-virtual-hard-disks-VHDs-to-premium-storage)
* [使用進階儲存體來建立 Azure 虛擬機器](#create-azure-virtual-machine-using-premium-storage)

您可以將 VM 從其他平台移轉到 Azure 進階儲存體，或將現有的 Azure VM 從標準儲存體移轉到進階儲存體。 本指南涵蓋這兩種案例的步驟。 請根據您的案例，依照相關小節中所指定的步驟操作。

> [!NOTE]
> 如需進階儲存體的功能概觀和價格，請參閱[進階儲存體：Azure 虛擬機器工作負載適用的高效能儲存體](../../virtual-machines/windows/premium-storage.md)。 建議您將任何需要高 IOPS 的虛擬機器磁碟移轉到「Azure 進階儲存體」，以發揮應用程式最佳效能。 如果您的磁碟不需要高 IOPS，您可以在「標準儲存體」中維護它來限制成本，這會將虛擬機器磁碟資料儲存在「硬碟機 (HDD)」上而非 SSD 上。
>

要完整地完成移轉程序，可能需要在本指南中提供的步驟之前和之後執行其他動作。 例如，設定虛擬網路或端點，或是在應用程式本身中進行程式碼變更，應用程式可能都需要一些停機時間。 這些動作會隨著每個應用程式而不同，您應完成這些動作以及本指南中所提供的步驟，才能盡可能順暢地完整轉換至進階儲存體。

## <a name="plan-the-migration-to-premium-storage"></a>規劃移轉至進階儲存體
這一節確保您準備好遵循本文中的移轉步驟，並協助您對於 VM 和磁碟類型做出最佳的決策。

### <a name="prerequisites"></a>必要條件
* 您將需要 Azure 訂用帳戶。 如果您沒有訂用帳戶，可以建立一個月的[免費試用](https://azure.microsoft.com/pricing/free-trial/)訂用帳戶，或造訪 [Azure 價格](https://azure.microsoft.com/pricing/)以了解其他選項。
* 若要執行 PowerShell Cmdlet，您將需要 Microsoft Azure PowerShell 模組。 如需安裝點和安裝指示的詳細資訊，請參閱 [如何安裝和設定 Azure PowerShell](/powershell/azure/overview) 。
* 當您計劃使用在進階儲存體上執行的 Azure VM 時，您需要使用可支援進階儲存體的 VM。 您可以將「標準」和「進階」儲存體磁碟與支援「進階儲存體」的 VM 搭配使用。 未來進階儲存體磁碟將可搭配更多 VM 類型使用。 如需所有可用 Azure VM 磁碟類型和大小的詳細資訊，請參閱[虛擬機器的大小](../../virtual-machines/windows/sizes.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json)和[雲端服務的大小](../../cloud-services/cloud-services-sizes-specs.md)。

### <a name="considerations"></a>考量
Azure VM 支援連接數個「進階儲存體」磁碟，讓您應用程式的每一 VM 最多可擁有 256 TB 的儲存體。 使用「進階儲存體」時，您應用程式的每一 VM 可達到 80,000 IOPS (每秒輸入/輸出作業)，而每一 VM 的每秒磁碟輸送量為 2000 MB，且讀取作業的延遲極低。 您有各種 VM 和磁碟的選項。 這一節協助您尋找最適合您工作負載的選項。

#### <a name="vm-sizes"></a>VM 大小
Azure VM 大小的規格已列在 [虛擬機器的大小](../../virtual-machines/windows/sizes.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json)一文中。 請檢閱使用於進階儲存體的虛擬機器效能特性，然後選擇最適合您的工作負載的 VM 大小。 確定 VM 上有足夠的磁碟流量頻寬。

#### <a name="disk-sizes"></a>磁碟大小
有五種類型的磁碟可以搭配您的 VM 使用，而且每種都有特定的 IOP 和輸送量限制。 為您的 VM 選擇磁碟類型時，請根據應用程式在容量、效能、延展性以及尖峰負載方面的需求，將這些限制考量進去。

| 進階磁碟類型  | P10   | P20   | P30            | P40            | P50            | 
|:-------------------:|:-----:|:-----:|:--------------:|:--------------:|:--------------:|
| 磁碟大小           | 128 GB| 512 GB| 1024 GB (1 TB) | 2048 GB (2 TB) | 4095 GB (4 TB) | 
| 每一磁碟的 IOPS       | 500   | 2300  | 5000           | 7500           | 7500           | 
| 每一磁碟的輸送量 | 每秒 100 MB | 每秒 150 MB | 每秒 200 MB | 每秒 250 MB | 每秒 250 MB |

根據您的工作負載，決定您的 VM 是否需要額外的資料磁碟。 您可以將數個持續性資料磁碟連接至您的 VM。 如有需要，您可以跨磁碟等量磁碟區以增加磁碟區的容量和效能。 (請參閱[這裡](../../virtual-machines/windows/premium-storage-performance.md#disk-striping)的磁碟等量化說明。)如果您使用[儲存空間][4]等量進階儲存體資料磁碟，應該對所使用的每個磁碟中的每個資料行進行設定。 否則，等量磁碟區的整體效能可能會因為磁碟之間的流量分配不平均而低於預期。 而對於 Linux VM，您可以使用 *mdadm* 公用程式來達到相同的效果。 如需詳細資訊，請參閱文章 [在 Linux 上設定軟體 RAID](../../virtual-machines/linux/configure-raid.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json) 。

#### <a name="storage-account-scalability-targets"></a>儲存體帳戶延展性目標
進階儲存體帳戶除了 [Azure 儲存體延展性和效能目標](storage-scalability-targets.md)之外，還有以下延展性目標。 如果您的應用程式需求超出單一儲存體帳戶的延展性目標，請建置使用多個儲存體帳戶的應用程式，並將資料分散到那些儲存體帳戶中。

| 總帳戶容量 | 本地備援儲存體帳戶總頻寬 |
|:--- |:--- |
| 磁碟容量：35 TB<br />快照容量：10 TB |每秒最多 50 Gbps (輸入 + 輸出) |

如需有關「進階儲存體」規格的詳細資訊，請查看[使用進階儲存體時的延展性和效能目標](../../virtual-machines/windows/premium-storage.md#scalability-and-performance-targets)。

#### <a name="disk-caching-policy"></a>磁碟快取原則
根據預設，所有 Premium 資料磁碟的磁碟快取原則都是*唯讀*，而連接至 VM 的 Premium 作業系統磁碟的磁碟快取原則則是*讀寫*。 為使應用程式的 IO 達到最佳效能，建議使用此組態設定。 對於頻繁寫入或唯寫的資料磁碟 (例如 SQL Server 記錄檔)，停用磁碟快取可獲得更佳的應用程式效能。 而您可以使用 [Azure 入口網站](https://portal.azure.com)或 *Set-AzureDataDisk* Cmdlet 的 *-HostCaching* 參數來更新現有資料磁碟的快取設定。

#### <a name="location"></a>位置
挑選 Azure 進階儲存體可用的位置。 如需可使用 Azure 服務之地點的最新資訊，請參閱[依區域提供的 Azure 服務](https://azure.microsoft.com/regions/#services)。 相較於與儲存 VM 磁碟的儲存體帳戶位於不同區域的 VM，位於相同區域將提供更優越的效能。

#### <a name="other-azure-vm-configuration-settings"></a>其他 Azure VM 組態設定
建立 Azure VM 時，系統會要求您設定某些 VM 設定。 請記住，有一些對於 VM 存留期的設定是固定的，但是您稍後可以修改或新增其他設定。 檢閱這些 Azure VM 組態設定，並確定會適當地設定這些設定以符合您的工作負載需求。

### <a name="optimization"></a>最佳化
[Azure 進階儲存體︰針對高效能進行設計](../../virtual-machines/windows/premium-storage-performance.md)提供使用 Azure 進階儲存體來建置高效能應用程式的指導方針。 您可以遵循指定方針，並根據您的應用程式所採用的技術，結合適合的效能最佳作法。

## <a name="prepare-and-copy-virtual-hard-disks-VHDs-to-premium-storage"></a>準備並將虛擬硬碟 (VHD) 複製到進階儲存體
下一節提供從您的 VM 準備 VHD 以及將 VHD 複製到 Azure 儲存體的指導方針。

* [案例 1：「將現有的 Azure VM 移轉到 Azure 進階儲存體。」](#scenario1)
* [案例 2：「我要將其他平台上的 VM 移轉到 Azure 進階儲存體。」](#scenario2)

### <a name="prerequisites"></a>必要條件
若要準備移轉 VHD，您需要︰

* Azure 訂用帳戶、儲存體帳戶，以及該儲存體帳戶中存放所複製 VHD 的容器。 請注意，目的地儲存體帳戶可以是標準或進階儲存體帳戶，端視您的需求而定。
* 將 VHD 一般化的工具 (如果您打算透過 VHD 建立多個 VM 執行個體)。 例如，適用於 Windows 的 sysprep 或適用於 Ubuntu 的 virt-sysprep。
* 將 VHD 檔案上傳至儲存體帳戶的工具。 請參閱[使用 AzCopy 命令列公用程式傳輸資料](storage-use-azcopy.md)或使用 [Azure 儲存體總管](http://blogs.msdn.com/b/windowsazurestorage/archive/2014/03/11/windows-azure-storage-explorers-2014.aspx)。 本指南說明使用 AzCopy 工具複製 VHD 的方法。

> [!NOTE]
> 如果您選擇 AzCopy 的同步複製選項，為了達到最佳效能，請使用 Azure VM 中的上述其中一項工具複製 VHD，此 Azure VM 必須和目的地儲存體帳戶位於同一區域。 如果從不同區域的 Azure VM 複製 VHD，您的效能可能會變慢。
>
> 若要在頻寬有限的情況下複製大量資料，請考慮使用 [Azure 匯入/匯出服務來將資料傳送至 Blob 儲存體](../storage-import-export-service.md)，透過將硬碟運送到 Azure 資料中心的方式來傳輸資料。 使用 Azure 匯入/匯出服務時，您僅能將資料複製到標準儲存體帳戶。 當資料在您的標準儲存體帳戶中之後，您就可以使用 [複製 Blob API](https://msdn.microsoft.com/library/azure/dd894037.aspx) 或 AzCopy，將資料移轉到您的進階儲存體帳戶。
>
> 請注意，Microsoft Azure 僅支援固定大小的 VHD 檔案。 不支援 VHDX 檔案或動態 VHD。 如果您有動態 VHD，可以使用 [Convert-VHD](http://technet.microsoft.com/library/hh848454.aspx) Cmdlet，將其轉換成固定的大小。
>
>

### <a name="scenario1"></a>案例 1：「將現有的 Azure VM 移轉到 Azure 進階儲存體。」
如果您要移轉現有的 Azure VM，請停止 VM，依據您想要的 VHD 類型準備 VHD，然後使用 AzCopy 或 PowerShell 複製 VHD。

VM 必須完全關閉，才能以全新狀態移轉。 在移轉完成之前會有停機時間。

#### <a name="step-1-prepare-vhds-for-migration"></a>步驟 1. 準備 VHD 進行移轉
如果是將現有的 Azure VM 移轉到進階儲存體，您的 VHD 可能是：

* 一般化作業系統映像
* 唯一的作業系統磁碟
* 資料磁碟

下面將逐步解說準備 VHD 的 3 個案例。

##### <a name="use-a-generalized-operating-system-vhd-to-create-multiple-vm-instances"></a>使用一般化作業系統 VHD 建立多個 VM 執行個體
如果您要上傳將用於建立多個一般 Azure VM 執行個體的 VHD，您必須先使用 sysprep 公用程式將 VHD 一般化。 這適用於內部部署或雲端的 VHD。 Sysprep 會移除 VHD 中所有電腦特定的資訊。

> [!IMPORTANT]
> 將 VM 一般化之前，請先擷取快照或備份。 執行 sysprep 會停止並解除配置 VM 執行個體。 按照下方的步驟操作，以在 Windows OS VHD 中執行 sysprep。 請注意，執行 Sysprep 命令會要求您關閉虛擬機器。 如需 Sysprep 的詳細資訊，請參閱 [Sysprep 概觀](http://technet.microsoft.com/library/hh825209.aspx)或 [Sysprep 技術參考](http://technet.microsoft.com/library/cc766049.aspx)。
>
>

1. 以系統管理員身分開啟 [命令提示字元] 視窗。
2. 輸入下列命令以開啟 Sysprep：

    ```
    %windir%\system32\sysprep\sysprep.exe
    ```

3. 在 [系統準備工具] 中依序選取 [進入系統全新體驗 (OOBE)]、[一般化] 核取方塊、[關機]，然後按一下 [確定] \(如下方圖片所示)。 Sysprep 會將作業系統一般化並關閉系統。

    ![][1]

若是 Ubuntu VM，使用 virt-sysprep 可達到相同效果。 如需詳細資訊，請參閱 [virt-sysprep](http://manpages.ubuntu.com/manpages/precise/man1/virt-sysprep.1.html) 。 至於其他 Linux 作業系統，請參閱 [Linux 伺服器佈建軟體](http://www.cyberciti.biz/tips/server-provisioning-software.html) 的一些開放原始碼。

##### <a name="use-a-unique-operating-system-vhd-to-create-a-single-vm-instance"></a>使用唯一作業系統 VHD 建立單一 VM 執行個體
如果您有應用程式在需要機器專屬資料的 VM 上執行，請勿將 VHD 一般化。 非一般化的 VHD 可用來建立唯一的 Azure VM 執行個體。 例如，如果您的 VHD 上有網域控制站，執行 sysprep 將會使它與網域控制站一樣沒有效率。 檢閱在您的 VM 上執行的應用程式，以及對這些應用程式執行 sysprep 的影響，然後再將 VHD 一般化。

##### <a name="register-data-disk-vhd"></a>註冊資料磁碟 VHD
如果您在 Azure 中有要移轉的資料磁碟，必須確定使用這些資料磁碟的 VM 已關閉。

遵循以下所述的步驟，將 VHD 複製到 Azure 進階儲存體並註冊為佈建的資料磁碟。

#### <a name="step-2-create-the-destination-for-your-vhd"></a>步驟 2. 為您的 VHD 建立目的地
建立儲存體帳戶來維護您的 VHD。 規劃儲存 VHD 的位置時，請考量下列幾點：

* 目標進階儲存體帳戶。
* 儲存體帳戶位置必須與您將在最終階段建立之可支援進階儲存體的 Azure VM 相同。 您可以複製到新的儲存體帳戶，或打算根據您的需求，使用相同的儲存體帳戶。
* 為下一個階段複製並儲存目的地儲存體帳戶的儲存體帳戶金鑰。

若是資料磁碟，您可以選擇將一些資料磁碟保留在標準儲存體帳戶中 (例如，具有散熱器儲存體的磁碟)，但我們強烈建議您移動生產工作負載的所有資料以使用進階儲存體。

#### <a name="copy-vhd-with-azcopy-or-powershell"></a>步驟 3. 使用 AzCopy 或 PowerShell 複製 VHD
您必須尋找您的容器路徑和儲存體帳戶金鑰，才能處理這兩個選項之一。 容器路徑和儲存體帳戶金鑰位於 **Azure 入口網站** > **儲存體**。 容器 URL 會類似 「https://myaccount.blob.core.windows.net/mycontainer/」。

##### <a name="option-1-copy-a-vhd-with-azcopy-asynchronous-copy"></a>選項 1︰使用 AzCopy 複製 VHD (非同步複製)
您可以使用 AzCopy，透過網際網路輕鬆上傳 VHD。 根據 VHD 的大小，這可能需要一些時間。 使用這個選項時，請記得檢查儲存體帳戶輸入/輸出限制。 如需詳細資訊，請參閱 [Azure 儲存體延展性和效能目標](storage-scalability-targets.md) 。

1. 從這裡下載並安裝 AzCopy： [AzCopy 的最新版本](http://aka.ms/downloadazcopy)
2. 開啟 Azure PowerShell，並移至安裝 AzCopy 所在的資料夾。
3. 使用下列命令，將 VHD 檔案從「來源」複製到「目的地」。

    ```azcopy
    AzCopy /Source: <source> /SourceKey: <source-account-key> /Dest: <destination> /DestKey: <dest-account-key> /BlobType:page /Pattern: <file-name>
    ```

    範例：

    ```azcopy
    AzCopy /Source:https://sourceaccount.blob.core.windows.net/mycontainer1 /SourceKey:key1 /Dest:https://destaccount.blob.core.windows.net/mycontainer2 /DestKey:key2 /Pattern:abc.vhd
    ```

    以下是有關用於 AzCopy 命令中之參數的描述：

   * **/Source: *&lt;source&gt;：*** 包含 VHD 的資料夾或儲存體容器 URL 的位置。
   * **/SourceKey: *&lt;source-account-key&gt;：* \**來源儲存體帳戶的儲存體帳戶金鑰。
   * **/Dest: *&lt;destination&gt;：* \**要複製 VHD 的目的地儲存體容器 URL。
   * **/DestKey: *&lt;dest-account-key&gt;：* \**目的地儲存體帳戶的儲存體帳戶金鑰。
   * **/Pattern: *&lt;file-name&gt;：*** 指定要複製 VHD 的目標檔案名稱。

如需使用 AzCopy 工具的詳細資訊，請參閱 [使用 AzCopy 命令列公用程式傳輸資料](storage-use-azcopy.md)》\。

##### <a name="option-2-copy-a-vhd-with-powershell-synchronized-copy"></a>選項 2：使用 PowerShell 複製 VHD (同步處理的複製)
您也可以使用 PowerShell Cmdlet Start-AzureStorageBlobCopy 來複製 VHD 檔案。 在 Azure PowerShell 上使用下列命令來複製 VHD。 將 <> 中的值取代為您的來源和目的地儲存體帳戶中的對應值。 若要使用此命令，您的目的地儲存體帳戶中必須具有名為 vhds 的容器。 如果此容器不存在，請先加以建立，再執行命令。

```powershell
$sourceBlobUri = <source-vhd-uri>

$sourceContext = New-AzureStorageContext  –StorageAccountName <source-account> -StorageAccountKey <source-account-key>

$destinationContext = New-AzureStorageContext  –StorageAccountName <dest-account> -StorageAccountKey <dest-account-key>

Start-AzureStorageBlobCopy -srcUri $sourceBlobUri -SrcContext $sourceContext -DestContainer <dest-container> -DestBlob <dest-disk-name> -DestContext $destinationContext
```

範例：

```powershell
C:\PS> $sourceBlobUri = "https://sourceaccount.blob.core.windows.net/vhds/myvhd.vhd"

C:\PS> $sourceContext = New-AzureStorageContext  –StorageAccountName "sourceaccount" -StorageAccountKey "J4zUI9T5b8gvHohkiRg"

C:\PS> $destinationContext = New-AzureStorageContext  –StorageAccountName "destaccount" -StorageAccountKey "XZTmqSGKUYFSh7zB5"

C:\PS> Start-AzureStorageBlobCopy -srcUri $sourceBlobUri -SrcContext $sourceContext -DestContainer "vhds" -DestBlob "myvhd.vhd" -DestContext $destinationContext
```

### <a name="scenario2"></a>案例 2：「我要將其他平台上的 VM 移轉到 Azure 進階儲存體。」
如果您將 VHD 從非 Azure 雲端儲存體移轉至 Azure，您必須先將 VHD 匯出至本機目錄。 讓儲存 VHD 所在本機目錄的完整來源路徑位於方便取得的位置，然後使用 AzCopy 將它上傳至 Azure 儲存體。

#### <a name="step-1-export-vhd-to-a-local-directory"></a>步驟 1. 將 VHD 匯出至本機目錄
##### <a name="copy-a-vhd-from-aws"></a>從 AWS 複製 VHD
1. 如果您使用的是 AWS，請將 EC2 執行個體匯出至 Amazon S3 貯體中的 VHD。 請依照 Amazon 文件中所述的「匯出 Amazon EC2 執行個體」步驟，安裝 Amazon EC2 命令列介面 (CLI) 工具並執行 create-instance-export-task 命令，將 EC2 執行個體匯出到 VHD 檔案。 執行 **create-instance-export-task** 命令時，請務必讓 DISK&#95;IMAGE&#95;FORMAT 變數使用 **VHD**。 已匯出的 VHD 檔案會儲存在您在該程序期間所指定的 Amazon S3 貯體中。

    ```
    aws ec2 create-instance-export-task --instance-id ID --target-environment TARGET_ENVIRONMENT \
      --export-to-s3-task DiskImageFormat=DISK_IMAGE_FORMAT,ContainerFormat=ova,S3Bucket=BUCKET,S3Prefix=PREFIX
    ```

2. 從 S3 貯體下載 VHD 檔案。 然後選取 VHD 檔案，再依序按一下 [動作] > [下載]。

    ![][3]

##### <a name="copy-a-vhd-from-other-non-azure-cloud"></a>從其他非 Azure 雲端複製 VHD
如果您將 VHD 從非 Azure 雲端儲存體移轉至 Azure，您必須先將 VHD 匯出至本機目錄。 複製儲存 VHD 所在本機目錄的完整來源路徑。

##### <a name="copy-a-vhd-from-on-premises"></a>從內部部署複製 VHD
如果您要從內部部署環境移轉 VHD，將需要儲存 VHD 的完整來源路徑。 來源路徑可能是伺服器位置或檔案共用。

#### <a name="step-2-create-the-destination-for-your-vhd"></a>步驟 2. 為您的 VHD 建立目的地
建立儲存體帳戶來維護您的 VHD。 規劃儲存 VHD 的位置時，請考量下列幾點：

* 目標儲存體帳戶可能是 Standard 或進階儲存體，端視您的應用程式需求而定。
* 儲存體帳戶區域必須與您將在最終階段建立之可支援進階儲存體的 Azure VM 相同。 您可以複製到新的儲存體帳戶，或打算根據您的需求，使用相同的儲存體帳戶。
* 為下一個階段複製並儲存目的地儲存體帳戶的儲存體帳戶金鑰。

強烈建議您移動生產工作負載的所有資料以使用進階儲存體。

#### <a name="step-3-upload-the-vhd-to-azure-storage"></a>步驟 3. 將 VHD 上傳至 Azure 儲存體
現在，您的 VHD 位於本機目錄中，您可以使用 AzCopy 或 AzurePowerShell 將 .vhd 檔案上傳至 Azure 儲存體。 下面提供兩個選項︰

##### <a name="option-1-using-azure-powershell-add-azurevhd-to-upload-the-vhd-file"></a>選項 1︰使用 Azure PowerShell Add-AzureVhd 上傳 .vhd 檔案

```powershell
Add-AzureVhd [-Destination] <Uri> [-LocalFilePath] <FileInfo>
```

範例 <Uri> 可能是 ***"https://storagesample.blob.core.windows.net/mycontainer/blob1.vhd"***。 範例 <FileInfo> 可能是 ***"C:\path\to\upload.vhd"***。

##### <a name="option-2-using-azcopy-to-upload-the-vhd-file"></a>選項 2︰使用 AzCopy 上傳 .vhd 檔案
您可以使用 AzCopy，透過網際網路輕鬆上傳 VHD。 根據 VHD 的大小，這可能需要一些時間。 使用這個選項時，請記得檢查儲存體帳戶輸入/輸出限制。 如需詳細資訊，請參閱 [Azure 儲存體延展性和效能目標](storage-scalability-targets.md) 。

1. 從這裡下載並安裝 AzCopy： [AzCopy 的最新版本](http://aka.ms/downloadazcopy)
2. 開啟 Azure PowerShell，並移至安裝 AzCopy 所在的資料夾。
3. 使用下列命令，將 VHD 檔案從「來源」複製到「目的地」。

    ```azcopy
    AzCopy /Source: <source> /SourceKey: <source-account-key> /Dest: <destination> /DestKey: <dest-account-key> /BlobType:page /Pattern: <file-name>
    ```

    範例：

    ```azcopy
    AzCopy /Source:https://sourceaccount.blob.core.windows.net/mycontainer1 /SourceKey:key1 /Dest:https://destaccount.blob.core.windows.net/mycontainer2 /DestKey:key2 /BlobType:page /Pattern:abc.vhd
    ```

    以下是有關用於 AzCopy 命令中之參數的描述：

   * **/Source: *&lt;source&gt;：*** 包含 VHD 的資料夾或儲存體容器 URL 的位置。
   * **/SourceKey: *&lt;source-account-key&gt;：* \**來源儲存體帳戶的儲存體帳戶金鑰。
   * **/Dest: *&lt;destination&gt;：* \**要複製 VHD 的目的地儲存體容器 URL。
   * **/DestKey: *&lt;dest-account-key&gt;：* \**目的地儲存體帳戶的儲存體帳戶金鑰。
   * **/BlobType: page:** 將目的地指定為分頁 Blob。
   * **/Pattern: *&lt;file-name&gt;：* \**指定要複製 VHD 的目標檔案名稱。

如需使用 AzCopy 工具的詳細資訊，請參閱 [使用 AzCopy 命令列公用程式傳輸資料](storage-use-azcopy.md)》\。

##### <a name="other-options-for-uploading-a-vhd"></a>上傳 VHD 的其他選項
您也可以使用下方其中一種方法將 VHD 上傳至您的儲存體帳戶：

* [Azure 儲存體複製 Blob API](https://msdn.microsoft.com/library/azure/dd894037.aspx)
* [Azure 儲存體總管上傳 Blob](https://azurestorageexplorer.codeplex.com/)
* [儲存體匯入/匯出服務 REST API 參考](https://msdn.microsoft.com/library/dn529096.aspx)

> [!NOTE]
> 如果預估的上傳時間長度超過 7 天，我們建議使用匯入/匯出服務。 您可以使用 [DataTransferSpeedCalculator](https://github.com/Azure-Samples/storage-dotnet-import-export-job-management/blob/master/DataTransferSpeedCalculator.html) 從資料大小和傳輸單位來評估時間。
>
> 匯入/匯出可用來複製到標準儲存體帳戶。 您必須使用 AzCopy 之類的工具，從 Standard 儲存體帳戶複製到進階儲存體帳戶。
>
>

## <a name="create-azure-virtual-machine-using-premium-storage"></a>使用進階儲存體建立 Azure VM
將 VHD 上傳或複製到所需的儲存體帳戶之後，請遵循本節中的指示，根據您的情況，將 VHD 註冊為作業系統映像或作業系統磁碟，然後從該映像或磁碟建立 VM 執行個體。 一旦建立之後，資料磁碟 VHD 就可以連接至 VM。
本節的結尾會提供範例移轉指令碼。 這個簡單的指令碼不符合所有案例。 您可能需要更新指令碼以符合您的特定案例。 若要查看此指令碼是否適用於您的案例，請參閱下面的[範例移轉指令碼](#a-sample-migration-script)。

### <a name="checklist"></a>檢查清單
1. 等到所有的 VHD 磁碟複製完成。
2. 請確定進階儲存體可在您要移轉到的區域中使用。
3. 決定您將要使用的新 VM 系列。 這應可支援進階儲存體，而且大小取決於區域中的可用性和您的需求。
4. 決定您將要使用的確切 VM 大小。 VM 大小必須足以支援您所擁有的資料磁碟數目。 例如 如果您有 4 個資料磁碟，VM 必須有 2 個或多個核心。 也請考慮處理能力、記憶體和網路頻寬需求。
5. 在目標區域中建立進階儲存體帳戶。 這是您將用於新 VM 的帳戶。
6. 請備妥目前 VM 的詳細資料，包括磁碟和對應 VHD Blob 的清單。

為您的應用程式做好停機準備。 若要進行全新移轉，您必須停止目前系統中的所有處理。 只有這樣，您才可以使其維持在可移轉至新平台的一致狀態。 停機持續時間將取決於磁碟中要移轉的資料量。

> [!NOTE]
> 如果您要從指定的 VHD 磁碟建立 Azure Resource Manager VM，請參閱[此範本](https://github.com/Azure/azure-quickstart-templates/tree/master/201-vm-specialized-vhd)以便使用現有的磁碟部署 Azure Resource Manager VM。
>
>

### <a name="register-your-vhd"></a>註冊 VHD
若要透過 OS VHD 建立 VM，或將資料磁碟連接到新的 VM，您必須先註冊 VHD。 根據您的 VHD 案例，遵循以下步驟。

#### <a name="generalized-operating-system-vhd-to-create-multiple-azure-vm-instances"></a>建立多個 Azure VM 執行個體的一般化的作業系統 VHD
將一般化的作業系統映像 VHD 上傳至儲存體帳戶之後，將其註冊為 **Azure VM 映像**，以便您從該映像建立一個或多個 VM 執行個體。 使用下列 PowerShell Cmdlet 將您的 VHD 註冊為 Azure VM 作業系統映像。 提供存放複製的 VHD 的完整容器 URL。

```powershell
Add-AzureVMImage -ImageName "OSImageName" -MediaLocation "https://storageaccount.blob.core.windows.net/vhdcontainer/osimage.vhd" -OS Windows
```

複製並儲存這個新 Azure VM 映像的名稱。 在上述範例中，該名稱是 *OSImageName*。

#### <a name="unique-operating-system-vhd-to-create-a-single-azure-vm-instance"></a>建立單一 Azure VM 執行個體的唯一作業系統 VHD
將唯一的作業系統 VHD 上傳至儲存體帳戶之後，將其註冊為 **Azure 作業系統磁碟**，以便您從該磁碟建立 VM 執行個體。 使用下列 PowerShell Cmdlet 將您的 VHD 註冊為 Azure 作業系統磁碟。 提供存放複製的 VHD 的完整容器 URL。

```powershell
Add-AzureDisk -DiskName "OSDisk" -MediaLocation "https://storageaccount.blob.core.windows.net/vhdcontainer/osdisk.vhd" -Label "My OS Disk" -OS "Windows"
```

複製並儲存這個新 Azure 作業系統磁碟的名稱。 在上述範例中，該名稱是「OSDisk」 。

#### <a name="data-disk-vhd-to-be-attached-to-new-azure-vm-instances"></a>連接至新 Azure VM 執行個體的資料磁碟 VHD
將資料磁碟 VHD 上傳至儲存體帳戶之後，將其註冊為 Azure 資料磁碟，便可以連接到新的 DS 系列、DSv2 系列或 GS 系列 Azure VM 執行個體。

使用下列 PowerShell Cmdlet 將您的 VHD 註冊為 Azure 資料磁碟。 提供存放複製的 VHD 的完整容器 URL。

```powershell
Add-AzureDisk -DiskName "DataDisk" -MediaLocation "https://storageaccount.blob.core.windows.net/vhdcontainer/datadisk.vhd" -Label "My Data Disk"
```

複製並儲存這個新 Azure 資料磁碟的名稱。 在上述範例中，該名稱是「DataDisk」 。

### <a name="create-a-premium-storage-capable-vm"></a>建立可支援進階儲存體的 VM
註冊 OS 映像或 OS 磁碟之後，請建立新的 DS 系列、DSv2 系列或 GS 系列 VM。 您將使用您註冊的作業系統映像或作業系統磁碟名稱。 從進階儲存體層選取 VM 類型。 在下列範例中，我們使用的 VM 大小為「Standard_DS2」。

> [!NOTE]
> 更新磁碟大小以確定它符合您的容量、效能需求，和可用的 Azure 磁碟大小。
>
>

依照下方的 PowerShell Cmdlet 逐步建立新的 VM。 首先，設定一般參數：

```powershell
$serviceName = "yourVM"
$location = "location-name" (e.g., West US)
$vmSize ="Standard_DS2"
$adminUser = "youradmin"
$adminPassword = "yourpassword"
$vmName ="yourVM"
$vmSize = "Standard_DS2"
```

首先，建立您將在當中裝載新 VM 的雲端服務。

```powershell
New-AzureService -ServiceName $serviceName -Location $location
```

接下來，根據您的情況，透過您註冊的 OS 映像或 OS 磁碟，建立 Azure VM 執行個體。

#### <a name="generalized-operating-system-vhd-to-create-multiple-azure-vm-instances"></a>建立多個 Azure VM 執行個體的一般化的作業系統 VHD
使用您註冊的 **Azure 作業系統映像** ，建立一個或多個新的 DS 系列 Azure VM 執行個體。 如下方所示，在建立新的 VM 時，於 VM 組態中指定此 OS 映像名稱。

```powershell
$OSImage = Get-AzureVMImage –ImageName "OSImageName"

$vm = New-AzureVMConfig -Name $vmName –InstanceSize $vmSize -ImageName $OSImage.ImageName

Add-AzureProvisioningConfig -Windows –AdminUserName $adminUser -Password $adminPassword –VM $vm

New-AzureVM -ServiceName $serviceName -VM $vm
```

#### <a name="unique-operating-system-vhd-to-create-a-single-azure-vm-instance"></a>建立單一 Azure VM 執行個體的唯一作業系統 VHD
使用您註冊的 **Azure 作業系統磁碟** ，建立新的 DS 系列 Azure VM 執行個體。 如下方所示，在建立新的 VM 時，於 VM 組態中指定此 OS 的磁碟名稱。

```powershell
$OSDisk = Get-AzureDisk –DiskName "OSDisk"

$vm = New-AzureVMConfig -Name $vmName -InstanceSize $vmSize -DiskName $OSDisk.DiskName

New-AzureVM -ServiceName $serviceName –VM $vm
```

指定其他 Azure VM 資訊，例如雲端服務、區域、儲存體帳戶、可用性設定組以及快取原則。 請注意，VM 執行個體必須與相關的作業系統或資料磁碟共置，因此選取的雲端服務、區域和儲存體帳戶全都必須與這些磁碟的基礎 VHD 位於相同的位置。

### <a name="attach-data-disk"></a>連結資料磁碟
最後，如果您已經註冊資料磁碟 VHD，請將它們連接至可支援進階儲存體的新 Azure VM。

使用下列 PowerShell Cmdlet，將資料磁碟連接至新的 VM，並指定快取原則。 在下列範例中，快取原則設定為「ReadOnly」 。

```powershell
$vm = Get-AzureVM -ServiceName $serviceName -Name $vmName

Add-AzureDataDisk -ImportFrom -DiskName "DataDisk" -LUN 0 –HostCaching ReadOnly –VM $vm

Update-AzureVM  -VM $vm
```

> [!NOTE]
> 可能有一些支援應用程式所需的其他步驟不包含在本指南中。
>
>

### <a name="checking-and-plan-backup"></a>檢查和規劃備份
在新的 VM 啟動並執行後，請使用與原始 VM 相同的登入識別碼和密碼加以存取，並確認一切如預期般運作。 所有設定 (包括等量磁碟區) 都會出現在新的 VM 中。

最後一個步驟是依據應用程式的需求為新的 VM 規劃備份和維護排程。

### <a name="a-sample-migration-script"></a>範例移轉指令碼
如果您有多個要移轉的 VM，透過 PowerShell 指令碼將其自動化會很有幫助。 以下是將 VM 的移轉自動化的範例指令碼。 請注意，以下指令碼只是範例，且對於目前的 VM 磁碟做了幾個相關假設。 您可能需要更新指令碼以符合您的特定案例。

假設如下：

* 您正在建立傳統 Azure VM。
* 您的來源作業系統磁碟和來源資料磁碟位於相同儲存體帳戶和相同容器中。 如果您的作業系統磁碟和資料磁碟不在相同的地方，您可以使用 AzCopy 或 Azure PowerShell 來複製不同儲存體帳戶和容器之間的 VHD。 請參閱上一個步驟︰[使用 AzCopy 或 PowerShell 複製 VHD](#copy-vhd-with-azcopy-or-powershell)。 編輯此指令碼以符合您的案例是另一個選擇，但我們建議使用 AzCopy 或 PowerShell，因為比較容易且快速。

以下提供自動化指令碼。 以您的資訊取代文字，並更新指令碼以符合您的特定案例。

> [!NOTE]
> 使用現有的指令碼不會保留來源 VM 的網路組態。 您必須在移轉後的 VM 上重新設定網路設定。
>
>

```
    <#
    .Synopsis
    This script is provided as an EXAMPLE to show how to migrate a VM from a standard storage account to a premium storage account. You can customize it according to your specific requirements.

    .Description
    The script will copy the vhds (page blobs) of the source VM to the new storage account.
    And then it will create a new VM from these copied vhds based on the inputs that you specified for the new VM.
    You can modify the script to satisfy your specific requirement, but please be aware of the items specified
    in the Terms of Use section.

    .Terms of Use
    Copyright © 2015 Microsoft Corporation.  All rights reserved.

    THIS CODE AND ANY ASSOCIATED INFORMATION ARE PROVIDED "AS IS" WITHOUT WARRANTY OF ANY KIND,
    EITHER EXPRESSED OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE IMPLIED WARRANTIES OF MERCHANTABILITY
    AND/OR FITNESS FOR A PARTICULAR PURPOSE. THE ENTIRE RISK OF USE, INABILITY TO USE, OR
    RESULTS FROM THE USE OF THIS CODE REMAINS WITH THE USER.

    .Example (Save this script as Migrate-AzureVM.ps1)

    .\Migrate-AzureVM.ps1 -SourceServiceName CurrentServiceName -SourceVMName CurrentVMName –DestStorageAccount newpremiumstorageaccount -DestServiceName NewServiceName -DestVMName NewDSVMName -DestVMSize "Standard_DS2" –Location "Southeast Asia"

    .Link
    To find more information about how to set up Azure PowerShell, refer to the following links.
    http://azure.microsoft.com/documentation/articles/powershell-install-configure/
    http://azure.microsoft.com/documentation/articles/storage-powershell-guide-full/
    http://azure.microsoft.com/blog/2014/10/22/migrate-azure-virtual-machines-between-storage-accounts/

    #>

    Param(
    # the cloud service name of the VM.
    [Parameter(Mandatory = $true)]
    [string] $SourceServiceName,

    # The VM name to copy.
    [Parameter(Mandatory = $true)]
    [String] $SourceVMName,

    # The destination storage account name.
    [Parameter(Mandatory = $true)]
    [String] $DestStorageAccount,

    # The destination cloud service name
    [Parameter(Mandatory = $true)]
    [String] $DestServiceName,

    # the destination vm name
    [Parameter(Mandatory = $true)]
    [String] $DestVMName,

    # the destination vm size
    [Parameter(Mandatory = $true)]
    [String] $DestVMSize,

    # the location of destination VM.
    [Parameter(Mandatory = $true)]
    [string] $Location,

    # whether or not to copy the os disk, the default is only copy data disks
    [Parameter(Mandatory = $false)]
    [Bool] $DataDiskOnly = $true,

    # how frequently to report the copy status in sceconds
    [Parameter(Mandatory = $false)]
    [Int32] $CopyStatusReportInterval = 15,

    # the name suffix to add to new created disks to avoid conflict with source disk names
    [Parameter(Mandatory = $false)]
    [String]$DiskNameSuffix = "-prem"

    ) #end param

    #######################################################################
    #  Verify Azure PowerShell module and version
    #######################################################################

    #import the Azure PowerShell module
    Write-Host "`n[WORKITEM] - Importing Azure PowerShell module" -ForegroundColor Yellow
    $azureModule = Import-Module Azure -PassThru

    if ($azureModule -ne $null)
    {
        Write-Host "`tSuccess" -ForegroundColor Green
    }
    else
    {
        #show module not found interaction and bail out
        Write-Host "[ERROR] - PowerShell module not found. Exiting." -ForegroundColor Red
        Exit
    }


    #Check the Azure PowerShell module version
    Write-Host "`n[WORKITEM] - Checking Azure PowerShell module verion" -ForegroundColor Yellow
    If ($azureModule.Version -ge (New-Object System.Version -ArgumentList "0.8.14"))
    {
        Write-Host "`tSuccess" -ForegroundColor Green
    }
    Else
    {
        Write-Host "[ERROR] - Azure PowerShell module must be version 0.8.14 or higher. Exiting." -ForegroundColor Red
        Exit
    }

    #Check if there is an azure subscription set up in PowerShell
    Write-Host "`n[WORKITEM] - Checking Azure Subscription" -ForegroundColor Yellow
    $currentSubs = Get-AzureSubscription -Current
    if ($currentSubs -ne $null)
    {
        Write-Host "`tSuccess" -ForegroundColor Green
        Write-Host "`tYour current azure subscription in PowerShell is $($currentSubs.SubscriptionName)." -ForegroundColor Green
    }
    else
    {
        Write-Host "[ERROR] - There is no valid Azure subscription found in PowerShell. Please refer to this article http://azure.microsoft.com/documentation/articles/powershell-install-configure/ to connect an Azure subscription. Exiting." -ForegroundColor Red
        Exit
    }


    #######################################################################
    #  Check if the VM is shut down
    #  Stopping the VM is a required step so that the file system is consistent when you do the copy operation.
    #  Azure does not support live migration at this time..
    #######################################################################

    if (($sourceVM = Get-AzureVM –ServiceName $SourceServiceName –Name $SourceVMName) -eq $null)
    {
        Write-Host "[ERROR] - The source VM doesn't exist in the current subscription. Exiting." -ForegroundColor Red
        Exit
    }

    # check if VM is shut down
    if ( $sourceVM.Status -notmatch "Stopped" )
    {
        Write-Host "[Warning] - Stopping the VM is a required step so that the file system is consistent when you do the copy operation. Azure does not support live migration at this time. If you'd like to create a VM from a generalized image, sys-prep the Virtual Machine before stopping it." -ForegroundColor Yellow
        $ContinueAnswer = Read-Host "`n`tDo you wish to stop $SourceVMName now? Input 'N' if you want to shut down the VM manually and come back later.(Y/N)"
        If ($ContinueAnswer -ne "Y") { Write-Host "`n Exiting." -ForegroundColor Red;Exit }
        $sourceVM | Stop-AzureVM

        # wait until the VM is shut down
        $VMStatus = (Get-AzureVM –ServiceName $SourceServiceName –Name $vmName).Status
        while ($VMStatus -notmatch "Stopped")
        {
            Write-Host "`n[Status] - Waiting VM $vmName to shut down" -ForegroundColor Green
            Sleep -Seconds 5
            $VMStatus = (Get-AzureVM –ServiceName $SourceServiceName –Name $vmName).Status
        }
    }

    # exporting the sourve vm to a configuration file, you can restore the original VM by importing this config file
    # see more information for Import-AzureVM
    $workingDir = (Get-Location).Path
    $vmConfigurationPath = $env:HOMEPATH + "\VM-" + $SourceVMName + ".xml"
    Write-Host "`n[WORKITEM] - Exporting VM configuration to $vmConfigurationPath" -ForegroundColor Yellow
    $exportRe = $sourceVM | Export-AzureVM -Path $vmConfigurationPath


    #######################################################################
    #  Copy the vhds of the source vm
    #  You can choose to copy all disks including os and data disks by specifying the
    #  parameter -DataDiskOnly to be $false. The default is to copy only data disk vhds
    #  and the new VM will boot from the original os disk.
    #######################################################################

    $sourceOSDisk = $sourceVM.VM.OSVirtualHardDisk
    $sourceDataDisks = $sourceVM.VM.DataVirtualHardDisks

    # Get source storage account information, not considering the data disks and os disks are in different accounts
    $sourceStorageAccountName = $sourceOSDisk.MediaLink.Host -split "\." | select -First 1
    $sourceStorageKey = (Get-AzureStorageKey -StorageAccountName $sourceStorageAccountName).Primary
    $sourceContext = New-AzureStorageContext –StorageAccountName $sourceStorageAccountName -StorageAccountKey $sourceStorageKey

    # Create destination context
    $destStorageKey = (Get-AzureStorageKey -StorageAccountName $DestStorageAccount).Primary
    $destContext = New-AzureStorageContext –StorageAccountName $DestStorageAccount -StorageAccountKey $destStorageKey

    # Create a container of vhds if it doesn't exist
    if ((Get-AzureStorageContainer -Context $destContext -Name vhds -ErrorAction SilentlyContinue) -eq $null)
    {
        Write-Host "`n[WORKITEM] - Creating a container vhds in the destination storage account." -ForegroundColor Yellow
        New-AzureStorageContainer -Context $destContext -Name vhds
    }


    $allDisksToCopy = $sourceDataDisks
    # check if need to copy os disk
    $sourceOSVHD = $sourceOSDisk.MediaLink.Segments[2]
    if ($DataDiskOnly)
    {
        # copy data disks only, this option requires deleting the source VM so that dest VM can boot
        # from the same vhd blob.
        $ContinueAnswer = Read-Host "`n`t[Warning] You chose to copy data disks only. Moving VM requires removing the original VM (the disks and backing vhd files will NOT be deleted) so that the new VM can boot from the same vhd. This is an irreversible action. Do you wish to proceed right now? (Y/N)"
        If ($ContinueAnswer -ne "Y") { Write-Host "`n Exiting." -ForegroundColor Red;Exit }
        $destOSVHD = Get-AzureStorageBlob -Blob $sourceOSVHD -Container vhds -Context $sourceContext
        Write-Host "`n[WORKITEM] - Removing the original VM (the vhd files are NOT deleted)." -ForegroundColor Yellow
        Remove-AzureVM -Name $SourceVMName -ServiceName $SourceServiceName

        Write-Host "`n[WORKITEM] - Waiting utill the OS disk is released by source VM. This may take up to several minutes."
        $diskAttachedTo = (Get-AzureDisk -DiskName $sourceOSDisk.DiskName).AttachedTo
        while ($diskAttachedTo -ne $null)
        {
            Start-Sleep -Seconds 10
            $diskAttachedTo = (Get-AzureDisk -DiskName $sourceOSDisk.DiskName).AttachedTo
        }

    }
    else
    {
        # copy the os disk vhd
        Write-Host "`n[WORKITEM] - Starting copying os disk $($disk.DiskName) at $(get-date)." -ForegroundColor Yellow
        $allDisksToCopy += @($sourceOSDisk)
        $targetBlob = Start-AzureStorageBlobCopy -SrcContainer vhds -SrcBlob $sourceOSVHD -DestContainer vhds -DestBlob $sourceOSVHD -Context $sourceContext -DestContext $destContext -Force
        $destOSVHD = $targetBlob
    }


    # Copy all data disk vhds
    # Start all async copy requests in parallel.
    foreach($disk in $sourceDataDisks)
    {
        $blobName = $disk.MediaLink.Segments[2]
        # copy all data disks
        Write-Host "`n[WORKITEM] - Starting copying data disk $($disk.DiskName) at $(get-date)." -ForegroundColor Yellow
        $targetBlob = Start-AzureStorageBlobCopy -SrcContainer vhds -SrcBlob $blobName -DestContainer vhds -DestBlob $blobName -Context $sourceContext -DestContext $destContext -Force
        # update the media link to point to the target blob link
        $disk.MediaLink = $targetBlob.ICloudBlob.Uri.AbsoluteUri
    }

    # Wait until all vhd files are copied.
    $diskComplete = @()
    do
    {
        Write-Host "`n[WORKITEM] - Waiting for all disk copy to complete. Checking status every $CopyStatusReportInterval seconds." -ForegroundColor Yellow
        # check status every 30 seconds
        Sleep -Seconds $CopyStatusReportInterval
        foreach ( $disk in $allDisksToCopy)
        {
            if ($diskComplete -contains $disk)
            {
                Continue
            }
            $blobName = $disk.MediaLink.Segments[2]
            $copyState = Get-AzureStorageBlobCopyState -Blob $blobName -Container vhds -Context $destContext
            if ($copyState.Status -eq "Success")
            {
                Write-Host "`n[Status] - Success for disk copy $($disk.DiskName) at $($copyState.CompletionTime)" -ForegroundColor Green
                $diskComplete += $disk
            }
            else
            {
                if ($copyState.TotalBytes -gt 0)
                {
                    $percent = ($copyState.BytesCopied / $copyState.TotalBytes) * 100
                    Write-Host "`n[Status] - $('{0:N2}' -f $percent)% Complete for disk copy $($disk.DiskName)" -ForegroundColor Green
                }
            }
        }
    }
    while($diskComplete.Count -lt $allDisksToCopy.Count)

    #######################################################################
    #  Create a new vm
    #  the new VM can be created from the copied disks or the original os disk.
    #  You can ddd your own logic here to satisfy your specific requirements of the vm.
    #######################################################################

    # Create a VM from the existing os disk
    if ($DataDiskOnly)
    {
        $vm = New-AzureVMConfig -Name $DestVMName -InstanceSize $DestVMSize -DiskName $sourceOSDisk.DiskName
    }
    else
    {
        $newOSDisk = Add-AzureDisk -OS $sourceOSDisk.OS -DiskName ($sourceOSDisk.DiskName + $DiskNameSuffix) -MediaLocation $destOSVHD.ICloudBlob.Uri.AbsoluteUri
        $vm = New-AzureVMConfig -Name $DestVMName -InstanceSize $DestVMSize -DiskName $newOSDisk.DiskName
    }
    # Attached the copied data disks to the new VM
    foreach ($dataDisk in $sourceDataDisks)
    {
        # add -DiskLabel $dataDisk.DiskLabel if there are labels for disks of the source vm
        $diskLabel = "drive" + $dataDisk.Lun
        $vm | Add-AzureDataDisk -ImportFrom -DiskLabel $diskLabel -LUN $dataDisk.Lun -MediaLocation $dataDisk.MediaLink
    }

    # Edit this if you want to add more custimization to the new VM
    # $vm | Add-AzureEndpoint -Protocol tcp -LocalPort 443 -PublicPort 443 -Name 'HTTPs'
    # $vm | Set-AzureSubnet "PubSubnet","PrivSubnet"

    New-AzureVM -ServiceName $DestServiceName -VMs $vm -Location $Location
```

#### <a name="optimization"></a>最佳化
您目前的 VM 組態可以自訂，以適用於標準磁碟。 例如，在等量磁碟區中使用多個磁碟，以提高效能。 例如，您無須在進階儲存體上個別使用 4 個磁碟，而可以藉由單一磁碟發揮成本效益。 這樣最佳化必須就個別案例來處理，且在移轉之後需執行自訂步驟。 另請注意，此程序可能不適用於依賴設定中所定義之磁碟配置的資料庫和應用程式。

##### <a name="preparation"></a>準備工作
1. 依照前一節中的說明完成簡單移轉。 在移轉之後，將會在新 VM 上執行最佳化。
2. 定義最佳化組態所需的新磁碟大小。
3. 判斷目前磁碟/磁碟區與新磁碟規格之間的對應。

##### <a name="execution-steps"></a>執行步驟
1. 在進階儲存體 VM 上以正確的大小建立新的磁碟。
2. 登入 VM，並將資料從目前的磁碟區複製到對應至該磁碟區的新磁碟。 請為所有需要對應至新磁碟的目前磁碟區執行這項操作。
3. 接下來，請變更應用程式設定以切換至新的磁碟，並卸離舊磁碟區。

如需調整應用程式以獲得更好的磁碟效能，請參閱[最佳化應用程式效能](../../virtual-machines/windows/premium-storage-performance.md#optimizing-application-performance)。

### <a name="application-migrations"></a>應用程式移轉
資料庫和其他複雜的應用程式在移轉時，可能需要應用程式提供者所定義的特殊步驟。 請參閱個別的應用程式文件。 例如 資料庫通常可透過備份和還原來移轉。

## <a name="next-steps"></a>後續步驟
如需移轉虛擬機器的特定案例，請參閱下列資源：

* [在儲存體帳戶之間移轉 Azure 虛擬機器](https://azure.microsoft.com/blog/2014/10/22/migrate-azure-virtual-machines-between-storage-accounts/)
* [建立並上傳 Windows Server VHD 到 Azure。](../../virtual-machines/windows/upload-generalized-managed.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json)
* [建立 Linux VHD 並上傳至 Azure](../../virtual-machines/linux/create-upload-generic.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json)
* [將虛擬機器從 Amazon AWS 移轉至 Microsoft Azure](http://channel9.msdn.com/Series/Migrating-Virtual-Machines-from-Amazon-AWS-to-Microsoft-Azure)

若要深入了解 Azure 儲存體和 Azure 虛擬機器，也請參閱下列資源：

* [Azure 儲存體](https://azure.microsoft.com/documentation/services/storage/)
* [Azure 虛擬機器](https://azure.microsoft.com/documentation/services/virtual-machines/)
* [Premium 儲存體：Azure 虛擬機器工作負載適用的高效能儲存體](../../virtual-machines/windows/premium-storage.md)

[1]:./media/storage-migration-to-premium-storage/migration-to-premium-storage-1.png
[2]:./media/storage-migration-to-premium-storage/migration-to-premium-storage-1.png
[3]:./media/storage-migration-to-premium-storage/migration-to-premium-storage-3.png
[4]: http://technet.microsoft.com/library/hh831739.aspx
