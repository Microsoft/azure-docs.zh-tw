## <a name="sign-in-to-the-azure-portal"></a>登入 Azure 入口網站

登入 [Azure 入口網站](https://portal.azure.com/)。

## <a name="create-a-blank-sql-database"></a>建立空白 SQL Database

Azure SQL Database 會使用一組定義的[計算和儲存體資源](../articles/sql-database/sql-database-service-tiers-dtu.md)建立。 此資料庫建立於 [Azure 資源群組](../articles/azure-resource-manager/resource-group-overview.md)和 [Azure SQL Database 邏輯伺服器](../articles/sql-database/sql-database-features.md)內。 

遵循以下步驟來建立空白 SQL 資料庫。 

1. 按一下 Azure 入口網站左上角的 [建立資源]。

2. 從 [新增] 頁面中選取 [資料庫]，然後在 [新增] 頁面的 [SQL Database] 下選取 [建立]。

   ![建立空白資料庫](../articles/sql-database/media/sql-database-design-first-database/create-empty-database.png)

3. 使用下列資訊填寫 SQL Database 表單，如上圖所示︰   

   | 設定       | 建議的值 | 說明 | 
   | ------------ | ------------------ | ------------------------------------------------- | 
   | **資料庫名稱** | mySampleDatabase | 如需有效的資料庫名稱，請參閱[資料庫識別碼](https://docs.microsoft.com/sql/relational-databases/databases/database-identifiers)。 | 
   | **訂用帳戶** | 您的訂用帳戶  | 如需訂用帳戶的詳細資訊，請參閱[訂用帳戶](https://account.windowsazure.com/Subscriptions)。 |
   | **資源群組** | myResourceGroup | 如需有效的資源群組名稱，請參閱[命名規則和限制](https://docs.microsoft.com/azure/architecture/best-practices/naming-conventions)。 |
   | **選取來源** | 空白資料庫 | 指定應建立空白資料庫。 |

4. 按一下 [伺服器] 為您的新資料庫建立及設定新的伺服器。 在**新伺服器表單**表單中填寫下列資訊︰ 

   | 設定       | 建議的值 | 說明 | 
   | ------------ | ------------------ | ------------------------------------------------- | 
   | **伺服器名稱** | 任何全域唯一名稱 | 如需有效的伺服器名稱，請參閱[命名規則和限制](https://docs.microsoft.com/azure/architecture/best-practices/naming-conventions)。 | 
   | **伺服器管理員登入** | 任何有效名稱 | 如需有效的登入名稱，請參閱[資料庫識別碼](https://docs.microsoft.com/sql/relational-databases/databases/database-identifiers)。|
   | **密碼** | 任何有效密碼 | 您的密碼至少要有 8 個字元，而且必須包含下列幾種字元的其中三種︰大寫字元、小寫字元、數字和非英數字元。 |
   | **位置** | 任何有效位置 | 如需區域的相關資訊，請參閱 [Azure 區域](https://azure.microsoft.com/regions/)。 |

   ![建立資料庫伺服器](../articles/sql-database/media/sql-database-design-first-database/create-database-server.png)

5. 按一下 [選取] 。

6. 按一下 [定價層] 可指定服務層、DTU 數目和儲存體數量。 瀏覽 DTU 數量的選項，以及可供您每個服務層使用的儲存體。 

7. 在此教學課程中，選取 [標準] 服務層，然後使用滑桿選取 **100 DTU (S3)** 和 **400** GB 的儲存體。

   ![建立資料庫-s1](../articles/sql-database/media/sql-database-design-first-database/create-empty-database-pricing-tier.png)

8. 若要使用 [附加元件儲存體] 選項，請接受預覽條款。 

   > [!IMPORTANT]
   > \* 大於內含儲存體數量的儲存體大小尚在預覽中，而且會產生額外成本。 如需詳細資訊，請參閱 [SQL Database 定價](https://azure.microsoft.com/pricing/details/sql-database/)。 
   >
   >\* 在進階層中，超過 1 TB 的儲存體目前在下列區域為可用狀態：加拿大中部、加拿大東部、法國中部、德國中部、日本東部、南韓中部、美國中南部、東南亞、美國東部 2、美國西部、美國維吉尼亞州政府及西歐。 請參閱 [P11-P15 目前限制](../articles/sql-database/sql-database-dtu-resource-limits-single-databases.md#single-database-limitations-of-p11-and-p15-when-the-maximum-size-greater-than-1-tb)。  
   > 

9. 在選取伺服器層、DTU 數目和儲存體數量之後，按一下 [套用]。  

10. 為空白資料庫選取**定序** (此教學課程中使用預設值)。 如需定序的詳細資訊，請參閱[定序](https://docs.microsoft.com/sql/t-sql/statements/collations)。

11. 按一下 [建立] 即可佈建資料庫。 佈建完成所需時間約 1.5 分。 

12. 在工具列上，按一下 [通知] 以監視部署程序。
    
     ![通知](../articles/sql-database/media/sql-database-get-started-portal/notification.png)

## <a name="create-a-server-level-firewall-rule"></a>建立伺服器層級防火牆規則

SQL Database 服務會在伺服器層級建立防火牆，防止外部應用程式和工具連線到伺服器或伺服器上的任何資料庫，除非建立防火牆規則以針對特定的 IP 位址開啟防火牆。 請遵循下列步驟來為您用戶端的 IP 位址建立 [SQL Database 伺服器層級防火牆規則](../articles/sql-database/sql-database-firewall-configure.md)，並讓外部連線僅能夠穿過您 IP 位址的 SQL Database 防火牆。 

> [!NOTE]
> SQL Database 會透過連接埠 1433 通訊。 如果您嘗試從公司網路進行連線，您網路的防火牆可能不允許透過連接埠 1433 的輸出流量。 若情況如此，除非 IT 部門開啟連接埠 1433，否則您無法連線至 Azure SQL Database 伺服器。
>

1. 部署完成之後，按一下左側功能表中的 [SQL Database]，然後按一下 [SQL Database] 頁面上的 [mySampleDatabase]。 資料庫的 [概觀] 頁面隨即開啟，其中會顯示完整伺服器名稱 (例如 **mynewserver20170824.database.windows.net**)，並提供進一步的組態選項。 

2. 在後續的快速入門中，請複製此完整伺服器名稱，才能用來連線到伺服器及其資料庫。 

   ![伺服器名稱](../articles/sql-database/media/sql-database-get-started-portal/server-name.png) 

3. 在工具列上按一下 [設定伺服器防火牆]。 SQL Database 伺服器的 [防火牆設定] 頁面隨即開啟。 

   ![伺服器防火牆規則](../articles/sql-database/media/sql-database-get-started-portal/server-firewall-rule.png) 

4. 按一下工具列上的 [新增用戶端 IP]，將目前的 IP 位址新增至新的防火牆規則。 防火牆規則可以針對單一 IP 位址或 IP 位址範圍開啟連接埠 1433。

5. 按一下 [檔案] 。 系統便會為目前的 IP 位址建立伺服器層級防火牆規則，以便在邏輯伺服器上開啟連接埠 1433。

6. 依序按一下 [確定]，然後關閉 [防火牆設定] 頁面。

您現在可以利用 SQL Server Management Studio 或選擇的其他工具，使用先前建立的伺服器管理帳戶從這個 IP 位址連線至 SQL Database 伺服器及其資料庫。


> [!IMPORTANT]
> 根據預設，已對所有 Azure 服務啟用透過 SQL Database 防火牆存取。 按一下此頁面上的 [關閉] 即可對所有 Azure 服務停用。

## <a name="sql-server-connection-information"></a>SQL Server 連線資訊

在 Azure 入口網站中取得 Azure SQL Database 伺服器的完整伺服器名稱。 透過 SQL Server Management Studio，您可使用此完整伺服器名稱連接到您的伺服器。

1. 登入 [Azure 入口網站](https://portal.azure.com/)。
2. 從左側功能表中選取 [SQL Database]，按一下 [SQL Database]頁面上您的資料庫。 
3. 在 Azure 入口網站中您資料庫的 [基本資訊] 窗格中，找到後複製 [伺服器名稱]。

   ![連線資訊](../articles/sql-database/media/sql-database-get-started-portal/server-name.png)
