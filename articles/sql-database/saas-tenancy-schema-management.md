---
title: 在單一租用戶應用程式中管理 Azure SQL Database 結構描述 | Microsoft Docs
description: 在使用 Azure SQL Database 的單一租用戶應用程式中管理多個租用戶的結構描述
services: sql-database
ms.service: sql-database
ms.subservice: scenario
ms.custom: ''
ms.devlang: ''
ms.topic: conceptual
author: stevestein
ms.author: sstein
ms.reviewer: billgib
manager: craigg
ms.date: 09/19/2018
ms.openlocfilehash: 2f747eb09fd13647c4b6764ce3cc4fe72c00bcf0
ms.sourcegitcommit: 715813af8cde40407bd3332dd922a918de46a91a
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "47054841"
---
# <a name="manage-schema-in-a-saas-application-using-the-database-per-tenant-pattern-with-azure-sql-database"></a>使用每一租用戶一個資料庫的模式，透過 Azure SQL Database 管理 SaaS 應用程式中的結構描述
 
隨著資料庫應用程式的進化，勢必要對資料庫結構描述或參考資料進行變更。  並且需要定期執行資料庫維護工作。 若要對採用每一租用戶一個資料庫模式的應用程式進行管理，您需要將這些變更和維護工作套用至多個租用戶資料庫。

本教學課程會探討兩種案例：為所有租用戶部署參考資料更新，以及針對包含參考資料的資料表重建索引。 在所有租用戶資料庫及建立新租用戶資料庫所用的範本資料庫上，[彈性作業](sql-database-elastic-jobs-overview.md)功能會用來執行這些動作。

在本教學課程中，您將了解如何：

> [!div class="checklist"]

> * 建立作業代理程式
> * 使 T-SQL 作業在所有租用戶資料庫上執行
> * 更新所有租用戶資料庫中的參考資料
> * 針對所有租用戶資料庫中的資料表建立索引


若要完成本教學課程，請確定符合下列必要條件：

* 已部署 Wingtip Tickets SaaS Database Per Tenant 應用程式。 若要在五分鐘內完成部署，請參閱[部署及探索每一租用戶一個資料庫的 Wingtip Tickets SaaS 應用程式](saas-dbpertenant-get-started-deploy.md)
* 已安裝 Azure PowerShell。 如需詳細資料，請參閱[開始使用 Azure PowerShell](https://docs.microsoft.com/powershell/azure/get-started-azureps)
* 已安裝最新版的 SQL Server Management Studio (SSMS)。 [下載並安裝 SSMS](https://docs.microsoft.com/sql/ssms/download-sql-server-management-studio-ssms)

> [!NOTE]
> 本教學課程使用的 SQL Database 服務功能處於有限預覽版狀態 (彈性資料庫工作)。 如果想要進行本教學課程，請將您的訂用帳戶識別碼提供給 SaaSFeedback@microsoft.com，並使用主旨 Elastic Jobs Preview (彈性作業預覽)。 在您收到訂用帳戶已啟用的確認之後，請[下載並安裝最新的發行前版本作業 Cmdlet (英文)](https://github.com/jaredmoo/azure-powershell/releases)。 這是有限預覽版，如果有相關問題或需要支援，請連絡 SaaSFeedback@microsoft.com。

## <a name="introduction-to-saas-schema-management-patterns"></a>SaaS 結構描述管理模式的簡介

每一租用戶一個資料庫的模式可有效地隔離租用戶資料，但是會增加要管理及維護的資料庫數目。 [彈性作業](sql-database-elastic-jobs-overview.md)有助於 SQL 資料庫的管理。 這些作業可讓您安全且可靠地對一組資料庫執行工作 (Transact-SQL 指令碼)。 作業可在應用程式中跨所有租用戶資料庫部署結構描述和一般參考資料變更。 彈性作業也可以用來維護建立新租用戶所使用的「範本」資料庫，確保它隨時具有最新的結構描述和參考資料。

![畫面](media/saas-tenancy-schema-management/schema-management-dpt.png)


## <a name="elastic-jobs-limited-preview"></a>彈性作業有限預覽版

新版本的「彈性作業」現在是 Azure SQL Database 的整合功能。 這個新的「彈性作業」版本目前處於有限預覽版狀態。 有限預覽版目前支援使用 PowerShell 建立作業代理程式，以及使用 T-SQL 建立及管理作業。

> [!NOTE]
> 本教學課程使用的 SQL Database 服務功能處於有限預覽版狀態 (彈性資料庫工作)。 如果想要進行本教學課程，請將您的訂用帳戶識別碼提供給 SaaSFeedback@microsoft.com，並使用主旨 Elastic Jobs Preview (彈性作業預覽)。 在您收到訂用帳戶已啟用的確認之後，請[下載並安裝最新的發行前版本作業 Cmdlet (英文)](https://github.com/jaredmoo/azure-powershell/releases)。 這是有限預覽版，如果有相關問題或需要支援，請連絡 SaaSFeedback@microsoft.com。

## <a name="get-the-wingtip-tickets-saas-database-per-tenant-application-scripts"></a>取得每一租用戶一個資料庫的 Wingtip Tickets SaaS 應用程式指令碼

在 [WingtipTicketsSaaS-DbPerTenant](https://github.com/Microsoft/WingtipTicketsSaaS-DbPerTenant) GitHub 存放庫可取得應用程式原始程式碼和管理指令碼。 關於下載和解除封鎖 Wingtip Tickets SaaS 指令碼的步驟，請參閱[一般指引](saas-tenancy-wingtip-app-guidance-tips.md)。

## <a name="create-a-job-agent-database-and-new-job-agent"></a>建立作業代理程式資料庫和新的作業代理程式

本教學課程要求您必須使用 PowerShell 建立作業代理程式和其支援作業代理程式資料庫。 作業代理程式資料庫會保存作業定義、作業狀態和記錄。 一旦建立作業代理程式和其資料庫之後，您就可以立即建立及監視作業。

1. 在 [PowerShell ISE] 中開啟 …\\Learning Modules\\Schema Management\\*Demo-SchemaManagement.ps1*。
1. 按 **F5** 以執行指令碼。

Demo-SchemaManagement.ps1 指令碼會呼叫 Deploy-SchemaManagement.ps1 指令碼，以在目錄伺服器上建立名為 osagent 的 SQL 資料庫。 接著，該指令碼會建立作業代理程式，以參數的形式使用資料庫。

## <a name="create-a-job-to-deploy-new-reference-data-to-all-tenants"></a>建立作業以將新的參考資料部署到所有租用戶

在 Wingtip Tickets 應用程式中，每個租用戶資料庫都包含一組支援的場地類型。 每個場地都屬於特定場地類型，其中定義了可舉辦的事件種類，並決定應用程式中使用的背景影像。 若要讓應用程式支援新的事件類型，則必須更新此參考資料及新增場地類型。  在這個練習中，您要將更新部署到所有租用戶資料庫，以新增兩個額外的場地類型：*Motorcycle Racing* (機車賽) 和 *Swimming Club* (游泳俱樂部)。

首先，檢閱每個租用戶資料庫中的場地類型。 連線至 SQL Server Management Studio (SSMS) 中的其中一個租用戶資料庫，並檢查 VenueTypes 資料表。  您也可以在 Azure 入口網站的查詢編輯器中查詢此資料表 (從資料庫頁面進行存取)。 

1. 開啟 SSMS 並連線到租用戶伺服器：tenants1-dpt-&lt;user&gt;.database.windows.net
1. 若要確認目前「尚未」包含 Motorcycle Racing (機車賽) 和 Swimming Club (游泳俱樂部)，請瀏覽至 tenants1-dpt-&lt;user&gt; 伺服器上的 contosoconcerthall 資料庫，並查詢 VenueTypes 資料表。

現在讓我們建立作業以更新所有租用戶資料庫中的 *VenueTypes* 資料表，以新增場地類型。

為了建立新的作業，您會使用作業代理程式建立時，在 jobagent 資料庫中建立的一組作業系統預存程序。

1. 在 SSMS 中，連線到目錄伺服器：catalog-dpt-&lt;user&gt;.database.windows.net 伺服器 
1. 在 SSMS 中，開啟檔案 …\\Learning Modules\\Schema Management\\DeployReferenceData.sql
1. 修改陳述式：SET @wtpUser = &lt;user&gt; ，並使用您在部署 Wingtip Tickets SaaS 每一租用戶一個資料庫應用程式時使用的 User 值來取代
1. 確定您已連線到 jobagent 資料庫，然後按 **F5** 以執行指令碼

請觀察 *DeployReferenceData.sql* 指令碼中的下列元素：
* **sp\_add\_target\_group** 會建立目標群組名稱 DemoServerGroup。
* **sp\_add\_target\_group\_member** 會用來定義一組目標資料庫。  首先會新增 tenants1-dpt-&lt;User&gt; 伺服器。  將伺服器新增為目標，會使該伺服器中的資料庫在作業執行時納入作業中。 接著，basetenantdb 資料庫和 adhocreporting 資料庫 (將在之後的教學課程中使用) 會新增為目標。
* **sp\_add\_job** 會建立稱為「參考資料部署」的作業。
* **sp\_add\_jobstep** 會建立作業步驟，包含更新參考 VenueTypes 資料表的 T-SQL 命令文字。
* 指令碼中的其餘檢視會顯示物件是否存在，以及監視作業執行。 使用這些查詢來檢閱 **lifecycle** 資料行中的狀態值，以判斷作業在所有目標資料庫上完成的時間。

當指令碼完成之後，您可以驗證參考資料是否已更新。  在 SSMS 中，瀏覽至 tenants1-dpt-&lt;user&gt; 伺服器上的 contosoconcerthall 資料庫，並查詢 VenueTypes 資料表。  確認現在**已存在** Motorcycle Racing (機車賽) 和 Swimming Club (游泳俱樂部)。


## <a name="create-a-job-to-manage-the-reference-table-index"></a>建立作業以管理參考資料表索引

此練習會在參考資料表主索引鍵上使用作業重建索引。  這是典型的資料庫維護作業，可能會在載入大量資料後執行。

使用相同的作業「系統」預存程序建立作業。

1. 開啟 SSMS 並連線到 catalog-dpt-&lt;user&gt;.database.windows.net 伺服器
1. 開啟檔案 _…\\Learning Modules\\Schema Management\\OnlineReindex.sql_
1. 按一下滑鼠右鍵並選取 [連線]，然後連線到 catalog-dpt-&lt;user&gt;.database.windows.net 伺服器 (如果您尚未連線)
1. 確定您已連線到 jobagent 資料庫，然後按 **F5** 以執行指令碼

請觀察 OnlineReindex.sql 指令碼中的下列元素：
* **sp\_add\_job** 會建立稱為「線上 PK 索引重建\_\_VenueTyp\_\_265E44FD7FD4C885」的新作業
* **sp\_add\_jobstep** 會建立作業步驟，其中包含要更新索引的 T-SQL 命令文字
* 指令碼中的其餘檢視會監視作業執行。 使用這些查詢來檢閱 **lifecycle** 資料行中的狀態值，以判斷作業在所有目標群組成員上成功完成的時間。



## <a name="next-steps"></a>後續步驟

在本教學課程中，您已了解如何：

> [!div class="checklist"]

> * 建立可跨多個資料庫執行 T-SQL 作業的作業代理程式
> * 更新所有租用戶資料庫中的參考資料
> * 針對所有租用戶資料庫中的資料表建立索引

接下來，請嘗試[隨選報表教學課程](saas-tenancy-cross-tenant-reporting.md)來探索跨租用戶資料庫執行的分散式查詢。


## <a name="additional-resources"></a>其他資源

* [在 Tickets SaaS Database Per Tenant 應用程式部署上建置的其他教學課程](saas-dbpertenant-wingtip-app-overview.md#sql-database-wingtip-saas-tutorials)
* [管理相應放大的雲端資料庫](sql-database-elastic-jobs-overview.md)
* [建立和管理相應放大的雲端資料庫](sql-database-elastic-jobs-create-and-manage.md)