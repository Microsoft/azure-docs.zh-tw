---
title: Azure 維護排程 (預覽版) | Microsoft Docs
description: 維護排程可讓客戶規劃必要排程維護事件，Azure SQL 資料倉儲會使用它們來推出新功能、升級與補充程式。
services: sql-data-warehouse
author: antvgski
manager: craigg
ms.service: sql-data-warehouse
ms.topic: conceptual
ms.component: design
ms.date: 10/07/2018
ms.author: anvang
ms.reviewer: igorstan
ms.openlocfilehash: a6eedc0bac7aab69a9138f4f63d0d9d802e74dfc
ms.sourcegitcommit: ad08b2db50d63c8f550575d2e7bb9a0852efb12f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/26/2018
ms.locfileid: "47228105"
---
# <a name="change-a-maintenance-schedule"></a>變更維護排程 

## <a name="portal"></a>入口網站
維護排程可隨時更新或變更。 不過，若選取的執行個體目前正進行維護週期作業，將儲存設定，而且只有在下次識別的維護期間才會變成有效。 [深入了解](https://docs.microsoft.com/azure/service-health/resource-health-overview)如何在作用中維護事件期間監視您的資料倉儲。 

在預覽版中，我們將會要求您在 7 天期間內選取兩個維護時段。 每個維護時段都介於 3 到 8 小時之間，而且目前 3 小時是最短的可用選項。 維護可在識別的維護時段內的任何時間發生，但將不會在未事先通知您的情況下在那些識別的時段內發生，而且當服務部署新的程式碼到您的資料倉儲時，您的連線將短暫中斷。 

## <a name="identifying-the-primary-and-secondary-windows"></a>識別主要與次要時段

主要與次要時段必須在不同的日期範圍內識別 (亦即，主要時段 (週二 – 週四)M，次要時段 (週六 – 週日)

完成下列步驟以變更已套用到您在入口網站中之資料倉儲的維護排程。
1.  登入 [Azure 入口網站](https://portal.azure.com/)。
2.  選取您要更新的資料倉儲。 頁面將在概觀刀鋒視窗上開啟。 
3.  維護排程設定頁面可透過按一下概觀刀鋒視窗中的 [維護排程 (預覽版)] 摘要連結來存取，或透過左邊 [資源] 功能表中的 [維護排程] 選項來存取。  

    ![概觀刀鋒視窗選項](media/sql-data-warehouse-maintenance-scheduling/maintenance-change-option.png)

4. 您可以使用頁面頂端的選項按鈕來識別您主要維護時段的慣用日期範圍。 此選取範圍會決定您的主要時段將在週間發生或在週末發生。 您的選取範圍將會相應更新下面下拉式清單的值。 在預覽期間，某些區域可能尚未支援可用日期選項的完整集合。 這些值將在未來幾個月內更新。

   ![維護設定刀鋒視窗](media/sql-data-warehouse-maintenance-scheduling/maintenance-settings-page.png)

5. 使用下面的日期、開始時間與時段下拉式清單，來選擇偏好的主要與次要維護時段。 刀鋒視窗底部的排程摘要將根據選取的下拉式清單值來更新。

#### <a name="dropdown-options"></a>下拉式清單選項
- 日期：想要在選取的時段內執行維護的偏好日期。
- 開始時間：偏好的維護時段開始時間。
- 時段：時段的偏好期間。

  選取偏好的維護時段之後，請按一下 [儲存]。 將出現確認訊息供您確認您的新排程未啟用。 若將排程儲存在還不支援維護排程的區域中，則會出現下列訊息。 將儲存您的設定，而且當功能在您選取的區域中可用之後變成有效。    

    ![在區域中無效的快顯通知](media/sql-data-warehouse-maintenance-scheduling/maintenance-notactive-toast.png)

## <a name="next-steps"></a>後續步驟
- [深入了解](https://docs.microsoft.com/azure/monitoring-and-diagnostics/monitor-alerts-unified-log-webhook) 記錄警示規則的 Webhook 動作。
- [深入了解](https://docs.microsoft.com/azure/service-health/service-health-overview) Azure 服務健康狀態


