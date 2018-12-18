---
title: Azure 服務匯流排與 Event Grid 的整合概觀 | Microsoft Docs
description: 描述服務匯流排傳訊與 Event Grid 的整合
services: service-bus-messaging
documentationcenter: .net
author: spelluru
manager: timlt
editor: ''
ms.assetid: f99766cb-8f4b-4baf-b061-4b1e2ae570e4
ms.service: service-bus-messaging
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: multiple
ms.topic: get-started-article
ms.date: 09/15/2018
ms.author: spelluru
ms.openlocfilehash: 2173a24eb3bbaafe9dcf25461eb5357ba3a6e7ad
ms.sourcegitcommit: c282021dbc3815aac9f46b6b89c7131659461e49
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/12/2018
ms.locfileid: "49166894"
---
# <a name="azure-service-bus-to-event-grid-integration-overview"></a>Azure 服務匯流排與 Event Grid 的整合概觀

Azure 服務匯流排已開始與 Azure Event Grid 進行全新整合。 此功能的主要案例就是具有少量訊息的服務匯流排佇列或訂用帳戶，不必讓接收者持續輪詢訊息。 

當佇列或訂用帳戶中有訊息，但沒有任何接收者存在時，服務匯流排現在可以將事件發出至 Event Grid。 您可以在服務匯流排命名空間中建立 Event Grid 訂用帳戶，接聽這些事件，然後啟動接收者來回應事件。 利用此功能，您可以在反應靈敏的程式設計模型中使用服務匯流排。

若要啟用此功能，您需要下列項目：

* 具有至少一個服務匯流排佇列的服務匯流排進階命名空間或具有至少一個訂用帳戶的服務匯流排主題。
* 服務匯流排命名空間的參與者存取權。
* 此外，您還需要適用於服務匯流排命名空間的 Event Grid 訂用帳戶。 此訂用帳戶會從有訊息可供挑選的 Event Grid 接收通知。 典型訂閱者可能是 Azure App Service 的 Logic Apps 功能、Azure Functions，或聯繫 Web 應用程式的 Web Hook。 訂閱者會接著處理訊息。 

![19][]

### <a name="verify-that-you-have-contributor-access"></a>確認您具有參與者存取權

移至您的服務匯流排命名空間，然後選取如下所示的 [存取控制 (IAM)]：

![1][]

### <a name="events-and-event-schemas"></a>事件和事件結構描述

服務匯流排目前會針對兩種案例傳送事件：

* [ActiveMessagesWithNoListenersAvailable](#active-messages-available-event)
* [DeadletterMessagesAvailable](#dead-lettered-messages-available-event)

此外服務匯流排會使用標準 Event Grid 安全性和[驗證機制](https://docs.microsoft.com/azure/event-grid/security-authentication)。

如需詳細資訊，請參閱 [Azure Event Grid 事件結構描述](https://docs.microsoft.com/azure/event-grid/event-schema)。

#### <a name="active-messages-available-event"></a>作用中訊息可用事件

如果佇列或訂用帳戶有作用中的訊息，但沒有接聽的接收者，就會產生此事件。

此事件的結構描述如下所示︰

```JSON
{
  "topic": "/subscriptions/<subscription id>/resourcegroups/DemoGroup/providers/Microsoft.ServiceBus/namespaces/<YOUR SERVICE BUS NAMESPACE WILL SHOW HERE>",
  "subject": "topics/<service bus topic>/subscriptions/<service bus subscription>",
  "eventType": "Microsoft.ServiceBus.ActiveMessagesAvailableWithNoListeners",
  "eventTime": "2018-02-14T05:12:53.4133526Z",
  "id": "dede87b0-3656-419c-acaf-70c95ddc60f5",
  "data": {
    "namespaceName": "YOUR SERVICE BUS NAMESPACE WILL SHOW HERE",
    "requestUri": "https://YOUR-SERVICE-BUS-NAMESPACE-WILL-SHOW-HERE.servicebus.windows.net/TOPIC-NAME/subscriptions/SUBSCRIPTIONNAME/messages/head",
    "entityType": "subscriber",
    "queueName": "QUEUE NAME IF QUEUE",
    "topicName": "TOPIC NAME IF TOPIC",
    "subscriptionName": "SUBSCRIPTION NAME"
  },
  "dataVersion": "1",
  "metadataVersion": "1"
}
```

#### <a name="dead-letter-messages-available-event"></a>無法傳送的訊息可用事件

您可對每個無效信件佇列取得至少一個事件，而此種佇列具有訊息，但沒有作用中的接收者。

此事件的結構描述如下所示︰

```JSON
[{
  "topic": "/subscriptions/<subscription id>/resourcegroups/DemoGroup/providers/Microsoft.ServiceBus/namespaces/<YOUR SERVICE BUS NAMESPACE WILL SHOW HERE>",
  "subject": "topics/<service bus topic>/subscriptions/<service bus subscription>",
  "eventType": "Microsoft.ServiceBus.DeadletterMessagesAvailableWithNoListener",
  "eventTime": "2018-02-14T05:12:53.4133526Z",
  "id": "dede87b0-3656-419c-acaf-70c95ddc60f5",
  "data": {
    "namespaceName": "YOUR SERVICE BUS NAMESPACE WILL SHOW HERE",
    "requestUri": "https://YOUR-SERVICE-BUS-NAMESPACE-WILL-SHOW-HERE.servicebus.windows.net/TOPIC-NAME/subscriptions/SUBSCRIPTIONNAME/$deadletterqueue/messages/head",
    "entityType": "subscriber",
    "queueName": "QUEUE NAME IF QUEUE",
    "topicName": "TOPIC NAME IF TOPIC",
    "subscriptionName": "SUBSCRIPTION NAME"
  },
  "dataVersion": "1",
  "metadataVersion": "1"
}]
```

### <a name="how-many-events-are-emitted-and-how-often"></a>發出事件的數量和頻率為何？

如果命名空間中有多個佇列和主題或訂用帳戶，您會對每個佇列和每個訂用帳戶取得至少一個事件。 如果服務匯流排實體中沒有任何訊息，而且有一則新訊息送達，系統就會立即發出事件。 除非服務匯流排偵測到作用中的接收者，否則會每兩分鐘發出事件。 瀏覽訊息並不會干擾事件。

依預設，服務匯流排會針對命名空間中的所有實體發出事件。 如果您只想取得特定實體的事件，請參閱下一節。

### <a name="use-filters-to-limit-where-you-get-events-from"></a>使用篩選條件來限制您取得事件的來源

如果您只想從命名空間內的一個佇列或一個訂用帳戶取得事件，您可以使用 Event Grid 所提供的「開頭為」或「結尾為」篩選條件。 在某些介面中，篩選條件稱為「前置」和「尾碼」篩選條件。 如果您想取得多個佇列和訂用帳戶 (但並非所有佇列和訂用帳戶) 的事件，您可以建立多個 Event Grid 訂用帳戶並且為每個訂用帳戶提供一個篩選條件。

## <a name="create-event-grid-subscriptions-for-service-bus-namespaces"></a>如何為服務匯流排命名空間建立 Event Grid 訂用帳戶

為服務匯流排命名空間建立 Event Grid 訂用帳戶的方式有三種：

* 在 [Azure 入口網站](#portal-instructions)中
* 在 [Azure CLI](#azure-cli-instructions) 中
* 在 [PowerShell](#powershell-instructions) 中

## <a name="azure-portal-instructions"></a>Azure 入口網站指示

若要建立新的 Event Grid 訂用帳戶，請執行下列作業：
1. 在 Azure 入口網站中，移至您的命名空間。
2. 在左窗格中，選取 [Event Grid]。 
3. 選取 [事件訂用帳戶]。  

   下圖顯示一個命名空間，而該命名空間有事件方格訂用帳戶：

   ![事件方格訂用帳戶](./media/service-bus-to-event-grid-integration-concept/sbtoeventgridportal.png)

   下圖顯示如何在不使用任何特定篩選條件的情況下，訂閱函式或 Web Hook：

   ![21][]

## <a name="azure-cli-instructions"></a>Azure CLI 指示

首先，確定已安裝 Azure CLI 2.0 版或更新版本。 [下載安裝程式](https://docs.microsoft.com/cli/azure/install-azure-cli?view=azure-cli-latest)。 選取 [Windows + X]，然後以系統管理員權限開啟新的 PowerShell 主控台。 或者，也可以在 Azure 入口網站中使用命令 shell。

執行以下程式碼：

 ```azurecli-interactive
az login

az account set -s “THE SUBSCRIPTION YOU WANT TO USE”

$namespaceid=(az resource show --namespace Microsoft.ServiceBus --resource-type namespaces --name “<yourNamespace>“--resource-group “<Your Resource Group Name>” --query id --output tsv)

az eventgrid event-subscription create --resource-id $namespaceid --name “<YOUR EVENT GRID SUBSCRIPTION NAME (CAN BE ANY NOT EXISTING)>” --endpoint “<your_function_url>” --subject-ends-with “<YOUR SERVICE BUS SUBSCRIPTION NAME>”
```

## <a name="powershell-instructions"></a>PowerShell 指示

確定您已安裝 Azure PowerShell。 [下載安裝程式](https://docs.microsoft.com/powershell/azure/install-azurerm-ps?view=azurermps-5.4.0)。 選取 [Windows + X]，然後以系統管理員權限開啟新的 PowerShell 主控台。 或者，也可以在 Azure 入口網站中使用命令 shell。

```PowerShell-interactive
Connect-AzureRmAccount

Select-AzureRmSubscription -SubscriptionName "<YOUR SUBSCRIPTION NAME>"

# This might be installed already
Install-Module AzureRM.ServiceBus

$NSID = (Get-AzureRmServiceBusNamespace -ResourceGroupName "<YOUR RESOURCE GROUP NAME>" -Na
mespaceName "<YOUR NAMESPACE NAME>").Id

New-AzureRmEVentGridSubscription -EventSubscriptionName “<YOUR EVENT GRID SUBSCRIPTION NAME (CAN BE ANY NOT EXISTING)>” -ResourceId $NSID -Endpoint "<YOUR FUNCTION URL>” -SubjectEndsWith “<YOUR SERVICE BUS SUBSCRIPTION NAME>”
```

從這裡，您可以瀏覽其他安裝程式選項或[測試事件是否流動](#test-that-events-are-flowing)。

## <a name="next-steps"></a>後續步驟

* 取得服務匯流排和 Event Grid [範例](service-bus-to-event-grid-integration-example.md)。
* 深入了解 [Event Grid](https://docs.microsoft.com/azure/event-grid/)。
* 深入了解 [Azure Functions](https://docs.microsoft.com/azure/azure-functions/)。
* 深入了解 [Logic Apps](https://docs.microsoft.com/azure/logic-apps/)。
* 深入了解[服務匯流排](https://docs.microsoft.com/azure/service-bus/)。

[1]: ./media/service-bus-to-event-grid-integration-concept/sbtoeventgrid1.png
[19]: ./media/service-bus-to-event-grid-integration-concept/sbtoeventgriddiagram.png
[8]: ./media/service-bus-to-event-grid-integration-example/sbtoeventgrid8.png
[9]: ./media/service-bus-to-event-grid-integration-example/sbtoeventgrid9.png
[20]: ./media/service-bus-to-event-grid-integration-example/sbtoeventgridportal.png
[21]: ./media/service-bus-to-event-grid-integration-example/sbtoeventgridportal2.png
