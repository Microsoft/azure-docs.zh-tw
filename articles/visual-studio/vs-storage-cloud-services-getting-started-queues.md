---
title: 開始使用佇列儲存體和 Visual Studio 已連接服務 (雲端服務) | Microsoft Docs
description: 在使用 Visual Studio 已連接服務連接到儲存體帳戶之後，如何在 Visual Studio 雲端服務專案中開始使用 Azure 佇列儲存體
services: storage
author: ghogen
manager: douge
ms.assetid: da587aac-5e64-4e9a-8405-44cc1924881d
ms.prod: visual-studio-dev15
ms.technology: vs-azure
ms.workload: azure
ms.topic: conceptual
ms.date: 12/02/2016
ms.author: ghogen
ms.openlocfilehash: c856bfb691c8d1b43822718bbfb86ff2122f4988
ms.sourcegitcommit: fa493b66552af11260db48d89e3ddfcdcb5e3152
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 04/23/2018
ms.locfileid: "31795597"
---
# <a name="getting-started-with-azure-queue-storage-and-visual-studio-connected-services-cloud-services-projects"></a>開始使用 Azure 佇列儲存體和 Visual Studio 已連接服務 (雲端服務專案)
[!INCLUDE [storage-try-azure-tools-queues](../../includes/storage-try-azure-tools-queues.md)]

## <a name="overview"></a>概觀
本文描述當您在雲端服務專案中建立或參考 Azure 儲存體帳戶之後，如何在 Visual Studio 中使用 [加入已連接服務] 對話方塊，開始使用 Azure 佇列儲存體。

我們將會示範如何在程式碼中建立佇列。 我們也將顯示如何執行基本的佇列作業，例如新增、修改、讀取和讀取佇列訊息。 這些範例均以 C# 程式碼撰寫，並使用 [Microsoft Azure Storage Client Library for .NET](https://msdn.microsoft.com/library/azure/dn261237.aspx)。

[ **新增連接的服務** ] 作業會安裝適當的 NuGet 封裝，以存取專案中的 Azure 儲存體，並將儲存體帳戶的連接字串新增至您的專案組態檔。

* 如需以程式碼處理佇列的詳細資訊，請參閱 [以 .NET 開始使用 Azure 佇列儲存體](../storage/queues/storage-dotnet-how-to-use-queues.md) 。
* 如需 Azure 儲存體的一般資訊，請參閱 [儲存體文件](https://azure.microsoft.com/documentation/services/storage/) 。
* 如需 Azure 雲端服務的一般資訊，請參閱 [雲端服務文件](https://azure.microsoft.com/documentation/services/cloud-services/) 。
* 如需 ASP.NET 應用程式設計的詳細資訊，請參閱 [ASP.NET](http://www.asp.net) 。

Azure 佇列儲存體是一項儲存大量訊息的服務，全球任何地方都可利用 HTTP 或 HTTPS 並透過驗證的呼叫來存取這些訊息。 單一佇列訊息的大小上限為 64 KB，而一個佇列可以包含數百萬個訊息，以儲存體帳戶的總容量為限。

## <a name="access-queues-in-code"></a>在程式碼中存取佇列
若要在 Visual Studio 雲端服務專案中存取佇列，您需要將下列項目加入至任何 C# 原始程式檔，以存取 Azure 佇列儲存體。

1. 請確定 C# 檔案頂端的命名空間宣告包含這些 **using** 陳述式。
   
        using Microsoft.Framework.Configuration;
        using Microsoft.WindowsAzure.Storage;
        using Microsoft.WindowsAzure.Storage.Queue;
2. 取得 **CloudStorageAccount** 物件，其代表您的儲存體帳戶資訊。 使用下列程式碼，從 Azure 服務組態取得您的儲存體連接字串和儲存體帳戶資訊。
   
         CloudStorageAccount storageAccount = CloudStorageAccount.Parse(
           CloudConfigurationManager.GetSetting("<storage-account-name>_AzureStorageConnectionString"));
3. 取得 **CloudQueueClient** 物件以參考儲存體帳戶中的佇列物件。  
   
        // Create the queue client.
        CloudQueueClient queueClient = storageAccount.CreateCloudQueueClient();
4. 取得 **CloudQueue** 物件以參考特定的佇列。
   
        // Get a reference to a queue named "messageQueue"
        CloudQueue messageQueue = queueClient.GetQueueReference("messageQueue");

**注意：** 請在下列範例中的程式碼前面使用上述所有程式碼。

## <a name="create-a-queue-in-code"></a>在程式碼中建立佇列
若要在程式碼中建立佇列，請加入 **CreateIfNotExists**呼叫。

    // Create the CloudQueue if it does not exist
    messageQueue.CreateIfNotExists();

## <a name="add-a-message-to-a-queue"></a>將訊息新增至佇列
若要將訊息插入現有佇列，請建立新的 **CloudQueueMessage** 物件，然後呼叫 **AddMessage** 方法。

您可以從字串 (採用 UTF-8 格式) 或位元組陣列建立 **CloudQueueMessage** 物件。

以下是插入訊息 'Hello, World' 的範例。

    // Create a message and add it to the queue.
    CloudQueueMessage message = new CloudQueueMessage("Hello, World");
    messageQueue.AddMessage(message);

## <a name="read-a-message-in-a-queue"></a>讀取佇列中的訊息
透過呼叫 **PeekMessage** 方法，您可以在佇列前面查看訊息，而無需將它從佇列中移除。

    // Peek at the next message
    CloudQueueMessage peekedMessage = messageQueue.PeekMessage();

## <a name="read-and-remove-a-message-in-a-queue"></a>讀取並移除佇列中的訊息
您的程式碼可以使用兩個步驟將訊息從佇列中移除 (清除佇列)。

1. 呼叫 **GetMessage** 以取得佇列中的下一個訊息。 從 **GetMessage** 傳回的訊息，對於從此佇列讀取訊息的任何其他程式碼而言將會是不可見的。 依預設，此訊息會維持 30 秒的不可見狀態。
2. 若要完成從佇列中移除訊息，請呼叫 **DeleteMessage**。

這個移除訊息的兩步驟程序可確保您的程式碼因為硬體或軟體故障而無法處理訊息時，另一個程式碼的執行個體可以取得相同訊息並再試一次。 下列程式碼會在處理完訊息之後立即呼叫 **DeleteMessage** 。

    // Get the next message in the queue.
    CloudQueueMessage retrievedMessage = messageQueue.GetMessage();

    // Process the message in less than 30 seconds

    // Then delete the message.
    await messageQueue.DeleteMessage(retrievedMessage);


## <a name="use-additional-options-to-process-and-remove-queue-messages"></a>使用其他選項來處理和移除佇列訊息
自訂從佇列中擷取訊息的方法有兩種。

* 您可以取得一批訊息 (最多 32 個)。
* 您可以設定較長或較短的可見度逾時，讓您的程式碼有較長或較短的時間可以完全處理每個訊息。 下列程式碼範例將使用 **GetMessages** 方法，在一次呼叫中取得 20 個訊息。 接著它會使用 **foreach** 迴圈處理每個訊息。 它也會將可見度逾時設定為每個訊息五分鐘。 請注意，系統會針對所有訊息同時開始計時 5 分鐘，所以從呼叫 **GetMessages**開始的 5 分鐘後，任何尚未刪除的訊息都會重新出現。

以下是範例：

    foreach (CloudQueueMessage message in messageQueue.GetMessages(20, TimeSpan.FromMinutes(5)))
    {
        // Process all messages in less than 5 minutes, deleting each message after processing.

        // Then delete the message after processing
        messageQueue.DeleteMessage(message);

    }

## <a name="get-the-queue-length"></a>取得佇列長度
您可以取得佇列中的估計訊息數目。 **FetchAttributes** 方法會要求佇列服務擷取佇列屬性，其中包含訊息計數。 **ApproximateMethodCount** 屬性會傳回 **FetchAttributes** 方法所擷取的最後一個值，而無需呼叫佇列服務。

    // Fetch the queue attributes.
    messageQueue.FetchAttributes();

    // Retrieve the cached approximate message count.
    int? cachedMessageCount = messageQueue.ApproximateMessageCount;

    // Display number of messages.
    Console.WriteLine("Number of messages in queue: " + cachedMessageCount);

## <a name="use-the-async-await-pattern-with-common-azure-queue-apis"></a>搭配使用 Async-Await 模式和通用 Azure 佇列 API
這個範例示範如何搭配使用 Async-Await 模式和通用 Azure 佇列 API。 此範例會呼叫每個指定方法的非同步版本，這可從每個方法的 **Async** 字尾看出。 使用非同步方法時，async-await 模式會暫停本機執行，直到呼叫完成為止。 這種行為可讓目前的執行緒執行其他工作，有助於避免發生效能瓶頸並提升應用程式的整體回應。 如需在 .NET 中使用 Async-Await 模式的詳細資訊，請參閱 [Async 和 Await (C# 和 Visual Basic)](https://msdn.microsoft.com/library/hh191443.aspx)

    // Create a message to put in the queue
    CloudQueueMessage cloudQueueMessage = new CloudQueueMessage("My message");

    // Add the message asynchronously
    await messageQueue.AddMessageAsync(cloudQueueMessage);
    Console.WriteLine("Message added");

    // Async dequeue the message
    CloudQueueMessage retrievedMessage = await messageQueue.GetMessageAsync();
    Console.WriteLine("Retrieved message with content '{0}'", retrievedMessage.AsString);

    // Delete the message asynchronously
    await messageQueue.DeleteMessageAsync(retrievedMessage);
    Console.WriteLine("Deleted message");

## <a name="delete-a-queue"></a>刪除佇列
若要刪除佇列及其內含的所有訊息，請在佇列物件上呼叫 **Delete** 方法。

    // Delete the queue.
    messageQueue.Delete();

## <a name="next-steps"></a>後續步驟
[!INCLUDE [vs-storage-dotnet-queues-next-steps](../../includes/vs-storage-dotnet-queues-next-steps.md)]

