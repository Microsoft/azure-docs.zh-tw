---
title: 在適用於 MySQL 的 Azure 資料庫中設定服務參數
description: 本文說明如何使用 Azure CLI 命令列公用程式，在適用於 MySQL 的 Azure 資料庫中設定服務參數。
services: mysql
author: ajlam
ms.author: andrela
manager: kfile
editor: jasonwhowell
ms.service: mysql
ms.devlang: azure-cli
ms.topic: article
ms.date: 07/18/2018
ms.openlocfilehash: b45975bc019da2859da87a40e46970d849312e38
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46957052"
---
# <a name="customize-server-configuration-parameters-by-using-azure-cli"></a>使用 Azure CLI 自訂伺服器設定參數
您可以使用 Azure CLI (Azure 命令列公用程式)，來列出、顯示和更新適用於 MySQL 的 Azure 資料庫伺服器的設定參數。 有一部分的引擎設定會在伺服器層級公開而且可供修改。 

## <a name="prerequisites"></a>必要條件
若要逐步執行本作法指南，您需要︰
- [適用於 MySQL 的 Azure 資料庫伺服器](quickstart-create-mysql-server-database-using-azure-cli.md)
- [Azure CLI](/cli/azure/install-azure-cli) 命令列公用程式，或在瀏覽器中使用 Azure Cloud Shell。

## <a name="list-server-configuration-parameters-for-azure-database-for-mysql-server"></a>列出適用於 MySQL 的 Azure 資料庫伺服器的伺服器設定參數
若要列出伺服器中所有可修改的參數及其值，請執行 [az mysql server configuration list](/cli/azure/mysql/server/configuration#az-mysql-server-configuration-list) 命令。

您可以針對資源群組 **myresourcegroup** 下的伺服器 **mydemoserver.mysql.database.azure.com**，列出伺服器組態參數。
```azurecli-interactive
az mysql server configuration list --resource-group myresourcegroup --server mydemoserver
```
如需每個列出參數的定義，請參閱[伺服器系統變數](https://dev.mysql.com/doc/refman/5.7/en/server-system-variables.html) \(英文\) 中的 MySQL 參考小節。

## <a name="show-server-configuration-parameter-details"></a>顯示伺服器設定參數的詳細資料
若要顯示有關伺服器特定設定參數的詳細資訊，請執行 [az mysql server configuration show](/cli/azure/mysql/server/configuration#az-mysql-server-configuration-show) 命令。

此範例會針對資源群組 **myresourcegroup** 下的伺服器 **mydemoserver.mysql.database.azure.com**，顯示 **slow\_query\_log** 伺服器組態參數的詳細資料。
```azurecli-interactive
az mysql server configuration show --name slow_query_log --resource-group myresourcegroup --server mydemoserver
```
## <a name="modify-a-server-configuration-parameter-value"></a>修改伺服器設定參數值
您也可以修改特定伺服器設定參數的值，以更新適用於 MySQL 伺服器引擎的基礎設定值。 若要更新設定，請使用 [az mysql server configuration set](/cli/azure/mysql/server/configuration#az-mysql-server-configuration-set) 命令。 

若要針對資源群組 **myresourcegroup** 下的伺服器 **mydemoserver.mysql.database.azure.com** 更新 **slow\_query\_log** 伺服器組態參數。
```azurecli-interactive
az mysql server configuration set --name slow_query_log --resource-group myresourcegroup --server mydemoserver --value ON
```
如果您想要重設設定參數的值，請省略選擇性的 `--value` 參數，而服務會套用預設值。 針對上述範例，看起來應該像這樣：
```azurecli-interactive
az mysql server configuration set --name slow_query_log --resource-group myresourcegroup --server mydemoserver
```
此程式碼會將 **slow\_query\_log** 設定重設為預設值 **OFF**。 

## <a name="working-with-the-time-zone-parameter"></a>使用時區參數

### <a name="populating-the-time-zone-tables"></a>填入時區資料表

伺服器上的時區資料表，可以藉由從 MySQL 命令列或 MySQL Workbench 等工具呼叫 `az_load_timezone` 預存程序來填入。

> [!NOTE]
> 如果您是從 MySQL Workbench 執行 `az_load_timezone` 命令，您可能需要先執行 `SET SQL_SAFE_UPDATES=0;` 以關閉安全更新模式。

```sql
CALL mysql.az_load_timezone();
```

若要檢視可用的時區值，請執行以下命令：

```sql
SELECT name FROM mysql.time_zone_name;
```

### <a name="setting-the-global-level-time-zone"></a>設定全域層級時區

全域層級時區可以使用 [az mysql server configuration set](/cli/azure/mysql/server/configuration#az-mysql-server-configuration-set) 命令來設定。

以下命令會將資源群組 **myresourcegroup** 下的伺服器 **mydemoserver.mysql.database.azure.com** 的 **time\_zone** 伺服器設定參數更新為 **US/Pacific**。

```azurecli-interactive
az mysql server configuration set --name time_zone --resource-group myresourcegroup --server mydemoserver --value "US/Pacific"
```

### <a name="setting-the-session-level-time-zone"></a>設定工作階段層級時區

工作階段層級時區可以藉由從 MySQL 命令列或 MySQL Workbench 等工具呼叫 `SET time_zone` 命令來設定。 以下範例將時區設為 **US/Pacific** 時區。  

```sql
SET time_zone = 'US/Pacific';
```

如需[日期和時間函式](https://dev.mysql.com/doc/refman/5.7/en/date-and-time-functions.html#function_convert-tz) \(英文\) 的詳細資訊，請參閱 MySQL 文件。


## <a name="next-steps"></a>後續步驟

- 如何設定 [Azure 入口網站中的伺服器參數](howto-server-parameters.md)