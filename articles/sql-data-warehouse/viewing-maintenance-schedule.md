---
title: Azure 維護排程 (預覽版) | Microsoft Docs
description: 維護排程可讓客戶規劃必要排程維護事件，Azure SQL 資料倉儲會使用它們來推出新功能、升級與修補程式。
services: sql-data-warehouse
author: antvgski
manager: craigg
ms.service: sql-data-warehouse
ms.topic: conceptual
ms.component: design
ms.date: 10/06/2018
ms.author: anvang
ms.reviewer: igorstan
ms.openlocfilehash: 6fbf914c8035344d33e8d2739fb9d92d5757c3d1
ms.sourcegitcommit: ad08b2db50d63c8f550575d2e7bb9a0852efb12f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/26/2018
ms.locfileid: "47228056"
---
# <a name="viewing-a-maintenance-schedule"></a>檢視維護排程 

## <a name="portal"></a>入口網站

根據預設，所有新建立的 Azure SQL 資料倉儲執行個體都有 8 小時系統定義在部署期間套用的主要和次要維護時段，完成部署之後可以編輯這個時段。 如果沒有事先通知，就不會在指定的維護時段以外進行維護。
完成下列步驟以檢視已套用到您在入口網站中之資料倉儲的維護排程。

完成下列步驟以檢視已套用到您在入口網站中之資料倉儲的維護排程。
1.  登入 [Azure 入口網站](https://portal.azure.com/)。
2.  選取您要檢視的資料倉儲。 
3.  選取的 Azure SQL 資料倉儲將會在 [概觀] 刀鋒視窗上開啟。 套用至所選資料倉儲的維護排程將會顯示在維護排程 (預覽) 下方。

![[概觀] 刀鋒視窗](media/sql-data-warehouse-maintenance-scheduling/clear-overview-blade.PNG)

## <a name="next-steps"></a>後續步驟
[深入了解](https://docs.microsoft.com/azure/monitoring-and-diagnostics/monitor-alerts-unified-usage)如何使用 Azure 監視器建立、檢視和管理警示。
[深入了解](https://docs.microsoft.com/azure/monitoring-and-diagnostics/monitor-alerts-unified-log-webhook)記錄警示規則的 Webhook 動作。
[深入了解](https://docs.microsoft.com/azure/service-health/service-health-overview) Azure 服務健康狀態


