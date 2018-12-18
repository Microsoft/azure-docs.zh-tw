---
title: SaaS 多租用戶 Azure 中的佈建 | Microsoft Docs
description: 了解如何在 Azure SQL Database 多租用戶 SaaS 應用程式中佈建及編目新租用戶
keywords: SQL Database Azure
services: sql-database
author: MightyPen
manager: craigg
ms.reviewer: billgib;andrela;genemi
ms.service: sql-database
ms.custom: saas apps
ms.topic: conceptual
ms.date: 04/01/2018
ms.author: billgib
ms.openlocfilehash: 42f4aff50a5e3b89ee58f59c0db87f6a174e9be2
ms.sourcegitcommit: 266fe4c2216c0420e415d733cd3abbf94994533d
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/01/2018
ms.locfileid: "34645958"
---
# <a name="provision-and-catalog-new-tenants-in-a-saas-application-using-a-sharded-multi-tenant-azure-sql-database"></a>使用分區化多租用 Azure SQL 資料庫在 SaaS 應用程式中對新的租用戶進行佈建及編目

本文涵蓋在「多租用戶分區化資料庫」模型或模式中，對新的租用戶進行佈建和編目。

本文有兩個主要部分：

- 對新租用戶進行佈建和編目的[概念討論](#goto_2_conceptual)。

- 強調說明完成佈建和編目之 PowerShell 指令碼的[教學課程](#goto_1_tutorial)。
    - 本教學課程使用 Wingtip Tickets SaaS 應用程式，並針對多租用戶分區化資料庫模式進行調整。

<a name="goto_2_conceptual"/>

## <a name="database-pattern"></a>資料庫模式

本節及後續小節將討論多租用戶分區化資料庫模式的概念。

在這個多租用戶分區化模型中，每個資料庫內的資料表結構描述都會在儲存租用戶資料的資料表主索引鍵中包含一個租用戶金鑰。 租用戶金鑰讓每個個別的資料庫能夠儲存 0、1 或多個租用戶。 使用分區化資料庫可讓應用程式系統輕鬆地支援非常大量的租用戶。 任何一個租用戶的所有資料都會儲存在一個資料庫中。 大量租用戶會分散到多個分區化資料庫。 目錄資料庫會將每個租用戶的對應儲存至其資料庫。

#### <a name="isolation-versus-lower-cost"></a>隔離與較低的成本

本身具有資料庫的租用戶可享有隔離的好處。 該租用戶可將資料庫還原到較早的日期，而不會因為影響到其他租用戶而受到限制。 資料庫效能可以微調，以單獨針對該租用戶進行最佳化，而不需與其他租用戶妥協。 問題在於隔離成本超過它與其他租用戶共用資料庫的成本。

佈建新的租用戶時，它可以與其他租用戶共用資料庫，或將它置於自己的新資料庫中。 您稍後可以改變心意，讓資料庫處於另一種狀況。

具有多租用戶和單一租用戶的資料庫可以在相同的 SaaS 應用程式中混合使用，以將每個租用戶的成本或隔離最佳化。

   ![具有租用戶目錄的多租用戶資料庫應用程式](media/saas-multitenantdb-provision-and-catalog/MultiTenantCatalog.png)

## <a name="tenant-catalog-pattern"></a>租用戶目錄模式

當您擁有兩個以上的資料庫且每個都包含至少一個租用戶時，應用程式必須有方法來探索哪一個資料庫儲存目前感興趣的租用戶。 目錄資料庫會儲存此對應。

#### <a name="tenant-key"></a>租用戶金鑰

針對每個租用戶，Wingtip 應用程式可以衍生一個唯一金鑰 (即為租用戶金鑰)。 應用程式會從網頁 URL 擷取租用戶名稱。 應用程式會對名稱進行雜湊以取得金鑰。 應用程式會使用金鑰來存取目錄。 目錄會對租用戶儲存所在之資料庫的相關資訊進行交互參照。 應用程式會使用資料庫資訊來連線。 也可以使用其他租用戶金鑰的配置。

使用目錄可在佈建之後變更租用戶資料庫的名稱或位置，而不會中斷應用程式。 在多租用戶資料庫模型中，目錄適用於在資料庫之間移動租用戶。

#### <a name="tenant-metadata-beyond-location"></a>位置以外的租用戶中繼資料

目錄也可指出租用戶是否會離線進行維護或其他動作。 此外，可以擴充目錄來儲存其他租用戶或資料庫中繼資料，例如下列項目：
- 資料庫的服務層或版本。
- 資料庫結構描述的版本。
- 租用戶名稱及其 SLA (服務等級協定)。
- 啟用應用程式管理、客戶支援或 DevOps 程序的資訊。  

目錄也可用來就分析目的啟用跨租用戶報表、結構描述管理並擷取資料。 

### <a name="elastic-database-client-library"></a>彈性資料庫用戶端程式庫 

在 Wingtip 中，目錄會在 *tenantcatalog* 資料庫中實作。 *tenantcatalog* 是使用[彈性資料庫用戶端程式庫 (EDCL)](sql-database-elastic-database-client-library.md) 的分區管理功能來建立的。 此程式庫可讓應用程式建立、管理及使用資料庫中所儲存的「分區對應」。 分區對應會使用其分區來對租用戶金鑰進行交互參照，這表示它的分區化資料庫。

在租用戶佈建期間，可從應用程式或 PowerShell 指令碼使用 EDCL 函式來建立分區對應中的項目。 稍後可使用 EDCL 函式來連線到正確的資料庫。 EDCL 會快取連線資訊，以將對目錄資料庫的流量降到最低，並加快連線處理的速度。

> [!IMPORTANT]
> 請勿透過直接存取來編輯目錄資料庫中的資料！ 不支援直接更新，因其會導致資料損毀的高風險。 請改為僅使用 EDCL API 編輯對應資料。

## <a name="tenant-provisioning-pattern"></a>租用戶佈建模式

#### <a name="checklist"></a>檢查清單

當您想要將新的租用戶佈建到現有的共用資料庫時，您必須詢問下列有關該共用資料庫的問題：
- 它是否有剩餘足夠空間可供新的租用戶使用？
- 它的資料表是否具有新租用戶所需的參考資料，或是否可以新增資料？
- 它是否具有適用於新租用戶的適當基底結構描述變體？
- 它是否位於接近新租用戶的適當地理位置？
- 它是否位於適用於新租用戶的正確服務層？

當您想要將新租用戶隔離在其自己的資料庫中時，您可以建立它來符合租用戶的規格。

佈建完成之後，您必須在目錄中註冊該租用戶。 最後，可以新增租用戶對應來參考適當的分區。

#### <a name="template-database"></a>範本資料庫

執行 SQL 指令碼、部署 bacpac 或複製範本資料庫來佈建資料庫。 Wingtip 應用程式會複製範本資料庫來建立新的租用戶資料庫。

就像任何應用程式，Wingtip 將隨著時間持續發展。 有時候，Wingtip 將需對資料庫進行變更。 變更可能包括下列項目：
- 新的或已變更的結構描述。
- 新的或已變更的參考資料。
- 例行性資料庫維護工作，以確保最佳的應用程式效能。

有了 SaaS 應用程式，這些變更需要以經過協調的方式部署到可能非常大規模的租用戶資料庫群。 變更也需要整合到佈建程序中，以供未來在租用戶資料庫中使用。 這個挑戰會在[結構描述管理教學課程](saas-tenancy-schema-management.md)中進一步探索。

#### <a name="scripts"></a>指令碼

本教學課程中的租用戶佈建指令碼支援下列這兩種案例：
- 將租用戶佈建到與其他租用戶共用的現有資料庫。
- 將租用戶佈建到它自己的資料庫中。

然後會將租用戶資料初始化，並在目錄分區對應中加以註冊。 在範例應用程式中，包含多個租用戶的資料庫會被指定通用名稱，例如 *tenants1* 或 *tenants2*。 包含單一租用戶的資料庫會被賦予該租用戶的名稱。 範例中使用的特定命名慣例並非模式中的重要部分，因為使用目錄可允許將任何名稱指派給資料庫。  

<a name="goto_1_tutorial"/>

## <a name="tutorial-begins"></a>教學課程開始

在本教學課程中，您了解如何：

> [!div class="checklist"]
> * 將租用戶佈建到多租用戶資料庫
> * 將租用戶佈建到單一租用戶資料庫
> * 將一批租用戶佈建到多租用戶和單一租用戶資料庫
> * 在目錄中註冊資料庫和租用戶對應

#### <a name="prerequisites"></a>先決條件

若要完成本教學課程，請確定已完成下列必要條件：

- 已安裝 Azure PowerShell。 如需詳細資料，請參閱[開始使用 Azure PowerShell](https://docs.microsoft.com/powershell/azure/get-started-azureps)

- 已部署 Wingtip Tickets SaaS 多租用戶資料庫應用程式。 若要在五分鐘內完成部署，請參閱[部署及探索 Wingtip Tickets SaaS 多租用戶資料庫應用程式](saas-multitenantdb-get-started-deploy.md)

- 取得 Wingtip 指令碼和原始程式碼：
    - 可在 [WingtipTicketsSaaS MultitenantDB](https://github.com/microsoft/WingtipTicketsSaaS-MultiTenantDB) GitHub 存放庫中使用 Wingtip Tickets SaaS 多租用戶資料庫指令碼和應用程式來源程式碼。
    - 如需下載 Wingtip 指令碼以及將該指令碼解除封鎖的步驟，請參閱[一般指導方針](saas-tenancy-wingtip-app-guidance-tips.md)。 

## <a name="provision-a-tenant-into-a-database-shared-with-other-tenants"></a>在與其他租用戶「共用」的資料庫中佈建租用戶

在本節中，您會看到一份由 PowerShell 指令碼用於佈建的主要動作清單。 接著，您可以使用 PowerShell ISE 偵錯工具逐步執行指令碼，以查看程式碼中的動作。

#### <a name="major-actions-of-provisioning"></a>佈建的主要動作

以下是您逐步執行佈建工作流程的重要元素：

- **計算新的租用戶金鑰**：使用雜湊函式，從租用戶名稱建立租用戶金鑰。
- **檢查租用戶金鑰是否已經存在**：檢查目錄以確保尚未註冊金鑰。
- **初始化預設租用戶資料庫中的租用戶**：更新租用戶資料庫，以新增新的租用戶資訊。  
- **在目錄中註冊租用戶**：將新租用戶金鑰與現有 tenants1 資料庫之間的對應新增至目錄。 
- **將租用戶的名稱新增至目錄延伸模組資料表**：將場地名稱新增至目錄中的租用戶資料表。  這個新增項目說明如何擴充目錄資料庫，以支援其他應用程式特定的資料。
- **開啟新租用戶的事件頁面**：*Bushwillow Blues* 事件頁面隨即在瀏覽器中開啟。

   ![活動](media/saas-multitenantdb-provision-and-catalog/bushwillow.png)

#### <a name="debugger-steps"></a>偵錯工具步驟

若要了解 Wingtip 應用程式如何在共用資料庫中實作新的租用戶佈建，請新增中斷點並逐步執行工作流程：

1. 在 *PowerShell ISE* 中，開啟 ...\\Learning Modules\\ProvisionTenants\\*Demo-ProvisionTenants.ps1* 並設定下列參數：
   - **$TenantName** = **Bushwillow Blues**，新場地的名稱。
   - **$VenueType** = **blues**，其中一個預先定義的場地類型：blues、classicalmusic、dance、jazz、judo、motorracing、multipurpose、opera、rockmusic、soccer (小寫、不含空格)。
   - **$DemoScenario** = **1**，在與其他租用戶共用的資料庫中佈建租用戶。

2. 將游標置於第 38 行 (該行顯示︰*New-Tenant `*) 上的任意位置來新增中斷點，然後按 **F9**。

   ![中斷點](media/saas-multitenantdb-provision-and-catalog/breakpoint.png)

3. 按 **F5** 執行指令碼。

4. 在指令碼於中斷點停止執行之後，請按 **F11** 進入程式碼。

   ![debug](media/saas-multitenantdb-provision-and-catalog/debug.png)

5. 使用 [偵錯] 功能表選項、**F10** 和 **F11** 追蹤指令碼的執行，可以跳過或進入呼叫的函式。

如需對 PowerShell 指令碼進行偵錯的詳細資訊，請參閱[使用 PowerShell 指令碼及對其進行偵錯的祕訣](https://msdn.microsoft.com/powershell/scripting/core-powershell/ise/how-to-debug-scripts-in-windows-powershell-ise) \(英文\)。

## <a name="provision-a-tenant-in-its-own-database"></a>在它「自己」的資料庫中佈建租用戶

#### <a name="major-actions-of-provisioning"></a>佈建的主要動作

以下是您在追蹤指令碼時，逐步執行工作流程的重要元素：

- **計算新的租用戶金鑰**：使用雜湊函式，從租用戶名稱建立租用戶金鑰。
- **檢查租用戶金鑰是否已經存在**：檢查目錄以確保尚未註冊金鑰。
- **建立新的租用戶資料庫**：使用 Resource Manager 範本複製 *basetenantdb* 資料庫以建立資料庫。  新的資料庫名稱是以租用戶的名稱作為基礎。
- **新增要編目的資料庫**：新的租用戶資料庫會在目錄中註冊為分區。
- **初始化預設租用戶資料庫中的租用戶**：更新租用戶資料庫，以新增新的租用戶資訊。  
- **在目錄中註冊租用戶**：將新租用戶金鑰與 *sequoiasoccer* 資料庫之間的對應新增至目錄。
- **將租用戶名稱新增至目錄**：將場地名稱新增至目錄中的租用戶延伸模組資料表。
- **開啟新租用戶的事件頁面**：*Sequoia Soccer* 事件頁面隨即在瀏覽器中開啟。

   ![活動](media/saas-multitenantdb-provision-and-catalog/sequoiasoccer.png)

#### <a name="debugger-steps"></a>偵錯工具步驟

現在將進行在它自己資料庫中建立租用戶的指令碼程序逐步解說：

1. 仍在 ...\\Learning Modules\\ProvisionTenants\\*Demo-ProvisionTenants.ps1* 中，設定下列參數：
   - **$TenantName** = **Sequoia Soccer**，新場地的名稱。
   - **$VenueType** = **soccer**，其中一個預先定義的場地類型：blues、classicalmusic、dance、jazz、judo、motorracing、multipurpose、opera、rockmusic、soccer (小寫、不含空格)。
   - **$DemoScenario** = **2**，在它自己的資料庫中佈建租用戶。

2. 將游標置於以第 57 行 (該行顯示︰&&nbsp;$PSScriptRoot\New-TenantAndDatabase `) 上的任意位置來新增中斷點，然後按 **F9**。

   ![中斷點](media/saas-multitenantdb-provision-and-catalog/breakpoint2.png)

3. 按 **F5** 執行指令碼。

4. 在指令碼於中斷點停止執行之後，請按 **F11** 進入程式碼。  使用 **F10** 和 **F11** 可以跳過及進入函式追蹤執行。

## <a name="provision-a-batch-of-tenants"></a>佈建一批租用戶

此練習會佈建一批 17 個租用戶。 建議您在開始其他 Wingtip Tickets 教學課程之前佈建這一批租用戶，才會有較多資料庫可以使用。

1. 在 PowerShell ISE 中，開啟 ...\\Learning Modules\\ProvisionTenants\\*Demo-ProvisionTenants.ps1*，並將 $DemoScenario 參數變更為 4：
   - **$DemoScenario** = **4**，在共用資料庫中佈建一批租用戶。

2. 按 **F5** 並執行指令碼。

### <a name="verify-the-deployed-set-of-tenants"></a>驗證租用戶的已部署集合 

在這個階段中，您要將混合的租用戶部署到共用資料庫，並將租用戶部署到其自己的資料庫。 Azure 入口網站可以用來檢查所建立的資料庫。 在 [Azure 入口網站](https://portal.azure.com)中，瀏覽至 SQL 伺服器的清單以開啟 **tenants1-mt-\<USER\>** 伺服器。  **SQL 資料庫**清單應該包括共用的 **tenants1** 資料庫，以及位於其自己資料庫中的租用戶之資料庫：

   ![資料庫清單](media/saas-multitenantdb-provision-and-catalog/Databases.png)

儘管 Azure 入口網站會顯示租用戶資料庫，但它不會讓您查看共用資料庫內的租用戶。 Wingtip 的 [事件中樞] 網頁中以及瀏覽目錄都可看到租用戶的完整清單。

#### <a name="using-wingtip-tickets-events-hub-page"></a>使用 Wingtip Tickets 事件中樞頁面

在瀏覽器中開啟 [事件中樞] 頁面 (http:events.wingtip-mt.\<USER\>.trafficmanager.net)  

#### <a name="using-catalog-database"></a>使用目錄資料庫

目錄中會提供租用戶和每個租用戶之對應資料庫的完整清單。 目前提供一個 SQL 檢視來將租用戶名稱聯結至資料庫名稱。 此檢視會妥善示範如何擴充目錄中所儲存之中繼資料的值。
- 您可以在 tenantcatalog 資料庫中找到此 SQL 檢視。
- 租用戶名稱會儲存在租用戶資料表中。
- 資料庫名稱會儲存在分區管理資料表中。

1. 在 SQL Server Management Studio (SSMS) 中，使用登入 = **developer** 和密碼 = **P@ssword1** 連線到 **catalog-mt.\<USER\>.database.windows.net** 上的租用戶伺服器

    ![SSMS 連線對話方塊](media/saas-multitenantdb-provision-and-catalog/SSMSConnection.png)

2. 在 [SSMS 物件總管] 中，瀏覽至 *tenantcatalog* 資料庫中的檢視。

3. 以滑鼠右鍵按一下 *TenantsExtended* 檢視並選擇 [選取前 1000 個資料列]。 請注意租用戶名稱與不同租用戶的資料庫之間的對應。

    ![SSMS 中的 ExtendedTenants 檢視](media/saas-multitenantdb-provision-and-catalog/extendedtenantsview.png)
      
## <a name="other-provisioning-patterns"></a>其他佈建模式

本節將討論其他有趣的佈建模式。

#### <a name="pre-provisioning-databases-in-elastic-pools"></a>彈性集區中的預先佈建資料庫

預先佈建模式會利用在使用彈性集區時，針對集區而非資料庫計費這個事實。 因此，可以在需要資料庫而未產生額外成本之前，先將資料庫新增至彈性集區。 這個預先佈建可大幅減少將租用戶佈建至資料庫所花費的時間。 可以視需要調整預先佈建的資料庫數目，以針對預期的佈建率保留適當的緩衝。

#### <a name="auto-provisioning"></a>自動佈建

在自動佈建模式中，系統會使用專用的佈建服務，視需要自動佈建伺服器、集區與資料庫。 此自動化包括在彈性集區中預先佈建資料庫。 此外，如果已將資料庫解除委任並刪除，則可視需要由佈建服務填入其在彈性集區中建立的間隔。

這種類型的自動化服務可以很簡單，也可以很複雜。 例如，自動化可以處理跨多個地理位置的佈建，而且可針對災害復原設定異地複寫。 若使用自動佈建模式，用戶端應用程式或指令碼會將佈建要求提交到由佈建服務處理的佇列。 指令碼接著會輪詢以偵測是否完成。 如果使用預先佈建，就會快速處理要求，同時背景服務會管理替代資料庫的佈建。

## <a name="additional-resources"></a>其他資源

<!-- - Additional [tutorials that build upon the Wingtip SaaS application](saas-dbpertenant-wingtip-app-overview.md#sql-database-wingtip-saas-tutorials)-->
- [彈性資料庫用戶端程式庫](sql-database-elastic-database-client-library.md)
- [如何在 Windows PowerShell ISE 中針對指令碼進行偵錯](https://msdn.microsoft.com/powershell/scripting/core-powershell/ise/how-to-debug-scripts-in-windows-powershell-ise)


## <a name="next-steps"></a>後續步驟

在本教學課程中，您已了解如何：

> [!div class="checklist"]
> * 將單一新租用戶佈建到共用的多租用戶資料庫和其自己的資料庫
> * 佈建一批額外的租用戶
> * 逐步了解佈建租用戶和將它們註冊到目錄的細節

試用[效能監視教學課程](saas-multitenantdb-performance-monitoring.md)。

