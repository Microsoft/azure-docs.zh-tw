---
title: 將 Azure 事件方格的自訂事件傳送至儲存體佇列 | Microsoft Docs
description: 使用 Azure 事件格線和 Azure CLI 來發佈主題，以及訂閱該事件。 儲存體佇列會用於端點。
services: event-grid
keywords: ''
author: tfitzmac
ms.author: tomfitz
ms.date: 10/09/2018
ms.topic: quickstart
ms.service: event-grid
ms.openlocfilehash: 7ca8311c97faed980555c46d977a5df85c20353d
ms.sourcegitcommit: 7b0778a1488e8fd70ee57e55bde783a69521c912
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/10/2018
ms.locfileid: "49067446"
---
# <a name="route-custom-events-to-azure-queue-storage-with-azure-cli-and-event-grid"></a>使用 Azure CLI 和事件方格將自訂事件路由至 Azure 佇列

Azure Event Grid 是一項雲端事件服務。 Azure 佇列儲存體是其中一個支援的事件處理常式。 在本文中，您可使用 Azure CLI 建立自訂主題、訂閱自訂主題，以及觸發事件來檢視結果。 您會將事件傳送至佇列儲存體。

[!INCLUDE [quickstarts-free-trial-note.md](../../includes/quickstarts-free-trial-note.md)]

[!INCLUDE [event-grid-preview-feature-note.md](../../includes/event-grid-preview-feature-note.md)]

## <a name="create-a-resource-group"></a>建立資源群組

Event Grid 為 Azure 資源，必須放入 Azure 資源群組中。 資源群組是在其中部署與管理 Azure 資源的邏輯集合。

使用 [az group create](/cli/azure/group#az-group-create) 命令來建立資源群組。 

下列範例會在 westus2 位置建立名為 gridResourceGroup 的資源群組。

```azurecli-interactive
az group create --name gridResourceGroup --location westus2
```

[!INCLUDE [event-grid-register-provider-cli.md](../../includes/event-grid-register-provider-cli.md)]

## <a name="create-a-custom-topic"></a>建立自訂主題

Event Grid 主題會提供使用者定義的端點，作為您發佈事件的目的地。 下列範例可在您的資源群組中建立自訂主題。 以自訂主題的唯一名稱取代 `<topic_name>`。 事件方格的主題名稱必須是唯一的，因為它由 DNS 項目表示。

```azurecli-interactive
# if you have not already installed the extension, do it now.
# This extension is required for preview features.
az extension add --name eventgrid

az eventgrid topic create --name <topic_name> -l westus2 -g gridResourceGroup
```

## <a name="create-queue-storage"></a>建立佇列儲存體

訂閱自訂主題之前，讓我們建立事件訊息的端點。 您將建立佇列儲存體來收集事件。

```azurecli-interactive
storagename="<unique-storage-name>"
queuename="eventqueue"

az storage account create -n $storagename -g gridResourceGroup -l westus2 --sku Standard_LRS
az storage queue create --name $queuename --account-name $storagename
```

## <a name="subscribe-to-a-custom-topic"></a>訂閱自訂主題

您可訂閱自訂主題，告知事件方格您想要追蹤的事件。下列範例可訂閱您所建立的自訂主題，以及傳遞端點的佇列儲存體資源識別碼。 利用 Azure CLI，您可以傳遞佇列儲存體識別碼作為端點。 端點的格式如下：

`/subscriptions/<subscription-id>/resourcegroups/<resource-group-name>/providers/Microsoft.Storage/storageAccounts/<storage-name>/queueservices/default/queues/<queue-name>`

下列指令碼會取得佇列的儲存體帳戶資源識別碼。 它會建構佇列儲存體的識別碼，並訂閱事件方格主題。 並將端點類型設為 `storagequeue`，然後使用端點的佇列識別碼。

```azurecli-interactive
storageid=$(az storage account show --name $storagename --resource-group gridResourceGroup --query id --output tsv)
queueid="$storageid/queueservices/default/queues/$queuename"

az eventgrid event-subscription create \
  --topic-name <topic_name> \
  -g gridResourceGroup \
  --name <event_subscription_name> \
  --endpoint-type storagequeue \
  --endpoint $queueid
```

建立事件訂用帳戶的帳戶必須具有佇列儲存體的寫入權限。

如果您使用 REST API 來建立訂用帳戶時，可以傳遞儲存體帳戶的識別碼和佇列的名稱作為個別參數。

```json
"destination": {
  "endpointType": "storagequeue",
  "properties": {
    "queueName":"eventqueue",
    "resourceId": "/subscriptions/<subscription-id>/resourcegroups/<resource-group-name>/providers/Microsoft.Storage/storageAccounts/<storage-name>"
  }
  ...
```

## <a name="send-an-event-to-your-custom-topic"></a>將事件傳送至自訂主題

讓我們觸發事件以了解 Event Grid 如何將訊息散發至您的端點。 首先，讓我們取得自訂主題的 URL 和金鑰。 再次，將您的自訂主題名稱用於 `<topic_name>`。

```azurecli-interactive
endpoint=$(az eventgrid topic show --name <topic_name> -g gridResourceGroup --query "endpoint" --output tsv)
key=$(az eventgrid topic key list --name <topic_name> -g gridResourceGroup --query "key1" --output tsv)
```

若要簡化這篇文章，您可使用要傳送至自訂主題的範例事件資料。 一般而言，應用程式或 Azure 服務就會傳送事件資料。 CURL 是可傳送 HTTP 要求的公用程式。 本文使用 CURL 將事件傳送到自訂主題。  下列範例會將三個事件傳送至事件方格主題：

```azurecli-interactive
for i in 1 2 3
do
   event='[ {"id": "'"$RANDOM"'", "eventType": "recordInserted", "subject": "myapp/vehicles/motorcycles", "eventTime": "'`date +%Y-%m-%dT%H:%M:%S%z`'", "data":{ "make": "Ducati", "model": "Monster"},"dataVersion": "1.0"} ]'
   curl -X POST -H "aeg-sas-key: $key" -d "$event" $endpoint
done
```

請在入口網站中瀏覽至佇列，並注意事件方格會傳送這三個事件至佇列。

![顯示訊息](./media/custom-event-to-queue-storage/messages.png)


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
