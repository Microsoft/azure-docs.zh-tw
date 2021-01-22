---
title: CLI 指令碼 - 調整伺服器 - 適用於 MariaDB 的 Azure 資料庫
description: 此範例 CLI 指令碼會在查詢計量之後，將「適用於 MariaDB 的 Azure 資料庫」伺服器調整為不同的效能等級。
author: savjani
ms.author: pariks
ms.service: jroth
ms.devlang: azurecli
ms.topic: sample
ms.custom: mvc, devx-track-azurecli
ms.date: 12/02/2019
ms.openlocfilehash: ac59ee30b75f4d6cab7d773b3561fcea542cb778
ms.sourcegitcommit: 52e3d220565c4059176742fcacc17e857c9cdd02
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 01/21/2021
ms.locfileid: "98664546"
---
# <a name="monitor-and-scale-an-azure-database-for-mariadb-server-using-azure-cli"></a>使用 Azure CLI 監視和調整適用於 MariaDB 的 Azure 資料庫伺服器
此範例 CLI 指令碼會在查詢計量之後，對單一「適用於 MariaDB 的 Azure 資料庫」伺服器調整計算和儲存。 計算可以擴大或縮小。 儲存體只能擴大。

[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]

[!INCLUDE [azure-cli-prepare-your-environment.md](../../../includes/azure-cli-prepare-your-environment.md)]

- 本文需要 2.0 版或更新版本的 Azure CLI。 如果您是使用 Azure Cloud Shell，就已安裝最新版本。 

## <a name="sample-script"></a>範例指令碼
使用您的訂用帳戶識別碼來更新指令碼。
[!code-azurecli-interactive[main](../../../cli_scripts/mariadb/scale-mariadb-server/scale-mariadb-server.sh "Create and scale Azure Database for MariaDB.")]

## <a name="clean-up-deployment"></a>清除部署
執行指令碼之後，請使用下列命令來移除資源群組及其相關聯的所有資源。 
[!code-azurecli-interactive[main](../../../cli_scripts/mariadb/scale-mariadb-server/delete-mariadb.sh  "Delete the resource group.")]

## <a name="script-explanation"></a>指令碼說明
此指令碼會使用下表中簡述的命令：

| **命令** | **注意事項** |
|---|---|
| [az group create](/cli/azure/group#az-group-create) | 建立用來存放所有資源的資源群組。 |
| [az mariadb server create](/cli/azure/mariadb/server#az-mariadb-server-create) | 建立託管資料庫的 MariaDB 伺服器。 |
| [az mariadb server update](/cli/azure/mariadb/server#az-mariadb-server-update) | 更新 MariaDB 伺服器的屬性。 |
| [az monitor metrics list](/cli/azure/monitor/metrics#az-monitor-metrics-list) | 列出資源的度量值。 |
| [az group delete](/cli/azure/group#az-group-delete) | 刪除資源群組，包括所有的巢狀資源。 |

## <a name="next-steps"></a>後續步驟
- 深入了解[適用於 MariaDB 的 Azure 資料庫的計算和儲存](../concepts-pricing-tiers.md)
- 嘗試額外的指令碼：[「適用於 MariaDB 的 Azure 資料庫」的 Azure CLI 範例](../sample-scripts-azure-cli.md)
- 深入了解 [Azure CLI](/cli/azure)
