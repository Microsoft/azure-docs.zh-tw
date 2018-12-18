---
title: Azure CLI 指令碼 - 調整適用於 MySQL 的 Azure 資料庫伺服器
description: 此範例 CLI 指令碼會在查詢計量之後，將「適用於 MySQL 的 Azure 資料庫」伺服器調整為不同的效能等級。
services: mysql
author: ajlam
ms.author: andrela
manager: kfile
editor: jasonwhowell
ms.service: mysql
ms.devlang: azure-cli
ms.topic: sample
ms.custom: mvc
ms.date: 04/05/2018
ms.openlocfilehash: add0c01079bf2a5536bc7ec3425de012daab4306
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46957273"
---
# <a name="monitor-and-scale-an-azure-database-for-mysql-server-using-azure-cli"></a>使用 Azure CLI 監視和調整適用於 MySQL 的 Azure 資料庫伺服器
此範例 CLI 指令碼會在查詢計量之後，將單一「適用於 MySQL 的 Azure 資料庫」伺服器調整為不同的效能等級。

[!INCLUDE [cloud-shell-try-it](../../../includes/cloud-shell-try-it.md)]

如果您選擇在本執行 CLI，本文需要 Azure CLI 2.0 版或更新版本。 執行 `az --version` 以檢查版本。 請參閱[安裝 Azure CLI]( /cli/azure/install-azure-cli) 來安裝或升級 Azure CLI 版本。 

## <a name="sample-script"></a>範例指令碼
在此範例指令碼中，請編輯醒目提示的命令列，將系統管理員使用者名稱和密碼更新為您自己的使用者名稱和密碼。 請以您自己的訂用帳戶識別碼取代 `az monitor` 命令中使用的訂用帳戶識別碼。[!code-azurecli-interactive[main](../../../cli_scripts/mysql/scale-mysql-server/scale-mysql-server.sh?highlight=18-19 "Create and scale Azure Database for MySQL.")]

## <a name="clean-up-deployment"></a>清除部署
執行指令碼之後，請使用下列命令來移除資源群組及其相關聯的所有資源。 
[!code-azurecli-interactive[main](../../../cli_scripts/mysql/scale-mysql-server/delete-mysql.sh  "Delete the resource group.")]

## <a name="script-explanation"></a>指令碼說明
此指令碼會使用下表中簡述的命令：

| **命令** | **注意事項** |
|---|---|
| [az group create](/cli/azure/group#az-group-create) | 建立用來存放所有資源的資源群組。 |
| [az mysql server create](/cli/azure/mysql/server#az-mysql-server-create) | 建立主控資料庫的 MySQL 伺服器。 |
| [az monitor metrics list](/cli/azure/monitor/metrics#az-monitor-metrics-list) | 列出資源的度量值。 |
| [az group delete](/cli/azure/group#az-group-delete) | 刪除資源群組，包括所有的巢狀資源。 |

## <a name="next-steps"></a>後續步驟
- 了解 Azure CLI 的詳細資訊：[Azure CLI 文件](/cli/azure)。
- 嘗試其他指令碼：[「適用於 MySQL 的 Azure 資料庫」的 Azure CLI 範例](../sample-scripts-azure-cli.md)
- 如需有關調整的詳細資訊，請參閱[服務層](../concepts-service-tiers.md)及[計算單位和儲存體單位](../concepts-compute-unit-and-storage.md)。
