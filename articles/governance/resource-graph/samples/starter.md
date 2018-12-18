---
title: 入門 Azure Resource Graph 查詢
description: 使用 Azure Resource Graph 執行一些入門查詢。
services: resource-graph
author: DCtheGeek
ms.author: dacoulte
ms.date: 09/18/2018
ms.topic: quickstart
ms.service: resource-graph
manager: carmonm
ms.custom: mvc
ms.openlocfilehash: bcc6c6e9d38948cb4be09c9f52049790ccd5ac7e
ms.sourcegitcommit: ad08b2db50d63c8f550575d2e7bb9a0852efb12f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/26/2018
ms.locfileid: "47227022"
---
# <a name="starter-resource-graph-queries"></a>入門 Resource Graph 查詢

透過 Azure Resource Graph 了解查詢的第一個步驟是對[查詢語言](../concepts/query-language.md)進行基本的認識。 如果您還不熟悉 [Azure 資料總管](../../../data-explorer/data-explorer-overview.md)，建議檢閱基本概念，以了解如何撰寫所需資源的要求。

我們將逐步解說下列入門查詢︰

> [!div class="checklist"]
> - [計算的 Azure 資源計數](#count-resources)
> - [列出依名稱排序的資源](#list-resources)
> - [依名稱遞減順序顯示所有虛擬機器](#show-vms)
> - [依名稱顯示前五個虛擬機器及其作業系統類型](#show-sorted)
> - [依作業系統類型計算的虛擬機器計數](#count-os)
> - [顯示包含儲存體的資源](#show-storage)
> - [列出所有公用 IP 位址](#list-publicip)
> - [計算具有按訂用帳戶設定之 IP 位址的資源計數](#count-resources-by-ip)
> - [列出具有特定標籤值的資源](#list-tag)
> - [列出具有特定標籤值的所有儲存體帳戶](#list-specific-tag)

如果您沒有 Azure 訂用帳戶，請在開始前建立 [免費帳戶](https://azure.microsoft.com/free)。

## <a name="language-support"></a>語言支援

Azure CLI (透過擴充功能) 與 Azure PowerShell (透過模組) 支援 Azure Resource Graph。 在執行下列任何查詢之前，請檢查您的環境已準備就緒。 請參閱 [Azure CLI](../first-query-azurecli.md#add-the-resource-graph-extension) 與 [Azure PowerShell](../first-query-powershell.md#add-the-resource-graph-module)，以了解安裝及驗證所選殼層環境的步驟。

## <a name="count-resources">計算的 Azure 資源計數</a>

此查詢會傳回您具有存取權之訂用帳戶中存在的 Azure 資源的數目。 這也是很好的查詢，用於驗證您選擇的殼層是否已安裝適當的 Azure Resource Graph 元件，並處於正常運作狀態。

```Query
summarize count()
```

```azurecli-interactive
az graph query -q "summarize count()"
```

```powershell
Search-AzureRmGraph -Query "summarize count()"
```

## <a name="list-resources"></a>列出依名稱排序的資源

在不限制任何類型的資源或特定比對屬性的情況下，此查詢僅傳回 Azure 資源的**名稱**、**類型**，以及**位置**，但會使用 `order by` 以依照**名稱**屬性，按遞增 (`asc`) 順序排序。

```Query
project name, type, location
| order by name asc
```

```azurecli-interactive
az graph query -q "project name, type, location | order by name asc"
```

```powershell
Search-AzureRmGraph -Query "project name, type, location | order by name asc"
```

## <a name="show-vms"></a>依名稱遞減順序顯示所有虛擬機器

如果我們只需要虛擬機器的清單 (類型為 `Microsoft.Compute/virtualMachines`)，而非取得所有 Azure 資源，我們可以在結果中比對屬性**類型**。
與上述查詢類似，`desc` 會將 `order by` 變更為遞減。 類型比對中的 `=~` 會告知 Resource Graph 不區分大小寫。

```Query
project name, location, type
| where type =~ 'Microsoft.Compute/virtualMachines'
| order by name desc
```

```azurecli-interactive
az graph query -q "project name, location, type| where type =~ 'Microsoft.Compute/virtualMachines' | order by name desc"
```

```powershell
Search-AzureRmGraph -Query "project name, location, type| where type =~ 'Microsoft.Compute/virtualMachines' | order by name desc"
```

## <a name="show-sorted"></a>依名稱顯示前五個虛擬機器及其作業系統類型

此查詢會使用 `limit`，僅擷取按名稱排序的五個相符記錄。 Azure 資源類型為 `Microsoft.Compute/virtualMachines`。 `project` 會告訴 Azure Resource Graph 要包含哪些屬性。

```Query
where type =~ 'Microsoft.Compute/virtualMachines'
| project name, properties.storageProfile.osDisk.osType
| top 5 by name desc
```

```azurecli-interactive
az graph query -q "where type =~ 'Microsoft.Compute/virtualMachines' | project name, properties.storageProfile.osDisk.osType | top 5 by name desc"
```

```powershell
Search-AzureRmGraph -Query "where type =~ 'Microsoft.Compute/virtualMachines' | project name, properties.storageProfile.osDisk.osType | top 5 by name desc"
```

## <a name="count-os"></a>依作業系統類型計算的虛擬機器計數

在上一個查詢的基礎上，我們仍會依 `Microsoft.Compute/virtualMachines` 類型的 Azure 資源限制，但不再限制傳回的記錄數目。
相反地，我們使用 `summarize` 和 `count()` 來定義如何依屬性分組和彙總值，在此範例中為 `properties.storageProfile.osDisk.osType`。 如需此字串在完整物件中顯示方式的範例，請參閱[探索資源 - 虛擬機器探索](../concepts/explore-resources.md#virtual-machine-discovery)。

```Query
where type =~ 'Microsoft.Compute/virtualMachines'
| summarize count() by tostring(properties.storageProfile.osDisk.osType)
```

```azurecli-interactive
az graph query -q "where type =~ 'Microsoft.Compute/virtualMachines' | summarize count() by tostring(properties.storageProfile.osDisk.osType)"
```

```powershell
Search-AzureRmGraph -Query "where type =~ 'Microsoft.Compute/virtualMachines' | summarize count() by tostring(properties.storageProfile.osDisk.osType)"
```

撰寫相同查詢的另一種方式是 `extend` 屬性，並為其提供臨時的名稱，以便用於查詢，在此情況下為 **os**。 **os** 接著會由 `summarize` 與 `count()` 使用，如同上述範例所述。

```Query
where type =~ 'Microsoft.Compute/virtualMachines'
| extend os = properties.storageProfile.osDisk.osType
| summarize count() by tostring(os)
```

```azurecli-interactive
az graph query -q "where type =~ 'Microsoft.Compute/virtualMachines' | extend os = properties.storageProfile.osDisk.osType | summarize count() by tostring(os)"
```

```powershell
Search-AzureRmGraph -Query "where type =~ 'Microsoft.Compute/virtualMachines' | extend os = properties.storageProfile.osDisk.osType | summarize count() by tostring(os)"
```

> [!NOTE]
> 請注意，雖然 `=~` 允許不區分大小寫的比對，但在查詢中使用屬性 (例如 **properties.storageProfile.osDisk.osType**) 要求大小寫正確無誤。 如果屬性的大小寫錯誤，它仍然可以傳回值，但群組或摘要會不正確。

## <a name="show-storage"></a>顯示包含儲存體的資源

此範例查詢並非明確定義要比對的類型，而是尋找 `contains` 單字 **storage** 的任何 Azure 資源。

```Query
where type contains 'storage' | distinct type
```

```azurecli-interactive
az graph query -q "where type contains 'storage' | distinct type"
```

```powershell
Search-AzureRmGraph -Query "where type contains 'storage' | distinct type"
```

## <a name="list-publicip"></a>列出所有公用 IP 位址

與上述查詢類似，尋找單字 **publicIPAddresses** 之類型的所有項目。 此查詢擴展該模式，以排除 **properties.ipAddress** 為 Null 的結果，僅傳回 **properties.ipAddress**，並將結果 `limit` 在前 100 個。 您可能需要逸出的引號，根據您所選擇的殼層而定。

```Query
where type contains 'publicIPAddresses' and properties.ipAddress != ''
| project properties.ipAddress
| limit 100
```

```azurecli-interactive
az graph query -q "where type contains 'publicIPAddresses' and properties.ipAddress != '' | project properties.ipAddress | limit 100"
```

```powershell
Search-AzureRmGraph -Query "where type contains 'publicIPAddresses' and properties.ipAddress != '' | project properties.ipAddress | limit 100"
```

## <a name="count-resources-by-ip"></a>計算具有按訂用帳戶設定之 IP 位址的資源計數

使用上一個範例查詢，並新增 `summarize` 和 `count()`，我們可以取得具有已設定 IP 位址之資源的訂用帳戶清單。

```Query
where type contains 'publicIPAddresses' and properties.ipAddress != ''
| summarize count () by subscriptionId
```

```azurecli-interactive
az graph query -q "where type contains 'publicIPAddresses' and properties.ipAddress != '' | summarize count () by subscriptionId"
```

```powershell
Search-AzureRmGraph -Query "where type contains 'publicIPAddresses' and properties.ipAddress != '' | summarize count () by subscriptionId"
```

## <a name="list-tag"></a>列出具有特定標籤值的資源

我們可以根據 Azure 資源類型以外的屬性限制結果，例如標籤。 在此範例中，我們會篩選標籤名稱為 **Environment** 的 Azure 資源進行篩選，其值為 **Internal**。

```Query
where tags.environment=~'internal'
| project name
```

```azurecli-interactive
az graph query -q "where tags.environment=~'internal' | project name"
```

```powershell
Search-AzureRmGraph -Query "where tags.environment=~'internal' | project name"
```

如果需要同時提供資源具有的標籤及其值，則可以透過將屬性**標籤**加入 `project`。

```Query
where tags.environment=~'internal'
| project name, tags
```

```azurecli-interactive
az graph query -q "where tags.environment=~'internal' | project name, tags"
```

```powershell
Search-AzureRmGraph -Query "where tags.environment=~'internal' | project name, tags"
```

## <a name="list-specific-tag"></a>列出具有特定標籤值的所有儲存體帳戶

將先前範例的篩選功能結合使用依**類型**屬性的 Azure 資源類型篩選，我們可以使用特定標記名稱和值，限制 Azure 資源之特定類型的搜尋。

```Query
where type =~ 'Microsoft.Storage/storageAccounts'
| where tags['tag with a space']=='Custom value'
```

```azurecli-interactive
az graph query -q "where type =~ 'Microsoft.Storage/storageAccounts' | where tags['tag with a space']=='Custom value'"
```

```powershell
Search-AzureRmGraph -Query "where type =~ 'Microsoft.Storage/storageAccounts' | where tags['tag with a space']=='Custom value'"
```

> [!NOTE]
> 此範例會使用 `==` 進行比對，而非使用 `=~` 條件。 `==` 是區分大小寫的比對。

## <a name="next-steps"></a>後續步驟

- 深入了解[查詢語言](../concepts/query-language.md)
- 了解[探索資源](../concepts/explore-resources.md)
- 請參閱[進階查詢](advanced.md)的範例