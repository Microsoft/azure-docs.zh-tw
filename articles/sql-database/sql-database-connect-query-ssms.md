---
title: 'SSMS: 在 Azure SQL Database 中連接和查詢資料 | Microsoft Docs'
description: 了解如何使用 SQL Server Management Studio (SSMS) 在 Azure 上連接到 SQL Database。 然後，執行 TRANSACT-SQL (T-SQL) 陳述式來查詢和編輯資料。
keywords: 連接到 sql database,sql server management studio
services: sql-database
ms.service: sql-database
ms.subservice: operations
ms.custom: ''
ms.devlang: ''
ms.topic: quickstart
author: CarlRabeler
ms.author: carlrab
ms.reviewer: ''
manager: craigg
ms.date: 08/01/2018
ms.openlocfilehash: 4c8b8a609fe5aa531570805d7a58141a6df84090
ms.sourcegitcommit: cc4fdd6f0f12b44c244abc7f6bc4b181a2d05302
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/25/2018
ms.locfileid: "47063212"
---
# <a name="azure-sql-database-use-sql-server-management-studio-to-connect-and-query-data"></a>Azure SQL Database：使用 SQL Server Management Studio 連接及查詢資料

[SQL Server Management Studio][ssms-install-latest-84g] (SSMS) 是整合式的環境，可用來管理任何 SQL 基礎結構，範圍從 Microsoft Windows 的 SQL Server 到 SQL Database。 此快速入門示範如何使用 SSMS 來連線至 Azure SQL Database，然後使用 Transact-SQL 陳述式來查詢、插入、更新和刪除資料庫中的資料。 

## <a name="prerequisites"></a>必要條件

本快速入門可作為在其中一個快速入門中所建立資源的起點︰

[!INCLUDE [prerequisites-create-db](../../includes/sql-database-connect-query-prerequisites-create-db-includes.md)]

本快速入門也需要您設定伺服器層級的防火牆規則。 如需示範如何執行這項操作的快速入門，請參閱[建立伺服器層級的防火牆規則](sql-database-get-started-portal-firewall.md)。

#### <a name="install-the-latest-ssms"></a>安裝最新的 SSMS

開始之前，確定您已安裝最新版的 [SSMS][ssms-install-latest-84g]。 

## <a name="sql-server-connection-information"></a>SQL Server 連線資訊

[!INCLUDE [prerequisites-server-connection-info](../../includes/sql-database-connect-query-prerequisites-server-connection-info-includes.md)]

## <a name="connect-to-your-database"></a>連接到您的資料庫

使用 SQL Server Management Studio (SSMS) 建立對 Azure SQL Database 伺服器的連線。 

> [!IMPORTANT]
> Azure SQL Database 邏輯伺服器會接聽連接埠 1433。 如果您嘗試從公司防火牆連線至 Azure SQL Database 邏輯伺服器，則必須在公司防火牆中開啟此連接埠，您才能成功連線。
>

1. 開啟 SQL Server Management Studio。

2. 在 [連接到伺服器] 對話方塊中，輸入下列資訊：

   | 設定      | 建議的值    | 說明 | 
   | ------------ | ------------------ | ----------- | 
   | **伺服器類型** | 資料庫引擎 | 需要此值。 |
   | **伺服器名稱** | 完整伺服器名稱 | 此名稱應該類似這樣︰**mynewserver20170313.database.windows.net**。 |
   | **驗證** | SQL Server 驗證 | 在本教學課程中，我們只設定了 SQL 驗證這個驗證類型。 |
   | **登入** | 伺服器管理帳戶 | 這是您在建立伺服器時所指定的帳戶。 |
   | **密碼** | 伺服器管理帳戶的密碼 | 這是您在建立伺服器時所指定的密碼。 |
   ||||

   ![連接到伺服器](./media/sql-database-connect-query-ssms/connect.png)  

3. 按一下 [連接到伺服器] 對話方塊中的 [選項]。 在 [連線到資料庫] 區段中，輸入 **mySampleDatabase** 以連線到這個資料庫。

   ![連線到伺服器上的 DB](./media/sql-database-connect-query-ssms/options-connect-to-db.png)  

4. 按一下 [ **連接**]。 [物件總管] 視窗隨即在 SSMS 中開啟。 

   ![已連接到伺服器](./media/sql-database-connect-query-ssms/connected.png)  

5. 在 [物件總管] 中，展開 [資料庫]，然後展開 [mySampleDatabase] 以檢視範例資料庫中的物件。

## <a name="query-data"></a>查詢資料

使用下列程式碼，可藉由使用 [SELECT](https://msdn.microsoft.com/library/ms189499.aspx) Transact-SQL 陳述式來依照類別查詢前 20 項產品。

1. 在 [物件總管] 中，於 **mySampleDatabase** 上按一下滑鼠右鍵，然後按一下 [新增查詢]。 隨即開啟已連線到您資料庫的空白查詢視窗。
2. 在查詢視窗中，輸入下列查詢︰

   ```sql
   SELECT pc.Name as CategoryName, p.name as ProductName
   FROM [SalesLT].[ProductCategory] pc
   JOIN [SalesLT].[Product] p
   ON pc.productcategoryid = p.productcategoryid;
   ```

3. 在工具列上，按一下 [執行] 來擷取 Product 和 ProductCategory 資料表中的資料。

    ![query](./media/sql-database-connect-query-ssms/query.png)

## <a name="insert-data"></a>插入資料

使用下列程式碼，藉由使用 [INSERT](https://msdn.microsoft.com/library/ms174335.aspx) Transact-SQL 陳述式將新產品插入 SalesLT.Product 資料表中。

1. 在查詢視窗中，以下列查詢取代先前的查詢︰

   ```sql
   INSERT INTO [SalesLT].[Product]
           ( [Name]
           , [ProductNumber]
           , [Color]
           , [ProductCategoryID]
           , [StandardCost]
           , [ListPrice]
           , [SellStartDate]
           )
     VALUES
           ('myNewProduct'
           ,123456789
           ,'NewColor'
           ,1
           ,100
           ,100
           ,GETDATE() );
   ```

2. 在工具列上，按一下 [執行] 以在Product 資料表中插入新資料列。

    <img src="./media/sql-database-connect-query-ssms/insert.png" alt="insert" style="width: 780px;" />

## <a name="update-data"></a>更新資料

使用下列程式碼，藉由使用 [UPDATE](https://msdn.microsoft.com/library/ms177523.aspx) Transact-SQL 陳述式更新您先前新增的產品。

1. 在查詢視窗中，以下列查詢取代先前的查詢︰

   ```sql
   UPDATE [SalesLT].[Product]
   SET [ListPrice] = 125
   WHERE Name = 'myNewProduct';
   ```

2. 在工具列上，按一下 [執行] 以在Product 資料表中更新指定的資料列。

    <img src="./media/sql-database-connect-query-ssms/update.png" alt="update" style="width: 780px;" />

## <a name="delete-data"></a>刪除資料

使用下列程式碼，藉由使用 [DELETE](https://msdn.microsoft.com/library/ms189835.aspx) Transact-SQL 陳述式刪除您先前新增的產品。

1. 在查詢視窗中，以下列查詢取代先前的查詢︰

   ```sql
   DELETE FROM [SalesLT].[Product]
   WHERE Name = 'myNewProduct';
   ```

2. 在工具列上，按一下 [執行] 以在Product 資料表中刪除指定的資料列。

    <img src="./media/sql-database-connect-query-ssms/delete.png" alt="delete" style="width: 780px;" />

## <a name="next-steps"></a>後續步驟

- 如需有關 SSMS 的資訊，請參閱 [使用 SQL Server Management Studio](https://msdn.microsoft.com/library/ms174173.aspx)。
- 若要使用 Azure 入口網站連線及查詢，請參閱[使用 Azure 入口網站 SQL 查詢編輯器連線和查詢](sql-database-connect-query-portal.md)。
- 若要使用 Visual Studio Code 進行連線和查詢，請參閱[使用 Visual Studio Code 進行連線和查詢](sql-database-connect-query-vscode.md)。
- 若要使用 .NET 進行連線和查詢，請參閱[使用 .NET 進行連線和查詢](sql-database-connect-query-dotnet.md)。
- 若要使用 PHP 進行連線和查詢，請參閱[使用 PHP 進行連線和查詢](sql-database-connect-query-php.md)。
- 若要使用 Node.js 進行連線和查詢，請參閱[使用 Node.js 進行連線和查詢](sql-database-connect-query-nodejs.md)。
- 若要使用 Java 進行連線和查詢，請參閱[使用 Java 進行連線和查詢](sql-database-connect-query-java.md)。
- 若要使用 Python 進行連線和查詢，請參閱[使用 Python 進行連線和查詢](sql-database-connect-query-python.md)。
- 若要使用 Ruby 進行連線和查詢，請參閱[使用 Ruby 進行連線和查詢](sql-database-connect-query-ruby.md)。


<!-- Article link references. -->

[ssms-install-latest-84g]: https://docs.microsoft.com/sql/ssms/sql-server-management-studio-ssms

