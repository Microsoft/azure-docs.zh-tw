---
title: NoSQL 教學課程：適用於 Azure Cosmos DB Java SDK 的 SQL API | Microsoft Docs
description: NoSQL 教學課程，將使用適用於 Azure Cosmos DB 的 SQL API 來建立線上資料庫以及 Java 主控台應用程式。 Azure SQL 是 JSON 的 NoSQL 資料庫。
keywords: nosql 教學課程, 線上資料庫, java 主控台應用程式
services: cosmos-db
author: SnehaGunda
manager: kfile
ms.service: cosmos-db
ms.component: cosmosdb-sql
ms.devlang: java
ms.topic: tutorial
ms.date: 05/22/2017
ms.author: sngun
ms.openlocfilehash: 99accb8a822aedc387bbe137e8e80e298fe7cd1d
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46991176"
---
# <a name="nosql-tutorial-build-a-sql-api-java-console-application"></a>NoSQL 教學課程：建置 SQL API Java 主控台應用程式

> [!div class="op_single_selector"]
> * [.NET](sql-api-get-started.md)
> * [.NET Core](sql-api-dotnetcore-get-started.md)
> * [Java](sql-api-java-get-started.md)
> * [非同步 Java](sql-api-async-java-get-started.md)
> * [Node.js](sql-api-nodejs-get-started.md)
> 

歡迎使用適用於 Azure Cosmos DB Java SDK 之 SQL API 的 NoSQL 教學課程！ 完成本教學課程之後，您將會有一個主控台應用程式，可用來建立和查詢 Azure Cosmos DB 資源。

我們會說明︰

* 建立及連線至 Azure Cosmos DB 帳戶
* 設定 Visual Studio 方案
* 建立線上資料庫
* 建立集合
* 建立 JSON 文件
* 查詢集合
* 建立 JSON 文件
* 查詢集合
* 取代文件
* 刪除文件
* 刪除資料庫

讓我們開始吧！

## <a name="prerequisites"></a>必要條件
請確定您具有下列項目：

* 使用中的 Azure 帳戶。 如果您沒有帳戶，您可以註冊 [免費帳戶](https://azure.microsoft.com/free/)。 

  [!INCLUDE [cosmos-db-emulator-docdb-api](../../includes/cosmos-db-emulator-docdb-api.md)]

* [Git](https://git-scm.com/downloads)。
* [Java Development Kit (JDK) 7+](http://www.oracle.com/technetwork/java/javase/downloads/index.html)。
* [Maven](http://maven.apache.org/download.cgi).

## <a name="step-1-create-an-azure-cosmos-db-account"></a>步驟 1：建立 Azure Cosmos DB 帳戶
讓我們來建立 Azure Cosmos DB 帳戶。 如果您已經擁有想要使用的帳戶，就可以跳到[複製 GitHub 專案](#GitClone)。 如果您是使用 Azure Cosmos DB 模擬器，請遵循 [Azure Cosmos DB 模擬器](local-emulator.md)的步驟來設定模擬器，並請直接跳到[複製 GitHub 專案](#GitClone)。

[!INCLUDE [cosmos-db-create-dbaccount](../../includes/cosmos-db-create-dbaccount.md)]

## <a id="GitClone"></a>步驟 2︰複製 GitHub 專案
您可以先從複製[開始使用 Azure Cosmos DB 和 Java](https://github.com/Azure-Samples/documentdb-java-getting-started) 的 GitHub 存放庫著手。 例如，請從本機目錄執行下列命令將範例專案擷取到本機。

    git clone git@github.com:Azure-Samples/azure-cosmos-db-documentdb-java-getting-started.git

    cd azure-cosmos-db-documentdb-java-getting-started

目錄中包含專案的 `pom.xml` 和 `src` 資料夾，此資料夾中所含的 Java 原始程式碼包括 `Program.java`)，它會顯示如何使用 Azure Cosmos DB 執行簡單的作業，例如建立文件和查詢集合內的資料。 `pom.xml` 包含 [Maven 上的 Azure Cosmos DB Java SDK](https://mvnrepository.com/artifact/com.microsoft.azure/azure-documentdb) 上的相依性。

    <dependency>
        <groupId>com.microsoft.azure</groupId>
        <artifactId>azure-documentdb</artifactId>
        <version>LATEST</version>
    </dependency>

## <a id="Connect"></a>步驟 3：連接至 Azure Cosmos DB 帳戶
接下來，回到 [Azure 入口網站](https://portal.azure.com)擷取您的端點和主要金鑰。 必須提供 Azure Cosmos DB 端點 URL 和主要金鑰，您的應用程式才能了解所要連線的位置，而 Azure Cosmos DB 才會信任您的應用程式連線。

在 Azure 入口網站中，瀏覽至 Azure Cosmos DB 帳戶，然後按一下 [金鑰]。 從入口網站複製 URI，並將它貼到 Program.java 檔案的 `https://FILLME.documents.azure.com` 中。 然後從入口網站複製主要金鑰，並將它貼到 `FILLME`中。

    this.client = new DocumentClient(
        "https://FILLME.documents.azure.com",
        "FILLME"
        , new ConnectionPolicy(),
        ConsistencyLevel.Session);

![NoSQL 教學課程用來建立 Java 主控台應用程式之 Azure 入口網站的螢幕擷取畫面。 顯示 Azure Cosmos DB 帳戶，內含反白顯示的 [主動式] 集線器、[Azure Cosmos DB 帳戶] 刀鋒視窗上反白顯示的 [金鑰] 按鈕、[金鑰] 刀鋒視窗上反白顯示的 [URI]、[主要金鑰] 和 [次要金鑰] 值][keys]

## <a name="step-4-create-a-database"></a>步驟 4：建立資料庫
可以使用 **DocumentClient** 類別的 [createDatabase](/java/api/com.microsoft.azure.documentdb._document_client.createdatabase) 方法建立 Azure Cosmos DB [資料庫](sql-api-resources.md#databases)。 資料庫是分割給多個集合之 JSON 文件儲存體的邏輯容器。

    Database database = new Database();
    database.setId("familydb");
    this.client.createDatabase(database, null);

## <a id="CreateColl"></a>步驟 5：建立集合
> [!WARNING]
> **createCollection** 會建立含有保留輸送量且具有價格含意的新集合。 如需詳細資訊，請造訪[定價頁面](https://azure.microsoft.com/pricing/details/cosmos-db/)。
> 
> 

可以使用 **DocumentClient** 類別的 [createCollection](/java/api/com.microsoft.azure.documentdb._document_client.createcollection) 方法建立[集合](sql-api-resources.md#collections)。 集合是 JSON 文件和相關聯 JavaScript 應用程式邏輯的容器。


    DocumentCollection collectionInfo = new DocumentCollection();
    collectionInfo.setId("familycoll");

    // Azure Cosmos DB collections can be reserved with throughput specified in request units/second. 
    // Here we create a collection with 400 RU/s.
    RequestOptions requestOptions = new RequestOptions();
    requestOptions.setOfferThroughput(400);

    this.client.createCollection("/dbs/familydb", collectionInfo, requestOptions);

## <a id="CreateDoc"></a>步驟 6：建立 JSON 文件
您可以使用 **DocumentClient** 類別的 [createDocument](/java/api/com.microsoft.azure.documentdb._document_client.createdocument) 方法來建立[文件](sql-api-resources.md#documents)。 文件會是使用者定義的 (任意) JSON 內容。 現在可插入一或多份文件。 如果您已經有想要儲存於資料庫中的資料，就可以使用 Azure Cosmos DB 的[資料移轉工具](import-data.md)，將資料匯入資料庫中。

    // Insert your Java objects as documents 
    Family andersenFamily = new Family();
    andersenFamily.setId("Andersen.1");
    andersenFamily.setLastName("Andersen")

    // More initialization skipped for brevity. You can have nested references
    andersenFamily.setParents(new Parent[] { parent1, parent2 });
    andersenFamily.setDistrict("WA5");
    Address address = new Address();
    address.setCity("Seattle");
    address.setCounty("King");
    address.setState("WA");

    andersenFamily.setAddress(address);
    andersenFamily.setRegistered(true);

    this.client.createDocument("/dbs/familydb/colls/familycoll", family, new RequestOptions(), true);

![說明 NoSQL 教學課程用來建立 Java 主控台應用程式之帳戶、線上資料庫、集合和文件之間階層式關聯性的圖表](./media/sql-api-get-started/nosql-tutorial-account-database.png)

## <a id="Query"></a>步驟 7︰查詢 Azure Cosmos DB 資源
Azure Cosmos DB 支援針對儲存於每個集合的 JSON 文件進行豐富[查詢](sql-api-sql-query.md)。  下列範例程式碼示範如何使用 SQL 語法與 [queryDocuments](/java/api/com.microsoft.azure.documentdb._document_client.querydocuments) 方法來查詢 Azure Cosmos DB 中的文件。

    FeedResponse<Document> queryResults = this.client.queryDocuments(
        "/dbs/familydb/colls/familycoll",
        "SELECT * FROM Family WHERE Family.lastName = 'Andersen'", 
        null);

    System.out.println("Running SQL query...");
    for (Document family : queryResults.getQueryIterable()) {
        System.out.println(String.format("\tRead %s", family));
    }

## <a id="ReplaceDocument"></a>步驟 8︰取代 JSON 文件
Azure Cosmos DB 支援使用 [replaceDocument](/java/api/com.microsoft.azure.documentdb._document_client.replacedocument) 方法來更新 JSON 文件。

    // Update a property
    andersenFamily.Children[0].Grade = 6;

    this.client.replaceDocument(
        "/dbs/familydb/colls/familycoll/docs/Andersen.1", 
        andersenFamily,
        null);

## <a id="DeleteDocument"></a>步驟 9︰刪除 JSON 文件
同樣地，Azure Cosmos DB 支援使用 [deleteDocument](/java/api/com.microsoft.azure.documentdb._document_client.deletedocument) 方法來將 JSON 文件刪除。  

    this.client.delete("/dbs/familydb/colls/familycoll/docs/Andersen.1", null);

## <a id="DeleteDatabase"></a>步驟 10：刪除資料庫
刪除已建立的資料庫會移除資料庫和所有子系資源 (集合、文件等)。

    this.client.deleteDatabase("/dbs/familydb", null);

## <a id="Run"></a>步驟 11：一起執行您的 Java 主控台應用程式！
若要從主控台執行應用程式，瀏覽至專案資料夾並使用 Maven 進行編譯：
    
    mvn package

執行 `mvn package` 會從 Maven 下載最新的 Azure Cosmos DB 文件庫，並產生 `GetStarted-0.0.1-SNAPSHOT.jar`。 然後執行下列命令來執行應用程式︰

    mvn exec:java -D exec.mainClass=GetStarted.Program

恭喜！ 您已經完成本 NoSQL 教學課程，並擁有運作中的 Java 主控台應用程式！

## <a name="next-steps"></a>後續步驟
* 想要 Java Web 應用程式教學課程嗎？ 請參閱[使用 Azure Cosmos DB 以 Java 建置 Web 應用程式](sql-api-java-application.md)。
* 了解如何[監視 Azure Cosmos DB 帳戶](monitor-accounts.md)。
* 在 [Query Playground](https://www.documentdb.com/sql/demo)中，針對範例資料集執行查詢。

[keys]: media/sql-api-get-started/nosql-tutorial-keys.png
