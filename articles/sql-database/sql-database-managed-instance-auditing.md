---
title: Azure SQL Database 受控執行個體稽核 | Microsoft Docs
description: 了解如何透過 T-SQL 開始使用 Azure SQL Database 受控執行個體稽核
services: sql-database
ms.service: sql-database
ms.subservice: security
ms.custom: ''
ms.devlang: ''
ms.topic: conceptual
f1_keywords:
- mi.azure.sqlaudit.general.f1
author: ronitr
ms.author: ronitr
ms.reviewer: vanto
manager: craigg
ms.date: 09/20/2018
ms.openlocfilehash: 375e1a0ba4be9483986c088195e380e856d32a67
ms.sourcegitcommit: 4047b262cf2a1441a7ae82f8ac7a80ec148c40c4
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/11/2018
ms.locfileid: "49093743"
---
# <a name="get-started-with-azure-sql-database-managed-instance-auditing"></a>開始使用 Azure SQL Database 受控執行個體稽核

[Azure SQL Database 受控執行個體](sql-database-managed-instance.md)稽核會追蹤資料庫事件，並將事件寫入您 Azure 儲存體帳戶中的稽核記錄。 稽核也具備下列功能：
- 協助您保持法規遵循、了解資料庫活動，以及深入了解可指出商務考量或疑似安全違規的不一致和異常。
- 啟用及推動遵循法規標準，但不保證符合法規。 如需有關支援標準法規的 Azure 程式詳細資訊，請參閱 [Azure 信任中心](https://azure.microsoft.com/support/trust-center/compliance/)。


## <a name="set-up-auditing-for-your-server"></a>設定伺服器的稽核

下節描述在您受控執行個體上進行稽核的設定。
1. 移至 [Azure 入口網站](https://portal.azure.com)。
2. 下列步驟會建立儲存稽核記錄的 Azure 儲存體**容器**。

   - 巡覽至您要儲存稽核記錄的 Azure 儲存體。

     > [!IMPORTANT]
     > 請在與受控執行個體伺服器相同的區域中使用儲存體帳戶，以避免跨區域讀取/寫入。

   - 在儲存體帳戶中，移至 [概觀]，然後按一下 [Blob]。

     ![瀏覽窗格][1]

   - 在上方功能表中，按一下 [+ 容器] 以建立新的容器。

     ![瀏覽窗格][2]

   - 提供一個容器 [名稱]，將公用存取層級設定為 [私人]，然後按一下 [確定]。

     ![瀏覽窗格][3]

   - 在容器清單中，按一下新建立的容器，然後按一下 [容器屬性]。

     ![瀏覽窗格][4]

   - 按一下複製圖示來複製容器 URL，然後儲存 URL (例如，儲存在 [記事本] 中) 供之後使用。 容器 URL 的格式應為 `https://<StorageName>.blob.core.windows.net/<ContainerName>`

     ![瀏覽窗格][5]

3. 下列步驟會產生 Azure 儲存體 **SAS 權杖**，用來將受控執行個體稽核存取權限授予儲存體帳戶。

   - 巡覽至您在上一個步驟中建立容器的 Azure 儲存體帳戶。

   - 在 [儲存體設定] 功能表中按一下 [共用存取簽章]。

     ![瀏覽窗格][6]

   - 請依照下列方式設定 SAS：
     - **允許的服務**：Blob
     - **開始日期**：若要避免時區相關的問題，建議使用昨天的日期。
     - **結束日期**：選擇此 SAS 權杖到期的日期。 

       > [!NOTE]
       > 在過期時更新權杖，以避免稽核失敗。

     - 按一下 [產生 SAS]。

       ![瀏覽窗格][7]

   - 按一下 [產生 SAS] 後，SAS 權杖就會出現在底部。 按一下複製圖示來複製權杖，然後儲存權杖 (例如，儲存在 [記事本] 中) 供之後使用。

     > [!IMPORTANT]
     > 將權杖開頭的問號 (“?”) 移除。

     ![瀏覽窗格][8]

4. 透過 SQL Server Management Studio (SSMS) 連線到您的受控執行個體。

5. 執行下列 T-SQL 陳述式，以使用您在上一個步驟中建立的容器 URL 和 SAS 權杖來**建立新的認證**：

    ```SQL
    CREATE CREDENTIAL [<container_url>]
    WITH IDENTITY='SHARED ACCESS SIGNATURE',
    SECRET = '<SAS KEY>'
    GO
    ```

6. 執行下列 T-SQL 陳述式來建立新的伺服器稽核 (選擇您自己的稽核名稱，使用您在上一個步驟中建立的容器 URL)：

    ```SQL
    CREATE SERVER AUDIT [<your_audit_name>]
    TO URL ( PATH ='<container_url>' [, RETENTION_DAYS =  integer ])
    GO
    ```

    如果未指定，則 `RETENTION_DAYS` 預設為 0 (無限制的保留期)。

    如需其他資訊：
    - [受控執行個體、Azure SQL DB 和 SQL Server 之間的稽核差異](#auditing-differences-between-managed-instance-azure-sql-database-and-sql-server)
    - [CREATE SERVER AUDIT](https://docs.microsoft.com/sql/t-sql/statements/create-server-audit-transact-sql)
    - [ALTER SERVER AUDIT](https://docs.microsoft.com/sql/t-sql/statements/alter-server-audit-transact-sql)

7. 建立您會針對 SQL Server 建立的「伺服器稽核規格」或「資料庫稽核規格」：
    - [建立伺服器稽核規格 T-SQL 指南](https://docs.microsoft.com/sql/t-sql/statements/create-server-audit-specification-transact-sql)
    - [建立資料庫稽核規格 T-SQL 指南](https://docs.microsoft.com/sql/t-sql/statements/create-database-audit-specification-transact-sql)

8. 啟用您在步驟 6 中建立的伺服器稽核：

    ```SQL
    ALTER SERVER AUDIT [<your_audit_name>]
    WITH (STATE=ON);
    GO
    ```

## <a name="analyze-audit-logs"></a>分析稽核記錄
有幾種方法可以用於檢視 Blob 稽核記錄。

- 使用系統函式 `sys.fn_get_audit_file` (T-SQL) 以表格格式傳回稽核記錄資料。 如需有關如何使用此函數的詳細資訊，請參閱 [sys.fn_get_audit_file 文件](https://docs.microsoft.com/sql/relational-databases/system-functions/sys-fn-get-audit-file-transact-sql) (英文)。

- 如需稽核記錄耗用方法的完整清單，請參閱[開始使用 SQL 資料庫稽核](https://docs.microsoft.com/ azure/sql-database/sql-database-auditing)。

> [!IMPORTANT]
> 從 Azure 入口網站 ([稽核記錄] 窗格) 檢視稽核記錄的方法目前不適用於「受控執行個體」。

## <a name="auditing-differences-between-managed-instance-azure-sql-database-and-sql-server"></a>受控執行個體、Azure SQL Database 和 SQL Server 之間的稽核差異

SQL 稽核在受控執行個體、Azure SQL Database 和 SQL Server 內部部署之間的主要差異如下：

- 在受控執行個體中，SQL 稽核會在伺服器層級運作，並將 `.xel` 記錄檔存放在 Azure Blob 儲存體帳戶。
- 在 Azure SQL Database 中，SQL 稽核則在資料庫層級運作。
- 在 SQL Server 內部部署 / 虛擬機器中，SQL 稽核會在伺服器層級運作，但會將事件儲存在檔案系統/windows 事件記錄。

受控執行個體中的 XEvent 稽核支援 Azure Blob 儲存體目標。 **不支援**檔案與 windows 記錄。

向 Azure Blob 儲存體進行稽核的 `CREATE AUDIT` 語法有一個主要差異：
- 已提供新的 `TO URL` 語法，可讓您指定放置 `.xel` 檔案的 Azure Blob 儲存體容器 URL。
- `TO FILE` 語法**不受支援**，因為受控執行個體無法存取 Windows 檔案共用。
- **不支援**關機選項。
- **不支援**使用 0 的 `queue_delay`。


## <a name="next-steps"></a>後續步驟

- 如需稽核記錄耗用方法的完整清單，請參閱[開始使用 SQL 資料庫稽核](https://docs.microsoft.com/ azure/sql-database/sql-database-auditing)。
- 如需有關支援標準法規的 Azure 程式詳細資訊，請參閱 [Azure 信任中心](https://azure.microsoft.com/support/trust-center/compliance/)。


<!--Anchors-->
[Set up auditing for your server]: #subheading-1
[Analyze audit logs]: #subheading-2
[Auditing differences between Managed Instance, Azure SQL DB and SQL Server]: #subheading-3

<!--Image references-->
[1]: ./media/sql-managed-instance-auditing/1_blobs_widget.png
[2]: ./media/sql-managed-instance-auditing/2_create_container_button.png
[3]: ./media/sql-managed-instance-auditing/3_create_container_config.png
[4]: ./media/sql-managed-instance-auditing/4_container_properties_button.png
[5]: ./media/sql-managed-instance-auditing/5_container_copy_name.png
[6]: ./media/sql-managed-instance-auditing/6_storage_settings_menu.png
[7]: ./media/sql-managed-instance-auditing/7_sas_configure.png
[8]: ./media/sql-managed-instance-auditing/8_sas_copy.png
