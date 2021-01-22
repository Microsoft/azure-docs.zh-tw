---
title: 快速入門：調整專用 SQL 集區 (先前稱為 SQL DW) 中的計算 - T-SQL
description: 使用 T-SQL 和 SQL Server Management Studio (SSMS) 調整專用 SQL 集區 (先前為 SQL DW) 中的計算。 擴增計算以提升效能，或將計算調整回來以節省成本。
services: synapse-analytics
author: Antvgski
manager: craigg
ms.service: synapse-analytics
ms.topic: quickstart
ms.subservice: sql-dw
ms.date: 04/17/2018
ms.author: anvang
ms.reviewer: igorstan
ms.custom: seo-lt-2019, azure-synapse
ms.openlocfilehash: ffffeb38aeb9d1f01f376d58a52323bb7b84b306
ms.sourcegitcommit: b39cf769ce8e2eb7ea74cfdac6759a17a048b331
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 01/22/2021
ms.locfileid: "98676318"
---
# <a name="quickstart-scale-compute-for-dedicated-sql-pool-formerly-sql-dw-in-azure-synapse-analytics-using-t-sql"></a>快速入門：使用 T-SQL 調整 Azure Synapse Analytics 中的專用 SQL 集區 (先前稱為 SQL DW) 的計算

使用 T-SQL 和 SQL Server Management Studio (SSMS) 調整專用 SQL 集區 (先前為 SQL DW) 中的計算。 [擴增計算](sql-data-warehouse-manage-compute-overview.md)以提升效能，或將計算調整回來以節省成本。

如果您沒有 Azure 訂用帳戶，請在開始前建立[免費帳戶](https://azure.microsoft.com/free/)。

## <a name="before-you-begin"></a>開始之前

下載並安裝最新版的 [SQL Server Management Studio](/sql/ssms/download-sql-server-management-studio-ssms?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest&preserve-view=true) (SSMS)。

## <a name="create-a-dedicated-sql-pool-formerly-sql-dw"></a>建立專用 SQL 集區 (先前稱為 SQL DW)

使用 [快速入門：建立與連線 - 入口網站](create-data-warehouse-portal.md)建立名為 **mySampleDataWarehouse** 的專用 SQL 集區 (先前稱為 SQL DW)。 完成快速入門以確定您擁有防火牆規則，並可從 SQL Server Management Studio 內連線至專用 SQL 集區 (先前稱為 SQL DW)。

## <a name="connect-to-the-server-as-server-admin"></a>以伺服器系統管理員身分連線到伺服器

本節使用 [SQL Server Management Studio](/sql/ssms/download-sql-server-management-studio-ssms?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest&preserve-view=true) (SSMS) 建立對 Azure SQL Server 的連線。

1. 開啟 SQL Server Management Studio。

2. 在 [連線至伺服器]  對話方塊中，輸入下列資訊：

   | 設定       | 建議的值 | 描述 |
   | ------------ | ------------------ | ------------------------------------------------- |
   | 伺服器類型 | 資料庫引擎 | 這是必要值 |
   | 伺服器名稱 | 完整伺服器名稱 | 範例如下：**mySampleDataWarehouseservername.database.windows.net**。 |
   | 驗證 | SQL Server 驗證 | SQL 驗證是本教學課程中設定的唯一驗證類型。 |
   | 登入 | 伺服器系統管理員帳戶 | 您在建立伺服器時所指定的帳戶。 |
   | 密碼 | 伺服器系統管理員帳戶的密碼 | 您在建立伺服器時所指定的密碼。 |

    ![連線至伺服器](./media/quickstart-scale-compute-tsql/connect-to-server.png)

3. 按一下 [ **連接**]。 [物件總管] 視窗會在 SSMS 中開啟。

4. 在 [物件總管] 中展開 [資料庫]  。 然後，展開 [mySampleDataWarehouse]  ，以檢視新資料庫中的物件。

    ![資料庫物件](./media/quickstart-scale-compute-tsql/connected.png)

## <a name="view-service-objective"></a>檢視服務目標

服務目標設定包含資料倉儲的專用 SQL 集區 (先前稱為 SQL DW) 單位數目。

若要檢視專用 SQL 集區 (先前稱為 SQL DW) 目前的資料倉儲單位：

1. 在 **mySampleDataWarehouseservername.database.windows.net** 的連線下，展開 [系統資料庫]  。
2. 以滑鼠右鍵按一下 [主要]  ，然後選取 [新增查詢]  。 隨即開啟 [新增查詢] 視窗。
3. 執行下列查詢，以從 sys.database_service_objectives 動態管理檢視中選取。

    ```sql
    SELECT
        db.name [Database]
    ,    ds.edition [Edition]
    ,    ds.service_objective [Service Objective]
    FROM
         sys.database_service_objectives ds
    JOIN
        sys.databases db ON ds.database_id = db.database_id
    WHERE
        db.name = 'mySampleDataWarehouse'
    ```

4. 下列結果顯示 **mySampleDataWarehouse** 具有 DW400 的服務目標。

    ![iew-current-dwu](./media/quickstart-scale-compute-tsql/view-current-dwu.png)

## <a name="scale-compute"></a>調整計算

在專用 SQL 集區 (先前稱為 SQL DW) 中，您可以藉由調整資料倉儲單位來增加或減少計算資源。 [建立與連線 - 入口網站](create-data-warehouse-portal.md)已建立 **mySampleDataWarehouse**，並以 400 DWU 加以初始化。 下列步驟會調整 **mySampleDataWarehouse** 的 DWU。

若要變更資料倉儲單位：

1. 以滑鼠右鍵按一下 [主要]  ，然後選取 [新增查詢]  。
2. 使用 [ALTER DATABASE](/sql/t-sql/statements/alter-database-azure-sql-database?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest&preserve-view=true) T-SQL 陳述式來修改服務目標。 執行下列查詢，將服務目標變更為 DW300。

    ```Sql
    ALTER DATABASE mySampleDataWarehouse
    MODIFY (SERVICE_OBJECTIVE = 'DW300c');
    ```

## <a name="monitor-scale-change-request"></a>監視級別變更要求

若要查看先前變更要求的進度，您可以使用 `WAITFORDELAY` T-SQL 語法來輪詢 sys.dm_operation_status 動態管理檢視 (DMV)。

若要輪詢服務物件變更狀態：

1. 以滑鼠右鍵按一下 [主要]  ，然後選取 [新增查詢]  。
2. 執行下列查詢，以輪詢 sys.dm_operation_status DMV。

    ```sql
    WHILE
    (
        SELECT TOP 1 state_desc
        FROM sys.dm_operation_status
        WHERE
            1=1
            AND resource_type_desc = 'Database'
            AND major_resource_id = 'mySampleDataWarehouse'
            AND operation = 'ALTER DATABASE'
        ORDER BY
            start_time DESC
    ) = 'IN_PROGRESS'
    BEGIN
        RAISERROR('Scale operation in progress',0,0) WITH NOWAIT;
        WAITFOR DELAY '00:00:05';
    END
    PRINT 'Complete';
    ```

3. 產生的輸出會顯示輪詢狀態的記錄。

    ![作業狀態](./media/quickstart-scale-compute-tsql/polling-output.png)

## <a name="check-dedicated-sql-pool-formerly-sql-dw-state"></a>檢查專用 SQL 集區 (先前稱為 SQL DW) 狀態

當專用 SQL 集區 (先前稱為 SQL DW) 暫停時，您無法使用 T-SQL 與其連線。 若要查看專用 SQL 集區 (先前稱為 SQL DW) 的目前狀態，您可以使用 PowerShell Cmdlet。 如需範例，請參閱[檢查專用 SQL 集區 (先前稱為 SQL DW) 狀態 - PowerShell](quickstart-scale-compute-powershell.md#check-data-warehouse-state)。

## <a name="check-operation-status"></a>檢查作業狀態

若要傳回針對專用 SQL 集區 (先前稱為 SQL DW) 進行之各種管理作業的相關資訊，請對 [sys.dm_operation_status](/sql/relational-databases/system-dynamic-management-views/sys-dm-operation-status-azure-sql-database?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest&preserve-view=true) DMV 執行下列查詢。 例如，它會傳回作業和作業狀態 (IN_PROGRESS 或 COMPLETED)。

```sql
SELECT *
FROM
    sys.dm_operation_status
WHERE
    resource_type_desc = 'Database'
AND
    major_resource_id = 'mySampleDataWarehouse'
```

## <a name="next-steps"></a>後續步驟

您現已了解如何調整專用 SQL 集區 (先前稱為 SQL DW) 的計算。 若要深入了解 Azure Synapse Analytics，請繼續進行載入資料的教學課程。

> [!div class="nextstepaction"]
>[將資料載入專用 SQL 集區中](./load-data-from-azure-blob-storage-using-copy.md)