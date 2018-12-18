---
title: 在多租用戶 SaaS 應用程式中還原 Azure SQL 資料庫 | Microsoft Docs
description: 了解如何在不小心刪除資料之後還原單一租用戶 SQL 資料庫
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
ms.date: 04/01/2018
ms.openlocfilehash: 228f5135165cbf8806516e5e932f210586013402
ms.sourcegitcommit: 715813af8cde40407bd3332dd922a918de46a91a
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "47056738"
---
# <a name="restore-a-single-tenant-with-a-database-per-tenant-saas-application"></a>使用每一租用戶一個資料庫的 SaaS 應用程式還原單一租用戶

透過每一租用戶一個資料庫的模型，可讓您輕鬆將單一租用戶還原至先前的時間點，而不會影響其他租用戶。

在本教學課程中，您將了解兩個資料復原模式：

> [!div class="checklist"]

> * 將資料庫還原到平行資料庫 (並存)。
> * 就地還原資料庫，取代現有的資料庫。


|||
|:--|:--|
| 還原至平行資料庫 | 此模式適用於諸如檢閱、稽核及合規性等工作，可允許租用戶從較早時間點檢查他們的資料。 租用戶目前的資料庫仍保持連線而不變。 |
| 就地還原 | 此模式通常是在租用戶不小心刪除或損毀資料後，用來將租用戶復原到較早的時間點。 原始資料庫將會離線，然後被還原的資料庫取代。 |
|||

若要完成本教學課程，請確定已完成下列必要條件：

* 已部署 Wingtip SaaS 應用程式。 若要在 5 分鐘內完成部署，請參閱[部署及探索 Wingtip SaaS 應用程式](saas-dbpertenant-get-started-deploy.md)。
* 已安裝 Azure PowerShell。 如需詳細資料，請參閱[開始使用 Azure PowerShell](https://docs.microsoft.com/powershell/azure/get-started-azureps)。

## <a name="introduction-to-the-saas-tenant-restore-patterns"></a>SaaS 租用戶還原模式簡介

有兩個可還原個別租用戶資料的簡單模式。 由於租用戶資料庫彼此互相隔離，因此還原一個租用戶並不會影響到任何其他租用戶的資料。 Azure SQL Database 的還原時間點 (PITR) 功能可在這兩種模式中使用。 PITR 一律會建立新的資料庫。   

* **平行還原**：在第一個模式中，系統會在租用戶目前的資料庫旁建立新的平行資料庫。 然後租用戶會有此已還原資料庫的唯讀存取權。 您可以檢閱已還原的資料，也可能可以用它來覆寫目前的資料值。 這取決於應用程式設計者決定如何讓用戶端存取還原的資料庫，以及提供哪些復原選項。 在某些情況下，可能只需要允許租用戶檢閱較早時間點的資料。 

* **就地還原**：如果資料已遺失或損毀，而租用戶需要還原到較早的時間點，那麼第二個模式就十分實用。 還原資料庫時，租用戶採離線狀態。 原始資料庫會遭到刪除，並重新命名已還原的資料庫。 刪除原始資料庫後，您仍可存取該資料庫的備份鏈結，讓您可以將資料庫還原至更早的時間點 (如有需要)。

如果資料庫使用[異地複寫](sql-database-geo-replication-overview.md)且採用平行還原，建議您從已還原的複本將任何所需的資料複製到原始資料庫。 如果您以已還原的資料庫取代原始資料庫，則必須重新設定並重新同步處理異地複寫。

## <a name="get-the-wingtip-tickets-saas-database-per-tenant-application-scripts"></a>取得每一租用戶一個資料庫的 Wingtip Tickets SaaS 應用程式指令碼

可在 [WingtipTicketsSaaS-DbPerTenant](https://github.com/Microsoft/WingtipTicketsSaaS-DbPerTenant) GitHub 存放庫中使用 Wingtip Tickets SaaS 多租用戶資料庫指令碼和應用程式來源程式碼。 如需下載和解除封鎖 Wingtip Tickets SaaS 指令碼的步驟，請參閱[一般指引](saas-tenancy-wingtip-app-guidance-tips.md)。

## <a name="before-you-start"></a>開始之前

資料庫建立完成後，可能需要 10 到 15 分鐘來建立還原要用的第一次完整備份。 如果您才剛安裝好應用程式，可能需要等待數分鐘，才能嘗試此案例。

## <a name="simulate-a-tenant-accidentally-deleting-data"></a>模擬租用戶不小心刪除資料的情況

為了示範這些復原情況，請先「不小心」刪除其中一個租用戶資料庫中的事件。 

### <a name="open-the-events-app-to-review-the-current-events"></a>開啟事件應用程式以檢閱目前的事件

1. 開啟 [事件中樞] \(http://events.wtp.&lt;user&gt;.trafficmanager.net)，然後選取 [Contoso Concert Hall]。

   ![事件中樞](media/saas-dbpertenant-restore-single-tenant/events-hub.png)

2. 捲動事件清單，然後記下清單中的最後一個事件。

   ![顯示最後一個事件](media/saas-dbpertenant-restore-single-tenant/last-event.png)


### <a name="accidentally-delete-the-last-event"></a>「不小心」刪除最後一個事件

1. 在 PowerShell ISE 中開啟 ...\\Learning Modules\\Business Continuity and Disaster Recovery\\RestoreTenant\\*Demo-RestoreTenant.ps1*，然後設定下列值：

   * **$DemoScenario** = **1**，*刪除最後一個事件 (沒有銷售門票)*。
2. 按 F5 以執行指令碼，並刪除最後一個事件。 隨即出現下列確認訊息：

   ```Console
   Deleting last unsold event from Contoso Concert Hall ...
   Deleted event 'Seriously Strauss' from Contoso Concert Hall venue.
   ```

3. Contoso 事件頁面隨即開啟。 請向下捲動並確認該事件已消失。 如果該事件仍在清單中，請選取 [重新整理] 並確認它已消失。

   ![移除的最後一個事件](media/saas-dbpertenant-restore-single-tenant/last-event-deleted.png)


## <a name="restore-a-tenant-database-in-parallel-with-the-production-database"></a>將租用戶資料庫以與生產環境資料戶平行的方式還原

此練習會將 Contoso Concert Hall 資料庫還原到刪除事件前的時間點。 此案例假設您需要在平行資料庫中檢閱已刪除的資料。

 Restore-TenantInParallel.ps1 指令碼會建立一個名為 ContosoConcertHall\_old 的平行租用戶資料庫，以及一個平行目錄項目。 此復原模式最適用於從較不嚴重的資料遺失進行復原。 如果您需要就合規性或稽核用途檢閱資料，也可以使用此模式。 當您使用的是[異地複寫](sql-database-geo-replication-overview.md)時，這是建議採用的方法。

1. 完成[模擬租用戶不小心刪除資料的情況](#simulate-a-tenant-accidentally-deleting-data)一節。
2. 在 PowerShell ISE 中開啟 ...\\Learning Modules\\Business Continuity and Disaster Recovery\\RestoreTenant\\_Demo-RestoreTenant.ps1_。
3. 設定 **$DemoScenario** = **2**，平行還原租用戶。
4. 若要執行指令碼，請按 F5。

此指令碼會將租用戶資料庫還原到刪除事件前的時間點。 資料庫會還原至名為 _ContosoConcertHall\_old_ 的新資料庫。 系統會將存在於還原資料庫中的目錄中繼資料刪除，並使用從 ContosoConcertHall\_old 名稱建構的索引鍵將資料庫新增至目錄。

此示範指令碼會在瀏覽器中開啟此租用戶資料庫的事件頁面。 請注意 URL：```http://events.wingtip-dpt.&lt;user&gt;.trafficmanager.net/contosoconcerthall_old```，此頁面顯示的資料來自所還原的資料庫，其中在名稱新增了 _old。

請捲動瀏覽器中列出的事件，以確認在上一節中刪除的事件已經還原。

將已還原的租用戶公開作為額外的租用戶 (具有自己的「事件」應用程式)，不太可能是您將所還原資料的存取權提供給租用戶的方式。 它可用來說明還原模式。 一般而言，您可以提供舊資料的唯讀存取權，以及只依據一段定義的期間來保留已還原的資料庫。 在此範例中，在您完成作業之後，即可執行_移除已還原的租用戶_案例來刪除已還原的租用戶項目。

1. 設定 **$DemoScenario** = **4**，「移除已還原的租用戶」。
2. 若要執行指令碼，請按 F5。
3. *ContosoConcertHall\_old* 項目現在已從目錄中刪除。 請在您的瀏覽器中關閉此租用戶的事件頁面。


## <a name="restore-a-tenant-in-place-replacing-the-existing-tenant-database"></a>就地還原租用戶，取代現有的租用戶資料庫

此練習會將 Contoso Concert Hall 租用戶還原到刪除事件前的時間點。 Restore-TenantInPlace 指令碼會將租用戶資料庫還原到新的資料庫，並刪除原始資料庫。 此復原模式最適用於從嚴重的資料損毀進行復原，以及租用戶可能必須對重大資料遺失有所考慮的情況。

1. 在 PowerShell ISE 中開啟 **Demo-RestoreTenant.ps1** 檔案。
2. 設定 **$DemoScenario** = **5**，就地還原租用戶。
3. 若要執行指令碼，請按 F5。

此指令碼會將租用戶資料庫還原到刪除事件前的時間點。 其會先使 Contoso Concert Hall 租用戶離線，以避免有進一步的更新。 然後，藉由從還原點進行還原，來平行建立資料庫。 已還原的資料庫會以時間戳記命名，確保資料庫名稱不會與現有的租用戶資料庫名稱衝突。 接著，會刪除舊的租用戶資料庫，並將已還原的資料庫重新命名成原始資料庫名稱。 最後，會讓 Contoso Concert Hall 上線以允許應用程式存取已還原的資料庫。

您已成功地將資料庫還原到刪除事件前的時間點。 開啟 [事件] 頁面時，請確認最後一個事件已還原。

在還原資料庫後，還需要 10 到 15 分鐘的時間進行第一次完整備份，以用於再次還原。 

## <a name="next-steps"></a>後續步驟

在本教學課程中，您已了解如何：

> [!div class="checklist"]

> * 將資料庫還原到平行資料庫 (並存)。
> * 就地還原資料庫。

嘗試[管理租用戶資料庫結構描述](saas-tenancy-schema-management.md)教學課程。

## <a name="additional-resources"></a>其他資源

* [其他以 Wingtip SaaS 應用程式為基礎的教學課程](saas-dbpertenant-wingtip-app-overview.md#sql-database-wingtip-saas-tutorials)
* [使用 Azure SQL Database 的商務持續性概觀](sql-database-business-continuity.md)
* [了解 SQL Database 備份](sql-database-automated-backups.md)
