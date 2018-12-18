---
title: 線上移轉到適用於 MySQL 的 Azure 資料庫已知問題/移轉限制相關文章 | Microsoft Docs
description: 深入了解線上移轉到適用於 MySQL 的 Azure 資料庫已知問題/移轉限制。
services: database-migration
author: HJToland3
ms.author: scphang
manager: ''
ms.reviewer: ''
ms.service: database-migration
ms.workload: data-services
ms.custom: mvc
ms.topic: article
ms.date: 10/09/2018
ms.openlocfilehash: 6e82c10d8e9109279045095c1b856520245a5a6f
ms.sourcegitcommit: 55952b90dc3935a8ea8baeaae9692dbb9bedb47f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/09/2018
ms.locfileid: "48884505"
---
# <a name="known-issuesmigration-limitations-with-online-migrations-to-azure-db-for-mysql"></a>線上移轉到適用於 MySQL 的 Azure 資料庫已知問題/移轉限制

從 MySQL 線上移轉到適用於 MySQL 的 Azure 資料庫相關聯已知問題和限制，如下所述。 

## <a name="online-migration-configuration"></a>線上移轉組態
- 來源 MySQL 伺服器版本必須是 5.6.35、5.7.18 或更新版本
- 適用於 MySQL 的 Azure 資料庫支援：
    - MySQL 社群版
    - InnoDB 引擎
- 相同版本移轉。 不支援將 MySQL 5.6 移轉到適用於 MySQL 5.7 的 Azure 資料庫。
- 在 my.ini (Windows) 或 my.cnf (Unix) 中啟用二進位記錄
    - 將 Server_id 設為大於或等於 1 的任何數字，例如，Server_id=1 (僅適用於 MySQL 5.6)
    - 設定 log-bin = <path> (僅適用於 MySQL 5.6)
    - 設定 binlog_format = row
    - Expire_logs_days = 5 (建議 - 僅適用於 MySQL 5.6)
- 使用者必須擁有 ReplicationAdmin 角色。
- 針對來源 MySQL 資料庫所定義的定序，與在目標適用於 MySQL 的 Azure 資料庫所定義的定序相同。
- 來源 MySQL 資料庫與適用於 MySQL 的 Azure 資料庫中目標資料庫，兩者之間的結構描述必須相符。
- 適用於 MySQL 的 Azure 資料庫中結構描述不能具有外部索引鍵。 請使用以下查詢來卸除外部索引鍵：
    ```
    SET group_concat_max_len = 8192;
    SELECT SchemaName, GROUP_CONCAT(DropQuery SEPARATOR ';\n') as DropQuery, GROUP_CONCAT(AddQuery SEPARATOR ';\n') as AddQuery
    FROM
    (SELECT 
    KCU.REFERENCED_TABLE_SCHEMA as SchemaName, KCU.TABLE_NAME, KCU.COLUMN_NAME,
        CONCAT('ALTER TABLE ', KCU.TABLE_NAME, ' DROP FOREIGN KEY ', KCU.CONSTRAINT_NAME) AS DropQuery,
        CONCAT('ALTER TABLE ', KCU.TABLE_NAME, ' ADD CONSTRAINT ', KCU.CONSTRAINT_NAME, ' FOREIGN KEY (`', KCU.COLUMN_NAME, '`) REFERENCES `', KCU.REFERENCED_TABLE_NAME, '` (`', KCU.REFERENCED_COLUMN_NAME, '`) ON UPDATE ',RC.UPDATE_RULE, ' ON DELETE ',RC.DELETE_RULE) AS AddQuery
        FROM INFORMATION_SCHEMA.KEY_COLUMN_USAGE KCU, information_schema.REFERENTIAL_CONSTRAINTS RC
        WHERE
          KCU.CONSTRAINT_NAME = RC.CONSTRAINT_NAME
          AND KCU.REFERENCED_TABLE_SCHEMA = RC.UNIQUE_CONSTRAINT_SCHEMA
      AND KCU.REFERENCED_TABLE_SCHEMA = ['schema_name']) Queries
      GROUP BY SchemaName;
    ```

    在查詢結果中執行卸除外部索引鍵 (這是第二個資料行)。
- 適用於 MySQL 的 Azure 資料庫中結構描述不能具有任何觸發程序。 若要在目標資料庫中卸除觸發程序：
    ```
    SELECT Concat('DROP TRIGGER ', Trigger_Name, ';') FROM  information_schema.TRIGGERS WHERE TRIGGER_SCHEMA = 'your_schema';
    ```

## <a name="datatype-limitations"></a>資料類型限制
- **限制**：如果在來源 MySQL 資料庫中有 JSON 資料類型，持續同步期間移轉會失敗。

    **因應措施**：在來源 MySQL 資料庫中將 JSON 資料類型修改為中長文字或長文字。

- **限制**：如果資料表中沒有主索引鍵，持續同步會失敗。
 
    **因應措施**：暫時設定資料表的主索引鍵讓移轉繼續。 您可以在資料移轉完成之後，移除主索引鍵。

## <a name="lob-limitations"></a>LOB 限制
大型物件 (LOB) 資料行是可能會在大小上增長的資料行。 針對 MySQL，中長文字、長文字、Blob、Mediumblob、Longblob 等等，是一些 LOB 的資料類型。

- **限制**：如果 LOB 資料類型作為主索引鍵，移轉將會失敗。

    **因應措施**：使用不是 LOB 的其他資料類型或資料行，取代主索引鍵。

- **限制**：如果大型物件 (LOB) 資料行的長度大於 32 KB，目標的資料可能會遭到截斷。 您可以使用以下查詢，檢查 LOB 資料行的長度：
    ```
    SELECT max(length(description)) as LEN from catalog;
    ```

    **因應措施**：如果您有大於 32 KB 的 LOB 物件，請連絡工程小組：[dmsfeedback@microsoft.com](mailto:dmsfeedback@microsoft.com)。 

## <a name="other-limitations"></a>其他限制
- 不支援在開頭或結尾具有左右大括弧 {  } 的密碼字串。 限制同時適用於連線到來源 MySQL，以及與目標適用於 MySQL 的 Azure 資料庫。
- 不支援下列 DDL：
    - 所有資料分割 DDL
    - 卸除資料表
    - 重新命名資料表
- 不支援使用 alter table <table_name> add column <column_name> 陳述式，將資料行新增至資料表開頭或中間。 alter table <table_name> add column <column_name> 會在資料表結尾處新增資料行。
- 不支援只在資料行資料的一部分建立索引。 下列陳述式是僅使用資料行資料的一部分，建立索引的範例：
    ``` 
    CREATE INDEX partial_name ON customer (name(10));
    ```

- 在 DMS 中，於單一移轉活動中遷移的資料庫上限為四個。
