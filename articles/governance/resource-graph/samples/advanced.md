---
title: 進階查詢範例
description: 使用 Azure Resource Graph 執行某些進階查詢，包括使用資料行、列出使用的標籤，以及使用規則運算式比對資源。
ms.date: 01/27/2021
ms.topic: sample
ms.openlocfilehash: b0a13fb798a9403b240c97fb207bf9b76e01028f
ms.sourcegitcommit: 436518116963bd7e81e0217e246c80a9808dc88c
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 01/27/2021
ms.locfileid: "98918863"
---
# <a name="advanced-resource-graph-query-samples"></a>進階 Resource Graph 查詢範例

透過 Azure Resource Graph 了解查詢的第一個步驟是對[查詢語言](../concepts/query-language.md)進行基本的認識。 如果您還不熟悉 [Azure 資料總管](/azure/data-explorer/data-explorer-overview)，建議檢閱基本概念，以了解如何撰寫所需資源的要求。

我們將逐步解說下列進階查詢︰

- [顯示每個資源類型的 API 版本](#apiversion)
- [取得虛擬機器擴展集的容量和大小](#vmss-capacity)
- [移除結果中的資料行](#remove-column)
- [列出所有標記名稱](#list-all-tags)
- [依 RegEx 比對虛擬機器](#vm-regex)
- [列出具有特定寫入位置的 Cosmos DB](#mvexpand-cosmosdb)
- [金鑰保存庫與訂用帳戶名稱](#join)
- [列出 SQL 資料庫和其彈性集區](#join-sql)
- [列出虛擬機器及其網路介面和公用 IP](#join-vmpip)
- [列出虛擬機器主機上安裝的所有擴充功能](#join-vmextension)
- [在資源群組上尋找具有特定標籤的儲存體帳戶](#join-findstoragetag)
- [將兩個查詢的結果合併成單一結果](#unionresults)
- [包含租用戶和訂用帳戶名稱與 DisplayNames](#displaynames)
- [依電源狀態擴充屬性來摘要說明虛擬機器](#vm-powerstate)
- [不符合規範的來賓設定指派計數](#count-gcnoncompliant)
- [查詢來賓設定指派報告的詳細資料](#query-gcreports)
- [尋找電腦不符合來賓設定指派規範的所有原因](#query-gcmachinedetails)

如果您沒有 Azure 訂用帳戶，請在開始前建立[免費帳戶](https://azure.microsoft.com/free)。

## <a name="language-support"></a>語言支援

Azure CLI (透過擴充功能) 與 Azure PowerShell (透過模組) 支援 Azure Resource Graph。 在執行下列任何查詢之前，請檢查您的環境已準備就緒。 請參閱 [Azure CLI](../first-query-azurecli.md#add-the-resource-graph-extension) 與 [Azure PowerShell](../first-query-powershell.md#add-the-resource-graph-module)，以了解安裝及驗證所選殼層環境的步驟。

## <a name="show-resource-types-and-api-versions"></a><a name="apiversion"></a>顯示資源類型與 API 版本

在更新期間，Resource Graph 主要使用資源提供者 API 的最新非預覽版本來 `GET` 資源屬性。 在某些情況下，所使用的 API 版本會遭到覆寫，以在結果中提供更新或更廣泛使用的屬性。 下列查詢會詳細說明用來在每個資源類型上收集屬性的 API 版本：

```kusto
Resources
| distinct type, apiVersion
| where isnotnull(apiVersion)
| order by type asc
```

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

```azurecli-interactive
az graph query -q "Resources | distinct type, apiVersion | where isnotnull(apiVersion) | order by type asc"
```

# <a name="azure-powershell"></a>[Azure PowerShell](#tab/azure-powershell)

```azurepowershell-interactive
Search-AzGraph -Query "Resources | distinct type, apiVersion | where isnotnull(apiVersion) | order by type asc"
```

# <a name="portal"></a>[入口網站](#tab/azure-portal)

:::image type="icon" source="../media/resource-graph-small.png":::在 Azure Resource Graph 總管中試用此查詢：

- Azure 入口網站：<a href="https://portal.azure.com/?feature.customportal=false#blade/HubsExtension/ArgQueryBlade/query/Resources%0D%0A%7C%20distinct%20type%2C%20apiVersion%0D%0A%7C%20where%20isnotnull%28apiVersion%29%0D%0A%7C%20order%20by%20type%20asc" target="_blank">portal.azure.com <span class="docon docon-navigate-external x-hidden-focus"></span></a>
- Azure Government 入口網站：<a href="https://portal.azure.us/?feature.customportal=false#blade/HubsExtension/ArgQueryBlade/query/Resources%0D%0A%7C%20distinct%20type%2C%20apiVersion%0D%0A%7C%20where%20isnotnull%28apiVersion%29%0D%0A%7C%20order%20by%20type%20asc" target="_blank">portal.azure.us <span class="docon docon-navigate-external x-hidden-focus"></span></a>
- Azure China 21Vianet 入口網站：<a href="https://portal.azure.cn/?feature.customportal=false#blade/HubsExtension/ArgQueryBlade/query/Resources%0D%0A%7C%20distinct%20type%2C%20apiVersion%0D%0A%7C%20where%20isnotnull%28apiVersion%29%0D%0A%7C%20order%20by%20type%20asc" target="_blank">portal.azure.cn <span class="docon docon-navigate-external x-hidden-focus"></span></a>

---

## <a name="get-virtual-machine-scale-set-capacity-and-size"></a><a name="vmss-capacity"></a>取得虛擬機器擴展集的容量和大小

此查詢會尋找虛擬機器擴展集資源，並取得各種詳細資料，包括虛擬機器大小與擴展集的容量。 此查詢會使用 `toint()` 函式，將容量轉換為數字以便進行排序。 最後，資料行會重新命名為自訂的具名屬性。

```kusto
Resources
| where type=~ 'microsoft.compute/virtualmachinescalesets'
| where name contains 'contoso'
| project subscriptionId, name, location, resourceGroup, Capacity = toint(sku.capacity), Tier = sku.name
| order by Capacity desc
```

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

```azurecli-interactive
az graph query -q "Resources | where type=~ 'microsoft.compute/virtualmachinescalesets' | where name contains 'contoso' | project subscriptionId, name, location, resourceGroup, Capacity = toint(sku.capacity), Tier = sku.name | order by Capacity desc"
```

# <a name="azure-powershell"></a>[Azure PowerShell](#tab/azure-powershell)

```azurepowershell-interactive
Search-AzGraph -Query "Resources | where type=~ 'microsoft.compute/virtualmachinescalesets' | where name contains 'contoso' | project subscriptionId, name, location, resourceGroup, Capacity = toint(sku.capacity), Tier = sku.name | order by Capacity desc"
```

# <a name="portal"></a>[入口網站](#tab/azure-portal)

:::image type="icon" source="../media/resource-graph-small.png":::在 Azure Resource Graph 總管中試用此查詢：

- Azure 入口網站：<a href="https://portal.azure.com/?feature.customportal=false#blade/HubsExtension/ArgQueryBlade/query/Resources%0D%0A%7C%20where%20type%3D~%20%27microsoft.compute%2Fvirtualmachinescalesets%27%0D%0A%7C%20where%20name%20contains%20%27contoso%27%0D%0A%7C%20project%20subscriptionId%2C%20name%2C%20location%2C%20resourceGroup%2C%20Capacity%20%3D%20toint%28sku.capacity%29%2C%20Tier%20%3D%20sku.name%0D%0A%7C%20order%20by%20Capacity%20desc" target="_blank">portal.azure.com <span class="docon docon-navigate-external x-hidden-focus"></span></a>
- Azure Government 入口網站：<a href="https://portal.azure.us/?feature.customportal=false#blade/HubsExtension/ArgQueryBlade/query/Resources%0D%0A%7C%20where%20type%3D~%20%27microsoft.compute%2Fvirtualmachinescalesets%27%0D%0A%7C%20where%20name%20contains%20%27contoso%27%0D%0A%7C%20project%20subscriptionId%2C%20name%2C%20location%2C%20resourceGroup%2C%20Capacity%20%3D%20toint%28sku.capacity%29%2C%20Tier%20%3D%20sku.name%0D%0A%7C%20order%20by%20Capacity%20desc" target="_blank">portal.azure.us <span class="docon docon-navigate-external x-hidden-focus"></span></a>
- Azure China 21Vianet 入口網站：<a href="https://portal.azure.cn/?feature.customportal=false#blade/HubsExtension/ArgQueryBlade/query/Resources%0D%0A%7C%20where%20type%3D~%20%27microsoft.compute%2Fvirtualmachinescalesets%27%0D%0A%7C%20where%20name%20contains%20%27contoso%27%0D%0A%7C%20project%20subscriptionId%2C%20name%2C%20location%2C%20resourceGroup%2C%20Capacity%20%3D%20toint%28sku.capacity%29%2C%20Tier%20%3D%20sku.name%0D%0A%7C%20order%20by%20Capacity%20desc" target="_blank">portal.azure.cn <span class="docon docon-navigate-external x-hidden-focus"></span></a>

---

## <a name="remove-columns-from-results"></a><a name="remove-column"></a>移除結果中的資料行

下列查詢會使用 `summarize` 來依訂用帳戶計算資源，使用 `join` 來與 _ResourceContainers_  資料表中的訂用帳戶詳細資料結合，然後使用 `project-away` 來移除部分資料行。

```kusto
Resources
| summarize resourceCount=count() by subscriptionId
| join (ResourceContainers | where type=='microsoft.resources/subscriptions' | project SubName=name, subscriptionId) on subscriptionId
| project-away subscriptionId, subscriptionId1
```

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

```azurecli-interactive
az graph query -q "Resources | summarize resourceCount=count() by subscriptionId | join (ResourceContainers | where type=='microsoft.resources/subscriptions' | project SubName=name, subscriptionId) on subscriptionId| project-away subscriptionId, subscriptionId1"
```

# <a name="azure-powershell"></a>[Azure PowerShell](#tab/azure-powershell)

```azurepowershell-interactive
Search-AzGraph -Query "Resources | summarize resourceCount=count() by subscriptionId | join (ResourceContainers | where type=='microsoft.resources/subscriptions' | project SubName=name, subscriptionId) on subscriptionId| project-away subscriptionId, subscriptionId1"
```

# <a name="portal"></a>[入口網站](#tab/azure-portal)

:::image type="icon" source="../media/resource-graph-small.png":::在 Azure Resource Graph 總管中試用此查詢：

- Azure 入口網站：<a href="https://portal.azure.com/?feature.customportal=false#blade/HubsExtension/ArgQueryBlade/query/Resources%0D%0A%7C%20summarize%20resourceCount%3Dcount%28%29%20by%20subscriptionId%0D%0A%7C%20join%20%28ResourceContainers%20%7C%20where%20type%3D%3D%27microsoft.resources%2Fsubscriptions%27%20%7C%20project%20SubName%3Dname%2C%20subscriptionId%29%20on%20subscriptionId%0D%0A%7C%20project-away%20subscriptionId%2C%20subscriptionId1" target="_blank">portal.azure.com <span class="docon docon-navigate-external x-hidden-focus"></span></a>
- Azure Government 入口網站：<a href="https://portal.azure.us/?feature.customportal=false#blade/HubsExtension/ArgQueryBlade/query/Resources%0D%0A%7C%20summarize%20resourceCount%3Dcount%28%29%20by%20subscriptionId%0D%0A%7C%20join%20%28ResourceContainers%20%7C%20where%20type%3D%3D%27microsoft.resources%2Fsubscriptions%27%20%7C%20project%20SubName%3Dname%2C%20subscriptionId%29%20on%20subscriptionId%0D%0A%7C%20project-away%20subscriptionId%2C%20subscriptionId1" target="_blank">portal.azure.us <span class="docon docon-navigate-external x-hidden-focus"></span></a>
- Azure China 21Vianet 入口網站：<a href="https://portal.azure.cn/?feature.customportal=false#blade/HubsExtension/ArgQueryBlade/query/Resources%0D%0A%7C%20summarize%20resourceCount%3Dcount%28%29%20by%20subscriptionId%0D%0A%7C%20join%20%28ResourceContainers%20%7C%20where%20type%3D%3D%27microsoft.resources%2Fsubscriptions%27%20%7C%20project%20SubName%3Dname%2C%20subscriptionId%29%20on%20subscriptionId%0D%0A%7C%20project-away%20subscriptionId%2C%20subscriptionId1" target="_blank">portal.azure.cn <span class="docon docon-navigate-external x-hidden-focus"></span></a>

---

## <a name="list-all-tag-names"></a><a name="list-all-tags"></a>列出所有標記名稱

此查詢的開頭為標記，而且會建置 JSON 物件，列出所有的唯一標記名稱及其對應的類型。

```kusto
Resources
| project tags
| summarize buildschema(tags)
```

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

```azurecli-interactive
az graph query -q "Resources | project tags | summarize buildschema(tags)"
```

# <a name="azure-powershell"></a>[Azure PowerShell](#tab/azure-powershell)

```azurepowershell-interactive
Search-AzGraph -Query "Resources | project tags | summarize buildschema(tags)"
```

# <a name="portal"></a>[入口網站](#tab/azure-portal)

:::image type="icon" source="../media/resource-graph-small.png":::在 Azure Resource Graph 總管中試用此查詢：

- Azure 入口網站：<a href="https://portal.azure.com/?feature.customportal=false#blade/HubsExtension/ArgQueryBlade/query/Resources%0D%0A%7C%20project%20tags%0D%0A%7C%20summarize%20buildschema%28tags%29" target="_blank">portal.azure.com <span class="docon docon-navigate-external x-hidden-focus"></span></a>
- Azure Government 入口網站：<a href="https://portal.azure.us/?feature.customportal=false#blade/HubsExtension/ArgQueryBlade/query/Resources%0D%0A%7C%20project%20tags%0D%0A%7C%20summarize%20buildschema%28tags%29" target="_blank">portal.azure.us <span class="docon docon-navigate-external x-hidden-focus"></span></a>
- Azure China 21Vianet 入口網站：<a href="https://portal.azure.cn/?feature.customportal=false#blade/HubsExtension/ArgQueryBlade/query/Resources%0D%0A%7C%20project%20tags%0D%0A%7C%20summarize%20buildschema%28tags%29" target="_blank">portal.azure.cn <span class="docon docon-navigate-external x-hidden-focus"></span></a>

---

## <a name="virtual-machines-matched-by-regex"></a><a name="vm-regex"></a>依 RegEx 比對虛擬機器

此查詢會尋找符合 [規則運算式](/dotnet/standard/base-types/regular-expression-language-quick-reference) (亦稱為 _RegEx_) 的虛擬機器。 **matches regex \@** 可讓我們定義要比對的 RegEx，也就是 `^Contoso(.*)[0-9]+$`。
該 RegEx 定義說明如下：

- `^`：比對必須從字串的開頭開始。
- `Contoso`：區分大小寫的字串。
- `(.*)`：子運算式比對：
  - `.`：比對任何單一字元 (除了新行符號)。
  - `*`：比對前一個元素零或多次。
- `[0-9]`：比對從 0 到 9 的字元群組。
- `+`：比對前一個元素一或多次。
- `$`：前一個元素的比對必須發生在字串結尾。

在依名稱比對之後，查詢會投射名稱，並依名稱遞增排序。

```kusto
Resources
| where type =~ 'microsoft.compute/virtualmachines' and name matches regex @'^Contoso(.*)[0-9]+$'
| project name
| order by name asc
```

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

```azurecli-interactive
az graph query -q "Resources | where type =~ 'microsoft.compute/virtualmachines' and name matches regex @'^Contoso(.*)[0-9]+$' | project name | order by name asc"
```

# <a name="azure-powershell"></a>[Azure PowerShell](#tab/azure-powershell)

```azurepowershell-interactive
Search-AzGraph -Query "Resources | where type =~ 'microsoft.compute/virtualmachines' and name matches regex @'^Contoso(.*)[0-9]+$' | project name | order by name asc"
```

# <a name="portal"></a>[入口網站](#tab/azure-portal)

:::image type="icon" source="../media/resource-graph-small.png":::在 Azure Resource Graph 總管中試用此查詢：

- Azure 入口網站：<a href="https://portal.azure.com/?feature.customportal=false#blade/HubsExtension/ArgQueryBlade/query/Resources%0D%0A%7C%20where%20type%20%3D~%20%27microsoft.compute%2Fvirtualmachines%27%20and%20name%20matches%20regex%20%40%27%5EContoso%28.%2A%29%5B0-9%5D%2B%24%27%0D%0A%7C%20project%20name%0D%0A%7C%20order%20by%20name%20asc" target="_blank">portal.azure.com <span class="docon docon-navigate-external x-hidden-focus"></span></a>
- Azure Government 入口網站：<a href="https://portal.azure.us/?feature.customportal=false#blade/HubsExtension/ArgQueryBlade/query/Resources%0D%0A%7C%20where%20type%20%3D~%20%27microsoft.compute%2Fvirtualmachines%27%20and%20name%20matches%20regex%20%40%27%5EContoso%28.%2A%29%5B0-9%5D%2B%24%27%0D%0A%7C%20project%20name%0D%0A%7C%20order%20by%20name%20asc" target="_blank">portal.azure.us <span class="docon docon-navigate-external x-hidden-focus"></span></a>
- Azure China 21Vianet 入口網站：<a href="https://portal.azure.cn/?feature.customportal=false#blade/HubsExtension/ArgQueryBlade/query/Resources%0D%0A%7C%20where%20type%20%3D~%20%27microsoft.compute%2Fvirtualmachines%27%20and%20name%20matches%20regex%20%40%27%5EContoso%28.%2A%29%5B0-9%5D%2B%24%27%0D%0A%7C%20project%20name%0D%0A%7C%20order%20by%20name%20asc" target="_blank">portal.azure.cn <span class="docon docon-navigate-external x-hidden-focus"></span></a>

---

## <a name="list-cosmos-db-with-specific-write-locations"></a><a name="mvexpand-cosmosdb"></a>列出具有特定寫入位置的 Cosmos DB

下列查詢受限於 Cosmos DB 資源，使用 `mv-expand` 來展開 **properties.writeLocations** 的屬性包，然後預測特定欄位並將結果進一步限制為符合「美國東部」或「美國西部」的 **properties.writeLocations.locationName** 值。

```kusto
Resources
| where type =~ 'microsoft.documentdb/databaseaccounts'
| project id, name, writeLocations = (properties.writeLocations)
| mv-expand writeLocations
| project id, name, writeLocation = tostring(writeLocations.locationName)
| where writeLocation in ('East US', 'West US')
| summarize by id, name
```

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

```azurecli-interactive
az graph query -q "Resources | where type =~ 'microsoft.documentdb/databaseaccounts' | project id, name, writeLocations = (properties.writeLocations) | mv-expand writeLocations | project id, name, writeLocation = tostring(writeLocations.locationName) | where writeLocation in ('East US', 'West US') | summarize by id, name"
```

# <a name="azure-powershell"></a>[Azure PowerShell](#tab/azure-powershell)

```azurepowershell-interactive
Search-AzGraph -Query "Resources | where type =~ 'microsoft.documentdb/databaseaccounts' | project id, name, writeLocations = (properties.writeLocations) | mv-expand writeLocations | project id, name, writeLocation = tostring(writeLocations.locationName) | where writeLocation in ('East US', 'West US') | summarize by id, name"
```

# <a name="portal"></a>[入口網站](#tab/azure-portal)

:::image type="icon" source="../media/resource-graph-small.png":::在 Azure Resource Graph 總管中試用此查詢：

- Azure 入口網站：<a href="https://portal.azure.com/?feature.customportal=false#blade/HubsExtension/ArgQueryBlade/query/Resources%0D%0A%7C%20where%20type%20%3D~%20%27microsoft.documentdb%2Fdatabaseaccounts%27%0D%0A%7C%20project%20id%2C%20name%2C%20writeLocations%20%3D%20%28properties.writeLocations%29%0D%0A%7C%20mv-expand%20writeLocations%0D%0A%7C%20project%20id%2C%20name%2C%20writeLocation%20%3D%20tostring%28writeLocations.locationName%29%0D%0A%7C%20where%20writeLocation%20in%20%28%27East%20US%27%2C%20%27West%20US%27%29%0D%0A%7C%20summarize%20by%20id%2C%20name" target="_blank">portal.azure.com <span class="docon docon-navigate-external x-hidden-focus"></span></a>
- Azure Government 入口網站：<a href="https://portal.azure.us/?feature.customportal=false#blade/HubsExtension/ArgQueryBlade/query/Resources%0D%0A%7C%20where%20type%20%3D~%20%27microsoft.documentdb%2Fdatabaseaccounts%27%0D%0A%7C%20project%20id%2C%20name%2C%20writeLocations%20%3D%20%28properties.writeLocations%29%0D%0A%7C%20mv-expand%20writeLocations%0D%0A%7C%20project%20id%2C%20name%2C%20writeLocation%20%3D%20tostring%28writeLocations.locationName%29%0D%0A%7C%20where%20writeLocation%20in%20%28%27East%20US%27%2C%20%27West%20US%27%29%0D%0A%7C%20summarize%20by%20id%2C%20name" target="_blank">portal.azure.us <span class="docon docon-navigate-external x-hidden-focus"></span></a>
- Azure China 21Vianet 入口網站：<a href="https://portal.azure.cn/?feature.customportal=false#blade/HubsExtension/ArgQueryBlade/query/Resources%0D%0A%7C%20where%20type%20%3D~%20%27microsoft.documentdb%2Fdatabaseaccounts%27%0D%0A%7C%20project%20id%2C%20name%2C%20writeLocations%20%3D%20%28properties.writeLocations%29%0D%0A%7C%20mv-expand%20writeLocations%0D%0A%7C%20project%20id%2C%20name%2C%20writeLocation%20%3D%20tostring%28writeLocations.locationName%29%0D%0A%7C%20where%20writeLocation%20in%20%28%27East%20US%27%2C%20%27West%20US%27%29%0D%0A%7C%20summarize%20by%20id%2C%20name" target="_blank">portal.azure.cn <span class="docon docon-navigate-external x-hidden-focus"></span></a>

---

## <a name="key-vaults-with-subscription-name"></a><a name="join"></a>金鑰保存庫與訂用帳戶名稱

下列查詢會顯示使用 **種類** 作為 _leftouter_ 的 `join` 複雜用法。 此查詢會將聯結的資料表限制為訂用帳戶資源，並利用 `project` 使其僅包含原始欄位 _subscriptionId_ 和已重新命名為 _SubName_ 的 _name_ 欄位。 欄位重新命名可避免 `join` 將它新增為 _name1_，因為此欄位已經存在於 _resources_ 中。 原始資料表會使用 `where` 進行篩選，而下列 `project` 包含兩個資料表的資料行。 查詢結果是所有金鑰保存庫顯示類型、金鑰保存庫名稱，及其所在訂用帳戶的名稱。

```kusto
Resources
| join kind=leftouter (ResourceContainers | where type=='microsoft.resources/subscriptions' | project SubName=name, subscriptionId) on subscriptionId
| where type == 'microsoft.keyvault/vaults'
| project type, name, SubName
```

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

```azurecli-interactive
az graph query -q "Resources | join kind=leftouter (ResourceContainers | where type=='microsoft.resources/subscriptions' | project SubName=name, subscriptionId) on subscriptionId | where type == 'microsoft.keyvault/vaults' | project type, name, SubName"
```

# <a name="azure-powershell"></a>[Azure PowerShell](#tab/azure-powershell)

```azurepowershell-interactive
Search-AzGraph -Query "Resources | join kind=leftouter (ResourceContainers | where type=='microsoft.resources/subscriptions' | project SubName=name, subscriptionId) on subscriptionId | where type == 'microsoft.keyvault/vaults' | project type, name, SubName"
```

# <a name="portal"></a>[入口網站](#tab/azure-portal)

:::image type="icon" source="../media/resource-graph-small.png":::在 Azure Resource Graph 總管中試用此查詢：

- Azure 入口網站：<a href="https://portal.azure.com/?feature.customportal=false#blade/HubsExtension/ArgQueryBlade/query/Resources%0D%0A%7C%20join%20kind%3Dleftouter%20%28ResourceContainers%20%7C%20where%20type%3D%3D%27microsoft.resources%2Fsubscriptions%27%20%7C%20project%20SubName%3Dname%2C%20subscriptionId%29%20on%20subscriptionId%0D%0A%7C%20where%20type%20%3D%3D%20%27microsoft.keyvault%2Fvaults%27%0D%0A%7C%20project%20type%2C%20name%2C%20SubName" target="_blank">portal.azure.com <span class="docon docon-navigate-external x-hidden-focus"></span></a>
- Azure Government 入口網站：<a href="https://portal.azure.us/?feature.customportal=false#blade/HubsExtension/ArgQueryBlade/query/Resources%0D%0A%7C%20join%20kind%3Dleftouter%20%28ResourceContainers%20%7C%20where%20type%3D%3D%27microsoft.resources%2Fsubscriptions%27%20%7C%20project%20SubName%3Dname%2C%20subscriptionId%29%20on%20subscriptionId%0D%0A%7C%20where%20type%20%3D%3D%20%27microsoft.keyvault%2Fvaults%27%0D%0A%7C%20project%20type%2C%20name%2C%20SubName" target="_blank">portal.azure.us <span class="docon docon-navigate-external x-hidden-focus"></span></a>
- Azure China 21Vianet 入口網站：<a href="https://portal.azure.cn/?feature.customportal=false#blade/HubsExtension/ArgQueryBlade/query/Resources%0D%0A%7C%20join%20kind%3Dleftouter%20%28ResourceContainers%20%7C%20where%20type%3D%3D%27microsoft.resources%2Fsubscriptions%27%20%7C%20project%20SubName%3Dname%2C%20subscriptionId%29%20on%20subscriptionId%0D%0A%7C%20where%20type%20%3D%3D%20%27microsoft.keyvault%2Fvaults%27%0D%0A%7C%20project%20type%2C%20name%2C%20SubName" target="_blank">portal.azure.cn <span class="docon docon-navigate-external x-hidden-focus"></span></a>

---

## <a name="list-sql-databases-and-their-elastic-pools"></a><a name="join-sql"></a>列出 SQL 資料庫和其彈性集區

下列查詢使用 **leftouter** `join` 將 SQL Database 資源及其相關的彈性集區 (如果有的話) 結合在一起。

```kusto
Resources
| where type =~ 'microsoft.sql/servers/databases'
| project databaseId = id, databaseName = name, elasticPoolId = tolower(tostring(properties.elasticPoolId))
| join kind=leftouter (
    Resources
    | where type =~ 'microsoft.sql/servers/elasticpools'
    | project elasticPoolId = tolower(id), elasticPoolName = name, elasticPoolState = properties.state)
on elasticPoolId
| project-away elasticPoolId1
```

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

```azurecli-interactive
az graph query -q "Resources | where type =~ 'microsoft.sql/servers/databases' | project databaseId = id, databaseName = name, elasticPoolId = tolower(tostring(properties.elasticPoolId)) | join kind=leftouter ( Resources | where type =~ 'microsoft.sql/servers/elasticpools' | project elasticPoolId = tolower(id), elasticPoolName = name, elasticPoolState = properties.state) on elasticPoolId | project-away elasticPoolId1"
```

# <a name="azure-powershell"></a>[Azure PowerShell](#tab/azure-powershell)

```azurepowershell-interactive
Search-AzGraph -Query "Resources | where type =~ 'microsoft.sql/servers/databases' | project databaseId = id, databaseName = name, elasticPoolId = tolower(tostring(properties.elasticPoolId)) | join kind=leftouter ( Resources | where type =~ 'microsoft.sql/servers/elasticpools' | project elasticPoolId = tolower(id), elasticPoolName = name, elasticPoolState = properties.state) on elasticPoolId | project-away elasticPoolId1"
```

# <a name="portal"></a>[入口網站](#tab/azure-portal)

:::image type="icon" source="../media/resource-graph-small.png":::在 Azure Resource Graph 總管中試用此查詢：

- Azure 入口網站：<a href="https://portal.azure.com/?feature.customportal=false#blade/HubsExtension/ArgQueryBlade/query/Resources%0D%0A%7C%20where%20type%20%3D~%20%27microsoft.sql%2Fservers%2Fdatabases%27%0D%0A%7C%20project%20databaseId%20%3D%20id%2C%20databaseName%20%3D%20name%2C%20elasticPoolId%20%3D%20tolower%28tostring%28properties.elasticPoolId%29%29%0D%0A%7C%20join%20kind%3Dleftouter%20%28%0D%0A%20%20%20%20Resources%0D%0A%20%20%20%20%7C%20where%20type%20%3D~%20%27microsoft.sql%2Fservers%2Felasticpools%27%0D%0A%20%20%20%20%7C%20project%20elasticPoolId%20%3D%20tolower%28id%29%2C%20elasticPoolName%20%3D%20name%2C%20elasticPoolState%20%3D%20properties.state%29%0D%0Aon%20elasticPoolId%0D%0A%7C%20project-away%20elasticPoolId1" target="_blank">portal.azure.com <span class="docon docon-navigate-external x-hidden-focus"></span></a>
- Azure Government 入口網站：<a href="https://portal.azure.us/?feature.customportal=false#blade/HubsExtension/ArgQueryBlade/query/Resources%0D%0A%7C%20where%20type%20%3D~%20%27microsoft.sql%2Fservers%2Fdatabases%27%0D%0A%7C%20project%20databaseId%20%3D%20id%2C%20databaseName%20%3D%20name%2C%20elasticPoolId%20%3D%20tolower%28tostring%28properties.elasticPoolId%29%29%0D%0A%7C%20join%20kind%3Dleftouter%20%28%0D%0A%20%20%20%20Resources%0D%0A%20%20%20%20%7C%20where%20type%20%3D~%20%27microsoft.sql%2Fservers%2Felasticpools%27%0D%0A%20%20%20%20%7C%20project%20elasticPoolId%20%3D%20tolower%28id%29%2C%20elasticPoolName%20%3D%20name%2C%20elasticPoolState%20%3D%20properties.state%29%0D%0Aon%20elasticPoolId%0D%0A%7C%20project-away%20elasticPoolId1" target="_blank">portal.azure.us <span class="docon docon-navigate-external x-hidden-focus"></span></a>
- Azure China 21Vianet 入口網站：<a href="https://portal.azure.cn/?feature.customportal=false#blade/HubsExtension/ArgQueryBlade/query/Resources%0D%0A%7C%20where%20type%20%3D~%20%27microsoft.sql%2Fservers%2Fdatabases%27%0D%0A%7C%20project%20databaseId%20%3D%20id%2C%20databaseName%20%3D%20name%2C%20elasticPoolId%20%3D%20tolower%28tostring%28properties.elasticPoolId%29%29%0D%0A%7C%20join%20kind%3Dleftouter%20%28%0D%0A%20%20%20%20Resources%0D%0A%20%20%20%20%7C%20where%20type%20%3D~%20%27microsoft.sql%2Fservers%2Felasticpools%27%0D%0A%20%20%20%20%7C%20project%20elasticPoolId%20%3D%20tolower%28id%29%2C%20elasticPoolName%20%3D%20name%2C%20elasticPoolState%20%3D%20properties.state%29%0D%0Aon%20elasticPoolId%0D%0A%7C%20project-away%20elasticPoolId1" target="_blank">portal.azure.cn <span class="docon docon-navigate-external x-hidden-focus"></span></a>

---

## <a name="list-virtual-machines-with-their-network-interface-and-public-ip"></a><a name="join-vmpip"></a>列出虛擬機器及其網路介面和公用 IP

此查詢會使用兩個 **leftouter** `join` 命令，將使用 Resource Manager 部署模型建立的虛擬機器、其相關的網路介面，以及任何與這些網路介面相關的公用 IP 位址結合在一起。

```kusto
Resources
| where type =~ 'microsoft.compute/virtualmachines'
| extend nics=array_length(properties.networkProfile.networkInterfaces) 
| mv-expand nic=properties.networkProfile.networkInterfaces 
| where nics == 1 or nic.properties.primary =~ 'true' or isempty(nic) 
| project vmId = id, vmName = name, vmSize=tostring(properties.hardwareProfile.vmSize), nicId = tostring(nic.id) 
| join kind=leftouter (
    Resources
    | where type =~ 'microsoft.network/networkinterfaces'
    | extend ipConfigsCount=array_length(properties.ipConfigurations) 
    | mv-expand ipconfig=properties.ipConfigurations 
    | where ipConfigsCount == 1 or ipconfig.properties.primary =~ 'true'
    | project nicId = id, publicIpId = tostring(ipconfig.properties.publicIPAddress.id))
on nicId
| project-away nicId1
| summarize by vmId, vmName, vmSize, nicId, publicIpId
| join kind=leftouter (
    Resources
    | where type =~ 'microsoft.network/publicipaddresses'
    | project publicIpId = id, publicIpAddress = properties.ipAddress)
on publicIpId
| project-away publicIpId1
```

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

```azurecli-interactive
az graph query -q "Resources | where type =~ 'microsoft.compute/virtualmachines' | extend nics=array_length(properties.networkProfile.networkInterfaces) | mv-expand nic=properties.networkProfile.networkInterfaces | where nics == 1 or nic.properties.primary =~ 'true' or isempty(nic) | project vmId = id, vmName = name, vmSize=tostring(properties.hardwareProfile.vmSize), nicId = tostring(nic.id) | join kind=leftouter ( Resources | where type =~ 'microsoft.network/networkinterfaces' | extend ipConfigsCount=array_length(properties.ipConfigurations) | mv-expand ipconfig=properties.ipConfigurations | where ipConfigsCount == 1 or ipconfig.properties.primary =~ 'true' | project nicId = id, publicIpId = tostring(ipconfig.properties.publicIPAddress.id)) on nicId | project-away nicId1 | summarize by vmId, vmName, vmSize, nicId, publicIpId | join kind=leftouter ( Resources | where type =~ 'microsoft.network/publicipaddresses' | project publicIpId = id, publicIpAddress = properties.ipAddress) on publicIpId | project-away publicIpId1"
```

# <a name="azure-powershell"></a>[Azure PowerShell](#tab/azure-powershell)

```azurepowershell-interactive
Search-AzGraph -Query "Resources | where type =~ 'microsoft.compute/virtualmachines' | extend nics=array_length(properties.networkProfile.networkInterfaces) | mv-expand nic=properties.networkProfile.networkInterfaces | where nics == 1 or nic.properties.primary =~ 'true' or isempty(nic) | project vmId = id, vmName = name, vmSize=tostring(properties.hardwareProfile.vmSize), nicId = tostring(nic.id) | join kind=leftouter ( Resources | where type =~ 'microsoft.network/networkinterfaces' | extend ipConfigsCount=array_length(properties.ipConfigurations) | mv-expand ipconfig=properties.ipConfigurations | where ipConfigsCount == 1 or ipconfig.properties.primary =~ 'true' | project nicId = id, publicIpId = tostring(ipconfig.properties.publicIPAddress.id)) on nicId | project-away nicId1 | summarize by vmId, vmName, vmSize, nicId, publicIpId | join kind=leftouter ( Resources | where type =~ 'microsoft.network/publicipaddresses' | project publicIpId = id, publicIpAddress = properties.ipAddress) on publicIpId | project-away publicIpId1"
```

# <a name="portal"></a>[入口網站](#tab/azure-portal)

:::image type="icon" source="../media/resource-graph-small.png":::在 Azure Resource Graph 總管中試用此查詢：

- Azure 入口網站：<a href="https://portal.azure.com/?feature.customportal=false#blade/HubsExtension/ArgQueryBlade/query/Resources%0D%0A%7C%20where%20type%20%3D~%20%27microsoft.compute%2Fvirtualmachines%27%0D%0A%7C%20extend%20nics%3Darray_length%28properties.networkProfile.networkInterfaces%29%20%0D%0A%7C%20mv-expand%20nic%3Dproperties.networkProfile.networkInterfaces%20%0D%0A%7C%20where%20nics%20%3D%3D%201%20or%20nic.properties.primary%20%3D~%20%27true%27%20or%20isempty%28nic%29%20%0D%0A%7C%20project%20vmId%20%3D%20id%2C%20vmName%20%3D%20name%2C%20vmSize%3Dtostring%28properties.hardwareProfile.vmSize%29%2C%20nicId%20%3D%20tostring%28nic.id%29%20%0D%0A%7C%20join%20kind%3Dleftouter%20%28%0D%0A%20%20%20%20Resources%0D%0A%20%20%20%20%7C%20where%20type%20%3D~%20%27microsoft.network%2Fnetworkinterfaces%27%0D%0A%20%20%20%20%7C%20extend%20ipConfigsCount%3Darray_length%28properties.ipConfigurations%29%20%0D%0A%20%20%20%20%7C%20mv-expand%20ipconfig%3Dproperties.ipConfigurations%20%0D%0A%20%20%20%20%7C%20where%20ipConfigsCount%20%3D%3D%201%20or%20ipconfig.properties.primary%20%3D~%20%27true%27%0D%0A%20%20%20%20%7C%20project%20nicId%20%3D%20id%2C%20publicIpId%20%3D%20tostring%28ipconfig.properties.publicIPAddress.id%29%29%0D%0Aon%20nicId%0D%0A%7C%20project-away%20nicId1%0D%0A%7C%20summarize%20by%20vmId%2C%20vmName%2C%20vmSize%2C%20nicId%2C%20publicIpId%0D%0A%7C%20join%20kind%3Dleftouter%20%28%0D%0A%20%20%20%20Resources%0D%0A%20%20%20%20%7C%20where%20type%20%3D~%20%27microsoft.network%2Fpublicipaddresses%27%0D%0A%20%20%20%20%7C%20project%20publicIpId%20%3D%20id%2C%20publicIpAddress%20%3D%20properties.ipAddress%29%0D%0Aon%20publicIpId%0D%0A%7C%20project-away%20publicIpId1" target="_blank">portal.azure.com <span class="docon docon-navigate-external x-hidden-focus"></span></a>
- Azure Government 入口網站：<a href="https://portal.azure.us/?feature.customportal=false#blade/HubsExtension/ArgQueryBlade/query/Resources%0D%0A%7C%20where%20type%20%3D~%20%27microsoft.compute%2Fvirtualmachines%27%0D%0A%7C%20extend%20nics%3Darray_length%28properties.networkProfile.networkInterfaces%29%20%0D%0A%7C%20mv-expand%20nic%3Dproperties.networkProfile.networkInterfaces%20%0D%0A%7C%20where%20nics%20%3D%3D%201%20or%20nic.properties.primary%20%3D~%20%27true%27%20or%20isempty%28nic%29%20%0D%0A%7C%20project%20vmId%20%3D%20id%2C%20vmName%20%3D%20name%2C%20vmSize%3Dtostring%28properties.hardwareProfile.vmSize%29%2C%20nicId%20%3D%20tostring%28nic.id%29%20%0D%0A%7C%20join%20kind%3Dleftouter%20%28%0D%0A%20%20%20%20Resources%0D%0A%20%20%20%20%7C%20where%20type%20%3D~%20%27microsoft.network%2Fnetworkinterfaces%27%0D%0A%20%20%20%20%7C%20extend%20ipConfigsCount%3Darray_length%28properties.ipConfigurations%29%20%0D%0A%20%20%20%20%7C%20mv-expand%20ipconfig%3Dproperties.ipConfigurations%20%0D%0A%20%20%20%20%7C%20where%20ipConfigsCount%20%3D%3D%201%20or%20ipconfig.properties.primary%20%3D~%20%27true%27%0D%0A%20%20%20%20%7C%20project%20nicId%20%3D%20id%2C%20publicIpId%20%3D%20tostring%28ipconfig.properties.publicIPAddress.id%29%29%0D%0Aon%20nicId%0D%0A%7C%20project-away%20nicId1%0D%0A%7C%20summarize%20by%20vmId%2C%20vmName%2C%20vmSize%2C%20nicId%2C%20publicIpId%0D%0A%7C%20join%20kind%3Dleftouter%20%28%0D%0A%20%20%20%20Resources%0D%0A%20%20%20%20%7C%20where%20type%20%3D~%20%27microsoft.network%2Fpublicipaddresses%27%0D%0A%20%20%20%20%7C%20project%20publicIpId%20%3D%20id%2C%20publicIpAddress%20%3D%20properties.ipAddress%29%0D%0Aon%20publicIpId%0D%0A%7C%20project-away%20publicIpId1" target="_blank">portal.azure.us <span class="docon docon-navigate-external x-hidden-focus"></span></a>
- Azure China 21Vianet 入口網站：<a href="https://portal.azure.cn/?feature.customportal=false#blade/HubsExtension/ArgQueryBlade/query/Resources%0D%0A%7C%20where%20type%20%3D~%20%27microsoft.compute%2Fvirtualmachines%27%0D%0A%7C%20extend%20nics%3Darray_length%28properties.networkProfile.networkInterfaces%29%20%0D%0A%7C%20mv-expand%20nic%3Dproperties.networkProfile.networkInterfaces%20%0D%0A%7C%20where%20nics%20%3D%3D%201%20or%20nic.properties.primary%20%3D~%20%27true%27%20or%20isempty%28nic%29%20%0D%0A%7C%20project%20vmId%20%3D%20id%2C%20vmName%20%3D%20name%2C%20vmSize%3Dtostring%28properties.hardwareProfile.vmSize%29%2C%20nicId%20%3D%20tostring%28nic.id%29%20%0D%0A%7C%20join%20kind%3Dleftouter%20%28%0D%0A%20%20%20%20Resources%0D%0A%20%20%20%20%7C%20where%20type%20%3D~%20%27microsoft.network%2Fnetworkinterfaces%27%0D%0A%20%20%20%20%7C%20extend%20ipConfigsCount%3Darray_length%28properties.ipConfigurations%29%20%0D%0A%20%20%20%20%7C%20mv-expand%20ipconfig%3Dproperties.ipConfigurations%20%0D%0A%20%20%20%20%7C%20where%20ipConfigsCount%20%3D%3D%201%20or%20ipconfig.properties.primary%20%3D~%20%27true%27%0D%0A%20%20%20%20%7C%20project%20nicId%20%3D%20id%2C%20publicIpId%20%3D%20tostring%28ipconfig.properties.publicIPAddress.id%29%29%0D%0Aon%20nicId%0D%0A%7C%20project-away%20nicId1%0D%0A%7C%20summarize%20by%20vmId%2C%20vmName%2C%20vmSize%2C%20nicId%2C%20publicIpId%0D%0A%7C%20join%20kind%3Dleftouter%20%28%0D%0A%20%20%20%20Resources%0D%0A%20%20%20%20%7C%20where%20type%20%3D~%20%27microsoft.network%2Fpublicipaddresses%27%0D%0A%20%20%20%20%7C%20project%20publicIpId%20%3D%20id%2C%20publicIpAddress%20%3D%20properties.ipAddress%29%0D%0Aon%20publicIpId%0D%0A%7C%20project-away%20publicIpId1" target="_blank">portal.azure.cn <span class="docon docon-navigate-external x-hidden-focus"></span></a>

---

## <a name="list-all-extensions-installed-on-a-virtual-machine"></a><a name="join-vmextension"></a>列出虛擬機器主機上安裝的所有擴充功能

首先，此查詢會使用虛擬機器資源類型上的 `extend`，以大寫 (`toupper()`) 識別碼取得識別碼、取得作業系統名稱和類型，以及取得虛擬機器大小。
以大寫取得資源識別碼是準備加入另一個屬性的好方法。 然後，此查詢會使用 `join` 搭配 **kind** 做為 _leftouter_，藉由比對擴充功能識別碼的大寫 `substring` 來取得虛擬機器擴充功能。 「/extensions/\<ExtensionName\>」之前的識別碼部分與虛擬機器識別碼的格式相同，因此我們將此屬性用於 `join`。 `summarize` 接著會與虛擬機器擴充功能名稱上的 `make_list` 搭配使用，以結合每個擴充功能的名稱，其中 _id_、_OSName_、_OSType_ 和 _VMSize_ 相同於單一陣列屬性。 最後，我們 `order by` 小寫的 _OSName_ 與 **asc**。 根據預設，`order by` 會遞減。

```kusto
Resources
| where type == 'microsoft.compute/virtualmachines'
| extend
    JoinID = toupper(id),
    OSName = tostring(properties.osProfile.computerName),
    OSType = tostring(properties.storageProfile.osDisk.osType),
    VMSize = tostring(properties.hardwareProfile.vmSize)
| join kind=leftouter(
    Resources
    | where type == 'microsoft.compute/virtualmachines/extensions'
    | extend 
        VMId = toupper(substring(id, 0, indexof(id, '/extensions'))),
        ExtensionName = name
) on $left.JoinID == $right.VMId
| summarize Extensions = make_list(ExtensionName) by id, OSName, OSType, VMSize
| order by tolower(OSName) asc
```

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

```azurecli-interactive
az graph query -q "Resources | where type == 'microsoft.compute/virtualmachines' | extend JoinID = toupper(id), OSName = tostring(properties.osProfile.computerName), OSType = tostring(properties.storageProfile.osDisk.osType), VMSize = tostring(properties.hardwareProfile.vmSize) | join kind=leftouter( Resources | where type == 'microsoft.compute/virtualmachines/extensions' | extend VMId = toupper(substring(id, 0, indexof(id, '/extensions'))), ExtensionName = name ) on $left.JoinID == $right.VMId | summarize Extensions = make_list(ExtensionName) by id, OSName, OSType, VMSize | order by tolower(OSName) asc"
```

# <a name="azure-powershell"></a>[Azure PowerShell](#tab/azure-powershell)

```azurepowershell-interactive
Search-AzGraph -Query "Resources | where type == 'microsoft.compute/virtualmachines' | extend JoinID = toupper(id), OSName = tostring(properties.osProfile.computerName), OSType = tostring(properties.storageProfile.osDisk.osType), VMSize = tostring(properties.hardwareProfile.vmSize) | join kind=leftouter( Resources | where type == 'microsoft.compute/virtualmachines/extensions' | extend VMId = toupper(substring(id, 0, indexof(id, '/extensions'))), ExtensionName = name ) on $left.JoinID == $right.VMId | summarize Extensions = make_list(ExtensionName) by id, OSName, OSType, VMSize | order by tolower(OSName) asc"
```

# <a name="portal"></a>[入口網站](#tab/azure-portal)

:::image type="icon" source="../media/resource-graph-small.png":::在 Azure Resource Graph 總管中試用此查詢：

- Azure 入口網站：<a href="https://portal.azure.com/?feature.customportal=false#blade/HubsExtension/ArgQueryBlade/query/Resources%0A%7C%20where%20type%20%3D%3D%20'microsoft.compute%2Fvirtualmachines'%0A%7C%20extend%0A%20%20%20%20JoinID%20%3D%20toupper(id)%2C%0A%20%20%20%20OSName%20%3D%20tostring(properties.osProfile.computerName)%2C%0A%20%20%20%20OSType%20%3D%20tostring(properties.storageProfile.osDisk.osType)%2C%0A%20%20%20%20VMSize%20%3D%20tostring(properties.hardwareProfile.vmSize)%0A%7C%20join%20kind%3Dleftouter(%0A%20%20%20%20Resources%0A%20%20%20%20%7C%20where%20type%20%3D%3D%20'microsoft.compute%2Fvirtualmachines%2Fextensions'%0A%20%20%20%20%7C%20extend%20%0A%20%20%20%20%20%20%20%20VMId%20%3D%20toupper(substring(id%2C%200%2C%20indexof(id%2C%20'%2Fextensions')))%2C%0A%20%20%20%20%20%20%20%20ExtensionName%20%3D%20name%0A)%20on%20%24left.JoinID%20%3D%3D%20%24right.VMId%0A%7C%20summarize%20Extensions%20%3D%20make_list(ExtensionName)%20by%20id%2C%20OSName%2C%20OSType%2C%20VMSize%0A%7C%20order%20by%20tolower(OSName)%20asc" target="_blank">portal.azure.com <span class="docon docon-navigate-external x-hidden-focus"></span></a>
- Azure Government 入口網站：<a href="https://portal.azure.us/?feature.customportal=false#blade/HubsExtension/ArgQueryBlade/query/Resources%0A%7C%20where%20type%20%3D%3D%20'microsoft.compute%2Fvirtualmachines'%0A%7C%20extend%0A%20%20%20%20JoinID%20%3D%20toupper(id)%2C%0A%20%20%20%20OSName%20%3D%20tostring(properties.osProfile.computerName)%2C%0A%20%20%20%20OSType%20%3D%20tostring(properties.storageProfile.osDisk.osType)%2C%0A%20%20%20%20VMSize%20%3D%20tostring(properties.hardwareProfile.vmSize)%0A%7C%20join%20kind%3Dleftouter(%0A%20%20%20%20Resources%0A%20%20%20%20%7C%20where%20type%20%3D%3D%20'microsoft.compute%2Fvirtualmachines%2Fextensions'%0A%20%20%20%20%7C%20extend%20%0A%20%20%20%20%20%20%20%20VMId%20%3D%20toupper(substring(id%2C%200%2C%20indexof(id%2C%20'%2Fextensions')))%2C%0A%20%20%20%20%20%20%20%20ExtensionName%20%3D%20name%0A)%20on%20%24left.JoinID%20%3D%3D%20%24right.VMId%0A%7C%20summarize%20Extensions%20%3D%20make_list(ExtensionName)%20by%20id%2C%20OSName%2C%20OSType%2C%20VMSize%0A%7C%20order%20by%20tolower(OSName)%20asc" target="_blank">portal.azure.us <span class="docon docon-navigate-external x-hidden-focus"></span></a>
- Azure China 21Vianet 入口網站：<a href="https://portal.azure.cn/?feature.customportal=false#blade/HubsExtension/ArgQueryBlade/query/Resources%0A%7C%20where%20type%20%3D%3D%20'microsoft.compute%2Fvirtualmachines'%0A%7C%20extend%0A%20%20%20%20JoinID%20%3D%20toupper(id)%2C%0A%20%20%20%20OSName%20%3D%20tostring(properties.osProfile.computerName)%2C%0A%20%20%20%20OSType%20%3D%20tostring(properties.storageProfile.osDisk.osType)%2C%0A%20%20%20%20VMSize%20%3D%20tostring(properties.hardwareProfile.vmSize)%0A%7C%20join%20kind%3Dleftouter(%0A%20%20%20%20Resources%0A%20%20%20%20%7C%20where%20type%20%3D%3D%20'microsoft.compute%2Fvirtualmachines%2Fextensions'%0A%20%20%20%20%7C%20extend%20%0A%20%20%20%20%20%20%20%20VMId%20%3D%20toupper(substring(id%2C%200%2C%20indexof(id%2C%20'%2Fextensions')))%2C%0A%20%20%20%20%20%20%20%20ExtensionName%20%3D%20name%0A)%20on%20%24left.JoinID%20%3D%3D%20%24right.VMId%0A%7C%20summarize%20Extensions%20%3D%20make_list(ExtensionName)%20by%20id%2C%20OSName%2C%20OSType%2C%20VMSize%0A%7C%20order%20by%20tolower(OSName)%20asc" target="_blank">portal.azure.cn <span class="docon docon-navigate-external x-hidden-focus"></span></a>

---

## <a name="find-storage-accounts-with-a-specific-tag-on-the-resource-group"></a><a name="join-findstoragetag"></a>在資源群組上尋找具有特定標籤的儲存體帳戶

下列查詢使用 **inner** `join`，來連接儲存體帳戶與具有指定標籤名稱 (區分大小寫) 和標籤值的資源群組。

```kusto
Resources
| where type =~ 'microsoft.storage/storageaccounts'
| join kind=inner (
    ResourceContainers
    | where type =~ 'microsoft.resources/subscriptions/resourcegroups'
    | where tags['Key1'] =~ 'Value1'
    | project subscriptionId, resourceGroup)
on subscriptionId, resourceGroup
| project-away subscriptionId1, resourceGroup1
```

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

```azurecli-interactive
az graph query -q "Resources | where type =~ 'microsoft.storage/storageaccounts' | join kind=inner ( ResourceContainers | where type =~ 'microsoft.resources/subscriptions/resourcegroups' | where tags['Key1'] =~ 'Value1' | project subscriptionId, resourceGroup) on subscriptionId, resourceGroup | project-away subscriptionId1, resourceGroup1"
```

# <a name="azure-powershell"></a>[Azure PowerShell](#tab/azure-powershell)

```azurepowershell-interactive
Search-AzGraph -Query "Resources | where type =~ 'microsoft.storage/storageaccounts' | join kind=inner ( ResourceContainers | where type =~ 'microsoft.resources/subscriptions/resourcegroups' | where tags['Key1'] =~ 'Value1' | project subscriptionId, resourceGroup) on subscriptionId, resourceGroup | project-away subscriptionId1, resourceGroup1"
```

# <a name="portal"></a>[入口網站](#tab/azure-portal)

:::image type="icon" source="../media/resource-graph-small.png":::在 Azure Resource Graph 總管中試用此查詢：

- Azure 入口網站：<a href="https://portal.azure.com/?feature.customportal=false#blade/HubsExtension/ArgQueryBlade/query/Resources%0D%0A%7C%20where%20type%20%3D~%20%27microsoft.storage%2Fstorageaccounts%27%0D%0A%7C%20join%20kind%3Dinner%20%28%0D%0A%20%20%20%20ResourceContainers%0D%0A%20%20%20%20%7C%20where%20type%20%3D~%20%27microsoft.resources%2Fsubscriptions%2Fresourcegroups%27%0D%0A%20%20%20%20%7C%20where%20tags%5B%27Key1%27%5D%20%3D~%20%27Value1%27%0D%0A%20%20%20%20%7C%20project%20subscriptionId%2C%20resourceGroup%29%0D%0Aon%20subscriptionId%2C%20resourceGroup%0D%0A%7C%20project-away%20subscriptionId1%2C%20resourceGroup1" target="_blank">portal.azure.com <span class="docon docon-navigate-external x-hidden-focus"></span></a>
- Azure Government 入口網站：<a href="https://portal.azure.us/?feature.customportal=false#blade/HubsExtension/ArgQueryBlade/query/Resources%0D%0A%7C%20where%20type%20%3D~%20%27microsoft.storage%2Fstorageaccounts%27%0D%0A%7C%20join%20kind%3Dinner%20%28%0D%0A%20%20%20%20ResourceContainers%0D%0A%20%20%20%20%7C%20where%20type%20%3D~%20%27microsoft.resources%2Fsubscriptions%2Fresourcegroups%27%0D%0A%20%20%20%20%7C%20where%20tags%5B%27Key1%27%5D%20%3D~%20%27Value1%27%0D%0A%20%20%20%20%7C%20project%20subscriptionId%2C%20resourceGroup%29%0D%0Aon%20subscriptionId%2C%20resourceGroup%0D%0A%7C%20project-away%20subscriptionId1%2C%20resourceGroup1" target="_blank">portal.azure.us <span class="docon docon-navigate-external x-hidden-focus"></span></a>
- Azure China 21Vianet 入口網站：<a href="https://portal.azure.cn/?feature.customportal=false#blade/HubsExtension/ArgQueryBlade/query/Resources%0D%0A%7C%20where%20type%20%3D~%20%27microsoft.storage%2Fstorageaccounts%27%0D%0A%7C%20join%20kind%3Dinner%20%28%0D%0A%20%20%20%20ResourceContainers%0D%0A%20%20%20%20%7C%20where%20type%20%3D~%20%27microsoft.resources%2Fsubscriptions%2Fresourcegroups%27%0D%0A%20%20%20%20%7C%20where%20tags%5B%27Key1%27%5D%20%3D~%20%27Value1%27%0D%0A%20%20%20%20%7C%20project%20subscriptionId%2C%20resourceGroup%29%0D%0Aon%20subscriptionId%2C%20resourceGroup%0D%0A%7C%20project-away%20subscriptionId1%2C%20resourceGroup1" target="_blank">portal.azure.cn <span class="docon docon-navigate-external x-hidden-focus"></span></a>

---

若需要尋找不區分大小寫的標籤名稱和標籤值，請使用 `mv-expand` 搭配 **bagexpansion** 參數。 此查詢所使用的配額高於先前的查詢，因此請在必要時才使用 `mv-expand`。

```kusto
Resources
| where type =~ 'microsoft.storage/storageaccounts'
| join kind=inner (
    ResourceContainers
    | where type =~ 'microsoft.resources/subscriptions/resourcegroups'
    | mv-expand bagexpansion=array tags
    | where isnotempty(tags)
    | where tags[0] =~ 'key1' and tags[1] =~ 'value1'
    | project subscriptionId, resourceGroup)
on subscriptionId, resourceGroup
| project-away subscriptionId1, resourceGroup1
```

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

```azurecli-interactive
az graph query -q "Resources | where type =~ 'microsoft.storage/storageaccounts' | join kind=inner ( ResourceContainers | where type =~ 'microsoft.resources/subscriptions/resourcegroups' | mv-expand bagexpansion=array tags | where isnotempty(tags) | where tags[0] =~ 'key1' and tags[1] =~ 'value1' | project subscriptionId, resourceGroup) on subscriptionId, resourceGroup | project-away subscriptionId1, resourceGroup1"
```

# <a name="azure-powershell"></a>[Azure PowerShell](#tab/azure-powershell)

```azurepowershell-interactive
Search-AzGraph -Query "Resources | where type =~ 'microsoft.storage/storageaccounts' | join kind=inner ( ResourceContainers | where type =~ 'microsoft.resources/subscriptions/resourcegroups' | mv-expand bagexpansion=array tags | where isnotempty(tags) | where tags[0] =~ 'key1' and tags[1] =~ 'value1' | project subscriptionId, resourceGroup) on subscriptionId, resourceGroup | project-away subscriptionId1, resourceGroup1"
```

# <a name="portal"></a>[入口網站](#tab/azure-portal)

:::image type="icon" source="../media/resource-graph-small.png":::在 Azure Resource Graph 總管中試用此查詢：

- Azure 入口網站：<a href="https://portal.azure.com/?feature.customportal=false#blade/HubsExtension/ArgQueryBlade/query/Resources%0D%0A%7C%20where%20type%20%3D~%20%27microsoft.storage%2Fstorageaccounts%27%0D%0A%7C%20join%20kind%3Dinner%20%28%0D%0A%20%20%20%20ResourceContainers%0D%0A%20%20%20%20%7C%20where%20type%20%3D~%20%27microsoft.resources%2Fsubscriptions%2Fresourcegroups%27%0D%0A%20%20%20%20%7C%20mv-expand%20bagexpansion%3Darray%20tags%0D%0A%20%20%20%20%7C%20where%20isnotempty%28tags%29%0D%0A%20%20%20%20%7C%20where%20tags%5B0%5D%20%3D~%20%27key1%27%20and%20tags%5B1%5D%20%3D~%20%27value1%27%0D%0A%20%20%20%20%7C%20project%20subscriptionId%2C%20resourceGroup%29%0D%0Aon%20subscriptionId%2C%20resourceGroup%0D%0A%7C%20project-away%20subscriptionId1%2C%20resourceGroup1" target="_blank">portal.azure.com <span class="docon docon-navigate-external x-hidden-focus"></span></a>
- Azure Government 入口網站：<a href="https://portal.azure.us/?feature.customportal=false#blade/HubsExtension/ArgQueryBlade/query/Resources%0D%0A%7C%20where%20type%20%3D~%20%27microsoft.storage%2Fstorageaccounts%27%0D%0A%7C%20join%20kind%3Dinner%20%28%0D%0A%20%20%20%20ResourceContainers%0D%0A%20%20%20%20%7C%20where%20type%20%3D~%20%27microsoft.resources%2Fsubscriptions%2Fresourcegroups%27%0D%0A%20%20%20%20%7C%20mv-expand%20bagexpansion%3Darray%20tags%0D%0A%20%20%20%20%7C%20where%20isnotempty%28tags%29%0D%0A%20%20%20%20%7C%20where%20tags%5B0%5D%20%3D~%20%27key1%27%20and%20tags%5B1%5D%20%3D~%20%27value1%27%0D%0A%20%20%20%20%7C%20project%20subscriptionId%2C%20resourceGroup%29%0D%0Aon%20subscriptionId%2C%20resourceGroup%0D%0A%7C%20project-away%20subscriptionId1%2C%20resourceGroup1" target="_blank">portal.azure.us <span class="docon docon-navigate-external x-hidden-focus"></span></a>
- Azure China 21Vianet 入口網站：<a href="https://portal.azure.cn/?feature.customportal=false#blade/HubsExtension/ArgQueryBlade/query/Resources%0D%0A%7C%20where%20type%20%3D~%20%27microsoft.storage%2Fstorageaccounts%27%0D%0A%7C%20join%20kind%3Dinner%20%28%0D%0A%20%20%20%20ResourceContainers%0D%0A%20%20%20%20%7C%20where%20type%20%3D~%20%27microsoft.resources%2Fsubscriptions%2Fresourcegroups%27%0D%0A%20%20%20%20%7C%20mv-expand%20bagexpansion%3Darray%20tags%0D%0A%20%20%20%20%7C%20where%20isnotempty%28tags%29%0D%0A%20%20%20%20%7C%20where%20tags%5B0%5D%20%3D~%20%27key1%27%20and%20tags%5B1%5D%20%3D~%20%27value1%27%0D%0A%20%20%20%20%7C%20project%20subscriptionId%2C%20resourceGroup%29%0D%0Aon%20subscriptionId%2C%20resourceGroup%0D%0A%7C%20project-away%20subscriptionId1%2C%20resourceGroup1" target="_blank">portal.azure.cn <span class="docon docon-navigate-external x-hidden-focus"></span></a>

---

## <a name="combine-results-from-two-queries-into-a-single-result"></a><a name="unionresults"></a>將兩個查詢的結果合併成單一結果

下列查詢會使用 `union` 從 _ResourceContainers_ 資料表取得結果，然後將其新增至 _Resources_ 資料表的結果中。

```kusto
ResourceContainers
| where type=='microsoft.resources/subscriptions/resourcegroups' | project name, type  | limit 5
| union  (Resources | project name, type | limit 5)
```

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

```azurecli-interactive
az graph query -q "ResourceContainers | where type=='microsoft.resources/subscriptions/resourcegroups' | project name, type  | limit 5 | union  (Resources | project name, type | limit 5)"
```

# <a name="azure-powershell"></a>[Azure PowerShell](#tab/azure-powershell)

```azurepowershell-interactive
Search-AzGraph -Query "ResourceContainers | where type=='microsoft.resources/subscriptions/resourcegroups' | project name, type  | limit 5 | union  (Resources | project name, type | limit 5)"
```

# <a name="portal"></a>[入口網站](#tab/azure-portal)

:::image type="icon" source="../media/resource-graph-small.png":::在 Azure Resource Graph 總管中試用此查詢：

- Azure 入口網站：<a href="https://portal.azure.com/?feature.customportal=false#blade/HubsExtension/ArgQueryBlade/query/ResourceContainers%0D%0A%7C%20where%20type%3D%3D%27microsoft.resources%2Fsubscriptions%2Fresourcegroups%27%20%7C%20project%20name%2C%20type%20%20%7C%20limit%205%0D%0A%7C%20union%20%20%28Resources%20%7C%20project%20name%2C%20type%20%7C%20limit%205%29" target="_blank">portal.azure.com <span class="docon docon-navigate-external x-hidden-focus"></span></a>
- Azure Government 入口網站：<a href="https://portal.azure.us/?feature.customportal=false#blade/HubsExtension/ArgQueryBlade/query/ResourceContainers%0D%0A%7C%20where%20type%3D%3D%27microsoft.resources%2Fsubscriptions%2Fresourcegroups%27%20%7C%20project%20name%2C%20type%20%20%7C%20limit%205%0D%0A%7C%20union%20%20%28Resources%20%7C%20project%20name%2C%20type%20%7C%20limit%205%29" target="_blank">portal.azure.us <span class="docon docon-navigate-external x-hidden-focus"></span></a>
- Azure China 21Vianet 入口網站：<a href="https://portal.azure.cn/?feature.customportal=false#blade/HubsExtension/ArgQueryBlade/query/ResourceContainers%0D%0A%7C%20where%20type%3D%3D%27microsoft.resources%2Fsubscriptions%2Fresourcegroups%27%20%7C%20project%20name%2C%20type%20%20%7C%20limit%205%0D%0A%7C%20union%20%20%28Resources%20%7C%20project%20name%2C%20type%20%7C%20limit%205%29" target="_blank">portal.azure.cn <span class="docon docon-navigate-external x-hidden-focus"></span></a>

---

## <a name="summarize-virtual-machine-by-the-power-states-extended-property"></a><a name="vm-powerstate"></a>依電源狀態擴充屬性來摘要說明虛擬機器

此查詢會使用虛擬機器上的[擴充屬性](../concepts/query-language.md#extended-properties)，以便依電源狀態進行摘要說明。


```kusto
Resources
| where type == 'microsoft.compute/virtualmachines'
| summarize count() by tostring(properties.extended.instanceView.powerState.code)
```

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

```azurecli-interactive
az graph query -q "Resources | where type == 'microsoft.compute/virtualmachines' | summarize count() by tostring(properties.extended.instanceView.powerState.code)"
```

# <a name="azure-powershell"></a>[Azure PowerShell](#tab/azure-powershell)

```azurepowershell-interactive
Search-AzGraph -Query "Resources | where type == 'microsoft.compute/virtualmachines' | summarize count() by tostring(properties.extended.instanceView.powerState.code)"
```

# <a name="portal"></a>[入口網站](#tab/azure-portal)

:::image type="icon" source="../media/resource-graph-small.png":::在 Azure Resource Graph 總管中試用此查詢：

- Azure 入口網站：<a href="https://portal.azure.com/?feature.customportal=false#blade/HubsExtension/ArgQueryBlade/query/Resources%20%7C%20where%20type%20%3D%3D%20%27microsoft.compute%2Fvirtualmachines%27%20%7C%20summarize%20count%28%29%20by%20tostring%28properties.extended.instanceView.powerState.code%29" target="_blank">portal.azure.com <span class="docon docon-navigate-external x-hidden-focus"></span></a>
- Azure Government 入口網站：<a href="https://portal.azure.us/?feature.customportal=false#blade/HubsExtension/ArgQueryBlade/query/Resources%20%7C%20where%20type%20%3D%3D%20%27microsoft.compute%2Fvirtualmachines%27%20%7C%20summarize%20count%28%29%20by%20tostring%28properties.extended.instanceView.powerState.code%29" target="_blank">portal.azure.us <span class="docon docon-navigate-external x-hidden-focus"></span></a>
- Azure China 21Vianet 入口網站：<a href="https://portal.azure.cn/?feature.customportal=false#blade/HubsExtension/ArgQueryBlade/query/Resources%20%7C%20where%20type%20%3D%3D%20%27microsoft.compute%2Fvirtualmachines%27%20%7C%20summarize%20count%28%29%20by%20tostring%28properties.extended.instanceView.powerState.code%29" target="_blank">portal.azure.cn <span class="docon docon-navigate-external x-hidden-focus"></span></a>

---

## <a name="include-the-tenant-and-subscription-names-with-displaynames"></a><a name="displaynames"></a>包含租用戶和訂用帳戶名稱與 DisplayNames

此查詢使用 **Include** 參數搭配選項 DisplayNames 來將 **subscriptionDisplayName** 和 **tenantDisplayName** 新增至結果。 此參數僅適用於 Azure CLI 和 Azure PowerShell。

```azurecli-interactive
az graph query -q "limit 1" --include displayNames
```

```azurepowershell-interactive
Search-AzGraph -Query "limit 1" -Include DisplayNames
```

可供取得訂用帳戶名稱的替代方法是使用 `join` 運算子，並連結到 **ResourceContainers** 資料表和 `Microsoft.Resources/subscriptions` 類型。 `join` 適用於 Azure CLI、Azure PowerShell、入口網站和所有支援的 SDK。 如需範例，請參閱[範例 - 金鑰保存庫與訂用帳戶名稱](#join)。

> [!NOTE]
> 如果查詢未使用 **project** 來指定傳回的屬性，則 **subscriptionDisplayName** 和 **tenantDisplayName** 就會自動包含在結果中。
> 如果查詢有使用 **project**，則每個 [DisplayName] 欄位都必須明確包含在 **project** 中，否則不會在結果中傳回，即使使用 **Include** 參數也一樣。 **Include** 參數無法與 [tables](../concepts/query-language.md#resource-graph-tables) 搭配運作。

---

## <a name="count-of-non-compliant-guest-configuration-assignments"></a><a name="count-gcnoncompliant"></a>不符合規範的來賓設定指派計數

顯示每個[來賓設定指派原因](../../policy/how-to/determine-non-compliance.md#compliance-details-for-guest-configuration)的不符合規範電腦計數。 將效能的結果限制為前 100 個。

```kusto
GuestConfigurationResources
| extend vmid = split(properties.targetResourceId,'/')
| where properties.complianceStatus == 'NonCompliant'
| mvexpand properties.latestAssignmentReport.resources
| mvexpand properties_latestAssignmentReport_resources.reasons
| project machine = tostring(vmid[(-1)]),
    type = tostring(vmid[(-3)]),
    name,
    status = tostring(properties.complianceStatus),
    resource = tostring(properties_latestAssignmentReport_resources.resourceId),
    phrase = tostring(properties_latestAssignmentReport_resources_reasons.phrase)
| summarize count() by resource, name
| order by count_
| limit 100
```

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

```azurecli-interactive
az graph query -q "GuestConfigurationResources | extend vmid = split(properties.targetResourceId,'/') | where properties.complianceStatus == 'NonCompliant' | mvexpand properties.latestAssignmentReport.resources | mvexpand properties_latestAssignmentReport_resources.reasons | project machine = tostring(vmid[(-1)]), type = tostring(vmid[(-3)]), name, status = tostring(properties.complianceStatus), resource = tostring(properties_latestAssignmentReport_resources.resourceId), phrase = tostring(properties_latestAssignmentReport_resources_reasons.phrase) | summarize count() by resource, name | order by count_ | limit 100"
```

# <a name="azure-powershell"></a>[Azure PowerShell](#tab/azure-powershell)

```azurepowershell-interactive
Search-AzGraph -Query "GuestConfigurationResources | extend vmid = split(properties.targetResourceId,'/') | where properties.complianceStatus == 'NonCompliant' | mvexpand properties.latestAssignmentReport.resources | mvexpand properties_latestAssignmentReport_resources.reasons | project machine = tostring(vmid[(-1)]), type = tostring(vmid[(-3)]), name, status = tostring(properties.complianceStatus), resource = tostring(properties_latestAssignmentReport_resources.resourceId), phrase = tostring(properties_latestAssignmentReport_resources_reasons.phrase) | summarize count() by resource, name | order by count_ | limit 100"
```

# <a name="portal"></a>[入口網站](#tab/azure-portal)

:::image type="icon" source="../media/resource-graph-small.png":::在 Azure Resource Graph 總管中試用此查詢：

- Azure 入口網站：<a href="https://portal.azure.com/?feature.customportal=false#blade/HubsExtension/ArgQueryBlade/query/GuestConfigurationResources%20%7C%20extend%20vmid%20%3D%20split(properties.targetResourceId%2C%22%2F%22)%20%7C%20where%20properties.complianceStatus%20%3D%3D%20'NonCompliant'%20%7C%20mvexpand%20properties.latestAssignmentReport.resources%20%7C%20mvexpand%20properties_latestAssignmentReport_resources.reasons%20%7C%20project%20machine%20%3D%20tostring(vmid%5B(-1)%5D)%2C%20type%20%3D%20tostring(vmid%5B(-3)%5D)%2C%20name%2C%20status%20%3D%20tostring(properties.complianceStatus)%2C%20resource%20%3D%20tostring(properties_latestAssignmentReport_resources.resourceId)%2C%20phrase%20%3D%20tostring(properties_latestAssignmentReport_resources_reasons.phrase)%20%7C%20summarize%20count()%20by%20resource%2C%20name%20%7C%20order%20by%20count_%20%7C%20limit%20100" target="_blank">portal.azure.com <span class="docon docon-navigate-external x-hidden-focus"></span></a>

---

## <a name="query-details-of-guest-configuration-assignment-reports"></a><a name="query-gcreports"></a>查詢來賓設定指派報告的詳細資料

顯示[來賓設定指派原因](../../policy/how-to/determine-non-compliance.md#compliance-details-for-guest-configuration)詳細資料 中的報告。 在下列範例中，查詢只會傳回來賓指派名稱為 `installed_application_linux` 的結果，而輸出包含字串 `Python` 以列出已安裝套件的所有 Linux 電腦，其中包含 **Python** 這個名稱。 若要查詢特定指派的所有電腦合規性，請移除第二個 `where` 子句。

```kusto
GuestConfigurationResources
| extend vmid = split(properties.targetResourceId,'/')
| mvexpand properties.latestAssignmentReport.resources
| mvexpand properties_latestAssignmentReport_resources.reasons
| where name in ('installed_application_linux')
| where properties_latestAssignmentReport_resources_reasons.phrase contains 'Python'
| project machine = tostring(vmid[(-1)]),
    type = tostring(vmid[(-3)]),
    name,
    status = tostring(properties.complianceStatus),
    resource = tostring(properties_latestAssignmentReport_resources.resourceId),
    phrase = tostring(properties_latestAssignmentReport_resources_reasons.phrase)
```

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

```azurecli-interactive
az graph query -q "GuestConfigurationResources | extend vmid = split(properties.targetResourceId,'/') | mvexpand properties.latestAssignmentReport.resources | mvexpand properties_latestAssignmentReport_resources.reasons | where name in ('installed_application_linux') | where properties_latestAssignmentReport_resources_reasons.phrase contains 'Python' | project machine = tostring(vmid[(-1)]), type = tostring(vmid[(-3)]), name, status = tostring(properties.complianceStatus), resource = tostring(properties_latestAssignmentReport_resources.resourceId), phrase = tostring (properties_latestAssignmentReport_resources_reasons.phrase)"
```

# <a name="azure-powershell"></a>[Azure PowerShell](#tab/azure-powershell)

```azurepowershell-interactive
Search-AzGraph -Query "GuestConfigurationResources | extend vmid = split(properties.targetResourceId,'/') | mvexpand properties.latestAssignmentReport.resources | mvexpand properties_latestAssignmentReport_resources.reasons | where name in ('installed_application_linux') | where properties_latestAssignmentReport_resources_reasons.phrase contains 'Python' | project machine = tostring(vmid[(-1)]), type = tostring(vmid[(-3)]), name, status = tostring(properties.complianceStatus), resource = tostring(properties_latestAssignmentReport_resources.resourceId), phrase = tostring (properties_latestAssignmentReport_resources_reasons.phrase)"
```

# <a name="portal"></a>[入口網站](#tab/azure-portal)

:::image type="icon" source="../media/resource-graph-small.png":::在 Azure Resource Graph 總管中試用此查詢：

- Azure 入口網站：<a href="https://portal.azure.com/?feature.customportal=false#blade/HubsExtension/ArgQueryBlade/query/GuestConfigurationResources%20%7C%20extend%20vmid%20%3D%20split(properties.targetResourceId%2C'%2F')%20%7C%20mvexpand%20properties.latestAssignmentReport.resources%20%7C%20mvexpand%20properties_latestAssignmentReport_resources.reasons%20%7C%20where%20name%20in%20('installed_application_linux')%20%7C%20where%20properties_latestAssignmentReport_resources_reasons.phrase%20contains%20'Python'%20%7C%20project%20machine%20%3D%20tostring(vmid%5B(-1)%5D)%2C%20type%20%3D%20tostring(vmid%5B(-3)%5D)%2C%20name%2C%20status%20%3D%20tostring(properties.complianceStatus)%2C%20resource%20%3D%20tostring(properties_latestAssignmentReport_resources.resourceId)%2C%20phrase%20%3D%20tostring%20(properties_latestAssignmentReport_resources_reasons.phrase)" target="_blank">portal.azure.com <span class="docon docon-navigate-external x-hidden-focus"></span></a>

---

## <a name="find-all-reasons-a-machine-is-non-compliant-for-guest-configuration-assignments"></a><a name="query-gcmachinedetails"></a>尋找電腦不符合來賓設定指派規範的所有原因

顯示特定電腦的所有[來賓設定指派原因](../../policy/how-to/determine-non-compliance.md#compliance-details-for-guest-configuration)。 移除第一個 `where` 子句，同時包含電腦符合規範的稽核。

```kusto
GuestConfigurationResources
| where properties.complianceStatus == 'NonCompliant'
| extend vmid = split(properties.targetResourceId,'/')
| mvexpand properties.latestAssignmentReport.resources
| mvexpand properties_latestAssignmentReport_resources.reasons
| extend machine = tostring(vmid[(-1)])
| where machine == 'MACHINENAME'
| project phrase = tostring(properties_latestAssignmentReport_resources_reasons.phrase),
    resource = tostring(properties_latestAssignmentReport_resources.resourceId),
    name,
    machine,
    resourceGroup,
    subscriptionId
```

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

```azurecli-interactive
az graph query -q "GuestConfigurationResources | where properties.complianceStatus == 'NonCompliant' | extend vmid = split(properties.targetResourceId,'/') | mvexpand properties.latestAssignmentReport.resources | mvexpand properties_latestAssignmentReport_resources.reasons | extend machine = tostring(vmid[(-1)]) | where machine == 'MACHINENAME' | project phrase = tostring(properties_latestAssignmentReport_resources_reasons.phrase), resource = tostring(properties_latestAssignmentReport_resources.resourceId), name, machine, resourceGroup, subscriptionId"
```

# <a name="azure-powershell"></a>[Azure PowerShell](#tab/azure-powershell)

```azurepowershell-interactive
Search-AzGraph -Query "GuestConfigurationResources | where properties.complianceStatus == 'NonCompliant' | extend vmid = split(properties.targetResourceId,'/') | mvexpand properties.latestAssignmentReport.resources | mvexpand properties_latestAssignmentReport_resources.reasons | extend machine = tostring(vmid[(-1)]) | where machine == 'MACHINENAME' | project phrase = tostring(properties_latestAssignmentReport_resources_reasons.phrase), resource = tostring(properties_latestAssignmentReport_resources.resourceId), name, machine, resourceGroup, subscriptionId"
```

# <a name="portal"></a>[入口網站](#tab/azure-portal)

:::image type="icon" source="../media/resource-graph-small.png":::在 Azure Resource Graph 總管中試用此查詢：

- Azure 入口網站：<a href="https://portal.azure.com/?feature.customportal=false#blade/HubsExtension/ArgQueryBlade/query/GuestConfigurationResources%20%7C%20where%20properties.complianceStatus%20%3D%3D%20'NonCompliant'%20%7C%20extend%20vmid%20%3D%20split(properties.targetResourceId%2C'%2F')%20%7C%20mvexpand%20properties.latestAssignmentReport.resources%20%7C%20mvexpand%20properties_latestAssignmentReport_resources.reasons%20%7C%20extend%20machine%20%3D%20tostring(vmid%5B(-1)%5D)%20%7C%20where%20machine%20%3D%3D%20'MACHINENAME'%20%7C%20project%20phrase%20%3D%20tostring(properties_latestAssignmentReport_resources_reasons.phrase)%2C%20resource%20%3D%20tostring(properties_latestAssignmentReport_resources.resourceId)%2C%20name%2C%20machine%2C%20resourceGroup%2C%20subscriptionId" target="_blank">portal.azure.com <span class="docon docon-navigate-external x-hidden-focus"></span></a>

## <a name="next-steps"></a>後續步驟

- 請參閱[入門查詢](starter.md)的範例。
- 深入了解[查詢語言](../concepts/query-language.md)。
- 深入了解如何[探索資源](../concepts/explore-resources.md)。