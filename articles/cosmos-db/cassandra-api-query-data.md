---
title: 從 Azure Cosmos DB Cassandra API 帳戶查詢資料
description: 本文說明如何使用 Java 應用程式，從 Azure Cosmos DB Cassandra API 帳戶查詢使用者資料。
services: cosmos-db
ms.service: cosmos-db
author: kanshiG
ms.author: govindk
ms.reviewer: sngun
ms.component: cosmosdb-cassandra
ms.topic: tutorial
ms.date: 09/24/2018
ms.openlocfilehash: c1fb4c27f897e3c0952ed6419e167613ac8204f7
ms.sourcegitcommit: ad08b2db50d63c8f550575d2e7bb9a0852efb12f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/26/2018
ms.locfileid: "47223486"
---
# <a name="query-data-from-an-azure-cosmos-db-cassandra-api-account"></a>從 Azure Cosmos DB Cassandra API 帳戶查詢資料

本教學課程說明如何使用 Java 應用程式，從 Azure Cosmos DB Cassandra API 帳戶查詢使用者資料。 Java 應用程式會使用 [Java 驅動程式](https://github.com/datastax/java-driver)並查詢使用者資料，例如使用者識別碼、使用者名稱、使用者所在城市。 

本教學課程涵蓋下列工作：

> [!div class="checklist"]
> * 從 Cassandra 資料表查詢資料
> * 執行應用程式

## <a name="prerequisites"></a>必要條件

* 本文為多部分教學課程的一部分。 在開始之前，請務必完成上述步驟來[建立 Cassandra API 帳戶、keyspace、資料表](create-cassandra-api-account-java.md)，並[將範例資料載入到資料表](cassandra-api-load-data.md)。 

## <a name="query-data"></a>查詢資料

開啟 `src\main\java\com\azure\cosmosdb\cassandra` 資料夾下的 `UserRepository.java` 檔案。 附加下列程式碼區塊。 此程式碼提供三個函式：查詢資料庫中的所有使用者、依使用者識別碼篩選查詢特定使用者，以及刪除資料表。 

```java
/**
* Select all rows from user table
*/
public void selectAllUsers() {

    final String query = "SELECT * FROM uprofile.user";
    List<Row> rows = session.execute(query).all();

    for (Row row : rows) {
       LOGGER.info("Obtained row: {} | {} | {} ", row.getInt("user_id"), row.getString("user_name"), row.getString("user_bcity"));
    }
}

/**
* Select a row from user table
*
* @param id user_id
*/
public void selectUser(int id) {
    final String query = "SELECT * FROM uprofile.user where user_id = 3";
    Row row = session.execute(query).one();

    LOGGER.info("Obtained row: {} | {} | {} ", row.getInt("user_id"), row.getString("user_name"), row.getString("user_bcity"));
}

/**
* Delete user table.
*/
public void deleteTable() {
   final String query = "DROP TABLE IF EXISTS uprofile.user";
   session.execute(query);
}
```

開啟 `src\main\java\com\azure\cosmosdb\cassandra` 資料夾下的 `UserProfile.java` 檔案。 此類別包含會呼叫您先前所定義 createKeyspace、createTable 和 insert data 方法的主要方法。 現在，附加查詢所有使用者或特定使用者的下列程式碼：

```java
LOGGER.info("Select all users");
repository.selectAllUsers();

LOGGER.info("Select a user by id (3)");
repository.selectUser(3);

LOGGER.info("Delete the users profile table");
repository.deleteTable();
```

## <a name="run-the-java-app"></a>執行 Java 應用程式
1. 開啟命令提示字元或終端機視窗。 貼上下列程式碼區塊。 

   此程式碼會將目錄變更為 (cd) 您建立專案的資料夾路徑。 然後，會執行 `mvn clean install` 命令來在目標資料夾內產生 `cosmosdb-cassandra-examples.jar` 檔案。 最後，會執行 Java 應用程式。

   ```bash
   cd "cassandra-demo"
   
   mvn clean install
   
   java -cp target/cosmosdb-cassandra-examples.jar com.azure.cosmosdb.cassandra.examples.UserProfile
   ```

2. 現在，在 Azure 入口網站中，開啟 [資料總管] 並確認已刪除使用者資料表。

## <a name="next-steps"></a>後續步驟

* 在本教學課程中，您已了解如何從 Azure Cosmos DB Cassandra API 帳戶查詢資料。 您現在可以繼續進行下一篇文章：

> [!div class="nextstepaction"]
> [將資料移轉到 Cassandra API 帳戶](cassandra-import-data.md)


