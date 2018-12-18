---
title: 從 Node.js 連線到適用於 MySQL 的 Azure 資料庫
description: 本快速入門提供數個 Node.js 程式碼範例，供您用來在從 Azure Database for MySQL 連線和查詢資料。
services: mysql
author: jasonwhowell
ms.author: jasonh
manager: kfile
editor: jasonwhowell
ms.service: mysql
ms.custom: mvc
ms.devlang: nodejs
ms.topic: quickstart
ms.date: 02/28/2018
ms.openlocfilehash: e0edd31027480df3592b46a4c1246462611da26e
ms.sourcegitcommit: 1b8665f1fff36a13af0cbc4c399c16f62e9884f3
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/11/2018
ms.locfileid: "35265324"
---
# <a name="azure-database-for-mysql-use-nodejs-to-connect-and-query-data"></a>Azure Database for MySQL︰使用 Node.js 連線及查詢資料
本快速入門示範如何從 Windows、Ubuntu Linux 和 Mac 平台使用 [Node.js](https://nodejs.org/) 來連線到 Azure Database for MySQL。 它會顯示如何使用 SQL 陳述式來查詢、插入、更新和刪除資料庫中的資料。 本主題假設您已熟悉使用 Node.js 進行開發，但不熟悉適用於 MySQL 的 Azure 資料庫。

## <a name="prerequisites"></a>先決條件
本快速入門使用在以下任一指南中建立的資源作為起點︰
- [使用 Azure 入口網站建立適用於 MySQL 的 Azure 資料庫伺服器](./quickstart-create-mysql-server-database-using-azure-portal.md)
- [使用 Azure CLI 建立適用於 MySQL 的 Azure 資料庫伺服器](./quickstart-create-mysql-server-database-using-azure-cli.md)

您也需要：
- 安裝 [Node.js](https://nodejs.org) 執行階段。
- 安裝 [mysql2](https://www.npmjs.com/package/mysql2) 套件，以從 Node.js 應用程式連線至 MySQL。 

## <a name="install-nodejs-and-the-mysql-connector"></a>安裝 Node.js 和 MySQL 連接器
根據您的平台，遵循適當小節中的指示來安裝 Node.js。 使用 npm 將 mysql2 套件及其相依性安裝到您的專案資料夾中。

### <a name="windows"></a>**Windows**
1. 造訪 [Node.js 下載頁面](https://nodejs.org/en/download/) \(英文\)，然後選取您需要的 Windows 安裝程式選項。
2. 建立本機專案資料夾，例如 `nodejsmysql`。 
3. 啟動命令提示字元，然後將目錄變更為專案資料夾，例如 `cd c:\nodejsmysql\`
4. 執行 NPM 工具將 mysql2 程式庫安裝至專案資料夾中。

   ```cmd
   cd c:\nodejsmysql\
   "C:\Program Files\nodejs\npm" install mysql2
   "C:\Program Files\nodejs\npm" list
   ```

5. 檢查 `mysql2@1.3.5` 的 `npm list` 輸出文字，以確認安裝。

### <a name="linux-ubuntu"></a>**Linux (Ubuntu)**
1. 執行下列命令，以安裝 **Node.js** 和適用於 Node.js 的 **npm** 套件管理員。

   ```bash
   sudo apt-get install -y nodejs npm
   ```

2. 執行下列命令來建立專案資料夾 `mysqlnodejs`，然後將 mysql2 封裝安裝到該資料夾。

   ```bash
   mkdir nodejsmysql
   cd nodejsmysql
   npm install --save mysql2
   npm list
   ```
3. 檢查 `mysql2@1.3.5` 的 npm 清單輸出文字，以確認安裝。

### <a name="mac-os"></a>**Mac OS**
1. 輸入下列命令以安裝 **brew**、適用於 Mac OS X 的簡易使用封裝管理員和 **Node.js**。

   ```bash
   ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
   brew install node
   ```
2. 執行下列命令來建立專案資料夾 `mysqlnodejs`，然後將 mysql2 封裝安裝到該資料夾。

   ```bash
   mkdir nodejsmysql
   cd nodejsmysql
   npm install --save mysql2
   npm list
   ```

3. 檢查 `mysql2@1.3.6` 的 `npm list` 輸出文字，以確認安裝。 版本號碼可能會隨著新的修補檔案發行而有所不同。

## <a name="get-connection-information"></a>取得連線資訊
取得連線到 Azure Database for MySQL 所需的連線資訊。 您需要完整的伺服器名稱和登入認證。

1. 登入 [Azure 入口網站](https://portal.azure.com/)。
2. 從 Azure 入口網站的左側功能表中，按一下 [所有資源]，然後搜尋您所建立的伺服器 (例如 **mydemoserver**)。
3. 按一下伺服器名稱。
4. 從伺服器的 [概觀] 面板，記下 [伺服器名稱] 和 [伺服器管理員登入名稱]。 如果您忘記密碼，您也可以從此面板重設密碼。
 ![Azure Database for MySQL 伺服器名稱](./media/connect-nodejs/1_server-overview-name-login.png)

## <a name="running-the-javascript-code-in-nodejs"></a>在 Node.js 中執行 JavaScript 程式碼
1. 將 JavaScript 程式碼貼入文字檔，然後使用副檔名 .js 來將它儲存到專案資料夾 (例如 C:\nodejsmysql\createtable.js 或 /home/username/nodejsmysql/createtable.js)。
2. 啟動命令提示字元或 Bash 殼層，然後將目錄變更為您的專案資料夾 `cd nodejsmysql`。
3. 若要執行應用程式，請鍵入後接檔案名稱的節點命令，例如 `node createtable.js`。
4. 在 Windows 上，如果節點應用程式不在您的環境變數路徑中，則可能需要使用完整路徑來啟動節點應用程式，例如 `"C:\Program Files\nodejs\node.exe" createtable.js`

## <a name="connect-create-table-and-insert-data"></a>連線、建立資料表及插入資料
使用下列程式碼搭配 **CREATE TABLE** 和 **INSERT INTO** SQL 陳述式來連線和載入資料。

[mysql.createConnection()](https://github.com/mysqljs/mysql#establishing-connections) 方法用來與 MySQL 伺服器互動。 [connect()](https://github.com/mysqljs/mysql#establishing-connections) 函式用來建立伺服器連線。 [query()](https://github.com/mysqljs/mysql#performing-queries) 函式用來對 MySQL 資料庫執行 SQL 查詢。 

將 `host`、`user`、`password` 和 `database` 參數取代為您在建立伺服器和資料庫時所指定的值。

```javascript
const mysql = require('mysql2');

var config =
{
    host: 'mydemoserver.mysql.database.azure.com',
    user: 'myadmin@mydemoserver',
    password: 'your_password',
    database: 'quickstartdb',
    port: 3306,
    ssl: true
};

const conn = new mysql.createConnection(config);

conn.connect(
    function (err) { 
    if (err) { 
        console.log("!!! Cannot connect !!! Error:");
        throw err;
    }
    else
    {
       console.log("Connection established.");
           queryDatabase();
    }   
});

function queryDatabase(){
       conn.query('DROP TABLE IF EXISTS inventory;', function (err, results, fields) { 
            if (err) throw err; 
            console.log('Dropped inventory table if existed.');
        })
       conn.query('CREATE TABLE inventory (id serial PRIMARY KEY, name VARCHAR(50), quantity INTEGER);', 
            function (err, results, fields) {
                if (err) throw err;
            console.log('Created inventory table.');
        })
       conn.query('INSERT INTO inventory (name, quantity) VALUES (?, ?);', ['banana', 150], 
            function (err, results, fields) {
                if (err) throw err;
            else console.log('Inserted ' + results.affectedRows + ' row(s).');
        })
       conn.query('INSERT INTO inventory (name, quantity) VALUES (?, ?);', ['orange', 154], 
            function (err, results, fields) {
                if (err) throw err;
            console.log('Inserted ' + results.affectedRows + ' row(s).');
        })
       conn.query('INSERT INTO inventory (name, quantity) VALUES (?, ?);', ['apple', 100], 
        function (err, results, fields) {
                if (err) throw err;
            console.log('Inserted ' + results.affectedRows + ' row(s).');
        })
       conn.end(function (err) { 
        if (err) throw err;
        else  console.log('Done.') 
        });
};
```

## <a name="read-data"></a>讀取資料
使用下列程式碼搭配 **SELECT** SQL 陳述式來連線和讀取資料。 

[mysql.createConnection()](https://github.com/mysqljs/mysql#establishing-connections) 方法用來與 MySQL 伺服器互動。 [connect()](https://github.com/mysqljs/mysql#establishing-connections) 方法用來建立伺服器連線。 [query()](https://github.com/mysqljs/mysql#performing-queries) 方法用來執行對 MySQL 資料庫的 SQL 查詢。 結果陣列用來保留查詢的結果。

將 `host`、`user`、`password` 和 `database` 參數取代為您在建立伺服器和資料庫時所指定的值。

```javascript
const mysql = require('mysql2');

var config =
{
    host: 'mydemoserver.mysql.database.azure.com',
    user: 'myadmin@mydemoserver',
    password: 'your_password',
    database: 'quickstartdb',
    port: 3306,
    ssl: true
};

const conn = new mysql.createConnection(config);

conn.connect(
    function (err) { 
        if (err) { 
            console.log("!!! Cannot connect !!! Error:");
            throw err;
        }
        else {
            console.log("Connection established.");
            readData();
        }   
    });

function readData(){
        conn.query('SELECT * FROM inventory', 
            function (err, results, fields) {
                if (err) throw err;
                else console.log('Selected ' + results.length + ' row(s).');
                for (i = 0; i < results.length; i++) {
                    console.log('Row: ' + JSON.stringify(results[i]));
                }
                console.log('Done.');
            })
       conn.end(
           function (err) { 
                if (err) throw err;
                else  console.log('Closing connection.') 
        });
};
```

## <a name="update-data"></a>更新資料
使用下列程式碼搭配 **UPDATE** SQL 陳述式來連線和讀取資料。 

[mysql.createConnection()](https://github.com/mysqljs/mysql#establishing-connections) 方法用來與 MySQL 伺服器互動。 [connect()](https://github.com/mysqljs/mysql#establishing-connections) 方法用來建立伺服器連線。 [query()](https://github.com/mysqljs/mysql#performing-queries) 方法用來執行對 MySQL 資料庫的 SQL 查詢。 

將 `host`、`user`、`password` 和 `database` 參數取代為您在建立伺服器和資料庫時所指定的值。

```javascript
const mysql = require('mysql2');

var config =
{
    host: 'mydemoserver.mysql.database.azure.com',
    user: 'myadmin@mydemoserver',
    password: 'your_password',
    database: 'quickstartdb',
    port: 3306,
    ssl: true
};

const conn = new mysql.createConnection(config);

conn.connect(
    function (err) { 
        if (err) { 
            console.log("!!! Cannot connect !!! Error:");
            throw err;
        }
        else {
            console.log("Connection established.");
            updateData();
        }   
    });

function updateData(){
       conn.query('UPDATE inventory SET quantity = ? WHERE name = ?', [200, 'banana'], 
            function (err, results, fields) {
                if (err) throw err;
                else console.log('Updated ' + results.affectedRows + ' row(s).');
        })
       conn.end(
           function (err) { 
                if (err) throw err;
                else  console.log('Done.') 
        });
};
```

## <a name="delete-data"></a>刪除資料
使用下列程式碼搭配 **DELETE** SQL 陳述式來連線和讀取資料。 

[mysql.createConnection()](https://github.com/mysqljs/mysql#establishing-connections) 方法用來與 MySQL 伺服器互動。 [connect()](https://github.com/mysqljs/mysql#establishing-connections) 方法用來建立伺服器連線。 [query()](https://github.com/mysqljs/mysql#performing-queries) 方法用來執行對 MySQL 資料庫的 SQL 查詢。 

將 `host`、`user`、`password` 和 `database` 參數取代為您在建立伺服器和資料庫時所指定的值。

```javascript
const mysql = require('mysql2');

var config =
{
    host: 'mydemoserver.mysql.database.azure.com',
    user: 'myadmin@mydemoserver',
    password: 'your_password',
    database: 'quickstartdb',
    port: 3306,
    ssl: true
};

const conn = new mysql.createConnection(config);

conn.connect(
    function (err) { 
        if (err) { 
            console.log("!!! Cannot connect !!! Error:");
            throw err;
        }
        else {
            console.log("Connection established.");
            deleteData();
        }   
    });

function deleteData(){
       conn.query('DELETE FROM inventory WHERE name = ?', ['orange'], 
            function (err, results, fields) {
                if (err) throw err;
                else console.log('Deleted ' + results.affectedRows + ' row(s).');
        })
       conn.end(
           function (err) { 
                if (err) throw err;
                else  console.log('Done.') 
        });
};
```

## <a name="next-steps"></a>後續步驟
> [!div class="nextstepaction"]
> [使用匯出和匯入來移轉資料庫](./concepts-migrate-import-export.md)
