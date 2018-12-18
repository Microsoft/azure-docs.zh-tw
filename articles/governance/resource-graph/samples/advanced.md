---
title: 進階 Azure Resource Graph 查詢
description: 使用 Azure Resource Graph 執行一些進階查詢。
services: resource-graph
author: DCtheGeek
ms.author: dacoulte
ms.date: 09/18/2018
ms.topic: quickstart
ms.service: resource-graph
manager: carmonm
ms.custom: mvc
ms.openlocfilehash: a9281bc333b3edd12501a6813f12b9e7ad1bf59f
ms.sourcegitcommit: ad08b2db50d63c8f550575d2e7bb9a0852efb12f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/26/2018
ms.locfileid: "47219262"
---
# <a name="advanced-resource-graph-queries"></a>進階 Resource Graph 查詢

透過 Azure Resource Graph 了解查詢的第一個步驟是對[查詢語言](../concepts/query-language.md)進行基本的認識。 如果您還不熟悉 [Azure 資料總管](../../../data-explorer/data-explorer-overview.md)，建議檢閱基本概念，以了解如何撰寫所需資源的要求。

我們將逐步解說下列進階查詢︰

> [!div class="checklist"]
> - [取得 VMSS 容量和大小](#vmss-capacity)
> - [列出所有標記名稱](#list-all-tags)
> - [依 RegEx 比對虛擬機器](#vm-regex)

如果您沒有 Azure 訂用帳戶，請在開始前建立 [免費帳戶](https://azure.microsoft.com/free)。

## <a name="language-support"></a>語言支援

Azure CLI (透過擴充功能) 與 Azure PowerShell (透過模組) 支援 Azure Resource Graph。 在執行下列任何查詢之前，請檢查您的環境已準備就緒。 請參閱 [Azure CLI](../first-query-azurecli.md#add-the-resource-graph-extension) 與 [Azure PowerShell](../first-query-powershell.md#add-the-resource-graph-module)，以了解安裝及驗證所選殼層環境的步驟。

## <a name="vmss-capacity"></a>取得 VMSS 容量和大小

此查詢會尋找虛擬機器擴展集 (VMSS) 資源，並取得各種詳細資料 (包括虛擬機器大小與擴展集的容量)。 此資訊會使用 `toint()` 函式，將容量轉換為數字以便進行排序。 這也會將傳回到自訂具名屬性的值重新命名。

```Query
where type=~ 'microsoft.compute/virtualmachinescalesets'
| where name contains 'contoso'
| project subscriptionId, name, location, resourceGroup, Capacity = toint(sku.capacity), Tier = sku.name
| order by Capacity desc
```

```azurecli-interactive
az graph query -q "where type=~ 'microsoft.compute/virtualmachinescalesets' | where name contains 'contoso' | project subscriptionId, name, location, resourceGroup, Capacity = toint(sku.capacity), Tier = sku.name | order by Capacity desc"
```

```powershell
Search-AzureRmGraph -Query "where type=~ 'microsoft.compute/virtualmachinescalesets' | where name contains 'contoso' | project subscriptionId, name, location, resourceGroup, Capacity = toint(sku.capacity), Tier = sku.name | order by Capacity desc"
```

## <a name="list-all-tags"></a>列出所有標記名稱

此查詢的開頭為標記，而且會建置 JSON 物件，列出所有的唯一標記名稱及其對應的類型。

```Query
project tags
| summarize buildschema(tags)
```

```azurecli-interactive
az graph query -q "project tags | summarize buildschema(tags)"
```

```powershell
Search-AzureRmGraph -Query "project tags | summarize buildschema(tags)"
```

## <a name="vm-regex"></a>依 RegEx 比對虛擬機器

此查詢會尋找符合[規則運算式](/dotnet/standard/base-types/regular-expression-language-quick-reference) (亦稱為 _RegEx_) 的虛擬機器。
**matches regex @** 可讓我們定義要比對的 RegEx，也就是 **^Contoso(.*)[0-9]+$**。 該 RegEx 定義說明如下：

- `^`：比對必須從字串的開頭開始。
- `Contoso`：我們要比對的核心字串 (區分大小寫)。
- `(.*)`：子運算式比對：
  - `.`：比對任何單一字元 (除了新行符號)。
  - `*`：比對前一個元素零或多次。
- `[0-9]`：比對從 0 到 9 的字元群組。
- `+`：比對前一個元素一或多次。
- `$`：前一個元素的比對必須發生在字串結尾。

在依名稱比對之後，查詢會投射名稱，並依名稱遞增排序。

```Query
where type =~ 'microsoft.compute/virtualmachines' and name matches regex @'^Contoso(.*)[0-9]+$'
| project name
| order by name asc
```

```azurecli-interactive
az graph query -q "where type =~ 'microsoft.compute/virtualmachines' and name matches regex @'^Contoso(.*)[0-9]+$' | project name | order by name asc"
```

```powershell
Search-AzureRmGraph -Query "where type =~ 'microsoft.compute/virtualmachines' and name matches regex @'^Contoso(.*)[0-9]+$' | project name | order by name asc"
```

## <a name="next-steps"></a>後續步驟

- 請參閱[入門查詢](starter.md)的範例
- 深入了解[查詢語言](../concepts/query-language.md)
- 了解[探索資源](../concepts/explore-resources.md)