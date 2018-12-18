---
title: Azure 儲存體中的資料複寫 | Microsoft Docs
description: 系統會 複製Microsoft Azure 儲存體帳戶中的資料，以維持持久性和高可用性。 複寫選項包括本地備援儲存體 (LRS)、區域備援儲存體 (ZRS)、異地備援儲存體 (GRS) 和讀取權限異地備援儲存體 (RA-GRS)。
services: storage
author: tamram
ms.service: storage
ms.topic: article
ms.date: 01/21/2018
ms.author: tamram
ms.component: common
ms.openlocfilehash: 18dfdce827a76d924494e66ceb0d03e2bb3a3ffe
ms.sourcegitcommit: 9819e9782be4a943534829d5b77cf60dea4290a2
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/06/2018
ms.locfileid: "39523755"
---
# <a name="azure-storage-replication"></a>Azure 儲存體複寫

Microsoft Azure 儲存體帳戶中的資料一律會進行複寫以確保持久性及高可用性。 「Azure 儲存體」複寫會複製您的資料，以保護該資料不受計劃性和非計劃性事件影響，包括暫時性硬體故障、網路或電力中斷、大規模天然災害等。 您可以選擇在相同的資料中內、跨相同地區內的不同區域資料中心，甚至是跨地區，進行資料複寫。

複寫可確保您的儲存體帳戶符合[儲存體的服務等級協定 (SLA)](https://azure.microsoft.com/support/legal/sla/storage/) 即使遇到失敗時也一樣。 請參閱 SLA 以取得 Azure 儲存體持續性和可用性保證的相關資訊。

## <a name="choosing-a-replication-option"></a>選擇複寫選項

建立儲存體帳戶時，您可以選取下列其中一個複寫選項：

* [本地備援儲存體 (LRS)](storage-redundancy-lrs.md)
* [區域備援儲存體 (ZRS)](storage-redundancy-zrs.md)
* [異地備援儲存體 (GRS)](storage-redundancy-grs.md)
* [讀取權限異地備援儲存體 (RA-GRS)](storage-redundancy-grs.md#read-access-geo-redundant-storage)

下表快速簡要說明針對所指定類型的事件 (或具有類似影響的事件)，每個複寫策略將為您提供的持久性和可用性範圍。

| 案例                                                                                                 | LRS                             | ZRS                              | GRS                                  | RA-GRS                               |
| :------------------------------------------------------------------------------------------------------- | :------------------------------ | :------------------------------- | :----------------------------------- | :----------------------------------- |
| 資料中心內的節點無法供使用                                                                 | 是                             | yes                              | yes                                  | 是                                  |
| 整個資料中心 (區域或非區域) 變成無法供使用                                           | 否                              | yes                              | yes                                  | 是                                  |
| 全區域服務中斷                                                                                     | 否                              | 否                               | yes                                  | 是                                  |
| 在全區域服務無法供使用的情況下對資料 (位於遠端、異地複寫區域) 進行讀取存取 | 否                              | 否                               | 否                                   | 是                                  |
| 設計為可在指定的一年中讓物件持久性達到 ___                                          | 至少 99.999999999% (11 個 9) | 至少 99.9999999999% (12 個 9) | 至少 99.99999999999999% (16 個 9) | 至少 99.99999999999999% (16 個 9) |
| 支援的儲存體帳戶類型                                                                   | GPv1、GPv2、Blob                | GPv2                             | GPv1、GPv2、Blob                     | GPv1、GPv2、Blob                     |

如需有關不同備援選項的定價資訊，請參閱 [Azure 儲存體定價](https://azure.microsoft.com/pricing/details/storage/)。

> [!NOTE]
> 進階儲存體僅支援本地備援儲存體 (LRS)。 如需進階儲存體的相關資訊，請參閱 [進階儲存體：Azure 虛擬機器工作負載適用的高效能儲存體](../../virtual-machines/windows/premium-storage.md)。

## <a name="changing-replication-strategy"></a>變更複寫策略
您可以使用 [Azure 入口網站](https://portal.azure.com/)、[Azure Powershell](storage-powershell-guide-full.md)、[Azure CLI](https://docs.microsoft.com/cli/azure/install-azure-cli?view=azure-cli-latest) 或眾多 [Azure 用戶端程式庫](https://docs.microsoft.com/azure/index?view=azure-dotnet#pivot=sdkstools)其中之一，來變更儲存體帳戶的複寫策略。 變更儲存體帳戶的複寫類型並不會造成停機。

   > [!NOTE]
   > 目前，您無法使用入口網站或 API 將帳戶轉換成 ZRS。 如果您想要將帳戶的複寫轉換成 ZRS，請參閱[區域備援儲存體 (ZRS)](storage-redundancy-zrs.md) 以取得詳細資料。
    
### <a name="are-there-any-costs-to-changing-my-accounts-replication-strategy"></a>變更我帳戶的複寫策略是否會產生任何費用？
這取決於您的轉換途徑。 從費用最低到費用最高的備援供應項目依序為 LRS、ZRS、GRS 及 RA-GRS。 例如，「從」LRS 傳換成任何方案都會產生額外費用，因為是移至更複雜的備援層級。 移「至」GRS 或 RA-GRS 會產生輸出頻寬費用，因為您的資料 (在您的主要區域中) 會複寫至遠端次要區域。 這是在初始設定時所收取的一次性費用。 在複製資料之後，即沒有任何進一步的轉換費用。 您將只有在複寫任何新資料或對現有資料的更新時，才需支付費用。 如需有關頻寬費用的詳細資料，請參閱 [Azure 儲存體定價頁面](https://azure.microsoft.com/pricing/details/storage/blobs/)找到。

如果您從 GRS 變更為 LRS，將不會有任何額外的費用，但系統會將您的已複寫資料從次要區位置中刪除。

## <a name="see-also"></a>另請參閱

- [本地備援儲存體 (LRS)：適用於 Azure 儲存體的低成本資料備援](storage-redundancy-lrs.md)
- [區域備援儲存體 (ZRS)：高可用性 Azure 儲存體應用程式](storage-redundancy-zrs.md)
- [異地備援儲存體 (GRS)：適用於 Azure 儲存體的跨區域複寫](storage-redundancy-grs.md)
- [Azure 儲存體的延展性與效能目標](storage-scalability-targets.md)
- [使用 RA-GRS 儲存體設計高可用性應用程式](../storage-designing-ha-apps-with-ragrs.md)
- [Microsoft Azure 儲存體備援選項和讀取權限異地備援儲存體 ](http://blogs.msdn.com/b/windowsazurestorage/archive/2013/12/11/introducing-read-access-geo-replicated-storage-ra-grs-for-windows-azure-storage.aspx)
- [SOSP 文件：Azure 儲存體：具有高度一致性的高可用性雲端儲存體服務](http://blogs.msdn.com/b/windowsazurestorage/archive/2011/11/20/windows-azure-storage-a-highly-available-cloud-storage-service-with-strong-consistency.aspx)
