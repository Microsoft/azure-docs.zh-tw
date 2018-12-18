---
title: 從 Python 連線至適用於 PostgreSQL 的 Azure 資料庫
description: 本快速入門提供 Python 程式碼範例，您可用於從 Azure Database for PostgreSQL 連線及查詢資料。
services: postgresql
author: rachel-msft
ms.author: raagyema
manager: kfile
editor: jasonwhowell
ms.service: postgresql
ms.custom: mvc, devcenter
ms.devlang: python
ms.topic: quickstart
ms.date: 02/28/2018
ms.openlocfilehash: 926b35f49169c5a87e82996ba714aad40ba5244c
ms.sourcegitcommit: c765cbd9c379ed00f1e2394374efa8e1915321b9
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 02/28/2018
ms.locfileid: "29687314"
---
# <a name="azure-database-for-postgresql-use-python-to-connect-and-query-data"></a>Azure Database for PostgreSQL︰使用 Python 連線及查詢資料
本快速入門示範如何使用 [Python](https://python.org) 連線至 Azure Database for PostgreSQL。 它也會示範如何使用 SQL 陳述式查詢、插入、更新和刪除 macOS、Ubuntu Linux 和 Windows 平台的資料庫中的資料。 本文中的步驟假設您已熟悉使用 Python 進行開發，但不熟悉 Azure Database for PostgreSQL。

## <a name="prerequisites"></a>先決條件
本快速入門使用在以下任一指南中建立的資源作為起點︰
- [建立 DB - 入口網站](quickstart-create-server-database-portal.md)
- [建立 DB - CLI](quickstart-create-server-database-azure-cli.md)

您也會需要：
- 已安裝 [python](https://www.python.org/downloads/)
- 已安裝 [pip](https://pip.pypa.io/en/stable/installing/) 套件 (如果您使用從 [python.org](https://python.org) 下載的 Python 2 >=2.7.9 或 Python 3 >=3.4 二進位檔，便已安裝 pip)。

## <a name="install-the-python-connection-libraries-for-postgresql"></a>安裝適用於 PostgreSQL 的 Python 連線程式庫
安裝 [psycopg2](http://initd.org/psycopg/docs/install.html) 套件，這可讓您連線及查詢資料庫。 psycopg2 [可在 PyPI 上取得](https://pypi.python.org/pypi/psycopg2/)，其形式為適用於大多數常見平台 (Linux、OSX、Windows) 的 [wheel](http://pythonwheels.com/) 套件。 使用 pip install 來取得模組的二進位版本 (包括所有相依性)。

1. 在自己的電腦上啟動命令列介面：
    - 在 Linux 上啟動 Bash 殼層。
    - 在 macOS 上啟動終端機。
    - 在 Windows 上，從 [開始] 功能表啟動命令提示字元。
2. 執行以下命令，確定您使用最新版的 pip：
    ```cmd
    pip install -U pip
    ```

3. 執行以下命令來安裝 psycopg2 套件：
    ```cmd
    pip install psycopg2
    ```

## <a name="get-connection-information"></a>取得連線資訊
取得連線到 Azure Database for PostgreSQL 所需的連線資訊。 您需要完整的伺服器名稱和登入認證。

1. 登入 [Azure 入口網站](https://portal.azure.com/)。
2. 從 Azure 入口網站的左側功能表中，按一下 [所有資源]，然後搜尋您所建立的伺服器 (例如 **mydemoserver**)。
3. 按一下伺服器名稱。
4. 從伺服器的 [概觀] 面板，記下 [伺服器名稱] 和 [伺服器管理員登入名稱]。 如果您忘記密碼，您也可以從此面板重設密碼。
 ![適用於 PostgreSQL 的 Azure 資料庫伺服器名稱](./media/connect-python/1-connection-string.png)

## <a name="how-to-run-python-code"></a>如何執行 Python 程式碼
本文總共包含四個程式碼範例，各自會執行一項特定函數。 下列指示指出如何建立文字檔、插入程式碼區塊，然後儲存檔案，以便稍後執行。 請務必建立四個不同的檔案，每個程式碼區塊使用一個檔案。

- 使用您慣用的文字編輯器，建立新的檔案。
- 將下列各節中的其中一個程式碼範例複製並貼在文字檔案中。 以您建立伺服器和資料庫時所指定的值，取代 **host**、**dbname**、**user** 和 **password** 參數。
- 將具有 .Py 副檔名的檔案 (例如 postgres.py) 儲存到您的專案資料夾中。 如果您是執行 Windows，請務必在儲存檔案時選取 UTF-8 編碼。 
- 啟動命令提示字元、終端機或 Bash 殼層，然後將目錄變更為您的專案資料夾，例如 `cd postgres`。
-  若要執行程式碼，請鍵入後面接著檔案名稱的 python 命令，例如 `Python postgres.py`。

> [!NOTE]
> 從 Python 第 3 版開始，執行下列程式碼區塊時，您可能會看到錯誤 `SyntaxError: Missing parentheses in call to 'print'`：如果發生這種情況，請使用括號，將命令 `print "string"` 的每個呼叫取代為函式呼叫，例如 `print("string")`。

## <a name="connect-create-table-and-insert-data"></a>連線、建立資料表及插入資料
使用 [psycopg2.connect](http://initd.org/psycopg/docs/connection.html) 函式搭配 **INSERT** SQL 陳述式，利用下列程式碼來連線和載入資料。 [cursor.execute](http://initd.org/psycopg/docs/cursor.html#execute) 函式用來對 PostgreSQL 資料庫執行 SQL 查詢。 以建立伺服器和資料庫時所指定的值，取代主機、資料庫名稱、使用者和密碼參數。

```Python
import psycopg2

# Update connection string information obtained from the portal
host = "mydemoserver.postgres.database.azure.com"
user = "mylogin@mydemoserver"
dbname = "mypgsqldb"
password = "<server_admin_password>"
sslmode = "require"

# Construct connection string
conn_string = "host={0} user={1} dbname={2} password={3} sslmode={4}".format(host, user, dbname, password, sslmode)
conn = psycopg2.connect(conn_string) 
print "Connection established"

cursor = conn.cursor()

# Drop previous table of same name if one exists
cursor.execute("DROP TABLE IF EXISTS inventory;")
print "Finished dropping table (if existed)"

# Create table
cursor.execute("CREATE TABLE inventory (id serial PRIMARY KEY, name VARCHAR(50), quantity INTEGER);")
print "Finished creating table"

# Insert some data into table
cursor.execute("INSERT INTO inventory (name, quantity) VALUES (%s, %s);", ("banana", 150))
cursor.execute("INSERT INTO inventory (name, quantity) VALUES (%s, %s);", ("orange", 154))
cursor.execute("INSERT INTO inventory (name, quantity) VALUES (%s, %s);", ("apple", 100))
print "Inserted 3 rows of data"

# Cleanup
conn.commit()
cursor.close()
conn.close()
```

程式碼執行成功之後，輸出會如下所示：

![命令列輸出](media/connect-python/2-example-python-output.png)

## <a name="read-data"></a>讀取資料
使用 [cursor.execute](http://initd.org/psycopg/docs/cursor.html#execute) 函式搭配 **SELECT** SQL 陳述式，利用下列程式碼來讀取插入的資料。 此函式會接受查詢並傳回結果集，而您可以使用 [cursor.fetchall()](http://initd.org/psycopg/docs/cursor.html#cursor.fetchall) 反覆查詢結果集。 以建立伺服器和資料庫時所指定的值，取代主機、資料庫名稱、使用者和密碼參數。

```Python
import psycopg2

# Update connection string information obtained from the portal
host = "mydemoserver.postgres.database.azure.com"
user = "mylogin@mydemoserver"
dbname = "mypgsqldb"
password = "<server_admin_password>"
sslmode = "require"

# Construct connection string
conn_string = "host={0} user={1} dbname={2} password={3} sslmode={4}".format(host, user, dbname, password, sslmode)
conn = psycopg2.connect(conn_string) 
print "Connection established"

cursor = conn.cursor()

# Fetch all rows from table
cursor.execute("SELECT * FROM inventory;")
rows = cursor.fetchall()

# Print all rows
for row in rows:
    print "Data row = (%s, %s, %s)" %(str(row[0]), str(row[1]), str(row[2]))

# Cleanup
conn.commit()
cursor.close()
conn.close()
```

## <a name="update-data"></a>更新資料
使用下列程式碼，藉由使用 [cursor.execute](http://initd.org/psycopg/docs/cursor.html#execute) 函式搭配 **UPDATE** SQL 陳述式來更新您先前插入的清查資料列。 以建立伺服器和資料庫時所指定的值，取代主機、資料庫名稱、使用者和密碼參數。

```Python
import psycopg2

# Update connection string information obtained from the portal
host = "mydemoserver.postgres.database.azure.com"
user = "mylogin@mydemoserver"
dbname = "mypgsqldb"
password = "<server_admin_password>"
sslmode = "require"

# Construct connection string
conn_string = "host={0} user={1} dbname={2} password={3} sslmode={4}".format(host, user, dbname, password, sslmode)
conn = psycopg2.connect(conn_string) 
print "Connection established"

cursor = conn.cursor()

# Update a data row in the table
cursor.execute("UPDATE inventory SET quantity = %s WHERE name = %s;", (200, "banana"))
print "Updated 1 row of data"

# Cleanup
conn.commit()
cursor.close()
conn.close()
```

## <a name="delete-data"></a>刪除資料
使用下列程式碼，藉由使用 [cursor.execute](http://initd.org/psycopg/docs/cursor.html#execute) 函式搭配 **DELETE** SQL 陳述式來刪除您先前插入的清查項目。 以建立伺服器和資料庫時所指定的值，取代主機、資料庫名稱、使用者和密碼參數。

```Python
import psycopg2

# Update connection string information obtained from the portal
host = "mydemoserver.postgres.database.azure.com"
user = "mylogin@mydemoserver"
dbname = "mypgsqldb"
password = "<server_admin_password>"
sslmode = "require"

# Construct connection string
conn_string = "host={0} user={1} dbname={2} password={3} sslmode={4}".format(host, user, dbname, password, sslmode)
conn = psycopg2.connect(conn_string) 
print "Connection established"

cursor = conn.cursor()

# Delete data row from table
cursor.execute("DELETE FROM inventory WHERE name = %s;", ("orange",))
print "Deleted 1 row of data"

# Cleanup
conn.commit()
cursor.close()
conn.close()
```

## <a name="next-steps"></a>後續步驟
> [!div class="nextstepaction"]
> [使用匯出和匯入來移轉資料庫](./howto-migrate-using-export-and-import.md)
