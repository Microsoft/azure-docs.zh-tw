---
title: 使用 Azure CLI 執行您的第一個 Resource Graph 查詢
description: 此文章將引導您完成為 Azure CLI 啟用 Resource Graph 延伸模組並執行第一個查詢的步驟。
services: resource-graph
author: DCtheGeek
ms.author: dacoulte
ms.date: 09/18/2018
ms.topic: quickstart
ms.service: resource-graph
ms.custom: mvc
manager: carmonm
ms.openlocfilehash: df01a57cda063e301efbc4d99f3bac8dbb4b2e66
ms.sourcegitcommit: 715813af8cde40407bd3332dd922a918de46a91a
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "47054161"
---
# <a name="run-your-first-resource-graph-query-using-azure-cli"></a>使用 Azure CLI 執行您的第一個 Resource Graph 查詢

使用 Azure Resource Graph 的第一個步驟，是確定已安裝適用於 [Azure CLI](/cli/azure/) 的延伸模組。 此快速入門逐步引導您完成將延伸模組新增至您的 Azure CLI 安裝的程序。 您可以透過在本機安裝的 Azure CLI，或是透過 [Azure Cloud Shell](https://shell.azure.com) 使用延伸模組。

在此程序結束時，延伸模組將新增到您選擇的 Azure CLI 安裝中，並執行您的第一個 Resource Graph 查詢。

如果您沒有 Azure 訂用帳戶，請在開始前建立[免費帳戶](https://azure.microsoft.com/free/)。

## <a name="add-the-resource-graph-extension"></a>新增 Resource Graph 延伸模組

若要讓 Azure CLI 可查詢 Azure Resource Graph，您必須新增延伸模組。 此延伸模組適用於可使用 Azure CLI 的任何地方，包括 [Windows 10 的 Bash](/windows/wsl/install-win10)、[Cloud Shell](https://shell.azure.com) (獨立與內部入口網站)、[Azure CLI Docker 映像](https://hub.docker.com/r/microsoft/azure-cli/)，或在本機安裝。

1. 確認已安裝最新的 Azure CLI (至少 **2.0.45**)。 如果未安裝，請依照[這些步驟](/cli/azure/install-azure-cli-windows?view=azure-cli-latest)操作。

1. 在您選擇的 Azure CLI 環境中，使用下列命令匯入：

   ```azurecli-interactive
   # Add the Resource Graph extension to the Azure CLI environment
   az extension add --name resource-graph
   ```

1. 驗證延伸模組已安裝，且為預期的版本 (至少為 **0.1.7**)：

   ```azurecli-interactive
   # Check the extension list (note that you may have other extensions installed)
   az extension list

   # Run help for graph query options
   az graph query -h
   ```

## <a name="run-your-first-resource-graph-query"></a>執行第一個 Resource Graph 查詢

現在，Azure CLI 延伸模組已新增至您選擇的環境，您可以試試看簡單的 Resource Graph 查詢。 查詢會傳回前五個 Azure 資源支每個資源的**名稱**與**資源類型**。

1. 使用 `graph` 延伸模組與 `query` 命令，執行您的第一個 Azure Resource Graph 查詢：

   ```azurecli-interactive
   # Login first with az login if not using Cloud Shell

   # Run Azure Resource Graph query
   az graph query -q 'project name, type | limit 5'
   ```

   > [!NOTE]
   > 當此查詢範例未提供排序修飾詞，例如 `order by`，多次執行此查詢可能會為每個要求產生不同的資源集。

1. 將查詢更新為 `order by` **名稱**屬性：

   ```azurecli-interactive
   # Run Azure Resource Graph query with 'order by'
   az graph query -q 'project name, type | limit 5 | order by name asc'
   ```

  > [!NOTE]
  > 如同第一個查詢一樣，多次執行此查詢可能會為每個要求產生不同的資源集。 查詢命令的順序很重要。 在此範例中，`order by` 會出現在 `limit` 之後。 這會先限制查詢結果，然後將結果進行排序。

1. 將查詢更新為第一個 `order by`，**名稱**屬性，然後 `limit` 為前 5 個結果：

   ```azurecli-interactive
   # Run Azure Resource Graph query with `order by` first, then with `limit`
   az graph query -q 'project name, type | order by name asc | limit 5'
   ```

多次執行最終查詢時，假設您的環境中未變更任何內容，傳回的結果將會一致且符合預期 - 依照**名稱**屬性排序，但仍限制為前 5 個結果。

## <a name="cleanup"></a>清除

如果您想要從 Azure CLI 環境中移除 Resource Graph 延伸模組，則可以使用下列命令：

```azurecli-interactive
# Remove the Resource Graph extension from the Azure CLI environment
az extension remove -n resource-graph
```

> [!NOTE]
> 這不會刪除先前下載的延伸模組檔案。 它只會從正在執行的 Azure CLI 環境中移除。

## <a name="next-steps"></a>後續步驟

- 了解關於[查詢語言](./concepts/query-language.md)的詳細資訊
- 了解[探索資源](./concepts/explore-resources.md)
- 使用 [Azure PowerShell](first-query-powershell.md) 執行第一個查詢
- 請參閱[入門查詢](./samples/starter.md)的範例
- 請參閱[進階查詢](./samples/advanced.md)的範例
- 在 [UserVoice](https://feedback.azure.com/forums/915958-azure-governance) 上提供意見反應