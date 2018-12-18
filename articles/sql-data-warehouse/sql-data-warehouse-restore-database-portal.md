---
title: 還原 Azure SQL 資料倉儲 (Azure 入口網站) | Microsoft Docs
description: 還原 Azure SQL 資料倉儲的 Azure 入口網站工作。
services: sql-data-warehouse
author: kevinvngo
manager: craigg
ms.service: sql-data-warehouse
ms.topic: conceptual
ms.component: manage
ms.date: 04/17/2018
ms.author: kevin
ms.reviewer: igorstan
ms.openlocfilehash: 8752bc1977bac75928651a62576272ffc673ee7e
ms.sourcegitcommit: 1fb353cfca800e741678b200f23af6f31bd03e87
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/30/2018
ms.locfileid: "43306295"
---
# <a name="restore-azure-sql-data-warehouse-portal"></a>還原 Azure SQL 資料倉儲 (入口網站)
> [!div class="op_single_selector"]
> * [概觀][Overview]
> * [入口網站][Portal]
> * [PowerShell][PowerShell]
> * [REST][REST]
>
>
在本文中，您將了解如何使用 Azure 入口網站來還原 Azure SQL 資料倉儲。

## <a name="before-you-begin"></a>開始之前
**請驗證您的 DTU 容量。** SQL 資料倉儲的每個執行個體均由具有預設資料配額單位 (DTU) 的 SQL Database 裝載 (例如，myserver.database.windows.net)。 在您還原 SQL 資料倉儲之前，請確認您的 SQL 伺服器有足夠的剩餘 DTU 配額可供要還原的資料庫使用。 若要了解如何計算所需 DTU 配額或要求更多 DTU，請參閱 [要求 DTU 配額變更][Request a DTU quota change]。

## <a name="restore-an-active-or-paused-database"></a>還原作用中或已暫停的資料庫
還原資料庫：

1. 登入 [Azure 入口網站][Azure portal]。
2. 在左窗格中，選取 [瀏覽]，然後選取 [SQL Server]。

    ![選取 [瀏覽] > [SQL Server]](./media/sql-data-warehouse-restore-database-portal/01-browse-for-sql-server.png)
3. 尋找您的伺服器，然後選取它。

    ![選取您的伺服器](./media/sql-data-warehouse-restore-database-portal/01-select-server.png)
4. 尋找您想要還原的 SQL 資料倉儲的執行個體，然後選取它。

    ![選取要還原的 SQL 資料倉儲的執行個體](./media/sql-data-warehouse-restore-database-portal/01-select-active-dw.png)
5. 在 [資料倉儲] 刀鋒視窗頂端，選取 [還原]。

    ![選取 [還原]](./media/sql-data-warehouse-restore-database-portal/01-select-restore-from-active.png)
6. 指定新的 **資料庫名稱**。
7. 選取最新的 **還原點**。

   確定您選擇了最新的還原點。 還原點會以國際標準時間 (UTC) 顯示，因此預設選項可能不是最新的還原點。

      ![選取還原點](./media/sql-data-warehouse-restore-database-portal/01-restore-blade-from-active.png)
8. 選取 [確定] 。
9. 資料庫還原程序將會開始，您可以使用 [通知] 來監視此程序。

> [!NOTE]
> 還原完成後，您可以遵循 [在復原之後設定資料庫][Configure your database after recovery]來設定復原的資料庫。
>
>

## <a name="restore-a-deleted-database"></a>還原已刪除的資料庫
還原已刪除的資料庫：

1. 登入 [Azure 入口網站][Azure portal]。
2. 在左窗格中，選取 [瀏覽]，然後選取 [SQL Server]。

    ![選取 [瀏覽] > [SQL Server]](./media/sql-data-warehouse-restore-database-portal/01-browse-for-sql-server.png)
3. 尋找您的伺服器，然後選取它。

    ![選取您的伺服器](./media/sql-data-warehouse-restore-database-portal/02-select-server.png)
4. 在伺服器的刀鋒視窗上捲動至 [作業] 區段。
5. 選取 [已刪除的資料庫] 圖格。

    ![選取 [已刪除的資料庫] 圖格](./media/sql-data-warehouse-restore-database-portal/02-select-deleted-dws.png)
6. 選取您想要還原的已刪除資料庫。

    ![選取要還原的資料庫](./media/sql-data-warehouse-restore-database-portal/02-select-deleted-dw.png)
7. 指定新的 **資料庫名稱**。

    ![新增資料庫的名稱](./media/sql-data-warehouse-restore-database-portal/02-restore-blade-from-deleted.png)
8. 選取 [確定] 。
9. 資料庫還原程序將會開始，您可以使用 [通知] 來監視此程序。

> [!NOTE]
> 若要在還原完成之後設定資料庫，請參閱 [在復原之後設定資料庫][Configure your database after recovery]。
>
>

## <a name="next-steps"></a>後續步驟
若要深入了解 Azure SQL Database 版本的商務持續性功能，請閱讀 [Azure SQL Database 商務持續性概觀][Azure SQL Database business continuity overview]。

<!--Image references-->

<!--Article references-->
[Azure SQL Database business continuity overview]: ../sql-database/sql-database-business-continuity.md
[Overview]: ./sql-data-warehouse-restore-database-overview.md
[Portal]: ./sql-data-warehouse-restore-database-portal.md
[PowerShell]: ./sql-data-warehouse-restore-database-powershell.md
[REST]: ./sql-data-warehouse-restore-database-rest-api.md
[Configure your database after recovery]: ../sql-database/sql-database-disaster-recovery.md#configure-your-database-after-recovery
[Request a DTU quota change]: ./sql-data-warehouse-get-started-create-support-ticket.md

<!--MSDN references-->

<!--Blog references-->

<!--Other Web references-->
[Azure portal]: https://portal.azure.com/
