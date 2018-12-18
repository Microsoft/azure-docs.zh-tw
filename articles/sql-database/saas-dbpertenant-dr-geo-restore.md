---
title: 使用 Azure SQL Database 異地備援備份進行 SaaS 應用程式的災害復原 | Microsoft Docs
description: 了解在發生中斷的狀況時，如何使用 Azure SQL Database 異地備援備份來復原多租用戶 SaaS 應用程式
services: sql-database
ms.service: sql-database
ms.subservice: scenario
ms.custom: ''
ms.devlang: ''
ms.topic: conceptual
author: AyoOlubeko
ms.author: ayolubek
ms.reviewer: sstein
manager: craigg
ms.date: 04/16/2018
ms.openlocfilehash: adb3778c723f7c56b5f31381c804e39a8b782709
ms.sourcegitcommit: 715813af8cde40407bd3332dd922a918de46a91a
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "47056184"
---
# <a name="use-geo-restore-to-recover-a-multitenant-saas-application-from-database-backups"></a>使用異地還原從資料庫備份復原多租用戶 SaaS 應用程式

本教學課程會針對使用每一租用戶一個資料庫模型實作的多租用戶 SaaS 應用程式，探索其完整的災害復原案例。 您可使用[異地還原](https://docs.microsoft.com/azure/sql-database/sql-database-recovery-using-backups)，將目錄和租用戶資料庫從自動維護的異地備援備份復原到替代的復原區域。 在中斷問題解決之後，您可使用[地理複寫](https://docs.microsoft.com/azure/sql-database/sql-database-geo-replication-overview)將變更的資料庫回復到其原始區域。

![Geo-restore-architecture](media/saas-dbpertenant-dr-geo-restore/geo-restore-architecture.png)

異地還原是 Azure SQL Database 成本最低的災害復原解決方案。 不過，從異地備援備份還原可能會導致資料遺失最多一小時。 這可能需要相當長的時間，端視每個資料庫的大小而定。 

> [!NOTE]
> 使用異地複寫 (而不是異地還原)，盡可能以最低的 RPO 和 RTO 復原應用程式。

本教學課程會探索還原和回復工作流程。 您會了解如何：
> [!div class="checklist"]

>* 將資料庫和彈性集區組態資訊同步至租用戶目錄中。
>* 在復原區域中設定包含應用程式、伺服器和集區的鏡像映像環境。   
>* 使用異地還原來復原目錄和租用戶資料庫。
>* 在中斷問題解決之後，使用異地複寫來回復租用戶目錄和變更的租用戶資料庫。
>* 在每個資料庫還原 (或回復) 時更新目錄，以追蹤每個租用戶資料庫的使用中複本的目前位置。
>* 確定應用程式和租用戶資料庫一律共置於相同的 Azure 區域中，以降低延遲。 
 

在開始本教學課程之前，請完成下列先決條件：
* 部署每個租用戶應用程式的 Wingtip Tickets SaaS 資料庫。 若要在五分鐘內完成部署，請參閱[部署及探索每個租用戶應用程式的 Wingtip Tickets SaaS 應用程式](saas-dbpertenant-get-started-deploy.md)。 
* 安裝 Azure PowerShell。 如需詳細資料，請參閱[開始使用 Azure PowerShell](https://docs.microsoft.com/powershell/azure/get-started-azureps)。

## <a name="introduction-to-the-geo-restore-recovery-pattern"></a>地理還原復原模式簡介

災害復原 (DR) 對許多應用程式而言都是重要的考量，無論是基於合規性因素，還是業務持續性。 如果有長時間的服務中斷，妥善的 DR 計畫將盡可能縮短運作中斷的時間。 根據地理還原的 DR 計畫必須完成數個目標：
 * 盡快保留所選復原區域中所有所需的容量，以便確保可用於還原租用戶資料庫。
 * 建立反映原始集區和資料庫組態的鏡像映像修復環境。 
 * 如果原始區域重新上限，則允許中途取消還原程序。
 * 快速啟用租用戶佈建，以便盡快重新啟動新的租用戶上線。
 * 以最佳化方式依優先順序還原租用戶。
 * 在可行的情況下平行執行步驟，以便儘速進行最佳化讓租用戶上線。
 * 有彈性地處理失敗、可重新啟動且具有等冪性。
 * 在中斷問題解決之後，將資料庫回復至其原始區域，對租用戶的影響最小。  

> [!NOTE]
> 應用程式會復原到部署應用程式所在區域的配對的區域中。 如需詳細資訊，請參閱 [Azure 配對區域](https://docs.microsoft.com/azure/best-practices-availability-paired-regions)。   

本教學課程會使用 Azure SQL Database 和 Azure 平台的功能來因應下列挑戰：

* [Azure Resource Manager 範本](https://docs.microsoft.com/azure/azure-resource-manager/resource-manager-create-first-template)，以盡快保留所有所需的容量。 Azure Resource Manager 範本是用來在復原區域中佈建原始伺服器和彈性集區的鏡像映像。 也會針對佈建新租用戶，建立不同的伺服器和集區。
* [彈性資料庫用戶端程式庫](https://docs.microsoft.com/azure/sql-database/sql-database-elastic-database-client-library) (EDCL) 可建立及維護租用戶資料庫目錄。 擴充的目錄包含定期重新整理的集區和資料庫組態資訊。
* EDCL 的[分區管理復原功能](https://docs.microsoft.com/azure/sql-database/sql-database-elastic-database-recovery-manager)可在復原與活負過程中維護目錄內的資料庫位置項目。  
* [異地還原](https://docs.microsoft.com/azure/sql-database/sql-database-disaster-recovery)可從自動維護的異地備援備份復原目錄和租用戶資料庫。 
* 以租用戶優先順序發送的[非同步還原作業](https://docs.microsoft.com/azure/azure-resource-manager/resource-manager-async-operations)，這些作業依照系統針對每個集區住列，並已批次方式處理，因此集區不會多載。 在執行之前或執行期間，如有必要，可以取消這些作業。   
* [異地複寫](https://docs.microsoft.com/azure/sql-database/sql-database-geo-replication-overview)可在中斷之後將資料庫回復至原始區域。 當您使用異地複寫時，不會遺失任何資料且對租用戶的影響最小。
* [SQL Server DNS 別名](https://docs.microsoft.com/azure/sql-database/dns-alias-overview)可讓目錄同步程序連線至使用中的目錄，無論目錄位於何處。  

## <a name="get-the-disaster-recovery-scripts"></a>取得災害復原指令碼

在[每一租用戶一個資料庫的 Wingtip Tickets SaaS GitHub 存放庫](https://github.com/Microsoft/WingtipTicketsSaaS-DbPerTenant)中，可取得在本教學課程中使用的 DR 指令碼。 請參閱[一般指引](saas-tenancy-wingtip-app-guidance-tips.md)，了解關於下載和解除封鎖 Wingtip Tickets 管理指令碼的步驟。

> [!IMPORTANT]
> 如同所有的 Wingtip Tickets 管理令碼，DR 指令碼同樣也屬於樣本品質，而不應在生產環境中使用。

## <a name="review-the-healthy-state-of-the-application"></a>檢視應用程式的健康狀態
在開始進行復原程序之前，請先查看應用程式的正常健康狀態。

1. 在您的網頁瀏覽器中，開啟 Wingtip Tickets 事件中樞 (http://events.wingtip-dpt.&lt;user&gt;.trafficmanager.net，將 &lt;user&gt; 取代為部署的使用者值)。
    
   捲動至頁面底部，並查看頁尾處的目錄伺服器名稱和位置。 此位置是您部署應用程式的區域。    

   > [!TIP]
   > 將滑鼠停留在位置上，可放大顯示。

   ![原始區域中的事件中樞健康狀態](media/saas-dbpertenant-dr-geo-restore/events-hub-original-region.png)

2. 選取 Contoso Concert Hall 租用戶，並開啟其事件頁面。

   在頁尾中，查看租用戶伺服器名稱。 此位置會與目錄伺服器的位置相同。

   ![Contoso Concert Hall 原始區域](media/saas-dbpertenant-dr-geo-restore/contoso-original-location.png) 

3. 在 [Azure 入口網站](https://portal.azure.com)中，檢閱及開啟您部署應用程式的資源群組。

   請留意部署應用程式服務元件和 SQL Database 伺服器的資源與區域。

## <a name="sync-the-tenant-configuration-into-the-catalog"></a>將租用戶組態同步至目錄中

在此工作中，您會開始進行將伺服器、彈性集區和資料庫的組態同步至租用戶目錄中的程序。 此資訊會在稍後用來在復原區域中設定鏡像映像環境。

> [!IMPORTANT]
> 為了簡單起見，在這些範例中，同步過程和其他長時間執行的復原與回復過程，會實作為本機 PowerShell 作業或在您用戶端使用者登入下執行的工作階段。 在您登入時核發的驗證權杖會在數小時後到期，屆時作業即會失敗。 在生產環境中，應以某種可靠、在服務主體下執行的 Azure 服務來實作長時間執行的程序。 請參閱[使用 Azure PowerShell 建立具有憑證的服務主體](https://docs.microsoft.com/azure/azure-resource-manager/resource-group-authenticate-service-principal)。 

1. 在 PowerShell ISE 中，開啟 ...\Learning Modules\UserConfig.psm1 檔案。 將第 10 和 11 行上的 `<resourcegroup>` 與 `<user>` 取代為您部署應用程式時所使用的值。 儲存檔案。

2. 在 PowerShell ISE 中，開啟 ...\Learning Modules\Business Continuity and Disaster Recovery\DR-RestoreFromBackup\Demo-RestoreFromBackup.ps1 指令碼。

    在本教學課程中，您會執行此 PowerShell 指令碼中的每個案例，因此請將此檔案保持開啟。

3. 進行下列設定：

    $DemoScenario = 1：啟動將租用戶伺服器和集區組態同步至目錄中的背景作業。

4. 若要執行同步指令碼，請選取 F5。 

    此資訊會在稍後用來確保復原會在復原區域中建立伺服器、集區和資料庫的鏡像映像。  

    ![同步程序](media/saas-dbpertenant-dr-geo-restore/sync-process.png)

讓 PowerShell 視窗持續在背景執行，並繼續進行本教學課程的其餘部分。

> [!NOTE]
> 同步程序會透過 DNS 別名連線至目錄。 此別名會在還原和回復期間進行修改，以指向使用中的目錄。 同步程序會透過在復原區域中所做的任何資料庫或集區組態變更，將目錄保持在最新狀態。 在回復期間，這些變更會套用至原始區域中對等的資源。

## <a name="geo-restore-recovery-process-overview"></a>異地還原復原程序概觀

異地還原復原程序會部署應用程式，並將資料庫從備份還原到復原區域中。

復原程序會執行下列動作：

1. 停用原始區域內 Web 應用程式的 Azure 流量管理員端點。 如果原始區域在復原過程中上線，停用此端點可防止使用者連線至處於無效狀態的應用程式。

2. 在復原區域內佈建復原目錄伺服器、異地還原目錄資料庫，以及更新 activecatalog 別名以指向已還原的目錄伺服器。 變更目錄別名可確保目錄同步程序一律同步到使用中的目錄。

3. 在還原復原目錄中的所有現有租用戶之前，將他們標示為離線，以防止存取租用戶資料庫。

4. 將應用程式的執行個體佈建在復原區域內，並將它設定為使用該區域內的已還原目錄。 為了使延遲維持在最低限度，範例應用程式已設計為一律連線至相同區域內的租用戶資料庫。

5. 佈建伺服器和彈性集區，其中會佈建新的租用戶。 建立這些資源，可確保佈建新租用戶不會干擾現有租用戶的復原。

6. 更新 new-tenant 別名以指向復原區域內新租用戶資料庫的伺服器。 變更此別名可確保任何新租用戶的資料庫都會在復原區域中佈建。
        
7. 為還原租用戶資料庫，在復原區域內佈建伺服器和彈性集區。 這些伺服器和集區是原始區域內組態的鏡像映像。 將集區佈建在最前面，可保留還原所有資料庫所需的容量。

    區域內發生中斷，可能會對配對區域內的可用資源造成顯著壓力。 如果您依賴 DR 的異地還原，則建議您立即保留資源。 如果在特定區域中還原應用程式極為重要，請考慮使用異地複寫。 

8. 啟用復原區域中的 Web 應用程式的流量管理員端點。 啟用此端點可讓應用程式佈建至新的租用戶。 在此階段中，現有的租用戶仍處於離線狀態。

9. 提交批次要求，以依照優先順序還原資料庫。 

    * 批次會進行組織，讓資料庫以平行方式在所有集區間還原。  

    * 還原要求會以非同步方式提交，如此才能快速提交它們並佇列，以便在每個集區中執行。

    * 由於還原要求是以平行方式在所有集區中處理，因此最好將重要的租用戶分散到多個集區。 

10. 監視 SQL Database 服務以判斷資料庫在何時進行還原。 還原租用戶資料庫之後，會在目錄中將它標示為上線，並且會記錄租用戶資料庫的 rowversion 總和。 

    * 租用戶資料庫在目錄中標示為「已上線」後，即可供應用程式存取。

    * 租用戶資料庫中的 rowversion 值的總和會儲存在目錄中。 此總和會作為指紋，以允許回復程序判斷資料庫是否已在復原區域中更新。       

## <a name="run-the-recovery-script"></a>執行復原指令碼

> [!IMPORTANT]
> 本教學課程會從異地備援備份還原資料庫。 雖然這些備份通常在 10 分鐘內可用，但最多可能需要一個小時。 此指令碼將暫停，直到它們可用為止。

請假設應用程式部署所在的區域發生中斷情況，並執行復原指令碼：

1. 在 PowerShell ISE 中的 ...\Learning Modules\Business Continuity and Disaster Recovery\DR-RestoreFromBackup\Demo-RestoreFromBackup.ps1 指令碼中，設定下列值：

    $DemoScenario = 2：藉著從異地備援備份還原，將應用程式復原到復原區域內。

2. 若要執行此指令碼，請選取 F5。  

    * 此指令碼會在新的 PowerShell 視窗中開啟，然後啟動一組平行執行的 PowerShell 作業。 這些作業會將伺服器、集區和資料庫還原到復原區域。

    * 復原區域是與您的應用程式部署所在的 Azure 區域相關聯的配對區域。 如需詳細資訊，請參閱 [Azure 配對區域](https://docs.microsoft.com/azure/best-practices-availability-paired-regions)。 

3. 在 PowerShell 視窗中監視復原程序的狀態。

    ![復原程序](media/saas-dbpertenant-dr-geo-restore/dr-in-progress.png)

> [!NOTE]
> 若要瀏覽復原作業的程式碼，請檢閱 ...\Learning Modules\Business Continuity and Disaster Recovery\DR-RestoreFromBackup\RecoveryJobs 資料夾中的 PowerShell 指令碼。

## <a name="review-the-application-state-during-recovery"></a>檢閱復原期間的應用程式狀態
當應用程式端點在流量管理員中停用時，應用程式無法使用。 目錄已還原，而且所有租用戶皆標示為離線。 然後會啟用復原區域內的應用程式端點，而且應用程式會重新上線。 雖然應用程式功能可用，但是租用戶仍會在事件中樞內顯示為離線，直到他們的資料庫還原為止。 請務必設計您的應用程式以處理離線的租用戶資料庫。

* 在復原目錄資料庫後，但在租用戶重新上線之前，在網頁瀏覽器中重新整理 Wingtip Tickets 事件中樞。

    * 在頁尾中，請留意目錄伺服器名稱此時具有 -recovery 尾碼，且位於復原區域中。

    * 請注意，尚未還原的租用戶會標示為離線，且無法選取。   
 
    ![復原程序](media/saas-dbpertenant-dr-geo-restore/events-hub-tenants-offline-in-recovery-region.png)    

    * 如果您在租用戶離線時直接開啟租用戶的 [事件] 頁面，該頁面會顯示「租用戶離線」通知。 例如，在 Contoso Concert Hall 處於離線狀態時，嘗試開啟 http://events.wingtip-dpt.&lt;user&gt;.trafficmanager.net/contosoconcerthall。

    ![復原程序](media/saas-dbpertenant-dr-geo-restore/dr-in-progress-offline-contosoconcerthall.png)

## <a name="provision-a-new-tenant-in-the-recovery-region"></a>在復原區域中佈建新租用戶
即使在租用戶資料庫還原之前，您也可以在復原區域中佈建新的租用戶。 在復原區域中佈建的新租用戶資料庫，會在稍後與復原的資料庫一起回復。   

1. 在 PowerShell ISE 中的 ...\Learning Modules\Business Continuity and Disaster Recovery\DR-RestoreFromBackup\Demo-RestoreFromBackup.ps1 指令碼中，設定下列屬性：

    $DemoScenario = 3：在復原區域中佈建新租用戶。

2. 若要執行此指令碼，請選取 F5。

3. 佈建完成時，會在瀏覽器中開啟 Hawthorn Hall 事件頁面。 

    請注意，Hawthorn Hall 資料庫位於復原區域中。

    ![在復原區域中佈建的 Hawthorn Hall](media/saas-dbpertenant-dr-geo-restore/hawthorn-hall-provisioned-in-recovery-region.png)

4. 在瀏覽器中重新整理 Wingtip Tickets 事件中樞頁面，以確認 Hawthorn Hall 包含在內。 

    如果您在其他租用戶還原之前即佈建 Hawthorn Hall，其他租用戶可能仍處於離線狀態。

## <a name="review-the-recovered-state-of-the-application"></a>檢閱應用程式的復原狀態

復原程序完成時，應用程式和所有租用戶均可在復原區域中正常運作。 

1. 在 PowerShell 主控台視窗中的顯示畫面指出所有租用戶皆已復原後，請重新整理事件中樞。 

    租用戶全都顯示為已上線，包括新的租用戶 Hawthorn Hall 在內。

    ![事件中樞內已復原和新的租用戶](media/saas-dbpertenant-dr-geo-restore/events-hub-with-hawthorn-hall.png)

2. 按一下 Contoso Concert Hall，並開啟其事件頁面。 

    在頁尾中，請注意資料庫位於位在復原區域中的復原伺服器上。

    ![復原區域中的 Contoso](media/saas-dbpertenant-dr-geo-restore/contoso-recovery-location.png)

3. 在 [Azure 入口網站](https://portal.azure.com)中，開啟資源群組的清單。  

    請查看您已部署的資源群組，以及具有 -recovery 尾碼的復原資源群組。 復原資源群組中包含所有在復原程序期間建立的資源，以及在中斷期間建立的新資源。 

4. 開啟復原資源群組，並請查看下列項目：

    * 目錄的復原版本和 tenants1 伺服器 (具有 -recovery 尾碼)。 這些伺服器上已還原的目錄和租用戶資料庫都具有在原始區域中使用的名稱。

    * tenants2-dpt-&lt;user&gt;-recovery SQL 伺服器。 這是在中斷期間用來佈建新租用戶的伺服器。

    * 名為 events-wingtip-dpt-&lt;recoveryregion&gt;-&lt;user&gt; 的 App Service，這是事件應用程式的復原執行個體。

    ![復原區域中的 Contoso 資源](media/saas-dbpertenant-dr-geo-restore/resources-in-recovery-region.png) 
    
5. 開啟 tenants2-dpt-&lt;user&gt;-recovery SQL 伺服器。 請留意其中包含資料庫 hawthornhall 和彈性集區 Pool1。 Hawthornhall 資料庫設定為 Pool1 彈性集區中的彈性資料庫。

## <a name="change-the-tenant-data"></a>變更租用戶資料 
在此工作中，您會更新其中一個已還原租用戶資料庫。 回復程序會將已變更的已還原資料庫複製到原始區域。 

1. 在瀏覽器中找出 Contoso Concert Hall 的事件清單，捲動事件並注意最後一個事件 Seriously Strauss。

2. 在 PowerShell ISE 中的 ...\Learning Modules\Business Continuity and Disaster Recovery\DR-RestoreFromBackup\Demo-RestoreFromBackup.ps1 指令碼中，設定下列值：

    $DemoScenario = 4：從復原區域的租用戶中刪除事件。

3. 若要執行此指令碼，請選取 F5。

4. 重新整理 Contoso Concert Hall 事件頁面 (http://events.wingtip-dpt.&lt;user&gt;.trafficmanager.net/contosoconcerthall) 並注意已缺少事件 Seriously Strauss。

在此時的教學課程中，您已復原應用程式，該應用程式現在正在復原區域中執行。 您已在復原區域中佈建一個新租用戶，而且修改其中一個已還原租用戶的資料。  

> [!NOTE]
> 範例中的其他教學課程並不是設計成與復原狀態中的應用程式一起執行。 如果要探索其他教學課程，請務必先回復應用程式。

## <a name="repatriation-process-overview"></a>回復程序概觀

在中斷問題解決之後，回復程序會將應用程式及其資料庫還原到其原始區域。

![異地還原回復](media/saas-dbpertenant-dr-geo-restore/geo-restore-repatriation.png) 

程序：

1. 停止所有進行中的還原活動，並取消任何未處理或進行中的資料庫還原要求。

2. 在原始區域內重新啟動中斷後未曾變更的租用戶資料庫。 這些資料庫包含尚未復原的資料庫，以及已復原但之後未變更的資料庫。 重新啟動的資料庫會與其租用戶上次存取時一模一樣。

3. 在原始區域內佈建新租用戶伺服器和彈性集區的鏡像映像。 完成此動作之後，新租用戶別名會更新為指向此伺服器。 更新別名會導致新的租用戶上線，並出現在原始區域，而非復原區域中。

3. 使用異地複寫，將目錄從復原區域移至原始區域。

4. 更新原始區域中的集區組態，如此才能與中斷期間在復原區域內所做的變更一致。

5. 建立必要的伺服器和集區來裝載中斷期間建立的任何新資料庫。

6. 使用地理複寫，回復在還原後已更新的租用戶資料庫，以及在中斷期間佈建的所有新租用戶資料庫。 

7. 清除還原程序期間在復原區域中建立的資源。

為了限制必須回復的租用戶資料庫數量，會立即完成步驟 1 至 3。  

如果復原區域內的目錄在中斷期間已修改，則只完成步驟 4。 如果已建立新租用戶，或者復原區域內的任何資料庫或集區組態已變更，則會更新目錄。

重要的是，步驟 7 對租用戶造成的中斷最小，而且不會遺失任何資料。 為了達成這個目標，處理程序會使用異地複寫。

在異地複寫每個資料庫之前，會刪除原始區域中對應的資料庫。 接著會異地複寫復原區域中的資料庫，在原始區域中建立次要複本。 複寫完成之後，租用戶會在目錄中標示為離線，這會中斷與復原區域中之資料庫的任何連線。 然後會容錯移轉資料庫，造成任何擱置交易在次要資料庫上處理，因此不會遺失資料。 

在容錯移轉時，資料庫角色會反轉。 原始區域中的次要資料庫會變成主要讀寫資料庫，而在復原區域中的資料庫則會變成唯讀次要資料庫。 目錄中的租用戶項目會更新，以參考原始區域中的資料庫，而且租用戶會標示為上線。 此時，資料庫的回復已完成。 

應用程式應該以重試邏輯加以撰寫，以確保它們在連線中斷時會自動重新連線。 當它們使用目錄來代理重新連線時，它們會連線到原始區域中已回復的資料庫。 雖然此斷線很短暫且通常難以察覺，但您仍可選擇在非營業時間回復資料庫。

資料庫回復之後，即可刪除復原區域中的次要資料庫。 然後，原始區域中的資料庫會再次依賴 DR 保護的異地還原。

在步驟 8 中，會刪除復原區域中的資源，包括復原伺服器和集區。

## <a name="run-the-repatriation-script"></a>執行回復指令碼
我們假設中斷的問題已解決，而要執行回復指令碼。

如果您已遵照教學課程進行，指令碼會立即重新啟動原始區域中的 Fabrikam Jazz Club 和 Dogwood Dojo，因為這兩個都沒有變更。 然後，它會回復新租用戶 Hawthorn Hall，以及 Contoso Concert Hall，因為它已修改過。 指令碼也會回復目錄，此目錄在佈建 Hawthorn Hall 時已更新過。
  
1. 在 PowerShell ISE 中的 ...\Learning Modules\Business Continuity and Disaster Recovery\DR-RestoreFromBackup\Demo-RestoreFromBackup.ps1 指令碼中，確認目錄同步程序仍在其 PowerShell 執行個體中執行。 如有必要，請藉由下列設定加以重新啟動：

    $DemoScenario = 1：開始將租用戶伺服器、集區和資料庫組態資訊同步處理至目錄中。

    若要執行此指令碼，請選取 F5。

2.  然後，若要啟動回復程序，請設定：

    $DemoScenario = 5：將應用程式回復至其原始區域。

    若要在新的 PowerShell 視窗中執行復原指令碼，請選取 F5。 回復需要幾分鐘的時間，您可以在 PowerShell 視窗中加以監視。

3. 在指令碼執行時，重新整理 [事件中樞] 頁面 (http://events.wingtip-dpt.&lt;user&gt;.trafficmanager.net)。

    留意到在這整個程序中，所有租用戶皆處於線上狀態，且可供存取。

4. 選取 Fabrikam Jazz Club 將它開啟。 如果您沒有修改此租用戶，請留意頁尾，伺服器已還原為原始伺服器。

5. 開啟或重新整理 Contoso Concert Hall 事件頁面。 請留意頁尾，資料庫仍在最初的 -recovery 伺服器上。 

6. 回復程序完成時，請重新整理 Contoso Concert Hall 事件頁面，並留意資料庫現在是在您原始區域中。

7. 再次重新整理事件中樞並開啟 Hawthorn Hall。 請注意，其資料庫也位於原始區域中。 

## <a name="clean-up-recovery-region-resources-after-repatriation"></a>回復之後清除復原區域資源
回復完成之後，可以安全無虞地刪除復原區域中的資源。 

> [!IMPORTANT]
> 立即刪除這些資源，可停止它們的所有計費。

還原程序會建立復原資源群組中的所有復原資源。 清理程序會刪除此資源群組，並從目錄移除所有對資源的參考。 

1. 在 PowerShell ISE 中的 ...\Learning Modules\Business Continuity and Disaster Recovery\DR-RestoreFromBackup\Demo-RestoreFromBackup.ps1 指令碼中，設定：
    
    $DemoScenario = 6：從復原區域刪除已過時的資源。

2. 若要執行此指令碼，請選取 F5。

清除指令碼後，應用程式會回到開始的位置。 此時，您可以再次執行指令碼，或試試其他教學課程。

## <a name="designing-the-application-to-ensure-that-the-app-and-the-database-are-co-located"></a>設計應用程式以確保應用程式和資料庫共置 
應用程式已設計為一律從與租用戶資料庫所在相同區域中的執行個體連線。 此設計可降低應用程式與資料庫之間的延遲。 此最佳化假設應用程式對資料庫的互動比使用者對應用程式的互動更為頻繁。  

在回復期間的某些時刻，租用戶資料庫可能分散於復原和原始區域間。 對於各個資料庫，應用程式會執行租用戶伺服器名稱的 DNS 查閱，以查閱資料庫所在的區域。 在 SQL Database 中，伺服器名稱會是別名。 使用別名的伺服器名稱會包含區域名稱。 應用程式若位於與資料庫不同的區域，則會重新導向至資料庫伺服器所在相同區域中的執行個體。 重新導向至資料庫所在之相同區域中的執行個體，可盡量縮短應用程式與資料庫之間的延遲。  

## <a name="next-steps"></a>後續步驟

在本教學課程中，您已了解如何：
> [!div class="checklist"]

>* 使用租用戶目錄來保存定期重新整理的組態資訊，如此可在另一個區域中建立鏡像映像復原環境。
>* 使用異地還原將 Azure SQL 資料庫復原到復原區域中。
>* 更新租用戶目錄以反映已還原租用戶資料庫位置。 
>* 使用 DNS 別名，可讓應用程式連線至整個租用戶目錄，而無需重新設定。
>* 在中斷問題解決之後，使用異地複寫將已復原的資料庫重新回復到其原始區域。

嘗試[使用資料庫異地複寫，為多租用戶 SaaS 應用程式進行災害復原](saas-dbpertenant-dr-geo-replication.md)教學課程，以了解如何使用異地複寫來大幅減少復原大規模的多租用戶應用程式所需的時間。

## <a name="additional-resources"></a>其他資源

[其他以 Wingtip SaaS 應用程式為基礎的教學課程](https://docs.microsoft.com/azure/sql-database/sql-database-wtp-overview#sql-database-wingtip-saas-tutorials)
