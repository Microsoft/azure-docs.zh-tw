---
title: Azure SQL 資料倉儲中的稽核 | Microsoft Docs
description: 了解稽核，以及如何在 Azure SQL 資料倉儲中設定稽核。
services: sql-data-warehouse
author: kavithaj
manager: craigg
ms.service: sql-data-warehouse
ms.topic: conceptual
ms.component: manage
ms.date: 04/11/2018
ms.author: kavithaj
ms.reviewer: igorstan
ms.openlocfilehash: 18a6234eb56902625fdcc8f150a41b7551d229cc
ms.sourcegitcommit: 74941e0d60dbfd5ab44395e1867b2171c4944dbe
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/15/2018
ms.locfileid: "49318532"
---
# <a name="auditing-in-azure-sql-data-warehouse"></a>Azure SQL 資料倉儲中的稽核

了解稽核，以及如何在 Azure SQL 資料倉儲中設定稽核。

## <a name="what-is-auditing"></a>稽核是什麼？
SQL 資料倉儲稽核可讓您將資料庫中的事件記錄到 Azure 儲存體帳戶中的稽核記錄。 稽核可協助您保持法規遵循、了解資料庫活動，以及深入了解可指出商務考量或疑似安全違規的不一致和異常。

稽核工具啟用及推動遵循法規標準，但不保證符合法規。 如需有關支援標準法規的 Azure 程式詳細資訊，請參閱 [Azure 信任中心](https://azure.microsoft.com/support/trust-center/compliance/)。

## <a id="subheading-1"></a>稽核的基本概念
SQL 資料倉儲資料庫稽核可讓您：

* **保留** 所選事件的稽核記錄。 您可以定義要稽核的資料庫動作類別。
* **報告** 資料庫活動。 您可以使用預先設定的報告和儀表板，以便快速開始使用活動和事件報告。
* **分析** 報告。 您可以尋找可疑事件、異常活動及趨勢。

稽核記錄會儲存在 Azure 儲存體帳戶。 您可以定義稽核記錄保留期間。


## <a id="subheading-4"></a>定義伺服器層級與資料庫層級的稽核原則

您可以針對特定資料庫定義稽核原則，或將稽核原則定義為預設伺服器原則：

* 伺服器原則會**套用至伺服器上所有現有和新建立的資料庫**。

* 如果「伺服器 Blob 稽核已啟用」，它「一律」會套用到資料庫。 不論資料庫稽核資料是什麼，都會稽核資料庫。

* 如果在伺服器和資料庫上都啟用稽核，這「不會」覆寫或變更伺服器 Blob 稽核的任何設定。 這兩種稽核將會並存。 換句話說，系統將會對資料庫進行兩次相同的稽核；一次是由伺服器原則，一次是由資料庫原則。

> [!NOTE]
> 建議**只啟用伺服器層級 Blob 稽核**，並讓所有資料庫的資料庫層級稽核保留在停用狀態。
> 您應該避免同時啟用伺服器稽核與資料庫稽核，除非：
> * 您需要為特定資料庫使用不同的儲存體帳戶或保留期間。
> * 您想要針對伺服器上不同於其餘資料庫的特定資料庫，稽核其事件類型或類別。 例如，您可能只需要針對特定資料庫稽核資料表插入。
> * 您想要使用威脅偵測，此功能目前僅支援資料庫層級稽核。

> [!IMPORTANT]
>在 Azure SQL 資料倉儲或擁有 Azure SQL 資料倉儲的伺服器上啟用稽核，**將導致資料倉儲恢復運作** (即使先前已暫停)。 **啟用稽核之後，請務必再次暫停資料倉儲**。

## <a id="subheading-5"></a>為所有資料庫設定伺服器層級稽核

伺服器稽核原則會套用至伺服器上**所有現有和新建立的資料庫**。

下節描述使用 Azure 入口網站進行稽核的設定。

1. 移至 [Azure 入口網站](https://portal.azure.com)。
2. 移至您希望稽核的 **SQL 伺服器** (請務必確保所檢視的是 SQL 伺服器，而不是特定的資料庫/DW)。 在 [安全性] 功能表中，選取 [稽核和威脅偵測]。

    ![瀏覽窗格][6]
4. 在 [稽核與威脅偵測] 刀鋒視窗的 [稽核] 中，選取 [開啟]。 此稽核原則會套用至此伺服器上所有現有和新建立的資料庫。

    ![瀏覽窗格][7]
5. 若要開啟 [稽核記錄儲存體] 刀鋒視窗，請選取 [儲存體詳細資料]。 選取或建立將儲存記錄的 Azure 儲存體帳戶，然後選取保留期間 (系統會刪除舊記錄)。 然後按一下 [確定] 。

    ![瀏覽窗格][8]

    > [!IMPORTANT]
    > 系統會將伺服器層級的稽核記錄寫入 Azure 訂用帳戶中 Azure Blob 儲存體的**附加 Blob**。
    >
    > * 附加 Blob 目前**不支援** **進階儲存體**。
    > * 目前**不支援** **VNet 中的儲存體**。

8. 按一下 [檔案] 。



## <a id="subheading-2"></a>針對單一資料庫設定資料庫層級稽核

您可以針對特定資料庫定義稽核原則，或將稽核原則定義為預設伺服器原則。

如[定義伺服器層級及資料庫層級稽核原則](#subheading-4)所述，強烈建議使用伺服器層級稽核，而非資料庫層級稽核

在設定稽核之前，請檢查您是否正在使用 [「下層用戶端」](sql-data-warehouse-auditing-downlevel-clients.md)。


1. 啟動 [Azure 入口網站](https://portal.azure.com)。
2. 移至您想要進行稽核之 SQL 資料倉儲的 [設定]。 選取 [稽核與威脅偵測]。

    ![][1]
3. 接下來，按一下 [開啟]  按鈕，來啟用稽核。

    ![][3]
4. 在 [稽核組態] 面板中，選取 [儲存體詳細資料]，以開啟 [稽核記錄儲存體] 面板。 選取記錄檔的 Azure 儲存體帳戶和保留期限。
    >[!TIP]
    >在所有稽核的資料庫中使用相同的儲存體帳戶，以充分利用預先設定的報告範本。

    ![][4]

5. 按一下 [確定]  按鈕，以儲存儲存體的詳細資料組態。
6. 在 [依事件記錄] 下，按一下 [成功] 和 [失敗] 以記錄所有事件，或選擇個別的事件類別。
7. 如果您正在為資料庫設定稽核，您可能需要變更用戶端的連接字串，以確保系統會正確擷取資料稽核。 請參閱 [修改連接字串中的伺服器 FDQN](sql-data-warehouse-auditing-downlevel-clients.md) 主題，以取得下層用戶端連線。
8. 按一下 [確定]。

## <a id="subheading-3"></a>分析稽核記錄和報告

### <a name="server-level-policy-audit-logs"></a>伺服器層級原則稽核記錄
系統會將伺服器層級的稽核記錄寫入 Azure 訂用帳戶中 Azure Blob 儲存體的**附加 Blob**。 這些稽核記錄是以 Blob 檔案集合的方式儲存在名為 **sqldbauditlogs** 的容器內。

如需有關儲存體資料夾階層、命名慣例、記錄格式的進一步詳細資訊，請參閱 [Blob 稽核記錄格式參考](https://go.microsoft.com/fwlink/?linkid=829599)。

有幾種方法可以用於檢視 Blob 稽核記錄：

* 使用 SQL Server Management Studio (SSMS 17 或更新版本) 中的 [合併稽核檔案]：
    1. 從 SSMS 功能表選取 [檔案] > [開啟] > [合併稽核檔案]。

    2. 隨即開啟 [新增稽核檔案] 對話方塊。 選取其中一個 [新增] 選項以選擇是否要從本機磁碟合併稽核檔案，或從 Azure 儲存體匯入稽核檔案。 您將需要提供您的 Azure 儲存體詳細資料和帳戶金鑰。

    3. 已新增要合併的所有檔案之後，請按一下 [確定] 以完成合併作業。

    4. 合併的檔案會在 SSMS 中開啟，您可以在其中檢視和分析該檔案，以及將其匯出至 XEL 或 CSV 檔案或資料表。

* 使用我們建立的[同步應用程式](https://github.com/Microsoft/Azure-SQL-DB-auditing-OMS-integration)。 其會在 Azure 中執行，並利用 Log Analytics 公開 API 將 SQL 稽核記錄推送至 Log Analytics。 同步應用程式會透過 Log Analytics 儀表板，將 SQL 稽核記錄推送至 Log Analytics。

* 使用 Power BI。 您可以在 Power BI 中檢視和分析稽核記錄資料。 深入了解 [Power BI，並存取可下載的範本](https://blogs.msdn.microsoft.com/azuresqldbsupport/2017/05/26/sql-azure-blob-auditing-basic-power-bi-dashboard/)。

* 透過入口網站或使用工具 (例如 [Azure 儲存體總管](http://storageexplorer.com/)) 從 Azure 儲存體 Blob 容器下載記錄檔。
    * 在您將記錄檔下載到本機之後，您可以按兩下檔案，以在 SSMS 中開啟、檢視及分析記錄。
    * 您也可以透過 Azure 儲存體總管同時下載多個檔案。 以滑鼠右鍵按一下特定子資料夾，然後選取 [另存新檔] 儲存在本機資料夾。

* 其他方法：
   * 下載多個檔案或包含記錄檔案的子資料夾後，可以在本機合併這些檔案，如稍早所述的 SSMS 合併稽核檔案指示中所述。

   * 以程式設計方式檢視 Blob 稽核記錄：

     * 使用[擴充事件讀取器](https://blogs.msdn.microsoft.com/extended_events/2011/07/20/introducing-the-extended-events-reader/) C# 程式庫。
     * 使用 PowerShell [查詢擴充事件檔案](https://sqlscope.wordpress.com/2014/11/15/reading-extended-event-files-using-client-side-tools-only/)。



<br>
### <a name="database-level-policy-audit-logs"></a>資料庫層級原則稽核記錄
在安裝期間，會在所選擇的 Azure 儲存體帳戶中具有 **SQLDBAuditLogs** 首碼的 [儲存資料表] 集合中，彙總資料庫層級稽核記錄。 您可以使用工具 (例如 [Azure 儲存體總管](http://azurestorageexplorer.codeplex.com)) 來檢視記錄檔。

預先設定的儀表板報告範本會以[可下載的 Excel 試算表](http://go.microsoft.com/fwlink/?LinkId=403540)形式提供，以協助您快速分析記錄資料。 若要在稽核記錄上使用範本，您需要 Excel 2013 (或更新版本) 和 Power Query (可從 [此處](http://www.microsoft.com/download/details.aspx?id=39379)下載)。

範本中包含虛構的範例資料，您可以設定 Power Query 直接從 Azure 儲存體帳戶匯入稽核記錄。

## <a id="subheading-4"></a>儲存體金鑰重新產生
在生產中，您可能會定期重新整理儲存體金鑰。 重新整理金鑰時，您需要儲存該原則。 程序如下：

1. 在 [稽核組態] 面板 (已在先前的設定稽核一節中說明) 中，將 [儲存體存取金鑰] 從 [主要] 變更為 [次要]，然後 [儲存]。

   ![][4]
2. 移至 [儲存體組態] 面板，並**重新產生**「主要存取金鑰」。
3. 返回 [稽核組態] 面板，
4. 將 [儲存體存取金鑰] 從 [次要] 切換回 [主要]，並按 [儲存]。
4. 返回儲存體 UI 並**重新產生**「次要存取金鑰」 (為了下一個金鑰重新整理週期做準備)。

## <a id="subheading-5"></a>自動化 (PowerShell/REST API)
您也可以使用下列自動化工具在 Azure SQL 資料倉儲中設定稽核：

* **PowerShell Cmdlet**：

   * [Get-AzureRMSqlDatabaseAuditingPolicy](/powershell/module/azurerm.sql/get-azurermsqldatabaseauditingpolicy)
   * [Get-AzureRMSqlServerAuditingPolicy](/powershell/module/azurerm.sql/Get-AzureRMSqlServerAuditingPolicy)
   * [Remove-AzureRMSqlDatabaseAuditing](/powershell/module/azurerm.sql/Remove-AzureRMSqlDatabaseAuditing)
   * [Remove-AzureRMSqlServerAuditing](/powershell/module/azurerm.sql/Remove-AzureRMSqlServerAuditing)
   * [Set-AzureRMSqlDatabaseAuditingPolicy](/powershell/module/azurerm.sql/Set-AzureRMSqlDatabaseAuditingPolicy)
   * [Set-AzureRMSqlServerAuditingPolicy](/powershell/module/azurerm.sql/Set-AzureRMSqlServerAuditingPolicy)
   * [Use-AzureRMSqlServerAuditingPolicy](/powershell/module/azurerm.sql/Use-AzureRMSqlServerAuditingPolicy)


## <a name="downlevel-clients-support-for-auditing-and-dynamic-data-masking"></a>下層用戶端支援稽核與動態資料遮罩
稽核可與支援 TDS 重新導向的 SQL 用戶端搭配使用。

實作 TDS 7.4 的任何用戶端應該也支援重新導向。 例外包括其中未完全支援重新導向功能的 JDBC 4.0，和其中未實作重新導向的 Tedious for Node.JS。

針對支援 TDS 版本 7.3 與更舊版本的「下層用戶端」，請修改連接字串中的伺服器 FQDN，如下所示：

- 連接字串中的原始伺服器 FQDN：<*伺服器名稱*>.database.windows.net
- 連接字串中已修改的伺服器 FQDN：<*伺服器名稱*>.database.**secure**.windows.net

「舊版用戶端」的部分清單包括：

* .NET 4.0 和以下版本，
* ODBC 10.0 和以下版本。
* JDBC (雖然 JDBC 支援 TDS 7.4，但並未完整支援 TDS 重新導向功能)
* Tedious (適用於 Node.JS)

**備註：** 前述伺服器 FDQN 修改可能會對於套用 SQL Server 層級稽核原則有所助益，不需要每個資料庫中的組態步驟 (暫存緩和)。     




<!--Anchors-->
[Database Auditing basics]: #subheading-1
[Set up auditing for your database]: #subheading-2
[Analyze audit logs and reports]: #subheading-3
[Define server-level vs. database-level auditing policy]: #subheading-4
[Set up server-level auditing for all databases]: #subheading-5

<!--Image references-->
[1]: ./media/sql-data-warehouse-auditing-overview/sql-data-warehouse-auditing.png
[2]: ./media/sql-data-warehouse-auditing-overview/sql-data-warehouse-auditing-inherit.png
[3]: ./media/sql-data-warehouse-auditing-overview/sql-data-warehouse-auditing-enable.png
[4]: ./media/sql-data-warehouse-auditing-overview/sql-data-warehouse-auditing-storage-account.png
[5]: ./media/sql-data-warehouse-auditing-overview/sql-data-warehouse-auditing-dashboard.png
[6]: ./media/sql-data-warehouse-auditing-overview/sql-data-warehouse-auditing-server_1_overview.png
[7]: ./media/sql-data-warehouse-auditing-overview/sql-data-warehouse-auditing-server_2_enable.png
[8]: ./media/sql-data-warehouse-auditing-overview/sql-data-warehouse-auditing-server_3_storage.png
