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
ms.date: 09/22/2018
ms.openlocfilehash: e30ffe2dd79d55e856ef297608745b60578cf7e7
ms.sourcegitcommit: f10653b10c2ad745f446b54a31664b7d9f9253fe
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/18/2018
ms.locfileid: "46131028"
---
# <a name="known-issuesmigration-limitations-with-online-migrations-to-azure-db-for-postgresql"></a>線上移轉到適用於 PostgreSQL 的 Azure DB 的已知問題/移轉限制

下列小節描述從 PostgreSQL 線上移轉到適用於 PostgreSQL 的 Azure 資料庫的相關已知問題和限制。 

## <a name="online-migration-configuration"></a>線上移轉組態
- 來源 PostgreSQL 伺服器必須執行 9.5.11、9.6.7 或 10.3 版或更新版本。 如需詳細資訊，請參閱[支援的 PostgreSQL 資料庫版本](1.2.%09https:/docs.microsoft.com/azure/postgresql/concepts-supported-versions)一文。
- 僅支援相同版本之間的移轉。 例如，不支援將 PostgreSQL 9.5.11 移轉至適用於 PostgreSQL 9.6.7 的 Azure 資料庫。
- 若要在**來源 PostgreSQL postgresql.conf** 檔案中啟用邏輯複寫，請設定下列參數：
    - **wal_level** = logical
    - **max_replication_slots** = [要移轉之資料庫的最大數目]；如果您要移轉 4 個資料庫，請將值設定為 4
    - **max_wal_senders** = [要並行執行之資料庫的最大數目]；建議的值為 10
- 將 DMS 代理程式 IP 新增到來源 PostgresSQL pg_hba.conf
    1. 完成 DMS 執行個體的佈建之後，請記下 DMS IP 位址。
    2. 將該 IP 位址新增到 pg_hba.conf 檔案，如下所示：

        host    all     172.16.136.18/10    md5  host    replication postgres    172.16.136.18/10    md5

- 使用者在裝載來源資料庫的伺服器上必須具有進階使用者權限
- 除了在來源資料庫結構描述中必須具有 ENUM 之外，來源和目標資料庫結構描述也必須相符。
- 目標適用於 PostgreSQL 的 Azure 資料庫中的結構描述不可有外部索引鍵。 請使用以下查詢來卸除外部索引鍵：

    ```
                                SELECT Queries.tablename
           ,concat('alter table ', Queries.tablename, ' ', STRING_AGG(concat('DROP CONSTRAINT ', Queries.foreignkey), ',')) as DropQuery
                ,concat('alter table ', Queries.tablename, ' ', 
                                                STRING_AGG(concat('ADD CONSTRAINT ', Queries.foreignkey, ' FOREIGN KEY (', column_name, ')', 'REFERENCES ', foreign_table_name, '(', foreign_column_name, ')' ), ',')) as AddQuery
        FROM
        (SELECT
        tc.table_schema, 
        tc.constraint_name as foreignkey, 
        tc.table_name as tableName, 
        kcu.column_name, 
        ccu.table_schema AS foreign_table_schema,
        ccu.table_name AS foreign_table_name,
        ccu.column_name AS foreign_column_name 
    FROM 
        information_schema.table_constraints AS tc 
        JOIN information_schema.key_column_usage AS kcu
          ON tc.constraint_name = kcu.constraint_name
          AND tc.table_schema = kcu.table_schema
        JOIN information_schema.constraint_column_usage AS ccu
          ON ccu.constraint_name = tc.constraint_name
          AND ccu.table_schema = tc.table_schema
    WHERE constraint_type = 'FOREIGN KEY') Queries
      GROUP BY Queries.tablename;
    
    ```

    在查詢結果中執行卸除外部索引鍵 (這是第二個資料行)。

- 目標適用於 PostgreSQL 的 Azure 資料庫中的結構描述不可有任何觸發程序。 請使用下列命令來停用目標資料庫中的觸發程序：

     ```
    SELECT Concat('DROP TRIGGER ', Trigger_Name, ';') FROM  information_schema.TRIGGERS WHERE TRIGGER_SCHEMA = 'your_schema';
     ```

## <a name="datatype-limitations"></a>資料類型限制

- **限制**：如果來源 PostgreSQL 資料庫中有 ENUM 資料類型，移轉在持續同步期間將會失敗。

    **因應措施**：將 ENUM 資料類型修改成與適用於 PostgreSQL 的 Azure 資料庫中不同的字元。

- **限制**：如果資料表中沒有主索引鍵，持續同步會失敗。

    **因應措施**：暫時為資料表設定主索引鍵來讓移轉繼續。 您可以在資料移轉完成之後，移除主索引鍵。

## <a name="lob-limitations"></a>LOB 限制
大型物件 (LOB) 資料行是可能會變大的資料行。 針對 PostgreSQL，LOB 資料類型的範例包括 XML、JSON、IMAGE、TEXT 等等。

- **限制**：如果 LOB 資料類型作為主索引鍵，移轉將會失敗。

    **因應措施**：使用不是 LOB 的其他資料類型或資料行來取代主索引鍵。

- **限制**：如果大型物件 (LOB) 資料行的長度大於 32 KB，目標的資料可能會遭到截斷。 您可以使用以下查詢，檢查 LOB 資料行的長度：

    ```
    SELECT max(length(cast(body as text))) as body FROM customer_mail
    ```

    **因應措施**：如果您有大於 32 KB 的 LOB 物件，請連絡工程小組：[dmsfeedback@microsoft.com](mailto:dmsfeedback@microsoft.com)。

- **限制**：如果資料表中有 LOB 資料行，且沒有為資料表設定主索引鍵，則系統可能不會針對此資料表移轉資料。

    **因應措施**：暫時為資料表設定主索引鍵來讓移轉繼續。 您可以在資料移轉完成之後，移除主索引鍵。

## <a name="postgresql10-workaround"></a>PostgreSQL10 因應措施
由於 PostgreSQL 10.x 針對 pg_xlog 資料夾名稱做出數個變更，因而造成移轉沒有如預期般執行。 如果您要從 PostgreSQL 10.x 移轉到適用於 PostgreSQL 10.3 的 Azure 資料庫，請在來源 PostgreSQL 資料庫上執行下列指令碼，以在 pg_xlog 函式周圍建立包裝函式。

```
BEGIN;
CREATE SCHEMA IF NOT EXISTS fnRenames;
CREATE OR REPLACE FUNCTION fnRenames.pg_switch_xlog() RETURNS pg_lsn AS $$ 
   SELECT pg_switch_wal(); $$ LANGUAGE SQL;
CREATE OR REPLACE FUNCTION fnRenames.pg_xlog_replay_pause() RETURNS VOID AS $$ 
   SELECT pg_wal_replay_pause(); $$ LANGUAGE SQL;
CREATE OR REPLACE FUNCTION fnRenames.pg_xlog_replay_resume() RETURNS VOID AS $$ 
   SELECT pg_wal_replay_resume(); $$ LANGUAGE SQL;
CREATE OR REPLACE FUNCTION fnRenames.pg_current_xlog_location() RETURNS pg_lsn AS $$ 
   SELECT pg_current_wal_lsn(); $$ LANGUAGE SQL;
CREATE OR REPLACE FUNCTION fnRenames.pg_is_xlog_replay_paused() RETURNS boolean AS $$ 
   SELECT pg_is_wal_replay_paused(); $$ LANGUAGE SQL;
CREATE OR REPLACE FUNCTION fnRenames.pg_xlogfile_name(lsn pg_lsn) RETURNS TEXT AS $$ 
   SELECT pg_walfile_name(lsn); $$ LANGUAGE SQL;
CREATE OR REPLACE FUNCTION fnRenames.pg_last_xlog_replay_location() RETURNS pg_lsn AS $$ 
   SELECT pg_last_wal_replay_lsn(); $$ LANGUAGE SQL;
CREATE OR REPLACE FUNCTION fnRenames.pg_last_xlog_receive_location() RETURNS pg_lsn AS $$ 
   SELECT pg_last_wal_receive_lsn(); $$ LANGUAGE SQL;
CREATE OR REPLACE FUNCTION fnRenames.pg_current_xlog_flush_location() RETURNS pg_lsn AS $$ 
   SELECT pg_current_wal_flush_lsn(); $$ LANGUAGE SQL;
CREATE OR REPLACE FUNCTION fnRenames.pg_current_xlog_insert_location() RETURNS pg_lsn AS $$ 
   SELECT pg_current_wal_insert_lsn(); $$ LANGUAGE SQL;
CREATE OR REPLACE FUNCTION fnRenames.pg_xlog_location_diff(lsn1 pg_lsn, lsn2 pg_lsn) RETURNS NUMERIC AS $$ 
   SELECT pg_wal_lsn_diff(lsn1, lsn2); $$ LANGUAGE SQL;
CREATE OR REPLACE FUNCTION fnRenames.pg_xlogfile_name_offset(lsn pg_lsn, OUT TEXT, OUT INTEGER) AS $$ 
   SELECT pg_walfile_name_offset(lsn); $$ LANGUAGE SQL;
CREATE OR REPLACE FUNCTION fnRenames.pg_create_logical_replication_slot(slot_name name, plugin name, 
   temporary BOOLEAN DEFAULT FALSE, OUT slot_name name, OUT xlog_position pg_lsn) RETURNS RECORD AS $$ 
   SELECT slot_name::NAME, lsn::pg_lsn FROM pg_catalog.pg_create_logical_replication_slot(slot_name, plugin, 
   temporary); $$ LANGUAGE SQL;
ALTER USER PG_User SET search_path = fnRenames, pg_catalog, "$user", public;

-- DROP SCHEMA fnRenames CASCADE;
-- ALTER USER PG_User SET search_path TO DEFAULT;
COMMIT;
```

## <a name="other-limitations"></a>其他限制
- 資料庫名稱不能包含分號 (;)。
- 不支援有左大括號和右大括號 {  } 的密碼字串。 限制同時適用於連線到來源 PostgreSQL 及目標適用於 PostgreSQL 的 Azure 資料庫的情況。
- 擷取資料表必須有主索引鍵。 如果資料表沒有主索引鍵，則 DELETE 和 UPDATE 記錄作業將會產生無法預期的結果。
- 系統會忽略主索引鍵片段的更新。 在這類情況下，套用這類更新會使目標將更新識別為沒有更新任何資料列，並導致系統將記錄寫入至例外狀況資料表。
- 移轉多個具相同名稱但大小寫不同的資料表 (如 table1、TABLE1 和 Table1) 可能會造成無法預期的行為，因此沒有受到支援。
- 支援 [CREATE | ALTER | DROP] 資料表 DDL 的變更處理，除非它們位在內部函式/程序主體區塊或其他巢狀建構中。 例如，系統將不會擷取下列變更：

    ```
    CREATE OR REPLACE FUNCTION pg.create_distributors1() RETURNS void
    LANGUAGE plpgsql
    AS $$
    BEGIN
    create table pg.distributors1(did serial PRIMARY KEY,name varchar(40)
    NOT NULL);
    END;
    $$;
    ```

- 不支援 TRUNCATE 作業的變更處理 (持續同步)。 不支援移轉資料分割資料表。 偵測到資料分割資料表時，會發生下列情況：
    - 資料庫將會報告父和子資料表清單。
    - 該資料表會在目標上建立為一般資料表，且具有和所選資料表相同的屬性。
    - 如果來源資料庫中的父資料表具有和其子資料表相同的主索引鍵值，則會產生「索引鍵重複」錯誤。
- 在 DMS 中，於單一移轉活動中移轉的資料庫上限為四個。