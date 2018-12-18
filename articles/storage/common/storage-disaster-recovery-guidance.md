---
title: Azure 儲存體發生中斷時該怎麼辦 | Microsoft Docs
description: Azure 儲存體發生中斷時該怎麼辦
services: storage
author: tamram
ms.service: storage
ms.devlang: dotnet
ms.topic: article
ms.date: 09/13/2018
ms.author: tamram
ms.component: common
ms.openlocfilehash: 20db515e99f3e7535ba7b60bbd84f050e33b7acb
ms.sourcegitcommit: 4ecc62198f299fc215c49e38bca81f7eb62cdef3
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "47033918"
---
# <a name="what-to-do-if-an-azure-storage-outage-occurs"></a>如果 Azure 儲存體發生中斷怎麼辦
在 Microsoft，我們竭力確保我們的服務總是可供使用。 有時候會因為不可抗拒之因素，而造成服務在一或多個區域內中斷。 為了協助您處理這類罕見的狀況，我們提供下列 Azure 儲存體服務的高階指引。

## <a name="how-to-prepare"></a>如何做好準備
每位客戶備妥自己的災害復原計畫是相當重要的。 自儲存體中斷的狀況復原，通常需要作業人員與自動化程序，以將應用程式重新啟動回運作狀態。 請參閱下列 Azure 文件，以建立您的災害復原計畫：

* [可用性檢查清單](https://docs.microsoft.com/azure/architecture/checklist/availability)
* [為 Azure 設計有彈性的應用程式](https://docs.microsoft.com/azure/architecture/resiliency/)
* [Azure Site Recovery 服務](https://azure.microsoft.com/services/site-recovery/)
* [Azure 儲存體複寫](https://docs.microsoft.com/azure/storage/common/storage-redundancy)
* [Azure 備份服務](https://azure.microsoft.com/services/backup/)

## <a name="how-to-detect"></a>如何偵測
判斷 Azure 服務狀態的建議方式是訂閱 [Azure 服務健康狀態儀表板](https://azure.microsoft.com/status/)。

## <a name="what-to-do-if-a-storage-outage-occurs"></a>如果儲存體發生中斷怎麼辦
若一或多個儲存體服務，暫時無法在一或多個區域供使用，您有兩個選項可以考慮。 如果您希望立刻存取您的資料，請考慮「選項 2」。

### <a name="option-1-wait-for-recovery"></a>選項 1︰等待復原
在此情況下，您不需要採取任何動作。 我們正在努力還原 Azure 服務的可用性。 您可以在 [Azure 服務健康狀態儀表板](https://azure.microsoft.com/status/)上監視目前的服務狀態。

### <a name="option-2-copy-data-from-secondary"></a>選項 2︰從次要區域複製資料
如果您為儲存體帳戶選取 [讀取權限異地備援儲存體(RA-GRS)](storage-redundancy-grs.md#read-access-geo-redundant-storage) (建議)，您將會有從次要地區讀取資料的權限。 您可以使用 [AzCopy](storage-use-azcopy.md)、[Azure PowerShell](storage-powershell-guide-full.md) 及 [Azure Data Movement 程式庫](https://azure.microsoft.com/blog/introducing-azure-storage-data-movement-library-preview-2/)，將資料從次要地區複製到其他未受影響之區域內的儲存體帳戶，然後將應用程式的讀取和寫入可用性都指向該儲存體帳戶。

## <a name="what-to-expect-if-a-storage-failover-occurs"></a>如果發生儲存體容錯移轉該預期什麼
如果您選擇[異地備援儲存體 (GRS)](storage-redundancy-grs.md) 或[讀取權限異地備援儲存體 (RA-GRS)](storage-redundancy-grs.md#read-access-geo-redundant-storage) (建議)，Azure 儲存體會將您的資料持久保留在兩個區域 (主要和次要)。 在兩個區域中，Azure 儲存體會持續維護多個您資料的複本。

當您的主要區域受到區域性災害影響時，我們會先嘗試還原該區域內的服務，以提供 RTO 與 RPO 的最佳組合。 視災害的本質及其影響而定，在某些罕見的情況下我們可能無法還原主要區域。 這時候，我們會執行異地複寫容錯移轉。 跨區域資料複寫是非同步的程序，因此可能會有延遲，而且可能會遺失尚未複寫到次要地區的變更。

關於儲存體異地複寫容錯移轉體驗有幾點事項：

* 儲存體異地複寫容錯移轉只會由 Azure 儲存體團隊發動 – 客戶無須採取行動。 當 Azure 儲存體小組已用盡在相同區域中還原資料的所有選項時，就會觸發容錯移轉，以提供 RTO 與 RPO 的最佳組合。
* 您現有的 Blob、資料表、佇列及檔案之儲存體服務端點，在容錯移轉後都會維持原狀；需要更新 Microsoft 提供的 DNS 項目，才能從主要區域切換到次要地區。 Microsoft 將會在異地容錯移轉的過程中，自動執行這項更新。
* 由於災害的影響，在異地複寫容錯移轉之前與期間內，您將不會有您儲存體帳戶的寫入權限，但如果您的儲存體帳戶已設為 RA-GRS，您仍然可以從次要區域讀取資料。
* 當異地複寫容錯移轉已完成且 DNS 變更已傳播時，如果您有 GRS 或 RA-GRS，則會恢復對於儲存體帳戶的讀取和寫入權限。 您先前的次要端點會成為您的主要端點。 
* 您可以檢查主要位置的狀態，以及查詢您儲存體帳戶的上次異地容錯移轉時間。 如需詳細資訊，請參閱[儲存體帳戶 - 取得屬性](https://docs.microsoft.com/rest/api/storagerp/storageaccounts/getproperties)。
* 容錯移轉之後，您的儲存體帳戶就會正常運作，不過是處於「降級」狀態，因為它是裝載在無法進行異地複寫的獨立區域中。 為了降低此風險，我們將會還原原始的主要區域，然後進行異地容錯回復以還原為原始狀態。 如果無法復原為主要區域的原始狀態，我們會配置另一個次要地區。

## <a name="best-practices-for-protecting-your-data"></a>保護資料的最佳做法
對於定期備份儲存體資料有一些建議做法。

* VM 磁碟 – 使用 [Azure 備份服務](https://azure.microsoft.com/services/backup/) 來備份您的 Azure 虛擬機器所使用的 VM 磁碟。
* 區塊 Blob – 開啟[虛刪除](../blobs/storage-blob-soft-delete.md)來防止物件層級刪除和覆寫，或使用 [AzCopy](storage-use-azcopy.md)、[Azure PowerShell](storage-powershell-guide-full.md) 或 [Azure Data Movement 程式庫](https://azure.microsoft.com/blog/introducing-azure-storage-data-movement-library-preview-2/)，將 Blob 複製到其他區域的其他儲存體帳戶。
* 資料表 – 使用 [AzCopy](storage-use-azcopy.md) 將資料表資料匯出到位於其他區域的其他儲存體帳戶。
* 檔案 – 使用 [AzCopy](storage-use-azcopy.md) 或 [Azure PowerShell](storage-powershell-guide-full.md) 將您的檔案複製到位於其他區域的其他儲存體帳戶。

如需建立應用程式以充分利用 RA-GRS 功能的相關資訊，請造訪[使用 RA-GRS 儲存體設計高可用性的應用程式 (英文)](../storage-designing-ha-apps-with-ragrs.md)
