---
title: 將資料從 StorSimple 5000-7000 系列移轉至 Azure 檔案同步 | Microsoft Docs
description: 說明如何將資料從 StorSimple 5000/7000 系列移轉至 Azure 檔案同步 (AFS)。
services: storsimple
documentationcenter: NA
author: alkohli
manager: twooley
ms.assetid: ''
ms.service: storsimple
ms.devlang: NA
ms.topic: article
ms.tgt_pltfrm: NA
ms.workload: NA
ms.date: 08/23/2018
ms.author: alkohli
ms.openlocfilehash: d54394a8dc74d04f1922a78826b10a4a421d193b
ms.sourcegitcommit: 58c5cd866ade5aac4354ea1fe8705cee2b50ba9f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/24/2018
ms.locfileid: "42814585"
---
# <a name="migrate-data-from-storsimple-5000-7000-series-to-azure-file-sync"></a>將資料從 StorSimple 5000-7000 系列移轉至 Azure 檔案同步

> [!IMPORTANT]
> StorSimple 5000/7000 系列將於 2019 年 7 月 31 日到達結束支援 (EOS) 狀態。 建議 StorSimple 5000/7000 系列的客戶遷移至文件中所述的其中一個替代方案。

資料移轉是將資料從一個儲存體位置移至另一個的程序。 為此，必須要為組織目前的資料建立從一個裝置到另一個裝置的確切複本 (最好不會中斷或停用作用中的應用程式)，然後將所有輸入/輸出 (I/O) 活動重新導向至新的裝置。 

StorSimple 5000 和 7000 系列儲存體裝置將在 2019 年 7 月終止服務。 這意味著在 2019 年 7 月之後，Microsoft 將無法再提供 StorSimple 5000/7000 系列的軟硬體支援。 使用這些裝置的客戶應將其 StorSimple 資料移轉至 Azure 上的其他混合式儲存體解決方案。 本文說明如何將資料從 StorSimple 5000/7000 系列裝置移轉至 Azure 檔案同步 (AFS)。

## <a name="intended-audience"></a>目標對象

本文的適用對象為負責在資料中心部署和管理 StorSimple 5000/7000 系列裝置的資訊技術 (IT) 專業人員和知識工作者。 使用 StorSimple 裝置處理檔案伺服器工作負載 (使用 Windows Server) 的客戶，會發現此移轉路徑的特出之處。 如果您認為 Azure 檔案同步的功能適用於您的組織，則本文章將可協助您了解如何從 StorSimple 移轉至這些解決方案。

## <a name="migration-considerations"></a>移轉考量

此程序適用於已使用 StorSimple 磁碟區設定 Windows 檔案共用以進行儲存的客戶。 要將資料從 StorSimple 5000/7000 移轉至 Azure 檔案同步，必須將該檔案共用位置轉換為[伺服器端點](https://docs.microsoft.com/azure/storage/files/storage-sync-files-planning)，然後使用本機連接的磁碟機作為後續將成為新位置的另一個端點。 

移轉至 AFS 時，應考量以下幾點：

1. Azure 檔案服務目前有每一共用 5 TB 的限制。 您可以使用 Azure 檔案同步將資料分散到多個 Azure 檔案共用，以克服這項限制。 如需詳細資訊，請檢閱 [Azure 檔案服務部署的資料成長模式](https://docs.microsoft.com/azure/storage/files/storage-files-planning)。
2. 從內部部署裝置執行的複製資料完成後，此移轉會將整個主要資料集下載到內部部署裝置。 請確定您有足夠的頻寬可支應此傳輸。
3. 此程序不會保留已建立的快照集。 它只會移轉主要資料。 此程序也不會保留相關聯的頻寬範本或備份原則。 在資料移轉至 Azure 檔案共用後，請[使用 Azure 備份](https://docs.microsoft.com/azure/backup/backup-azure-files)來設定備份原則。
4. StorSimple 會提供第一方硬體。 但在使用 Azure 檔案服務/Azure 檔案同步時，您應使用自己的本機 Windows Server 硬體作為本機快取。 您必須確定有足夠的儲存體容量可將您選擇的資料集保存在本機上。 如需關於分層處理和設定所需可用空間目標的詳細資訊，請檢閱如何[在部署 Azure 檔案同步時建立伺服器端點](https://docs.microsoft.com/azure/storage/files/storage-sync-files-deployment-guide?tabs=portal)。 
5. 請檢閱 [Azure 檔案同步的定價](https://azure.microsoft.com/pricing/details/storage/files/)，因為其價格與 StorSimple 不同。 AFS 並不像 StorSimple 一樣具有重複資料刪除和壓縮功能。

## <a name="migration-prerequisites"></a>移轉必要條件

以下您會看到從舊版 5000 或 7000 系列裝置移轉至 Azure 檔案同步的必要條件。開始之前，請確定您︰

- 可以存取目前執行軟體版本為 v2.1.1.518 或更新版本的 StorSimple 5000/7000 系列裝置。 不支援舊版。 StorSimple 裝置的 Web UI 右上角應會顯示正在執行的軟體版本。  
    如果您的裝置並非執行 v2.1.1.518，請將系統升級至必要的最低版本。 如需詳細說明，請參閱[將系統升級至 v2.1.1.518](http://onlinehelp.storsimple.com/111_Appliance/6_System_Upgrade_Guides/Current_(v2.1.1)/000_Software_Patch_Upgrade_Guide_v2.1.1.518)。
- 檢查來源裝置上是否在執行任何作用中的備份作業。 StorSimple 資料保護主控台主機上的作業也包括在內。 請等候目前的工作完成。 
- 可存取已連線至 StorSimple 5000-7000 系列裝置的 Windows Server 主機。 此主機必須執行支援的 Windows Server 版本，如 [Azure 檔案同步互通性](https://docs.microsoft.com/azure/storage/files/storage-sync-files-planning)所說明。
- 在主機上掛接 StorSimple 磁碟區，並包含檔案共用。
- 主機有足夠的本機儲存體可容納您在本機快取的資料。
- 可對將用來部署 Azure 檔案同步的 Azure 訂用帳戶進行擁有者層級存取。如果您沒有擁有者或管理員層級權限，您在建立同步群組的雲端端點時可能會發生問題。
- 可存取[一般用途 v2 儲存體帳戶](https://docs.microsoft.com/azure/storage/common/storage-account-options) (內含要作為同步目標的 Azure 檔案共用)。 如需詳細資訊，請移至 
 - 如何[建立一般用途 v2 儲存體帳戶](https://docs.microsoft.com/azure/storage/common/storage-quickstart-create-account?tabs=portal#create-a-general-purpose-storage-account)。
 - 如何[建立 Azure 檔案共用](https://docs.microsoft.com/azure/storage/files/storage-how-to-create-file-share#create-file-share-through-the-azure-portal)。

## <a name="migration-process"></a>移轉程序

將資料從 StorSimple 5000-7000 移轉至 AFS 是兩步驟的程序：
1.  複寫資料從掛接 StorSimple 磁碟區的內部部署檔案伺服器複寫至 Azure 檔案共用。  複寫會透過您所安裝的 AFS 代理程式來完成。
2.  中斷 StorSimple 裝置的連線。 本機磁碟會隨即成為本機快取。

### <a name="migration-steps"></a>移轉步驟

執行下列步驟，將設定於 StorSimple 磁碟區的 Windows 檔案共用移轉至 Azure 檔案同步共用。 
1.  在掛接 StorSimple 磁碟區的相同 Windows Server 主機上執行下列步驟，或以不同的系統執行。 
    - [準備 Windows Server 以搭配 Azure 檔案同步使用](https://docs.microsoft.com/azure/storage/files/storage-sync-files-deployment-guide#prepare-windows-server-to-use-with-azure-file-sync)。
    - [安裝 Azure 檔案同步代理程式](https://docs.microsoft.com/azure/storage/files/storage-sync-files-deployment-guide#install-the-azure-file-sync-agent)。
    - [部署儲存體同步服務](https://docs.microsoft.com/azure/storage/files/storage-sync-files-deployment-guide#deploy-the-storage-sync-service)。 
    - [向儲存體同步服務註冊 Windows Server](https://docs.microsoft.com/azure/storage/files/storage-sync-files-deployment-guide#register-windows-server-with-storage-sync-service)。 
    - [建立同步群組和雲端端點](https://docs.microsoft.com/azure/storage/files/storage-sync-files-deployment-guide#create-a-sync-group-and-a-cloud-endpoint)。 對於每個需要從主機移轉的 Windows 檔案共用，都必須建立同步群組。
    - [建立伺服器端點](https://docs.microsoft.com/azure/storage/files/storage-sync-files-deployment-guide?tabs=portal#create-a-server-endpoint)。 請將路徑指定為您的檔案共用資料所在的 StorSimple 磁碟區路徑。 例如，如果 StorSimple 磁碟區是磁碟機 `J`，而您的資料位於 `J:/<myafsshare>` 中，則應將此路徑新增為伺服器端點。 將 [分層] 保留為 [已停用]。
2.  等候檔案伺服器同步完成。 針對給定同步群組中的每個伺服器，請確定：
    - 上傳和下載的「上次嘗試的同步」時間戳記都是最新的。
    - 上傳和下載的狀態都是綠色的。
    - [同步活動] 中僅顯示少許或沒有待同步的檔案。
    - 上傳和下載的 [檔案無法同步] 皆為 0。
    如需關於伺服器同步何時完成的詳細資訊，請移至[對 Azure 檔案同步進行疑難排解](https://docs.microsoft.com/azure/storage/files/storage-sync-files-troubleshoot?tabs=portal1%2Cportal#how-do-i-know-if-my-servers-are-in-sync-with-each-other)。同步可能需要數小時到數天的時間，視資料大小和頻寬而定。 同步完成後，您所有的資料都會安全地放在 Azure 檔案共用中。 
3.  移至 StorSimple 磁碟區上的共用。 選取共用、按一下滑鼠右鍵，然後選取 [屬性]。 請記下 [安全性] 下方的共用權限。 在稍後的步驟中，必須手動將這些權限套用到新的共用。
4.  根據您是使用相同的 Windows Server 主機還是不同的主機，後續步驟將會不同。

    如果您使用不同的 Windows Server 主機，請略過此步驟並移至下一個步驟。 如果您將相同的 Windows 檔案伺服器用於 AFS，此時將會出現幾分鐘的停機時間。 
    - **停機時間開始** - 刪除您在*步驟 1F* 中建立的伺服器端點。 
    - 使用您後續要用來存放資料的路徑，建立新的伺服器端點。
    - 當伺服器端點呈現為「狀況良好」時 (這可能需要幾分鐘的時間)，您會在這個新的位置中看到資料。 現在，您可以設定 Windows Server 主機以從這個新位置提供檔案。 - **停機時間結束**。
5.  如果您將其他 Windows 檔案伺服器用於 Azure 檔案同步，則不會出現任何停機時間。 
    - 使用您準備要替代 StorSimple 裝置作為快取的本機儲存體所使用的路徑，新增另一個伺服器端點。 
    - 幾分鐘後您即可在新的伺服器中看到檔案。 您可以隨時從 StorSimple 裝置切換到主機上的這個新位置。

    > [!TIP] 
    > 請考慮以與原先相同的名稱和路徑設定這個新的檔案共用，以盡可能避免混淆。 如果使用 DFS-N，您可能必須在組態中進行變更。
6.  重新設定在*步驟 3* 中記下的共用權限。

如果您在資料移轉期間遇到任何問題，請[連絡 Microsoft 支援服務](storsimple-8000-contact-microsoft-support.md)。 



## <a name="next-steps"></a>後續步驟

如果 AFS 不是您所適用的解決方案，請了解如何[將資料從 StorSimple 5000-7000 系列移轉至 8000 系列裝置](storsimple-8000-migrate-from-5000-7000.md)。

