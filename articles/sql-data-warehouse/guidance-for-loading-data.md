---
title: 資料載入最佳做法 - Azure SQL 資料倉儲 | Microsoft Docs
description: 將資料載入 Azure SQL 資料倉儲的建議和效能最佳化。
services: sql-data-warehouse
author: ckarst
manager: craigg
ms.service: sql-data-warehouse
ms.topic: conceptual
ms.component: implement
ms.date: 04/17/2018
ms.author: cakarst
ms.reviewer: igorstan
ms.openlocfilehash: a2cc5b02744c04752ba11cbba14fe95c487d737c
ms.sourcegitcommit: 2b2129fa6413230cf35ac18ff386d40d1e8d0677
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/30/2018
ms.locfileid: "43248112"
---
# <a name="best-practices-for-loading-data-into-azure-sql-data-warehouse"></a>將資料載入 Azure SQL 資料倉儲的最佳做法
將資料載入 Azure SQL 資料倉儲的建議和效能最佳化。 

- 若要深入了解 PolyBase 及如何設計擷取、載入和轉換 (ELT) 程序，請參閱[設計 SQL 資料倉儲的 ELT](design-elt-data-loading.md)。
- 如需載入教學課程，請參閱[使用 PolyBase 將資料從 Azure Blob 儲存體載入 SQL 資料倉儲中](load-data-from-azure-blob-storage-using-polybase.md)。


## <a name="preparing-data-in-azure-storage"></a>在 Azure 儲存體中準備資料
若要將延遲降至最低，請共置您的儲存層與資料倉儲。

將資料匯出成 ORC 檔案格式時，如有大量文字資料行，則可能發生 Java 記憶體不足錯誤。 若要解決這項限制，只能匯出部分資料行。

PolyBase 無法載入具有超過 1 百萬個位元組之資料的資料列。 當您將資料放入 Azure Blob 儲存體或 Azure Data Lake Store 中的文字檔案時，這些檔案必須有少於 1 百萬個位元組的資料。 不論資料表結構描為何，此位元組限制皆成立。

每種檔案格式具有不同的效能特性。 若要最快載入，使用壓縮的分隔文字檔案。 UTF-8 和 UTF-16 效能之間的差異最小。

將大型的壓縮檔案分成較小的壓縮檔案。

## <a name="running-loads-with-enough-compute"></a>使用足夠的計算資源執行載入

如需最快的載入速度，一次只執行一項載入作業。 如果不可行，請同時執行數量最少的載入。 如果您預期會有大量載入作業，請考慮在載入前將資料倉儲相應增加。

若要以適當的計算資源執行載入，請建立為了執行載入而指定的載入使用者。 將每個載入使用者指派給特定的資源類別。 若要執行載入，請以其中一個載入使用者的身分登入，然後執行載入。 載入會利用使用者的資源類別來執行。  相較於嘗試變更使用者的資源類別，以符合目前的資源類別需求，這個方法比較簡單。

### <a name="example-of-creating-a-loading-user"></a>建立載入使用者的範例
此範例會為 staticrc20 資源類別建立載入使用者。 第一個步驟是**連線到 主要資料庫**並建立登入。

```sql
   -- Connect to master
   CREATE LOGIN LoaderRC20 WITH PASSWORD = 'a123STRONGpassword!';
```
連線到資料倉儲並建立使用者。 下列程式碼假設您已連線到名為 mySampleDataWarehouse 的資料庫。 它會顯示如何建立名為 LoaderRC20 的使用者，並為使用者提供資料庫的控制權限。 然後將使用者新增為 staticrc20 資料庫角色的成員。  

```sql
   -- Connect to the database
   CREATE USER LoaderRC20 FOR LOGIN LoaderRC20;
   GRANT CONTROL ON DATABASE::[mySampleDataWarehouse] to LoaderRC20;
   EXEC sp_addrolemember 'staticrc20', 'LoaderRC20';
```
若要使用 staticRC20 資源類別的資源執行載入，只需以 LoaderRC20 身分登入並執行載入。

在靜態資源類別，而不是動態資源類別之下執行載入。 不論您的[資料倉儲單位](what-is-a-data-warehouse-unit-dwu-cdwu.md)為何，使用靜態資源類別可保證相同的資源。 如果您使用動態資源類別，資源就會根據您的服務層級而有所不同。 對於動態類別，較低服務層級表示您可能需要對您的載入使用者使用較大的資源類別。

## <a name="allowing-multiple-users-to-load"></a>允許多個使用者載入

通常需要讓多個使用者將資料載入資料倉儲中。 使用 [CREATE TABLE AS SELECT (Transact-SQL)](/sql/t-sql/statements/create-table-as-select-azure-sql-data-warehouse) 載入所需資料庫的 CONTROL 權限。  CONTROL 權限可控制所有結構描述的存取。 您可能不希望所有的載入使用者都能控制所有結構描述的存取。 若要限制權限，請使用 DENY CONTROL 陳述式。

例如，請考慮將資料庫結構描述 schema_A 用於 dept A 以及 schema_B 用於 dept B，讓資料庫使用者 user_A 和 user_B 個別成為 dept A 及 B 中載入的 PolyBase 之使用者。 這兩個使用者皆已獲得 CONTROL 資料庫權限。 結構描述 A 和 B 的建立者現在是使用 DENY 鎖定其結構描述：

```sql
   DENY CONTROL ON SCHEMA :: schema_A TO user_B;
   DENY CONTROL ON SCHEMA :: schema_B TO user_A;
```

user_A 和 user_B 現在已從其他部門的結構描述加以鎖定。


## <a name="loading-to-a-staging-table"></a>載入至暫存表格

若要達到以最快載入速度將資料移入資料倉儲資料表中，請將資料載入暫存資料表中。  將暫存資料表定義為堆積，並使用循環配置資源作為散發選項。 

請考慮載入通常為兩個步驟的程序，您會先載入至暫存資料表，然後將資料插入生產資料倉儲資料表中。 如果生產資料表使用雜湊散發，您若定義採用雜湊散發的暫存資料表，則載入和插入的總時間可能比較快。 載入至暫存表格所需的時間比較長，但是將資料列插入生產資料表的第二個步驟不會導致資料四處移動。

## <a name="loading-to-a-columnstore-index"></a>載入至資料行存放區索引

資料行存放區索引需要大量的記憶體，才能將資料壓縮成高品質的資料列群組。 為了最佳的壓縮和索引效率，資料行存放區索引需要將多達 1,048,576 個資料列壓縮到每個資料列群組中。 當記憶體不足時，資料行存放區索引可能無法達到最大的壓縮率。 這反而會影響查詢效能。 如需深入探討，請參閱[資料行存放區記憶體最佳化](sql-data-warehouse-memory-optimizations-for-columnstore-compression.md)。

- 若要確保載入使用者有足夠的記憶體可達到最大的壓縮率，請使用是中型或大型資源類別成員的載入使用者。 
- 載入足夠的資料列，完全填滿新的資料列群組。 在大量載入期間，每 1,048,576 個資料列會以完整資料列群組形式直接壓縮到資料行存放區中。 若載入的資料列少於 102,400 個，則會將資料列傳送至差異存放區，其中的資料列會保存在 b 型樹狀結構索引中。 如果您載入太少資料列，這些資料列可能全都會移至差異存放區，並不會立即壓縮成資料行存放區格式。


## <a name="handling-loading-failures"></a>處理載入失敗

使用外部資料表的載入可能會失敗，並顯示「查詢已中止 -- 從外部來源讀取時已達最大拒絕閾值」錯誤訊息。 此訊息表示您的外部資料包含「錯誤」記錄。 如果資料類型和資料行數不符合外部資料表的資料行定義，或資料不符合指定的外部檔案格式，則會將資料記錄視為「錯誤」。 

若要修正「錯誤」記錄，請確定您的外部資料表及外部檔案格式定義皆正確，且這些定義與您的外部資料相符。 萬一外部資料記錄的子集有錯誤，您可以使用 CREATE EXTERNAL TABLE 中的拒絕選項，選擇拒絕這些查詢記錄。

## <a name="inserting-data-into-a-production-table"></a>將資料插入生產資料表中
使用 [INSERT 陳述式](/sql/t-sql/statements/insert-transact-sql)單次載入小型資料表，或甚至定期重新載入查閱，可能會與使用 `INSERT INTO MyLookup VALUES (1, 'Type 1')` 之類的陳述式有一樣好的效果。  不過，單一插入的效率不如執行大量載入。 

如果您整天有數千個或更多單一插入，請將插入分批，以便進行大量載入。  開發將單一插入附加至檔案的程序，然後建立另一個可定期載入檔案的程序。

## <a name="creating-statistics-after-the-load"></a>建立載入後的統計資料

為了改善查詢效能，在首次載入資料或資料發生重大變更之後，建立所有資料表的所有資料行統計資料非常重要。  如需統計資料的詳細說明，請參閱 [統計資料](sql-data-warehouse-tables-statistics.md)。 下列範例會在 Customer_Speed 資料表的五個資料行上建立統計資料。

```sql
create statistics [SensorKey] on [Customer_Speed] ([SensorKey]);
create statistics [CustomerKey] on [Customer_Speed] ([CustomerKey]);
create statistics [GeographyKey] on [Customer_Speed] ([GeographyKey]);
create statistics [Speed] on [Customer_Speed] ([Speed]);
create statistics [YearMeasured] on [Customer_Speed] ([YearMeasured]);
```

## <a name="rotate-storage-keys"></a>輪替儲存體金鑰
基於安全性考量，請定期變更您 blob 儲存體的存取金鑰。 您的 blob 儲存體帳戶有兩個儲存體金鑰，這可讓您轉換金鑰。

若要輪替 Azure 儲存體帳戶金鑰：

對於金鑰已變更的每個儲存體帳戶，發出 [ALTER DATABASE SCOPED CREDENTIAL](/sql/t-sql/statements/alter-database-scoped-credential-transact-sql)。

範例：

建立原始金鑰

```sql
CREATE DATABASE SCOPED CREDENTIAL my_credential WITH IDENTITY = 'my_identity', SECRET = 'key1'
``` 

將金鑰從 key 1 輪替為 key 2

```sql
ALTER DATABASE SCOPED CREDENTIAL my_credential WITH IDENTITY = 'my_identity', SECRET = 'key2' 
```

不需要對基礎外部資料來源進行其他變更。


## <a name="next-steps"></a>後續步驟
若要監視資料載入，請參閱[使用 DMV 監視工作負載](sql-data-warehouse-manage-monitor.md)。



