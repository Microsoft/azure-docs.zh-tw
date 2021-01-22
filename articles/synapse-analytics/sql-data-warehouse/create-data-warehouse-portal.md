---
title: 快速入門：建立和查詢專用 SQL 集區 (先前稱為 SWL DW) (Azure 入口網站)
description: 使用 Azure 入口網站建立和查詢專用 SQL 集區 (先前稱為 SWL DW)
services: synapse-analytics
author: kevinvngo
manager: craigg
ms.service: synapse-analytics
ms.topic: quickstart
ms.subservice: sql-dw
ms.date: 05/28/2019
ms.author: pimorano
ms.reviewer: igorstan
ms.custom: seo-lt-2019, azure-synapse
ms.openlocfilehash: ecab573f7a98bcee79231b8e20077bb32bffd48c
ms.sourcegitcommit: b39cf769ce8e2eb7ea74cfdac6759a17a048b331
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 01/22/2021
ms.locfileid: "98680109"
---
# <a name="quickstart-create-and-query-a-dedicated-sql-pool-formerly-sql-dw-in-azure-synapse-analytics-using-the-azure-portal"></a>快速入門：使用 Azure 入口網站建立和查詢 Azure Synapse Analytics 中的專用 SQL 集區 (先前稱為 SWL DW)

使用 Azure 入口網站快速建立和查詢 Azure Synapse Analytics 中的專用 SQL 集區 (先前稱為 SWL DW)。

## <a name="prerequisites"></a>必要條件

1. 如果您沒有 Azure 訂用帳戶，請在開始前建立[免費帳戶](https://azure.microsoft.com/free/)。

   > [!NOTE]
   > 在 Azure Synapse 中建立專用 SQL 集區 (先前稱為 SQL DW) 可能會產生新的可計費服務。 如需詳細資訊，請參閱 [Azure Synapse Analytics 定價](https://azure.microsoft.com/pricing/details/synapse-analytics/)。

2. 下載並安裝最新版的 [SQL Server Management Studio](/sql/ssms/download-sql-server-management-studio-ssms?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest&preserve-view=true) (SSMS)。

## <a name="sign-in-to-the-azure-portal"></a>登入 Azure 入口網站

登入 [Azure 入口網站](https://portal.azure.com/)。

## <a name="create-a-sql-pool"></a>建立 SQL 集區

使用專用 SQL 集區 (先前稱為 SQL DW)，在 Azure Synapse Analytics 中建立資料倉儲 。 專用 SQL 集區 (先前稱為 SQL DW) 會使用一組已定義的[計算資源](memory-concurrency-limits.md)來建立。 此資料庫建立於 [Azure 資源群組](../../azure-resource-manager/management/overview.md?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json)和[邏輯 SQL 伺服器](../../azure-sql/database/logical-servers.md?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json)內。

請遵循下列步驟來建立包含 **AdventureWorksDW** 範例資料的專用 SQL 集區 (先前稱為 SQL DW)。

1. 選取 Azure 入口網站左上角的 [建立資源]  。

   ![在 Azure 入口網站中建立資源](./media/create-data-warehouse-portal/create-a-resource.png)

2. 在搜尋列中，輸入「專用 SQL 集區」，選取專用的 SQL 集區 (先前稱為 SQL DW)。 在開啟的頁面上，選取 [建立]。

   ![建立空的資料倉儲](./media/create-data-warehouse-portal/create-a-data-warehouse.png)

3. 在 **基本資料** 中，提供您的訂用帳戶、資源群組、專用 SQL 集區 (先前稱為 SQL DW) 名稱和伺服器名稱：

   | 設定 | 建議的值 | 描述 |
   | :------ | :-------------- | :---------- |
   | **訂用帳戶** | 您的訂用帳戶 | 如需訂用帳戶的詳細資訊，請參閱[訂用帳戶](https://account.windowsazure.com/Subscriptions)。 |
   | **資源群組** | myResourceGroup | 如需有效的資源群組名稱，請參閱[命名規則和限制](/azure/architecture/best-practices/resource-naming?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json)。 |
   | **SQL 集區名稱** | 任何全域唯一名稱 (例如 *mySampleDataWarehouse*) | 如需有效的資料庫名稱，請參閱[資料庫識別碼](/sql/relational-databases/databases/database-identifiers?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest&preserve-view=true)。  |
   | **Server** | 任何全域唯一名稱 | 選取現有的伺服器，或建立新的伺服器名稱，然後選取 [新建]。 如需有效的伺服器名稱，請參閱[命名規則和限制](/azure/architecture/best-practices/resource-naming?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json)。 |

   ![建立資料倉儲的基本詳細資料](./media/create-data-warehouse-portal/create-sql-pool-basics.png)

4. 在 [效能等級] 中，選取 [選取效能等級]，以選擇性地使用滑桿來變更您的設定。

   ![變更資料倉儲效能等級](./media/create-data-warehouse-portal/create-sql-pool-performance-level.png)  

   如需效能等級的詳細資訊，請參閱[管理 Azure Synapse Analytics 中的計算能力](sql-data-warehouse-manage-compute-overview.md)。

5. 選取 [其他設定]，在 [使用現有資料] 底下選擇 [範例]，以將 AdventureWorksDW 建立為範例資料庫。

    ![選取 [使用現有資料]](./media/create-data-warehouse-portal/create-sql-pool-additional-1.png)

6. 現在您已完成 Azure Synapse Analytics 表單的 [基本] 索引標籤，接下來請選取 [檢閱 + 建立]，然後選取 [建立] 以建立 SQL 集區。 佈建需要幾分鐘的時間。

   ![選取 [檢閱 + 建立]](./media/create-data-warehouse-portal/create-sql-pool-review-create.png)

   ![選取 [建立]](./media/create-data-warehouse-portal/create-sql-pool-create.png)

7. 在工具列上選取 [通知]，以監視部署程序。

   ![此螢幕擷取畫面顯示部署進行中的通知。](./media/create-data-warehouse-portal/notification.png)

## <a name="create-a-server-level-firewall-rule"></a>建立伺服器層級防火牆規則

Azure Synapse 服務會在伺服器層級建立防火牆。 此防火牆會防止外部應用程式和工具連線到伺服器或伺服器上的任何資料庫。 若要啟用連線，您可以新增防火牆規則以啟用特定 IP 位址之連線。 遵循以下步驟建立用戶端 IP 位址的[伺服器層級防火牆規則](../../azure-sql/database/firewall-configure.md?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json)。

> [!NOTE]
> Azure Synapse 會透過連接埠 1433 進行通訊。 如果您嘗試從公司網路內進行連線，您網路的防火牆可能不允許透過連接埠 1433 的輸出流量。 若是如此，除非 IT 部門開啟連接埠 1433，否則您無法連線到您的伺服器。

1. 完成部署之後，請從左側功能表選取 [所有服務]。 選取 [資料庫]，選取 [Azure Synapse Analytics] 旁的星號，將 Azure Synapse Analytics 新增至您的最愛。

2. 從左側功能表中選取 [Azure Synapse Analytics]，然後在 [Azure Synapse Analytics] 頁面上選取 [mySampleDataWarehouse]。 資料庫的概觀頁面隨即開啟，其中會顯示完整的伺服器名稱 (例如 **sqlpoolservername.database.windows.net**)，並提供進一步設定的選項。

3. 在這個及其他快速入門中，請複製此完整伺服器名稱，才能用來連線到伺服器及其資料庫。 若要開啟伺服器設定，請選取伺服器名稱。

   ![尋找伺服器名稱](./media/create-data-warehouse-portal/find-server-name.png)

4. 選取 [顯示防火牆設定]。

   ![伺服器設定](./media/create-data-warehouse-portal/server-settings.png)

5. 伺服器的 [防火牆設定] 頁面會隨即開啟。

   ![伺服器防火牆規則](./media/create-data-warehouse-portal/server-firewall-rule.png)

6. 若要將目前的 IP 位址新增至新的防火牆規則，請選取工具列上的 [新增用戶端 IP]。 防火牆規則可以針對單一 IP 位址或 IP 位址範圍開啟連接埠 1433。

7. 選取 [儲存]。 系統便會為目前的 IP 位址建立伺服器層級防火牆規則，在伺服器上開啟連接埠 1433。

8. 選取 [確定]，然後關閉 [防火牆設定] 頁面。

您現在可以使用這個 IP 位址，連線到伺服器及其 SQL 集區。 可從 SQL Server Management Studio 或您選擇的另一個工具來運作連線。 當您連線時，請使用先前建立的 ServerAdmin 帳戶。

> [!IMPORTANT]
> 根據預設，已對所有 Azure 服務啟用透過 SQL Database 防火牆存取。 在此頁面上選取 [關閉]，然後選取 [儲存] 以停用所有 Azure 服務的防火牆。

## <a name="get-the-fully-qualified-server-name"></a>取得完整的伺服器名稱

請在 Azure 入口網站中取得伺服器的完整伺服器名稱。 稍後您在連線到伺服器時，會使用完整伺服器名稱。

1. 登入 [Azure 入口網站](https://portal.azure.com/)。

2. 從左側功能表中選取 [Azure Synapse Analytics]，然後在 [Azure Synapse Analytics] 頁面上選取您的。

3. 在 Azure 入口網站中您資料庫的 [基本資訊] 窗格中，找到後複製 [伺服器名稱]。 在此範例中，完整名稱是 sqlpoolservername.database.windows.net。

    ![連線資訊](./media/create-data-warehouse-portal/find-server-name.png)

## <a name="connect-to-the-server-as-server-admin"></a>以伺服器系統管理員身分連線到伺服器

本節使用 [SQL Server Management Studio](/sql/ssms/download-sql-server-management-studio-ssms?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest&preserve-view=true) (SSMS) 建立與伺服器的連線。

1. 開啟 SQL Server Management Studio。

2. 在 [連線至伺服器] 對話方塊中，輸入下列資訊：

   | 設定 | 建議的值 | 描述 |
   | :------ | :-------------- | :---------- |
   | 伺服器類型 | 資料庫引擎 | 這是必要值 |
   | 伺服器名稱 | 完整伺服器名稱 | 範例如下：**sqlpoolservername.database.windows.net**。 |
   | 驗證 | SQL Server 驗證 | SQL 驗證是本教學課程中設定的唯一驗證類型。 |
   | 登入 | 伺服器系統管理員帳戶 | 您在建立伺服器時所指定的帳戶。 |
   | 密碼 | 伺服器系統管理員帳戶的密碼 | 這是您在建立伺服器時所指定的密碼。 |
   ||||

   ![連線至伺服器](./media/create-data-warehouse-portal/connect-to-server-ssms.png)

3. 選取 [連線]。 [物件總管] 視窗會在 SSMS 中開啟。

4. 在 [物件總管] 中展開 [資料庫]。 然後展開 [mySampleDatabase] 可檢視新資料庫中的物件。

   ![資料庫物件](./media/create-data-warehouse-portal/connected-ssms.png)

## <a name="run-some-queries"></a>執行一些查詢

不建議在以伺服器管理員身分登入時執行大型查詢，因為這麼做會使用[有限的資源類別](resource-classes-for-workload-management.md)。 請改為設定[工作負載隔離](./quickstart-configure-workload-isolation-tsql.md)，如[教學課程所述](./load-data-wideworldimportersdw.md#create-a-user-for-loading-data)。

Azure Synapse Analytics 使用 T-SQL 作為查詢語言。 若要開啟查詢視窗並執行一些 T-SQL 查詢，請使用下列步驟：

1. 以滑鼠右鍵選取 [mySampleDataWarehouse]，然後選取 [新增查詢]。 隨即開啟 [新增查詢] 視窗。

2. 在新的查詢視窗中，輸入下列命令可查看資料庫清單。

    ```sql
    SELECT * FROM sys.databases
    ```

3. 選取 [執行]。 查詢結果顯示兩個資料庫：**master** 和 **mySampleDataWarehouse**。

   ![查詢資料庫](./media/create-data-warehouse-portal/query-databases.png)

4. 若要查看一些資料，請使用下列命令以查看姓氏為 Adams 且家裡有三個孩子的客戶數目。 結果列出 6 個客戶。

    ```sql
    SELECT LastName, FirstName FROM dbo.dimCustomer
    WHERE LastName = 'Adams' AND NumberChildrenAtHome = 3;
    ```

   ![查詢 dbo.dimCustomer](./media/create-data-warehouse-portal/query-customer.png)

## <a name="clean-up-resources"></a>清除資源

您需對資料倉儲單位和專用 SQL 集區 (先前稱為 SQL DW) 中儲存的資料付費。 這些計算和儲存體資源會分開計費。

- 如果您需要將資料保留在儲存體中，可以在您不使用專用 SQL 集區 (先前稱為 SQL DW) 時暫停計算。 暫停計算，您只需支付資料儲存體的費用。 當您準備要使用資料時，您可以繼續計算。

- 如果您想要移除未來的費用，可以將專用 SQL 集區 (先前稱為 SQL DW) 刪除。

遵循下列步驟，清除不再需要的資源。

1. 登入 [Azure 入口網站](https://portal.azure.com)，然後選取您的專用 SQL 集區 (先前稱為 SQL DW)。

   ![清除資源](./media/create-data-warehouse-portal/clean-up-resources.png)

2. 若要暫停計算，請選取 [暫停] 按鈕。 專用 SQL 集區 (先前稱為 SQL DW) 暫停時，您會看到 [繼續] 按鈕。 若要繼續計算，請選取 [繼續]。

3. 若要移除專用 SQL 集區 (先前稱為 SQL DW) 並不再支付計算或儲存體的費用，請選取 [刪除]。

4. 若要移除所建立的伺服器，請選取上圖中的 [sqlpoolservername.database.windows.net]，然後選取 [刪除]。 請謹慎使用刪除，因為刪除伺服器也會刪除所有指派給伺服器的資料庫。

5. 若要移除此資源群組，請選取 [myResourceGroup]，然後選取 [刪除資源群組]。

想要最佳化並節省您的雲端費用嗎？

[!INCLUDE [cost-management-horizontal](../../../includes/cost-management-horizontal.md)]

## <a name="next-steps"></a>後續步驟

若要深入了解如何將資料載入到專用 SQL 集區 (先前稱為 SQL DW)，請繼續[將資料載入專用 SQL 集區](load-data-from-azure-blob-storage-using-copy.md)一文。