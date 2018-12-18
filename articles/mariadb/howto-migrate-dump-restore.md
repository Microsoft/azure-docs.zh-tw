---
title: 使用傾印和還原將 MariaDB 資料庫移轉至適用於 MariaDB 的 Azure 資料庫
description: 此文章將說明兩個常見方法，讓您可在適用於 MariaDB 的 Azure 資料庫中用來備份和還原資料庫，使用如 mysqldump、MySQL Workbench 與 PHPMyAdmin 的工具。
author: ajlam
ms.author: andrela
editor: jasonwhowell
services: mariadb
ms.service: mariadb
ms.topic: conceptual
ms.date: 09/24/2018
ms.openlocfilehash: 551fe303994f6c72f8a4bf39e76f12c62f58026b
ms.sourcegitcommit: 3a02e0e8759ab3835d7c58479a05d7907a719d9c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/13/2018
ms.locfileid: "49309530"
---
# <a name="migrate-your-mariadb-database-to-azure-database-for-mariadb-using-dump-and-restore"></a>使用傾印和還原來將 MariaDB 資料庫移轉至適用於 MariaDB 的 Azure 資料庫
此文章將說明兩個常見方法，讓您可在適用於 MariaDB 的 Azure 資料庫中用來備份和還原資料庫
- 從命令列傾印和還原 (使用 mysqldump) 
- 使用 PHPMyAdmin 傾印和還原

## <a name="before-you-begin"></a>開始之前
若要逐步執行本作法指南，您需要具備：
- [建立適用於 MariaDB 的 Azure 資料庫伺服器 - Azure 入口網站](quickstart-create-mariadb-server-database-using-azure-portal.md)
- 已安裝於電腦上的 [mysqldump](https://mariadb.com/kb/en/library/mysqldump/) 命令列公用程式。
- MySQL Workbench [MySQL Workbench 下載](https://dev.mysql.com/downloads/workbench/)、Toad、Navicat 或用來執行傾印和還原命令的其他第三方 MySQL 工具。

## <a name="use-common-tools"></a>使用一般工具
使用一般公用程式和工具 (例如 MySQL Workbench、mysqldump、Toad 或 Navicat) 從遠端連線，然後將資料還原至適用於 MariaDB 的 Azure 資料庫。 在具有網際網路連接的用戶端電腦上使用這類工具，來連線到適用於 MariaDB 的 Azure 資料庫。 如需使用 SSL 加密連接的最佳安全性作法，請參閱[在適用於 MariaDB 的 Azure 資料庫中設定 SSL 連線能力](concepts-ssl-connection-security.md)。 在移轉到適用於 MariaDB 的 Azure 資料庫時，您不需要將傾印檔案移至任何特定的雲端位置。 

## <a name="common-uses-for-dump-and-restore"></a>傾印和還原的常見用途
您可以在數個常見案例中使用 MySQL 公用程式 (例如 mysqldump 與 mysqlpump)，將資料庫傾印及載入至適用於 MariaDB 的 Azure 資料庫伺服器。 

<!--In other scenarios, you may use the [Import and Export](howto-migrate-import-export.md) approach instead.-->

- 當您移轉整個資料庫時，使用資料庫傾印。 此建議在您移動大量資料，或當您想要將即時網站或應用程式的服務中斷時間降到最低時會保留。 
-  將資料載入至適用於 MariaDB 的 Azure 資料庫時，請確定資料庫中的所有資料表都使用 InnoDB 儲存引擎。 適用於 MariaDB 的 Azure 資料庫僅支援 InnoDB 儲存引擎，因此不支援其他儲存引擎。 如果您的資料表是使用其他儲存引擎設定，請將它們轉換成 InnoDB 引擎格式，然後再移轉至適用於 MariaDB 的 Azure 資料庫。
   例如，如果您的 WordPress 或 WebApp 使用 MyISAM 資料表，請先藉由移轉至 InnoDB 格式來轉換這些資料表，然後再還原至適用於 MariaDB 的 Azure 資料庫。 使用子句 `ENGINE=InnoDB` 以設定建立新資料表時使用的引擎，然後在還原之前將資料傳送到相容的資料表。 

   ```sql
   INSERT INTO innodb_table SELECT * FROM myisam_table ORDER BY primary_key_columns
   ```
- 若要避免任何相容性問題，請確定當傾印資料庫時，在來源和目的地系統上使用相同版本的 MariaDB。 例如，如果現有的 MariaDB 伺服器是 10.2 版，則您應該將適用於 MariaDB 的 Azure 資料庫設定為執行 10.2 版。 `mysql_upgrade` 命令在適用於 MariaDB 的 Azure 資料庫伺服器中無法運作，因此並不支援。 如果您要在 MariaDB 版本之間升級，請先將較低版本的資料庫傾印或匯出到自己環境中較高版本的 MariaDB。 接著，執行 `mysql_upgrade`，之後再嘗試移轉至適用於 MariaDB 的 Azure 資料庫。

## <a name="performance-considerations"></a>效能考量
若要最佳化效能，請在傾印大型資料庫時注意這些考量：
-   傾印資料庫時在 mysqldump 中使用 `exclude-triggers` 選項。 從傾印檔案排除觸發程序以避免在資料還原期間引發觸發程序命令。 
-   使用 `single-transaction` 選項將交易隔離模式設為 REPEATABLE READ，然後在傾印資料之前，將 START TRANSACTION 的 SQL 陳述式傳送到伺服器。 在單一交易中傾印許多資料表會導致在還原期間耗用某些額外的儲存體。 `single-transaction` 選項和 `lock-tables` 選項是互斥的，因為 LOCK TABLES 會導致隱含認可任何暫止交易。 若要傾印大型資料表，請結合 `single-transaction` 選項與 `quick` 選項。 
-   使用包含數個 VALUE 清單的 `extended-insert` 多個資料列語法。 這會產生較小的傾印檔案，並在重新載入檔案時加速插入。
-  傾印資料庫時在 mysqldump 中使用 `order-by-primary` 選項，以便將資料以主索引鍵的順序編寫指令碼。
-   傾印資料時在 mysqldump 中使用 `disable-keys` 選項，以在載入之前停用外部索引鍵限制式。 停用外部索引鍵檢查會提供效能提升。 啟用限制式並且確認載入之後的資料，以確保參考完整性。
-   適當時使用資料分割資料表。
-   平行載入資料。 避免會導致您達到資源限制的太多平行處理原則，以及使用 Azure 入口網站中可用的計量監視資源。 
-   傾印資料庫時在 mysqlpump 中使用 `defer-table-indexes` 選項，以便在載入資料表資料之後建立索引。
-   請將備份檔案複製到 Azure blob/存放區，並從該處執行還原，這樣應該會比在網際網路上執行還原更快。

## <a name="create-a-backup-file"></a>建立備份檔案
若要在本機內部部署伺服器或虛擬機器中備份現有的 MariaDB 資料庫，請使用 mysqldump 執行下列命令： 
```bash
$ mysqldump --opt -u [uname] -p[pass] [dbname] > [backupfile.sql]
```

提供的參數如下：
- [uname] 您的資料庫使用者名稱 
- [pass] 您的資料庫密碼 (請注意 -p 與密碼之間沒有空格) 
- [dbname] 您的資料庫名稱 
- [backupfile.sql] 資料庫備份的檔案名稱 
- [--opt] mysqldump 選項 

例如，若要將 MariaDB 伺服器上使用者名稱為 'testuser' 且無密碼之名為 'testdb' 的資料庫備份到 testdb_backup.sql 檔案，請使用下列命令。 此命令會將 `testdb` 資料庫備份至名為 `testdb_backup.sql` 的檔案，其中包含重新建立資料庫所需的所有 SQL 陳述式。 

```bash
$ mysqldump -u root -p testdb > testdb_backup.sql
```
若要在資料庫中選取要備份的特定資料表，請列出以空格分隔的資料表名稱。 例如，如果只要從 'testdb' 備份 table1 和 table2 資料表，請遵循下列範例： 
```bash
$ mysqldump -u root -p testdb table1 table2 > testdb_tables_backup.sql
```
若要一次備份多個資料庫，請使用 --database 參數，並列出以空格分隔的資料庫名稱。 
```bash
$ mysqldump -u root -p --databases testdb1 testdb3 testdb5 > testdb135_backup.sql 
```
若要一次備份伺服器中的所有資料庫，您應該使用 --all-databases 選項。
```bash
$ mysqldump -u root -p --all-databases > alldb_backup.sql 
```

## <a name="create-a-database-on-the-target-server"></a>在目標伺服器上建立資料庫
在您要移轉資料的目標適用於 MariaDB 的 Azure 資料庫伺服器上建立空白資料庫。 使用例如 MySQL Workbench、Toad 或 Navicat 的工具來建立資料庫。 資料庫名稱可以與包含傾印資料的資料庫名稱相同，或者您可以建立名稱不同的資料庫。

若要連線，請在適用於 MariaDB 的 Azure 資料庫的 [概觀] 中尋找連線資訊。

![在 Azure 入口網站中尋找連線資訊](./media/howto-migrate-dump-restore/1_server-overview-name-login.png)

將連線資訊新增至 MySQL Workbench。

![MySQL Workbench 連接字串](./media/howto-migrate-dump-restore/2_setup-new-connection.png)

## <a name="restore-your-mariadb-database"></a>還原您的 MariaDB 資料庫
建立目標資料庫後，您可以使用 mysql 命令或 MySQL Workbench，從傾印檔案將資料還原至新建立的特定資料庫。
```bash
mysql -h [hostname] -u [uname] -p[pass] [db_to_restore] < [backupfile.sql]
```
在此範例中，將資料還原至目標適用於 MariaDB 的 Azure 資料庫伺服器上新建立的資料庫。
```bash
$ mysql -h mydemoserver.mariadb.database.azure.com -u myadmin@mydemoserver -p testdb < testdb_backup.sql
```

## <a name="export-using-phpmyadmin"></a>使用 PHPMyAdmin 匯出
若要匯出，您可以使用一般工具 phpMyAdmin，而您可能已在環境中本機安裝此工具。 使用 PHPMyAdmin 匯出 MariaDB 資料庫：
1. 開啟 phpMyAdmin。
2. 選取您的資料庫。 按一下左邊清單中的資料庫名稱。 
3. 按一下 [匯出] 連結。 新的分頁隨即出現，以供檢視資料庫的傾印。
4. 在 [匯出] 區域中，按一下 [全選] 連結來選擇資料庫中的資料表。 
5. 在 [SQL 選項] 區域中，按一下適當的選項。 
6. 依序按一下 [另存新檔] 和對應的壓縮選項，然後按一下 [執行] 按鈕。 接著應該會出現一個對話方塊，提示您在本機儲存檔案。

## <a name="import-using-phpmyadmin"></a>使用 PHPMyAdmin 匯入
匯入資料庫的程序與匯出類似。 請執行下列動作：
1. 開啟 phpMyAdmin。 
2. 在 phpMyAdmin 設定頁面中，按一下 [新增] 以新增您的適用於 MariaDB 的 Azure 資料庫伺服器。 提供連線詳細資料和登入資訊。
3. 建立已適當命名的資料庫，然後在畫面左邊選取它。 若要重寫現有的資料庫，按一下資料庫名稱、選取資料表名稱旁的所有核取方塊，然後選取 [捨棄] 以刪除現有的資料表。 
4. 按一下 **SQL** 連結，以顯示您可以在其中輸入 SQL 命令或上傳 SQL 檔案的分頁。 
5. 您可以使用**瀏覽**按鈕來尋找資料庫檔案。 
6. 按一下 [執行] 按鈕以匯出備份、執行 SQL 命令，並重新建立您的資料庫。

## <a name="next-steps"></a>後續步驟
- [將應用程式連線至適用於 MariaDB 的 Azure 資料庫](./howto-connection-string.md)。
 
<!--
- For more information about migrating databases to Azure Database for MariaDB, see the [Database Migration Guide](http://aka.ms/datamigration).
-->