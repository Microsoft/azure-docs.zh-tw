---
title: 使用 SQL Database 異地複寫進行 SaaS 應用程式的災害復原 | Microsoft Docs
description: 了解在發生中斷的狀況時如何使用 Azure SQL Database 異地複寫復原多租用戶 SaaS 應用程式
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
ms.date: 04/09/2018
ms.openlocfilehash: f24c76fb6b7ca24573a97aa122659fe5ca019550
ms.sourcegitcommit: 715813af8cde40407bd3332dd922a918de46a91a
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "47056330"
---
# <a name="disaster-recovery-for-a-multi-tenant-saas-application-using-database-geo-replication"></a>使用資料庫異地複寫進行多租用戶 SaaS 應用程式的災害復原

在本教學課程中，您將針對使用每一租用戶一個資料庫的模型實作的多租用戶 SaaS 應用程式，探索其完整的災害復原案例。 若要防止應用程式中斷，您應使用[_異地複寫_](https://docs.microsoft.com/azure/sql-database/sql-database-geo-replication-overview)，在替代的復原區域中建立目錄和租用戶資料庫的複本。 如果發生中斷的情況，您將可快速地容錯移轉至這些複本，以繼續進行正常的商務作業。 在容錯移轉時，原始區域中的資料庫會成為復原區域中資料庫的次要複本。 這些複本在重新上線後，將會自動更新至復原區域中資料庫的狀態。 中斷的問題解決後，您即可容錯回復至原始生產區域中的資料庫。

本教學課程會探索容錯移轉和容錯回復的工作流程。 您將學習如何：
> [!div classs="checklist"]

>* 將資料庫和彈性集區組態資訊同步至租用戶目錄中
>* 在替代區域中設定由應用程式、伺服器和集區組成的復原環境
>* 使用_異地複寫_將目錄和租用戶資料庫複寫至復原區域
>* 將應用程式以及目錄和租用戶資料庫容錯移轉至復原區域 
>* 隨後，在中斷問題解決後將應用程式、目錄和租用戶資料庫重新容錯移轉至原始區域
>* 在每個租用戶資料庫容錯移轉後更新目錄，以追蹤每個租用戶資料庫的主要位置
>* 確定應用程式和主要租用戶資料庫一律共置於相同的 Azure 區域中，以降低延遲  
 

在開始本教學課程前，請確定已符合下列必要條件：
* 已部署每一租用戶一個資料庫的 Wingtip Tickets SaaS 應用程式。 若要在五分鐘內完成部署，請參閱[部署及探索每一租用戶一個資料庫的 Wingtip Tickets SaaS 應用程式](saas-dbpertenant-get-started-deploy.md)  
* 已安裝 Azure PowerShell。 如需詳細資料，請參閱[開始使用 Azure PowerShell](https://docs.microsoft.com/powershell/azure/get-started-azureps)

## <a name="introduction-to-the-geo-replication-recovery-pattern"></a>異地複寫復原模式簡介

![復原架構](media/saas-dbpertenant-dr-geo-replication/recovery-architecture.png)
 
災害復原 (DR) 對許多應用程式而言都是重要的考量，無論是基於合規性因素，還是業務持續性。 如果預期會有長時間的服務中斷，妥善的 DR 計畫將盡可能縮短運作中斷的時間。 使用異地複寫將可在復原區域中維護能夠在短時間內容錯移轉的資料庫複本，而提供最低的 RPO 和 RTO。

以異地複寫為基礎的 DR 計畫包含三個不同的部分：
* 設定 - 建立和維護復原環境
* 復原 - 在發生中斷情況時將應用程式和資料庫容錯移轉至復原環境，
* 回復 - 在應用程式中斷問題解決後，將應用程式和資料庫重新容錯移轉至原始區域 

這三方面都必須審慎考量，尤其是在大規模運作時。 整體來說，此計畫必須完成數個目標：

* 設定
    * 在復原區域中建立及維護鏡像映像環境。 在此復原環境中建立彈性集區並複寫任何單一資料庫，可保留復原區域中的容量。 維護此環境的工作，包括在新的租用戶資料庫佈建時加以複寫。  
* 復原
    * 使用相應減少的復原環境來降低日常成本時，必須相應增加集區與單一資料庫，以達到復原區域中的完整運作容量
    * 讓新的租用戶盡快佈建於復原區域中  
    * 以最佳化方式依優先順序還原租用戶
    * 在可行的情況下平行執行步驟，以最佳化方式儘速將租用戶上線
    * 有彈性地處理失敗、可重新啟動且具有等冪性
    * 在原始區域重新上線時，能夠中途取消還原程序。
* 回復 
    * 在租用戶受到最小影響的情況下，將資料庫從復原區域容錯移轉至原始區域：不會遺失資料，並盡可能縮短每個租用戶的離線期間。   

在本教學課程中，會使用 Azure SQL Database 和 Azure 平台的功能來因應下列挑戰：

* [Azure Resource Manager 範本](https://docs.microsoft.com/azure/azure-resource-manager/resource-manager-create-first-template)，以盡快保留所有所需的容量。 Azure Resource Manager 範本可在復原區域中用來佈建生產伺服器和彈性集區的鏡像映像。
* [異地複寫](https://docs.microsoft.com/azure/sql-database/sql-database-geo-replication-overview)，以透過非同步方式為所有資料庫建立複寫的唯讀次要複本。 在中斷期間，您可以容錯移轉至復原區域中的複本。  中斷的問題解決後，您即可容錯回復至原始區域中的資料庫，且不會遺失資料。
* 依租用戶優先順序傳送的[非同步](https://docs.microsoft.com/azure/azure-resource-manager/resource-manager-async-operations)容錯移轉作業，以盡可能縮短大量資料庫進行容錯移轉的時間。
* [分區管理復原功能](https://docs.microsoft.com/azure/sql-database/sql-database-elastic-database-recovery-manager)，以在復原和回復期間變更目錄中的資料庫項目。 這些功能可讓應用程式無需重新設定即可從任何位置連線至租用戶資料庫。
* [SQL Server DNS 別名](https://docs.microsoft.com/azure/sql-database/dns-alias-overview)，讓新的租用戶可順暢地進行佈建，無論應用程式在哪個區域運作皆可。 DNS 別名也可用來讓目錄同步程序連線至使用中的目錄，無論目錄位於何處。

## <a name="get-the-disaster-recovery-scripts"></a>取得災害復原指令碼 

> [!IMPORTANT]
> 如同所有的 Wingtip Tickets 管理令碼，DR 指令碼同樣也屬於樣本品質，而不應在生產環境中使用。 

在[每一租用戶一個資料庫的 Wingtip Tickets SaaS GitHub 存放庫](https://github.com/Microsoft/WingtipTicketsSaaS-DbPerTenant/)中，可取得在本教學課程中使用的復原指令碼和 Wingtip 應用程式的原始程式碼。 請參閱[一般指引](saas-tenancy-wingtip-app-guidance-tips.md)，了解關於下載和解除封鎖 Wingtip Tickets 管理指令碼的步驟。

## <a name="tutorial-overview"></a>教學課程概觀
在本教學課程中，您會先使用異地複寫在不同的區域建立 Wingtip Tickets 應用程式及其資料庫的複本。 然後，您會容錯移轉至此區域以模擬從中斷情況復原的程序。 完成後，應用程式會在復原區域中正常運作。

隨後，在個別的回復步驟中，您會將復原區域中的目錄和租用戶資料庫容錯移轉至原始區域。 在整個回復期間，應用程式和資料庫皆可供使用。 完成後，應用程式會在原始區域中正常運作。

> [!Note]
> 應用程式會復原到部署應用程式之區域的_配對區域_中。 如需詳細資訊，請參閱 [Azure 配對區域](https://docs.microsoft.com/azure/best-practices-availability-paired-regions)。

## <a name="review-the-healthy-state-of-the-application"></a>檢視應用程式的健康狀態

在開始進行復原程序之前，請先查看應用程式的正常健康狀態。
1. 在您的網頁瀏覽器中，開啟 Wingtip Tickets 事件中樞 (http://events.wingtip-dpt.&lt;user&gt;.trafficmanager.net，將 &lt;user&gt; 取代為部署的使用者值)。
    * 捲動至頁面底部，並查看頁尾處的目錄伺服器名稱和位置。 此位置是您部署應用程式的區域。
    *提示：將滑鼠置於此位置上方，可放大顯示畫面。*
    ![原始區域中的事件中樞健康狀態](media/saas-dbpertenant-dr-geo-replication/events-hub-original-region.png)

2. 按一下 Contoso Concert Hall 租用戶，並開啟其事件頁面。
    * 在頁尾中，查看租用戶伺服器名稱。 位置會與目錄伺服器的位置相同。

3. 在 [Azure 入口網站](https://portal.azure.com)中，開啟應用程式部署所在的資源群組
    * 查看伺服器部署所在的區域。 

## <a name="sync-tenant-configuration-into-catalog"></a>將租用戶組態同步至目錄中

在此工作中，您會開始進行將伺服器、彈性集區和資料庫的組態同步至租用戶目錄中的程序。 此程序會將目錄中的這些資訊保持在最新狀態。  此程序會處理使用中的目錄，無論目錄位於原始區域還是復原區域中。 組態資訊會作為復原程序的一部分，以確保復原環境與原始環境的一致性，並且在隨後的回復期間確保原始區域會透過在復原環境中所做的任何變更保有一致性。 此目錄也可用來追蹤租用戶資源的復原狀態

> [!IMPORTANT]
> 為了方便說明，在這些教學課程中，一律會以在您的用戶端使用者登入下執行的本機 Powershell 作業或工作階段，來實作同步程序和其他長時間執行的復原與回復程序。 在您登入時核發的驗證權杖將在數小時後到期，屆時作業即會失敗。 在生產環境中，應以某種可靠、在服務主體下執行的 Azure 服務來實作長時間執行的程序。 請參閱[使用 Azure PowerShell 建立具有憑證的服務主體](https://docs.microsoft.com/azure/azure-resource-manager/resource-group-authenticate-service-principal)。

1. 在 _PowerShell ISE_ 中，開啟 ...\Learning Modules\UserConfig.psm1 檔案。 請將第 10 和 11 行上的 `<resourcegroup>` 與 `<user>` 取代為您部署應用程式時所使用的值。  儲存檔案。

2. 在 *PowerShell ISE* 中，開啟 ...\Learning Modules\Business Continuity and Disaster Recovery\DR-FailoverToReplica\Demo-FailoverToReplica.ps1 指令碼，並設定：
    * **$DemoScenario = 1**，啟動將租用戶伺服器和集區組態同步至目錄中的背景作業

3. 按 **F5** 以執行同步指令碼。 新的 PowerShell 工作階段隨即開啟，以同步租用戶資源的組態。
![同步程序](media/saas-dbpertenant-dr-geo-replication/sync-process.png)

讓 PowerShell 視窗持續在背景執行，並繼續進行教學課程的其餘部分。 

> [!Note]
> 同步程序會透過 DNS 別名連線至目錄。 此別名會在還原和回復期間進行修改，以指向使用中的目錄。 同步程序會透過在復原區域中所做的任何資料庫或集區組態變更，將目錄保持在最新狀態。  在回復期間，這些變更會套用至原始區域中對等的資源。

## <a name="create-secondary-database-replicas-in-the-recovery-region"></a>在復原區域中建立次要資料庫複本

在此工作中，您會開始進行部署重複的應用程式執行個體，並將目錄和所有租用戶資料庫複寫至復原區域的程序。

> [!Note]
> 本教學課程會將異地複寫保護新增至 Wingtip Tickets 範例應用程式。 在應用程式使用異地複寫的生產環境中，每個租用戶一開始都會以異地複寫的資料庫進行佈建。 請參閱[使用 Azure SQL Database 設計高可用性服務](https://docs.microsoft.com/azure/sql-database/sql-database-designing-cloud-solutions-for-disaster-recovery#scenario-1-using-two-azure-regions-for-business-continuity-with-minimal-downtime)

1. 在 *PowerShell ISE* 中，開啟 ...\Learning Modules\Business Continuity and Disaster Recovery\DR-FailoverToReplica\Demo-FailoverToReplica.ps1 指令碼，並設定下列值：
    * **$DemoScenario = 2**，建立鏡像映像復原環境，並複寫目錄和租用戶資料庫

2. 按 **F5** 以執行指令碼。 新的 PowerShell 工作階段隨即開啟以建立複本。
![同步程序](media/saas-dbpertenant-dr-geo-replication/replication-process.png)  

## <a name="review-the-normal-application-state"></a>查看正常的應用程式狀態
此時，應用程式會在原始區域中正常執行，並受到異地複寫的保護。  所有資料庫的復原區域中都會有唯讀次要複本存在。 
1. 在 Azure 入口網站中查看您的資源群組，並留意復原區域中已建立具有 -recovery 尾碼的資源群組。 

1. 瀏覽復原資源群組中的資源。  

1. 按一下 _tenants1-dpt-&lt;user&gt;-recovery_ 伺服器上的 Contoso Concert Hall 資料庫。  按一下左側的 [異地複寫]。 

    ![Contoso Concert 異地複寫連結](media/saas-dbpertenant-dr-geo-replication/contoso-geo-replication.png) 

在 Azure 區域對應中，查看原始區域中的主要複本與復原區域中的次要複本之間的異地複寫連結。  

## <a name="fail-over-the-application-into-the-recovery-region"></a>將應用程式容錯移轉至復原區域中

### <a name="geo-replication-recovery-process-overview"></a>異地複寫復原程序概觀

復原指令碼會執行下列工作：

1. 停用原始區域內的 Web 應用程式的流量管理員端點。 如果原始區域在復原過程中上線，停用此端點可防止使用者連線至處於無效狀態的應用程式。

1. 在復原區域中使用目錄資料庫的容錯移轉並使其成為主要資料庫，然後更新 _activecatalog_ 別名以指向復原目錄伺服器。

1. 更新 _newtenant_ 別名，以指向復原區域中的租用戶伺服器。 變更此別名可確保任何新租用戶的資料庫都會在復原區域中佈建。 

1. 將復原目錄中的所有現有租用戶都標示為離線，以防止租用戶資料庫在容錯移轉之前受到存取。

1. 更新復原區域中所有彈性集區與已複寫的單一資料庫的設定，以鏡像其在原始區域中的設定。 (只有在正常作業期間相應減少復原環境中的集區或複寫資料庫以降低成本時，才需要執行此工作)。

1. 啟用復原區域中的 Web 應用程式的流量管理員端點。 啟用此端點可讓應用程式佈建至新的租用戶。 在此階段中，現有的租用戶仍處於離線狀態。

1. 提交批次要求，以強制依照優先順序容錯移轉資料庫。
    * 批次會進行組織，讓資料庫以平行方式在所有集區間容錯移轉。
    * 容錯移轉要求會使用非同步作業提交，以利快速提交並同時處理多個要求。

   > [!Note]
   > 發生中斷的情況時，原始區域中的主要資料庫會離線。  對次要資料庫強制執行容錯移轉，會中斷主要資料庫的連線，且不會嘗試套用任何已排入佇列的其餘交易。 在類似於本教學課程的 DR 演練案例中，如果在容錯移轉期間發生任何更新活動，則可能會遺失部分資料。 隨後在回復期間，當您將復原區域中的資料庫重新容錯移轉至原始區域時，將使用一般容錯移轉來確保不會遺失任何資料。

1. 監視 SQL 資料庫服務以判斷資料庫在何時進行容錯移轉。 租用戶資料庫在容錯移轉後，會更新目錄以記錄租用戶資料庫的復原狀態，並將租用戶標示為「已上線」。
    * 租用戶資料庫在目錄中標示為「已上線」後，即可供應用程式存取。
    * 租用戶資料庫中的 rowversion 值的總和會儲存在目錄中。 此值會作為指紋，據以允許回復程序判斷資料庫是否已在復原區域中更新。

### <a name="run-the-script-to-fail-over-to-the-recovery-region"></a>執行容錯移轉至復原區域的指令碼

現在，請假設應用程式部署所在的區域發生中斷情況，並執行復原指令碼：

1. 在 *PowerShell ISE* 中，開啟 ...\Learning Modules\Business Continuity and Disaster Recovery\DR-FailoverToReplica\Demo-FailoverToReplica.ps1 指令碼，並設定下列值：
    * **$DemoScenario = 3**，藉由容錯移轉至複本，將應用程式復原到復原區域中

2. 按 **F5** 以執行指令碼。  
    * 此指令碼會在新的 PowerShell 視窗中開啟，然後啟動一系列平行執行的 PowerShell 作業。 這些作業會將租用戶資料庫容錯移轉至復原區域。
    * 復原區域是與您的應用程式部署所在的 Azure 區域相關聯的_配對區域_。 如需詳細資訊，請參閱 [Azure 配對區域](https://docs.microsoft.com/azure/best-practices-availability-paired-regions)。 

3. 在 PowerShell 視窗中監視復原程序的狀態。
    ![容錯移轉程序](media/saas-dbpertenant-dr-geo-replication/failover-process.png)

> [!Note]
> 若要瀏覽復原作業的程式碼，請檢閱 ...\Learning Modules\Business Continuity and Disaster Recovery\DR-FailoverToReplica\RecoveryJobs 資料夾中的 PowerShell 指令碼。

### <a name="review-the-application-state-during-recovery"></a>查看復原期間的應用程式狀態
當應用程式端點在流量管理員中停用時，應用程式無法使用。 當目錄容錯移轉至復原區域，且所有租用戶皆標示為離線後，應用程式即會重新上線。 雖然應用程式可供使用，但每個租用戶在事件中樞內都會顯示為離線，直到其資料庫容錯移轉為止。 請務必設計您的應用程式以處理離線的租用戶資料庫。

1. 在復原目錄資料庫後，立即在網頁瀏覽器中重新整理 Wingtip Tickets 事件中樞。
    * 在頁尾中，請留意目錄伺服器名稱此時具有 _-recovery_ 尾碼，且位於復原區域中。
    * 留意到尚未還原的租用戶會標示為離線，且無法選取。  

    > [!Note]
    > 如果只有幾個資料庫要復原，您可能無法在復原完成前重新整理瀏覽器，因此您在租用戶處於離線狀態時可能不會看見租用戶。 
 
    ![事件中樞離線](media/saas-dbpertenant-dr-geo-replication/events-hub-offlinemode.png) 

    * 如果您直接開啟離線租用戶的 [事件] 頁面，頁面上會顯示「租用戶離線」通知。 例如，在 Contoso Concert Hall 處於離線狀態時，嘗試開啟 http://events.wingtip-dpt.&ltuser&gt;.trafficmanager.net/contosoconcerthall ![Contoso 離線頁面](media/saas-dbpertenant-dr-geo-replication/dr-in-progress-offline-contosoconcerthall.png) 

### <a name="provision-a-new-tenant-in-the-recovery-region"></a>在復原區域中佈建新租用戶
即使在所有現有的租用戶資料庫皆完成容錯移轉之前，您也可以在復原區域中佈建新的租用戶。  

1. 在 *PowerShell ISE* 中，開啟 ...\Learning Modules\Business Continuity and Disaster Recovery\DR-FailoverToReplica\Demo-FailoverToReplica.ps1 指令碼，並設定下列屬性：
    * **$DemoScenario = 4**，在復原區域中佈建新租用戶

2. 按 **F5** 以執行指令碼，並佈建新的租用戶。 

3. 完成時，會在瀏覽器中開啟 Hawthorn Hall 事件頁面。 請注意，在頁尾中，Hawthorn Hall 資料庫已佈建於復原區域中。
    ![Hawthorn Hall 事件頁面](media/saas-dbpertenant-dr-geo-replication/hawthornhallevents.png) 

4. 在瀏覽器中重新整理 Wingtip Tickets 事件中樞頁面，以確認 Hawthorn Hall 包含在內。 
    * 如果您在其他租用戶還原之前即佈建 Hawthorn Hall，其他租用戶可能仍處於離線狀態。


## <a name="review-the-recovered-state-of-the-application"></a>檢閱應用程式的復原狀態

復原程序完成時，應用程式和所有租用戶均可在復原區域中正常運作。 

1. 在 PowerShell 主控台視窗中的顯示畫面指出所有租用戶皆已復原後，請重新整理事件中樞。  租用戶全都會顯示為已上線，包括新的租用戶 Hawthorn Hall 在內。

    ![事件中樞內已復原和新的租用戶](media/saas-dbpertenant-dr-geo-replication/events-hub-with-hawthorn-hall.png)

2. 在 [Azure 入口網站](https://portal.azure.com)中，開啟資源群組的清單。  
    * 請查看您已部署的資源群組，以及具有 _-recovery_ 尾碼的復原資源群組。  復原資源群組中包含所有在復原程序期間建立的資源，以及在中斷期間建立的新資源。  

3. 開啟復原資源群組，並請查看下列項目：
    * 目錄的復原版本和 tenants1 伺服器 (具有 _-recovery_ 尾碼)。  這些伺服器上已還原的目錄和租用戶資料庫都具有在原始區域中使用的名稱。

    * _tenants2-dpt-&lt;user&gt;-recovery_ SQL 伺服器。  這是在中斷期間用來佈建新租用戶的伺服器。
    *   名為 _events-wingtip-dpt-&lt;recoveryregion&gt;-&lt;user&gt_; 的 App Service，這是事件應用程式的復原執行個體。 

    ![Azure 復原資源 ](media/saas-dbpertenant-dr-geo-replication/resources-in-recovery-region.png)    
    
4. 開啟 _tenants2-dpt-&lt;user&gt;-recovery_ SQL 伺服器。  請留意其中包含資料庫 _hawthornhall_ 和彈性集區 _Pool1_。  _Hawthornhall_ 資料庫設定為 _Pool1_ 彈性集區中的彈性資料庫。

5. 往回瀏覽至資源群組，並按一下 _tenants1-dpt-&lt;user&gt;-recovery_ 伺服器上的 Contoso Concert Hall 資料庫。 按一下左側的 [異地複寫]。
    
    ![容錯移轉後的 Contoso 資料庫](media/saas-dbpertenant-dr-geo-replication/contoso-geo-replication-after-failover.png)

## <a name="change-tenant-data"></a>變更租用戶資料 
在此工作中，您會更新其中一個租用戶資料庫。 

1. 在瀏覽器中找出 Contoso Concert Hall 的事件清單，並查看最後一個事件名稱。
2. 在 *PowerShell ISE* 中，進入 ...\Learning Modules\Business Continuity and Disaster Recovery\DR-FailoverToReplica\Demo-FailoverToReplica.ps1 指令碼，並設定下列值：
    * **$DemoScenario = 5**，從復原區域的租用戶中刪除事件
3. 按 **F5** 以執行指令碼
4. 重新整理 Contoso Concert Hall 事件頁面 (http://events.wingtip-dpt.&lt;user&gt;.trafficmanager.net/contosoconcerthall - 請將 &lt;user&gt; 替換為部署的使用者值)，並留意系統已刪除最後一個事件。

## <a name="repatriate-the-application-to-its-original-production-region"></a>將應用程式回復至其原始生產區域

此工作會將應用程式回復至其原始區域。 在實際案例中，您會在中斷的問題解決後起始回復作業。

### <a name="repatriation-process-overview"></a>回復程序概觀

![回復架構](media/saas-dbpertenant-dr-geo-replication/repatriation-architecture.png)

回復程序：
1. 取消任何未處理或進行中的資料庫還原要求。
2. 更新 _newtenant_ 別名，以指向原始區域中的租用戶伺服器。 變更此別名可確保任何新租用戶的資料庫此時都會在原始區域中佈建。
3. 將任何已變更的租用戶資料植入原始區域中
4. 依優先順序容錯移轉租用戶資料庫。

容錯移轉會有效地將資料庫移至原始區域。 在資料庫容錯移轉期間，會有幾秒鐘的時間，任何開啟的連線都會被捨棄，且無法使用資料庫。 您應為應用程式撰寫重試邏輯，以確保它們能夠重新連線。  雖然此斷線很短暫且通常難以察覺，但您仍可選擇在非營業時間回復資料庫。 


### <a name="run-the-repatriation-script"></a>執行回復指令碼
現在，我們假設中斷的問題已解決，而要執行回復指令碼。

1. 在 *PowerShell ISE* 中，進入 ...\Learning Modules\Business Continuity and Disaster Recovery\DR-FailoverToReplica\Demo-FailoverToReplica.ps1 指令碼。

2. 確認目錄同步程序仍在其 PowerShell 執行個體中執行。  如有必要，請藉由下列設定加以重新啟動：
    * **$DemoScenario = 1**，開始將租用戶伺服器、集區和資料庫組態資訊同步處理至目錄中
    * 按 **F5** 以執行指令碼。

3.  然後，若要啟動回復程序，請設定：
    * **$DemoScenario = 6**，將應用程式回復至其原始區域
    * 按 **F5** 在新的 PowerShell 視窗中執行復原指令碼。  回復需要幾分鐘的時間，您可以在 PowerShell 視窗中加以監視。
    ![回復程序](media/saas-dbpertenant-dr-geo-replication/repatriation-process.png)

4. 執行指令碼時，重新整理 [事件中樞] 頁面 (http://events.wingtip-dpt.&lt;user&gt;.trafficmanager.net)
    * 留意到在這整個程序中，所有租用戶皆處於線上狀態，且可供存取。

5. 回復程序完成後，請重新整理事件中樞，並開啟 Hawthorn Hall 的事件頁面。 留意到此資料庫已回復至其原始區域。
    ![已回復的事件中樞](media/saas-dbpertenant-dr-geo-replication/events-hub-repatriated.png)


## <a name="designing-the-application-to-ensure-app-and-database-are-colocated"></a>設計應用程式以確保應用程式和資料庫的共置 
應用程式經過適當設計，因此應用程式的連線一律會來自租用戶資料庫所在相同區域的執行個體。 此設計可降低應用程式與資料庫之間的延遲。 此最佳化假設應用程式對資料庫的互動比使用者對應用程式的互動更為頻繁。  

在回復期間的某些時刻，租用戶資料庫可能分散於復原和原始區域間。 對於各個資料庫，應用程式會執行租用戶伺服器名稱的 DNS 查閱，以查閱資料庫所在的區域。 在 SQL Database 中，伺服器名稱會是別名。 使用別名的伺服器名稱會包含區域名稱。 應用程式若位於與資料庫不同的區域，則會重新導向至資料庫伺服器所在相同區域中的執行個體。  重新導向至資料庫所在之相同區域中的執行個體，可盡量縮短應用程式與資料庫之間的延遲。 

## <a name="next-steps"></a>後續步驟

在本教學課程中，您已了解如何：
> [!div classs="checklist"]

>* 將資料庫和彈性集區組態資訊同步至租用戶目錄中
>* 在替代區域中設定由應用程式、伺服器和集區組成的復原環境
>* 使用_異地複寫_將目錄和租用戶資料庫複寫至復原區域
>* 將應用程式以及目錄和租用戶資料庫容錯移轉至復原區域 
>* 在中斷問題解決後，將應用程式、目錄和租用戶資料庫容錯回復至原始區域

您可以在[商務持續性概觀](sql-database-business-continuity.md)文件中深入了解 Azure SQL 資料庫為了啟用商務持續性而提供的技術。

## <a name="additional-resources"></a>其他資源

* [其他以 Wingtip SaaS 應用程式為基礎的教學課程](https://docs.microsoft.com/azure/sql-database/sql-database-wtp-overview#sql-database-wingtip-saas-tutorials)
