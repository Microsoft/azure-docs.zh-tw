---
title: 使用 Java 應用程式來建立 Azure Cosmos DB Cassandra API 帳戶、資料庫及資料表
description: 本文介紹如何使用 Java 應用程式建立 Cassandra API 帳戶，並將資料庫 (也稱為 Keyspace) 和資料表新增至該帳戶。
author: kanshiG
ms.author: govindk
ms.reviewer: sngun
services: cosmos-db
ms.service: cosmos-db
ms.component: cosmosdb-cassandra
ms.topic: tutorial
ms.date: 09/24/2018
ms.openlocfilehash: b5e3d87e026b65a602b7bdf2e52365d13b21f62f
ms.sourcegitcommit: c282021dbc3815aac9f46b6b89c7131659461e49
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/12/2018
ms.locfileid: "49166826"
---
# <a name="create-an-azure-cosmos-db-cassandra-api-account-database-and-a-table-by-using-a-java-application"></a>使用 Java 應用程式來建立 Azure Cosmos DB Cassandra API 帳戶、資料庫及資料表

本教學課程說明如何使用 Java 應用程式來建立 Azure Cosmos DB Cassandra API 帳戶、新增資料庫 (也稱為 keyspace) 以及新增資料表。 Java 應用程式會使用 [Java 驅動程式](https://github.com/datastax/java-driver)來建立使用者資料庫，其中包含使用者識別碼、使用者名稱、使用者所在城市等詳細資料。  

本教學課程涵蓋下列工作：

> [!div class="checklist"]
> * 建立 Cassandra 資料庫帳戶
> * 取得帳戶連接字串
> * 建立 Maven 專案和相依性
> * 新增資料庫和資料表
> * 執行應用程式

## <a name="prerequisites"></a>必要條件 

* 如果您沒有 Azure 訂用帳戶，請在開始前建立  [免費帳戶](https://azure.microsoft.com/free/?ref=microsoft.com&utm_source=microsoft.com&utm_medium=docs&utm_campaign=visualstudio) 。 或者，您可以 [免費試用 Azure Cosmos DB](https://azure.microsoft.com/try/cosmosdb/) ，無需 Azure 訂用帳戶，也無需任何費用和履約承諾。 

* 取得最新版本的 [Java Development Kit (JDK)](http://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html) 

* [下載](http://maven.apache.org/download.cgi)及[安裝](http://maven.apache.org/install.html) [Maven](http://maven.apache.org/) 二進位封存 
  - 在 Ubuntu 上，您可以執行  `apt-get install maven` 來安裝 Maven。 

## <a name="create-a-database-account"></a>建立資料庫帳戶 

1. 登入  [Azure 入口網站](https://portal.azure.com/)。 

2. 選取 [建立資源]  **** >[資料庫] **** >[Azure Cosmos DB]  ****。 

3. 在 [新增帳戶]  窗格中，輸入新 Azure Cosmos DB 帳戶的設定。 **** 

   |設定   |建議的值  |說明  |
   |---------|---------|---------|
   |ID   |   輸入唯一名稱    | 輸入唯一名稱來識別此 Azure Cosmos DB 帳戶。 <br/><br/>因為 cassandra.cosmosdb.azure.com 會附加到您所提供的識別碼以建立接觸點，所以請使用可供辨識的唯一識別碼。         |
   |API    |  Cassandra   |  API 會決定要建立的帳戶類型。 <br/> 選取 [Cassandra]，因為在本文中您將建立可以使用 CQL 語法來查詢的寬欄資料庫。  |
   |訂用帳戶    |  您的訂用帳戶        |  選取您要用於此 Azure Cosmos DB 帳戶的 Azure 訂用帳戶。        |
   |資源群組   | 輸入名稱    |  選取 [新建] ，然後為您的帳戶輸入新的資源群組名稱。 **** 為求簡化，您可以使用和識別碼相同的名稱。    |
   |位置    |  選取最接近使用者的區域    |  選取用來主控 Azure Cosmos DB 帳戶的地理位置。 使用最接近使用者的位置，以便他們能以最快速度存取資料。    |

   ![使用入口網站建立帳戶](./media/create-cassandra-api-account-java/create-account.png)

4. 接下來，選取 [建立]。 **** <br/>建立帳戶需要幾分鐘的時間。 建立資源之後，您可以在入口網站的右上角看到**部署成功**通知。

## <a name="get-the-connection-details-of-your-account"></a>取得您帳戶的連線詳細資料  

從 Azure 入口網站取得連接字串資訊，並將其複製到 Java 設定檔。 這可讓您的應用程式與託管資料庫進行通訊。 

1. 從  [Azure 入口網站](http://portal.azure.com/)瀏覽至您的 Azure Cosmos DB 帳戶。 

2. 開啟 [連接字串] 窗格。 ****  

3. 複製**接觸點**、**連接埠**、**使用者名稱**以及**主要密碼**的值，以用於後續步驟。

## <a name="create-maven-project-dependencies-and-utility-classes"></a>建立 Maven 專案、相依性以及公用程式類別 

您在本文中使用的 Java 範例專案裝載於 GitHub。 您可從 [azure-cosmos-db-cassandra-java-getting-started](https://github.com/Azure-Samples/azure-cosmos-db-cassandra-java-getting-started) 存放庫加以下載。 

下載檔案之後，更新 `java-examples\src\main\resources\config.properties` 檔案內的連接字串資訊，然後執行。  

```java
cassandra_host=<FILLME_with_CONTACT POINT> 
cassandra_port = 10350 
cassandra_username=<FILLME_with_USERNAME> 
cassandra_password=<FILLME_with_PRIMARY PASSWORD> 
```

或者，您也可以從頭開始建置範例。  

1. 從終端機或命令提示字元中，建立名為 Cassandra-demo 的新 Maven 專案。 

   ```bash
   mvn archetype:generate -DgroupId=com.azure.cosmosdb.cassandra -DartifactId=cassandra-demo -DarchetypeArtifactId=maven-archetype-quickstart -DinteractiveMode=false 
   ```
 
2. 找到 `cassandra-demo` 資料夾。 使用文字編輯器，開啟產生的 `pom.xml` 檔案。 

   新增 Cassandra 相依性，並建置專案所需的外掛程式，如 [pom.xml](https://github.com/Azure-Samples/azure-cosmos-db-cassandra-java-getting-started/blob/master/java-examples/pom.xml) 檔案中所示。  

3. 在 `cassandra-demo\src\main` 資料夾下方，建立名為 `resources` 的新資料夾。  在資源資料夾下方，新增 config.properties 和 log4j.properties 檔案：

   - [Config.properties](https://github.com/Azure-Samples/azure-cosmos-db-cassandra-java-getting-started/blob/master/java-examples/src/main/resources/config.properties) 檔案會儲存 Azure Cosmos DB Cassandra API 連線端點和金鑰值。 
   
   - [Log4j.properties](https://github.com/Azure-Samples/azure-cosmos-db-cassandra-java-getting-started/blob/master/java-examples/src/main/resources/log4j.properties) 檔案會定義使用 Cassandra API 互動時所需的記錄層級。  

4. 接下來，瀏覽至 `src/main/java/com/azure/cosmosdb/cassandra/` 資料夾。 在 Cassandra 資料夾內，建立名為 `utils` 的另一個資料夾。 新資料夾會儲存連線至 Cassandra API 帳戶時所需的公用程式類別。 

   新增 [CassandraUtils](https://github.com/Azure-Samples/azure-cosmos-db-cassandra-java-getting-started/blob/master/java-examples/src/main/java/com/azure/cosmosdb/cassandra/util/CassandraUtils.java) 類別來建立叢集，以及開啟和關閉 Cassandra 工作階段。 叢集會連線至 Azure Cosmos DB Cassandra API，並傳回要存取的工作階段。 使用 [Configurations](https://github.com/Azure-Samples/azure-cosmos-db-cassandra-java-getting-started/blob/master/java-examples/src/main/java/com/azure/cosmosdb/cassandra/util/Configurations.java) 類別來讀取 config.properties 檔案中的連接字串資訊。 

5. Java 範例會以使用者的資訊 (例如使用者名稱、使用者識別碼、使用者所在城市) 來建立資料庫。 您需要定義 get 和 set 方法，以在main 函式中存取使用者詳細資料。
 
   使用 get 和 set 方法，在 `src/main/java/com/azure/cosmosdb/cassandra/` 資料下方建立 [User.java](https://github.com/Azure-Samples/azure-cosmos-db-cassandra-java-getting-started/blob/master/java-examples/src/main/java/com/azure/cosmosdb/cassandra/User.java) 類別。 

## <a name="add-a-database-and-a-table"></a>新增資料庫和資料表  

本節說明如何使用 Cassandra 查詢語言 (CQL) 來新增資料庫 (Keyspace) 和資料表。 若要深入了解這些命令的 CQL 語法，請參閱 [create keyspace](https://docs.datastax.com/en/cql/3.3/cql/cql_reference/cqlCreateKeyspace.html) 和 [create table](https://docs.datastax.com/en/cql/3.3/cql/cql_reference/cqlCreateTable.html#cqlCreateTable) 查詢語法。 

1. 在 `src\main\java\com\azure\cosmosdb\cassandra` 資料夾下方，建立名為 `repository` 的新資料夾。 

2. 接下來，建立 `UserRepository` Java 類別，並將下列程式碼新增至該類別： 

   ```java
   package com.azure.cosmosdb.cassandra.repository; 
   import java.util.List; 
   import com.datastax.driver.core.BoundStatement; 
   import com.datastax.driver.core.PreparedStatement; 
   import com.datastax.driver.core.Row; 
   import com.datastax.driver.core.Session; 
   import org.slf4j.Logger; 
   import org.slf4j.LoggerFactory; 
   
   /** 
    * Create a Cassandra session 
    */ 
   public class UserRepository { 
   
       private static final Logger LOGGER = LoggerFactory.getLogger(UserRepository.class); 
       private Session session; 
       public UserRepository(Session session) { 
           this.session = session; 
       } 
   
       /** 
       * Create keyspace uprofile in cassandra DB 
        */ 
   
       public void createKeyspace() { 
            final String query = "CREATE KEYSPACE IF NOT EXISTS uprofile WITH REPLICATION = { 'class' : 'NetworkTopologyStrategy', 'datacenter1' : 1 }"; 
           session.execute(query); 
           LOGGER.info("Created keyspace 'uprofile'"); 
       } 
   
       /** 
        * Create user table in cassandra DB 
        */ 
   
       public void createTable() { 
           final String query = "CREATE TABLE IF NOT EXISTS uprofile.user (user_id int PRIMARY KEY, user_name text, user_bcity text)"; 
           session.execute(query); 
           LOGGER.info("Created table 'user'"); 
       } 
   } 
   ```

3. 找到 `src\main\java\com\azure\cosmosdb\cassandra` 資料夾並建立名為 `examples` 的新子資料夾。

4. 接下來，建立 `UserProfile` Java 類別。 此類別包含會呼叫您先前所定義 createKeyspace 和 createTable 方法的主要方法： 

   ```java
   package com.azure.cosmosdb.cassandra.examples; 
   import java.io.IOException; 
   import com.azure.cosmosdb.cassandra.repository.UserRepository; 
   import com.azure.cosmosdb.cassandra.util.CassandraUtils; 
   import com.datastax.driver.core.PreparedStatement; 
   import com.datastax.driver.core.Session; 
   import org.slf4j.Logger; 
   import org.slf4j.LoggerFactory; 
   
   /** 
    * Example class which will demonstrate following operations on Cassandra Database on CosmosDB 
    * - Create Keyspace 
    * - Create Table 
    * - Insert Rows 
    * - Select all data from a table 
    * - Select a row from a table 
    */ 
   
   public class UserProfile { 
   
       private static final Logger LOGGER = LoggerFactory.getLogger(UserProfile.class); 
       public static void main(String[] s) throws Exception { 
           CassandraUtils utils = new CassandraUtils(); 
           Session cassandraSession = utils.getSession(); 
   
           try { 
               UserRepository repository = new UserRepository(cassandraSession); 
               //Create keyspace in cassandra database 
               repository.createKeyspace(); 
               //Create table in cassandra database 
               repository.createTable(); 
   
           } finally { 
               utils.close(); 
               LOGGER.info("Please delete your table after verifying the presence of the data in portal or from CQL"); 
           } 
       } 
   } 
   ```
 
## <a name="run-the-app"></a>執行應用程式 

1. 開啟命令提示字元或終端機視窗。 貼上下列程式碼區塊。 

   此程式碼會將目錄變更為 (cd) 您建立專案的資料夾路徑。 然後，會執行 `mvn clean install` 命令來在目標資料夾內產生 `cosmosdb-cassandra-examples.jar` 檔案。 最後，會執行 Java 應用程式。

   ```bash
   cd cassandra-demo

   mvn clean install 

   java -cp target/cosmosdb-cassandra-examples.jar com.azure.cosmosdb.cassandra.examples.UserProfile 
   ```

   終端機視窗會顯示 keyspace 和資料表已建立的通知。 
   
2. 現在，在 Azure 入口網站中開啟 [資料總管]，確認已建立 Keyspace 和資料表。

## <a name="next-steps"></a>後續步驟

在本教學課程中，您已了解如何使用 Java 應用程式來建立 Azure Cosmos DB Cassandra API 帳戶、資料庫及資料表。 您現在可以繼續進行下一篇文章：

> [!div class="nextstepaction"]
> [將範例資料載入 Cassandra API 資料表中](cassandra-api-load-data.md)。
