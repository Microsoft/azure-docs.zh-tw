---
title: 適用於 Azure 儲存體中跨區域持久性的異地備援儲存體 (GRS) | Microsoft Docs
description: 異地備援儲存體 (GRS) 會在相距數百英哩的兩個區域之間複寫資料。 GRS 可以針對資料中心內的硬體故障提供防護，也可以針對區域性災害提供防護。
services: storage
author: tolandmike
ms.service: storage
ms.topic: article
ms.date: 03/20/2018
ms.author: jeking
ms.component: common
ms.openlocfilehash: eee9998280f2105fee5144b520bc8000b086f941
ms.sourcegitcommit: 616e63d6258f036a2863acd96b73770e35ff54f8
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/14/2018
ms.locfileid: "45603920"
---
# <a name="geo-redundant-storage-grs-cross-regional-replication-for-azure-storage"></a>異地備援儲存體 (GRS)：適用於 Azure 儲存體的跨區域複寫
[!INCLUDE [storage-common-redundancy-GRS](../../../includes/storage-common-redundancy-grs.md)]

## <a name="read-access-geo-redundant-storage"></a>讀取權限異地備援儲存體
讀取權限異地備援儲存體 (RA-GRS) 可為儲存體帳戶提供最大的可用性。 除了跨兩個區域異地複寫之外，RA-GRS 還提供對次要位置中資料的唯讀存取權。

啟用次要區域資料的唯讀權限時，您的資料能在儲存體帳戶的次要端點以及主要端點中使用。 次要端點與主要端點類似，但會在帳戶名稱中附加尾碼 `–secondary` 。 例如，如果 Blob 服務的主要端點是 `myaccount.blob.core.windows.net`，則次要端點會是 `myaccount-secondary.blob.core.windows.net`。 主要和次要端點會有相同的儲存體帳戶存取金鑰。

使用 RA-GRS 時要記得考量：

* 您的應用程式必須管理在使用 RA-GRS 時，與哪一個端點進行互動。
* 由於非同步方式複寫會涉及延遲，如果資料無法從主要區域復原，則很有可能會遺失尚未複寫到次要區域的變更，例如發生地區性災難時。
* 您可以查看儲存體帳戶的上次同步處理時間。 上次同步處理時間是 GMT 日期/時間值。 在上次同步處理時間之前完成的所有主要位置寫入都已成功寫入到次要位置，這表示現在已經可以從次要位置讀取這些資料。 在上次同步處理時間之後完成的主要位置寫入可能已可讀取，也可能無法讀取。 您可以使用 [Azure 入口網站](https://portal.azure.com/)、[Azure PowerShell](storage-powershell-guide-full.md)、或 Azure 儲存體用戶端程式庫之一查詢這個值。
* 如果 Microsoft 可起始容錯移轉到次要區域，您將會在完成容錯移轉之後具有該資料的讀取和寫入權限。 如需詳細資訊，請參閱[災害復原指南](storage-disaster-recovery-guidance.md)。
* 如需如何切換到次要區域的詳細資訊，請參閱[如果 Azure 儲存體發生中斷怎麼辦](storage-disaster-recovery-guidance.md)。
* RA-GRS 適用於高可用性目的。 如需延展性方面的指引，請檢閱[效能檢查清單](storage-performance-checklist.md)。
* 如需有關 RA-GRS 高可用性的設計建議，請參閱[使用 RA-GRS 儲存體設計高可用性應用程式](storage-designing-ha-apps-with-ragrs.md)。

## <a name="what-is-the-rpo-and-rto-with-grs"></a>RPO 和使用 GRS 的 RTO 為何？
**復原點目標 (RPO)：** 在 GRS 和 RA-GRS 中，儲存體服務會以非同步方式將資料從主要位置異地複寫到次要位置。 在主要區域發生區域性重大災難時，Microsoft 會容錯移轉至次要區域。 發生容錯移轉時，可能會遺失尚未異地複寫的最近變更。 可能遺失資料的分鐘數稱為 RPO，這是資料還可以復原的時間點。 Azure 儲存體的 RPO 通常低於 15 分鐘，但目前並沒有關於異地複寫時間長短的 SLA。

**復原時間目標 (RTO)：** RTO 是執行容錯移轉並使儲存體帳戶恢復上線的測量標準。 執行容錯移轉的時間包括下列動作：

   * Microsoft 判斷是否可以在主要位置復原資料、或是否需要容錯移轉所需的時間。
   * 變更主要 DNS 項目以指向次要位置，來執行儲存體帳戶的容錯移轉。

   Microsoft 慎重負責保存您的資料。 如果有機會在主要區域復原資料，Microsoft 會延遲容錯移轉，著重於復原您的資料。 

## <a name="paired-regions"></a>配對的區域 
建立儲存體帳戶時，您可以為帳戶選取主要區域。 配對的次要區域會視主要區域而定，且無法變更。 如需 Azure 支援區域的最新資訊，請參閱[商務持續性和災害復原 (BCDR)：Azure 配對的區域](../../best-practices-availability-paired-regions.md)。

## <a name="see-also"></a>另請參閱
- [Azure 儲存體複寫](storage-redundancy.md)
- [本地備援儲存體 (LRS)：適用於 Azure 儲存體的低成本資料備援](storage-redundancy-lrs.md)
- [區域備援儲存體 (ZRS)：高可用性 Azure 儲存體應用程式](storage-redundancy-zrs.md)