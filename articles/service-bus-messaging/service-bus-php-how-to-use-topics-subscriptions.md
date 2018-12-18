---
title: 如何將服務匯流排主題搭配 PHP 使用 | Microsoft Docs
description: 了解如何在 Azure 中搭配使用服務匯流排主題與 PHP。
services: service-bus-messaging
documentationcenter: php
author: spelluru
manager: timlt
editor: ''
ms.assetid: faaa4bbd-f6ef-42ff-aca7-fc4353976449
ms.service: service-bus-messaging
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: PHP
ms.topic: article
ms.date: 09/06/2018
ms.author: spelluru
ms.openlocfilehash: 8b2cd62d9f1c2010956604a9f3c753d893f7c2ad
ms.sourcegitcommit: b7e5bbbabc21df9fe93b4c18cc825920a0ab6fab
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/27/2018
ms.locfileid: "47407275"
---
# <a name="how-to-use-service-bus-topics-and-subscriptions-with-php"></a>如何透過 PHP 使用服務匯流排主題和訂用帳戶

[!INCLUDE [service-bus-selector-topics](../../includes/service-bus-selector-topics.md)]

此文章示範如何使用服務匯流排主題和訂用帳戶。 這些範例均是以 PHP 撰寫，並使用 [Azure SDK for PHP](../php-download-sdk.md) (英文)。 涵蓋的案例包括**建立主題和訂用帳戶**、**建立訂用帳戶篩選器**、**傳送訊息至主題**、**接收訂用帳戶的訊息**，以及**刪除主題和訂用帳戶**。

[!INCLUDE [howto-service-bus-topics](../../includes/howto-service-bus-topics.md)]

## <a name="create-a-php-application"></a>建立 PHP 應用程式
若要建立 PHP 應用程式並使其存取 Azure Blob 服務，唯一要求就是在您的程式碼中參考 [Azure SDK for PHP](../php-download-sdk.md) 中的類別。 您可以使用任何開發工具來建立應用程式，或記事本。

> [!NOTE]
> 您的 PHP 安裝也必須已安裝並啟用 [OpenSSL 延伸模組](http://php.net/openssl)。
> 
> 

此文章說明如何使用可從 PHP 應用程式內本機呼叫的服務功能，或可在 Azure Web 角色、背景工作角色或網站內執行的程式碼中呼叫的服務功能。

## <a name="get-the-azure-client-libraries"></a>取得 Azure 用戶端程式庫
[!INCLUDE [get-client-libraries](../../includes/get-client-libraries.md)]

## <a name="configure-your-application-to-use-service-bus"></a>設定應用程式以使用服務匯流排
若要使用服務匯流排 API：

1. 使用 [require_once][require-once] 陳述式來參考自動換片器檔案。
2. 參考任何您可能使用的類別。

下列範例說明如何納入自動換片器檔案及參考 **ServiceBusService** 類別。

> [!NOTE]
> 此範例 (和此文章中的其他範例) 假設您已透過編輯器安裝 PHP Client Libraries for Azure。 如果您以手動方式或以 PEAR 套件方式安裝程式庫，則必須參考 **WindowsAzure.php** 自動換片器檔案。
> 
> 

```php
require_once 'vendor\autoload.php';
use WindowsAzure\Common\ServicesBuilder;
```

在下列各範例中，一律會顯示 `require_once` 陳述式，但只會參考要執行之範例所需的類別。

## <a name="set-up-a-service-bus-connection"></a>設定服務匯流排連接
若要具現化服務匯流排用戶端，您必須具備符合下列格式的有效連接字串：

```
Endpoint=[yourEndpoint];SharedAccessKeyName=RootManageSharedAccessKey;SharedAccessKey=[Primary Key]
```

其中，`Endpoint` 的格式通常是 `https://[yourNamespace].servicebus.windows.net`。

若要建立任何 Azure 服務用戶端，您必須使用 `ServicesBuilder` 類別。 您可以：

* 直接將連接字串傳遞給它。
* 使用 **CloudConfigurationManager (CCM)** 到多種外部來源檢查連接字串：
  * 預設已支援一種外部來源，即環境變數。
  * 您可以擴充 `ConnectionStringSource` 類別以加入新來源。

在此文章的各範例中，將會直接傳遞連接字串。

```php
require_once 'vendor/autoload.php';

use WindowsAzure\Common\ServicesBuilder;

$connectionString = "Endpoint=[yourEndpoint];SharedAccessKeyName=RootManageSharedAccessKey;SharedAccessKey=[Primary Key]";

$serviceBusRestProxy = ServicesBuilder::getInstance()->createServiceBusService($connectionString);
```

## <a name="create-a-topic"></a>建立主題
您可以透過 `ServiceBusRestProxy` 類別來執行服務匯流排主題的管理作業。 `ServiceBusRestProxy` 物件可透過 `ServicesBuilder::createServiceBusService` Factory 方法，使用含有權杖權限以管理它的適當連接字串來建構。

下列範例將說明如何具現化 `ServiceBusRestProxy` 並呼叫 `ServiceBusRestProxy->createTopic`，以便在 `MySBNamespace` 命名空間內建立名為 `mytopic` 的主題：

```php
require_once 'vendor/autoload.php';

use WindowsAzure\Common\ServicesBuilder;
use WindowsAzure\Common\ServiceException;
use WindowsAzure\ServiceBus\Models\TopicInfo;

// Create Service Bus REST proxy.
$serviceBusRestProxy = ServicesBuilder::getInstance()->createServiceBusService($connectionString);

try    {        
    // Create topic.
    $topicInfo = new TopicInfo("mytopic");
    $serviceBusRestProxy->createTopic($topicInfo);
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
> 您可以在 `ServiceBusRestProxy` 物件上使用 `listTopics` 方法，檢查服務命名空間內是否已有指定名稱的主題存在。
> 
> 

## <a name="create-a-subscription"></a>建立訂用帳戶
`ServiceBusRestProxy->createSubscription` 方法也能用來建立主題訂用帳戶。 為訂閱命名，且能包含選擇性篩選器，以用來限制傳遞至訂閱的虛擬佇列的訊息集合。

### <a name="create-a-subscription-with-the-default-matchall-filter"></a>使用預設 (MatchAll) 篩選器建立訂用帳戶
如果在建立新的訂用帳戶時未指定篩選器，將會使用 **MatchAll** 篩選器 (預設)。 使用 **MatchAll** 篩選器時，所有發佈至主題的訊息都會被置於訂用帳戶的虛擬佇列中。 下列範例將建立名為 'mysubscription' 的訂用帳戶，並使用預設的 **MatchAll** 篩選器。

```php
require_once 'vendor/autoload.php';

use WindowsAzure\Common\ServicesBuilder;
use WindowsAzure\Common\ServiceException;
use WindowsAzure\ServiceBus\Models\SubscriptionInfo;

// Create Service Bus REST proxy.
$serviceBusRestProxy = ServicesBuilder::getInstance()->createServiceBusService($connectionString);

try    {
    // Create subscription.
    $subscriptionInfo = new SubscriptionInfo("mysubscription");
    $serviceBusRestProxy->createSubscription("mytopic", $subscriptionInfo);
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

### <a name="create-subscriptions-with-filters"></a>使用篩選器建立訂用帳戶
您也可以設定篩選器，讓您指定傳送至主題的哪些訊息應出現在特定主題訂用帳戶中。 訂用帳戶所支援的最具彈性篩選器類型是實作 SQL92 子集的 [SqlFilter](/dotnet/api/microsoft.servicebus.messaging.sqlfilter#microsoft_servicebus_messaging_sqlfilter)。 SQL 篩選器會對發佈至主題之訊息的屬性運作。 如需 SqlFilters 的詳細資訊，請參閱 [SqlFilter.SqlExpression 屬性][sqlfilter]。

> [!NOTE]
> 訂用帳戶的每個規則可獨立處理傳入的訊息，並將其結果訊息新增至訂用帳戶。 此外，每個新訂用帳戶都有具篩選器的預設 **Rule** 物件，而篩選器會將主題中的所有訊息新增至訂用帳戶。 To receive only messages matching your filter, you must remove the default rule. 您可以使用 `ServiceBusRestProxy->deleteRule` 方法以移除預設規則。
> 
> 

以下範例將建立名為 `HighMessages` 的訂用帳戶，而且所含的 **SqlFilter** 只會選取自訂 `MessageNumber` 屬性大於 3 的訊息。 請參閱[傳送訊息至主題](#send-messages-to-a-topic)，以取得將自訂屬性新增至訊息的相關資訊。

```php
$subscriptionInfo = new SubscriptionInfo("HighMessages");
$serviceBusRestProxy->createSubscription("mytopic", $subscriptionInfo);

$serviceBusRestProxy->deleteRule("mytopic", "HighMessages", '$Default');

$ruleInfo = new RuleInfo("HighMessagesRule");
$ruleInfo->withSqlFilter("MessageNumber > 3");
$ruleResult = $serviceBusRestProxy->createRule("mytopic", "HighMessages", $ruleInfo);
```

此程式碼必須使用額外的命名空間：`WindowsAzure\ServiceBus\Models\SubscriptionInfo`。

同樣地，下列範例將建立名為 `LowMessages` 的訂用帳戶，而且所含的 `SqlFilter` 只會選取 `MessageNumber` 屬性小於或等於 3 的訊息。

```php
$subscriptionInfo = new SubscriptionInfo("LowMessages");
$serviceBusRestProxy->createSubscription("mytopic", $subscriptionInfo);

$serviceBusRestProxy->deleteRule("mytopic", "LowMessages", '$Default');

$ruleInfo = new RuleInfo("LowMessagesRule");
$ruleInfo->withSqlFilter("MessageNumber <= 3");
$ruleResult = $serviceBusRestProxy->createRule("mytopic", "LowMessages", $ruleInfo);
```

現在，當訊息傳送至 `mytopic` 主題時，一律會將該訊息傳遞至已訂閱 `mysubscription` 訂用帳戶的接收者，並選擇性地傳遞至已訂閱 `HighMessages` 和 `LowMessages` 訂用帳戶的接收者 (視訊息內容而定)。

## <a name="send-messages-to-a-topic"></a>傳送訊息至主題
若要將訊息傳送至服務匯流排主題，應用程式會呼叫 `ServiceBusRestProxy->sendTopicMessage` 方法。 下列程式碼示範如何將訊息傳送至先前在 `MySBNamespace` 服務命名空間中建立的 `mytopic` 主題。

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
    $serviceBusRestProxy->sendTopicMessage("mytopic", $message);
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

傳送至服務匯流排主題的訊息是 [BrokeredMessage][BrokeredMessage] 類別的執行個體。 [BrokeredMessage][BrokeredMessage] 物件具有一組標準屬性和方法，以及可用來保存自訂應用程式特定屬性的屬性。 下列範例示範如何將五則測試訊息傳送至先前建立的 `mytopic` 主題。 `setProperty` 方法會用來將自訂屬性 (`MessageNumber`) 新增至每個訊息。 每個訊息的 `MessageNumber` 屬性值有不同之處 (您可以使用此值來判斷哪些訂用帳戶會接收訊息，如[建立訂用帳戶](#create-a-subscription)一節所述)：

```php
for($i = 0; $i < 5; $i++){
    // Create message.
    $message = new BrokeredMessage();
    $message->setBody("my message ".$i);

    // Set custom property.
    $message->setProperty("MessageNumber", $i);

    // Send message.
    $serviceBusRestProxy->sendTopicMessage("mytopic", $message);
}
```

服務匯流排主題支援的訊息大小上限：在[標準層](service-bus-premium-messaging.md)中為 256 KB 以及在[進階層](service-bus-premium-messaging.md)中為 1 MB。 標頭 (包含標準和自訂應用程式屬性) 可以容納 64 KB 的大小上限。 主題中所保存的訊息數目沒有限制，但主題所保存的訊息大小總計會有最高限制。 主題大小的這個上限為 5 GB。 如需有關配額的詳細資訊，請參閱[服務匯流排配額][Service Bus quotas]。

## <a name="receive-messages-from-a-subscription"></a>自訂用帳戶接收訊息
從訂用帳戶接收訊息的最佳方式是使用 `ServiceBusRestProxy->receiveSubscriptionMessage` 方法。 訊息可以兩種不同的模式接收：[*ReceiveAndDelete* 和 *PeekLock*](https://docs.microsoft.com/dotnet/api/microsoft.servicebus.messaging.receivemode)。 **PeekLock** 是預設值。

使用 [ReceiveAndDelete](https://docs.microsoft.com/dotnet/api/microsoft.servicebus.messaging.receivemode) 模式時，接收是一次性作業；也就是說，當服務匯流排在訂用帳戶中收到訊息的讀取要求時，它會將此訊息標示為已使用，並將它傳回應用程式。 [ReceiveAndDelete](https://docs.microsoft.com/dotnet/api/microsoft.servicebus.messaging.receivemode) * 模式是最簡單的模型，且最適合可容許在發生失敗時不處理訊息的應用程式案例。 若要了解這一點，請考慮取用者發出接收要求，接著系統在處理此要求之前當機的案例。 因為服務匯流排已將訊息標示為已取用，所以，當應用程式重新啟動並開始重新取用訊息時，它會遺漏當機前已取用的訊息。

在預設的 [PeekLock](https://docs.microsoft.com/dotnet/api/microsoft.servicebus.messaging.receivemode) 模式中，接收訊息會變成兩階段作業，因此可以支援無法容許遺漏訊息的應用程式。 當服務匯流排收到要求時，它會尋找要取用的下一個訊息、將其鎖定以防止其他取用者接收此訊息，然後將它傳回應用程式。 在應用程式完成處理訊息 (或可靠地儲存此訊息以供未來處理) 之後，它會將已接收的訊息傳遞至 `ServiceBusRestProxy->deleteMessage`，以完成接收程序的第二個階段。 當服務匯流排看到 `deleteMessage` 呼叫時，它會將訊息標示為已取用，並將它從佇列中移除。

下列範例說明如何使用 [PeekLock](https://docs.microsoft.com/dotnet/api/microsoft.servicebus.messaging.receivemode) 模式 (預設模式) 來接收與處理訊息。 

```php
require_once 'vendor/autoload.php';

use WindowsAzure\Common\ServicesBuilder;
use WindowsAzure\Common\ServiceException;
use WindowsAzure\ServiceBus\Models\ReceiveMessageOptions;

// Create Service Bus REST proxy.
$serviceBusRestProxy = ServicesBuilder::getInstance()->createServiceBusService($connectionString);

try    {
    // Set receive mode to PeekLock (default is ReceiveAndDelete)
    $options = new ReceiveMessageOptions();
    $options->setPeekLock();

    // Get message.
    $message = $serviceBusRestProxy->receiveSubscriptionMessage("mytopic", "mysubscription", $options);

    echo "Body: ".$message->getBody()."<br />";
    echo "MessageID: ".$message->getMessageId()."<br />";

    /*---------------------------
        Process message here.
    ----------------------------*/

    // Delete message. Not necessary if peek lock is not set.
    echo "Deleting message...<br />";
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

與在佇列內鎖定訊息相關的還有逾時，如果應用程式無法在鎖定逾時到期之前處理訊息 (例如，如果應用程式當機)，則服務匯流排會自動解除鎖定訊息，並讓訊息可以被重新接收。

如果應用程式在處理訊息之後，但尚未發出 `deleteMessage` 要求時當機，則會在應用程式重新啟動時將訊息重新傳遞給該應用程式。 這種處理類型通常稱為*至少處理一次*，也就是說，每個訊息至少會被處理一次，但在特定狀況下，可能會重新傳遞相同訊息。 如果案例無法容許重複處理，則應用程式開發人員應在應用程式中加入其他邏輯，以處理重複的訊息傳遞。 通常您可使用訊息的 `getMessageId` 方法來達到此目的，該方法在各個傳遞嘗試中保持不變。

## <a name="delete-topics-and-subscriptions"></a>刪除主題和訂用帳戶
若要刪除主題或訂用帳戶，請分別使用 `ServiceBusRestProxy->deleteTopic` 或 `ServiceBusRestProxy->deleteSubscripton` 方法。 刪除主題也將會刪除對主題註冊的任何訂用帳戶。

下列範例示範如何刪除名為 `mytopic` 的主題及其註冊的訂用帳戶。

```php
require_once 'vendor/autoload.php';

use WindowsAzure\ServiceBus\ServiceBusService;
use WindowsAzure\ServiceBus\ServiceBusSettings;
use WindowsAzure\Common\ServiceException;

// Create Service Bus REST proxy.
$serviceBusRestProxy = ServicesBuilder::getInstance()->createServiceBusService($connectionString);

try    {        
    // Delete topic.
    $serviceBusRestProxy->deleteTopic("mytopic");
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

使用 `deleteSubscription` 方法可讓您個別刪除某個訂用帳戶：

```php
$serviceBusRestProxy->deleteSubscription("mytopic", "mysubscription");
```

## <a name="next-steps"></a>後續步驟
如需詳細資訊，請參閱[佇列、主題和訂用帳戶][Queues, topics, and subscriptions]。

[BrokeredMessage]: /dotnet/api/microsoft.servicebus.messaging.brokeredmessage
[Queues, topics, and subscriptions]: service-bus-queues-topics-subscriptions.md
[sqlfilter]: /dotnet/api/microsoft.servicebus.messaging.sqlfilter#microsoft_servicebus_messaging_sqlfilter
[require-once]: http://php.net/require_once
[Service Bus quotas]: service-bus-quotas.md
