---
title: 查詢 Azure Event Grid 訂用帳戶
description: 說明如何列出 Azure Event Grid 訂用帳戶。
services: event-grid
author: tfitzmac
manager: timlt
ms.service: event-grid
ms.topic: conceptual
ms.date: 04/04/2018
ms.author: tomfitz
ms.openlocfilehash: 2b46cde4a352e647ee97669f116a6c1926879fa0
ms.sourcegitcommit: 688a394c4901590bbcf5351f9afdf9e8f0c89505
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 05/18/2018
ms.locfileid: "34302410"
---
# <a name="query-event-grid-subscriptions"></a>查詢 Event Grid 訂用帳戶 

本文章說明如何列出您的 Azure 訂用帳戶中的 Event Grid 訂用帳戶。 查詢現有的 Event Grid 訂用帳戶時，請務必瞭解不同類型的訂用帳戶。 您可根據您想要取得的訂用帳戶類型來提供不同的參數。

## <a name="resource-groups-and-azure-subscriptions"></a>資源群組和 Azure 訂用帳戶

Azure 訂用帳戶和資源群組都不是 Azure 資源。 因此，資源群組的 Event Grid 訂用帳戶或 Azure 訂用帳戶沒有與 Azure 資源的 Event Grid 訂用帳戶相同的屬性。 資源群組的 Event Grid 訂用帳戶或 Azure 訂用帳戶會被視為全域訂用帳戶。

若要取得 Azure 訂用帳戶及其資源群組的 Event Grid 訂用帳戶，您不需要提供任何參數。 確定您已選取您想要查詢的 Azure 訂用帳戶。 下列範例不會取得自訂主題或 Azure 資源的 Event Grid 訂用帳戶。

對於 Azure CLI，請使用：

```azurecli-interactive
az account set -s "My Azure Subscription"
az eventgrid event-subscription list
```

對於 PowerShell，請使用：

```azurepowershell-interactive
Set-AzureRmContext -Subscription "My Azure Subscription"
Get-AzureRmEventGridSubscription
```

若要取得 Azure 訂用帳戶的 Event Grid 訂用帳戶，請提供 **Microsoft.Resources.Subscriptions** 主題類型。

對於 Azure CLI，請使用：

```azurecli-interactive
az eventgrid event-subscription list --topic-type-name "Microsoft.Resources.Subscriptions"
```

對於 PowerShell，請使用：

```azurepowershell-interactive
Get-AzureRmEventGridSubscription -TopicTypeName "Microsoft.Resources.Subscriptions"
```

若要取得 Azure 訂用帳戶內所有資源群組的 Event Grid 訂用帳戶，請提供 **Microsoft.Resources.ResourceGroups** 主題類型。

對於 Azure CLI，請使用：

```azurecli-interactive
az eventgrid event-subscription list --topic-type-name "Microsoft.Resources.ResourceGroups"
```

對於 PowerShell，請使用：

```azurepowershell-interactive
Get-AzureRmEventGridSubscription -TopicTypeName "Microsoft.Resources.ResourceGroups"
```

若要取得指定資源群組的 Event Grid 訂用帳戶，請提供資源群組的名稱作為參數。

對於 Azure CLI，請使用：

```azurecli-interactive
az eventgrid event-subscription list --resource-group myResourceGroup
```

對於 PowerShell，請使用：

```azurepowershell-interactive
Get-AzureRmEventGridSubscription -ResourceGroupName myResourceGroup
```

## <a name="custom-topics-and-azure-resources"></a>自訂主題和 Azure 資源

Event Grid 自訂主題是 Azure 資源。 因此，您可以使用相同的方式，查詢自訂主題和其他資源的 Event Grid 訂用帳戶，例如 Blob 儲存體帳戶。 若要取得自訂主題的 Event Grid 訂用帳戶，您必須提供可識別資源或識別資源位置的參數。 不可能廣泛查詢 Azure 訂用帳戶中所有資源的 Event Grid 訂用帳戶。

若要取得某個位置中自訂主題和其他資源的 Event Grid 訂用帳戶，請提供位置名稱。

對於 Azure CLI，請使用：

```azurecli-interactive
az eventgrid event-subscription list --location westus2
```

對於 PowerShell，請使用：

```azurepowershell-interactive
Get-AzureRmEventGridSubscription -Location westus2
```

若要取得某個位置中自訂主題的訂用帳戶，請提供位置和 **Microsoft.EventGrid.Topics** 主題類型。

對於 Azure CLI，請使用：

```azurecli-interactive
az eventgrid event-subscription list --topic-type-name "Microsoft.EventGrid.Topics" --location "westus2"
```

對於 PowerShell，請使用：

```azurepowershell-interactive
Get-AzureRmEventGridSubscription -TopicTypeName "Microsoft.EventGrid.Topics" -Location westus2
```

若要取得某個位置中儲存體帳戶的訂用帳戶，請提供位置和 **Microsoft.Storage.StorageAccounts** 主題類型。

對於 Azure CLI，請使用：

```azurecli-interactive
az eventgrid event-subscription list --topic-type "Microsoft.Storage.StorageAccounts" --location westus2
```

對於 PowerShell，請使用：

```azurepowershell-interactive
Get-AzureRmEventGridSubscription -TopicTypeName "Microsoft.Storage.StorageAccounts" -Location westus2
```

若要取得自訂主題的 Event Grid 訂用帳戶，請提供自訂主題名稱及其資源群組名稱。

對於 Azure CLI，請使用：

```azurecli-interactive
az eventgrid event-subscription list --topic-name myCustomTopic --resource-group myResourceGroup
```

對於 PowerShell，請使用：

```azurepowershell-interactive
Get-AzureRmEventGridSubscription -TopicName myCustomTopic -ResourceGroupName myResourceGroup
```

若要取得特定資源的 Event Grid 訂用帳戶，請提供資源識別碼。

對於 Azure CLI，請使用：

```azurecli-interactive
resourceid=$(az resource show -n mystorage -g myResourceGroup --resource-type "Microsoft.Storage/storageaccounts" --query id --output tsv)
az eventgrid event-subscription list --resource-id $resourceid
```

對於 PowerShell，請使用：

```azurepowershell-interactive
$resourceid = (Get-AzureRmResource -Name mystorage -ResourceGroupName myResourceGroup).ResourceId
Get-AzureRmEventGridSubscription -ResourceId $resourceid
```

## <a name="next-steps"></a>後續步驟

* 如需事件傳遞和重試的相關資訊，請參閱[事件格線訊息傳遞與重試](delivery-and-retry.md)。
* 如需 Event Grid 的簡介，請參閱[關於 Event Grid](overview.md)。
* 若要快速地開始使用 Event Grid，請參閱[使用 Azure Event Grid 建立和路由傳送自訂事件](custom-event-quickstart.md)。
