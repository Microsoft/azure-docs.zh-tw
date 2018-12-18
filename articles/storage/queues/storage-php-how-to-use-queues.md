---
title: 如何使用 PHP 的佇列儲存體 | Microsoft Docs
description: 了解如何使用 Azure 佇列儲存體服務來建立和刪除佇列，以及插入、取得和刪除訊息。 範例是以 PHP 撰寫的。
services: storage
author: roygara
ms.service: storage
ms.tgt_pltfrm: na
ms.devlang: php
ms.topic: article
ms.date: 01/11/2018
ms.author: rogarana
ms.component: queues
ms.openlocfilehash: 7ab755972760d5f262e00ca4aaf284c8d2847460
ms.sourcegitcommit: 9819e9782be4a943534829d5b77cf60dea4290a2
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/06/2018
ms.locfileid: "39528277"
---
# <a name="how-to-use-queue-storage-from-php"></a>如何使用 PHP 的佇列儲存體
[!INCLUDE [storage-selector-queue-include](../../../includes/storage-selector-queue-include.md)]

[!INCLUDE [storage-try-azure-tools-queues](../../../includes/storage-try-azure-tools-queues.md)]

## <a name="overview"></a>概觀
本指南說明如何使用 Azure 佇列儲存體服務執行一般案例。 這些範例是透過[適用於 PHP 的 Azure 儲存體用戶端程式庫][download]的類別所撰寫。 所涵蓋的案例包括插入、查看、取得和刪除佇列訊息，以及建立和刪除佇列。

[!INCLUDE [storage-queue-concepts-include](../../../includes/storage-queue-concepts-include.md)]

[!INCLUDE [storage-create-account-include](../../../includes/storage-create-account-include.md)]

## <a name="create-a-php-application"></a>建立 PHP 應用程式
若要建立 PHP 應用程式並使其存取 Azure 佇列儲存體，唯一要求就是在您的程式碼中參考[適用於 PHP 的 Azure 儲存體用戶端程式庫][download]中的類別。 您可以使用任何開發工具來建立應用程式 (包括 [記事本])。

在本指南中，您將使用可從 PHP 應用程式內本機呼叫的佇列儲存體服務功能，或可在 Azure Web 角色、背景工作角色或網站內執行的程式碼中呼叫的佇列儲存體功能。

## <a name="get-the-azure-client-libraries"></a>取得 Azure 用戶端程式庫
### <a name="install-via-composer"></a>透過編輯器安裝
1. 在專案的根目錄中建立名為 **composer.json** 的檔案，並新增下列程式碼：
   
    ```json
    {
      "require": {
        "microsoft/azure-storage-queue": "*"
      }
    }
    ```
2. 將 **[composer.phar][composer-phar]** 下載到專案根目錄中。
3. 開啟命令提示字元，在專案根目錄中執行下列命令
   
    ```
    php composer.phar install
    ```

或者，移至 GitHub 上的 [Azure 儲存體 PHP 用戶端程式庫][download]來複製原始程式碼。

## <a name="configure-your-application-to-access-queue-storage"></a>設定您的應用程式以存取佇列儲存體
若要使用 Azure 佇列儲存體的 API，您需要：

1. 使用 [require_once] 陳述式來參考自動換片器檔案。
2. 參考任何您可能使用的類別。

下列範例會示範如何納入自動換片器檔案及參考 **QueueRestProxy** 類別。

```php
require_once 'vendor/autoload.php';
use MicrosoftAzure\Storage\Queue\QueueRestProxy;
```

在下列各範例中，一律會顯示 `require_once` 陳述式，但只會參考要執行之範例所需的類別。

## <a name="set-up-an-azure-storage-connection"></a>設定 Azure 儲存體連接
若要具現化 Azure 佇列儲存體用戶端，您必須先具備有效的連接字串。 佇列服務的連接字串格式如下。

用於存取即時服務：

```php
DefaultEndpointsProtocol=[http|https];AccountName=[yourAccount];AccountKey=[yourKey]
```

用於存取模擬器儲存體：

```php
UseDevelopmentStorage=true
```

若要建立 Azure 佇列服務用戶端，您需要使用 **QueueRestProxy** 類別。 您可以使用下列其中一種方式：

* 直接將連接字串傳遞給它。
* 使用 Web 應用程式中的環境變數來儲存連接字串。 請參閱 [Azure Web 應用程式組態設定](../../app-service/web-sites-configure.md)文件來設定連接字串。
在本文的各範例中，將會直接傳遞連接字串。

```php
require_once 'vendor/autoload.php';

use MicrosoftAzure\Storage\Queue\QueueRestProxy;

$connectionString = "DefaultEndpointsProtocol=http;AccountName=<accountNameHere>;AccountKey=<accountKeyHere>";
$queueClient = QueueRestProxy::createQueueService($connectionString);
```

## <a name="create-a-queue"></a>建立佇列
**QueueRestProxy** 物件可讓您使用 **createQueue** 方法建立佇列。 建立佇列時，您可以在佇列上設定選項，但這並非必要動作。 (下列範例示範如何在佇列上設定中繼資料。)

```php
require_once 'vendor/autoload.php';

use MicrosoftAzure\Storage\Queue\QueueRestProxy;
use MicrosoftAzure\Storage\Common\Exceptions\ServiceException;
use MicrosoftAzure\Storage\Queue\Models\CreateQueueOptions;

$connectionString = "DefaultEndpointsProtocol=http;AccountName=<accountNameHere>;AccountKey=<accountKeyHere>";

// Create queue REST proxy.
$queueClient = QueueRestProxy::createQueueService($connectionString);

// OPTIONAL: Set queue metadata.
$createQueueOptions = new CreateQueueOptions();
$createQueueOptions->addMetaData("key1", "value1");
$createQueueOptions->addMetaData("key2", "value2");

try    {
    // Create queue.
    $queueClient->createQueue("myqueue", $createQueueOptions);
}
catch(ServiceException $e){
    // Handle exception based on error codes and messages.
    // Error codes and messages are here:
    // http://msdn.microsoft.com/library/azure/dd179446.aspx
    $code = $e->getCode();
    $error_message = $e->getMessage();
    echo $code.": ".$error_message."<br />";
}
```

> [!NOTE]
> 您不應該倚賴大小寫來區分中繼資料索引鍵。 從服務讀取索引鍵時，所有索引鍵都是視為小寫。
> 
> 

## <a name="add-a-message-to-a-queue"></a>將訊息新增至佇列
若要將訊息新增至佇列，請使用 **QueueRestProxy->createMessage**。 此方法會接受佇列名稱、訊息文字以及訊息選項 (選用)。

```php
require_once 'vendor/autoload.php';

use MicrosoftAzure\Storage\Queue\QueueRestProxy;
use MicrosoftAzure\Storage\Common\Exceptions\ServiceException;
use MicrosoftAzure\Storage\Queue\Models\CreateMessageOptions;

$connectionString = "DefaultEndpointsProtocol=http;AccountName=<accountNameHere>;AccountKey=<accountKeyHere>";

// Create queue REST proxy.
$queueClient = QueueRestProxy::createQueueService($connectionString);

try    {
    // Create message.
    $queueClient->createMessage("myqueue", "Hello World!");
}
catch(ServiceException $e){
    // Handle exception based on error codes and messages.
    // Error codes and messages are here:
    // http://msdn.microsoft.com/library/azure/dd179446.aspx
    $code = $e->getCode();
    $error_message = $e->getMessage();
    echo $code.": ".$error_message."<br />";
}
```

## <a name="peek-at-the-next-message"></a>查看下一個訊息
透過呼叫 **QueueRestProxy->peekMessages** 方法，您可以在佇列前面查看訊息，而無需將它從佇列中移除。 **peekMessage** 預設會傳回單一訊息，但是您可以使用 **PeekMessagesOptions->setNumberOfMessages** 方法變更該值。

```php
require_once 'vendor/autoload.php';

use MicrosoftAzure\Storage\Queue\QueueRestProxy;
use MicrosoftAzure\Storage\Common\Exceptions\ServiceException;
use MicrosoftAzure\Storage\Queue\Models\PeekMessagesOptions;

$connectionString = "DefaultEndpointsProtocol=http;AccountName=<accountNameHere>;AccountKey=<accountKeyHere>";

// Create queue REST proxy.
$queueClient = QueueRestProxy::createQueueService($connectionString);

// OPTIONAL: Set peek message options.
$message_options = new PeekMessagesOptions();
$message_options->setNumberOfMessages(1); // Default value is 1.

try    {
    $peekMessagesResult = $queueClient->peekMessages("myqueue", $message_options);
}
catch(ServiceException $e){
    // Handle exception based on error codes and messages.
    // Error codes and messages are here:
    // http://msdn.microsoft.com/library/azure/dd179446.aspx
    $code = $e->getCode();
    $error_message = $e->getMessage();
    echo $code.": ".$error_message."<br />";
}

$messages = $peekMessagesResult->getQueueMessages();

// View messages.
$messageCount = count($messages);
if($messageCount <= 0){
    echo "There are no messages.<br />";
}
else{
    foreach($messages as $message)    {
        echo "Peeked message:<br />";
        echo "Message Id: ".$message->getMessageId()."<br />";
        echo "Date: ".date_format($message->getInsertionDate(), 'Y-m-d')."<br />";
        echo "Message text: ".$message->getMessageText()."<br /><br />";
    }
}
```

## <a name="de-queue-the-next-message"></a>將下一個訊息清除佇列
您的程式碼可以使用兩個步驟來將訊息從佇列中移除。 首先，您需呼叫 **QueueRestProxy->listMessages**，這會讓從佇列讀取資料的任何其他程式碼無法看見此訊息。 依預設，此訊息會維持 30 秒的不可見狀態。 (如果未在此期間內刪除訊息，訊息就會再次於佇列中變成可見。)若要完成從佇列中移除訊息，您必須呼叫 **QueueRestProxy->deleteMessage**。 這個移除訊息的兩步驟程序可確保您的程式碼因為硬體或軟體故障而無法處理訊息時，另一個程式碼的執行個體可以取得相同訊息並再試一次。 您的程式碼會在處理完訊息之後立即呼叫 **deleteMessage** 。

```php
require_once 'vendor/autoload.php';

use MicrosoftAzure\Storage\Queue\QueueRestProxy;
use MicrosoftAzure\Storage\Common\Exceptions\ServiceException;

$connectionString = "DefaultEndpointsProtocol=http;AccountName=<accountNameHere>;AccountKey=<accountKeyHere>";

// Create queue REST proxy.
$queueClient = QueueRestProxy::createQueueService($connectionString);

// Get message.
$listMessagesResult = $queueClient->listMessages("myqueue");
$messages = $listMessagesResult->getQueueMessages();
$message = $messages[0];

/* ---------------------
    Process message.
   --------------------- */

// Get message ID and pop receipt.
$messageId = $message->getMessageId();
$popReceipt = $message->getPopReceipt();

try    {
    // Delete message.
    $queueClient->deleteMessage("myqueue", $messageId, $popReceipt);
}
catch(ServiceException $e){
    // Handle exception based on error codes and messages.
    // Error codes and messages are here:
    // http://msdn.microsoft.com/library/azure/dd179446.aspx
    $code = $e->getCode();
    $error_message = $e->getMessage();
    echo $code.": ".$error_message."<br />";
}
```

## <a name="change-the-contents-of-a-queued-message"></a>變更佇列訊息的內容
透過呼叫 **QueueRestProxy->updateMessage**，您可以在佇列中就地變更訊息的內容。 如果訊息代表工作作業，則您可以使用此功能來更新工作作業的狀態。 下列程式碼將使用新的內容更新佇列訊息，並將可見度逾時設定延長 60 秒。 這可儲存與訊息相關的工作狀態，並提供用戶端多一分鐘的時間繼續處理訊息。 您可以使用此技巧來追蹤佇列訊息上的多步驟工作流程，如果因為硬體或軟體故障而導致某個處理步驟失敗，將無需從頭開始。 通常，您也會保留重試計數，如果訊息重試超過 *n* 次，您會將它刪除。 這麼做可防止每次處理時便觸發應用程式錯誤的訊息。

```php
require_once 'vendor/autoload.php';

use MicrosoftAzure\Storage\Queue\QueueRestProxy;
use MicrosoftAzure\Storage\Common\Exceptions\ServiceException;

// Create queue REST proxy.
$queueClient = QueueRestProxy::createQueueService($connectionString);

$connectionString = "DefaultEndpointsProtocol=http;AccountName=<accountNameHere>;AccountKey=<accountKeyHere>";

// Get message.
$listMessagesResult = $queueClient->listMessages("myqueue");
$messages = $listMessagesResult->getQueueMessages();
$message = $messages[0];

// Define new message properties.
$new_message_text = "New message text.";
$new_visibility_timeout = 5; // Measured in seconds.

// Get message ID and pop receipt.
$messageId = $message->getMessageId();
$popReceipt = $message->getPopReceipt();

try    {
    // Update message.
    $queueClient->updateMessage("myqueue",
                                $messageId,
                                $popReceipt,
                                $new_message_text,
                                $new_visibility_timeout);
}
catch(ServiceException $e){
    // Handle exception based on error codes and messages.
    // Error codes and messages are here:
    // http://msdn.microsoft.com/library/azure/dd179446.aspx
    $code = $e->getCode();
    $error_message = $e->getMessage();
    echo $code.": ".$error_message."<br />";
}
```

## <a name="additional-options-for-de-queuing-messages"></a>其他將訊息移出佇列的選項
自訂從佇列中擷取訊息的方法有兩種。 首先，您可以取得一批訊息 (最多 32 個)。 其次，您可以設定較長或較短的可見度逾時，讓您的程式碼有較長或較短的時間可以完整處理每個訊息。 下列程式碼範例將使用 **getMessages** 方法，在一次呼叫中取得 16 個訊息。 接著它會使用 **for** 迴圈處理每個訊息。 它也會將可見度逾時設定為每個訊息五分鐘。

```php
require_once 'vendor/autoload.php';

use MicrosoftAzure\Storage\Queue\QueueRestProxy;
use MicrosoftAzure\Storage\Common\Exceptions\ServiceException;
use MicrosoftAzure\Storage\Queue\Models\ListMessagesOptions;

$connectionString = "DefaultEndpointsProtocol=http;AccountName=<accountNameHere>;AccountKey=<accountKeyHere>";

// Create queue REST proxy.
$queueClient = QueueRestProxy::createQueueService($connectionString);

// Set list message options.
$message_options = new ListMessagesOptions();
$message_options->setVisibilityTimeoutInSeconds(300);
$message_options->setNumberOfMessages(16);

// Get messages.
try{
    $listMessagesResult = $queueClient->listMessages("myqueue",
                                                     $message_options);
    $messages = $listMessagesResult->getQueueMessages();

    foreach($messages as $message){

        /* ---------------------
            Process message.
        --------------------- */

        // Get message Id and pop receipt.
        $messageId = $message->getMessageId();
        $popReceipt = $message->getPopReceipt();

        // Delete message.
        $queueClient->deleteMessage("myqueue", $messageId, $popReceipt);
    }
}
catch(ServiceException $e){
    // Handle exception based on error codes and messages.
    // Error codes and messages are here:
    // http://msdn.microsoft.com/library/azure/dd179446.aspx
    $code = $e->getCode();
    $error_message = $e->getMessage();
    echo $code.": ".$error_message."<br />";
}
```

## <a name="get-queue-length"></a>取得佇列長度
您可以取得佇列中的估計訊息數目。 **QueueRestProxy->getQueueMetadata** 方法會要求佇列服務傳回佇列本身的中繼資料。 在傳回的物件上呼叫 **getApproximateMessageCount** 方法會提供佇列中的訊息計數。 此計數只是一個約略值，因為在佇列服務回應您的要求之後，仍有新增或移除訊息的可能。

```php
require_once 'vendor/autoload.php';

use MicrosoftAzure\Storage\Queue\QueueRestProxy;
use MicrosoftAzure\Storage\Common\Exceptions\ServiceException;

$connectionString = "DefaultEndpointsProtocol=http;AccountName=<accountNameHere>;AccountKey=<accountKeyHere>";

// Create queue REST proxy.
$queueClient = QueueRestProxy::createQueueService($connectionString);

try    {
    // Get queue metadata.
    $queue_metadata = $queueClient->getQueueMetadata("myqueue");
    $approx_msg_count = $queue_metadata->getApproximateMessageCount();
}
catch(ServiceException $e){
    // Handle exception based on error codes and messages.
    // Error codes and messages are here:
    // http://msdn.microsoft.com/library/azure/dd179446.aspx
    $code = $e->getCode();
    $error_message = $e->getMessage();
    echo $code.": ".$error_message."<br />";
}

echo $approx_msg_count;
```

## <a name="delete-a-queue"></a>刪除佇列
若要刪除佇列及其所有訊息，請呼叫 **QueueRestProxy->deleteQueue** 方法。

```php
require_once 'vendor/autoload.php';

use MicrosoftAzure\Storage\Queue\QueueRestProxy;
use MicrosoftAzure\Storage\Common\Exceptions\ServiceException;

$connectionString = "DefaultEndpointsProtocol=http;AccountName=<accountNameHere>;AccountKey=<accountKeyHere>";

// Create queue REST proxy.
$queueClient = QueueRestProxy::createQueueService($connectionString);

try    {
    // Delete queue.
    $queueClient->deleteQueue("myqueue");
}
catch(ServiceException $e){
    // Handle exception based on error codes and messages.
    // Error codes and messages are here:
    // http://msdn.microsoft.com/library/azure/dd179446.aspx
    $code = $e->getCode();
    $error_message = $e->getMessage();
    echo $code.": ".$error_message."<br />";
}
```

## <a name="next-steps"></a>後續步驟
既然已了解 Azure 佇列儲存體的基本概念，請遵循下列連結以了解更複雜的儲存體工作。

* 請瀏覽 [Azure 儲存體 PHP 用戶端程式庫的 API 參考](http://azure.github.io/azure-storage-php/)
* 請參閱[進階佇列範例](https://github.com/Azure/azure-storage-php/blob/master/samples/QueueSamples.php)。

如需詳細資訊，另請參閱 [PHP 開發人員中心](/develop/php/)。

[download]: https://github.com/Azure/azure-storage-php
[require_once]: http://www.php.net/manual/en/function.require-once.php
[Azure Portal]: https://portal.azure.com
[composer-phar]: http://getcomposer.org/composer.phar

