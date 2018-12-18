---
title: 如何在適用於 MySQL 的 Azure 資料庫中備份與還原伺服器
description: 了解如何使用 Azure CLI，在適用於 MySQL 的 Azure 資料庫中備份和還原伺服器。
services: mysql
author: rachel-msft
ms.author: raagyema
manager: kfile
editor: jasonwhowell
ms.service: mysql
ms.devlang: azure-cli
ms.topic: article
ms.date: 04/01/2018
ms.openlocfilehash: f48bab5cec1e8c3836ab5044fbff1a843ede249b
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46978367"
---
# <a name="how-to-back-up-and-restore-a-server-in-azure-database-for-mysql-using-the-azure-cli"></a>如何使用 Azure CLI 在適用於 MySQL 的 Azure 資料庫中備份和還原伺服器

## <a name="backup-happens-automatically"></a>備份會自動進行
為了能使用還原功能，適用於 MySQL 的 Azure 資料庫伺服器會定期備份。 透過此功能，您可以將伺服器和其所有資料庫還原至更早的時間點 (在新的伺服器上)。

## <a name="prerequisites"></a>必要條件
若要完成本操作說明指南，您需要：
- [適用於 MySQL 的 Azure 資料庫伺服器和資料庫](quickstart-create-mysql-server-database-using-azure-cli.md)

[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)]

 

> [!IMPORTANT]
> 本操作說明指南會要求您使用 Azure CLI 2.0 版或更新版本。 若要確認版本，請在 Azure CLI 命令提示字元中輸入 `az --version`。 若要安裝或升級，請參閱[安裝 Azure CLI]( /cli/azure/install-azure-cli)。

## <a name="set-backup-configuration"></a>設定備份組態

建立伺服器時，您需選擇要將伺服器設定為使用本地備援備份還是異地備援備份。 

> [!NOTE]
> 伺服器建立後，就無法切換其備援類型 (異地備援和本地備援)。
>

透過 `az mysql server create` 命令建立伺服器時，`--geo-redundant-backup` 參數會決定您的「備份備援選項」。 如果是 `Enabled`，就會採用異地備援備份。 如果是 `Disabled`，則會採用本地備援備份。 

備份保留期會由 `--backup-retention` 參數設定。 

如需有關在建立期間設定這些值的詳細資訊，請參閱[適用於 MySQL 的 Azure 資料庫伺服器 CLI 快速入門](quickstart-create-mysql-server-database-using-azure-cli.md)。

您可以依下列方式變更伺服器的備份保留期：

```azurecli-interactive
az mysql server update --name mydemoserver --resource-group myresourcegroup --backup-retention 10
```

上述範例會將 mydemoserver 的備份保留期變更為 10 天。

備份保留期限會控制可往回多少時間來擷取時間點還原，因為這會以可用的備份為基礎。 下一節會進一步說明時間點還原。

## <a name="server-point-in-time-restore"></a>時間點還原
您可以將伺服器還原至先前的時間點。 還原的資料會複製到新的伺服器，而現有的伺服器則保持原狀。 例如，如果資料表在今天中午意外卸除，您可以還原到中午之前的時間。 然後，您可從伺服器的還原複本擷取遺失的資料表和資料。 

若要還原伺服器，請使用 Azure CLI [az mysql server restore](/cli/azure/mysql/server#az-mysql-server-restore) 命令。

### <a name="run-the-restore-command"></a>執行 restore 命令

若要還原伺服器，請在 Azure CLI 命令提示字元中輸入下列命令：

```azurecli-interactive
az mysql server restore --resource-group myresourcegroup --name mydemoserver-restored --restore-point-in-time 2018-03-13T13:59:00Z --source-server mydemoserver
```

`az mysql server restore` 命令需要下列參數：
| 設定 | 建議的值 | 說明  |
| --- | --- | --- |
| resource-group |  myresourcegroup |  來源伺服器所在的資源群組。  |
| name | mydemoserver-restored | 還原命令所建立之新伺服器的名稱。 |
| restore-point-in-time | 2018-03-13T13:59:00Z | 選取要還原的時間點。 這個日期和時間必須在來源伺服器的備份保留期限內。 請使用 ISO8601 日期和時間格式。 例如，您可以使用自己的當地時區，例如 `2018-03-13T05:59:00-08:00`。 您也可以使用 UTC Zulu 格式，例如 `2018-03-13T13:59:00Z`。 |
| source-server | mydemoserver | 要進行還原的來源伺服器之名稱或識別碼。 |

當您將伺服器還原到之前的時間點時，會建立新的伺服器。 指定時間點的原始伺服器及其資料庫會複製到新的伺服器。

已還原伺服器的位置與定價層值與原始伺服器相同。 

完成還原程序後，找出新的伺服器，確認資料如預期般還原。

## <a name="geo-restore"></a>異地還原
如果您已將伺服器設定為使用異地備援備份，則可以從現有伺服器的備份建立新的伺服器。 您可以在任何可使用「適用於 MySQL 的 Azure 資料庫」的區域中建立這個新伺服器。  

若要使用異地備援備份來建立伺服器，請使用 Azure CLI `az mysql server georestore` 命令。

> [!NOTE]
> 第一次建立伺服器時，可能無法立即用來進行異地還原。 必要的中繼資料可能需要幾小時才會填入。
>

若要異地還原伺服器，請在 Azure CLI 命令提示字元中輸入下列命令：

```azurecli-interactive
az mysql server georestore --resource-group myresourcegroup --name mydemoserver-georestored --source-server mydemoserver --location eastus --sku-name GP_Gen4_8 
```
此命令會在美國東部建立一個名為 *mydemoserver-georestored*、將屬於 *myresourcegroup* 的新伺服器。 這是一部一般用途、具有 8 個 vCore 的第 4 代伺服器。 此伺服器是從 *mydemoserver* (同樣位於 *myresourcegroup* 資源群組) 的異地備援備份建立的

如果您想要在與現有伺服器不同的來源群組中建立新伺服器，則在 `--source-server` 參數中，您將限定伺服器名稱，如以下範例所示：

```azurecli-interactive
az mysql server georestore --resource-group newresourcegroup --name mydemoserver-georestored --source-server "/subscriptions/$<subscription ID>/resourceGroups/$<resource group ID>/providers/Microsoft.DBforMySQL/servers/mydemoserver" --location eastus --sku-name GP_Gen4_8

```

`az mysql server georestore` 命令需要下列參數：
| 設定 | 建議的值 | 說明  |
| --- | --- | --- |
|resource-group| myresourcegroup | 新伺服器將所屬的資源群組名稱。|
|name | mydemoserver-georestored | 新伺服器的名稱。 |
|source-server | mydemoserver | 所使用之異地備援備份所屬的現有伺服器名稱。 |
|location | eastus | 新伺服器的位置。 |
|sku-name| GP_Gen4_8 | 此參數會設定新伺服器的定價層、計算世代及 vCore 數目。 GP_Gen4_8 所對應的是一般用途、具有 8 個 vCore 的第 4 代伺服器。|


>[!Important]
>透過異地還原建立新伺服器時，它會繼承與來源伺服器相同的儲存體大小和定價層。 在建立期間無法變更這些值。 在建立新伺服器之後，您可以調高儲存體大小。

完成還原程序後，找出新的伺服器，確認資料如預期般還原。

## <a name="next-steps"></a>後續步驟
- 深入了解服務的[備份](concepts-backup.md)。
- 深入了解[商務持續性](concepts-business-continuity.md)選項。
