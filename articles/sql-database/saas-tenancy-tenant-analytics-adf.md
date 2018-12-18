---
title: 使用 Azure SQL 資料倉儲對租用戶資料庫執行分析查詢 | Microsoft Docs
description: 使用從 Azure SQL Database、SQL 資料倉儲、Azure Data Factory 或 Power BI 擷取的資料，進行跨租用戶分析查詢。
services: sql-database
ms.service: sql-database
ms.subservice: scenario
ms.custom: ''
ms.devlang: ''
ms.topic: conceptual
author: anumjs
ms.author: anjangsh
ms.reviewer: MightyPen
manager: craigg
ms.date: 09/19/2018
ms.openlocfilehash: 034fd2434d3b824c4356e640a1c1665dff542de6
ms.sourcegitcommit: 715813af8cde40407bd3332dd922a918de46a91a
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "47056586"
---
# <a name="explore-saas-analytics-with-azure-sql-database-sql-data-warehouse-data-factory-and-power-bi"></a>使用 Azure SQL Database、SQL 資料倉儲、Data Factory 及 Power BI 探索 SaaS 分析

在此教學課程中，您會逐步完成端對端分析案例。 此案例示範如何透過租用戶資料分析讓軟體廠商做出智慧決策。 使用從每個租用戶資料庫擷取的資料，您使用分析來取得租用戶行為的深入解析，包括他們對於範例 Wingtip Tickets SaaS 應用程式的使用。 這個案例牽涉到三個步驟： 

1.  從每個租用戶資料庫**擷取資料**至分析存放區，在此案例中為 SQL 資料倉儲。
2.  針對分析處理**最佳化擷取的資料**。
3.  使用**商業智慧**工具來繪製出有用的深入解析，可以引導決策進行。 

在本教學課程中，您將了解如何：

> [!div class="checklist"]
> - 建立載入用的租用戶分析存放區。
> - 使用 Azure Data Factory (ADF) 以從每個租用戶資料庫將資料擷取至分析資料倉儲。
> - 最佳化擷取的資料 (重新組織為星狀結構描述)。
> - 查詢分析資料倉儲。
> - 針對資料視覺效果使用 Power BI，反白顯示租用戶資料的趨勢，並對改善提出建議。

![architectureOverView](media/saas-tenancy-tenant-analytics/adf_overview.png)

## <a name="analytics-over-extracted-tenant-data"></a>透過擷取的租用戶資料執行分析

SaaS 應用程式會保留雲端中可能非常大量的租用戶資料。 此資料可提供應用程式之作業和使用方式，以及租用戶行為之深入解析的豐富來源。 這些深入解析可以引導應用程式和平台中的功能開發、使用性改進及其他投資。

當所有資料都在一個多租用戶資料庫時，存取所有租用戶的資料就相當簡單。 但是當資料散佈在上千個資料庫時，存取就更加複雜。 有效駕馭複雜度的一種方法是將資料擷取至分析資料庫或資料倉儲進行查詢。

此教學課程會呈現 Wingtip Tickets 應用程式的端對端分析案例。 首先，[Azure Data Factory (ADF)](../data-factory/introduction.md) 可作為協調流程工具，從每個租用戶資料庫擷取票證銷售和相關資料。 這項資料會載入分析存放區中的暫存表格。 分析存放區可以是 SQL Database 或 SQL 資料倉儲。 本教學課程使用 [SQL 資料倉儲](https://docs.microsoft.com/azure/sql-data-warehouse/sql-data-warehouse-overview-what-is)作為分析存放區。

接下來，擷取的資料會轉換成一組[星型結構描述](https://www.wikipedia.org/wiki/Star_schema)資料表並載入。 資料表是由一個中央的事實資料表，再加上相關的維度資料表所組成：

- 星狀結構描述中的中央事實資料表包含票證資料。
- 維度資料表包含有關地點、事件、客戶和購買日期的資料。

中央和維度資料表一起可以促成有效分析處理。 本教學課程中使用的星狀結構描述會顯示在下列映像中：
 
![architectureOverView](media/saas-tenancy-tenant-analytics/starschematables.JPG)

最後，會查詢星狀結構描述資料表。 使用 Power BI 的查詢結果會以視覺化方式顯示，以反白顯示租用戶行為的深入解析以及其對於應用程式的使用。 有了這個星型結構描述，您可以執行公開的查詢：

- 誰購買票證、從哪個地點購買。
- 票證銷售中的模式和趨勢。
- 每個地點的相對熱門程度。

本教學課程提供可以從 Wingtip Tickets 資料收集之深入解析的基本範例。 例如，了解每個地點使用服務的方式，可能會讓 Wingtip Tickets 廠商針對或多或少的使用中場地思考不同的服務方案。 

## <a name="setup"></a>設定

### <a name="prerequisites"></a>必要條件

> [!NOTE]
> 本教學課程會使用目前處於有限預覽 (連結的服務參數化) 的 Azure Data Factory 功能。 如果想要進行本教學課程，請[在此](https://forms.office.com/Pages/ResponsePage.aspx?id=v4j5cvGGr0GRqy180BHbRxrVywox1_tHk9wgd5P8SVJUNlFINjNEOElTVFdMUEREMjVVUlJCUDdIRyQlQCN0PWcu)提供訂用帳戶識別碼。 只要您的訂用帳戶一啟用，我們就會寄送確認函給您。

若要完成本教學課程，請確定符合下列必要條件：
- 已部署 Wingtip Tickets SaaS Database Per Tenant 應用程式。 若要在 5 分鐘內完成部署，請參閱[部署及探索 Wingtip SaaS 應用程式](saas-dbpertenant-get-started-deploy.md)。
- Wingtip Tickets SaaS Database Per Tenant 指令碼和應用程式[原始程式碼](https://github.com/Microsoft/WingtipTicketsSaaS-DbPerTenant/)是從 GitHub 下載。 請參閱下載指示。 請務必在擷取檔案內容之前解除封鎖 zip 檔案。
- Power BI Desktop 已安裝。 [下載 Power BI Desktop](https://powerbi.microsoft.com/downloads/)。
- 已佈建額外租用戶的批次，請參閱[**佈建租用戶教學課程**](saas-dbpertenant-provision-and-catalog.md)。

### <a name="create-data-for-the-demo"></a>建立資料以供示範

本教學課程探索票證銷售資料的分析。 在此步驟中，您會為所有租用戶產生票證資料。 在稍後的步驟中，會擷取此資料進行分析。 _請確定您已佈建租用戶的批次_ (如先前所述)，所以您有足夠的資料來公開各種不同票證購買模式。

1. 在 PowerShell ISE 中開啟 *…\Learning Modules\Operational Analytics\Tenant Analytics DW\Demo-TenantAnalyticsDW.ps1*，然後設定下列值：
    - **$DemoScenario** = **1** 購買各地事件的票證
2. 按 **F5** 以執行指令碼並且為所有場地建立票證購買歷程記錄。 有 20 個租用戶時，指令碼會產生數以萬計的票證，可能需要 10 分鐘或更長時間。

### <a name="deploy-sql-data-warehouse-data-factory-and-blob-storage"></a>部署 SQL 資料倉儲、Data Factory 和 Blob 儲存體 
在 Wingtip Tickets 應用程式中，租用戶的交易資料會透過多個資料庫發佈。 您可使用 Azure Data Factory (ADF) 將此資料的擷取、載入和轉換 (ELT) 作業協調至資料倉儲。 為了有效地將資料載入 SQL 資料倉儲，ADF 會將資料擷取至中繼 blob 檔案，然後使用 [PolyBase](https://docs.microsoft.com/azure/sql-data-warehouse/design-elt-data-loading) 將資料載入資料倉儲。   

在此步驟中，您會部署教學課程中使用的其他資源：名為 _tenantanalytics_ 的 SQL 資料倉儲、名為 _dbtodwload-\<user\>_ 的 Azure Data Factory，以及名為 _wingtipstaging\<user\>_ 的 Azure 儲存體帳戶。 儲存體帳戶可在擷取的資料檔案載入資料倉儲之前，以 blob 形式暫時保存這些檔案。 這個步驟也會部署資料倉儲結構描述，並定義協調 ELT 程序的 ADF 管道。
1. 在 PowerShell ISE 中開啟 *…\Learning Modules\Operational Analytics\Tenant Analytics DW\Demo-TenantAnalyticsDW.ps1*，然後設定：
    - **$DemoScenario** = **2** 部署租用戶分析資料倉儲、blob 儲存體及資料處理站 
1. 按 **F5** 執行示範指令碼並部署 Azure 資源。 

現在，請檢閱您所部署的 Azure 資源：
#### <a name="tenant-databases-and-analytics-store"></a>租用戶資料庫和分析存放區
使用 [SQL Server Management Studio (SSMS)](https://docs.microsoft.com/sql/ssms/download-sql-server-management-studio-ssms) 連線到 **tenants1-dpt-&lt;user&gt;** 和 **catalog-dpt-&lt;user&gt;** 伺服器。 將 &lt;user&gt; 取代為您部署應用程式時所使用的值。 使用者登入 = *developer*，密碼 = *P@ssword1*。 如需詳細指引，請參閱[簡介教學課程](saas-dbpertenant-wingtip-app-overview.md)。

![從 SSMS 連線到 SQL Database 伺服器](media/saas-tenancy-tenant-analytics/ssmsSignIn.JPG)

在物件總管中：

1. 展開 *tenants1-dpt-&lt;user&gt;* 伺服器。
1. 展開 [資料庫] 節點，並查看租用戶資料庫的清單。
1. 展開 *catalog-dpt-&lt;user&gt;* 伺服器。
1. 請確認您看到分析存放區包含下列物件：
    1. 資料表 **raw_Tickets**、**raw_Customers**、**raw_Events** 及 **raw_Venues** 保存來自租用戶資料庫的原始擷取資料。
    1. 星狀結構描述資料表是 **fact_Tickets**、**dim_Customers**、**dim_Venues**、**dim_Events** 和 **dim_Dates**。
    1. 預存程序 **sp_transformExtractedData** 可用來轉換資料，並將資料載入星型結構描述資料表。

![DWtables](media/saas-tenancy-tenant-analytics/DWtables.JPG)

#### <a name="blob-storage"></a>Blob 儲存體
1. 在 [Azure 入口網站](https://ms.portal.azure.com)中，巡覽至您用來部署應用程式的資源群組。 確認已新增名為 **wingtipstaging\<user\>** 的儲存體帳戶。

  ![DWtables](media/saas-tenancy-tenant-analytics/adf-staging-storage.PNG)

1. 按一下 [wingtipstaginguser]**\<\>** 儲存體帳戶，探索現有物件。
1. 按一下 [Blob] 圖格
1. 按一下容器 [configfile]
1. 確認 **configfile** 包含名為 **TableConfig.json** 的 JSON 檔案。 這個檔案包含來源和目的地資料表名稱、資料行名稱以及追蹤器資料行名稱。

#### <a name="azure-data-factory-adf"></a>Azure Data Factory (ADF)
在資源群組的 [Azure 入口網站](https://ms.portal.azure.com)中，確認已新增名為 _dbtodwload-\<user\>_ 的 Azure Data Factory。 

 ![adf_portal](media/saas-tenancy-tenant-analytics/adf-data-factory-portal.png)

本節探索建立的 Data Factory。 請依照下列步驟來啟動 Data Factory：
1. 在入口網站中，按一下名為 **dbtodwload-\<user\>** 的 Data Factory。
2. 按一下 [撰寫與監視] 圖格，以在個別索引標籤中啟動 Data Factory 設計工具。 

## <a name="extract-load-and-transform-data"></a>擷取、載入和轉換資料
Azure Data Factory 可用來協調擷取、載入及轉換資料。 在本教學課程中，您必須從每個租用戶資料庫擷取四個不同 SQL 檢視的資料：**rawTickets**、**rawCustomers**、**rawEvents** 及 **rawVenues**。 這些檢視包括地點識別碼，因此您可以區分資料倉儲中每個地點的資料。 將資料載入資料倉儲中對應的暫存表格：**raw_Tickets**、**raw_customers**、**raw_Events** 及 **raw_Venue**。 預存程序轉換未經處理資料並填入星型結構描述資料表：**fact_Tickets**、**dim_Customers**、**dim_Venues**、**dim_Events** 及 **dim_Dates**。

在上一節中，您已部署所需的 Azure 資源並將其初始化，包括資料處理站。 已部署的資料處理站包括管道、資料集、連結的服務等，需要擷取、載入及轉換租用戶資料。 讓我們進一步探索這些物件，然後觸發管道，將資料從租用戶資料庫移到資料倉儲。

### <a name="data-factory-pipeline-overview"></a>資料處理站管道概觀
本節探索在資料處理站建立的物件。 下圖描述本教學課程中使用的 ADF 管道整體工作流程。 如果您想要稍後探索管道並且先查看結果，請跳至下一節**觸發管道執行**。

![adf_overview](media/saas-tenancy-tenant-analytics/adf-data-factory.PNG)

在 [概觀] 頁面中，切換至左側面板的 [建立者] 索引標籤，並觀察有三個 [管線](https://docs.microsoft.com/azure/data-factory/concepts-pipelines-activities) 和三個已建立的 [資料集](https://docs.microsoft.com/azure/data-factory/concepts-datasets-linked-services)。
![adf_author](media/saas-tenancy-tenant-analytics/adf_author_tab.JPG)

三個巢狀管道為：SQLDBToDW、DBCopy 及 TableCopy。

**管道 1 - SQLDBToDW** 查詢儲存在目錄資料庫的租用戶資料庫名稱 (資料表名稱：[__ShardManagement].[ShardsGlobal])，並針對每個租用戶資料庫執行 **DBCopy** 管道。 完成時，會執行提供的 **sp_TransformExtractedData** 預存程序結構描述。 這個預存程序會轉換暫存表格中載入的資料，並填入星型結構描述資料表。

**管道 2 - DBCopy** 從儲存在 blob 儲存體的組態檔查詢來源資料表和資料行的名稱。  接著會針對四個資料表執行 **TableCopy** 管道：TicketFacts、CustomerFacts、EventFacts 及 VenueFacts。 針對所有 20 個資料庫同時執行 **[Foreach](https://docs.microsoft.com/azure/data-factory/control-flow-for-each-activity)** 活動。 ADF 允許最多同時執行 20 個迴圈反覆項目。 請考慮為多個資料庫建立多個管道。    

**管道 3 - TableCopy** 使用 SQL Database 中的資料列版本號碼 (_rowversion_) 來識別已變更或更新的資料列。 此活動查詢開始和結束資料列版本，用以擷取來自來源資料表的資料列。 儲存在每個租用戶資料庫中的 **CopyTracker** 資料表，會追蹤每次執行中從各來源資料表擷取的最後一個資料列。 全新或變更過的資料列會複製到資料倉儲中對應的暫存表格：**raw_Tickets**、**raw_Customers**、**raw_Venues** 及 **raw_Events**。 最後，最後一個資料列版本會儲存在 **CopyTracker** 資料表，做為下次擷取的初始資料列版本。 

此外還有三個參數化連結服務，將資料處理站連結至來源 SQL Database、目標 SQL 資料倉儲以及中繼 Blob 儲存體。 在 [建立者] 索引標籤上，按一下 [連線] 探索連結的服務，如下列影像所示：

![adf_linkedservices](media/saas-tenancy-tenant-analytics/linkedservices.JPG)

對應至三個連結的服務，有三個資料集參考您在管道活動中使用的資料做為輸入或輸出。 探索每個資料集，觀察連線和使用的參數。 _AzureBlob_ 指向包含來源、目標資料表和資料行，以及每個來源之追蹤程式資料行的組態檔。
  
### <a name="data-warehouse-pattern-overview"></a>資料倉儲模式概觀
SQL 資料倉儲可做為分析存放區使用，以便彙總租用戶資料。 在此範例中，PolyBase 會將資料載入 SQL 資料倉儲。 未經處理資料會載入具有識別欄位的暫存表格，以追蹤已轉換成星型結構描述資料表的資料列。 下列影像顯示載入模式：![loadingpattern](media/saas-tenancy-tenant-analytics/loadingpattern.JPG)

此範例使用緩時變維度 (SCD) 類型 1 的維度資料表。 每個維度都具有使用識別欄位定義的 Surrogate 索引鍵。 最佳做法是預先填入日期維度資料表，節省時間。 如需其他維度資料表，可使用 CREATE TABLE AS SELECT...(CTAS) 陳述式建立暫存資料表，其中包含現有修改和未修改的資料列，以及 Surrogate 索引鍵。 可使用 IDENTITY_INSERT=ON 進行。 然後將新資料列插入附有 IDENTITY_INSERT=OFF 的資料表。 若要簡單復原，可重新命名現有的維度資料表和暫存資料表，即可變成新的維度資料表。 每次執行之前，會刪除舊的維度資料表。

維度資料表會在事實資料表之前載入。 此排序可確保對於每個到達的事實，所有參考維度皆已存在。 載入事實時，每個對應維度的商務索引鍵會相符，而且對應的 Surrogate 索引鍵會新增至每個事實。

轉換的最後一個步驟會刪除暫存資料，以便準備好執行下一個管道。
   
### <a name="trigger-the-pipeline-run"></a>觸發管道執行
依照下列步驟，為所有租用戶資料庫執行完整的擷取、載入和轉換管道：
1. 在 ADF 使用者介面的 [建立者] 索引標籤中，選取左側窗格 **SQLDBToDW** 管道。
1. 按一下 [觸發] 並且從下拉式功能表按一下 [立即觸發]。 這個動作會立即執行管道。 在生產案例中，您會定義執行管道的時間表，以便重新整理排程中的資料。
  ![adf_trigger](media/saas-tenancy-tenant-analytics/adf_trigger.JPG)
1. 在 [管道執行] 頁面上，按一下 [完成]。
 
### <a name="monitor-the-pipeline-run"></a>監視管道執行
1. 在 ADF 使用者介面中，從左側功能表切換至 [監視器] 索引標籤。
1. 按一下 [重新整理]，直到 SQLDBToDW 管道的狀態顯示 [已成功]。
  ![adf_monitoring](media/saas-tenancy-tenant-analytics/adf_monitoring.JPG)
1. 使用 SSMS 連線到資料倉儲，並查詢星型結構描述資料表，以確認資料已載入這些資料表中。

當管道完成之後，事實資料表會保留所有場地的票證銷售資料，而維度資料表會填入對應的場地、事件及客戶。

## <a name="data-exploration"></a>資料探索

### <a name="visualize-tenant-data"></a>視覺化租用戶資料

星型結構描述中的資料會提供分析所需的所有票證銷售資料。 以圖形方式將資料視覺化，可以更方便查看大型資料集的趨勢。 在本節中，您使用 **Power BI** 操作資料倉儲中的租用戶資料並將其視覺化。

使用下列步驟以連線至 Power BI，以及匯入您稍早建立的檢視：

1. 啟動 Power BI Desktop。
2. 從 [常用] 功能區選取 [取得資料]，然後選取 [更多…] 。
3. 在 [取得資料] 視窗中，選取 [Azure SQL Database]。
4. 在資料庫登入視窗中，輸入您的伺服器名稱 (**catalog-dpt-&lt;User&gt;.database.windows.net**)。 針對 [資料連線模式] 選取 [匯入]，然後按一下 [確定]。 

    ![sign-in-to-power-bi](./media/saas-tenancy-tenant-analytics/powerBISignIn.PNG)

5. 在左窗格中選取 [資料庫]，然後輸入使用者名稱 = developer，輸入密碼 = P@ssword1。 按一下 [ **連接**]。  

    ![database-sign-in](./media/saas-tenancy-tenant-analytics/databaseSignIn.PNG)

6. 在分析資料庫底下的 [導覽] 窗格中，選取星型結構描述資料表：**fact_Tickets**、**dim_Events**、**dim_Venues**、**dim_Customers** 及 **dim_Dates**。 然後選取 [載入]。 

恭喜！ 您已成功將資料載入 Power BI。 現在可以探索有趣的視覺效果，讓您深入解析您的租用戶。 您會逐步了解分析如何將部分資料驅動的建議提供給 Wingtip Tickets 業務小組。 建議可協助最佳化商務模型和客戶體驗。

從分析票證銷售資料查看跨地點使用方式的變化來開始。 選取 Power BI 中顯示的選項，以依據每個地點銷售的票證總數，繪製橫條圖。 (由於票證產生器中的隨機變化，您的結果可能會不同。)
 
![TotalTicketsByVenues](./media/saas-tenancy-tenant-analytics/TotalTicketsByVenues-DW.PNG)

上圖確認每個地點銷售的票證數目會有所不同。 銷售較多票證的地點會比銷售較少票證的地點更常使用您的服務。 有機會根據不同的租用戶需求，量身打造資源配置。

您可以進一步分析資料，以查看票證銷售如何隨著時間而變化。 在 Power BI 中選取下列影像所顯示的選項，以繪製 60 天期間內每天的票證銷售總數。
 
![SaleVersusDate](./media/saas-tenancy-tenant-analytics/SaleVersusDate-DW.PNG)

上述圖表會顯示某些地點的該票證銷售尖峰。 這些尖峰會強化某些地點可能不成比例地耗用系統資源的概念。 目前為止何時出現尖峰並沒有任何明顯模式。

接下來調查這些尖峰銷售日期的顯著性。 在票證開始銷售之後何時發生這些尖峰？ 若要繪製每日銷售的票證，請在 Power BI 中選取下列影像所顯示的選項。

![SaleDayDistribution](./media/saas-tenancy-tenant-analytics/SaleDistributionPerDay-DW.PNG)

此圖顯示某些地點在銷售的第一天即銷售大量票證。 票證一開始在這些地點銷售時，似乎有瘋狂的搶購熱潮。 少數地點的活動爆發可能會影響其他租用戶的服務。

您可以再次向下鑽研資料，以查看這些地點主辦的所有事件是否都會造成這樣的搶購熱潮。 在先前的繪圖中，您看到 Contoso Concert Hall 銷售許多票證，而且 Contoso 在特定日子也有票證銷售尖峰。 試用 Power BI 選項來繪製 Contoso Concert Hall 的累計票證銷售，聚焦在每個事件的銷售趨勢。 所有事件是否遵循相同的銷售模式？ 嘗試產生與下圖類似的繪圖。

![ContosoSales](media/saas-tenancy-tenant-analytics/EventSaleTrends.PNG)

Contoso Concert Hall 經過一段時間為每個活動累積的票證銷售繪圖顯示，並非所有事件都發生搶購熱潮。 試用篩選選項，以探索其他地點的銷售趨勢。

票證銷售模式的深入解析可能會引導 Wingtip Tickets 最佳化其商務模型。 Wingtip 或許應該引進不同計算大小的服務層，而非對所有租用戶平均收費。 可以為需要每日銷售更多票證的較大地點，提供具有較高服務等級協定 (SLA) 的較高層級。 這些地點可以將其資料庫放在具有較高每個資料庫資源限制的集區中。 每個服務層可以有每小時銷售配置，針對超過的配置收取額外費用。 具有定期銷售爆發的較大地點可以從較高的層級獲益，Wingtip Tickets 也可以更有效率地從其服務創造營收。

同時，有些 Wingtip Tickets 客戶抱怨，他們在銷售足夠票證以攤平服務成本方面有難處。 或許在這些深入解析中，有機會可以為表現不佳的地點促進票證銷售。 較高的銷售會增加服務的認知值。 以滑鼠右鍵按一下 fact_Tickets，然後選取 [新的量值]。 針對稱為 **AverageTicketsSold** 的新量值輸入下列運算式：

```
AverageTicketsSold = DIVIDE(DIVIDE(COUNTROWS(fact_Tickets),DISTINCT(dim_Venues[VenueCapacity]))*100, COUNTROWS(dim_Events))
```

選取下列視覺效果選項，來繪製每個地點銷售票證的百分比，決定其相對成功程度。

![AvgTicketsByVenues](media/saas-tenancy-tenant-analytics/AvgTicketsByVenues-DW.PNG)

上圖顯示即使大部分地點的票證銷售數量皆超過 80%，但某些地點連填滿一半的座位都辦不到。 試用 Values Well 以選取每個地點銷售票證的最大或最小百分比。

## <a name="embedding-analytics-in-your-apps"></a>在您的應用程式中內嵌分析 
本教學課程著重在跨租用戶分析，用以改善軟體供應商對其租用戶的了解。 分析也可以為_租用戶_提供深入見解，協助他們更有效地管理自己的業務。 

在 Wingtip Tickets 範例中，您先前探索到票證銷售傾向遵循可預測的模式。 此見解可用來協助表現不佳的地點提升票證收入。 或許有機會運用機器學習技術來預測事件的票證銷售。 此外亦可能得以模擬價格變更的效果，以便預測提供折扣的影響。 Power BI Embedded 可以整合至事件管理應用程式，以視覺化方式檢視預測的銷售，包括售出的座位總數折扣和低銷售活動營收之影響。 有了 Power BI Embedded，您甚至可以透過視覺化體驗，針對票價實際套用折扣。


## <a name="next-steps"></a>後續步驟

在本教學課程中，您已了解如何：

> [!div class="checklist"]
> * 針對租用戶分析部署使用星型結構描述填入的 SQL 資料倉儲。
> * 使用 Azure Data Factory 從每個租用戶資料庫將資料擷取至分析資料倉儲。
> * 最佳化擷取的資料 (重新組織為星狀結構描述)。
> * 查詢分析資料倉儲。 
> * 使用 Power BI 以視覺化方式呈現所有租用戶的資料趨勢。

恭喜！

## <a name="additional-resources"></a>其他資源

- 其他[以 Wingtip SaaS 應用程式為基礎的教學課程](saas-dbpertenant-wingtip-app-overview.md#sql-database-wingtip-saas-tutorials)。
