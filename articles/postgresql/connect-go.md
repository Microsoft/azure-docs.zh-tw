---
title: 使用 Go 語言連線到適用於 PostgreSQL 的 Azure 資料庫
description: 本快速入門提供 Go 程式設計語言範例，您可用於從 Azure Database for PostgreSQL 連線及查詢資料。
services: postgresql
author: rachel-msft
ms.author: raagyema
manager: kfile
editor: jasonwhowell
ms.service: postgresql
ms.custom: mvc
ms.devlang: go
ms.topic: quickstart
ms.date: 02/28/2018
ms.openlocfilehash: d3bcfb3369510bdbcf325eab41fb7eacf3e2a228
ms.sourcegitcommit: d78bcecd983ca2a7473fff23371c8cfed0d89627
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 05/14/2018
ms.locfileid: "34166313"
---
# <a name="azure-database-for-postgresql-use-go-language-to-connect-and-query-data"></a>Azure Database for PostgreSQL︰使用 Go 語言連線及查詢資料
本快速入門示範如何使用以 [Go](https://golang.org/) 語言 (golang) 撰寫的程式碼來連線到 Azure Database for PostgreSQL。 它會顯示如何使用 SQL 陳述式來查詢、插入、更新和刪除資料庫中的資料。 本文假設您已熟悉使用 Go 進行開發，但不熟悉 Azure Database for PostgreSQL。

## <a name="prerequisites"></a>先決條件
本快速入門使用在以下任一指南中建立的資源作為起點︰
- [建立 DB - 入口網站](quickstart-create-server-database-portal.md)
- [建立 DB - Azure CLI](quickstart-create-server-database-azure-cli.md)

## <a name="install-go-and-pq-connector"></a>安裝 Go 和 pq 連接器
在您自己的機器上安裝 [Go](https://golang.org/doc/install) 和 [Pure Go Postgres 驅動程式 (pq)](https://github.com/lib/pq)。 根據您的平台，遵循適當的步驟：

### <a name="windows"></a>Windows
1. 根據[安裝指示](https://golang.org/doc/install)，[下載](https://golang.org/dl/)並安裝 Go for Microsoft Windows。
2. 從 [開始] 功能表啟動命令提示字元。
3. 為您的專案產生資料夾，例如 `mkdir  %USERPROFILE%\go\src\postgresqlgo`。
4. 將目錄切換到專案資料夾，例如 `cd %USERPROFILE%\go\src\postgresqlgo`。
5. 將 GOPATH 環境變數設定為指向來源程式碼目錄。 `set GOPATH=%USERPROFILE%\go`。
6. 執行 `go get github.com/lib/pq` 命令來安裝 [Pure Go Postgres 驅動程式 (pq)](https://github.com/lib/pq)。

   總而言之，就是安裝 Go，然後在命令提示字元中執行下列命令：
   ```cmd
   mkdir  %USERPROFILE%\go\src\postgresqlgo
   cd %USERPROFILE%\go\src\postgresqlgo
   set GOPATH=%USERPROFILE%\go
   go get github.com/lib/pq
   ```

### <a name="linux-ubuntu"></a>Linux (Ubuntu)
1. 啟動 Bash 殼層。 
2. 執行 `sudo apt-get install golang-go` 以安裝 Go。
3. 在主目錄中為您的專案產生資料夾，例如 `mkdir -p ~/go/src/postgresqlgo/`。
4. 將目錄切換到此資料夾，例如 `cd ~/go/src/postgresqlgo/`。
5. 將 GOPATH 環境變數設定為指向有效的來源目錄，例如目前主目錄的 go 資料夾。 在 Bash 殼層，執行 `export GOPATH=~/go` 以將 go 目錄新增為目前殼層工作階段的 GOPATH。
6. 執行 `go get github.com/lib/pq` 命令來安裝 [Pure Go Postgres 驅動程式 (pq)](https://github.com/lib/pq)。

   總而言之，就是執行下列 bash 命令：
   ```bash
   sudo apt-get install golang-go
   mkdir -p ~/go/src/postgresqlgo/
   cd ~/go/src/postgresqlgo/
   export GOPATH=~/go/
   go get github.com/lib/pq
   ```

### <a name="apple-macos"></a>Apple macOS
1. 根據符合您平台的[安裝指示](https://golang.org/doc/install)，下載並安裝 Go。 
2. 啟動 Bash 殼層。 
3. 在主目錄中為您的專案產生資料夾，例如 `mkdir -p ~/go/src/postgresqlgo/`。
4. 將目錄切換到此資料夾，例如 `cd ~/go/src/postgresqlgo/`。
5. 將 GOPATH 環境變數設定為指向有效的來源目錄，例如目前主目錄的 go 資料夾。 在 Bash 殼層，執行 `export GOPATH=~/go` 以將 go 目錄新增為目前殼層工作階段的 GOPATH。
6. 執行 `go get github.com/lib/pq` 命令來安裝 [Pure Go Postgres 驅動程式 (pq)](https://github.com/lib/pq)。

   總而言之，就是安裝 Go，然後執行下列 bash 命令：
   ```bash
   mkdir -p ~/go/src/postgresqlgo/
   cd ~/go/src/postgresqlgo/
   export GOPATH=~/go/
   go get github.com/lib/pq
   ```

## <a name="get-connection-information"></a>取得連線資訊
取得連線到 Azure Database for PostgreSQL 所需的連線資訊。 您需要完整的伺服器名稱和登入認證。

1. 登入 [Azure 入口網站](https://portal.azure.com/)。
2. 從 Azure 入口網站的左側功能表中，按一下 [所有資源]，然後搜尋您所建立的伺服器 (例如 **mydemoserver**)。
3. 按一下伺服器名稱。
4. 從伺服器的 [概觀] 面板，記下 [伺服器名稱] 和 [伺服器管理員登入名稱]。 如果您忘記密碼，您也可以從此面板重設密碼。
 ![適用於 PostgreSQL 的 Azure 資料庫伺服器名稱](./media/connect-go/1-connection-string.png)

## <a name="build-and-run-go-code"></a>建置並執行 Go 程式碼 
1. 若要撰寫 Golang 程式碼，您可以使用純文字編輯器，例如 Microsoft Windows 的記事本、Ubuntu 的 [vi](http://manpages.ubuntu.com/manpages/xenial/man1/nvi.1.html#contenttoc5) 或 [Nano](https://www.nano-editor.org/)，或 macOS 的 TextEdit。 如果想要使用更豐富的互動式開發環境 (IDE)，您可以選擇 Jetbrains 的 [Gogland](https://www.jetbrains.com/go/)、Microsoft 的 [Visual Studio Code](https://code.visualstudio.com/)，或 [Atom](https://atom.io/)。
2. 將下列各節的 Golang 程式碼貼到文字檔中，並加上副檔名 \*.go 來儲存到專案資料夾，例如 Windows 路徑 `%USERPROFILE%\go\src\postgresqlgo\createtable.go` 或 Linux 路徑 `~/go/src/postgresqlgo/createtable.go`。
3. 在程式碼中找出 `HOST`、`DATABASE`、`USER` 和 `PASSWORD` 常數，然後將範例值換成您自己的值。  
4. 啟動命令提示字元或 bash 殼層。 將目錄切換到專案資料夾。 例如，在 Windows 上為 `cd %USERPROFILE%\go\src\postgresqlgo\`。 在 Linux 上執行 `cd ~/go/src/postgresqlgo/`。 提及的部分 IDE 環境提供偵錯和執行階段功能，並不需要殼層命令。
5. 輸入命令 `go run createtable.go` 來編譯應用程式並加以執行，以執行程式碼。 
6. 或者，若要將程式碼建置到原生應用程式，請執行 `go build createtable.go`，然後啟動 `createtable.exe` 來執行應用程式。

## <a name="connect-and-create-a-table"></a>連線及建立資料表
使用下列程式碼搭配 **CREATE TABLE** SQL 陳述式 (後面接著 **INSERT INTO** SQL 陳述式) 來連線和建立資料表，進而將資料列新增至資料表中。

程式碼會匯入三個封裝：[sql 套件](https://golang.org/pkg/database/sql/)、[pq 封裝](http://godoc.org/github.com/lib/pq) 會作為驅動程式來與 PostgreSQL 伺服器通訊，而 [fmt 封裝](https://golang.org/pkg/fmt/)會用於命令列上的列印輸入和輸出。

程式碼會呼叫 [sql.Open()](http://godoc.org/github.com/lib/pq#Open) 方法來連線至適用於 PostgreSQL 的 Azure 資料庫，並使用 [db.Ping()](https://golang.org/pkg/database/sql/#DB.Ping) 方法檢查連線。 [資料庫控制代碼](https://golang.org/pkg/database/sql/#DB)會到處使用，並保留資料庫伺服器的連線集區。 程式碼會呼叫 [Exec()](https://golang.org/pkg/database/sql/#DB.Exec) 方法數次以執行數個 SQL 命令。 每次自訂 checkError() 方法皆會檢查是否發生錯誤，一旦發生錯誤即會緊急結束。

以您自己的值取代 `HOST`、`DATABASE`、`USER` 和 `PASSWORD` 參數。 

```go
package main

import (
    "database/sql"
    "fmt"
    _ "github.com/lib/pq"
)

const (
    // Initialize connection constants.
    HOST     = "mydemoserver.postgres.database.azure.com"
    DATABASE = "mypgsqldb"
    USER     = "mylogin@mydemoserver"
    PASSWORD = "<server_admin_password>"
)

func checkError(err error) {
    if err != nil {
        panic(err)
    }
}

func main() {
    // Initialize connection string.
    var connectionString string = fmt.Sprintf("host=%s user=%s password=%s dbname=%s sslmode=require", HOST, USER, PASSWORD, DATABASE)

    // Initialize connection object.
    db, err := sql.Open("postgres", connectionString)
    checkError(err)

    err = db.Ping()
    checkError(err)
    fmt.Println("Successfully created connection to database")

    // Drop previous table of same name if one exists.
    _, err = db.Exec("DROP TABLE IF EXISTS inventory;")
    checkError(err)
    fmt.Println("Finished dropping table (if existed)")

    // Create table.
    _, err = db.Exec("CREATE TABLE inventory (id serial PRIMARY KEY, name VARCHAR(50), quantity INTEGER);")
    checkError(err)
    fmt.Println("Finished creating table")

    // Insert some data into table.
    sql_statement := "INSERT INTO inventory (name, quantity) VALUES ($1, $2);"
    _, err = db.Exec(sql_statement, "banana", 150)
    checkError(err)
    _, err = db.Exec(sql_statement, "orange", 154)
    checkError(err)
    _, err = db.Exec(sql_statement, "apple", 100)
    checkError(err)
    fmt.Println("Inserted 3 rows of data")
}
```

## <a name="read-data"></a>讀取資料
使用下列程式碼搭配 **SELECT** SQL 陳述式來連線和讀取資料。 

程式碼會匯入三個封裝：[sql 套件](https://golang.org/pkg/database/sql/)、[pq 封裝](http://godoc.org/github.com/lib/pq) 會作為驅動程式來與 PostgreSQL 伺服器通訊，而 [fmt 封裝](https://golang.org/pkg/fmt/)會用於命令列上的列印輸入和輸出。

程式碼會呼叫 [sql.Open()](http://godoc.org/github.com/lib/pq#Open) 方法來連線至適用於 PostgreSQL 的 Azure 資料庫，並使用 [db.Ping()](https://golang.org/pkg/database/sql/#DB.Ping) 方法檢查連線。 [資料庫控制代碼](https://golang.org/pkg/database/sql/#DB)會到處使用，並保留資料庫伺服器的連線集區。 呼叫 [db.Query()](https://golang.org/pkg/database/sql/#DB.Query) 方法可執行 select 查詢，而結果產生的資料列會保留在類型為 [rows](https://golang.org/pkg/database/sql/#Rows) 的變數中。 程式碼會使用 [rows.Scan()](https://golang.org/pkg/database/sql/#Rows.Scan) 方法讀取目前資料列中的資料行資料值，並使用 [rows.Next()](https://golang.org/pkg/database/sql/#Rows.Next) 迭代器對資料列執行迴圈，直到再也沒有資料列存在為止。 每個資料列的資料行值都會列印到主控台。每次自訂 checkError() 方法皆會檢查是否發生錯誤，一旦發生錯誤即會緊急結束。

以您自己的值取代 `HOST`、`DATABASE`、`USER` 和 `PASSWORD` 參數。 

```go
package main

import (
    "database/sql"
    "fmt"
    _ "github.com/lib/pq"
)

const (
    // Initialize connection constants.
    HOST     = "mydemoserver.postgres.database.azure.com"
    DATABASE = "mypgsqldb"
    USER     = "mylogin@mydemoserver"
    PASSWORD = "<server_admin_password>"
)

func checkError(err error) {
    if err != nil {
        panic(err)
    }
}

func main() {

    // Initialize connection string.
    var connectionString string = fmt.Sprintf("host=%s user=%s password=%s dbname=%s sslmode=require", HOST, USER, PASSWORD, DATABASE)

    // Initialize connection object.
    db, err := sql.Open("postgres", connectionString)
    checkError(err)

    err = db.Ping()
    checkError(err)
    fmt.Println("Successfully created connection to database")

    // Read rows from table.
    var id int
    var name string
    var quantity int

    sql_statement := "SELECT * from inventory;"
    rows, err := db.Query(sql_statement)
    checkError(err)
    defer rows.Close()

    for rows.Next() {
        switch err := rows.Scan(&id, &name, &quantity); err {
        case sql.ErrNoRows:
            fmt.Println("No rows were returned")
        case nil:
            fmt.Printf("Data row = (%d, %s, %d)\n", id, name, quantity)
        default:
            checkError(err)
        }
    }
}
```

## <a name="update-data"></a>更新資料
使用下列程式碼搭配 **UPDATE** SQL 陳述式來連線和更新資料。

程式碼會匯入三個套件：[sql 套件](https://golang.org/pkg/database/sql/)、[pq 套件](http://godoc.org/github.com/lib/pq)作為驅動程式來與 Postgres 伺服器通訊，以及 [fmt 套件](https://golang.org/pkg/fmt/) (適用於命令列上列印的輸入和輸出)。

程式碼會呼叫 [sql.Open()](http://godoc.org/github.com/lib/pq#Open) 方法來連線至適用於 PostgreSQL 的 Azure 資料庫，並使用 [db.Ping()](https://golang.org/pkg/database/sql/#DB.Ping) 方法檢查連線。 [資料庫控制代碼](https://golang.org/pkg/database/sql/#DB)會到處使用，並保留資料庫伺服器的連線集區。 程式碼會呼叫 [Exec()](https://golang.org/pkg/database/sql/#DB.Exec) 方法來執行可更新資料表的 SQL 陳述式。 使用自訂 checkError() 方法檢查是否發生錯誤，一旦發生錯誤即會緊急結束。

以您自己的值取代 `HOST`、`DATABASE`、`USER` 和 `PASSWORD` 參數。 
```go
package main

import (
  "database/sql"
  _ "github.com/lib/pq"
  "fmt"
)

const (
    // Initialize connection constants.
    HOST     = "mydemoserver.postgres.database.azure.com"
    DATABASE = "mypgsqldb"
    USER     = "mylogin@mydemoserver"
    PASSWORD = "<server_admin_password>"
)

func checkError(err error) {
    if err != nil {
        panic(err)
    }
}

func main() {
    
    // Initialize connection string.
    var connectionString string = 
        fmt.Sprintf("host=%s user=%s password=%s dbname=%s sslmode=require", HOST, USER, PASSWORD, DATABASE)

    // Initialize connection object.
    db, err := sql.Open("postgres", connectionString)
    checkError(err)

    err = db.Ping()
    checkError(err)
    fmt.Println("Successfully created connection to database")

    // Modify some data in table.
    sql_statement := "UPDATE inventory SET quantity = $2 WHERE name = $1;"
    _, err = db.Exec(sql_statement, "banana", 200)
    checkError(err)
    fmt.Println("Updated 1 row of data")
}
```

## <a name="delete-data"></a>刪除資料
使用下列程式碼搭配 **DELETE** SQL 陳述式來連線和刪除資料。 

程式碼會匯入三個套件：[sql 套件](https://golang.org/pkg/database/sql/)、[pq 套件](http://godoc.org/github.com/lib/pq)作為驅動程式來與 Postgres 伺服器通訊，以及 [fmt 套件](https://golang.org/pkg/fmt/) (適用於命令列上列印的輸入和輸出)。

程式碼會呼叫 [sql.Open()](http://godoc.org/github.com/lib/pq#Open) 方法來連線至適用於 PostgreSQL 的 Azure 資料庫，並使用 [db.Ping()](https://golang.org/pkg/database/sql/#DB.Ping) 方法檢查連線。 [資料庫控制代碼](https://golang.org/pkg/database/sql/#DB)會到處使用，並保留資料庫伺服器的連線集區。 程式碼會呼叫 [Exec()](https://golang.org/pkg/database/sql/#DB.Exec) 方法，來執行從資料表中刪除資料列的 SQL 陳述式。 使用自訂 checkError() 方法檢查是否發生錯誤，一旦發生錯誤即會緊急結束。

以您自己的值取代 `HOST`、`DATABASE`、`USER` 和 `PASSWORD` 參數。 
```go
package main

import (
  "database/sql"
  _ "github.com/lib/pq"
  "fmt"
)

const (
    // Initialize connection constants.
    HOST     = "mydemoserver.postgres.database.azure.com"
    DATABASE = "mypgsqldb"
    USER     = "mylogin@mydemoserver"
    PASSWORD = "<server_admin_password>"
)

func checkError(err error) {
    if err != nil {
        panic(err)
    }
}

func main() {
    
    // Initialize connection string.
    var connectionString string = 
        fmt.Sprintf("host=%s user=%s password=%s dbname=%s sslmode=require", HOST, USER, PASSWORD, DATABASE)

    // Initialize connection object.
    db, err := sql.Open("postgres", connectionString)
    checkError(err)

    err = db.Ping()
    checkError(err)
    fmt.Println("Successfully created connection to database")

    // Delete some data from table.
    sql_statement := "DELETE FROM inventory WHERE name = $1;"
    _, err = db.Exec(sql_statement, "orange")
    checkError(err)
    fmt.Println("Deleted 1 row of data")
}
```

## <a name="next-steps"></a>後續步驟
> [!div class="nextstepaction"]
> [使用匯出和匯入來移轉資料庫](./howto-migrate-using-export-and-import.md)
