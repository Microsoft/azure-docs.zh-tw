---
title: 存取慢速查詢記錄-Azure CLI-適用於 MariaDB 的 Azure 資料庫
description: 本文說明如何使用 Azure CLI 命令列公用程式，存取適用於 MariaDB 的 Azure 資料庫中的慢速記錄。
author: savjani
ms.author: pariks
ms.service: jroth
ms.devlang: azurecli
ms.topic: how-to
ms.date: 4/13/2020
ms.custom: devx-track-azurecli
ms.openlocfilehash: 9c8f69f00ed4314fbe8b3fd1958b52c82ce55d99
ms.sourcegitcommit: 52e3d220565c4059176742fcacc17e857c9cdd02
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 01/21/2021
ms.locfileid: "98662384"
---
# <a name="configure-and-access-azure-database-for-maria-db-slow-query-logs-by-using-azure-cli"></a>使用 Azure CLI 來設定和存取適用于 Maria 資料庫慢速查詢記錄的 Azure 資料庫

您可以使用 Azure 命令列公用程式 Azure CLI 來下載適用於 MariaDB 的 Azure 資料庫慢速查詢記錄。

## <a name="prerequisites"></a>必要條件
若要逐步執行本作法指南，您需要︰
- [適用於 MariaDB 的 Azure 資料庫伺服器](quickstart-create-mariadb-server-database-using-azure-cli.md)
- [Azure CLI](/cli/azure/install-azure-cli) 或瀏覽器中的 Azure Cloud Shell

## <a name="configure-logging"></a>設定記錄
您可以採取下列步驟，設定伺服器以存取 MySQL 慢速查詢記錄檔：
1. 藉由將 **慢速 \_ 查詢 \_** 記錄參數設定為 on，開啟慢速查詢記錄。
2. 選取要使用 **記錄 \_ 輸出** 輸出記錄檔的位置。 若要將記錄傳送至本機儲存體和 Azure 監視器診斷記錄， **請選取 [** 檔案]。 若只要將記錄傳送至 Azure 監視器記錄檔，請選取 [**無**]
3. 調整其他參數，例如 **long\_query\_time** 和 **log\_slow\_admin\_statements**。

若要了解如何透過 Azure CLI 設定這些參數的值，請參閱[如何設定伺服器參數](howto-configure-server-parameters-cli.md)。

例如，下列 CLI 命令會開啟慢速查詢記錄檔、將長時間查詢的時間設定為 10 秒，並且會關閉慢速管理陳述式的記錄。 最後，它會列出設定選項供您檢閱。
```azurecli-interactive
az mariadb server configuration set --name slow_query_log --resource-group myresourcegroup --server mydemoserver --value ON
az mariadb server configuration set --name log_output --resource-group myresourcegroup --server mydemoserver --value FILE
az mariadb server configuration set --name long_query_time --resource-group myresourcegroup --server mydemoserver --value 10
az mariadb server configuration set --name log_slow_admin_statements --resource-group myresourcegroup --server mydemoserver --value OFF
az mariadb server configuration list --resource-group myresourcegroup --server mydemoserver
```

## <a name="list-logs-for-azure-database-for-mariadb-server"></a>列出適用於 MariaDB 的 Azure 資料庫伺服器之記錄
如果 **log_output** 設定為 "File"，您可以直接從伺服器的本機儲存體存取記錄。 若要列出伺服器的可用慢速查詢記錄檔，請執行 [az 適用于 mariadb server-logs list](/cli/azure/mariadb/server-logs#az-mariadb-server-logs-list) 命令。

您可以針對資源群組 **myresourcegroup** 下的伺服器 **mydemoserver.mariadb.database.azure.com** 列出記錄檔。 然後將記錄檔的清單導向名為 **log\_files\_list.txt** 的文字檔。
```azurecli-interactive
az mariadb server-logs list --resource-group myresourcegroup --server mydemoserver > log_files_list.txt
```
## <a name="download-logs-from-the-server"></a>從伺服器下載記錄
如果 **log_output** 設定為 "File"，您可以使用 [az 適用于 mariadb server-logs 下載](/cli/azure/mariadb/server-logs#az-mariadb-server-logs-download) 命令，從您的伺服器下載個別的記錄檔。

使用下列範例，針對資源群組 **myresourcegroup** 下的伺服器 **mydemoserver.mariadb.database.azure.com**，將特定的記錄檔下載至您的本機環境。
```azurecli-interactive
az mariadb server-logs download --name mysql-slow-mydemoserver-2018110800.log --resource-group myresourcegroup --server mydemoserver
```

## <a name="next-steps"></a>下一步
- 瞭解 [適用於 MariaDB 的 Azure 資料庫中的慢速查詢記錄](concepts-server-logs.md)。
