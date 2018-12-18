---
title: 保護 Azure SQL Database | Microsoft Docs
description: 深入了解保護 Azure SQL Database 的技術和功能。
services: sql-database
ms.service: sql-database
ms.subservice: security
ms.custom: ''
ms.devlang: ''
ms.topic: tutorial
author: DRediske
ms.author: daredis
ms.reviewer: vanto, carlrab
manager: craigg
ms.date: 09/07/2018
ms.openlocfilehash: ceed69503900b38d7f6a29bbe116ab9a4d54e396
ms.sourcegitcommit: 67abaa44871ab98770b22b29d899ff2f396bdae3
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/08/2018
ms.locfileid: "48857953"
---
# <a name="secure-your-azure-sql-database"></a>保護 Azure SQL Database

SQL Database 保護您資料的方式： 
- 使用防火牆規則限制資料庫的存取權 
- 使用需要其身分識別的驗證機制
- 透過以角色為基礎的成員資格和權限進行資料授權 
- 資料列層級安全性
- 動態資料遮罩

SQL Database 也具有複雜的監視、稽核和威脅偵測功能。 

只需要幾個簡單步驟，您就可以讓資料庫預防惡意使用者或未經授權的存取。 您會在本教學課程中學到： 

> [!div class="checklist"]
> * 在 Azure 入口網站中，為伺服器設定伺服器層級的防火牆規則
> * 使用 SSMS 為資料庫設定資料庫層級的防火牆規則
> * 使用安全的連接字串來與資料庫連線
> * 管理使用者的存取
> * 使用加密來保護您的資料
> * 啟用 SQL Database 稽核
> * 啟用 SQL Database 威脅偵測

如果您沒有 Azure 訂用帳戶，請在開始之前先[建立免費帳戶](https://azure.microsoft.com/free/)。

## <a name="prerequisites"></a>必要條件

若要完成本教學課程，請確定您具有下列項目︰

- 已安裝最新版的 [SQL Server Management Studio](https://docs.microsoft.com/sql/ssms/download-sql-server-management-studio-ssms) (SSMS)。 
- 已安裝 Microsoft Excel
- 已建立 Azure SQL Server 和 SQL Database - 請參閱[在 Azure 入口網站中建立 Azure SQL Database](sql-database-get-started-portal.md)、[使用 Azure CLI 建立單一 Azure SQL Database](sql-database-cli-samples.md) 和[使用 PowerShell 建立單一 Azure SQL Database](sql-database-powershell-samples.md)。 

## <a name="log-in-to-the-azure-portal"></a>登入 Azure 入口網站

登入 [Azure 入口網站](https://portal.azure.com/)。

## <a name="create-a-server-level-firewall-rule-in-the-azure-portal"></a>在 Azure 入口網站中建立伺服器層級的防火牆規則

Azure 的防火牆會保護 SQL Database。 依預設，伺服器與其內部資料庫的所有連線皆會遭拒，除非是來自其他 Azure 服務的連線。 如需詳細資訊，請參閱 [Azure SQL Database 伺服器層級和資料庫層級防火牆規則](sql-database-firewall-configure.md)

最安全的設定是將「允許存取 Azure 服務」設定為「關閉」。 如果您需要從 Azure VM 或雲端服務連線到資料庫，您應建立[保留的 IP](../virtual-network/virtual-networks-reserved-public-ip.md)，並僅允許保留的 IP 位址可通過防火牆進行存取。 

遵循以下步驟建立 [SQL Database 伺服器層級防火牆規則](sql-database-firewall-configure.md)，以讓您的伺服器允許來自特定 IP 位址的連線。 

> [!NOTE]
> 如果您已使用先前其中一個教學課程或快速入門在 Azure 中建立範例資料庫，而且您執行此教學課程的電腦與逐步進行那些教學課程時的電腦有相同的 IP 位址，表示您已建立伺服器層級的防火牆規則，所以您就可以略過此步驟。
>

1. 按一下左側功能表上的 [SQL Database]，然後在 [SQL Database] 頁面上按一下您要設定防火牆規則的資料庫。 資料庫的概觀頁面隨即開啟，其中會顯示完整伺服器名稱 (例如 **mynewserver20170313.database.windows.net**)，並提供進一步的組態選項。

      ![伺服器防火牆規則](./media/sql-database-security-tutorial/server-firewall-rule.png) 

2. 如先前映像所示，按一下工具列上的 [設定伺服器防火牆]。 SQL Database 伺服器的 [防火牆設定] 頁面隨即開啟。 

3. 按一下工具列上的 [新增用戶端 IP]，新增與入口網站連線的電腦公用 IP 位址，或手動輸入防火牆規則並按一下 [儲存]。

      ![設定伺服器防火牆規則](./media/sql-database-security-tutorial/server-firewall-rule-set.png) 

4. 依序按一下 [確定] 和 [X] 以關閉 [防火牆設定] 頁面。

您現在可以連線到資料庫中任何具有指定 IP 位址或 IP 位址範圍的伺服器。

> [!NOTE]
> SQL Database 會透過連接埠 1433 通訊。 如果您嘗試從公司網路進行連線，您網路的防火牆可能不允許透過連接埠 1433 的輸出流量。 若是如此，除非 IT 部門開啟連接埠 1433，否則將無法連線至 Azure SQL Database 伺服器。
>

## <a name="create-a-database-level-firewall-rule-using-ssms"></a>使用 SSMS 建立資料庫層級防火牆規則

資料庫層級防火牆規則可讓您在相同邏輯伺服器內為不同資料庫建立不同的防火牆設定，以及建立具有可攜性的防火牆規則 - 表示這些規則在[容錯移轉](sql-database-geo-replication-overview.md)期間是跟隨著資料庫而不是儲存在 SQL Server 中。 資料庫層級的防火牆規則只能使用 Transact-SQL 陳述式設定，且僅可在您已設定第一個伺服器層級防火牆規則之後進行設定。 如需詳細資訊，請參閱 [Azure SQL Database 伺服器層級和資料庫層級防火牆規則](sql-database-firewall-configure.md)

請依照下列步驟來建立資料庫專屬的防火牆規則。

1. 連線至您的資料庫，例如使用 [SQL Server Management Studio](./sql-database-connect-query-ssms.md)。

2. 在物件總管中，以滑鼠右鍵按一下您要新增防火牆規則的資料庫，然後按一下[新增查詢]。 隨即開啟已連線到您資料庫的空白查詢視窗。

3. 在查詢視窗中，將 IP 位址修改為您的公用 IP 位址，然後執行下列查詢：

    ```sql
    EXECUTE sp_set_database_firewall_rule N'Example DB Rule','0.0.0.4','0.0.0.4';
    ```

4. 在工具列上按一下 [執行]以建立防火牆規則。

## <a name="view-how-to-connect-an-application-to-your-database-using-a-secure-connection-string"></a>檢視如何使用安全的連接字串讓應用程式與資料庫連線

若要確保用戶端應用程式與 SQL Database 之間有安全且加密的連線，連接字串必須設定為：

- 要求加密的連線，並且
- 不要信任伺服器憑證。 

此動作會建立使用傳輸層安全性 (TLS) 的連線，並降低發生攔截式攻擊的風險。 從 Azure 入口網站，您可以取得受支援用戶端驅動程式的 SQL Database 連接字串 (已正確設定)，如同此螢幕截圖中的 ADO.net 所示。 如需 TLS 和連線能力的相關資訊，請參閱 [TLS 考量](sql-database-connect-query.md#tls-considerations-for-sql-database-connectivity)。

1. 從左側功能表中選取 [SQL Database]，按一下 [SQL Database] 頁面上您的資料庫。

2. 在您資料庫的 [概觀]頁面上，按一下 [顯示資料庫連接字串]。

3. 檢閱完整的 **ADO.NET** 連接字串。

    ![ADO.NET 連接字串](./media/sql-database-security-tutorial/adonet-connection-string.png)

## <a name="creating-database-users"></a>建立資料庫使用者

建立任何使用者之前，您必須先選取一個或兩個 Azure SQL Database 支援的驗證類型： 

**SQL 驗證**，使用登入作業和使用者適用的使用者名稱和密碼，而登入作業和使用者僅在邏輯伺服器內的特定資料庫環境中有效。 

**Azure Active Directory 驗證**，使用由 Azure Active Directory 管理的身分識別。 

如果您想要使用 [Azure Active Directory](./sql-database-aad-authentication.md) 對 SQL Database 進行驗證，則必須有填入的 Azure Active Directory 才可繼續。

請遵循以下步驟使用 SQL Authentication 建立使用者：

1. 使用 [SQL Server Management Studio](./sql-database-connect-query-ssms.md) 等方式以使用您的伺服器系統管理員認證與資料庫連線。

2. 在物件總管中，以滑鼠右鍵按一下您要新增使用者的資料庫，然後按一下[新增查詢]。 隨即開啟已連線到所選資料庫的空白查詢視窗。

3. 在查詢視窗中，輸入下列查詢︰

    ```sql
    CREATE USER ApplicationUser WITH PASSWORD = 'YourStrongPassword1';
    ```

4. 在工具列上按一下 [執行] 以建立使用者。

5. 依預設，使用者可與資料庫連線，但沒有權限可讀取或寫入資料。 若要將這些權限授與新建立的使用者，請在新的查詢視窗中執行下列兩個命令

    ```sql
    ALTER ROLE db_datareader ADD MEMBER ApplicationUser;
    ALTER ROLE db_datawriter ADD MEMBER ApplicationUser;
    ```

若要在資料庫層級建立這些非系統管理員帳戶來與您資料庫連線，這是最佳的作法，除非您需要執行如建立新使用者的系統管理員工作。 請檢閱 [Azure Active Directory 教學課程](./sql-database-aad-authentication-configure.md)以了解如何使用 Azure Active Directory 進行驗證。


## <a name="protect-your-data-with-encryption"></a>使用加密來保護您的資料

Azure SQL Database 透明資料加密 (TDE) 會自動為您的待用資料加密，且應用程式不須做任何變更來存取加密的資料庫。 如果是新建立的資料庫，TDE 預設為開啟。 若要為您的資料庫啟用 TDE，或確認已開啟 TDE，請遵循下列步驟：

1. 從左側功能表中選取 [SQL Database]，按一下 [SQL Database] 頁面上您的資料庫。 

2. 按一下 [透明資料加密] 以開啟 TDE 的組態頁面。

    ![透明資料加密](./media/sql-database-security-tutorial/transparent-data-encryption-enabled.png)

3. 如有必要，請將 [資料加密] 設為「開啟」，然後按一下 [儲存]。

加密程序會在背景中啟動。 使用 [SQL Server Management Studio](./sql-database-connect-query-ssms.md) 連線至 SQL Database，以及查詢 [sys.dm_database_encryption_keys](https://docs.microsoft.com/sql/relational-databases/system-dynamic-management-views/sys-dm-database-encryption-keys-transact-sql?view=sql-server-2017) 檢視的 encryption_state 資料欄，即可監視進度。 狀態 3 表示資料庫已加密。 

## <a name="enable-sql-database-auditing-if-necessary"></a>啟用 SQL Database 稽核 (如有必要)

Azure SQL Database 稽核會追蹤資料庫事件並將事件寫入您 Azure 儲存體帳戶中的稽核記錄。 稽核可協助您保持法規合規性、了解資料庫活動，以及深入了解可能指出潛在安全違規的不一致和異常。 請遵循下列步驟為 SQL Database 建立預設的稽核原則︰

1. 從左側功能表中選取 [SQL Database]，按一下 [SQL Database] 頁面上您的資料庫。 

2. 在 [設定] 刀鋒視窗中，選取 [稽核和威脅偵測]。 請注意，伺服器層級稽核已停用，而且有一個**檢視伺服器設定**連結，可讓您從此內容檢視或修改伺服器稽核設定。

    ![稽核刀鋒視窗](./media/sql-database-security-tutorial/auditing-get-started-settings.png)

3. 如果您想要啟用的稽核類型 (或位置？) 與伺服器層級所指定的類型不同，請**開啟**稽核，並選擇 [Blob] 稽核類型。 如果已啟用伺服器 Blob 稽核，資料庫設定的稽核將會與伺服器 Blob 稽核並存。

    ![開啟稽核](./media/sql-database-security-tutorial/auditing-get-started-turn-on.png)

4. 選取 [儲存體詳細資料] 以開啟 [稽核記錄儲存體] 刀鋒視窗。 選取將儲存記錄的 Azure 儲存體帳戶，以及將舊記錄刪除之前的保留期間，然後按一下底部的 [確定]。 

   > [!TIP]
   > 讓所有稽核的資料庫都使用相同的儲存體帳戶，以充分利用稽核報告範本。
   > 

5. 按一下 [檔案] 。

> [!IMPORTANT]
> 如果您想要自訂稽核事件，您可以透過 PowerShell 或 REST API 來自訂- 詳細資訊請參閱 [SQL 資料庫稽核](sql-database-auditing.md)。
>

## <a name="enable-sql-database-threat-detection"></a>啟用 SQL Database 威脅偵測

威脅偵測提供新的一層安全性，在發生異常活動時會提供安全性警示，讓客戶偵測並回應潛在威脅。 使用者可以使用 SQL Database 稽核來探索可疑的事件，以判斷事件的原因是否是有人嘗試存取、破壞或利用資料庫中的資料。 您不必是安全性專家，也不需要管理進階的安全性監視系統，威脅偵測讓您輕鬆解決資料庫的潛在威脅。
威脅偵測會偵測異常資料庫活動，指出潛在的 SQL 插入式攻擊。 SQL 插入式攻擊是網際網路上常見的 Web 應用程式安全性問題之一，用於攻擊資料導向應用程式。 攻擊者利用應用程式弱點將惡意的 SQL 陳述式插入應用程式輸入欄位，以破壞或修改資料庫中的資料。

1. 瀏覽至您要監視的 SQL Database 組態刀鋒視窗。 在 [設定] 刀鋒視窗中，選取 [稽核和威脅偵測]。

    ![瀏覽窗格](./media/sql-database-security-tutorial/auditing-get-started-settings.png)
2. 在 [稽核與威脅偵測] 組態刀鋒視窗中，[開啟] 稽核，這會顯示威脅偵測設定。

3. [開啟] 威脅偵測。

4. 設定在偵測到異常資料庫活動時將收到安全性警示的電子郵件清單。

5. 按一下 [稽核與威脅偵測] 刀鋒視窗中的 [儲存]，以儲存新的或已更新的稽核與威脅偵測原則。

    ![瀏覽窗格](./media/sql-database-security-tutorial/td-turn-on-threat-detection.png)

    如果偵測到異常的資料庫活動，您會在一偵測到異常的資料庫活動時，就收到電子郵件通知。 電子郵件將提供可疑安全性事件的相關資訊，包括異常活動的性質、資料庫名稱、伺服器名稱和事件時間。 此外，還會提供可能原因和建議動作的相關資訊，以協助您調查和減輕資料庫的潛在威脅。 以下步驟會逐步說明收到以下類型的電子郵件時，您該怎麼做：

    ![威脅偵測電子郵件](./media/sql-database-threat-detection-get-started/4_td_email.png)

6. 在電子郵件中，按一下 [Azure SQL 稽核記錄] 連結會啟動 Azure 入口網站，並顯示可疑事件發生時間前後的相關稽核記錄。

    ![稽核記錄](./media/sql-database-threat-detection-get-started/5_td_audit_records.png)

7. 按一下稽核記錄以檢視可疑資料庫活動的詳細資訊，例如 SQL 陳述式、失敗原因和用戶端的 IP。

    ![記錄詳細資料](./media/sql-database-security-tutorial/6_td_audit_record_details.png)

8. 在 [稽核記錄] 刀鋒視窗中，按一下 [在 Excel 中開啟] 開啟預先設定的 Excel 範本，以匯入可疑事件前後的稽核記錄，執行更深入的分析。

    > [!NOTE]
    > 在 Excel 2010 或更新版本中，必須要有 Power Query 和 [快速合併] 設定。

    ![在 Excel 中開啟記錄](./media/sql-database-threat-detection-get-started/7_td_audit_records_open_excel.png)

9. 若要設定 [快速合併] 設定 - 在 [POWER QUERY] 功能區索引標籤中，選取 [選項] 以顯示 [選項] 對話方塊。 選取 [隱私權] 區段，選擇第二個選項 - [忽略隱私權等級並可能改善效能]：

    ![Excel 快速合併](./media/sql-database-threat-detection-get-started/8_td_excel_fast_combine.png)

10. 若要載入 SQL 稽核記錄檔，請確定 [設定] 索引標籤中的參數已正確設定，然後選取 [資料] 功能區，並按一下 [全部重新整理] 按鈕。

    ![Excel 參數](./media/sql-database-threat-detection-get-started/9_td_excel_parameters.png)

11. 結果會出現在 [SQL 稽核記錄檔]  工作表，可讓您對偵測到的異常活動執行更深入的分析，減輕應用程式中的安全性事件造成的影響。


## <a name="next-steps"></a>後續步驟
在本教學課程中，您已了解只需要幾個簡單步驟，就可以讓資料庫預防惡意使用者或未經授權的存取。  您已了解如何︰ 

> [!div class="checklist"]
> * 設定伺服器和資料庫的防火牆規則
> * 使用安全的連接字串來與資料庫連線
> * 管理使用者的存取
> * 使用加密來保護您的資料
> * 啟用 SQL Database 稽核
> * 啟用 SQL Database 威脅偵測

前進至下一個教學課程，以了解如何實作異地分散式資料庫。

> [!div class="nextstepaction"]
>[實作異地分散式資料庫](sql-database-implement-geo-distributed-database.md)

