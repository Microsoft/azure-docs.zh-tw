---
title: 使用 Azure CLI 建立和管理適用於 PostgreSQL 的 Azure 資料庫防火牆規則
description: 本文說明如何使用 Azure CLI 命令列，建立和管理適用於 PostgreSQL 的 Azure 資料庫防火牆規則。
services: postgresql
author: rachel-msft
ms.author: raagyema
manager: kfile
editor: jasonwhowell
ms.service: postgresql
ms.devlang: azure-cli
ms.topic: article
ms.date: 05/4/2018
ms.openlocfilehash: f5133b5da055710208390bfe7fd5d6d7d85696df
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46965332"
---
# <a name="create-and-manage-azure-database-for-postgresql-firewall-rules-using-azure-cli"></a>使用 Azure CLI 建立和管理適用於 PostgreSQL 的 Azure 資料庫防火牆規則
伺服器等級防火牆規則可讓系統管理員從特定的 IP 位址或 IP 位址範圍，管理和存取適用於 PostgreSQL 的 Azure 資料庫伺服器。 透過方便的 Azure CLI 命令，您可以建立、更新、刪除、列出及顯示防火牆規則，以管理您的伺服器。 如需「適用於 PostgreSQL 的 Azure 資料庫」防火牆規則的概觀，請參閱[適用於 PostgreSQL 的 Azure 資料庫伺服器防火牆規則](concepts-firewall-rules.md)

## <a name="prerequisites"></a>必要條件
若要逐步執行本作法指南，您需要︰
- 安裝 [Azure CLI](/cli/azure/install-azure-cli) 命令列公用程式，或在瀏覽器中使用 Azure Cloud Shell。
- [「適用於 PostgreSQL 的 Azure 資料庫」伺服器和資料庫](quickstart-create-server-database-azure-cli.md)。

## <a name="configure-firewall-rules-for-azure-database-for-postgresql"></a>設定「適用於 PostgreSQL 的 Azure 資料庫」的防火牆規則
[az postgres server firewall-rule](/cli/azure/postgres/server/firewall-rule) 命令可用於設定防火牆規則。

## <a name="list-firewall-rules"></a>列出防火牆規則 
若要列出現有的伺服器防火牆規則，請執行 [az postgres server firewall-rule list](/cli/azure/postgres/server/firewall-rule#az_postgres_server_firewall_rule_list) 命令。
```azurecli-interactive
az postgres server firewall-rule list --resource-group myresourcegroup --server-name mydemoserver
```
輸出預設會以 JSON 格式列出防火牆規則 (若有的話)。 您可以使用參數 `--output table`，以更容易閱讀的資料表格式呈現輸出。
```azurecli-interactive
az postgres server firewall-rule list --resource-group myresourcegroup --server-name mydemoserver --output table
```
## <a name="create-firewall-rule"></a>建立防火牆規則
若要在伺服器上建立新的防火牆規則，請執行 [az postgres server firewall-rule create](/cli/azure/postgres/server/firewall-rule#az_postgres_server_firewall_rule_create) 命令。 

藉由將 0.0.0.0 指定為 `--start-ip-address` 並將 255.255.255.255 指定為 `--end-ip-address` 範圍，以下範例可讓所有 IP 位址存取 **mydemoserver.postgres.database.azure.com** 伺服器
```azurecli-interactive
az postgres server firewall-rule create --resource-group myresourcegroup --server-name mydemoserver --name AllowIpRange --start-ip-address 0.0.0.0 --end-ip-address 255.255.255.255
```
若要允許單一 IP 位址進行存取，請在 `--start-ip-address` 和 `--end-ip-address` 中提供相同的位址，如以下範例所示。
```azurecli-interactive
az postgres server firewall-rule create --resource-group myresourcegroup --server-name mydemoserver --name AllowSingleIpAddress --start-ip-address 13.83.152.1 --end-ip-address 13.83.152.1
```
若要讓來自 Azure IP 位址的應用程式連線到您適用於 PostgreSQL 的 Azure 資料庫伺服器，請提供 IP 位址 0.0.0.0 作為起始 IP 和結束 IP，如此範例所示。
```azurecli-interactive
az postgres server firewall-rule create --resource-group myresourcegroup --server-name mydemoserver --name AllowAllAzureIps --start-ip-address 0.0.0.0 --end-ip-address 0.0.0.0
```

> [!IMPORTANT]
> 這個選項會設定防火牆，以允許所有來自 Azure 的連線，包括來自其他客戶訂用帳戶的連線。 選取這個選項時，請確定您的登入和使用者權限會限制為只有授權的使用者才能存取。
> 

成功時，命令輸出會列出您已建立之防火牆規則的詳細資料，預設為 JSON 格式。 如果失敗，輸出就會改為顯示錯誤訊息。

## <a name="update-firewall-rule"></a>更新防火牆規則 
請使用 [az postgres server firewall-rule update](/cli/azure/postgres/server/firewall-rule#az_postgres_server_firewall_rule_update) 命令更新伺服器上現有的防火牆規則。 提供現有防火牆規則的名稱作為輸入，以及要更新的起始 IP 和結束 IP 屬性。
```azurecli-interactive
az postgres server firewall-rule update --resource-group myresourcegroup --server-name mydemoserver --name AllowIpRange --start-ip-address 13.83.152.0 --end-ip-address 13.83.152.255
```
成功時，命令輸出會列出您已更新之防火牆規則的詳細資料，預設為 JSON 格式。 如果失敗，輸出就會改為顯示錯誤訊息。
> [!NOTE]
> 如果防火牆規則不存在，更新命令則會相關規則。

## <a name="show-firewall-rule-details"></a>顯示防火牆規則詳細資料
您也可以執行 [az postgres server firewall-rule show](/cli/azure/postgres/server/firewall-rule#az_postgres_server_firewall_rule_show) 命令，來顯示現有伺服器層級防火牆規則的詳細資料。
```azurecli-interactive
az postgres server firewall-rule show --resource-group myresourcegroup --server-name mydemoserver --name AllowIpRange
```
成功時，命令輸出會列出您已指定之防火牆規則的詳細資料，預設為 JSON 格式。 如果失敗，輸出就會改為顯示錯誤訊息。

## <a name="delete-firewall-rule"></a>刪除防火牆規則
若要撤銷某個 IP 範圍對伺服器的存取權，請執行 [az postgres server firewall-rule delete](/cli/azure/postgres/server/firewall-rule#az_postgres_server_firewall_rule_delete) 命令來刪除現有的防火牆規則。 提供現有防火牆規則的名稱。
```azurecli-interactive
az postgres server firewall-rule delete --resource-group myresourcegroup --server-name mydemoserver --name AllowIpRange
```
成功時，沒有任何輸出。 發生錯誤時，就會傳回錯誤訊息文字。

## <a name="next-steps"></a>後續步驟
- 同樣地，您也可以透過網頁瀏覽器，[使用 Azure 入口網站建立和管理適用於 PostgreSQL 的 Azure 資料庫防火牆規則](howto-manage-firewall-using-portal.md)。
- 進一步了解[適用於 PostgreSQL 的 Azure 資料庫伺服器防火牆規則](concepts-firewall-rules.md)。
- 如需連線至「適用於 PostgreSQL 的 Azure 資料庫」伺服器的說明，請參閱[適用於 PostgreSQL 的 Azure 資料庫的連線庫](concepts-connection-libraries.md)。
