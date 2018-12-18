---
title: 將 Azure 事件方格的自訂事件傳送至混合式連線 | Microsoft Docs
description: 使用 Azure 事件格線和 Azure CLI 來發佈主題，以及訂閱該事件。 混合式連線用於端點。
services: event-grid
keywords: ''
author: tfitzmac
ms.author: tomfitz
ms.date: 10/02/2018
ms.topic: tutorial
ms.service: event-grid
ms.openlocfilehash: d56a07bf6fcb368f50e081a1f56b7cfb022c05ca
ms.sourcegitcommit: 3856c66eb17ef96dcf00880c746143213be3806a
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/02/2018
ms.locfileid: "48042235"
---
# <a name="route-custom-events-to-azure-relay-hybrid-connections-with-azure-cli-and-event-grid"></a>使用 Azure CLI 和事件方格將自訂事件路由至 Azure 轉送混合式連線

Azure Event Grid 是一項雲端事件服務。 Azure 轉送混合式連線是支援的事件處理常式之一。 當您需要從沒有公用端點的應用程式處理事件時，您可以使用混式連線作為事件處理常式。 這些應用程式可能位在您公司的企業網路中。 在本文中，您可使用 Azure CLI 建立自訂主題、訂閱自訂主題，以及觸發事件來檢視結果。 您將事件傳送到混合式連線。

## <a name="prerequisites"></a>必要條件

本文假設您已經有混合式連線和接聽項應用程式。 若要開始使用混合式連線，請參閱[開始使用轉送混合式連線 - .NET](../service-bus-relay/relay-hybrid-connections-dotnet-get-started.md) (英文)，或[開始使用轉送混合式連線 - 節點](../service-bus-relay/relay-hybrid-connections-node-get-started.md) (英文)。

[!INCLUDE [event-grid-preview-feature-note.md](../../includes/event-grid-preview-feature-note.md)]

## <a name="create-a-resource-group"></a>建立資源群組

Event Grid 為 Azure 資源，必須放入 Azure 資源群組中。 資源群組是在其中部署與管理 Azure 資源的邏輯集合。

使用 [az group create](/cli/azure/group#az-group-create) 命令來建立資源群組。 

下列範例會在 westus2 位置建立名為 gridResourceGroup 的資源群組。

```azurecli-interactive
az group create --name gridResourceGroup --location westus2
```

## <a name="create-a-custom-topic"></a>建立自訂主題

Event Grid 主題會提供使用者定義的端點，作為您發佈事件的目的地。 下列範例可在您的資源群組中建立自訂主題。 以自訂主題的唯一名稱取代 `<topic_name>`。 事件方格的主題名稱必須是唯一的，因為它由 DNS 項目表示。

```azurecli-interactive
# if you have not already installed the extension, do it now.
# This extension is required for preview features.
az extension add --name eventgrid

az eventgrid topic create --name <topic_name> -l westus2 -g gridResourceGroup
```

## <a name="subscribe-to-a-custom-topic"></a>訂閱自訂主題

您可訂閱事件方格主題，告知事件方格您想要追蹤的事件。下列範例可訂閱您所建立的自訂主題，以及傳遞端點的混合式連線資源識別碼。 混合式連線識別碼的格式為：

`/subscriptions/<subscription-id>/resourceGroups/<resource-group-name>/providers/Microsoft.Relay/namespaces/<relay-namespace>/hybridConnections/<hybrid-connection-name>`

下列指令碼會取得轉送命名空間的資源識別碼。 它會建構混合式連線的識別碼，並訂閱事件方格主題。 此指令碼會將端點類型設為 `hybridconnection`，並為端點使用混合型連線識別碼。

```azurecli-interactive
relayname=<namespace-name>
relayrg=<resource-group-for-relay>
hybridname=<hybrid-name>

relayid=$(az resource show --name $relayname --resource-group $relayrg --resource-type Microsoft.Relay/namespaces --query id --output tsv)
hybridid="$relayid/hybridConnections/$hybridname"

az eventgrid event-subscription create \
  --topic-name <topic_name> \
  -g gridResourceGroup \
  --name <event_subscription_name> \
  --endpoint-type hybridconnection \
  --endpoint $hybridid
```

## <a name="create-application-to-process-events"></a>建立應用程式來處理事件

您需要可以從混合式連線擷取事件的應用程式。 [適用於 C# 的 Microsoft Azure Event Grid 混合式連線取用者範例](https://github.com/Azure-Samples/event-grid-dotnet-hybridconnection-destination)會執行該作業。 您已經完成必要的步驟。

1. 確定您有 Visual Studio 2017 Version 15.5 或更新版本。

1. 將存放庫複製到本機電腦。

1. 在 Visual Studio 中載入 HybridConnectionConsumer 專案。

1. 在 Program.cs 中，以您建立的轉送連接字串和混合式連線名稱取代 `<relayConnectionString>` 和 `<hybridConnectionName>`。

1. 從 Visual Studio 編譯和執行應用程式。

## <a name="send-an-event-to-your-topic"></a>將事件傳送至主題

讓我們觸發事件以了解 Event Grid 如何將訊息散發至您的端點。 本文說明如何使用 Azure CLI 來觸發事件。 或者，您可以使用 [Event Grid 發行者應用程式](https://github.com/Azure-Samples/event-grid-dotnet-publish-consume-events/tree/master/EventGridPublisher)。

首先，讓我們取得自訂主題的 URL 和金鑰。 再次，將您的自訂主題名稱用於 `<topic_name>`。

```azurecli-interactive
endpoint=$(az eventgrid topic show --name <topic_name> -g gridResourceGroup --query "endpoint" --output tsv)
key=$(az eventgrid topic key list --name <topic_name> -g gridResourceGroup --query "key1" --output tsv)
```

若要簡化這篇文章，您可使用要傳送至自訂主題的範例事件資料。 一般而言，應用程式或 Azure 服務就會傳送事件資料。 CURL 是可傳送 HTTP 要求的公用程式。 本文使用 CURL 將事件傳送到自訂主題。

```azurecli-interactive
event='[ {"id": "'"$RANDOM"'", "eventType": "recordInserted", "subject": "myapp/vehicles/motorcycles", "eventTime": "'`date +%Y-%m-%dT%H:%M:%S%z`'", "data":{ "make": "Ducati", "model": "Monster"},"dataVersion": "1.0"} ]'
curl -X POST -H "aeg-sas-key: $key" -d "$event" $endpoint
```

您的接聽項應用程式應該會收到事件訊息。

## <a name="clean-up-resources"></a>清除資源
如果您打算繼續使用此事件，請勿清除在本文中建立的資源。 否則，請使用下列命令來刪除您在本文建立的資源。

```azurecli-interactive
az group delete --name gridResourceGroup
```

## <a name="next-steps"></a>後續步驟

您現在知道如何建立主題和事件訂閱，深入了解 Event Grid 可協助您：

- [關於 Event Grid](overview.md)
- [將 Blob 儲存體事件路由至自訂的 Web 端點](../storage/blobs/storage-blob-event-quickstart.md?toc=%2fazure%2fevent-grid%2ftoc.json)
- [使用 Azure Event Grid 和 Logic Apps 監視虛擬機器變更](monitor-virtual-machine-changes-event-grid-logic-app.md)
- [將巨量資料串流處理至資料倉儲](event-grid-event-hubs-integration.md)
