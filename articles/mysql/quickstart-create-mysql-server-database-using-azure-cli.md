---
title: 快速入門：建立適用於 MySQL 的 Azure 資料庫伺服器 - Azure CLI
description: 此快速入門說明如何使用 Azure CLI 在 Azure 資源群組中建立 Azure Database for MySQL 伺服器。
services: mysql
author: ajlam
ms.author: andrela
manager: kfile
editor: jasonwhowell
ms.service: mysql
ms.devlang: azure-cli
ms.topic: quickstart
ms.date: 09/14/2018
ms.custom: mvc
ms.openlocfilehash: c5d76141de6f5d256c7591da928c1395e83a8362
ms.sourcegitcommit: b7e5bbbabc21df9fe93b4c18cc825920a0ab6fab
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/27/2018
ms.locfileid: "47406017"
---
# <a name="create-an-azure-database-for-mysql-server-using-azure-cli"></a>使用 Azure CLI 建立 Azure Database for MySQL 伺服器
此快速入門說明如何使用 Azure CLI，在大約 5 分鐘內於 Azure 資源群組中建立 Azure Database for MySQL 伺服器。 Azure CLI 可用來從命令列或在指令碼中建立和管理 Azure 資源。

如果您沒有 Azure 訂用帳戶，請在開始前建立[免費帳戶](https://azure.microsoft.com/free/)。

[!INCLUDE [cloud-shell-try-it](../../includes/cloud-shell-try-it.md)]

如果您選擇在本機安裝和使用 CLI，此文章會要求您執行 Azure CLI 2.0 版或更新版本。 執行 `az --version` 以尋找版本。 如果您需要安裝或升級，請參閱[安裝 Azure CLI]( /cli/azure/install-azure-cli)。 

如果您有多個訂用帳戶，請選擇資源所在或作為計費對象的適當訂用帳戶。 使用 [az account set](/cli/azure/account#az-account-set) 命令來選取您帳戶底下的特定訂用帳戶 ID。
```azurecli-interactive
az account set --subscription 00000000-0000-0000-0000-000000000000
```

## <a name="create-a-resource-group"></a>建立資源群組
使用 [az group create](https://docs.microsoft.com/azure/azure-resource-manager/resource-group-overview) 命令建立 [Azure 資源群組](/cli/azure/group#az-group-create)。 資源群組是在其中以群組方式部署與管理 Azure 資源的邏輯容器。

下列範例會在 `westus` 位置建立名為 `myresourcegroup` 的資源群組。

```azurecli-interactive
az group create --name myresourcegroup --location westus
```

## <a name="create-an-azure-database-for-mysql-server"></a>建立適用於 MySQL 的 Azure 資料庫伺服器
使用 **[az mysql server create](/cli/azure/mysql/server#az-mysql-server-create)** 命令來建立「適用於 MySQL 的 Azure 資料庫」伺服器。 一部伺服器可以管理多個資料庫。 一般而言，每個專案或每個使用者會使用個別的資料庫。

**設定** | **範例值** | **說明**
---|---|---
name | mydemoserver | 選擇可識別 Azure Database for MySQL 伺服器的唯一名稱。 伺服器名稱只能包含小寫字母、數字及連字號 (-) 字元。 此名稱必須包含 3 到 63 個字元。
resource-group | myresourcegroup | 提供 Azure 資源群組的名稱。
sku-name | GP_Gen4_2 | SKU 的名稱。 遵循簡短形式的慣例 {pricing tier}_{compute generation}_{vCores}。 請參閱下表，以取得有關 sku-name 參數的詳細資訊。
backup-retention | 7 | 備份應保留的時間長度。 單位為天。 範圍為 7-35 天。 
geo-redundant-backup | 已停用 | 是否應針對此伺服器啟用異地備援備份。 允許的值：Enabled、Disabled。
location | westus | 伺服器的 Azure 位置。
ssl-enforcement | 已啟用 | 是否應針對此伺服器啟用 SSL。 允許的值：Enabled、Disabled。
storage-size | 51200 | 伺服器的儲存體容量 (單位為 MB)。 有效 storage-size 的最小值為 5120 MB，並以 1024 MB 的增量增加。 如需儲存體大小限制的詳細資訊，請參閱[定價層](./concepts-pricing-tiers.md)文件。 
version | 5.7 | MySQL 主要版本。
admin-user | myadmin | 適用於系統管理員登入的使用者名稱。 此名稱不得為 **azure_superuser**、**admin**、**administrator**、**root**、**guest** 或 **public**。
admin-password | *安全密碼* | 系統管理員使用者的密碼。 它必須包含 8 到 128 個字元。 您的密碼必須包含下列三個類別的字元：英文大寫字母、英文小寫字母、數字與非英數字元。


sku-name 參數值會遵循慣例 {pricing tier}\_{compute generation}\_{vCores}，如下列範例所示：
+ `--sku-name B_Gen4_4` 對應於基本、第 4 代和 4 個 vCore。
+ `--sku-name GP_Gen5_32` 對應於一般用途、第 5 代和 32 個 vCore。
+ `--sku-name MO_Gen5_2` 對應於記憶體最佳化、第 5 代和 2 個 vCore。

請參閱[定價層](./concepts-pricing-tiers.md)文件，以了解每個區域和每一層的有效值。

下列範例會使用伺服器系統管理員登入 `myadmin` 在 `myresourcegroup` 您的資源群組中建立且名為 `mydemoserver` 的 MySQL 5.7 伺服器 (位於美國西部)。 這是**第 4 代****一般用途**伺服器，它具有 2 個 **vCore**。 將 `<server_admin_password>` 替換成您自己的值。
```azurecli-interactive
az mysql server create --resource-group myresourcegroup --name mydemoserver  --location westus --admin-user myadmin --admin-password <server_admin_password> --sku-name GP_Gen4_2 --version 5.7
```




## <a name="configure-firewall-rule"></a>設定防火牆規則
使用 **[az mysql server firewall-rule create](/cli/azure/mysql/server/firewall-rule#az-mysql-server-firewall-rule-create)** 命令建立 Azure Database for MySQL 伺服器層級的防火牆規則。 伺服器層級的防火牆規則允許外部應用程式 (例如 **mysql.exe** 命令列工具或 MySQL Workbench) 穿過 Azure MySQL 服務防火牆連線到您的伺服器。 

下列範例會建立名為 `AllowMyIP` 的防火牆規則，以允許來自特定 IP 位址 192.168.0.1 的連線。 請替換為與您的連線來源相對應的 IP 位址或 IP 位址範圍。 

```azurecli-interactive
az mysql server firewall-rule create --resource-group myresourcegroup --server mydemoserver --name AllowMyIP --start-ip-address 192.168.0.1 --end-ip-address 192.168.0.1
```

> [!NOTE]
> Azure Database for MySQL 的連線透過連接埠 3306 進行通訊。 如果您嘗試從公司網路內進行連線，可能不允許透過連接埠 3306 的輸出流量。 若是如此，除非 IT 部門開啟連接埠 3306，否則您無法連線到您的伺服器。
> 


## <a name="configure-ssl-settings"></a>進行 SSL 設定
預設會強制執行您的伺服器與用戶端應用程式之間的 SSL 連線。 此預設值可透過加密透過網際網路的資料流，確保「移動中」資料的安全性。 為了讓此快速入門更容易進行，請停用您伺服器的 SSL 連線。 不建議對生產伺服器停用 SSL。 如需詳細資訊，請參閱[在您的應用程式中設定 SSL 連線能力，以安全地連線至適用於 MySQL 的 Azure 資料庫](./howto-configure-ssl.md)。

下列範例會停用在 MySQL 伺服器上強制執行 SSL。
 
 ```azurecli-interactive
 az mysql server update --resource-group myresourcegroup --name mydemoserver --ssl-enforcement Disabled
 ```

## <a name="get-the-connection-information"></a>取得連線資訊

若要連線到您的伺服器，您必須提供主機資訊和存取認證。

```azurecli-interactive
az mysql server show --resource-group myresourcegroup --name mydemoserver
```

結果會採用 JSON 格式。 請記下 **fullyQualifiedDomainName** 和 **administratorLogin**。
```json
{
  "administratorLogin": "myadmin",
  "earliestRestoreDate": null,
  "fullyQualifiedDomainName": "mydemoserver.mysql.database.azure.com",
  "id": "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/myresourcegroup/providers/Microsoft.DBforMySQL/servers/mydemoserver",
  "location": "westus",
  "name": "mydemoserver",
  "resourceGroup": "myresourcegroup",
  "sku": {
    "capacity": 2,
    "family": "Gen4",
    "name": "GP_Gen4_2",
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
  "type": "Microsoft.DBforMySQL/servers",
  "userVisibleState": "Ready",
  "version": "5.7"
}
```

## <a name="connect-to-the-server-using-the-mysqlexe-command-line-tool"></a>使用 mysql.exe 命令列工具連線到伺服器
使用 **mysql.exe** 命令列工具連線到伺服器。 您可以從[這裡](https://dev.mysql.com/downloads/)下載 MySQL，再將它安裝於電腦上。 相反地，您也可以按一下程式碼範例上的 [試用] 按鈕，或是 Azure 入口網站右上角工具列的 `>_` 按鈕，並啟動 **Azure Cloud Shell**。

輸入下一個命令： 

1. 使用 **mysql** 命令列工具連線到伺服器︰
```azurecli-interactive
 mysql -h mydemoserver.mysql.database.azure.com -u myadmin@mydemoserver -p
```

2. 檢視伺服器狀態：
```sql
 mysql> status
```
如果一切順利，命令列工具應輸出下列文字︰

```dos
C:\Users\>mysql -h mydemoserver.mysql.database.azure.com -u myadmin@mydemoserver -p
Enter password: ***********
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 65512
Server version: 5.6.26.0 MySQL Community Server (GPL)

Copyright (c) 2000, 2016, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> status
--------------
mysql  Ver 14.14 Distrib 5.6.35, for Win64 (x86_64)

Connection id:          65512
Current database:
Current user:           myadmin@116.230.243.143
SSL:                    Not in use
Using delimiter:        ;
Server version:         5.6.26.0 MySQL Community Server (GPL)
Protocol version:       10
Connection:             mydemoserver.mysql.database.azure.com via TCP/IP
Server characterset:    latin1
Db     characterset:    latin1
Client characterset:    gbk
Conn.  characterset:    gbk
TCP port:               3306
Uptime:                 2 days 9 hours 47 min 20 sec

Threads: 4  Questions: 34833  Slow queries: 2  Opens: 84  Flush tables: 4  Open tables: 1  Queries per second avg: 0.167
--------------

mysql>
```

> [!TIP]
> 如需其他命令，請參閱 [MySQL 5.7 參考手冊 - 第 4.5.1 章](https://dev.mysql.com/doc/refman/5.7/en/mysql.html)。

## <a name="connect-to-the-server-using-the-mysql-workbench-gui-tool"></a>使用 MySQL Workbench GUI 工具連線到伺服器
1.  在您的用戶端電腦上啟動 MySQL Workbench 應用程式。 您可以從[這裡](https://dev.mysql.com/downloads/workbench/)下載並安裝 MySQL Workbench。

2.  在 [設定新連線] 對話方塊的 [參數] 索引標籤上輸入下列資訊︰

   ![設定新連線](./media/quickstart-create-mysql-server-database-using-azure-cli/setup-new-connection.png)

| **設定** | **建議的值** | **說明** |
|---|---|---|
|   連線名稱 | 我的連線 | 指定此連線的標籤 (這可以是任何項目) |
| 連線方式 | 選擇標準 (TCP/IP) | 使用 TCP/IP 通訊協定來連線到 MySQL> 的 Azure 資料庫 |
| 主機名稱 | mydemoserver.mysql.database.azure.com | 您先前記下的伺服器名稱。 |
| 連接埠 | 3306 | 會使用 MySQL 的預設連接埠。 |
| 使用者名稱 | myadmin@mydemoserver | 先前記下的伺服器管理員登入。 |
| 密碼 | **** | 使用您稍早設定的管理帳戶密碼。 |

按一下 [測試連線] 以測試所有參數是否都已設定正確。
您現在可以按一下連線，以成功連線到伺服器。

## <a name="clean-up-resources"></a>清除資源
如果您不需要這些資源來進行其他快速入門/教學課程，可以執行下列命令加以刪除︰ 

```azurecli-interactive
az group delete --name myresourcegroup
```

若您只想要刪除新建立的伺服器，則可執行 **[az mysql server delete](/cli/azure/mysql/server#az-mysql-server-delete)** 命令。
```azurecli-interactive
az mysql server delete --resource-group myresourcegroup --name mydemoserver
```

## <a name="next-steps"></a>後續步驟

> [!div class="nextstepaction"]
> [使用 Azure CLI 設計 MySQL 資料庫](./tutorial-design-database-using-cli.md)
