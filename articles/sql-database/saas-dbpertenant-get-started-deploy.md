---
title: 每一租用戶一個資料庫 SaaS 教學課程 - Azure SQL Database | Microsoft Docs
description: 部署及探索 Wingtip Tickets SaaS 多租用戶應用程式，其使用 Azure SQL Database 示範每一租用戶一個資料庫模式和其他 SaaS 模式。
services: sql-database
ms.service: sql-database
ms.subservice: scenario
ms.custom: ''
ms.devlang: ''
ms.topic: conceptual
author: MightyPen
ms.author: genemi
ms.reviewer: sstein
manager: craigg
ms.date: 04/01/2018
ms.openlocfilehash: 77e3cdcbd18a4a5313160b947ce278a75f3e3de3
ms.sourcegitcommit: 715813af8cde40407bd3332dd922a918de46a91a
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "47056381"
---
# <a name="deploy-and-explore-a-multitenant-saas-app-that-uses-the-database-per-tenant-pattern-with-sql-database"></a>部署及探索搭配 SQL Database 使用每一租用戶一個資料庫模式的多租用戶 SaaS 應用程式

在本教學課程中，您將部署及探索 Wingtip Tickets SaaS 每一租用戶一個資料庫應用程式 (Wingtip)。 本應用程式使用每一租用戶一個資料庫模式來儲存多租用戶的資料。 本應用程式專為展示簡化 SaaS 案例啟用方式的 Azure SQL Database 功能而設計。

選取 [部署至 Azure] 後五分鐘，多租用戶 SaaS 應用程式便部署完成。 應用程式包含在雲端中執行的 SQL 資料庫。 應用程式中部署了三個範例租用戶，每個租用戶都有自己的資料庫。 所有資料庫均已部署到 SQL 彈性集區。 應用程式會部署到您的 Azure 訂用帳戶。 您可以完整存取，以便探索和使用應用程式的每個元件。 您可以在 [WingtipTicketsSaaS-DbPerTenant GitHub 存放庫][github-wingtip-dpt]，取得應用程式 C# 原始程式碼和管理指令碼。

您會在本教學課程中學到：

> [!div class="checklist"]
> - 如何部署 Wingtip SaaS 應用程式。
> - 在何處取得應用程式原始程式碼和管理指令碼。
> - 關於組成應用程式的伺服器、集區和資料庫。
> - 如何使用「目錄」將租用戶對應到其資料。
> - 如何佈建新租用戶。
> - 如何監視應用程式中的租用戶活動。

我們提供了[一系列的相關教學課程](saas-dbpertenant-wingtip-app-overview.md#sql-database-wingtip-saas-tutorials)，帶領您探索各式各樣的 SaaS 設計和管理模式。 這些教學課程所涵蓋的內容，超出本文的初始部署程序。 閱讀教學課程時，您可以檢查我們提供的指令碼，了解各個 SaaS 模式的實作方式。 指令碼示範 SQL Database 的功能如何簡化 SaaS 應用程式的開發過程。

## <a name="prerequisites"></a>必要條件

若要完成本教學課程，請確定您已安裝 Azure PowerShell。 如需詳細資訊，請參閱[開始使用 Azure PowerShell](https://docs.microsoft.com/powershell/azure/get-started-azureps)。

## <a name="deploy-the-wingtip-tickets-saas-application"></a>部署 Wingtip Tickets SaaS 應用程式

#### <a name="plan-the-names"></a>計劃名稱

在這一節的步驟中，您要提供一個使用者值，用來確定資源名稱是全域唯一的。 您還要提供一個資源群組的名稱，該資源群組包含應用程式部署所建立的所有資源。 假設虛構使用者的姓名為 Ann Finley，我們建議：

- **使用者**：af1，是由 Ann Finley 的縮寫加上數字組成。 如果您是第二次部署應用程式，請使用另一個值。 例如，af2。
- **資源群組**：wingtip-dpt-af1 表示這是每一租用戶一個資料庫應用程式。 加上使用者名稱 af1，會使資源群組名稱與其中各項資源的名稱相互關聯。

現在請選擇您的名稱，然後將它們寫下來。 

#### <a name="steps"></a>步驟

1. 若要在 Azure 入口網站中開啟 Wingtip Tickets SaaS 每一租用戶一個資料庫的部署範本，請選取 [部署至 Azure]。

   <a href="https://aka.ms/deploywingtipdpt" target="_blank"><img src="http://azuredeploy.net/deploybutton.png"/></a>

1. 在範本中，為必要參數輸入值。

    > [!IMPORTANT]
    > 為了示範的目的，已刻意將某些驗證和伺服器防火牆設為不安全。 建議您建立新的資源群組。 請勿使用現有的資源群組、伺服器或集區。 請勿將此應用程式、指令碼或任何部署的資源用於生產環境。 不使用應用程式之後，請將此資源群組刪除，以停止相關計費。

    - **資源群組**：選取 [新建]，並提供您先前為資源群組選擇的唯一名稱。 
    - **位置**：從下拉式清單中選取位置。
    - **使用者**：使用您先前選擇的使用者名稱值。

1. 部署應用程式。

    a. 選取以同意條款和條件。

    b. 選取 [購買]。

1. 若要監視部署狀態，請選取 [通知] \(搜尋方塊右側的鈴鐺圖示)。 部署 Wingtip Tickets SaaS 應用程式大約需要五分鐘。

   ![部署成功](media/saas-dbpertenant-get-started-deploy/succeeded.png)

## <a name="download-and-unblock-the-wingtip-tickets-management-scripts"></a>下載並解除封鎖 Wingtip Tickets 管理指令碼

部署應用程式時，請下載原始程式碼和管理指令碼。

> [!IMPORTANT]
> 從外部來源下載 zip 檔案並進行解壓縮時，Windows 可能會封鎖可執行的內容 (指令碼和 DLL)。 遵循步驟將 .zip 檔案解除封鎖後，再擷取指令碼。 解除封鎖可確保系統允許指令碼執行。

1. 瀏覽至 [WingtipTicketsSaaS-DbPerTenant GitHub 存放庫][github-wingtip-dpt]。
1. 選取 [複製或下載] 。
1. 選取 [下載 ZIP]，並儲存檔案。
1. 以滑鼠右鍵按一下 **WingtipTicketsSaaS-DbPerTenant-master.zip** 檔案，然後選取 [屬性]。
1. 在 [一般] 索引標籤上，選取 [解除封鎖] > [套用]。
1. 選取 [確定]，並將檔案解壓縮

指令碼位於 \\WingtipTicketsSaaS-DbPerTenant-master\\Learning Modules 資料夾中。

## <a name="update-the-user-configuration-file-for-this-deployment"></a>更新此部署的使用者組態檔

執行任何指令碼之前，請更新 User Config 檔案中「資源群組」和「使用者」的值。 將這些變數設定為您在部署期間使用的值。

1. 在 [PowerShell ISE] 中，開啟 ...\\Learning Modules\\**UserConfig.psm1** 
1. 將 ResourceGroupName 和 Name 更新為您部署的特定值 (只在第 10 行和第 11 行)。
1. 儲存變更。

幾乎每一個指令碼中都會參考這些數值。

## <a name="run-the-application"></a>執行應用程式

應用程式會示範裝載事件的場地。 場地類型包括音樂廳、爵士俱樂部和運動俱樂部。 在 Wingtip Tickets 中，場地會註冊為租用戶。 成為租用戶讓場地能夠輕鬆列出活動，並對其客戶銷售票券。 每個場地都有客製化網站可列出其活動及售票。

在應用程式內部，每個租用戶都有一個部署在 SQL 彈性集區中的 SQL 資料庫。

中央 [事件中樞] 頁面提供部署中租用戶的連結清單。

1. 在網頁瀏覽器中使用 URL 開啟事件中樞： http://events.wingtip-dpt.&lt;user&gt;.trafficmanager.net。 將 &lt;user&gt; 取代為部署的使用者值。

    ![事件中樞](media/saas-dbpertenant-get-started-deploy/events-hub.png)

1. 選取 [事件中樞] 中的 [Fabrikam Jazz Club]。

    ![活動](./media/saas-dbpertenant-get-started-deploy/fabrikam.png)

#### <a name="azure-traffic-manager"></a>Azure 流量管理員

Wingtip 應用程式使用 [Azure 流量管理員](../traffic-manager/traffic-manager-overview.md)來控制傳入要求的散發方式。 存取特定租用戶之事件頁面的 URL 使用下列格式：

- http://events.wingtip-dpt.&lt;user&gt;.trafficmanager.net/fabrikamjazzclub

    下表有前述格式各部分的解說。

    | URL 部分        | 說明       |
    | :-------------- | :---------------- |
    | http://events.wingtip-dpt | Wingtip 應用程式的事件部分。<br /><br /> -dpt 能將 Wingtip Tickets 的「每一租用戶一個資料庫」實作與其他實作進行區分。 例如，「獨立的」每一租用戶一個應用程式 (-sa)，或「多租用戶資料庫」 (-mt) 實作。 |
    | .&lt;user&gt; | 範例中的 af1。 |
    | .trafficmanager.net/ | 流量管理員，基底 URL。 |
    | fabrikamjazzclub | 識別名為 Fabrikam Jazz Club 的租用戶。 |
    | &nbsp; | &nbsp; |

* 租用戶名稱由事件應用程式從 URL 剖析而來。
* 租用戶名稱可用來建立索引鍵。
* 索引鍵可用來存取目錄，以取得租用戶資料庫的位置。
    - 目錄是使用「分區對應管理」進行實作。
* [事件中樞] 會使用目錄中的延伸中繼資料，建構每個租用戶的事件頁面 URL 清單。

在生產環境下，一般來說，會建立 CNAME DNS 記錄以[將公司的網際網路網域指向](../traffic-manager/traffic-manager-point-internet-domain.md)流量管理員的 DNS 名稱。

## <a name="start-generating-load-on-the-tenant-databases"></a>開始在租用戶資料庫上產生負載

應用程式已部署完成，請立即使用。

Demo-LoadGenerator PowerShell 指令碼會啟動對所有租用戶資料庫執行的工作負載。 許多 SaaS 應用程式的實際負載是偶發且無法預期的。 為了模擬這種類型的負載，產生器會在每個租用戶上產生出現隨機高峰或活動暴增的負載。 暴增的活動會依照隨機間隔發生。 負載模式需要幾分鐘時間才會出現。 請讓產生器先執行至少三到四分鐘後，再監視負載。

1. 在 [PowerShell ISE] 中開啟 ...\\Learning Modules\\Utilities\\*Demo-LoadGenerator.ps1* 指令碼。
1. 按 F5 執行指令碼及啟動負載產生器。 目前請先保留預設參數值。
1. 如有需要，請登入 Azure 帳戶，然後選取您要使用的訂用帳戶。

負載產生器指令碼會為目錄中的每個資料庫啟動背景作業，然後停止。 如果您重新執行負載產生器指令碼，它會先停止所有執行中的背景作業後，再啟動新的作業。

#### <a name="monitor-the-background-jobs"></a>監視背景作業

如果您需要控制和監視背景作業，可以使用下列 Cmdlet：

- `Get-Job`
- `Receive-Job`
- `Stop-Job`

#### <a name="demo-loadgeneratorps1-actions"></a>Demo-LoadGenerator.ps1 動作

Demo-LoadGenerator.ps1 會模擬客戶交易的作用中工作負載。 下列步驟描述 Demo-LoadGenerator.ps1 起始之動作的順序：

1. Demo-LoadGenerator.ps1 會在前景啟動 LoadGenerator.ps1。

    - 這兩個 .ps1 檔案都儲存在 Learning Modules\\Utilities\\ 資料夾下方。

1. LoadGenerator.ps1 會對目錄中的所有租用戶資料庫執行迴圈。

1. LoadGenerator.ps1 會針對每個租用戶資料庫啟動背景 PowerShell 作業：

    - 依預設，背景作業會執行 120 分鐘。
    - 每個作業都會執行 sp_CpuLoadGenerator，以在一個租用戶資料庫上引發 CPU 負載。 負載的強度和持續時間取決於 `$DemoScenario`。 
    - sp_CpuLoadGenerator 會圍繞著引發高 CPU 負載的 SQL SELECT 陳述式執行迴圈。 發出 SELECT 的時間間隔取決於參數值，以便建立可控制的 CPU 負載。 系統會隨機選擇負載層級和時間間隔，以使模擬的負載更真實。
    - .sql 檔案儲存在 WingtipTenantDB\\dbo\\StoredProcedures\\ 中。

1. 如果 `$OneTime = $false`，負載產生器會啟動背景作業，然後再繼續執行。 每隔 10 秒鐘，系統會監視所有已佈建的新租用戶。 如果您設定 `$OneTime = $true`，LoadGenerator 將啟動背景作業，然後在前景中停止執行。 請針對本教學課程預留 `$OneTime = $false` 的時間。

  如果您需要停止或重新啟動負載產生器，請使用 Ctrl-C 或停止作業 Ctrl-Break。 

  如果您將負載產生器保留在前景執行，請使用另一個 PowerShell ISE 執行個體來執行其他 PowerShell 指令碼。

&nbsp;

在繼續進行下一節之前，請讓負載產生器繼續以作業叫用狀態執行。

## <a name="provision-a-new-tenant"></a>佈建新租用戶

初始部署會建立三個範例租用戶。 現在您可以建立另一個租用戶，看看對已部署的應用程式會造成什麼影響。 在 Wingtip 應用程式中佈建新租用戶的工作流程如[佈建和目錄教學課程](saas-dbpertenant-provision-and-catalog.md)所述。 在本階段，您將以不到一分鐘的時間建立新租用戶。

1. 開啟新的 PowerShell ISE。
1. 開啟 ...\\Learning Modules\Provision and Catalog\\Demo-ProvisionAndCatalog.ps1。
1. 若要執行指令碼，請按 F5。 目前請先保留預設值。

   > [!NOTE]
   > 許多 Wingtip SaaS 指令碼都會使用 $PSScriptRoot 來瀏覽資料夾，以在其他指令碼中呼叫函式。 按 F5 執行完整的指令碼時，才會評估這個變數。 使用 F8 來醒目提示及執行選取項目可能會導致錯誤。 若要執行指令碼，請按 F5。

新的租用戶資料庫：

- 建立在 SQL 彈性集區中。
- 已初始化。
- 已在目錄中註冊。

成功佈建之後，瀏覽器中就會顯示新租用戶的「事件」網站。

![新租用戶](./media/saas-dbpertenant-get-started-deploy/red-maple-racing.png)

重新整理事件中樞，讓新租用戶出現在清單中。

## <a name="explore-the-servers-pools-and-tenant-databases"></a>探索伺服器、集區和租用戶資料庫

既然您已經開始對租用戶集合執行負載，讓我們看看已部署的一些資源。

1. 在 [Azure 入口網站](http://portal.azure.com)中，瀏覽到您的 SQL 伺服器清單。 然後開啟 **catalog-dpt-&lt;USER&gt;** 伺服器。
    - 目錄伺服器包含兩個資料庫，**tenantcatalog** 和 **basetenantdb** (basetenantdb 是複製的範本資料庫，用以建立新租用戶)。

   ![資料庫](./media/saas-dbpertenant-get-started-deploy/databases.png)

1. 返回 SQL Server 清單。

1. 開啟內含租用戶資料庫的 **tenants1-dpt-&lt;USER&gt;** 伺服器。

1. 查看下列項目：

    - 每個租用戶資料庫都是 50-eDTU 標準集區中的「彈性標準」資料庫。
    - Red Maple Racing 資料庫是您先前佈建的租用戶資料庫。

   ![包含資料庫的伺服器](./media/saas-dbpertenant-get-started-deploy/server.png)

## <a name="monitor-the-pool"></a>監視集區

*LoadGenerator.ps1* 執行幾分鐘之後，您應該能獲得足夠的資料來觀察一些監視功能。 這些監視功能已內建在集區和資料庫中。

瀏覽至 **tenants1-dpt-&lt;user&gt;** 伺服器，然後選取 [Pool1] 檢視集區的資源使用率。 下列圖表中的負載產生器執行了一個小時。

   ![監視集區](./media/saas-dbpertenant-get-started-deploy/monitor-pool.png)

- 第一個圖表 (標示為**資源使用率**) 顯示集區的 eDTU 使用率。
- 第二個圖表顯示集區中五個最常使用之資料庫的 eDTU 使用率。

這兩個圖表說明彈性集區和 SQL Database 非常適合無法預期的 SaaS 應用程式工作負載。 圖表顯示這 4 個資料庫的使用率均爆增至 40 eDTU，不過所有資料庫都受到一個 50-eDTU 集區的支援，所以不需擔心。 50-eDTU 集區甚至能支援更繁重的工作負載。 如果這些資料庫是以單一資料庫形式佈建，則它們每個都必須是 S2 (50 DTU) 才能支援高載。 4 個獨立 S2 資料庫的成本幾乎是集區價格的 3 倍。 在現實情況下，SQL Database 客戶會在 200 eDTU 集區中執行多達 500 個資料庫。 如需詳細資訊，請參閱[效能監視教學課程](saas-dbpertenant-performance-monitoring.md)。

## <a name="additional-resources"></a>其他資源

- 如需詳細資訊，請參閱其他[以 Wingtip Tickets SaaS 每一租用戶一個資料庫應用程式為基礎的其他教學課程](saas-dbpertenant-wingtip-app-overview.md#sql-database-wingtip-saas-tutorials)。
- 若要了解彈性集區，請參閱[什麼是 Azure SQL 彈性集區？](sql-database-elastic-pool.md)。
- 若要了解彈性作業，請參閱[管理相應放大的雲端資料庫](sql-database-elastic-jobs-overview.md)。
- 若要了解多租用戶 SaaS 應用程式，請參閱[多租用戶 SaaS 應用程式的設計模式](saas-tenancy-app-design-patterns.md)。


## <a name="next-steps"></a>後續步驟

在本教學課程中，您已了解：

> [!div class="checklist"]
> - 如何部署 Wingtip Tickets SaaS 應用程式。
> - 關於組成應用程式的伺服器、集區和資料庫。
> - 如何使用「目錄」將租用戶對應到其資料。
> - 如何佈建新的租用戶。
> - 如何檢視集區使用率以監視租用戶活動。
> - 如何刪除範例資源以停止相關計費。

接下來請嘗試[佈建和目錄教學課程](saas-dbpertenant-provision-and-catalog.md)。



<!-- Link references. -->

[github-wingtip-dpt]: https://github.com/Microsoft/WingtipTicketsSaaS-DbPerTenant 

