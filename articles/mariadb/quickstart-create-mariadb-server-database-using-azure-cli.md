---
title: 快速入門：建立適用於 MariaDB 的 Azure 資料庫伺服器 - Azure CLI
description: 本快速入門說明如何使用 Azure CLI，在 Azure 資源群組中建立適用於 MariaDB 的 Azure 資料庫伺服器。
author: ajlam
ms.author: andrela
editor: jasonwhowell
services: mariadb
ms.service: mariadb
ms.devlang: azure-cli
ms.topic: quickstart
ms.date: 09/24/2018
ms.custom: mvc
ms.openlocfilehash: d500a5cab4373d21b729a177ef847c40c2f4211b
ms.sourcegitcommit: 8e06d67ea248340a83341f920881092fd2a4163c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/16/2018
ms.locfileid: "49354014"
---
# <a name="create-an-azure-database-for-mariadb-server-by-using-the-azure-cli"></a>使用 Azure CLI 建立適用於 MariaDB 的 Azure 資料庫伺服器

您可以使用 Azure CLI 從命令列或在指令碼中建立和管理 Azure 資源。 本快速入門說明如何使用 Azure CLI，大約 5 分鐘內，在 Azure 資源群組中建立適用於 MariaDB 的 Azure 資料庫伺服器。 

如果您沒有 Azure 訂用帳戶，請在開始前建立[免費帳戶](https://azure.microsoft.com/free/)。

[!INCLUDE [cloud-shell-try-it](../../includes/cloud-shell-try-it.md)]

如果您在本機安裝並使用 CLI，針對本快速入門，您必須執行 Azure CLI 2.0 版或更新版本。 執行 `az --version` 以尋找版本。 如果您需要安裝或升級 CLI，請參閱[安裝 Azure CLI 2.0]( /cli/azure/install-azure-cli)。 

如果您有多個訂用帳戶，請選擇包含此資源的訂用帳戶或您在其中計費的訂用帳戶。 使用 [az account set](/cli/azure/account#az-account-set) 命令，選取您帳戶中的特定訂用帳戶識別碼：

```azurecli-interactive
az account set --subscription 00000000-0000-0000-0000-000000000000
```

## <a name="create-a-resource-group"></a>建立資源群組

使用 [az group create](https://docs.microsoft.com/azure/azure-resource-manager/resource-group-overview) 命令，建立 [Azure 資源群組](/cli/azure/group#az-group-create)。 資源群組是一個邏輯容器，Azure 資源會在其中以群組方式部署及管理。

下列範例會在 `westus` 位置建立名為 `myresourcegroup` 的資源群組：

```azurecli-interactive
az group create --name myresourcegroup --location westus
```

## <a name="create-an-azure-database-for-mariadb-server"></a>建立適用於 MariaDB 的 Azure 資料庫伺服器

使用 [az mariadb server create](/cli/azure/mariadb/server#az-mariadb-server-create) 命令來建立適用於 MariaDB 的 Azure 資料庫伺服器。 一部伺服器可以管理多個資料庫。 一般而言，每個專案或每個使用者會使用個別的資料庫。

設定 | 範例值 | 說明
---|---|---
name | **mydemoserver** | 輸入可識別適用於 MariaDB 的 Azure 資料庫伺服器的唯一名稱。 伺服器名稱只能包含小寫字母、數字及連字號 (-) 字元。 它必須包含 3 到 63 個字元。
resource-group | **myresourcegroup** | 輸入 Azure 資源群組的名稱。
sku-name | **GP_Gen5_2** | SKU 的名稱。 遵循簡短形式的慣例 *pricing tier*\_*compute generation*\_*vCores*。 如需 **sku-name** 參數的詳細資訊，請參閱此表格之後的章節。
backup-retention | **7** | 備份應保留的時間長度。 單位為天。 範圍：7 到 35。 
geo-redundant-backup | **Disabled** | 是否應針對此伺服器啟用異地備援備份。 允許的值：**Enabled**、**Disabled**。
location | **westus** | 伺服器的 Azure 位置。
ssl-enforcement | **已啟用** | 是否應針對此伺服器啟用 SSL。 允許的值：**Enabled**、**Disabled**。
storage-size | **51200** | 伺服器的儲存體容量 (單位為 MB)。 有效的儲存體大小為 5120 MB (最小值)，並以 1024 MB 的增量增加。 如需儲存體大小限制的詳細資訊，請參閱[定價層](./concepts-pricing-tiers.md)。 
version | **10.2** | MariaDB 主要引擎版本。
admin-user | myadmin | 適用於系統管理員登入的使用者名稱。 **admin-user** 參數不能是 **azure_superuser**、**admin**、**administrator**、**root**、**guest** 或 **public**。
admin-password | *您的密碼* | 系統管理員使用者的密碼。 您的密碼必須包含 8 到 128 個字元。 其中必須包含下列三個類別的字元：英文大寫字母、英文小寫字母、數字與非英數字元。

**sku-name** 參數值會遵循慣例 *pricing tier*\_*compute generation*\_*vCores*，如下列範例所示：
+ `--sku-name B_Gen5_4` 會對應至基本定價層、 第 5 代計算世代及 4 個 vCore。
+ `--sku-name GP_Gen5_32` 會對應至一般用途定價層、第 5 代計算世代及 32 個 vCore。
+ `--sku-name MO_Gen5_2` 會對應至記憶體最佳化定價層、第 5 代計算世代及 2 個 vCore。

如需各區域和階層的有效值，請參閱[定價層](./concepts-pricing-tiers.md)。

下列範例會在美國西部區域建立一個名為 **mydemoserver** 的伺服器。 此伺服器位於 **myresourcegroup** 資源群組且具有伺服器管理員登入 **myadmin**。 此伺服器為一般用途定價層中的第 5 代伺服器，並具有 2 個 vCore。 伺服器名稱會對應至 DNS 名稱，而且在 Azure 中必須是全域唯一的。 以您自己的伺服器管理員密碼取代 `<server_admin_password>`。

```azurecli-interactive
az mariadb server create --resource-group myresourcegroup --name mydemoserver  --location westus --admin-user myadmin --admin-password <server_admin_password> --sku-name GP_Gen5_2 --version 10.2
```

## <a name="configure-a-firewall-rule"></a>設定防火牆規則

使用 [az mariadb server firewall-rule create](/cli/azure/mariadb/server/firewall-rule#az-mariadb-server-firewall-rule-create) 命令，建立適用於 MariaDB 的 Azure 資料庫伺服器層級的防火牆規則。 伺服器層級的防火牆規則允許外部應用程式 (例如 mysql 命令列工具或 MySQL Workbench) 透過適用於 MariaDB 的 Azure 資料庫服務防火牆連線到您的伺服器。 

下列範例會建立名為 `AllowMyIP` 的防火牆規則，以允許來自特定 IP 位址 192.168.0.1 的連線。 請替換為與您的連線來源位置相對應的 IP 位址或 IP 位址範圍。 

```azurecli-interactive
az mariadb server firewall-rule create --resource-group myresourcegroup --server mydemoserver --name AllowMyIP --start-ip-address 192.168.0.1 --end-ip-address 192.168.0.1
```

> [!NOTE]
> 適用於 MariaDB 的 Azure 資料庫連線會透過連接埠 3306 進行通訊。 如果您嘗試從公司網路內部進行連線，可能不允許透過連接埠 3306 的輸出流量。 若是如此，唯有 IT 部門開啟連接埠 3306，您才能連線到您的伺服器。
> 

## <a name="configure-ssl-settings"></a>進行 SSL 設定

預設會強制執行您的伺服器與用戶端應用程式之間的 SSL 連線。 此預設設定可藉由加密透過網際網路的資料流，確保「移動中」資料的安全性。 針對本快速入門，請停用您伺服器的 SSL 連線。 不建議對生產伺服器停用 SSL。 如需詳細資訊，請參閱[在您的應用程式中設定 SSL 連線能力，以安全地連線至適用於 MariaDB 的 Azure 資料庫](./howto-configure-ssl.md)。

下列範例會停用在適用於 MariaDB 的 Azure 資料庫伺服器上強制執行 SSL：
 
```azurecli-interactive
az mariadb server update --resource-group myresourcegroup --name mydemoserver --ssl-enforcement Disabled
```

## <a name="get-connection-information"></a>取得連線資訊

若要連線到您的伺服器，您必須提供主機資訊和存取認證。 若要取得連線資訊，請執行下列命令：

```azurecli-interactive
az mariadb server show --resource-group myresourcegroup --name mydemoserver
```

結果會採用 JSON 格式。 請記下 **fullyQualifiedDomainName** 和 **administratorLogin** 的值。

```json
{
  "administratorLogin": "myadmin",
  "earliestRestoreDate": null,
  "fullyQualifiedDomainName": "mydemoserver.mariadb.database.azure.com",
  "id": "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/myresourcegroup/providers/Microsoft.DBforMariaDB/servers/mydemoserver",
  "location": "westus",
  "name": "mydemoserver",
  "resourceGroup": "myresourcegroup",
  "sku": {
    "capacity": 2,
    "family": "Gen5",
    "name": "GP_Gen5_2",
    "size": null,
    "tier": "GeneralPurpose"
  },
  "sslEnforcement": "Enabled",
  "storageProfile": {
    "backupRetentionDays": 7,
    "geoRedundantBackup": "Disabled",
    "storageMb": 5120
  },
  "tags": null,
  "type": "Microsoft.DBforMariaDB/servers",
  "userVisibleState": "Ready",
  "version": "10.2"
}
```

## <a name="connect-to-the-server-by-using-the-mysql-command-line-tool"></a>使用 mysql 命令列工具連線到伺服器

使用 mysql 命令列工具連線到伺服器。 您可以[下載](https://dev.mysql.com/downloads/)此命令列工具，並將它安裝於電腦上。 選取本文中程式碼範例上的 [試試看] 按鈕，也可以存取此命令列工具。 另一種存取此命令列工具的方法是選取 Azure 入口網站右上方工具列上的 **>_** 按鈕，以開啟 **Azure Cloud Shell**。

若要使用 mysql 命令列工具連線到伺服器：

1. 連接到伺服器：

  ```azurecli-interactive
  mysql -h mydemoserver.mariadb.database.azure.com -u myadmin@mydemoserver -p
  ```

2. 在 `mysql>` 提示字元上檢視伺服器狀態：

  ```sql
  status
  ```
  您應該會看到類似下面的文字：

  ```bash
  C:\Users\>mysql -h mydemoserver.mariadb.database.azure.com -u myadmin@mydemoserver -p
  Enter password: ***********
  Welcome to the MySQL monitor.  Commands end with ; or \g.
  Your MySQL connection id is 65512
  Server version: 5.6.39.0 MariaDB Server

  Copyright (c) 2000, 2016, Oracle and/or its affiliates. All rights reserved.

  Oracle is a registered trademark of Oracle Corporation and/or its
  affiliates. Other names may be trademarks of their respective
  owners.

  Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

  mysql> status
  --------------
  mysql  Ver 14.14 Distrib 5.7.23, for Linux (x86_64)

  Connection id:          64681
  Current database:
  Current user:           myadmin@40.118.201.21
  SSL:                    Cipher in use is AES256-SHA
  Current pager:          stdout
  Using outfile:          ''
  Using delimiter:        ;
  Server version:         5.6.39.0 MariaDB Server
  Protocol version:       10
  Connection:             mydemoserver.mariadb.database.azure.com via TCP/IP
  Server characterset:    latin1
  Db     characterset:    latin1
  Client characterset:    utf8
  Conn.  characterset:    utf8
  TCP port:               3306
  Uptime:                 1 day 3 hours 28 min 50 sec

  Threads: 10  Questions: 29002  Slow queries: 0  Opens: 33  Flush tables: 3  Open tables: 1  Queries per second avg: 0.293
  --------------

  mysql>
  ```

> [!TIP]
> 如需其他命令，請參閱 [MySQL 5.7 參考手冊 - 第 4.5.1 章](https://dev.mysql.com/doc/refman/5.7/en/mysql.html)。

## <a name="connect-to-the-server-by-using-mysql-workbench"></a>使用 MySQL Workbench 來連線到伺服器

1.  開啟用戶端電腦上的 MySQL Workbench。 如果尚未安裝，請[下載](https://dev.mysql.com/downloads/workbench/)並安裝此應用程式。

2.  在 [設定新連線] 對話方塊的 [參數] 索引標籤上，輸入下列資訊︰

 ![設定新的連線](./media/quickstart-create-mariadb-server-database-using-azure-cli/setup-new-connection.png)

  | 設定 | 建議的值 | 說明 |
  |---|---|---|
  | 連線名稱 | **示範連線** | 輸入此連線的標籤 (連線名稱不拘) |
  | 連線方式 | **標準 (TCP/IP)** | 使用 TCP/IP 通訊協定連線到適用於 MariaDB 的 Azure 資料庫 |
  | 主機名稱 | **mydemoserver.mariadb.database.azure.com** | 您先前記下的伺服器名稱。 |
  | Port | **3306** | 適用於 MariaDB 的 Azure 資料庫的預設連接埠。 |
  | 使用者名稱 | **myadmin@mydemoserver** | 您先前記下的伺服器管理員登入。 |
  | 密碼 | *您的密碼* | 使用您稍早設定的管理帳戶密碼。 |

3. 若要檢查所有參數是否設定正確，請選取 [測試連線]。

4. 選取此連線以成功連線到伺服器。

## <a name="clean-up-resources"></a>清除資源

如果您不需要本快速入門中使用的資源來進行其他快速入門或教學課程，您可以執行下列命令加以刪除︰ 

```azurecli-interactive
az group delete --name myresourcegroup
```

如果您只想刪除您在本快速入門中建立的伺服器，請執行 [az mariadb server delete](/cli/azure/mariadb/server#az-mariadb-server-delete) 命令：

```azurecli-interactive
az mariadb server delete --resource-group myresourcegroup --name mydemoserver
```

<!--
## Next steps

> [!div class="nextstepaction"]
> [Design a MariaDB Database with Azure CLI](./tutorial-design-database-using-cli.md)
-->