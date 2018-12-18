---
title: 以 .NET 開始使用 Azure 佇列儲存體 | Microsoft Docs
description: Azure 佇列可在應用程式元件之間提供可靠的非同步傳訊。 雲端傳訊可讓您的應用程式元件獨立擴充。
services: storage
author: tamram
ms.service: storage
ms.devlang: dotnet
ms.topic: hero-article
ms.date: 06/13/2018
ms.author: tamram
ms.component: queues
ms.openlocfilehash: 11b43037a16072bcb2056b70305eeef9d3b5d4ce
ms.sourcegitcommit: 1b561b77aa080416b094b6f41fce5b6a4721e7d5
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/17/2018
ms.locfileid: "45729003"
---
# <a name="get-started-with-azure-queue-storage-using-net"></a>以 .NET 開始使用 Azure 佇列儲存體
[!INCLUDE [storage-selector-queue-include](../../../includes/storage-selector-queue-include.md)]

[!INCLUDE [storage-check-out-samples-dotnet](../../../includes/storage-check-out-samples-dotnet.md)]

## <a name="overview"></a>概觀
Azure 佇列儲存體可提供應用程式元件之間的雲端傳訊。 設計擴充性的應用程式時，會經常分離應用程式元件，以便進行個別擴充。 佇列儲存體可針對應用程式元件間的通訊，提供非同步傳訊，無論應用程式元件是在雲端、桌面、內部部署伺服器或行動裝置上執行。 佇列儲存體也支援管理非同步工作並建置處理工作流程。

### <a name="about-this-tutorial"></a>關於本教學課程
本教學課程說明如何使用 Azure 佇列儲存體撰寫一些常見案例的 .NET 程式碼。 本文說明的案例包括建立和刪除佇列，以及新增、讀取和刪除佇列訊息。

**預估完成時間：** 45 分鐘

**先決條件：**

* [Microsoft Visual Studio](https://www.visualstudio.com/downloads/)
* [適用於 .NET 的 Azure 儲存體用戶端程式庫](https://www.nuget.org/packages/WindowsAzure.Storage/)
* [適用於.NET 的 Azure 設定管理員](https://www.nuget.org/packages/Microsoft.WindowsAzure.ConfigurationManager/)
* [Azure 儲存體帳戶](../common/storage-quickstart-create-account.md)

[!INCLUDE [storage-queue-concepts-include](../../../includes/storage-queue-concepts-include.md)]

[!INCLUDE [storage-create-account-include](../../../includes/storage-create-account-include.md)]

[!INCLUDE [storage-development-environment-include](../../../includes/storage-development-environment-include.md)]

### <a name="add-using-directives"></a>新增 using 指示詞
將下列 `using` 指示詞新增至 `Program.cs` 檔案的開頭處：

```csharp
using Microsoft.Azure; // Namespace for CloudConfigurationManager
using Microsoft.WindowsAzure.Storage; // Namespace for CloudStorageAccount
using Microsoft.WindowsAzure.Storage.Queue; // Namespace for Queue storage types
```

### <a name="copy-your-credentials-from-the-azure-portal"></a>從 Azure 入口網站複製您的認證

範例程式碼需要授與對您儲存體帳戶的存取權。 若要進行授權，您必須以連接字串的形式為應用程式提供您的儲存體帳戶認證。 若要檢視您的儲存體帳戶認證：

1. 瀏覽至 [Azure 入口網站](https://portal.azure.com)。
2. 找出您的儲存體帳戶。
3. 在儲存體帳戶概觀的 [設定] 區段中，選取 [存取金鑰]。 您的帳戶存取金鑰隨即出現，此外也會顯示每個金鑰的完整連接字串。   
4. 尋找 [金鑰1] 下方的 [連接字串] 值，然後按一下 [複製] 按鈕來複製連接字串。 在下一個步驟中，您會將連接字串值新增至環境變數。

    ![顯示如何從 Azure 入口網站複製連接字串的螢幕擷取畫面](media/storage-dotnet-how-to-use-queues/portal-connection-string.png)

### <a name="parse-the-connection-string"></a>解析連接字串
[!INCLUDE [storage-cloud-configuration-manager-include](../../../includes/storage-cloud-configuration-manager-include.md)]

### <a name="create-the-queue-service-client"></a>建立佇列服務用戶端
**CloudQueueClient** 類別可讓您擷取佇列儲存體中儲存的佇列。 以下是建立服務用戶端的其中一種方式：

```csharp
CloudQueueClient queueClient = storageAccount.CreateCloudQueueClient();
```
    
您現在可以開始撰寫程式碼，以讀取佇列儲存體的資料並將資料寫入其中。

## <a name="create-a-queue"></a>建立佇列
此範例說明如何建立尚不存在的佇列：

```csharp
// Retrieve storage account from connection string.
CloudStorageAccount storageAccount = CloudStorageAccount.Parse(
    CloudConfigurationManager.GetSetting("StorageConnectionString"));

// Create the queue client.
CloudQueueClient queueClient = storageAccount.CreateCloudQueueClient();

// Retrieve a reference to a container.
CloudQueue queue = queueClient.GetQueueReference("myqueue");

// Create the queue if it doesn't already exist
queue.CreateIfNotExists();
```

## <a name="insert-a-message-into-a-queue"></a>將訊息插入佇列
若要將訊息插入現有佇列，請先建立新的 **CloudQueueMessage**。 接著，呼叫 **AddMessage** 方法。 您可以從字串 (採用 UTF-8 格式) 或**位元組**陣列建立 **CloudQueueMessage**。 以下是建立佇列 (如果佇列不存在) 並插入訊息 'Hello, World' 的程式碼：

```csharp
// Retrieve storage account from connection string.
CloudStorageAccount storageAccount = CloudStorageAccount.Parse(
    CloudConfigurationManager.GetSetting("StorageConnectionString"));

// Create the queue client.
CloudQueueClient queueClient = storageAccount.CreateCloudQueueClient();

// Retrieve a reference to a queue.
CloudQueue queue = queueClient.GetQueueReference("myqueue");

// Create the queue if it doesn't already exist.
queue.CreateIfNotExists();

// Create a message and add it to the queue.
CloudQueueMessage message = new CloudQueueMessage("Hello, World");
queue.AddMessage(message);
```

## <a name="peek-at-the-next-message"></a>查看下一個訊息
透過呼叫 **PeekMessage** 方法，您可以在佇列前面查看訊息，而無需將它從佇列中移除。

```csharp
// Retrieve storage account from connection string
CloudStorageAccount storageAccount = CloudStorageAccount.Parse(
    CloudConfigurationManager.GetSetting("StorageConnectionString"));

// Create the queue client
CloudQueueClient queueClient = storageAccount.CreateCloudQueueClient();

// Retrieve a reference to a queue
CloudQueue queue = queueClient.GetQueueReference("myqueue");

// Peek at the next message
CloudQueueMessage peekedMessage = queue.PeekMessage();

// Display message.
Console.WriteLine(peekedMessage.AsString);
```

## <a name="change-the-contents-of-a-queued-message"></a>變更佇列訊息的內容
您可以在佇列中就地變更訊息內容。 如果訊息代表工作作業，則您可以使用此功能來更新工作作業的狀態。 下列程式碼將使用新的內容更新佇列訊息，並將可見度逾時設定延長 60 秒。 這可儲存與訊息相關的工作狀態，並提供用戶端多一分鐘的時間繼續處理訊息。 您可以使用此技巧來追蹤佇列訊息上的多步驟工作流程，如果因為硬體或軟體故障而導致某個處理步驟失敗，將無需從頭開始。 通常，您也會保留重試計數，如果訊息重試超過 *n* 次，您會將它刪除。 這麼做可防止每次處理時便觸發應用程式錯誤的訊息。

```csharp
// Retrieve storage account from connection string.
CloudStorageAccount storageAccount = CloudStorageAccount.Parse(
    CloudConfigurationManager.GetSetting("StorageConnectionString"));

// Create the queue client.
CloudQueueClient queueClient = storageAccount.CreateCloudQueueClient();

// Retrieve a reference to a queue.
CloudQueue queue = queueClient.GetQueueReference("myqueue");

// Get the message from the queue and update the message contents.
CloudQueueMessage message = queue.GetMessage();
message.SetMessageContent("Updated contents.");
queue.UpdateMessage(message,
    TimeSpan.FromSeconds(60.0),  // Make it invisible for another 60 seconds.
    MessageUpdateFields.Content | MessageUpdateFields.Visibility);
```

## <a name="de-queue-the-next-message"></a>將下一個訊息清除佇列
您的程式碼可以使用兩個步驟將訊息自佇列中清除佇列。 呼叫 **GetMessage**時，您會取得佇列中的下一個訊息。 從 **GetMessage** 傳回的訊息，對於從此佇列讀取訊息的任何其他程式碼而言將會是不可見的。 依預設，此訊息會維持 30 秒的不可見狀態。 若要完成從佇列中移除訊息，您還必須呼叫 **DeleteMessage**。 這個移除訊息的兩步驟程序可確保您的程式碼因為硬體或軟體故障而無法處理訊息時，另一個程式碼的執行個體可以取得相同訊息並再試一次。 您的程式碼會在處理完訊息之後立即呼叫 **DeleteMessage** 。

```csharp
// Retrieve storage account from connection string
CloudStorageAccount storageAccount = CloudStorageAccount.Parse(
    CloudConfigurationManager.GetSetting("StorageConnectionString"));

// Create the queue client
CloudQueueClient queueClient = storageAccount.CreateCloudQueueClient();

// Retrieve a reference to a queue
CloudQueue queue = queueClient.GetQueueReference("myqueue");

// Get the next message
CloudQueueMessage retrievedMessage = queue.GetMessage();

//Process the message in less than 30 seconds, and then delete the message
queue.DeleteMessage(retrievedMessage);
```

## <a name="use-async-await-pattern-with-common-queue-storage-apis"></a>搭配通用佇列儲存體 API 使用 Async-Await 模式
這個範例示範如何搭配通用佇列儲存體 API 使用 Async-Await 模式。 此範例會呼叫每個指定方法的非同步版本，就像每個方法的非同步 *Async* 尾碼所指示的一樣。 使用非同步方法時，async-await 模式會暫停本機執行，直到呼叫完成為止。 這種行為可讓目前的執行緒執行其他工作，有助於避免發生效能瓶頸並提升應用程式的整體回應。 如需在 .NET 中使用 Async-Await 模式的詳細資訊，請參閱 [Async 和 Await (C# 和 Visual Basic)](https://msdn.microsoft.com/library/hh191443.aspx)

```csharp
// Create the queue if it doesn't already exist
if(await queue.CreateIfNotExistsAsync())
{
    Console.WriteLine("Queue '{0}' Created", queue.Name);
}
else
{
    Console.WriteLine("Queue '{0}' Exists", queue.Name);
}

// Create a message to put in the queue
CloudQueueMessage cloudQueueMessage = new CloudQueueMessage("My message");

// Async enqueue the message
await queue.AddMessageAsync(cloudQueueMessage);
Console.WriteLine("Message added");

// Async dequeue the message
CloudQueueMessage retrievedMessage = await queue.GetMessageAsync();
Console.WriteLine("Retrieved message with content '{0}'", retrievedMessage.AsString);

// Async delete the message
await queue.DeleteMessageAsync(retrievedMessage);
Console.WriteLine("Deleted message");
```
    
## <a name="leverage-additional-options-for-de-queuing-messages"></a>運用清除佇列訊息的其他選項
自訂從佇列中擷取訊息的方法有兩種。
首先，您可以取得一批訊息 (最多 32 個)。 其次，您可以設定較長或較短的可見度逾時，讓您的程式碼有較長或較短的時間可以完全處理每個訊息。 下列程式碼範例將使用 **GetMessages** 方法，在一次呼叫中取得 20 個訊息。 接著它會使用 **foreach** 迴圈處理每個訊息。 它也會將可見度逾時設定為每個訊息五分鐘。 請注意，系統會針對所有訊息同時開始計時 5 分鐘，所以從呼叫 **GetMessages**開始的 5 分鐘後，任何尚未刪除的訊息都會重新出現。

```csharp
// Retrieve storage account from connection string.
CloudStorageAccount storageAccount = CloudStorageAccount.Parse(
    CloudConfigurationManager.GetSetting("StorageConnectionString"));

// Create the queue client.
CloudQueueClient queueClient = storageAccount.CreateCloudQueueClient();

// Retrieve a reference to a queue.
CloudQueue queue = queueClient.GetQueueReference("myqueue");

foreach (CloudQueueMessage message in queue.GetMessages(20, TimeSpan.FromMinutes(5)))
{
    // Process all messages in less than 5 minutes, deleting each message after processing.
    queue.DeleteMessage(message);
}
```

## <a name="get-the-queue-length"></a>取得佇列長度
您可以取得佇列中的估計訊息數目。 **FetchAttributes** 方法會要求佇列服務擷取佇列屬性，其中包含訊息計數。 **ApproximateMessageCount** 屬性會傳回 **FetchAttributes** 方法所擷取的最後一個值，而無需呼叫佇列服務。

```csharp
// Retrieve storage account from connection string.
CloudStorageAccount storageAccount = CloudStorageAccount.Parse(
    CloudConfigurationManager.GetSetting("StorageConnectionString"));

// Create the queue client.
CloudQueueClient queueClient = storageAccount.CreateCloudQueueClient();

// Retrieve a reference to a queue.
CloudQueue queue = queueClient.GetQueueReference("myqueue");

// Fetch the queue attributes.
queue.FetchAttributes();

// Retrieve the cached approximate message count.
int? cachedMessageCount = queue.ApproximateMessageCount;

// Display number of messages.
Console.WriteLine("Number of messages in queue: " + cachedMessageCount);
```

## <a name="delete-a-queue"></a>刪除佇列
若要刪除佇列及其內含的所有訊息，請在佇列物件上呼叫 **Delete** 方法。

```csharp
// Retrieve storage account from connection string.
CloudStorageAccount storageAccount = CloudStorageAccount.Parse(
    CloudConfigurationManager.GetSetting("StorageConnectionString"));

// Create the queue client.
CloudQueueClient queueClient = storageAccount.CreateCloudQueueClient();

// Retrieve a reference to a queue.
CloudQueue queue = queueClient.GetQueueReference("myqueue");

// Delete the queue.
queue.Delete();
```
    

## <a name="next-steps"></a>後續步驟
了解佇列儲存體的基礎概念之後，請參考下列連結以了解有關更複雜的儲存工作。

* 如需可用 API 的完整詳細資訊，請檢視佇列服務參考文件：
  * [Storage Client Library for .NET 參考資料](http://go.microsoft.com/fwlink/?LinkID=390731&clcid=0x409)
  * [REST API 參考資料](http://msdn.microsoft.com/library/azure/dd179355)
* 了解如何使用 [Azure WebJobs SDK](https://github.com/Azure/azure-webjobs-sdk/wiki)，來簡化您撰寫以使用 Azure 儲存體的程式碼。
* 如需了解 Azure 中的其他資料儲存選項，請檢視更多功能指南。
  * [以 .NET 開始使用 Azure 表格儲存體](../../cosmos-db/table-storage-how-to-use-dotnet.md) 以儲存結構化資料。
  * [以 .NET 開始使用 Azure Blob 儲存體](../blobs/storage-dotnet-how-to-use-blobs.md) 以儲存非結構化資料。
  * [使用 .NET (C#) 連接到 SQL Database ](../../sql-database/sql-database-connect-query-dotnet-core.md) 以儲存關聯式資料。

[Download and install the Azure SDK for .NET]: /develop/net/
[.NET client library reference]: http://go.microsoft.com/fwlink/?LinkID=390731&clcid=0x409
[Creating a Azure Project in Visual Studio]: http://msdn.microsoft.com/library/azure/ee405487.aspx
[Azure Storage Team Blog]: http://blogs.msdn.com/b/windowsazurestorage/
[OData]: http://nuget.org/packages/Microsoft.Data.OData/5.0.2
[Edm]: http://nuget.org/packages/Microsoft.Data.Edm/5.0.2
[Spatial]: http://nuget.org/packages/System.Spatial/5.0.2
