---
title: 使用 Go 查詢 Azure SQL Database | Microsoft Docs
description: 使用 Go 來建立連線到 Azure SQL Database 的程式，並使用 Transact-SQL 陳述式查詢並修改資料。
services: sql-database
ms.service: sql-database
ms.subservice: development
ms.custom: ''
ms.devlang: go
ms.topic: quickstart
author: David-Engel
ms.author: v-daveng
ms.reviewer: MightyPen
manager: craigg
ms.date: 09/07/2018
ms.openlocfilehash: 94da0ad79b0e01f3baa7f20661e45f873c742fd6
ms.sourcegitcommit: cc4fdd6f0f12b44c244abc7f6bc4b181a2d05302
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/25/2018
ms.locfileid: "47063365"
---
# <a name="use-go-to-query-an-azure-sql-database"></a>使用 Go 查詢 Azure SQL 資料庫

本快速入門示範如何使用 [Go](https://godoc.org/github.com/denisenkom/go-mssqldb) 連線至 Azure SQL 資料庫。 也會示範用以查詢及修改資料的 TRANSACT-SQL 陳述式。

## <a name="prerequisites"></a>必要條件

若要完成本快速入門，請確定您具備下列必要條件︰

[!INCLUDE [prerequisites-create-db](../../includes/sql-database-connect-query-prerequisites-create-db-includes.md)]

- 在此快速入門中，您所使用電腦的公用 IP 位址[伺服器層級防火牆規則](sql-database-get-started-portal-firewall.md)。

- 您已安裝適用於您作業系統的 Go 和相關軟體：

    - **MacOS**：安裝 Homebrew 和 GoLang。 請參閱[步驟 1.2](https://www.microsoft.com/sql-server/developer-get-started/go/mac/)。
    - **Ubuntu**：安裝 GoLang。 請參閱[步驟 1.2](https://www.microsoft.com/sql-server/developer-get-started/go/ubuntu/)。
    - **Windows**：安裝 GoLang。 請參閱[步驟 1.2](https://www.microsoft.com/sql-server/developer-get-started/go/windows/)。    

## <a name="sql-server-connection-information"></a>SQL Server 連線資訊

[!INCLUDE [prerequisites-server-connection-info](../../includes/sql-database-connect-query-prerequisites-server-connection-info-includes.md)]

## <a name="create-go-project-and-dependencies"></a>建立 Go 專案和相依性

1. 從終端機建立名為 **SqlServerSample** 的新專案資料夾。 

   ```bash
   mkdir SqlServerSample
   ```

2. 將目錄變更為 **SqlServerSample**，然後取得並安裝適用於 Go 的 SQL Server 驅動程式：

   ```bash
   cd SqlServerSample
   go get github.com/denisenkom/go-mssqldb
   go install github.com/denisenkom/go-mssqldb
   ```

## <a name="create-sample-data"></a>建立範例資料

1. 使用您慣用的文字編輯器，在 **SqlServerSample** 資料夾中建立名為 **CreateTestData.sql** 的檔案。 複製下列 T-SQL 程式碼並在該檔案中貼上。 此程式碼會建立結構描述、資料表，並插入幾個資料列。

   ```sql
   CREATE SCHEMA TestSchema;
   GO

   CREATE TABLE TestSchema.Employees (
     Id       INT IDENTITY(1,1) NOT NULL PRIMARY KEY,
     Name     NVARCHAR(50),
     Location NVARCHAR(50)
   );
   GO

   INSERT INTO TestSchema.Employees (Name, Location) VALUES
     (N'Jared',  N'Australia'),
     (N'Nikita', N'India'),
     (N'Tom',    N'Germany');
   GO

   SELECT * FROM TestSchema.Employees;
   GO
   ```

2. 使用 sqlcmd 連線到資料庫，並執行 SQL 指令碼來建立結構描述、資料表，並插入幾個資料列。 將您的伺服器、資料庫、使用者和密碼取代為適當的值。

   ```bash
   sqlcmd -S your_server.database.windows.net -U your_username -P your_password -d your_database -i ./CreateTestData.sql
   ```

## <a name="insert-code-to-query-sql-database"></a>插入程式碼以查詢 SQL 資料庫

1. 在 **SqlServerSample** 資料夾中建立名為 **sample.go** 的檔案。

2. 開啟檔案，並以下列程式碼取代其內容。 為您的伺服器、資料庫、使用者和密碼新增適當的值。 本範例會使用 GoLang Context 方法，以確保與資料庫伺服器的連線是作用中狀態。

   ```go
   package main

   import (
       _ "github.com/denisenkom/go-mssqldb"
       "database/sql"
       "context"
       "log"
       "fmt"
       "errors"
   )

   var db *sql.DB

   var server = "your_server.database.windows.net"
   var port = 1433
   var user = "your_username"
   var password = "your_password"
   var database = "your_database"

   func main() {
       // Build connection string
       connString := fmt.Sprintf("server=%s;user id=%s;password=%s;port=%d;database=%s;",
           server, user, password, port, database)

       var err error

       // Create connection pool
       db, err = sql.Open("sqlserver", connString)
       if err != nil {
           log.Fatal("Error creating connection pool: ", err.Error())
       }
       ctx := context.Background()
       err = db.PingContext(ctx)
       if err != nil {
           log.Fatal(err.Error())
       }
       fmt.Printf("Connected!\n")

       // Create employee
       createID, err := CreateEmployee("Jake", "United States")
       if err != nil {
           log.Fatal("Error creating Employee: ", err.Error())
       }
       fmt.Printf("Inserted ID: %d successfully.\n", createID)

       // Read employees
       count, err := ReadEmployees()
       if err != nil {
           log.Fatal("Error reading Employees: ", err.Error())
       }
       fmt.Printf("Read %d row(s) successfully.\n", count)

       // Update from database
       updatedRows, err := UpdateEmployee("Jake", "Poland")
       if err != nil {
           log.Fatal("Error updating Employee: ", err.Error())
       }
       fmt.Printf("Updated %d row(s) successfully.\n", updatedRows)

       // Delete from database
       deletedRows, err := DeleteEmployee("Jake")
       if err != nil {
           log.Fatal("Error deleting Employee: ", err.Error())
       }
       fmt.Printf("Deleted %d row(s) successfully.\n", deletedRows)
   }

   // CreateEmployee inserts an employee record
   func CreateEmployee(name string, location string) (int64, error) {
       ctx := context.Background()
       var err error

       if db == nil {
           err = errors.New("CreateEmployee: db is null")
           return -1, err
       }

       // Check if database is alive.
       err = db.PingContext(ctx)
       if err != nil {
           return -1, err
       }

       tsql := "INSERT INTO TestSchema.Employees (Name, Location) VALUES (@Name, @Location); select convert(bigint, SCOPE_IDENTITY());"

       stmt, err := db.Prepare(tsql)
       if err != nil {
          return -1, err
       }
       defer stmt.Close()

       row := stmt.QueryRowContext(
           ctx,
           sql.Named("Name", name),
           sql.Named("Location", location))
       var newID int64
       err = row.Scan(&newID)
       if err != nil {
           return -1, err
       }

       return newID, nil
   }

   // ReadEmployees reads all employee records
   func ReadEmployees() (int, error) {
       ctx := context.Background()

       // Check if database is alive.
       err := db.PingContext(ctx)
       if err != nil {
           return -1, err
       }

       tsql := fmt.Sprintf("SELECT Id, Name, Location FROM TestSchema.Employees;")

       // Execute query
       rows, err := db.QueryContext(ctx, tsql)
       if err != nil {
           return -1, err
       }

       defer rows.Close()

       var count int

       // Iterate through the result set.
       for rows.Next() {
           var name, location string
           var id int

           // Get values from row.
           err := rows.Scan(&id, &name, &location)
           if err != nil {
               return -1, err
           }

           fmt.Printf("ID: %d, Name: %s, Location: %s\n", id, name, location)
           count++
       }

       return count, nil
   }

   // UpdateEmployee updates an employee's information
   func UpdateEmployee(name string, location string) (int64, error) {
       ctx := context.Background()

       // Check if database is alive.
       err := db.PingContext(ctx)
       if err != nil {
           return -1, err
       }

       tsql := fmt.Sprintf("UPDATE TestSchema.Employees SET Location = @Location WHERE Name = @Name")

       // Execute non-query with named parameters
       result, err := db.ExecContext(
           ctx,
           tsql,
           sql.Named("Location", location),
           sql.Named("Name", name))
       if err != nil {
           return -1, err
       }

       return result.RowsAffected()
   }

   // DeleteEmployee deletes an employee from the database
   func DeleteEmployee(name string) (int64, error) {
       ctx := context.Background()

       // Check if database is alive.
       err := db.PingContext(ctx)
       if err != nil {
           return -1, err
       }

       tsql := fmt.Sprintf("DELETE FROM TestSchema.Employees WHERE Name = @Name;")

       // Execute non-query with named parameters
       result, err := db.ExecContext(ctx, tsql, sql.Named("Name", name))
       if err != nil {
           return -1, err
       }

       return result.RowsAffected()
   }
   ```

## <a name="run-the-code"></a>執行程式碼

1. 在命令提示字元中，執行下列命令：

   ```bash
   go run sample.go
   ```

2. 確認輸出：

   ```text
   Connected!
   Inserted ID: 4 successfully.
   ID: 1, Name: Jared, Location: Australia
   ID: 2, Name: Nikita, Location: India
   ID: 3, Name: Tom, Location: Germany
   ID: 4, Name: Jake, Location: United States
   Read 4 row(s) successfully.
   Updated 1 row(s) successfully.
   Deleted 1 row(s) successfully.
   ```

## <a name="next-steps"></a>後續步驟

- [設計您的第一個 Azure SQL Database](sql-database-design-first-database.md)
- [Go Driver for Microsoft SQL Server](https://github.com/denisenkom/go-mssqldb)
- [回報問題或發問](https://github.com/denisenkom/go-mssqldb/issues)

