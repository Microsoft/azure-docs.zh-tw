---
title: 使用 Ruby 連線至適用於 PostgreSQL 的 Azure 資料庫
description: 本快速入門提供 Ruby 程式碼範例，您可用於從 Azure Database for PostgreSQL 連線及查詢資料。
services: postgresql
author: rachel-msft
ms.author: raagyema
manager: kfile
editor: jasonwhowell
ms.service: postgresql
ms.custom: mvc
ms.devlang: ruby
ms.topic: quickstart
ms.date: 02/28/2018
ms.openlocfilehash: 911dcd49273edb202c64d046424418b7db048291
ms.sourcegitcommit: c765cbd9c379ed00f1e2394374efa8e1915321b9
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 02/28/2018
ms.locfileid: "29692414"
---
# <a name="azure-database-for-postgresql-use-ruby-to-connect-and-query-data"></a>Azure Database for PostgreSQL︰使用 Ruby 連線及查詢資料
本快速入門示範如何使用 [Ruby](https://www.ruby-lang.org) 應用程式來連線到 Azure Database for PostgreSQL。 它會顯示如何使用 SQL 陳述式來查詢、插入、更新和刪除資料庫中的資料。 本文中的步驟假設您已熟悉使用 Ruby 進行開發，但不熟悉適用於 PostgreSQL 的 Azure 資料庫。

## <a name="prerequisites"></a>先決條件
本快速入門使用在以下任一指南中建立的資源作為起點︰
- [建立 DB - 入口網站](quickstart-create-server-database-portal.md)
- [建立 DB - Azure CLI](quickstart-create-server-database-azure-cli.md)

## <a name="install-ruby"></a>安裝 Ruby
在自己的電腦上安裝 Ruby。 

### <a name="windows"></a>Windows
- 下載並安裝最新版的 [Ruby](http://rubyinstaller.org/downloads/)。
- 在 MSI 安裝程式的完成畫面上，核取表示「執行 'ridk install' 以安裝 MSYS2 和開發工具鏈」的方塊。 然後按一下 [完成] 以啟動下一個安裝程式。
- RubyInstaller2 for Windows 安裝程式隨即啟動。 輸入 2 以安裝 MSYS2 存放庫更新。 在完成並返回安裝提示之後，請關閉命令視窗。
- 從 [開始] 功能表啟動新的命令提示 (cmd)。
- 測試 Ruby 安裝 `ruby -v` 以查看安裝的版本。
- 測試 Gem 安裝 `gem -v` 以查看安裝的版本。
- 執行 `gem install pg` 命令以使用 Gem 建置 PostgreSQL 模組。

### <a name="macos"></a>MacOS
- 執行 `brew install ruby` 命令以使用 Homebrew 安裝 Ruby。 如需其他安裝選項，請參閱 Ruby [安裝文件](https://www.ruby-lang.org/en/documentation/installation/#homebrew)
- 測試 Ruby 安裝 `ruby -v` 以查看安裝的版本。
- 測試 Gem 安裝 `gem -v` 以查看安裝的版本。
- 執行 `gem install pg` 命令以使用 Gem 建置 PostgreSQL 模組。

### <a name="linux-ubuntu"></a>Linux (Ubuntu)
- 執行 `sudo apt-get install ruby-full` 命令來安裝 Ruby。 如需其他安裝選項，請參閱 Ruby [安裝文件](https://www.ruby-lang.org/en/documentation/installation/)。
- 測試 Ruby 安裝 `ruby -v` 以查看安裝的版本。
- 執行 `sudo gem update --system` 命令以安裝 Gem 的最新更新。
- 測試 Gem 安裝 `gem -v` 以查看安裝的版本。
- 執行 `sudo apt-get install build-essential` 命令以安裝 gcc、make 和其他建置工具。
- 執行 `sudo apt-get install libpq-dev` 命令以安裝 PostgreSQL 程式庫。
- 執行 `sudo gem install pg` 命令以使用 Gem 建置 Ruby pg 模組。

## <a name="run-ruby-code"></a>執行 Ruby 程式碼 
- 將程式碼儲存到副檔名為 .rb 的文字檔中，並將該檔案儲存到專案資料夾中，例如 `C:\rubypostgres\read.rb` 或 `/home/username/rubypostgres/read.rb`
- 若要執行程式碼，請啟動命令提示字元或 bash shell。 將目錄變更為您的專案資料夾 `cd rubypostgres`，然後輸入 `ruby read.rb` 命令來執行應用程式。

## <a name="get-connection-information"></a>取得連線資訊
取得連線到 Azure Database for PostgreSQL 所需的連線資訊。 您需要完整的伺服器名稱和登入認證。

1. 登入 [Azure 入口網站](https://portal.azure.com/)。
2. 從 Azure 入口網站的左側功能表中，按一下 [所有資源]，然後搜尋您所建立的伺服器 (例如 **mydemoserver**)。
3. 按一下伺服器名稱。
4. 從伺服器的 [概觀] 面板，記下 [伺服器名稱] 和 [伺服器管理員登入名稱]。 如果您忘記密碼，您也可以從此面板重設密碼。
 ![適用於 PostgreSQL 的 Azure 資料庫伺服器名稱](./media/connect-ruby/1-connection-string.png)

## <a name="connect-and-create-a-table"></a>連線及建立資料表
使用下列程式碼搭配 **CREATE TABLE** SQL 陳述式 (後面接著 **INSERT INTO** SQL 陳述式) 來連線和建立資料表，進而將資料列新增至資料表中。

程式碼會使用 [PG::Connection](http://www.rubydoc.info/gems/pg/PG/Connection) 物件搭配建構函式 [new()](http://www.rubydoc.info/gems/pg/PG%2FConnection:initialize)，以連線至 Azure Database for PostgreSQL。 然後它會呼叫 [exec()](http://www.rubydoc.info/gems/pg/PG/Connection#exec-instance_method) 方法來執行 DROP、CREATE TABLE 和 INSERT INTO 命令。 程式碼會使用 [PG::Error](http://www.rubydoc.info/gems/pg/PG/Error) 類別檢查錯誤。 然後它會呼叫 [close()](http://www.rubydoc.info/gems/pg/PG/Connection#lo_close-instance_method) 方法，在終止前關閉連線。

以您自己的值取代 `host`、`database`、`user` 和 `password` 字串。 
```ruby
require 'pg'

begin
    # Initialize connection variables.
    host = String('mydemoserver.postgres.database.azure.com')
    database = String('postgres')
    user = String('mylogin@mydemoserver')
    password = String('<server_admin_password>')

    # Initialize connection object.
    connection = PG::Connection.new(:host => host, :user => user, :dbname => database, :port => '5432', :password => password)
    puts 'Successfully created connection to database'

    # Drop previous table of same name if one exists
    connection.exec('DROP TABLE IF EXISTS inventory;')
    puts 'Finished dropping table (if existed).'

    # Drop previous table of same name if one exists.
    connection.exec('CREATE TABLE inventory (id serial PRIMARY KEY, name VARCHAR(50), quantity INTEGER);')
    puts 'Finished creating table.'

    # Insert some data into table.
    connection.exec("INSERT INTO inventory VALUES(1, 'banana', 150)")
    connection.exec("INSERT INTO inventory VALUES(2, 'orange', 154)")
    connection.exec("INSERT INTO inventory VALUES(3, 'apple', 100)")
    puts 'Inserted 3 rows of data.'

rescue PG::Error => e
    puts e.message 
    
ensure
    connection.close if connection
end
```

## <a name="read-data"></a>讀取資料
使用下列程式碼搭配 **SELECT** SQL 陳述式來連線和讀取資料。 

程式碼會使用 [PG::Connection](http://www.rubydoc.info/gems/pg/PG/Connection) 物件搭配建構函式 [new()](http://www.rubydoc.info/gems/pg/PG%2FConnection:initialize)，以連線至 Azure Database for PostgreSQL。 然後它會呼叫 [exec()](http://www.rubydoc.info/gems/pg/PG/Connection#exec-instance_method) 方法來執行 SELECT 命令，並將結果保留在結果集中。 結果集的集合會使用 `resultSet.each do` 迴圈反覆運算，並將目前的資料列值保留在 `row` 變數中。 程式碼會使用 [PG::Error](http://www.rubydoc.info/gems/pg/PG/Error) 類別檢查錯誤。 然後它會呼叫 [close()](http://www.rubydoc.info/gems/pg/PG/Connection#lo_close-instance_method) 方法，在終止前關閉連線。

以您自己的值取代 `host`、`database`、`user` 和 `password` 字串。 

```ruby
require 'pg'

begin
    # Initialize connection variables.
    host = String('mydemoserver.postgres.database.azure.com')
    database = String('postgres')
    user = String('mylogin@mydemoserver')
    password = String('<server_admin_password>')

    # Initialize connection object.
    connection = PG::Connection.new(:host => host, :user => user, :database => dbname, :port => '5432', :password => password)
    puts 'Successfully created connection to database.'

    resultSet = connection.exec('SELECT * from inventory;')
    resultSet.each do |row|
        puts 'Data row = (%s, %s, %s)' % [row['id'], row['name'], row['quantity']]
    end

rescue PG::Error => e
    puts e.message 
    
ensure
    connection.close if connection
end
```

## <a name="update-data"></a>更新資料
使用下列程式碼搭配 **UPDATE** SQL 陳述式來連線和更新資料。

程式碼會使用 [PG::Connection](http://www.rubydoc.info/gems/pg/PG/Connection) 物件搭配建構函式 [new()](http://www.rubydoc.info/gems/pg/PG%2FConnection:initialize)，以連線至 Azure Database for PostgreSQL。 然後它會呼叫 [exec()](http://www.rubydoc.info/gems/pg/PG/Connection#exec-instance_method) 方法來執行 UPDATE 命令。 程式碼會使用 [PG::Error](http://www.rubydoc.info/gems/pg/PG/Error) 類別檢查錯誤。 然後它會呼叫 [close()](http://www.rubydoc.info/gems/pg/PG/Connection#lo_close-instance_method) 方法，在終止前關閉連線。

以您自己的值取代 `host`、`database`、`user` 和 `password` 字串。 

```ruby
require 'pg'

begin
    # Initialize connection variables.
    host = String('mydemoserver.postgres.database.azure.com')
    database = String('postgres')
    user = String('mylogin@mydemoserver')
    password = String('<server_admin_password>')

    # Initialize connection object.
    connection = PG::Connection.new(:host => host, :user => user, :dbname => database, :port => '5432', :password => password)
    puts 'Successfully created connection to database.'

    # Modify some data in table.
    connection.exec('UPDATE inventory SET quantity = %d WHERE name = %s;' % [200, '\'banana\''])
    puts 'Updated 1 row of data.'

rescue PG::Error => e
    puts e.message 
    
ensure
    connection.close if connection
end
```


## <a name="delete-data"></a>刪除資料
使用下列程式碼搭配 **DELETE** SQL 陳述式來連線和讀取資料。 

程式碼會使用 [PG::Connection](http://www.rubydoc.info/gems/pg/PG/Connection) 物件搭配建構函式 [new()](http://www.rubydoc.info/gems/pg/PG%2FConnection:initialize)，以連線至 Azure Database for PostgreSQL。 然後它會呼叫 [exec()](http://www.rubydoc.info/gems/pg/PG/Connection#exec-instance_method) 方法來執行 UPDATE 命令。 程式碼會使用 [PG::Error](http://www.rubydoc.info/gems/pg/PG/Error) 類別檢查錯誤。 然後它會呼叫 [close()](http://www.rubydoc.info/gems/pg/PG/Connection#lo_close-instance_method) 方法，在終止前關閉連線。

以您自己的值取代 `host`、`database`、`user` 和 `password` 字串。 

```ruby
require 'pg'

begin
    # Initialize connection variables.
    host = String('mydemoserver.postgres.database.azure.com')
    database = String('postgres')
    user = String('mylogin@mydemoserver')
    password = String('<server_admin_password>')

    # Initialize connection object.
    connection = PG::Connection.new(:host => host, :user => user, :dbname => database, :port => '5432', :password => password)
    puts 'Successfully created connection to database.'

    # Modify some data in table.
    connection.exec('DELETE FROM inventory WHERE name = %s;' % ['\'orange\''])
    puts 'Deleted 1 row of data.'

rescue PG::Error => e
    puts e.message 
    
ensure
    connection.close if connection
end
```

## <a name="next-steps"></a>後續步驟
> [!div class="nextstepaction"]
> [使用匯出和匯入來移轉資料庫](./howto-migrate-using-export-and-import.md)
