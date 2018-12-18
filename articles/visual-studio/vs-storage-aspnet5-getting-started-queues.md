---
title: 開始使用佇列儲存體和 Visual Studio 已連接服務 (ASP.NET Core) | Microsoft Docs
description: 如何開始在 Visual Studio 的 ASP.NET Core 專案中使用 Azure 佇列儲存體
services: storage
author: ghogen
manager: douge
ms.assetid: 04977069-5b2d-4cba-84ae-9fb2f5eb1006
ms.prod: visual-studio-dev15
ms.technology: vs-azure
ms.custom: vs-azure
ms.workload: azure-vs
ms.topic: article
ms.date: 11/14/2017
ms.author: ghogen
ms.openlocfilehash: 237294c1bc603402c349f7a56f20c34ed8d210fe
ms.sourcegitcommit: 30c7f9994cf6fcdfb580616ea8d6d251364c0cd1
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/18/2018
ms.locfileid: "42145754"
---
# <a name="get-started-with-queue-storage-and-visual-studio-connected-services-aspnet-core"></a>開始使用佇列儲存體和 Visual Studio 已連接服務 (ASP.NET Core)

[!INCLUDE [storage-try-azure-tools-queues](../../includes/storage-try-azure-tools-queues.md)]

本文說明如何在您使用 Visual Studio 的**已連線的服務**功能建立或參考 ASP.NET Core 專案中的 Azure 儲存體帳戶之後，開始在 Visual Studio 使用 Azure 佇列儲存體。 **已連線的服務**作業會安裝適當的 NuGet 套件，以存取專案中的 Azure 儲存體，並將儲存體帳戶的連接字串新增至您的專案設定檔。 (如需 Azure 儲存體的一般資訊，請參閱[儲存體文件](https://azure.microsoft.com/documentation/services/storage/)。)

Azure 佇列儲存體是一項儲存大量訊息的服務，全球任何地方都可利用 HTTP 或 HTTPS 並透過驗證的呼叫來存取這些訊息。 單一佇列訊息的大小上限為 64 KB，而一個佇列可以包含數百萬個訊息，以儲存體帳戶的總容量為限。 如需以程式設計方式處理佇列的詳細資料，另請參閱[以 .NET 開始使用 Azure 佇列儲存體](../storage/queues/storage-dotnet-how-to-use-queues.md)。

若要開始，請先在儲存體帳戶中建立 Azure 佇列。 本文章將接著示範如何以 C# 建立佇列，以及如何執行基本的佇列作業，例如新增、修改、讀取和移除佇列訊息。  程式碼使用適用於 .NET 的 Azure 儲存體用戶端程式庫。 如需 ASP.NET 的詳細資訊，請參閱 [ASP.NET](http://www.asp.net)。

某些 Azure 儲存體 API 是非同步的，本文章中的程式碼假設我們正在使用非同步方法。 如需詳細資訊，請參閱[非同步程式設計](https://docs.microsoft.com/dotnet/csharp/async)。

## <a name="access-queues-in-code"></a>在程式碼中存取佇列

若要存取 ASP.NET Core 專案中的佇列，請將下列項目包含在存取 Azure 佇列儲存體的任何 C# 來源檔案中。 請將這整段程式碼用於後續章節的程式碼之前。

1. 加入必要的 `using` 陳述式：
    ```cs
    using Microsoft.Framework.Configuration;
    using Microsoft.WindowsAzure.Storage;
    using Microsoft.WindowsAzure.Storage.Queue;
    using System.Threading.Tasks;
    using LogLevel = Microsoft.Framework.Logging.LogLevel;
    ```

1. 取得代表儲存體帳戶資訊的 `CloudStorageAccount` 物件。 使用下列程式碼，從 Azure 服務組態取得儲存體連接字串和儲存體帳戶資訊：

    ```cs
    CloudStorageAccount storageAccount = CloudStorageAccount.Parse(
        CloudConfigurationManager.GetSetting("<storage-account-name>_AzureStorageConnectionString"));
    ```

1. 取得 `CloudQueueClient` 物件以參考儲存體帳戶中的佇列物件：

    ```cs
    // Create the CloudQueueClient object for the storage account.
    CloudQueueClient queueClient = storageAccount.CreateCloudQueueClient();
    ```
1. 取得`CloudQueue` 物件以參考特定的佇列：

    ```cs
    // Get a reference to the CloudQueue named "messagequeue"
    CloudQueue messageQueue = queueClient.GetQueueReference("messagequeue");
    ```

### <a name="create-a-queue-in-code"></a>在程式碼中建立佇列

若要在程式碼中建立 Azure 佇列，請呼叫 ``CreateIfNotExistsAsync`：

```cs
// Create the CloudQueue if it does not exist.
await messageQueue.CreateIfNotExistsAsync();
```

## <a name="add-a-message-to-a-queue"></a>將訊息新增至佇列

若要將訊息插入現有佇列，請建立新的 `CloudQueueMessage` 物件，然後呼叫 `AddMessageAsync` 方法。 您可以從字串 (採用 UTF-8 格式) 或位元組陣列建立 `CloudQueueMessage` 物件。

```cs
// Create a message and add it to the queue.
CloudQueueMessage message = new CloudQueueMessage("Hello, World");
await messageQueue.AddMessageAsync(message);
```

## <a name="read-a-message-in-a-queue"></a>讀取佇列中的訊息

透過呼叫 `PeekMessageAsync` 方法，您可以在佇列前面查看訊息，而無需將其從佇列中移除：

```cs
// Peek the next message in the queue.
CloudQueueMessage peekedMessage = await messageQueue.PeekMessageAsync();
```

## <a name="read-and-remove-a-message-in-a-queue"></a>讀取並移除佇列中的訊息

您的程式碼可以使用兩個步驟將訊息從佇列中移除 (清除佇列)。

1. 呼叫 `GetMessageAsync` 以取得佇列中的下一個訊息。 從 `GetMessageAsync` 傳回的訊息，對於從此佇列讀取訊息的任何其他程式碼而言，將會是不可見的。 依預設，此訊息會維持 30 秒的不可見狀態。
1. 若要完成從佇列中移除訊息，請呼叫 `DeleteMessageAsync`。

這個移除訊息的兩步驟程序可確保您的程式碼因為硬體或軟體故障而無法處理訊息時，另一個程式碼的執行個體可以取得相同訊息並再試一次。 下列程式碼會在處理完訊息之後立即呼叫 `DeleteMessageAsync`：

```cs
// Get the next message in the queue.
CloudQueueMessage retrievedMessage = await messageQueue.GetMessageAsync();

// Process the message in less than 30 seconds.

// Then delete the message.
await messageQueue.DeleteMessageAsync(retrievedMessage);
```

## <a name="additional-options-for-dequeuing-messages"></a>清除佇列訊息的其他選項

自訂從佇列中擷取訊息的方法有兩種。 首先，您可以取得一批訊息 (最多 32 個)。 其次，您可以設定較長或較短的可見度逾時，讓您的程式碼有較長或較短的時間可以完全處理每個訊息。 下列程式碼範例將使用 `GetMessages` 方法，在一次呼叫中取得 20 個訊息。 接著其會使用 `foreach` 迴圈處理每個訊息。 它也會將可見度逾時設定為每個訊息五分鐘。 請注意，系統會針對所有訊息同時啟動五分鐘計時器，所以五分鐘過後，任何尚未刪除的訊息都會重新出現。

```cs
// Retrieve 20 messages at a time, keeping those messages invisible for 5 minutes, 
//   delete each message after processing.

foreach (CloudQueueMessage message in messageQueue.GetMessages(20, TimeSpan.FromMinutes(5)))
{
    // Process all messages in less than 5 minutes, deleting each message after processing.
    queue.DeleteMessage(message);
}
```

## <a name="get-the-queue-length"></a>取得佇列長度

您可以取得佇列中的估計訊息數目。 `FetchAttributes` 方法會要求佇列服務擷取佇列屬性，其中包含訊息計數。 `ApproximateMethodCount` 屬性會傳回 `FetchAttributes` 方法所擷取的最後一個值，而無須呼叫佇列服務。

```cs
// Fetch the queue attributes.
messageQueue.FetchAttributes();

// Retrieve the cached approximate message count.
int? cachedMessageCount = messageQueue.ApproximateMessageCount;

// Display the number of messages.
Console.WriteLine("Number of messages in queue: " + cachedMessageCount);
```

## <a name="use-the-async-await-pattern-with-common-queue-apis"></a>搭配使用 Async-Await 模式和通用佇列 API

這個範例示範如何搭配使用 async-await 模式，以及結尾是 `Async` 的通用佇列 API。 使用非同步方法時，async-await 模式會暫止本機執行，直到呼叫完成為止。 這種行為可讓目前的執行緒執行其他工作，有助於避免發生效能瓶頸並提升應用程式的整體回應。

```cs
// Create a message to add to the queue.
CloudQueueMessage cloudQueueMessage = new CloudQueueMessage("My message");

// Async enqueue the message.
await messageQueue.AddMessageAsync(cloudQueueMessage);
Console.WriteLine("Message added");

// Async dequeue the message.
CloudQueueMessage retrievedMessage = await messageQueue.GetMessageAsync();
Console.WriteLine("Retrieved message with content '{0}'", retrievedMessage.AsString);

// Async delete the message.
await messageQueue.DeleteMessageAsync(retrievedMessage);
Console.WriteLine("Deleted message");
```

## <a name="delete-a-queue"></a>刪除佇列

若要刪除佇列及其內含的所有訊息，請在佇列物件上呼叫 `Delete` 方法：

```cs
// Delete the queue.
messageQueue.Delete();
```

## <a name="next-steps"></a>後續步驟

[!INCLUDE [vs-storage-dotnet-queues-next-steps](../../includes/vs-storage-dotnet-queues-next-steps.md)]
