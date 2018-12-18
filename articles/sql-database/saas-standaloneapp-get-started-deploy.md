---
title: 單一租用戶 SaaS 教學課程 - Azure SQL Database | Microsoft Docs
description: 部署及探索使用 Azure SQL Database 的獨立單一租用戶 SaaS 應用程式。
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
ms.date: 09/19/2018
ms.openlocfilehash: 1e74a0c915471fecb3b8e0662559eb5bfe189b3d
ms.sourcegitcommit: 42405ab963df3101ee2a9b26e54240ffa689f140
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/28/2018
ms.locfileid: "47423012"
---
# <a name="deploy-and-explore-a-standalone-single-tenant-application-that-uses-azure-sql-database"></a>部署及探索使用 Azure SQL Database 的獨立單一租用戶應用程式

在本教學課程中，您將部署和探索 Wingtip Tickets SaaS 範例應用程式，其乃使用獨立應用程式 (亦稱為每一租用戶一個應用程式) 模式開發。  應用程式專為展示簡化多租用戶 SaaS 案例的 Azure SQL Database 功能而設計。

獨立應用程式 (每一租用戶一個應用程式) 模式會為每個租用戶部署一個應用程式執行個體。  每個應用程式都已針對特定租用戶設定，並部署在個別的 Azure 資源群組中。 可以佈建應用程式的多個執行個體來提供多租用戶解決方案。 這個模式最適合少量租用戶，因為租用戶隔離是第一要務。 Azure 的夥伴程式可讓服務提供者將資源部署到租用戶的訂用帳戶中，並代表租用戶管理這些資源。 

在本教學課程中，您會將三個租用戶的三個獨立應用程式部署到您的 Azure 訂用帳戶中。  您可以完整存取探索和使用個別應用程式元件。

在 [WingtipTicketsSaaS-StandaloneApp](https://github.com/Microsoft/WingtipTicketsSaaS-StandaloneApp) GitHub 存放庫可取得應用程式原始程式碼和管理指令碼。 應用程式是使用 Visual Studio 2015 建立的，若未更新，則無法在 Visual Studio 2017 中成功開啟及編譯。


您會在本教學課程中學到：

> [!div class="checklist"]
> * 如何部署 Wingtip Tickets SaaS Standalone 應用程式。
> * 在何處取得應用程式原始程式碼和管理指令碼。
> * 關於組成應用程式的伺服器和資料庫。

將會推出其他教學課程。 這些教學課程可讓您可以此應用程式模式作為基礎，探索各種管理情節。   

## <a name="deploy-the-wingtip-tickets-saas-standalone-application"></a>部署 Wingtip Tickets SaaS Standalone 應用程式

部署三個所提供之租用戶的應用程式：

1. 按一下每個藍色的 [部署至 Azure] 按鈕可在 [Azure 入口網站](https://portal.azure.com)中開啟部署範本。 每個範本需要兩個參數值：新資源群組的名稱，以及區分此部署與應用程式中其他部署的使用者名稱。 下一個步驟將詳細說明如何設定這些值。<br><br>
    <a href="http://aka.ms/deploywingtipsa-contoso" target="_blank"><img style="vertical-align:middle" src="media/saas-standaloneapp-get-started-deploy/deploy.png"/></a> &nbsp; **Contoso Concert Hall**
<br><br>
    <a href="http://aka.ms/deploywingtipsa-dogwood" target="_blank"><img style="vertical-align:middle" src="media/saas-standaloneapp-get-started-deploy/deploy.png"/></a> &nbsp; **Dogwood Dojo**
<br><br>
    <a href="http://aka.ms/deploywingtipsa-fabrikam" target="_blank"><img style="vertical-align:middle" src="media/saas-standaloneapp-get-started-deploy/deploy.png"/></a> &nbsp; **Fabrikam Jazz Club**

2. 輸入每個部署所需的參數值。

    > [!IMPORTANT]
    > 為了示範的目的，已刻意將某些驗證和伺服器防火牆設為不安全。 為每個應用程式部署**建立新的新資源群組**。  請勿使用現有的資源群組。 請不要將此應用程式或任何它所建立的資源用於生產環境。 當您不使用應用程式時，請刪除所有資源群組，以停止相關計費。

    資源名稱中最好只使用小寫字母、數字和連字號。
    * 針對**資源群組**選取 [新建]，並為資源群組提供小寫的 [名稱]。 我們建議使用 **wingtip-sa-\<venueName\>-\<user\>** 模式。  請將 \<venueName\> 替換為場地名稱，名稱中不可有空格。 請將 \<user\> 替換為下方的使用者名稱。  採用此模式後，資源群組名稱可能會是 wingtip-sa-contosoconcerthall-af1、wingtip-sa-dogwooddojo-af1、wingtip-sa-fabrikamjazzclub-af1。
    * 從下拉式清單中選取 [位置]。

    * 針對**使用者** - 建議您選擇簡短的使用者 值，可以用您的姓名首字母加上數字，例如：*af1*。


3. **部署應用程式**。

    * 按一下以同意條款和條件。
    * 按一下 [購買]。

4. 按一下 [通知] (搜尋方塊右側的鈴鐺圖示) 來監視所有三個部署的部署狀態。 部署應用程式大約需要五分鐘的時間。


## <a name="run-the-applications"></a>執行應用程式

應用程式會示範裝載事件的場地。  場地是指應用程式的租用戶。 每個場地都有客製化網站可刊登其活動及售票。 場地類型包括音樂廳、爵士俱樂部和運動俱樂部。 在範例中，場地的類型決定了場地網站上顯示的背景相片。   在獨立應用程式模式中，每個場地都有個別的應用程式執行個體，該執行個體有自己的獨立 SQL 資料庫。

1. 在不同的瀏覽器索引標籤中開啟三個租用戶的 [事件] 頁面：

    - http://events.contosoconcerthall.&lt;user&gt;.trafficmanager.net
    - http://events.dogwooddojo.&lt;user&gt;.trafficmanager.net
    - http://events.fabrikamjazzclub.&lt;user&gt;.trafficmanager.net

    (在每個 URL 中，將 &lt;user&gt; 取代為部署時用的使用者值)。

   ![活動](./media/saas-standaloneapp-get-started-deploy/fabrikam.png)

為了控制連入要求的發佈，應用程式會使用 [*Azure 流量管理員*](../traffic-manager/traffic-manager-overview.md)。 每個租用戶專屬的應用程式執行個體都包含租用戶名稱，以在 URL 中作為網域名稱的一部分。 所有租用戶 URL 都包含您的特定 [使用者] 值。 URL 的格式如下︰
- http://events.&lt;venuename&gt;.&lt;user&gt;.trafficmanager.net

每個租用戶的資料庫 [位置] 都包含在對應已部署應用程式的應用程式設定中。

在生產環境中，您通常會建立 CNAME DNS 記錄以[將公司網際網路網域指向](../traffic-manager/traffic-manager-point-internet-domain.md)流量管理員設定檔的 URL。


## <a name="explore-the-servers-and-tenant-databases"></a>探索伺服器和租用戶資料庫

讓我們看看一些已部署的資源：

1. 在 [Azure 入口網站](http://portal.azure.com)中，瀏覽至資源群組的清單。
2. 您應該會看到三個租用戶資源群組。
3. 開啟 **wingtip-sa-fabrikam-&lt;user&gt;** 資源群組，其中包含給 Fabrikam Jazz Club 部署的資源。  **fabrikamjazzclub-&lt;user&gt;** 伺服器內含 **fabrikamjazzclub** 資料庫。

每個租用戶資料庫都是 50 DTU 獨立資料庫。

## <a name="additional-resources"></a>其他資源

<!--
* Additional [tutorials that build on the Wingtip SaaS application](sql-database-wtp-overview.md#sql-database-wingtip-saas-tutorials)
* To learn about elastic pools, see [*What is an Azure SQL elastic pool*](https://docs.microsoft.com/azure/sql-database/sql-database-elastic-pool)
* To learn about elastic jobs, see [*Managing scaled-out cloud databases*](https://docs.microsoft.com/azure/sql-database/sql-database-elastic-jobs-overview)
-->

- 若要了解多租用戶 SaaS 應用程式，請參閱[多租用戶 SaaS 應用程式的設計模式](saas-tenancy-app-design-patterns.md)。

 
## <a name="delete-resource-groups-to-stop-billing"></a>刪除資源群組以停止計費 ##

使用完範例後，請刪除您建立的所有資源群組以停止相關計費。

## <a name="next-steps"></a>後續步驟

在本教學課程中，您已了解：

> [!div class="checklist"]
> * 如何部署 Wingtip Tickets SaaS Standalone 應用程式。
> * 關於組成應用程式的伺服器和資料庫。
> * 如何刪除範例資源以停止相關計費。

接下來請嘗試[佈建和目錄](saas-standaloneapp-provision-and-catalog.md)教學課程，探索如何使用租用戶目錄來滿足多種跨租用戶案例的需求，例如結構描述管理和租用戶分析。
 

