---
title: 如何使用佇列儲存體 (C++) | Microsoft Docs
description: 了解如何在 Azure 中使用佇列儲存體服務。 範例是以 C++ 撰寫的。
services: storage
author: cbrooksmsft
ms.service: storage
ms.tgt_pltfrm: na
ms.devlang: cpp
ms.topic: article
ms.date: 05/11/2017
ms.author: cbrooksmsft
ms.component: queues
ms.openlocfilehash: 36fa2e5bc7eda7c47017713008aec2a245213462
ms.sourcegitcommit: 9819e9782be4a943534829d5b77cf60dea4290a2
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/06/2018
ms.locfileid: "39521562"
---
# <a name="how-to-use-queue-storage-from-c"></a>如何使用 C++ 的佇列儲存體
[!INCLUDE [storage-selector-queue-include](../../../includes/storage-selector-queue-include.md)]

[!INCLUDE [storage-try-azure-tools-queues](../../../includes/storage-try-azure-tools-queues.md)]

## <a name="overview"></a>概觀
本指南將示範如何使用 Azure 佇列儲存體服務執行一般案例。 這些範例均以 C++ 撰寫，並使用 [Azure Storage Client Library for C++](http://github.com/Azure/azure-storage-cpp/blob/master/README.md)。 所涵蓋的案例包括「插入」、「查看」、「取得」和「刪除」佇列訊息，以及「建立和刪除佇列」。

> [!NOTE]
> 本指南以 Azure Storage Client Library for C++ 1.0.0 版和更新版本為對象。 建議的版本是 Storage Client Library 2.2.0，可透過 [NuGet](http://www.nuget.org/packages/wastorage) 或 [GitHub](http://github.com/Azure/azure-storage-cpp/) 取得。
> 
> 

[!INCLUDE [storage-queue-concepts-include](../../../includes/storage-queue-concepts-include.md)]

[!INCLUDE [storage-create-account-include](../../../includes/storage-create-account-include.md)]

## <a name="create-a-c-application"></a>建立 C++ 應用程式
在本指南中，您將使用可以在 C++ 應用程式內執行的儲存體功能。

若要這樣做，您需要安裝 Azure Storage Client Library for C++，並在 Azure 訂用帳戶中建立 Azure 儲存體帳戶。

若要安裝 Azure Storage Client Library for C++，您可以使用下列方法：

* **Linux：** 遵循 [Azure Storage Client Library for C++ 讀我檔案](https://github.com/Azure/azure-storage-cpp/blob/master/README.md) 頁面中提供的指示進行。
* **Windows：** 在 Visual Studio 中，按一下 [工具] > [NuGet 套件管理員] > [套件管理員主控台]。 在 [NuGet 套件管理員主控台](http://docs.nuget.org/docs/start-here/using-the-package-manager-console) 中輸入下列命令，然後按下 **Enter**。

```  
Install-Package wastorage
```

## <a name="configure-your-application-to-access-queue-storage"></a>設定您的應用程式以存取佇列儲存體
在您要使用 Azure 儲存體 API 來存取佇列的 C++ 檔案頂端，加入下列 include 陳述式：  

```cpp
#include <was/storage_account.h>
#include <was/queue.h>
```

## <a name="set-up-an-azure-storage-connection-string"></a>設定 Azure 儲存體連接字串
Azure 儲存體用戶端會使用儲存體連接字串來儲存存取資料管理服務時所用的端點與認證。 在用戶端應用程式中執行時，您必須以下列格式提供儲存體連接字串 (其中的 *AccountName* 和 *AccountKey* 值要使用您儲存體帳戶的名稱，以及在 [Azure 入口網站](https://portal.azure.com)中針對該儲存體帳戶而列出的儲存體存取金鑰)。 如需有關儲存體帳戶和存取金鑰的資訊，請參閱 [關於 Azure 儲存體帳戶](../common/storage-create-storage-account.md?toc=%2fazure%2fstorage%2fqueues%2ftoc.json)。 本範例將示範如何宣告靜態欄位來存放連接字串：  

```cpp
// Define the connection-string with your values.
const utility::string_t storage_connection_string(U("DefaultEndpointsProtocol=https;AccountName=your_storage_account;AccountKey=your_storage_account_key"));
```

若要在本機 Windows 電腦中測試您的應用程式，可以使用隨 [Azure SDK](https://azure.microsoft.com/downloads/) 一起安裝的 Microsoft Azure [儲存體模擬器](../common/storage-use-emulator.md?toc=%2fazure%2fstorage%2fqueues%2ftoc.json)。 儲存體模擬器是一個公用程式，可在本機開發電腦上模擬 Azure 提供的 Blob、佇列和表格服務。 下列範例示範如何宣告靜態欄位以便將連接字串存放到本機儲存體模擬器中：  

```cpp
// Define the connection-string with Azure Storage Emulator.
const utility::string_t storage_connection_string(U("UseDevelopmentStorage=true;"));  
```

若要啟動 Azure 儲存體模擬器，選取 [開始] 按鈕或按下 [Windows] 鍵。 開始輸入 **Azure 儲存體模擬器**，然後從應用程式清單選取 [Microsoft Azure 儲存體模擬器]。

下列範例假設您已經使用這兩個方法之一來取得儲存體連接字串。

## <a name="retrieve-your-connection-string"></a>擷取連接字串
您可以使用 **cloud_storage_account** 類別來代表儲存體帳戶資訊。 若要從儲存體連接字串擷取儲存體帳戶資訊，您可以使用 **parse** 方法。

```cpp
// Retrieve storage account from connection string.
azure::storage::cloud_storage_account storage_account = azure::storage::cloud_storage_account::parse(storage_connection_string);
```

## <a name="how-to-create-a-queue"></a>作法：建立佇列
**cloud_queue_client** 物件可讓您取得佇列的參考物件。 下列程式碼會建立一個 **cloud_queue_client** 物件。

```cpp
// Retrieve storage account from connection string.
azure::storage::cloud_storage_account storage_account = azure::storage::cloud_storage_account::parse(storage_connection_string);

// Create a queue client.
azure::storage::cloud_queue_client queue_client = storage_account.create_cloud_queue_client();
```

使用 **cloud_queue_client** 物件取得想要使用的佇列的參考。 如果佇列不存在，您可以建立佇列。

```cpp
// Retrieve a reference to a queue.
azure::storage::cloud_queue queue = queue_client.get_queue_reference(U("my-sample-queue"));

// Create the queue if it doesn't already exist.
 queue.create_if_not_exists();  
```

## <a name="how-to-insert-a-message-into-a-queue"></a>作法：將訊息插入佇列中
若要將訊息插入現有佇列，請先建立一個新的 **cloud_queue_message**。 接著，呼叫 **add_message** 方法。 **cloud_queue_message** 便可以從字串或 **byte** 陣列建立。 以下是建立佇列 (如果佇列不存在) 並插入訊息 'Hello, World' 的程式碼：

```cpp
// Retrieve storage account from connection-string.
azure::storage::cloud_storage_account storage_account = azure::storage::cloud_storage_account::parse(storage_connection_string);

// Create the queue client.
azure::storage::cloud_queue_client queue_client = storage_account.create_cloud_queue_client();

// Retrieve a reference to a queue.
azure::storage::cloud_queue queue = queue_client.get_queue_reference(U("my-sample-queue"));

// Create the queue if it doesn't already exist.
queue.create_if_not_exists();

// Create a message and add it to the queue.
azure::storage::cloud_queue_message message1(U("Hello, World"));
queue.add_message(message1);  
```

## <a name="how-to-peek-at-the-next-message"></a>作法：查看下一個訊息
您可以呼叫 **peek_message** 方法，以便在佇列前面查看訊息，而無需將它從佇列中移除。

```cpp
// Retrieve storage account from connection-string.
azure::storage::cloud_storage_account storage_account = azure::storage::cloud_storage_account::parse(storage_connection_string);

// Create the queue client.
azure::storage::cloud_queue_client queue_client = storage_account.create_cloud_queue_client();

// Retrieve a reference to a queue.
azure::storage::cloud_queue queue = queue_client.get_queue_reference(U("my-sample-queue"));

// Peek at the next message.
azure::storage::cloud_queue_message peeked_message = queue.peek_message();

// Output the message content.
std::wcout << U("Peeked message content: ") << peeked_message.content_as_string() << std::endl;
```

## <a name="how-to-change-the-contents-of-a-queued-message"></a>作法：變更佇列訊息的內容
您可以在佇列中就地變更訊息內容。 如果訊息代表工作作業，則您可以使用此功能來更新工作作業的狀態。 下列程式碼將使用新的內容更新佇列訊息，並將可見度逾時設定延長 60 秒。 這可儲存與訊息相關的工作狀態，並提供用戶端多一分鐘的時間繼續處理訊息。 您可以使用此技巧來追蹤佇列訊息上的多步驟工作流程，如果因為硬體或軟體故障而導致某個處理步驟失敗，將無需從頭開始。 通常，您也會保留重試計數，如果訊息重試超過 n 次，您會將它刪除。 這麼做可防止每次處理時便觸發應用程式錯誤的訊息。

```cpp
// Retrieve storage account from connection-string.
azure::storage::cloud_storage_account storage_account = azure::storage::cloud_storage_account::parse(storage_conection_string);

// Create the queue client.
azure::storage::cloud_queue_client queue_client = storage_account.create_cloud_queue_client();

// Retrieve a reference to a queue.
azure::storage::cloud_queue queue = queue_client.get_queue_reference(U("my-sample-queue"));

// Get the message from the queue and update the message contents.
// The visibility timeout "0" means make it visible immediately.
// The visibility timeout "60" means the client can get another minute to continue
// working on the message.
azure::storage::cloud_queue_message changed_message = queue.get_message();

changed_message.set_content(U("Changed message"));
queue.update_message(changed_message, std::chrono::seconds(60), true);

// Output the message content.
std::wcout << U("Changed message content: ") << changed_message.content_as_string() << std::endl;  
```

## <a name="how-to-de-queue-the-next-message"></a>作法：在下一個訊息清除佇列
您的程式碼可以使用兩個步驟將訊息自佇列中清除佇列。 呼叫 **get_message** 時，您會取得佇列中的下一個訊息。 對於從此佇列讀取訊息的其他任何程式碼而言，將無法看到從 **get_message** 傳回的訊息。 若要完成從佇列中移除訊息的作業，您還必須呼叫 **delete_message**。 這個移除訊息的兩步驟程序可確保您的程式碼因為硬體或軟體故障而無法處理訊息時，另一個程式碼的執行個體可以取得相同訊息並再試一次。 您的程式碼會在處理完訊息之後立即呼叫 **delete_message**。

```cpp
// Retrieve storage account from connection-string.
azure::storage::cloud_storage_account storage_account = azure::storage::cloud_storage_account::parse(storage_connection_string);

// Create the queue client.
azure::storage::cloud_queue_client queue_client = storage_account.create_cloud_queue_client();

// Retrieve a reference to a queue.
azure::storage::cloud_queue queue = queue_client.get_queue_reference(U("my-sample-queue"));

// Get the next message.
azure::storage::cloud_queue_message dequeued_message = queue.get_message();
std::wcout << U("Dequeued message: ") << dequeued_message.content_as_string() << std::endl;

// Delete the message.
queue.delete_message(dequeued_message);
```

## <a name="how-to-leverage-additional-options-for-de-queuing-messages"></a>作法：運用清除佇列訊息的其他選項
自訂從佇列中擷取訊息的方法有兩種。 首先，您可以取得一批訊息 (最多 32 個)。 其次，您可以設定較長或較短的可見度逾時，讓您的程式碼有較長或較短的時間可以完全處理每個訊息。 下列程式碼範例將使用 **get_messages** 方法，在一次呼叫中取得 20 個訊息。 接著它會使用 **for** 迴圈處理每個訊息。 它也會將可見度逾時設定為每個訊息五分鐘。 請注意，系統會針對所有訊息同時開始計時 5 分鐘，所以從呼叫 **get_messages** 開始的 5 分鐘後，任何尚未刪除的訊息都會重新出現。

```cpp
// Retrieve storage account from connection-string.
azure::storage::cloud_storage_account storage_account = azure::storage::cloud_storage_account::parse(storage_connection_string);

// Create the queue client.
azure::storage::cloud_queue_client queue_client = storage_account.create_cloud_queue_client();

// Retrieve a reference to a queue.
azure::storage::cloud_queue queue = queue_client.get_queue_reference(U("my-sample-queue"));

// Dequeue some queue messages (maximum 32 at a time) and set their visibility timeout to
// 5 minutes (300 seconds).
azure::storage::queue_request_options options;
azure::storage::operation_context context;

// Retrieve 20 messages from the queue with a visibility timeout of 300 seconds.
std::vector<azure::storage::cloud_queue_message> messages = queue.get_messages(20, std::chrono::seconds(300), options, context);

for (auto it = messages.cbegin(); it != messages.cend(); ++it)
{
    // Display the contents of the message.
    std::wcout << U("Get: ") << it->content_as_string() << std::endl;
}
```

## <a name="how-to-get-the-queue-length"></a>作法：取得佇列長度
您可以取得佇列中的估計訊息數目。 **download_attributes** 方法會要求佇列服務擷取佇列屬性，其中包含訊息計數。 **approximate_message_count** 方法會取得佇列中大約的訊息數目。

```cpp
// Retrieve storage account from connection-string.
azure::storage::cloud_storage_account storage_account = azure::storage::cloud_storage_account::parse(storage_connection_string);

// Create the queue client.
azure::storage::cloud_queue_client queue_client = storage_account.create_cloud_queue_client();

// Retrieve a reference to a queue.
azure::storage::cloud_queue queue = queue_client.get_queue_reference(U("my-sample-queue"));

// Fetch the queue attributes.
queue.download_attributes();

// Retrieve the cached approximate message count.
int cachedMessageCount = queue.approximate_message_count();

// Display number of messages.
std::wcout << U("Number of messages in queue: ") << cachedMessageCount << std::endl;  
```

## <a name="how-to-delete-a-queue"></a>作法：刪除佇列
若要刪除佇列及其內含的所有訊息，請針對佇列物件呼叫 **delete_queue_if_exists** 方法。

```cpp
// Retrieve storage account from connection-string.
azure::storage::cloud_storage_account storage_account = azure::storage::cloud_storage_account::parse(storage_connection_string);

// Create the queue client.
azure::storage::cloud_queue_client queue_client = storage_account.create_cloud_queue_client();

// Retrieve a reference to a queue.
azure::storage::cloud_queue queue = queue_client.get_queue_reference(U("my-sample-queue"));

// If the queue exists and delete it.
queue.delete_queue_if_exists();  
```

## <a name="next-steps"></a>後續步驟
了解佇列儲存體的基礎概念之後，請依照下列連結深入了解 Azure 儲存體。

* [如何使用 C++ 的 Blob 儲存體](../blobs/storage-c-plus-plus-how-to-use-blobs.md)
* [如何使用 C++ 的資料表儲存體](../../cosmos-db/table-storage-how-to-use-c-plus.md)
* [以 C++ 列出 Azure 儲存體資源](../common/storage-c-plus-plus-enumeration.md?toc=%2fazure%2fstorage%2fqueues%2ftoc.json)
* [Storage Client Library for C++ 參考資料](http://azure.github.io/azure-storage-cpp)
* [Azure 儲存體文件](https://azure.microsoft.com/documentation/services/storage/)