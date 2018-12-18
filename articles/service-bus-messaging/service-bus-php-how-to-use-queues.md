---
title: 如何將服務匯流排佇列搭配 PHP 使用 | Microsoft Docs
description: 了解如何使用 Azure 中的服務匯流排佇列。 程式碼範例以 PHP 撰寫。
services: service-bus-messaging
documentationcenter: php
author: spelluru
manager: timlt
editor: ''
ms.assetid: e29c829b-44c5-4350-8f2e-39e0c380a9f2
ms.service: service-bus-messaging
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: PHP
ms.topic: article
ms.date: 09/10/2018
ms.author: spelluru
ms.openlocfilehash: 08894741f4b7c4d3ccb808a4e70ec1eeb4f6af49
ms.sourcegitcommit: d1aef670b97061507dc1343450211a2042b01641
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/27/2018
ms.locfileid: "47392188"
---
# <a name="how-to-use-service-bus-queues-with-php"></a>如何將服務匯流排佇列搭配 PHP 使用
[!INCLUDE [service-bus-selector-queues](../../includes/service-bus-selector-queues.md)]

本指南將說明如何使用服務匯流排佇列。 這些範例均是以 PHP 撰寫，並使用 [Azure SDK for PHP](../php-download-sdk.md) (英文)。 本文說明的案例包括**建立佇列**、**傳送並接收訊息**，以及**刪除佇列**。

[!INCLUDE [howto-service-bus-queues](../../includes/howto-service-bus-queues.md)]

## <a name="create-a-php-application"></a>建立 PHP 應用程式
若要建立 PHP 應用程式並使其存取 Azure Blob 服務，唯一要求就是在您的程式碼中參考 [Azure SDK for PHP](../php-download-sdk.md) 中的類別。 您可以使用任何開發工具來建立應用程式，或記事本。

> [!NOTE]
> 您的 PHP 安裝也必須已安裝並啟用 [OpenSSL 延伸模組](http://php.net/openssl)。
> 
> 

在本指南中，您將使用可從 PHP 應用程式內本機呼叫的服務功能，或可從 Azure Web 角色、背景工作角色或網站內執行的程式碼中呼叫的資料表服務功能。

## <a name="get-the-azure-client-libraries"></a>取得 Azure 用戶端程式庫
[!INCLUDE [get-client-libraries](../../includes/get-client-libraries.md)]

## <a name="configure-your-application-to-use-service-bus"></a>設定應用程式以使用服務匯流排
若要使用服務匯流排佇列 API，請執行下列動作：

1. 使用 [require_once][require_once] 陳述式來參考自動換片器檔案。
2. 參考任何您可能使用的類別。

下列範例顯示如何納入自動換片器檔案及參考 `ServicesBuilder` 類別。

> [!NOTE]
> 此範例 (和本文中的其他範例) 假設您已透過編輯器安裝 PHP Client Libraries for Azure。 如果您以手動方式或以 PEAR 套件方式安裝程式庫，則必須參考 **WindowsAzure.php** 自動換片器檔案。
> 
> 

```php
require_once 'vendor/autoload.php';
use WindowsAzure\Common\ServicesBuilder;
```

在下列各範例中，一律會顯示 `require_once` 陳述式，但只會參考要執行之範例所需的類別。

## <a name="set-up-a-service-bus-connection"></a>設定服務匯流排連接
若要具現化服務匯流排用戶端，您必須具備符合下列格式的有效連接字串：

```
Endpoint=[yourEndpoint];SharedAccessKeyName=RootManageSharedAccessKey;SharedAccessKey=[Primary Key]
```

其中，`Endpoint` 的格式通常是 `[yourNamespace].servicebus.windows.net`。

若要建立任何 Azure 服務用戶端，您必須使用 `ServicesBuilder` 類別。 您可以：

* 直接將連接字串傳遞給它。
* 使用 **CloudConfigurationManager (CCM)** 到多種外部來源檢查連接字串：
  * 預設已支援一種外部來源，即環境變數
  * 您可以擴充 `ConnectionStringSource` 類別以加入新來源

在本文的各範例中，將會直接傳遞連接字串。

```php
require_once 'vendor/autoload.php';

use WindowsAzure\Common\ServicesBuilder;

$connectionString = "Endpoint=[yourEndpoint];SharedAccessKeyName=RootManageSharedAccessKey;SharedAccessKey=[Primary Key]";

$serviceBusRestProxy = ServicesBuilder::getInstance()->createServiceBusService($connectionString);
```

## <a name="create-a-queue"></a>建立佇列
您可以使用 `ServiceBusRestProxy` 類別來執行服務匯流排佇列的管理作業。 `ServiceBusRestProxy` 物件可透過 `ServicesBuilder::createServiceBusService` Factory 方法，使用含有權杖權限加以管理的適當連接字串來建構。

下列範例將說明如何具現化 `ServiceBusRestProxy` 並呼叫 `ServiceBusRestProxy->createQueue`，以便在 `MySBNamespace` 服務命名空間內建立名為 `myqueue` 的佇列：

```php
require_once 'vendor/autoload.php';

use WindowsAzure\Common\ServicesBuilder;
use WindowsAzure\Common\ServiceException;
use WindowsAzure\ServiceBus\Models\QueueInfo;

// Create Service Bus REST proxy.
$serviceBusRestProxy = ServicesBuilder::getInstance()->createServiceBusService($connectionString);

try    {
    $queueInfo = new QueueInfo("myqueue");

    // Create queue.
    $serviceBusRestProxy->createQueue($queueInfo);
}
catch(ServiceException $e){
    // Handle exception based on error codes and messages.
    // Error codes and messages are here: 
    // https://docs.microsoft.com/rest/api/storageservices/Common-REST-API-Error-Codes
    $code = $e->getCode();
    $error_message = $e->getMessage();
    echo $code.": ".$error_message."<br />";
}
```

> [!NOTE]
> 您可以在 `ServiceBusRestProxy` 物件上使用 `listQueues` 方法，來檢查命名空間內是否已有指定名稱的佇列存在。
> 
> 

## <a name="send-messages-to-a-queue"></a>傳送訊息至佇列
若要將訊息傳送至服務匯流排佇列，應用程式會呼叫 `ServiceBusRestProxy->sendQueueMessage` 方法。 下列程式碼示範如何將訊息傳送至先前在 `MySBNamespace` 服務命名空間中建立的 `myqueue` 佇列。

```php
require_once 'vendor/autoload.php';

use WindowsAzure\Common\ServicesBuilder;
use WindowsAzure\Common\ServiceException;
use WindowsAzure\ServiceBus\Models\BrokeredMessage;

// Create Service Bus REST proxy.
$serviceBusRestProxy = ServicesBuilder::getInstance()->createServiceBusService($connectionString);

try    {
    // Create message.
    $message = new BrokeredMessage();
    $message->setBody("my message");

    // Send message.
    $serviceBusRestProxy->sendQueueMessage("myqueue", $message);
}
catch(ServiceException $e){
    // Handle exception based on error codes and messages.
    // Error codes and messages are here: 
    // https://docs.microsoft.com/rest/api/storageservices/Common-REST-API-Error-Codes
    $code = $e->getCode();
    $error_message = $e->getMessage();
    echo $code.": ".$error_message."<br />";
}
```

傳送至 (和接收自) 服務匯流排佇列的訊息是 [BrokeredMessage][BrokeredMessage] 類別的執行個體。 [BrokeredMessage][BrokeredMessage] 物件具有一組標準方法和屬性，可用來保存自訂的應用程式特定屬性，以及任意的應用程式資料。

服務匯流排佇列支援的訊息大小上限：在[標準層](service-bus-premium-messaging.md)中為 256 KB 以及在[進階層](service-bus-premium-messaging.md)中為 1 MB。 標頭 (包含標準和自訂應用程式屬性) 可以容納 64 KB 的大小上限。 佇列中所保存的訊息數目沒有限制，但佇列所保存的訊息大小總計會有最高限制。 佇列大小的這項上限為 5 GB。

## <a name="receive-messages-from-a-queue"></a>從佇列接收訊息

從佇列接收訊息的最佳方式是使用 `ServiceBusRestProxy->receiveQueueMessage` 方法。 訊息可以兩種不同的模式接收：[*ReceiveAndDelete*](/dotnet/api/microsoft.servicebus.messaging.receivemode) 和 [*PeekLock*](/dotnet/api/microsoft.servicebus.messaging.receivemode#Microsoft_ServiceBus_Messaging_ReceiveMode_PeekLock)。 **PeekLock** 是預設值。

使用 [ReceiveAndDelete](/dotnet/api/microsoft.servicebus.messaging.receivemode) 模式時，接收是單發作業；也就是說，當服務匯流排收到佇列中訊息的讀取要求時，它會將此訊息標示為已使用，並將它傳回應用程式。 [ReceiveAndDelete](/dotnet/api/microsoft.servicebus.messaging.receivemode) 模式是最簡單的模型，且最適合可容許在發生失敗時不處理訊息的應用程式案例。 若要了解這一點，請考慮取用者發出接收要求，接著系統在處理此要求之前當機的案例。 因為服務匯流排會將訊息標示為已取用，當應用程式重新啟動並開始重新取用訊息時，它將會遺漏當機前已取用的訊息。

在預設的 [PeekLock](/dotnet/api/microsoft.servicebus.messaging.receivemode#Microsoft_ServiceBus_Messaging_ReceiveMode_PeekLock) 模式中，接收訊息會變成兩階段作業，因此可以支援無法容許遺漏訊息的應用程式。 當服務匯流排收到要求時，它會尋找要取用的下一個訊息、將其鎖定以防止其他取用者接收此訊息，然後將它傳回應用程式。 在應用程式完成處理訊息 (或可靠地儲存此訊息以供未來處理) 之後，它會將已接收的訊息傳遞至 `ServiceBusRestProxy->deleteMessage`，以完成接收程序的第二個階段。 當服務匯流排看到 `deleteMessage` 呼叫時，它會將訊息標示為已取用，並將它從佇列中移除。

下列範例說明如何使用 [PeekLock](/dotnet/api/microsoft.servicebus.messaging.receivemode#Microsoft_ServiceBus_Messaging_ReceiveMode_PeekLock) 模式 (預設模式) 來接收與處理訊息。

```php
require_once 'vendor/autoload.php';

use WindowsAzure\Common\ServicesBuilder;
use WindowsAzure\Common\ServiceException;
use WindowsAzure\ServiceBus\Models\ReceiveMessageOptions;

// Create Service Bus REST proxy.
$serviceBusRestProxy = ServicesBuilder::getInstance()->createServiceBusService($connectionString);

try    {
    // Set the receive mode to PeekLock (default is ReceiveAndDelete).
    $options = new ReceiveMessageOptions();
    $options->setPeekLock();

    // Receive message.
    $message = $serviceBusRestProxy->receiveQueueMessage("myqueue", $options);
    echo "Body: ".$message->getBody()."<br />";
    echo "MessageID: ".$message->getMessageId()."<br />";

    /*---------------------------
        Process message here.
    ----------------------------*/

    // Delete message. Not necessary if peek lock is not set.
    echo "Message deleted.<br />";
    $serviceBusRestProxy->deleteMessage($message);
}
catch(ServiceException $e){
    // Handle exception based on error codes and messages.
    // Error codes and messages are here:
    // https://docs.microsoft.com/rest/api/storageservices/Common-REST-API-Error-Codes
    $code = $e->getCode();
    $error_message = $e->getMessage();
    echo $code.": ".$error_message."<br />";
}
```

## <a name="how-to-handle-application-crashes-and-unreadable-messages"></a>如何處理應用程式當機與無法讀取的訊息

服務匯流排提供一種功能，可協助您從應用程式的錯誤或處理訊息的問題中順利復原。 如果接收者應用程式因為某些原因無法處理訊息，它可以在接收訊息上呼叫 `unlockMessage` 方法 (而不是 `deleteMessage` 方法)。 這將導致服務匯流排將佇列中的訊息解除鎖定，讓此訊息可以被相同取用應用程式或其他取用應用程式重新接收。

與在佇列內鎖定之訊息相關的還有逾時，如果應用程式無法在鎖定逾時到期之前處理訊息 (例如，如果應用程式當機)，則服務匯流排會自動解除鎖定訊息，並讓訊息可以被重新接收。

如果應用程式在處理訊息之後，但尚未發出 `deleteMessage` 要求時當機，則會在應用程式重新啟動時將訊息重新傳遞給該應用程式。 這通常稱為「至少處理一次」，也就是說，每個訊息至少會被處理一次，但在特定狀況下，可能會重新傳遞相同訊息。 如果案例無法容許重複處理，建議您在應用程式中新增其他邏輯，以處理重複的訊息傳遞。 通常您可使用訊息的 `getMessageId` 方法來達到此目的，該方法在各個傳遞嘗試中保持不變。

## <a name="next-steps"></a>後續步驟
現在您已了解服務匯流排佇列的基本概念，請參閱[佇列、主題和訂用帳戶][Queues, topics, and subscriptions]，以取得詳細資訊。

如需詳細資訊，另請造訪 [PHP 開發人員中心](https://azure.microsoft.com/develop/php/)。

[BrokeredMessage]: /dotnet/api/microsoft.servicebus.messaging.brokeredmessage
[Queues, topics, and subscriptions]: service-bus-queues-topics-subscriptions.md
[require_once]: http://php.net/require_once


