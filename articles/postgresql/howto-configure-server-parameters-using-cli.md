---
title: 在適用於 PostgreSQL 的 Azure 資料庫中設定服務參數
description: 本文說明如何使用 Azure CLI 命令列，在適用於 PostgreSQL 的 Azure 資料庫中設定服務參數。
services: postgresql
author: rachel-msft
ms.author: raagyema
manager: kfile
editor: jasonwhowell
ms.service: postgresql
ms.devlang: azure-cli
ms.topic: article
ms.date: 02/28/2018
ms.openlocfilehash: 5520c08d2bf5dba85ece1de0bca7329286625911
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46968045"
---
# <a name="customize-server-configuration-parameters-using-azure-cli"></a>使用 Azure CLI 自訂伺服器設定參數
您可以使用命令列介面 (Azure CLI)，來列出、顯示和更新 Azure PostgreSQL 伺服器的設定參數。 有一部分的引擎設定會在伺服器層級公開而且可供修改。 

## <a name="prerequisites"></a>必要條件
若要逐步執行本作法指南，您需要︰
- 遵循[建立適用於 PostgreSQL 的 Azure 資料庫](quickstart-create-server-database-azure-cli.md)，來建立適用於 PostgreSQL 的 Azure 資料庫伺服器和資料庫。
- 在電腦上安裝 [Azure CLI](/cli/azure/install-azure-cli) 命令列介面，或透過您的瀏覽器在 Azure 入口網站中使用 [Azure Cloud Shell](../cloud-shell/overview.md)。

## <a name="list-server-configuration-parameters-for-azure-database-for-postgresql-server"></a>列出適用於 PostgreSQL 伺服器之 Azure 資料庫的伺服器組態參數
若要列出伺服器中所有可修改的參數及其值，請執行 [az postgres server configuration list](/cli/azure/postgres/server/configuration#az_postgres_server_configuration_list) 命令。

您可以針對資源群組 **myresourcegroup** 下的伺服器 **mydemoserver.postgres.database.azure.com**，列出伺服器組態參數。
```azurecli-interactive
az postgres server configuration list --resource-group myresourcegroup --server mydemoserver
```
## <a name="show-server-configuration-parameter-details"></a>顯示伺服器設定參數的詳細資料
若要顯示有關伺服器特定組態參數的詳細資訊，請執行 [az postgres server configuration show](/cli/azure/postgres/server/configuration#az_postgres_server_configuration_show) 命令。

此範例會針對資源群組 **myresourcegroup** 下的伺服器 **mydemoserver.postgres.database.azure.com**，顯示 **log\_min\_messages** 伺服器組態參數的詳細資料。
```azurecli-interactive
az postgres server configuration show --name log_min_messages --resource-group myresourcegroup --server mydemoserver
```
## <a name="modify-server-configuration-parameter-value"></a>修改伺服器設定參數值
您也可以修改特定伺服器設定參數的值，以更新 PostgreSQL 伺服器引擎的基礎設定值。 若要更新設定，請使用 [az postgres server configuration set](/cli/azure/postgres/server/configuration#az_postgres_server_configuration_set) 命令。 

若要針對資源群組 **myresourcegroup** 下的伺服器 **mydemoserver.postgres.database.azure.com** 更新 **log\_min\_messages** 伺服器組態參數。
```azurecli-interactive
az postgres server configuration set --name log_min_messages --resource-group myresourcegroup --server mydemoserver --value INFO
```
如果您想要重設設定參數的值，只需選擇保留選擇性的 `--value` 參數即可，而服務會套用預設值。 在上述範例中，看起來應該像這樣：
```azurecli-interactive
az postgres server configuration set --name log_min_messages --resource-group myresourcegroup --server mydemoserver
```
這會將 **log\_min\_messages** 設定重設為預設值 **WARNING**。 如需伺服器設定和允許值的詳細資訊，請參閱有關[伺服器設定](https://www.postgresql.org/docs/9.6/static/runtime-config.html) \(英文\) 的 PostgreSQL 文件。

## <a name="next-steps"></a>後續步驟
- 若要設定及存取伺服器記錄，請參閱[適用於 PostgreSQL 的 Azure 資料庫中的伺服器記錄](concepts-server-logs.md)
