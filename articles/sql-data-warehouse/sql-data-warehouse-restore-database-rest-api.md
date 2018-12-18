---
title: 還原 Azure SQL 資料倉儲 - REST API | Microsoft Docs
description: 使用 REST API 還原 Azure SQL 資料倉儲。
services: sql-data-warehouse
author: kevinvngo
manager: craigg
ms.service: sql-data-warehouse
ms.topic: conceptual
ms.component: manage
ms.date: 04/17/2018
ms.author: kevin
ms.reviewer: igorstan
ms.openlocfilehash: 71236afd3f72497887f42667f971539ed294bef1
ms.sourcegitcommit: 1fb353cfca800e741678b200f23af6f31bd03e87
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/30/2018
ms.locfileid: "43307531"
---
# <a name="restore-an-azure-sql-data-warehouse-with-rest-apis"></a>以 REST API 還原 Azure SQL 資料倉儲
使用 REST API 還原 Azure SQL 資料倉儲。

## <a name="before-you-begin"></a>開始之前
**請驗證您的 DTU 容量。** 每個 SQL 資料倉儲均由具有預設 [DTU 配額](../sql-database/sql-database-what-is-a-dtu.md)的本機 SQL 伺服器裝載 (例如 myserver.database.windows.net)。  在您還原 SQL 資料倉儲之前，請確認您的 SQL 伺服器有足夠的剩餘 DTU 配額供要還原的資料庫使用。 您可以[建立支援票證](sql-data-warehouse-get-started-create-support-ticket.md)，來要求更多 DTU。

## <a name="restore-an-active-or-paused-data-warehouse"></a>還原作用中或已暫停的資料倉儲
還原資料倉儲：

1. 使用 Get 資料庫還原點作業取得資料庫還原點清單。
2. 使用 [建立資料庫還原要求](https://msdn.microsoft.com/library/azure/dn509571.aspx) 作業來開始還原。
3. 使用 [資料庫作業狀態](https://msdn.microsoft.com/library/azure/dn720371.aspx) 作業來追蹤還原狀態。

> [!NOTE]
> 還原完成後，您可以遵循[在復原之後設定資料庫](../sql-database/sql-database-disaster-recovery.md#configure-your-database-after-recovery)來設定復原的資料倉儲。
> 
> 

## <a name="restore-a-deleted-data-warehouse"></a>還原刪除的資料倉儲
還原刪除的資料倉儲：

1. 使用[列出可還原的已卸除資料庫](https://msdn.microsoft.com/library/azure/dn509562.aspx)作業來列出所有可還原的已刪除資料倉儲。
2. 使用[取得可還原的已卸除資料庫][取得可還原的已卸除資料庫]作業，以取得想要還原的已刪除資料倉儲相關詳細資料。
3. 使用 [建立資料庫還原要求](https://msdn.microsoft.com/library/azure/dn509571.aspx) 作業來開始還原。
4. 使用 [資料庫作業狀態](https://msdn.microsoft.com/library/azure/dn720371.aspx) 作業來追蹤還原狀態。

> [!NOTE]
> 若要在還原完成之後設定資料倉儲，請參閱[在復原之後設定資料庫](../sql-database/sql-database-disaster-recovery.md#configure-your-database-after-recovery)。
> 
> 

## <a name="next-steps"></a>後續步驟
若要深入了解 Azure SQL Database 版本的商務持續性功能，請閱讀 [Azure SQL Database 商務持續性概觀](../sql-database/sql-database-business-continuity.md)。
