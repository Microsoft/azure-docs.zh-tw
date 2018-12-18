---
title: 設定 Azure SQL 資料同步 | Microsoft Docs
description: 本教學課程說明如何設定 Azure SQL 資料同步
services: sql-database
author: allenwux
manager: craigg
ms.service: sql-database
ms.custom: load & move data
ms.topic: conceptual
ms.date: 04/10/2018
ms.author: xiwu
ms.reviewer: douglasl
ms.openlocfilehash: 7cb7ce0ea81cf99c2ee14a6f209864894563d237
ms.sourcegitcommit: 5a7f13ac706264a45538f6baeb8cf8f30c662f8f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/29/2018
ms.locfileid: "37111774"
---
# <a name="set-up-sql-data-sync"></a>設定 SQL 資料同步
在本教學課程中，您將了解如何使用包含 Azure SQL Database 和 SQL Server 執行個體的混合式同步群組設定 Azure SQL 資料同步。 新的同步處理群組會依照您設定的排程完整設定和同步。

本教學課程假設您先前至少有一些使用 SQL Database 和 SQL Server 的經驗。 

如需 SQL 資料同步的概觀，請參閱[使用 Azure SQL 資料同步，跨多個雲端和內部部署資料庫同步處理資料](sql-database-sync-data.md)。

如需示範如何設定 SQL 資料同步的完整 PowerShell 範例，請參閱下列文章：
-   [使用 PowerShell 在多個 Azure SQL Database 之間進行同步處理](scripts/sql-database-sync-data-between-sql-databases.md)
-   [使用 PowerShell 設定「資料同步」在內部部署的 Azure SQL Database 和 SQL Server 之間進行同步處理](scripts/sql-database-sync-data-between-azure-onprem.md)

## <a name="step-1---create-sync-group"></a>步驟 1 - 建立同步群組

### <a name="locate-the-data-sync-settings"></a>尋找資料同步設定

1.  在瀏覽器中導覽至 Azure 入口網站。

2.  在入口網站中，從您的儀表板或從工具列上的 SQL Database 圖示找出 SQL 資料庫。

    ![Azure SQL 資料庫清單](media/sql-database-get-started-sql-data-sync/datasync-preview-sqldbs.png)

3.  在 [SQL 資料庫] 頁面上，選取您想要作為資料同步的中樞資料庫使用的現有 SQL 資料庫。[SQL 資料庫] 頁面隨即開啟。

    中樞資料庫是同步拓撲的中央端點，其中的同步群組具有多個資料庫端點。 同一個同步群組中的所有其他資料庫端點 (也就是，所有成員資料庫) 都會與中樞資料庫進行同步。

4.  在所選取資料庫的 [SQL 資料庫] 頁面上，選取 [同步至其他資料庫]。 [資料同步] 頁面隨即開啟。

    ![[同步至其他資料庫] 選項](media/sql-database-get-started-sql-data-sync/datasync-preview-newsyncgroup.png)

### <a name="create-a-new-sync-group"></a>建立新的同步群組

1.  在 [資料同步] 頁面上，選取 [新的同步群組]。 [新的同步群組] 頁面隨即開啟，且系統會反白顯示步驟 1：[建立同步群組]。 [建立資料同步群組] 頁面同時也會開啟。

2.  在 [建立資料同步群組] 頁面上，執行下列步驟：

    1.  在 [同步群組名稱] 欄位中，輸入新同步群組的名稱。

    2.  在 [同步中繼資料的資料庫] 區段中，選擇要建立新的資料庫 (建議) 或使用現有的資料庫。

        > [!NOTE]
        > Microsoft 建議您建立新的空白資料庫作為同步中繼資料的資料庫。 資料同步會在此資料庫中建立資料表，並頻繁執行工作負載。 這個資料庫會作為同步中繼資料的資料庫，自動和選取區域中所有同步群組共用。 您無法在不卸除同步中繼資料資料庫或其名稱的情況下變更它。

        如果您選擇 [新資料庫]，請選取 [建立新的資料庫]。 [SQL Database] 頁面隨即開啟。 在 [SQL Database] 頁面上，命名並設定新的資料庫。 然後選取 [確定]。

        如果您選擇 [現有的資料庫]，請從清單中選取資料庫。

    3.  在 [自動同步] 區段中，先選取 [開啟] 或 [關閉]。

        如果您選擇 [開啟]，請在 [同步處理頻率] 區段中輸入數字，然後選取 [秒]、[分鐘]、[小時] 或 [天]。

        ![指定同步處理頻率](media/sql-database-get-started-sql-data-sync/datasync-preview-syncfreq.png)

    4.  在 [衝突解決] 區段中，選取 [中樞獲勝] 或 [成員獲勝]。

        「中樞獲勝」表示，發生衝突時，中樞資料庫中的資料會覆寫成員資料庫中的衝突資料。 「成員獲勝」表示，發生衝突時，成員資料庫中的資料會覆寫中樞資料庫中的衝突資料。 

        ![指定解決衝突的方式](media/sql-database-get-started-sql-data-sync/datasync-preview-conflictres.png)

    5.  選取 [確定] 並等候新的同步群組建立和部署完成。

## <a name="step-2---add-sync-members"></a>步驟 2 - 新增同步成員

在建立和部署新的同步群組之後，系統會反白顯示 [新的同步群組] 頁面中的步驟 2：[新增同步成員]。

在 [中樞資料庫] 區段中，輸入中樞資料庫所在 SQL Database 伺服器的現有認證。 請不要在此區段輸入「新」的認證。

![中樞資料庫已新增至同步群組](media/sql-database-get-started-sql-data-sync/datasync-preview-hubadded.png)

### <a name="add-an-azure-sql-database"></a>新增 Azure SQL Database

在 [成員資料庫] 區段中，選取 [新增 Azure 資料庫]，可選擇性地將 Azure SQL Database 新增至同步群組。 [設定 Azure 資料庫] 頁面隨即開啟。

在 [設定 Azure 資料庫] 頁面上，執行下列步驟：

1.  在 [同步成員名稱] 欄位中，提供新同步成員的名稱。 這個名稱與資料庫本身的名稱不同。

2.  在 [訂用帳戶] 欄位中，選取相關聯的 Azure 訂用帳戶以便計費。

3.  在 [Azure SQL Server] 欄位中，選取現有的 SQL 資料庫伺服器。

4.  在 [Azure SQL Database] 欄位中，選取現有的 SQL 資料庫。

5.  在 [同步方向] 欄位中，選取 [雙向同步]、[至中樞] 或 [來自中樞]。

    ![新增 SQL Database 同步處理成員](media/sql-database-get-started-sql-data-sync/datasync-preview-memberadding.png)

6.  在 [使用者名稱] 和 [密碼] 欄位中，輸入成員資料庫所在 SQL Database 伺服器的現有認證。 請不要在此區段輸入「新」的認證。

7.  選取 [確定] 並等候新的同步成員建立和部署完成。

    ![已新增 SQL Database 同步處理成員](media/sql-database-get-started-sql-data-sync/datasync-preview-memberadded.png)

### <a name="add-on-prem"></a> 新增內部部署 SQL Server 資料庫

在 [成員資料庫] 區段中，選取 [新增內部部署資料庫]，可選擇性地將內部部署 SQL Server 新增至同步群組。 [設定內部部署] 頁面隨即開啟。

在 [設定內部部署] 頁面上，執行下列步驟：

1.  選取 [選擇同步代理程式閘道]。 [選取同步代理程式] 頁面隨即開啟。

    ![選擇同步代理程式閘道](media/sql-database-get-started-sql-data-sync/datasync-preview-choosegateway.png)

2.  在 [選擇同步代理程式閘道] 頁面上，選擇要使用現有的代理程式，或建立新的代理程式。

    如果您選擇 [現有代理程式]，請從清單中選取現有的代理程式。

    在 [建立新的代理程式] 刀鋒視窗中，請執行下列步驟：

    1.  從提供的連結下載用戶端同步代理程式軟體，並安裝在 SQL Server 所在的電腦上。
 
        > [!IMPORTANT]
        > 您必須在防火牆開啟輸出 TCP 連接埠 1433，以讓用戶端代理程式和伺服器通訊。


    2.  輸入代理程式的名稱。

    3.  選取 [建立並產生金鑰]。

    4.  將代理程式金鑰複製到剪貼簿。
        
        ![建立新同步代理程式](media/sql-database-get-started-sql-data-sync/datasync-preview-selectsyncagent.png)

    5.  選取 [確定] 以關閉 [選取同步代理程式] 頁面。

    6.  在 SQL Server 電腦上，找出並執行用戶端同步代理程式應用程式。

        ![資料同步用戶端代理程式應用程式](media/sql-database-get-started-sql-data-sync/datasync-preview-clientagent.png)

    7.  在同步處理代理程式應用程式中，選取 [提交代理程式金鑰]。 [同步中繼資料的資料庫組態] 對話方塊隨即開啟。

    8.  在 [同步中繼資料的資料庫組態] 對話方塊中，貼上從 Azure 入口網站複製的代理程式金鑰。 還要提供輸入中繼資料資料庫所在 Azure SQL Database 伺服器的現有認證。 (如果您已建立新的中繼資料資料庫，此資料庫會位在和中樞資料庫相同的伺服器)。選取 [確定] 並等待完成組態。

        ![輸入代理程式金鑰和伺服器認證](media/sql-database-get-started-sql-data-sync/datasync-preview-agent-enterkey.png)

        >   [!NOTE] 
        >   如果在此時收到防火牆錯誤，您必須在 Azure 上建立防火牆規則，以允許來自 SQL Server 電腦的傳入流量。 您可以在入口網站手動建立規則，但在 SQL Server Management Studio (SSMS) 中建立可能會更容易。 在 SSMS 中，嘗試連線到 Azure 上的中樞資料庫。 請將其名稱輸入為 <hub_database_name>.database.windows.net。 若要設定 Azure 防火牆規則，請按照對話方塊中的步驟操作。 然後返回用戶端同步代理程式應用程式。

    9.  在用戶端同步代理程式應用程式中，按一下 [註冊]，以向代理程式註冊 SQL Server 資料庫。 [SQL Server 組態] 對話方塊隨即開啟。

        ![新增和設定 SQL Server 資料庫](media/sql-database-get-started-sql-data-sync/datasync-preview-agent-adddb.png)

    10. 在 [SQL Server 組態] 對話方塊方塊中，選擇要使用 SQL Server 驗證或 Windows 驗證來連線。 如果您選擇 SQL Server 驗證，請輸入現有的認證。 提供 SQL Server 名稱和您要同步之資料庫的名稱。選取 [測試連接] 來測試您的設定。 然後選取 [儲存]。 已註冊的資料庫會出現在清單中。

        ![現在已註冊 SQL Server 資料庫](media/sql-database-get-started-sql-data-sync/datasync-preview-agent-dbadded.png)

    11. 您現在可以關閉用戶端同步代理程式應用程式。

    12. 在入口網站中的 [設定內部部署] 頁面上，選取 [選取資料庫]。 [選取資料庫] 頁面隨即開啟。

    13. 在 [選取資料庫] 頁面上的 [同步成員名稱] 欄位中，提供新同步成員的名稱。 這個名稱與資料庫本身的名稱不同。 從清單中選取資料庫。 在 [同步方向] 欄位中，選取 [雙向同步]、[至中樞] 或 [來自中樞]。

        ![選取內部部署資料庫](media/sql-database-get-started-sql-data-sync/datasync-preview-selectdb.png)

    14. 選取 [確定] 以關閉 [選取資料庫] 頁面。 然後選取 [確定] 以關閉 [設定內部部署] 頁面，並等候系統建立並部署新的同步成員。 最後，按一下 [確定] 以關閉 [選取同步成員] 頁面。

        ![內部部署資料庫已新增至同步群組](media/sql-database-get-started-sql-data-sync/datasync-preview-onpremadded.png)

3.  若要連接到 [SQL 資料同步] 和本機代理程式，請將您的使用者名稱新增至 `DataSync_Executor` 角色。 資料同步會在 SQL Server 執行個體上建立此角色。

## <a name="step-3---configure-sync-group"></a>步驟 3 - 設定同步群組

在建立並部署新的同步群組成員之後，系統會反白顯示 [新的同步群組] 頁面中的步驟 3：[設定同步群組]。

1.  在 [資料表] 頁面上，從同步群組成員清單中選取資料庫，然後選取 [重新整理結構描述]。

2.  從可用資料表清單中，選取您想要同步的資料表。

    ![選取要同步的資料表](media/sql-database-get-started-sql-data-sync/datasync-preview-tables.png)

3.  預設會選取資料表中的所有資料行。 如果您不想同步所有資料行，請取消選取您不想要同步的資料行核取方塊。請務必保留選取的主索引鍵資料行。

    ![選取要同步的欄位](media/sql-database-get-started-sql-data-sync/datasync-preview-tables2.png)

4.  最後，選取 [儲存]。

## <a name="faq-about-setup-and-configuration"></a>關於安裝和設定的常見問題集

### <a name="how-frequently-can-data-sync-synchronize-my-data"></a>資料同步多久會同步我的資料一次？ 
至少每隔五分鐘。

### <a name="does-sql-data-sync-fully-create-and-provision-tables"></a>SQL 資料同步是否會完整地建立和佈建資料表？

如果尚未在目的地資料庫中建立同步結構描述資料表，SQL 資料同步會使用您選取的資料行來建立它們。 然而，此行為不會導致結構描述完整無缺，原因如下：

-   只會在目的地資料表中建立您選取的資料行。 如果來源資料表中有些資料行不是同步群組的一部分，那些資料行便不會佈建在目的地資料表中。

-   只會針對所選資料行建立索引。 如果來源資料表索引具有不是同步群組一部分的資料行，那些索引便不會佈建在目的地資料表中。

-   不會佈建 XML 型別資料行的索引。

-   不會佈建 CHECK 條件約束。

-   不會佈建來源資料表上的現有觸發程序。

-   檢視和預存程序不會建立在目的地資料庫上。

由於有這些限制，我們的建議事項如下：
-   在生產環境自行佈建完整無缺結構描述。
-   試用服務時，非常適合使用 SQL 資料同步的自動佈建功能。

### <a name="why-do-i-see-tables-that-i-did-not-create"></a>為什麼我會看到並非自己建立的資料表？  
資料同步會在資料庫中建立側邊資料表，以便進行變更追蹤。 請勿刪除它們，否則資料同步無法正常運作。

### <a name="is-my-data-convergent-after-a-sync"></a>同步處理之後我的資料會聚合嗎？

不一定。 在有一個中樞和三個支點 (A、B 及 C) 的同步群組中，同步處理是以中樞對 A 點、中樞對 B 點及中樞對 C 點的方式進行。如果在中樞對 A 點同步*之後*，才變更資料庫 A，在進行下次同步工作之前，該變更不會寫入資料庫 B 或資料庫 C。

### <a name="how-do-i-get-schema-changes-into-a-sync-group"></a>如何變更同步群組中的結構描述？

您必須以手動方式進行及傳播所有結構描述變更。
1. 以手動方式將結構描述變更複寫至中樞和所有的同步成員。
2. 更新同步結構描述。

**新增資料表和資料行**。 新的資料表和資料行不會影響目前的同步。資料同步會忽略新的資料表和資料行，直到您將它們新增至同步結構描述。 當您新增新的資料庫物件時，這是要遵循的最佳順序：
1. 將新的資料表或資料行新增至中樞和所有的同步成員。
2. 將新的資料表或資料行新增至同步結構描述。
3. 開始將值插入新的資料表和資料行。

**變更資料行的資料類型**。 當您變更現有資料行的資料類型時，只要新的值符合同步結構描述中定義的原始資料類型，資料同步就會繼續運作。 例如，如果您將來源資料庫中的類型從 **int** 變更為 **bigint**，資料同步就會繼續運作，直到您將對於 **int** 資料類型太大的值插入為止。 若要完成變更，請以手動方式將結構描述變更複寫至中樞和所有的同步成員，然後更新同步結構描述。

### <a name="how-can-i-export-and-import-a-database-with-data-sync"></a>如何使用資料同步匯出和匯入資料庫？
將資料庫匯出為 `.bacpac` 檔案，並將該檔案匯入以建立新資料庫之後，您必須執行下列兩個動作，才能在新的資料庫中使用資料同步：
1.  使用[這個指令碼](https://github.com/Microsoft/sql-server-samples/blob/master/samples/features/sql-data-sync/clean_up_data_sync_objects.sql)清除**新資料庫**上的資料同步物件和側邊資料表。 這個指令碼會刪除資料庫中所有必要的資料同步物件。
2.  利用新資料庫重新建立同步群組。 如果您不再需要舊有的同步群組，請將它刪除。

## <a name="faq-about-the-client-agent"></a>關於用戶端代理程式的常見問題集

### <a name="why-do-i-need-a-client-agent"></a>我為何需要用戶端代理程式？

SQL 資料同步服務會透過用戶端代理程式來與 SQL Server 資料庫通訊。 這項安全性功能可防止直接與防火牆後方的資料庫通訊。 當 SQL 資料同步服務與代理程式通訊時，會使用加密的連線與唯一的權杖或「代理程式金鑰」來執行此動作。 SQL Server 資料庫會使用連接字串和代理程式金鑰來驗證代理程式。 這項設計可為您的資料提供高階安全性。

### <a name="how-many-instances-of-the-local-agent-ui-can-be-run"></a>一共可執行多少個本機代理程式 UI 執行個體？

只能執行一個 UI 執行個體。

### <a name="how-can-i-change-my-service-account"></a>如何變更我的服務帳戶？

安裝用戶端代理程式之後，變更服務帳戶的唯一方式是予以解除安裝，並使用新的服務帳戶來安裝新的用戶端代理程式。

### <a name="how-do-i-change-my-agent-key"></a>如何變更我的代理程式金鑰？

代理程式只會使用代理程式金鑰一次。 當您移除再重新安裝新的代理程式時，並不能重複使用它，也不能由多個代理程式使用。 如果您需要為現有代理程式建立新的金鑰，務必確定會以用戶端代理程式和 SQL 資料同步服務來記錄相同的金鑰。

### <a name="how-do-i-retire-a-client-agent"></a>如何淘汰用戶端代理程式？

若要讓代理程式立即失效或淘汰，請在入口網站中重新產生其金鑰，但不在代理程式 UI 中提交。 不論對應的代理程式處於連線或離線，重新產生金鑰即可讓先前的金鑰失效。

### <a name="how-do-i-move-a-client-agent-to-another-computer"></a>如何將用戶端代理程式移至另一部電腦？

如果您想要從另一部電腦執行本機代理程式，而非它目前所在的電腦，請執行下列動作：

1. 在所需電腦上安裝代理程式。

2. 登入 SQL 資料同步入口網站，然後為新的代理程式重新產生代理程式金鑰。

3. 使用新的代理程式 UI 來提交新的代理程式金鑰。

4. 請等待用戶端代理程式下載先前所註冊內部部署資料庫的清單。

5. 提供資料庫認證給顯示為無法連線的所有資料庫。 這些資料庫必須要能從安裝該代理程式的新電腦連線。

## <a name="next-steps"></a>後續步驟
恭喜！ 您已建立一個同時包含 SQL Database 執行個體與 SQL Server 資料庫的同步群組。

如需 SQL 資料同步的詳細資訊，請參閱：

-   [使用 Azure SQL 資料同步，跨多個雲端和內部部署資料庫同步處理資料](sql-database-sync-data.md)
-   [Azure SQL 資料同步最佳做法](sql-database-best-practices-data-sync.md)
-   [透過 Log Analytics 監視 Azure SQL 資料同步](sql-database-sync-monitor-oms.md)
-   [對 Azure SQL 資料同步的問題進行疑難排解](sql-database-troubleshoot-data-sync.md)

-   示範如何設定 SQL 資料同步的完整 PowerShell 範例：
    -   [使用 PowerShell 在多個 Azure SQL Database 之間進行同步處理](scripts/sql-database-sync-data-between-sql-databases.md)
    -   [使用 PowerShell 設定「資料同步」在內部部署的 Azure SQL Database 和 SQL Server 之間進行同步處理](scripts/sql-database-sync-data-between-azure-onprem.md)

-   [下載 SQL 資料同步 REST API 文件](https://github.com/Microsoft/sql-server-samples/raw/master/samples/features/sql-data-sync/Data_Sync_Preview_REST_API.pdf?raw=true)

如需 SQL Database 的詳細資訊，請參閱：

-   [SQL Database 概觀](sql-database-technical-overview.md)
-   [資料庫生命週期管理](https://msdn.microsoft.com/library/jj907294.aspx)
