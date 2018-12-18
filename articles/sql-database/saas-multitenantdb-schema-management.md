---
title: 在多租用戶應用程式中管理 Azure SQL Database 結構描述 | Microsoft Docs
description: 在使用 Azure SQL Database 的多租用戶應用程式中，管理多租用戶的結構描述
keywords: SQL Database Azure
services: sql-database
author: MightyPen
manager: craigg
ms.service: sql-database
ms.custom: scale out apps
ms.topic: conceptual
ms.date: 09/19/2018
ms.reviewers: billgib
ms.author: genemi
ms.openlocfilehash: 2f22ed862f9b45334a961f80e47ee2b4634e46fb
ms.sourcegitcommit: 8b694bf803806b2f237494cd3b69f13751de9926
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/20/2018
ms.locfileid: "46498329"
---
# <a name="manage-schema-in-a-saas-application-that-uses-sharded-multi-tenant-sql-databases"></a>在使用分區化多租用戶 SQL 資料庫的 SaaS 應用程式中管理結構描述
 
本教學課程會檢視在 SaaS 應用程式中維護多個資料庫的挑戰。 將針對跨多個資料庫散開的結構描述變更示範解決方案。

就像任何應用程式一樣，Wingtip Tickets SaaS 應用程式會隨著時間演進，而且會需要變更資料庫。 變更可能會影響結構描述或參考資料，或產生資料庫維護工作。 由於 SaaS 應用程式使用每一租用戶一個資料庫的模式，因此可能必須在大量租用戶資料庫間協調變更。 此外，您必須將這些變更併入資料庫佈建程序，確保建立新資料庫時會涵蓋這些變更。

#### <a name="two-scenarios"></a>兩種情況

本教學課程會探討下列兩種情況：
- 為所有租用戶部署參考資料更新。
- 在包含參考資料的資料表上重建索引。

Azure SQL Database 的[彈性作業](sql-database-elastic-jobs-overview.md) 功能可用來執行這些跨租用戶資料庫的作業。 並且在「範本」租用戶資料庫上執行作業。 Wingtip Tickets 範例應用程式會複製範本資料庫來佈建新的租用戶資料庫。

在本教學課程中，您將了解如何：

> [!div class="checklist"]
> * 建立作業代理程式。
> * 在多租用戶資料庫上執行 T-SQL 查詢。
> * 更新所有租用戶資料庫中的參考資料。
> * 針對所有租用戶資料庫中的資料表建立索引。

## <a name="prerequisites"></a>必要條件

- 必須已部署 Wingtip Tickets 多租用戶資料庫應用程式：
    - 如需相關指示，請參閱第一個教學課程，其中會介紹 Wingtip Tickets SaaS 多租戶資料庫應用程式：<br />[部署及探索使用 Azure SQL Database 的分區化多租用戶應用程式](saas-multitenantdb-get-started-deploy.md)。
        - 部署程序會在五分鐘內執行完成。
    - 您必須已安裝 Wingtip 的「分區化多租用戶」版本。 本教學課程不支援「獨立」和「每一租用戶一個資料庫」的版本。

- 必須已安裝最新版的 SQL Server Management Studio (SSMS)。 [下載並安裝 SSMS](https://docs.microsoft.com/sql/ssms/download-sql-server-management-studio-ssms)。

- 必須已安裝 Azure PowerShell。 如需詳細資料，請參閱[開始使用 Azure PowerShell](https://docs.microsoft.com/powershell/azure/get-started-azureps)。

> [!NOTE]
> 本教學課程使用的 Azure SQL Database 服務功能處於有限預覽版狀態 ([彈性資料庫作業](sql-database-elastic-database-client-library.md))。 如果想要進行本教學課程，請將您的訂用帳戶識別碼提供給 SaaSFeedback@microsoft.com，並使用主旨 Elastic Jobs Preview (彈性作業預覽)。 在您收到訂用帳戶已啟用的確認之後，請[下載並安裝最新的發行前版本作業 Cmdlet (英文)](https://github.com/jaredmoo/azure-powershell/releases)。 這是有限預覽版，如果有相關問題或需要支援，請連絡 SaaSFeedback@microsoft.com。

## <a name="introduction-to-saas-schema-management-patterns"></a>SaaS 結構描述管理模式的簡介

此範例中使用的分區化多租用戶資料庫模型，讓租用戶資料庫可包含一個或多個租用戶。 此範例會啟用「混合式」租用戶管理模型，探索混合使用多租戶與單一租用戶資料庫的可能性。 管理這些資料庫的變更可能很複雜。 [彈性作業](sql-database-elastic-jobs-overview.md)有助於大量資料庫的管理。 這些作業可讓您安全且可靠地對一組租用戶資料庫執行 Transact-SQL 指令碼工作。 這些工作與使用者互動或輸入無關。 此方法可用來在應用程式中跨所有租用戶部署結構描述或一般參考資料變更。 彈性作業也可用來維護資料庫標準範本的複本。 範本會用來建立新的租用戶，並確保永遠使用最新的結構描述和參考資料。

![畫面](media/saas-multitenantdb-schema-management/schema-management.png)

## <a name="elastic-jobs-limited-preview"></a>彈性作業有限預覽版

新版本的「彈性作業」現在是 Azure SQL Database 的整合功能。 這個新的「彈性作業」版本目前處於有限預覽版狀態。 有限預覽版目前支援使用 PowerShell 建立作業代理程式，以及使用 T-SQL 建立及管理作業。
> [!NOTE] 
> 本教學課程使用的 SQL Database 服務功能處於有限預覽版狀態 (彈性資料庫工作)。 如果想要進行本教學課程，請將您的訂用帳戶識別碼提供給 SaaSFeedback@microsoft.com，並使用主旨 Elastic Jobs Preview (彈性作業預覽)。 在您收到訂用帳戶已啟用的確認之後，請下載並安裝最新的發行前版本作業 Cmdlet。 這是有限預覽版，如果有相關問題或需要支援，請連絡 SaaSFeedback@microsoft.com。

## <a name="get-the-wingtip-tickets-saas-multi-tenant-database-application-source-code-and-scripts"></a>取得 Wingtip Tickets SaaS 多租用戶資料庫應用程式原始碼和指令碼

可在 GitHub 的 [WingtipTicketsSaaS-MultitenantDB](https://github.com/microsoft/WingtipTicketsSaaS-MultiTenantDB) 存放庫中使用 Wingtip Tickets SaaS 多租用戶資料庫指令碼和應用程式來源程式碼。 關於下載和解除封鎖 Wingtip Tickets SaaS 指令碼的步驟，請參閱[一般指引](saas-tenancy-wingtip-app-guidance-tips.md)。 

## <a name="create-a-job-agent-database-and-new-job-agent"></a>建立作業代理程式資料庫和新的作業代理程式

本教學課程要求您必須使用 PowerShell 建立作業代理程式資料庫和作業代理程式。 如同 SQL Agent 使用的 MSDB 資料庫，作業代理程式會使用 Azure SQL 資料庫來儲存作業定義、作業狀態和記錄。 一旦建立作業代理程式後，您就可以立即建立及監視作業。

1. 在 **PowerShell ISE** 中開啟...\\Learning Modules\\Schema Management\\Demo-SchemaManagement.ps1。
2. 按 **F5** 以執行指令碼。

Demo-SchemaManagement.ps1 指令碼會呼叫 Deploy-SchemaManagement.ps1 指令碼，以在類別目錄伺服器上建立名為 jobagent 的資料庫。 指令碼接著會建立作業代理程式，將 jobagent 資料庫作為參數來傳遞。

## <a name="create-a-job-to-deploy-new-reference-data-to-all-tenants"></a>建立作業以將新的參考資料部署到所有租用戶

#### <a name="prepare"></a>準備

每個租用戶的資料庫都會在 **VenueTypes** 資料表中包含一組場地類型。 每個場地類型都會定義可在場地舉辦的事件種類。 這些場地類型會對應至您在租用戶活動應用程式中看到的背景影像。  在這個練習中，您要將更新部署到所有資料庫，以新增兩個額外的場地類型：*Motorcycle Racing* (機車賽) 和 *Swimming Club* (游泳俱樂部)。 

首先，檢閱每個租用戶資料庫中的場地類型。 連線至 SQL Server Management Studio (SSMS) 中的其中一個租用戶資料庫，並檢查 VenueTypes 資料表。  您也可以在 Azure 入口網站的查詢編輯器中查詢此資料表 (從資料庫頁面進行存取)。 

1. 開啟 SSMS 並連線到租用戶伺服器：tenants1-dpt-&lt;user&gt;.database.windows.net
1. 若要確認目前「尚未」包含 Motorcycle Racing (機車賽) 和 Swimming Club (游泳俱樂部)，請瀏覽至 tenants1-dpt-&lt;user&gt; 伺服器上的 contosoconcerthall 資料庫，並查詢 VenueTypes 資料表。



#### <a name="steps"></a>步驟

現在您會透過新增兩個新的場地類型，來建立作業並更新每個租用戶資料庫中的 **VenueTypes** 資料表。

若要建立新的作業，您可以使用我們在 _jobagent_ 資料庫中建立的一組作業系統預存程序。 作業代理程式建好後即會建立預存程序。

1. 在 SSMS 中，連線到租用戶伺服器：tenants1-mt-&lt;user&gt;.database.windows.net

2. 瀏覽至 tenants1 資料庫。

3. 查詢 VenueTypes 資料表以確認 Motorcycle Racing (機車賽) 和 Swimming Club (游泳俱樂部) 尚不在結果清單中。

4. 連線到目錄伺服器，也就是 *catalog-mt-&lt;user&gt;.database.windows.net*。

5. 連線至目錄伺服器中的 jobagent 資料庫。

6. 在 SSMS 中，開啟檔案…\\Learning Modules\\Schema Management\\DeployReferenceData.sql。

7. 修改陳述式：set @User = &lt;user&gt; ，並使用您在部署 Wingtip Tickets SaaS 多租用戶資料庫應用程式時使用的 User 值來取代。

8. 按 **F5** 以執行指令碼。

#### <a name="observe"></a>觀察

觀察 DeployReferenceData.sql 指令碼中的下列項目：

- **sp\_add\_target\_group** 會建立目標群組名稱 DemoServerGroup，並將目標成員新增至該群組。

- **sp\_add\_target\_group\_member** 會新增下列項目：
    - server 目標成員類型。
        - 這是包含在租用戶資料庫中的 *tenants1-mt-&lt;user&gt;* 伺服器。
        - 所包含的伺服器包括在作業執行時存在的租用戶資料庫。
    - 範本資料庫 (*basetenantdb*) 的 database 目標成員類型，範本資料庫位於 *catalog-mt-&lt;user&gt;* 伺服器。
    - 包含 adhocreporting 資料庫以用於之後教學課程的 database 目標成員類型。

- **sp\_add\_job** 會建立稱為「參考資料部署」的作業。

- **sp\_add\_jobstep** 會建立作業步驟，包含更新參考 VenueTypes 資料表的 T-SQL 命令文字。

- 指令碼中的其餘檢視會顯示物件是否存在，以及監視作業執行。 使用這些查詢來檢閱 **lifecycle** 資料行中的狀態值，以判斷作業完成的時間。 作業會更新租用戶資料庫，並更新包含參考資料表的其他兩個資料庫。

在 SSMS 中，瀏覽至 tenants1-mt-&lt;user&gt; 伺服器上的租用戶資料庫。 查詢 VenueTypes 資料表以確認 Motorcycle Racing (機車賽) 和 Swimming Club (游泳俱樂部) 現在已新增至資料表中。 場地類型的總數應已增加兩個。

## <a name="create-a-job-to-manage-the-reference-table-index"></a>建立作業以管理參考資料表索引

此練習會在所有租用戶資料庫的參考資料表主索引鍵上，建立重建索引的作業。 索引重建是典型的資料庫管理作業，為提升效能，系統管理員可能會在載入大量資料負載後執行此作業。

1. 在 SSMS 中，連線到 catalog-mt-&lt;User&gt;.database.windows.net 伺服器中的 jobagent 資料庫。

2. 在 SSMS 中，開啟...\\Learning Modules\\Schema Management\\OnlineReindex.sql。

3. 按 **F5** 以執行指令碼。

#### <a name="observe"></a>觀察

請觀察 OnlineReindex.sql 指令碼中的下列項目：

* **sp\_add\_job** 會建立稱為 Online Reindex PK\_\_VenueTyp\_\_265E44FD7FD4C885 的新作業。

* **sp\_add\_jobstep** 會建立作業步驟，其中包含要更新索引的 T-SQL 命令文字。

* 指令碼中的其餘檢視會監視作業執行。 使用這些查詢來檢閱 **lifecycle** 資料行中的狀態值，以判斷作業在所有目標群組成員上成功完成的時間。

## <a name="additional-resources"></a>其他資源

<!-- TODO: Additional tutorials that build upon the Wingtip Tickets SaaS Multi-tenant Database application deployment (*Tutorial link to come*)
(saas-multitenantdb-wingtip-app-overview.md#sql-database-wingtip-saas-tutorials)
-->
* [管理相應放大的雲端資料庫](sql-database-elastic-jobs-overview.md)
* [建立和管理相應放大的雲端資料庫](sql-database-elastic-jobs-create-and-manage.md)

## <a name="next-steps"></a>後續步驟

在本教學課程中，您已了解如何：

> [!div class="checklist"]
> * 建立可跨多個資料庫執行 T-SQL 作業的作業代理程式
> * 更新所有租用戶資料庫中的參考資料
> * 針對所有租用戶資料庫中的資料表建立索引

接下來，請嘗試[隨選報表教學課程](saas-multitenantdb-adhoc-reporting.md)來探索跨租用戶資料庫執行的分散式查詢。

