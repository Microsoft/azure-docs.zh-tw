---
title: 開始使用 Azure SQL 資料庫稽核 | Microsoft Docs
description: 使用 Azure SQL Database 稽核來將資料庫事件追蹤至稽核記錄。
services: sql-database
ms.service: sql-database
ms.subservice: security
ms.custom: ''
ms.devlang: ''
ms.topic: conceptual
author: giladmit
ms.author: giladm
ms.reviewer: vanto
manager: craigg
ms.date: 09/10/2018
ms.openlocfilehash: 8ba07b22d247cb9263890a747bd166d63af27e3b
ms.sourcegitcommit: d1aef670b97061507dc1343450211a2042b01641
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/27/2018
ms.locfileid: "47395742"
---
# <a name="get-started-with-sql-database-auditing"></a>開始使用 SQL Database 稽核
Azure SQL 資料庫稽核會追蹤資料庫事件，並將事件寫入您 Azure 儲存體帳戶中的稽核記錄。 稽核也具備下列功能：

* 協助您保持法規遵循、了解資料庫活動，以及深入了解可指出商務考量或疑似安全違規的不一致和異常。

* 啟用及推動遵循法規標準，但不保證符合法規。 如需有關支援標準法規的 Azure 程式詳細資訊，請參閱 [Azure 信任中心](https://azure.microsoft.com/support/trust-center/compliance/)。


## <a id="subheading-1"></a>Azure SQL 資料庫稽核概觀
您可以使用 SQL 資料庫稽核完成下列工作：


* **保留** 所選事件的稽核記錄。 您可以定義要稽核的資料庫動作類別。
* **報告** 資料庫活動。 您可以使用預先設定的報告和儀表板，以便快速開始使用活動和事件報告。
* **分析** 報告。 您可以尋找可疑事件、異常活動及趨勢。

您可以依照[設定資料庫的稽核](#subheading-2)一節中的說明，針對不同類型的事件類別設定稽核。

> [!IMPORTANT]
> 系統會將稽核記錄寫入 Azure 訂用帳戶中 Azure Blob 儲存體的**附加 Blob**。
>
> * 附加 Blob 目前**不支援****進階儲存體**。
> * 目前**不支援** **VNet 中的儲存體**。

## <a id="subheading-8"></a>定義伺服器層級與資料庫層級的稽核原則

您可以針對特定資料庫定義稽核原則，或將稽核原則定義為預設伺服器原則：

* 伺服器原則會套用至伺服器上所有現有和新建立的資料庫。

* 如果「伺服器 Blob 稽核已啟用」，它「一律」會套用到資料庫。 不論資料庫稽核資料是什麼，都會稽核資料庫。

* 如果在伺服器和資料庫上都啟用 Blob 稽核，這「不會」覆寫或變更伺服器 Blob 稽核的任何設定。 這兩種稽核將會並存。 換句話說，系統將會對資料庫進行兩次相同的稽核；一次是由伺服器原則，一次是由資料庫原則。

   > [!NOTE]
   > 您應該避免同時啟用伺服器 Blob 稽核與資料庫 Blob 稽核，除非：
    > * 您需要為特定資料庫使用不同的儲存體帳戶或保留期間。
    > * 您想要針對伺服器上不同於其餘資料庫的特定資料庫，稽核其事件類型或類別。 例如，您可能只需要針對特定資料庫稽核資料表插入。
   >
   > 否則，建議只啟用伺服器層級 Blob 稽核，並讓所有資料庫的資料庫層級稽核保留在停用狀態。


## <a id="subheading-2"></a>設定資料庫的稽核
下節描述使用 Azure 入口網站進行稽核的設定。

1. 移至 [Azure 入口網站](https://portal.azure.com)。
2. 瀏覽至 SQL 資料庫/伺服器窗格中 [安全性] 標題下的 [稽核]。

    <a id="auditing-screenshot"></a> ![導覽窗格][1]

3. 如果您想要設定伺服器稽核原則，可以選取資料庫稽核頁面上的 [檢視伺服器設定] 連結。 然後，您可以檢視或修改伺服器稽核設定。 伺服器稽核原則會套用至此伺服器上所有現有和新建立的資料庫。

    ![瀏覽窗格][2]

4. 如果您偏向在資料庫層級啟用稽核，請將 [稽核] 切換到 [開啟]。

    如果已啟用伺服器稽核，資料庫設定的稽核將會與伺服器稽核並存。

    ![瀏覽窗格][3]

5. **新增** - 您現在有多個選項可設定要寫入稽核記錄的位置。 您可以將記錄寫入至 Azure 儲存體帳戶、OMS 工作區 (以供 Log Analytics 取用) 或事件中樞 (以供事件中樞取用)。 您可以設定這些選項的任何組合，並將稽核記錄寫入至每個組合。

    ![儲存體選項](./media/sql-database-auditing-get-started/auditing-select-destination.png)

6. 若要設定將稽核記錄寫入至儲存體帳戶，請選取 [儲存體]，然後開啟 [儲存體詳細資料]。 選取將儲存記錄的 Azure 儲存體帳戶，然後選取保留期間。 舊的記錄將被刪除。 然後按一下 [確定] 。

    ![storage account](./media/sql-database-auditing-get-started/auditing_select_storage.png)

7. 若要設定將稽核記錄寫入至 OMS 工作區，請選取 [Log Analytics (預覽)]，然後開啟 [Log Analytics 詳細資料]。 選取或建立要寫入記錄的 OMS 工作區，然後按一下 [確定]。

    ![OMS](./media/sql-database-auditing-get-started/auditing_select_oms.png)

8. 若要設定將稽核記錄寫入至事件中樞，請選取 [事件中樞 (預覽)]，然後開啟 [事件中樞詳細資料]。 選取要寫入記錄的事件中樞，然後按一下 [確定]。 請確定事件中樞與您的資料庫和伺服器位於相同的區域。

    ![事件中樞](./media/sql-database-auditing-get-started/auditing_select_event_hub.png)

9. 按一下 [檔案] 。
10. 如果您想要自訂稽核的事件，您可以透過 [PowerShell Cmdlet](#subheading-7) 或 [REST API](#subheading-9) 來自訂。
11. 設定您的稽核設定之後，您可以開啟新的威脅偵測功能，並設定電子郵件以接收安全性警示。 使用威脅偵測時，您會接收與指示潛在安全性威脅的異常資料庫活動相關的主動式警示。 如需詳細資訊，請參閱[開始使用威脅偵測](sql-database-threat-detection-get-started.md)。 

## <a id="subheading-3"></a>分析稽核記錄和報告
如果您選擇將稽核記錄寫入至 Log Analytics：
- 使用 [Azure 入口網站](https://portal.azure.com)。  開啟相關的資料庫。 在資料庫的 [稽核] 頁面頂端，按一下 [檢視稽核記錄]。

    ![檢視稽核記錄](./media/sql-database-auditing-get-started/7_auditing_get_started_blob_view_audit_logs.png)

- 然後，在 [稽核記錄] 頁面頂端，按一下 [在 OMS 中開啟] 以在 Log Analytics 中開啟記錄檢視，您可以在其中自訂時間範圍和搜尋查詢。

    ![在 OMS 中開啟](./media/sql-database-auditing-get-started/auditing_open_in_oms.png)

- 或者，您也可以從 Log Analytics 刀鋒視窗存取稽核記錄。 開啟 Log Analytics 工作區，然後在 [一般] 區段下，按一下 [記錄]。 您可以從簡單的查詢開始，例如：「搜尋 "SQLSecurityAuditEvents"」以檢視稽核記錄。
    從這裡，您也可以使用 [Operations Management Suite (OMS) Log Analytics](../log-analytics/log-analytics-log-search.md) 對您的稽核記錄資料執行進階搜尋。 Log Analytics 可讓您在所有工作負載和伺服器之間，使用整合式搜尋和自訂儀表板輕易地分析數百萬筆記錄，提供您即時的深入操作資訊。 如需 OMS Log Analytics 搜尋語言和命令的其他實用資訊，請參閱 [Log Analytics 搜尋參考](../log-analytics/log-analytics-log-search.md)。

如果您選擇將稽核記錄寫入至事件中樞：
- 若要取用來自事件中樞的稽核記錄資料，您必須設定資料流取用事件並將其寫入至目標。 如需詳細資訊，請參閱 [Azure 事件中樞文件](https://docs.microsoft.com/azure/event-hubs/)。

如果您選擇將稽核記錄寫入至 Azure 儲存體帳戶，您可使用數種方法來檢視記錄：
- 稽核記錄會在您於設定期間選擇的帳戶中彙總。 您可以使用工具 (例如 [Azure 儲存體總管](http://storageexplorer.com/)) 來查看稽核記錄。 在 Azure 儲存體中，稽核記錄是以 Blob 檔案集合的方式儲存在名為 **sqldbauditlogs** 的容器內。 如需有關儲存體資料夾階層、命名慣例、記錄格式的進一步詳細資訊，請參閱 [Blob 稽核記錄格式參考](https://go.microsoft.com/fwlink/?linkid=829599)。

- 使用 [Azure 入口網站](https://portal.azure.com)。  開啟相關的資料庫。 在資料庫的 [稽核] 頁面頂端，按一下 [檢視稽核記錄]。

    ![瀏覽窗格][7]

    隨即開啟 [稽核記錄]，您可以在其中檢視記錄。

    - 您可以按一下 [稽核記錄] 頁面頂端的 [篩選] 來檢視特定日期。
    - 切換 [稽核來源]，即可在由「伺服器稽核原則」和「資料庫稽核原則」建立的稽核記錄之間切換。
    - 如果勾選 [只顯示 SQL 插入的稽核記錄] 核取方塊，只可以檢視 SQL 插入相關的稽核記錄。

       ![瀏覽窗格][8]

- 使用系統函數 **sys.fn_get_audit_file** (T-SQL) 以表格格式傳回稽核記錄資料。 如需使用此函式的詳細資訊，請參閱 [sys.fn_get_audit_file](https://docs.microsoft.com/sql/relational-databases/system-functions/sys-fn-get-audit-file-transact-sql)。


- 使用 SQL Server Management Studio (SSMS 17 或更新版本) 中的 [合併稽核檔案]：
    1. 從 SSMS 功能表選取 [檔案] > [開啟] > [合併稽核檔案]。

        ![瀏覽窗格][9]
    2. 隨即開啟 [新增稽核檔案] 對話方塊。 選取其中一個 [新增] 選項以選擇是否要從本機磁碟合併稽核檔案，或從 Azure 儲存體匯入稽核檔案。 您將需要提供您的 Azure 儲存體詳細資料和帳戶金鑰。

    3. 已新增要合併的所有檔案之後，請按一下 [確定] 以完成合併作業。

    4. 合併的檔案會在 SSMS 中開啟，您可以在其中檢視和分析該檔案，以及將其匯出至 XEL 或 CSV 檔案，或是匯出至資料表。

- 使用 Power BI。 您可以在 Power BI 中檢視和分析稽核記錄資料。 如需詳細資訊，以及若要存取可下載的範本，請參閱 [Analyzie audit log data in Power BI](https://blogs.msdn.microsoft.com/azuresqldbsupport/2017/05/26/sql-azure-blob-auditing-basic-power-bi-dashboard/) (在 Power BI 中分析稽核記錄資料)。
- 透過入口網站或使用工具 (例如 [Azure 儲存體總管](http://storageexplorer.com/)) 從 Azure 儲存體 Blob 容器下載記錄檔。
    * 在您將記錄檔下載到本機之後，按兩下檔案，以在 SSMS 中開啟、檢視及分析記錄。
    * 您也可以透過 Azure 儲存體總管同時下載多個檔案。 若要執行這項作業，請以滑鼠右鍵按一下特定子資料夾，然後選取 [另存新檔] 儲存在本機資料夾。

* 其他方法：
   * 下載多個檔案或包含記錄檔的子資料夾後，可以在本機合併這些檔案，如先前所述的 SSMS 合併稽核檔案指示中所述。
   * 以程式設計方式檢視 Blob 稽核記錄：

     * 使用[擴充事件讀取器](https://blogs.msdn.microsoft.com/extended_events/2011/07/20/introducing-the-extended-events-reader/) C# 程式庫。
     * 使用 PowerShell [查詢擴充事件檔案](https://sqlscope.wordpress.com/2014/11/15/reading-extended-event-files-using-client-side-tools-only/)。

## <a id="subheading-5"></a>實際作法
<!--The description in this section refers to preceding screen captures.-->

### <a id="subheading-6">稽核異地複寫資料庫</a>
使用異地複寫資料庫，當您在主要資料庫啟用稽核，次要資料庫會有相同的稽核原則。 也可以在**次要伺服器**上啟用稽核，設定次要資料庫稽核，和主要資料庫分開。

* 伺服器層級 (**建議**)：啟動**主要伺服器**和**次要伺服器**上的稽核 - 將根據其個別的伺服器層級原則對主要和次要資料庫分開進行稽核。

* 資料庫層級：只能從主要資料庫稽核設定來設定次要資料庫的資料庫層級稽核。
   * 必須在「主要資料庫本身」 (而不是在伺服器上) 啟用稽核。
   * 在主要資料庫上啟用稽核之後，它也會在次要資料庫上變成啟用狀態。

    >[!IMPORTANT]
    >使用資料庫層級稽核時，次要資料庫的儲存體設定將會和主要資料庫上的設定完全相同，這會導致跨地區流量。 建議您只啟用伺服器層級稽核，並讓所有資料庫的資料庫層級稽核保留在停用狀態。
<br>

### <a id="subheading-6">儲存體金鑰重新產生</a>
在生產中，您可能會定期重新整理儲存體金鑰。 當您將稽核記錄寫入至 Azure 儲存體時，您需要在重新整理金鑰期間重新儲存稽核原則。 程序如下：

1. 開啟 [儲存體詳細資料]。 在 [儲存體存取金鑰] 方塊中，選取 [次要]，然後按一下 [確定]。 然後按一下稽核設定頁面頂端的 [儲存]。

    ![瀏覽窗格][5]
2. 移至儲存體設定頁面，並重新產生主要存取金鑰。

    ![瀏覽窗格][6]
3. 返回稽核設定頁面，將儲存體存取金鑰從次要切換成主要，然後按一下 [確定]。 然後按一下稽核設定頁面頂端的 [儲存]。
4. 返回儲存體設定頁面，並重新產生次要存取金鑰 (為下一個金鑰重新整理週期做準備)。

## <a name="additional-information"></a>其他資訊

* 如需有關記錄格式、儲存體資料夾階層和命名慣例的詳細資訊，請參閱 [Blob 稽核記錄格式參考](https://go.microsoft.com/fwlink/?linkid=829599)。

    > [!IMPORTANT]
    > Azure SQL Database 稽核會在稽核記錄中的字元欄位儲存 4000 個字元的資料。 當**陳述式**或從可稽核的動作傳回的 **data_sensitivity_information** 值包含超過 4000 個字元，超過前 4000 個字元的任何資料將會**截斷且不會稽核**。

* 系統會將稽核記錄寫入 Azure 訂用帳戶中 Azure Blob 儲存體的**附加 Blob**：
    * 附加 Blob 目前**不支援** **進階儲存體**。
    * 目前**不支援** **VNet 中的儲存體**。

* 預設稽核原則包含所有動作和下列一組動作群組，這會稽核對資料庫執行的所有查詢和預存程序，以及成功和失敗的登入：

    BATCH_COMPLETED_GROUP<br>
    SUCCESSFUL_DATABASE_AUTHENTICATION_GROUP<br>
    FAILED_DATABASE_AUTHENTICATION_GROUP

    您可以使用 PowerShell 設定不同動作和動作群組類型的稽核，如[使用 Azure PowerShell 管理 SQL 資料庫稽核](#subheading-7)一節中所述。

## <a id="subheading-7"></a>使用 Azure PowerShell 管理 SQL 資料庫稽核

**PowerShell Cmdlet**：

* [建立或更新資料庫 Blob 稽核原則 (Set-AzureRMSqlDatabaseAuditing)][105]
* [建立或更新伺服器 Blob 稽核原則 (Set-AzureRMSqlServerAuditing)][106]
* [取得資料庫稽核原則 (Get-AzureRMSqlDatabaseAuditing)][101]
* [取得伺服器 Blob 稽核原則 (Get-AzureRMSqlServerAuditing)][102]

如需指令碼範例，請參閱[使用 PowerShell 設定稽核與威脅偵測](scripts/sql-database-auditing-and-threat-detection-powershell.md)。

## <a id="subheading-9"></a>使用 REST API 管理 SQL 資料庫稽核

**REST API - Blob 稽核**：

* [建立或更新資料庫 Blob 稽核原則](https://docs.microsoft.com/rest/api/sql/database%20auditing%20settings/createorupdate)
* [建立或更新伺服器 Blob 稽核原則](https://docs.microsoft.com/rest/api/sql/server%20auditing%20settings/createorupdate)
* [取得資料庫 Blob 稽核原則](https://docs.microsoft.com/rest/api/sql/database%20auditing%20settings/get)
* [取得伺服器 Blob 稽核原則](https://docs.microsoft.com/rest/api/sql/server%20auditing%20settings/get)

具有 WHERE 子句而可支援其他篩選的擴充原則：
* [建立或更新資料庫「擴充的」Blob 稽核原則](https://docs.microsoft.com/rest/api/sql/database%20extended%20auditing%20settings/createorupdate)
* [建立或更新伺服器「擴充的」Blob 稽核原則](https://docs.microsoft.com/rest/api/sql/server%20extended%20auditing%20settings/createorupdate)
* [取得資料庫「擴充的」Blob 稽核原則](https://docs.microsoft.com/rest/api/sql/database%20extended%20auditing%20settings/get)
* [取得伺服器「擴充的」Blob 稽核原則](https://docs.microsoft.com/rest/api/sql/server%20extended%20auditing%20settings/get)

<!--Anchors-->
[Azure SQL Database Auditing overview]: #subheading-1
[Set up auditing for your database]: #subheading-2
[Analyze audit logs and reports]: #subheading-3
[Practices for usage in production]: #subheading-5
[Storage Key Regeneration]: #subheading-6
[Manage SQL database auditing using Azure PowerShell]: #subheading-7
[Blob/Table differences in Server auditing policy inheritance]: (#subheading-8)
[Manage SQL database auditing using REST API]: #subheading-9

<!--Image references-->
[1]: ./media/sql-database-auditing-get-started/1_auditing_get_started_settings.png
[2]: ./media/sql-database-auditing-get-started/2_auditing_get_started_server_inherit.png
[3]: ./media/sql-database-auditing-get-started/3_auditing_get_started_turn_on.png
[4]: ./media/sql-database-auditing-get-started/4_auditing_get_started_storage_details.png
[5]: ./media/sql-database-auditing-get-started/5_auditing_get_started_storage_key_regeneration.png
[6]: ./media/sql-database-auditing-get-started/6_auditing_get_started_regenerate_key.png
[7]: ./media/sql-database-auditing-get-started/7_auditing_get_started_blob_view_audit_logs.png
[8]: ./media/sql-database-auditing-get-started/8_auditing_get_started_blob_audit_records.png
[9]: ./media/sql-database-auditing-get-started/9_auditing_get_started_ssms_1.png
[10]: ./media/sql-database-auditing-get-started/10_auditing_get_started_ssms_2.png

[101]: /powershell/module/azurerm.sql/get-azurermsqldatabaseauditing
[102]: /powershell/module/azurerm.sql/Get-AzureRMSqlServerAuditing
[103]: /powershell/module/azurerm.sql/Remove-AzureRMSqlDatabaseAuditing
[104]: /powershell/module/azurerm.sql/Remove-AzureRMSqlServerAuditing
[105]: /powershell/module/azurerm.sql/Set-AzureRMSqlDatabaseAuditing
[106]: /powershell/module/azurerm.sql/Set-AzureRMSqlServerAuditing
