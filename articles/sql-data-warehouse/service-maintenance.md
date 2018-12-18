---
title: 計劃性維護 - Azure SQL 資料倉儲 | Microsoft Docs
description: 了解如何為 Azure SQL 資料倉儲的計劃性維護事件做準備。
services: sql-data-warehouse
author: antvgski
manager: craigg
ms.service: sql-data-warehouse
ms.topic: conceptual
ms.component: manage
ms.date: 04/19/2018
ms.author: anvang
ms.reviewer: igorstan
ms.openlocfilehash: 8b5b94c959ae9820cab338f3ee3317fb3ee166d6
ms.sourcegitcommit: f94f84b870035140722e70cab29562e7990d35a3
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/30/2018
ms.locfileid: "43286596"
---
# <a name="planning-for-maintenance-on-your-azure-sql-data-warehouse"></a>規劃 Azure SQL 資料倉儲的維護

了解如何為 Azure SQL 資料倉儲的計劃性維護事件做準備。

## <a name="what-is-a-planned-maintenance-event"></a>什麼是計劃性維護事件？
計劃性維護事件是當資料倉儲需要離線進行維護作業時的時間範圍。 這些事件是套用服務升級、新功能或修補程式的機會。 

## <a name="frequency"></a>頻率
平均而言，每個月至少發生一次計劃性維護事件。 

## <a name="notifications-and-downtime"></a>通知和停機時間
您會在每個計劃性維護事件之前收到通知。 維護事件會取消所有正在執行的查詢，並且讓您的資料倉儲離線。 每個資料倉儲的預期停機時間大約 30 分鐘。 您預計會在維護完成時收到通知。 

## <a name="setting-up-alerts"></a>設定警示

我們建議使用 [Azure 監視器](../monitoring-and-diagnostics/monitoring-activity-log-alerts-on-service-notifications.md)來設定計劃性維護記錄警示。 警示可協助您規劃必要的維護，以對您的業務造成最小影響。 

若要設定通知，請使用這些[記錄警示指示](../monitoring-and-diagnostics/monitoring-activity-log-alerts-on-service-notifications.md)。 

## <a name="next-steps"></a>後續步驟
如需有關監視的詳細資訊，請參閱[監視您的工作負載](sql-data-warehouse-manage-monitor.md)。
