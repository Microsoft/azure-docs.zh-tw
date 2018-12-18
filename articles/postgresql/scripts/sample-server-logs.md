---
title: Azure CLI 指令碼 - 下載「適用於 PostgreSQL 的 Azure 資料庫」中的伺服器記錄
description: 此範例 Azure CLI 指令碼會示範如何啟用並下載「適用於 PostgreSQL 的 Azure 資料庫」伺服器的伺服器記錄。
services: postgresql
author: rachel-msft
ms.author: raagyema
manager: kfile
editor: jasonwhowell
ms.service: postgresql
ms.devlang: azure-cli
ms.topic: sample
ms.custom: mvc
ms.date: 02/28/2018
ms.openlocfilehash: 21c2ed0e58bddb58633968b426e06bc95c44ee99
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46948208"
---
# <a name="enable-and-download-server-slow-query-logs-of-an-azure-database-for-postgresql-server-using-azure-cli"></a>使用 Azure CLI 啟用並下載「適用於 PostgreSQL 的 Azure 資料庫」伺服器的伺服器慢速查詢記錄
此範例 CLI 指令碼會啟用並下載單一「適用於 PostgreSQL 的 Azure 資料庫」伺服器的慢速查詢記錄。

[!INCLUDE [cloud-shell-try-it](../../../includes/cloud-shell-try-it.md)]

如果您選擇在本執行 CLI，本文需要 Azure CLI 2.0 版或更新版本。 執行 `az --version` 以檢查版本。 請參閱[安裝 Azure CLI]( /cli/azure/install-azure-cli) 來安裝或升級 Azure CLI 版本。

## <a name="sample-script"></a>範例指令碼
在此範例指令碼中，請編輯醒目提示的命令列，將系統管理員使用者名稱和密碼更新為您自己的使用者名稱和密碼。 請以您自己的伺服器記錄檔名稱取代 `az monitor` 命令中的 <log_file_name>。
[!code-azurecli-interactive[main](../../../cli_scripts/postgresql/server-logs/server-logs.sh?highlight=18-19 "Manipulate with server logs.")]

## <a name="clean-up-deployment"></a>清除部署
執行指令碼之後，請使用下列命令來移除資源群組及其相關聯的所有資源。 
[!code-azurecli-interactive[main](../../../cli_scripts/postgresql/server-logs/delete-postgresql.sh  "Delete the resource group.")]

## <a name="script-explanation"></a>指令碼說明
此指令碼會使用下表中簡述的命令：

| **命令** | **注意事項** |
|---|---|
| [az group create](/cli/azure/group#az_group_create) | 建立用來存放所有資源的資源群組。 |
| [az postgres server create](/cli/azure/postgres/server#az_msql_server_create) | 建立主控資料庫的 PostgreSQL 伺服器。 |
| [az postgres server configuration list](/cli/azure/postgres/server/configuration#az_postgres_server_configuration_list) | 列出伺服器的設定值。 |
| [az postgres server configuration set](/cli/azure/postgres/server/configuration#az_postgres_server_configuration_set) | 更新伺服器的設定。 |
| [az postgres server-logs list](/cli/azure/postgres/server-logs#az_postgres_server_logs_list) | 列出伺服器的記錄檔。 |
| [az postgres server-logs download](/cli/azure/postgres/server-logs#az_postgres_server_logs_download) | 下載記錄檔。 |
| [az group delete](/cli/azure/group#az_group_delete) | 刪除資源群組，包括所有的巢狀資源。 |

## <a name="next-steps"></a>後續步驟
- 了解 Azure CLI 的詳細資訊：[Azure CLI 文件](/cli/azure)。
- 嘗試其他指令碼：[「適用於 PostgreSQL 的 Azure 資料庫」的 Azure CLI 範例](../sample-scripts-azure-cli.md)
- [在 Azure 入口網站中設定和存取伺服器記錄](../howto-configure-server-logs-in-portal.md)
