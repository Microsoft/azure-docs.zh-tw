---
title: 教學課程：Azure SQL 資料倉儲的彈性查詢 | Microsoft Docs
description: 本教學課程使用彈性查詢功能，從 Azure SQL Database 查詢 Azure SQL 資料倉儲。
services: sql-data-warehouse
author: hirokib
manager: craigg
ms.service: sql-data-warehouse
ms.topic: conceptual
ms.component: implement
ms.date: 04/14/2018
ms.author: elbutter
ms.reviewer: igorstan
ms.openlocfilehash: 355ae1c27d0af8f77c2c9bda61c3581562050fc4
ms.sourcegitcommit: 1fb353cfca800e741678b200f23af6f31bd03e87
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/30/2018
ms.locfileid: "43307087"
---
# <a name="tutorial-use-elastic-query-to-access-data-in-azure-sql-data-warehouse-from-azure-sql-database"></a>教學課程：使用彈性查詢，從 Azure SQL Database 存取 Azure SQL 資料倉儲中的資料

本教學課程使用彈性查詢功能，從 Azure SQL Database 查詢 Azure SQL 資料倉儲。 

## <a name="prerequisites-for-the-tutorial"></a>教學課程的必要條件

開始進行本教學課程之前，您必須具備下列必要條件：

1. 已安裝 SQL Server Management Studio (SSMS)。
2. 已建立具有資料庫與資料倉儲的 Azure SQL 伺服器。
3. 設定用以存取 Azure SQL 伺服器的防火牆規則。

## <a name="set-up-connection-between-sql-data-warehouse-and-sql-database-instances"></a>設定 SQL 資料倉儲和 SQL Database 執行個體之間的連線 

1. 使用 SSMS 或其他查詢用戶端，開啟對邏輯伺服器上 **master** 資料庫的新查詢。

2. 建立代表 SQL 資料庫對資料倉儲連線的登入和使用者。

   ```sql
   CREATE LOGIN SalesDBLogin WITH PASSWORD = 'aReallyStrongPassword!@#';
   ```

3. 使用 SSMS 或其他查詢用戶端，開啟對邏輯伺服器上 **SQL 資料倉儲執行個體**的新查詢。

4. 利用步驟 2 中所建立的登入，在資料倉儲執行個體上建立使用者

   ```sql
   CREATE USER SalesDBUser FOR LOGIN SalesDBLogin;
   ```

5. 針對步驟 4 的使用者，授與 SQL Database 的執行權限。 在此範例中，只會對特定結構描述上的 SELECT 授與權限，藉此說明如何將 SQL 資料庫查詢限制在特定網域。 

   ```sql
   GRANT SELECT ON SCHEMA :: [dbo] TO SalesDBUser;
   ```

6. 使用 SSMS 或其他查詢用戶端，開啟對邏輯伺服器上 **SQL 資料庫執行個體**的新查詢。

7. 如果您還沒有主要金鑰，請建立主要金鑰。 

   ```sql
   CREATE MASTER KEY; 
   ```

8. 使用步驟 2 建立的認證來建立資料庫範圍認證。

   ```sql
   CREATE DATABASE SCOPED CREDENTIAL SalesDBElasticCredential
   WITH IDENTITY = 'SalesDBLogin',
   SECRET = 'aReallyStrongPassword@#!';
   ```

9. 建立指向資料倉儲執行個體的外部資料來源。

   ```sql
   CREATE EXTERNAL DATA SOURCE EnterpriseDwSrc WITH 
       (TYPE = RDBMS, 
       LOCATION = '<SERVER NAME>.database.windows.net', 
       DATABASE_NAME = '<SQL DATA WAREHOUSE NAME>', 
       CREDENTIAL = SalesDBElasticCredential, 
   ) ;
   ```

10. 現在您可以建立參考這個外部資料來源的外部資料表。 使用這些資料表的查詢會傳送到資料倉儲執行個體，處理之後再傳送回資料庫執行個體。


## <a name="elastic-query-from-sql-database-to-sql-data-warehouse"></a>從 SQL 資料庫對 SQL 資料倉儲發出彈性查詢

在接下來的步驟中，我們將會以數個值在資料倉儲執行個體中建立資料表， 然後示範如何設定外部資料表，以便從資料庫執行個體來查詢資料倉儲執行個體。

1. 使用 SSMS 或其他查詢用戶端，開啟對邏輯伺服器上 **SQL 資料倉儲執行個體**的新查詢。

2. 提交下列查詢，以便在資料倉儲執行個體中建立 **OrdersInformation** 資料表。

   ```sql
   CREATE TABLE [dbo].[OrderInformation]
   ( 
       [OrderID] [int] NOT NULL 
   ,   [CustomerID] [int] NOT NULL 
   ) 
   INSERT INTO [dbo].[OrderInformation] ([OrderID], [CustomerID]) VALUES (123, 1) 
   INSERT INTO [dbo].[OrderInformation] ([OrderID], [CustomerID]) VALUES (149, 2) 
   INSERT INTO [dbo].[OrderInformation] ([OrderID], [CustomerID]) VALUES (857, 2) 
   INSERT INTO [dbo].[OrderInformation] ([OrderID], [CustomerID]) VALUES (321, 1) 
   INSERT INTO [dbo].[OrderInformation] ([OrderID], [CustomerID]) VALUES (564, 8)
   ```

3. 使用 SSMS 或其他查詢用戶端，開啟對邏輯伺服器上 **SQL 資料庫**的新查詢。

4. 提交下列查詢，以建立指向資料倉儲執行個體中 **OrdersInformation** 資料表的外部資料表定義。

   ```sql
   CREATE EXTERNAL TABLE [dbo].[OrderInformation]
   ( 
       [OrderID] [int] NOT NULL
   ,   [CustomerID] [int] NOT NULL 
   ) 
   WITH 
   (
        DATA_SOURCE = EnterpriseDwSrc
   ,    SCHEMA_NAME = N'dbo'
   ,    OBJECT_NAME = N'OrderInformation'
   )
   ```

5. 請注意 **SQL 資料庫執行個體**中現在有外部資料表定義。

   ![彈性查詢外部資料表定義](media/sql-data-warehouse-elastic-query-with-sql-database/elastic-query-external-table.png)


6. 提交下列會查詢資料倉儲執行個體的查詢。 您應該會收到在步驟 2 中插入的五個值。 

```sql
SELECT * FROM [dbo].[OrderInformation];
```

> [!NOTE]
>
> 請注意，儘管只有幾個值，這個查詢需要耗費相當長的時間才會傳回。 使用彈性查詢於資料倉儲時，應考慮查詢處理和網路傳輸所產生的額外負荷成本。 只有當首要考量為運算能力而非延遲時，才應使用彈性查詢遠端執行。

恭喜，您已設定最基本的彈性查詢。 

## <a name="next-steps"></a>後續步驟
如需建議，請參閱[於 Azure SQL 資料倉儲使用彈性查詢的最佳做法](how-to-use-elastic-query-with-sql-data-warehouse.md)。