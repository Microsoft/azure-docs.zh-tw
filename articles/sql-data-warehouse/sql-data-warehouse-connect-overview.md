---
title: 連線到 Azure SQL 資料倉儲 | Microsoft Docs
description: 連線到 Azure SQL 資料倉儲。
services: sql-data-warehouse
author: kavithaj
manager: craigg
ms.service: sql-data-warehouse
ms.topic: conceptual
ms.component: consume
ms.date: 04/17/2018
ms.author: kavithaj
ms.reviewer: igorstan
ms.openlocfilehash: 0b2d8cec03c54ebd5bd780a2524da61d718a9673
ms.sourcegitcommit: 2b2129fa6413230cf35ac18ff386d40d1e8d0677
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/30/2018
ms.locfileid: "43247853"
---
# <a name="connect-to-azure-sql-data-warehouse"></a>連接到 Azure SQL 資料倉儲
連線到 Azure SQL 資料倉儲。

## <a name="find-your-server-name"></a>尋找您的伺服器名稱
下列範例中的伺服器名稱是 samplesvr.database.windows.net。 若要尋找完整的伺服器名稱：

1. 移至 [Azure 入口網站][Azure portal]。
2. 按一下 [SQL 資料倉儲]。
3. 按一下您想連線的資料倉儲。
4. 找出完整的伺服器名稱。
   
    ![完整伺服器名稱][1]

## <a name="supported-drivers-and-connection-strings"></a>支援的驅動程式和連接字串
Azure SQL 資料倉儲支援 [ADO.NET][ADO.NET]、[ODBC][ODBC]、[PHP][PHP] 和 [JDBC][JDBC]。 若要尋找最新版本和文件，請按一下前述的其中一個驅動程式。 若要從 Azure 入口網站自動為您使用的驅動程式產生連接字串，按一下前述範例中的 [顯示資料庫連接字串]。 下列一些範例顯示每個驅動程式的連接字串。

> [!NOTE]
> 請考慮將連線逾時設定為 300 秒，以便在短時間無法使用時能夠維持連線。
> 
> 

### <a name="adonet-connection-string-example"></a>ADO.NET 連接字串範例
```csharp
Server=tcp:{your_server}.database.windows.net,1433;Database={your_database};User ID={your_user_name};Password={your_password_here};Encrypt=True;TrustServerCertificate=False;Connection Timeout=30;
```

### <a name="odbc-connection-string-example"></a>ODBC 連接字串範例
```csharp
Driver={SQL Server Native Client 11.0};Server=tcp:{your_server}.database.windows.net,1433;Database={your_database};Uid={your_user_name};Pwd={your_password_here};Encrypt=yes;TrustServerCertificate=no;Connection Timeout=30;
```

### <a name="php-connection-string-example"></a>PHP 連接字串範例
```PHP
Server: {your_server}.database.windows.net,1433 \r\nSQL Database: {your_database}\r\nUser Name: {your_user_name}\r\n\r\nPHP Data Objects(PDO) Sample Code:\r\n\r\ntry {\r\n   $conn = new PDO ( \"sqlsrv:server = tcp:{your_server}.database.windows.net,1433; Database = {your_database}\", \"{your_user_name}\", \"{your_password_here}\");\r\n    $conn->setAttribute( PDO::ATTR_ERRMODE, PDO::ERRMODE_EXCEPTION );\r\n}\r\ncatch ( PDOException $e ) {\r\n   print( \"Error connecting to SQL Server.\" );\r\n   die(print_r($e));\r\n}\r\n\rSQL Server Extension Sample Code:\r\n\r\n$connectionInfo = array(\"UID\" => \"{your_user_name}\", \"pwd\" => \"{your_password_here}\", \"Database\" => \"{your_database}\", \"LoginTimeout\" => 30, \"Encrypt\" => 1, \"TrustServerCertificate\" => 0);\r\n$serverName = \"tcp:{your_server}.database.windows.net,1433\";\r\n$conn = sqlsrv_connect($serverName, $connectionInfo);
```

### <a name="jdbc-connection-string-example"></a>JDBC 連接字串範例
```Java
jdbc:sqlserver://yourserver.database.windows.net:1433;database=yourdatabase;user={your_user_name};password={your_password_here};encrypt=true;trustServerCertificate=false;hostNameInCertificate=*.database.windows.net;loginTimeout=30;
```

## <a name="connection-settings"></a>連線設定
SQL 資料倉儲會在連線和物件建立期間將一些設定標準化。 這些設定不能覆寫，其中包括︰

| 資料庫設定 | 值 |
|:--- |:--- |
| [ANSI_NULLS][ANSI_NULLS] |開啟 |
| [QUOTED_IDENTIFIERS][QUOTED_IDENTIFIERS] |開啟 |
| [DATEFORMAT][DATEFORMAT] |mdy |
| [DATEFIRST][DATEFIRST] |7 |

## <a name="next-steps"></a>後續步驟
若要使用 Visual Studio 連線及查詢，請參閱 [使用 Visual Studio 查詢][Query with Visual Studio]。 若要深入了解驗證選項，請參閱 [適用於 Azure SQL 資料倉儲的驗證][Authentication to Azure SQL Data Warehouse]。

<!--Articles-->
[Query with Visual Studio]: ./sql-data-warehouse-query-visual-studio.md
[Authentication to Azure SQL Data Warehouse]: ./sql-data-warehouse-authentication.md

<!--MSDN references-->
[ADO.NET]: https://msdn.microsoft.com/library/e80y5yhx(v=vs.110).aspx
[ODBC]: https://msdn.microsoft.com/library/jj730314.aspx
[PHP]: https://msdn.microsoft.com/library/cc296172.aspx?f=255&MSPPError=-2147217396
[JDBC]: https://msdn.microsoft.com/library/mt484311(v=sql.110).aspx
[ANSI_NULLS]: https://msdn.microsoft.com/library/ms188048.aspx
[QUOTED_IDENTIFIERS]: https://msdn.microsoft.com/library/ms174393.aspx
[DATEFORMAT]: https://msdn.microsoft.com/library/ms189491.aspx
[DATEFIRST]: https://msdn.microsoft.com/library/ms181598.aspx

<!--Other-->
[Azure portal]: https://portal.azure.com

<!--Image references-->
[1]: media/sql-data-warehouse-connect-overview/server-connect.PNG


