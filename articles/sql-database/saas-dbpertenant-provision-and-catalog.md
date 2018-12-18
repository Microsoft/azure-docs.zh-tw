---
title: 在使用 Azure SQL Database 在多租用戶應用程式中佈建新租用戶 | Microsoft Docs
description: 了解如何在 Azure SQL Database 多租用戶 SaaS 應用程式中佈建新租用戶並將它們編入目錄中
keywords: SQL Database Azure
services: sql-database
author: stevestein
manager: craigg
ms.service: sql-database
ms.custom: scale out apps
ms.topic: conceptual
ms.date: 04/01/2018
ms.author: sstein
ms.openlocfilehash: 7c526f65be5e4a3ea50de4603441e6184abf8edd
ms.sourcegitcommit: 266fe4c2216c0420e415d733cd3abbf94994533d
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/01/2018
ms.locfileid: "34643612"
---
# <a name="learn-how-to-provision-new-tenants-and-register-them-in-the-catalog"></a>了解如何佈建新的租用戶並在目錄中註冊它們

在本教學課程中，您會了解如何佈建 SaaS 模式並將它們編入目錄中。 您也會了解在每一租用戶一個資料庫的 Wingtip Tickets SaaS 應用程式中實作它們的方式。 您會建立新的租用戶資料庫並將這些資料庫初始化，然後在應用程式的租用戶目錄中註冊它們。 「目錄」是維護許多 SaaS 應用程式租用戶及其資料間之對應的資料庫。 目錄扮演將應用程式和管理要求導向正確資料庫的重要角色。

在本教學課程中，您了解如何：

> [!div class="checklist"]

> * 佈建單一新租用戶。
> * 佈建一批額外的租用戶。


若要完成本教學課程，請確定已完成下列必要條件：

* 已部署每一租用戶一個資料庫的 Wingtip Tickets SaaS 應用程式。 若要在五分鐘內完成部署，請參閱[部署及探索每一租用戶一個資料庫的 Wingtip Tickets SaaS 應用程式](saas-dbpertenant-get-started-deploy.md)。
* 已安裝 Azure PowerShell。 如需詳細資訊，請參閱[開始使用 Azure PowerShell](https://docs.microsoft.com/powershell/azure/get-started-azureps)。

## <a name="introduction-to-the-saas-catalog-pattern"></a>SaaS 目錄模式簡介

在以資料庫為後盾的多租用戶 SaaS 應用程式中，了解每個租用戶的資訊儲存位置相當重要。 在 SaaS 目錄模式中，會使用目錄資料庫來保存每個租用戶與其資料儲存所在資料庫之間的對應。 此模式適用於租用戶資料分散在多個資料庫時。

每個租用戶都會藉由目錄中的索引鍵識別，該目錄會對應至其資料庫的位置。 在 Wingtip Tickets 應用程式中，金鑰是由租用戶名稱的雜湊組成。 此配置可讓應用程式從應用程式 URL 中所含的租用戶名稱建構金鑰。 您也可以使用其他租用戶金鑰配置。  

目錄可讓資料庫的名稱或位置在對應用程式影響最低的情況下進行變更。 在多租用戶資料庫模型中，此功能也有助於在資料庫之間移動租用戶。 目錄也可用來指出租用戶或資料庫是否離線進行維護或其他動作。 在[還原單一租用戶教學課程](saas-dbpertenant-restore-single-tenant.md)中會探索此功能。

目錄也可以儲存額外的租用戶或資料庫中繼資料，例如結構描述版本、服務方案或提供給租用戶的 SLA。 目錄可以儲存可啟用應用程式管理、客戶支援或 DevOps 的其他資訊。 

SaaS 應用程式以外，目錄可以啟用資料庫工具。 在每一租用戶一個資料庫的 Wingtip Tickets SaaS 範例中，會使用目錄來啟用跨租用戶的查詢，在[隨選報表教學課程](saas-tenancy-cross-tenant-reporting.md)中會探索此功能。 在[結構描述管理](saas-tenancy-schema-management.md)和[租用戶分析](saas-tenancy-tenant-analytics.md)教學課程中則會探索跨資料庫作業管理。 

在 Wingtip Tickets SaaS 範例中，會使用[彈性資料庫用戶端程式庫 (EDCL)](sql-database-elastic-database-client-library.md) 的「分區管理」功能來實作目錄。 EDCL 可供在 Java 和 .Net Framework 中使用。 EDCL 可讓應用程式建立、管理及使用資料庫為基礎的分區對應。 

分區對應包含分區 (資料庫) 清單，以及金鑰 (租用戶) 與分區之間的對應。 在租用戶佈建期間，會使用 EDCL 函式來建立分區對應中的項目。 在執行階段，應用程式會使用它們來連線到正確的資料庫。 EDCL 會快取連線資訊，以將對目錄資料庫的流量降到最低，並加快應用程式的速度。 

> [!IMPORTANT]
> 您可以在類別目錄資料庫中存取對應資料，但「請勿編輯它」。 請只使用「彈性資料庫用戶端程式庫 API」來編輯對應資料。 直接操作對應資料會有造成目錄損毀的風險，而且也不支援這樣做。


## <a name="introduction-to-the-saas-provisioning-pattern"></a>SaaS 佈建模式簡介

當您在使用單一租用戶資料庫模型的 SaaS 應用程式中新增租用戶時，必須佈建新的租用戶資料庫。 必須在適當的位置和服務層建立資料庫。 此外，也必須以適當的結構描述和參考資料將其初始化。 必須在目錄中適當的租用戶金鑰下註冊此資料庫。 

您可以使用不同的資料庫佈建方法。 您可以執行 SQL 指令碼、部署 bacpac 或複製範本資料庫。 

資料庫佈建必須是您結構描述管理策略的一部分。 您必須確定為新資料庫佈建最新的結構描述。 在[結構描述管理接學課程](saas-tenancy-schema-management.md)中會探索此需求。 

每一租用戶一個資料庫的 Wingtip Tickets 應用程式會藉由複製目錄伺服器上所部署、名為 _basetenantdb_ 的範本資料庫，佈建新的租用戶。 您可以將佈建整合到應用程式作為註冊體驗的一部分。 此外，也可以藉由使用指令碼以離線方式支援。 本教學課程會使用 PowerShell 來探索佈建。 

佈建指令碼會複製 _basetenantdb_ 資料庫，以在彈性集區中建立新的租用戶資料庫。 租用戶資料庫會建立在與 _newtenant_ DNS 別名對應的租用戶伺服器中。 此別名會持續為用來佈建新租用戶的伺服器提供參考，並更新為指向災害復原教學課程 ([使用異地還原的 DR](saas-dbpertenant-dr-geo-restore.md)、[使用異地複寫的 DR](saas-dbpertenant-dr-geo-replication.md)) 中的復原租用戶伺服器。 接著，指令碼會以租用戶特定資訊將資料庫初始化，然後在目錄分區對應中註冊該資料庫。 租用戶資料庫會根據租用戶名稱來命名。 此命名配置不是此模式的重要部分。 目錄會將租用戶金鑰與資料庫名稱對應，因此可以使用任何命名慣例。 


## <a name="get-the-wingtip-tickets-saas-database-per-tenant-application-scripts"></a>取得每一租用戶一個資料庫的 Wingtip Tickets SaaS 應用程式指令碼

在 [WingtipTicketsSaaS-DbPerTenant](https://github.com/Microsoft/WingtipTicketsSaaS-DbPerTenant) GitHub 存放庫可取得 Wingtip Tickets SaaS 指令碼和應用程式原始程式碼。 關於下載和解除封鎖 Wingtip Tickets SaaS 指令碼的步驟，請參閱[一般指引](saas-tenancy-wingtip-app-guidance-tips.md)。


## <a name="provision-and-catalog-detailed-walkthrough"></a>佈建及編目的詳細逐步解說

若要了解 Wingtip Tickets 應用程式如何實作新的租用戶佈建，請在佈建租用戶時，新增中斷點並依循工作流程進行操作。

1. 在 PowerShell ISE 中，開啟 ...\\Learning Modules\\ProvisionAndCatalog\\_Demo-ProvisionAndCatalog.ps1_，然後設定下列參數：

   * **$TenantName** = 新場地的名稱 (例如，*Bushwillow Blues*)。
   * **$VenueType** = 其中一個預先定義的場地類型：_blues、classicalmusic、dance、jazz、judo、motor racing、multipurpose、opera、rockmusic、soccer_。
   * **$DemoScenario** = **1**，佈建單一租用戶。

2. 若要新增中斷點，請將游標置於顯示 *New-Tenant `* 之行上的任意位置。 然後按 F9。

   ![中斷點](media/saas-dbpertenant-provision-and-catalog/breakpoint.png)

3. 若要執行指令碼，請按 F5。

4. 在指令碼於中斷點停止執行之後，按 F11 來逐步執行程式碼。

   ![Debugging](media/saas-dbpertenant-provision-and-catalog/debug.png)



請使用 [偵錯] 功能表選項來追蹤指令碼的執行情況。 按 F10 和 F11 來不進入或逐步執行所呼叫的函式。 如需對 PowerShell 指令碼進行偵錯的詳細資訊，請參閱[使用 PowerShell 指令碼及對其進行偵錯的祕訣](https://msdn.microsoft.com/powershell/scripting/core-powershell/ise/how-to-debug-scripts-in-windows-powershell-ise) \(英文\)。


您不一定要明確依照此工作流程進行操作。 它說明如何對指令碼進行偵錯。

* **匯入 CatalogAndDatabaseManagement.psm1 模組。** 它提供對[分區管理](sql-database-elastic-scale-shard-map-management.md)函式的目錄和租用戶層級抽象概念。 此模組包含大部分的編目模式且值得探討。
* **匯入 SubscriptionManagement.psm1 模組。** 它包含用於登入 Azure 及選取您想要使用之 Azure 訂用帳戶的函式。
* **取得設定詳細資料。** 使用 F11 來逐步執行 Get-Configuration，然後查看指定應用程式設定的方式。 資源名稱及其他應用程式特定值都會在此處定義。 在您熟悉指令碼之前，請勿變更這些值。
* **取得目錄物件。** 逐步執行 Get-Catalog，這會撰寫並傳回在較高層級指令碼中使用的目錄物件。 此函式會使用從 **AzureShardManagement.psm1** 匯入的分區管理函式。 目錄物件是由下列元素組成：

   * $catalogServerFullyQualifiedName 的建構方式是使用標準主幹再加上您的使用者名稱：_catalog-\<user\>.database.windows .net_。
   * $catalogDatabaseName 是從下列設定擷取：*tenantcatalog*。
   * $shardMapManager 物件是從類別目錄資料庫初始化。
   * $shardMap 物件是從類別目錄資料庫中的 _tenantcatalog_ 分區對應初始化。 系統會組成並傳回目錄物件。 此物件會用於較高層級的指令碼中。
* **計算新的租用戶金鑰。** 會使用雜湊函式從租用戶名稱建立租用戶索引鍵。
* **檢查租用戶金鑰是否已經存在。** 系統會檢查目錄以確定該金鑰可供使用。
* **租用戶資料庫是使用 New-TenantDatabase 來佈建。** 請使用 F11 來逐步執行，並使用 [Azure Resource Manager 範本](../azure-resource-manager/resource-manager-template-walkthrough.md)來了解資料庫的佈建方式。

    資料庫名稱是使用租用戶名稱來建構，使分區和租用戶之間的從屬關係一目瞭然 您也可以使用其他資料庫命名慣例。 Resource Manager 範本會建立租用戶資料庫，方法是在目錄伺服器上複製範本資料庫 (baseTenantDB)。 另一個方法是建立一個資料庫，然後匯入 bacpac 來將它初始化。 或者，您也可以從已知的位置執行初始化指令碼。

    Resource Manager 範本位於 …\Learning Modules\Common\ 資料夾：tenantdatabasecopytemplate.json

* **將租用戶資料庫進一步初始化。** 這會新增場地 (租用戶) 名稱和場地類型。 您也可以在此處進行其他初始化。

* **在目錄中註冊租用戶資料庫。** 這會使用租用戶金鑰以 *Add-TenantDatabaseToCatalog* 進行註冊。 請使用 F11 來逐步執行以查看詳細資料：

    * 類別目錄資料庫已新增到分區對應 (已知資料庫的清單)。
    * 已建立將索引鍵值連結到分區的對應。
    * 已將租用戶的其他相關中繼資料 (場地的名稱) 新增至目錄中的「租用戶」資料表。 「租用戶」資料表不是「分區管理」結構描述的一部分，且不是由 EDCL 安裝。 此資料表說明如何延伸目錄資料庫，以支援其他應用程式特定資料。


佈建完成之後，執行會返回原始的 *Demo-ProvisionAndCatalog* 指令碼。 瀏覽器中會開啟新租用戶的 [事件] 頁面。

   ![[事件] 頁面](media/saas-dbpertenant-provision-and-catalog/new-tenant.png)


## <a name="provision-a-batch-of-tenants"></a>佈建一批租用戶

此練習會佈建一批 17 個租用戶。 建議您在開始進行其他每一租用戶一個資料庫的 Wingtip Tickets SaaS 教學課程之前，先佈建這一批租用戶。 這樣就會不止有幾個資料庫可供使用。

1. 在 PowerShell ISE 中，開啟 ...\\Learning Modules\\ProvisionAndCatalog\\*Demo-ProvisionAndCatalog.ps1*。 將 *$DemoScenario* 參數變更為 3：

   * **$DemoScenario** = **3**，佈建一批租用戶。
2. 若要執行指令碼，請按 F5。

此指令碼會部署一批額外的租用戶。 它會使用 [Azure Resource Manager 範本](../azure-resource-manager/resource-manager-template-walkthrough.md)，此範本會控制該批次並將每個資料庫的佈建委派給所連結的範本。 以此方式使用範本可讓 Azure Resource Manager 代理指令碼的佈建程序。 這些範本會平行佈建資料庫，並視需要處理重試。 指令碼為等冪，如果它失敗，或因為任何原因而停止，請再次執行。

### <a name="verify-the-batch-of-tenants-that-successfully-deployed"></a>確認已成功部署的一批租用戶

* 在 [Azure 入口網站](https://portal.azure.com) 中，瀏覽至您的伺服器清單並開啟 tenants1 伺服器。 選取 [SQL 資料庫]，然後確認清單中現在有該批 17 個額外的資料庫。

   ![資料庫清單](media/saas-dbpertenant-provision-and-catalog/database-list.png)



## <a name="other-provisioning-patterns"></a>其他佈建模式

本教學課程中未包含的其他佈建模式：

**預先佈建資料庫**：預先佈建模式會利用彈性集區中的資料庫並不會增加額外成本的事實。 計費只會針對彈性集區，不會針對資料庫。 閒置的資料庫不會耗用任何資源。 藉由在集區中預先佈建資料庫，並在需要時才配置它們，您便可以縮減新增租用戶的時間。 可以視需要調整預先佈建的資料庫數目，以針對預期的佈建率保留適當的緩衝。

**自動佈建**：在自動佈建模式中，佈建服務會視需要自動佈建伺服器、集區及資料庫。 如果您想要，也可以包含彈性集區中的預先佈建資料庫。 如果資料庫已被解除委任並刪除，可以由佈建服務填補彈性集區中的間隔。 這類服務可以相當簡單，也可以相當複雜，例如處理跨多個地理位置的佈建，以及針對災害復原設定異地複寫。 

使用自動佈建模式時，用戶端應用程式或指令碼會將佈建要求提交給要由佈建服務處理的佇列。 接著，它會輪詢服務以判斷完成情況。 如果使用預先佈建，則會快速處理要求。 服務會在背景佈建一個替代資料庫。


## <a name="next-steps"></a>後續步驟

在本教學課程中，您已了解如何：

> [!div class="checklist"]

> * 佈建單一新租用戶。
> * 佈建一批額外的租用戶。
> * 了解佈建租用戶和將它們註冊到目錄中的細節。

試用[效能監視教學課程](saas-dbpertenant-performance-monitoring.md)。

## <a name="additional-resources"></a>其他資源

* [以每一租用戶一個資料庫的 Wingtip Tickets SaaS 應用程式為基礎的其他教學課程](saas-dbpertenant-wingtip-app-overview.md#sql-database-wingtip-saas-tutorials)
* [彈性資料庫用戶端程式庫](sql-database-elastic-database-client-library.md)
* [在 Windows PowerShell ISE 中針對指令碼進行偵錯](https://msdn.microsoft.com/powershell/scripting/core-powershell/ise/how-to-debug-scripts-in-windows-powershell-ise)
