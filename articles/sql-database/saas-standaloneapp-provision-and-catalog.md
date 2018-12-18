---
title: 多租用戶 SaaS 教學課程 - Azure SQL Database | Microsoft Docs
description: 使用獨立應用程式模式佈建新租用戶並編目
keywords: SQL Database Azure
services: sql-database
author: stevestein
manager: craigg
ms.service: sql-database
ms.custom: SaaS
ms.topic: conceptual
ms.date: 01/31/2018
ms.author: billgib
ms.openlocfilehash: 0f2495ddc5d5053582d67bd44cdf80d018f79e42
ms.sourcegitcommit: 266fe4c2216c0420e415d733cd3abbf94994533d
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/01/2018
ms.locfileid: "34646148"
---
# <a name="provision-and-catalog-new-tenants-using-the--application-per-tenant-saas-pattern"></a>使用每一租用戶一個應用程式 SaaS 模式，佈建新租用戶及編目

本文說明如何使用每一租用戶一個獨立應用程式 SaaS 模式佈建新租用戶及編目。
本文包含兩個主要部分：
* 佈建新租用戶及編目的概念討論
* 強調說明完成佈建和編目之範例 PowerShell 指令碼的教學課程
    * 本教學課程使用 Wingtip Tickets SaaS 範例 SaaS 應用程式，並已針對每一租用戶一個獨立應用程式模式進行調整。

## <a name="standalone-application-per-tenant-pattern"></a>每一租用戶一個獨立應用程式模式
每一租用戶一個獨立應用程式模式，是幾個多租用戶 SaaS 應用程式模式的其中之一。  在這個模式中，每個租用戶都有一個已佈建的應用程式。 應用程式包含應用程式層級元件和 SQL 資料庫。  每個租用戶應用程式都能部署到廠商的訂用帳戶中。  Azure 另提供[受控應用程式方案](https://docs.microsoft.com/azure/managed-applications/overview)，將應用程式部署於租用戶的訂用帳戶中，再由廠商代為控管。 

   ![每一租用戶一個應用程式模式](media/saas-standaloneapp-provision-and-catalog/standalone-app-pattern.png)

在部署租用戶的應用程式時，應用程式和資料庫會佈建在為租用戶建立的新資源群組中。  使用各自的資源群組能將每個租用戶的應用程式資源區隔開來，以便獨立控管。 在每個資源群組中，每個應用程式執行個體均已設定為直接存取對應的資料庫。  相較於其他使用目錄來代理應用程式與資料庫間連線的模式，這樣的連線模式存在顯著差異。  由於缺少資源共用，所以為每個租用戶資料庫佈建的資源必須足以處理尖峰負載。 這個模式傾向用於租用戶較少的 SaaS 應用程式，因為它主要強調租用戶隔離，較少著墨於資源成本。  

## <a name="using-a-tenant-catalog-with-the-application-per-tenant-pattern"></a>搭配使用租用戶目錄與每一租用戶一個應用程式模式
雖然每個租用戶的應用程式和資料庫均徹底隔離，不過還是有幾個涉及多個租用戶的管理和分析案例。  舉例來說，若要為新版應用程式套用結構描述變更，就必須變更每個租用戶資料庫的結構描述。 報告和分析案例也需要存取所有租用戶資料庫才能完成，無論資料庫部署在何處皆然。

   ![每一租用戶一個應用程式模式](media/saas-standaloneapp-provision-and-catalog/standalone-app-pattern-with-catalog.png)

租用戶目錄能保存租用戶識別碼與租用戶資料庫之間的對應，允許將識別碼解析為伺服器和資料庫名稱。  在 Wingtip SaaS 應用程式中，租用戶識別碼採用租用戶名稱雜湊的形式加以計算，不過您也可以使用其他機制。  雖然獨立應用程式不需要目錄來管理連線，不過目錄能針對一組租用戶資料庫界定採取動作時的範圍。 舉例來說，彈性查詢能使用目錄來決定一組資料庫，界定要將跨租用戶報告的查詢要散發到哪些資料庫。

## <a name="elastic-database-client-library"></a>彈性資料庫用戶端程式庫
在 Wingtip SaaS 範例應用程式中，目錄的實作乃使用[彈性資料庫用戶端程式庫](https://docs.microsoft.com/azure/sql-database/sql-database-elastic-database-client-library) (EDCL) 的分區管理功能。  此程式庫可讓應用程式建立、管理及使用資料庫中所儲存的分區對應。 在 Wingtip Tickets 範例中，目錄會儲存在「租用戶目錄」資料庫中。  分區能讓租用戶索引鍵與儲存租用戶資料的分區 (資料庫) 相對應。  EDCL 函式負責管理儲存在「租用戶目錄」資料庫表格中的「全域分區對應」，以及儲存在每個分區中的「本機分區對應」。

您可以從應用程式或 PowerShell 指令碼叫用 EDCL 函式來建立及管理分區對應中的項目， 也可以使用其他 EDCL 函式來擷取分區集合，或讓特定租用戶索引鍵與正確的資料庫連線。 
    
> [!IMPORTANT] 
> 切勿直接編輯目錄資料庫中的資料，或是編輯租用戶資料庫中的本機分區對應。 由於資料毀損的風險過高，因此我們不支援直接更新的做法。 請改為僅使用 EDCL API 編輯對應資料。

## <a name="tenant-provisioning"></a>租用戶佈建 
每個租用戶都需要新的 Azure 資源群組，而您必須先建立資源群組才能在其中佈建資源。 等到資源群組建立後，您就可以使用 Azure 資源管理範本部署應用程式元件和資料庫，接著再設定資料庫連線。 若要將資料庫結構描述初始化，您可以將 bacpac 檔案匯入範本。  您也可以採用「範本」資料庫副本的形式建立資料庫。  接下來，您可以進一步更新資料庫的初始場地資料，並在目錄中註冊資料庫。

## <a name="tutorial"></a>教學課程

在本教學課程中，您將了解如何：
* 佈建目錄
* 在目錄中註冊先前部署的範例租用戶資料庫
* 佈建額外租用戶並在目錄中註冊

Azure Resource Manager 範本可用來部署及設定應用程式、建立租用戶資料庫，然後再匯入 bacpac 檔案並加以初始化。 系統可能會先將匯入要求排入佇列幾分鐘，隨後才實際執行動作。

本教學課程結束後，您將擁有一組獨立租用戶應用程式，並擁有每個已在目錄中註冊的資料庫。

## <a name="prerequisites"></a>先決條件
若要完成本教學課程，請確定已完成下列必要條件： 
* 已安裝 Azure PowerShell。 如需詳細資料，請參閱[開始使用 Azure PowerShell](https://docs.microsoft.com/powershell/azure/get-started-azureps)
* 已部署三個範例租用戶應用程式。 若要在五分鐘內完成這些應用程式的部署，請參閱[部署及探索 Wingtip Tickets SaaS 獨立應用程式模式](https://docs.microsoft.com/azure/sql-database/saas-standaloneapp-get-started-deploy)。

## <a name="provision-the-catalog"></a>佈建目錄
在這項工作中，您將了解如何佈建用來註冊所有租用戶資料庫的目錄。 您將： 
* 使用 Azure 資源管理範本**佈建目錄資料庫**。 資料庫將會在 bacpac 檔案匯入後初始化。  
* **註冊先前部署的範例租用戶應用程式**。  每個租用戶在註冊時，都會使用從租用戶名稱雜湊建構而成的索引鍵。  租用戶名稱也會儲存在目錄的延伸模組資料表中。

1. 在 PowerShell ISE 中，開啟 *...\Learning Modules\UserConfig.psm*，接著將 **\<user\>** 值更新為部署三個範例應用程式時使用的值。  **儲存檔案**。  
1. 在 PowerShell ISE 中，開啟 *...\Learning Modules\ProvisionTenants\Demo-ProvisionAndCatalog.ps1*，並設定 **$Scenario = 1**。 部署租用戶目錄並註冊預先定義的租用戶。

1. 將游標置於顯示 `& $PSScriptRoot\New-Catalog.ps1` 該行的任意位置，然後按 **F9** 新增中斷點。

    ![設定追蹤中斷點](media/saas-standaloneapp-provision-and-catalog/breakpoint.png)

1. 按 **F5** 執行指令碼。 
1.  在指令碼於中斷點停止執行之後，請按 **F11** 進入 New-Catalog.ps1 指令碼。
1.  使用 [偵錯] 功能表選項以及 F10 和 F11 追蹤指令碼的執行，可以跳過或進入呼叫的函式。
    *   如需對 PowerShell 指令碼進行偵錯的詳細資訊，請參閱[使用 PowerShell 指令碼及對其進行偵錯的祕訣](https://msdn.microsoft.com/powershell/scripting/core-powershell/ise/how-to-debug-scripts-in-windows-powershell-ise) \(英文\)。

指令碼完成之後，目錄隨即會存在，而且所有範例租用戶將會完成註冊。 

現在，請查看您建立的資源。

1. 開啟 [Azure 入口網站](https://portal.azure.com/)，然後瀏覽至資源群組。  開啟 **wingtip-sa-catalog-\<user\>** 資源群組，並記下目錄伺服器和資料庫。
1. 在入口網站中開啟資料庫，再從左側功能表選取 [資料總管]。  按一下 [登入] 命令，然後輸入密碼 = **P@ssword1**。


1. 探索 *tenantcatalog* 資料庫的結構描述。  
   * `__ShardManagement` 結構描述中的物件皆由彈性資料庫用戶端程式庫所提供。
   * `Tenants` 資料表和 `TenantsExtended` 檢視為加入範例中的延伸模組，目的在於示範如何擴充目錄來提供額外價值。
1. 執行查詢，`SELECT * FROM dbo.TenantsExtended`。          

   ![資料總管](media/saas-standaloneapp-provision-and-catalog/data-explorer-tenantsextended.png)

    除了使用資料總管之外，您還可以從 SQL Server Management Studio 與資料庫連線。 若要建立連線，請與伺服器 wingtip- 連線 

    
    注意，請勿直接編輯目錄中的資料，一律使用分區管理 API 進行操作。 

## <a name="provision-a-new-tenant-application"></a>佈建新租用戶應用程式

在這項工作中，您將了解如何佈建單一租用戶應用程式。 您將：  
* 為租用戶**建立新資源群組**。 
* 使用 Azure 資源管理範本將**應用程式和資料庫**佈建到新資源群組中。  該動作透過匯入 bacpac 檔案，將通用結構描述和參考資料將資料庫初始化。 
* **利用基本租用戶資訊將資料庫初始化**。 此動作包括指定場地類型，決定要在活動網站上當做背景的相片。 
* **在目錄資料庫中註冊資料庫**。 

1. 在 PowerShell ISE 中，開啟 *...\Learning Modules\ProvisionTenants\Demo-ProvisionAndCatalog.ps1*，並設定 **$Scenario = 2**。 部署租用戶目錄及註冊預先定義的租用戶

1. 將游標置於第 49 行 (顯示 `& $PSScriptRoot\New-TenantApp.ps1`) 的任意位置，然後按 **F9** 在指令碼中新增中斷點。
1. 按 **F5** 執行指令碼。 
1.  在指令碼於中斷點停止執行之後，請按 **F11** 進入 New-Catalog.ps1 指令碼。
1.  使用 [偵錯] 功能表選項以及 F10 和 F11 追蹤指令碼的執行，可以跳過或進入呼叫的函式。

租用戶佈建之後，新租用戶活動網站隨即會開啟。

   ![red maple racing](media/saas-standaloneapp-provision-and-catalog/redmapleracing.png)

接著，您可以在 Azure 入口網站中檢查新建立的資源。 

   ![red maple racing 資源](media/saas-standaloneapp-provision-and-catalog/redmapleracing-resources.png)


## <a name="to-stop-billing-delete-resource-groups"></a>若要停止計費，請刪除資源群組 ##

完成範例的探索之後，請刪除您建立的所有資源群組以停止相關計費。

## <a name="additional-resources"></a>其他資源

- 若要進一步了解多租用戶 SaaS 資料庫應用程式，請參閱[多租用戶 SaaS 應用程式的設計模式](saas-tenancy-app-design-patterns.md)。

## <a name="next-steps"></a>後續步驟

在本教學課程中，您已了解：

> [!div class="checklist"]
> * 如何部署 Wingtip Tickets SaaS Standalone 應用程式。
> * 關於組成應用程式的伺服器和資料庫。
> * 如何刪除範例資源以停止相關計費。

對於各種使用每一租用戶一個資料庫版本之 [Wingtip Tickets SaaS 應用程式](https://docs.microsoft.com/azure/sql-database/saas-dbpertenant-wingtip-app-overview)的跨租用戶案例，您可以探索如何使用目錄來提供支援。  
