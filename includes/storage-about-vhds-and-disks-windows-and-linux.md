---
title: 包含檔案
description: 包含檔案
services: storage
author: rogara
ms.service: storage
ms.topic: include
ms.date: 04/09/2018
ms.author: rogarana
ms.custom: include file
ms.openlocfilehash: 26c3e8a096a3ee7bc5146759f8de62e5c1c7fed1
ms.sourcegitcommit: ad08b2db50d63c8f550575d2e7bb9a0852efb12f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/26/2018
ms.locfileid: "47389970"
---
## <a name="about-vhds"></a>關於 VHD

Azure 中使用的 VHD 是以分頁 Blob 儲存在 Azure 標準或進階儲存體帳戶中的 .vhd 檔案。 如需分頁 Blob 的詳細資訊，請參閱 [了解區塊 Blob 和分頁 Blob](/rest/api/storageservices/Understanding-Block-Blobs--Append-Blobs--and-Page-Blobs/)。 如需進階儲存體的詳細資訊，請參閱[高效能的進階儲存體和 Azure VM](../articles/virtual-machines/windows/premium-storage.md)。

Azure 支援 VHD 格式的固定磁碟。 固定格式會線性地陳列檔案內部的邏輯磁碟，因此磁碟位移 X 會儲存於 Blob 位移 X。Blob 最後的頁尾將說明 VHD 屬性。 由於大多數磁碟內都有相當大的未用範圍，因此固定格式通常會浪費空間。 不過，Azure 會以疏鬆格式來儲存 .vhd 檔案，因此您可同時享有固定和動態磁碟的好處。 如需詳細資訊，請參閱[開始使用虛擬硬碟](https://technet.microsoft.com/library/dd979539.aspx)。

Azure 中您想要用作建立磁碟或映像來源的所有 VHD 檔案都是唯讀的，但使用者上傳或複製至 Azure 儲存體的 .vhd 檔案除外 (這些檔案可能是讀寫或唯讀的)。 Azure 會在您建立磁碟或映像時製作來源 .vhd 檔案的複本。 取決於您使用 VHD 的方式，這些複本可為唯讀或讀取及寫入。

當您從映像建立虛擬機器時，Azure 會以來源 .vhd 檔案複本為虛擬機器建立磁碟。 若要防止意外刪除，Azure 會在任何用來建立映像、作業系統磁碟或資料磁碟的來源 .vhd 檔案上加上租用。

您必須先刪除磁碟或映像來移除租約，才能刪除來源 .vhd 檔案。 若要刪除虛擬機器做為作業系統磁碟使用的 .vhd 檔案，您只要刪除虛擬機器並刪除所有關聯的磁碟，就可以一次刪除虛擬機器、作業系統磁碟和來源 .vhd 檔案。 不過，刪除做為資料磁碟來源的 .vhd 檔案檔案需要依設定的順序執行幾個步驟。 首先，您需將磁碟與虛擬機器中斷連結，接著刪除磁碟，然後再刪除 .vhd 檔案。

> [!WARNING]
> 如果您刪除儲存體中的來源 .vhd 檔案從或刪除您的儲存體帳戶，Microsoft 便無法為您復原該資料。

## <a name="types-of-disks"></a>磁碟類型

Azure 磁碟設計成確保可用性達 99.999%。 Azure 磁碟一直提供企業級持久性，其年度失敗率為零，領先業界。

在建立磁碟時，您有三個可供選擇的儲存體效能層級 - 進階 SSD 磁碟、標準 SSD (預覽)和標準 HDD 儲存體。 此外，磁碟也有兩種類型 - 非受控和受控。

### <a name="standard-hdd-disks"></a>標準 HDD 磁碟

標準 HDD 磁碟以 HDD 作為後盾，可提供符合成本效益的儲存體。 標準 HDD 儲存體可複寫到一個資料中心的本機位置，或是利用主要和次要資料中心提供異地備援。 如需儲存體複寫的詳細資訊，請參閱 [Azure 儲存體複寫](../articles/storage/common/storage-redundancy.md)。 

如需使用標準 HDD 磁碟的詳細資訊，請參閱[標準儲存體和磁碟](../articles/virtual-machines/windows/standard-storage.md)。

### <a name="standard-ssd-disks"></a>標準 SSD 磁碟

標準 SSD 磁碟的設計是為了處理和標準 HDD 磁碟相同類型的工作負載，但可提供比 HDD 還要好的一致效能和可靠性。 標準 SSD 磁碟會合併進階 SSD 磁碟和標準 HDD 磁碟的元素，可針對磁碟上不需要高 IOPS 的網頁伺服器等應用程式，構成最適合的解決方案，且符合成本效益。 情況適合時，標準 SSD 磁碟是大部分工作負載的建議部署選項。 標準 SSD 磁碟在所有區域僅提供受控磁碟形式，但目前只有本地備援儲存體 (LRS) 復原類型。

### <a name="premium-ssd-disks"></a>進階 SSD 磁碟

進階 SSD 磁碟是以 SSD 為後盾，可針對執行時需要大量 I/O 之工作負載的 VM 提供高效能、低延遲的磁碟支援。 一般來說，您可以使用在系列名稱中包含 "s" 的大小來搭配進階 SSD 磁碟。 例如，在 Dv3 系列和 Dsv3 系列中，能搭配進階 SSD 磁碟使用的為 Dsv3 系列。  如需詳細資訊，請參閱[進階儲存體](../articles/virtual-machines/windows/premium-storage.md)。

### <a name="unmanaged-disks"></a>非受控磁碟

非受控磁碟是 VM 已在使用的傳統磁碟類型。 利用這些磁碟，您可以建立自己的儲存體帳戶，並在建立磁碟時指定該儲存體帳戶。 請確保相同儲存體帳戶中未放入過多磁碟，否則您可能會超出儲存體帳戶的[延展性目標](../articles/storage/common/storage-scalability-targets.md) (例如，20,000 IOPS)，而導致 VM 遭到節流。 使用非受控磁碟時，您必須了解如何充分運用一或多個儲存體帳戶，以獲得最佳的 VM 效能。

### <a name="managed-disks"></a>受控磁碟

受控磁碟可在背景中為您處理儲存體帳戶的建立/管理作業，確保您不需要擔心儲存體帳戶的延展性限制。 您只需指定磁碟大小和效能層級 (標準/進階)，Azure 就會為您建立和管理磁碟。 當您新增磁碟或相應增加和減少 VM 時，都不必擔心所使用的儲存體。

您也可以在每個 Azure 區域中使用單一儲存體帳戶管理自訂映像，並使用映像在相同訂用帳戶中建立數百個 VM。 如需受控磁碟的詳細資訊，請參閱 [Azure 受控磁碟概觀](../articles/virtual-machines/windows/managed-disks-overview.md)。

建議您對新 VM 使用 Azure 受控磁碟，並將先前建立的非受控磁碟轉換成受控磁碟，以充分利用受控磁碟中提供的許多功能。

### <a name="disk-comparison"></a>磁碟比較

下表會比較標準 HDD、標準 SSD 和進階 SSD 的非受控磁碟和受控磁碟，以協助您決定要使用何者。 以星號表示的大小目前為預覽狀態。

|    | Azure 進階磁碟 |Azure 標準 SSD 磁碟 | Azure 標準 HDD 磁碟
|--- | ------------------ | ------------------------------- | -----------------------
| 磁碟類型 | 固態硬碟 (SSD) | 固態硬碟 (SSD) | 硬碟 (HDD)  
| 概觀  | 以 SSD 為基礎，針對執行時需要大量 I/O 之工作負載的 VM 或裝載任務關鍵性生產環境的 VM，提供高效能、低延遲的磁碟支援 |比 HDD 還要好的一致效能和可靠性。 已針對低 IOPS 工作負載最佳化| 符合成本效益的 HDD 型磁碟，適用於存取不頻繁的情況
| 案例  | 生產環境和重視效能的工作負載 |網頁伺服器，輕量使用的企業應用程式和開發/測試| 備份、不重要、存取不頻繁
| 磁碟大小 | P4: 32 GiB (僅限受控磁碟)<br>P6: 64 GiB (僅限受控磁碟)<br>P10: 128 GiB<br>P15: 256 GiB (僅限受控磁碟)<br>P20: 512 GiB<br>P30: 1024 GiB<br>P40: 2048 GiB<br>P50: 4,095 GiB<br>P60: 8,192 GiB * (8 TiB)<br>P70: 16,384 GiB * (16 TiB)<br>P80: 32,767 GiB * (32 TiB) |僅限受控磁碟：<br>E10: 128 GiB<br>E15: 256 GiB<br>E20: 512 GiB<br>E30: 1024 GiB<br>E40: 2048 GiB<br>E50: 4095 GiB<br>E60: 8,192 GiB * (8 TiB)<br>E70: 16,384 GiB * (16 TiB)<br> E80: 32,767 GiB * (32 TiB) | 非受控磁碟：1 GiB – 4 TiB (4095 GiB) <br><br>受控磁碟：<br> S4: 32 GiB <br>S6: 64 GiB <br>S10: 128 GiB <br>S15: 256 GiB <br>S20: 512 GiB <br>S30: 1024 GiB <br>S40: 2048 GiB<br>S50: 4095 GiB<br>S60: 8,192 GiB * (8 TiB)<br>S70: 16,384 GiB * (16 TiB)<br>S80: 32,384 GiB * (32 TiB)
| 每一磁碟的輸送量上限 | P4: 25 MiB/s<br> P6: 50 MiB/s<br> P10: 100 MiB/s<br> P15: 200 MiB/s<br> P20: 150 MiB/s<br> P30: 200 MiB/s<br> P40-P50: 250 MiB/s<br> P60: 480 MiB/s<br> P70-P80: 750 MiB/s | E10-E50: 最多 60 MiB/s<br> E60: 最多 300 MiB/s *<br> E70-E80: 500 MiB/s *| S4 - S50: Upt o 60 MiB/s<br> S60: 最多 300 MiB/s *<br> S70-S80: 最多 500 MiB/s *
| 每一磁碟的 IOPS 上限 | P4: 120 IOPS<br> P6: 240 IOPS<br> P10: 500 IOPS<br> P15: 1100 IOPS<br> P20: 2300 IOPS<br> P30: 5000 IOPS<br> P40-P50: 7500 IOPS<br> P60: 12,500 IOPS *<br> P70: 15,000 IOPS *<br> P80: 20,000 IOPS * | E10-E50: 最多 500 IOPS<br> E60: 最多 1300 IOPS *<br> E70-E80: 最多 2000 IOPS * | S4-S50: 最多 500 IOPS<br> S60: 最多 1300 IOPS *<br> S70-S80: 最多 2000 IOPS *