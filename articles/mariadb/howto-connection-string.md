---
title: 將應用程式連線至適用於 MariaDB 的 Azure 資料庫
description: 此文件列出目前支援讓應用程式與「適用於 MariaDB 的 Azure 資料庫」連線的所有連接字串，包括 ADO.NET (C#)、JDBC、Node.js、ODBC、PHP、Python 與 Ruby。
author: ajlam
ms.author: andrela
editor: jasonwhowell
services: mariadb
ms.service: mariadb
ms.topic: conceptual
ms.date: 09/24/2018
ms.openlocfilehash: 706bf395df98bd78fa36273a4948a11d4b59067d
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46967926"
---
# <a name="how-to-connect-applications-to-azure-database-for-mariadb"></a>如何將應用程式連線至適用於 MariaDB 的 Azure 資料庫
此主題列出適用於 MariaDB 的 Azure 資料庫所支援的連接字串類型，以及範本和範例。 您可以在連接字串中使用不同的參數和設定。

- 若要取得憑證，請參閱[如何設定 SSL](./howto-configure-ssl.md)。
- {your_host} = [servername].mariadb.database.azure.com
- {your_user}@{servername} = 正確驗證的 userID 格式。  如果您只使用 userID，驗證將會失敗。

## <a name="adonet"></a>ADO.NET
```csharp
Server={your_host}; Port=3306; Database={your_database}; Uid={username@servername}; Pwd={your_password}; SslMode=Preferred;
```

在此範例中，伺服器名稱是 `mydemoserver`、資料庫名稱是 `wpdb`、使用者名稱是 `WPAdmin`，而密碼是 `mypassword!2`。 因此，連接字串應該是：

```csharp
Server= "mydemoserver.mariadb.database.azure.com"; Port=3306; Database= "wpdb"; Uid= "WPAdmin@mydemoserver"; Pwd="mypassword!2"; SslMode=Required;
```

## <a name="jdbc"></a>JDBC
```java
String url ="jdbc:mariadb://{your_host}:3306/{your_database}?useSSL=true&trustServerCertificate=true"; myDbConn = DriverManager.getConnection(url, "{username@servername}", {your_password});
```

## <a name="nodejs"></a>Node.js
```javascript
var conn = mysql.createConnection({host: "{your_host}", user: "{your_username}", password: {your_password}, database: {your_database}, port: 3306, ssl:{ca:fs.readFileSync({ca-cert filename})}});
```

## <a name="odbc"></a>ODBC
```cpp
DRIVER={MARIADB ODBC 3.0 Driver}; Server="{your_host}"; Port=3306; Database={your_database}; Uid="{username@servername}"; Pwd={your_password}; sslca={ca-cert filename}; sslverify=1;
```

## <a name="php"></a>PHP
```php
$con=mysqli_init(); mysqli_ssl_set($con, NULL, NULL, {ca-cert filename}, NULL, NULL); mysqli_real_connect($con, "{your_host}", "{username@servername}", {your_password}, {your_database}, 3306);
```

## <a name="python"></a>Python
```python
cnx = mysql.connector.connect(user="{username@servername}", password={your_password}, host="{your_host}", port=3306, database={your_database}, ssl_ca={ca-cert filename}, ssl_verify_cert=true)
```

## <a name="ruby"></a>Ruby
```ruby
client = Mysql2::Client.new(username: "{username@servername}", password: {your_password}, database: {your_database}, host: "{your_host}", port: 3306, sslca:{ca-cert filename}, sslverify:false, sslcipher:'AES256-SHA')
```

## <a name="get-the-connection-string-details-from-the-azure-portal"></a>從 Azure 入口網站取得連接字串詳細資料
在 [Azure 入口網站](https://portal.azure.com)中，移至適用於 MariaDB 的 Azure 資料庫伺服器，然後按一下 [連接字串] 以取得您執行個體的字串清單︰![Azure 入口網站中的 [連接字串] 窗格](./media/howto-connection-strings/connection-strings-on-portal.png)

此字串會提供詳細資料，例如驅動程式、伺服器和其他資料庫連線參數。 修改這些範例，以使用您自己的參數 (例如資料庫名稱、密碼等)。 接著，您可以使用這個字串從您的程式碼和應用程式連接到伺服器。

<!-- 
## Next steps
- For more information about connection libraries, see [Concepts - Connection libraries](./concepts-connection-libraries.md).
- -->
