---
title: 跨多個 Azure SQL Database 執行隨選報表查詢 | Microsoft Docs
description: 在多租用戶應用程式範例中，跨多個 SQL 資料庫執行隨選報表查詢。
keywords: SQL Database Azure
services: sql-database
author: stevestein
manager: craigg
ms.service: sql-database
ms.custom: scale out apps
ms.workload: Inactive
ms.tgt_pltfrm: na
ms.devlang: ''
ms.topic: conceptual
ms.date: 04/01/2018
ms.author: AyoOlubeko
ms.openlocfilehash: 95ea5f2a40bd629384e1bc1f7d707eb30d9eaf5f
ms.sourcegitcommit: 266fe4c2216c0420e415d733cd3abbf94994533d
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/01/2018
ms.locfileid: "34644268"
---
# <a name="run-ad-hoc-analytics-queries-across-multiple-azure-sql-databases"></a>跨多個 Azure SQL Database 執行臨機操作分析查詢

在本教學課程中，您會在整個租用戶資料庫集合執行分散式查詢以啟用臨機操作互動式報表。 這些查詢可以擷取藏在 Wingtip Tickets SaaS 應用程式日常操作資料中的深入解析。 若要執行這些擷取，您要將額外的分析資料庫部署到目錄伺服器，並使用「彈性查詢」來啟用分散式查詢。


您會在本教學課程中學到：

> [!div class="checklist"]

> * 如何部署隨選報表資料庫
> * 如何對所有租用戶資料庫執行分散式查詢


若要完成本教學課程，請確定已完成下列必要條件：

* 已部署 Wingtip Tickets SaaS 多租用戶資料庫應用程式。 若要在五分鐘內完成部署，請參閱[部署及探索 Wingtip Tickets SaaS 多租用戶資料庫應用程式](saas-multitenantdb-get-started-deploy.md)
* 已安裝 Azure PowerShell。 如需詳細資料，請參閱[開始使用 Azure PowerShell](https://docs.microsoft.com/powershell/azure/get-started-azureps)
* 已安裝 SQL Server Management Studio (SSMS)。 若要下載和安裝 SSMS，請參閱[下載 SQL Server Management Studio (SSMS)](https://docs.microsoft.com/sql/ssms/download-sql-server-management-studio-ssms)。


## <a name="ad-hoc-reporting-pattern"></a>隨選報表模式

![隨選報表模式](media/saas-multitenantdb-adhoc-reporting/adhocreportingpattern_shardedmultitenantDB.png)

SaaS 應用程式可以分析集中儲存在雲端中的大量租用戶資料。 分析會揭露深入了解應用程式的作業與使用方式。 這些深入解析可以引導應用程式和服務中的功能開發、使用性改進及其他投資。

在單一多租用戶資料庫中存取此資料很容易，但資料大規模分散於可能數千個資料庫時則不太容易存取。 其中一個方法是使用[彈性查詢](sql-database-elastic-query-overview.md)，這可對一組具有共用結構描述的分散式資料庫啟用查詢。 這些資料庫可以分散於不同的資源群組和訂用帳戶。 但一個一般登入必須可從所有資料庫存取擷取資料。 彈性查詢會使用單一 head 資料庫，其中定義的外部資料表會鏡射分散式 (租用戶) 資料庫中的資料表或檢視。 提交至此 head 資料庫的查詢會經過編譯，以產生分散式查詢計劃 (包含視需要往下推送到租用戶資料庫的查詢部分)。 彈性查詢會使用目錄資料庫中的分區對應，判斷所有租用戶資料庫的位置。 安裝程式和查詢會直接使用標準 [Transact-SQL](https://docs.microsoft.com/sql/t-sql/language-reference)，以及支援從 Power BI 和 Excel 等工具進行臨機操作查詢。

彈性查詢將查詢分散到整個租用戶資料庫，能夠立即深入了解即時的實際執行資料。 不過，因為彈性查詢可能會從多個資料庫提取資料，所以查詢延遲有時可能會高於提交至單一多租用戶資料庫的對等查詢。 請務必設計查詢來最小化傳回的資料。 彈性查詢通常最適合查詢少量的即時資料，而非建立常用或複雜的分析查詢或報告。 如果查詢的效能不佳，請查看[執行計畫](https://docs.microsoft.com/sql/relational-databases/performance/display-an-actual-execution-plan)以查看查詢的哪個部分已向下推送至遠端資料庫。 並評估會傳回多少資料。 需要複雜分析處理的查詢，透過將擷取的租用戶資料儲存到針對分析查詢最佳化的資料庫來提供服務，可能會比較好。 SQL Database 和 SQL 資料倉儲可以裝載此類分析資料庫。

[租用戶分析教學課程](saas-multitenantdb-tenant-analytics.md)會說明此分析模式。

## <a name="get-the-wingtip-tickets-saas-multi-tenant-database-application-source-code-and-scripts"></a>取得 Wingtip Tickets SaaS 多租用戶資料庫應用程式原始碼和指令碼

可在 [WingtipTicketsSaaS MultitenantDB](https://github.com/microsoft/WingtipTicketsSaaS-MultiTenantDB) GitHub 存放庫中使用 Wingtip Tickets SaaS 多租用戶資料庫指令碼和應用程式來源程式碼。 關於下載和解除封鎖 Wingtip Tickets SaaS 指令碼的步驟，請參閱[一般指引](saas-tenancy-wingtip-app-guidance-tips.md)。

## <a name="create-ticket-sales-data"></a>建立票證銷售資料

若要針對更有趣的資料集執行查詢，請執行票證產生器來建立票證銷售資料。

1. 在 PowerShell ISE 中開啟 ...\\Learning Modules\\Operational Analytics\\Adhoc Reporting\\Demo-AdhocReporting.ps1 指令碼，然後設定下列值：
   * **$DemoScenario** = 1，**購買各地事件的票證**。
2. 按 **F5** 以執行指令碼並產生票證銷售。 執行指令碼時，請繼續本教學課程中的步驟。 票證資料會在 [執行臨機操作分散式查詢] 區段中查詢，請等候它完成。

## <a name="explore-the-tenant-tables"></a>探索租用戶資料表 

在 Wingtip Tickets SaaS 多租用戶資料庫應用程式中，租用戶會儲存在混合式租用戶管理模型中 - 當中的租用戶資料會儲存在多租用戶資料庫或單一租用戶資料庫中，且可以在兩者之間移動。 在跨所有租用戶資料庫查詢時，務必讓彈性查詢可以將資料視為是依租用戶分區之單一邏輯資料庫的一部分。 

若要達到此模式，所有租用戶資料表都要包括 VenueId，指出資料所屬的租用戶。 VenueId 是以 Venue 名稱的雜湊來計算，但您可以使用任何方法來導入此資料行的唯一值。 這個方法與計算用於目錄中租用戶金鑰的方法類似。 「彈性查詢」會使用包含 VenueId 的資料表來平行處理查詢，並將其推送到適當的遠端租用戶資料庫。 這將可大幅減少所傳回的資料量並導致增加效能，特別是當多個租用戶的資料儲存在單一租用戶資料庫時。

## <a name="deploy-the-database-used-for-ad-hoc-distributed-queries"></a>部署用於臨機操作分散式查詢的資料庫

此練習會部署 adhocreporting 資料庫。 這就是包含用來查詢所有租用戶資料庫之結構描述的 head 資料庫。 此資料庫會部署到現有的目錄伺服器，也就是在範例應用程式中供所有管理相關資料庫使用的伺服器。

1. 在 [PowerShell ISE] 中開啟 ...\\Learning Modules\\Operational Analytics\\Adhoc Reporting\\Demo-AdhocReporting.ps1 並設定下列值：
   * **$DemoScenario** = 2，**部署臨機操作分析資料庫**。

2. 按 **F5** 以執行指令碼並建立 adhocreporting 資料庫。

在下節中，您要將結構描述新增至資料庫，以便能夠用來執行分散式查詢。

## <a name="configure-the-head-database-for-running-distributed-queries"></a>設定 'head' 資料庫來執行分散式查詢

此練習將結構描述 (外部資料來源和外部資料表定義) 新增至隨選報表資料庫，讓您能跨所有租用戶資料庫進行查詢。

1. 開啟 SQL Server Management Studio，並連線到您在上一個步驟中建立的隨選報表資料庫。 資料庫的名稱是 adhocreporting。
2. 在 SSMS 中開啟 ...\Learning Modules\Operational Analytics\Adhoc Reporting\ Initialize-AdhocReportingDB.sql。
3. 檢閱 SQL 指令碼並注意下列事項︰

   彈性查詢會使用資料庫範圍的認證來存取每個租用戶資料庫。 此認證必須可用於所有資料庫，且通常應獲得啟用這些臨機操作查詢所需的最小權限。

    ![建立認證](media/saas-multitenantdb-adhoc-reporting/create-credential.png)

   使用目錄資料庫作為外部資料來源後，在執行查詢時，查詢會分散到在目錄中註冊的所有資料庫。 因為每個部署的伺服器名稱不同，所以此初始化指令碼會擷取指令碼執行所在的目前伺服器 (@@servername)，取得目錄資料庫的位置。

    ![建立外部資料來源](media/saas-multitenantdb-adhoc-reporting/create-external-data-source.png)

   參考租用戶資料表的外部資料表會以 **DISTRIBUTION = SHARDED(VenueId)** 來定義。 這會將特定 VenueId 的查詢路由至適當資料庫，並改善許多情節的效能，如下一節中所示。

    ![建立外部資料表](media/saas-multitenantdb-adhoc-reporting/external-tables.png)

   已建立和填入的本機資料表 *VenueTypes*。 此參考資料表在所有租用戶資料庫中很常見，因此在此可以表示為本機資料表，並使用一般的資料填入。 對於某些查詢而言，這可能會減少租用戶資料庫與 adhocreporting 資料庫之間移動的資料量。

    ![建立資料表](media/saas-multitenantdb-adhoc-reporting/create-table.png)

   如果您以這種方式包含參考資料表，每當您更新租用戶資料庫時，請務必更新資料表結構描述和資料。

4. 按 **F5** 以執行指令碼並初始化 adhocreporting 資料庫。 

現在您可以執行分散式查詢，然後收集所有租用戶之間的深入資訊！

## <a name="run-ad-hoc-distributed-queries"></a>執行臨機操作分散式查詢

既然 adhocreporting 資料庫已完成設定，請繼續執行一些分散式查詢。 請包含執行計畫，以便進一步了解發生查詢處理的位置。 

檢查執行計畫時，將滑鼠停留在計畫圖示上方，以取得詳細資料。 

1. 在 *SSMS* 中開啟 ...\\Learning Modules\\Operational Analytics\\Adhoc Reporting\\*Demo-AdhocReportingQueries.sql*。
2. 確定您已連線到 **adhocreporting** 資料庫。
3. 選取 [查詢] 功能表，然後按一下 [包括實際執行計畫]
4. 反白顯示 [Which venues are currently registered?] \(目前註冊哪些地點?\) 查詢，然後按 **F5**。

   此查詢會傳回整份地點清單，說明跨所有租用戶進行查詢非常快速而且容易，並傳回每個租用戶的資料。

   檢查計畫，您會發現全部成本都是遠端查詢，因為我們只是到每個租用戶資料庫，選取地點資訊。

   ![SELECT * FROM dbo.Venues](media/saas-multitenantdb-adhoc-reporting/query1-plan.png)

5. 選取下一個查詢，然後按 **F5**。

   此查詢會聯結租用戶資料庫的資料和本機 VenueTypes 資料表的資料 (本機是因為它是 adhocreporting 資料庫中的資料表)。

   檢查計畫，您會發現大多數的成本是遠端查詢，因為我們查詢每個租用戶的地點資訊 (dbo.Venues)，然後與本機 *VenueTypes* 資料表執行快速的本機聯結，以顯示易記名稱。

   ![遠端資料與本機資料的聯結](media/saas-multitenantdb-adhoc-reporting/query2-plan.png)

6. 現在選取 [On which day were the most tickets sold?] \(哪一天銷售最多票證?\) 查詢，然後按 **F5**。

   此查詢會執行比較複雜的聯結和彙總。 值得注意的是，大部分處理都在遠端電腦上進行，同樣地，我們只取回我們所需的資料列，只傳回每天每個地點彙總的票證銷售計數的單一資料列。

   ![query](media/saas-multitenantdb-adhoc-reporting/query3-plan.png)


## <a name="next-steps"></a>後續步驟

在本教學課程中，您已了解如何：

> [!div class="checklist"]

> * 對所有租用戶資料庫執行分散式查詢
> * 部署隨選報表資料庫，並將結構描述新增至資料庫中，以執行分散式查詢。

現在可以嘗試一下[租用戶分析教學課程](saas-multitenantdb-tenant-analytics.md)，了解如何將資料擷取到另一個分析資料庫，進行更複雜的分析處理。

## <a name="additional-resources"></a>其他資源

<!-- ??
* Additional [tutorials that build upon the Wingtip Tickets SaaS Multi-tenant Database application](saas-multitenantdb-wingtip-app-overview.md#sql-database-wingtip-saas-tutorials)
-->

* [彈性查詢](sql-database-elastic-query-overview.md)
