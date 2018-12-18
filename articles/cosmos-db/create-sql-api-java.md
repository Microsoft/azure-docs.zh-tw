---
title: 使用 Java 建立 Azure Cosmos DB 文件資料庫 | Microsoft Docs'
description: 提供 Java 程式碼範例，您可用來連線及查詢 Azure Cosmos DB SQL API
services: cosmos-db
author: SnehaGunda
manager: kfile
ms.service: cosmos-db
ms.component: cosmosdb-sql
ms.custom: quick start connect, mvc, devcenter
ms.devlang: java
ms.topic: quickstart
ms.date: 03/26/2018
ms.author: sngun
ms.openlocfilehash: 85a7a6f5b1224c732f5a385789aef13e1d7bd1db
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46961241"
---
# <a name="azure-cosmos-db-create-a-document-database-using-java-and-the-azure-portal"></a>Azure Cosmos DB︰使用 Java 和 Azure 入口網站建立文件資料庫

> [!div class="op_single_selector"]
> * [.NET](create-sql-api-dotnet.md)
> * [Java](create-sql-api-java.md)
> * [Node.js](create-sql-api-nodejs.md)
> * [Python](create-sql-api-python.md)
> * [Xamarin](create-sql-api-xamarin-dotnet.md)
>  

Azure Cosmos DB 是 Microsoft 的全域分散式多模型資料庫服務。 您可使用 Azure Cosmos DB 快速地建立和查詢受控文件、資料表及圖形資料庫。

本快速入門會使用 Azure Cosmos DB [SQL API](sql-api-introduction.md)適用的 Azure 入口網站工具建立文件資料庫。 本快速入門也會說明如何使用 [SQL Java API](sql-api-sdk-java.md) 快速建立 Java 主控台應用程式。 本快速入門中的指示可運用在任何足以執行 Java 應用程式的作業系統上。 完成本快速入門，您就會熟悉如何在 UI 中或以程式設計的方式，建立和修改文件資料庫資源 (不論您偏好哪種方式)。

## <a name="prerequisites"></a>必要條件

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)] 
[!INCLUDE [cosmos-db-emulator-docdb-api](../../includes/cosmos-db-emulator-docdb-api.md)]

此外： 

* [Java Development Kit (JDK) 1.7+](http://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html)
    * 在 Ubuntu 上，執行 `apt-get install default-jdk` 來安裝 JDK。
    * 務必設定 JAVA_HOME 環境變數，以指向 JDK 安裝所在的資料夾。
* [下載](http://maven.apache.org/download.cgi)和[安裝 ](http://maven.apache.org/install.html) [Maven](http://maven.apache.org/) 二進位封存檔
    * 在 Ubuntu 上，您可以執行 `apt-get install maven` 來安裝 Maven。
* [Git](https://www.git-scm.com/)
    * 在 Ubuntu 上，您可以執行 `sudo apt-get install git` 來安裝 Git。

## <a name="create-a-database-account"></a>建立資料庫帳戶

您必須先使用 Azure Cosmos DB 建立 SQL API 帳戶，才可以建立文件資料庫。

[!INCLUDE [cosmos-db-create-dbaccount](../../includes/cosmos-db-create-dbaccount.md)]

## <a name="add-a-collection"></a>新增集合

[!INCLUDE [cosmos-db-create-collection](../../includes/cosmos-db-create-collection.md)]

<a id="add-sample-data"></a>
## <a name="add-sample-data"></a>新增範例資料

[!INCLUDE [cosmos-db-create-sql-api-add-sample-data](../../includes/cosmos-db-create-sql-api-add-sample-data.md)]

## <a name="query-your-data"></a>查詢資料

[!INCLUDE [cosmos-db-create-sql-api-query-data](../../includes/cosmos-db-create-sql-api-query-data.md)]

## <a name="clone-the-sample-application"></a>複製範例應用程式

現在讓我們切換為使用程式碼。 我們將從 GitHub 複製 SQL API 應用程式、設定連接字串，然後加以執行。 您會看到，以程式設計方式來處理資料有多麼的容易。 

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
    git clone https://github.com/Azure-Samples/azure-cosmos-db-documentdb-java-getting-started.git
    ```

## <a name="review-the-code"></a>檢閱程式碼

此為選用步驟。 若您想要瞭解如何在程式碼中建立資料庫資源，則可檢閱下列程式碼片段。 或者也可以直接跳至[更新您的連接字串](#update-your-connection-string)。 

下列程式碼片段皆是取自 C:\git-samples\azure-cosmos-db-documentdb-java-getting-started\src\GetStarted\Program.java 檔案。

* `DocumentClient` 初始化。 [DocumentClient](https://docs.microsoft.com/java/api/com.microsoft.azure.documentdb._document_client) 提供適用於 Azure Cosmos DB 資料庫服務的用戶端邏輯表示法。 此用戶端會用於設定和執行針對服務的要求。 此程式碼的 `FILLME` 部分稍後會在快速入門中更新。

    ```java
    this.client = new DocumentClient("https://FILLME.documents.azure.com",
            "FILLME", 
            new ConnectionPolicy(),
            ConsistencyLevel.Session);
    ```

* [資料庫](https://docs.microsoft.com/java/api/com.microsoft.azure.documentdb._database)建立。

    ```java
    Database database = new Database();
    database.setId(databaseName);
    
    this.client.createDatabase(database, null);
    ```

* [DocumentCollection](https://docs.microsoft.com/java/api/com.microsoft.azure.documentdb._document_collection) 建立。

    ```java
    DocumentCollection collectionInfo = new DocumentCollection();
    collectionInfo.setId(collectionName);

    ...

    this.client.createCollection(databaseLink, collectionInfo, requestOptions);
    ```

* 使用 [createDocument](https://docs.microsoft.com/java/api/com.microsoft.azure.documentdb._document_client.createdocument) 方法建立文件。

    ```java
    // Any Java object within your code can be serialized into JSON and written to Azure Cosmos DB
    Family andersenFamily = new Family();
    andersenFamily.setId("Andersen.1");
    andersenFamily.setLastName("Andersen");
    // More properties

    String collectionLink = String.format("/dbs/%s/colls/%s", databaseName, collectionName);
    this.client.createDocument(collectionLink, family, new RequestOptions(), true);
    ```

* 使用 [queryDocuments](https://docs.microsoft.com/java/api/com.microsoft.azure.documentdb._document_client.querydocuments) 方法，透過 JSON 執行 SQL 查詢。

    ```java
    FeedOptions queryOptions = new FeedOptions();
    queryOptions.setPageSize(-1);
    queryOptions.setEnableCrossPartitionQuery(true);

    String collectionLink = String.format("/dbs/%s/colls/%s", databaseName, collectionName);
    FeedResponse<Document> queryResults = this.client.queryDocuments(
        collectionLink,
        "SELECT * FROM Family WHERE Family.lastName = 'Andersen'", queryOptions);

    System.out.println("Running SQL query...");
    for (Document family : queryResults.getQueryIterable()) {
        System.out.println(String.format("\tRead %s", family));
    }
    ```    

## <a name="update-your-connection-string"></a>更新您的連接字串

現在，返回 Azure 入口網站以取得連接字串資訊，並將它複製到應用程式中。 這可讓您的應用程式與託管資料庫進行通訊。

1. 在 [Azure 入口網站](http://portal.azure.com/)中，按一下 [金鑰]。 

    使用畫面右方的複製按鈕來複製最高值，亦即 URI。

    ![在 Azure 入口網站的 [金鑰] 頁面中，檢視並複製存取金鑰](./media/create-sql-api-java/keys.png)

2. 從以下資料夾中開啟 `Program.java` 檔案：C:\git-samples\azure-cosmos-db-documentdb-java-getting-started\src\GetStarted。 

3. 從入口網站將 URI 值貼上至 `https://FILLME.documents.azure.com` 的行 45。

4. 返回入口網站並複製 PRIMARY KEY 值，如螢幕擷取畫面中所示。 從入口網站將 PRIMARY KEY 值貼上至 `FILLME` 的行 46。

    getStartedDemo method 現應如下所示： 
    
    ```java
    private void getStartedDemo() throws DocumentClientException, IOException {
        this.client = new DocumentClient("https://youraccountname.documents.azure.com:443/",
                "your-primary-key...RJhQrqQ5QQ==", 
                new ConnectionPolicy(),
                ConsistencyLevel.Session);
    ```

5. 儲存 Program.java 檔案。

## <a name="run-the-app"></a>執行應用程式

1. 在 git 終端機視窗中，`cd` 至 azure-cosmos-db-documentdb-java-getting-started 資料夾。

    ```git
    cd "C:\git-samples\azure-cosmos-db-documentdb-java-getting-started"
    ```

2. 在 git 終端機視窗中，使用下列命令來安裝必要的 Java 套件。

    ```
    mvn package
    ```

3. 在 git 終端機視窗中，使用下列命令以啟動 JAVA 應用程式。

    ```
    mvn exec:java -D exec.mainClass=GetStarted.Program
    ```

    終端機視窗會顯示已建立 FamilyDB 資料庫的通知。 
    
4. 請按任一按鍵來建立資料庫，然後按下另一個按鍵來建立集合。 

    程式結束時會刪除所有資源，因此請切換回瀏覽器中的資料總管，您會看到資料總管中現在已包含 FamilyDB 資料庫和 FamilyCollection 集合。

5. 切換至 [主控台] 視窗，然後按下任一按鍵來建立第一個文件，然後按下另一個按鍵以建立第二個文件。 然後切換回資料總管以檢視這些文件。 

6. 按下任一按鍵以執行查詢，並在 [主控台] 視窗中查看輸出。 

7. 您按下的下一個按鍵會刪除資源。 如果您想要保留資源，您可以在 [主控台] 視窗中按 CTRL + C 來結束程式。 否則，請按任意鍵來刪除帳戶中的資源，如此一來，您就不會產生費用。 

    ![主控台輸出](./media/create-sql-api-java/console-output.png)


## <a name="review-slas-in-the-azure-portal"></a>在 Azure 入口網站中檢閱 SLA

[!INCLUDE [cosmosdb-tutorial-review-slas](../../includes/cosmos-db-tutorial-review-slas.md)]

## <a name="clean-up-resources"></a>清除資源

[!INCLUDE [cosmosdb-delete-resource-group](../../includes/cosmos-db-delete-resource-group.md)]

## <a name="next-steps"></a>後續步驟

在本快速入門中，您已了解如何使用 [資料總管] 來建立 Azure Cosmos DB 帳戶、文件資料庫和集合，以及如何執行應用程式來以程式設計方式執行同樣的作業。 您現可將其他資料匯入至 Azure Cosmos DB 集合。 

> [!div class="nextstepaction"]
> [將資料匯入到 Azure Cosmos DB](import-data.md)


