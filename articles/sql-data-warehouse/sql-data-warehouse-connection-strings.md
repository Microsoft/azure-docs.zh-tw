---
title: 適用於 Azure SQL 資料倉儲的連接字串 | Microsoft Docs
description: 適用於 SQL 資料倉儲的連接字串
services: sql-data-warehouse
author: kavithaj
manager: craigg
ms.service: sql-data-warehouse
ms.topic: conceptual
ms.component: consume
ms.date: 04/17/2018
ms.author: kavithaj
ms.reviewer: igorstan
ms.openlocfilehash: de875731ffd2170f70d8034f0178d61351d86952
ms.sourcegitcommit: 2b2129fa6413230cf35ac18ff386d40d1e8d0677
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/30/2018
ms.locfileid: "43248016"
---
# <a name="connection-strings-for-azure-sql-data-warehouse"></a>適用於 Azure SQL 資料倉儲的連接字串
您可以使用數個不同的應用程式通訊協定 (例如 [ADO.NET][ADO.NET]、[ODBC][ODBC]、[PHP][PHP] 及 [JDBC][JDBC]) 連線到 SQL 資料倉儲。 以下是每個通訊協定連接字串的一些範例。  您也可以使用 Azure 入口網站來建立連接字串。  若要使用 Azure 入口網站建置連接字串，請瀏覽至您資料庫的刀鋒視窗，在 [程式集] 底下，按一下 [顯示資料庫連接字串]。

## <a name="sample-adonet-connection-string"></a>範例 ADO.NET 連接字串
```csharp
Server=tcp:{your_server}.database.windows.net,1433;Database={your_database};User ID={your_user_name};Password={your_password_here};Encrypt=True;TrustServerCertificate=False;Connection Timeout=30;
```

## <a name="sample-odbc-connection-string"></a>範例 ODBC 連接字串
```csharp
Driver={SQL Server Native Client 11.0};Server=tcp:{your_server}.database.windows.net,1433;Database={your_database};Uid={your_user_name};Pwd={your_password_here};Encrypt=yes;TrustServerCertificate=no;Connection Timeout=30;
```

## <a name="sample-php-connection-string"></a>範例 PHP 連接字串
```PHP
Server: {your_server}.database.windows.net,1433 \r\nSQL Database: {your_database}\r\nUser Name: {your_user_name}\r\n\r\nPHP Data Objects(PDO) Sample Code:\r\n\r\ntry {\r\n   $conn = new PDO ( \"sqlsrv:server = tcp:{your_server}.database.windows.net,1433; Database = {your_database}\", \"{your_user_name}\", \"{your_password_here}\");\r\n    $conn->setAttribute( PDO::ATTR_ERRMODE, PDO::ERRMODE_EXCEPTION );\r\n}\r\ncatch ( PDOException $e ) {\r\n   print( \"Error connecting to SQL Server.\" );\r\n   die(print_r($e));\r\n}\r\n\rSQL Server Extension Sample Code:\r\n\r\n$connectionInfo = array(\"UID\" => \"{your_user_name}\", \"pwd\" => \"{your_password_here}\", \"Database\" => \"{your_database}\", \"LoginTimeout\" => 30, \"Encrypt\" => 1, \"TrustServerCertificate\" => 0);\r\n$serverName = \"tcp:{your_server}.database.windows.net,1433\";\r\n$conn = sqlsrv_connect($serverName, $connectionInfo);
```

## <a name="sample-jdbc-connection-string"></a>範例 JDBC 連接字串
```Java
jdbc:sqlserver://yourserver.database.windows.net:1433;database=yourdatabase;user={your_user_name};password={your_password_here};encrypt=true;trustServerCertificate=false;hostNameInCertificate=*.database.windows.net;loginTimeout=30;
```

> [!NOTE]
> 請考慮將連線逾時設定為 300 秒，以便在短時間無法使用時能夠維持連線。
> 
> 

## <a name="next-steps"></a>後續步驟
若要透過 Visual Studio 和其他應用程式開始查詢您的資料倉儲，請參閱[使用 Visual Studio 查詢][Query with Visual Studio]。

<!--Image references-->

<!--Azure.com references-->
[Query with Visual Studio]: ./sql-data-warehouse-query-visual-studio.md

<!--MSDN references-->
[ADO.NET]: https://msdn.microsoft.com/library/e80y5yhx(v=vs.110).aspx
[ODBC]: https://msdn.microsoft.com/library/jj730314.aspx
[PHP]: https://msdn.microsoft.com/library/cc296172.aspx?f=255&MSPPError=-2147217396
[JDBC]: https://msdn.microsoft.com/library/mt484311(v=sql.110).aspx

<!--Other references-->
