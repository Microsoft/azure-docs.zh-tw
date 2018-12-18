---
title: 使用 .PHP 查詢 Azure SQL Database | Microsoft Docs
description: 本主題說明如何使用 PHP 來建立連線到 Azure SQL Database 的程式，並使用 Transact-SQL 陳述式查詢。
services: sql-database
author: CarlRabeler
manager: craigg
ms.service: sql-database
ms.custom: mvc,develop apps
ms.devlang: php
ms.topic: quickstart
ms.date: 04/01/2018
ms.author: carlrab
ms.openlocfilehash: 8fe343587336ff22f82ed0d1ef700fc56c86f577
ms.sourcegitcommit: 0a84b090d4c2fb57af3876c26a1f97aac12015c5
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/11/2018
ms.locfileid: "38561089"
---
# <a name="use-php-to-query-an-azure-sql-database"></a>使用 PHP 查詢 Azure SQL 資料庫

此快速入門示範如何使用 [PHP](http://php.net/manual/en/intro-whatis.php) 建立程式以連線至 Azure SQL 資料庫，並使用 Transact-SQL 陳述式來查詢資料。

## <a name="prerequisites"></a>先決條件

若要完成本快速入門，請確定您具備下列項目︰

[!INCLUDE [prerequisites-create-db](../../includes/sql-database-connect-query-prerequisites-create-db-includes.md)]

- 在此快速入門中，您所使用電腦的公用 IP 位址[伺服器層級防火牆規則](sql-database-get-started-portal.md#create-a-server-level-firewall-rule)。

- 您已安裝適用於您作業系統的 PHP 和相關軟體：

    - **MacOS**：安裝 Homebrew 和 PHP，安裝 ODBC 驅動程式和 SQLCMD，然後再安裝 PHP Driver for SQL Server。 請參閱[步驟 1.2、1.3 和 2.1](https://www.microsoft.com/sql-server/developer-get-started/php/mac/)。
    - **Ubuntu**：安裝 PHP 和其他必要的套件，然後安裝 PHP Driver for SQL Server。 請參閱[步驟 1.2 和 2.1](https://www.microsoft.com/sql-server/developer-get-started/php/ubuntu/)。
    - **Windows**：安裝 PHP for IIS Express 的最新版本、Microsoft Drivers for SQL Server in IIS Express 的最新版本、Chocolatey、ODBC 驅動程式，以及 SQLCMD。 請參閱[步驟 1.2 和 1.3](https://www.microsoft.com/sql-server/developer-get-started/php/windows/)。    

## <a name="sql-server-connection-information"></a>SQL Server 連線資訊

[!INCLUDE [prerequisites-server-connection-info](../../includes/sql-database-connect-query-prerequisites-server-connection-info-includes.md)]
    
## <a name="insert-code-to-query-sql-database"></a>插入程式碼以查詢 SQL 資料庫

1. 在您慣用的文字編輯器中，建立新的檔案 **sqltest.php**。  

2. 使用下列程式碼取代內容，並為您的伺服器、資料庫、使用者和密碼新增適當的值。

   ```PHP
   <?php
   $serverName = "your_server.database.windows.net";
   $connectionOptions = array(
       "Database" => "your_database",
       "Uid" => "your_username",
       "PWD" => "your_password"
   );
   //Establishes the connection
   $conn = sqlsrv_connect($serverName, $connectionOptions);
   $tsql= "SELECT TOP 20 pc.Name as CategoryName, p.name as ProductName
           FROM [SalesLT].[ProductCategory] pc
           JOIN [SalesLT].[Product] p
        ON pc.productcategoryid = p.productcategoryid";
   $getResults= sqlsrv_query($conn, $tsql);
   echo ("Reading data from table" . PHP_EOL);
   if ($getResults == FALSE)
       echo (sqlsrv_errors());
   while ($row = sqlsrv_fetch_array($getResults, SQLSRV_FETCH_ASSOC)) {
    echo ($row['CategoryName'] . " " . $row['ProductName'] . PHP_EOL);
   }
   sqlsrv_free_stmt($getResults);
   ?>
   ```

## <a name="run-the-code"></a>執行程式碼

1. 在命令提示字元中，執行下列命令：

   ```php
   php sqltest.php
   ```

2. 請確認前 20 個資料列已傳回，然後關閉應用程式視窗。

## <a name="next-steps"></a>後續步驟
- [設計您的第一個 Azure SQL Database](sql-database-design-first-database.md)
- [Microsoft PHP Drivers for SQL Server](https://github.com/Microsoft/msphpsql/)
- [回報問題或發問](https://github.com/Microsoft/msphpsql/issues)
- [重試邏輯範例：使用 PHP 彈性連接到 SQL][step-4-connect-resiliently-to-sql-with-php-p42h]


<!-- Link references. -->

[step-4-connect-resiliently-to-sql-with-php-p42h]: https://docs.microsoft.com/sql/connect/php/step-4-connect-resiliently-to-sql-with-php

