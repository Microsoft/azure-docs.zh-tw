---
title: 如何使用拼音的佇列儲存體 | Microsoft Docs
description: 了解如何使用 Azure 佇列服務來建立和刪除佇列，以及插入、取得和刪除訊息。 範例以 Ruby 撰寫。
services: storage
author: tamram
ms.service: storage
ms.tgt_pltfrm: na
ms.devlang: ruby
ms.topic: article
ms.date: 12/08/2016
ms.author: tamram
ms.component: queues
ms.openlocfilehash: 7074ae21c1816b5266cbcf69d0126751ded19439
ms.sourcegitcommit: 9819e9782be4a943534829d5b77cf60dea4290a2
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/06/2018
ms.locfileid: "39522599"
---
# <a name="how-to-use-queue-storage-from-ruby"></a>如何使用 Ruby 的佇列儲存體
[!INCLUDE [storage-selector-queue-include](../../../includes/storage-selector-queue-include.md)]

[!INCLUDE [storage-try-azure-tools-queues](../../../includes/storage-try-azure-tools-queues.md)]

## <a name="overview"></a>概觀
本指南示範如何使用 Microsoft Azure 佇列儲存體服務執行常見案例。 這些範例是以 Ruby Azure API 撰寫的。
所涵蓋的案例包括「插入」、「查看」、「取得」和「刪除」佇列訊息，以及「建立和刪除佇列」。

[!INCLUDE [storage-queue-concepts-include](../../../includes/storage-queue-concepts-include.md)]

[!INCLUDE [storage-create-account-include](../../../includes/storage-create-account-include.md)]

## <a name="create-a-ruby-application"></a>建立 Ruby 應用程式
建立 Ruby 應用程式。 如需說明，請參閱[在 Linux 上的 App Service 中建立 Ruby 應用程式](https://docs.microsoft.com/azure/app-service/containers/quickstart-ruby)。

## <a name="configure-your-application-to-access-storage"></a>設定您的應用程式以存取儲存體
若要使用 Azure 儲存體，您需要下載並使用 Ruby azure 套件，這包含一組便利程式庫，能與儲存體 REST 服務通訊。

### <a name="use-rubygems-to-obtain-the-package"></a>使用 RubyGems 來取得套件
1. 使用命令列介面，例如 **PowerShell** (Windows)、**Terminal** (Mac) 或 **Bash** (Unix)。
2. 在命令視窗中鍵入 "gem install azure" 以安裝 Gem 和相依性。

### <a name="import-the-package"></a>匯入封裝
使用您偏好的文字編輯器，將以下內容新增至您打算使用儲存體的 Ruby 檔案頂端：

```ruby
require "azure"
```

## <a name="setup-an-azure-storage-connection"></a>設定 Azure 儲存體連接
azure 模組會讀取環境變數 **AZURE\_STORAGE\_ACCOUNT** 及 **AZURE\_STORAGE\_ACCESS_KEY**，以取得連接 Azure 儲存體帳戶所需的資訊。 若未設定這些環境變數，您必須使用下列程式碼，在使用 **Azure::QueueService** 之前指定帳戶資訊：

```ruby
Azure.config.storage_account_name = "<your azure storage account>"
Azure.config.storage_access_key = "<your Azure storage access key>"
```

若要從 Azure 入口網站中的傳統或 Resource Manager 儲存體帳戶取得這些值：

1. 登入 [Azure 入口網站](https://portal.azure.com)。
2. 瀏覽到您要使用的儲存體帳戶。
3. 在右邊的 [設定] 刀鋒視窗中，按一下 [存取金鑰]。
4. [存取金鑰] 刀鋒視窗隨即顯示，您會看到存取金鑰 1 和存取金鑰 2。 您可以使用其中一個存取金鑰。 
5. 按一下複製圖示以將金鑰複製到剪貼簿。 

## <a name="how-to-create-a-queue"></a>作法：建立佇列
下列程式碼會建立一個 **Azure::QueueService** 物件，讓您能夠使用佇列。

```ruby
azure_queue_service = Azure::QueueService.new
```

使用 **create_queue()** 方法，建立具有指定名稱的佇列。

```ruby
begin
  azure_queue_service.create_queue("test-queue")
rescue
  puts $!
end
```

## <a name="how-to-insert-a-message-into-a-queue"></a>作法：將訊息插入佇列中
若要將訊息插入佇列中，請使用 **create_message()** 方法建立新訊息，並將其新增至佇列。

```ruby
azure_queue_service.create_message("test-queue", "test message")
```

## <a name="how-to-peek-at-the-next-message"></a>作法：預覽下一個訊息
透過呼叫 **peek\_messages()** 方法，您可以在佇列前面查看訊息，而無需將它從佇列中移除。 **peek\_messages()** 預設會查看單一訊息。 您也可以指定所要查看的訊息數。

```ruby
result = azure_queue_service.peek_messages("test-queue",
  {:number_of_messages => 10})
```

## <a name="how-to-dequeue-the-next-message"></a>作法：清除下一個佇列訊息
您可以使用兩個步驟將訊息從佇列中移除。

1. 呼叫 **list\_messages()** 時，您預設會取得佇列中的下一個訊息。 您也可以指定您要取得的訊息數。 從 **list\_messages()** 傳回的訊息，對於從此佇列讀取訊息的任何其他程式碼而言將會是不可見的。 您可以傳入以秒為單位的可見性逾時，作為參數。
2. 若要完成從佇列中移除訊息的作業，您還必須呼叫 **delete_message()**。

這個移除訊息的兩步驟程序可確保您的程式碼因為硬體或軟體故障而無法處理訊息時，另一個程式碼的執行個體可以取得相同訊息並再試一次。 處理完訊息之後，您的程式碼會立即呼叫 **delete\_message()**。

```ruby
messages = azure_queue_service.list_messages("test-queue", 30)
azure_queue_service.delete_message("test-queue", 
  messages[0].id, messages[0].pop_receipt)
```

## <a name="how-to-change-the-contents-of-a-queued-message"></a>作法：變更佇列訊息的內容
您可以在佇列中就地變更訊息內容。 以下程式碼會使用 **update_message()** 方法來更新訊息。 此方法會傳回一個 Tuple (其中包含佇列訊息的 pop receipt) 和一個 UTC 日期時間值 (代表訊息將會顯示在佇列上的時間)。

```ruby
message = azure_queue_service.list_messages("test-queue", 30)
pop_receipt, time_next_visible = azure_queue_service.update_message(
  "test-queue", message.id, message.pop_receipt, "updated test message", 
  30)
```

## <a name="how-to-additional-options-for-dequeuing-messages"></a>作法：清除佇列訊息的其他選項
自訂從佇列中擷取訊息的方法有兩種。

1. 您可以取得一批訊息。
2. 您可以設定較長或較短的可見度逾時，讓您的程式碼有較長或較短的時間可以完全處理每個訊息。

下列程式碼範例將使用 **list\_messages()** 方法，在一次呼叫中取得 15 個訊息。 接著，它會列出每個訊息，並加以刪除。 它也會將可見度逾時設定為每個訊息五分鐘。

```ruby
azure_queue_service.list_messages("test-queue", 300
  {:number_of_messages => 15}).each do |m|
  puts m.message_text
  azure_queue_service.delete_message("test-queue", m.id, m.pop_receipt)
end
```

## <a name="how-to-get-the-queue-length"></a>作法：取得佇列長度
您可以取得佇列中的估計訊息數目。 **get\_queue\_metadata()** 方法會要求佇列服務傳回約略的訊息計數和佇列的相關中繼資料。

```ruby
message_count, metadata = azure_queue_service.get_queue_metadata(
  "test-queue")
```

## <a name="how-to-delete-a-queue"></a>作法：刪除佇列
若要刪除佇列及其內含的所有訊息，請在佇列物件上呼叫 **delete\_queue()** 方法。

```ruby
azure_queue_service.delete_queue("test-queue")
```

## <a name="next-steps"></a>後續步驟
了解佇列儲存體的基礎概念之後，請參考下列連結以了解有關更複雜的儲存工作。

* 造訪 [Azure 儲存體團隊部落格](http://blogs.msdn.com/b/windowsazurestorage/)
* 請造訪 GitHub 上的 [Azure SDK for Ruby](https://github.com/WindowsAzure/azure-sdk-for-ruby) 儲存機制 (英文)。

若要比較本文所討論的 Azure 佇列服務與[如何使用服務匯流排佇列](/develop/ruby/how-to-guides/service-bus-queues/)一文中討論的 Azure 服務匯流排佇列，請參閱 [Azure 佇列和服務匯流排佇列 - 比較和對照](../../service-bus-messaging/service-bus-azure-and-service-bus-queues-compared-contrasted.md)