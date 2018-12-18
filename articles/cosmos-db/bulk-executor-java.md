---
title: 在 Azure Cosmos DB 中使用大量執行程式 Java 程式庫執行大量作業 | Microsoft Docs
description: 使用 Azure Cosmos DB 的大量執行程式 Java 程式庫將文件大量匯入並更新至 Azure Cosmos DB 容器。
keywords: Java 大量執行程式
services: cosmos-db
author: tknandu
manager: kfile
ms.service: cosmos-db
ms.devlang: java
ms.topic: conceptual
ms.date: 05/07/2018
ms.author: ramkris
ms.openlocfilehash: 9285b0ea50b7207aa40cea2dcab50f79863ffda9
ms.sourcegitcommit: ebd06cee3e78674ba9e6764ddc889fc5948060c4
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/07/2018
ms.locfileid: "44050182"
---
# <a name="use-bulk-executor-java-library-to-perform-bulk-operations-on-azure-cosmos-db-data"></a>在 Azure Cosmos DB 資料上使用大量執行程式 Java 程式庫執行大量作業

本教學課程說明如何使用 Azure Cosmos DB 大量執行程式的 Java 程式庫來匯入和更新 Azure Cosmos DB 文件。 若要深入了解大量執行程式程式庫，以及它如何協助您利用大量輸送量與儲存體，請參閱[大量執行程式程式庫概觀](bulk-executor-overview.md)一文。 在本教學課程中，您將建置會產生隨機文件的 Java 應用程式，而這些文件會大量匯入至 Azure Cosmos DB 容器。 匯入之後，您會大量更新文件的某些屬性。 

## <a name="prerequisites"></a>必要條件

* 如果您沒有 Azure 訂用帳戶，請在開始前建立 [免費帳戶](https://azure.microsoft.com/free/?ref=microsoft.com&utm_source=microsoft.com&utm_medium=docs&utm_campaign=visualstudio)。  

* 您可以[免費試用 Azure Cosmos DB](https://azure.microsoft.com/try/cosmosdb/)，無須 Azure 訂用帳戶，也無須任何費用和約定付款。 或者，您也可以搭配使用 [Azure Cosmos DB 模擬器](https://docs.microsoft.com/azure/cosmos-db/local-emulator)與 `https://localhost:8081` URI。 [驗證要求](local-emulator.md#authenticating-requests)中會提供主索引鍵。  

* [Java Development Kit (JDK) 1.7+](http://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html)  
  - 在 Ubuntu 上，執行 `apt-get install default-jdk` 來安裝 JDK。  

  - 務必設定 JAVA_HOME 環境變數，以指向 JDK 安裝所在的資料夾。

* [下載](http://maven.apache.org/download.cgi)和[安裝 ](http://maven.apache.org/install.html) [Maven](http://maven.apache.org/) 二進位封存檔  
  
  - 在 Ubuntu 上，您可以執行 `apt-get install maven` 來安裝 Maven。

* 使用 Java 快速入門文章中＜[建立資料庫帳戶](create-sql-api-java.md#create-a-database-account)＞一節所述的步驟，建立 Azure Cosmos DB SQL API 帳戶。

## <a name="clone-the-sample-application"></a>複製範例應用程式

現在我們會從 GitHub 下載 Java 應用程式範例，來轉換為使用程式碼。 此應用程式會對 Azure Cosmos DB 資料執行大量作業。 若要複製應用程式，請開啟命令提示字元，瀏覽至要從其中複製應用程式的目錄，然後執行下列命令：

```
 git clone https://github.com/Azure/azure-cosmosdb-bulkexecutor-java-getting-started.git 
```

複製的存放庫包含兩個與 "\azure-cosmosdb-bulkexecutor-java-getting-started\samples\bulkexecutor-sample\src\main\java\com\microsoft\azure\cosmosdb\bulkexecutor" 資料夾相關的 "bulkimport" 和 "bulkupdate" 範例。 "bulkimport" 應用程式會產生隨機文件，並將其匯入至 Azure Cosmos DB。 "bulkupdate" 應用程式會更新 Azure Cosmos DB 中的某些文件。 在後面幾節中，我們將檢閱這裡每一個範例應用程式中的程式碼。 

## <a name="bulk-import-data-to-azure-cosmos-db"></a>將資料大量匯入至 Azure Cosmos DB

1. Azure Cosmos DB 的連接字串會以引數的形式受到讀取，並且指派給 CmdLineConfiguration.java 檔案中定義的變數。  

2. 接著，DocumentClient 物件會使用下列陳述式進行初始化：  

   ```java
   ConnectionPolicy connectionPolicy = new ConnectionPolicy();
   connectionPolicy.setMaxPoolSize(1000);
   DocumentClient client = new DocumentClient(
      HOST,
      MASTER_KEY, 
      connectionPolicy,
      ConsistencyLevel.Session)
   ```

3. 針對等候時間和節流的要求，DocumentBulkExecutor 物件會使用高 retry 值進行初始化。 接著，這些值會設定為 0，如此會將壅塞控制傳遞給 DocumentBulkExecutor 以決定其存留期。  

   ```java
   // Set client's retry options high for initialization
   client.getConnectionPolicy().getRetryOptions().setMaxRetryWaitTimeInSeconds(30);
   client.getConnectionPolicy().getRetryOptions().setMaxRetryAttemptsOnThrottledRequests(9);

   // Builder pattern
   Builder bulkExecutorBuilder = DocumentBulkExecutor.builder().from(
     client,
     DATABASE_NAME,
     COLLECTION_NAME,
     collection.getPartitionKey(),
     offerThroughput) // throughput you want to allocate for bulk import out of the container's total throughput

   // Instantiate DocumentBulkExecutor
   DocumentBulkExecutor bulkExecutor = bulkExecutorBuilder.build()

   // Set retries to 0 to pass complete control to bulk executor
   client.getConnectionPolicy().getRetryOptions().setMaxRetryWaitTimeInSeconds(0);
   client.getConnectionPolicy().getRetryOptions().setMaxRetryAttemptsOnThrottledRequests(0);
```

4. 呼叫 importAll API，以產生隨機文件來大量匯入至 Azure Cosmos DB 容器。 您可以設定 CmdLineConfiguration.java 檔案內的命令列組態。

   ```java
   BulkImportResponse bulkImportResponse = bulkExecutor.importAll(documents, false, true, null);
```
   大量匯入 API 會接受 JSON 序列化的文件集合，並且具有下列語法。如需詳細資訊，請參閱 [API 文件](https://docs.microsoft.com/java/api/com.microsoft.azure.documentdb.bulkexecutor)：

   ```java
   public BulkImportResponse importAll(
        Collection<String> documents,
        boolean isUpsert,
        boolean disableAutomaticIdGeneration,
        Integer maxConcurrencyPerPartitionRange) throws DocumentClientException;   
   ```

   importAll 方法可接受下列參數：
 
   |**參數**  |**說明**  |
   |---------|---------|
   |isUpsert    |   啟用文件更新插入的旗標。 如果具有指定識別碼的文件已經存在，文件會進行更新。  |
   |disableAutomaticIdGeneration     |   停用自動產生識別碼的旗標。 預設會設定為 true。   |
   |maxConcurrencyPerPartitionRange    |  每個分割區索引鍵範圍的最大並行程度。 預設值為 20。  |

   **大量匯入回應物件定義**大量匯入 API 呼叫的結果包含下列 get 方法：

   |**參數**  |**說明**  |
   |---------|---------|
   |int getNumberOfDocumentsImported()  |   在提供給大量匯入 API 呼叫的文件中，成功匯入的文件總數。      |
   |double getTotalRequestUnitsConsumed()   |  大量匯入 API 呼叫取用的要求單位 (RU) 總數。       |
   |Duration getTotalTimeTaken()   |    大量匯入 API 呼叫完成執行的時間總計。     |
   |List<Exception> getErrors() |  如果提供給大量匯入 API 呼叫的批次中有部分文件無法插入，則會取得錯誤清單。       |
   |List<Object> getBadInputDocuments()  |    格式錯誤而未成功匯入大量匯入 API 呼叫的文件清單。 使用者應該修正傳回的文件，然後再次嘗試匯入。 格式錯誤的文件包含其識別碼值不是字串 (Null 或任何其他視為無效的資料類型) 的文件。     |

5. 大量匯入應用程式已備妥之後，請使用 'mvn clean package' 命令從來源建置命令列工具。 此命令會在目標資料夾中產生 jar 檔案：  

   ```java
   mvn clean package
   ```

6. 產生的目標相依性之後，您可以使用下列命令來叫用大量匯入工具應用程式：  

   ```java
   java -Xmx12G -jar bulkexecutor-sample-1.0-SNAPSHOT-jar-with-dependencies.jar -serviceEndpoint *<Fill in your Azure Cosmos DB’s endpoint URI>*  -masterKey *<Fill in your Azure Cosmos DB’s master key>* -databaseId bulkImportDb -collectionId bulkImportColl -operation import -shouldCreateCollection -collectionThroughput 1000000 -partitionKey /profileid -maxConnectionPoolSize 6000 -numberOfDocumentsForEachCheckpoint 1000000 -numberOfCheckpoints 10
   ```

   大量匯入工具會使用資料庫名稱、集合名稱和 App.config 檔案中指定的輸送量值來建立新的資料庫與集合。 

## <a name="bulk-update-data-in-azure-cosmos-db"></a>在 Azure Cosmos DB 中大量更新資料

您可以使用 BulkUpdateAsync API 來更新現有的文件。 在此範例中，您會將 [名稱] 欄位設為新的值，並移除現有文件中的 [描述] 欄位。 如需完整的支援欄位更新作業集，請參閱 [API 文件](https://docs.microsoft.com/java/api/com.microsoft.azure.documentdb.bulkexecutor)。 

1. 與相對應的欄位更新作業一起定義及更新項目。 在此範例中，您將使用 SetUpdateOperation 更新 [名稱] 欄位，以及使用 UnsetUpdateOperation 從所有文件中移除 [描述] 欄位。 您也可以執行其他作業，例如透過指定值來遞增文件欄位、將特定值推送至陣列欄位中，或是從陣列欄位中移除特定值。 若要了解大量更新 API 提供的不同方法，請參閱 [API 文件](https://docs.microsoft.com/java/api/com.microsoft.azure.documentdb.bulkexecutor)。  

   ```java
   SetUpdateOperation<String> nameUpdate = new SetUpdateOperation<>("Name","UpdatedDocValue");
   UnsetUpdateOperation descriptionUpdate = new UnsetUpdateOperation("description");

   ArrayList<UpdateOperationBase> updateOperations = new ArrayList<>();
   updateOperations.add(nameUpdate);
   updateOperations.add(descriptionUpdate);

   List<UpdateItem> updateItems = new ArrayList<>(cfg.getNumberOfDocumentsForEachCheckpoint());
   IntStream.range(0, cfg.getNumberOfDocumentsForEachCheckpoint()).mapToObj(j -> {                      
    return new UpdateItem(Long.toString(prefix + j), Long.toString(prefix + j), updateOperations);
    }).collect(Collectors.toCollection(() -> updateItems));
   ```

2. 呼叫 updateAll API，以產生隨機文件來大量匯入至 Azure Cosmos DB 容器。 您可以設定要傳遞到 CmdLineConfiguration.java 檔案內的命令列組態。

   ```java
   BulkUpdateResponse bulkUpdateResponse = bulkExecutor.updateAll(updateItems, null)
   ```

   大量更新 API 會接受要更新的項目集合。 每個更新項目會指定要在文件上執行的欄位更新作業清單，而文件會由識別碼和分割區索引鍵定義。 如需詳細資料，請參閱 [API 文件](https://docs.microsoft.com/java/api/com.microsoft.azure.documentdb.bulkexecutor)：

   ```java
   public BulkUpdateResponse updateAll(
        Collection<UpdateItem> updateItems,
        Integer maxConcurrencyPerPartitionRange) throws DocumentClientException;
   ```

   updateAll 方法可接受下列參數：

   |**參數** |**說明** |
   |---------|---------|
   |maxConcurrencyPerPartitionRange   |  每個分割區索引鍵範圍的最大並行程度。 預設值為 20。  |
 
   **大量匯入回應物件定義**大量匯入 API 呼叫的結果包含下列 get 方法：

   |**參數** |**說明**  |
   |---------|---------|
   |int getNumberOfDocumentsUpdated()  |   在提供給大量更新 API 呼叫的文件中，成功更新的文件總數。      |
   |double getTotalRequestUnitsConsumed() |  大量更新 API 呼叫取用的要求單位 (RU) 總數。       |
   |Duration getTotalTimeTaken()  |   大量更新 API 呼叫完成執行的時間總計。      |
   |List<Exception> getErrors()   |     如果提供給大量更新 API 呼叫的批次中有部分文件無法插入，則會取得錯誤清單。      |

3. 大量更新應用程式已備妥之後，請使用 'mvn clean package' 命令從來源建置命令列工具。 此命令會在目標資料夾中產生 jar 檔案：  

   ```
   mvn clean package
   ```

4. 產生的目標相依性之後，您可以使用下列命令來叫用大量更新應用程式：

   ```
   java -Xmx12G -jar bulkexecutor-sample-1.0-SNAPSHOT-jar-with-dependencies.jar -serviceEndpoint **<Fill in your Azure Cosmos DB’s endpoint URI>* -masterKey **<Fill in your Azure Cosmos DB’s master key>* -databaseId bulkUpdateDb -collectionId bulkUpdateColl -operation update -collectionThroughput 1000000 -partitionKey /profileid -maxConnectionPoolSize 6000 -numberOfDocumentsForEachCheckpoint 1000000 -numberOfCheckpoints 10
   ```

## <a name="performance-tips"></a>效能秘訣 

使用大量執行程式程式庫時，請考量下列各點以提升效能：

* 為達到最佳效能，請從位於與 Cosmos DB 帳戶寫入區域相同區域的 Azure VM 中執行應用程式。  
* 為達到更高的輸送量：  

   * 將 JVM 堆積大小設定為夠大的數字，以避免處理大量文件時發生任何記憶體問題。 建議的堆積大小：max(3GB, 3 * sizeof(在一個批次中傳遞至大量輸入 API 的所有文件))。  
   * 具有前置處理時間，在您對大量文件執行大量作業時，這可讓您取得更高的輸送量。 因此，如果您想要匯入 10,000,000 份文件，較好的方式是對 10 份大量文件 (每份中有 1,000,000 份文件) 執行 10 次大量匯入，而不是對 100 份大量文件 (每份中有 100,000 份文件) 執行 100 次大量匯入。  

* 在對應到特定 Azure Cosmos DB 容器的單一虛擬機器中，建議您為整個應用程式具現化單一 DocumentBulkExecutor 物件。  

* 單一大量作業 API 執行會取用大量用戶端機器的 CPU 和網路 IO。 這是因為由內部繁衍出多個工作，因此請避免在每次執行大量作業 API 呼叫時，您的應用程式處理程序內繁衍出多個並行工作。 如果在單一虛擬機器上執行的單一大量作業 API 呼叫無法取用整個容器的輸送量 (如果容器的輸送量 > 1 百萬 RU/s)，建議您建立個別虛擬機器來並行執行大量作業 API 呼叫。

    
## <a name="next-steps"></a>後續步驟
* 若要深入了解 maven 套件及大量執行程式 Java 程式庫的版本資訊，請參閱[大量執行程式 SDK 詳細資料](sql-api-sdk-bulk-executor-java.md)。


