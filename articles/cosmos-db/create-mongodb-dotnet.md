---
title: Azure CosmosDB︰使用 .NET 和 MongoDB API 建置 Web 應用程式 | Microsoft Docs
description: 提供 .NET 程式碼範例，您可用來連線及查詢 Azure Cosmos DB MongoDB API
services: cosmos-db
author: slyons
manager: kfile
ms.service: cosmos-db
ms.component: cosmosdb-mongo
ms.custom: quick start connect, mvc
ms.devlang: dotnet
ms.topic: quickstart
ms.date: 05/22/2018
ms.author: sclyon
ms.openlocfilehash: 7ab02cf2cc9a25a5c4c7aa6d782d37d932dc8369
ms.sourcegitcommit: cb61439cf0ae2a3f4b07a98da4df258bfb479845
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/05/2018
ms.locfileid: "43701946"
---
# <a name="azure-cosmos-db-build-a-mongodb-api-web-app-with-net-and-the-azure-portal"></a>Azure CosmosDB︰使用 .NET 和 Azure 入口網站建置 MongoDB API Web 應用程式

> [!div class="op_single_selector"]
> * [.NET](create-mongodb-dotnet.md)
> * [Java](create-mongodb-java.md)
> * [Node.js](create-mongodb-nodejs.md)
> * [Python](create-mongodb-flask.md)
> * [Xamarin](create-mongodb-xamarin.md)
> * [Golang](create-mongodb-golang.md)
>  

Azure Cosmos DB 是 Microsoft 的全域分散式多模型資料庫服務。 您可以快速建立及查詢文件、索引鍵/值及圖形資料庫，所有這些都受惠於位於 Azure Cosmos DB 核心的全域散發和水平調整功能。 

此快速入門示範如何使用 Azure 入口網站建立 Azure Cosmos DB [MongoDB API](mongodb-introduction.md) 帳戶、文件資料庫和集合。 您會接著建置和部署以 [MongoDB .NET 驅動程式](https://docs.mongodb.com/ecosystem/drivers/csharp/)為基礎的工作清單 Web 應用程式。

## <a name="prerequisites-to-run-the-sample-app"></a>執行範例應用程式的必要條件

若要執行範例，您將需要 [Visual Studio](https://www.visualstudio.com/downloads/) 和有效的 Azure CosmosDB 帳戶。

如果您還沒有 Visual Studio，請下載 [Visual Studio 2017 Community Edition](https://www.visualstudio.com/downloads/)，其在安裝時會一併安裝 **ASP.NET 及 Web 開發**工作負載。

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)] 

<a id="create-account"></a>
## <a name="create-a-database-account"></a>建立資料庫帳戶

[!INCLUDE [cosmos-db-create-dbaccount](../../includes/cosmos-db-create-dbaccount-mongodb.md)]

本文所述的範例與 MongoDB.Driver 2.6.1 版相容。

## <a name="clone-the-sample-app"></a>複製範例應用程式

首先，從 GitHub 下載 MongoDB API 範例應用程式。 它會使用 MongoDB 的文件儲存體模型實作工作清單。

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
    git clone https://github.com/Azure-Samples/azure-cosmos-db-mongodb-dotnet-getting-started.git
    ```

如果您不想使用 git，也可以[下載 ZIP 檔格式的專案](https://github.com/Azure-Samples/azure-cosmos-db-mongodb-dotnet-getting-started/archive/master.zip)。

## <a name="review-the-code"></a>檢閱程式碼

此為選用步驟。 若您想要瞭解如何在程式碼中建立資料庫資源，則可檢閱下列程式碼片段。 或者也可以直接跳至[更新您的連接字串](#update-your-connection-string)。 

下列程式碼片段皆取自 DAL 目錄中的 Dal.cs 檔案。

* 初始化 Mongo 用戶端。

    ```cs
        MongoClientSettings settings = new MongoClientSettings();
        settings.Server = new MongoServerAddress(host, 10255);
        settings.UseSsl = true;
        settings.SslSettings = new SslSettings();
        settings.SslSettings.EnabledSslProtocols = SslProtocols.Tls12;

        MongoIdentity identity = new MongoInternalIdentity(dbName, userName);
        MongoIdentityEvidence evidence = new PasswordEvidence(password);

        settings.Credential = new MongoCredential("SCRAM-SHA-1", identity, evidence);

        MongoClient client = new MongoClient(settings);
    ```

* 擷取資料庫和集合。

    ```cs
    private string dbName = "Tasks";
    private string collectionName = "TasksList";

    var database = client.GetDatabase(dbName);
    var todoTaskCollection = database.GetCollection<MyTask>(collectionName);
    ```

* 擷取所有文件。

    ```cs
    collection.Find(new BsonDocument()).ToList();
    ```

* 建立工作，並將其插入到 MongoDB 集合

   ```csharp
    public void CreateTask(MyTask task)
    {
        var collection = GetTasksCollectionForEdit();
        try
        {
            collection.InsertOne(task);
        }
        catch (MongoCommandException ex)
        {
            string msg = ex.Message;
        }
    }
   ```
   同樣地，您可以使用 [collection.UpdateOne()](https://docs.mongodb.com/stitch/mongodb/actions/collection.updateOne/index.html) 和 [collection.DeleteOne()](https://docs.mongodb.com/stitch/mongodb/actions/collection.deleteOne/index.html) 方法來更新和刪除文件。 

## <a name="update-your-connection-string"></a>更新您的連接字串

現在，返回 Azure 入口網站以取得連接字串資訊，並將它複製到應用程式中。

1. 在 [Azure 入口網站](http://portal.azure.com/)中，於您 Azure Cosmos DB 帳戶的左側瀏覽區中，按一下 [連接字串]，然後按一下 [讀寫金鑰]。 在下一個步驟中，您將使用畫面右側的複製按鈕，將使用者名稱、密碼和主機複製到 Dal.cs 檔案中。

2. 開啟 **DAL** 目錄中的 **Dal.cs** 檔案。 

3. 從入口網站複製您的 **username** 值 (使用 [複製] 按鈕)，並使它成為 **Dal.cs** 檔案中的 **username** 值。 

4. 然後從入口網站複製您的 **host** 值，並使它成為 **Dal.cs** 檔案中的 **host** 值。 

5. 最後從入口網站複製您的 **password** 值，並使它成為 **Dal.cs** 檔案中的 **password** 值。 

您現已更新應用程式，使其具有與 Azure Cosmos DB 通訊所需的所有資訊。 
    
## <a name="run-the-web-app"></a>執行 Web 應用程式

1. 在 Visual Studio 中，於 [方案總管] 中的專案上按一下滑鼠右鍵，然後按一下 [管理 NuGet 套件]。 

2. 在 NuGet [瀏覽] 方塊中，輸入 MongoDB.Driver。

3. 從結果中，安裝 **MongoDB.Driver** 程式庫。 這會安裝 MongoDB.Driver 套件以及所有相依性。

4. 按 CTRL + F5 來執行應用程式。 您的應用程式會顯示在瀏覽器中。 

5. 按一下瀏覽器中的 [建立]，然後在您的工作清單應用程式中建立一些新工作。

## <a name="review-slas-in-the-azure-portal"></a>在 Azure 入口網站中檢閱 SLA

[!INCLUDE [cosmosdb-tutorial-review-slas](../../includes/cosmos-db-tutorial-review-slas.md)]

## <a name="clean-up-resources"></a>清除資源

[!INCLUDE [cosmosdb-delete-resource-group](../../includes/cosmos-db-delete-resource-group.md)]

## <a name="next-steps"></a>後續步驟

在本快速入門中，您已了解如何建立 Azure Cosmos DB 帳戶，以及如何使用適用於 MongoDB 的 API 來執行 Web 應用程式。 您現在可以將其他資料匯入到 Cosmos DB 帳戶。 

> [!div class="nextstepaction"]
> [將資料匯入 MongoDB API 的 Azure Cosmos DB](mongodb-migrate.md)

