---
title: Azure CLI 指令碼 - 將適用於 MySQL 的 Azure 資料庫伺服器還原至先前的時間點
description: 此範例 CLI 指令碼會將適用於 MySQL 的 Azure 資料庫伺服器還原至先前的時間點。
services: mysql
author: ajlam
ms.author: andrela
manager: kfile
editor: jasonwhowell
ms.service: mysql
ms.devlang: azure-cli
ms.topic: sample
ms.custom: mvc
ms.date: 02/28/2018
ms.openlocfilehash: 83ff7b0a5f4abd6b8a1cddb80d5cf130df4d19f0
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46986741"
---
# <a name="restore-an-azure-database-for-mysql-server-using-azure-cli"></a>使用 Azure CLI 還原「適用於 MySQL 的 Azure 資料庫」伺服器
此範例 CLI 指令碼會將單一「適用於 MySQL 的 Azure 資料庫」伺服器還原至先前的時間點。

[!INCLUDE [cloud-shell-try-it](../../../includes/cloud-shell-try-it.md)]

如果您選擇在本機安裝並使用 CLI，此範例會要求您執行 Azure CLI 2.0 版或更新版本。 執行 `az --version` 以尋找版本。 如果您需要安裝或升級，請參閱[安裝 Azure CLI]( /cli/azure/install-azure-cli)。 

## <a name="sample-script"></a>範例指令碼
在此範例指令碼中，請變更醒目提示的命令列以自訂系統管理員使用者名稱和密碼。 請以您自己的訂用帳戶識別碼取代用於 az monitor 命令中的訂用帳戶識別碼。
[!code-azurecli-interactive[main](../../../cli_scripts/mysql/backup-restore-pitr/backup-restore.sh?highlight=18-19 "Restore Azure Database for MySQL.")]

## <a name="clean-up-deployment"></a>清除部署
在執行過指令碼範例之後，您可以使用下列命令來移除資源群組和所有與其相關聯的資源。
[!code-azurecli-interactive[main](../../../cli_scripts/mysql/backup-restore-pitr/delete-mysql.sh  "Delete the resource group.")]

## <a name="script-explanation"></a>指令碼說明
此指令碼會使用下列命令。 下表中的每個命令都會連結至命令特定的文件。

| **命令** | **注意事項** |
|---|---|
| [az group create](/cli/azure/group#create) | 建立用來存放所有資源的資源群組。 |
| [az mysql server create](/cli/azure/mysql/server#create) | 建立主控資料庫的 MySQL 伺服器。 |
| [az mysql server restore](/cli/azure/mysql/server#restore) | 從備份還原伺服器。 |
| [az group delete](/cli/azure/group#delete) | 刪除資源群組，包括所有的巢狀資源。 |

## <a name="next-steps"></a>後續步驟
- 了解 Azure CLI 的詳細資訊：[Azure CLI 文件](/cli/azure)。
- 嘗試其他指令碼：[「適用於 MySQL 的 Azure 資料庫」的 Azure CLI 範例](../sample-scripts-azure-cli.md)
