---
title: 在 Azure Cosmos DB 中使用大量執行程式 .NET 程式庫執行大量作業 | Microsoft Docs
description: 使用 Azure Cosmos DB 的大量執行程式 .NET 程式庫將文件大量匯入並更新至 Azure Cosmos DB 容器。
keywords: .Net 大量執行程式
services: cosmos-db
author: tknandu
manager: kfile
ms.service: cosmos-db
ms.devlang: dotnet
ms.topic: conceptual
ms.date: 05/07/2018
ms.author: ramkris
ms.openlocfilehash: cc0faa44501ea130309a02bb48d02f9c5b33febd
ms.sourcegitcommit: ebd06cee3e78674ba9e6764ddc889fc5948060c4
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/07/2018
ms.locfileid: "44053375"
---
# <a name="use-bulk-executor-net-library-to-perform-bulk-operations-in-azure-cosmos-db"></a>在 Azure Cosmos DB 中使用大量執行程式 .NET 程式庫執行大量作業

本教學課程說明如何使用 Azure Cosmos DB 的大量執行程式 .NET 程式庫將文件匯入並更新至 Azure Cosmos DB 容器。 若要深入了解大量執行程式程式庫，以及它如何協助您利用大量輸送量與儲存體，請參閱[大量執行程式程式庫概觀](bulk-executor-overview.md)一文。 本教學課程將引導您使用 .NET 應用程式範例，將隨機產生的文件大量匯入至 Azure Cosmos DB 容器。 匯入之後，應用程式會說明如何將修補程式指定為可在特定文件欄位上執行的作業，來大量更新匯入的資料。

## <a name="prerequisites"></a>必要條件

* 如果尚未安裝 Visual Studio 2017，您可以下載並使用 [Visual Studio 2017 Community 版本](https://www.visualstudio.com/downloads/)。 務必在 Visual Studio 設定期間啟用 Azure 開發。

* 如果您沒有 Azure 訂用帳戶，請在開始前建立 [免費帳戶](https://azure.microsoft.com/free/?ref=microsoft.com&utm_source=microsoft.com&utm_medium=docs&utm_campaign=visualstudio)。 

* 您可以[免費試用 Azure Cosmos DB](https://azure.microsoft.com/try/cosmosdb/)，無須 Azure 訂用帳戶，也無須任何費用和約定付款。 或者，您也可以搭配使用 [Azure Cosmos DB 模擬器](https://docs.microsoft.com/azure/cosmos-db/local-emulator)與 `https://localhost:8081` URI。 [驗證要求](local-emulator.md#authenticating-requests)中會提供主索引鍵。

* 使用 .NET 快速入門文章中＜[建立資料庫帳戶](create-sql-api-dotnet.md#create-a-database-account)＞一節所述的步驟，建立 Azure Cosmos DB SQL API 帳戶。 

## <a name="clone-the-sample-application"></a>複製範例應用程式

現在我們會從 GitHub 下載一些 .NET 應用程式範例，來轉換為使用程式碼。 這些應用程式會對 Azure Cosmos DB 資料執行大量作業。 若要複製應用程式，請開啟命令提示字元，瀏覽至要從其中複製應用程式的目錄，然後執行下列命令：

```
git clone https://github.com/Azure/azure-cosmosdb-bulkexecutor-dotnet-getting-started.git
```

複製的存放庫會包含兩個範例 "BulkImportSample" 和 "BulkUpdateSample"。 您可以開啟其中一個應用程式範例，使用 Azure Cosmos DB 帳戶的連接字串更新 App.config 檔案中的連接字串，然後建置解決方案並加以執行。 

"BulkImportSample" 應用程式會產生隨機文件，並將其大量匯入至 Azure Cosmos DB。 "BulkUpdateSample" 應用程式會將修補程式指定為可在特定文件欄位上執行的作業，來大量更新匯入的文件。 在後面幾節中，您將檢閱這裡每一個範例應用程式中的程式碼。

## <a name="bulk-import-data-to-azure-cosmos-db"></a>將資料大量匯入至 Azure Cosmos DB

1. 瀏覽至 "BulkImportSample" 資料夾，然後開啟 "BulkImportSample.sln" 檔案。  

2. Azure Cosmos DB 的連接字串會從 App.config 檔案中擷取，如下列程式碼所示：  

   ```csharp
   private static readonly string EndpointUrl = ConfigurationManager.AppSettings["EndPointUrl"];
   private static readonly string AuthorizationKey = ConfigurationManager.AppSettings["AuthorizationKey"];
   private static readonly string DatabaseName = ConfigurationManager.AppSettings["DatabaseName"];
   private static readonly string CollectionName = ConfigurationManager.AppSettings["CollectionName"];
   private static readonly int CollectionThroughput = int.Parse(ConfigurationManager.AppSettings["CollectionThroughput"]);
   ```

   大量匯入工具會使用資料庫名稱、集合名稱和 App.config 檔案中指定的輸送量值來建立新的資料庫與集合。 

3. 接下來，DocumentClient 物件會使用直接 TCP 連線模式進行初始化：  

   ```csharp
   ConnectionPolicy connectionPolicy = new ConnectionPolicy
   {
      ConnectionMode = ConnectionMode.Direct,
      ConnectionProtocol = Protocol.Tcp
   };
   DocumentClient client = new DocumentClient(new Uri(endpointUrl),authorizationKey,
   connectionPolicy)
   ```

4. 針對等候時間和節流的要求，BulkExecutor 物件會使用高 retry 值進行初始化。 接著，這些值會設定為 0，如此會將壅塞控制傳遞給 BulkExecutor 以決定其存留期。  

   ```csharp
   // Set retry options high during initialization (default values).
   client.ConnectionPolicy.RetryOptions.MaxRetryWaitTimeInSeconds = 30;
   client.ConnectionPolicy.RetryOptions.MaxRetryAttemptsOnThrottledRequests = 9;

   IBulkExecutor bulkExecutor = new BulkExecutor(client, dataCollection);
   await bulkExecutor.InitializeAsync();

   // Set retries to 0 to pass complete control to bulk executor.
   client.ConnectionPolicy.RetryOptions.MaxRetryWaitTimeInSeconds = 0;
   client.ConnectionPolicy.RetryOptions.MaxRetryAttemptsOnThrottledRequests = 0;
   ```

5. 應用程式會叫用 BulkImportAsync API。 .NET 程式庫會提供兩個大量匯入 API 的多載 - 一個接受一份已序列化的 JSON 文件清單，而另一個接受已還原序列化的 POCO 文件清單。 若要了解這裡每個多載方法的定義，請參閱 [API 文件](https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmosdb.bulkexecutor.bulkexecutor.bulkimportasync?view=azure-dotnet)。

   ```csharp
   BulkImportResponse bulkImportResponse = await bulkExecutor.BulkImportAsync(
     documents: documentsToImportInBatch,
     enableUpsert: true,
     disableAutomaticIdGeneration: true,
     maxConcurrencyPerPartitionKeyRange: null,
     maxInMemorySortingBatchSize: null,
     cancellationToken: token);
   ```
   **BulkImportAsync 方法可接受下列參數：**
   
   |**參數**  |**說明** |
   |---------|---------|
   |enableUpsert    |   啟用文件更新插入的旗標。 如果具有指定識別碼的文件已經存在，文件會進行更新。 預設會設定為 false。      |
   |disableAutomaticIdGeneration    |    停用自動產生識別碼的旗標。 預設會設定為 true。     |
   |maxConcurrencyPerPartitionKeyRange    | 每個資料分割索引鍵範圍的最大並行程度，若設定為 Null，程式庫會使用預設值 20。 |
   |maxInMemorySortingBatchSize     |  從文件列舉程式提取的最大文件數，也就是在每個階段中傳遞至 API 呼叫的最大文件數。  針對大量匯入之前的記憶體內前置處理排序階段，若設定為 Null，程式庫將會使用預設值 min(documents.count, 1000000)。       |
   |cancellationToken    |    用於依正常程序結束大量匯入的取消權杖。     |

   **大量匯入回應物件定義**大量匯入 API 呼叫的結果包含下列屬性：

   |**參數**  |**說明**  |
   |---------|---------|
   |NumberOfDocumentsImported (long)   |  在提供給大量匯入 API 呼叫的文件中，成功匯入的文件總數。       |
   |TotalRequestUnitsConsumed (double)   |   大量匯入 API 呼叫取用的要求單位 (RU) 總數。      |
   |TotalTimeTaken (TimeSpan)    |   大量匯入 API 呼叫完成執行的時間總計。      |
   |BadInputDocuments (List<object>)   |     格式錯誤而未成功匯入大量匯入 API 呼叫的文件清單。 使用者應該修正傳回的文件，然後再次嘗試匯入。 格式錯誤的文件包含其識別碼值不是字串 (Null 或任何其他視為無效的資料類型) 的文件。    |

## <a name="bulk-update-data-in-azure-cosmos-db"></a>在 Azure Cosmos DB 中大量更新資料

您可以使用 BulkUpdateAsync API 來更新現有的文件。 在此範例中，您會將 [名稱] 欄位設為新的值，並移除現有文件中的 [描述] 欄位。 如需完整的支援欄位更新作業集，請參閱 [API 文件](https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmosdb.bulkexecutor.bulkupdate?view=azure-dotnet)。 

1. 瀏覽至 "BulkUpdateSample" 資料夾，然後開啟 "BulkUpdateSample.sln" 檔案。  

2. 與相對應的欄位更新作業一起定義及更新項目。 在此範例中，您將使用 SetUpdateOperation 更新 [名稱] 欄位，以及使用 UnsetUpdateOperation 從所有文件中移除 [描述] 欄位。 您也可以執行其他作業，例如透過指定值來遞增文件欄位、將特定值推送至陣列欄位中，或是從陣列欄位中移除特定值。 若要了解大量更新 API 提供的不同方法，請參閱 [API 文件](https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmosdb.bulkexecutor.bulkupdate?view=azure-dotnet)。

   ```csharp
   SetUpdateOperation<string> nameUpdate = new SetUpdateOperation<string>("Name", "UpdatedDoc");
   UnsetUpdateOperation descriptionUpdate = new UnsetUpdateOperation("description");

   List<UpdateOperation> updateOperations = new List<UpdateOperation>();
   updateOperations.Add(nameUpdate);
   updateOperations.Add(descriptionUpdate);

   List<UpdateItem> updateItems = new List<UpdateItem>();
   for (int i = 0; i < 10; i++)
   {
    updateItems.Add(new UpdateItem(i.ToString(), i.ToString(), updateOperations));
   }
   ```

3. 應用程式會叫用 BulkUpdateAsync API。 若要了解 BulkUpdateAsync 方法的定義，請參閱 [API 文件](https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmosdb.bulkexecutor.ibulkexecutor.bulkupdateasync?view=azure-dotnet)。  

   ```csharp
   BulkUpdateResponse bulkUpdateResponse = await bulkExecutor.BulkUpdateAsync(
     updateItems: updateItems,
     maxConcurrencyPerPartitionKeyRange: null,
     maxInMemorySortingBatchSize: null,
     cancellationToken: token);
   ```  
   **BulkUpdateAsync 方法可接受下列參數：**

   |**參數**  |**說明** |
   |---------|---------|
   |maxConcurrencyPerPartitionKeyRange    |   每個資料分割索引鍵範圍的最大並行程度，若設定為 Null，程式庫會使用預設值 20。   |
   |maxInMemorySortingBatchSize    |    從更新項目列舉程式提取的最大更新項目數，也就是在每個階段中傳遞至 API 呼叫的最大更新項目數，針對大量更新之前的記憶體內前置處理排序階段，若設定為 Null，程式庫將會使用預設值 min(updateItems.count, 1000000)。     |
   | cancellationToken|用於依正常程序結束大量更新的取消權杖。 |

   **大量更新回應物件定義**大量更新 API 呼叫的結果包含下列屬性：

   |**參數**  |**說明** |
   |---------|---------|
   |NumberOfDocumentsUpdated (long)    |   在提供給大量更新 API 呼叫的文件中，成功更新的文件總數。      |
   |TotalRequestUnitsConsumed (double)   |    大量更新 API 呼叫取用的要求單位 (RU) 總數。    |
   |TotalTimeTaken (TimeSpan)   | 大量更新 API 呼叫完成執行的時間總計。 |
    
## <a name="performance-tips"></a>效能秘訣 

使用大量執行程式程式庫時，請考量下列各點以提升效能：

* 為達到最佳效能，請從位於與 Cosmos DB 帳戶寫入區域相同區域的 Azure 虛擬機器中執行應用程式。  

* 在對應到特定 Cosmos DB 容器的單一虛擬機器中，建議您為整個應用程式具現化單一 BulkExecutor 物件。  

* 單一大量作業 API 執行會取用大量用戶端機器的 CPU 和網路 IO。 這是因為由內部繁衍出多個工作，因此請避免在每次執行大量作業 API 呼叫時，您的應用程式處理程序內繁衍出多個並行工作。 如果在單一虛擬機器上執行的單一大量作業 API 呼叫無法取用整個容器的輸送量 (如果容器的輸送量 > 1 百萬 RU/s)，建議您建立個別虛擬機器來並行執行大量作業 API 呼叫。  

* 請確定 InitializeAsync() 是在具現化 BulkExecutor 物件之後叫用，以便提取目標 Cosmos DB 容器資料分割對應。  

* 在應用程式的 App.Config 中，為擁有最佳效能，請確保已啟用 **gcServer**
  ```xml  
  <runtime>
    <gcServer enabled="true" />
  </runtime>
  ```
* 程式庫會發出可收集到記錄檔或在主控台的追蹤。 若要兩者都啟用，請將下列程式碼新增至應用程式的 App.Config。

  ```xml
  <system.diagnostics>
    <trace autoflush="false" indentsize="4">
      <listeners>
        <add name="logListener" type="System.Diagnostics.TextWriterTraceListener" initializeData="application.log" />
        <add name="consoleListener" type="System.Diagnostics.ConsoleTraceListener" />
      </listeners>
    </trace>
  </system.diagnostics>
```

## <a name="next-steps"></a>後續步驟
* 若要深入了解 Nuget 套件及大量執行程式 .Net 程式庫的版本資訊，請參閱[大量執行程式 SDK 詳細資料](sql-api-sdk-bulk-executor-dot-net.md)。 
