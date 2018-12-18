---
title: 儲存多達 10 年的 Azure SQL Database 備份 | Microsoft Docs
description: 了解 Azure SQL Database 如何支援儲存多達 10 年的完整資料庫備份。
services: sql-database
ms.service: sql-database
ms.subservice: operations
ms.custom: ''
ms.devlang: ''
ms.topic: conceptual
author: anosov1960
ms.author: sashan
ms.reviewer: carlrab
manager: craigg
ms.date: 07/16/2018
ms.openlocfilehash: b2d4166c25014416efbb7392acda3f3f028b4fa9
ms.sourcegitcommit: 51a1476c85ca518a6d8b4cc35aed7a76b33e130f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/25/2018
ms.locfileid: "47162033"
---
# <a name="store-azure-sql-database-backups-for-up-to-10-years"></a>儲存多達 10 年的 Azure SQL Database 備份

許多應用程式具有法規、相容性或其他商務用途，需要您保留 Azure SQL Database [自動備份](sql-database-automated-backups.md)所提供超過 7-35 天的資料庫備份。 使用長期保留 (LTR) 功能可讓您將指定的 SQL Database 完整備份儲存在 [RA-GRS](../storage/common/storage-redundancy-grs.md#read-access-geo-redundant-storage) blob 儲存體中多達 10 年。 之後您可以將任何備份還原為新的資料庫。

## <a name="how-sql-database-long-term-retention-works"></a>SQL Database 長期保留如何運作

長期備份保留運用針對時間點還原 (PITR) 建立的[自動 SQL Database 備份](sql-database-automated-backups.md)。 您可以為每個 SQL Database 設定長期保留原則，並指定要將備份複製至長期儲存體的頻率。 為了獲得這樣的彈性，您可以用這四個參數的組合定義原則：每週備份保留 (W)、每月備份保留 (M)、每年備份的保留期 (Y) 及年度週次 (WeekOfYear)。 如果指定 W，每週一次的備份會複製到長期儲存體。 如果指定 M，每月第一週進行的一次備份會複製到長期儲存體。 如果指定 Y，在以 WeekOfYear 指定的當週進行的一次備份會複製到長期儲存體。 在長期儲存體中，每個備份會保留以這些參數所指定的期間。 

範例：

-  W=0, M=0, Y=5, WeekOfYear=3

   每年的第 3 個完整備份會保留 5 年。
- W=0, M=3, Y=0

   每月的第 1 個完整備份會保留 3 個月。

- W=12, M=0, Y=0

   每個每週完整備份皆會保留 12 週。

- W=6, M=12, Y=10, WeekOfYear=16

   每個每週完整備份皆會保留 6 個月。 但每月的第 1 個完整備份除外，這個備份會保留 12 個月。 以及每年的第 16 週的完整備份除外，這個備份會保留 10 年。 

下表說明以下原則的長期備份日程和到期日：

W=12 週 (84 天)、M=12 個月 (365 天)、Y=10 年 (3650 天)、WeekOfYear=15 (4 月 15 日後那一周)

   ![ITR 範例](./media/sql-database-long-term-retention/ltr-example.png)


 
如果您修改上述原則，設定成 W = 0 (沒有每週備份)，複製備份的日程會變更為上表中醒目提示的日期。 用來保留這些備份所需的儲存體數量會跟著減少。 

> [!NOTE]
1. LTR 複製是由 Azure 儲存體服務執行，因此複製程序對現有資料庫的效能沒有影響。
2. 該原則適用於未來的備份。 例如 如果在設定原則時指定的 WeekOfYear 為過去時間，則將在下一個年度建立第一個 LTR 備份。 
3. 若要從 LTR 儲存體還原資料庫，可以依時間戳記選取特定備份。   可將資料庫還原至原始資料庫相同訂用帳戶底下的任何現有伺服器。 
> 

## <a name="geo-replication-and-long-term-backup-retention"></a>異地複寫和長期備份保留

如果您使用主動式異地複寫或容錯移轉群組作為商務持續性解決方案，您應為最終容錯移轉最好準備，並在不同地區的次要資料庫中設定相同 LTR 原則。 由於備份不是從次要區域產生的，因此這不會增加 LTR 的儲存體成本。 只有當次要變成主要時，備份才會建立。 如此一來，當觸發容錯移轉，以及主要區域移到次要區域時，您可保證 LTR 備份可以不受中斷地產生。 

> [!NOTE]
當原本的主要資料庫從造成容錯移轉的中斷情況中復原時，它會變成新的次要資料庫。 因此，備份的建立不會繼續，而且現有的 LTR 原則將不會生效，除非它再次變成主要資料庫。 
> 

## <a name="configure-long-term-backup-retention"></a>設定長期備份保留期

若要了解如何使用 Azure 入口網站或使用 PowerShell 設定長期保留，請參閱[設定長期備份保留](sql-database-long-term-backup-retention-configure.md)。

## <a name="next-steps"></a>後續步驟

因為資料庫備份可保護資料免於意外損毀或刪除，是商務持續性和災害復原策略中不可或缺的一環。 若要深入了解其他 SQL Database 商務持續性解決方案，請參閱[商務持續性概觀](sql-database-business-continuity.md)。
