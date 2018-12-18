---
title: 開始使用 Azure SQL Database 中的時態表 | Microsoft Docs
description: 了解如何開始使用 Azure SQL Database 中的時態表。
services: sql-database
ms.service: sql-database
ms.subservice: development
ms.custom: ''
ms.devlang: ''
ms.topic: conceptual
author: bonova
ms.author: bonova
ms.reviewer: carlrab
manager: craigg
ms.date: 03/21/2018
ms.openlocfilehash: d18630f9b4cea28bd19b2ac24e7b8c3d1822e17c
ms.sourcegitcommit: 51a1476c85ca518a6d8b4cc35aed7a76b33e130f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/25/2018
ms.locfileid: "47166413"
---
# <a name="getting-started-with-temporal-tables-in-azure-sql-database"></a>開始使用 Azure SQL Database 中的時態表
時態表是 Azure SQL Database 的一個新的可程式性功能，可讓您追蹤和分析資料變更的完整歷程記錄，而不需要撰寫自訂程式碼。 時態表會保持資料與時間內容之間的密切關係，因此只有在特定期間內，才會將預存的事實解譯為有效。 時態表的這個屬性允許進行以有效時間為基礎的分析，並可從資料演進中取得獨到見解。

## <a name="temporal-scenario"></a>時態表案例
本文說明在應用程式案例中使用時態表的步驟。 假設您想要從頭開始追蹤正在開發的新網站上的使用者活動，或您想要使用使用者活動分析擴充的現有網站上的使用者活動。 在這個簡化的範例中，我們假設在一段時間內瀏覽過的網頁數目是必須在裝載於 Azure SQL Database 的網站資料庫中擷取和監視的指標。 使用者活動歷史分析的目標是要獲得重新設計網站的意見，並為訪客提供更好的經驗。

此案例的資料庫模型非常簡單：使用者活動度量是以單一整數欄位 **PageVisited** 表示，而且會與使用者設定檔上的基本資訊一起被擷取。 此外，對於以時間為基礎的分析，您要為每個使用者保留一連串的資料列，其中每個資料列都代表一段特定時間內特定使用者瀏覽過的頁數。

![結構描述](./media/sql-database-temporal-tables/AzureTemporal1.png)

幸運的是，您不需要在您的 app 上花太多精力，就可以維護此活動資訊。 您可以使用時態表，將此程序自動化：讓您在網站設計期間有完整的彈性以及更多的時間，得以將重點放在資料分析本身。 您只需要確保將 **WebSiteInfo** 資料表設定為 [時態系統設定版本](https://msdn.microsoft.com/library/dn935015.aspx#Anchor_0)。 在此案例中使用時態表的確切步驟如下所述。

## <a name="step-1-configure-tables-as-temporal"></a>步驟 1︰將資料表設定為時態表
根據您要開始新的開發工作，還是升級現有的應用程式，您將會建立時態表，或透過新增時態屬性來修改現有的資料表。 在一般情況下，您的案例可能會混用這兩個選項。 使用 [SQL Server Management Studio](https://msdn.microsoft.com/library/mt238290.aspx) (SSMS)、[SQL Server Data Tools](https://msdn.microsoft.com/library/mt204009.aspx) (SSDT) 或其他任何 Transact-SQL 開發工具執行下列動作。

> [!IMPORTANT]
> 建議您一律使用最新版本的 Management Studio 保持與 Microsoft Azure 及 SQL Database 更新同步。 [更新 SQL Server Management Studio](https://msdn.microsoft.com/library/mt238290.aspx)。
> 
> 

### <a name="create-new-table"></a>建立新資料表
在 SSMS 的 [物件總管] 中使用內容功能表項目 [新系統設定版本的資料表] 開啟查詢編輯器與時態表範本指令碼，然後使用 [指定範本參數的值] \(Ctrl + Shift + M) 填入範本：

![SSMSNewTable](./media/sql-database-temporal-tables/AzureTemporal2.png)

在 SSDT 中，將新項目新增至資料庫專案時，請選擇 [時態表 (系統建立版本)] 範本。 這將會開啟資料表設計工具，並讓您輕鬆地指定資料表配置︰

![SSDTNewTable](./media/sql-database-temporal-tables/AzureTemporal3.png)

您也可以透過直接指定 Transact-SQL 陳述式來建立時態表，如下列範例所示。 請注意，每個時態表的必要元素為 PERIOD 定義以及可參照將儲存歷史資料列版本的另一個使用者資料表的 SYSTEM_VERSIONING 子句︰

````
CREATE TABLE WebsiteUserInfo 
(  
    [UserID] int NOT NULL PRIMARY KEY CLUSTERED 
  , [UserName] nvarchar(100) NOT NULL
  , [PagesVisited] int NOT NULL 
  , [ValidFrom] datetime2 (0) GENERATED ALWAYS AS ROW START
  , [ValidTo] datetime2 (0) GENERATED ALWAYS AS ROW END
  , PERIOD FOR SYSTEM_TIME (ValidFrom, ValidTo)
 )  
 WITH (SYSTEM_VERSIONING = ON (HISTORY_TABLE = dbo.WebsiteUserInfoHistory));
````

當您建立系統設定版本的時態表時，會自動建立隨附預設組態的歷程記錄資料表。 預設的歷程記錄資料表包含期間資料行 (結束、開始) 上啟用頁面壓縮的叢集 B 型樹狀目錄索引。 此組態適合使用時態表的大部分案例，特別是用於 [資料稽核](https://msdn.microsoft.com/library/mt631669.aspx#Anchor_0)。 

在此特殊案例中，我們的目標是針對一段較長的資料歷程記錄以及較大的資料集，執行以時間為基礎的趨勢分析，因此歷程記錄表格的儲存體選擇為叢集資料行存放區索引。 叢集資料行存放區為分析查詢提供很好的壓縮和效能。 時態表提供您完全獨立地設定目前資料表和時態表的索引的彈性。 

> [!NOTE]
> 資料行存放區索引可用於進階層和標準層 S3 和更新版本。
>

下列指令碼示範如何將歷程記錄資料表的預設索引變更為叢集資料行存放區︰

````
CREATE CLUSTERED COLUMNSTORE INDEX IX_WebsiteUserInfoHistory
ON dbo.WebsiteUserInfoHistory
WITH (DROP_EXISTING = ON); 
````

時態表在 [物件總管] 中會以特定圖示表示，讓您更容易識別，而其歷程記錄資料表則會以子節點顯示。

![AlterTable](./media/sql-database-temporal-tables/AzureTemporal4.png)

### <a name="alter-existing-table-to-temporal"></a>將現有的資料表變更為時態表
讓我們來看看替代案例，其中 WebsiteUserInfo 資料表已存在，但不是針對保留變更的歷程記錄而設計。 在此情況下，您只能擴充現有的資料表，使其成為時態表，如以下範例所示︰

````
ALTER TABLE WebsiteUserInfo 
ADD 
    ValidFrom datetime2 (0) GENERATED ALWAYS AS ROW START HIDDEN  
        constraint DF_ValidFrom DEFAULT DATEADD(SECOND, -1, SYSUTCDATETIME())
    , ValidTo datetime2 (0)  GENERATED ALWAYS AS ROW END HIDDEN   
        constraint DF_ValidTo DEFAULT '9999.12.31 23:59:59.99'
    , PERIOD FOR SYSTEM_TIME (ValidFrom, ValidTo); 

ALTER TABLE WebsiteUserInfo  
SET (SYSTEM_VERSIONING = ON (HISTORY_TABLE = dbo.WebsiteUserInfoHistory));
GO

CREATE CLUSTERED COLUMNSTORE INDEX IX_WebsiteUserInfoHistory
ON dbo.WebsiteUserInfoHistory
WITH (DROP_EXISTING = ON); 
````

## <a name="step-2-run-your-workload-regularly"></a>步驟 2：定期執行您的工作負載
時態表的主要優點是您不需要以任何方式變更或調整您的網站，就可以執行變更追蹤。 一旦建立時態表之後，當您每次對資料進行修改時，便會自動保存先前的資料列版本。 

若要為這個特定案例使用自動變更追蹤功能，我們只需要在每次使用者結束網站上的工作階段時，更新資料行 **PagesVisited** 即可︰

````
UPDATE WebsiteUserInfo  SET [PagesVisited] = 5 
WHERE [UserID] = 1;
````

請務必注意，更新查詢不需要知道實際作業進行的時間，也不需要知道如何保留歷史資料以供未來分析之用。 Azure SQL Database 會自動處理這兩方面。 下圖說明如何在每次更新時產生歷程記錄資料。

![TemporalArchitecture](./media/sql-database-temporal-tables/AzureTemporal5.png)

## <a name="step-3-perform-historical-data-analysis"></a>步驟 3︰執行歷史資料分析
現在當時態系統設定版本功能啟用時，您只需要一個查詢，就可以進行歷史資料分析。 在本文中，我們將提供一些解決常見分析案例的範例。若要了解所有詳細資料，請瀏覽使用 [FOR SYSTEM_TIME](https://msdn.microsoft.com/library/dn935015.aspx#Anchor_3) 子句所導入的各種選項。

若要查看依瀏覽網頁次數排序的前 10 名使用者，請執行以下查詢︰

````
DECLARE @hourAgo datetime2 = DATEADD(HOUR, -1, SYSUTCDATETIME());
SELECT TOP 10 * FROM dbo.WebsiteUserInfo FOR SYSTEM_TIME AS OF @hourAgo
ORDER BY PagesVisited DESC
````

您可以輕鬆地修改此查詢，以分析截至一天前、一個月前，或您希望的任何過去時間點的網站瀏覽記錄。

若要進行前一天的基本統計分析，請使用以下範例︰

````
DECLARE @twoDaysAgo datetime2 = DATEADD(DAY, -2, SYSUTCDATETIME());
DECLARE @aDayAgo datetime2 = DATEADD(DAY, -1, SYSUTCDATETIME());

SELECT UserID, SUM (PagesVisited) as TotalVisitedPages, AVG (PagesVisited) as AverageVisitedPages,
MAX (PagesVisited) AS MaxVisitedPages, MIN (PagesVisited) AS MinVisitedPages,
STDEV (PagesVisited) as StDevViistedPages
FROM dbo.WebsiteUserInfo 
FOR SYSTEM_TIME BETWEEN @twoDaysAgo AND @aDayAgo
GROUP BY UserId
````

若要搜尋特定使用者在某段時間的活動，請使用 CONTAINED IN 子句︰

````
DECLARE @hourAgo datetime2 = DATEADD(HOUR, -1, SYSUTCDATETIME());
DECLARE @twoHoursAgo datetime2 = DATEADD(HOUR, -2, SYSUTCDATETIME());
SELECT * FROM dbo.WebsiteUserInfo 
FOR SYSTEM_TIME CONTAINED IN (@twoHoursAgo, @hourAgo)
WHERE [UserID] = 1;
````

圖形視覺效果對於時態查詢特別方便，因為您可以以直覺方式，非常輕鬆地顯示趨勢和使用模式︰

![TemporalGraph](./media/sql-database-temporal-tables/AzureTemporal6.png)

## <a name="evolving-table-schema"></a>不斷演進的資料表結構描述
一般而言，您必須在開發 app 的同時，變更時態表結構描述。 因此，只要執行一般 ALTER TABLE 陳述式，Azure SQL Database 就會適當地傳播歷程記錄資料表的變更。 下列指令碼示範如何新增要追蹤的其他屬性︰

````
/*Add new column for tracking source IP address*/
ALTER TABLE dbo.WebsiteUserInfo 
ADD  [IPAddress] varchar(128) NOT NULL CONSTRAINT DF_Address DEFAULT 'N/A';
````

同樣地，您可以在您的工作負載正在作用中時，變更資料行定義︰

````
/*Increase the length of name column*/
ALTER TABLE dbo.WebsiteUserInfo 
    ALTER COLUMN  UserName nvarchar(256) NOT NULL;
````

最後，您可以移除您不再需要的資料行。

````
/*Drop unnecessary column */
ALTER TABLE dbo.WebsiteUserInfo 
    DROP COLUMN TemporaryColumn; 
````

或者，當您連線到資料庫 (線上模式)，或您屬於資料庫專案的一部分 (離線模式) 時，使用最新的 [SSDT](https://msdn.microsoft.com/library/mt204009.aspx) 變更時態表結構描述。

## <a name="controlling-retention-of-historical-data"></a>控制歷史資料的保留期
透過系統設定版本的時態表，歷程記錄資料表可以將資料庫大小增加到超過一般資料表。 一個大型且不斷成長的歷程記錄表格可能會因為單純的儲存體成本，以及對時態查詢效能所徵收的稅額而變成一個問題。 因此，開發資料保留原則來管理歷程記錄資料表中的資料是規劃及管理每個時態表生命週期的重要環節。 使用 Azure SQL Database 時，您有下列方法可以管理時態表中的歷史資料︰

* [資料表分割](https://msdn.microsoft.com/library/mt637341.aspx#Anchor_2)
* [自訂清除指令碼](https://msdn.microsoft.com/library/mt637341.aspx#Anchor_3)

## <a name="next-steps"></a>後續步驟
如需有關時態表的詳細資訊，請參閱 [MSDN 文件](https://msdn.microsoft.com/library/dn935015.aspx)。
瀏覽 Channel 9，聽聽[真實客戶的時態表實作成功案例](https://channel9.msdn.com/Blogs/jsturtevant/Azure-SQL-Temporal-Tables-with-RockStep-Solutions)，並觀看[時態表的即時示範](https://channel9.msdn.com/Shows/Data-Exposed/Temporal-in-SQL-Server-2016)。

