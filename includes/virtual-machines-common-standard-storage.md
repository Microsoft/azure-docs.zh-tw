---
title: 包含檔案
description: 包含檔案
services: storage
author: yuemlu
ms.service: storage
ms.topic: include
ms.date: 06/05/2018
ms.author: yuemlu
ms.custom: include file
ms.openlocfilehash: e3d904358282f303a2d1ab35cf4fdc8026d7db55
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "47060342"
---
# <a name="cost-effective-standard-storage-and-unmanaged-and-managed-azure-vm-disks"></a>符合成本效益的標準儲存體及非受控和受控 Azure VM 磁碟

如果 VM 執行的工作負載不在乎延遲，Azure 標準儲存體可以提供可靠、低成本的磁碟支援。 它也支援 blob、資料表、佇列和檔案。 使用標準儲存體時，資料會儲存在硬碟 (HDD)。 使用 VM 時，您可以對開發/測試案例和較不重要的工作負載使用標準 SSD 和 HDD 磁碟，而對關鍵任務的實際執行應用程式使用進階 SSD 磁碟。 所有 Azure 區域中都可以使用標準儲存體。 

本文將著重在使用標準 SSD 和 HDD 磁碟。 如需有關儲存體用於 blob、資料表、佇列和檔案的詳細資訊，請參閱[儲存體簡介](../articles/storage/common/storage-introduction.md)。

## <a name="disk-types"></a>磁碟類型

有兩種方式可以為 Azure VM 建立標準磁碟︰

**非受控磁碟**︰這種類型的磁碟是原始方法，由您管理儲存對應至 VM 磁碟的 VHD 檔案時所用的儲存體帳戶。 VHD 檔案會以分頁 Blob 的形式儲存在儲存體帳戶中。 非受控磁碟可以附加至任何 Azure VM 大小，包括主要使用進階儲存體的 VM，例如 DSv2 和 GS 系列。 Azure VM 支援附加數個標準磁碟，每個 VM 最多可以有 256 PiB 的儲存體。 如果您使用預覽磁碟大小，您的每個 VM 最多可以有約 2 PiB 的儲存體。 

[**Azure 受控磁碟**](../articles/virtual-machines/windows/managed-disks-overview.md)︰此功能會為您管理用於 VM 磁碟的儲存體帳戶。 您只需要指定類型 (進階 SSD、標準 SSD 或標準 HDD)，還有您需要的磁碟大小，Azure 就會替您建立並管理磁碟。 您不必擔心需要將磁碟分配至多個儲存體帳戶，才能維持在儲存體帳戶的延展性限制內。這個部分 Azure 會為您處理。

即使這兩種磁碟都可用，我們建議您使用受控磁碟，以利用它的許多功能。

若要開始使用 Azure 標準儲存體，請參閱[免費開始使用](https://azure.microsoft.com/pricing/free-trial/)。 

如需有關如何以受控磁碟建立 VM 的資訊，請參閱下列其中一份文件。

* [使用 Resource Manager 和 PowerShell 建立 VM](../articles/virtual-machines/windows/quick-create-powershell.md)
* [使用 CLI 建立 Linux VM](../articles/virtual-machines/linux/quick-create-cli.md)

## <a name="standard-storage-features"></a>標準儲存體功能

讓我們看看標準儲存體的一些功能。 如需詳細資訊，請參閱 [Azure 儲存體簡介](../articles/storage/common/storage-introduction.md)。

**標準儲存體**：Azure 標準儲存體支援 Azure 磁碟、Azure Blob、Azure 檔案、Azure 資料表和 Azure 佇列。 若要使用標準儲存體服務，首先請[建立 Azure 儲存體帳戶](../articles/storage/common/storage-quickstart-create-account.md)。

**標準 SSD 磁碟：** 相較於標準 HDD 磁碟，標準 SSD 磁碟能提供更可靠的效能，且目前可供使用。 如需提供標準 SSD 磁碟服務的區域相關資訊，請參閱[標準 SSD 磁碟的區域可用性](../articles/virtual-machines/windows/faq-for-disks.md#standard-ssds-azure-regions)。

**標準 HDD 磁碟︰** 標準 HDD 磁碟可以附加至所有 Azure VM，包括進階儲存體使用的大小系列 VM，例如 DSv2 和 GS 系列。 標準 HDD 磁碟只能附加至一個 VM。 不過，您可以將一或多個這些磁碟附加至 VM，最多為該 VM 大小所定義的最大磁碟計數。 在下一節的「標準儲存體延展性和效能目標」中，我們會更詳細地說明規格。

**標準分頁 Blob**︰標準分頁 Blob 用來保存 VM 的持續性磁碟，也可以透過 REST 直接存取，如同其他類型的 Azure Blob 一樣。 [分頁 Blob](/rest/api/storageservices/Understanding-Block-Blobs--Append-Blobs--and-Page-Blobs)是已最佳化隨機讀取和寫入作業的一組 512 位元組分頁。 

**儲存體複寫︰** 在大部分區域，標準儲存體帳戶中的資料可以在本地複寫，或跨多個資料中心進行異地複寫。 可用的四種複寫包括本地備援儲存體 (LRS)、區域備援儲存體 (ZRS)、異地備援儲存體 (GRS) 和讀取權限異地備援儲存體 (RA-GRS)。 標準儲存體中的受控磁碟目前只支援本地備援儲存體 (LRS)。 如需詳細資訊，請參閱[儲存體複寫](../articles/storage/common/storage-redundancy.md)。

## <a name="scalability-and-performance-targets"></a>擴充和效能目標

在本節中，我們說明使用標準儲存體時需要考慮的延展性和效能目標。

### <a name="account-limits--does-not-apply-to-managed-disks"></a>帳戶限制 - 不會套用至受控磁碟

| **Resource** | **預設限制** |
|--------------|-------------------|
| 每一儲存體帳戶的 TB  | 500 TB |
| 每一儲存體帳戶的輸入上限<sup>1</sup> (美國地區) | 如果啟用 GRS/ZRS，則為 10 Gbps，LRS 為 20 Gbps |
| 每一儲存體帳戶的輸出上限<sup>1</sup> (美國地區) | 如果啟用 RA-GRS/GRS/ZRS，則為 20 Gbps，LRS 為 30 Gbps |
| 每一儲存體帳戶的輸入上限<sup>1</sup> (歐洲和亞洲地區) | 如果啟用 GRS/ZRS，則為 5 Gbps，LRS 為 10 Gbps |
| 每一儲存體帳戶的輸出上限<sup>1</sup> (歐洲和亞洲地區) | 如果啟用 RA-GRS/GRS/ZRS，則為 10 Gbps，LRS 為 15 Gbps |
| 每一儲存體帳戶的總要求率 (假設 1KB 物件大小) | 每秒實體或每秒訊息最多 20,000 個 IOPS |

<sup>1</sup>輸入是指傳送至某個儲存體帳戶的所有資料 (要求)。 輸出是指從某個儲存體帳戶接收的所有資料 (回應)。

如需詳細資訊，請參閱 [Azure 儲存體延展性和效能目標](../articles/storage/common/storage-scalability-targets.md)。

如果您的應用程式需求超出單一儲存體帳戶的延展性目標，請建置使用多個儲存體帳戶的應用程式，並將資料分散到那些儲存體帳戶中。 或者，您可以使用 Azure 受控磁碟，則 Azure 便會為您管理資料的分割和位置。

### <a name="standard-disks-limits"></a>標準磁碟限制

不同於進階磁碟，標準磁碟沒有佈建每秒輸入/輸出作業 (IOPS) 和輸送量 (頻寬)。 標準磁碟的效能隨著磁碟附加的 VM 大小而不同，而非取決於磁碟大小。 您可能會達到下表中列出的效能限制。

**標準磁碟限制 (受控和非受控)**

| **VM 層**            | **基本層 VM** | **標準層 VM** |
|------------------------|-------------------|----------------------|
| 最大磁碟大小          | 32,767 GiB           | 32,767 GiB        |
| 每一磁碟的 IOPS 上限為 8 KB | 最多 2,000         | 最多 2,000        |
| 每一磁碟的最大頻寬 (MB/秒) | 最多 500 MB/秒     | 最多 500 MB/秒      |

如果您的工作負載需要高效能、低延遲磁碟支援，您應該考慮使用進階儲存體。 若要知道進階儲存體的其他優點，請參閱[高效能進階儲存體和 Azure VM 磁碟](../articles/virtual-machines/windows/premium-storage.md)。

## <a name="snapshots-and-copy-blob"></a>快照集和複製 Blob

對於儲存體服務而言，VHD 檔案是分頁 Blob。 您可以建立分頁 Blob 的快照集，然後複製到另一個位置，例如不同的儲存體帳戶。

### <a name="unmanaged-disks"></a>非受控磁碟

就像對標準儲存體使用快照集一樣，您可以使用相同方式為非受控標準磁碟建立[增量快照集](../articles/virtual-machines/windows/incremental-snapshots.md)。 如果來源磁碟在本地備援儲存體帳戶中，我們建議您建立快照集，然後將那些快照集複製到異地備援標準儲存體帳戶。 如需詳細資訊，請參閱 [Azure 儲存體備援選項](../articles/storage/common/storage-redundancy.md)。

如果磁碟已附加至 VM，則磁碟上不允許某些 API 作業。 例如，只要磁碟連接至 VM，您就無法在該 Blob 上執行 [複製 Blob](/rest/api/storageservices/Copy-Blob) 作業。 您必須先使用[快照 Blob](/rest/api/storageservices/Snapshot-Blob) REST API 方法建立該 Blob 的快照，然後對該快照執行[複製 Blob](/rest/api/storageservices/Copy-Blob) 以複製連接的磁碟。 或者，您可以中斷連結磁碟，再執行任何必要的作業。

若要維護快照集的異地備援副本，您可以使用 AzCopy 或「複製 Blob」，將本地備援儲存體帳戶中的快照集，複製到異地備援標準儲存體帳戶。 如需詳細資訊，請參閱[使用 AzCopy 命令列公用程式傳輸資料](../articles/storage/common/storage-use-azcopy.md)和[複製 Blob](/rest/api/storageservices/Copy-Blob)。

如需對標準儲存體帳戶中的分頁 Blob 執行 REST 作業的詳細資訊，請參閱 [Azure 儲存體服務 REST API](/rest/api/storageservices/Azure-Storage-Services-REST-API-Reference)。

### <a name="managed-disks"></a>受控磁碟

受控磁碟的快照集是受控磁碟的唯讀複本，儲存為標準受控磁碟。 受控磁碟目前不支援增量快照集，但未來會支援。

如果受控磁碟已附加至 VM，則磁碟上不允許某些 API 作業。 例如，當磁碟已附加至 VM 時，您無法產生共用存取簽章 (SAS) 來執行複製作業。 請先建立磁碟的快照集，再複製快照集。 或者，您可以中斷連結磁碟，然後產生共用存取簽章 (SAS) 來執行複製作業。

## <a name="pricing-and-billing"></a>價格和計費

使用標準儲存體時，需考量下列計費資訊：

* 標準儲存體非受控磁碟/資料大小 
* 標準受控磁碟
* 標準儲存體快照集
* 輸出資料傳輸
* 交易

**非受控儲存體資料和磁碟大小︰** 對於非受控磁碟和其他資料 (blob、資料表、佇列和檔案)，您只需支付您使用的空間量。 例如，如果 VM 的分頁 Blob 佈建為 127 GB，但 VM 實際上只使用 10 GB 的空間，則會向您收取 10 GB 空間的費用。 我們支援最大 8191 GB 的標準儲存體，以及最大 4095 GB 的標準非受控磁碟。 

**受控磁碟**：標準受控磁碟的計費取決於磁碟的佈建大小。 Azure 會將佈建大小對應 (無條件進位) 至下表中指定的最接近受控磁碟選項。 每一個受控磁碟對應至其中一個支援的佈建大小，並據此計費。 例如，如果您建立標準受控磁碟，並指定 200 GiB 的佈建大小，就會依據 S15 磁碟類型的定價向您收費。

| **標準 HDD 受控<br>磁碟類型** | **S4** | **S6** | **S10** | **S15** | **S20** | **S30** | **S40** | **S50** | **S60** | **S70** | **S80** |
|------------------|---------|---------|--------|--------|--------|----------------|----------------|----------------|----------------|----------------|----------------|
| 磁碟大小        | 32 GiB  | 64 GiB  | 128 GB | 256 GiB | 512 GB | 1,024 GiB (1 TiB) | 2,048 GiB (2 TiB) | 4,095 GiB (4 TiB) | 8,192 GiB (8 TiB) | 16,385 GiB (16 TiB) | 32,767 GiB (32 TiB) |


**快照集**：標準儲存體的快照集會因為使用的額外容量而產生費用。 如需有關快照的資訊，請參閱[建立 Blob 的快照](/rest/api/storageservices/Creating-a-Snapshot-of-a-Blob)。

**輸出資料傳輸**： [輸出資料傳輸](https://azure.microsoft.com/pricing/details/data-transfers/) (Azure 資料中心送出的資料) 會產生頻寬使用量費用。

**交易**：在標準儲存體上，Azure 每 100000 筆交易會收取 $0.0036。 交易一概包括對儲存體進行的讀取和寫入作業。

如需標準儲存體、虛擬機器和受控磁碟價格的詳細資訊，請參閱：

* [Azure 儲存體定價](https://azure.microsoft.com/pricing/details/storage/)
* [虛擬機器定價](https://azure.microsoft.com/pricing/details/virtual-machines/)
* [受控磁碟價格](https://azure.microsoft.com/pricing/details/managed-disks)

## <a name="azure-backup-service-support"></a>Azure 備份服務支援

您可以使用 Azure 備份來備份具有非受控磁碟的虛擬機器。 [其他詳細資訊](../articles/backup/backup-azure-vms-first-look-arm.md)。

您也可以對受控磁碟使用 Azure 備份服務，利用以時間為基礎的備份、簡易 VM 還原和備份保留原則，建立備份作業。 如需深入了解，請參閱[針對具有受控磁碟的 VM 使用 Azure 備份服務](../articles/backup/backup-introduction-to-azure-backup.md#using-managed-disk-vms-with-azure-backup)。

## <a name="next-steps"></a>後續步驟

* [Azure 儲存體簡介](../articles/storage/common/storage-introduction.md)

* [建立儲存體帳戶](../articles/storage/common/storage-quickstart-create-account.md)

* [受控磁碟概觀](../articles/virtual-machines/linux/managed-disks-overview.md)

* [使用 Resource Manager 和 PowerShell 建立 VM](../articles/virtual-machines/windows/quick-create-powershell.md)

* [使用 CLI 建立 Linux VM](../articles/virtual-machines/linux/quick-create-cli.md)
