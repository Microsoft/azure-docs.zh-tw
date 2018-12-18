---
title: 如何在適用於 PostgreSQL 的 Azure 資料庫中傾印和還原
description: 說明如何在適用於 PostgreSQL 的 Azure 資料庫中，將 PostgreSQL 資料庫擷取到傾印檔案中，並從 pg_dump 所建立的檔案還原。
services: postgresql
author: rachel-msft
ms.author: raagyema
manager: kfile
editor: jasonwhowell
ms.service: postgresql
ms.topic: conceptual
ms.date: 09/22/2018
ms.openlocfilehash: b6e6e8eeea7ee442ccdbb0524cafb2f51ff30268
ms.sourcegitcommit: b7e5bbbabc21df9fe93b4c18cc825920a0ab6fab
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/27/2018
ms.locfileid: "47409604"
---
# <a name="migrate-your-postgresql-database-using-dump-and-restore"></a>使用傾印和還原來移轉 PostgreSQL 資料庫
您可以使用 [pg_dump](https://www.postgresql.org/docs/9.3/static/app-pgdump.html) 將 PostgreSQL 資料庫擷取到傾印檔案，並使用 [pg_restore](https://www.postgresql.org/docs/9.3/static/app-pgrestore.html) 從 pg_dump 所建立的封存檔案還原 PostgreSQL 資料庫。

## <a name="prerequisites"></a>必要條件
若要逐步執行本作法指南，您需要︰
- [適用於 PostgreSQL 的 Azure 資料庫伺服器](quickstart-create-server-database-portal.md)，而且防火牆規則要允許存取其中的資料庫。
- 安裝 [pg_dump](https://www.postgresql.org/docs/9.6/static/app-pgdump.html) 和 [pg_restore](https://www.postgresql.org/docs/9.6/static/app-pgrestore.html) 命令列公用程式

請遵循下列步驟來傾印和還原 PostgreSQL 資料庫：

## <a name="create-a-dump-file-using-pgdump-that-contains-the-data-to-be-loaded"></a>使用 pg_dump 建立傾印檔案，其中包含要載入的資料
若要備份內部部署或 VM 中的現有 PostgreSQL 資料庫，請執行下列命令︰
```bash
pg_dump -Fc -v --host=<host> --username=<name> --dbname=<database name> > <database>.dump
```
例如，如果您有本機伺服器和名為 **testdb** 的資料庫
```bash
pg_dump -Fc -v --host=localhost --username=masterlogin --dbname=testdb > testdb.dump
```


## <a name="restore-the-data-into-the-target-azure-database-for-postrgesql-using-pgrestore"></a>使用 pg_restore 將資料還原至目標「適用於 PostrgeSQL 的 Azure 資料庫」
在建立目標資料庫後，您可以使用 pg_restore 命令和 -d、--dbname 參數，從傾印檔案將資料還原至目標資料庫。
```bash
pg_restore -v --no-owner –-host=<server name> --port=<port> --username=<user@servername> --dbname=<target database name> <database>.dump
```
包括 --no-owner 參數會導致在還原期間建立的所有物件都由使用 --username 指定的使用者所擁有。 如需詳細資訊，請參閱 [pg_restore](https://www.postgresql.org/docs/9.6/static/app-pgrestore.html) 上的官方 PostgreSQL 文件。

> [!NOTE]
> 如果您的 PostgreSQL 伺服器需要 SSL 連線 (在適用於 PostgreSQL 的 Azure 資料庫中預設為開啟)，請設定環境變數 `PGSSLMODE=require`，以便 pg_restore 工具使用 SSL 連線。 若沒有 SSL，可能顯示錯誤訊息：`FATAL:  SSL connection is required. Please specify SSL options and retry.`
>
> 在 Windows 命令列中，執行命令 `SET PGSSLMODE=require`，然後再執行 pg_restore 命令。 在 Linux 或 Bash中，執行命令 `export PGSSLMODE=require`，然後再執行 pg_restore 命令。
>

在此範例中，請從傾印檔案 **testdb.dump**，將資料還原至目標伺服器 **mydemoserver.postgres.database.azure.com** 上的資料庫 **mypgsqldb**。 
```bash
pg_restore -v --no-owner --host=mydemoserver.postgres.database.azure.com --port=5432 --username=mylogin@mydemoserver --dbname=mypgsqldb testdb.dump
```

## <a name="optimizing-the-migration-process"></a>將移轉流程最佳化

將現有 PostgreSQL 資料庫移轉至 適用於 PostgreSQL 的 Azure 資料庫服務，方法之一就是備份來源上的資料庫，再於 Azure 中還原。 若要讓完成移轉所需的時間降到最低，請考慮搭配備份與還原命令命令使用下列參數。

> [!NOTE]
> 如需詳細的語法資訊，請參閱 [pg_dump](https://www.postgresql.org/docs/9.6/static/app-pgdump.html) 與 [pg_restore](https://www.postgresql.org/docs/9.6/static/app-pgrestore.html) 文章。
>

### <a name="for-the-backup"></a>進行備份
- 使用 -Fc 參數建立備份，能夠平行執行還原，加快執行速度。 例如︰

    ```
    pg_dump -h MySourceServerName -U MySourceUserName -Fc -d MySourceDatabaseName > Z:\Data\Backups\MyDatabaseBackup.dump
    ```

### <a name="for-the-restore"></a>進行還原
- 建議您將備份檔案移至 Azure VM，而其和您即將移轉到的適用於 PostgreSQL 的 Azure 資料庫伺服器位於相同區域，然後從該 VM 執行 pg_restore，以減少網路延遲。 我們也建議您建立 VM 時，啟用[加速網路](..\virtual-network\create-vm-accelerated-networking-powershell.md)。
- 根據預設，這應已完成，但請開啟傾印檔案，確認 Create Index 陳述式位在插入的資料之後。 若非如此，請將 Create Index 陳述式移至插入的資料之後。
- 還原時使用參數-Fc 和-j *#*，可對還原進行平行處理。 *#* 是目標伺服器上的核心數目。 您也可以嘗試將 *#* 設定為目標伺服器核心數目的兩倍，以查看所產生的影響。 例如︰

    ```
    pg_restore -h MyTargetServer.postgres.database.azure.com -U MyAzurePostgreSQLUserName -Fc -j 4 -d MyTargetDatabase Z:\Data\Backups\MyDatabaseBackup.dump
    ```

- 您也可以在開頭處新增 *set synchronous_commit = off;* 命令並在結尾處新增 *set synchronous_commit = on;*，來編輯傾印檔案。 在應用程式變更資料之前，未在結尾處將它設為開啟，可能會造成後續遺失資料。

請務必在測試環境中測試和驗證這些命令之後，再於在生產環境中使用。

## <a name="next-steps"></a>後續步驟
- 若要使用匯出和匯入來移轉 PostgreSQL 資料庫，請參閱[使用匯出和匯入移轉 PostgreSQL 資料庫](howto-migrate-using-export-and-import.md)。
- 如需有關將資料庫移轉至適用於 PostgreSQL 的 Azure 資料庫詳細資訊，請參閱[資料庫移轉指南](http://aka.ms/datamigration)。
