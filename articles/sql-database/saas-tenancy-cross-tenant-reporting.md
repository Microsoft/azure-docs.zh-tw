---
title: 跨多個 Azure SQL Database 執行報表查詢 | Microsoft Docs
description: 使用分散式查詢跨租用戶報告。
services: sql-database
ms.service: sql-database
ms.subservice: scenario
ms.custom: ''
ms.devlang: ''
ms.topic: conceptual
author: stevestein
ms.author: sstein
ms.reviewers: billgib,AyoOlubeko
manager: craigg
ms.date: 04/01/2018
ms.openlocfilehash: 3bba0eb552338f1b436ea25306a84029d352f1f3
ms.sourcegitcommit: 715813af8cde40407bd3332dd922a918de46a91a
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "47055283"
---
# <a name="cross-tenant-reporting-using-distributed-queries"></a>使用分散式查詢跨租用戶報告

在本教學課程中，您會在整個租用戶資料庫集合執行分散式查詢以報告。 這些查詢可以擷取藏在 Wingtip Tickets SaaS 租用戶日常操作資料中的深入解析。 若要這麼做，您可將額外的報表資料庫部署到目錄伺服器，並使用「彈性查詢」來啟用分散式查詢。


您會在本教學課程中學到：

> [!div class="checklist"]

> * 如何部署報表資料庫
> * 如何對所有租用戶資料庫執行分散式查詢
> * 每個資料庫中的全域檢視可以如何跨租用戶啟用高效率查詢


若要完成本教學課程，請確定已完成下列必要條件：


* 已部署 Wingtip Tickets SaaS Database Per Tenant 應用程式。 若要在五分鐘內完成部署，請參閱[部署及探索 Wingtip Tickets SaaS Database Per Tenant 應用程式](saas-dbpertenant-get-started-deploy.md)
* 已安裝 Azure PowerShell。 如需詳細資料，請參閱[開始使用 Azure PowerShell](https://docs.microsoft.com/powershell/azure/get-started-azureps)
* 已安裝 SQL Server Management Studio (SSMS)。 若要下載和安裝 SSMS，請參閱[下載 SQL Server Management Studio (SSMS)](https://docs.microsoft.com/sql/ssms/download-sql-server-management-studio-ssms)。


## <a name="cross-tenant-reporting-pattern"></a>跨租用戶報告模式

![跨租用戶分散式查詢模式](media/saas-tenancy-cross-tenant-reporting/cross-tenant-distributed-query.png)

SaaS 應用程式的其中一個機會是使用儲存在雲端的大量租用戶資料，深入解析應用程式的作業與使用方式。 這些深入解析可以引導應用程式和服務中的功能開發、使用性改進及其他投資。

在單一多租用戶資料庫中存取此資料很容易，但資料大規模分散於可能數千個資料庫時則不太容易存取。 其中一個方法是使用[彈性查詢](sql-database-elastic-query-overview.md)，這可對一組具有共用結構描述的分散式資料庫啟用查詢。 這些資料庫可以分散於不同的資源群組和訂用帳戶，但需要共用一般登入。 彈性查詢會使用單一 head 資料庫，其中定義的外部資料表會鏡射分散式 (租用戶) 資料庫中的資料表或檢視。 提交至此 head 資料庫的查詢會經過編譯，以產生分散式查詢計劃 (包含視需要往下推送到租用戶資料庫的查詢部分)。 彈性查詢會使用目錄資料庫中的分區對應，判斷所有租用戶資料庫的位置。 前端資料庫的安裝程式和查詢會直接使用標準 [Transact-SQL](https://docs.microsoft.com/sql/t-sql/language-reference)，以及支援從 Power BI 和 Excel 等工具進行查詢。

彈性查詢將查詢分散到整個租用戶資料庫，能夠立即深入了解即時的實際執行資料。 因為彈性查詢可能會從多個資料庫提取資料，所以查詢延遲會高於提交至單一多租用戶資料庫的對等查詢。 設計查詢來最小化前端資料庫傳回的資料。 彈性查詢通常最適合查詢少量的即時資料，而非建立常用或複雜的分析查詢或報告。 如果查詢的效能不佳，請查看[執行計畫](https://docs.microsoft.com/sql/relational-databases/performance/display-an-actual-execution-plan)以查看查詢的哪個部分已向下推送至遠端資料庫，以及傳回多少資料。 需要複雜彙總或分析處理的查詢，在某些情況下透過將租用戶資料擷取到針對分析查詢最佳化的資料庫或資料倉儲來處理，可能會比較好。 [租用戶分析教學課程](saas-tenancy-tenant-analytics.md)會說明此模式。 

## <a name="get-the-wingtip-tickets-saas-database-per-tenant-application-scripts"></a>取得 Wingtip Tickets SaaS Database Per Tenant 應用程式指令碼

可在 [WingtipTicketsSaaS-DbPerTenant](https://github.com/Microsoft/WingtipTicketsSaaS-DbPerTenant) GitHub 存放庫中使用 Wingtip Tickets SaaS 多租用戶資料庫指令碼和應用程式來源程式碼。 關於下載和解除封鎖 Wingtip Tickets SaaS 指令碼的步驟，請參閱[一般指引](saas-tenancy-wingtip-app-guidance-tips.md)。

## <a name="create-ticket-sales-data"></a>建立票證銷售資料

若要針對更有趣的資料集執行查詢，請執行票證產生器來建立票證銷售資料。

1. 在 PowerShell ISE 中開啟 ...\\Learning Modules\\Operational Analytics\\Adhoc Reporting\\Demo-AdhocReporting.ps1 指令碼，然後設定下列值：
   * **$DemoScenario** = 1，**購買各地事件的票證**。
2. 按 **F5** 以執行指令碼並產生票證銷售。 執行指令碼時，請繼續本教學課程中的步驟。 票證資料會在 [執行臨機操作分散式查詢] 區段中查詢，請等候它完成。

## <a name="explore-the-global-views"></a>瀏覽全域檢視

在 Wingtip Tickets SaaS 每一租用戶一個資料庫應用程式中，會提供每個租用戶一個資料庫。 因此，資料庫資料表中所包含資料的範圍僅限單一租用戶。 不過，在跨所有資料庫查詢時，務必讓彈性查詢可以將資料視為是依租用戶分區之單一邏輯資料庫的一部分。 

若要模擬此模式，需將一組「全域」檢視新增至租用戶資料庫，將租用戶識別碼對應到全域查詢的每個資料表。 例如，*VenueEvents* 檢視將計算的 *VenueId* 新增至從 *Events* 資料表對應的資料行。 同樣地，*VenueTicketPurchases* 和 *VenueTickets* 檢視可從對應的資料表新增經計算的預計 *VenueId* 資料行。 「彈性查詢」會使用這些檢視來平行處理查詢，並在 *VenueId* 資料行存在時，將其推送到適當的遠端租用戶資料庫。 這會大幅減少傳回的資料量，並大幅增加許多查詢的效能。 這些全域檢視已在所有租用戶資料庫預先建立。

1. 開啟 SSMS 並[連線到 tenants1-&lt;USER&gt; 伺服器](saas-tenancy-wingtip-app-guidance-tips.md#explore-database-schema-and-execute-sql-queries-using-ssms)。
1. 展開 [資料庫]，以滑鼠右鍵按一下 _contosoconcerthall_，然後選取 [新增查詢]。
1. 執行下列查詢來探索單一租用戶資料表與全域檢視之間的差異︰

   ```T-SQL
   -- The base Venue table, that has no VenueId associated.
   SELECT * FROM Venue

   -- Notice the plural name 'Venues'. This view projects a VenueId column.
   SELECT * FROM Venues

   -- The base Events table, which has no VenueId column.
   SELECT * FROM Events

   -- This view projects the VenueId retrieved from the Venues table.
   SELECT * FROM VenueEvents
   ```

在這些檢視中，*VenueId* 是以 Venue 名稱的雜湊來計算，但您可以使用任何方法來導入唯一值。 這個方法與計算用於目錄中租用戶金鑰的方法類似。

檢查 *Venues* 檢視的定義：

1. 在 [物件總管] 中，展開 [contosoconcethall] > [檢視]：

   ![檢視](media/saas-tenancy-cross-tenant-reporting/views.png)

2. 以滑鼠右鍵按一下 **dbo.Venues**。
3. 選取 [產生檢視的指令碼為] > [CREATE To] > [新的查詢編輯器視窗]

產生其他任何 *Venue* 檢視的指令碼，以查看如何新增 *VenueId*。

## <a name="deploy-the-database-used-for-distributed-queries"></a>部署用於分散式查詢的資料庫

此練習會部署 adhocreporting 資料庫。 這就是包含用來查詢所有租用戶資料庫之結構描述的 head 資料庫。 此資料庫會部署到現有的目錄伺服器，也就是在範例應用程式中供所有管理相關資料庫使用的伺服器。

1. 在 PowerShell ISE 中開啟 ...\\Learning Modules\\Operational Analytics\\Adhoc Reporting\\Demo-AdhocReporting.ps1。 

1. 設定 **$DemoScenario = 2**，_部署臨機操作報表資料庫_。

1. 按 **F5** 以執行指令碼並建立 adhocreporting 資料庫。

在下節中，您要將結構描述新增至資料庫，以便能夠用來執行分散式查詢。

## <a name="configure-the-head-database-for-running-distributed-queries"></a>設定 'head' 資料庫來執行分散式查詢

此練習將結構描述 (外部資料來源和外部資料表定義) 新增至 _adhocreporting_ 資料庫，讓您能跨所有租用戶資料庫進行查詢。

1. 狀況良好 - 若報表資料庫、報表 Web 服務及報表管理服務的 URL 與加密金鑰均已設定。狀況良好 - 若報表資料庫、報表 Web 服務及報表管理服務的 URL 與加密金鑰均已設定。開啟 SQL Server Management Studio，並連線到您在上一個步驟中建立的隨選報表資料庫。 資料庫的名稱是 adhocreporting。
2. 在 SSMS 中開啟 ...\Learning Modules\Operational Analytics\Adhoc Reporting\ Initialize-AdhocReportingDB.sql。
3. 檢閱 SQL 指令碼並注意︰

   彈性查詢會使用資料庫範圍的認證來存取每個租用戶資料庫。 此認證必須可用於所有資料庫，且通常應獲得啟用這些查詢所需的最小權限。

    ![建立認證](media/saas-tenancy-cross-tenant-reporting/create-credential.png)

   使用目錄資料庫作為外部資料來源後，在執行查詢時，查詢會分散到在目錄中註冊的所有資料庫。 因為每個部署的伺服器名稱不同，所以此指令碼會從指令碼執行所在的目前伺服器 (@@servername)，取得目錄資料庫的位置。

    ![建立外部資料來源](media/saas-tenancy-cross-tenant-reporting/create-external-data-source.png)

   參考前一節所述之全域檢視，並使用 **DISTRIBUTION = SHARDED(VenueId)** 定義的外部資料表。 因為每個 *VenueId* 都對應到單一資料庫，所以這可改善許多情況 (如下一節所示) 的效能。

    ![建立外部資料表](media/saas-tenancy-cross-tenant-reporting/external-tables.png)

   已建立和填入的本機資料表 _VenueTypes_。 此參考資料表在所有租用戶資料庫中很常見，因此在此可以表示為本機資料表，並使用一般的資料填入。 對於某些查詢，讓此資料表在前端資料庫中定義，可以減少必須移到前端資料庫的資料量。

    ![建立資料表](media/saas-tenancy-cross-tenant-reporting/create-table.png)

   如果您以這種方式包含參考資料表，每當您更新租用戶資料庫時，請務必更新資料表結構描述和資料。

4. 按 **F5** 以執行指令碼並初始化 adhocreporting 資料庫。 

現在您可以執行分散式查詢，然後收集所有租用戶之間的深入資訊！

## <a name="run-distributed-queries"></a>執行分散式查詢

既然 adhocreporting 資料庫已完成設定，請繼續執行一些分散式查詢。 請包含執行計畫，以便進一步了解發生查詢處理的位置。 

檢查執行計畫時，將滑鼠停留在計畫圖示上方，以取得詳細資料。 

值得注意的是，定義外部資料來源時的設定 **DISTRIBUTION = SHARDED(VenueId)** 可改善許多情況的效能。 因為每個 *VenueId* 都對應到單一資料庫，所以篩選會輕鬆地在遠端完成，只傳回所需的資料。

1. 在 SSMS 中開啟 ...\\Learning Modules\\Operational Analytics\\Adhoc Reporting\\Demo-AdhocReportingQueries.sql。
2. 確定您已連線到 **adhocreporting** 資料庫。
3. 選取 [查詢] 功能表，然後按一下 [包括實際執行計畫]
4. 反白顯示 [Which venues are currently registered?] \(目前註冊哪些地點?\) 查詢，然後按 **F5**。

   此查詢會傳回整份地點清單，說明跨所有租用戶進行查詢非常快速而且容易，並傳回每個租用戶的資料。

   檢查計劃，您會發現全部成本都是遠端查詢。每個租用戶資料庫都會遠端執行查詢，並且將其場地資訊傳回給前端資料庫。

   ![SELECT * FROM dbo.Venues](media/saas-tenancy-cross-tenant-reporting/query1-plan.png)

5. 選取下一個查詢，然後按 **F5**。

   此查詢會聯結租用戶資料庫的資料和本機 VenueTypes 資料表的資料 (本機是因為它是 adhocreporting 資料庫中的資料表)。

   檢查計畫，您會發現大部分的成本是遠端查詢。 每個租用戶資料庫傳回其地點資料，並執行本機聯結，以便本機 *VenueTypes* 資料表顯示易記名稱。

   ![遠端資料與本機資料的聯結](media/saas-tenancy-cross-tenant-reporting/query2-plan.png)

6. 現在選取 [On which day were the most tickets sold?] \(哪一天銷售最多票證?\) 查詢，然後按 **F5**。

   此查詢會執行比較複雜的聯結和彙總。 大部分處理是在遠端發生。  只有其中包含每個場地每日票證銷售計數的單一資料列，會傳回給前端資料庫。

   ![query](media/saas-tenancy-cross-tenant-reporting/query3-plan.png)


## <a name="next-steps"></a>後續步驟

在本教學課程中，您已了解如何：

> [!div class="checklist"]

> * 對所有租用戶資料庫執行分散式查詢
> * 部署報表資料庫，並且定義執行分散式查詢所需的結構描述。


現在可以嘗試一下[租用戶分析教學課程](saas-tenancy-tenant-analytics.md)，了解如何將資料擷取到另一個分析資料庫，進行更複雜的分析處理。

## <a name="additional-resources"></a>其他資源

* [在 Tickets SaaS Database Per Tenant 應用程式上建置的其他教學課程](saas-dbpertenant-wingtip-app-overview.md#sql-database-wingtip-saas-tutorials)
* [彈性查詢](sql-database-elastic-query-overview.md)
