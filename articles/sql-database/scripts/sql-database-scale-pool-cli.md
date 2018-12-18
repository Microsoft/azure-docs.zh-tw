---
title: CLI 範例 - 調整 SQL 彈性集區 - Azure SQL Database | Microsoft Docs
description: 在 Azure SQL Database 中調整 SQL 彈性集區的 Azure CLI 範例指令碼
services: sql-database
ms.service: sql-database
ms.subservice: performance
ms.custom: ''
ms.devlang: azurecli
ms.topic: sample
author: CarlRabeler
ms.author: carlrab
ms.reviewer: ''
manager: craigg
ms.date: 09/20/2018
ms.openlocfilehash: 2da2686d2d851856bae0841648dc27550c810112
ms.sourcegitcommit: 715813af8cde40407bd3332dd922a918de46a91a
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "47054436"
---
# <a name="use-cli-to-scale-a-sql-elastic-pool-in-azure-sql-database"></a>使用 CLI 在 Azure SQL Database 中調整 SQL 彈性集區

此 Azure CLI 指令碼範例會建立 SQL 彈性集區、移動集區資料庫，以及變更彈性集區的計算大小。 

[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]

[!INCLUDE [cloud-shell-try-it.md](../../../includes/cloud-shell-try-it.md)]

如果您選擇在本機安裝和使用 CLI，本主題會要求您執行 Azure CLI 2.0 版或更新版本。 執行 `az --version` 以尋找版本。 如果您需要安裝或升級，請參閱[安裝 Azure CLI]( /cli/azure/install-azure-cli)。 

## <a name="sample-script"></a>範例指令碼

[!code-azurecli-interactive[main](../../../cli_scripts/sql-database/scale-pool/scale-pool.sh "Move database between pools")]

## <a name="clean-up-deployment"></a>清除部署

在執行過指令碼範例之後，您可以使用下列命令來移除資源群組和所有與其相關聯的資源。

```azurecli-interactive
az group delete --name myResourceGroup
```

## <a name="script-explanation"></a>指令碼說明

此指令碼使用下列命令來建立資源群組、邏輯伺服器、SQL Database 和防火牆規則。 下表中的每個命令都會連結至命令特定的文件。

| 命令 | 注意 |
|---|---|
| [az group create](https://docs.microsoft.com/cli/azure/group#az-group-create) | 建立用來存放所有資源的資源群組。 |
| [az sql server create](https://docs.microsoft.com/cli/azure/sql/server#az-sql-server-create) | 建立主控 SQL Database 的邏輯伺服器。 |
| [az sql elastic-pools create](https://docs.microsoft.com/cli/azure/sql/elastic-pool#az-sql-elastic-pool-create) | 在邏輯伺服器內建立彈性資料庫集區。 |
| [az sql db create](https://docs.microsoft.com/cli/azure/sql/db#az-sql-db-create) | 在邏輯伺服器中建立 SQL Database。 |
| [az sql elastic-pools update](https://docs.microsoft.com/cli/azure/sql/elastic-pool#az-sql-elastic-pool-update) | 更新彈性資料庫集區，在此範例中是變更指派的 eDTU。 |
| [az group delete](https://docs.microsoft.com/cli/azure/vm/extension#az-vm-extension-set) | 刪除資源群組，包括所有的巢狀資源。 |

## <a name="next-steps"></a>後續步驟

如需 Azure CLI 的詳細資訊，請參閱 [Azure CLI 文件](https://docs.microsoft.com/cli/azure)。

其他的 SQL Database CLI 指令碼範例可於 [Azure SQL Database 文件](../sql-database-cli-samples.md)中找到。
