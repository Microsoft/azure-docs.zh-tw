---
title: 使用 Ruby 查詢 Azure SQL Database | Microsoft Docs
description: 本主題說明如何使用 Ruby 來建立連線到 Azure SQL Database 的程式，並使用 Transact-SQL 陳述式查詢。
services: sql-database
ms.service: sql-database
ms.subservice: development
ms.custom: ''
ms.devlang: ruby
ms.topic: quickstart
author: CarlRabeler
ms.author: carlrab
ms.reviewer: ''
manager: craigg
ms.date: 04/01/2018
ms.openlocfilehash: 7b2652c1e25c2815518ac533bde5bced5b3ee635
ms.sourcegitcommit: cc4fdd6f0f12b44c244abc7f6bc4b181a2d05302
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/25/2018
ms.locfileid: "47063399"
---
# <a name="use-ruby-to-query-an-azure-sql-database"></a>使用 Ruby 查詢 Azure SQL 資料庫

此快速入門示範如何使用 [Ruby](https://www.ruby-lang.org) 建立程式以連線至 Azure SQL 資料庫，並使用 Transact-SQL 陳述式來查詢資料。

## <a name="prerequisites"></a>必要條件

若要完成本快速入門，請確定您具備下列必要條件︰

[!INCLUDE [prerequisites-create-db](../../includes/sql-database-connect-query-prerequisites-create-db-includes.md)]

- 在此快速入門中，您所使用電腦的公用 IP 位址[伺服器層級防火牆規則](sql-database-get-started-portal-firewall.md)。

- 您已安裝適用於您作業系統的 Ruby 和相關軟體：
    - **MacOS**：安裝 Homebrew、安裝 rbenv 和 ruby-build、安裝 Ruby，然後安裝 FreeTDS。 請參閱[步驟 1.2、1.3、1.4 和 1.5](https://www.microsoft.com/sql-server/developer-get-started/ruby/mac/)。
    - **Ubuntu**：安裝 Ruby 的必要條件、安裝 rbenv 和 ruby-build、安裝 Ruby，然後安裝 FreeTDS。 請參閱[步驟 1.2、1.3、1.4 和 1.5](https://www.microsoft.com/sql-server/developer-get-started/ruby/ubuntu/)。

## <a name="sql-server-connection-information"></a>SQL Server 連線資訊

[!INCLUDE [prerequisites-server-connection-info](../../includes/sql-database-connect-query-prerequisites-server-connection-info-includes.md)]

> [!IMPORTANT]
> 在您執行本教學課程的電腦上，公用 IP 位址必須有防火牆規則。 如果您在不同電腦上或有不同的公用 IP 位址，請建立[使用 Azure 入口網站的伺服器層級防火牆規則](sql-database-get-started-portal-firewall.md)。 

## <a name="insert-code-to-query-sql-database"></a>插入程式碼以查詢 SQL 資料庫

1. 在您慣用的文字編輯器中，建立新的檔案 **sqltest.rb**

2. 使用下列程式碼取代內容，並為您的伺服器、資料庫、使用者和密碼新增適當的值。

```ruby
require 'tiny_tds'
server = 'your_server.database.windows.net'
database = 'your_database'
username = 'your_username'
password = 'your_password'
client = TinyTds::Client.new username: username, password: password, 
    host: server, port: 1433, database: database, azure: true

puts "Reading data from table"
tsql = "SELECT TOP 20 pc.Name as CategoryName, p.name as ProductName
        FROM [SalesLT].[ProductCategory] pc
        JOIN [SalesLT].[Product] p
        ON pc.productcategoryid = p.productcategoryid"
result = client.execute(tsql)
result.each do |row|
    puts row
end
```

## <a name="run-the-code"></a>執行程式碼

1. 在命令提示字元中，執行下列命令：

   ```bash
   ruby sqltest.rb
   ```

2. 請確認前 20 個資料列已傳回，然後關閉應用程式視窗。


## <a name="next-steps"></a>後續步驟
- [設計您的第一個 Azure SQL Database](sql-database-design-first-database.md)
- [適用於 TinyTDS 的 GitHub 存放庫](https://github.com/rails-sqlserver/tiny_tds)
- [回報或發問有關 TinyTDS 的問題](https://github.com/rails-sqlserver/tiny_tds/issues)
- [Ruby Drivers for SQL Server](https://docs.microsoft.com/sql/connect/ruby/ruby-driver-for-sql-server/)
