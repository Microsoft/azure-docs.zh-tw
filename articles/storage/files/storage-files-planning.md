---
title: 規劃 Azure 檔案服務部署 | Microsoft Docs
description: 了解規劃 Azure 檔案服務部署時的考量事項。
services: storage
author: wmgries
ms.service: storage
ms.topic: article
ms.date: 06/12/2018
ms.author: wgries
ms.component: files
ms.openlocfilehash: 19adbbfc456303b471251c28cd984d1676786b19
ms.sourcegitcommit: e2348a7a40dc352677ae0d7e4096540b47704374
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/05/2018
ms.locfileid: "43783146"
---
# <a name="planning-for-an-azure-files-deployment"></a>規劃 Azure 檔案服務部署
[Azure 檔案服務](storage-files-introduction.md)可提供在雲端中完全受控的檔案共用，可透過業界標準 SMB 通訊協定加以存取。 因為 Azure 檔案服務受到完整管理，所以部署於生產環境案例遠易於部署及管理檔案伺服器或 NAS 裝置。 針對在組織中部署生產環境使用的 Azure 檔案共用，本文說明應考慮的主題。

## <a name="management-concepts"></a>管理概念
 下圖說明 Azure 檔案服務管理建構：

![檔案結構](./media/storage-files-introduction/files-concepts.png)

* **儲存體帳戶**：一律透過儲存體帳戶來存取 Azure 儲存體。 如需關於儲存體帳戶容量的詳細資訊，請參閱[延展性和效能目標](../common/storage-scalability-targets.md?toc=%2fazure%2fstorage%2ffiles%2ftoc.json)。

* **共用**：檔案儲存體共用是 Azure 中的 SMB 檔案共用。 所有的目錄和檔案必須在上層共用中建立。 帳戶可包含無限數目的共用，而共用可儲存無限數目的檔案，最多可達檔案共用的 5 TiB 總容量。

* **目錄**：選擇性的目錄階層。

* **檔案**：共用中的檔案。 檔案的大小可高達 1 TiB。

* **URL 格式**：若是透過檔案 REST 通訊協定以進行 Azure 檔案共用的要求，可以使用下列 URL 格式來定址檔案：

    ```
    https://<storage account>.file.core.windows.net/<share>/<directory>/directories>/<file>
    ```

## <a name="data-access-method"></a>資料存取方法
Azure 檔案服務提供兩種方便的內建資料存取方法，您可以個別或結合使用來存取資料：

1. **雲端直接存取**：利用業界標準伺服器訊息區 (SMB) 通訊協定或透過檔案 REST API，[Windows](storage-how-to-use-files-windows.md)、[macOS](storage-how-to-use-files-mac.md) 及/或 [Linux](storage-how-to-use-files-linux.md) 可掛接任何 Azure 檔案共用。 共用上的檔案藉由 SMB 讀取和寫入時，會直接在 Azure 中的檔案共用上進行。 若要由 Azure 中的 VM 掛接，作業系統中的 SMB 用戶端必須至少支援 SMB 2.1。 若要掛接在內部 (例如使用者的工作站上)，工作站所支援的 SMB 用戶端必須至少支援 SMB 3.0 (含加密)。 除了 SMB 之外，新的應用程式或服務可以透過檔案 REST 直接存取檔案共用，為軟體開發提供了方便且可擴充的應用程式開發介面。
2. **Azure 檔案同步**：藉由 Azure 檔案同步，共用可以複寫到內部部署或 Azure 中的 Windows Server。 您的使用者可透過 Windows Server 存取檔案共用，像是透過 SMB 或 NFS 共用。 這適用於從遠離 Azure 資料中心的位置 (例如分公司) 存取和修改資料的情況。 資料可以在多個 Windows Server 端點之間複寫，例如多個分公司之間。 最後，資料可能會分層至 Azure 檔案服務，因此透過伺服器仍然可以存取所有資料，但伺服器並沒有資料的完整複本。 資料實際上是在使用者開啟時順暢地取回。

下表說明使用者和應用程式如何存取您的 Azure 檔案共用：

| | 雲端直接存取 | Azure 檔案同步 |
|------------------------|------------|-----------------|
| 您需要使用哪些通訊協定？ | Azure 檔案服務支援 SMB 2.1、SMB 3.0 和檔案 REST API。 | 透過 Windows Server 上任何支援的通訊協定 (SMB、NFS、FTPS 等) 存取您的 Azure 檔案共用 |  
| 您會在何處執行工作負載？ | **Azure 中**：Azure 檔案服務提供對資料的直接存取。 | **低速網路的內部部署**：Windows、Linux 及 macOS 用戶端可掛接區域內部部署的 Windows 檔案共用，作為 Azure 檔案共用的快速快取。 |
| 您需要何種 ACL 層級？ | 共用和檔案層級。 | 共用、檔案和使用者層級。 |

## <a name="data-security"></a>資料安全性
Azure 檔案服務具有數個內建的選項，可用於確保資料安全性：

* 支援兩種網路通訊協定的加密：SMB 3.0 加密和經由 HTTPS 的檔案 REST。 依照預設： 
    * 支援 SMB 3.0 加密的用戶端可透過加密通道來傳送及接收資料。
    * 不支援 SMB 3.0 的用戶端可以透過 SMB 2.1 或 SMB 3.0 進行無加密的資料中心內部通訊。 請注意，不允許用戶端透過 SMB 2.1 或 SMB 3.0 進行無加密的資料中心之間通訊。
    * 用戶端可以藉由 HTTP 或 HTTPS 透過檔案 REST 進行通訊。
* 待用加密 ([Azure 儲存體服務加密](../common/storage-service-encryption.md?toc=%2fazure%2fstorage%2ffiles%2ftoc.json))：為所有儲存體帳戶啟用「儲存體服務加密」(SSE)。 靜止資料是使用完全受控金鑰加密。 待用加密不會增加儲存成本或降低效能。 
* 加密傳輸資料的選擇性需求：選取時，Azure 檔案服務拒絕透過未加密的通道存取資料。 具體來說，只允許具有加密連線的 HTTPS 和 SMB 3.0。 

    > [!Important]  
    > 要求資料安全傳輸將導致舊版 SMB 用戶端通訊失敗，因為它無法與加密的 SMB 3.0 通訊。 如需詳細資訊，請參閱[掛接在 Windows 上](storage-how-to-use-files-windows.md)、[掛接在 Linux 上](storage-how-to-use-files-linux.md)、[掛接在 macOS 上](storage-how-to-use-files-mac.md)。

為了有最高安全性，當您使用新式用戶端來存取資料時，強烈建議您一律啟用加密待用資料與加密傳輸資料。 例如，如果您需要在僅支援 SMB 2.1 的 Windows Server 2008 R2 VM 上掛接共用，因為 SMB 2.1 不支持加密，因此儲存體帳戶必須允許未加密的流量。

如果您使用 Azure 檔案同步來存取 Azure 檔案共用，則無論是否需要對待用資料進行加密，我們一律會使用加密的 HTTPS 和 SMB 3.0 將資料同步處理至 Windows Server。

## <a name="data-redundancy"></a>資料備援
Azure 檔案服務支援三個資料備援選項：本地備援儲存體 (LRS)、區域備援儲存體 (ZRS) 和異地備援儲存體 (GRS)。 下列各節說明不同備援選項之間的差異：

### <a name="locally-redundant-storage"></a>本地備援儲存體
[!INCLUDE [storage-common-redundancy-LRS](../../../includes/storage-common-redundancy-LRS.md)]

### <a name="zone-redundant-storage"></a>區域備援儲存體
[!INCLUDE [storage-common-redundancy-ZRS](../../../includes/storage-common-redundancy-ZRS.md)]

### <a name="geo-redundant-storage"></a>異地備援儲存體
[!INCLUDE [storage-common-redundancy-GRS](../../../includes/storage-common-redundancy-GRS.md)]

## <a name="data-growth-pattern"></a>資料成長模式
目前，Azure 檔案共用的大小上限是 5 TiB。 由於目前的此一限制，當您在部署 Azure 檔案共用時，必須考量預期的資料成長。 請注意，Azure 儲存體帳戶可以儲存多個共用，所有共用總計為 500 TiB。

使用 Azure 檔案同步可以將多個 Azure 檔案共用同步處理到單一 Windows 檔案伺服器。這可確保內部部署上較舊的超大檔案共用可以帶入 Azure 檔案同步。如需詳細資訊，請參閱[規劃 Azure 檔案同步部署](storage-files-planning.md)。

## <a name="data-transfer-method"></a>資料傳輸方法
有許多簡單的選項可從現有檔案共用 (例如內部部署檔案共用) 大量傳輸資料到 Azure 檔案服務。 部分常用方法包括 (非完整清單)：

* **Azure 檔案同步**：在 Azure 檔案共用 (雲端端點) 和 Windows 目錄命名空間 (伺服器端點) 之間的初次同步處理中，Azure 檔案同步會從現有檔案共用將所有資料複寫至 Azure 檔案服務。
* **[Azure 匯入/匯出](../common/storage-import-export-service.md?toc=%2fazure%2fstorage%2ffiles%2ftoc.json)**：Azure 匯入/匯出服務可讓您將硬碟寄送到 Azure 資料中心，以便將大量資料安全地傳入 Azure 檔案共用。 
* **[Robocopy](https://technet.microsoft.com/library/cc733145.aspx)**：Robocopy 是知名的複製工具，隨附於 Windows 和 Windows Server。 Robocopy 可在本機掛接檔案共用，然後將掛接位置作為 Robocopy 命令中的目的地使用，以將資料傳輸到 Azure 檔案服務中。
* **[AzCopy](../common/storage-use-azcopy.md?toc=%2fazure%2fstorage%2ffiles%2ftoc.json#upload-files-to-an-azure-file-share)**：AzCopy 是命令列公用程式，設計為使用最佳效能的簡單命令，將資料複製到 Azure 檔案服務與 Azure Blob 儲存體，以及從其中複製資料。 Windows 和 Linux 都有 AzCopy 可供使用。

## <a name="next-steps"></a>後續步驟
* [規劃 Azure 檔案同步部署](storage-sync-files-planning.md)
* [部署 Azure 檔案服務](storage-files-deployment-guide.md)
* [部署 Azure 檔案同步](storage-sync-files-deployment-guide.md)
