---
title: 快速入門：建立和查詢 Azure SQL 資料倉儲 - Azure 入口網站 | Microsoft Docs
description: 在 Azure 入口網站中透過 Azure SQL 資料倉儲建立和查詢資料倉儲。
services: sql-data-warehouse
author: kevinvngo
manager: craigg
ms.service: sql-data-warehouse
ms.topic: conceptual
ms.component: manage
ms.date: 08/01/2018
ms.author: kevin
ms.reviewer: igorstan
ms.openlocfilehash: 004ae5c8c171f731a244e926dd51d677300630f1
ms.sourcegitcommit: 2b2129fa6413230cf35ac18ff386d40d1e8d0677
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/30/2018
ms.locfileid: "43248071"
---
# <a name="quickstart-create-and-query-an-azure-sql-data-warehouse-in-the-azure-portal"></a>快速入門：如何在 Azure 入口網站中建立及查詢 Azure SQL 資料倉儲

使用 Azure 入口網站快速建立及查詢 Azure SQL 資料倉儲。

如果您沒有 Azure 訂用帳戶，請在開始前建立[免費帳戶](https://azure.microsoft.com/free/)。

> [!NOTE]
> 建立 SQL 資料倉儲可能會導致新的可計費服務。  如需詳細資訊，請參閱 [SQL 資料倉儲價格](https://azure.microsoft.com/pricing/details/sql-data-warehouse/)。
>
>

## <a name="before-you-begin"></a>開始之前

下載並安裝最新版的 [SQL Server Management Studio](/sql/ssms/download-sql-server-management-studio-ssms) (SSMS)。

## <a name="sign-in-to-the-azure-portal"></a>登入 Azure 入口網站

登入 [Azure 入口網站](https://portal.azure.com/)。

## <a name="create-a-data-warehouse"></a>建立資料倉儲

Azure SQL 資料倉儲會使用一組定義的[計算資源](memory-and-concurrency-limits.md)建立。 此資料庫建立於 [Azure 資源群組](../azure-resource-manager/resource-group-overview.md)和 [Azure SQL 邏輯伺服器](../sql-database/sql-database-logical-servers.md)內。 

依照下列步驟來建立包含 AdventureWorksDW 範例資料的 SQL 資料倉儲。 

1. 按一下 Azure 入口網站左上角的 [建立資源]。

2. 從 [新增] 頁面中選取 [資料庫]，然後在 [新增] 頁面的 [精選] 下選取 [SQL 資料倉儲]。

    ![建立空的資料倉儲](media/load-data-from-azure-blob-storage-using-polybase/create-empty-data-warehouse.png)

3. 在 SQL 資料倉儲表單中填寫下列資訊︰   

    | 設定 | 建議的值 | 說明 | 
    | ------- | --------------- | ----------- | 
    | **資料庫名稱** | mySampleDataWarehouse | 如需有效的資料庫名稱，請參閱[資料庫識別碼](/sql/relational-databases/databases/database-identifiers)。 請注意，資料倉儲是一種資料庫。| 
    | **訂用帳戶** | 您的訂用帳戶  | 如需訂用帳戶的詳細資訊，請參閱[訂用帳戶](https://account.windowsazure.com/Subscriptions)。 |
    | **資源群組** | myResourceGroup | 如需有效的資源群組名稱，請參閱[命名規則和限制](https://docs.microsoft.com/azure/architecture/best-practices/naming-conventions)。 |
    | **選取來源** | 範例 | 指定要載入範例資料庫。 請注意，資料倉儲是一種資料庫。 |
    | **選取範例** | AdventureWorksDW | 指定要載入 AdventureWorksDW 範例資料庫。  |

    ![建立資料倉儲](media/create-data-warehouse-portal/select-sample.png)

4. 按一下 [伺服器] 為您的新資料庫建立及設定新的伺服器。 在**新伺服器表單**表單中填寫下列資訊︰ 

    | 設定 | 建議的值 | 說明 | 
    | ------------ | ------------------ | ------------------------------------------------- | 
    | **伺服器名稱** | 任何全域唯一名稱 | 如需有效的伺服器名稱，請參閱[命名規則和限制](https://docs.microsoft.com/azure/architecture/best-practices/naming-conventions)。 | 
    | **伺服器管理員登入** | 任何有效名稱 | 如需有效的登入名稱，請參閱[資料庫識別碼](https://docs.microsoft.com/sql/relational-databases/databases/database-identifiers)。|
    | **密碼** | 任何有效密碼 | 您的密碼至少要有 8 個字元，而且必須包含下列幾種字元的其中三種︰大寫字元、小寫字元、數字和非英數字元。 |
    | **位置** | 任何有效位置 | 如需區域的相關資訊，請參閱 [Azure 區域](https://azure.microsoft.com/regions/)。 |

    ![建立資料庫伺服器](media/load-data-from-azure-blob-storage-using-polybase/create-database-server.png)

5. 按一下 [選取] 。

6. 按一下 [效能等級] 以指定資料倉儲的效能設定。

7. 針對本教學課程，請選取 [Gen2]。 根據預設，滑桿會設定為 [DW1000c]。  請嘗試向上和向下移動以查看其運作方式。 

    ![設定效能](media/load-data-from-azure-blob-storage-using-polybase/configure-performance.png)

8. 按一下 [套用]。

9. 您現在已完成 SQL 資料倉儲表單，請按一下 [建立] 以佈建資料庫。 佈建需要幾分鐘的時間。 

    ![按一下 [建立]](media/load-data-from-azure-blob-storage-using-polybase/click-create.png)

10. 在工具列上，按一下 [通知] 以監視部署程序。
    
     ![通知](media/load-data-from-azure-blob-storage-using-polybase/notification.png)

## <a name="create-a-server-level-firewall-rule"></a>建立伺服器層級防火牆規則

SQL 資料倉儲服務會在伺服器層級建立防火牆，防止外部應用程式和工具連線到伺服器或伺服器上的任何資料庫。 若要啟用連線，您可以新增防火牆規則以啟用特定 IP 位址之連線。  遵循以下步驟建立用戶端 IP 位址的[伺服器層級防火牆規則](../sql-database/sql-database-firewall-configure.md)。 

> [!NOTE]
> SQL 資料倉儲會透過連接埠 1433 通訊。 如果您嘗試從公司網路內進行連線，您網路的防火牆可能不允許透過連接埠 1433 的輸出流量。 若情況如此，除非 IT 部門開啟連接埠 1433，否則您無法連線至 Azure SQL Database 伺服器。
>

1. 部署完成之後，按一下左側功能表中的 [SQL 資料倉儲]，然後按一下 [SQL 資料倉儲] 頁面上的 [mySampleDatabase]。 資料庫的概觀頁面隨即開啟，其中會顯示完整伺服器名稱 (例如 **mynewserver-20180430.database.windows.net**)，並提供進一步的設定選項。 

2. 在後續的快速入門中，請複製此完整伺服器名稱，才能用來連線到伺服器及其資料庫。 若要開啟伺服器設定，請按一下伺服器名稱。

   ![尋找伺服器名稱](media/load-data-from-azure-blob-storage-using-polybase/find-server-name.png) 

3. 若要開啟伺服器設定， 
4. 按一下伺服器名稱。

   ![伺服器設定](media/load-data-from-azure-blob-storage-using-polybase/server-settings.png) 

5. 按一下 [顯示防火牆設定]。 SQL Database 伺服器的 [防火牆設定] 頁面隨即開啟。 

   ![伺服器防火牆規則](media/load-data-from-azure-blob-storage-using-polybase/server-firewall-rule.png) 

4. 若要將目前的 IP 位址新增至新的防火牆規則，按一下工具列上的 [新增用戶端 IP]。 防火牆規則可以針對單一 IP 位址或 IP 位址範圍開啟連接埠 1433。

5. 按一下 [檔案] 。 系統便會為目前的 IP 位址建立伺服器層級防火牆規則，以便在邏輯伺服器上開啟連接埠 1433。

6. 依序按一下 [確定]，然後關閉 [防火牆設定] 頁面。

您現在可以使用這個 IP 位址連線到 SQL Server 及其資料倉儲。 可從 SQL Server Management Studio 或您選擇的另一個工具來運作連線。 當您連線時，請使用先前建立的 ServerAdmin 帳戶。  

> [!IMPORTANT]
> 根據預設，已對所有 Azure 服務啟用透過 SQL Database 防火牆存取。 在此頁面上按一下 [關閉]，然後按一下 [儲存] 可停用所有 Azure 服務的防火牆。

## <a name="get-the-fully-qualified-server-name"></a>取得完整的伺服器名稱

請在 Azure 入口網站中取得 SQL 伺服器的完整伺服器名稱。 稍後您在連線到伺服器時，會使用完整伺服器名稱。

1. 登入 [Azure 入口網站](https://portal.azure.com/)。
2. 從左側功能表中選取 [SQL 資料倉儲]，在 [SQL 資料倉儲] 頁面上按一下您的資料倉儲。 
3. 在 Azure 入口網站中您資料庫的 [基本資訊] 窗格中，找到後複製 [伺服器名稱]。 在此範例中，完整名稱是 mynewserver 20180430.database.windows.net。 

    ![連線資訊](media/load-data-from-azure-blob-storage-using-polybase/find-server-name.png)  

## <a name="connect-to-the-server-as-server-admin"></a>以伺服器系統管理員身分連線到伺服器

本節使用 [SQL Server Management Studio](/sql/ssms/download-sql-server-management-studio-ssms) (SSMS) 建立對 Azure SQL Server 的連線。

1. 開啟 SQL Server Management Studio。

2. 在 [連接到伺服器] 對話方塊中，輸入下列資訊：

   | 設定       | 建議的值 | 說明 | 
   | ------------ | ------------------ | ------------------------------------------------- | 
   | 伺服器類型 | 資料庫引擎 | 這是必要值 |
   | 伺服器名稱 | 完整伺服器名稱 | 以下是範例：**mynewserver-20180430.database.windows.net**。 |
   | 驗證 | SQL Server 驗證 | SQL 驗證是本教學課程中設定的唯一驗證類型。 |
   | 登入 | 伺服器管理帳戶 | 這是您在建立伺服器時所指定的帳戶。 |
   | 密碼 | 伺服器管理帳戶的密碼 | 這是您在建立伺服器時所指定的密碼。 |

    ![連接到伺服器](media/load-data-from-azure-blob-storage-using-polybase/connect-to-server.png)

4. 按一下 [ **連接**]。 [物件總管] 視窗隨即在 SSMS 中開啟。 

5. 在 [物件總管] 中，展開 [資料庫]。 然後展開 [mySampleDatabase] 可檢視新資料庫中的物件。

    ![資料庫物件](media/create-data-warehouse-portal/connected.png) 

## <a name="run-some-queries"></a>執行一些查詢

SQL 資料倉儲會使用 T-SQL 作為查詢語言。 若要開啟查詢視窗並執行一些 T-SQL 查詢，請使用下列步驟：

1. 以滑鼠右鍵按一下 [mySampleDataWarehouse]，然後選取 [新增查詢]。  新的查詢視窗隨即開啟。
2. 在新的查詢視窗中，輸入下列命令可查看資料庫清單。

    ```sql
    SELECT * FROM sys.databases
    ```

3. 按一下 [Execute (執行)] 。  查詢結果顯示兩個資料庫：**master** 和 **mySampleDataWarehouse**。

    ![查詢資料庫](media/create-data-warehouse-portal/query-databases.png)

4. 若要查看一些資料，請使用下列命令以查看姓氏為 Adams 且家裡有三個孩子的客戶數目。 結果列出 6 個客戶。 

    ```sql
    SELECT LastName, FirstName FROM dbo.dimCustomer
    WHERE LastName = 'Adams' AND NumberChildrenAtHome = 3;
    ```

    ![查詢 dbo.dimCustomer](media/create-data-warehouse-portal/query-customer.png)

## <a name="clean-up-resources"></a>清除資源

您需對資料倉儲單位和資料倉儲上儲存的資料付費。 這些計算和儲存體資源會分開計費。 

- 如果您需要將資料保留在儲存體中，可以在您不使用資料倉儲時暫停計算。 暫停計算，您只需支付資料儲存體的費用。 當您準備要使用資料時，您可以繼續計算。
- 如果您需要移除未來的費用，可以將資料倉儲刪除。 

遵循下列步驟，視需要清除資源。

1. 登入 [Azure 入口網站](https://portal.azure.com)，按一下您的資料倉儲。

    ![清除資源](media/load-data-from-azure-blob-storage-using-polybase/clean-up-resources.png)

1. 若要暫停計算，請按一下 [暫停] 按鈕。 資料倉儲暫停時，您會看到 [啟動] 按鈕。  若要繼續計算，請按一下 [啟動]。

2. 若要移除資料倉儲，而不再支付計算或儲存體的費用，請按一下 [刪除]。

3. 若要移除您所建立的 SQL Server，請按一下先前映像中的 [mynewserver 20180430.database.windows.net]，然後按一下 [刪除]。  請謹慎使用刪除，因為刪除伺服器也會刪除所有指派給伺服器的資料庫。

4. 若要移除此資源群組，請按一下 [myResourceGroup]，然後按一下 [刪除資源群組]。


## <a name="next-steps"></a>後續步驟
您現在已建立資料倉儲、建立防火牆規則、連線到您的資料倉儲，並執行了一些查詢。 若要深入了解 Azure SQL 資料倉儲，請繼續進行載入資料的教學課程。
> [!div class="nextstepaction"]
>[將資料載入 SQL 資料倉儲](load-data-from-azure-blob-storage-using-polybase.md)
