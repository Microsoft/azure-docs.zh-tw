---
title: Azure Cosmos DB：使用 JavaScript SDK 建置 Node.js 應用程式來管理 Azure Cosmos DB SQL API 資料 | Microsoft Docs
description: 提供 Node.js 程式碼範例，您可用來連線及查詢 Azure Cosmos DB SQL API
services: cosmos-db
author: deborahc
ms.service: cosmos-db
ms.component: cosmosdb-sql
ms.custom: quick start connect, mvc
ms.devlang: nodejs
ms.topic: quickstart
ms.date: 09/24/2018
ms.author: dech
ms.openlocfilehash: a8a81556002bec82325bf4cf53b68ff49b8b6917
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46982319"
---
# <a name="azure-cosmos-db-build-a-nodejs-app-using-javascript-sdk-to-manage-azure-cosmos-db-sql-api-data"></a>Azure Cosmos DB：使用 JavaScript SDK 建置 Node.js 應用程式來管理 Azure Cosmos DB SQL API 資料

> [!div class="op_single_selector"]
> * [.NET](create-sql-api-dotnet.md)
> * [Java](create-sql-api-java.md)
> * [Node.js](create-sql-api-nodejs.md)
> * [Python](create-sql-api-python.md)
> * [Xamarin](create-sql-api-xamarin-dotnet.md)
>  

Azure Cosmos DB 是 Microsoft 的全域分散式多模型資料庫服務。 您可以快速建立及查詢文件、索引鍵/值及圖形資料庫，所有這些都受惠於位於 Azure Cosmos DB 核心的全域散發和水平調整功能。 

此快速入門示範如何使用 Azure 入口網站建立 Azure Cosmos DB [SQL API](sql-api-introduction.md) 帳戶、文件資料庫和容器。 接著，您要建置和執行以 [SQL JavaScript SDK](sql-api-sdk-node.md) 為基礎的主控台應用程式。 此快速入門使用 2.0 版的 [JavaScript SDK](https://www.npmjs.com/package/@azure/cosmos)。

## <a name="prerequisites"></a>必要條件

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)] 
[!INCLUDE [cosmos-db-emulator-docdb-api](../../includes/cosmos-db-emulator-docdb-api.md)]

* 此外：
    * [Node.js](https://nodejs.org/en/) v6.0.0 版或更高版本
    * [Git](http://git-scm.com/)

## <a name="create-a-database-account"></a>建立資料庫帳戶

[!INCLUDE [cosmos-db-create-dbaccount](../../includes/cosmos-db-create-dbaccount.md)]

## <a name="add-a-collection"></a>新增集合

[!INCLUDE [cosmos-db-create-collection](../../includes/cosmos-db-create-collection.md)]

## <a name="add-sample-data"></a>新增範例資料

[!INCLUDE [cosmos-db-create-sql-api-add-sample-data](../../includes/cosmos-db-create-sql-api-add-sample-data.md)]

## <a name="query-your-data"></a>查詢資料

[!INCLUDE [cosmos-db-create-sql-api-query-data](../../includes/cosmos-db-create-sql-api-query-data.md)]

## <a name="clone-the-sample-application"></a>複製範例應用程式

現在，我們將從 GitHub 複製 SQL API 應用程式、設定連接字串，然後加以執行。

1. 開啟命令提示字元，建立名為 git-samples 的新資料夾，然後關閉命令提示字元。

    ```bash
    md "C:\git-samples"
    ```

2. 開啟 git 終端機視窗 (例如 git bash)，並使用 `cd` 命令變更至要安裝範例應用程式的新資料夾。

    ```bash
    cd "C:\git-samples"
    ```

3. 執行下列命令來複製範例存放庫。 此命令會在您的電腦上建立範例應用程式副本。

    ```bash
    git clone https://github.com/Azure-Samples/azure-cosmos-db-sql-api-nodejs-getting-started.git
    ```

## <a name="review-the-code"></a>檢閱程式碼

此為選用步驟。 若您想要了解如何在程式碼中建立資料庫資源，則可檢閱下列程式碼片段。 或者也可以直接跳至[更新您的連接字串](#update-your-connection-string)。 

請注意，如果您熟悉舊版 JavaScript SDK，您可能已習慣看到「集合」和「文件」等字詞。 因為 Azure Cosmos DB 支援[多個 API 模型](https://docs.microsoft.com/azure/cosmos-db/introduction#key-capabilities)，2.0+ 版的 JavaScript SDK 會使用「容器」這個泛用字詞，此字詞可能是用來說明容器內容的集合、圖表或資料表和「項目」。

下列程式碼片段全部取自 **app.js** 檔案。

* 已初始化 `CosmosClient`。

    ```nodejs
    const client = new CosmosClient({ endpoint: endpoint, auth: { masterKey: masterKey } });
    ```

* 已建立新資料庫。

    ```nodejs
    const { database } = await client.databases.createIfNotExists({ id: databaseId });
    ```

* 已建立新容器 (集合)。

    ```nodejs
    const { container } = await client.database(databaseId).containers.createIfNotExists({ id: containerId });
    ```

* 已建立項目 (文件)。

    ```nodejs
    const { item } = await client.database(databaseId).container(containerId).items.create(itemBody);
    ```

* 已透過 JSON 執行 SQL 查詢。

    ```nodejs
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
    ```    

## <a name="update-your-connection-string"></a>更新您的連接字串

現在，返回 Azure 入口網站以取得連接字串資訊，並將它複製到應用程式中。

1. 在 [Azure 入口網站](http://portal.azure.com/)中，於您 Azure Cosmos DB 帳戶的左側瀏覽區中，按一下 [金鑰]，然後按一下 [讀寫金鑰]。 在下一個步驟中，您將使用畫面右側的複製按鈕，將 URI 和主要金鑰複製到 `config.js` 檔案。

    ![在 Azure 入口網站的 [金鑰] 刀鋒視窗中檢視並複製存取金鑰](./media/create-sql-api-dotnet/keys.png)

2. 開啟 `config.js` 檔案。 

3. 從入口網站複製您的 URI 值 (使用 [複製] 按鈕)，並使它成為 `config.js` 中的端點金鑰值。 

    `config.endpoint = "https://FILLME.documents.azure.com"`

4. 然後，從入口網站複製您的主要金鑰值，並使它成為 `config.js` 中的 `config.primaryKey` 值。 您現已更新應用程式，使其具有與 Azure Cosmos DB 通訊所需的所有資訊。 

    `config.primaryKey = "FILLME"`
    
## <a name="run-the-app"></a>執行應用程式
1. 在終端機中執行 `npm install` 以安裝必要的 npm 模組

2. 在終端機中執行 `node app.js` 來啟動您的節點應用程式。

您現在可以返回 [資料總管]，以查看、查詢、修改及使用這項新資料。 

## <a name="review-slas-in-the-azure-portal"></a>在 Azure 入口網站中檢閱 SLA

[!INCLUDE [cosmosdb-tutorial-review-slas](../../includes/cosmos-db-tutorial-review-slas.md)]

## <a name="clean-up-resources"></a>清除資源

[!INCLUDE [cosmosdb-delete-resource-group](../../includes/cosmos-db-delete-resource-group.md)]

## <a name="next-steps"></a>後續步驟

在本快速入門中，您已了解如何建立 Azure Cosmos DB 帳戶、如何使用 [資料總管] 建立集合，以及如何執行應用程式。 您現在可以將其他資料匯入到 Cosmos DB 帳戶。 

> [!div class="nextstepaction"]
> [將資料匯入到 Azure Cosmos DB](import-data.md)


