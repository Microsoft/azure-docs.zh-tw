---
title: 關於線上移轉到 Azure SQL Database 已知問題/移轉限制的文章 | Microsoft Docs
description: 深入了解線上移轉到 Azure SQL Database 的已知問題/移轉限制。
services: database-migration
author: HJToland3
ms.author: jtoland
manager: ''
ms.reviewer: ''
ms.service: database-migration
ms.workload: data-services
ms.custom: mvc
ms.topic: article
ms.date: 08/24/2018
ms.openlocfilehash: 1f8e3ede4140ab5346285f7c247864f8ef8e2d48
ms.sourcegitcommit: f1e6e61807634bce56a64c00447bf819438db1b8
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/24/2018
ms.locfileid: "42889081"
---
# <a name="known-issuesmigration-limitations-with-online-migrations-to-azure-sql-db"></a>線上移轉到 Azure SQL DB 的已知問題/移轉限制

從 SQL Server 線上移轉到 Azure SQL Database 的相關聯已知問題和限制，如下所述。

- 不支援移轉時態表

    **徵兆**

    如果您的來源資料庫包含一或多個時態表，則「完整資料載入」作業期間，資料庫移轉會失敗，您可能會看到下列訊息：

    { "resourceId":"/subscriptions/<subscription id>/resourceGroups/migrateready/providers/Microsoft.DataMigration/services/<DMS Service name>", "errorType":"Database migration error", "errorEvents":"["Capture functionalities could not be set. RetCode: SQL_ERROR SqlState: 42000 NativeError: 13570 Message: [Microsoft][SQL Server Native Client 11.0][SQL Server]The use of replication is not supported with system-versioned temporal table '[Application. Cities]' Line: 1 Column: -1 "]" }
 
   ![時態表錯誤範例](media\known-issues-azure-sql-online\dms-temporal-tables-errors.png)

   **因應措施**

   1. 請使用以下的查詢，在您的來源結構描述中尋找時態表。
        ``` 
       select name,temporal_type,temporal_type_desc,* from sys.tables where temporal_type <>0
        ```
   2. 從 [進行移轉設定] 刀鋒視窗 (您在其中指定要移轉的資料表) 中，排除這些資料表。
   3. 重新執行移轉活動。

    **資源**

    如需詳細資訊，請參閱[時態表](https://docs.microsoft.com/sql/relational-databases/tables/temporal-tables?view=sql-server-2017)一文。
 
- 移轉資料表包含具有 hierarchyid 資料類型的一或多個資料行

    **徵兆**

    您可能會看到 SQL 例外狀況，表示在「完整資料載入」期間「ntext 與 hierarchyid 不相容」：
     
    ![hierarchyid 錯誤範例](media\known-issues-azure-sql-online\dms-hierarchyid-errors.png)

    **因應措施**

    1. 請使用以下查詢，尋找使用者資料表，該資料表包含具有 hierarchyid 資料類型的資料行。

        ``` 
        select object_name(object_id) 'Table name' from sys.columns where system_type_id =240 and object_id in (select object_id from sys.objects where type='U')
        ``` 

    2.  從 [進行移轉設定] 刀鋒視窗 (您在其中指定要移轉的資料表) 中，排除這些資料表。
    3.  重新執行移轉活動。

- 移轉失敗，具有「完整資料載入」或「增量資料同步處理」期間，結構描述中使用中觸發程序的各種完整性違規

    **因應措施**
    1. 請使用以下查詢，尋找來源資料庫中目前使用中的觸發程序：
        ```
        select * from sys.triggers where is_disabled =0
        ```
    2.  使用 [DISABLE TRIGGER (Transact-SQL)](https://docs.microsoft.com/sql/t-sql/statements/disable-trigger-transact-sql?view=sql-server-2017) 一文中提供的步驟，停用來源資料庫中的觸發程序。
    3.  重新執行移轉活動。

- 支援 LOB 資料類型

    **徵兆**

    如果大型物件 (LOB) 資料行的長度大於 32 KB，目標的資料可能會遭到截斷。 您可以使用以下查詢，檢查 LOB 資料行的長度： 

    ``` 
    SELECT max(len(ColumnName)) as LEN from TableName
    ```

    **因應措施**

    如果您有大於 32 KB 的 LOB 資料行，請連絡工程小組：[ dmsfeedback@microsoft.com](mailto:dmsfeedback@microsoft.com)。

- 時間戳記資料行的問題

    **徵兆**

    DMS 不會遷移來源時間戳記值；相反地，DMS 會在目標資料表中產生新的時間戳記值。

    **因應措施**

    如果您需要 DMS 遷移儲存在來源資料表中的確切時間戳記值，請連絡工程小組：[dmsfeedback@microsoft.com](mailto:dmsfeedback@microsoft.com)。

- 資料移轉錯誤不會在資料庫詳細狀態刀鋒視窗上，提供其他詳細資料。

    **徵兆**

    當您在資料庫詳細資料狀態檢視中遇到移轉失敗時，選取頂端功能區上的 [資料移轉錯誤] 連結，可能不會提供移轉失敗的特定其他詳細資料。

     ![資料移轉錯誤沒有詳細資料範例](media\known-issues-azure-sql-online\dms-data-migration-errors-no-details.png)

    **因應措施**

    若要取得特定失敗詳細資料，請遵循下列步驟。

    1.  關閉資料庫詳細狀態刀鋒視窗，以顯示移轉活動畫面。

     ![移轉活動畫面](media\known-issues-azure-sql-online\dms-migration-activity-screen.png)

    2. 選取 [查看錯誤詳細資料]，以檢視特定錯誤訊息，協助您針對移轉錯誤進行疑難排解。
