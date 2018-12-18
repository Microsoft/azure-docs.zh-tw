---
title: 將 Azure 安全性資料匯出至 SIEM - 管線設定 [預覽] | Microsoft Docs
description: 本文記載將 Azure 資訊安全中心記錄移至 SIEM 的程序
services: security-center
documentationcenter: na
author: Barclayn
manager: MBaldwin
editor: ''
ms.assetid: ''
ms.service: security-center
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 02/01/2018
ms.author: barclayn
ms.openlocfilehash: aede60a729fe9c0594ded485e189c0b467e34271
ms.sourcegitcommit: f3bd5c17a3a189f144008faf1acb9fabc5bc9ab7
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/10/2018
ms.locfileid: "44298228"
---
# <a name="azure-security-data-export-to-siem--pipeline-configuration-preview"></a>將 Azure 安全性資料匯出至 SIEM - 管線設定 [預覽]

本文件詳述將 Azure 資訊安全中心的安全性資料匯出到至 SIEM 的程序。

系統會將由 Azure 資訊安全中心所產生的已處理事件發行到 Azure [活動記錄](../monitoring-and-diagnostics/monitoring-overview-activity-logs.md)，此為可透過 Azure 監視器取得的記錄類型之一。 Azure 監視器提供合併的管線，將您的所有監視資料路由傳送至 SIEM 工具。 完成此動作的方式是將該資料串流到事件中樞，然後將它提取至合作夥伴工具。

這個管道會使用 [Azure 監視單一管線](../monitoring-and-diagnostics/monitor-stream-monitoring-data-event-hubs.md)，從您的 Azure 環境存取監視資料。 這可讓您輕鬆地設定 SIEM 和監視工具來取用資料。

以下幾節說明如何設定資料以串流到事件中樞。 這些步驟假設您已經在 Azure 訂用帳戶中設定 Azure 資訊安全中心。

高階概述

![高階概述](media/security-center-export-data-to-siem/overview.png)

## <a name="what-is-the-azure-security-data-exposed-to-siem"></a>公開至 SIEM 的 Azure 安全性資料為何？

在此預覽版本中，我們會公開[安全性警示](../security-center/security-center-managing-and-responding-alerts.md)。 在即將發行版本中，我們將使用安全性建議來擴充此資料集。

## <a name="how-to-setup-the-pipeline"></a>如何設定管線？ 

### <a name="create-an-event-hub"></a>建立事件中心 

開始之前，您必須[建立事件中樞命名空間](../event-hubs/event-hubs-create.md)。 此命名空間和事件中樞是所有監視資料的目的地。

### <a name="stream-the-azure-activity-log-to-event-hubs"></a>將 Azure 活動記錄檔串流至事件中樞

請參閱[將活動記錄串流至事件中樞](../monitoring-and-diagnostics/monitoring-stream-activity-logs-event-hubs.md)一文

### <a name="install-a-partner-siem-connector"></a>安裝合作夥伴 SIEM 連接器 

使用 Azure 監視器將監視資料路由傳送到事件中樞，可讓您輕鬆地與合作夥伴 SIEM 和監視工具整合。

請參閱下列連結，以查看[支援的 SIEM](../monitoring-and-diagnostics/monitor-stream-monitoring-data-event-hubs.md#what-can-i-do-with-the-monitoring-data-being-sent-to-my-event-hub) 清單

## <a name="example-for-querying-data"></a>查詢資料範例 

以下是幾個可讓您用來提取警示資料的 Splunk 查詢：

| **查詢描述**                                | **查詢**                                                                                                                              |
|---------------------------------------------------------|----------------------------------------------------------------------------------------------------------------------------------------|
| 所有警示                                              | index=main Microsoft.Security/locations/alerts                                                                                         |
| 依作業名稱對作業計數進行總結             | index=main sourcetype="amal:security" \| table operationName \| stats count by operationName                                |
| 取得警示資訊：時間、名稱、狀態、識別碼和訂用帳戶 | index=main Microsoft.Security/locations/alerts \| table \_time, properties.eventName, State, properties.operationId, am_subscriptionId |


## <a name="next-steps"></a>後續步驟

- [支援的 SIEM](../monitoring-and-diagnostics/monitor-stream-monitoring-data-event-hubs.md#what-can-i-do-with-the-monitoring-data-being-sent-to-my-event-hub)
- [將活動記錄串流至事件中樞](../monitoring-and-diagnostics/monitoring-stream-activity-logs-event-hubs.md)
- [安全性警示](../security-center/security-center-managing-and-responding-alerts.md)。