---
title: 針對 Azure SQL Database 執行分析查詢 | Microsoft Docs
description: 在多租用戶應用程式中使用從多個 Azure SQL Database 資料庫擷取的資料執行跨租用戶分析查詢。
services: sql-database
ms.service: sql-database
ms.subservice: scenario
ms.custom: ''
ms.devlang: ''
ms.topic: conceptual
author: stevestein
ms.author: sstein
ms.reviewer: anjangsh,billgib,genemi
manager: craigg
ms.date: 09/19/2018
ms.openlocfilehash: 5de707f3f2e6a82d880363eea91fb8ce644fb3aa
ms.sourcegitcommit: 715813af8cde40407bd3332dd922a918de46a91a
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "47055032"
---
# <a name="cross-tenant-analytics-using-extracted-data---multi-tenant-app"></a>使用擷取的資料執行跨租用戶分析 - 多租用戶應用程式
 
在此教學課程中，您會逐步完成多租用戶實作的整個分析案例。 案例會示範分析如何讓企業做出智慧的決策。 使用從分區資料庫擷取的資料，您使用分析來取得租用戶行為的深入解析，包括他們對於範例 Wingtip Tickets SaaS 應用程式的使用。 這個案例牽涉到三個步驟： 

1.  從每個租用戶資料庫**擷取資料**至分析存放區。
2.  針對分析處理**最佳化擷取的資料**。
3.  使用**商業智慧**工具來繪製出有用的深入解析，可以引導決策進行。 

在本教學課程中，您將了解如何：

> [!div class="checklist"]
> - 建立要將資料擷取至其中的租用戶分析存放區。
> - 使用彈性作業以從每個租用戶資料庫將資料擷取至分析存放區。
> - 最佳化擷取的資料 (重新組織為星狀結構描述)。
> - 查詢分析資料庫。
> - 針對資料視覺效果使用 Power BI，反白顯示租用戶資料的趨勢，並對改善提出建議。

![architectureOverView](media/saas-multitenantdb-tenant-analytics/architectureOverview.png)

## <a name="offline-tenant-analytics-pattern"></a>離線租用戶分析模式

您開發的 SaaS 應用程式具有儲存在雲端中大量租用戶資料的存取權。 資料提供應用程式之作業和使用方式以及租用戶行為之深入解析的豐富來源。 這些深入解析可以引導應用程式和平台中的功能開發、使用性改進及其他投資。

當所有資料都在一個多租用戶資料庫時，存取所有租用戶的資料就相當簡單。 但是當資料散佈在上千個資料庫時，存取就更加複雜。 有效駕馭複雜度的一種方法是將資料擷取至分析資料庫或資料倉儲。 然後，您要查詢資料倉儲，以從所有租用戶的票證資料蒐集深入解析。

此教學課程會呈現此範例 SaaS 應用程式的完整分析案例。 首先，會使用彈性作業來排程從每個租用戶資料庫擷取資料。 資料會傳送至分析存放區。 分析存放區可以是 SQL Database 或 SQL 資料倉儲。 針對大規模資料擷取，建議使用 [Azure Data Factory](../data-factory/introduction.md)。

接下來，彙總的資料會切割成一組[星狀結構描述](https://www.wikipedia.org/wiki/Star_schema)資料表。 資料表是由一個中央的事實資料表，再加上相關的維度資料表所組成：

- 星狀結構描述中的中央事實資料表包含票證資料。
- 維度資料表包含有關地點、事件、客戶和購買日期的資料。

中央和維度資料表一起可以促成有效分析處理。 本教學課程中使用的星狀結構描述會顯示在下列映像中：
 
![StarSchema](media/saas-multitenantdb-tenant-analytics/StarSchema.png)

最後，會查詢星狀結構描述資料表。 查詢結果會以視覺化方式顯示，以反白顯示租用戶行為的深入解析以及其對於應用程式的使用。 您可以使用這個星狀結構描述執行查詢，協助探索如下所示的項目：

- 誰購買票證、從哪個地點購買。
- 在下列區域隱藏模式和趨勢：
    - 票證的銷售。
    - 每個地點的相對熱門程度。

了解每個租用戶使用服務的一致性，會提供機會得以建立符合其需求的服務方案。 本教學課程提供可以從租用戶資料收集之深入解析的基本範例。

## <a name="setup"></a>設定

### <a name="prerequisites"></a>必要條件

若要完成本教學課程，請確定符合下列必要條件：

- 已部署 Wingtip Tickets SaaS 多租用戶資料庫應用程式。 若要在五分鐘內完成部署，請參閱[部署及探索 Wingtip Tickets SaaS 多租用戶資料庫應用程式](saas-multitenantdb-get-started-deploy.md)
- Wingtip SaaS 指令碼和應用程式[原始程式碼](https://github.com/Microsoft/WingtipTicketsSaaS-MultiTenantDB)是從 GitHub 下載。 請務必在擷取檔案內容之前解除封鎖 zip 檔案。 關於下載和解除封鎖 Wingtip Tickets SaaS 指令碼的步驟，請參閱[一般指引](saas-tenancy-wingtip-app-guidance-tips.md)。
- Power BI Desktop 已安裝。 [下載 Power BI Desktop](https://powerbi.microsoft.com/downloads/)
- 已佈建額外租用戶的批次，請參閱[**佈建租用戶教學課程**](saas-multitenantdb-provision-and-catalog.md)。
- 已建立作業代理程式和作業代理程式資料庫。 請參閱[**結構描述管理教學課程**](saas-multitenantdb-schema-management.md#create-a-job-agent-database-and-new-job-agent)中的適當步驟。

### <a name="create-data-for-the-demo"></a>建立資料以供示範

在本教學課程中，分析是在票證銷售資料上執行。 在目前的步驟中，您會為所有租用戶產生票證資料。  稍後會擷取此資料以進行分析。 請確定您已如先前所述佈建租用戶的批次，以便獲得有意義的資料數量。 足夠數量的資料可以公開不同票證購買模式的範圍。

1. 在 **PowerShell ISE** 中，開啟 …\Learning Modules\Operational Analytics\Tenant Analytics\Demo-TenantAnalytics.ps1，然後設定下列值：
    - **$DemoScenario** = **1** 購買各地事件的票證
2. 按 **F5** 以執行指令碼並建立各地點中各個事件的票證購買歷程記錄。  指令碼會執行數分鐘以產生數以萬計的票證。

### <a name="deploy-the-analytics-store"></a>部署分析存放區
通常會有數個交易式分區資料庫一起保留所有租用戶資料。 您必須從分區資料庫將租用戶資料彙總至一個分析存放區。 彙總可以有效查詢資料。 在本教學課程中，Azure SQL Database 資料庫是用來儲存彙總的資料。

在下列步驟中，您會部署分析存放區，稱為 **tenantanalytics**。 您也可以部署預先定義的資料表，稍後會在本教學課程中填入：
1. 在 PowerShell ISE 中，開啟 …\Learning Modules\Operational Analytics\Tenant Analytics\Demo-TenantAnalytics.ps1 
2. 在指令碼中設定 $DemoScenario 變數，以符合您對於分析存放區的選擇。 基於學習之目的，建議您使用沒有資料行存放區的 SQL 資料庫。
    - 若要使用沒有資料行存放區的 SQL 資料庫，請設定 **$DemoScenario** = **2**
    - 若要使用具有資料行存放區的 SQL 資料庫，請設定 **$DemoScenario** = **3**  
3. 按 **F5** 以執行可建立租用戶分析存放區的示範指令碼 (它會呼叫 Deploy-TenantAnalytics<XX>.ps1 指令碼)。 

您已經部署應用程式，並且使用感興趣的租用戶資料填入，請使用 [SQL Server Management Studio (SSMS)](https://docs.microsoft.com/sql/ssms/download-sql-server-management-studio-ssms) 來連線 **tenants1-mt-\<User\>** 和 **catalog-mt-\<User\>** 伺服器，使用登入 = developer，密碼 = P@ssword1。

![architectureOverView](media/saas-multitenantdb-tenant-analytics/ssmsSignIn.png)

在 [物件總管] 中，執行下列步驟：

1. 展開 tenants1-mt-\<User\> 伺服器。
2. 展開 [資料庫] 節點，並查看包含多個租用戶的 tenants1 資料庫。
3. 展開 catalog-mt-\<User\> 伺服器。
4. 請確認您看到分析存放區和 jobaccount 資料庫。

藉由展開分析存放區節點，查看 SSMS 物件總管中的下列資料庫項目：

- 資料表 **TicketsRawData** 和 **EventsRawData** 會保存來自租用戶資料庫的未經處理擷取資料。
- 星狀結構描述資料表是 **fact_Tickets**、**dim_Customers**、**dim_Venues**、**dim_Events** 和 **dim_Dates**。
- **sp_ShredRawExtractedData**預存程序是用來從未經處理資料資料表填入星狀結構描述資料表。

![tenantAnalytics](media/saas-multitenantdb-tenant-analytics/tenantAnalytics.png)

## <a name="data-extraction"></a>資料擷取 

### <a name="create-target-groups"></a>建立目標群組 

繼續之前，請確定您已部署作業帳戶與 jobaccount 資料庫。 在下一組步驟中，彈性作業是用來從分區的租用戶資料庫擷取資料，然後將資料儲存在分析存放區。 然後，第二個作業會切割資料，並將它儲存到星狀結構描述中的資料表。 這兩個作業會針對兩個不同的目標群組執行，也就是 **TenantGroup** 和 **AnalyticsGroup**。 擷取作業會針對 TenantGroup 執行，該群組包含所有租用戶資料庫。 切割作業會針對 AnalyticsGroup 執行，該群組只包含分析存放區。 使用下列步驟建立目標群組：

1. 在 SSMS 中，連線至 catalog-mt-\<User\> 中的 **jobaccount** 資料庫。
2. 在 SSMS 中，開啟 …\Learning Modules\Operational Analytics\Tenant Analytics\ TargetGroups.sql 
3. 在指令碼上方修改 @User 變數，將 <User> 取代為您部署 Wingtip Tickets SaaS 多租用戶資料庫應用程式時使用的使用者值。
4. 按下 **F5** 以執行指令碼，該指令碼會建立這兩個目標群組。

### <a name="extract-raw-data-from-all-tenants"></a>從所有租用戶擷取未經處理資料

票證和客戶資料可能會比事件和場地資料更常發生交易。 因此，相較於擷取事件和地點資料，請考慮個別且更頻繁地擷取票證和客戶資料。 在本節中，您會定義及排程兩個個別的作業：

- 擷取票證和客戶資料。
- 擷取事件和地點資料。

每個作業都會擷取其資料，並將它張貼至分析存放區。 個別作業會將擷取的資料分割為分析星狀結構描述。

1. 在 SSMS 中，連線至 catalog-mt-\<User\> 伺服器中的 **jobaccount** 資料庫。
2. 在 SSMS 中，開啟 ...\Learning Modules\Operational Analytics\Tenant Analytics\ExtractTickets.sql。
3. 在指令碼上方修改 @User，將 <User> 取代為您部署 Wingtip Tickets SaaS 多租用戶資料庫應用程式時使用的使用者名稱。 
4. 按下 **F5** 以執行指令碼，該指令碼會建立和執行作業，從每個租用戶資料庫擷取票證和客戶資料。 作業會將資料儲存至分析存放區。
5. 查詢 tenantanalytics 資料庫中的 TicketsRawData 資料表，以確定資料表已填入來自所有租用戶的票證資訊。

![ticketExtracts](media/saas-multitenantdb-tenant-analytics/ticketExtracts.png)

重複上述步驟，但是這次在步驟 2 中將 **\ExtractTickets.sql** 取代為 **\ExtractVenuesEvents.sql**。

已成功使用來自所有租用戶的新事件和地點資訊，執行作業填入分析存放區中的 EventsRawData 資料表。 

## <a name="data-reorganization"></a>資料重新組織

### <a name="shred-extracted-data-to-populate-star-schema-tables"></a>切割擷取的資料以填入星狀結構描述資料表

下一個步驟是將擷取的未經處理資料切割成一組針對分析查詢最佳化的資料表。 會使用星狀結構描述。 中央事實資料表會保存個別票證銷售記錄。 維度資料表會填入有關場地、事件、客戶和購買日期的資料。 

在本章節的教學課程中，您會定義和執行作業，合併擷取的未經處理資料與星狀結構描述資料表中的資料。 合併作業完成之後，未經處理資料會遭到刪除，讓資料表準備好由下一個租用戶資料擷取作業填入。

1. 在 SSMS 中，連線至 catalog-mt-\<User\> 中的 **jobaccount** 資料庫。
2. 在 SSMS 中，開啟 …\Learning Modules\Operational Analytics\Tenant Analytics\ShredRawExtractedData.sql。
3. 按下 **F5** 以執行指令碼，定義會呼叫分析存放區中 sp_ShredRawExtractedData 預存程序的作業。
4. 允許足夠時間讓作業成功執行。
    - 請檢查 jobs.jobs_execution 資料表的**生命週期**資料行，以取得作業的狀態。 請確認作業「已成功」，再繼續作業。 成功的執行會顯示類似下圖的資料：

![shreddingJob](media/saas-multitenantdb-tenant-analytics/shreddingJob.PNG)

## <a name="data-exploration"></a>資料探索

### <a name="visualize-tenant-data"></a>視覺化租用戶資料

星狀結構描述資料表中的資料會提供分析所需的所有票證銷售資料。 若要讓查看大型資料集中的趨勢更簡單，您需要以圖形方式將它視覺化。  在本節中，您會了解如何使用 **Power BI** 以操作及視覺化您已擷取並組織的租用戶資料。

使用下列步驟以連線至 Power BI，以及匯入您稍早建立的檢視：

1. 啟動 Power BI Desktop。
2. 從 [常用] 功能區選取 [取得資料]，然後選取 [更多…] 。
3. 在 [取得資料] 視窗中，選取 [Azure SQL Database]。
4. 在資料庫登入視窗中，輸入您的伺服器名稱 (catalog-mt-\<User\>.database.windows.net)。 針對 [資料連線模式] 選取 [匯入]，然後按一下 [確定]。 

    ![powerBISignIn](media/saas-multitenantdb-tenant-analytics/powerBISignIn.PNG)

5. 在左窗格中選取 [資料庫]，然後輸入使用者名稱 = developer，輸入密碼 = P@ssword1。 按一下 [ **連接**]。  

    ![DatabaseSignIn](media/saas-multitenantdb-tenant-analytics/databaseSignIn.PNG)

6. 在分析資料庫底下的 [導覽] 窗格中，選取星狀結構描述資料表：fact_Tickets dim_Events、dim_Venues、dim_Customers 和 dim_Dates。 然後選取 [載入]。 

恭喜！ 您已成功將資料載入 Power BI。 現在您可以開始探索有趣的視覺效果，協助您深入解析您的租用戶。 接下來您會逐步了解分析如何讓您將資料驅動的建議提供給 Wingtip 票證業務小組。 建議可協助最佳化商務模型和客戶體驗。

您會從分析票證銷售資料來查看跨地點之使用方式的變化來開始。 在 Power BI 中選取下列選項，以依據每個地點銷售的票證總數，繪製橫條圖。 由於票證產生器中的隨機變化，您的結果可能會不同。
 
![TotalTicketsByVenues](media/saas-multitenantdb-tenant-analytics/TotalTicketsByVenues.PNG)

上圖確認每個地點銷售的票證數目會有所不同。 銷售較多票證的地點會比銷售較少票證的地點更常使用您的服務。 有機會根據不同的租用戶需求，量身打造資源配置。

您可以進一步分析資料，以查看票證銷售如何隨著時間而變化。 在 Power BI 中選取下列選項，以繪製 60 天期間內每天的票證銷售總數。
 
![SaleVersusDate](media/saas-multitenantdb-tenant-analytics/SaleVersusDate.PNG)

上述圖表會顯示某些地點的該票證銷售尖峰。 這些尖峰會強化某些地點可能不成比例地耗用系統資源的概念。 目前為止何時出現尖峰並沒有任何明顯模式。

接下來您想要進一步調查這些尖峰銷售日期的重要性。 在票證開始銷售之後何時發生這些尖峰？ 若要繪製每日銷售的票證，請在 Power BI 中選取下列選項。

![SaleDayDistribution](media/saas-multitenantdb-tenant-analytics/SaleDistributionPerDay.PNG)

上圖顯示某些地點在銷售的第一天即銷售大量票證。 票證一開始在這些地點銷售時，似乎有瘋狂的搶購熱潮。 少數地點的活動爆發可能會影響其他租用戶的服務。

您可以再次向下鑽研資料，以查看這些地點主辦的所有事件是否都會造成這樣的搶購熱潮。 在先前的繪圖中，您觀察到 Contoso Concert Hall 銷售大量票證，而且 Contoso 在特定日子也有票證銷售尖峰。 試用 Power BI 選項來繪製 Contoso Concert Hall 的累計票證銷售，聚焦在每個事件的銷售趨勢。 所有事件是否遵循相同的銷售模式？

![ContosoSales](media/saas-multitenantdb-tenant-analytics/EventSaleTrends.PNG)

Contoso Concert Hall 的上述繪圖顯示並非所有事件都發生搶購熱潮。 試用篩選選項，以查看其他地點的銷售趨勢。

票證銷售模式的深入解析可能會引導 Wingtip Tickets 最佳化其商務模型。 Wingtip 或許應該引進不同計算大小的服務層，而非對所有租用戶平均收費。 可以為需要每日銷售更多票證的較大地點，提供具有較高服務等級協定 (SLA) 的較高層級。 這些地點可以將其資料庫放在具有較高每個資料庫資源限制的集區中。 每個服務層可以有每小時銷售配置，針對超過的配置收取額外費用。 具有定期銷售爆發的較大地點可以從較高的層級獲益，Wingtip Tickets 也可以更有效率地從其服務創造營收。

同時，有些 Wingtip Tickets 客戶抱怨，他們在銷售足夠票證以攤平服務成本方面有難處。 或許在這些深入解析中，有機會可以為表現不佳的地點促進票證銷售。 較高的銷售會增加服務的認知值。 以滑鼠右鍵按一下 fact_Tickets，然後選取 [新的量值]。 針對稱為 **AverageTicketsSold** 的新量值輸入下列運算式：

```
AverageTicketsSold = DIVIDE(DIVIDE(COUNTROWS(fact_Tickets),DISTINCT(dim_Venues[VenueCapacity]))*100, COUNTROWS(dim_Events))
```

選取下列視覺效果選項，來繪製每個地點銷售票證的百分比，決定其相對成功程度。

![analyticsViews](media/saas-multitenantdb-tenant-analytics/AvgTicketsByVenues.PNG)

上圖顯示即使大部分地點銷售票證的 80% 以上，但是有一些地點連填滿一半的座位都辦不到。 試用 Values Well 以選取每個地點銷售票證的最大或最小百分比。

先前您已深化分析來探索票證銷售傾向於遵循可預測的模式。 此探索可以讓 Wingtip Tickets 藉由建議動態定價來協助表現不佳的地點促進票證銷售。 此探索會揭露機會，運用機器學習技術來預測每個事件的票證銷售。 也可以針對提供票證銷售折扣的營收影響進行預測。 Power BI Embedded 可以整合至事件管理應用程式。 整合可以協助視覺化不同折扣的預測銷售和效果。 應用程式可以協助設計直接從分析顯示套用最佳折扣。

您已經從 Wingtip Tickets SaaS 多租用戶資料庫應用程式觀察到租用戶資料的趨勢。 您可以思考應用程式為 SaaS 應用程式廠商通知商務決策的其他方式。 廠商可以更佳地符合其租用戶的需求。 希望本教學課程已讓您具備必要的工具，可以執行租用戶資料的分析，讓您的企業做出資料驅動的決策。

## <a name="next-steps"></a>後續步驟

在本教學課程中，您已了解如何：

> [!div class="checklist"]
> - 使用預先定義的星狀結構描述資料表部署租用戶分析資料庫
> - 使用彈性作業以從所有租用戶資料庫擷取資料
> - 將擷取的資料合併到針對分析目的設計的星狀結構描述中的資料表
> - 查詢分析資料庫 
> - 使用適用於資料視覺效果的 Power BI 來觀察租用戶資料的趨勢 

恭喜！

## <a name="additional-resources"></a>其他資源

其他[以 Wingtip SaaS 應用程式為基礎的教學課程](saas-dbpertenant-wingtip-app-overview.md#sql-database-wingtip-saas-tutorials)。 
- [彈性作業](sql-database-elastic-jobs-overview.md)。
- [使用擷取的資料執行跨租用戶分析 - 單一租用戶應用程式](saas-tenancy-tenant-analytics.md) 