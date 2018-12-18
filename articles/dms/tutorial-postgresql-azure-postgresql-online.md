---
title: 使用 Azure 資料庫移轉服務在線上將 PostgreSQL 移轉至適用於 MySQL 的 Azure 資料庫 | Microsoft Docs
description: 了解如何使用 Azure 資料庫移轉服務，在線上將內部部署的 PostgreSQL 移轉至適用於 PostgreSQL 的 Azure 資料庫。
services: dms
author: HJToland3
ms.author: scphang
manager: craigg
ms.reviewer: ''
ms.service: dms
ms.workload: data-services
ms.custom: mvc, tutorial
ms.topic: article
ms.date: 09/26/2018
ms.openlocfilehash: 51103f6661be22e355aafe01783af5afddd51f54
ms.sourcegitcommit: 5b8d9dc7c50a26d8f085a10c7281683ea2da9c10
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/26/2018
ms.locfileid: "47182462"
---
# <a name="migrate-postgresql-to-azure-database-for-postgresql-online-using-dms"></a>在線上使用 DMS 將 PostgreSQL 移轉至適用於 PostgreSQL 的 Azure 資料庫
您可以使用 Azure 資料庫移轉服務，在最短的停機時間內將資料庫從內部部署 PostgreSQL 執行個體移轉至[適用於 PostgreSQL 的 Azure 資料庫](https://docs.microsoft.com/azure/postgresql/)。 換句話說，移轉可在最短的應用程式停止運作時間內完成。 在此教學課程中，您會在 Azure 資料庫移轉服務中使用線上移轉活動，將 **DVD 出租**範例資料庫從內部部署的 PostgreSQL 9.6 執行個體移轉至適用於 PostgreSQL 的 Azure 資料庫。

在此教學課程中，您了解如何：
> [!div class="checklist"]
> * 使用 pgdump 公用程式移轉範例結構描述。
> * 建立 Azure 資料庫移轉服務的執行個體。
> * 使用 Azure 資料庫移轉服務來建立移轉專案。
> * 執行移轉。
> * 監視移轉。

> [!IMPORTANT]
> 為了獲得最佳的移轉體驗，Microsoft 建議在目標資料庫所在的同一個 Azure 區域中，建立 Azure 資料庫移轉服務的執行個體。 跨區域或地理位置移動資料可能使移轉程序變慢，並產生錯誤。

## <a name="prerequisites"></a>先決條件
若要完成此教學課程，您需要：

- 下載並安裝 [PostgreSQL 社群版](https://www.postgresql.org/download/) 9.5、9.6 或 10.3。 來源 PostgreSQL 伺服器版本必須是 9.5.11、9.6.7，10.3 或更新版本。 如需詳細資訊，請參閱[支援的 PostgreSQL 資料庫版本](https://docs.microsoft.com/azure/postgresql/concepts-supported-versions)一文。

    此外，內部部署 PostgreSQL 版本必須符合適用於 PostgreSQL 的 Azure 資料庫版本。 例如，PostgreSQL 9.5.11.5 只能移轉至「適用於 PostgreSQL 的 Azure 資料庫」9.5.11，而無法移轉至 9.6.7。

- [在適用於 PostgreSQL 的 Azure 資料庫中建立執行個體](https://docs.microsoft.com/azure/postgresql/quickstart-create-server-database-portal)。  
- 使用 Azure Resource Manager 部署模型來建立 Azure 資料庫移轉服務的 VNET，其使用 [ExpressRoute](https://docs.microsoft.com/azure/expressroute/expressroute-introduction) 或 [VPN](https://docs.microsoft.com/azure/vpn-gateway/vpn-gateway-about-vpngateways) 提供站對站連線能力給您的內部部署來源伺服器。
- 確定您的 Azure 虛擬網路 (VNET) 網路安全性群組規則不會封鎖下列連接埠 443、53、9354、445、12000。 如需 Azure VNET NSG 流量篩選的詳細資訊，請參閱[使用網路安全性群組來篩選網路流量](https://docs.microsoft.com/azure/virtual-network/virtual-network-vnet-plan-design-arm)。
- 設定[用於 Database Engine 存取的 Windows 防火牆](https://docs.microsoft.com/sql/database-engine/configure-windows/configure-a-windows-firewall-for-database-engine-access)。
- 開啟您的 Windows 防火牆以允許 Azure 資料庫移轉服務存取來源 PostgreSQL Server (依預設會使用 TCP 連接埠 5432)。
- 使用來源資料庫前面的防火牆應用裝置時，您可能必須新增防火牆規則，才能讓 Azure 資料庫移轉服務存取來源資料庫，以進行移轉。
- 為適用於 PostgreSQL 的 Azure 資料庫建立伺服器層級的[防火牆規則](https://docs.microsoft.com/azure/sql-database/sql-database-firewall-configure)，以允許 Azure 資料庫移轉服務存取目標資料庫。 提供用於 Azure 資料庫移轉服務之 VNET 的子網路範圍。
- 叫用 CLI 有兩種方法：
    - 在 Azure 入口網站右上角，選取 [Cloud Shell] 按鈕：
 
       ![Azure 入口網站中的 [Cloud Shell] 按鈕](media\tutorial-postgresql-to-azure-postgresql-online\cloud-shell-button.png)
 
    - 在本機安裝並執行 CLI。 CLI 2.0 是可用於管理 Azure 資源的命令列工具。
     
       若要下載 CLI，請依照[安裝 Azure CLI 2.0](https://docs.microsoft.com/cli/azure/install-azure-cli?view=azure-cli-latest) 一文中的指示進行。 該文章也列出支援 CLI 2.0 的平台。
         
       若要設定適用於 Linux 的 Windows 子系統 (WSL)，請依照 [Windows 10 安裝指南](https://docs.microsoft.com/windows/wsl/install-win10)中的指示執行
 
- 在 postgresql.config 檔案中啟用邏輯複寫，並設定下列參數：
    - wal_level = **logical**
    - max_replication_slots = [插槽數目]，建議設定為 **5 個插槽**
    - max_wal_senders =[並行工作數目] - max_wal_senders 參數設定可執行的並行工作數目，建議設定為 **10 個工作**

## <a name="migrate-the-sample-schema"></a>移轉範例結構描述
若要完成所有資料表物件 (例如資料表結構描述、索引和預存程序)，我們必須從來源資料庫擷取結構描述，並套用至資料庫。

1. 使用 pg_dump-s 命令來建立資料庫的結構描述傾印檔案。 

    ```
    pg_dump -o -h hostname -U db_username -d db_name -s > your_schema.sql
    ```

    例如，若要傾印結構描述檔案 dvdrental 資料庫：
    ```
    pg_dump -o -h localhost -U postgres -d dvdrental -s  > dvdrentalSchema.sql
    ```
 
    如需有關使用 pg_dump 公用程式的詳細資訊，請參閱 [pg-dump](https://www.postgresql.org/docs/9.6/static/app-pgdump.html#PG-DUMP-EXAMPLES) 教學課程中的範例。
 
2. 在您的目標環境中建立一個空的資料庫，即適用於 PostgreSQL 的 Azure 資料庫。

    如需有關如何連線及建立資料庫的詳細資訊，請參閱[在 Azure 入口網站中建立適用於 PostgreSQL 的 Azure 資料庫伺服器](https://docs.microsoft.com/azure/postgresql/quickstart-create-server-database-portal)一文。

3. 透過還原結構描述傾印檔案，將結構描述匯入到您所建立的目標資料庫中。

    ```
    psql -h hostname -U db_username -d db_name < your_schema.sql 
    ```

    例如︰

    ```
    psql -h mypgserver-20170401.postgres.database.azure.com  -U postgres -d dvdrental < dvdrentalSchema.sql
    ```
4. 如果您的結構描述中有外部索引鍵，則移轉的初始載入和持續同步將會失敗。 在 PgAdmin 或 psql 中執行下列指令碼，以擷取 drop 外部索引鍵指令碼，並在目的地 (適用於 PostgreSQL 的 Azure 資料庫) 新增外部索引鍵。
    
    ```
    SELECT Queries.tablename
           ,concat('alter table ', Queries.tablename, ' ', STRING_AGG(concat('DROP CONSTRAINT ', Queries.foreignkey), ',')) as DropQuery
                ,concat('alter table ', Queries.tablename, ' ', 
                                                STRING_AGG(concat('ADD CONSTRAINT ', Queries.foreignkey, ' FOREIGN KEY (', column_name, ')', 'REFERENCES ', foreign_table_name, '(', foreign_column_name, ')' ), ',')) as AddQuery
        FROM
        (SELECT
        tc.table_schema, 
        tc.constraint_name as foreignkey, 
        tc.table_name as tableName, 
        kcu.column_name, 
        ccu.table_schema AS foreign_table_schema,
        ccu.table_name AS foreign_table_name,
        ccu.column_name AS foreign_column_name 
    FROM 
        information_schema.table_constraints AS tc 
        JOIN information_schema.key_column_usage AS kcu
          ON tc.constraint_name = kcu.constraint_name
          AND tc.table_schema = kcu.table_schema
        JOIN information_schema.constraint_column_usage AS ccu
          ON ccu.constraint_name = tc.constraint_name
          AND ccu.table_schema = tc.table_schema
    WHERE constraint_type = 'FOREIGN KEY') Queries
      GROUP BY Queries.tablename;
     ``` 

    在查詢結果中執行卸除外部索引鍵 (這是第二個資料行)。

5.  資料中的觸發程序 (Insert 或 Update 觸發程序) 會在來源中的複寫資料傳入之前，在目標中強制執行資料完整性。 建議您在移轉期間停用**目標上**所有資料表中的觸發程序，並在移轉完成後重新啟用觸發程序。

    若要停用目標資料庫中的觸發程序，請使用下列命令：

    ```
    select concat ('alter table ', event_object_table, ' disable trigger ', trigger_name)
    from information_schema.triggers;
    ```

6.  如果任何資料表中有 ENUM 資料型別，建議您暫時將其更新為目標資料表中的 'character varying' 資料型別。 完成資料複寫後，將資料型別還原為 ENUM。

## <a name="provisioning-an-instance-of-dms-using-the-cli"></a>使用 CLI 佈建 DMS 執行個體

1.  安裝 dms 同步處理延伸模組：
    - 執行下列命令以登入 Azure：        
        ```
        az login
        ```

    - 出現提示時，請開啟網頁瀏覽器並輸入代碼，以驗證您的裝置。 依照列出的指示執行。
    - 新增 dms 延伸模組：
        - 若要列出可用的延伸模組，請執行下列命令：

            ```
            az extension list-available –otable
            ```
        - 若要安裝延伸模組，請執行下列命令：

            ```
            az extension add –n dms-preview
            ```

    - 若要確認是否已正確安裝 dms 延伸模組，請執行下列命令：
 
        ```
        az extension list -otable
        ```
        您應該會看見下列輸出：     

        ```
        ExtensionType    Name
        ---------------  ------
        whl              dms
        ```

    - 隨時透過執行下列命令查看 DMS 中支援的所有命令：
        ```
        az dms -h
        ```
    - 如果您有多個 Azure 訂用帳戶，請執行下列命令來設定您要用來佈建 DMS 服務執行個體的訂用帳戶。

         ```
        az account set -s 97181df2-909d-420b-ab93-1bff15acb6b7
         ```

2.  透過執行下列命令來佈建 DMS 執行個體：

    ```
    az dms create -l [location] -n <newServiceName> -g <yourResourceGroupName> --sku-name BusinessCritical_4vCores --subnet/subscriptions/{vnet subscription id}/resourceGroups/{vnet resource group}/providers/Microsoft.Network/virtualNetworks/{vnet name}/subnets/{subnet name} –tags tagName1=tagValue1 tagWithNoValue
    ```

    例如，下列命令會在以下位置建立服務：
    - 位置：美國東部 2
    - 訂用帳戶：97181df2-909d-420b-ab93-1bff15acb6b7
    - 資源群組名稱：PostgresDemo
    - DMS 服務名稱：PostgresCLI

    ```
    az dms create -l eastus2 -g PostgresDemo -n PostgresCLI --subnet /subscriptions/97181df2-909d-420b-ab93-1bff15acb6b7/resourceGroups/ERNetwork/providers/Microsoft.Network/virtualNetworks/AzureDMS-CORP-USC-VNET-5044/subnets/Subnet-1 --sku-name BusinessCritical_4vCores
    ```
    建立 DMS 服務執行個體需要大約 10-12 分鐘的時間。

3. 若要識別 DMS 代理程式的 IP 位址，以便將它加入 Postgres pg_hba.conf 檔案，請執行下列命令：

    ```
    az network nic list -g <ResourceGroupName>--query '[].ipConfigurations | [].privateIpAddress'
    ```
    例如︰

    ```
    az network nic list -g PostgresDemo --query '[].ipConfigurations | [].privateIpAddress'
    ```

    您應該會取得類似下列位址的結果： 

    ```
    [
      "172.16.136.18"
    ]
    ```

4. 將 DMS 代理程式的 IP 位址加入 Postgres pg_hba.conf 檔案中。
    - 在 DMS 中完成佈建後，請記下 DMS IP 位址。
    - 將 IP 位址加入到來源上的 pg_hba.conf 檔案中，類似於下列項目：

        ```
        host    all     all     172.16.136.18/10    md5
        host    replication     postgres    172.16.136.18/10    md5
        ```

5. 接下來，執行下列命令來建立 PostgreSQL 移轉專案：
    
    ```
    az dms project create -l <location> -g <ResourceGroupName> --service-name <yourServiceName> --source-platform PostgreSQL --target-platform AzureDbforPostgreSQL -n <newProjectName>
    ```
    例如，下列命令會使用這些參數建立專案：

      - 位置：美國中西部
      - 資源群組名稱：PostgresDemo
      - 服務名稱：PostgresCLI
      - 專案名稱：PGMigration
      - 來源平台：PostgreSQL
      - 目標平台：AzureDbForPostgreSql
 
    ```
    az dms project create -l eastus2 -n PGMigration -g PostgresDemo --service-name PostgresCLI --source-platform PostgreSQL --target-platform AzureDbForPostgreSql
    ```
                
6. 使用下列步驟建立 PostgreSQL 移轉工作。

    此步驟包括使用來源 IP、使用者識別碼與密碼、目的地 IP、使用者識別碼、密碼與工作類型來建立連線。

    - 若要查看完整的選項清單，請執行命令：
        ```
        az dms project task create -h
        ```

        針對來源與目標連線，輸入參數指的是具有物件清單的 json 檔案。
 
        適用於 PostgreSQL 連線的連線 JSON 物件格式。
        
        ```
        {
                    "userName": "user name",    // if this is missing or null, you will be prompted
                    "password": null,           // if this is missing or null (highly recommended) you will
                be prompted
                    "serverName": "server name",
                    "databaseName": "database name", // if this is missing, it will default to the 'postgres'
                server
                    "port": 5432                // if this is missing, it will default to 5432
                }
        ```

    - 還有一個列出 json 物件的資料庫選項 json 檔案。 針對 PostgreSQL，資料庫選項 JSON 物件的格式如下所示：

        ```
        [
            {
                "name": "source database",
                "target_database_name": "target database",
            },
            ...n
        ]
        ```

    - 使用記事本建立 json 檔案、複製下列命令並貼到檔案中，然後將檔案儲存在 C:\DMS\source.json 中。
         ```
        {
                    "userName": "postgres",    
                    "password": null,           
                be prompted
                    "serverName": "13.51.14.222",
                    "databaseName": "dvdrental", 
                    "port": 5432                
                }
         ```
    - 建立名為 target.json 的另一個檔案，並另存為 C:\DMS\target.json。 包含下列命令：
        ```
        {
                "userName": " dms@builddemotarget",    
                "password": null,           
                "serverName": " builddemotarget.postgres.database.azure.com",
                "databaseName": "inventory", 
                "port": 5432                
            }
        ```
    - 建立一個資料庫選項 json 檔案，其中將詳細目錄列為要移轉的資料庫：
        ``` 
        [
            {
                "name": "dvdrental",
                "target_database_name": "dvdrental",
            }
        ]
        ```
    - 執行下列命令，該命令會接受來源、目的地與資料庫選項的 json 檔案。

        ``` 
        az dms project task create -g PostgresDemo --project-name PGMigration --source-platform postgresql --target-platform azuredbforpostgresql --source-connection-json c:\DMS\source.json --database-options-json C:\DMS\option.json --service-name PostgresCLI --target-connection-json c:\DMS\target.json –task-type OnlineMigration -n runnowtask    
        ``` 

    此時，您已成功送出移轉工作。

7.  若要顯示工作的進度，請執行下列命令：

    ```
    az dms project task show --service-name PostgresCLI --project-name PGMigration --resource-group PostgresDemo --name Runnowtask
    ```

    或

     ```
    az dms project task show --service-name PostgresCLI --project-name PGMigration --resource-group PostgresDemo --name Runnowtask --expand output
     ```

8. 您也可以從展開的輸出中查詢 migrationState：

    ```
    az dms project task show --service-name PostgresCLI --project-name PGMigration --resource-group PostgresDemo --name Runnowtask --expand output --query 'properties.output[].migrationState | [0]' "READY_TO_COMPLETE"
    ```

## <a name="understanding-migration-task-status"></a>了解移轉工作狀態
在輸出檔案中，有幾個參數指出移轉的進度。 例如，查看下面的輸出檔案：

    ```
    "output": [                                 Database Level
          {
            "appliedChanges": 0,        //Total incremental sync applied after full load
            "cdcDeleteCounter": 0       // Total delete operation  applied after full load
            "cdcInsertCounter": 0,      // Total insert operation applied after full load
            "cdcUpdateCounter": 0,      // Total update operation applied after full load
            "databaseName": "inventory",
            "endedOn": null,
            "fullLoadCompletedTables": 2,   //Number of tables completed full load
            "fullLoadErroredTables": 0, //Number of tables that contain migration error
            "fullLoadLoadingTables": 0, //Number of tables that are in loading status
            "fullLoadQueuedTables": 0,  //Number of tables that are in queued status
            "id": "db|inventory",
            "incomingChanges": 0,       //Number of changes after full load
            "initializationCompleted": true,
            "latency": 0,
            "migrationState": "READY_TO_COMPLETE",  //Status of migration task. READY_TO_COMPLETE means the database is ready for cutover
            "resultType": "DatabaseLevelOutput",
            "startedOn": "2018-07-05T23:36:02.27839+00:00"
          },
          {
            "databaseCount": 1,
            "endedOn": null,
            "id": "dd27aa3a-ed71-4bff-ab34-77db4261101c",
            "resultType": "MigrationLevelOutput",
            "sourceServer": "138.91.123.10",
            "sourceVersion": "PostgreSQL",
            "startedOn": "2018-07-05T23:36:02.27839+00:00",
            "state": "PENDING",
            "targetServer": "builddemotarget.postgres.database.azure.com",
            "targetVersion": "Azure Database for PostgreSQL"
          },
          {                                     Table 1
            "cdcDeleteCounter": 0,
            "cdcInsertCounter": 0,
            "cdcUpdateCounter": 0,
            "dataErrorsCount": 0,
            "databaseName": "inventory",
            "fullLoadEndedOn": "2018-07-05T23:36:20.740701+00:00",  //Full load completed time
            "fullLoadEstFinishTime": "1970-01-01T00:00:00+00:00",
            "fullLoadStartedOn": "2018-07-05T23:36:15.864552+00:00",    //Full load started time
            "fullLoadTotalRows": 10,                    //Number of rows loaded in full load
            "fullLoadTotalVolumeBytes": 7056,               //Volume in Bytes in full load
            "id": "or|inventory|public|actor",          
            "lastModifiedTime": "2018-07-05T23:36:16.880174+00:00",
            "resultType": "TableLevelOutput",
            "state": "COMPLETED",                   //State of migration for this table
            "tableName": "public.catalog",              //Table name
            "totalChangesApplied": 0                //Total sync changes that applied after full load
          },
          {                                     Table 2
            "cdcDeleteCounter": 0,
            "cdcInsertCounter": 50,
            "cdcUpdateCounter": 0,
            "dataErrorsCount": 0,
            "databaseName": "inventory",
            "fullLoadEndedOn": "2018-07-05T23:36:23.963138+00:00",
            "fullLoadEstFinishTime": "1970-01-01T00:00:00+00:00",
            "fullLoadStartedOn": "2018-07-05T23:36:19.302013+00:00",
            "fullLoadTotalRows": 112,
            "fullLoadTotalVolumeBytes": 46592,
            "id": "or|inventory|public|address",
            "lastModifiedTime": "2018-07-05T23:36:20.308646+00:00",
            "resultType": "TableLevelOutput",
            "state": "COMPLETED",
            "tableName": "public.orders",
            "totalChangesApplied": 0
          }
        ],                          DMS migration task state
        "state": "Running", //Migration task state – Running means it is still listening to any changes that might come in                  
        "taskType": null
      },
      "resourceGroup": "PostgresDemo",
      "type": "Microsoft.DataMigration/services/projects/tasks"
    ```

## <a name="cutover-migration-task"></a>完全移轉工作
完整載入完成後，資料庫準備好進行完全移轉。 根據來源伺服器處理新交易的忙碌程度，DMS 工作可能仍會在完整載入完成後套用變更。

若要確保擷取所有資料，請驗證來源與目標資料庫之間的資料列計數。 例如，您可以使用下列命令：

```
"migrationState": "READY_TO_COMPLETE", //Status of migration task. READY_TO_COMPLETE means database is ready for cutover
 "incomingChanges": 0,  //continue to check for a period of 5-10 minutes to make sure no new incoming changes that need to be applied to the target server
   "fullLoadTotalRows": 10, //full load for table 1
    "cdcDeleteCounter": 0,  //delete, insert and update counter on incremental sync after full load
    "cdcInsertCounter": 50,
    "cdcUpdateCounter": 0,
     "fullLoadTotalRows": 112,  //full load for table 2
```

1.  使用下列命令，執行完全移轉資料庫移轉工作：

    ```
    az dms project task cutover -h
    ```

    例如︰

    ```
    az dms project task cutover --service-name PostgresCLI --project-name PGMigration --resource-group PostgresDemo --name Runnowtask  --database-name Inventory
    ```

2.  若要監視移轉進度，請執行下列命令：

    ```
    az dms project task show --service-name PostgresCLI --project-name PGMigration --resource-group PostgresDemo --name Runnowtask
    ```

## <a name="service-project-task-cleanup"></a>服務、專案、工作清除
如果您要取消或刪除任何 DMS 工作、專案或服務，請依照以下順序取消：
- 取消任何執行中的工作
- 刪除工作
- 刪除專案 
- 刪除 DMS 服務

1.  若要取消執行中的工作，請使用下列命令：
    ```
    az dms project task cancel --service-name PostgresCLI --project-name PGMigration --resource-group PostgresDemo --name Runnowtask
     ```

2.  若要刪除執行中的工作，請使用下列命令：
    ```
    az dms project task delete --service-name PostgresCLI --project-name PGMigration --resource-group PostgresDemo --name Runnowtask
    ```

3.  若要取消執行中的專案，請使用下列命令：
     ```
    az dms project task cancel -n runnowtask --project-name PGMigration -g PostgresDemo --service-name PostgresCLI
     ```

4.  若要刪除執行中的專案，請使用下列命令：
    ```
    az dms project task delete -n runnowtask --project-name PGMigration -g PostgresDemo --service-name PostgresCLI
    ```

5.  若要刪除 DMS 服務，請使用下列命令：

     ```
    az dms delete -g ProgresDemo -n PostgresCLI
     ```

## <a name="next-steps"></a>後續步驟
- 如需在線上移轉至「適用於 PostgreSQL 的 Azure 資料庫」時的已知問題與限制，請參閱[適用於 PostgreSQL 的 Azure 資料庫線上移轉的已知問題與因應措施](known-issues-azure-postgresql-online.md)一文。
- 如需 Azure 資料庫移轉服務的相關資訊，請參閱[什麼是 Azure 資料庫移轉服務？](https://docs.microsoft.com/azure/dms/dms-overview)一文。
- 如需「適用於 MySQL 的 Azure 資料庫」的相關資訊，請參閱[什麼是適用於 PostgreSQL 的 Azure 資料庫？](https://docs.microsoft.com/azure/postgresql/overview)一文。