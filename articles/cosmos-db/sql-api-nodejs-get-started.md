---
title: 適用於 Azure Cosmos DB 的 SQL API 之 Node.js 教學課程 | Microsoft Docs
description: 示範如何使用 SQL API 連線及查詢 Azure Cosmos DB 的 Node.js 教學課程
services: cosmos-db
author: deborahc
editor: monicar
ms.service: cosmos-db
ms.component: cosmosdb-sql
ms.devlang: nodejs
ms.topic: tutorial
ms.date: 09/24/2018
ms.author: dech
ms.openlocfilehash: affa302c7fd2a0cb05a6d599050e72c75ef74479
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46969050"
---
# <a name="nodejs-tutorial-create-a-nodejs-console-application-with-javascript-sdk-to-manage-azure-cosmos-db-sql-api-data"></a>Node.js 教學課程：使用 JavaScript SDK 建立 Node.js 主控台應用程式，以管理 Azure Cosmos DB SQL API 資料

> [!div class="op_single_selector"]
> * [.NET](sql-api-get-started.md)
> * [.NET Core](sql-api-dotnetcore-get-started.md)
> * [Java](sql-api-java-get-started.md)
> * [非同步 Java](sql-api-async-java-get-started.md)
> * [Node.js](sql-api-nodejs-get-started.md)
> 


歡迎使用 Azure Cosmos DB JavaScript SDK 的 Node.js 教學課程！ 完成本教學課程之後，您將會有一個主控台應用程式，可用來建立和查詢 Azure Cosmos DB 資源。

本文將討論：

* 建立及連線至 Azure Cosmos DB 帳戶
* 設定您的應用程式
* 建立資料庫
* 建立容器
* 將 JSON 項目新增至容器
* 查詢容器
* 取代項目
* 刪除項目
* 刪除資料庫

沒有時間嗎？ 別擔心！ 您可以在 [GitHub](https://github.com/Azure-Samples/azure-cosmos-db-sql-api-nodejs-getting-started )上找到完整的方案。 請參閱 [取得完整的解決方案](#GetSolution) ，以取得簡要指示。

完成 Node.js 教學課程之後，請使用此頁面頂端和底部的投票按鈕來提供意見。 如果想要我們直接與您連絡，歡迎在留言中留下電子郵件地址。

讓我們開始吧！

## <a name="prerequisites-for-the-nodejs-tutorial"></a>Node.js 教學課程的必要條件

請確定您具有下列項目：

* 使用中的 Azure 帳戶。 如果您沒有帳戶，您可以註冊 [免費 Azure 試用](https://azure.microsoft.com/pricing/free-trial/)。 

  [!INCLUDE [cosmos-db-emulator-docdb-api](../../includes/cosmos-db-emulator-docdb-api.md)]

* [Node.js](https://nodejs.org/) v6.0.0 版或更新版本。

## <a name="step-1-create-an-azure-cosmos-db-account"></a>步驟 1：建立 Azure Cosmos DB 帳戶

讓我們來建立 Azure Cosmos DB 帳戶。 如果您已經擁有想要使用的帳戶，就可以直接跳到[設定您的 Node.js 應用程式](#SetupNode)。 如果您是使用「Azure Cosmos DB 模擬器」，請依照 [Azure Cosmos DB 模擬器](local-emulator.md)的步驟來設定模擬器，然後直接跳到[設定您的 Node.js 應用程式](#SetupNode)。 

[!INCLUDE [cosmos-db-create-dbaccount](../../includes/cosmos-db-create-dbaccount.md)]

## <a id="SetupNode"></a>步驟 2：設定您的 Node.js 應用程式

1. 開啟您偏好的終端機。
2. 找出您想儲存 Node.js 應用程式的資料夾或目錄位置。
3. 使用下列命令，建立兩個空白的 JavaScript 檔案：
   * Windows:
     * ```fsutil file createnew app.js 0```
     * ```fsutil file createnew config.js 0```
   * Linux/OS X：
     * ```touch app.js```
     * ```touch config.js```
4. 透過 npm 安裝 @azure/cosmos 模組。 使用下列命令：
   * ```npm install @azure/cosmos --save```

太棒了！ 現在已完成安裝程式，讓我們開始撰寫一些程式碼。

## <a id="Config"></a>步驟 3：設定您的應用程式設定

在您慣用的文字編輯器中開啟 ```config.js```。

然後，複製並貼上以下的程式碼片段，以及將屬性 ```config.endpoint``` 和 ```config.primaryKey``` 設定為您的 Azure Cosmos DB 端點 URI 和主要金鑰。 您可以在 [Azure 入口網站](https://portal.azure.com)找到這些設定。

![node.js 教學課程 - 顯示 Azure Cosmos DB 帳戶的 Azure 入口網站螢幕擷取畫面，內含反白顯示的 [主動式] 集線器、[Azure Cosmos DB 帳戶] 刀鋒視窗上反白顯示的 [金鑰] 按鈕、[金鑰] 刀鋒視窗上反白顯示的 [URI]、[主要金鑰] 和 [次要金鑰] 值 - 節點資料庫][keys]

```nodejs
// ADD THIS PART TO YOUR CODE
var config = {}

config.endpoint = "~your Azure Cosmos DB endpoint uri here~";
config.primaryKey = "~your primary key here~";
``` 

在您設定 ```config.endpoint``` 和 ```config.primaryKey``` 屬性的位置下面，複製 ```database```、```container``` 和 ```items``` 資料並貼到您的 ```config``` 物件。 如果您已經有想要儲存於資料庫中的資料，您可以使用 Azure Cosmos DB 的 [資料移轉工具](import-data.md)，而無須在此處定義資料。

```nodejs
var config = {}

config.endpoint = "~your Azure Cosmos DB account endpoint uri here~";
config.primaryKey = "~your primary key here~";

config.database = {
    "id": "FamilyDatabase"
};

config.container = {
    "id": "FamilyContainer"
};

config.items = {
    "Andersen": {
        "id": "Anderson.1",
        "lastName": "Andersen",
        "parents": [{
            "firstName": "Thomas"
        }, {
                "firstName": "Mary Kay"
            }],
        "children": [{
            "firstName": "Henriette Thaulow",
            "gender": "female",
            "grade": 5,
            "pets": [{
                "givenName": "Fluffy"
            }]
        }],
        "address": {
            "state": "WA",
            "county": "King",
            "city": "Seattle"
        }
    },
    "Wakefield": {
        "id": "Wakefield.7",
        "parents": [{
            "familyName": "Wakefield",
            "firstName": "Robin"
        }, {
                "familyName": "Miller",
                "firstName": "Ben"
            }],
        "children": [{
            "familyName": "Merriam",
            "firstName": "Jesse",
            "gender": "female",
            "grade": 8,
            "pets": [{
                "givenName": "Goofy"
            }, {
                    "givenName": "Shadow"
                }]
        }, {
                "familyName": "Miller",
                "firstName": "Lisa",
                "gender": "female",
                "grade": 1
            }],
        "address": {
            "state": "NY",
            "county": "Manhattan",
            "city": "NY"
        },
        "isRegistered": false
    }
};
```
請注意，如果您熟悉舊版 JavaScript SDK，您可能已習慣看到「集合」和「文件」等字詞。 Azure Cosmos DB 支援[多個 API 模型](https://docs.microsoft.com/azure/cosmos-db/introduction#key-capabilities)，因此 2.0+ 版的 JavaScript SDK 會使用「容器」和「項目」這些泛用字詞。 容器可以是集合、圖形或資料表。 項目可以是文件、邊緣/頂點或資料列，並且是容器內的內容。 

最後，匯出您的 ```config``` 物件，如此就可以在 ```app.js``` 檔案中參考它。
```nodejs
        },
        "isRegistered": false
    }
};

// ADD THIS PART TO YOUR CODE
module.exports = config;
```
## <a id="Connect"></a>步驟 4：連線至 Azure Cosmos DB 帳戶

在文字編輯器中開啟您的空白 ```app.js``` 檔案。 複製並貼上以下的程式碼，以匯入 ```@azure/cosmos``` 模組和新建的 ```config``` 模組。

```nodejs
// ADD THIS PART TO YOUR CODE
const CosmosClient = require('@azure/cosmos').CosmosClient;

const config = require('./config');
const url = require('url');
```

複製並貼上以下的程式碼，以使用先前儲存的 ```config.endpoint``` 和 ```config.primaryKey``` 來建立新的 CosmosClient。

```nodejs
const url = require('url');

// ADD THIS PART TO YOUR CODE
const endpoint = config.endpoint;
const masterKey = config.primaryKey;

const client = new CosmosClient({ endpoint: endpoint, auth: { masterKey: masterKey } });
```

您現在已有可將 Azure Cosmos DB 用戶端初始化的程式碼，接下來我們將討論如何使用 Azure Cosmos DB 資源。

## <a name="step-5-create-a-database"></a>步驟 5：建立資料庫

複製並貼上以下的程式碼，以設定 [找不到] 的 HTTP 狀態、資料庫識別碼和容器識別碼。這些識別碼可讓 Azure Cosmos DB 用戶端尋找正確的資料庫和容器。

```nodejs
const client = new CosmosClient({ endpoint: endpoint, auth: { masterKey: masterKey } });

// ADD THIS PART TO YOUR CODE
const HttpStatusCodes = { NOTFOUND: 404 };

const databaseId = config.database.id;
const containerId = config.container.id;
```

[資料庫](sql-api-resources.md#databases)可使用 **Databases** 類別的 [createIfNotExists](/javascript/api/%40azure/cosmos/databases) 或 [create](/javascript/api/%40azure/cosmos/databases) 函式來建立。 資料庫是分割於多個容器之項目的邏輯容器。 

複製 **createDatabase** 和 **readDatabase** 函式，並將其貼到 app.js 檔案中的 ```databaseId``` 和 ```containerId``` 定義下方。 **createDatabase** 函式會以識別碼 ```FamilyDatabase``` 建立新的資料庫；如果該識別碼尚不存在，則會從 ```config``` 物件指定。 **readDatabase** 函式會讀取資料庫的定義，以確定該資料庫確實存在。

```nodejs
/**
 * Create the database if it does not exist
 */
async function createDatabase() {
    const { database } = await client.databases.createIfNotExists({ id: databaseId });
    console.log(`Created database:\n${database.id}\n`);
}

/**
 * Read the database definition
 */
async function readDatabase() {
    const { body: databaseDefinition } = await client.database(databaseId).read();
    console.log(`Reading database:\n${databaseDefinition.id}\n`);
}
```

複製下方的程式碼，並將其貼到您設定 **createDatabase** 和 **readDatabase** 函式的位置，以新增會列印結束訊息的協助程式函式 **exit**。 

```nodejs
// ADD THIS PART TO YOUR CODE
function exit(message) {
    console.log(message);
    console.log('Press any key to exit');
    process.stdin.setRawMode(true);
    process.stdin.resume();
    process.stdin.on('data', process.exit.bind(process, 0));
};
```
複製下方的程式碼，並將其貼到您設定 **exit** 函式的位置，以呼叫 **createDatabase** 和 **readDatabase** 函式。

```nodejs
createDatabase()
    .then(() => readDatabase())
    .then(() => { exit(`Completed successfully`); })
    .catch((error) => { exit(`Completed with error \${JSON.stringify(error)}`) });
```

此時，您在 ```app.js``` 中的程式碼應會顯示如下：

```nodejs
const CosmosClient = require('@azure/cosmos').CosmosClient;

const config = require('./config');
const url = require('url');

const endpoint = config.endpoint;
const masterKey = config.primaryKey;

const client = new CosmosClient({ endpoint: endpoint, auth: { masterKey: masterKey } });

const HttpStatusCodes = { NOTFOUND: 404 };

const databaseId = config.database.id;
const containerId = config.container.id;


/**
 * Create the database if it does not exist
 */
async function createDatabase() {
    const { database } = await client.databases.createIfNotExists({ id: databaseId });
    console.log(`Created database:\n${database.id}\n`);
}

/**
 * Read the database definition
 */
async function readDatabase() {
    const { body: databaseDefinition } = await client.database(databaseId).read();
    console.log(`Reading database:\n${databaseDefinition.id}\n`);
}

/**
 * Exit the app with a prompt
 * @param {message} message - The message to display
 */
function exit(message) {
    console.log(message);
    console.log('Press any key to exit');
    process.stdin.setRawMode(true);
    process.stdin.resume();
    process.stdin.on('data', process.exit.bind(process, 0));
}

createDatabase()
    .then(() => readDatabase())
    .then(() => { exit(`Completed successfully`); })
    .catch((error) => { exit(`Completed with error ${JSON.stringify(error)}`) });
```

在終端機中，找出您的 ```app.js``` 檔案並執行命令： 
```bash 
node app.js
```

恭喜！ 您已成功建立 Azure Cosmos DB 資料庫。

## <a id="CreateContainer"></a>步驟 6：建立容器

> [!WARNING]
> 呼叫 **createContainer** 函式會建立具有定價含意的新容器。 如需詳細資訊，請造訪[定價頁面](https://azure.microsoft.com/pricing/details/cosmos-db/)。

容器可使用 **Containers** 類別的 [createIfNotExists](/javascript/api/%40azure/cosmos/containers) 或 [create](/javascript/api/%40azure/cosmos/containers) 函式來建立。 

容器中包含項目 (在 SQL API 的案例中為 JSON 文件) 和相關聯的 JavaScript 應用程式邏輯。

將 **createContainer** 和 **readContainer** 函式複製並貼到 app.js 檔案中的 **readDatabase** 函式下方。 **createContainer** 函式會以 ```containerId``` 建立新的容器；如果該識別碼尚不存在，則會從 ```config``` 物件指定。 **readContainer** 函式會讀取容器定義以確認容器是否存在。

```nodejs
/**
 * Create the container if it does not exist
 */
async function createContainer() {
    const { container } = await client.database(databaseId).containers.createIfNotExists({ id: containerId });
    console.log(`Created container:\n${config.container.id}\n`);
}

/**
 * Read the container definition
 */
async function readContainer() {
    const { body: containerDefinition } = await client.database(databaseId).container(containerId).read();
    console.log(`Reading container:\n${containerDefinition.id}\n`);
}
```

複製下列程式碼，並將其貼到 **readDatabase** 的呼叫下方，以執行 **createContainer** 和 **readContainer** 函式。
```nodejs
createDatabase()
    .then(() => readDatabase())

    // ADD THIS PART TO YOUR CODE
    .then(() => createContainer())
    .then(() => readContainer())
    // ENDS HERE

    .then(() => { exit(`Completed successfully`); })
    .catch((error) => { exit(`Completed with error ${JSON.stringify(error)}`) });
```

此時，您在 ```app.js``` 中的程式碼應會顯示如下：

```nodejs
const CosmosClient = require('@azure/cosmos').CosmosClient;

const config = require('./config');
const url = require('url');

const endpoint = config.endpoint;
const masterKey = config.primaryKey;

const client = new CosmosClient({ endpoint: endpoint, auth: { masterKey: masterKey } });

const HttpStatusCodes = { NOTFOUND: 404 };

const databaseId = config.database.id;
const containerId = config.container.id;

/**
 * Create the database if it does not exist
 */
async function createDatabase() {
    const { database } = await client.databases.createIfNotExists({ id: databaseId });
    console.log(`Created database:\n${database.id}\n`);
}

/**
 * Read the database definition
 */
async function readDatabase() {
    const { body: databaseDefinition } = await client.database(databaseId).read();
    console.log(`Reading database:\n${databaseDefinition.id}\n`);
}

/**
 * Create the container if it does not exist
 */
async function createContainer() {
    const { container } = await client.database(databaseId).containers.createIfNotExists({ id: containerId });
    console.log(`Created container:\n${config.container.id}\n`);
}

/**
 * Read the container definition
 */
async function readContainer() {
    const { body: containerDefinition } = await client.database(databaseId).container(containerId).read();
    console.log(`Reading container:\n${containerDefinition.id}\n`);
}

/**
 * Exit the app with a prompt
 * @param {message} message - The message to display
 */
function exit(message) {
    console.log(message);
    console.log('Press any key to exit');
    process.stdin.setRawMode(true);
    process.stdin.resume();
    process.stdin.on('data', process.exit.bind(process, 0));
}

createDatabase()
    .then(() => readDatabase())
    .then(() => createContainer())
    .then(() => readContainer())
    .then(() => { exit(`Completed successfully`); })
    .catch((error) => { exit(`Completed with error ${JSON.stringify(error)}`) });
```

在終端機中，找出您的 ```app.js``` 檔案並執行命令： 

```bash 
node app.js
```

恭喜！ 您已成功建立 Azure Cosmos DB 容器。

## <a id="CreateItem"></a>步驟 7：建立項目

項目可以使用 **Items** 類別的 [create](/javascript/api/%40azure/cosmos/items) 函式來建立。 使用 SQL API 時，預期的項目會是文件，這是使用者定義的 (任意) JSON 內容。 您現在可以將項目插入 Azure Cosmos DB 中。

將 **createFamilyItem** 函式複製並貼到 **readContainer** 函式下方。 **createFamilyItem** 函式會建立項目，其中包含儲存在 ```config``` 物件中的 JSON 資料。 在建立項目之前，我們會先確認沒有使用相同識別碼的項目存在。

```nodejs
/**
 * Create family item if it does not exist
 */
async function createFamilyItem(itemBody) {
    try {
        // read the item to see if it exists
        const { item } = await client.database(databaseId).container(containerId).item(itemBody.id).read();
        console.log(`Item with family id ${itemBody.id} already exists\n`);
    }
    catch (error) {
        // create the family item if it does not exist
        if (error.code === HttpStatusCodes.NOTFOUND) {
            const { item } = await client.database(databaseId).container(containerId).items.create(itemBody);
            console.log(`Created family item with id:\n${itemBody.id}\n`);
        } else {
            throw error;
        }
    }
};
```

將下列程式碼複製並貼到 **readContainer** 的呼叫下方，以執行 **createFamilyItem** 函式。
```nodejs
createDatabase()
    .then(() => readDatabase())
    .then(() => createContainer())
    .then(() => readContainer())

    // ADD THIS PART TO YOUR CODE
    .then(() => createFamilyItem(config.items.Andersen))
    .then(() => createFamilyItem(config.items.Wakefield))
    // ENDS HERE

    .then(() => { exit(`Completed successfully`); })
    .catch((error) => { exit(`Completed with error ${JSON.stringify(error)}`) });
```

在終端機中，找出您的 ```app.js``` 檔案並執行命令： 

```bash 
node app.js
```

恭喜！ 您已成功建立 Azure Cosmos DB 項目。


## <a id="Query"></a>步驟 8︰查詢 Azure Cosmos DB 資源
Azure Cosmos DB 支援針對儲存於每個容器中的 JSON 文件進行[豐富查詢](sql-api-sql-query.md)。 下列範例程式碼說明您可以對容器中的文件執行的查詢。

將 **queryContainer** 函式複製並貼到 app.js 檔案中的 **createFamilyItem** 函式下方。 Azure Cosmos DB 支援類 SQL 查詢，如下所示。 如需建立複雜查詢的詳細資訊，請參閱[查詢遊樂場](https://www.documentdb.com/sql/demo)和[查詢文件](sql-api-sql-query.md)。

```nodejs
/**
 * Query the container using SQL
 */
async function queryContainer() {
    console.log(`Querying container:\n${config.container.id}`);

    // query to return all children in a family
    const querySpec = {
        query: "SELECT VALUE r.children FROM root r WHERE r.lastName = @lastName",
        parameters: [
            {
                name: "@lastName",
                value: "Andersen"
            }
        ]
    };

    const { result: results } = await client.database(databaseId).container(containerId).items.query(querySpec).toArray();
    for (var queryResult of results) {
        let resultString = JSON.stringify(queryResult);
        console.log(`\tQuery returned ${resultString}\n`);
    }
};
```

將下列程式碼複製並貼到 **createFamilyItem** 的呼叫下方，以執行 **queryContainer** 函式。

```nodejs
createDatabase()
    .then(() => readDatabase())
    .then(() => createContainer())
    .then(() => readContainer())
    .then(() => createFamilyItem(config.items.Andersen))
    .then(() => createFamilyItem(config.items.Wakefield))

    // ADD THIS PART TO YOUR CODE
    .then(() => queryContainer())
    // ENDS HERE

    .then(() => { exit(`Completed successfully`); })
    .catch((error) => { exit(`Completed with error ${JSON.stringify(error)}`) });
```

在終端機中，找出您的 ```app.js``` 檔案並執行命令：

```bash 
node app.js
```

恭喜！ 您已成功查詢 Azure Cosmos DB 項目。

## <a id="ReplaceItem"></a>步驟 9：取代項目
Azure Cosmos DB 支援取代項目的內容。

將 **replaceFamilyItem** 函式複製並貼到 app.js 檔案中的 **queryContainer** 函式下方。 請注意，我們已將子系的屬性「等級」從先前的值 5 變更為 6。

```nodejs
// ADD THIS PART TO YOUR CODE
/**
 * Replace the item by ID.
 */
async function replaceFamilyItem(itemBody) {
    console.log(`Replacing item:\n${itemBody.id}\n`);
    // Change property 'grade'
    itemBody.children[0].grade = 6;
    const { item } = await client.database(databaseId).container(containerId).item(itemBody.id).replace(itemBody);
};
```
將下列程式碼複製並貼到 **queryContainer** 的呼叫下方，以執行 **replaceFamilyItem** 函式。 此外，再次將此程式碼加入至 **queryContainer** 呼叫，以確認已成功變更項目。

```nodejs
createDatabase()
    .then(() => readDatabase())
    .then(() => createContainer())
    .then(() => readContainer())
    .then(() => createFamilyItem(config.items.Andersen))
    .then(() => createFamilyItem(config.items.Wakefield))
    .then(() => queryContainer())

    // ADD THIS PART TO YOUR CODE
    .then(() => replaceFamilyItem(config.items.Andersen))
    .then(() => queryContainer())
    // ENDS HERE

    .then(() => { exit(`Completed successfully`); })
    .catch((error) => { exit(`Completed with error ${JSON.stringify(error)}`) });
```
在終端機中，找出您的 ```app.js``` 檔案並執行命令：

```bash 
node app.js
```

恭喜！ 您已成功取代 Azure Cosmos DB 項目。

## <a id="DeleteItem"></a>步驟 10：刪除項目

Azure Cosmos DB 支援刪除 JSON 項目。

將 **deleteFamilyItem** 函式複製並貼到 **replaceFamilyItem** 函式下方。

```nodejs
/**
 * Delete the item by ID.
 */
async function deleteFamilyItem(itemBody) {
    await client.database(databaseId).container(containerId).item(itemBody.id).delete(itemBody);
    console.log(`Deleted item:\n${itemBody.id}\n`);
};
```

將下列程式碼複製並貼到對第二個 **queryContainer** 的呼叫下方，以執行 **deleteFamilyItem** 函式。

```nodejs
createDatabase()
    .then(() => readDatabase())
    .then(() => createContainer())
    .then(() => readContainer())
    .then(() => createFamilyItem(config.items.Andersen))
    .then(() => createFamilyItem(config.items.Wakefield))
    .then(() => queryContainer
    ())
    .then(() => replaceFamilyItem(config.items.Andersen))
    .then(() => queryContainer())

    // ADD THIS PART TO YOUR CODE
    .then(() => deleteFamilyItem(config.items.Andersen))
    // ENDS HERE

    .then(() => { exit(`Completed successfully`); })
    .catch((error) => { exit(`Completed with error ${JSON.stringify(error)}`) });
```

在終端機中，找出您的 ```app.js``` 檔案並執行命令： 

```bash 
node app.js
```

恭喜！ 您已成功刪除 Azure Cosmos DB 項目。

## <a id="DeleteDatabase"></a>步驟 11：刪除資料庫

刪除已建立的資料庫將會移除資料庫和所有子系資源 (容器、項目等)。

將 **cleanup** 函式複製並貼到 **deleteFamilyItem** 函式下方，以移除資料庫及其所有的子系資源。

```nodejs
/**
 * Cleanup the database and container on completion
 */
async function cleanup() {
    await client.database(databaseId).delete();
}
```

將下列程式碼複製並貼到 **deleteFamilyItem** 的呼叫下方，以執行 **cleanup** 函式。

```nodejs
createDatabase()
    .then(() => readDatabase())
    .then(() => createContainer())
    .then(() => readContainer())
    .then(() => createFamilyItem(config.items.Andersen))
    .then(() => createFamilyItem(config.items.Wakefield))
    .then(() => queryContainer())
    .then(() => replaceFamilyItem(config.items.Andersen))
    .then(() => queryContainer())
    .then(() => deleteFamilyItem(config.items.Andersen))

    // ADD THIS PART TO YOUR CODE
    .then(() => cleanup())
    // ENDS HERE

    .then(() => { exit(`Completed successfully`); })
    .catch((error) => { exit(`Completed with error ${JSON.stringify(error)}`) });
```

## <a id="Run"></a>步驟 12：一起執行您的 Node.js 應用程式！

整體來看，您的程式碼應會顯示如下：
```nodejs
const CosmosClient = require('@azure/cosmos').CosmosClient;

const config = require('./config');
const url = require('url');

const endpoint = config.endpoint;
const masterKey = config.primaryKey;

const HttpStatusCodes = { NOTFOUND: 404 };

const databaseId = config.database.id;
const containerId = config.container.id;

const client = new CosmosClient({ endpoint: endpoint, auth: { masterKey: masterKey } });

/**
 * Create the database if it does not exist
 */
async function createDatabase() {
    const { database } = await client.databases.createIfNotExists({ id: databaseId });
    console.log(`Created database:\n${database.id}\n`);
}

/**
 * Read the database definition
 */
async function readDatabase() {
    const { body: databaseDefinition } = await client.database(databaseId).read();
    console.log(`Reading database:\n${databaseDefinition.id}\n`);
}

/**
 * Create the container if it does not exist
 */
async function createContainer() {
    const { container } = await client.database(databaseId).containers.createIfNotExists({ id: containerId });
    console.log(`Created container:\n${config.container.id}\n`);
}

/**
 * Read the container definition
 */
async function readContainer() {
    const { body: containerDefinition } = await client.database(databaseId).container(containerId).read();
    console.log(`Reading container:\n${containerDefinition.id}\n`);
}

/**
 * Create family item if it does not exist
 */
async function createFamilyItem(itemBody) {
    try {
        // read the item to see if it exists
        const { item } = await client.database(databaseId).container(containerId).item(itemBody.id).read();
        console.log(`Item with family id ${itemBody.id} already exists\n`);
    }
    catch (error) {
        // create the family item if it does not exist
        if (error.code === HttpStatusCodes.NOTFOUND) {
            const { item } = await client.database(databaseId).container(containerId).items.create(itemBody);
            console.log(`Created family item with id:\n${itemBody.id}\n`);
        } else {
            throw error;
        }
    }
};

/**
 * Query the container using SQL
 */
async function queryContainer() {
    console.log(`Querying container:\n${config.container.id}`);

    // query to return all children in a family
    const querySpec = {
        query: "SELECT VALUE r.children FROM root r WHERE r.lastName = @lastName",
        parameters: [
            {
                name: "@lastName",
                value: "Andersen"
            }
        ]
    };

    const { result: results } = await client.database(databaseId).container(containerId).items.query(querySpec).toArray();
    for (var queryResult of results) {
        let resultString = JSON.stringify(queryResult);
        console.log(`\tQuery returned ${resultString}\n`);
    }
};

/**
 * Replace the item by ID.
 */
async function replaceFamilyItem(itemBody) {
    console.log(`Replacing item:\n${itemBody.id}\n`);
    // Change property 'grade'
    itemBody.children[0].grade = 6;
    const { item } = await client.database(databaseId).container(containerId).item(itemBody.id).replace(itemBody);
};

/**
 * Delete the item by ID.
 */
async function deleteFamilyItem(itemBody) {
    await client.database(databaseId).container(containerId).item(itemBody.id).delete(itemBody);
    console.log(`Deleted item:\n${itemBody.id}\n`);
};

/**
 * Cleanup the database and container on completion
 */
async function cleanup() {
    await client.database(databaseId).delete();
}

/**
 * Exit the app with a prompt
 * @param {message} message - The message to display
 */
function exit(message) {
    console.log(message);
    console.log('Press any key to exit');
    process.stdin.setRawMode(true);
    process.stdin.resume();
    process.stdin.on('data', process.exit.bind(process, 0));
}

createDatabase()
    .then(() => readDatabase())
    .then(() => createContainer())
    .then(() => readContainer())
    .then(() => createFamilyItem(config.items.Andersen))
    .then(() => createFamilyItem(config.items.Wakefield))
    .then(() => queryContainer())
    .then(() => replaceFamilyItem(config.items.Andersen))
    .then(() => queryContainer())
    .then(() => deleteFamilyItem(config.items.Andersen))
    .then(() => cleanup())
    .then(() => { exit(`Completed successfully`); })
    .catch((error) => { exit(`Completed with error ${JSON.stringify(error)}`) });
```

在終端機中，找出您的 ```app.js``` 檔案並執行命令： 

```bash 
node app.js
```

您應該可以看到入門應用程式的輸出。 輸出應該會與下列範例文字相符。

    Created database:
    FamilyDatabase

    Reading database:
    FamilyDatabase

    Created container:
    FamilyContainer

    Reading container:
    FamilyContainer

    Created family item with id:
    Anderson.1

    Created family item with id:
    Wakefield.7

    Querying container:
    FamilyContainer
            Query returned [{"firstName":"Henriette Thaulow","gender":"female","grade":5,"pets":[{"givenName":"Fluffy"}]}]

    Replacing item:
    Anderson.1

    Querying container:
    FamilyContainer
            Query returned [{"firstName":"Henriette Thaulow","gender":"female","grade":6,"pets":[{"givenName":"Fluffy"}]}]

    Deleted item:
    Anderson.1

    Completed successfully
    Press any key to exit

恭喜！ 您已完成 Node.js 教學課程，並擁有您的第一個 Azure Cosmos DB 主控台應用程式！

## <a id="GetSolution"></a>取得完整的 Node.js 教學課程方案

如果您沒有時間完成本教學課程中的步驟，或只想要下載程式碼，您可以從 [GitHub](https://github.com/Azure-Samples/azure-cosmos-db-sql-api-nodejs-getting-started ) 取得程式碼。

若要執行包含本文中所有程式碼的入門解決方案，您將需要下列項目：

* [Azure Cosmos DB 帳戶][create-account]。
* 您可以在 GitHub 上找到[入門](https://github.com/Azure-Samples/azure-cosmos-db-sql-api-nodejs-getting-started)解決方案。

透過 npm 安裝 **@azure/cosmos** 模組。 使用下列命令：

* ```npm install @azure/cosmos --save```

接下來，將 ```config.js``` 檔案中的 config.endpoint 和 config.primaryKey 的值更新為如 [步驟 3：設定您的應用程式組態](#Config)中所述。 

然後，在終端機中找出您的 ```app.js``` 檔案，並執行命令： 

```bash 
node app.js
```

就這麼容易，繼續努力！ 

## <a name="next-steps"></a>後續步驟
* 需要更複雜的 Node.js 範例嗎？ 請參閱[使用 Azure Cosmos DB 來建置 Node.js Web 應用程式](sql-api-nodejs-application.md)。
* 了解如何[監視 Azure Cosmos DB 帳戶](monitor-accounts.md)。
* 在 [Query Playground](https://www.documentdb.com/sql/demo)中，針對範例資料集執行查詢。

[create-account]: create-sql-api-dotnet.md#create-account
[keys]: media/sql-api-nodejs-get-started/node-js-tutorial-keys.png