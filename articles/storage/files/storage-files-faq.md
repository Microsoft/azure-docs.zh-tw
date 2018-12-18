---
title: Azure 檔案服務的常見問題集 (FAQ) | Microsoft Docs
description: 尋找關於 Azure 檔案服務之常見問題集的解答。
services: storage
author: RenaShahMSFT
ms.service: storage
ms.date: 09/11/2018
ms.author: renash
ms.component: files
ms.openlocfilehash: 43acff5c4d37c46245566fb2e1d74d3e14d527bb
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46949837"
---
# <a name="frequently-asked-questions-faq-about-azure-files"></a>關於 Azure 檔案服務的常見問題集 (FAQ)
[Azure 檔案](storage-files-introduction.md)提供雲端中完全受控的檔案共用，可透過業界標準[伺服器訊息區 (SMB) 通訊協定](https://msdn.microsoft.com/library/windows/desktop/aa365233.aspx)來存取。 您可以同時在 Windows、Linux 和 macOS 的雲端或內部部署上掛接 Azure 檔案共用。 您也可以使用 Azure 檔案同步，在接近使用資料之處進行快速存取，藉以在 Windows Server 電腦上快取 Azure 檔案共用。

本文將回答有關 Azure 檔案服務特性與功能 (包括將 Azure 檔案同步與 Azure 檔案搭配使用) 的常見問題。 如果您找不到問題的答案，可透過下列管道 (依先後順序) 和我們連絡：

1. 本文的留言區。
2. [Azure 儲存體論壇](https://social.msdn.microsoft.com/forums/azure/home?forum=windowsazuredata)。
3. [Azure 檔案服務 UserVoice](https://feedback.azure.com/forums/217298-storage/category/180670-files) \(英文\)。 
4. Microsoft 支援服務。 若要建立新的支援要求，在 Azure 入口網站的 [說明] 索引標籤上，選取 [說明 + 支援] 按鈕，然後選取 [新增支援要求]。

## <a name="general"></a>一般
* <a id="why-files-useful"></a>
**Azure 檔案服務有多實用？**  
   您可以使用 Azure 檔案服務，在雲端中建立檔案共用，而不需負責管理實體伺服器、裝置或應用裝置的額外負荷。 我們會為您執行單調的工作，包括套用 OS 更新和替換損毀的磁碟。 若要深入了解 Azure 檔案服務可協助處理的案例，請參閱 [Azure 檔案服務為何很實用](storage-files-introduction.md#why-azure-files-is-useful)。

* <a id="file-access-options"></a>
**有哪些不同方式可以存取 Azure 檔案服務中的檔案？**  
    您可以使用 SMB 3.0 通訊協定，在本機電腦上掛接檔案共用，或者可以使用[儲存體總管](http://storageexplorer.com/)之類的工具來存取檔案共用中的檔案。 您可以從應用程式中，使用儲存體用戶端程式庫、REST API、PowerShell 或 Azure CLI 來存取 Azure 檔案共用中的檔案。

* <a id="what-is-afs"></a>
**什麼是 Azure 檔案同步？**  
    您可以使用 Azure 檔案同步，將組織的檔案共用集中在 Azure 檔案服務中，同時保有內部部署檔案伺服器的靈活度、效能及相容性。 Azure 檔案同步會將您的 Windows Server 電腦轉換成 Azure 檔案共用的快速快取。 您可以使用 Windows Server 上可用的任何通訊協定以從本機存取資料，包括 SMB、網路檔案系統 (NFS) 和檔案傳輸通訊協定服務 (FTPS)。 您可以視需要存取多個散佈於世界各地的快取。

* <a id="files-versus-blobs"></a>
**為什麼我要使用 Azure 檔案共用與 Azure Blob 儲存體來儲存我的資料？**  
    Azure 檔案服務和 Azure Blob 儲存體都會提供在雲端中儲存大量資料的方式，但其適用用途稍有不同。 
    
    Azure Blob 儲存體適用於需要儲存非結構化資料的大規模雲端原生應用程式。 為了充分發揮效能並進行擴充，Azure Blob 儲存體是比真實檔案系統更為簡單的儲存體抽象概念。 您只能透過 REST 型用戶端程式庫 (或直接透過 REST 型通訊協定) 存取 Azure Blob 儲存體。

    具體而言，Azure 檔案服務是一個檔案系統。 Azure 檔案服務具備您多年來熟知且喜愛用來與內部部署作業系統搭配使用的所有檔案摘要。 就像 Azure Blob 儲存體，Azure 檔案服務可提供 REST 介面和 REST 型用戶端程式庫。 不同於 Azure Blob 儲存體，Azure 檔案服務提供對 Azure 檔案共用的 SMB 存取。 藉由使用 SMB，您可以在內部部署或雲端 VM 中，直接將 Azure 檔案共用掛接在 Windows、Linux 或 macOS 上，而不需撰寫任何程式碼，或對檔案系統附加任何特殊的驅動程式。 您也可以使用 Azure 檔案同步，在接近使用資料之處進行快速存取，藉以在內部部署檔案伺服器上快取 Azure 檔案共用。 
   
    如需 Azure 檔案服務和 Azure Blob 儲存體之間差異的更深入說明，請參閱[決定何時使用 Azure Blob 儲存體、Azure 檔案服務或 Azure 磁碟](../common/storage-decide-blobs-files-disks.md?toc=%2fazure%2fstorage%2ffiles%2ftoc.json)。 若要深入了解 Azure Blob 儲存體，請參閱 [Blob 儲存體簡介](../blobs/storage-blobs-introduction.md)。

* <a id="files-versus-disks"></a>**為什麼我要使用 Azure 檔案共用，而不是 Azure 磁碟呢？**  
    Azure 磁碟中的磁碟就只是個磁碟。 若要從 Azure 磁碟取得值，您必須將磁碟連接至 Azure 中執行的虛擬機器。 Azure 磁碟適用於您要在內部部署伺服器上為其使用磁碟的所有項目。 您可以使用它作為 OS 系統磁碟、作為 OS 的交換空間，或者作為應用程式的專用儲存體。 Azure 磁碟的一個有趣用途是在雲端建立檔案伺服器，以便在您可能使用 Azure 檔案共用的相同位置中使用。 當您需要的部署選項 (例如，NFS 通訊協定支援或進階儲存體) 目前不受 Azure 檔案服務支援時，在 Azure 虛擬機器中部署檔案伺服器是在 Azure 中獲得檔案儲存體且具有極高效能的方式。 

    不過，基於數種理由，執行以 Azure 磁碟作為後端儲存體的檔案伺服器一般會比使用 Azure 檔案共用來得更昂貴。 首先，除了要支付磁碟儲存體的費用外，您還必須支付執行一或多個 Azure VM 的費用。 其次，您也必須管理用來執行檔案伺服器的 VM。 例如，您必須負責 OS 升級。 最後，如果您最終需要在內部部署中快取資料，就必須自行設定和管理複寫技術 (例如，分散式檔案系統複寫 (DFSR)) 以實現此目的。

    若要從 Azure 檔案服務和 Azure 虛擬機器中所裝載 (除了使用 Azure 磁碟作為後端儲存體) 的檔案伺服器獲取最大效益，其中一個方法是在雲端 VM 上所裝載的檔案伺服器上安裝 Azure 檔案同步。 如果 Azure 檔案共用位於和您檔案伺服器相同的區域，您可以啟用雲端分層，並將磁碟區可用空間百分比設定為最大值 (99%)。 這確保資料重複出現的機率最低。 您也可以使用任何您想要與檔案伺服器搭配使用的應用程式，例如，需要 NFS 通訊協定支援的應用程式。

    如需在 Azure 中設定高效能和高可用性檔案伺服器的選項相關資訊，請參閱[在 Microsoft Azure 中部署 IaaS VM 客體叢集](https://blogs.msdn.microsoft.com/clustering/2017/02/14/deploying-an-iaas-vm-guest-clusters-in-microsoft-azure/) \(英文\)。 如需 Azure 檔案服務和 Azure 磁碟之間差異的更深入說明，請參閱[決定何時使用 Azure Blob 儲存體、Azure 檔案服務或 Azure 磁碟](../common/storage-decide-blobs-files-disks.md?toc=%2fazure%2fstorage%2ffiles%2ftoc.json)。 若要深入了解 Azure 磁碟，請參閱 [Azure 受控磁碟概觀](../../virtual-machines/windows/managed-disks-overview.md)。

* <a id="get-started"></a>
**如何開始使用 Azure 檔案服務？**  
   Azure 檔案很容易就能開始使用。 首先，[建立檔案共用](storage-how-to-create-file-share.md)，然後在您慣用的作業系統中掛接它： 

    * [在 Windows 中掛接](storage-how-to-use-files-windows.md)
    * [在 Linux 中掛接](storage-how-to-use-files-linux.md)
    * [在 macOS 中掛接](storage-how-to-use-files-mac.md)

   如需更深入了解部署 Azure 檔案共用來取代組織中生產環境檔案共用的指引，請參閱[規劃 Azure 檔案服務部署](storage-files-planning.md)。

* <a id="redundancy-options"></a>
**Azure 檔案服務支援哪些儲存體備援選項？**  
    Azure 檔案服務目前支援本地備援儲存體 (LRS)、區域備援儲存體 (ZRS) 和異地備援儲存體 (GRS)。 我們計劃在未來支援讀取權限異地備援 (RA-GRS) 儲存體，但目前尚無確切時間表。

* <a id="tier-options"></a>
**Azure 檔案服務中支援哪些儲存層？**  
    Azure 檔案服務目前只支援標準儲存層。 對於進階儲存體和非經常性儲存體的支援，目前尚無確切時間表。 
    
    > [!NOTE]
    > 您無法從僅限 Blob 的儲存體帳戶或進階儲存體帳戶建立 Azure 檔案共用。

* <a id="give-us-feedback"></a>
**我真的希望 Azure 檔案服務中能加入某個特定功能。是否有此可能？**  
    Azure 檔案服務小組很樂意聽到您對於我們所提供之服務的所有意見反應。 請在 [Azure 檔案服務 UserVoice](https://feedback.azure.com/forums/217298-storage/category/180670-files) \(英文\) 上投票支持您認同的功能要求！ 我們期待能透過諸多新的功能讓您感到滿意。

## <a name="azure-file-sync"></a>Azure 檔案同步

* <a id="afs-region-availability"></a>
**哪些區域支援 Azure 檔案同步？**  
    如需可用區域的清單，請參閱 Azure 檔案同步規劃指南的[區域可用性](storage-sync-files-planning.md#region-availability)一節。 我們會持續新增其他區域的支援，包括非公用區域。

* <a id="cross-domain-sync"></a>
**相同的同步群組中是否可以同時有已加入網域和未加入網域的伺服器？**  
    是。 同步群組可以包含具有不同 Active Directory 成員資格的伺服器端點，即使它們是未加入網域的端點也一樣。 雖然就技術上來說，這樣的設定是可行的，但我們不建議您將此作為一般設定，因為針對某部伺服器上的檔案和資料夾所定義的存取控制清單 (ACL)，可能無法由同步群組中的其他伺服器強制執行。 為獲得最佳結果，建議您在相同 Active Directory 樹系的伺服器之間、在位於不同 Active Directory 樹系但已建立信任關係的伺服器之間，或是在不在網域中的伺服器之間進行同步。 我們建議您避免混用這些設定。

* <a id="afs-change-detection"></a>
**我直接在 Azure 檔案共用中使用 SMB 建立了檔案，或是在入口網站中建立了檔案。將該檔案與同步群組中的伺服器進行同步，需要花費多少時間？**  
    [!INCLUDE [storage-sync-files-change-detection](../../../includes/storage-sync-files-change-detection.md)]

* <a id="afs-conflict-resolution"></a>**如果同一個檔案近乎同時地在兩部伺服器上進行變更，會發生什麼事？**  
    Azure 檔案同步會使用簡單的衝突解決策略：我們會對已同時在兩部伺服器上變更的檔案，保留這兩個變更。 最新寫入的變更會保留原始檔案名稱。 較舊的檔案則會在名稱後面附加「來源」機器和衝突號碼。 它會遵循此分類法： 
   
    \<FileNameWithoutExtension\>-\<MachineName\>\[-#\].\<ext\>  

    例如，如果 CentralServer 是發生較舊寫入的位置，則 CompanyReport.docx 的第一個衝突會變成 CompanyReport CentralServer.docx。 第二個衝突會命名為 CompanyReport-CentralServer-1.docx。

* <a id="afs-storage-redundancy"></a>
**Azure 檔案同步是否支援異地備援儲存體？**  
    是，Azure 檔案服務同時支援本機備援儲存體 (LRS) 和異地備援儲存體 (GRS)。 如果配對區域之間發生 GRS 容錯移轉，則建議您將新的區域僅視為資料的備份。 Azure 檔案同步不會自動開始同步處理新的主要區域。 

* <a id="sizeondisk-versus-size"></a>
**使用 Azure 檔案同步之後，為什麼檔案的「磁碟大小」屬性不符合「大小」屬性？**  
    Windows 檔案總管會顯示兩個屬性來代表檔案的大小：**大小**和**磁碟大小**。 這些屬性的意義稍有不同。 **大小**代表檔案的完整大小。 **磁碟大小**代表儲存在磁碟上的檔案資料流大小。 這些屬性的值會因為各種不同的原因而不同，例如壓縮、使用重複資料刪除，或是利用 Azure 檔案同步進行雲端分層。如果將檔案分層到 Azure 檔案共用，則磁碟大小會是零，因為檔案資料流會儲存於 Azure 檔案共用中，而不是儲存在磁碟上。 檔案也可能部分分層 (或部分回收)。 在部分分層的檔案中，部分的檔案是在磁碟上。 當應用程式 (例如，多媒體播放程式或壓縮公用程式) 讀取部分檔案時，可能會發生這種情形。 

* <a id="is-my-file-tiered"></a>
**如何判斷檔案是否已分層？**  
    有幾個方法可用來確認是否已將檔案分層到 Azure 檔案共用：
    
   *  **檢查檔案的檔案屬性。**
     若要這樣做，請在檔案上按一下滑鼠右鍵，移至 [詳細資料]，然後向下捲動至 [屬性] 屬性。 分層的檔案具有下列屬性組：     
        
        | 屬性代號 | 屬性 | 定義 |
        |:----------------:|-----------|------------|
        | 具有使用  | 封存 | 指出該檔案應該由備份軟體進行備份。 不論檔案已分層還是完全儲存在磁碟上，一律會設定這個屬性。 |
        | P | 疏鬆檔案 | 指出該檔案是疏鬆檔案。 疏鬆檔案是 NTFS 所提供的特殊化檔案類型，可在磁碟資料流上的檔案大多空白時有效地加以使用。 Azure 檔案同步會使用疏鬆檔案，因為檔案已完全分層或已部分回收。 在完全分層的檔案中，檔案資料流會儲存於雲端。 在部分回收的檔案中，該部分的檔案已經在磁碟上。 如果檔案已完全回收到磁碟，Azure 檔案同步便會將它從疏鬆檔案轉換為一般檔案。 |
        | L | 重新分析點 | 指出該檔案有重新分析點。 重新分析點是可供檔案系統篩選器使用的特殊指標。 Azure 檔案同步會使用重新分析點來為 Azure 檔案同步的檔案系統篩選器 (StorageSync.sys) 定義儲存檔案的雲端位置。 這支援無縫存取。 使用者不需要知道正在使用 Azure 檔案同步，或是如何取得 Azure 檔案共用中檔案的存取權。 當檔案完全回收時，Azure 檔案同步就會從檔案中移除重新分析點。 |
        | O | 離線 | 指出部分或所有檔案的內容並未儲存在磁碟上。 當檔案完全回收時，Azure 檔案同步就會移除此屬性。 |

        ![檔案的 [屬性] 對話方塊，已選取 [詳細資料] 索引標籤](media/storage-files-faq/azure-file-sync-file-attributes.png)
        
        您可以在檔案總管的資料表顯示中新增 [屬性] 欄位，就能看見資料夾中所有檔案的屬性。 若要這樣做，以滑鼠右鍵按一下現有資料行 (例如，**大小**)，選取 [詳細]，然後從下拉式清單中選取 [屬性]。
        
   * **使用 `fsutil` 來檢查檔案的重新分析點。**
       如上述選項所述，已分層的檔案一律已設定重新分析點。 重新分析指標是適用於 Azure 檔案同步檔案系統篩選器 (StorageSync.sys) 的特殊指標。 若要檢查檔案是否有重新分析點，請在提升權限的命令提示字元或 PowerShell 視窗中，執行 `fsutil` 公用程式：
    
        ```PowerShell
        fsutil reparsepoint query <your-file-name>
        ```

        如果檔案有重新分析點，您可以預期會看到**重新分析標記值 : 0x8000001e**。 這個十六進位值是 Azure 檔案同步所擁有的重新分析點值。輸出也會包含重新分析資料，此資料會顯現您的檔案在 Azure 檔案共用中的路徑。

        > [!WARNING]  
        > `fsutil reparsepoint` 公用程式命令也能刪除重新分析點。 除非 Azure 檔案同步工程小組要求您，否則請勿執行此命令。 執行此命令可能導致資料遺失。 

* <a id="afs-recall-file"></a>**我想要使用的檔案已分層。如何將檔案回收到磁碟，以便在本機使用？**  
    將檔案回收到磁碟的最簡單方式就是開啟該檔案。 Azure 檔案同步的檔案系統篩選器 (StorageSync.sys) 會順暢地從 Azure 檔案共用下載檔案，您不必執行任何工作。 對於可以部分讀取的檔案類型 (例如，多媒體或 .zip 檔案)，開啟檔案並不會下載整個檔案。

    您也可以使用 PowerShell 來強制回收檔案。 如果您想要一次回收許多檔案 (例如資料夾內的所有檔案)，便適合使用這種方法。 在 Azure 檔案同步安裝所在的伺服器節點開啟 PowerShell 工作階段，然後執行下列 PowerShell 命令：
    
    ```PowerShell
    Import-Module "C:\Program Files\Azure\StorageSyncAgent\StorageSync.Management.ServerCmdlets.dll"
    Invoke-StorageSyncFileRecall -Path <file-or-directory-to-be-recalled>
    ```

* <a id="afs-force-tiering"></a>
**如何強制讓檔案或目錄分層？**  
    啟用雲端分層功能時，雲端分層會自動根據上次存取和修改時間來將檔案分層，以達到雲端端點上指定的磁碟區可用空間百分比。 不過，有時候您可能會想要以手動方式強制將檔案分層。 如果您要儲存長時間不打算再次使用的大型檔案，並且想要讓磁碟區上的可用空間現在可供其他檔案和資料夾使用，便適合使用這種方法。 您可以使用下列 PowerShell 命令來強制分層：

    ```PowerShell
    Import-Module "C:\Program Files\Azure\StorageSyncAgent\StorageSync.Management.ServerCmdlets.dll"
    Invoke-StorageSyncCloudTiering -Path <file-or-directory-to-be-tiered>
    ```

* <a id="afs-effective-vfs"></a>
**如果磁碟區上有多個伺服器端點，如何解譯「磁碟區可用空間」？**  
    當磁碟區上有一個以上的伺服器端點時，針對有效磁碟區上任何伺服器端點所指定的最大磁碟區可用空間，就是該磁碟區的可用空間臨界值。 檔案將依據其使用方式模式分層，不論其屬於哪一個伺服器端點。 例如，如果磁碟區上有兩個伺服器端點，端點 1 和端點 2，其中端點 1 有 25% 的磁碟區可用空間臨界值，端點 2 有 50% 的磁碟區可用空間臨界值，則這兩個伺服器端點的磁碟區可用空間臨界值就是 50%。

* <a id="afs-files-excluded"></a>
**Azure 檔案同步會自動排除哪些檔案或資料夾？**  
    根據預設，Azure 檔案同步會排除下列檔案：
    * desktop.ini
    * thumbs.db
    * ehthumbs.db
    * ~$\*.\*
    * \*.laccdb
    * \*.tmp
    * 635D02A9D91C401B97884B82B3BCDAEA.\*

    預設也會排除下列資料夾：

    * \System Volume Information
    * \$RECYCLE.BIN
    * \SyncShareState

* <a id="afs-os-support"></a>
**我可以將 Azure 檔案同步與 Windows Server 2008 R2、Linux 或網路連接儲存裝置 (NAS) 搭配使用嗎？**  
    Azure 檔案同步目前只支援 Windows Server 2016 和 Windows Server 2012 R2。 現階段我們沒有其他計劃，但會保持開放態度，並根據客戶需求來支援其他平台。 請前往 [Azure 檔案服務 UserVoice](https://feedback.azure.com/forums/217298-storage/category/180670-files) \(英文\)，讓我們知道您希望我們支援哪些平台。

* <a id="afs-tiered-files-out-of-endpoint"></a>
**分層的檔案為何存在於伺服器端點命名空間以外？**  
    在 Azure 檔案同步代理程式版本 3 之前，Azure 檔案同步會禁止移動位於伺服器端點以外外、但與伺服器端點位於相同磁碟區上的分層檔案。 非分層檔案的複製作業和移動，以及從分層磁碟區到其他磁碟區的移動，均不受影響。 之所以會有此行為，是因為系統隱含地假設在相同磁碟區上執行這些移動作業的檔案總管和其他 Windows API (幾近於) 是 即時的重新命名作業。 這表示，當 Azure 檔案同步回復雲端中的資料時，移動作業將會使檔案總管或其他的移動方法 (例如命令列或 PowerShell) 呈現為無回應的狀態。 從 [Azure 檔案同步代理程式 3.0.12.0 版](storage-files-release-notes.md#agent-version-30120)開始，Azure 檔案同步將可讓您移動伺服器端點以外的分層檔案。 我們讓分層的檔案以分層的形式存在於伺服器端點以外，然後在背景中回復檔案，以避免產生負面影響。 這表示，相同磁碟區上的移動會即時執行，而我們會在移動完成後，再執行所有將檔案回復至磁碟的工作。 

* <a id="afs-do-not-delete-server-endpoint"></a>
**我在伺服器上的 Azure 檔案同步有問題 (同步、雲端分層等)。我是否應移除並重新建立伺服器端點？**  
    [!INCLUDE [storage-sync-files-remove-server-endpoint](../../../includes/storage-sync-files-remove-server-endpoint.md)]
    
* <a id="afs-resource-move"></a>
**是否可以將儲存體同步服務及/或儲存體帳戶移至不同的資源群組或訂用帳戶？**  
   是，可以將儲存體同步服務及/或儲存體帳戶移至不同的資源群組或訂用帳戶。 如果移動儲存體帳戶，您需要將儲存體帳戶的存取權給予混合式檔案同步服務 (請參閱[確保 Azure 檔案同步有儲存體帳戶的存取權](https://docs.microsoft.com/en-us/azure/storage/files/storage-sync-files-troubleshoot?tabs=portal1%2Cportal#troubleshoot-rbac))。

* <a id="afs-ntfs-acls"></a>
**Azure 檔案同步是否會在 Azure 檔案中隨著資料保留目錄/檔案層級 NTFS AC？**

    來自內部部署檔案伺服器的 NTFS ACL 會由 Azure 檔案同步以中繼資料方式保存。 Azure 檔案不支援使用 Azure AD 認證來進行驗證，以存取由 Azure 檔案共用服務所管理的檔案共用。
    
## <a name="security-authentication-and-access-control"></a>安全性、驗證和存取控制
* <a id="ad-support"></a>
**Azure 檔案服務是否支援以 Active Directory 為基礎的驗證和存取控制？**  
    
    是，Azure 檔案支援身分識別型驗證與使用 Azure Active Directory (Azure AD) (預覽) 的存取控制。 針對 Azure 檔案透過 SMB 進行 Azure AD 驗證會利用 Azure Active Directory Domain Services 來讓已加入網域的 VM 使用 Azure AD 認證來存取共用、目錄與檔案。 如需詳細資訊，請參閱[針對 Azure 檔案透過 SMB 進行 Azure AD 驗證的概觀 (預覽)](storage-files-active-directory-overview.md)。 

    Azure 檔案服務提供兩種管理存取控制的額外方式：

    - 您可以使用共用存取簽章 (SAS)，來產生具有特定權限且在指定時間間隔內有效的權杖。 例如，您可以產生具有指定檔案唯讀存取權且會在 10 分鐘到期的權杖。 擁有權杖的任何人，在該權杖的 10 分鐘有效時間內，具有該檔案的唯讀存取權。 目前只能透過 REST API 或在用戶端程式庫中支援共用存取簽章金鑰。 您必須使用儲存體帳戶金鑰，透過 SMB 掛接 Azure 檔案共用。

    - Azure 檔案同步會保留並複製所有判別 ACL 或 DACL (不論是以 Active Directory 為基礎或本機) 到它要同步的所有伺服器端點。 由於 Windows Server 已經能夠利用 Active Directory 進行驗證，因此，在提供 Active Directory 型驗證的完整支援和 ACL 支援之前，Azure 檔案同步是一個有效的權宜選項。

* <a id="ad-support-regions"></a>
**針對 Azure 檔案透過 SMB 進行 Azure AD 驗證的預覽版在所有 Azure 區域是否都能使用？**

    預覽適用於所有公共區域，以下除外：美國西部、美國西部 2、美國中南部、美國東部、美國東部 2、美國中部、美國中北部、澳大利亞東部、西歐、北歐。

* <a id="ad-support-on-premises"></a>
**針對 Azure 檔案透過 SMB 進行 Azure AD 驗證是否支援使用來自內部部署機器的 Azure AD 進行？**

    否，在預覽版中，Azure 檔案不支援使用來自內部部署的 Azure AD 進行驗證。

* <a id="ad-support-devices"></a>
**針對 Azure 檔案透過 SMB 進行 Azure AD 驗證 (預覽) 是否支援使用來自已加入 Azure AD 或已向其註冊之 Azure AD 認證的 SMB 存取？**

    否，不支援此案例。

* <a id="ad-support-rest-apis"></a>
**是否有 REST API 支援取得/設定/複製目錄/檔案 NTFS ACL？**

    預覽版不支援 REST API 來取得、設定或複製目錄或檔案的 NTFS ACL。

* <a id="ad-vm-subscription"></a>
**我是否可以從不同訂用帳戶下的 VM 使用 Azure AD 認證來存取 Azure 檔案？**

    若檔案共用部署所在的訂用帳戶與和部署到 VM 所加入之網域之 Azure AD Domain Services 相同的 Azure AD 租用戶關聯，則您可以使用相同的 Azure AD 認證來存取 Azure 檔案。 不僅訂用帳戶有限制，關聯的 Azure AD 租用戶也有。    
    
* <a id="ad-support-subscription"></a>
**我用來啟用針對 Azure 檔案透過 SMB 進行 Azure AD 驗證的 Azure AD 租用戶是否能與檔案共用所關聯的主要租用戶不同？**

    否，Azure 檔案只支援使用位於與檔案共用之訂用帳戶相同的訂用帳戶的 Azure AD 租用戶來進行 Azure AD 整合。 只有一個訂用帳戶能與 Azure AD 租用戶關聯。

* <a id="ad-linux-vms"></a>
**針對 Azure 檔案透過 SMB 進行 Azure AD 驗證 (預覽) 是否支援 Linux VM？**

    否，預覽版不支援來自 Linux VM 的驗證。

* <a id="ad-aad-smb-afs"></a>
**我是否可以利用由 Azure 檔案同步所管理之共用上的透過 SMB 的 Azure AD 驗證？**

    否，Azure 檔案不支援在由 Azure 所管理的檔案共用上保留 NTFS ACL。來自內部部署檔案伺服器的檔案 ACL 會由 Azure 檔案同步保存。任何針對 Azure 檔案以原生方式設定的 NTFS ACL 都會由 Azure 檔案同步服務覆寫。 此外，Azure 檔案不支援使用 Azure AD 認證來進行驗證，以存取由 Azure 檔案共用服務所管理的檔案共用。

* <a id="encryption-at-rest"></a>
**如何確保我的 Azure 檔案共用會進行待用加密？**  

    Azure 儲存體服務加密預計在所有區域均預設為啟用。 針對這些區域，您不必執行任何動作來啟用加密。 針對其他區域，請參閱[伺服器端加密](../common/storage-service-encryption.md?toc=%2fazure%2fstorage%2ffiles%2ftoc.json)。

* <a id="access-via-browser"></a>
**如何使用網頁瀏覽器來提供對特定檔案的存取權？**  

    您可以使用共用存取簽章，來產生具有特定權限且在指定時間間隔內有效的權杖。 例如，您可以產生可在一段特定時間內提供特定檔案唯讀存取權限的權杖。 任何人只要擁有此 URL，就可以在權杖有效時間內，直接從任何網頁瀏覽器存取檔案。 您可以從類似儲存體總管的 UI，輕易產生共用存取簽章金鑰。

* <a id="file-level-permissions"></a>
**可以對共用內的資料夾指定唯讀或唯寫權限嗎？**  

    如果您使用 SMB 來掛接檔案共用，則您對權限沒有資料夾層級的控制。 不過，如果您使用 REST API 或用戶端程式庫建立共用存取簽章，就可以在共用中的資料夾上指定唯讀或唯寫權限。

* <a id="ip-restrictions"></a>
**我是否可以對 Azure 檔案共用實作 IP 限制？**  

    是。 您可以在儲存體帳戶層級限制對 Azure 檔案共用的存取。 如需詳細資訊，請參閱[設定 Azure 儲存體防火牆和虛擬網路](../common/storage-network-security.md?toc=%2fazure%2fstorage%2ffiles%2ftoc.json)。

* <a id="data-compliance-policies"></a>
**Azure 檔案服務支援哪些資料合規性原則？**  

   Azure 檔案服務和 Azure 儲存體的其他儲存體中所使用的服務都是在相同的儲存體架構上運作。 Azure 檔案服務會套用其他 Azure 儲存體服務中所使用的相同資料合規性原則。 如需 Azure 儲存體資料合規性的詳細資訊，您可以參閱 [Azure 儲存體合規性供應項目](https://docs.microsoft.com/en-us/azure/storage/common/storage-compliance-offerings)，並前往 [Microsoft 信任中心](https://microsoft.com/en-us/trustcenter/default.aspx)。

## <a name="on-premises-access"></a>內部部署存取
* <a id="expressroute-not-required"></a>
**我必須使用 Azure ExpressRoute 來連線到 Azure 檔案服務，還是必須在內部部署中使用 Azure 檔案同步？**  

    否。 不需要 ExpressRoute 就能存取 Azure 檔案共用。 如果您要直接在內部部署掛接 Azure 檔案共用，只需開啟連接埠 445 (TCP 輸出) 以進行網際網路存取 (這是 SMB 用來進行通訊的連接埠)。 如果您使用 Azure 檔案同步，只需連接埠 443 (TCP 輸出) 以進行 HTTPS 存取 (不需要 SMB)。 不過，您可以將 ExpressRoute 與這些其中一個選項搭配使用。

* <a id="mount-locally"></a>
**如何在本機電腦上掛接 Azure 檔案共用？**  

    如果已開啟連接埠 445 (TCP 輸出) 且您的用戶端支援 SMB 3.0 通訊協定 (例如，若您使用的是 Windows 10 或 Windows Server 2016)，即可使用 SMB 通訊協定掛接檔案共用。 如果組織的原則或您的 ISP 會封鎖連接埠 445，您可以使用 Azure 檔案同步來存取 Azure 檔案共用。

## <a name="backup"></a>Backup 
* <a id="backup-share"></a>
**如何備份我的 Azure 檔案共用？**  
    您可以使用定期[共用快照集](storage-snapshots-files.md)來防範意外刪除的情況。 您也可以使用 AzCopy、RoboCopy，或是可備份已掛接檔案共用的協力廠商備份工具。 Azure 備份會提供 Azure 檔案的備份。 深入了解如何[使用 Azure 備份來備份 Azure 檔案共用](https://docs.microsoft.com/en-us/azure/backup/backup-azure-files)。

## <a name="share-snapshots"></a>共用快照集

### <a name="share-snapshots-general"></a>共用快照集：一般
* <a id="what-are-snaphots"></a>
**檔案共用快照集有哪些？**  
    您可以使用 Azure 檔案共用快照集，來建立檔案共用的唯讀版本。 您也可以使用 Azure 檔案服務，將較舊版本的內容複製回到相同的共用或是 Azure 或內部部署的替代位置，以進行更多修改。 若要深入了解共用快照集，請參閱[共用快照集概觀](storage-snapshots-files.md)。

* <a id="where-are-snapshots-stored"></a>
**共用快照集會儲存在哪裡？**  
    共用快照集會儲存在與檔案共用相同的儲存體帳戶中。

* <a id="snapshot-perf-impact"></a>
**使用共用快照集是否會有任何效能影響？**  
    共用快照集並沒有任何效能負擔。

* <a id="snapshot-consistency"></a>
**共用快照集是否具有應用程式一致性？**  
    否，共用快照集不具應用程式一致性。 使用者必須先將寫入從應用程式排清到共用，然後再擷取共用快照集。

* <a id="snapshot-limits"></a>
**我可以使用的共用快照集數目是否有限制？**  
    是。 Azure 檔案服務最多可保留 200 個共用快照集。 共用快照集不會計入共用配額，因此，所有共用快照集所使用的總空間沒有每個共用的限制。 但儲存體帳戶限制依然有效。 保留 200 個共用快照集之後，必須先刪除舊的快照集，才能建立新的共用快照集。

* <a id="snapshot-cost"></a>
**共用快照集需要多少成本？**  
    使用快照集的費用包括標準交易成本和標準儲存體的費用。 快照集具有累加性質。 基底快照集便是共用本身。 所有後續的快照集都是累加的，而且只會儲存與上一個快照集相異之處。 這表示如果您的工作負載變換程度不高，帳單中列示的差異變更也不會太多。 如果需要標準 Azure 檔案的定價資訊，請參閱[定價頁面](https://azure.microsoft.com/pricing/details/storage/files/)。 現在判斷共用快照集耗用大小的方式，是比較計費的容量與已使用的容量。 目前正在設法開發改善報告功能的工具。

* <a id="ntfs-acls-snaphsots"></a>
**目錄與檔案上的 NTFS ACL 在共用快照中是否會保留下來？**
    目錄與檔案上的 NTFS ACL 在共用快照中會保留下來。

### <a name="create-share-snapshots"></a>建立共用快照集
* <a id="file-snaphsots"></a>
**我可以建立個別檔案的共用快照集嗎？**  
    共用快照集是在檔案共用層級建立的。 您可以從檔案共用快照集還原個別檔案，但您無法建立檔案層級的共用快照集。 不過，如果您已擷取共用層級的共用快照集，而且想要列出已變更特定檔案的共用快照集，就可以在 Windows 所掛接之共用上的 [舊版] 下方執行此動作。 
    
    如果您需要檔案快照集功能，請前往 [Azure 檔案服務 UserVoice](https://feedback.azure.com/forums/217298-storage/category/180670-files) \(英文\)，讓我們知道。

* <a id="encypted-snapshots"></a>
**我可以建立已加密檔案共用的共用快照集嗎？**  
    您可以對已啟用待用加密的 Azure 檔案共用擷取共用快照集。 您可以將共用快照集內的檔案還原至已加密的檔案共用。 如果您的共用已加密，則共用快照集也會加密。

* <a id="geo-redundant-snaphsots"></a>
**我的共用快照集是否會異地備援？**  
    共用快照集擁有和目標 Azure 檔案共用相同的備援。 如果您為帳戶選取了異地備援儲存體，則共用快照集也會在配對區域中進行備援儲存。

### <a name="manage-share-snapshots"></a>管理共用快照集
* <a id="browse-snapshots-linux"></a>
**我可以從 Linux 瀏覽共用快照集嗎？**  
    您可以使用 Azure CLI，在 Linux 中建立、列出、瀏覽及還原共用快照集。

* <a id="copy-snapshots-to-other-storage-account"></a>
**我可以將共用快照集複製到不同的儲存體帳戶嗎？**  
    您可以將共用快照集的檔案複製到其他位置，但無法複製共用快照集本身。

### <a name="restore-data-from-share-snapshots"></a>從共用快照集還原資料
* <a id="promote-share-snapshot"></a>
**我可以將共用快照集升階到基底共用嗎？**  
    您可以將共用快照集的資料複製到任何其他目的地。 您無法將共用快照集升階到基底共用。

* <a id="restore-snapshotted-file-to-other-share"></a>
**我可以將共用快照集的資料還原到不同的儲存體帳戶嗎？**  
    是。 您可以將共用快照集的檔案複製到原始位置或替代位置，其中包含相同區域或不同區域中的同一個儲存體帳戶或不同的儲存體帳戶。 您也可以將檔案複製到內部部署位置或任何其他雲端。    
  
### <a name="clean-up-share-snapshots"></a>清除共用快照集
* <a id="delete-share-keep-snapshots"></a>
**我是否可以刪除共用，而不刪除共用快照集？**  
    如果您的共用上有使用中的共用快照集，就無法刪除共用。 您可以使用 API 來刪除共用快照集以及該共用。 您也可以在 Azure 入口網站中同時刪除共用快照集和共用。

* <a id="delete-share-with-snapshots"></a>
**如果我刪除儲存體帳戶，共用快照集會發生什麼情況？**  
    如果您刪除儲存體帳戶，也會一併刪除共用快照集。

## <a name="billing-and-pricing"></a>計費和定價
* <a id="vm-file-share-network-traffic"></a>
**Azure VM 和 Azure 檔案共用之間的網路流量，會計算為向訂用帳戶收費的外部頻寬嗎？**  
    如果檔案共用和 VM 位於相同的 Azure 區域，就不會針對檔案共用和 VM 之間的流量收取額外費用。 如果檔案共用和 VM 位於不同的區域，則兩者之間的流量會以外部頻寬收費。

* <a id="share-snapshot-price"></a>
**共用快照集需要多少成本？**  
     在預覽期間，共用快照集容量是免費的。 適用標準儲存體的輸出和交易成本。 正式運作之後，將會針對共用快照集上的容量和交易來向訂用帳戶收費。
     
     共用快照集具有累加性質。 基底共用快照集便是共用本身。 所有後續的共用快照集都是累加的，而且只會儲存與上一個共用快照集相異之處。 變更的內容才會計費。 如果您的共用有 100 GiB 資料，但在上一個共用快照集之後只變更了 5 GiB，則共用快照集只會再耗用 5 GiB，並以 105 GiB 來計費。 如需交易和標準輸出費用的詳細資訊，請參閱[定價頁面](https://azure.microsoft.com/pricing/details/storage/files/)。

## <a name="scale-and-performance"></a>規模和效能
* <a id="files-scale-limits"></a>
**Azure 檔案服務有何規模限制？**  
    如需 Azure 檔案服務的延展性和效能目標相關資訊，請參閱 [Azure 檔案服務延展性和效能目標](storage-files-scale-targets.md)。

* <a id="need-larger-share"></a>
**我需要的檔案共用比 Azure 檔案服務目前所提供的更大。我可以提高 Azure 檔案共用的大小嗎？**  
    否。 Azure 檔案共用的大小上限是 5 TiB。 這是目前的固定限制，我們無法調整。 我們正在研究將共用大小提高到 100 TiB 的解決方案，但目前沒有時間表。

* <a id="open-handles-quota"></a>
**多少個用戶端可以同時存取相同的檔案？**   
    單一檔案的開啟控制代碼配額為 2,000 個。 當您擁有 2,000 個開啟控制代碼時，會顯示一則錯誤訊息以指出已達到配額。

* <a id="zip-slow-performance"></a>
**當我在 Azure 檔案服務中將檔案解壓縮時，效能變慢了。我該怎麼辦？**  
    若要將大量檔案傳輸到 Azure 檔案服務，建議您使用 AzCopy (適用於 Windows；針對 Linux/Unix 則是在預覽版中) 或 Azure PowerShell。 這些工具已針對網路傳輸最佳化。

* <a id="slow-perf-windows-81-2012r2"></a>
**當我在 Windows Server 2012 R2 或 Windows 8.1 上掛接 Azure 檔案共用之後，為什麼我的效能變慢了？**  
    在 Windows Server 2012 R2 和 Windows 8.1 上掛接 Azure 檔案共用時，有一個已知的問題。 此問題已在 Windows 8.1 和 Windows Server 2012 R2 的 2014 年 4 月累計更新中加以修補。 若要取得最佳效能，請確定 Windows Server 2012 R2 和 Windows 8.1 的所有執行個體都已套用這個修補程式 (您應該一律會透過 Windows Update 接收到 Windows 修補程式)。如需詳細資訊，請參閱相關的 Microsoft 知識庫文章：[當您從 Windows 8.1 或 Server 2012 R2 存取 Azure 檔案服務時效能變慢](https://support.microsoft.com/kb/3114025) \(機器翻譯\)。

## <a name="features-and-interoperability-with-other-services"></a>功能及與其他服務的互通性
* <a id="cluster-witness"></a>
**我可以使用 Azure 檔案共用作為 Windows Server 容錯移轉叢集的「檔案共用見證」嗎？**  
    Azure 檔案共用目前不支援此設定。 如需如何為 Azure Blob 儲存體設定此項的詳細資訊，請參閱[為容錯移轉叢集部署雲端見證](https://docs.microsoft.com/windows-server/failover-clustering/deploy-cloud-witness) \(機器翻譯\)。

* <a id="containers"></a>
**我是否可以在 Azure 容器執行個體上掛接 Azure 檔案共用？**  
    是，當您想要保存超過容器執行個體存留期的資訊時，Azure 檔案共用是一個絕佳的選項。 如需詳細資訊，請參閱[使用 Azure 容器執行個體來掛接 Azure 檔案共用](../../container-instances/container-instances-mounting-azure-files-volume.md)。

* <a id="rest-rename"></a>
**REST API 中是否有重新命名作業？**  
    目前沒有。

* <a id="nested-shares"></a>
**我可以設定巢狀共用嗎？也就是說，共用下的共用？**  
    否。 檔案共用是您可以掛接的虛擬驅動程式，因此不支援巢狀共用。

* <a id="ibm-mq"></a>
**如何將 Azure 檔案服務與 IBM MQ 搭配使用？**  
    IBM 已發行文件來協助 IBM MQ 客戶，利用 IBM 服務來設定 Azure 檔案服務。 如需詳細資訊，請參閱[如何使用 Microsoft Azure 檔案服務來設定 IBM MQ 多重執行個體佇列管理員](https://github.com/ibm-messaging/mq-azure/wiki/How-to-setup-IBM-MQ-Multi-instance-queue-manager-with-Microsoft-Azure-File-Service) \(英文\)。

## <a name="see-also"></a>另請參閱
* [針對 Windows 中的 Azure 檔案服務進行疑難排解](storage-troubleshoot-windows-file-connection-problems.md)
* [針對 Linux 中的 Azure 檔案服務進行疑難排解](storage-troubleshoot-linux-file-connection-problems.md)
* [針對 Azure 檔案同步進行移難排解](storage-sync-files-troubleshoot.md)
