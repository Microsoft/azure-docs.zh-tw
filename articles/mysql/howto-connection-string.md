---
title: 將應用程式連線至適用於 MySQL 的 Azure 資料庫
description: 本文件列出目前支援讓應用程式與「適用於 MySQL 的 Azure 資料庫」連線的所有連接字串，包括 ADO.NET (C#)、JDBC、Node.js、ODBC、PHP、Python 和 Ruby。
services: mysql
author: ajlam
ms.author: andrela
editor: jasonwhowell
manager: kfile
ms.service: mysql
ms.topic: article
ms.date: 02/28/2018
ms.openlocfilehash: 06bd91adb0a86198f7709d0989624657ce00dfa9
ms.sourcegitcommit: 387d7edd387a478db181ca639db8a8e43d0d75f7
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/10/2018
ms.locfileid: "40037981"
---
# <a name="how-to-connect-applications-to-azure-database-for-mysql"></a>如何將應用程式連線至適用於 MySQL 的 Azure 資料庫
本主題列出適用於 MySQL 的 Azure 資料庫所支援的連接字串類型，以及範本和範例。 您可以在連接字串中使用不同的參數和設定。

- 若要取得憑證，請參閱[如何設定 SSL](./howto-configure-ssl.md)。
- {your_host} = <servername>.mysql.database.azure.com
- {your_user}@{servername} = 正確驗證的 userID 格式。  如果您只使用 userID，驗證將會失敗。

## <a name="adonet"></a>ADO.NET
```ado.net
Server={your_host};Port={your_port};Database={your_database};Uid={username@servername};Pwd={your_password};[SslMode=Required;]
```

在此範例中，伺服器名稱是 `mydemoserver`、資料庫名稱是 `wpdb`、使用者名稱是 `WPAdmin`，而密碼是 `mypassword!2`。 因此，連接字串應該是：

```ado.net
Server= "mydemoserver.mysql.database.azure.com"; Port=3306; Database= "wpdb"; Uid= "WPAdmin@mydemoserver"; Pwd="mypassword!2"; SslMode=Required;
```

## <a name="jdbc"></a>JDBC
```jdbc
String url ="jdbc:mysql://%s:%s/%s[?verifyServerCertificate=true&useSSL=true&requireSSL=true]",{your_host},{your_port},{your_database}"; myDbConn = DriverManager.getConnection(url, {username@servername}, {your_password}";
```

## <a name="nodejs"></a>Node.js
```node.js
var conn = mysql.createConnection({host: {your_host}, user: {username@servername}, password: {your_password}, database: {your_database}, Port: {your_port}[, ssl:{ca:fs.readFileSync({ca-cert filename})}}]);
```

## <a name="odbc"></a>ODBC
```odbc
DRIVER={MySQL ODBC 5.3 UNICODE Driver};Server={your_host};Port={your_port};Database={your_database};Uid={username@servername};Pwd={your_password}; [sslca={ca-cert filename}; sslverify=1; Option=3;]
```

## <a name="php"></a>PHP
```php
$con=mysqli_init(); [mysqli_ssl_set($con, NULL, NULL, {ca-cert filename}, NULL, NULL);] mysqli_real_connect($con, {your_host}, {username@servername}, {your_password}, {your_database}, {your_port});
```

## <a name="python"></a>Python
```python
cnx = mysql.connector.connect(user={username@servername}, password={your_password}, host={your_host}, port={your_port}, database={your_database}[, ssl_ca={ca-cert filename}, ssl_verify_cert=true])
```

## <a name="ruby"></a>Ruby
```ruby
client = Mysql2::Client.new(username: {username@servername}, password: {your_password}, database: {your_database}, host: {your_host}, port: {your_port}[, sslca:{ca-cert filename}, sslverify:false, sslcipher:'AES256-SHA'])
```

## <a name="get-the-connection-string-details-from-the-azure-portal"></a>從 Azure 入口網站取得連接字串詳細資料
在 [Azure 入口網站](https://portal.azure.com)中，移至適用於 MySQL 的 Azure 資料庫伺服器，然後按一下 [連接字串] 以取得您執行個體的字串清單︰![Azure 入口網站中的 [連接字串] 窗格](./media/howto-connection-strings/connection-strings-on-portal.png)

此字串會提供詳細資料，例如驅動程式、伺服器和其他資料庫連線參數。 修改這些範例，以使用您自己的參數 (例如資料庫名稱、密碼等)。 接著，您可以使用這個字串從您的程式碼和應用程式連接到伺服器。

## <a name="next-steps"></a>後續步驟
- 如需有關連線程式庫的詳細資訊，請參閱[概念 - 連線程式庫](./concepts-connection-libraries.md)。
