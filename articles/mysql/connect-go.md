---
title: 使用 Go 連線到適用於 MySQL 的 Azure 資料庫
description: 本快速入門提供數個 Go 程式碼範例，您可用於從 Azure Database for MySQL 連線及查詢資料。
services: mysql
author: jasonwhowell
ms.author: jasonh
manager: kfile
editor: jasonwhowell
ms.service: mysql
ms.custom: mvc
ms.devlang: go
ms.topic: quickstart
ms.date: 02/28/2018
ms.openlocfilehash: 8f11453cd7ccdd878e20d80469f12263e72166b7
ms.sourcegitcommit: 1b8665f1fff36a13af0cbc4c399c16f62e9884f3
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/11/2018
ms.locfileid: "35264852"
---
# <a name="azure-database-for-mysql-use-go-language-to-connect-and-query-data"></a>Azure Database for MySQL︰使用 Go 語言連線及查詢資料
本快速入門示範如何從 Windows、Ubuntu Linux 和 Apple macOS 平台使用以 [Go](https://golang.org/) 語言撰寫的程式碼，連線到適用於 MySQL 的 Azure 資料庫。 它會顯示如何使用 SQL 陳述式來查詢、插入、更新和刪除資料庫中的資料。 本主題假設您已熟悉使用 Go 進行開發，但不熟悉適用於 MySQL 的 Azure 資料庫。

## <a name="prerequisites"></a>先決條件
本快速入門使用在以下任一指南中建立的資源作為起點︰
- [使用 Azure 入口網站建立適用於 MySQL 的 Azure 資料庫伺服器](./quickstart-create-mysql-server-database-using-azure-portal.md)
- [使用 Azure CLI 建立適用於 MySQL 的 Azure 資料庫伺服器](./quickstart-create-mysql-server-database-using-azure-cli.md)

## <a name="install-go-and-mysql-connector"></a>安裝 Go 與 MySQL 連接器
在自己的電腦上安裝 [Go](https://golang.org/doc/install) 和 [go-sql-driver for MySQL](https://github.com/go-sql-driver/mysql#installation)。 根據您的平台，遵循適當小節中的步驟：

### <a name="windows"></a>Windows
1. 根據[安裝指示](https://golang.org/doc/install)，[下載](https://golang.org/dl/)並安裝 Go for Microsoft Windows。
2. 從 [開始] 功能表啟動命令提示字元。
3. 為您的專案產生資料夾，例如 `mkdir  %USERPROFILE%\go\src\mysqlgo`。
4. 將目錄切換到專案資料夾，例如 `cd %USERPROFILE%\go\src\mysqlgo`。
5. 將 GOPATH 環境變數設定為指向來源程式碼目錄。 `set GOPATH=%USERPROFILE%\go`。
6. 執行 `go get github.com/go-sql-driver/mysql` 命令以安裝 [go-sql-driver for mysql](https://github.com/go-sql-driver/mysql#installation)。

   總而言之，就是安裝 Go，然後在命令提示字元中執行下列命令：
   ```cmd
   mkdir  %USERPROFILE%\go\src\mysqlgo
   cd %USERPROFILE%\go\src\mysqlgo
   set GOPATH=%USERPROFILE%\go
   go get github.com/go-sql-driver/mysql
   ```

### <a name="linux-ubuntu"></a>Linux (Ubuntu)
1. 啟動 Bash 殼層。 
2. 執行 `sudo apt-get install golang-go` 以安裝 Go。
3. 在主目錄中為您的專案產生資料夾，例如 `mkdir -p ~/go/src/mysqlgo/`。
4. 將目錄切換到此資料夾，例如 `cd ~/go/src/mysqlgo/`。
5. 將 GOPATH 環境變數設定為指向有效的來源目錄，例如目前主目錄的 go 資料夾。 在 Bash 殼層，執行 `export GOPATH=~/go` 以將 go 目錄新增為目前殼層工作階段的 GOPATH。
6. 執行 `go get github.com/go-sql-driver/mysql` 命令以安裝 [go-sql-driver for mysql](https://github.com/go-sql-driver/mysql#installation)。

   總而言之，就是執行下列 bash 命令：
   ```bash
   sudo apt-get install golang-go
   mkdir -p ~/go/src/mysqlgo/
   cd ~/go/src/mysqlgo/
   export GOPATH=~/go/
   go get github.com/go-sql-driver/mysql
   ```

### <a name="apple-macos"></a>Apple macOS
1. 根據符合您平台的[安裝指示](https://golang.org/doc/install)，下載並安裝 Go。 
2. 啟動 Bash 殼層。
3. 在主目錄中為您的專案產生資料夾，例如 `mkdir -p ~/go/src/mysqlgo/`。
4. 將目錄切換到此資料夾，例如 `cd ~/go/src/mysqlgo/`。
5. 將 GOPATH 環境變數設定為指向有效的來源目錄，例如目前主目錄的 go 資料夾。 在 Bash 殼層，執行 `export GOPATH=~/go` 以將 go 目錄新增為目前殼層工作階段的 GOPATH。
6. 執行 `go get github.com/go-sql-driver/mysql` 命令以安裝 [go-sql-driver for mysql](https://github.com/go-sql-driver/mysql#installation)。

   總而言之，就是安裝 Go，然後執行下列 bash 命令：
   ```bash
   mkdir -p ~/go/src/mysqlgo/
   cd ~/go/src/mysqlgo/
   export GOPATH=~/go/
   go get github.com/go-sql-driver/mysql
   ```

## <a name="get-connection-information"></a>取得連線資訊
取得連線到 Azure Database for MySQL 所需的連線資訊。 您需要完整的伺服器名稱和登入認證。

1. 登入 [Azure 入口網站](https://portal.azure.com/)。
2. 從 Azure 入口網站的左側功能表中，按一下 [所有資源]，然後搜尋您所建立的伺服器 (例如 **mydemoserver**)。
3. 按一下伺服器名稱。
4. 從伺服器的 [概觀] 面板，記下 [伺服器名稱] 和 [伺服器管理員登入名稱]。 如果您忘記密碼，您也可以從此面板重設密碼。
 ![Azure Database for MySQL 伺服器名稱](./media/connect-go/1_server-overview-name-login.png)
   

## <a name="build-and-run-go-code"></a>建置並執行 Go 程式碼 
1. 若要撰寫 Golang 程式碼，您可以使用簡單的文字編輯器，例如 Microsoft Windows 的記事本、Ubuntu 的 [vi](http://manpages.ubuntu.com/manpages/xenial/man1/nvi.1.html#contenttoc5) 或 [Nano](https://www.nano-editor.org/)，或 macOS 的 TextEdit。 如果想要使用更豐富的互動式開發環境 (IDE)，可以選擇 Jetbrains 的 [Gogland](https://www.jetbrains.com/go/)、Microsoft 的 [Visual Studio Code](https://code.visualstudio.com/)，或 [Atom](https://atom.io/)。
2. 將 Go 程式碼從下列區段貼到文字檔中，然後將它們儲存到專案資料夾中 (副檔名為 \*.go)，(例如 Windows 路徑 `%USERPROFILE%\go\src\mysqlgo\createtable.go` 或 Linux 路徑 `~/go/src/mysqlgo/createtable.go`)。
3. 在程式碼中找出 `HOST`、`DATABASE`、`USER` 和 `PASSWORD` 常數，然後將範例值取代成您自己的值。 
4. 啟動命令提示字元或 Bash 殼層。 將目錄切換到專案資料夾。 例如，在 Windows 上為 `cd %USERPROFILE%\go\src\mysqlgo\`。 在 Linux 上為 `cd ~/go/src/mysqlgo/`。  提及的部分 IDE 編輯器提供偵錯和執行階段功能，並不需要殼層命令。
5. 輸入命令 `go run createtable.go` 來編譯應用程式並加以執行，以執行程式碼。 
6. 或者，若要將程式碼建置到原生應用程式 `go build createtable.go`，則啟動 `createtable.exe` 來執行應用程式。

## <a name="connect-create-table-and-insert-data"></a>連線、建立資料表及插入資料
使用下列程式碼搭配 **INSERT** SQL 陳述式來連線至伺服器、建立資料表，以及載入資料。 

程式碼會匯入三個套件：[sql 套件](https://golang.org/pkg/database/sql/)、[適用於 mysql 的 go sql 驅動程式](https://github.com/go-sql-driver/mysql#installation) (作為驅動程式來與 Azure Database for MySQL 通訊)，以及 [fmt 套件](https://golang.org/pkg/fmt/) (適用於命令列上列印的輸入和輸出)。

程式碼會呼叫 [sql.Open()](http://go-database-sql.org/accessing.html) 以連線到Azure Database for MySQL，並使用 [db.Ping()](https://golang.org/pkg/database/sql/#DB.Ping) 方法檢查連線。 [資料庫控制代碼](https://golang.org/pkg/database/sql/#DB)會到處使用，並保留資料庫伺服器的連線集區。 程式碼會呼叫 [Exec()](https://golang.org/pkg/database/sql/#DB.Exec) 方法數次，以執行數個 DDL 命令。 程式碼也會使用 [Prepare()](http://go-database-sql.org/prepared.html) 和 Exec() 來執行具有不同參數的備妥陳述式，以插入三個資料列。 每次都會使用自訂 checkError() 方法來檢查是否在發生錯誤時會緊急結束。

以您自己的值取代 `host`、`database`、`user` 和 `password` 常數。 

```Go
package main

import (
    "database/sql"
    "fmt"

    _ "github.com/go-sql-driver/mysql"
)

const (
    host     = "mydemoserver.mysql.database.azure.com"
    database = "quickstartdb"
    user     = "myadmin@mydemoserver"
    password = "yourpassword"
)

func checkError(err error) {
    if err != nil {
        panic(err)
    }
}

func main() {

    // Initialize connection string.
    var connectionString = fmt.Sprintf("%s:%s@tcp(%s:3306)/%s?allowNativePasswords=true", user, password, host, database)

    // Initialize connection object.
    db, err := sql.Open("mysql", connectionString)
    checkError(err)
    defer db.Close()

    err = db.Ping()
    checkError(err)
    fmt.Println("Successfully created connection to database.")

    // Drop previous table of same name if one exists.
    _, err = db.Exec("DROP TABLE IF EXISTS inventory;")
    checkError(err)
    fmt.Println("Finished dropping table (if existed).")

    // Create table.
    _, err = db.Exec("CREATE TABLE inventory (id serial PRIMARY KEY, name VARCHAR(50), quantity INTEGER);")
    checkError(err)
    fmt.Println("Finished creating table.")

    // Insert some data into table.
    sqlStatement, err := db.Prepare("INSERT INTO inventory (name, quantity) VALUES (?, ?);")
    res, err := sqlStatement.Exec("banana", 150)
    checkError(err)
    rowCount, err := res.RowsAffected()
    fmt.Printf("Inserted %d row(s) of data.\n", rowCount)

    res, err = sqlStatement.Exec("orange", 154)
    checkError(err)
    rowCount, err = res.RowsAffected()
    fmt.Printf("Inserted %d row(s) of data.\n", rowCount)

    res, err = sqlStatement.Exec("apple", 100)
    checkError(err)
    rowCount, err = res.RowsAffected()
    fmt.Printf("Inserted %d row(s) of data.\n", rowCount)
    fmt.Println("Done.")
}

```

## <a name="read-data"></a>讀取資料
使用下列程式碼搭配 **SELECT** SQL 陳述式來連線和讀取資料。 

程式碼會匯入三個套件：[sql 套件](https://golang.org/pkg/database/sql/)、[適用於 mysql 的 go sql 驅動程式](https://github.com/go-sql-driver/mysql#installation) (作為驅動程式來與 Azure Database for MySQL 通訊)，以及 [fmt 套件](https://golang.org/pkg/fmt/) (適用於命令列上列印的輸入和輸出)。

程式碼會呼叫 [sql.Open()](http://go-database-sql.org/accessing.html) 以連線到Azure Database for MySQL，並使用 [db.Ping()](https://golang.org/pkg/database/sql/#DB.Ping) 方法檢查連線。 [資料庫控制代碼](https://golang.org/pkg/database/sql/#DB)會到處使用，並保留資料庫伺服器的連線集區。 程式碼會呼叫 [Query()](https://golang.org/pkg/database/sql/#DB.Query) 方法來執行 select 命令。 然後它會執行 [Next()](https://golang.org/pkg/database/sql/#Rows.Next) 來逐一查看結果集，以及執行 [Scan()](https://golang.org/pkg/database/sql/#Rows.Scan) 來剖析資料行值，並將值儲存到變數中。 每次都會使用自訂 checkError() 方法來檢查是否在發生錯誤時會緊急結束。

以您自己的值取代 `host`、`database`、`user` 和 `password` 常數。 

```Go
package main

import (
    "database/sql"
    "fmt"

    _ "github.com/go-sql-driver/mysql"
)

const (
    host     = "mydemoserver.mysql.database.azure.com"
    database = "quickstartdb"
    user     = "myadmin@mydemoserver"
    password = "yourpassword"
)

func checkError(err error) {
    if err != nil {
        panic(err)
    }
}

func main() {

    // Initialize connection string.
    var connectionString = fmt.Sprintf("%s:%s@tcp(%s:3306)/%s?allowNativePasswords=true", user, password, host, database)

    // Initialize connection object.
    db, err := sql.Open("mysql", connectionString)
    checkError(err)
    defer db.Close()

    err = db.Ping()
    checkError(err)
    fmt.Println("Successfully created connection to database.")

    // Variables for printing column data when scanned.
    var (
        id       int
        name     string
        quantity int
    )

    // Read some data from the table.
    rows, err := db.Query("SELECT id, name, quantity from inventory;")
    checkError(err)
    defer rows.Close()
    fmt.Println("Reading data:")
    for rows.Next() {
        err := rows.Scan(&id, &name, &quantity)
        checkError(err)
        fmt.Printf("Data row = (%d, %s, %d)\n", id, name, quantity)
    }
    err = rows.Err()
    checkError(err)
    fmt.Println("Done.")
}
```

## <a name="update-data"></a>更新資料
使用下列程式碼搭配 **UPDATE** SQL 陳述式來連線和更新資料。 

程式碼會匯入三個套件：[sql 套件](https://golang.org/pkg/database/sql/)、[適用於 mysql 的 go sql 驅動程式](https://github.com/go-sql-driver/mysql#installation) (作為驅動程式來與 Azure Database for MySQL 通訊)，以及 [fmt 套件](https://golang.org/pkg/fmt/) (適用於命令列上列印的輸入和輸出)。

程式碼會呼叫 [sql.Open()](http://go-database-sql.org/accessing.html) 以連線到Azure Database for MySQL，並使用 [db.Ping()](https://golang.org/pkg/database/sql/#DB.Ping) 方法檢查連線。 [資料庫控制代碼](https://golang.org/pkg/database/sql/#DB)會到處使用，並保留資料庫伺服器的連線集區。 程式碼會呼叫 [Exec()](https://golang.org/pkg/database/sql/#DB.Exec) 方法來執行 update 命令。 每次都會使用自訂 checkError() 方法來檢查是否在發生錯誤時會緊急結束。

以您自己的值取代 `host`、`database`、`user` 和 `password` 常數。 

```Go
package main

import (
    "database/sql"
    "fmt"

    _ "github.com/go-sql-driver/mysql"
)

const (
    host     = "mydemoserver.mysql.database.azure.com"
    database = "quickstartdb"
    user     = "myadmin@mydemoserver"
    password = "yourpassword"
)

func checkError(err error) {
    if err != nil {
        panic(err)
    }
}

func main() {

    // Initialize connection string.
    var connectionString = fmt.Sprintf("%s:%s@tcp(%s:3306)/%s?allowNativePasswords=true", user, password, host, database)

    // Initialize connection object.
    db, err := sql.Open("mysql", connectionString)
    checkError(err)
    defer db.Close()

    err = db.Ping()
    checkError(err)
    fmt.Println("Successfully created connection to database.")

    // Modify some data in table.
    rows, err := db.Exec("UPDATE inventory SET quantity = ? WHERE name = ?", 200, "banana")
    checkError(err)
    rowCount, err := rows.RowsAffected()
    fmt.Printf("Deleted %d row(s) of data.\n", rowCount)
    fmt.Println("Done.")
}
```

## <a name="delete-data"></a>刪除資料
使用下列程式碼搭配 **DELETE** SQL 陳述式來連線和移除資料。 

程式碼會匯入三個套件：[sql 套件](https://golang.org/pkg/database/sql/)、[適用於 mysql 的 go sql 驅動程式](https://github.com/go-sql-driver/mysql#installation) (作為驅動程式來與 Azure Database for MySQL 通訊)，以及 [fmt 套件](https://golang.org/pkg/fmt/) (適用於命令列上列印的輸入和輸出)。

程式碼會呼叫 [sql.Open()](http://go-database-sql.org/accessing.html) 以連線到Azure Database for MySQL，並使用 [db.Ping()](https://golang.org/pkg/database/sql/#DB.Ping) 方法檢查連線。 [資料庫控制代碼](https://golang.org/pkg/database/sql/#DB)會到處使用，並保留資料庫伺服器的連線集區。 程式碼會呼叫 [Exec()](https://golang.org/pkg/database/sql/#DB.Exec) 方法來執行 delete 命令。 每次都會使用自訂 checkError() 方法來檢查是否在發生錯誤時會緊急結束。

以您自己的值取代 `host`、`database`、`user` 和 `password` 常數。 

```Go
package main

import (
    "database/sql"
    "fmt"
    _ "github.com/go-sql-driver/mysql"
)

const (
    host     = "mydemoserver.mysql.database.azure.com"
    database = "quickstartdb"
    user     = "myadmin@mydemoserver"
    password = "yourpassword"
)

func checkError(err error) {
    if err != nil {
        panic(err)
    }
}

func main() {

    // Initialize connection string.
    var connectionString = fmt.Sprintf("%s:%s@tcp(%s:3306)/%s?allowNativePasswords=true", user, password, host, database)

    // Initialize connection object.
    db, err := sql.Open("mysql", connectionString)
    checkError(err)
    defer db.Close()

    err = db.Ping()
    checkError(err)
    fmt.Println("Successfully created connection to database.")

    // Modify some data in table.
    rows, err := db.Exec("DELETE FROM inventory WHERE name = ?", "orange")
    checkError(err)
    rowCount, err := rows.RowsAffected()
    fmt.Printf("Deleted %d row(s) of data.\n", rowCount)
    fmt.Println("Done.")
}
```

## <a name="next-steps"></a>後續步驟
> [!div class="nextstepaction"]
> [使用匯出和匯入來移轉資料庫](./concepts-migrate-import-export.md)
