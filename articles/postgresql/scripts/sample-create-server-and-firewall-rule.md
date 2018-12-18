---
title: Azure CLI 指令碼 - 建立適用於 PostgreSQL 的 Azure 資料庫
description: Azure CLI 指令碼範例 - 建立「適用於 PostgreSQL 的 Azure 資料庫」伺服器，並設定伺服器等級防火牆規則。
services: postgresql
author: rachel-msft
ms.author: raagyema
manager: kfile
editor: jasonwhowell
ms.service: postgresql
ms.custom: mvc
ms.devlang: azure-cli
ms.topic: sample
ms.date: 02/28/2018
ms.openlocfilehash: 58b8e4ee75a0dd676033f39afbd7a30efec80628
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46962245"
---
# <a name="create-an-azure-database-for-postgresql-server-and-configure-a-firewall-rule-using-the-azure-cli"></a>使用 Azure CLI 建立「適用於 PostgreSQL 的 Azure 資料庫」伺服器並設定防火牆規則
此範例 CLI 指令碼會建立「適用於 PostgreSQL 的 Azure 資料庫」伺服器，並設定伺服器等級防火牆規則。 成功執行指令碼後，即可從所有 Azure 服務和已設定的 IP 位址存取 PostgreSQL 伺服器。

[!INCLUDE [cloud-shell-try-it](../../../includes/cloud-shell-try-it.md)]

如果您選擇在本執行 CLI，本文需要 Azure CLI 2.0 版或更新版本。 執行 `az --version` 以檢查版本。 請參閱[安裝 Azure CLI]( /cli/azure/install-azure-cli) 來安裝或升級 Azure CLI 版本。

## <a name="sample-script"></a>範例指令碼
在此範例指令碼中，請編輯醒目提示的命令列，將系統管理員使用者名稱和密碼更新為您自己的使用者名稱和密碼。
[!code-azurecli-interactive[main](../../../cli_scripts/postgresql/create-postgresql-server-and-firewall-rule/create-postgresql-server-and-firewall-rule.sh?highlight=18-19 "Create an Azure Database for PostgreSQL, and server-level firewall rule.")]

## <a name="clean-up-deployment"></a>清除部署
執行指令碼之後，請使用下列命令來移除資源群組及其相關聯的所有資源。 
[!code-azurecli-interactive[main](../../../cli_scripts/postgresql/create-postgresql-server-and-firewall-rule/delete-postgresql.sh "Delete the resource group.")]

## <a name="script-explanation"></a>指令碼說明
此指令碼會使用下表中簡述的命令：

| **命令** | **注意事項** |
|---|---|
| [az group create](/cli/azure/group#az_group_create) | 建立用來存放所有資源的資源群組。 |
| [az postgres server create](/cli/azure/postgres/server#az_postgres_server_create) | 建立主控資料庫的 PostgreSQL 伺服器。 |
| [az postgres server firewall create](/cli/azure/postgres/server/firewall-rule#az_postgres_server_firewall_rule_create) | 建立防火牆規則以允許從輸入的 IP 位址範圍存取伺服器及其之下的資料庫。 |
| [az group delete](/cli/azure/group#az_group_delete) | 刪除資源群組，包括所有的巢狀資源。 |

## <a name="next-steps"></a>後續步驟
- 了解 Azure CLI 的詳細資訊：[Azure CLI 文件](/cli/azure)
- 嘗試其他指令碼：[「適用於 PostgreSQL 的 Azure 資料庫」的 Azure CLI 範例](../sample-scripts-azure-cli.md)
