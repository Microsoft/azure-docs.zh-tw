---
title: 從 Node.js 連線至適用於 PostgreSQL 的 Azure 資料庫
description: 本快速入門提供 Node.js 程式碼範例，供您在從 Azure Database for PostgreSQL 連線及查詢資料時使用。
services: postgresql
author: rachel-msft
ms.author: raagyema
manager: kfile
editor: jasonwhowell
ms.service: postgresql
ms.custom: mvc, devcenter
ms.devlang: nodejs
ms.topic: quickstart
ms.date: 02/28/2018
ms.openlocfilehash: ddccefd4d39de49809e5648454c8d9f50b34a7c5
ms.sourcegitcommit: c765cbd9c379ed00f1e2394374efa8e1915321b9
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 02/28/2018
ms.locfileid: "29691938"
---
# <a name="azure-database-for-postgresql-use-nodejs-to-connect-and-query-data"></a>Azure Database for PostgreSQL︰使用 Node.js 連線及查詢資料
本快速入門示範如何使用 [Node.js](https://nodejs.org/) 應用程式來連線到適用於 PostgreSQL 的 Azure 資料庫。 它會顯示如何使用 SQL 陳述式來查詢、插入、更新和刪除資料庫中的資料。 本文中的步驟假設您已熟悉使用 Node.js 進行開發，但不熟悉適用於 PostgreSQL 的 Azure 資料庫。

## <a name="prerequisites"></a>先決條件
本快速入門使用在以下任一指南中建立的資源作為起點︰
- [建立 DB - 入口網站](quickstart-create-server-database-portal.md)
- [建立 DB - CLI](quickstart-create-server-database-azure-cli.md)

您也需要：
- 安裝 [Node.js](https://nodejs.org)

## <a name="install-pg-client"></a>安裝 pg 用戶端
安裝 [pg](https://www.npmjs.com/package/pg)，也就是 PostgreSQL client for Node.js。

若要這樣做，請從命令列執行適用於 JavaScript 的節點套件管理員 (npm) 以安裝 pg 用戶端。
```bash
npm install pg
```

列出已安裝的套件以確認安裝。
```bash
npm list
```

## <a name="get-connection-information"></a>取得連線資訊
取得連線到 Azure Database for PostgreSQL 所需的連線資訊。 您需要完整的伺服器名稱和登入認證。

1. 登入 [Azure 入口網站](https://portal.azure.com/)。
2. 從 Azure 入口網站的左側功能表中，按一下 [所有資源]，然後搜尋您所建立的伺服器 (例如 **mydemoserver**)。
3. 按一下伺服器名稱。
4. 從伺服器的 [概觀] 面板，記下 [伺服器名稱] 和 [伺服器管理員登入名稱]。 如果您忘記密碼，您也可以從此面板重設密碼。
 ![適用於 PostgreSQL 的 Azure 資料庫伺服器名稱](./media/connect-nodejs/1-connection-string.png)

## <a name="running-the-javascript-code-in-nodejs"></a>在 Node.js 中執行 JavaScript 程式碼
您可藉由輸入 `node` 以從 Bash 殼層、終端機或 Windows 命令提示字元啟動 Node.js，然後複製範例 JavaScript 程式碼並將其貼至提示字元，以互動方式執行。 或者，您可以將 JavaScript 程式碼儲存成文字檔並以檔案名稱作為參數來啟動 `node filename.js`，進而執行它。

## <a name="connect-create-table-and-insert-data"></a>連線、建立資料表及插入資料
使用下列程式碼搭配 **CREATE TABLE** 和 **INSERT INTO** SQL 陳述式來連線和載入資料。
[pg.Client](https://github.com/brianc/node-postgres/wiki/Client) 物件用來與 PostgreSQL 伺服器連接。 [pg.Client.connect()](https://github.com/brianc/node-postgres/wiki/Client#method-connect) 函式用來建立伺服器連線。 [pg.Client.query()](https://github.com/brianc/node-postgres/wiki/Query) 函式用來對 PostgreSQL 資料庫執行 SQL 查詢。 

以建立伺服器和資料庫時所指定的值，取代主機、資料庫名稱、使用者和密碼參數。

```javascript
const pg = require('pg');

const config = {
    host: '<your-db-server-name>.postgres.database.azure.com',
    // Do not hard code your username and password.
    // Consider using Node environment variables.
    user: '<your-db-username>',     
    password: '<your-password>',
    database: '<name-of-database>',
    port: 5432,
    ssl: true
};

const client = new pg.Client(config);

client.connect(err => {
    if (err) throw err;
    else {
        queryDatabase();
    }
});

function queryDatabase() {
    const query = `
        DROP TABLE IF EXISTS inventory;
        CREATE TABLE inventory (id serial PRIMARY KEY, name VARCHAR(50), quantity INTEGER);
        INSERT INTO inventory (name, quantity) VALUES ('banana', 150);
        INSERT INTO inventory (name, quantity) VALUES ('orange', 154);
        INSERT INTO inventory (name, quantity) VALUES ('apple', 100);
    `;

    client
        .query(query)
        .then(() => {
            console.log('Table created successfully!');
            client.end(console.log('Closed client connection'));
        })
        .catch(err => console.log(err))
        .then(() => {
            console.log('Finished execution, exiting now');
            process.exit();
        });
}
```

## <a name="read-data"></a>讀取資料
使用下列程式碼搭配 **SELECT** SQL 陳述式來連線和讀取資料。 [pg.Client](https://github.com/brianc/node-postgres/wiki/Client) 物件用來與 PostgreSQL 伺服器連接。 [pg.Client.connect()](https://github.com/brianc/node-postgres/wiki/Client#method-connect) 函式用來建立伺服器連線。 [pg.Client.query()](https://github.com/brianc/node-postgres/wiki/Query) 函式用來對 PostgreSQL 資料庫執行 SQL 查詢。 

以建立伺服器和資料庫時所指定的值，取代主機、資料庫名稱、使用者和密碼參數。 

```javascript
const pg = require('pg');

const config = {
    host: '<your-db-server-name>.postgres.database.azure.com',
    // Do not hard code your username and password.
    // Consider using Node environment variables.
    user: '<your-db-username>',     
    password: '<your-password>',
    database: '<name-of-database>',
    port: 5432,
    ssl: true
};

const client = new pg.Client(config);

client.connect(err => {
    if (err) throw err;
    else { queryDatabase(); }
});

function queryDatabase() {
  
    console.log(`Running query to PostgreSQL server: ${config.host}`);

    const query = 'SELECT * FROM inventory;';

    client.query(query)
        .then(res => {
            const rows = res.rows;

            rows.map(row => {
                console.log(`Read: ${JSON.stringify(row)}`);
            });

            process.exit();
        })
        .catch(err => {
            console.log(err);
        });
}
```

## <a name="update-data"></a>更新資料
使用下列程式碼搭配 **UPDATE** SQL 陳述式來連線和讀取資料。 [pg.Client](https://github.com/brianc/node-postgres/wiki/Client) 物件用來與 PostgreSQL 伺服器連接。 [pg.Client.connect()](https://github.com/brianc/node-postgres/wiki/Client#method-connect) 函式用來建立伺服器連線。 [pg.Client.query()](https://github.com/brianc/node-postgres/wiki/Query) 函式用來對 PostgreSQL 資料庫執行 SQL 查詢。 

以建立伺服器和資料庫時所指定的值，取代主機、資料庫名稱、使用者和密碼參數。 

```javascript
const pg = require('pg');

const config = {
    host: '<your-db-server-name>.postgres.database.azure.com',
    // Do not hard code your username and password.
    // Consider using Node environment variables.
    user: '<your-db-username>',     
    password: '<your-password>',
    database: '<name-of-database>',
    port: 5432,
    ssl: true
};

const client = new pg.Client(config);

client.connect(err => {
    if (err) throw err;
    else {
        queryDatabase();
    }
});

function queryDatabase() {
    const query = `
        UPDATE inventory 
        SET quantity= 1000 WHERE name='banana';
    `;

    client
        .query(query)
        .then(result => {
            console.log('Update completed');
            console.log(`Rows affected: ${result.rowCount}`);
        })
        .catch(err => {
            console.log(err);
            throw err;
        });
}
```

## <a name="delete-data"></a>刪除資料
使用下列程式碼搭配 **DELETE** SQL 陳述式來連線和讀取資料。 [pg.Client](https://github.com/brianc/node-postgres/wiki/Client) 物件用來與 PostgreSQL 伺服器連接。 [pg.Client.connect()](https://github.com/brianc/node-postgres/wiki/Client#method-connect) 函式用來建立伺服器連線。 [pg.Client.query()](https://github.com/brianc/node-postgres/wiki/Query) 函式用來對 PostgreSQL 資料庫執行 SQL 查詢。 

以建立伺服器和資料庫時所指定的值，取代主機、資料庫名稱、使用者和密碼參數。 

```javascript
const pg = require('pg');

const config = {
    host: '<your-db-server-name>.postgres.database.azure.com',
    // Do not hard code your username and password.
    // Consider using Node environment variables.
    user: '<your-db-username>',     
    password: '<your-password>',
    database: '<name-of-database>',
    port: 5432,
    ssl: true
};

const client = new pg.Client(config);

client.connect(err => {
    if (err) {
        throw err;
    } else {
        queryDatabase();
    }
});

function queryDatabase() {
    const query = `
        DELETE FROM inventory 
        WHERE name = 'apple';
    `;

    client
        .query(query)
        .then(result => {
            console.log('Delete completed');
            console.log(`Rows affected: ${result.rowCount}`);
        })
        .catch(err => {
            console.log(err);
            throw err;
        });
}
```

## <a name="next-steps"></a>後續步驟
> [!div class="nextstepaction"]
> [使用匯出和匯入來移轉資料庫](./howto-migrate-using-export-and-import.md)
