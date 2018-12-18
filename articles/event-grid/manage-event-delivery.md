---
title: 適用於 Azure 事件方格訂用帳戶的無效信件與重試原則
description: 說明如何為事件方格自訂事件傳遞選項。 設定無效信件目的地，然後指定要重試傳遞的時間長度。
services: event-grid
author: tfitzmac
manager: timlt
ms.service: event-grid
ms.topic: conceptual
ms.date: 08/03/2018
ms.author: tomfitz
ms.openlocfilehash: 5a37fadc179157ba590b31a79fcd98f223cb1869
ms.sourcegitcommit: 9222063a6a44d4414720560a1265ee935c73f49e
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/03/2018
ms.locfileid: "39501944"
---
# <a name="dead-letter-and-retry-policies"></a>無效信件與重試原則

當建立事件訂用帳戶時，您可以自訂事件傳遞的設定。 您可以設定事件方格嘗試傳遞訊息的時間長度。 您可以設定要用於儲存 (無法傳遞至端點的) 事件的儲存體帳戶。

[!INCLUDE [event-grid-preview-feature-note.md](../../includes/event-grid-preview-feature-note.md)]

## <a name="set-dead-letter-location"></a>設定無效信件位置

當事件方格無法傳遞事件時，它可以將未傳遞事件傳送到儲存體帳戶。 這個程序稱為無效信件處理。 根據預設，事件方格不會開啟無效信件處理。 若要啟用它，您必須指定儲存體帳戶在建立事件訂用帳戶時保留未傳遞事件。 您可從這個儲存體帳戶提取事件，以解析傳遞項目。

如果事件方格嘗試完所有重試嘗試，或者如果收到錯誤訊息指出傳遞永遠不會成功，則它會將事件傳送到無效信件位置。 例如，如果事件方格在傳遞事件時收到格式不正確的錯誤訊息，就會將該事件傳送到無效信件位置。 在系統最後一次嘗試傳遞事件，以及將它傳遞至無效信件位置之間，具有五分鐘的延遲。 此延遲的目的是為了減少 Blob 儲存體的作業次數。 如果無效信件位置無法使用的時間達到四個小時，系統便會捨棄該事件。

在設定無效信件位置之前，您必須先有具備容器的儲存體帳戶。 建立事件訂用帳戶時，您必須提供這個容器的端點。 端點的格式如下：`/subscriptions/<subscription-id>/resourceGroups/<resource-group-name>/providers/Microsoft.Storage/storageAccounts/<storage-name>/blobServices/default/containers/<container-name>`

下列指令碼會取得現有儲存體帳戶的資源識別碼，並且建立事件訂用帳戶；該事件訂用帳戶會使用儲存體帳戶中的容器作為無效信件端點。

```azurecli-interactive
# if you have not already installed the extension, do it now.
# This extension is required for preview features.
az extension add --name eventgrid

storagename=demostorage
containername=testcontainer

storageid=$(az storage account show --name $storagename --resource-group gridResourceGroup --query id --output tsv)

az eventgrid event-subscription create \
  -g gridResourceGroup \
  --topic-name <topic_name> \
  --name <event_subscription_name> \
  --endpoint <endpoint_URL> \
  --deadletter-endpoint $storageid/blobServices/default/containers/$containername
```

若要使用事件方格來回應未傳遞事件，請為無效信件 Blob 儲存體[建立事件訂用帳戶](../storage/blobs/storage-blob-event-quickstart.md?toc=%2fazure%2fevent-grid%2ftoc.json)。 每當您的無效信件 Blob 儲存體收到未傳遞事件時，事件方格就會通知您的處理常式。 處理常式會以您希望採取的動作來回應，以便協調未傳遞事件。 

若要關閉無效信件處理，請重新執行命令以建立事件訂閱，但是不提供 `deadletter-endpoint` 的值。 您不需要刪除事件訂閱。

## <a name="set-retry-policy"></a>設定重試原則

當建立事件方格訂用帳戶時，您可以設定事件方格應該嘗試傳遞事件的時間長度。 根據預設，事件方格會嘗試 24 小時 (1440 分鐘)，以及嘗試最多 30 次。 您可以為事件方格訂用帳戶設定上述任一值。 事件存留時間的值必須是介於 1 和 1440 的整數。 傳遞嘗試上限的值必須是介於 1 和 30 的整數。

您無法設定[重試間隔](delivery-and-retry.md#retry-intervals-and-duration)。

若要將事件存留時間設定為 1440 分鐘以外的值，請使用：

```azurecli-interactive
# if you have not already installed the extension, do it now.
# This extension is required for preview features.
az extension add --name eventgrid

az eventgrid event-subscription create \
  -g gridResourceGroup \
  --topic-name <topic_name> \
  --name <event_subscription_name> \
  --endpoint <endpoint_URL> \
  --event-ttl 720
```

若要將重試嘗試上限設定為 30 以外的值，請使用：

```azurecli-interactive
az eventgrid event-subscription create \
  -g gridResourceGroup \
  --topic-name <topic_name> \
  --name <event_subscription_name> \
  --endpoint <endpoint_URL> \
  --max-delivery-attempts 18
```

如果您同時設定 `event-ttl` 和 `max-deliver-attempts`，事件方格會針對重試嘗試使用第一個到期的項目。

## <a name="next-steps"></a>後續步驟

* 如需使用 Azure 函式應用程式來處理無效信件事件的範例應用程式，請參閱[適用於 .NET 的 Azure 事件方格無效信件範例](https://azure.microsoft.com/resources/samples/event-grid-dotnet-handle-deadlettered-events/) \(英文\)。
* 如需事件傳遞和重試的相關資訊，請參閱[事件格線訊息傳遞與重試](delivery-and-retry.md)。
* 如需 Event Grid 的簡介，請參閱[關於 Event Grid](overview.md)。
* 若要快速地開始使用 Event Grid，請參閱[使用 Azure Event Grid 建立和路由傳送自訂事件](custom-event-quickstart.md)。
