---
title: 從 Python 連線到適用於 MySQL 的 Azure 資料庫
description: 本快速入門提供數個 Python 程式碼範例，您可用於從 Azure Database for MySQL 連線和查詢資料。
services: mysql
author: jasonwhowell
ms.author: jasonh
manager: kfile
editor: jasonwhowell
ms.service: mysql
ms.custom: mvc
ms.devlang: python
ms.topic: quickstart
ms.date: 02/28/2018
ms.openlocfilehash: 7eef3d71a35b5016e48e519b95c2573fbe3390e7
ms.sourcegitcommit: 1b8665f1fff36a13af0cbc4c399c16f62e9884f3
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/11/2018
ms.locfileid: "35265100"
---
# <a name="azure-database-for-mysql-use-python-to-connect-and-query-data"></a>Azure Database for MySQL︰使用 Python 連線和查詢資料
本快速入門示範如何使用 [Python](https://python.org) 連線至 Azure Database for MySQL。 它會使用 SQL 陳述式查詢、插入、更新和刪除 Mac OS、Ubuntu Linux 和 Windows 平台的資料庫中的資料。 本主題假設您已熟悉使用 Python 進行開發，但不熟悉適用於 MySQL 的 Azure 資料庫。

## <a name="prerequisites"></a>先決條件
本快速入門使用在以下任一指南中建立的資源作為起點︰
- [使用 Azure 入口網站建立適用於 MySQL 的 Azure 資料庫伺服器](./quickstart-create-mysql-server-database-using-azure-portal.md)
- [使用 Azure CLI 建立適用於 MySQL 的 Azure 資料庫伺服器](./quickstart-create-mysql-server-database-using-azure-cli.md)

## <a name="install-python-and-the-mysql-connector"></a>安裝 Python 和 MySQL 連接器
在自己的電腦上安裝 [Python](https://www.python.org/downloads/) 和 [Python 的 MySQL 連接器](https://dev.mysql.com/downloads/connector/python/)。 根據您的平台，遵循適當小節中的步驟：

### <a name="windows"></a>Windows
1. 從 [python.org](https://www.python.org/downloads/windows/) 下載並安裝 Python 2.7。 
2. 啟動命令提示字元，以檢查 Python 安裝。 執行使用大寫 V 切換參數的 `C:\python27\python.exe -V` 命令，來查看版本號碼。
3. 從對應至您 Python 版本的 [mysql.com](https://dev.mysql.com/downloads/connector/python/) 安裝 MySQL 的 Python 連接器。

### <a name="linux-ubuntu"></a>Linux (Ubuntu)
1. 在 Linux (Ubuntu) 中，Python 通常會安裝成預設安裝的一部分。
2. 啟動 bash shell，以檢查 Python 安裝。 執行使用大寫 V 切換參數的 `python -V` 命令，來查看版本號碼。
3. 執行 `pip show pip -V` 命令以檢查 PIP 安裝，來查看版本號碼。 
4. PIP 可能包含在某些 Python 版本中。 如果未安裝 PIP，您可執行 `sudo apt-get install python-pip` 命令來安裝 [PIP](https://pip.pypa.io/en/stable/installing/) 套件。
5. 執行 `pip install -U pip` 命令，以將 PIP 更新成最新版本。
6. 使用 PIP 命令，以安裝 Python 的 MySQL 連接器和其相依性：

   ```bash
   sudo pip install mysql-connector-python-rf
   ```
 
### <a name="macos"></a>MacOS
1. 在 Mac OS 中，Python 通常會安裝成預設 OS 安裝的一部分。
2. 啟動 bash shell，以檢查 Python 安裝。 執行使用大寫 V 切換參數的 `python -V` 命令，來查看版本號碼。
3. 執行 `pip show pip -V` 命令以檢查 PIP 安裝，來查看版本號碼。
4. PIP 可能包含在某些 Python 版本中。 如果未安裝 PIP，您可能會安裝 [PIP](https://pip.pypa.io/en/stable/installing/) 套件。
5. 執行 `pip install -U pip` 命令，以將 PIP 更新成最新版本。
6. 使用 PIP 命令，以安裝 Python 的 MySQL 連接器和其相依性：

   ```bash
   pip install mysql-connector-python-rf
   ```

## <a name="get-connection-information"></a>取得連線資訊
取得連線到 Azure Database for MySQL 所需的連線資訊。 您需要完整的伺服器名稱和登入認證。

1. 登入 [Azure 入口網站](https://portal.azure.com/)。
2. 從 Azure 入口網站的左側功能表中，按一下 [所有資源]，然後搜尋您所建立的伺服器 (例如 **mydemoserver**)。
3. 按一下伺服器名稱。
4. 從伺服器的 [概觀] 面板，記下 [伺服器名稱] 和 [伺服器管理員登入名稱]。 如果您忘記密碼，您也可以從此面板重設密碼。
 ![Azure Database for MySQL 伺服器名稱](./media/connect-python/1_server-overview-name-login.png)
   

## <a name="run-python-code"></a>執行 Python 程式碼
- 將程式碼貼入文字檔，然後使用 .py 副檔名來將檔案儲存到專案資料夾 (例如 C:\pythonmysql\createtable.py 或 /home/username/pythonmysql/createtable.py)。
- 若要執行程式碼，請啟動命令提示字元或 Bash 殼層。 將目錄切換到專案資料夾 `cd pythonmysql`。 然後，鍵入後接檔案名稱 `python createtable.py` 的 python 命令以執行應用程式。 在 Windows 作業系統上，如果找不到 python.exe，您可能需要提供可執行檔的完整路徑，或將 Python 路徑新增至 path 環境變數。 `C:\python27\python.exe createtable.py`

## <a name="connect-create-table-and-insert-data"></a>連線、建立資料表及插入資料
使用下列程式碼搭配 **INSERT** SQL 陳述式來連線至伺服器、建立資料表，以及載入資料。 

在程式碼中，匯入 mysql.connector 程式庫。 使用 [connect()](https://dev.mysql.com/doc/connector-python/en/connector-python-api-mysql-connector-connect.html) 函式，以使用 config 集合中的[連線引數](https://dev.mysql.com/doc/connector-python/en/connector-python-connectargs.html)來連線至 Azure Database for MySQL。 此程式碼使用連線上的資料指標，而 [cursor.execute()](https://dev.mysql.com/doc/connector-python/en/connector-python-api-mysqlcursor-execute.html) 方法會對 MySQL 資料庫執行 SQL 查詢。 

將 `host`、`user`、`password` 和 `database` 參數取代為您在建立伺服器和資料庫時所指定的值。

```Python
import mysql.connector
from mysql.connector import errorcode

# Obtain connection string information from the portal
config = {
  'host':'mydemoserver.mysql.database.azure.com',
  'user':'myadmin@mydemoserver',
  'password':'yourpassword',
  'database':'quickstartdb'
}

# Construct connection string
try:
   conn = mysql.connector.connect(**config)
   print("Connection established")
except mysql.connector.Error as err:
  if err.errno == errorcode.ER_ACCESS_DENIED_ERROR:
    print("Something is wrong with the user name or password")
  elif err.errno == errorcode.ER_BAD_DB_ERROR:
    print("Database does not exist")
  else:
    print(err)
else:
  cursor = conn.cursor()

  # Drop previous table of same name if one exists
  cursor.execute("DROP TABLE IF EXISTS inventory;")
  print("Finished dropping table (if existed).")

  # Create table
  cursor.execute("CREATE TABLE inventory (id serial PRIMARY KEY, name VARCHAR(50), quantity INTEGER);")
  print("Finished creating table.")

  # Insert some data into table
  cursor.execute("INSERT INTO inventory (name, quantity) VALUES (%s, %s);", ("banana", 150))
  print("Inserted",cursor.rowcount,"row(s) of data.")
  cursor.execute("INSERT INTO inventory (name, quantity) VALUES (%s, %s);", ("orange", 154))
  print("Inserted",cursor.rowcount,"row(s) of data.")
  cursor.execute("INSERT INTO inventory (name, quantity) VALUES (%s, %s);", ("apple", 100))
  print("Inserted",cursor.rowcount,"row(s) of data.")

  # Cleanup
  conn.commit()
  cursor.close()
  conn.close()
  print("Done.")
```

## <a name="read-data"></a>讀取資料
使用下列程式碼搭配 **SELECT** SQL 陳述式來連線和讀取資料。 

在程式碼中，匯入 mysql.connector 程式庫。 使用 [connect()](https://dev.mysql.com/doc/connector-python/en/connector-python-api-mysql-connector-connect.html) 函式，以使用 config 集合中的[連線引數](https://dev.mysql.com/doc/connector-python/en/connector-python-connectargs.html)來連線至 Azure Database for MySQL。 此程式碼使用連線上的資料指標，而 [cursor.execute()](https://dev.mysql.com/doc/connector-python/en/connector-python-api-mysqlcursor-execute.html) 方法會對 MySQL 資料庫執行 SQL 陳述式。 使用 [fetchall()](https://dev.mysql.com/doc/connector-python/en/connector-python-api-mysqlcursor-fetchall.html) 方法來讀取資料列。 結果集會保留在集合資料列中，並且使用 for 迭代器重複執行資料列。

將 `host`、`user`、`password` 和 `database` 參數取代為您在建立伺服器和資料庫時所指定的值。

```Python
import mysql.connector
from mysql.connector import errorcode

# Obtain connection string information from the portal
config = {
  'host':'mydemoserver.mysql.database.azure.com',
  'user':'myadmin@mydemoserver',
  'password':'yourpassword',
  'database':'quickstartdb'
}

# Construct connection string
try:
   conn = mysql.connector.connect(**config)
   print("Connection established")
except mysql.connector.Error as err:
  if err.errno == errorcode.ER_ACCESS_DENIED_ERROR:
    print("Something is wrong with the user name or password")
  elif err.errno == errorcode.ER_BAD_DB_ERROR:
    print("Database does not exist")
  else:
    print(err)
else:
  cursor = conn.cursor()

  # Read data
  cursor.execute("SELECT * FROM inventory;")
  rows = cursor.fetchall()
  print("Read",cursor.rowcount,"row(s) of data.")

  # Print all rows
  for row in rows:
    print("Data row = (%s, %s, %s)" %(str(row[0]), str(row[1]), str(row[2])))

  # Cleanup
  conn.commit()
  cursor.close()
  conn.close()
  print("Done.")
```

## <a name="update-data"></a>更新資料
使用下列程式碼搭配 **UPDATE** SQL 陳述式來連線和更新資料。 

在程式碼中，匯入 mysql.connector 程式庫。  使用 [connect()](https://dev.mysql.com/doc/connector-python/en/connector-python-api-mysql-connector-connect.html) 函式，以使用 config 集合中的[連線引數](https://dev.mysql.com/doc/connector-python/en/connector-python-connectargs.html)來連線至 Azure Database for MySQL。 此程式碼使用連線上的資料指標，而 [cursor.execute()](https://dev.mysql.com/doc/connector-python/en/connector-python-api-mysqlcursor-execute.html) 方法會對 MySQL 資料庫執行 SQL 陳述式。 

將 `host`、`user`、`password` 和 `database` 參數取代為您在建立伺服器和資料庫時所指定的值。

```Python
import mysql.connector
from mysql.connector import errorcode

# Obtain connection string information from the portal
config = {
  'host':'mydemoserver.mysql.database.azure.com',
  'user':'myadmin@mydemoserver',
  'password':'yourpassword',
  'database':'quickstartdb'
}

# Construct connection string
try:
   conn = mysql.connector.connect(**config)
   print("Connection established")
except mysql.connector.Error as err:
  if err.errno == errorcode.ER_ACCESS_DENIED_ERROR:
    print("Something is wrong with the user name or password")
  elif err.errno == errorcode.ER_BAD_DB_ERROR:
    print("Database does not exist")
  else:
    print(err)
else:
  cursor = conn.cursor()

  # Update a data row in the table
  cursor.execute("UPDATE inventory SET quantity = %s WHERE name = %s;", (200, "banana"))
  print("Updated",cursor.rowcount,"row(s) of data.")

  # Cleanup
  conn.commit()
  cursor.close()
  conn.close()
  print("Done.")
```

## <a name="delete-data"></a>刪除資料
使用下列程式碼搭配 **DELETE** SQL 陳述式來連線和移除資料。 

在程式碼中，匯入 mysql.connector 程式庫。  使用 [connect()](https://dev.mysql.com/doc/connector-python/en/connector-python-api-mysql-connector-connect.html) 函式，以使用 config 集合中的[連線引數](https://dev.mysql.com/doc/connector-python/en/connector-python-connectargs.html)來連線至 Azure Database for MySQL。 此程式碼使用連線上的資料指標，而 [cursor.execute()](https://dev.mysql.com/doc/connector-python/en/connector-python-api-mysqlcursor-execute.html) 方法會對 MySQL 資料庫執行 SQL 查詢。 

將 `host`、`user`、`password` 和 `database` 參數取代為您在建立伺服器和資料庫時所指定的值。

```Python
import mysql.connector
from mysql.connector import errorcode

# Obtain connection string information from the portal
config = {
  'host':'mydemoserver.mysql.database.azure.com',
  'user':'myadmin@mydemoserver',
  'password':'yourpassword',
  'database':'quickstartdb'
}

# Construct connection string
try:
   conn = mysql.connector.connect(**config)
   print("Connection established.")
except mysql.connector.Error as err:
  if err.errno == errorcode.ER_ACCESS_DENIED_ERROR:
    print("Something is wrong with the user name or password.")
  elif err.errno == errorcode.ER_BAD_DB_ERROR:
    print("Database does not exist.")
  else:
    print(err)
else:
  cursor = conn.cursor()

  # Delete a data row in the table
  cursor.execute("DELETE FROM inventory WHERE name=%(param1)s;", {'param1':"orange"})
  print("Deleted",cursor.rowcount,"row(s) of data.")

  # Cleanup
  conn.commit()
  cursor.close()
  conn.close()
  print("Done.")
```

## <a name="next-steps"></a>後續步驟
> [!div class="nextstepaction"]
> [使用匯出和匯入來移轉資料庫](./concepts-migrate-import-export.md)
