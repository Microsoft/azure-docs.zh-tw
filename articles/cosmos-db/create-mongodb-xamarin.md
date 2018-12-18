---
title: Azure CosmosDB︰使用 .NET 和 MongoDB API 建置 Xamarin.Forms 應用程式 | Microsoft Docs
description: 提供 Xamarin.Forms 程式碼範例，您可用來連線及查詢 Azure Cosmos DB MongoDB API
services: cosmos-db
author: codemillmatt
manager: kfile
ms.service: cosmos-db
ms.component: cosmosdb-mongo
ms.custom: quickstart, xamarin
ms.devlang: dotnet
ms.topic: quickstart
ms.date: 06/20/2018
ms.author: masoucou
ms.openlocfilehash: e002f99cc16721b78d85284105ce22ee7d1eb9a3
ms.sourcegitcommit: e2ea404126bdd990570b4417794d63367a417856
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/14/2018
ms.locfileid: "45574822"
---
# <a name="quickstart-build-a-mongodb-api-xamarinforms-app-with-net-and-the-azure-portal"></a>快速入門︰使用 .NET 和 Azure 入口網站建置 MongoDB API Xamarin.Forms 應用程式

> [!div class="op_single_selector"]
> * [.NET](create-mongodb-dotnet.md)
> * [Java](create-mongodb-java.md)
> * [Node.js](create-mongodb-nodejs.md)
> * [Python](create-mongodb-flask.md)
> * [Xamarin](create-mongodb-xamarin.md)
> * [Golang](create-mongodb-golang.md)
>  

Azure Cosmos DB 是 Microsoft 的全域分散式多模型資料庫服務。 您可以快速建立及查詢文件、索引鍵/值及圖形資料庫，所有這些都受惠於位於 Azure Cosmos DB 核心的全域散發和水平調整功能。

此快速入門示範如何使用 Azure 入口網站建立 Azure Cosmos DB [MongoDB API](mongodb-introduction.md) 帳戶、文件資料庫和集合。 然後，您將使用 [MongoDB.NET 驅動程式](https://docs.mongodb.com/ecosystem/drivers/csharp/)建置 ToDo 應用程式的 Xamarin.Forms 應用程式。

## <a name="prerequisites-to-run-the-sample-app"></a>執行範例應用程式的必要條件

若要執行範例，您將需要 [Visual Studio](https://www.visualstudio.com/downloads/) 或 [Visual Studio for Mac](https://visualstudio.microsoft.com/vs/mac/)，以及有效的 Azure CosmosDB 帳戶。

如果您還沒有 Visual Studio，請下載 [Visual Studio 2017 Community Edition](https://www.visualstudio.com/downloads/)，其在安裝時會一併安裝**使用 .NET 進行行動裝置開發**的工作負載。

如果您想要在 Mac 上作業，請下載 [Visual Studio for Mac](https://visualstudio.microsoft.com/vs/mac/) 並執行安裝程式。

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

<a id="create-account"></a>

## <a name="create-a-database-account"></a>建立資料庫帳戶

[!INCLUDE [cosmos-db-create-dbaccount](../../includes/cosmos-db-create-dbaccount-mongodb.md)]

本文所述的範例與 MongoDB.Driver 2.6.1 版相容。

## <a name="clone-the-sample-app"></a>複製範例應用程式

首先，從 GitHub 下載 MongoDB API 範例應用程式。 它會使用 MongoDB 的文件儲存體模型實作 ToDo 應用程式。

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
    git clone https://github.com/Azure-Samples/azure-cosmos-db-mongodb-xamarin-getting-started.git
    ```

如果您不想使用 git，也可以[下載 ZIP 檔格式的專案](https://github.com/Azure-Samples/azure-cosmos-db-mongodb-xamarin-getting-started/archive/master.zip)

## <a name="review-the-code"></a>檢閱程式碼

此為選用步驟。 若您想要瞭解如何在程式碼中建立資料庫資源，則可檢閱下列程式碼片段。 或者也可以直接跳至[更新您的連接字串](#update-your-connection-string)。

下列程式碼片段皆取自 `MongoService` 類別，可在下列路徑找到：src/TaskList.Core/Services/MongoService.cs。

* 初始化 Mongo 用戶端。
    ```cs
    MongoClientSettings settings = MongoClientSettings.FromUrl(
        new MongoUrl(APIKeys.ConnectionString)
    );

    settings.SslSettings =
        new SslSettings() { EnabledSslProtocols = SslProtocols.Tls12 };

    MongoClient mongoClient = new MongoClient(settings);
    ```

* 擷取資料庫和集合的參考。 MongoDB.NET SDK 會自動建立資料庫和集合 (如果尚未存在)。
    ```cs
    string dbName = "MyTasks";
    string collectionName = "TaskList";

    var db = mongoClient.GetDatabase(dbName);

    var collectionSettings = new MongoCollectionSettings {
        ReadPreference = ReadPreference.Nearest
    };

    tasksCollection = db.GetCollection<MyTask>(collectionName, collectionSettings);
    ```
* 擷取所有文件以作為清單。
    ```cs
    var allTasks = await TasksCollection
                    .Find(new BsonDocument())
                    .ToListAsync();
    ```

* 查詢特定文件。
    ```cs
    public async Task<List<MyTask>> GetIncompleteTasksDueBefore(DateTime date)
    {
        var tasks = await TasksCollection
                        .AsQueryable()
                        .Where(t => t.Complete == false)
                        .Where(t => t.DueDate < date)
                        .ToListAsync();

        return tasks;
    }
    ```

* 建立工作，並將其插入到 MongoDB 集合。
    ```cs
    public async Task CreateTask(MyTask task)
    {
        await TasksCollection.InsertOneAsync(task);
    }
    ```

* 更新 MongoDB 集合中的工作。
    ```cs
    public async Task UpdateTask(MyTask task)
    {
        await TasksCollection.ReplaceOneAsync(t => t.Id.Equals(task.Id), task);
    }
    ```

* 刪除 MongoDB 集合中的工作。
    ```cs
    public async Task DeleteTask(MyTask task)
    {
        await TasksCollection.DeleteOneAsync(t => t.Id.Equals(task.Id));
    }
    ```

<a id="update-your-connection-string"></a>

## <a name="update-your-connection-string"></a>更新您的連接字串

現在，返回 Azure 入口網站以取得連接字串資訊，並將它複製到應用程式中。

1. 在 [Azure 入口網站](http://portal.azure.com/)中，於您 Azure Cosmos DB 帳戶的左側瀏覽區中，按一下 [連接字串]，然後按一下 [讀寫金鑰]。 在後續步驟中，您將使用畫面右方的複製按鈕來複製主要連接字串。

2. 在 **TaskList.Core** 專案的 [Helpers]\(協助程式\) 目錄中，開啟 **APIKeys.cs** 檔案。

3. 從入口網站 (使用 [複製] 按鈕) 複製您的**主要連接字串**，並使它成為 **APIKeys.cs** 檔案中 [ConnectionString] 欄位的值。

您現已更新應用程式，使其具有與 Azure Cosmos DB 通訊所需的所有資訊。

## <a name="run-the-app"></a>執行應用程式

### <a name="visual-studio-2017"></a>Visual Studio 2017

1. 在 Visual Studio 中，於 [方案總管] 中的每個專案上按一下滑鼠右鍵，然後按一下 [管理 NuGet 套件]。
2. 按一下 [還原所有 Nuget 套件]。
3. 以滑鼠右鍵按一下 [TaskList.Android]，然後選取 [設定為啟始專案]。
4. 按 F5 開始對應用程式進行偵錯。
5. 如果您想要在 iOS 上執行，請先將您的機器連線到 Mac (這裡有如何執行此操作的[指示](https://docs.microsoft.com/xamarin/ios/get-started/installation/windows/introduction-to-xamarin-ios-for-visual-studio))。
6. 以滑鼠右鍵按一下 [TaskList.iOS] 專案，然後選取 [設定為啟始專案]。
7. 按 F5 開始對應用程式進行偵錯。

### <a name="visual-studio-for-mac"></a>Visual Studio for Mac

1. 根據您想要在哪個平台上執行作業，從平台下拉式清單中選取 [TaskList.iOS] 或 [TaskList.Android]。
2. 按 [cmd+Enter] 開始對應用程式進行偵錯。

## <a name="review-slas-in-the-azure-portal"></a>在 Azure 入口網站中檢閱 SLA

[!INCLUDE [cosmosdb-tutorial-review-slas](../../includes/cosmos-db-tutorial-review-slas.md)]

## <a name="clean-up-resources"></a>清除資源

[!INCLUDE [cosmosdb-delete-resource-group](../../includes/cosmos-db-delete-resource-group.md)]

## <a name="next-steps"></a>後續步驟

在本快速入門中，您已了解如何建立 Azure Cosmos DB 帳戶，以及如何使用適用於 MongoDB 的 API 來執行 Xamarin.Forms 應用程式。 您現在可以將其他資料匯入到 Cosmos DB 帳戶。

> [!div class="nextstepaction"]
> [將資料匯入 MongoDB API 的 Azure Cosmos DB](mongodb-migrate.md)