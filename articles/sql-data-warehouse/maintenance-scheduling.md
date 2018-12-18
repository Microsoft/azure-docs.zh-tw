---
title: Azure 維護排程 (預覽版) | Microsoft Docs
description: 維護排程可讓客戶規劃必要排程維護事件，Azure SQL 資料倉儲會使用它們來推出新功能、升級與修補程式。
services: sql-data-warehouse
author: antvgski
manager: craigg
ms.service: sql-data-warehouse
ms.topic: conceptual
ms.component: design
ms.date: 09/20/2018
ms.author: anvang
ms.reviewer: igorstan
ms.openlocfilehash: 85e8327d1cac17059b2e91b1ea21becc23002c8e
ms.sourcegitcommit: ad08b2db50d63c8f550575d2e7bb9a0852efb12f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/26/2018
ms.locfileid: "47228120"
---
# <a name="using-maintenance-schedules-to-manage-service-updates-and-maintenance"></a>使用維護排程管理服務更新和維護

Azure SQL 資料倉儲維護排程現在為預覽版。 此個新功能可以無縫整合服務健康狀態計劃性維護通知、資源健康狀態檢查監視器，以及 Azure SQL 資料倉儲維護排程服務。

維護排程可讓您排程一個時段，方便您接收新的功能、升級與修補程式。 客戶將在 7 天的期限內選取一個主要和一個次要的維護時段，亦即星期六 22:00 – 星期日 01:00 (主要) 以及星期三 19:00 – 22:00 (次要)。 如果我們無法在主要維護時段執行維護工作，將會在次要維護時段再嘗試一次。

所有新建立的 Azure SQL 資料倉儲執行個體在部署期間都會套用一個系統定義的維護排程。 一旦部署完畢，就可以編輯排程。

每個維護時段都介於 3 到 8 小時之間，而且目前 3 小時是最短的可用選項。 維護可能會在該時段內的任何時間進行，而且您的連線可能會短暫中斷，因為服務會將新的程式碼部署到您的資料倉儲。 

在功能預覽期間，我們會要求客戶在個別的日期範圍內，找出其主要和次要時段。   
所有維護作業都應該在排程的維護時段內完成，而且如果沒有事先通知，就不會在指定的維護時段以外進行維護。 如果您的資料倉儲已在排程維護期間暫停，它將會在繼續作業期間更新。  


## <a name="alerts-and-monitoring"></a>警示和監視

服務健康狀態通知與資源健康狀態檢查監視器無縫整合可讓客戶隨時掌握即將發生的維護活動。 新的自動化會充分利用 Azure 監視器，並可讓客戶決定他們想要收到即將發生的維護事件通知的方式，與應該觸發哪些自動化流程來管理停機時間，並將其作業的影響降到最低。

所有維護事件都會在 24 小時前預先通知。 若要將執行個體的停機時間降至最低，您應該在開始您所選擇的維護期間之前，確定資料倉儲上沒有長時間執行的交易。 維護開始時，將會取消所有作用中的工作階段、回復未認可的交易，而且您的資料倉儲將會經歷短暫的連線中斷。 您將會在資料倉儲的維護工作完成之後，立即收到通知。 

如果您收到維護將會進行的預先通知，但我們無法在這段期間執行維護工作，您會收到取消通知。 之後，維護將會在下一個排程維護期間繼續。
 
所有作用中的維護事件都會顯示在 [服務健康狀態 - 計劃性維護] 區段中。 過去事件的完整記錄將會保留為服務健康狀態歷程記錄的一部分。 在作用中的事件期間，維護可以透過 Azure 服務健康狀態檢查入口網站儀表板受到監視。

### <a name="maintenance-schedule-availability"></a>維護排程可用性

即使維護排程尚無法在您所選的區域中使用，但是您仍然可以隨時檢視和編輯維護排程。 當您的區域可以使用維護排程時，識別的排程將會立即在資料倉儲上變成作用中。


## <a name="next-steps"></a>後續步驟

- [深入了解](viewing-maintenance-schedule.md)如何檢視維護排程 
- [深入了解](changing-maintenance-schedule.md)如何變更維護排程
- [深入了解](https://docs.microsoft.com/azure/monitoring-and-diagnostics/monitor-alerts-unified-usage)如何使用 Azure 監視器建立、檢視和管理警示
- [深入了解](https://docs.microsoft.com/azure/monitoring-and-diagnostics/monitor-alerts-unified-log-webhook)記錄警示規則的 Webhook 動作
- [深入了解](https://docs.microsoft.com/azure/service-health/service-health-overview) Azure 服務健康狀態







