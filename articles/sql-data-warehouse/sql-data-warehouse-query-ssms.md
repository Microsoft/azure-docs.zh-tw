---
title: 連線到 Azure SQL 資料倉儲 - SSMS | Microsoft Docs
description: 使用 SQL Server Management Studio (SSMS) 連接及查詢 Azure SQL 資料倉儲。
services: sql-data-warehouse
author: kavithaj
manager: craigg
ms.service: sql-data-warehouse
ms.topic: conceptual
ms.component: consume
ms.date: 04/17/2018
ms.author: kavithaj
ms.reviewer: igorstan
ms.openlocfilehash: 6079c3064699da38fad20468517eb97d6ab107f8
ms.sourcegitcommit: 1fb353cfca800e741678b200f23af6f31bd03e87
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/30/2018
ms.locfileid: "43307195"
---
# <a name="connect-to-sql-data-warehouse-with-sql-server-management-studio-ssms"></a>連接 SQL 資料倉儲與 SQL Server Management Studio (SSMS)
> [!div class="op_single_selector"]
> * [Power BI](sql-data-warehouse-get-started-visualize-with-power-bi.md)
> * [Azure Machine Learning](sql-data-warehouse-get-started-analyze-with-azure-machine-learning.md)
> * [Visual Studio](sql-data-warehouse-query-visual-studio.md)
> * [sqlcmd](sql-data-warehouse-get-started-connect-sqlcmd.md) 
> * [SSMS](sql-data-warehouse-query-ssms.md)
> 
> 

使用 SQL Server Management Studio (SSMS) 連接及查詢 Azure SQL 資料倉儲。 

## <a name="prerequisites"></a>必要條件
若要使用本教學課程，您需要：

* 現有的 SQL 資料倉儲。 若要建立資料倉儲，請參閱 [建立 SQL 資料倉儲][Create a SQL Data Warehouse]。
* SQL Server Management Studio (SSMS) 已安裝。 如果您尚未安裝，可以免費[安裝 SSMS][Install SSMS]。
* 完整的 SQL 伺服器名稱。 若要找到此名稱，請參閱 [連線至 SQL 資料倉儲][Connect to SQL Data Warehouse]。

## <a name="1-connect-to-your-sql-data-warehouse"></a>1.連接到您的 SQL 資料倉儲
1. 開啟 SSMS。
2. 開啟物件總管。 若要這樣做，請選取 [檔案]  >  [連接物件總管]。
   
    ![SQL Server 物件總管][1]
3. 填寫 [連線到伺服器] 視窗中的欄位。
   
    ![連線到伺服器][2]
   
   * **伺服器名稱**。 輸入先前找到的 **伺服器名稱** 。
   * **驗證**。 選取 [SQL Server 驗證] 或 [Active Directory 整合式驗證]。
   * [使用者名稱] 和 [密碼]。 如果上面已選取 [SQL Server 驗證]，請輸入使用者名稱和密碼。
   * 按一下 [ **連接**]。
4. 若要瀏覽，請展開您的 Azure SQL 伺服器。 您可以檢視與伺服器相關聯的資料庫。 展開 AdventureWorksDW 以查看範例資料庫中的資料表。
   
    ![探索 AdventureWorksDW][3]

## <a name="2-run-a-sample-query"></a>2.執行範例查詢
現已建立對您的資料庫的連線，接著繼續撰寫查詢。

1. 在 [SQL Server 物件總管] 中您的資料庫上按一下滑鼠右鍵。
2. 選取 [新增查詢] 。 新的查詢視窗隨即開啟。
   
    ![新增查詢][4]
3. 將此 TSQL 查詢複製到查詢視窗中：
   
    ```sql
    SELECT COUNT(*) FROM dbo.FactInternetSales;
    ```
4. 執行查詢。 若要這麼做，請按一下 `Execute`，或使用下列快速鍵：`F5`。
   
    ![執行查詢][5]
5. 查看查詢結果。 在此範例中，FactInternetSales 資料表有 60398 個資料列。
   
    ![查詢結果][6]

## <a name="next-steps"></a>後續步驟
您現在可以連線並查詢，請嘗試[使用 PowerBI 將資料視覺化][visualizing the data with PowerBI]。

若要針對 Azure Active Directory 驗證設定您的環境，請參閱[驗證 SQL 資料倉儲][Authenticate to SQL Data Warehouse]。

<!--Arcticles-->
[Connect to SQL Data Warehouse]: sql-data-warehouse-connect-overview.md
[Create a SQL Data Warehouse]: sql-data-warehouse-get-started-provision.md
[Authenticate to SQL Data Warehouse]: sql-data-warehouse-authentication.md
[visualizing the data with PowerBI]: sql-data-warehouse-get-started-visualize-with-power-bi.md 

<!--Other-->
[Azure portal]: https://portal.azure.com
[Install SSMS]: https://msdn.microsoft.com/library/hh213248.aspx


<!--Image references-->

[1]: media/sql-data-warehouse-query-ssms/connect-object-explorer.png
[2]: media/sql-data-warehouse-query-ssms/connect-object-explorer1.png
[3]: media/sql-data-warehouse-query-ssms/explore-tables.png
[4]: media/sql-data-warehouse-query-ssms/new-query.png
[5]: media/sql-data-warehouse-query-ssms/execute-query.png
[6]: media/sql-data-warehouse-query-ssms/results.png
