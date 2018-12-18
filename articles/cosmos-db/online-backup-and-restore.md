---
title: 使用 Azure Cosmos DB 線上備份與還原 | Microsoft Docs
description: 了解如何在 Azure Cosmos DB 資料庫上執行自動備份與還原。
keywords: 備份與還原, 線上備份
services: cosmos-db
author: kanshiG
manager: kfile
ms.service: cosmos-db
ms.devlang: na
ms.topic: conceptual
ms.date: 11/15/2017
ms.author: govindk
ms.openlocfilehash: 613c61d9b881b7d736a50cadbf313c1f9aac57c9
ms.sourcegitcommit: 387d7edd387a478db181ca639db8a8e43d0d75f7
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/10/2018
ms.locfileid: "40038081"
---
# <a name="automatic-online-backup-and-restore-with-azure-cosmos-db"></a>使用 Azure Cosmos DB 進行自動線上備份及還原
Azure Cosmos DB 可以定期自動備份您的所有資料。 自動備份的進行不會影響資料庫作業的效能或可用性。 所有備份會儲存在另一個儲存體服務中，而且這些備份會全域複寫用於為區域性災害提供復原功能。 假設您不小心刪除 Cosmos DB 容器，需要資料復原或災害復原解決方案，這正是自動備份適用的案例。  

本文開頭先回顧 Cosmos DB 的資料備援和可用性，接著討論備份。 

## <a name="high-availability-with-cosmos-db---a-recap"></a>Cosmos DB 的高可用性 - 回顧
Cosmos DB 設計為[全域分散](distribute-data-globally.md) – 可讓您調整多個 Azure 區域的輸送量以及原則導向的容錯移轉和透明多路連接的 API。 Azure Cosmos DB 提供對一致性很寬鬆的所有單一區域帳戶和所有多重區域帳戶 [99.99% 可用性 SLA](https://azure.microsoft.com/support/legal/sla/cosmos-db) \(英文\)，而所有多重區域資料庫帳戶有 99.999% 的讀取可用性。 本機磁碟會永久認可 Azure Cosmos DB 中的所有寫入，這是在用戶端認可之前，由本機資料中心內的複本仲裁認可。 Cosmos DB 的高可用性倚賴的是本機儲存體，而不依賴任何外部儲存技術。 此外，如果您的資料庫帳戶與多個 Azure 區域相關聯，您的寫入也會在其他區域複寫。 若要在低延遲狀況下調整輸送量和存取資料，您可以有盡量多個與您的資料庫帳戶相關聯的讀取區域。 在每個讀取區域中，(複寫的) 資料會在複本集上永久保存。  

如下圖所示，單一 Cosmos DB 容器是[水平分割](partition-data.md)。 圖中的一個一個圓圈是一個「分割」，透過複本集每個分割都有高可用性。 這是在單一 Azure 區域內的本機分散 (以 X 軸表示)。 再者，每個分割 (及其對應的複本集) 會全域分散在與您的資料庫帳戶相關聯的多個區域，例如，此圖中有三個區域 - 美國東部、美國西部和印度中部。 「分割集」是全域分散的實體，由您的資料在每個區域中的多個複本組成 (以 Y 軸表示)。 您可以替資料庫帳戶相關聯的區域指派優先順序，發生災害時 Cosmos DB 會以透明方式容錯移轉至下一個區域。 您也可以手動模擬容錯移轉，以測試應用程式的端對端可用性。  

下圖說明 Cosmos DB 的高度備援性。

![Cosmos DB 的高度備援性](./media/online-backup-and-restore/redundancy.png)

![Cosmos DB 的高度備援性](./media/online-backup-and-restore/global-distribution.png)

## <a name="full-automatic-online-backups"></a>完整自動線上備份
糟糕，我刪掉容器或資料庫了！ 有了 Cosmos DB，不只您的資料，還有資料的備份都一併具有高度備援性，可針對區域性災害進行復原。 目前，這些自動化備份大約每四個小時備份一次，並隨時儲存 2 個最新的備份。 如果不小心捨棄或損毀資料，請在 8 小時內[聯絡 Azure 支援服務](https://azure.microsoft.com/support/options/)。 

備份的進行不會影響資料庫作業的效能或可用性。 Cosmos DB 會在背景中備份，不會使用您佈建的 RU 或影響效能，也不影響資料庫的可用性。 

不同於儲存在 Cosmos DB 內的資料，自動備份會儲存在 Azure Blob 儲存體服務中。 若要保證低延遲/有效率上傳，您的備份快照會上傳到 Azure Blob 儲存體的執行個體，它位於 Cosmos DB 資料庫帳戶目前的寫入區域的相同區域中。 針對區域性災難的復原功能，Azure Blob 儲存體中的每個備份資料快照透過異地備援儲存體 (GRS) 再次複寫到另一個區域。 下圖顯示，整個 Cosmos DB 容器 (在此範例中，三個主要磁碟分割都在美國西部) 是在美國西部的遠端 Azure Blob 儲存體帳戶中進行備份，然後再由 GRS 複寫到美國東部。 

下圖說明 GRS Azure 儲存體中所有 Cosmos DB 實體的定期完整備份。

![GRS Azure 儲存體中所有 Cosmos DB 實體的定期完整備份](./media/online-backup-and-restore/automatic-backup.png)

## <a name="backup-retention-period"></a>備份保留期限
如上所述，Azure Cosmos DB 會在資料分割層級每隔四個小時取得您資料的快照集。 任何時候都只會保留最後兩個快照集。 不過，如果刪除容器/資料庫，Azure Cosmos DB 就會為所指定容器/資料庫內所有已刪除的資料分割，保留現有的快照集 30 天。

對於 SQL API，如果您想要維護自己的快照集，可以使用 Azure Cosmos DB [資料移轉工具](import-data.md#export-to-json-file)中的 [匯出至 JSON] 選項，來排程其他備份。

> [!NOTE]
> 如果您「在資料庫層級上佈建一組容器的輸送量」，請您切記，還原會在完整資料庫帳戶層級上執行。 如果您不慎刪除容器，也請務必在 8 小時內與支援小組聯繫。 如果您未在 8 小時內與支援小組連絡，就無法還原資料。 


## <a name="restoring-a-database-from-an-online-backup"></a>從線上備份還原資料庫

如果您不小心刪除資料庫或容器，則可以[提出支援票證](https://portal.azure.com/?#blade/Microsoft_Azure_Support/HelpAndSupportBlade)或[連絡 Azure 支援](https://azure.microsoft.com/support/options/)，要求從最新的自動備份還原資料。 Azure 支援僅適用於所選的計劃 (例如標準、開發人員)，不適用於基本計劃。 若要深入了解不同的支援計劃，請參閱 [Azure 支援計劃](https://azure.microsoft.com/support/plans/)頁面。 

如果您因為資料損毀問題而需要還原資料庫 (包含刪除容器內文件的情況下)，請參閱[處理資料損毀](#handling-data-corruption)，因為您需要採取額外步驟來防止損毀的資料覆寫現有備份。 如果要還原特定的備份快照集，Cosmos DB 會需要該資料在該快照的備份週期持續時間內為可用狀態。

## <a name="handling-data-corruption"></a>處理資料損毀

Azure Cosmos DB 會保留資料庫帳戶中每個分割區的最後兩個備份。 當容器 (文件、圖表、資料表的集合) 或資料庫意外遭到刪除時，此模型會有效，因為您可以還原最後兩個版本的其中一個。 不過，若使用者可能會造成資料損毀問題，則 Azure Cosmos DB 可能不會察覺資料損毀情形，進而讓損毀可能覆寫現有備份。 

請在偵測到損毀時立即與客戶支援人員聯繫，同時提供帳戶和容器資訊及大約的損毀時間。 發生損毀情況 (資料刪除/更新) 時，使用者可以採取的另一個動作就是，使用者應該刪除損毀的容器 (集合/圖表/資料表)，以防止損毀的資料覆寫備份。  

下圖說明如何針對意外刪除或更新容器內資料的情況，透過 Azure 入口網站建立容器 (集合/圖表/資料表) 還原支援要求

![針對不當更新或刪除 Cosmos DB 中資料的情況還原容器](./media/online-backup-and-restore/backup-restore-support.png)

針對這類案例進行還原時，資料會還原至另一個帳戶 (會有 "-restored" 尾碼) 和容器。 未就地進行這項還原是為了讓客戶有機會驗證資料，並視需要移動資料。 還原的容器會位於相同的區域中，使用相同的 RU 和索引原則。 

## <a name="next-steps"></a>後續步驟

若要在多個資料中心複寫您的資料庫，請參閱[使用 Cosmos DB 全域散發您的資料](distribute-data-globally.md)。 

若要提出還原要求，請連絡 Azure 支援， [從 Azure 入口網站提出票證](https://portal.azure.com/?#blade/Microsoft_Azure_Support/HelpAndSupportBlade)。

