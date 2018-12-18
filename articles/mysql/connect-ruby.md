---
title: 使用 Ruby 連線到適用於 MySQL 的 Azure 資料庫
description: 本快速入門提供數個 Ruby 程式碼範例，供您用來從 Azure Database for MySQL 連線及查詢資料。
services: mysql
author: jasonwhowell
ms.author: jasonh
manager: kfile
editor: jasonwhowell
ms.service: mysql
ms.custom: mvc
ms.devlang: ruby
ms.topic: quickstart
ms.date: 02/28/2018
ms.openlocfilehash: cbd60be37dc7021ecbb961027dca40d048ed84cb
ms.sourcegitcommit: 1b8665f1fff36a13af0cbc4c399c16f62e9884f3
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/11/2018
ms.locfileid: "35265073"
---
# <a name="azure-database-for-mysql-use-ruby-to-connect-and-query-data"></a>Azure Database for MySQL︰使用 Ruby 來連線及查詢資料
本快速入門示範如何從 Windows、Ubuntu Linux 和 Mac 平台使用 [Ruby](https://www.ruby-lang.org) 應用程式和 [mysql2](https://rubygems.org/gems/mysql2) Gem，連線到 Azure Database for MySQL。 它會顯示如何使用 SQL 陳述式來查詢、插入、更新和刪除資料庫中的資料。 本主題假設您已熟悉使用 Ruby 進行開發，但不熟悉適用於 MySQL 的 Azure 資料庫。

## <a name="prerequisites"></a>先決條件
本快速入門使用在以下任一指南中建立的資源作為起點︰
- [使用 Azure 入口網站建立適用於 MySQL 的 Azure 資料庫伺服器](./quickstart-create-mysql-server-database-using-azure-portal.md)
- [使用 Azure CLI 建立適用於 MySQL 的 Azure 資料庫伺服器](./quickstart-create-mysql-server-database-using-azure-cli.md)

## <a name="install-ruby"></a>安裝 Ruby
在自己的電腦上安裝 Ruby、Gem 和 MySQL2 程式庫。 

### <a name="windows"></a>Windows
1. 下載並安裝 2.3 版的 [Ruby](http://rubyinstaller.org/downloads/)。
2. 從 [開始] 功能表啟動新的命令提示 (cmd)。
3. 將目錄變更為 2.3 版的 Ruby 目錄。 `cd c:\Ruby23-x64\bin`
4. 執行 `ruby -v` 命令以測試 Ruby 安裝，進而查看所安裝的版本。
5. 執行 `gem -v` 命令以測試 Gem 安裝，進而查看所安裝的版本。
6. 執行 `gem install mysql2` 命令以使用 Gem 建置適用於 Ruby 的 Mysql2 模組。

### <a name="macos"></a>MacOS
1. 執行 `brew install ruby` 命令以使用 Homebrew 安裝 Ruby。 如需其他安裝選項，請參閱 Ruby [安裝文件](https://www.ruby-lang.org/en/documentation/installation/#homebrew)。
2. 執行 `ruby -v` 命令以測試 Ruby 安裝，進而查看所安裝的版本。
3. 執行 `gem -v` 命令以測試 Gem 安裝，進而查看所安裝的版本。
4. 執行 `gem install mysql2` 命令以使用 Gem 建置適用於 Ruby 的 Mysql2 模組。

### <a name="linux-ubuntu"></a>Linux (Ubuntu)
1. 執行 `sudo apt-get install ruby-full` 命令來安裝 Ruby。 如需其他安裝選項，請參閱 Ruby [安裝文件](https://www.ruby-lang.org/en/documentation/installation/)。
2. 執行 `ruby -v` 命令以測試 Ruby 安裝，進而查看所安裝的版本。
3. 執行 `sudo gem update --system` 命令以安裝 Gem 的最新更新。
4. 執行 `gem -v` 命令以測試 Gem 安裝，進而查看所安裝的版本。
5. 執行 `sudo apt-get install build-essential` 命令以安裝 gcc、make 和其他建置工具。
6. 執行 `sudo apt-get install libmysqlclient-dev` 命令以安裝 MySQL 用戶端開發人員程式庫。
7. 執行 `sudo gem install mysql2` 命令以使用 Gem 建置適用於 Ruby 的 mysql2 模組。

## <a name="get-connection-information"></a>取得連線資訊
取得連線到 Azure Database for MySQL 所需的連線資訊。 您需要完整的伺服器名稱和登入認證。

1. 登入 [Azure 入口網站](https://portal.azure.com/)。
2. 從 Azure 入口網站的左側功能表中，按一下 [所有資源]，然後搜尋您所建立的伺服器 (例如 **mydemoserver**)。
3. 按一下伺服器名稱。
4. 從伺服器的 [概觀] 面板，記下 [伺服器名稱] 和 [伺服器管理員登入名稱]。 如果您忘記密碼，您也可以從此面板重設密碼。
 ![Azure Database for MySQL 伺服器名稱](./media/connect-ruby/1_server-overview-name-login.png)

## <a name="run-ruby-code"></a>執行 Ruby 程式碼 
1. 將 Ruby 程式碼從下列區段貼到文字檔中，然後使用副檔名 .rb 來將檔案儲存到專案資料夾 (例如 `C:\rubymysql\createtable.rb` 或 `/home/username/rubymysql/createtable.rb`)。
2. 若要執行程式碼，請啟動命令提示字元或 Bash 殼層。 將目錄切換到專案資料夾 `cd rubymysql`
3. 然後，輸入後面接著檔案名稱的 Ruby 命令 (例如 `ruby createtable.rb`) 以執行應用程式。
4. 在 Windows 作業系統上，如果 Ruby 應用程式不在您的 path 環境變數中，則可能需要使用完整路徑來啟動節點應用程式，例如 `"c:\Ruby23-x64\bin\ruby.exe" createtable.rb`

## <a name="connect-and-create-a-table"></a>連線及建立資料表
使用下列程式碼搭配 **CREATE TABLE** SQL 陳述式 (後面接著 **INSERT INTO** SQL 陳述式) 來連線和建立資料表，進而在資料表中新增資料列。

程式碼會使用 [mysql2::client](http://www.rubydoc.info/gems/mysql2/0.4.8) 類別 .new() 來連線到 Azure Database for MySQL。 然後它會呼叫 [query()](http://www.rubydoc.info/gems/mysql2/0.4.8#Usage) 方法數次來執行 DROP、CREATE TABLE 和 INSERT INTO 命令。 然後它會呼叫 [close()](http://www.rubydoc.info/gems/mysql2/0.4.8/Mysql2/Client#close-instance_method) 方法，在終止前關閉連線。

以您自己的值取代 `host`、`database`、`username` 和 `password` 字串。 
```ruby
require 'mysql2'

begin
    # Initialize connection variables.
    host = String('mydemoserver.mysql.database.azure.com')
    database = String('quickstartdb')
    username = String('myadmin@mydemoserver')
    password = String('yourpassword')

    # Initialize connection object.
    client = Mysql2::Client.new(:host => host, :username => username, :database => database, :password => password)
    puts 'Successfully created connection to database.'

    # Drop previous table of same name if one exists
    client.query('DROP TABLE IF EXISTS inventory;')
    puts 'Finished dropping table (if existed).'

    # Drop previous table of same name if one exists.
    client.query('CREATE TABLE inventory (id serial PRIMARY KEY, name VARCHAR(50), quantity INTEGER);')
    puts 'Finished creating table.'

    # Insert some data into table.
    client.query("INSERT INTO inventory VALUES(1, 'banana', 150)")
    client.query("INSERT INTO inventory VALUES(2, 'orange', 154)")
    client.query("INSERT INTO inventory VALUES(3, 'apple', 100)")
    puts 'Inserted 3 rows of data.'

# Error handling
rescue Exception => e
    puts e.message

# Cleanup
ensure
    client.close if client
    puts 'Done.'
end
```

## <a name="read-data"></a>讀取資料
使用下列程式碼搭配 **SELECT** SQL 陳述式來連線和讀取資料。 

程式碼會使用 [mysql2::client](http://www.rubydoc.info/gems/mysql2/0.4.8) 類別 .new() 來連線到適用於 MySQL 的 Azure 資料庫。 然後它會呼叫 [query()](http://www.rubydoc.info/gems/mysql2/0.4.8#Usage) 方法來執行 SELECT 命令。 然後它會呼叫 [close()](http://www.rubydoc.info/gems/mysql2/0.4.8/Mysql2/Client#close-instance_method) 方法，在終止前關閉連線。

以您自己的值取代 `host`、`database`、`username` 和 `password` 字串。 

```ruby
require 'mysql2'

begin
    # Initialize connection variables.
    host = String('mydemoserver.mysql.database.azure.com')
    database = String('quickstartdb')
    username = String('myadmin@mydemoserver')
    password = String('yourpassword')

    # Initialize connection object.
    client = Mysql2::Client.new(:host => host, :username => username, :database => database, :password => password)
    puts 'Successfully created connection to database.'

    # Read data
    resultSet = client.query('SELECT * from inventory;')
    resultSet.each do |row|
        puts 'Data row = (%s, %s, %s)' % [row['id'], row['name'], row['quantity']]
    end
    puts 'Read ' + resultSet.count.to_s + ' row(s).'

# Error handling
rescue Exception => e
    puts e.message

# Cleanup
ensure
    client.close if client
    puts 'Done.'
end
```

## <a name="update-data"></a>更新資料
使用下列程式碼搭配 **UPDATE** SQL 陳述式來連線和更新資料。

程式碼會使用 [mysql2::client](http://www.rubydoc.info/gems/mysql2/0.4.8) 類別 .new() 來連線到 Azure Database for MySQL。 然後它會呼叫 [query()](http://www.rubydoc.info/gems/mysql2/0.4.8#Usage) 方法來執行 UPDATE 命令。 然後它會呼叫 [close()](http://www.rubydoc.info/gems/mysql2/0.4.8/Mysql2/Client#close-instance_method) 方法，在終止前關閉連線。

以您自己的值取代 `host`、`database`、`username` 和 `password` 字串。 

```ruby
require 'mysql2'

begin
    # Initialize connection variables.
    host = String('mydemoserver.mysql.database.azure.com')
    database = String('quickstartdb')
    username = String('myadmin@mydemoserver')
    password = String('yourpassword')

    # Initialize connection object.
    client = Mysql2::Client.new(:host => host, :username => username, :database => database, :password => password)
    puts 'Successfully created connection to database.'

    # Update data
   client.query('UPDATE inventory SET quantity = %d WHERE name = %s;' % [200, '\'banana\''])
   puts 'Updated 1 row of data.'

# Error handling
rescue Exception => e
    puts e.message

# Cleanup
ensure
    client.close if client
    puts 'Done.'
end
```


## <a name="delete-data"></a>刪除資料
使用下列程式碼搭配 **DELETE** SQL 陳述式來連線和讀取資料。 

程式碼會使用 [mysql2::client](http://www.rubydoc.info/gems/mysql2/0.4.8) 類別 .new() 來連線到 Azure Database for MySQL。 然後它會呼叫 [query()](http://www.rubydoc.info/gems/mysql2/0.4.8#Usage) 方法來執行 DELETE 命令。 然後它會呼叫 [close()](http://www.rubydoc.info/gems/mysql2/0.4.8/Mysql2/Client#close-instance_method) 方法，在終止前關閉連線。

以您自己的值取代 `host`、`database`、`username` 和 `password` 字串。 

```ruby
require 'mysql2'

begin
    # Initialize connection variables.
    host = String('mydemoserver.mysql.database.azure.com')
    database = String('quickstartdb')
    username = String('myadmin@mydemoserver')
    password = String('yourpassword')

    # Initialize connection object.
    client = Mysql2::Client.new(:host => host, :username => username, :database => database, :password => password)
    puts 'Successfully created connection to database.'

    # Delete data
    resultSet = client.query('DELETE FROM inventory WHERE name = %s;' % ['\'orange\''])
    puts 'Deleted 1 row.'

# Error handling
rescue Exception => e
    puts e.message

# Cleanup
ensure
    client.close if client
    puts 'Done.'
end
```

## <a name="next-steps"></a>後續步驟
> [!div class="nextstepaction"]
> [使用匯出和匯入來移轉資料庫](./concepts-migrate-import-export.md)
