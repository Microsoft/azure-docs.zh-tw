---
title: 啟用 Azure SQL Database 的自動調整 | Microsoft Docs
description: 您可以輕鬆在 Azure SQL Database 上啟用自動調整。
services: sql-database
author: danimir
manager: craigg
ms.service: sql-database
ms.custom: monitor & tune
ms.topic: conceptual
ms.date: 04/01/2018
ms.author: v-daljep
ms.reviewer: carlrab
ms.openlocfilehash: d825ddd75bfc829aa82bab4cede0381dda2d36da
ms.sourcegitcommit: e2ea404126bdd990570b4417794d63367a417856
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/14/2018
ms.locfileid: "45575284"
---
# <a name="enable-automatic-tuning"></a>啟用自動微調

Azure SQL Database 是自動受控的資料服務，會不斷地監視您的查詢，並識別您為改善工作負載效能可以執行的動作。 您可以檢閱建議並加以手動套用，或讓 Azure SQL Database 自動套用矯正措施 - 這稱為**模式**。

您可以透過 [Azure 入口網站](sql-database-automatic-tuning-enable.md#azure-portal)、[REST API](sql-database-automatic-tuning-enable.md#rest-api) 呼叫和 [T-SQL](sql-database-automatic-tuning-enable.md#t-sql) 命令，在伺服器或資料庫層級啟用自動調整。

## <a name="enable-automatic-tuning-on-server"></a>在伺服器上啟用自動調整
在伺服器層級上，您可以選擇繼承「Azure 預設值」的自動調整設定，或不繼承設定。 Azure 預設值已啟用 FORCE_LAST_GOOD_PLAN 和 CREATE_INDEX，且已停用 DROP_INDEX。

### <a name="azure-portal"></a>Azure 入口網站
若要在 Azure SQL Database 邏輯**伺服器**上啟用自動調整，請導覽至 Azure 入口網站中的伺服器，然後選取功能表中的 [自動調整]。

![伺服器](./media/sql-database-automatic-tuning-enable/server.png)

> [!NOTE]
> 請注意，此時 [DROP_INDEX] 選項與使用分割區切換和索引提示的應用程式不相容，而且不應該在這些情況下啟用。
>

依序選取您想要啟用的自動調整以及 [套用]。

伺服器上的自動調整選項會套用到此伺服器上的所有資料庫。 根據預設，所有資料庫會都繼承其父伺服器的組態，但這可加以覆寫並針對每個資料庫個別加以指定。

### <a name="rest-api"></a>REST API

若要深入了解如何使用 REST API 在伺服器上啟用自動調整，請參閱 [SQL Server 自動調整的 UPDATE 和 GET HTTP 方法](https://docs.microsoft.com/rest/api/sql/serverautomatictuning)。


## <a name="enable-automatic-tuning-on-an-individual-database"></a>在個別的資料庫上啟用自動調整

Azure SQL Database 可讓您個別指定每個資料庫的自動調整設定。 在資料庫層級上，您可以選擇繼承父伺服器「Azure 預設值」的自動調整設定，或不繼承設定。 Azure 預設值會設為已啟用 FORCE_LAST_GOOD_PLAN 和 CREATE_INDEX，且已停用 DROP_INDEX。

> [!NOTE]
> 一般建議是在**伺服器層級**管理自動調整設定，以便在每一個資料庫上自動套用相同的組態設定。 只有在您要求該資料庫的設定不同於繼承自相同伺服器的其他設定時，設定自動調整個別的資料庫。
>

### <a name="azure-portal"></a>Azure 入口網站

若要在**單一資料庫**上啟用自動調整，請導覽至 Azure 入口網站中的資料庫，然後選取 [自動調整]。

您可以針對每個資料庫分開設定個別的自動調整設定。 您可以手動設定個別的自動調整選項，或指定選項從伺服器繼承其設定。

![資料庫](./media/sql-database-automatic-tuning-enable/database.png)

請注意，此時 [DROP_INDEX] 選項與使用分割區切換和索引提示的應用程式不相容，而且不應該在這些情況下啟用。

選取所需的組態後，按一下 [套用]。

### <a name="rest-api"></a>Rest API

若要深入了解如何使用 REST API 在單一資料庫上啟用自動調整，請參閱 [SQL Database 自動調整的 UPDATE 和 GET HTTP 方法](https://docs.microsoft.com/rest/api/sql/databaseautomatictuning)。

### <a name="t-sql"></a>T-SQL

若要透過 T-SQL 在單一資料庫上啟用自動調整，請連線到該資料庫並執行下列查詢：

   ```T-SQL
   ALTER DATABASE current SET AUTOMATIC_TUNING = AUTO | INHERIT | CUSTOM
   ```
   
將自動調整設定成 [AUTO]，將會套用 Azure 預設值。 設定成 [INHERIT]，將會從父伺服器繼承自動調整設定。 若選擇 [CUSTOM]，您將需要手動設定自動調整。

若要透過 T-SQL 設定個別的自動調整選項，請連線到資料庫並執行如下的查詢：

   ```T-SQL
   ALTER DATABASE current SET AUTOMATIC_TUNING (FORCE_LAST_GOOD_PLAN = ON, CREATE_INDEX = DEFAULT, DROP_INDEX = OFF)
   ```
   
將個別的調整選項設定成 [ON]，將會覆寫該資料庫所繼承的任何設定，並啟用該調整選項。 將它設定成 [OFF]，也會覆寫資料庫所繼承的任何設定，並停用該調整選項。 指定為 [DEFAULT] 的自動調整選項，將會繼承資料庫層級自動調整設定的組態。  

若要深入了解如何使用 T-SQL 選項來設定自動調整，請參閱 [SQL Database 邏輯伺服器的 ALTER DATABASE SET 選項 (Transact-SQL)](https://docs.microsoft.com/sql/t-sql/statements/alter-database-transact-sql-set-options?view=sql-server-2017&tabs=sqldbls#arguments-1)。

## <a name="disabled-by-the-system"></a>被系統停用
自動調整會監視它在資料庫上採取的所有動作，而且在某些情況下，它可以判斷自動調整無法適當地在資料庫上運作。 在此情況下，調整選項將會被系統停用。 在大部分情況下，發生此問題的原因是因為特定資料庫上未啟用查詢資料存放區，或是查詢存放區處於唯讀的狀態。

## <a name="configure-automatic-tuning-e-mail-notifications"></a>設定自動調整電子郵件通知

請參閱[自動調整電子郵件通知](sql-database-automatic-tuning-email-notifications.md)指南。

## <a name="next-steps"></a>後續步驟
* 閱讀[自動調整文章](sql-database-automatic-tuning.md)，進一步了解自動調整，以及它如何協助您改善效能。
* 如需 Azure SQL Database 效能建議的概觀，請參閱[效能建議](sql-database-advisor.md)。
* 請參閱[查詢效能深入解析](sql-database-query-performance.md)，以了解如何檢視排名最前面查詢的效能影響。
