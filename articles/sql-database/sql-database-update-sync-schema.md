---
title: 在 Azure SQL 資料同步中自動執行結構描述變更複寫 | Microsoft Docs
description: 了解如何在「Azure SQL 資料同步」中自動執行結構描述變更複寫。
services: sql-database
ms.service: sql-database
ms.subservice: data-movement
ms.custom: data sync
ms.devlang: ''
ms.topic: conceptual
author: allenwux
ms.author: xiwu
ms.reviewer: douglasl
manager: craigg
ms.date: 09/20/2018
ms.openlocfilehash: 3137b86dd186e628508111a932140fd9c1f59b5d
ms.sourcegitcommit: 51a1476c85ca518a6d8b4cc35aed7a76b33e130f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/25/2018
ms.locfileid: "47161418"
---
# <a name="automate-the-replication-of-schema-changes-in-azure-sql-data-sync"></a>在 Azure SQL 資料同步中自動執行結構描述變更複寫

「SQL 資料同步」可讓使用者在 Azure SQL Database 與內部部署 SQL Server 之間進行單向或雙向資料同步。 「SQL 資料同步」目前的其中一個限制是不支援複寫結構描述變更。 每次您變更資料表結構描述時，都必須手動在所有端點 (包括中樞及所有成員) 套用變更，然後更新同步結構描述。

本文介紹可自動將結構描述變更複寫至所有「SQL 資料同步」端點的解決方案。
1. 此解決方案使用 DDL 觸發程序來追蹤結構描述變更。
1. 此觸發程序會在追蹤資料表中插入結構描述變更命令。
1. 此追蹤資料表會透過「資料同步」服務同步至所有端點。
1. 插入後的 DML 觸發程序會用來將結構描述變更套用至其他端點。

本文使用 ALTER TABLE 作為結構描述變更的範例，但此解決方案也適用於其他類型的結構描述變更。

> [!IMPORTANT]
> 建議您先仔細閱讀本文，特別是有關[疑難排解](#troubleshoot)和[其他考量](#other)的小節，再開始在您的同步環境中實作自動化結構描述變更複寫。 此外，也建議您閱讀[使用 SQL 資料同步跨多個雲端和內部部署資料庫同步資料](sql-database-sync-data.md)。有些資料庫作業可能導致本文所述的解決方案無法運作。 您可能需要具備額外的 SQL Server 和 Transact-SQL 網域知識，才能針對這些問題進行疑難排解。

![自動執行結構描述變更複寫](media/sql-database-update-sync-schema/automate-schema-changes.png)

## <a name="set-up-automated-schema-change-replication"></a>設定自動化結構描述變更複寫

### <a name="create-a-table-to-track-schema-changes"></a>建立資料表來追蹤結構描述變更

建立資料表來追蹤同步群組中所有資料庫的結構描述變更：

```sql
CREATE TABLE SchemaChanges (
ID bigint IDENTITY(1,1) PRIMARY KEY,
SqlStmt nvarchar(max),
[Description] nvarchar(max)
)
```

此資料表包含身分識別資料行來追蹤結構描述變更順序。 您可以視需要新增更多欄位來記錄更多資訊。

### <a name="create-a-table-to-track-the-history-of-schema-changes"></a>建立資料表來追蹤結構描述變更歷程記錄

在所有端點上，建立資料表來追蹤最近所套用結構描述變更命令的識別碼。

```sql
CREATE TABLE SchemaChangeHistory (
LastAppliedId bigint PRIMARY KEY
)
GO

INSERT INTO SchemaChangeHistory VALUES (0)
```

### <a name="create-an-alter-table-ddl-trigger-in-the-database-where-schema-changes-are-made"></a>在進行結構描述變更的資料庫中建立 ALTER TABLE DDL 觸發程序

建立 ALTER TABLE 作業的 DDL 觸發程序。 您只需在進行結構描述變更的資料庫中建立此觸發程序。 為了避免衝突，請只允許在同步群組中的一個資料庫進行結構描述變更。

```sql
CREATE TRIGGER AlterTableDDLTrigger
ON DATABASE
FOR ALTER_TABLE
AS

-- You can add your own logic to filter ALTER TABLE commands instead of replicating all of them.

IF NOT (EVENTDATA().value('(/EVENT_INSTANCE/SchemaName)[1]', 'nvarchar(512)') like 'DataSync')

INSERT INTO SchemaChanges (SqlStmt, Description)
    VALUES (EVENTDATA().value('(/EVENT_INSTANCE/TSQLCommand/CommandText)[1]', 'nvarchar(max)'), 'From DDL trigger')
```

觸發程序會針對每個 ALTER TABLE 命令，在結構描述變更追蹤資料表中插入一個記錄。 此範例會新增篩選來避免複寫在 **DataSync** 結構描述底下進行的結構描述變更，因為這些最有可能是「資料同步」所進行的變更。 如果您只想要複寫特定類型的結構描述變更，請新增其他篩選。

您也可以新增更多觸發程序，來複寫其他類型的結構描述變更。 例如，建立 CREATE_PROCEDURE、ALTER_PROCEDURE 及 DROP_PROCEDURE 觸發程序來複寫對預存程序所做的變更。

### <a name="create-a-trigger-on-other-endpoints-to-apply-schema-changes-during-insertion"></a>在其他端點建立觸發程序以在插入時套用結構描述變更

此觸發程序會在同步至其他端點時，執行結構描述變更命令。 您必須在所有端點上建立此觸發程序，但進行結構描述變更的端點除外 (亦即上一個步驟中 DDL 觸發程序 `AlterTableDDLTrigger` 建立所在資料庫中的端點)。

```sql
CREATE TRIGGER SchemaChangesTrigger
ON SchemaChanges
AFTER INSERT
AS
DECLARE \@lastAppliedId bigint
DECLARE \@id bigint
DECLARE \@sqlStmt nvarchar(max)
SELECT TOP 1 \@lastAppliedId=LastAppliedId FROM SchemaChangeHistory
SELECT TOP 1 \@id = id, \@SqlStmt = SqlStmt FROM SchemaChanges WHERE id \> \@lastAppliedId ORDER BY id
IF (\@id = \@lastAppliedId + 1)
BEGIN
    EXEC sp_executesql \@SqlStmt
        UPDATE SchemaChangeHistory SET LastAppliedId = \@id
    WHILE (1 = 1)
    BEGIN
        SET \@id = \@id + 1
        IF exists (SELECT id FROM SchemaChanges WHERE ID = \@id)
            BEGIN
                SELECT \@sqlStmt = SqlStmt FROM SchemaChanges WHERE ID = \@id
                EXEC sp_executesql \@SqlStmt
                UPDATE SchemaChangeHistory SET LastAppliedId = \@id
            END
        ELSE
            BREAK;
    END
END
```

此觸發程序會在插入後執行，並且會檢查目前的命令是否應該是下一個執行的命令。 此程式碼邏輯可確保不會略過任何結構描述變更陳述式，且即使插入順序錯亂，仍然會套用所有變更。

### <a name="sync-the-schema-change-tracking-table-to-all-endpoints"></a>將結構描述變更追蹤資料表同步至所有端點

您可以使用現有的同步群組或新的同步群組，將結構描述變更追蹤資料表同步至所有端點。 請確定追蹤資料表中的變更可以同步至所有端點，特別是當您使用單向同步時。

請勿同步結構描述變更歷程記錄資料表，因為該資料表在不同的端點上會保有不同的狀態。

### <a name="apply-the-schema-changes-in-a-sync-group"></a>套用同步群組中的結構描述變更

系統只會複寫 DDL 觸發程序建立所在資料庫中的結構描述變更。 不會複寫在其他資料庫中進行的結構描述變更。

將結構描述變更複寫至所有端點之後，您還必須進行額外步驟來更新同步結構描述，才能開始或停止同步新的資料行。

#### <a name="add-new-columns"></a>新增資料行

1.  進行結構描述變更。

1.  避免進行任何涉及新資料行的資料變更，直到您已完成建立觸發程序的步驟為止。

1.  等候，直到將結構描述變更套用至所有端點為止。

1.  重新整理資料庫結構描述，然後將新資料行新增至同步結構描述。

1.  新資料行中的資料會在下一次同步作業期間進行同步。

#### <a name="remove-columns"></a>移除資料行

1.  從同步結構描述中移除資料行。 「資料同步」會停止同步這些資料行中的資料。

1.  進行結構描述變更。

1.  重新整理資料庫結構描述。

#### <a name="update-data-types"></a>更新資料類型

1.  進行結構描述變更。

1.  等候，直到將結構描述變更套用至所有端點為止。

1.  重新整理資料庫結構描述。

1.  如果新舊資料類型不完全相容 - 例如，從 `int` 變更為 `bigint` - 則同步可能會在建立觸發程序的步驟完成之前失敗。 在重試之後，同步即可成功。

#### <a name="rename-columns-or-tables"></a>重新命名資料行或資料表

重新命名資料行或資料表會讓「資料同步」停止運作。 請建立新的資料表或資料行、回填資料，然後刪除舊資料表或資料行，而不是重新命名。

#### <a name="other-types-of-schema-changes"></a>其他類型的結構描述變更

針對其他類型的結構描述變更 - 例如，建立預存程序或卸除索引- 並不需要更新同步結構描述。

## <a name="troubleshoot"></a> 針對自動化結構描述變更複寫進行疑難排解

本文所述的複寫邏輯在某些情況下會停止運作 - 例如，如果您在內部部署資料庫中進行 Azure SQL Database 所不支援的結構描述變更。 在該情況下，同步結構描述變更追蹤資料表會失敗。 您必須手動修正此問題：

1.  停用 DDL 觸發程序並避免任何進一步的結構描述變更，直到已修正此問題為止。

1.  在發生問題的端點資料庫中，於無法進行結構描述變更的端點上停用 AFTER INSERT 觸發程序。 此動作可允許同步結構描述變更命令。

1.  觸發同步來同步結構描述變更追蹤資料表。

1.  在發生問題的端點資料庫中，查詢結構描述變更歷程記錄資料表，以取得上次套用結構描述變更命令的識別碼。

1.  查詢結構描述變更追蹤資料表，以列出識別碼大於您在上一個步驟中所擷取識別碼值的所有命令。

    a.  忽略無法在端點資料庫中執行的命令。 您必須處理結構描述不一致的情況。 如果不一致情況會影響您的應用程式，請還原原始結構描述變更。

    b.  手動套用這些應該套用的命令。

1.  更新結構描述變更歷程記錄資料表，然後將上次套用識別碼設定為正確的值。

1.  請仔細檢查結構描述是否為最新。

1.  重新啟用在第二個步驟中停用的 AFTER INSERT 觸發程序。

1.  重新啟用在第一個步驟中停用的 DDL 觸發程序。

如果您想要清除結構描述變更追蹤資料表中的記錄，請使用 DELETE，而不是 TRUNCATE。 請一律不要使用 DBCC CHECKIDENT 在結構描述變更追蹤資料表中重新植入身分識別資料行。 如果需要重新植入，您可以建立新的結構描述變更追蹤資料表，然後在 DDL 觸發程序中更新資料表名稱。

## <a name="other"></a> 其他考量

-   設定中樞和成員資料庫的資料庫使用者必須有足夠的權限，才能執行結構描述變更命令。

-   您可以在 DDL 觸發程序中新增更多篩選，以只複寫所選資料表或作業中的結構描述變更。

-   您只能在建立 DDL 觸發程序的資料庫中進行結構描述變更。

-   如果您要在內部部署 SQL Server 資料庫中進行變更，請確定 Azure SQL Database 中支援該結構描述變更。

-   如果是在 DDL 觸發程序建立所在資料庫以外的資料庫進行結構描述變更，系統將不會複寫這類變更。 為了避免發生此問題，您可以建立 DDL 觸發程序來封鎖在其他端點上進行變更。

-   如果您需要變更結構描述變更追蹤資料表的結構描述，請先停用 DDL 觸發程序再進行變更，然後手動將變更套用至所有端點。 在相同資料表上的 AFTER INSERT 觸發程序中更新結構描述並不可行。

-   請勿使用 DBCC CHECKIDENT 來重新植入身分識別資料行。

-   請勿使用 TRUNCATE 來清除結構描述變更追蹤資料表中的資料。
