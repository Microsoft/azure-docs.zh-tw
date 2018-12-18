---
title: 使用 Node.js 查詢 Azure SQL Database | Microsoft Docs
description: 本主題說明如何使用 Node.js 來建立連線到 Azure SQL Database 的程式，並使用 Transact-SQL 陳述式查詢。
services: sql-database
ms.service: sql-database
ms.subservice: development
ms.custom: ''
ms.devlang: nodejs
ms.topic: quickstart
author: CarlRabeler
ms.author: carlrab
ms.reviewer: ''
manager: craigg
ms.date: 04/01/2018
ms.openlocfilehash: c6cfbd330f77c5a2f448093e58684032a84a41d9
ms.sourcegitcommit: cc4fdd6f0f12b44c244abc7f6bc4b181a2d05302
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/25/2018
ms.locfileid: "47063467"
---
# <a name="use-nodejs-to-query-an-azure-sql-database"></a>使用 Node.js 查詢 Azure SQL 資料庫

此快速入門示範如何使用 [Node.js](https://nodejs.org/en/) 建立程式以連線至 Azure SQL 資料庫，並使用 Transact-SQL 陳述式來查詢資料。

## <a name="prerequisites"></a>必要條件

若要完成本快速入門，請確定您具備下列項目︰

[!INCLUDE [prerequisites-create-db](../../includes/sql-database-connect-query-prerequisites-create-db-includes.md)]

- 在此快速入門中，您所使用電腦的公用 IP 位址[伺服器層級防火牆規則](sql-database-get-started-portal-firewall.md)。

- 您已安裝適用於您作業系統的 Node.js 和相關軟體：
    - **MacOS**：安裝 Homebrew 和 Node.js，然後安裝 ODBC 驅動程式和 SQLCMD。 請參閱[步驟 1.2 和 1.3](https://www.microsoft.com/sql-server/developer-get-started/node/mac/)。
    - **Ubuntu**：安裝 Node.js，然後安裝 ODBC 驅動程式和 SQLCMD。 請參閱[步驟 1.2 和 1.3](https://www.microsoft.com/sql-server/developer-get-started/node/ubuntu/)。
    - **Windows**：安裝 Chocolatey 和 Node.js，然後安裝 ODBC 驅動程式和 SQLCMD。 請參閱[步驟 1.2 和 1.3](https://www.microsoft.com/sql-server/developer-get-started/node/windows/)。

## <a name="sql-server-connection-information"></a>SQL Server 連線資訊

[!INCLUDE [prerequisites-server-connection-info](../../includes/sql-database-connect-query-prerequisites-server-connection-info-includes.md)]

> [!IMPORTANT]
> 在您執行本教學課程的電腦上，公用 IP 位址必須有防火牆規則。 如果您在不同電腦上或有不同的公用 IP 位址，請建立[使用 Azure 入口網站的伺服器層級防火牆規則](sql-database-get-started-portal-firewall.md)。 

## <a name="create-a-nodejs-project"></a>建立 Node.js 專案

開啟命令提示字元，並建立名為 sqltest 的資料夾。 瀏覽至您建立的資料夾中，然後執行下列命令：

    
    npm init -y
    npm install tedious
    npm install async
    

## <a name="insert-code-to-query-sql-database"></a>插入程式碼以查詢 SQL 資料庫

1. 在您的開發環境或慣用的文字編輯器中，建立新的檔案 **sqltest.js**。

2. 使用下列程式碼取代內容，並為您的伺服器、資料庫、使用者和密碼新增適當的值。

   ```js
   var Connection = require('tedious').Connection;
   var Request = require('tedious').Request;

   // Create connection to database
   var config = 
      {
        userName: 'someuser', // update me
        password: 'somepassword', // update me
        server: 'edmacasqlserver.database.windows.net', // update me
        options: 
           {
              database: 'somedb' //update me
              , encrypt: true
           }
      }
   var connection = new Connection(config);

   // Attempt to connect and execute queries if connection goes through
   connection.on('connect', function(err) 
      {
        if (err) 
          {
             console.log(err)
          }
       else
          {
              queryDatabase()
          }
      }
    );

   function queryDatabase()
      { console.log('Reading rows from the Table...');

          // Read all rows from table
        request = new Request(
             "SELECT TOP 20 pc.Name as CategoryName, p.name as ProductName FROM [SalesLT].[ProductCategory] pc JOIN [SalesLT].[Product] p ON pc.productcategoryid = p.productcategoryid",
                function(err, rowCount, rows) 
                   {
                       console.log(rowCount + ' row(s) returned');
                       process.exit();
                   }
               );
    
        request.on('row', function(columns) {
           columns.forEach(function(column) {
               console.log("%s\t%s", column.metadata.colName, column.value);
            });
                });
        connection.execSql(request);
      }
```

## <a name="run-the-code"></a>執行程式碼

1. 在命令提示字元中，執行下列命令：

   ```js
   node sqltest.js
   ```

2. 請確認前 20 個資料列已傳回，然後關閉應用程式視窗。

## <a name="next-steps"></a>後續步驟

- 了解 [Microsoft Node.js Driver for SQL Server](https://docs.microsoft.com/sql/connect/node-js/node-js-driver-for-sql-server/)
- 了解如何在 Windows/Linux/macOS 中[使用 .NET Core 來連線及查詢 Azure SQL 資料庫](sql-database-connect-query-dotnet-core.md)。  
- 了解 [使用命令列以開始使用在 Windows/Linux/macOS 上的 .NET Core](/dotnet/core/tutorials/using-with-xplat-cli)。
- 深入了解如何[使用 SSMS 設計您的第一個 Azure SQL 資料庫](sql-database-design-first-database.md)或[使用 .NET 設計您的第一個 Azure SQL 資料庫](sql-database-design-first-database-csharp.md)。
- 了解如何[使用 SSMS 連線及查詢](sql-database-connect-query-ssms.md)
- 了解如何[使用 Visual Studio Code 連線及查詢](sql-database-connect-query-vscode.md)。

