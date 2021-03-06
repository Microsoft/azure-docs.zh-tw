---
title: 教學課程：此 Java 應用程式用以將範例資料載入至 Azure Cosmos DB 中的 Cassandra API 資料表
description: 本教學課程說明如何使用 Java 應用程式，將範例使用者資料載入至 Azure Cosmos DB 中的 Cassandra API 資料表。
author: kanshiG
ms.service: cosmos-db
ms.subservice: cosmosdb-cassandra
ms.topic: tutorial
ms.date: 05/20/2019
ms.author: govindk
ms.reviewer: sngun
ms.openlocfilehash: ad3dfc8c86d22e58377ebfbdb94b2beb004ded56
ms.sourcegitcommit: 3bdeb546890a740384a8ef383cf915e84bd7e91e
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/30/2020
ms.locfileid: "93092461"
---
# <a name="tutorial-load-sample-data-into-a-cassandra-api-table-in-azure-cosmos-db"></a>教學課程：將範例資料載入至 Azure Cosmos DB 中的 Cassandra API 資料表
[!INCLUDE[appliesto-cassandra-api](includes/appliesto-cassandra-api.md)]

身為開發人員，您可能有使用索引鍵/值組的應用程式。 您可以使用 Azure Cosmos DB 中的 Cassandra API 帳戶來儲存和管理索引鍵/值資料。 本教學課程說明如何使用 Java 應用程式，將範例使用者資料載入至 Azure Cosmos DB Cassandra API 帳戶中的資料表。 Java 應用程式會使用 [Java 驅動程式](https://github.com/datastax/java-driver)並載入使用者資料，例如使用者識別碼、使用者名稱和使用者所在城市。 

本教學課程涵蓋下列工作：

> [!div class="checklist"]
> * 將資料載入至 Cassandra 資料表
> * 執行應用程式

如果您沒有 Azure 訂用帳戶，請在開始前建立[免費帳戶](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) 。

## <a name="prerequisites"></a>Prerequisites

* 本文為多部分教學課程的一部分。 開始本文件之前，請務必先[建立 Cassandra API 帳戶、Keyspace 及資料表](create-cassandra-api-account-java.md)。   

## <a name="load-data-into-the-table"></a>將資料載入到資料表

若要將資料載入您的 Cassandra API 資料表中，請使用下列步驟：

1. 開啟 “src\main\java\com\azure\cosmosdb\cassandra” 資料夾中的 “UserRepository.java” 檔案，然後附加程式碼，以將 user_id、user_name 及 user_bcity 欄位插入資料表中：

   ```java
   /**
   * Insert a row into user table
   *
   * @param id   user_id
   * @param name user_name
   * @param city user_bcity
   */
   public void insertUser(PreparedStatement statement, int id, String name, String city) {
        BoundStatement boundStatement = new BoundStatement(statement);
        session.execute(boundStatement.bind(id, name, city));
   }

   /**
   * Create a PrepareStatement to insert a row to user table
   *
   * @return PreparedStatement
   */
   public PreparedStatement prepareInsertStatement() {
      final String insertStatement = "INSERT INTO  uprofile.user (user_id, user_name , user_bcity) VALUES (?,?,?)";
   return session.prepare(insertStatement);
   }
   ```
 
2. 開啟 “src\main\java\com\azure\cosmosdb\cassandra” 資料夾中的 “UserProfile.java” 檔案。 此類別包含會呼叫您先前所定義 createKeyspace 和 createTable 方法的主要方法。 現在，附加下列程式碼，以將某些範例資料插入 Cassandra API 資料表中。

   ```java
   //Insert rows into user table
   PreparedStatement preparedStatement = repository.prepareInsertStatement();
     repository.insertUser(preparedStatement, 1, "JohnH", "Seattle");
     repository.insertUser(preparedStatement, 2, "EricK", "Spokane");
     repository.insertUser(preparedStatement, 3, "MatthewP", "Tacoma");
     repository.insertUser(preparedStatement, 4, "DavidA", "Renton");
     repository.insertUser(preparedStatement, 5, "PeterS", "Everett");
   ```

## <a name="run-the-app"></a>執行應用程式

開啟命令提示字元或終端機視窗，然後將資料夾路徑變更至您已建立專案的位置。 執行 “mvn clean install” 命令，以在目標資料夾中產生 cosmosdb-cassandra-examples.jar 檔案，然後執行應用程式。 

```bash
cd "cassandra-demo"

mvn clean install

java -cp target/cosmosdb-cassandra-examples.jar com.azure.cosmosdb.cassandra.examples.UserProfile
```

您現在可以在 Azure 入口網站中開啟「資料總管」，以確認是否已將使用者資訊新增到資料表中。
    
## <a name="next-steps"></a>後續步驟

在本教學課程中，您已了解如何將範例資料載入至 Azure Cosmos DB 中的 Cassandra API 帳戶。 您現在可以繼續進行下一篇文章：

> [!div class="nextstepaction"]
> [從 Cassandra API 帳戶查詢資料](cassandra-api-query-data.md)
