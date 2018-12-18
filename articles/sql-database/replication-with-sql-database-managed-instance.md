---
title: 使用 Azure SQL Database 受控執行個體進行複寫 | Microsoft Docs
description: 了解如何搭配 Azure SQL Database 受控執行個體使用 SQL Server 複寫
services: sql-database
ms.service: sql-database
ms.subservice: data-movement
ms.custom: ''
ms.devlang: ''
ms.topic: conceptual
author: allenwux
ms.author: xiwu
ms.reviewer: mathoma
manager: craigg
ms.date: 09/25/2018
ms.openlocfilehash: 95c27bcc99f08cb1e4998e43a6a2abd508bee0ac
ms.sourcegitcommit: ad08b2db50d63c8f550575d2e7bb9a0852efb12f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/26/2018
ms.locfileid: "47228059"
---
# <a name="replication-with-sql-database-managed-instance"></a>使用 SQL Database 受控執行個體進行複寫

複寫可在 [Azure SQL Database 受控執行個體](sql-database-managed-instance.md)的公開預覽版上使用。 受控執行個體可以裝載發行者、散發者和訂閱者資料庫。

## <a name="common-configurations"></a>一般設定

一般情況下，發行者和散發者必須都位在雲端或內部部署中。 不支援下列設定：

- **發行者與本機散發者在受控執行個體上**

   ![Replication-with-azure-sql-db-single-managed-instance-publisher-distributor](./media/replication-with-sql-database-managed-instance/01-single-instance-asdbmi-pubdist.png)

   在單一受控執行個體上設定發行者和 散發者資料庫。

- **發行者與遠端散發者在受控執行個體上**

   ![Replication-with-azure-sql-db-separate-managed-instances-publisher-distributor](./media/replication-with-sql-database-managed-instance/02-separate-instances-asdbmi-pubdist.png)

   在兩個受控執行個體上設定發行者和散發者。 在此設定中：

  - 這兩個受控執行個體位於相同的 vNet。

  - 這兩個受控執行個體位於相同的位置。

- **發行者和散發者在內部部署，而訂閱者在受控執行個體上**

   ![Replication-from-on-premises-to-azure-sql-db-subscriber](./media/replication-with-sql-database-managed-instance/03-azure-sql-db-subscriber.png)

   在此設定中，Azure SQL Database 是訂閱者。 此設定支援從內部部署移轉至 Azure。 在訂閱者角色中，SQL Database 不需要受控執行個體，但是您可以使用 SQL Database 受控執行個體作為從內部部署移轉至 Azure 的步驟。 如需 Azure SQL Database 訂閱者的詳細資訊，請參閱[複寫至 SQL Database](replication-to-sql-database.md)。

## <a name="requirements"></a>需求

Azure SQL Database 上的發行者和散發者需要：

- Azure SQL Database 受控執行個體。

   >[!NOTE]
   >未設定受控執行個體的 Azure SQL Database 只能是訂閱者。

- 所有 SQL Server 執行個體都必須位於相同的 vNet。

- 連線會在複寫參與者之間使用 SQL 驗證。

- 複寫工作目錄的 Azure 儲存體帳戶共用。

## <a name="features"></a>特性

支援：

- 混用內部部署和 Azure SQL Database 受控執行個體的交易式與快照式複寫。

- 訂閱者可以是內部部署、Azure SQL Database 中的單一資料庫，或 Azure SQL Database 彈性集區中的集區資料庫。

- 單向或雙向複寫

## <a name="configure-publishing-and-distribution-example"></a>設定發行與散發範例

1. 在入口網站中[建立 Azure SQL Database 受控執行個體](http://docs.microsoft.com/azure/sql-database/sql-database-managed-instance-create-tutorial-portal)。

1. 針為工作目錄[建立 Azure 儲存體帳戶](http://docs.microsoft.com/azure/storage/common/storage-create-storage-account#create-a-storage-account)。 請務必複製儲存體金鑰。 請參閱[檢視並複製儲存體存取金鑰](http://docs.microsoft.com/azure/storage/common/storage-create-storage-account#manage-your-storage-access-keys)。

1. 建立發行者的資料庫。

   在下列範例指令碼中，以此資料庫的名稱取代 `<Publishing_DB>`。

1. 使用散發者的 SQL 驗證建立資料庫使用者。 請參閱[建立資料庫使用者](http://docs.microsoft.com/azure/sql-database/sql-database-security-tutorial#creating-database-users)。 使用安全的密碼。

   在下列範例指令碼中，搭配此 SQL Server 帳戶資料庫使用者名稱和密碼使用 `<SQL_USER>` 和 `<PASSWORD>`。

1. [連線到 SQL Database 受控執行個體](http://docs.microsoft.com/azure/sql-database/sql-database-connect-query-ssms)。

1. 執行下列查詢，以新增散發者和散發資料庫。

   ```sql
   USE [master]
   GO
   EXEC sp_adddistributor @distributor = @@ServerName;
   EXEC sp_adddistributiondb @database = N'distribution';
   ```

1. 若要設定發行者以使用指定的散發資料庫，請更新並執行下列查詢。

   以 SQL Server 帳戶和密碼取代 `<SQL_USER>` 和 `<PASSWORD>`。

   以您的儲存體帳戶值取代 `\\<STORAGE_ACCOUNT>.file.core.windows.net\<SHARE>`。  

   以來自 Microsoft Azure 儲存體帳戶的 [存取金鑰] 索引標籤連接字串取代 `<STORAGE_CONNECTION_STRING>`。

   更新下列查詢之後，請執行它。 

   ```sql
   USE [master]
   EXEC sp_adddistpublisher @publisher = @@ServerName,
                @distribution_db = N'distribution',
                @security_mode = 0,
                @login = N'<SQL_USER>',
                @password = N'<PASSWORD>',
                @working_directory = N'\\<STORAGE_ACCOUNT>.file.core.windows.net\<SHARE>',
                @storage_connection_string = N'<STORAGE_CONNECTION_STRING>';
   GO
   ```

1. 設定發行者以進行複寫。 

    在下列查詢中，以發行者資料庫的名稱取代 `<Publishing_DB>`。

    以發行集的名稱取代 `<Publication_Name>`。

    以 SQL Server 帳戶和密碼取代 `<SQL_USER>` 和 `<PASSWORD>`。

    更新查詢之後，請執行它來建立發行集。

   ```sql
   USE [<Publishing_DB>]
   EXEC sp_replicationdboption @dbname = N'<Publishing_DB>',
                @optname = N'publish',
                @value = N'true';

   EXEC sp_addpublication @publication = N'<Publication_Name>',
                @status = N'active';

   EXEC sp_changelogreader_agent @publisher_security_mode = 0,
                @publisher_login = N'<SQL_USER>',
                @publisher_password = N'<PASSWORD>',
                @job_login = N'<SQL_USER>',
                @job_password = N'<PASSWORD>';

   EXEC sp_addpublication_snapshot @publication = N'<Publication_Name>',
                @frequency_type = 1,
                @publisher_security_mode = 0,
                @publisher_login = N'<SQL_USER>',
                @publisher_password = N'<PASSWORD>',
                @job_login = N'<SQL_USER>',
                @job_password = N'<PASSWORD>'
   ```

1. 新增發行項、訂用帳戶和發送訂閱代理程式。 

   若要新增這些物件，請更新下列指令碼。

   以發行集物件的名稱取代 `<Object_Name>`。

   以來源結構描述的名稱取代 `<Object_Schema>`。 

   取代用角括弧 `<>` 括住的其他參數以符合先前指令碼中的值。 

   ```sql
   EXEC sp_addarticle @publication = N'<Publication_Name>',
                @type = N'logbased',
                @article = N'<Object_Name>',
                @source_object = N'<Object_Name>',
                @source_owner = N'<Object_Schema>'

   EXEC sp_addsubscription @publication = N'<Publication_Name>',
                @subscriber = @@ServerName,
                @destination_db = N'<Subscribing_DB>',
                @subscription_type = N'Push'

   EXEC sp_addpushsubscription_agent @publication = N'<Publication_Name>',
                @subscriber = @@ServerName,
                @subscriber_db = N'<Subscribing_DB>',
                @subscriber_security_mode = 0,
                @subscriber_login = N'<SQL_USER>',
                @subscriber_password = N'<PASSWORD>',
                @job_login = N'<SQL_USER>', 
                @job_password = N'<PASSWORD>'
   GO
   ```

## <a name="limitations"></a>限制

不支援下列功能：

- 可更新的訂用帳戶

- 使用中的異地複寫

## <a name="see-also"></a>另請參閱

- [什麼是受控執行個體？](http://docs.microsoft.com/azure/sql-database/sql-database-managed-instance)
